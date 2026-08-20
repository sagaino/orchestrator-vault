---
title: "Declarative Multi-Repository Unit-of-Work Transaction Coordinator"
type: pattern
tags: [pattern, backend, golang, database, transaction, gorm, unit-of-work]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787109509918-5a621114
sources: ["Harvest 1787109509918 5a621114.json"]
---

# Declarative Multi-Repository Unit-of-Work Transaction Coordinator

Declarative Multi-Repository Unit-of-Work Transaction Coordinator managing GORM transactions across distinct repositories.

## 1. Overview & Architecture

Declarative unit-of-work transaction manager that coordinates multiple repositories within an atomic GORM transaction.

## 2. Implementation & Code Structure

pkg/db/dbTransaction.go provides DBTransaction struct and BaseRepository interface.

## 3. Key Implementation Points

- Decoupled transaction management from business usecase layer
- Transparent DB session switching across multiple participating repositories
- Atomic rollback or commit via unified End(err) method

## 4. Code Examples

### Multi-repository unit-of-work transaction coordinator in Go GORM

```go
package db

import (
	"gorm.io/gorm"
)

type BaseRepository interface {
	SetupConnection(db *gorm.DB)
}

type DBTransaction struct {
	db    *gorm.DB
	repos []BaseRepository
}

func NewDBTransaction(db *gorm.DB, repos ...BaseRepository) DBTransaction {
	result := DBTransaction{
		db:    db,
		repos: make([]BaseRepository, 0),
	}
	for _, repo := range repos {
		result.repos = append(result.repos, repo)
	}
	return result
}

func (main *DBTransaction) Begin() {
	tx := main.db.Begin()
	for _, rp := range main.repos {
		rp.SetupConnection(tx)
	}
	main.db = tx
}

func (main *DBTransaction) End(err error) error {
	if err != nil {
		if errTrx := main.db.Rollback().Error; errTrx != nil {
			return errTrx
		}
		return nil
	}
	return main.db.Commit().Error
}

```

## 5. Considerations & Best Practices

- Repositories must implement BaseRepository.SetupConnection to swap the active GORM session instance
- Always handle deferred rollback/commit with defer trx.End(err) to prevent hanging transactions

## 6. Related Knowledge

- [[01-Knowledge/patterns/backend/go-gorm-generic-repository-dynamic-expression-builder-pattern.md]]

## 7. Source

- Harvest 1787109509918 5a621114.json
