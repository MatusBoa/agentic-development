---
name: create-skill
description: Create a new Agent Skill following the agentskills.io specification and this project's conventions. Use when adding a skill to .agents/skills/, scaffolding SKILL.md frontmatter, structuring skill directories, or writing skill instructions.
compatibility: Any project using the Agent Skills specification (agentskills.io). Skills are stored under .agents/skills/.
---

# Create skill

Use this skill when creating a new agent skill from scratch.

## Directory structure

A skill is a directory containing at minimum a `SKILL.md` file:

```
.agents/skills/{skill-name}/
├── SKILL.md          # Required: YAML frontmatter + Markdown instructions
├── scripts/          # Optional: executable code the agent can run
├── references/       # Optional: detailed documentation loaded on demand
└── assets/           # Optional: templates, schemas, static resources
```

The directory name must exactly match the `name` field in the frontmatter.

## SKILL.md format

The file must contain YAML frontmatter delimited by `---`, followed by Markdown body content.

### Frontmatter fields

| Field | Required | Constraints |
|---|---|---|
| `name` | Yes | 1-64 chars. Lowercase `a-z`, digits, hyphens only. No leading/trailing/consecutive hyphens. Must match directory name. |
| `description` | Yes | 1-1024 chars. Describes what the skill does AND when to use it. Include keywords that help agents match tasks to this skill. |
| `compatibility` | No | 1-500 chars. Environment requirements: target product, system packages, network access, language version. |
| `license` | No | License name or reference to a bundled license file. |
| `metadata` | No | Arbitrary string-to-string key-value map for additional properties. |
| `allowed-tools` | No | Space-separated string of pre-approved tools. Experimental; support varies. |

### Name validation rules

- Lowercase alphanumeric and hyphens only: `^[a-z0-9]+(-[a-z0-9]+)*$`
- No uppercase, no underscores, no dots, no spaces
- No leading hyphen (`-foo`), no trailing hyphen (`foo-`), no consecutive hyphens (`foo--bar`)

Valid: `code-review`, `pdf-processing`, `data-analysis`
Invalid: `Code-Review`, `-pdf`, `pdf--processing`, `my_skill`

### Description quality

The description is the primary signal agents use to decide when to load a skill. It must answer two questions:

1. **What does the skill do?** (capabilities)
2. **When should it be loaded?** (trigger conditions)

Good:
```yaml
description: Apply this project's database migration conventions. Use when creating migrations, altering tables, adding indexes, or working with database schema files.
```

Bad:
```yaml
description: Helps with databases.
```

### Frontmatter examples

Minimal:

```yaml
---
name: my-skill
description: What this skill does. Use when [trigger conditions].
---
```

Full:

```yaml
---
name: my-skill
description: What this skill does. Use when [trigger conditions].
compatibility: Requires Node.js 20+ and Docker.
license: MIT
metadata:
  author: team-name
  version: "1.0"
---
```

## Body content

The Markdown body after frontmatter contains the skill instructions. There are no format restrictions. Write whatever helps agents perform the task correctly.

### This project's body style

Follow the pattern established by existing skills in `.agents/skills/`:

1. **Title**: `# Skill name` (sentence case, matches the skill's purpose)
2. **Opening line**: One sentence stating when to use the skill
3. **Sections**: Organized by topic with `##` headings
4. **Code examples**: Concrete, copy-pasteable snippets with correct syntax
5. **Rules/conventions**: Bulleted lists with clear, imperative statements
6. **Anti-patterns section**: End with a `## Do not` section listing common mistakes

Example body structure:

```markdown
# Code review

Use this skill when reviewing code changes for quality and correctness.

## Review checklist

- Check type safety: no `any` casts, no suppressed errors
- Verify error handling: no empty catch blocks
- ...

## Output format

Provide findings as a numbered list with file path, line, and description.

## Do not

- Do not approve changes that suppress type errors.
- Do not suggest stylistic changes unrelated to correctness.
```

### Progressive disclosure

Skills load progressively:

1. **Metadata** (~100 tokens): `name` and `description` are loaded at startup for all skills
2. **Instructions** (< 5000 tokens recommended): Full `SKILL.md` body loads when the skill activates
3. **Resources** (on demand): Files in `scripts/`, `references/`, `assets/` load only when needed

Keep `SKILL.md` under 500 lines. Move detailed reference material to separate files under `references/`.

### File references

Reference other files in the skill using relative paths from the skill root:

```markdown
See [the reference guide](references/REFERENCE.md) for the complete API.

Run the setup script:
scripts/setup.sh
```

Keep references one level deep from `SKILL.md`. Avoid deeply nested reference chains.

## Workflow

When creating a new skill:

1. **Pick a name**: lowercase, hyphenated, descriptive. Verify it matches `^[a-z0-9]+(-[a-z0-9]+)*$`.
2. **Create directory**: `.agents/skills/{skill-name}/`
3. **Write SKILL.md**: Start with frontmatter (`name`, `description`, optionally `compatibility`), then body.
4. **Write the body**: Follow this project's body style. Focus on actionable instructions, not theory.
5. **Add references if needed**: If the body would exceed ~400 lines, split detailed content into `references/`.
6. **Verify**: Confirm `name` field matches directory name. Confirm description answers "what" and "when".

## Specification reference

This skill is based on the Agent Skills specification: <https://agentskills.io/specification.md>

Consult the specification when unsure about format details, validation rules, or optional features not covered here.

## Do not

- Do not use uppercase, underscores, or dots in the skill name.
- Do not write a description that only says what the skill does without saying when to use it.
- Do not put everything in `SKILL.md` if it exceeds 500 lines; split into `references/`.
- Do not reference files with deeply nested paths; keep references one level deep.
- Do not omit code examples when the skill involves commands or code patterns.
- Do not forget to match the directory name to the `name` frontmatter field exactly.
- Do not include fields in frontmatter that have no value; omit optional fields rather than leaving them empty.
