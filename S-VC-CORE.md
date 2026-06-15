# 🏛️ S-VC Core v1.2 — Ядро системы

**Side_chaine_Vibe_Code** — протокол агентского governance для vibe-coding проектов.

---

## 1. РОЛИ

| Роль | Кодовое имя | Инстанс | Рекомендуемая модель | Что делает |
|------|-------------|---------|---------------------|-----------|
| **Ты** | Product Manager | — | Человек | Ставишь задачи. WHAT & WHEN |
| **Architect** | **001** | Сессия (manual) | `opencode-go/deepseek-v4-flash` | Внешний оркестратор. Аудит, frozen override |
| **Scout** | **007** | 1-й | `opencode-go/deepseek-v4-flash` | Разведка, MACRO/MICRO, DOC-CHECK |
| **Operator** | — | 2-й, изолированный | `opencode-go/deepseek-v4-flash` | Слепое исполнение TC |
| **Verifier** | **009** | 3-й | `opencode-go/kimi-k2.6` | Независимая верификация, registry |

**Принцип:** Roles are mandatory. Models are replaceable.
У тебя может быть другой стек моделей — просто замени ID в `.opencode/agent/*.md`.

---

## 1.1 🔄 Model Routing — как назначены модели

### Стратегия

**Модель НЕ переключается на ходу внутри одной сессии OpenCode.**  
Если нужно другое поведение — отдельный вызов / отдельная сессия / явный agent config.

Поэтому назначение фиксированное:

| Роль | Модель | Статус |
|------|--------|--------|
| **001** (Architect) | `opencode-go/deepseek-v4-flash` | 🪶 Внешний оркестратор |
| **007** (Scout) | `opencode-go/deepseek-v4-flash` | 🪶 Разведка |
| **009** (Verifier) | `opencode-go/kimi-k2.6` | 🏋️ Единственная Kimi-роль |
| **Operator** | `opencode-go/deepseek-v4-flash` | 🪶 Исполнение |

### 001 — внешняя роль

001 **не является subagent** в runtime. Он:
- Получает **AUDIT-PACK** (compact pack, а не весь контекст)
- Анализирует, принимает решение
- Инициирует следующую фазу
- Не прыгает между моделями

### 009 — единственный subagent с Kimi

009 получает **VERDICT-PACK**:
- Краткое описание задачи
- Diff summary
- Что проверять
- Никакого лишнего контекста

### Compact Pack Protocol

**AUDIT-PACK (для 001):**
```markdown
Goal:
Scope:
Candidates:
Proof:
Risks:
Recommended action:
Questions for 009:
```

**VERDICT-PACK (для 009):**
```markdown
Goal:
Changed files:
007 findings:
Diff summary:
Known risks:
What to verify:
Expected verdict:
```

### Известное ограничение OpenCode (подтверждено тестом)

**`model:` в frontmatter НЕ переключает модель при subagent-вызове.**

Подтверждено: 009.md с `model: opencode-go/kimi-k2.6` существует, но 009 запускается на модели родителя (007). Subagent наследует модель сессии.

**Решение:** Установи нужную модель как default в настройках OpenCode перед сессией. Временно переключи default на Kimi, когда нужна верификация.

### Почему DeepSeek для 001, 007, Operator?

- Бесплатный (в OpenCode)
- Быстрый — не требует глубоких рассуждений для рутины
- 001 внешний, 007 собирает контекст, Operator применяет TC

### Почему Kimi только для 009?

- Аналитическая — хорошо ловит несостыковки
- Единственная роль, где качество важнее скорости
- 009 проверяет результат всей команды

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
     ├── DOC-CHECK second pass (перепроверка)
     ├── registry validation
     └── VERDICT: ✅ PASS / ❌ FAIL / 🔄 REVISE
     ↓
9. Если ✅ PASS → COMPLETE
```

---

## 3. ПРИНЦИПЫ

1. **Ты диктуешь WHAT & WHEN. Агенты диктуют HOW.**
2. **007 собирает контекст. АРХИТЕКТОР решает. OPERATOR исполняет. 009 проверяет.**
3. **Ты не повторяешь контекст. Агент уже прочитал FULL-BASE.**
4. **Один TC — одно изменение.**
5. **Оператор без контекста — намеренно.**
6. **009 блокирует COMPLETE если нашёл проблему.**
7. **Error = Stop. Не чинить на ходу — сообщить.**
8. **Frozen zones — только с OVERRIDE.**
9. **DOC-CHECK после каждого TC.**
10. **Maintenance Cycle — proof-based, без новых документов.**
11. **Roles are mandatory. Models are replaceable.**

---

## 4. 📦 ФОРМАТЫ ПАКЕТОВ

### MACRO-PACK (007 → Architect)

```markdown
# 📦 MACRO-PACK: task-xxx
От: 007
Для: Архитектор

