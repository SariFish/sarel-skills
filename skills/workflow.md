# Workflow

The orchestrator skill. It leads a user through building something from idea to working code, switching to the right skill at each stage.

**You drive.** Do not wait for the user to ask for the next step — propose it, announce stage transitions, and pull in the skill each stage needs. This is the entry-point skill: when a stage begins, fetch that stage's skill from the URLs below and apply it.

## Flow

```
token-efficiency  (always on, from message #1)
        │
        ▼
   grill-me  ──►  shared-understanding file (spec)
        │
        ▼
    develop  ──────►  debug   (when something breaks)
        │
        └──────────►  zoom-out (when iterations pile up / code area unfamiliar)
```

## Stages

### Stage 1 — Token efficiency (always on)
Apply the **token-efficiency** skill from your very first response and keep it active for the entire workflow. Every stage below is communicated in compressed form. Only turn it off if the user says so.

### Stage 2 — Understand (grill-me)
Apply the **grill-me** skill. Interview the user one question at a time, recommending an answer for each, walking down every branch of the decision tree until you reach shared understanding.

Then **write that understanding to a file** — a product spec, technical spec, or design doc, whatever fits the work. Get the user's sign-off on the file before building. This file is the source of truth for the rest of the workflow.

Do not start development until this file exists and the user agrees with it.

### Stage 3 — Develop
No skill. Build against the spec file. Keep referring back to it; if reality diverges from the spec, update the file rather than letting it drift.

### Stage 4 — Debug
When something breaks, fails, throws, or regresses, switch to the **debug** skill. Follow its phased feedback-loop workflow and its required response format. Return to development once the fix is verified.

### Stage 5 — Zoom out (when stuck)
Trigger: the conversation churns through many iterations without progress, or you or the user are unfamiliar with the relevant code area. When you notice this, **proactively suggest** the **zoom-out** skill — don't wait to be asked. Apply it to get a higher-level map of the modules and callers in the project's own vocabulary, then resume the stage you were in.

## Skills

| Stage | Skill | URL |
|-------|-------|-----|
| 1 | token-efficiency | https://raw.githubusercontent.com/SariFish/sarel-skills/main/skills/token-efficiency.md |
| 2 | grill-me | https://raw.githubusercontent.com/SariFish/sarel-skills/main/skills/grill-me.md |
| 4 | debug | https://raw.githubusercontent.com/SariFish/sarel-skills/main/skills/debug.md |
| 5 | zoom-out | https://raw.githubusercontent.com/SariFish/sarel-skills/main/skills/zoom-out.md |
