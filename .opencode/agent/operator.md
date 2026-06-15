---
description: "Operator — Blind executor. Apply TC, typecheck, test, report. No context, no decisions."
mode: subagent
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
