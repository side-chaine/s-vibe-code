# 🔄 MODEL-ROUTING-GUIDE — Как назначить модели ролям S-VC

## Философия

**Модель НЕ переключается на ходу внутри одной сессии OpenCode.**
Если нужно другое поведение — нужен отдельный вызов, отдельная сессия или явный agent config.

Поэтому S-VC использует фиксированное назначение:

| Роль | Модель | Почему |
|------|--------|--------|
| **001** (Architect) | 🪶 DeepSeek V4 Flash | Внешний оркестратор. Не прыгает между моделями |
| **007** (Scout) | 🪶 DeepSeek V4 Flash | Разведка, упаковка — дёшево и быстро |
| **009** (Verifier) | 🏋️ Kimi K2.6 | Единственный subagent с аналитической моделью |
| **Operator** | 🪶 DeepSeek V4 Flash | Слепое исполнение — без лишних рассуждений |

---

## 1. Посмотри доступные модели

```bash
opencode models
```

Пример вывода:
```
opencode-go/deepseek-v4-flash
opencode-go/kimi-k2.6
opencode-go/kimi-k2.7-code
...
```

## 2. Назначь модели ролям

Отредактируй `.opencode/agent/*.md`, укажи `model:` с ID из `opencode models`.

**001.md** — внешняя роль, быстрая модель:
```yaml
model: opencode-go/deepseek-v4-flash
```

**007.md** — разведка, быстрая модель:
```yaml
model: opencode-go/deepseek-v4-flash
```

**009.md** — верификация, аналитическая модель:
```yaml
model: opencode-go/kimi-k2.6
```

**operator.md** — исполнение, быстрая модель:
```yaml
model: opencode-go/deepseek-v4-flash
```

## 3. Compact Pack Protocol

Так как модель не переключается на ходу, передача контекста между ролями делается через **compact packs** — сжатые пакеты без лишнего контекста.

### 001 получает AUDIT-PACK

```markdown
# AUDIT-PACK

Goal:
Scope:
Candidates:
Proof:
Risks:
Recommended action:
Questions for 009:
```

### 009 получает VERDICT-PACK

```markdown
# VERDICT-PACK

Goal:
Changed files:
007 findings:
Diff summary:
Known risks:
What to verify:
Expected verdict:
```

### Что НЕ передавать
- ❌ Весь FULL-BASE
- ❌ Весь history чата
- ❌ Лишние обсуждения
- ❌ Длинные отчёты без сжатия

## 4. Перезапусти OpenCode

Модель применяется при **новой сессии**. Закрой и открой OpenCode заново.

## 5. Проверь что получилось

Запусти задачу. В UI:

```
▣ 001 · DeepSeek V4 Flash
▣ 007 · DeepSeek V4 Flash
▣ 009 · Kimi K2.6
▣ Operator · DeepSeek V4 Flash
```

## ⚠️ Известные ограничения OpenCode

1. **Subagent наследует модель родителя.** При `task(subagent_type="009")` OpenCode может не переключить модель на указанную в 009.md. Проверяй в UI.

2. **001 — внешняя роль.** Он не вызывается через `task()` в runtime. Его конфигурация — для сессий где он primary.

3. **Если модель не переключилась:** Установи нужную модель как default в настройках OpenCode.

## 🎯 Presets для других платформ

### Claude Code
```yaml
001: claude-sonnet-4
007: claude-haiku-4.5
009: claude-sonnet-4
Operator: claude-haiku-4.5
```

### Cursor
```yaml
001: gpt-4o
007: gpt-4o-mini
009: gpt-4o
Operator: gpt-4o-mini
```

### Custom (OpenRouter)
```yaml
001: openrouter/deepseek/deepseek-v4-flash
007: openrouter/deepseek/deepseek-v4-flash
009: openrouter/moonshotai/kimi-k2.6
Operator: openrouter/deepseek/deepseek-v4-flash
```
