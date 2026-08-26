---
supersession: "ACTIVE"
review_by: "2027-02-22"
owner: "knowledge-curator"
confidence: 0.5
provenance_schema: 1
title: "Dynamic Reflective ORM Engine & SQL AST Clause Builder"
type: pattern
tags: [pattern, backend, orm, gorm, reflection, sql-builder, ast-clauses]
created: 2026-08-26
updated: 2026-08-26
orchestrator_run: "harvest-1787715147310-7bd85203"
sources: ["[[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]"]
---

# Dynamic Reflective ORM Engine & SQL AST Clause Builder

Reflective ORM Engine & SQL AST Clause Builder yang mengotomatisasi pemetaan kolom struct, transformasi snake_case, dan pembentukan query parameterized dinamis.

## 1. Overview & Architecture

Pola Dynamic Reflective ORM Engine mengabstraksi pembentukan query SQL kompleks dan dinamis di atas GORM. Dengan memanfaatkan Go runtime reflection dan GORM Statement Clauses, pola ini mampu menginspeksi struct domain secara otomatis, memetakan tag gorm ke nama kolom snake_case, menghasilkan query INSERT ber-UUID secara terstandardisasi, serta merangkai klausa SELECT dan JOIN tanpa hardcoded SQL string.

## 2. Implementation & Code Structure

pkg/db/
├── db-custome.go         # Core reflective ORM engine & SQL AST Clause generator
├── dbTransaction.go      # Atomic unit-of-work transaction coordinator
├── default.go            # GORM connection pool initializer
├── dto.go                # Pagination & query specification contracts
└── generic_repository.go # Base CRUD generic repository implementation

## 3. Key Implementation Points

- Ekstraksi skema dan metadata kolom struct via Go Reflection dan tag parser `gorm:"column:..."`.
- Otomatisasi transformasi nama kolom (PascalCase -> snake_case) menggunakan regular expressions dan buffer builder.
- Penyusunan Statement Clause AST dinamis (`clause.Select`, `clause.Join`) untuk menghindari manipulasi string SQL mentah.
- Dukungan context propagation melalui method chaining `WithContext(ctx context.Context)`.

## 4. Code Examples

### Struktur CustomORM dan factory constructor dengan context propagation.

```go
type CustomORM struct {
	db           *gorm.DB
	currSelected []string
	join         []interface{}
	currVal      []interface{}
	currQuery    string
	scope        func(db *gorm.DB) *gorm.DB
}

func NewCustomeORM(db *gorm.DB) *CustomORM {
	return &CustomORM{db: db}
}

func (repo *CustomORM) WithContext(ctx context.Context) *CustomORM {
	return &CustomORM{db: repo.db.WithContext(ctx)}
}
```

### Refleksi runtime untuk parsing gorm struct tags dan otomatisasi konversi penamaan PascalCase ke snake_case.

```go
func PascalToSnake(input string) string {
	var output bytes.Buffer
	re := regexp.MustCompile(`([a-z])([A-Z]+)`)
	snakeCaseStr := re.ReplaceAllString(input, "${1}_${2}")
	snakeCaseStr = strings.ToLower(snakeCaseStr)
	output.WriteString(snakeCaseStr)
	return output.String()
}

func GetColNames(table interface{}, excludeCols ...string) []string {
	sType := reflect.TypeOf(table)
	if sType.Kind() == reflect.Pointer {
		sType = sType.Elem()
	}
	var colNames []string

	for i := 0; i < sType.NumField(); i++ {
		f := sType.Field(i)
		gormTag := f.Tag.Get("gorm")
		colName := getColName(gormTag, f)
		if slices.Contains(excludeCols, colName) || colName == "id" {
			continue
		}
		colNames = append(colNames, colName)
	}

	return colNames
}

func getColName(gormTag string, f reflect.StructField) string {
	var colName string
	tags := strings.Split(gormTag, ";")
	for _, tag := range tags {
		if strings.Contains(tag, "column") {
			cn := strings.Split(tag, ":")
			if len(cn) < 2 {
				colName = PascalToSnake(f.Name)
				break
			}
			colName = cn[1]
			break
		}
		colName = PascalToSnake(f.Name)
	}
	return colName
}
```

### Eksekusi parameterized INSERT query dinamis dengan injeksi UUID generator native database dan sinkronisasi model.

```go
func (repo *CustomORM) Store(model interface{}, ignoreFields ...string) *CustomORM {
	colsName := GetColNames(model, ignoreFields...)
	actualVal := getValueFromModel(model, colsName)
	placeHolder := populatePlaceHolder(colsName, actualVal)

	pType := reflect.TypeOf(model)
	if pType.Kind() == reflect.Pointer {
		pType = pType.Elem()
	}
	tableName := repo.extractTableName(pType)

	query := "INSERT INTO \"" + tableName + "\" " +
		"( " +
		"\"id\", " + strings.Join(colsName, ", ") +
		" ) " +
		"values " +
		"(gen_random_uuid(), " + strings.Join(placeHolder, ", ") + ");"

	return &CustomORM{db: repo.db.
		Exec(
			query,
			actualVal...,
		).
		Find(&model),
	}
}
```

### Pembangunan AST Statement Clauses untuk query SELECT dan JOIN dinamis.

```go
func (repo *CustomORM) Find(model interface{}) *CustomORM {
	var colsName = GetColNames(model)
	if len(repo.currSelected) > 0 {
		colsName = repo.currSelected
	}

	pType := reflect.TypeOf(model)
	if pType.Kind() == reflect.Pointer {
		pType = pType.Elem()
	}
	tableName := repo.extractTableName(pType)
	var columns = make([]clause.Column, len(colsName))
	for i, s := range colsName {
		columns[i] = clause.Column{Table: tableName, Name: s}
	}

	var st gorm.Statement
	st.Table = tableName
	st.Clauses = map[string]clause.Clause{
		"SELECT": {Expression: clause.Select{
			Columns: columns,
		}},
	}
	if repo.scope != nil {
		st.DB.Scopes()
	}

	var jStm = make([]clause.Join, len(repo.join))
	for _, j := range repo.join {
		if t, ok := j.(string); ok {
			jStm = append(jStm, repo.joinWithQuery(t))
			continue
		}
		typeOf := reflect.TypeOf(j)
		if typeOf.Kind() != reflect.Struct {
			repo.db.Error = fmt.Errorf("format should struct")
		}

		jStm = append(jStm, repo.joinWithModel(j))
	}

	db := st.Callback().Query().Execute(repo.db)
	repo.db = db
	return repo
}
```

## 5. Considerations & Best Practices

- Reflection Go (reflect.TypeOf / reflect.ValueOf) memiliki overhead kecil saat pemanggilan berulang; caching field metadata dapat diterapkan untuk high-throughput critical path.
- SQL syntax 'gen_random_uuid()' diasosiasikan dengan PostgreSQL; untuk database lain (MySQL/SQLite) generator UUID dapat diatur secara adaptif.
- Semua parameter nilai wajib melalui placeholder '?' guna mencegah SQL injection vulnerabilities.

## 6. Related Knowledge

- [[01-Knowledge/patterns/backend/generic-type-safe-repository-with-atomic-transaction-coordinator.md]]
- Database Connection Pooling

## 7. Source

- [[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]
