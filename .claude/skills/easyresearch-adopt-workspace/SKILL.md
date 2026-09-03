---
name: easyresearch-adopt-workspace
description: Add the EasyResearch prompt-based research workflow to an existing workspace without overwriting its code, instructions, or skills. Use only when the user explicitly asks to adopt, attach, or initialize EasyResearch in that workspace.
---

# Adopt EasyResearch Workspace

## Purpose

Attach the EasyResearch research workflow to the current workspace safely and
incrementally. This skill is intentionally explicit-only in practice: do not
modify a workspace merely because the EasyResearch plugin or repository is
available.

## Safety boundary

Before writing, inspect the target workspace, its Git status, and its existing
agent-instruction and skill directories. Explain the exact additive files or
instruction changes proposed. Do not overwrite, delete, rename, relocate, or
silently merge existing code, `AGENTS.md`, `CLAUDE.md`, `.agents/skills/`, or
`.claude/skills/`.

Acquire the single-writer workspace lock before any write. If another agent
holds it, stop and wait for that owner rather than editing concurrently.

## Adoption modes

Choose the least invasive mode that satisfies the user's request:

1. **Plugin-only (default):** make no workspace changes. Use the installed
   namespaced EasyResearch skills for work in the current workspace.
2. **Additive workspace adoption:** only when the user explicitly requests
   persistent repository-local behavior, add a new, namespaced EasyResearch
   skill directory and an additive instruction block. Preserve all existing
   text and skills exactly; if a name collision exists, stop and ask for a
   different namespace rather than overwriting it.
3. **Template migration:** only when the user explicitly asks to restructure
   the workspace. First inventory conflicts and obtain a concrete migration
   decision; never infer permission to replace existing conventions.

## Required adoption record

For additive workspace adoption, create a small tracked document at
`docs/easyresearch/ADOPTION.md` when `docs/` is already the repository's
documentation root; otherwise ask the user where project documentation belongs.
Record the selected mode, installed EasyResearch version/source, date, and any
intentional integration decisions. Do not record credentials, private paths, or
tokens.

## Completion

Report exactly what was added and what was left untouched. Release the
single-writer lock after all writes are complete.
