---
title: "Encapsulated S3/MinIO Object Storage Driver with Stream Buffer Pipeline"
type: pattern
tags: [pattern, backend, golang, minio, s3, object-storage, stream-pipeline]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787128418632-596cfbe7
sources: ["Harvest 1787128418632 596cfbe7.json"]
---

# Encapsulated S3/MinIO Object Storage Driver with Stream Buffer Pipeline

Encapsulated S3/MinIO object storage adapter providing streaming upload/download pipelines and bucket-scoped operations.

## 1. Overview & Architecture

An encapsulated cloud and local object storage adapter that abstracts MinIO / AWS S3 SDK operations into a clean, stream-oriented storage interface. It isolates bucket names, credentials, and network endpoints while providing standardized methods for storing, fetching, and removing binary files.

## 2. Implementation & Code Structure

pkg/miniostorage/
└── storage.go               # StorageMinio struct, NewConnection, GetFile, StoreFile, DeleteFile

## 3. Key Implementation Points

- Clean wrapper around MinIO Go SDK decoupling business usecases from S3 vendor API details.
- Streaming upload StoreFile accepting any io.Reader with explicit fileSize for optimal chunked transfer.
- Buffered file reader GetFile returning bytes.Buffer with proper resource closure.
- Thread-safe, reusable storage client instance configured at application startup.

## 4. Code Examples

### StorageMinio client initialization with static credential binding and bucket isolation.

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

### Streaming file retrieval, chunked storage upload, and object deletion methods.

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

- Ensure large file uploads stream directly via io.Reader rather than buffering entire payloads into memory upfront.
- Always defer closing minio.Object handles in GetFile to avoid connection leaks in the HTTP pool.
- Configure Secure: true when communicating with TLS-enabled S3 endpoints in production environments.

## 6. Related Knowledge

- Object Storage Architecture
- Clean Architecture Skeleton

## 7. Source

- Harvest 1787128418632 596cfbe7.json
