# ⚡ QUICK-START: Подключи S-VC к своему проекту за 5 минут

## 1. Склонируй S-VC Core

```bash
git clone https://github.com/side-chaine/s-vibe-code.git
```

## 2. Скопируй AGENTS.md в корень проекта

```bash
cp s-vibe-code/AGENTS.md ./AGENTS.md
```

Это единственный обязательный файл. OpenCode читает `AGENTS.md` в корне проекта и настраивает агента по нему.

## 3. Скопируй чартеры

```bash
mkdir -p agents/charters
cp s-vibe-code/charter-007.md agents/charters/
cp s-vibe-code/charter-009.md agents/charters/
cp s-vibe-code/charter-operator.md agents/charters/
```

Чартеры — детальные инструкции для каждого агента. Без них агенты не знают протокола.

## 4. Настрой модели (опционально)

Узнай какие модели тебе доступны:

```bash
opencode models
```

Выбери схему:

| Агент | Рекомендуемая модель | Зачем |
|-------|---------------------|-------|
| **001** (Architect) | `opencode-go/kimi-k2.6` | Архитектурные решения |
| **007** (Scout) | `opencode-go/deepseek-v4-flash` | Разведка, контекст |
| **009** (Verifier) | `opencode-go/kimi-k2.6` | Верификация |
| **Operator** | `opencode-go/deepseek-v4-flash` | Исполнение TC |

Скопируй agent-файлы:

```bash
mkdir -p .opencode/agent
cp -r s-vibe-code/.opencode/agent/* .opencode/agent/
cp s-vibe-code/opencode.json .
```

Отредактируй `.opencode/agent/*.md` — замени `model:` на свои ID из `opencode models`.

**Подробнее:** `MODEL-ROUTING-GUIDE.md`

## 5. Настрой AGENTS.md под свой проект

Открой `AGENTS.md` и замени `[PROJECT]` на название своего проекта.

## 6. Напиши первую задачу

```markdown
# Задача: [что сделать]
```

Когда OpenCode запустится — 007 распознает задачу и начнёт цикл.

---

## Факультативно: контекст

Рекомендуемая структура:

```
your-project/
├── AGENTS.md
├── opencode.json
├── .opencode/agent/
│   ├── 001.md
│   ├── 007.md
│   ├── 009.md
│   └── operator.md
├── agents/
│   ├── charters/
│   │   ├── charter-007.md
│   │   ├── charter-009.md
│   │   └── charter-operator.md
│   └── packs/
│       └── 000-bootstrap.md    ← создай сам: краткий стартовый пакет
└── context/
    └── tasks/
```

## Перенастройка

Напиши `001/` — 007 запустит INIT protocol и проведёт тебя по шагам.

## Готово!

S-VC работает. Если что-то пошло не так — `Error = Stop`, агент доложит.

---

**Далее:** [Прочитай EXAMPLE.md](./EXAMPLE.md) — увидишь весь цикл в действии.
