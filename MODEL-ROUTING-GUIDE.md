# 🔄 MODEL-ROUTING-GUIDE — Как назначить модели ролям S-VC

## 1. Посмотри доступные модели

В терминале:

```bash
opencode models
```

Ты увидишь список ID моделей. Пример:

```
opencode-go/deepseek-v4-flash
opencode-go/kimi-k2.6
opencode-go/kimi-k2.7-code
openrouter/anthropic/claude-sonnet-4
openrouter/openai/gpt-5
...
```

## 2. Выбери схему под себя

### 🪙 Budget (дёшево)
```yaml
001 → opencode-go/kimi-k2.6        # архитектура
007 → opencode-go/deepseek-v4-flash  # разведка
009 → opencode-go/kimi-k2.6          # верификация
Operator → opencode-go/deepseek-v4-flash # исполнение
```

### ⚖️ Balanced (универсально)
```yaml
001 → openrouter/anthropic/claude-sonnet-4
007 → opencode-go/deepseek-v4-flash
009 → opencode-go/kimi-k2.7-code
Operator → opencode-go/deepseek-v4-flash
```

### 💎 Premium (максимум)
```yaml
001 → openrouter/anthropic/claude-opus-4
007 → openrouter/anthropic/claude-sonnet-4
009 → opencode-go/kimi-k2.7-code
Operator → openrouter/anthropic/claude-haiku-4.5
```

## 3. Запиши модели в agent-файлы

Каждая роль — файл в `.opencode/agent/`:

```bash
# Пример: назначить 001 на Kimi
echo '---
description: "001 — CEO Co-Architect"
model: opencode-go/kimi-k2.6
mode: all
---' > .opencode/agent/001.md
```

Или открой файл в редакторе и добавь строку:
```yaml
model: opencode-go/kimi-k2.6
```

## 4. Перезапусти OpenCode

Модель применяется при **новой сессии**. Закрой и открой OpenCode заново.

## 5. Проверь что получилось

Запусти задачу. В UI ты увидишь:

```
▣ 001 · Kimi K2.6
▣ 007 · DeepSeek V4 Flash
```

## ⚠️ Известные ограничения

### Subagent наследует модель родителя

**Факт:** Когда 007 вызывает `task(subagent_type="009")`, OpenCode может не переключить модель на указанную в 009.md. Subagent запускается на той же модели, что и родитель (007).

**Причина:** Платформенное ограничение OpenCode. `model:` в frontmatter агента не гарантирует переключение при subagent-вызове.

**Что делать:**
1. **Самый надёжный способ:** Установи default-модель в OpenCode под задачу
2. **Если нужна другая модель для 009:** Запусти отдельную сессию OpenCode с нужной default-моделью
3. **model: в frontmatter** — может сработать, но не гарантировано. Проверяй в UI

### Рекомендация

Не борись с платформой. S-VC Core работает на одной модели. Роли определяют **поведение** (кто что читает, пишет, проверяет), а не LLM под капотом.

```
Roles define behavior, not models.
```

Если тебе критично иметь разные модели для разных ролей — выбери платформу, которая это поддерживает, и адаптируй S-VC под неё.
