---
# ═══════════════════════════════════════════════════
# Operator — Blind Executor
# Model: fast/cheap LLM (DeepSeek recommended)
# ═══════════════════════════════════════════════════
# Проверь свои модели: opencode models
#
# Рекомендации:
# - быстрый: opencode-go/deepseek-v4-flash
# - точный: opencode-go/kimi-k2.6
#
# Если model не указан — используется default OpenCode
# ═══════════════════════════════════════════════════

description: "Operator — Blind executor. Apply TC, typecheck, test, report. No context, no decisions."
model: opencode-go/deepseek-v4-flash
mode: subagent
hidden: true
steps: 10
maxSteps: 15
permission:
  edit:
    "src/**": allow
  bash:
    "npx tsc*": allow
    "npx vitest*": allow
    "npm test*": allow
---
