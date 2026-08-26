---
supersession: "ACTIVE"
review_by: "2027-02-22"
owner: "knowledge-curator"
confidence: 0.5
provenance_schema: 1
title: "Resilient Redis Distributed Caching & Atomicity Gateway Pattern"
type: pattern
tags: [pattern, backend, redis, caching, distributed-lock, resp3, atomicity]
created: 2026-08-26
updated: 2026-08-26
orchestrator_run: "harvest-1787715147310-7bd85203"
sources: ["[[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]"]
---

# Resilient Redis Distributed Caching & Atomicity Gateway Pattern

Distributed Caching & Atomicity Gateway berbasis Redis RESP3 dengan atomic SetNX lock dan lifecycle TTL management.

## 1. Overview & Architecture

Pola Resilient Redis Distributed Caching & Atomicity Gateway menyediakan lapisan akses caching in-memory terdistribusi dengan fitur atomisitas tingkat tinggi. Memanfaatkan protokol RESP3 dan atomic primitive seperti `SetNX`, pola ini memfasilitasi distributed locking, pencegahan cache stampede, rate limiting, dan penyimpanan data sementara dengan kontrol TTL presisi.

## 2. Implementation & Code Structure

pkg/cache/
└── cache.go              # Redis client wrapper with atomic operations & lifecycle management

## 3. Key Implementation Points

- Inisialisasi Redis Client dengan dukungan RESP3 Protocol dan handshake liveness ping.
- Operasi atomic concurrency control `SetNX` (Set if Not Exists) untuk distributed lock dan idempotency guard.
- Abstraksi method `Set`, `Get`, `Delete` yang bersih dan terisolasi dari implementasi library spesifik.
- Contextual propagation untuk seluruh operasi I/O cache guna mendukung request lifecycle timeout.

## 4. Code Examples

### Inisialisasi klien Redis dengan protokol RESP3 dan verifikasi konektivitas startup via Ping.

```go
type DbClient struct {
	client *redis.Client
}

func Default() DbClient {
	password :[REDACTED] os.Getenv("REDIS_PASSWORD")
	host := os.Getenv("REDIS_HOST")
	newClient := redis.NewClient(&redis.Options{
		Addr:     host,
		Password: [REDACTED],
		Protocol: 3,
	})
	pong, err := newClient.Ping(context.Background()).Result()
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println("redis start... ", pong)
	return DbClient{
		client: newClient,
	}
}
```

### Operasi Atomic SetNX, Set dengan TTL, Get, dan Delete dengan context cancellation handling.

```go
// Set stores value in a key with expiration.
func (rdb *DbClient) Set(ctx context.Context, key string, value interface{}, exp time.Duration) error {
	err := rdb.client.Set(ctx, key, value, exp).Err()
	if err != nil {
		return err
	}
	return nil
}

// SetNX stores value if not exists (Not eXists) in a key with expiration.
func (rdb *DbClient) SetNX(ctx context.Context, key string, value interface{}, expiration time.Duration) error {
	err := rdb.client.SetNX(ctx, key, value, expiration).Err()
	if err != nil {
		return err
	}
	return nil
}

// Get retrieves key in form of string.
func (rdb *DbClient) Get(ctx context.Context, key string) (string, error) {
	value, err := rdb.client.Get(ctx, key).Result()
	if err != nil {
		return "", err
	}
	return value, nil
}

// Delete deletes keys.
func (rdb *DbClient) Delete(ctx context.Context, keys ...string) error {
	return rdb.client.Del(ctx, keys...).Err()
}
```

## 5. Considerations & Best Practices

- Operasi SetNX harus selalu menyertakan batas waktu kadaluarsa (TTL) untuk menghindari deadlock jika instance aplikasi mengalami crash sebelum rilis lock.
- Protokol RESP3 (Protocol: 3) memberikan efisiensi parsing data lebih tinggi dan respons tipe data native dibanding RESP2.
- Semua operasi menerima `context.Context` agar dapat dibatalkan jika HTTP request melebihi timeout (deadline exceeded).

## 6. Related Knowledge

- Distributed Idempotency Guard Request Pipeline
- In Memory Caching Strategies

## 7. Source

- [[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]
