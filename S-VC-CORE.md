# 🏛️ S-VC Core v1.2 — Ядро системы

**Side_chaine_Vibe_Code** — протокол агентского governance для vibe-coding проектов.

---

## 1. РОЛИ

| Роль | Кодовое имя | Инстанс | Модель | Что делает |
|------|-------------|---------|--------|-----------|
| **Ты** | Product Manager | — | Человек | Ставишь задачи. WHAT & WHEN |
| **Architect** | **001** | Сессия (manual) | Тяжёлая | Архитектурные решения, TC, frozen override. Вызывается on-demand |
| **Scout** | **007** | 1-й | Лёгкая | Разведка, упаковка MACRO/MICRO, DOC-CHECK first pass, registry |
| **Operator** | — | 2-й, изолированный | Лёгкая | Слепое исполнение TC. Не видит контекста, только инструкцию |
| **Verifier** | **009** | 3-й | Лёгкая | Независимая верификация, DOC-CHECK second pass, registry validation |

### Ты

Ты — Product Manager. Ты решаешь **ЧТО** и **КОГДА** делать. Остальное делают агенты.

### Architect (001)

Архитектор. Включается когда:
- Нужно архитектурное решение
- Нарушение frozen zones
- Конфликт между Scout и Verifier
- Governance freeze / консолидация

**Не пишет код, не создаёт TC в рутине.**

### Scout (007)

Разведчик. Это главный рабочий агент:
- Получает задачу от тебя
- Собирает контекст (MACRO-PACK)
- Упаковывает инструкцию для Operator (MICRO-PACK)
- Проверяет результат после Operator
- Делает DOC-CHECK first pass
- Обновляет registry

**Не пишет код, не принимает архитектурных решений.**

### Operator

Слепой исполнитель. Запускается в изолированном контексте:
- Получает только MICRO-PACK (не видит MACRO-PACK)
- Холодная верификация перед apply
- Применяет TC строго по инструкции
- Запускает tsc + тесты
- Error = Stop (не чинит, докладывает)

**Максимум 2 файла за задачу. Не рефакторит. Не импровизирует.**

### Verifier (009)

Независимый верификатор. Запускается после Scout:
- Runtime audit — проверка что изменения работают
- DOC-CHECK second pass — перепроверяет документацию (независимо от Scout)
- Registry validation — проверка статусов
- FULL-BASE drift — проверка архитектурных изменений
- Random audit — каждые 10 TC полный аудит без чтения отчёта Scout

**Имеет право заблокировать COMPLETE. Не пишет код, не предлагает фиксы.**

---

## 2. ПРОТОКОЛ (3 фазы)

```
╔══════════════════════════════════════════════════════════╗
║  PHASE 1 — RECON                                        ║
╚══════════════════════════════════════════════════════════╝
1. ТЫ → задача
     ↓
2. 007 → MACRO-PACK (контекст + разведка + frozen guard)
     ↓
3. АРХИТЕКТОР* → анализирует → TC Roadmap (только если нужно)
     ↓
4. 007 → MICRO-PACK

╔══════════════════════════════════════════════════════════╗
║  PHASE 2 — EXECUTION                                    ║
╚══════════════════════════════════════════════════════════╝
5. OPERATOR (изолированный инстанс)
     ├── холодная верификация (grep anchors, tsc --noEmit)
     ├── применяет TC
     ├── tsc --noEmit + test --related
     └── отчёт 007
     ↓
6. 007 → верификация (git diff + build + sanity)

╔══════════════════════════════════════════════════════════╗
║  PHASE 3 — VERIFICATION                                 ║
╚══════════════════════════════════════════════════════════╝
7. 007 → DOC-CHECK first pass
     ├── определить затронутые домены
     ├── DOC_OK / UPDATE / CREATE
     ├── если нужно: сформировать DOC-TC
     ├── обновить registry
     └── передать эстафету 009
     ↓
8. 009 → независимая верификация
     ├── runtime verification
     ├── DOC-CHECK second pass (cross-audit)
     ├── registry validation
     └── FULL-BASE drift audit
     ↓
9. 009 → VERDICT
     ├── ✅ PASS → COMPLETE
     └── ❌ FAIL → return to 007 + Architect
```

* Architect — on-demand. Для простых TC (naming, contract fix) 007 выдаёт TC напрямую.

---

## 3. ПРИНЦИПЫ

| Принцип | Правило |
|---------|---------|
| **Изоляция** | 007 и Operator — разные инстансы. Operator слеп. |
| **Атомарность** | 1 TC = 1 правка = 1 MICRO-PACK |
| **Single writer** | Только Operator пишет код |
| **007 не трогает код** | Разведка и упаковка |
| **Architect не спускается в рутину** | Архитектура + TC |
| **Frozen не трогать** | Без OVERRIDE от Architect |
| **Error = Stop** | Не чинить, доложить |
| **Batch = within-домен** | Только один ownership |
| **Verify before apply** | Холодная верификация + tsc + тесты |
| **Doc-Aware** | Каждая задача завершается DOC-CHECK |
| **DOC-CHECK before COMPLETE** | Задача не COMPLETE без DOC-CHECK |
| **009 override** | 009 блокирует COMPLETE если DOC-CHECK FAIL |
| **Independent verification** | 009 верифицирует независимо от 007 |

---

