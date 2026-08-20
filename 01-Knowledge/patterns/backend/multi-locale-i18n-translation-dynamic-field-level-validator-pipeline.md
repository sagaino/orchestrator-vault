---
title: "Multi-Locale i18n Translation & Dynamic Field-Level Validator Pipeline"
type: pattern
tags: [pattern, backend, i18n, localization, validator, reflection-binder]
created: 2026-08-19
updated: 2026-08-19
orchestrator_run: harvest-1787128418632-596cfbe7
sources: ["Harvest 1787128418632 596cfbe7.json"]
---

# Multi-Locale i18n Translation & Dynamic Field-Level Validator Pipeline

Multi-Locale i18n Translation & Dynamic Field-Level Validator Pipeline mengotomatisasi lokalisasi pesan API dan validasi struct berbasis refleksi dan tag kustom.

## 1. Overview & Architecture

Pipeline validasi input dan lokalisasi terintegrasi yang menggabungkan go-playground/validator/v10 dan go-i18n/v2 dengan otomatisasi scanning berkas JSON katalog bahasa, custom validator tags, dan binding query reflection.

## 2. Implementation & Code Structure

pkg/
├── localize/
│   └── localize.go
└── middleware/
    ├── custom-translation-validator.go
    ├── custome-validation.go
    └── validator.go

## 3. Key Implementation Points

- Otomatisasi pemuatan catalog JSON locale dari resource/message/*.json ke dalam i18n Bundle.
- Translasi pesan error validasi berbasis go-playground universal-translator.
- Custom validation tags (enum, monthyearformat) dengan custom message override.
- Reflection query binder (bindQuery) dengan automatic type coercion (UUID, timestamp with timezone, uint, bool).

## 4. Code Examples

### Multi-language bundle loader dan dynamic template interpolator.

```go
package localize

import (
	"base-be-golang/pkg/environment"
	"encoding/json"
	"os"
	"path"
	"path/filepath"
	"strings"
	"github.com/nicksnyder/go-i18n/v2/i18n"
	"golang.org/x/text/language"
)

type lang struct {
	bundle    *i18n.Bundle
	localizer map[string]*i18n.Localizer
	env       environment.ENV
}

func NewLanguage(basePath string) Language {
	defaultBundle := i18n.NewBundle(language.English)
	defaultBundle.RegisterUnmarshalFunc("json", json.Unmarshal)

	files, _ := os.ReadDir(basePath)
	var localizer = make(map[string]*i18n.Localizer)

	for _, f := range files {
		if filepath.Ext(f.Name()) == ".json" {
			filePath := path.Join(basePath, f.Name())
			_, _ = defaultBundle.LoadMessageFile(filePath)
			localCode := strings.Split(f.Name(), ".")[0]
			localizer[localCode] = i18n.NewLocalizer(defaultBundle, localCode, language.English.String())
		}
	}

	return &lang{
		bundle:    defaultBundle,
		localizer: localizer,
		env:       environment.NewEnvironment(),
	}
}

func (l *lang) GetLocalized(langCode string, messageId string, templates ...TemplatingData) string {
	if langCode == "" {
		langCode = l.env.Get("FALLBACK_LANG")
	}
	templateMap := make(map[string]string, len(templates))
	for _, t := range templates {
		templateMap[t.Name] = t.Value
	}

	loc, ok := l.localizer[langCode]
	if !ok {
		loc = l.localizer["en"]
	}
	res, err := loc.Localize(&i18n.LocalizeConfig{
		MessageID:    messageId,
		TemplateData: templateMap,
	})
	if err != nil {
		return messageId
	}
	return res
}
```

### Reflection-driven query parameter binding and validation engine.

```go
func (v Enigma) queryToFilter(c *gin.Context, payload interface{}, isDive bool) error {
	pVal, pType, vals, err := v.preparingReflection(payload, isDive)
	if err != nil {
		return err
	}

	for i := 0; i < vals.NumField(); i++ {
		field := pType.Elem().Field(i)
		tagKeyVal := strings.Split(field.Tag.Get("bindQuery"), ";")

		tags := map[string]string{}
		for _, item := range tagKeyVal {
			if kv := strings.Split(item, "="); len(kv) == 2 {
				tags[kv[0]] = kv[1]
			}
		}

		reqField := field.Tag.Get("json")
		if rn, ok := tags[reqName]; ok {
			reqField = rn
		}

		reqVal := getFromRequest(c, tags[reqPlace], reqField)
		if binder, ok := bindingRules[tags[dataType]]; ok {
			finalVal, err := binder(c, reqVal, tags)
			if err != nil {
				return err
			}
			vals.Field(i).Set(reflect.ValueOf(finalVal))
			continue
		}
		vals.Field(i).Set(reflect.ValueOf(reqVal))
	}
	return nil
}
```

### Custom validation rule tag dan error translation override.

```go
func OverrideTranslation(engine *validator.Validate, trans ut.Translator) error {
	err := engine.RegisterTranslation("trx-status", trans, func(ut ut.Translator) error {
		return ut.Add("trx-status", "{0} bukan status yang valid", true)
	}, func(ut ut.Translator, fe validator.FieldError) string {
		t, _ := ut.T("trx-status", fe.Field())
		return t
	})
	if err != nil {
		return err
	}

	return engine.RegisterTranslation("monthyearformat", trans, func(ut ut.Translator) error {
		return ut.Add("monthyearformat", "{0} format harus mm-yyyy", true)
	}, func(ut ut.Translator, fe validator.FieldError) string {
		t, _ := ut.T("monthyearformat", fe.Field())
		return t
	})
}
```

## 5. Considerations & Best Practices

- Katalog i18n dimuat ke dalam memori saat bootstrap; penambahan file JSON baru di disk memerlukan server reload.
- Timezone-aware date parsing mengekstrak data lokasi dari context token autentikasi user untuk parsing timestamp query yang presisi.

## 6. Related Knowledge

- I18n Localization
- Validator Reflection Binder

## 7. Source

- Harvest 1787128418632 596cfbe7.json
