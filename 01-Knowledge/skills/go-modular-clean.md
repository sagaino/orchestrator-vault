---
provenance_schema: 1
orchestrator_run: "manual-curation:p8-0-skills-20260827"
owner: "knowledge-curator"
confidence: 0.95
review_by: "2027-02-27"
supersession: "ACTIVE"
title: "Go Modular Clean Architecture Skill"
type: pattern
tags: ["skill", "backend", "golang", "clean-architecture", "gin", "gorm", "redis"]
created: 2026-08-27
updated: 2026-08-27
sources: ["[[01-Knowledge/patterns/backend/project-skeleton-template]]", "base-be-golang", "be-golang-app"]
---

# Go Modular Clean Architecture Skill (`go-modular-clean@1.0.0`)

Panduan teknis dan standar implementasi backend Go skala enterprise berbasis **Go 1.23+ + Gin Gonic + GORM PostgreSQL + Redis + 3-Layer Modular Clean Architecture**.

---

## 1. Arsitektur 3 Lapis & Pembagian Boundary

Setiap modul bisnis baru (`<feature>`) **wajib** mengikuti pembagian struktur folder berikut:

```text
internal/
├── core/
│   ├── domain/
│   │   └── <feature>.go                  # Entity struct (embed BaseEntity) & Repository Interface
│   └── usecase/
│       └── <feature>/
│           ├── dto.go                    # Request & Response structs (binding validation)
│           ├── usecase.go                # Business logic implementation (bebas dari Gin)
│           └── usecase_test.go           # Table-driven unit tests dengan Mock Repository
└── adapter/
    └── controller/
        ├── <feature>.go                  # Gin HTTP handler & Route registration
        └── <feature>_test.go             # Controller HTTP unit tests
```

---

## 2. Standar Domain Layer (`internal/core/domain/<feature>.go`)

### A. Aturan Wajib Domain:
1. Setiap Entity **wajib meng-embed `domain.BaseEntity`** (`ID`, `CreatedAt`, `UpdatedAt`, `DeletedAt`).
2. Setiap Entity **wajib mendefinisikan method `TableName() string`** untuk nama tabel GORM.
3. Mendefinisikan **Repository Interface** dan implementasi GORM-nya di file yang sama atau file repository terpisah.

### B. Contoh Kode Domain:
```go
package domain

import (
	"context"
	"gorm.io/gorm"
)

type Product struct {
	BaseEntity
	Name        string  `json:"name" gorm:"column:name;type:varchar(255);not null"`
	Description string  `json:"description" gorm:"column:description;type:text"`
	Price       float64 `json:"price" gorm:"column:price;type:decimal(10,2);not null"`
	Stock       int     `json:"stock" gorm:"column:stock;type:int;not null;default:0"`
	Category    string  `json:"category" gorm:"column:category;type:varchar(100);not null"`
}

func (Product) TableName() string {
	return "products"
}

type ProductRepository interface {
	Create(ctx context.Context, item *Product) error
	GetByID(ctx context.Context, id string) (*Product, error)
	List(ctx context.Context, limit, offset int, search string) ([]Product, int64, error)
	Update(ctx context.Context, item *Product) error
	Delete(ctx context.Context, id string) error
}

type productRepository struct {
	db *gorm.DB
}

func NewProductRepository(db *gorm.DB) ProductRepository {
	return &productRepository{db: db}
}

func (r *productRepository) Create(ctx context.Context, item *Product) error {
	return r.db.WithContext(ctx).Create(item).Error
}

func (r *productRepository) GetByID(ctx context.Context, id string) (*Product, error) {
	var item Product
	if err := r.db.WithContext(ctx).Where("id = ? AND deleted_at IS NULL", id).First(&item).Error; err != nil {
		return nil, err
	}
	return &item, nil
}

func (r *productRepository) List(ctx context.Context, limit, offset int, search string) ([]Product, int64, error) {
	var items []Product
	var total int64
	query := r.db.WithContext(ctx).Model(&Product{}).Where("deleted_at IS NULL")
	if search != "" {
		query = query.Where("name ILIKE ?", "%"+search+"%")
	}
	if err := query.Count(&total).Error; err != nil {
		return nil, 0, err
	}
	if limit > 0 {
		query = query.Limit(limit).Offset(offset)
	}
	if err := query.Order("created_at DESC").Find(&items).Error; err != nil {
		return nil, 0, err
	}
	return items, total, nil
}

func (r *productRepository) Update(ctx context.Context, item *Product) error {
	return r.db.WithContext(ctx).Save(item).Error
}

func (r *productRepository) Delete(ctx context.Context, id string) error {
	return r.db.WithContext(ctx).Where("id = ?", id).Delete(&Product{}).Error
}
```