## 🎯 ЗАДАЧА
[от тебя]

## 🏗️ АРХИТЕКТУРНЫЙ САММАРИ
[контекст из доков]

## ❄️ FROZEN GUARD
🚨 FROZEN INTRUSION / 🚧 PARTIAL / ✅ OK

## 📄 СЖАТЫЙ КОД
[сигнатуры + релевантные строки]

## 🔗 ГРАФ ЗАВИСИМОСТЕЙ
[imports, event bus, selectors]
```

### MICRO-PACK (007 → Operator)

```markdown
📦 MICRO-PACK: TC-XXX
ЦЕЛЬ: [одна фраза]
ФАЙЛ: path/to/file.ts
ЗОНА: строки N-M
ДЕЙСТВИЕ: [точный код]
КОНТЕКСТ: [3 строки до/после]
ЗАПРЕЩЕНО: [что не трогать]
ТЕСТ: tsc --noEmit && vitest --related
```

### VERDICT (009 → PM)

```markdown
VERDICT: ✅ PASS / ❌ FAIL / 🔄 REVISE
RUNTIME:    ✅ OK / ❌ FAIL
DOC-CHECK:  ✅ OK / ❌ UPDATE / ❌ CREATE
REGISTRY:   ✅ OK / ❌ STALE
FULL-BASE:  ✅ OK / ❌ DRIFT
NOTES: [что нашёл]
```

---

## 5. ❄️ FROZEN ZONES

### Как это работает

Каждый проект определяет свои frozen zones — код, который нельзя менять без Architect.

```
charter-007.md хранит список frozen зон проекта.
Изменение frozen зоны требует OVERRIDE от Architect (001).
```

### Пример

```yaml
# agents/charters/charter-007.md
❄️ src/core/engine.ts         — транспорт
❄️ src/bridges/               — слой синхронизации
❄️ src/legacy/                — legacy boundary shells
```

---

## 6. 📊 REGISTRY

### Формат

```yaml
# docs/sync/MASTER-SYNC-REGISTRY.yaml
sync_health:
  current:
    synced: 15
    stale: 1
    total: 16
    score: 93

documents:
  - id: some-doc
    status: synced
    last_synced: 2026-06-10
```

### Что даёт registry

- Прозрачность: какие документы актуальны, какие нет
- DOC-CHECK привязывается к реальному состоянию
- Maintenance Cycle находит stale/orphan docs

---

## 7. 📝 DOC-AWARE DEVELOPMENT

### DoD (Definition of Done)

Задача считается выполненной только после:

1. TC реализован
2. tsc --noEmit — 0 новых ошибок
3. tests — проходят
4. DOC-CHECK — first pass (007)
5. Registry — обновлён
6. 009 VERDICT — ✅ PASS

### DOC-CHECK шаги (007 first pass)

1. Определить затронутые домены
2. Проверить документацию
3. DOC_OK / DOC_UPDATE_REQUIRED / DOC_CREATE_REQUIRED
4. Сформировать DOC-TC если нужно
5. Обновить MASTER-SYNC-REGISTRY

### DOC-CHECK second pass (009)

Независимая перепроверка документации и registry. 009 не читает отчёт 007.

---

## 8. 📜 ЧАРТЕРЫ

| Файл | Назначение |
|------|-----------|
| `charter-007.md` | Правила разведчика (Scout). MACRO/MICRO, DOC-CHECK, frozen guard |
| `charter-009.md` | Правила верификатора (Verifier). VERDICT, random audit, gatekeeper |
| `charter-operator.md` | Правила исполнителя (Operator). Blind execution, error=stop |

---

## 9. 🔄 S-VC MAINTENANCE CYCLE

Регулярный цикл самоочистки системы.

**Триггер:** Пользователь пишет `001, Optimization!`

**Фазы:** A (Detect → 001) → B (Propose → 007) → C (Approve → 009) → D (Execute → Operator) → E (Writeback → 007) → Final Verify (009)

Подробно: `MAINTENANCE-CYCLE.md`
