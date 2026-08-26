---
supersession: "ACTIVE"
review_by: "2027-02-22"
owner: "knowledge-curator"
confidence: 0.5
provenance_schema: 1
title: "S3-Compatible Binary Object Storage & Stream Buffer Gateway"
type: pattern
tags: [pattern, backend, minio, s3, object-storage, binary-streaming, aws-v4]
created: 2026-08-26
updated: 2026-08-26
orchestrator_run: "harvest-1787715147310-7bd85203"
sources: ["[[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]"]
---

# S3-Compatible Binary Object Storage & Stream Buffer Gateway

S3/MinIO Binary Object Storage Gateway dengan streaming I/O io.Reader, Static V4 Credentials, dan pencegahan socket leak.

## 1. Overview & Architecture

Pola S3-Compatible Binary Object Storage & Stream Buffer Gateway membungkus komunikasi ke MinIO / Amazon S3 Object Store dalam sebuah adapter terisolasi. Pola ini memproses payload biner dalam bentuk stream `io.Reader` dan `bytes.Buffer`, menjaga jejak memori server tetap minimal serta memisahkan detail protokol AWS S3 dari domain bisnis utama.

## 2. Implementation & Code Structure

pkg/miniostorage/
└── storage.go            # S3/MinIO client connection, streaming upload/download, and object lifecycle

## 3. Key Implementation Points

- Abstraksi antarmuka penyimpanan biner yang kompatibel dengan protokol AWS S3 / MinIO.
- Autentikasi aman menggunakan Static V4 AWS signature credentials.
- Streaming file I/O berbasis standard Go `io.Reader` dan `bytes.Buffer` untuk efisiensi memori tingkat tinggi.
- Dukungan operasi lengkap lifecycle object (Upload, Download Stream, Remove) berlandaskan `context.Context`.

## 4. Code Examples

### Inisialisasi koneksi S3/MinIO client dengan Static V4 Credentials provider.

```go
type StorageMinio struct {
	client *minio.Client
	bucket string
}

type Conn struct {
	Endpoint  string `json:"endpoint"`
	Bucket    string `json:"bucket"`
	AccessKey string `json:"access_key"`
	SecretKey string `json:"secret_key"`
}

func NewConnection(conn Conn) StorageMinio {
	client, err := minio.New(conn.Endpoint, &minio.Options{
		Creds:  credentials.NewStaticV4(conn.AccessKey, conn.SecretKey, ""),
		Secure: false,
	})
	if err != nil {
		panic(fmt.Errorf("connection to miniostorage err => %s", err.Error()))
	}

	return StorageMinio{client: client, bucket: conn.Bucket}
}
```

### Operasi streaming I/O file (Store via io.Reader, Get via bytes.Buffer, Delete) dengan resource cleanup terjamin.

```go
func (st StorageMinio) GetFile(ctx context.Context, fileName string) (*bytes.Buffer, error) {
	obj, err := st.client.GetObject(ctx, st.bucket, fileName, minio.GetObjectOptions{})
	if err != nil {
		return nil, err
	}

	defer obj.Close()
	buf := new(bytes.Buffer)
	if _, err = buf.ReadFrom(obj); err != nil {
		return nil, err
	}

	return buf, nil
}

func (st StorageMinio) StoreFile(ctx context.Context, fileName string, file io.Reader, fileSize int64) (minio.UploadInfo, error) {
	uploadInfo, err := st.client.PutObject(ctx, st.bucket, fileName, file, fileSize, minio.PutObjectOptions{})
	if err != nil {
		return minio.UploadInfo{}, err
	}

	return uploadInfo, nil
}

func (st StorageMinio) DeleteFile(ctx context.Context, fileName string) error {
	return st.client.RemoveObject(ctx, st.bucket, fileName, minio.RemoveObjectOptions{})
}
```

## 5. Considerations & Best Practices

- Penggunaan `io.Reader` pada `StoreFile` memungkinkan streaming langsung dari multipart HTTP request tanpa perlu memuat seluruh file ke memori (zero-memory buffer explosion).
- Objek `*minio.Object` dari `GetObject` mengimplementasikan `io.ReadCloser` sehingga wajib ditutup (`defer obj.Close()`) guna menghindari socket leak.
- Konfigurasi `Secure` boolean harus diaktifkan (`true`) di lingkungan production (HTTPS/TLS) untuk melindungi integritas transmisi data biner.

## 6. Related Knowledge

- Object Storage Architecture
- Cross Cutting Port Facade

## 7. Source

- [[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]
