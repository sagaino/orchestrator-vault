---
title: "Polyglot Subquery Synthesizer & Heterogeneous View Aggregator"
type: pattern
tags: [pattern, backend, golang, gorm, sql-synthesis, view-aggregation, union-query]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787128418632-596cfbe7
sources: ["Harvest 1787128418632 596cfbe7.json"]
---

# Polyglot Subquery Synthesizer & Heterogeneous View Aggregator

Polyglot subquery synthesis and view aggregation pattern unifying heterogeneous entity tables into a single paginated query stream.

## 1. Overview & Architecture

A query synthesis and view aggregation pattern that compiles modular GORM subqueries to SQL strings at runtime, binds dynamic conditional filters, unions disparate database tables into a single unified virtual projection, and executes parameterized sorting and pagination.

## 2. Implementation & Code Structure

iam_module/internal/adapter/repository/
├── user_repo.go             # Subquery compilation, UNION view synthesis, and pagination
└── dto.go                   # UserListQuery, Filter parameters

## 3. Key Implementation Points

- db.ToSQL() method extracting SQL statements from GORM query definitions without executing database round-trips.
- Dynamic schema-aware filter builder reusing query condition logic across heterogeneous models.
- Dynamic query routing supporting single-table and multi-table UNION ALL execution branches.
- Two-pass aggregation: derived table count query and paginated ordered result fetch.

## 4. Code Examples

### Compiling independent GORM subqueries via ToSQL and combining heterogeneous entities using UNION ALL.

```go
func (repo userRepo) UserDashboardList(ctx context.Context, query UserListQuery) ([]domain.UserListItem, int, int, error) {
	db := repo.db.WithContext(ctx)

	buildConditions := func(baseQuery *gorm.DB, tableName string, statusCol string) *gorm.DB {
		var status = 0
		if query.StatusKey != "" {
			if query.StatusKey == domain.Active {
				status = 1
			}
			baseQuery = baseQuery.Where(fmt.Sprintf("%s.%s = ?", tableName, statusCol), status)
		}
		if query.Filter.Search != "" {
			searchPattern := "%" + query.Filter.Search + "%"
			baseQuery = baseQuery.Where(
				fmt.Sprintf("%s.full_name LIKE ? OR %s.email LIKE ?", tableName, tableName),
				searchPattern, searchPattern)
		}
		return baseQuery
	}

	mobileSql := db.ToSQL(func(tx *gorm.DB) *gorm.DB {
		return buildConditions(
			tx.Model(&domain.User{}).
				Select("`users`.id, full_name as name, email, if(is_verified=1, 'active','inactive') as status, last_active, 'USER' as role_name"),
			domain.User{}.TableName(), "is_verified",
		).Where("is_verified = 1").Find(&[]domain.User{})
	})

	dashboardSql := db.ToSQL(func(tx *gorm.DB) *gorm.DB {
		return buildConditions(
			tx.Model(&domain.UserAdmin{}).
				Select("`user_admins`.id, full_name as name, email, if(is_active=1, 'active','inactive') as status, last_active, master_roles.name as role_name").
				Joins("left join master_roles on master_roles.id = user_admins.role_id"),
			domain.UserAdmin{}.TableName(), "is_active",
		).Find(&[]domain.UserAdmin{})
	})

	var finalQuery string
	switch query.RoleName {
	case constant.RoleIsUser:
		finalQuery = mobileSql
	case constant.RoleIsAdmin:
		finalQuery = dashboardSql
	default:
		finalQuery = fmt.Sprintf("(%s) UNION ALL (%s)", mobileSql, dashboardSql)
	}
...
```

### Executing parameterized total count and paginated query over the synthesized UNION query.

```go
	var total int64
	countSQL := fmt.Sprintf("SELECT COUNT(*) FROM (%s) as union_count", finalQuery)
	err := db.Raw(countSQL).Scan(&total).Error
	if err != nil {
		return nil, 0, 0, fmt.Errorf("failed to count results: %w", err)
	}

	offset := (query.Filter.Page - 1) * query.Filter.PerPage
	totalPages := int((total + int64(query.Filter.PerPage) - 1) / int64(query.Filter.PerPage))

	finalSQL := fmt.Sprintf(`
		SELECT * FROM (%s) as union_table 
		ORDER BY last_active DESC 
		LIMIT ? OFFSET ?
	`, finalQuery)

	var results []domain.UserListItem
	err = db.Raw(finalSQL, query.Filter.PerPage, offset).Scan(&results).Error
	if err != nil {
		return nil, 0, 0, fmt.Errorf("failed to execute union query: %w", err)
	}

	return results, int(total), totalPages, nil
}
```

## 5. Considerations & Best Practices

- Ensure column aliases and types in all subqueries match identically for valid SQL UNION execution.
- Use db.Raw with parameterized limit and offset arguments to prevent SQL injection during final query execution.
- Index underlying table columns (e.g., last_active, is_verified, is_active) to avoid slow derived table temporary files.

## 6. Related Knowledge

- Gorm Persistence Layer
- Generic Gorm Repository

## 7. Source

- Harvest 1787128418632 596cfbe7.json
