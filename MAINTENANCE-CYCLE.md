# 🔄 S-VC MAINTENANCE CYCLE

Регулярный контур самоочистки системы. S-VC сама находит мёртвый груз и чистит его — с доказательствами и независимой верификацией.

---

## Purpose

Со временем в любом проекте накапливается:
- Мёртвый код (dead code)
- Устаревшая документация (stale docs)
- Неиспользуемые файлы
- Архитектурный дрифт

Maintenance Cycle — это процесс автоматической очистки и консолидации.

---

## Trigger

Цикл запускается одним из способов:

| Триггер | Как |
|---------|-----|
| **Пользователь** | ``001, Optimization!`` или ``001 — Оптимизация!`` |
| **Scout (007)** | Обнаружил проблему: stale docs, dead code, low health score |
| **Health score** | Registry health score упал ниже 75 |
| **Регулярно** | Раз в неделю / после крупного TC |

### Trigger Ownership Rule

При распознанном триггере агенты **не имеют права спрашивать** "что делать?".
Они автономно запускают фазы цикла.

---

## 5 фаз

```
┌────────────────────────────────────────────────────────────────┐
│  A (Detect)    001 — аудит системы                             │
│       ↓                                                        │
│  B (Propose)   007 — сбор Optimization Plan                    │
│       ↓                                                        │
│  C (Approve)   009 — проверка и одобрение плана               │
│       ↓                                                        │
│  D (Execute)   007 + Operator — применение TC                   │
│       ↓                                                        │
│  E (Writeback) 007 — обновление registry, log, FULL-BASE      │
│       ↓                                                        │
│  Final Verify  009 — VERDICT                                   │
└────────────────────────────────────────────────────────────────┘
```

### Phase A — Detect (001)

001 проводит аудит системы:
- Сканирование проекта: структура, файлы, документация
- Проверка registry: health score, drift
- Поиск dead code, stale docs, architectural drift
- Формирование AUDIT-REPORT

**Результат:** `AUDIT-REPORT` → 007

### Phase B — Propose (007)

007 для каждого найденного кандидата:
1. **Доказывает** — grep/export что код/document мёртв
2. **Оценивает impact** — кто зависит, кто сломается
3. **Оценивает risk** — 🟢 низкий / 🟡 средний / 🔴 высокий

Формирует **Optimization Plan**:
- Scope (что меняется)
- TC list (по одному на изменение)
- Expected gain (строк, файлов, сложности)
- Rollback plan (как откатить)

**Результат:** `Optimization Plan` → 009

### Phase C — Approve (009)

009 проверяет каждый candidate в плане:

| Критерий | Что проверяет |
|----------|--------------|
| **Proof** | Доказательство достаточно? Не "кажется", а факты? |
| **Scope** | Чётко определён? Docs ≠ code в разных TC? |
| **Risk** | Корректен? Rollback plan есть? |
| **Bias** | 007 мог ошибиться? Мог пропустить зависимость? |

**Результат:**
- ✅ **APPROVE** — весь план принят → Phase D
- 🟡 **PARTIAL APPROVE** — часть принята, часть отклонена
- ❌ **BLOCK** — весь план отклонён (причина обязательна)
- 🔄 **REQUEST MORE PROOF** — стоп-сигнал, нужны доказательства

### Phase D — Execute (007 + Operator)

1. 007 создаёт MICRO-PACK для каждого approved TC
2. Operator применяет TC строго по инструкции
3. 007 верифицирует после каждого TC:
   - git diff — только target changes
   - tsc --noEmit — 0 новых ошибок
   - tests — все проходят

**Результат:** `EXECUTION-REPORT` → 007

### Phase E — Writeback (007)

1. Обновление registry
2. Обновление DOC-TC backlog
3. Обновление FULL-BASE (только если архитектурные изменения)
4. Формирование `S-VC-MAINTENANCE-REPORT`
5. Вызов 009 для Final Verify

**Результат:** `S-VC-MAINTENANCE-REPORT` → 009

### Final Verify (009)

009 проверяет:
- Удаления были proof-based
- Ничего важного не сломалось
- Health score registry вырос
- Выносит VERDICT

---

## Autonomous Continuation

После завершения каждой фазы следующий агент вызывается **автоматически**. Пользователь не участвует в последовательности фаз.

```
Никита: "001, Optimization!"
    ↓
007 распознаёт триггер → вызывает 001 (Phase A)
    ↓
001 завершает Detect → AUDIT-REPORT → 007 (Phase B)
    ↓
007 завершает Propose → Optimization Plan → 009 (Phase C)
    ↓
009 завершает Approve → APPROVED → 007 + Operator (Phase D)
    ↓
007 + Operator завершают Execute → EXECUTION-REPORT → 007 (Phase E)
    ↓
007 завершает Writeback → вызывает 009 (Final Verify)
    ↓
009 завершает Verify → VERDICT
```

---

## Для чего это нужно

- **Самоочистка** — система не накапливает мусор
- **Доказательства** — ничего не удаляется без proof
- **Безопасность** — 009 как gatekeeper
- **Health** — registry показывает реальное состояние

---

## Пример Maintenance Report

```markdown
# S-VC MAINTENANCE REPORT — Cycle 01

## Summary
- Phase A (Detect): 001 — найдено 4 кандидата
- Phase B (Propose): 007 — план из 4 TC
- Phase C (Approve): 009 — APPROVED WITH CONDITIONS
- Phase D (Execute): 4 TC выполнено
- Phase E (Writeback): registry обновлён

## TC Applied
| TC | Что сделано | Gain | Risk |
|----|------------|------|------|
| TC-OPT-01 | Удалён dead component | -39 строк | 🟢 Low |
| TC-OPT-02 | Обновлена устаревшая документация | DOC-TC closed | 🟢 Low |
| TC-OPT-03 | Обновлена устаревшая документация | DOC-TC closed | 🟢 Low |
| TC-OPT-04 | Health score registry | 82 → 86 | 🟢 Low |

## ROI
- -39 строк кода
- 3 DOC-TC closed (из 10 → 7)
- Health score: 82 → 86
- Время: 40 минут
```

---

*S-VC Core 1.2 — June 2026*
