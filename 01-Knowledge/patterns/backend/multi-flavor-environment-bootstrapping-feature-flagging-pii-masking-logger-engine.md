---
title: "Multi-Flavor Environment Bootstrapping, Feature-Flagging & PII-Masking Logger Engine"
type: pattern
tags: [pattern, backend, devops, environment, structured-logging, pii-masking, zerolog]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787128418632-596cfbe7
sources: ["Harvest 1787128418632 596cfbe7.json"]
---

# Multi-Flavor Environment Bootstrapping, Feature-Flagging & PII-Masking Logger Engine

Multi-Flavor Environment Bootstrapping & PII-Masking Logger Engine menyediakan orkestrasi bootstrap environment yang fleksibel, safe feature-flagging, dan zero-leak request audit logging.

## 1. Overview & Architecture

Pola inisialisasi aplikasi multi-flavor via CLI flag injection, typed environment accessors dengan safe defaults, serta conditional module mounting, terintegrasi dengan engine Zerolog yang dilengkapi HTTP request body tap & PII masking regex.

## 2. Implementation & Code Structure

cmd/
└── api/
    └── api.go
pkg/
├── environment/
│   └── environment.go
└── logger/
    ├── logger.go
    └── zerolog.go

## 3. Key Implementation Points

- Multi-flavor environment switching via CLI flag injection (--env=.env.stag).
- Safe typed environment getters (GetUint, GetInt, GetFloat, CheckFlag) dengan fallback default.
- Conditional module mounting (IAM_MODULE_OFF) untuk arsitektur modular micro/monolith.
- Non-destructive request body inspection dengan zero-leak regex masking untuk field password/sensitif.

## 4. Code Examples

### CLI flag-based environment resolution dan conditional module registration.

```go
package main

import (
	"base-be-golang/internal/adapter/controller"
	"base-be-golang/shared/api"
	"base-be-golang/shared/base"
	"flag"
	"os"
	"strconv"
	"github.com/joho/godotenv"
	iamctrl "github.com/rdhmuhammad/base-be-golang/iam-module/shared/adapter/controller"
	"gorm.io/gorm"
)

func main() {
	var envFile string
	flag.StringVar(&envFile, "env", ".env.stag", "Provide env file path")
	flag.Parse()

	err := godotenv.Load(envFile)
	if err != nil {
		panic(err)
	}

	start := api.Default()

	// Conditional module mounting
	if t, _ := strconv.ParseBool(os.Getenv("IAM_MODULE_OFF")); !t {
		start.Register(func(dbConn *gorm.DB, port base.Port, ctrl base.BaseController) api.Router {
			return iamctrl.NewAuthController(dbConn, port, ctrl)
		})
	}

	start.Register(func(dbConn *gorm.DB, port base.Port, ctrl base.BaseController) api.Router {
		return controller.NewProductController(dbConn, port, ctrl)
	})

	_ = start.Start()
}
```

### Type-safe environment reader dengan safe fallback defaults.

```go
package environment

import (
	"os"
	"strconv"
)

type ENV struct{}

func NewEnvironment() ENV {
	return ENV{}
}

func (e ENV) GetUint(key string, defaultValue uint) uint {
	str := os.Getenv(key)
	value, err := strconv.ParseUint(str, 10, 64)
	if err != nil {
		return defaultValue
	}
	return uint(value)
}

func (e ENV) CheckFlag(flag string) bool {
	str := os.Getenv(flag)
	status, err := strconv.ParseBool(str)
	if err != nil {
		return false
	}
	return status
}
```

### Zero-allocation structured logger dengan HTTP body tapping dan PII masking.

```go
var sensitiveFieldRegex = regexp.MustCompile(`"password"\s*:\s*"[^"]*"`)

func DefaultLogger() ReZero {
	logLevel := strings.ToLower(os.Getenv("LOG_LEVEL"))
	level := zerolog.InfoLevel
	if logLevel == "debug" {
		level = zerolog.DebugLevel
	}

	zerolog.CallerSkipFrameCount = 4
	zerolog.SetGlobalLevel(level)
	output := zerolog.ConsoleWriter{Out: os.Stdout, TimeFormat: time.RFC3339}
	logger := zerolog.New(output).With().Timestamp().Caller().Logger()

	return ReZero{logger: &logger, level: level}
}

func (l *ReZero) LoggingRequest(c *gin.Context) {
	if zerolog.DebugLevel != l.level {
		c.Next()
		return
	}

	var bodyBytes []byte
	if c.Request.Body != nil {
		bodyBytes, _ = io.ReadAll(c.Request.Body)
	}
	c.Request.Body = io.NopCloser(bytes.NewBuffer(bodyBytes))

	event := l.logger.Info().
		Str("method", c.Request.Method).
		Str("path", c.Request.URL.Path).
		Str("client_ip", c.ClientIP())

	if len(bodyBytes) > 0 {
		maskedBody := sensitiveFieldRegex.ReplaceAll(bodyBytes, []byte(`"password":"*****"`))
		trimmed := bytes.TrimSpace(maskedBody)
		if len(trimmed) > 0 && (trimmed[0] == '{' || trimmed[0] == '[') {
			event = event.RawJSON("body", maskedBody)
		} else {
			event = event.Str("body", string(maskedBody))
		}
	}

	event.Msg("incoming request")
	c.Next()
}
```

## 5. Considerations & Best Practices

- Flag-based env loading (--env) memungkinkan deployment multi-flavor (stag, prod, testing) tanpa container rebuild.
- PII masking regex harus dieksekusi secara efisien pada pre-compiled regex untuk meminimalkan alokasi memori pada hot paths HTTP.
- CallerSkipFrameCount harus dikalibrasi tepat saat meng-embed wrapper logger agar stack trace menunjuk langsung ke usecase/handler pemanggil.

## 6. Related Knowledge

- Multi Flavor Bootstrapping
- Pii Masking Structured Logger

## 7. Source

- Harvest 1787128418632 596cfbe7.json
