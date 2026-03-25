---
name: skill-extractor
description: This skill should be used when the user mentions "extract skills", "consolidate skills", "acquire skills", or "add skills". It documents reusable patterns discovered during Claude sessions — including curl commands, local commands, and provided URLs or documents — and persists them to the liuyinglao/my-claude-skills GitHub repo.
version: 1.0.0
---

# Skill Extractor

Captures reusable knowledge from the current session and persists it to the `liuyinglao/my-claude-skills` repo so it's available on all devices.

## What to Extract

### 1. curl / HTTP fetches
When `curl` or any HTTP fetch was used to retrieve an external resource:
- Record the URL, purpose, and any flags/headers used
- Save as a reference entry under `references/` in the relevant skill folder

### 2. Local commands
When a shell command was run to complete a task:
- Document the command pattern, flags, and what it does
- Note any gotchas or preconditions (e.g. required tools, working directory)
- Save as a reusable recipe in the relevant skill

### 3. URLs and documents provided by the user
When the user provides a URL or attaches a document:
- Fetch and summarize the content if it contains reusable knowledge
- Copy the key content or a structured summary as a reference file

## Extraction Process

### Step 1 — Identify extractable patterns
Review the conversation for:
- Commands run with Bash
- URLs fetched or provided
- Techniques, workflows, or configurations that solved a problem
- Any knowledge the user explicitly shared

### Step 2 — Match against existing skills
Read the repo at `/Users/yinglaoliu/Downloads/liuyinglao/my-claude-skills/plugins/my-skills/skills/` and check each existing skill's `SKILL.md`.

- **If a match exists**: extend the existing `SKILL.md` with the new knowledge. Add reference files to that skill's folder if needed.
- **If no match**: create a new skill folder with a `SKILL.md` following the format below.

### Step 3 — Write the skill content

**New skill format:**
```markdown
---
name: skill-name
description: When to use this skill — include trigger phrases and keywords
version: 1.0.0
---

# Skill Name

## What This Skill Does
[One paragraph summary]

## Techniques / Commands
[Document commands, patterns, recipes]

## References
[Links or summaries of external resources]
```

**Reference file format** (for URLs/docs, save as `references/<topic>.md`):
```markdown
# <Topic>

Source: <URL or "user-provided">
Retrieved: <date>

## Summary
[Key information extracted]

## Raw / Key Excerpts
[Relevant content]
```

### Step 4 — Commit and push
After writing all files:
```bash
cd /Users/yinglaoliu/Downloads/liuyinglao/my-claude-skills
git add .
git commit -m "skill: <summary of what was extracted>"
git push
```

Confirm to the user which skills were updated or created, and the commit URL.

## Guidelines

- Keep skills focused — one domain per skill folder
- Prefer extending existing skills over creating new ones
- Reference files live inside the skill folder they support
- Never commit secrets, tokens, or personal data
- The description field is critical — write it so Claude knows exactly when to activate the skill
