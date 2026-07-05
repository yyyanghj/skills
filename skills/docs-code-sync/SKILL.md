---
name: docs-code-sync
description: Use when updating AGENTS.md, README files, handoff docs, task plans, or repository guidance to match current code, architecture, validation commands, or user-corrected rules.
---

# Docs Code Sync

## Overview

Keep repository docs grounded in the current code. Use this for documentation changes that must reflect real implementation details, project rules, or a user's corrected wording.

## Workflow

1. Confirm the doc target and scope. If the user names one file, edit only that file.
2. Read the existing doc before writing. Then inspect the relevant code/config paths that prove the statements you plan to add.
3. Preserve exact contract strings, command names, field names, and user-provided wording when they matter.
4. Update the doc directly and keep the text durable: owner, boundary, command, and validation rule over narrative.
5. Grep for stale terms when replacing or consolidating docs, especially removed API names, old protocol names, and deleted plan files.
6. Verify the doc-only change with focused checks such as `rg` for required/stale terms, `git diff --check`, and `git diff -- <doc>`.

## Defaults

- Treat docs updates as code-backed work, not memory-backed summaries.
- Keep documentation-only requests documentation-only. Do not opportunistically edit code.
- Prefer one authoritative doc over multiple stale handoff/task docs when the user asks to consolidate.
- For `AGENTS.md`, write durable repository rules, not one-off task notes.
- Do not invent future work, unsupported non-goals, or compatibility paths the user explicitly ruled out.

## Useful Checks

- `rg -n "<old term>|<new term>" <doc-or-scope>`
- `rg --files <docs-dir>`
- `git diff --check -- <doc>`
- `git status --short -- <doc-or-docs-dir>`
