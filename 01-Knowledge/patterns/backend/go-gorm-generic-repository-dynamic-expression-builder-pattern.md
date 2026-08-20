---
title: "Go GORM Generic Repository & Dynamic Expression Builder Pattern"
type: pattern
tags: [pattern, backend, golang, gorm, generic, repository, database, orm]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787109509918-5a621114
sources: ["Harvest 1787109509918 5a621114.json"]
---

# Go GORM Generic Repository & Dynamic Expression Builder Pattern

Type-safe generic repository abstraction with dynamic GORM expression builders and reflection-driven column projections.

## 1. Overview & Architecture

Generic repository pattern for GORM leveraging Go 1.18+ type parameters to provide type-safe CRUD, dynamic expression building, and reflection-based selective field loading.

## 2. Implementation & Code Structure

pkg/db/generic_repository.go defines GenericRepository[T schema.Tabler], AST expression helpers (Search, Equal, InArray, ExpressionDateRange), and reflection struct extractors.

## 3. Key Implementation Points

- Go generics constraint on schema.Tabler interface ensuring compile-time safety
- Parameterized AST clause expressions to prevent SQL injection and enable composable queries
- Dynamic reflection projection via extractSelection to fetch only requested columns

## 4. Code Examples

### Type-safe generic repository implementation with dynamic reflection-based column selection in GORM

```go
package db

import (
	"context"
	"fmt"
	"reflect"
	"strings"
	"gorm.io/gorm"
	"gorm.io/gorm/clause"
	"gorm.io/gorm/schema"
)

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

func (repo GenericRepository[T]) FindOneByIDSelection(
	ctx context.Context,
	entity interface{},
	id interface{},
) error {
	selection, err := repo.extractSelection(entity)
	if err != nil {
		return err
	}

	return repo.db.WithContext(ctx).
		Table(repo.model.TableName()).
		Select(selection).
		First(entity, "id = ?", id).Error
}

```

### Reusable AST clause expression builders for composable and parameterized queries

```go
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

## 5. Considerations & Best Practices

- Requires struct tags with gorm:"column:..." for dynamic column projections
- All repository methods receive context.Context for cancellation and timeout propagation
- Clause expressions use parameter binding to prevent SQL injection

## 6. Related Knowledge

- [[01-Knowledge/patterns/backend/declarative-multi-repository-unit-of-work-transaction-coordinator.md]]
- [[01-Knowledge/patterns/backend/structured-domain-error-hierarchy-i18n-response-mapping-pattern.md]]

## 7. Source

- Harvest 1787109509918 5a621114.json
