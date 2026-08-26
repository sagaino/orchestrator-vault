---
supersession: "ACTIVE"
review_by: "2027-02-22"
owner: "knowledge-curator"
confidence: 0.5
provenance_schema: 1
title: "zero-leakage-structured-observability-and-sentry-enrichment-pipeline"
type: pattern
tags: [pattern, backend, golang, observability, sentry, zerolog, devops, security]
created: 2026-08-26
updated: 2026-08-26
orchestrator_run: "harvest-1787715147310-7bd85203"
sources: ["[[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]"]
---

# zero-leakage-structured-observability-and-sentry-enrichment-pipeline

Pipeline observabilitas terstruktur dan telemetri Sentry dengan sanitasi data otomatis, frame-skipping logger, dan bootstrapping multi-lingkungan yang aman.

## 1. Overview & Architecture

Pola Zero-Leakage Structured Observability and Sentry Enrichment Pipeline menggabungkan penanganan konfigurasi multi-flavor environment, structured logging berperforma tinggi dengan Zerolog, sanitasi data kredensial otomatis, dan integrasi pelacakan insiden real-time melalui Sentry. Pola ini memastikan visibilitas operasional penuh tanpa mengorbankan keamanan data sensitif.

## 2. Implementation & Code Structure

cmd/
└── api/
    └── api.go                      # Multi-flavor flag parsing (-env) and container initialization
pkg/
├── logger/
│   ├── logger.go                   # Standard logger interface definition
│   └── zerolog.go                  # ReZero wrapper, frame skipper, sensitive field scrubbing
├── environment/
│   └── environment.go             # Type-safe ENV accessor (GetUint, GetInt, GetFloat, CheckFlag)
├── middleware/
│   └── sentry.go                   # Sentry hub cloning, scope enrichment, header scrubbing, CaptureError
└── health/
    └── health.go                   # Health check endpoint controller and uptime reporter
shared/
└── api/
    └── default.go                  # Engine setup, Sentry client initialization, and middleware attachment

## 3. Key Implementation Points

- Multi-flavor runtime switching berbasis command flag (-env .env.stag, -env .env.prod) dengan godotenv.
- Type-safe environment wrapper yang menangani fallback nilai default secara robust tanpa panic runtime.
- Structured JSON logging dengan Zerolog yang mendukung dynamic log level (debug hingga fatal).
- Sanitasi otomatis kredensial dan header sensitif pada logging request dan payload capture.
- Pengayaan konteks error Sentry dengan metadata pengguna (User ID, Role, Language, IP, URI).

## 4. Code Examples

### Structured logger initialization with frame skipping and Sentry contextual enrichment pipeline

```go
// DefaultLogger initializes Zerolog with dynamic level and caller frame skipping
func DefaultLogger() ReZero {
	logLevel := strings.ToLower(os.Getenv("LOG_LEVEL"))
	level := zerolog.InfoLevel
	switch logLevel {
	case "debug":
		level = zerolog.DebugLevel
	case "info":
		level = zerolog.InfoLevel
	case "warn":
		level = zerolog.WarnLevel
	case "error":
		level = zerolog.ErrorLevel
	}

	zerolog.CallerSkipFrameCount = 4
	zerolog.CallerMarshalFunc = func(pc uintptr, file string, line int) string {
		short := file
		for i := len(file) - 1; i > 0; i-- {
			if file[i] == '/' {
				short = file[i+1:]
				break
			}
		}
		return fmt.Sprintf("%s:%d", short, line)
	}

	output := zerolog.ConsoleWriter{Out: os.Stdout, TimeFormat: time.RFC3339}
	logger := zerolog.New(output).With().Timestamp().Caller().Logger()
	Log = &logger
	return ReZero{logger: &logger, level: level}
}

// SentryMiddleware enriches Sentry events with request details and sanitized headers
func SentryMiddleware() gin.HandlerFunc {
	return func(c *gin.Context) {
		hub := sentry.GetHubFromContext(c.Request.Context())
		if hub == nil {
			hub = sentry.CurrentHub().Clone()
		}
		ctx := sentry.SetHubOnContext(c.Request.Context(), hub)
		c.Request = c.Request.WithContext(ctx)

		hub.ConfigureScope(func(scope *sentry.Scope) {
			scope.SetRequest(c.Request)
			scope.SetTag("request.path", c.Request.URL.Path)
			scope.SetTag("request.method", c.Request.Method)
			scope.SetTag("request.remote_addr", c.ClientIP())
			scope.SetExtra("request.headers", convertHeaders(c.Request.Header))
		})

		c.Next()
		enrichSentryWithUserData(hub, c)
	}
}

func convertHeaders(headers http.Header) map[string]string {
	result := make(map[string]string)
	for key, values := range headers {
		if len(values) > 0 {
			if key == "Authorization" || key == "Cookie" || key == "X-Api-Key" {
				result[key] = "[Filtered]"
			} else {
				result[key] = values[0]
			}
		}
	}
	return result
}
```

## 5. Considerations & Best Practices

- Header sensitif (Authorization, Cookie, X-Api-Key) dan payload password otomatis disaring ([Filtered] / *****) untuk kepatuhan zero-leakage security.
- CallerSkipFrameCount memastikan log line menunjuk ke baris pemanggilan aktual di controller/usecase, bukan ke wrapper logger.
- Hub Sentry di-clone per request HTTP untuk mencegah data race antar thread/goroutine.

## 6. Related Knowledge

- [[01-Knowledge/patterns/backend/functional-router-registry-clean-architecture-bootstrapper.md]]
- Zero Leakage Observability

## 7. Source

- [[03-Sources/other/orchestrator-runs/harvest-1787715147310-7bd85203.json]]
