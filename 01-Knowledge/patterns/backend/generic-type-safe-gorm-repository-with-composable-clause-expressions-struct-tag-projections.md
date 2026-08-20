---
title: "Generic Type-Safe GORM Repository with Composable Clause Expressions & Struct Tag Projections"
type: pattern
tags: [pattern, backend, golang, gorm, generic-repository, sql-clauses, ast-projection]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787128418632-596cfbe7
sources: ["Harvest 1787128418632 596cfbe7.json"]
---

# Generic Type-Safe GORM Repository with Composable Clause Expressions & Struct Tag Projections

Type-safe generic GORM repository pattern with composable clause expression builders and reflection-driven AST column projection.

## 1. Overview & Architecture

A robust generic repository implementation in Go utilizing GORM and Go 1.18+ type parameters [T schema.Tabler]. It abstracts boilerplate CRUD operations while providing powerful composable clause expressions for complex dynamic filters, joins, conditioned preloading, and struct-tag AST column projections.

## 2. Implementation & Code Structure

pkg/db/
├── generic_repository.go    # Generic repository implementation, clause helpers, AST projection
├── dto.go                   # PaginationQuery and repository DTOs
└── default.go               # GORM database connection initialization

## 3. Key Implementation Points

- Parameterized generic repository definition GenericRepository[T schema.Tabler] with context propagation.
- Composable clause.Expression helper functions (Search, Equal, ExpressionDateRange, InArray, NotInArray).
- Unified FindPagedByExpression and FindPagedByExpressionAndPreloadConditioned with automated count tracking.
- Reflection-based column projection (extractSelection) for optimized payload retrieval without overhead.

## 4. Code Examples

### GenericRepository struct definition and composable GORM clause expression helpers.

```go
type GenericRepository[T schema.Tabler] struct {
	db    *gorm.DB
	model T
}

func NewGenericeRepoPointr[T schema.Tabler](db *gorm.DB, model T) *GenericRepository[T] {
	return &GenericRepository[T]{
		db:    db,
		model: model,
	}
}

var (
	Search = func(val string, col ...string) clause.Expression {
		var exps = make([]clause.Expression, len(col))
		for i, c := range col {
			tableName, colName := getColNameStr(c)
			exps[i] = clause.Like{
				Column: clause.Column{Name: colName, Table: tableName},
				Value:  "%" + val + "%",
			}
		}
		return clause.Or(exps...)
	}

	Equal = func(val interface{}, col string) clause.Expression {
		tableName, colName := getColNameStr(col)
		return clause.Eq{
			Column: clause.Column{Name: colName, Table: tableName},
			Value:  val,
		}
	}
)
```

### Paginated query with dynamic clause expressions, conditional preloads, and joined associations.

```go
func (repo GenericRepository[T]) FindPagedByExpressionAndPreloadConditioned(
	ctx context.Context,
	cond []clause.Expression,
	paginate PaginationQuery,
	joins []string,
	preload []PreloadWithCondition,
	expType int,
) ([]T, int, error) {
	var result []T
	var total int64

	db := repo.db.WithContext(ctx).Model(&result)
	db = repo.applyWhereClause(cond, expType, db)

	for _, j := range joins {
		db = db.Joins(j)
	}

	for _, s := range preload {
		if len(s.Args) == 0 {
			db = db.Preload(s.ColName)
		} else {
			db = db.Preload(s.ColName, s.Args...)
		}
	}

	if errCount := db.Count(&total).Error; errCount != nil {
		return nil, 0, errCount
	}

	offset := paginate.PerPage * (paginate.Page - 1)
	err := db.Offset(offset).Limit(paginate.PerPage).Find(&result).Error
	return result, int(total), err
}
```

### Runtime reflection extracting explicit SQL columns from struct GORM tags for selective query projection.

```go
func (repo GenericRepository[T]) extractSelection(entity interface{}) ([]string, error) {
	typeOf := reflect.TypeOf(entity)
	if typeOf.Kind() != reflect.Ptr {
		return nil, fmt.Errorf("should had parsing entity struct pointer to parameter")
	}

	var results = make([]string, 0)
	elem := typeOf.Elem()
	switch elem.Kind() {
	case reflect.Struct:
		results = repo.fromStruct(elem)
	case reflect.Array, reflect.Slice:
		results = repo.fromStruct(elem.Elem())
	}
	return results, nil
}

func (repo GenericRepository[T]) fromStruct(elem reflect.Type) []string {
	var results = make([]string, elem.NumField())
	for i := 0; i < elem.NumField(); i++ {
		field := elem.Field(i)
		gormTag := field.Tag.Get("gorm")
		var columnName string
		if strings.Contains(gormTag, "column:") {
			parts := strings.Split(gormTag, ";")
			for _, part := range parts {
				part = strings.TrimSpace(part)
				if strings.HasPrefix(part, "column:") {
					columnName = strings.TrimPrefix(part, "column:")
					break
				}
			}
		}
		if columnName != "" {
			results[i] = columnName
		}
	}
	return results
}
```

## 5. Considerations & Best Practices

- Ensures entities implement schema.Tabler interface so TableName() is reliably available.
- Reflection in extractSelection is cached or called per request; keep struct fields organized with explicit gorm:"column:name" tags.
- Avoid mixing raw unescaped SQL strings with GORM Clauses to maintain SQL injection resilience.

## 6. Related Knowledge

- Clean Architecture Skeleton
- Gorm Persistence Layer

## 7. Source

- Harvest 1787128418632 596cfbe7.json