---

## 3. Standar Usecase Layer (`internal/core/usecase/<feature>/`)

### A. DTO Structs (`dto.go`)
Gunakan tag `binding:"..."` untuk validasi otomatis Gin dan tag `json:"..."` untuk serialization:
```go
package product

type CreateProductRequest struct {
	Name        string  `json:"name" binding:"required,min=1,max=100"`
	Description string  `json:"description"`
	Price       float64 `json:"price" binding:"required,gt=0"`
	Stock       int     `json:"stock" binding:"gte=0"`
	Category    string  `json:"category" binding:"required"`
}

type UpdateProductRequest struct {
	Name        string  `json:"name" binding:"required,min=1,max=100"`
	Description string  `json:"description"`
	Price       float64 `json:"price" binding:"required,gt=0"`
	Stock       int     `json:"stock" binding:"gte=0"`
	Category    string  `json:"category" binding:"required"`
}

type ProductResponse struct {
	ID          string  `json:"id"`
	Name        string  `json:"name"`
	Description string  `json:"description"`
	Price       float64 `json:"price"`
	Stock       int     `json:"stock"`
	Category    string  `json:"category"`
	CreatedAt   string  `json:"createdAt"`
	UpdatedAt   string  `json:"updatedAt"`
}

type ProductListResponse struct {
	Items []ProductResponse `json:"items"`
	Total int64             `json:"total"`
}
```

### B. Business Logic & Error Handling (`usecase.go`)
* Dilarang mengimpor package `gin` di layer usecase.
* Gunakan `pkg/localerror` untuk error handling bisnis:
```go
package product

import (
	"context"
	"be-golang-app/internal/core/domain"
	"be-golang-app/pkg/localerror"
)

type Usecase interface {
	Create(ctx context.Context, req CreateProductRequest) (*ProductResponse, error)
	GetByID(ctx context.Context, id string) (*ProductResponse, error)
	List(ctx context.Context, limit, offset int, search string) (*ProductListResponse, error)
	Update(ctx context.Context, id string, req UpdateProductRequest) (*ProductResponse, error)
	Delete(ctx context.Context, id string) error
}

type usecase struct {
	repo domain.ProductRepository
}

func NewUsecase(repo domain.ProductRepository) Usecase {
	return &usecase{repo: repo}
}

func (u *usecase) Create(ctx context.Context, req CreateProductRequest) (*ProductResponse, error) {
	if req.Name == "" {
		return nil, localerror.NewInvalidData("Nama produk wajib diisi", "NAME_REQUIRED")
	}

	item := &domain.Product{
		Name:        req.Name,
		Description: req.Description,
		Price:       req.Price,
		Stock:       req.Stock,
		Category:    req.Category,
	}

	if err := u.repo.Create(ctx, item); err != nil {
		return nil, localerror.NewInternal("Gagal menyimpan produk ke database: "+err.Error(), "DB_ERROR")
	}

	return toProductResponse(item), nil
}

func (u *usecase) GetByID(ctx context.Context, id string) (*ProductResponse, error) {
	if id == "" {
		return nil, localerror.NewInvalidData("ID produk wajib diisi", "ID_REQUIRED")
	}

	item, err := u.repo.GetByID(ctx, id)
	if err != nil {
		return nil, localerror.NewNotFound("Produk tidak ditemukan", "PRODUCT_NOT_FOUND")
	}

	return toProductResponse(item), nil
}

func (u *usecase) List(ctx context.Context, limit, offset int, search string) (*ProductListResponse, error) {
	items, total, err := u.repo.List(ctx, limit, offset, search)
	if err != nil {
		return nil, localerror.NewInternal("Gagal mengambil data produk: "+err.Error(), "DB_ERROR")
	}

	responses := make([]ProductResponse, len(items))
	for i, item := range items {
		responses[i] = *toProductResponse(&item)
	}

	return &ProductListResponse{
		Items: responses,
		Total: total,
	}, nil
}

func (u *usecase) Update(ctx context.Context, id string, req UpdateProductRequest) (*ProductResponse, error) {
	item, err := u.repo.GetByID(ctx, id)
	if err != nil {
		return nil, localerror.NewNotFound("Produk tidak ditemukan", "PRODUCT_NOT_FOUND")
	}

	item.Name = req.Name
	item.Description = req.Description
	item.Price = req.Price
	item.Stock = req.Stock
	item.Category = req.Category

	if err := u.repo.Update(ctx, item); err != nil {
		return nil, localerror.NewInternal("Gagal memperbarui produk: "+err.Error(), "DB_ERROR")
	}

	return toProductResponse(item), nil
}

func (u *usecase) Delete(ctx context.Context, id string) error {
	if _, err := u.repo.GetByID(ctx, id); err != nil {
		return localerror.NewNotFound("Produk tidak ditemukan", "PRODUCT_NOT_FOUND")
	}
	return u.repo.Delete(ctx, id)
}

func toProductResponse(item *domain.Product) *ProductResponse {
	return &ProductResponse{
		ID:          item.ID,
		Name:        item.Name,
		Description: item.Description,
		Price:       item.Price,
		Stock:       item.Stock,
		Category:    item.Category,
		CreatedAt:   item.CreatedAt.Format("2006-01-02 15:04:05"),
		UpdatedAt:   item.UpdatedAt.Format("2006-01-02 15:04:05"),
	}
}
```

