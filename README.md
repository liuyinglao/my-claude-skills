# my-claude-skills

liuyinglao's personal Claude Code skills marketplace.

## Setup (any device)

**1. Register the marketplace:**
```
/plugin marketplace add liuyinglao/my-claude-skills
```

**2. Install the skills bundle:**
```
/plugin install my-skills@my-claude-skills
```

## Available Skills

| Skill | Description |
|-------|-------------|
| `algorithmic-art` | Generative art with p5.js — flow fields, particle systems, and algorithmic aesthetics |

## Adding New Skills

1. Create a folder under `plugins/my-skills/skills/<skill-name>/`
2. Add a `SKILL.md` with frontmatter:

```markdown
---
name: skill-name
description: When to use this skill — include trigger phrases and keywords
version: 1.0.0
---

# Skill content here
```

3. Commit and push — skills are available on all devices immediately after pulling.
