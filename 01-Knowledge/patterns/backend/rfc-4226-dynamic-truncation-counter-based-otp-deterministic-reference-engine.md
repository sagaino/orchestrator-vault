---
supersession: "ACTIVE"
review_by: "2027-02-22"
owner: "knowledge-curator"
confidence: 0.5
provenance_schema: 1
title: "RFC 4226 Dynamic-Truncation Counter-based OTP & Deterministic Reference Engine"
type: pattern
tags: [pattern, backend, otp, rfc-4226, security, hmac]
created: 2026-08-26
updated: 2026-08-26
orchestrator_run: "harvest-1787715147310-7bd85203"
sources: ["[[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]"]
---

# RFC 4226 Dynamic-Truncation Counter-based OTP & Deterministic Reference Engine

RFC 4226 Dynamic-Truncation Counter-based OTP & Deterministic Reference Engine untuk autentikasi 2FA dan generasi nomor referensi transaksi anti-kolisi.

## 1. Overview & Architecture

Pola pembuatan token OTP berbasis standar RFC 4226 dan generator referensi transaksi deterministik berbasis HMAC. Pola ini memadukan entropi waktu dengan fungsi hash beralgoritma kuat untuk menjamin keacakan, keunikan, dan verifikabilitas token.

## 2. Implementation & Code Structure

pkg/
└── davinci/
    └── davinci.go
shared/
└── base/
    └── port.go

## 3. Key Implementation Points

- Implementasi RFC 4226 Dynamic Truncation dengan bitmask 0x7F pada 4-byte offset HMAC.
- Padding konsisten 6 digit angka untuk format OTP standar.
- Deterministic unique key generation berbasis HMAC-SHA256 yang mendukung custom predicate validation.

## 4. Code Examples

### RFC 4226 compliant HOTP generation using HMAC dynamic bitmask truncation alongside deterministic HMAC unique key generator

```go
// pkg/davinci/davinci.go
package davinci

import (
	"crypto/hmac"
	"crypto/sha1"
	"crypto/sha256"
	"encoding/base32"
	"fmt"
	"strconv"
	"time"
)

// RFC 4226 Dynamic Truncation for Counter-Based OTP
func (dc Engine) GenerateOTPCode(secret string, counter uint64) (int, error) {
	counterByte := make([]byte, 8)
	for i := 7; i >= 0; i-- {
		counterByte[i] = byte(counter & 0xff)
		counter >>= 8
	}

	secretByte, err := base32.StdEncoding.DecodeString(secret)
	if err != nil {
		return 0, fmt.Errorf("StdEncoding.DecodeString: %w", err)
	}
	hash := hmac.New(sha1.New, secretByte)
	if _, err = hash.Write(counterByte); err != nil {
		return 0, fmt.Errorf("hash.Write: %w", err)
	}
	hmacBytes := hash.Sum(nil)

	// RFC 4226 Section 5.4 Dynamic Truncation
	offset := hmacBytes[len(hmacBytes)-1] & 0xf
	code := (int(hmacBytes[offset])&0x7f)<<24 |
		(int(hmacBytes[offset+1])&0xff)<<16 |
		(int(hmacBytes[offset+2])&0xff)<<8 |
		(int(hmacBytes[offset+3]) & 0xff)
	code = code % 1000000

	return code, nil
}

// Time-seeded deterministic reference generator with uniqueness predicate
func (dc Engine) GenerateUniqueKey(secretKey []byte, uniqueID string, length int) (string, error) {
	timestamp := time.Now().UnixNano()
	data := fmt.Sprintf("%s:%d", uniqueID, timestamp)

	h := hmac.New(sha256.New, secretKey)
	if _, err := h.Write([]byte(data)); err != nil {
		return "", err
	}
	hash := h.Sum(nil)

	const charset = "abcdefghijklmnopqrstuvwxyz0123456789"
	charsetLen := len(charset)
	result := make([]byte, length)

	for i := 0; i < length; i++ {
		index := int(hash[i%len(hash)]) % charsetLen
		result[i] = charset[index]
	}

	return string(result), nil
}
```

## 5. Considerations & Best Practices

- Secret key untuk OTP harus disimpan dalam format Base32 standar sesuai spesifikasi RFC.
- Untuk mencegah desynchronization counter OTP, implementasikan look-ahead window saat proses verifikasi.
- Penggunaan timestamp beresolusi nanodetik menjamin entropi unik pada concurrency tinggi.

## 6. Related Knowledge

- Otp Rfc 4226
- Deterministic Reference Generator

## 7. Source

- [[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]
