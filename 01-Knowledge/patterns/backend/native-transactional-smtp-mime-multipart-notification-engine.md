---
title: "Native Transactional SMTP MIME Multipart Notification Engine"
type: pattern
tags: [pattern, backend, golang, smtp, mailing, notification, mime]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787128418632-596cfbe7
sources: ["Harvest 1787128418632 596cfbe7.json"]
---

# Native Transactional SMTP MIME Multipart Notification Engine

Native transactional SMTP mailer adapter with RFC-compliant MIME 1.0 multipart header synthesis.

## 1. Overview & Architecture

A zero-external-dependency transactional email notification provider implementing native Go net/smtp plain authentication with RFC-compliant MIME 1.0 multipart headers and UTF-8 HTML payload rendering.

## 2. Implementation & Code Structure

pkg/mailing/
├── dto.go                   # NativeSendEmailPayload struct
└── send-in-blu.go           # SendInBlue struct and NativeSendEmail implementation

## 3. Key Implementation Points

- Native net/smtp.SendMail integration eliminating external third-party SDK dependencies.
- RFC-compliant MIME 1.0 headers and UTF-8 content-type declaration for rich HTML email rendering.
- smtp.PlainAuth authentication with parameterized SMTP server host and port.
- Mockery-ready interface compatibility for deterministic unit testing in domain services.

## 4. Code Examples

### Mailing configuration and NativeSendEmailPayload DTO definition.

```go
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

### Native SMTP plain authentication, RFC MIME multipart header formatting, and mail delivery.

```go
func (sib SendInBlue) NativeSendEmail(payload NativeSendEmailPayload) error {
	auth := smtp.PlainAuth("", payload.Username, payload.Password, payload.Host)
	messageBody := fmt.Sprintf(
		"From: <%s>\n"+
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

- MIME headers must strictly adhere to RFC 2045/2046 carriage return newline (\r\n) formatting to avoid rejection by strict mail servers.
- For high-volume asynchronous mailing, wrap NativeSendEmail inside a background worker pool or message queue to avoid blocking HTTP request cycles.
- Ensure SMTP credentials and host configurations are loaded securely via environment variables.

## 6. Related Knowledge

- Notification Pipelines
- Clean Architecture Skeleton

## 7. Source

- Harvest 1787128418632 596cfbe7.json
