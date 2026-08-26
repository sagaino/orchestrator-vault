---
supersession: "ACTIVE"
review_by: "2027-02-22"
owner: "knowledge-curator"
confidence: 0.5
provenance_schema: 1
title: "Generic Type-Safe Repository with Atomic Transaction Coordinator"
type: pattern
tags: [pattern, backend, golang, repository-pattern, generics, database-transactions, gorm]
created: 2026-08-26
updated: 2026-08-26
orchestrator_run: "harvest-1787715147310-7bd85203"
sources: ["[[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]"]
---

# Generic Type-Safe Repository with Atomic Transaction Coordinator

Pola repository generik Go berbasis GORM yang dipadukan dengan DBTransaction coordinator untuk propagasi transaksi multi-repository.

## 1. Overview & Architecture

Pola ini mengombinasikan Type-Safe Generic Repository menggunakan Go Generics dan GORM dengan DBTransaction Coordinator. Pola ini mengeliminasi boilerplate query CRUD standar sekaligus menyediakan mekanisme transaksi database multi-tabel yang terisolasi dan konsisten.

## 2. Implementation & Code Structure

pkg/
└── db/
    ├── generic_repository.go    # Generic CRUD & query builder implementation
    ├── dbTransaction.go         # Multi-repository transaction coordinator
    ├── default.go               # GORM database connection bootstrapper
    └── dto.go                   # Database filter and pagination DTOs

## 3. Key Implementation Points

- Penggunaan batasan tipe Go Generics [T schema.Tabler] untuk validasi tabel GORM pada compile-time.
- Interface BaseRepository { SetupConnection(*gorm.DB) } untuk mengizinkan pergantian koneksi dinamis ke transaction context.
- Metode Begin() dan End(err) yang mengotomatisasi proses commit dan rollback transaksi database.

## 4. Code Examples

### Generic Repository berbasis Go Generics dan GORM Tabler interface.

```go
package db

import (
	"context"
	"gorm.io/gorm"
	"gorm.io/gorm/schema"
)

type GenericRepository[T schema.Tabler] struct {
	db    *gorm.DB
	model T
}

func (repo *GenericRepository[T]) SetupConnection(db *gorm.DB) {
	repo.db = db
}

func NewGenericeRepoPointr[T schema.Tabler](db *gorm.DB, model T) *GenericRepository[T] {
	return &GenericRepository[T]{
		db:    db,
		model: model,
	}
}

func (repo *GenericRepository[T]) Store(ctx context.Context, data T) (T, error) {
	err := repo.db.WithContext(ctx).Create(&data).Error
	return data, err
}

func (repo *GenericRepository[T]) FindAll(ctx context.Context) ([]T, error) {
	var results []T
	err := repo.db.WithContext(ctx).Find(&results).Error
	return results, err
}
```

### Transaction Coordinator yang menyebarkan koneksi transaksi aktif (*gorm.DB) ke semua repository terdaftar.

```go
package db

import (
	"gorm.io/gorm"
)

type DBTransaction struct {
	db    *gorm.DB
	repos []BaseRepository
}

type BaseRepository interface {
	SetupConnection(db *gorm.DB)
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
	begin := main.db.Begin()
	for _, rp := range main.repos {
		rp.SetupConnection(begin)
	}
	main.db = begin
}

func (main *DBTransaction) End(err error) error {
	if err != nil {
		if errTrx := main.db.Rollback().Error; errTrx != nil {
			return errTrx
		}
		return nil
	}
	if errTx := main.db.Commit().Error; errTx != nil {
		return errTx
	}
	return nil
}
```

## 5. Considerations & Best Practices

- Pastikan method SetupConnection dipanggil pada instance pointer repository agar state koneksi transaksi termutasi dengan benar.
- Gunakan pola defer trx.End(err) pada usecase untuk menjamin rollback terpanggil saat terjadi panic atau error.

## 6. Related Knowledge

- Repository Pattern Generics
- Database Transactions

## 7. Source

- [[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]
