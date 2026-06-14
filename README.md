# sarel-skills

A collection of AI behavior rules. Model-agnostic — works with Claude, GPT, Gemini, Cursor, or any tool that accepts a system prompt.

## Available Skills

| Skill | Description |
|-------|-------------|
| [workflow](skills/workflow.md) | Orchestrator — leads the user from idea to working code, switching skills per stage |
| [debug](skills/debug.md) | Hypothesis-driven debugging — build a feedback loop, find root cause, verify the fix |
| [zoom-out](skills/zoom-out.md) | Go up a layer of abstraction — map relevant modules and callers when you're lost in code |
| [grill-me](skills/grill-me.md) | Stress-test a plan or spec — relentless one-at-a-time interview with recommended answers |
| [token-efficiency](skills/token-efficiency.md) | Ultra-compressed responses — cut ~75% of tokens, keep full technical accuracy |

## How to Use

### Paste directly
Copy the contents of any skill file and paste it into your AI's system prompt or conversation.

### Fetch by URL
Every skill is available at a stable raw URL:
```
https://raw.githubusercontent.com/SariFish/sarel-skills/main/skills/debug.md
```

### Claude Code (CLAUDE.md)
Add to your project's `CLAUDE.md`:
```
Follow the rules at https://raw.githubusercontent.com/SariFish/sarel-skills/main/skills/debug.md
```

### Cursor (.cursorrules)
```
https://raw.githubusercontent.com/SariFish/sarel-skills/main/skills/debug.md
```

### Any API (Python)
```python
from urllib.request import urlopen
skill = urlopen("https://raw.githubusercontent.com/SariFish/sarel-skills/main/skills/debug.md").read().decode()
```

### Any API (Node)
```js
const skill = await fetch("https://raw.githubusercontent.com/SariFish/sarel-skills/main/skills/debug.md").then(r => r.text())
```

### Discover all skills
```
https://raw.githubusercontent.com/SariFish/sarel-skills/main/index.json
```

## Adding a Skill

1. Create `skills/<name>.md`
2. Add an entry to `index.json`

## Credits

Several skills are adapted from [mattpocock/skills](https://github.com/mattpocock/skills):
- `debug` incorporates the **diagnose** skill
- `zoom-out`, `grill-me`, and `token-efficiency` (from **caveman**) are adapted directly

Used under the MIT License:

```
MIT License

Copyright (c) 2026 Matt Pocock

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
