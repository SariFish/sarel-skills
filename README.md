# sarel-skills

A collection of AI behavior rules. Model-agnostic — works with Claude, GPT, Gemini, Cursor, or any tool that accepts a system prompt.

## Available Skills

| Skill | Description |
|-------|-------------|
| [debug](skills/debug.md) | Hypothesis-driven debugging — no guessing, root cause focused |

## How to Use

### Paste directly
Copy the contents of any skill file and paste it into your AI's system prompt or conversation.

### Fetch by URL
Every skill is available at a stable raw URL:
```
https://raw.githubusercontent.com/sarel-fish/sarel-skills/main/skills/debug.md
```

### Claude Code (CLAUDE.md)
Add to your project's `CLAUDE.md`:
```
Follow the rules at https://raw.githubusercontent.com/sarel-fish/sarel-skills/main/skills/debug.md
```

### Cursor (.cursorrules)
```
https://raw.githubusercontent.com/sarel-fish/sarel-skills/main/skills/debug.md
```

### Any API (Python)
```python
from urllib.request import urlopen
skill = urlopen("https://raw.githubusercontent.com/sarel-fish/sarel-skills/main/skills/debug.md").read().decode()
```

### Any API (Node)
```js
const skill = await fetch("https://raw.githubusercontent.com/sarel-fish/sarel-skills/main/skills/debug.md").then(r => r.text())
```

### Discover all skills
```
https://raw.githubusercontent.com/sarel-fish/sarel-skills/main/index.json
```

## Adding a Skill

1. Create `skills/<name>.md`
2. Add an entry to `index.json`
