# Codex Repository Instructions

Before working in this repository, read and follow `MANAGE_RULES.md`. For
research tasks, also follow `HOW_TO_DO_RESEARCH.md`.

## Research skills (canonical framework)

Research work is executed through the autoresearch skill system under
`.agents/skills/`. The skills' `SKILL.md` files are the source of truth for
their procedures.

- `$autoresearch-orchestrator` — control plane: decides what happens next and
  coordinates the foundational skills. **Start here** for autonomous research.
- `$research-environment` — machine/resource discovery; maintains the
  Git-ignored `ENV.md` capability snapshot.
- `$research-git-ledger` — commit protocol: every agent-created commit maps to
  exactly one `docs/LOG.md` entry, joined by a shared `Research-Log-ID`.
- `$research-experiment` — experiment lifecycle: naming, light/heavy record
  layout, PRE-RUN/POST-RUN commits, CSV-based result persistence, failure and
  closure handling.

Do not invent experiment/commit conventions that conflict with these skills, and
never bypass the git-ledger protocol.

## GitHub pull/push network route

Use GitHub's dedicated SSH-over-443 endpoint through the local SOCKS5 proxy
on port 7897. The proxy target must be `ssh.github.com:443` (using
`github.com:443` can hang during the SSH banner exchange).

For pull/fetch:

```bash
git -c core.sshCommand='ssh -o HostName=ssh.github.com -o ConnectTimeout=15 -o ProxyCommand="nc -x 127.0.0.1:7897 -X 5 ssh.github.com 443" -p 443' pull
```

For push, use the same temporary SSH command:

```bash
git -c core.sshCommand='ssh -o HostName=ssh.github.com -o ConnectTimeout=15 -o ProxyCommand="nc -x 127.0.0.1:7897 -X 5 ssh.github.com 443" -p 443' push
```

This leaves the repository's remote URL unchanged. If local and remote
branches have diverged, omit `--ff-only` only when preserving local commits
and merging is intended.
