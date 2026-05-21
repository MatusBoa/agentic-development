---
name: beads-issues
description: Manage Beads (bd) issues. Use when creating, updating, closing, or listing issues; working the ready queue; managing dependencies between issues; building epic hierarchies; or doing any programmatic issue tracking with the bd CLI.
compatibility: Requires beads (bd) CLI installed. Works with any project using Beads issue tracking.
---

# Beads issue management

Use the `bd` CLI for all issue operations. Always prefer `--json` for programmatic output. Never guess at IDs; resolve them from `bd list` or `bd ready` output first.

## Issue IDs

IDs are hash-based, not sequential. Format: `bd-a1b2`.

- Partial matching works: `bd show a1b2`
- Full IDs in list output: `bd list --full-ids`
- IDs are collision-resistant across branches and agents
- Configure prefix: `bd config set id.prefix myproject`
- Configure hash length: `bd config set id.hash_length 6`

## Creating issues

Always include `--description` and `--json` when creating issues programmatically.

```bash
bd create "Title" -t {type} -p {priority} --description="Detailed description" --json
```

**Types:** `bug`, `feature`, `task`, `epic`, `chore`

**Priorities:**
- `0` critical
- `1` high
- `2` medium
- `3` low
- `4` backlog

**Labels:**
```bash
bd create "Title" -t task -p 2 --description="..." -l "label1,label2" --json
```

**Link to a discovered-from parent at creation time:**
```bash
bd create "Child issue" -t bug -p 1 --description="..." --deps discovered-from:{id} --json
```

## Showing and listing

```bash
# Show a single issue
bd show {id} --json

# List with filters
bd list --status open --json
bd list --status in_progress --json
bd list --status closed --json
bd list --priority 1 --json
bd list --type bug --json
bd list --label-any "label1,label2" --json
bd list --label-all "label1,label2" --json
bd list --full-ids --json
```

Valid statuses: `open`, `in_progress`, `closed`

## Updating issues

```bash
# Claim an issue (mark as in_progress and assign to self)
bd update {id} --claim

# Change priority
bd update {id} --priority 2

# Add a label
bd update {id} --add-label "needs-review"
```

## Closing issues

```bash
bd close {id} --reason "Implemented in commit abc123" --json
```

Always provide a meaningful `--reason` describing what was done or why the issue is closed.

## Dependencies

There are four dependency types:

| Type | Purpose |
|---|---|
| `blocks` | Affects the ready queue; dependent cannot start until dependency is done |
| `parent-child` | Hierarchical nesting (see epics section) |
| `discovered-from` | Links a new issue to the issue where it was found |
| `related` | Loose association, no queue effect |

```bash
# Add a blocking dependency: {dependent} cannot start until {dependency} is done
bd dep add {dependent} {dependency}

# View the full dependency tree for an issue
bd dep tree {id}

# See all issues currently blocked
bd blocked

# Detect dependency cycles
bd dep cycles

# Add a related (non-blocking) link
bd relate {id1} {id2}
```

## Ready queue

`bd ready` is not the same as `bd list --status open`. The ready queue computes the dependency graph and returns only unblocked work.

```bash
# Get unblocked issues
bd ready --json

# Explain why issues are blocked or ready
bd ready --explain

# Filter ready issues by priority
bd ready --priority 1 --json
```

Use `bd ready` to decide what to work on next. Use `bd list` when you need to see all issues regardless of blocking state.

## Working the queue

Follow this pattern when picking up and completing work:

1. Find unblocked work: `bd ready --json`
2. Claim the issue: `bd update {id} --claim`
3. Do the work
4. Close with reason: `bd close {id} --reason "Done: ..." --json`
5. Check what's unblocked next: `bd ready --json`

## Epics and hierarchical issues

Epics group related work. Children auto-number as `bd-xyz.1`, `bd-xyz.2`, up to 3 levels deep.

```bash
# Create an epic
bd create "Epic title" -t epic --description="..." --json

# Add a child to an epic
bd create "Child task" -t task -p 2 --description="..." --parent {epic-id} --json

# View the full hierarchy
bd dep tree {epic-id}
```

## Do not

- Do not omit `--json` when reading issue data programmatically; human-readable output is not stable.
- Do not omit `--description` when creating issues; a title alone is not enough context.
- Do not use `bd list --status open` as a substitute for `bd ready`; blocked issues will appear in the list but should not be started.
- Do not hardcode or guess issue IDs; always resolve them from CLI output.
- Do not add blocking dependencies in the wrong direction; `bd dep add A B` means A is blocked by B, not the other way around.
- Do not close an issue without a `--reason`; it loses traceability.
- Do not create more than 3 levels of nesting in epic hierarchies.
- Do not run `bd dep cycles` only after problems appear; check it proactively when adding many dependencies.
