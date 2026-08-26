---
supersession: "ACTIVE"
review_by: "2027-02-22"
owner: "knowledge-curator"
confidence: 0.5
provenance_schema: 1
title: "MIME-Compliant Native SMTP Mailer & Transactional Notification Bridge"
type: pattern
tags: [pattern, backend, smtp, email, mime, notifications, net-smtp]
created: 2026-08-26
updated: 2026-08-26
orchestrator_run: "harvest-1787715147310-7bd85203"
sources: ["[[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]"]
---

# MIME-Compliant Native SMTP Mailer & Transactional Notification Bridge

Native SMTP Mailer & Notification Bridge dengan RFC 2822 MIME 1.0 header builder dan SASL PlainAuth handshake.

## 1. Overview & Architecture

Pola MIME-Compliant Native SMTP Mailer & Transactional Notification Bridge menyediakan integrasi outbound messaging untuk pengiriman email transaksional secara langsung menggunakan protokol SMTP standar. Dengan membangun payload berstandar MIME 1.0 / RFC 2822 dan mengisolasi transport layer di balik payload DTO, pola ini membebaskan arsitektur dari keterikatan pada SDK proprietary vendor email.

## 2. Implementation & Code Structure

pkg/mailing/
├── dto.go                # Data Transfer Object for email notifications
└── send-in-blu.go        # Native SMTP sender & RFC 2822 / MIME 1.0 message builder

## 3. Key Implementation Points

- Pemanfaatan library bawaan `net/smtp` tanpa ketergantungan berlebih pada SDK vendor proprietary pihak ketiga.
- Penyusunan format pesan standar MIME-version 1.0 dengan encoding UTF-8 dan format HTML.
- Otentikasi aman berbasis SASL Plain Authentication (`smtp.PlainAuth`).
- Mendukung mocking otomatis via tooling Mockery (`//go:generate mockery`).

## 4. Code Examples

### Definisi DTO payload email dan konfigurasi struct SendInBlue.

```go
package mailing

type NativeSendEmailPayload struct {
	Username string
	Password string
	Host     string
	Port     string
	SendTo   string
	Subject  string
	HtmlBody string
}

type SendInBlue struct {
}

func NewConfig() SendInBlue {
	return SendInBlue{}
}
```

### Konstruksi header MIME-compliant RFC 2822 dan transmisi email melalui native net/smtp.PlainAuth.

```go
func (sib SendInBlue) NativeSendEmail(payload NativeSendEmailPayload) error {
	auth := smtp.PlainAuth("", payload.Username, payload.Password, payload.Host)
	messageBody := fmt.Sprintf(
		"From:  <%s>\n"+
			"To: <%s>\r\n"+
			"Subject: %s\r\n",
		payload.Username,
		payload.SendTo,
		payload.Subject,
	)
	messageBody += "MIME-version: 1.0;\r\n"
	messageBody += "Content-Type: text/html; charset=\"UTF-8\"\r\n"
	messageBody += payload.HtmlBody

	err := smtp.SendMail(
		payload.Host+":"+payload.Port,
		auth,
		payload.Username,
		[]string{payload.SendTo},
		[]byte(messageBody),
	)
	if err != nil {
		return err
	}

	return nil
}
```

## 5. Considerations & Best Practices

- Pengiriman email melalui SMTP bersifat synchronous blocking I/O; disarankan untuk dieksekusi di dalam goroutine atau background job queue pada use case skala tinggi.
- Format line break pada header email wajib mematuhi standar RFC 2822 (`\r\n`) guna mencegah rejection dari MTA (Mail Transfer Agent) tujuan.
- Credential SMTP wajib diinjeksikan melalui environment variables dan tidak di-hardcode ke dalam codebase.

## 6. Related Knowledge

- Transactional Email Delivery
- Fluent Typed Http Request Pipeline

## 7. Source

- [[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]