---

## 4. Standar Controller Layer (`internal/adapter/controller/<feature>.go`)

* Meng-embed `base.BaseController` dan menerima `port base.Port`.
* Output response **wajib menggunakan `c.port.SendResponse(ctx, code, payload.NewSuccessResponse/NewErrorResponse)`**:
```go
package controller

import (
	"net/http"
	"strconv"
	"github.com/gin-gonic/gin"
	"gorm.io/gorm"
	"be-golang-app/internal/core/domain"
	productUsecase "be-golang-app/internal/core/usecase/product"
	"be-golang-app/shared/base"
	"be-golang-app/shared/payload"
)

type ProductController struct {
	base.BaseController
	port    base.Port
	usecase productUsecase.Usecase
}

func NewProductController(dbConn *gorm.DB, port base.Port, ctrl base.BaseController) *ProductController {
	repo := domain.NewProductRepository(dbConn)
	uc := productUsecase.NewUsecase(repo)
	return &ProductController{
		BaseController: ctrl,
		port:           port,
		usecase:        uc,
	}
}

func (c *ProductController) RegisterRoutes(router *gin.RouterGroup) {
	group := router.Group("/products")
	{
		group.POST("", c.Create)
		group.GET("", c.List)
		group.GET("/:id", c.GetByID)
		group.PUT("/:id", c.Update)
		group.DELETE("/:id", c.Delete)
	}
}

func (c *ProductController) Create(ctx *gin.Context) {
	var req productUsecase.CreateProductRequest
	if err := ctx.ShouldBindJSON(&req); err != nil {
		c.port.SendResponse(ctx, http.StatusBadRequest, payload.NewErrorResponse(400, err.Error()))
		return
	}

	res, err := c.usecase.Create(ctx.Request.Context(), req)
	if err != nil {
		c.port.SendResponse(ctx, http.StatusBadRequest, payload.NewErrorResponse(400, err.Error()))
		return
	}

	c.port.SendResponse(ctx, http.StatusCreated, payload.NewSuccessResponse(201, "Produk berhasil dibuat", res))
}

func (c *ProductController) List(ctx *gin.Context) {
	limit, _ := strconv.Atoi(ctx.DefaultQuery("limit", "10"))
	page, _ := strconv.Atoi(ctx.DefaultQuery("page", "1"))
	search := ctx.Query("search")
	offset := (page - 1) * limit

	res, err := c.usecase.List(ctx.Request.Context(), limit, offset, search)
	if err != nil {
		c.port.SendResponse(ctx, http.StatusInternalServerError, payload.NewErrorResponse(500, err.Error()))
		return
	}

	c.port.SendResponse(ctx, http.StatusOK, payload.NewSuccessResponse(200, "Berhasil mengambil daftar produk", res))
}

func (c *ProductController) GetByID(ctx *gin.Context) {
	id := ctx.Param("id")
	res, err := c.usecase.GetByID(ctx.Request.Context(), id)
	if err != nil {
		c.port.SendResponse(ctx, http.StatusNotFound, payload.NewErrorResponse(404, err.Error()))
		return
	}

	c.port.SendResponse(ctx, http.StatusOK, payload.NewSuccessResponse(200, "Berhasil mengambil detail produk", res))
}

func (c *ProductController) Update(ctx *gin.Context) {
	id := ctx.Param("id")
	var req productUsecase.UpdateProductRequest
	if err := ctx.ShouldBindJSON(&req); err != nil {
		c.port.SendResponse(ctx, http.StatusBadRequest, payload.NewErrorResponse(400, err.Error()))
		return
	}

	res, err := c.usecase.Update(ctx.Request.Context(), id, req)
	if err != nil {
		c.port.SendResponse(ctx, http.StatusBadRequest, payload.NewErrorResponse(400, err.Error()))
		return
	}

	c.port.SendResponse(ctx, http.StatusOK, payload.NewSuccessResponse(200, "Produk berhasil diperbarui", res))
}

func (c *ProductController) Delete(ctx *gin.Context) {
	id := ctx.Param("id")
	if err := c.usecase.Delete(ctx.Request.Context(), id); err != nil {
		c.port.SendResponse(ctx, http.StatusNotFound, payload.NewErrorResponse(404, err.Error()))
		return
	}

	c.port.SendResponse(ctx, http.StatusOK, payload.NewSuccessResponse(200, "Produk berhasil dihapus", nil))
}
```