## 4. 📦 ФОРМАТЫ ПАКЕТОВ

### MACRO-PACK (007 → Architect)

```
Задача → Архитектура → Frozen Guard → Код → Граф зависимостей → git diff → Состояние → Perf
```

Полный разведывательный пакет. Содержит:
- Задачу от PM
- Архитектурный саммари (выдержки из доков)
- ❄️ Frozen guard — пересекается ли с frozen зонами?
- Сжатый код (сигнатуры + релевантные строки)
- Граф зависимостей (imports, events, selectors)
- git diff (если уже есть изменения)
- State snapshot
- Perf budget (если применимо)

### MICRO-PACK (007 → Operator)

```
TC-XXX: ЦЕЛЬ → ФАЙЛ → ЗОНА → ДЕЙСТВИЕ → КОНТЕКСТ → ЗАПРЕЩЕНО → ТЕСТ
```

Точная инструкция для Operator:
- **ЦЕЛЬ**: одна фраза
- **ФАЙЛ**: path/to/file.ts
- **ЗОНА**: строки N-M
- **ДЕЙСТВИЕ**: точный код
- **КОНТЕКСТ**: 3 строки до/после
- **ЗАПРЕЩЕНО**: что не трогать
- **ТЕСТ**: tsc --noEmit && test --related

### VERDICT (009 → PM)

```
VERDICT: ✅ PASS / ❌ FAIL
RUNTIME:    ✅ OK / ❌ [детали]
DOC-CHECK:  ✅ OK / ❌ [детали]
REGISTRY:   ✅ OK / ❌ [детали]
FULL-BASE:  ✅ OK / ❌ [детали]
```

---

## 5. ❄️ FROZEN ZONES

Frozen zones — части кода или архитектуры, которые **нельзя менять без решения Architect'а**.

### Как это работает

1. Каждый проект определяет свой список frozen zones
2. Frozen zones записываются в charter-007.md
3. Перед каждой задачей 007 проверяет: задача пересекается с frozen?
4. Если да — вызывает Architect для OVERRIDE
5. Без OVERRIDE — задача не выполняется

### Примеры frozen zones

```yaml
# Каждый проект определяет свои
frozen_zones:
  - "src/core/*"           # Ядро системы
  - "src/bridges/*"        # Слой интеграций
  - "database/schema/"     # Схема БД
  - "api/contracts/"       # API контракты
```

---

## 6. 📊 REGISTRY

Registry — YAML-файл, который хранит состояние документации.

### Формат

```yaml
version: 2
last_updated: 2026-06-15

sync_health:
  current:
    synced: 25
    needs_update: 3
    stale: 1
    total: 29
    score: 86

documents:
  - id: auth-system
    path: docs/auth-system.md
    domain: auth
    status: synced
    drift: none
    last_audit: 2026-06-10

  - id: api-contracts
    path: docs/api-contracts.md
    domain: api
    status: needs_update
    drift: minor — endpoint names changed
    doc_tc:
      - id: DOC-TC-003
        status: open
```

### Что даёт registry

- Единая карта документации проекта
- Health score — насколько docs актуальны
- История изменений
- DOC-TC backlog — задачи на обновление документации

---

## 7. 📝 DOC-AWARE DEVELOPMENT

**Каждая задача завершается проверкой документации.** Это не опционально.

### DoD (Definition of Done)

Задача COMPLETE только после:

```
✅ [MANDATORY] 1. Реализация Operator'ом завершена
✅ [MANDATORY] 2. Верификация Scout: git diff + tsc + тесты
✅ [MANDATORY] 3. DOC-CHECK (007 first pass)
✅ [MANDATORY] 4. Registry updated
✅ [MANDATORY] 5. FULL-BASE drift check
✅ [MANDATORY] 6. 009 independent verification
✅ [CONDITIONAL] 7. DOC-TC resolved (если был создан)
```

### DOC-CHECK шаги (007 first pass)

1. Определить затронутые домены
2. Проверить документацию — существует? Актуальна?
3. Классифицировать: `DOC_OK` / `DOC_UPDATE_REQUIRED` / `DOC_CREATE_REQUIRED`
4. Если UPDATE/CREATE — сформировать DOC-TC
5. Обновить registry
6. Передать эстафету 009

### DOC-CHECK second pass (009)

- Перепроверяет документацию **независимо** от 007
- Подтверждает DOC_OK / находит пропущенный drift
- Проверяет registry на корректность статусов

---

## 8. 📜 ЧАРТЕРЫ

Чартеры — детальные инструкции для каждого агента.

| Чартер | Для кого | Содержит |
|--------|----------|---------|
| `charter-007.md` | Scout (007) | Разведка, MACRO/MICRO форматы, frozen zones, DOC-CHECK протокол |
| `charter-009.md` | Verifier (009) | Independent verification, VERDICT, random audit, maintenance cycle gatekeeper |
| `charter-operator.md` | Operator | Blind execution, cold verification, error=stop, MAX 2 files |

Каждый чартер определяет:
- **КТО** агент
- **ЧТО ДЕЛАЕТ** и **ЧТО НЕ ДЕЛАЕТ**
- **ПРОТОКОЛ** — точный формат работы
- **ПРАВИЛА** — hard rules
- **ПЕРВЫЙ ШАГ** — что читать при старте

---

*S-VC Core 1.2 — June 2026*
