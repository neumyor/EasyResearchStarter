# Codex Repository Instructions

Before working in this repository, read and follow `MANAGE_RULES.md`. For
research tasks, also follow `HOW_TO_DO_RESEARCH.md`.

## Research skills (canonical framework)

Research work is executed through the autoresearch skill system under
`.agents/skills/`. This is Codex's repository-level skill location; each
skill is a directory containing a `SKILL.md` with `name` and `description`
front matter. The matching `.claude/skills/` tree is maintained for Claude
Code. The two trees MUST contain the same five skills and equivalent content.
The skills' `SKILL.md` files are the source of truth for their procedures.

- `$autoresearch-orchestrator` — control plane: decides what happens next and
  coordinates the foundational skills. **Start here** for autonomous research.
- `$research-environment` — machine/resource discovery; maintains the
  Git-ignored `ENV.md` capability snapshot.
- `$research-git-ledger` — commit protocol: every agent-created commit maps to
  exactly one newest-first `docs/LOG.md` entry, joined by a shared
  `Research-Log-ID`; read the ledger progressively from its head.
- `$research-experiment` — experiment lifecycle: naming, light/heavy record
  layout, PRE-RUN/POST-RUN commits, CSV-based result persistence, long-run
  background execution/progress reporting, failure and closure handling.
- `$easyresearch-adopt-workspace` — explicitly and additively adopts this
  workflow in an existing workspace; plugin-only operation is the default.

Do not invent experiment/commit conventions that conflict with these skills, and
never bypass the git-ledger protocol.

## Visual output review

For any material chart, plot, figure, or image generated in this workspace,
including with Matplotlib, use a host-provided direct image viewer or preview
tool at least once when one is available before relying on or delivering the
output. Check readability, labels/legends/units, clipping, layout, and agreement
with the source data. If no direct viewer is available, state that limitation
and do not claim visual inspection occurred.

## Reusable distribution

This repository is also a plugin marketplace. For safe reuse in another
workspace, install `easyresearch-workspace` from this repository's marketplace
instead of copying these directories over the target's existing skills. Its
namespaced `easyresearch-*` skills are non-invasive by default. Invoke
`$easyresearch-adopt-workspace` only when the user explicitly wants persistent
workspace-local adoption.

## GitHub pull/push network route

Do not assume a fixed proxy port. Before testing GitHub connectivity or
fetching/pulling/pushing, invoke `research-environment` to scan the local
listening ports and proxy-related environment/Git configuration, then test
candidate SOCKS5 ports against `ssh.github.com:443`. Record only the selected
port and the test outcome in Git-ignored `ENV.md`; never record credentials.

When a candidate works, route SSH through it and retain the existing remote
URL. Use GitHub's dedicated SSH-over-443 endpoint (`ssh.github.com:443`), not
`github.com:443`, because the latter can hang during the SSH banner exchange.
For example, substituting the discovered `<proxy-port>`:

```bash
git -c core.sshCommand='ssh -o HostName=ssh.github.com -o ConnectTimeout=15 -o ProxyCommand="nc -x 127.0.0.1:<proxy-port> -X 5 ssh.github.com 443" -p 443' ls-remote origin HEAD
```

Use the same temporary SSH command for `fetch`, `pull --ff-only`, or `push`
only after the read-only `ls-remote` check succeeds. If local and remote
branches have diverged, omit `--ff-only` only when preserving local commits
and merging is intended.
