# 🎬 EXAMPLE: Полный цикл A→B→C→D→E

Реальный пример работы S-VC на абстрактном проекте.

---

## Контекст

**Проект:** React-приложение для заметок (NoteApp)
**Технологии:** React 19, TypeScript, Vite
**Задача:** Добавить поддержку Markdown в редактор заметок
**Домен:** Editor

---

## PHASE 1 — RECON

### Шаг 1: PM → задача

```
Никита: Добавить поддержку Markdown в редактор заметок.
Поле ввода — textarea, нужно preview в реальном времени.
```

### Шаг 2: 007 → MACRO-PACK

После разведки 007 собирает пакет:

```markdown
# 📦 MACRO-PACK: 042-markdown-editor

## 🎯 ЗАДАЧА
Добавить поддержку Markdown в редактор заметок.
textarea → preview в реальном времени.

## 🏗️ АРХИТЕКТУРНЫЙ САММАРИ
- Editor: `src/editor/NoteEditor.tsx` (342 строки)
- Рендер: `src/editor/NoteRenderer.tsx` (156 строк)
- Хранилище: `src/stores/note.store.ts` (89 строк)
- Текущий формат: plain text

## ❄️ FROZEN GUARD
✅ OK — frozen зоны не затронуты

## 🔬 АЛЬТЕРНАТИВЫ
| Паттерн | Сейчас | Вариант | Почему лучше | Риск |
|---------|--------|---------|-------------|------|
| P-01 | plain text → regex | marked (NPM) | Стандарт, security | 🟢 Low |
| P-02 | синхронный рендер | lazy + debounce | Perf при длинном тексте | 🟢 Low |

## 📄 СЖАТЫЙ КОД
// src/editor/NoteEditor.tsx — текстовое поле без Markdown
function NoteEditor() {
  const [text, setText] = useState('');
  return <textarea value={text} onChange={...} />
}

// src/editor/NoteRenderer.tsx — рендер plain text
function NoteRenderer({ text }: { text: string }) {
  return <div>{text}</div>
}

## 🔗 ГРАФ ЗАВИСИМОСТЕЙ
note.store.ts ← NoteEditor.tsx ← App.tsx
                             ← NoteRenderer.tsx
                             ← NoteSidebar.tsx
```

### Шаг 3: Architect → решение

```
Architect: TC-042-01: Установить marked + DOMPurify (security).
TC-042-02: NoteRenderer — marked(text) с debounce 300ms.
TC-042-03: NoteRenderer — sanitize через DOMPurify.

Frozen guard: OK. Никаких frozen зон.
No-go: Не менять storage, не добавлять плагины marked.
```

### Шаг 4: 007 → MICRO-PACK

```markdown
📦 MICRO-PACK: TC-042-02
ЦЕЛЬ: NoteRenderer — marked(text) с debounce 300ms
ФАЙЛ: src/editor/NoteRenderer.tsx
ЗОНА: строки 10-25
ДЕЙСТВИЕ: Добавить marked() + debounce
КОНТЕКСТ:
  // ---- до ----
  function NoteRenderer({ text }: { text: string }) {
    return <div>{text}</div>
  }
  // ---- после ----
  import { marked } from 'marked';
  import { useMemo } from 'react';
  
  function NoteRenderer({ text }: { text: string }) {
    const html = useMemo(() => marked(text), [text]);
    return <div dangerouslySetInnerHTML={{ __html: html }} />
  }
ЗАПРЕЩЕНО: Не менять импорты кроме добавленных, не менять App.tsx
ТЕСТ: tsc --noEmit && vitest run --related src/editor
```

---

## PHASE 2 — EXECUTION

### Шаг 5: Operator → apply

Operator применяет TC строго по инструкции:

```
❄️ ОПЕРАТОР: TC-042-02

СТАТУС: ✅ ВЫПОЛНЕНО

ХОЛОДНАЯ ВЕРИФИКАЦИЯ:
- grep anchor at line 10: ⬛️ function NoteRenderer ... ✅
- anchors до/после: ✅ совпадают
- запрещённая зона: ✅ не затронута

ЧТО СДЕЛАНО:
- src/editor/NoteRenderer.tsx: строки 10-25 — добавлен marked()

ПРОВЕРКИ:
- tsc --noEmit: ✅ 0 errors
- vitest run --related: ✅ passed
```

### Шаг 6: 007 → верификация

```
✅ git diff — только target changes
✅ tsc --noEmit — 0 errors
✅ vitest — passed
✅ frozen zones — не тронуты
```

---

## PHASE 3 — VERIFICATION

### Шаг 7: 007 → DOC-CHECK first pass

```
DOC-CHECK: TC-042-02

Затронутые домены: Editor
Документация: docs/editor-system.md существует

Результат: DOC_UPDATE_REQUIRED
  - docs/editor-system.md не описывает Markdown поддержку
  - Нужно добавить раздел про формат ввода

DOC-TC: DOC-TC-042 — "Add Markdown section to editor-system.md"
Registry: docs/editor-system.md → needs_update
```

### Шаг 8: 009 → VERDICT

```
VERDICT: ✅ PASS

RUNTIME:    ✅ OK — изменения работают, preview отображается
DOC-CHECK:  ✅ OK — DOC-TC-042 создан, registry обновлён
REGISTRY:   ✅ OK — статусы корректны
FULL-BASE:  ✅ OK — архитектурная модель не изменилась

NOTES:
- DOC-TC-042 ожидает реализации (editor-system.md)
- В остальном — PASS
```

### Шаг 9: ✅ COMPLETE

Задача завершена.

---

## Итог

```
TC-042-01 ✅ Set up marked + DOMPurify
TC-042-02 ✅ NoteRenderer: preview in real-time
TC-042-03 ✅ XSS protection via DOMPurify
DOC-TC-042 ⏳ Update editor-system.md

Время: ~25 минут
Изменено: 2 файла, +15 строк, -3 строки
Проверено: 007 + 009
Статус: ✅ COMPLETE (кроме DOC-TC — фоновый)
```

---

## Что дал S-VC в этом примере

| Без S-VC | С S-VC |
|----------|--------|
| Агент мог сам выбрать marked/showdown/remark | Architect выбрал marked на основе альтернатив |
| XSS — забыли бы DOMPurify | 📍 Context7 показал marked → DOMPurify |
| Документация устарела бы | DOC-CHECK выявил drift → DOC-TC создан |
| Confirmation bias — никто не проверил | 009 дал независимый PASS |
| Хаос в "кто что менял" | MICRO-PACK — одна инструкция на одно изменение |
| "Работает — и ладно" | VERDICT — формальное подтверждение |

---

*S-VC Core 1.2 — June 2026*
