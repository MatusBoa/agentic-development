---
name: team-review
description: Run a scoped multi-agent code review through team_create. Use when the user asks to review a branch, diff, directory, file, or other explicit scope with code quality, performance, and security reviewers.
compatibility: Requires opencode team mode with team_create, team_send_message, team_task_create, team_status, shutdown, and delete tools enabled.
---

# Team review

Use this skill when a user asks for a scoped code review that should be performed by a team of specialized reviewers.

## Review scope

Start by identifying the review scope from the user's request.

Accept scopes such as:

- current branch or branch diff
- staged or unstaged changes
- a specific commit range
- a directory path
- a file path
- an explicitly listed set of paths

If the scope is missing or ambiguous, ask one precise question before creating the team. Do not assume the whole branch when the user may mean a single path.

## Create the review team

Create one ephemeral team with three category-backed reviewers. Use `team_create` with an inline spec like this, replacing `{scope}` with the exact scope text the user provided or confirmed:

```json
{
  "inline_spec": {
    "name": "scoped-review-team",
    "members": [
      {
        "name": "code-quality-reviewer",
        "kind": "category",
        "category": "unspecified-high",
        "prompt": "Review the requested scope for consistency with existing code, maintainability, correctness, type/interface safety where applicable, project conventions, unnecessary complexity, and regression risk. Report only actionable findings with file paths and line numbers when available. Scope: {scope}"
      },
      {
        "name": "performance-reviewer",
        "kind": "category",
        "category": "unspecified-high",
        "prompt": "Review the requested scope for performance risks: avoidable expensive operations, repeated data access, inefficient loops, unnecessary allocations, blocking I/O, over-fetching, cache misuse, missing indexes when data storage is in scope, and high-concurrency bottlenecks. Report only actionable findings with file paths and line numbers when available. Scope: {scope}"
      },
      {
        "name": "security-reviewer",
        "kind": "category",
        "category": "unspecified-high",
        "prompt": "Review the requested scope for security risks: missing authentication or authorization, access-control mistakes, data leaks, unsafe serialization or output encoding, unsafe object binding, injection, secret exposure, weak validation at trust boundaries, and privilege escalation. Report only actionable findings with file paths and line numbers when available. Scope: {scope}"
      }
    ]
  }
}
```

Use `unspecified-high` because review is analytical and cross-cutting. Do not use `quick` for these reviewers.

## Assign work

After `team_create` returns a `teamRunId`, create one task per reviewer. Each task must repeat the exact scope and expected output.

```json
{
  "teamRunId": "{teamRunId}",
  "subject": "Code quality review",
  "description": "Review {scope} for code quality, project conventions, maintainability, correctness, type/interface safety where applicable, and regression risk. Return findings ordered by severity with file path, line, impact, and suggested fix. If no findings, say so and list residual risks."
}
```

Create equivalent tasks for:

- `Performance review`
- `Security review`

Then use `team_send_message` to broadcast the scope and review rules:

```json
{
  "teamRunId": "{teamRunId}",
  "to": "*",
  "kind": "announcement",
  "summary": "Scoped review instructions",
  "body": "Review scope: {scope}\n\nReturn findings only. Format each finding as: severity, file:line, issue, impact, suggested fix. Do not modify files. If your area has no findings, explicitly say no findings and mention what you checked."
}
```

## Reviewer checklist

### Code quality reviewer

- Check consistency with nearby code and project skills.
- Check type/interface safety, contract accuracy, naming, imports or dependencies, and error handling.
- Flag speculative abstractions, duplicated domain logic, and incorrect platform or framework assumptions.
- Verify touched components, entry points, data models, services, and configuration follow existing project patterns.
- Check that public APIs, CLI commands, UI flows, background jobs, or integration points preserve documented behavior when they are in scope.

### Performance reviewer

- Check repeated data access, N+1-style behavior, and avoidable expensive operations.
- Check loops, batching, pagination, streaming, caching, and over-fetching.
- Check blocking I/O or expensive work on latency-sensitive or high-throughput paths.
- Check missing or inappropriate indexes, cache keys, resource limits, or concurrency controls when relevant to the scope.

### Security reviewer

- Check authentication and authorization gates before data access or mutation.
- Check role, permission, ownership, tenant, or policy checks using the project's existing access-control model.
- Check data exposure in APIs, UI output, errors, logs, serialized responses, artifacts, and exported files.
- Check validation and normalization at external boundaries and injection risks across queries, commands, templates, redirects, and generated content.
- Check secret handling, credential storage, dependency risk, and unsafe deserialization or object binding when they are in scope.

## Synthesize results

When reviewers respond, synthesize rather than concatenate.

Final review output must use this order:

1. Critical findings
2. High findings
3. Medium findings
4. Low findings
5. No-finding areas and residual risks

Each finding must include:

- severity
- file path and line when available
- issue
- impact
- suggested fix
- reviewer source

If reviewers disagree, resolve the conflict from the code and explain the decision briefly.

## Close the team

Teams are ephemeral. After all reviewer tasks are complete and no follow-up is needed:

1. Run `team_status`.
2. For each active member, call `team_shutdown_request` and then `team_approve_shutdown`.
3. Call `team_delete`.

Do this before giving the final answer unless the user explicitly asks to keep the team open.

## Do not

- Do not create the team before the review scope is known.
- Do not use `oracle` or other ineligible subagents as team members.
- Do not ask reviewers to edit files; this skill is for review only.
- Do not report vague findings without a concrete file, behavior, or risk.
- Do not treat style preferences as findings unless they conflict with project conventions.
- Do not leave the team running after the review is complete.