---

## 5. Composition Root & Wiring (`cmd/api/api.go`)

Daftarkan AutoMigrate entity dan daftarkan Controller ke router start engine:
```go
start.DB().AutoMigrate(&domain.Product{})
start.Register(controller.NewProductController)
```

---

## 6. Standar Unit Testing (Table-Driven + Mock Pattern)

Contoh `internal/core/usecase/<feature>/usecase_test.go`:
```go
package product_test

import (
	"context"
	"errors"
	"testing"
	"be-golang-app/internal/core/domain"
	productUsecase "be-golang-app/internal/core/usecase/product"
)

type mockProductRepository struct {
	createFn  func(ctx context.Context, item *domain.Product) error
	getByIDFn func(ctx context.Context, id string) (*domain.Product, error)
}

func (m *mockProductRepository) Create(ctx context.Context, item *domain.Product) error {
	if m.createFn != nil {
		return m.createFn(ctx, item)
	}
	return nil
}

func (m *mockProductRepository) GetByID(ctx context.Context, id string) (*domain.Product, error) {
	if m.getByIDFn != nil {
		return m.getByIDFn(ctx, id)
	}
	return nil, nil
}

func (m *mockProductRepository) List(ctx context.Context, limit, offset int, search string) ([]domain.Product, int64, error) {
	return nil, 0, nil
}

func (m *mockProductRepository) Update(ctx context.Context, item *domain.Product) error {
	return nil
}

func (m *mockProductRepository) Delete(ctx context.Context, id string) error {
	return nil
}

func TestUsecase_Create(t *testing.T) {
	tests := []struct {
		name    string
		req     productUsecase.CreateProductRequest
		mockFn  func(ctx context.Context, item *domain.Product) error
		wantErr bool
	}{
		{
			name: "success create product",
			req: productUsecase.CreateProductRequest{
				Name:     "Kamera CCTV",
				Price:    500000,
				Stock:    10,
				Category: "Security",
			},
			mockFn: func(ctx context.Context, item *domain.Product) error {
				item.ID = "prod-123"
				return nil
			},
			wantErr: false,
		},
		{
			name: "fail db error",
			req: productUsecase.CreateProductRequest{
				Name:  "Kamera CCTV",
				Price: 500000,
			},
			mockFn: func(ctx context.Context, item *domain.Product) error {
				return errors.New("db connection failed")
			},
			wantErr: true,
		},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			repo := &mockProductRepository{createFn: tt.mockFn}
			uc := productUsecase.NewUsecase(repo)
			res, err := uc.Create(context.Background(), tt.req)
			if (err != nil) != tt.wantErr {
				t.Fatalf("expected error: %v, got: %v", tt.wantErr, err)
			}
			if !tt.wantErr && res.ID != "prod-123" {
				t.Errorf("expected id 'prod-123', got '%s'", res.ID)
			}
		})
	}
}
```

---

## 7. Standar Verifikasi & Quality Check
* **Format & Compile**: Wajib lolos `go vet ./...`.
* **Unit Tests**: Wajib lolos `go test ./...`.
