---
name: research-environment
description: Discovers and records machine-specific research capabilities, including uv, Conda, compute resources, Git, and GitHub communication capabilities, in a Git-ignored ENV.md file.
---

# Research Environment

## Purpose

Maintain a machine-specific capability snapshot in repository-root `ENV.md`.

`ENV.md` is local to the machine and MUST NOT be committed.

If `ENV.md` is absent on a machine, create it by performing resource discovery in the required order.

## Invariants

MUST:

- Check `.gitignore` and ensure `/ENV.md` is ignored.
- Never write secrets, tokens, passwords, private keys, credential contents, or sensitive authentication material into `ENV.md`.
- Prefer inspection over mutation during discovery.
- Record timestamps in Beijing time (`UTC+08:00`) with seconds.
- Treat `ENV.md` as a capability snapshot, not as project documentation.

## Startup logic

If `ENV.md` exists:

1. read it;
2. verify that major claims are still plausible;
3. minimally re-check resources required for the current task;
4. update it only when meaningful machine capabilities have changed.

If `ENV.md` does not exist:

perform the following discovery sequence exactly:

1. uv;
2. Conda;
3. compute resources;
4. Git and GitHub capabilities.

## 1. uv discovery

Inspect:

- whether `uv` is installed;
- uv version;
- repository `pyproject.toml`;
- repository `uv.lock`;
- current Python interpreter;
- active virtual environment, if any;
- project Python requirements where visible.

Useful command patterns:

```bash
command -v uv
uv --version
python --version
which python
```

Inspect `pyproject.toml` and `uv.lock` when present.

Do not automatically install, upgrade, sync, or mutate the environment merely to complete discovery.

## 2. Conda discovery

Inspect whether Conda is available and list available environments.

Typical patterns:

```bash
command -v conda
conda env list
```

Record, when safely available:

- Conda implementation/version;
- environment names;
- environment paths;
- relevant Python versions.

Do not arbitrarily activate or modify Conda environments during discovery.

## 3. Compute discovery

Inspect machine resources relevant to research execution.

Record as available:

- operating system;
- architecture;
- CPU model/count;
- RAM;
- available disk space;
- accelerator type;
- GPU count;
- GPU model;
- GPU memory;
- driver/runtime information;
- CUDA, ROCm, MPS, or equivalent capability;
- current accelerator utilization where useful.

Typical patterns may include:

```bash
uname -a
df -h
nvidia-smi
```

Use platform-appropriate alternatives when necessary.

Do not assume NVIDIA hardware.

## 4. Git and GitHub discovery

Inspect:

- Git version;
- repository root;
- current branch;
- HEAD;
- configured remotes;
- configured commit identity;
- remote transport type;
- whether push appears possible;
- GitHub CLI availability;
- GitHub CLI authentication status if available;
- SSH vs HTTPS communication path;
- other available GitHub communication mechanisms relevant to the current repository.
- locally available proxy listeners and any proxy values already configured in
  environment variables or Git configuration;
- a read-only GitHub connectivity test through each plausible local SOCKS5
  listener, selecting a working port when one is available.

Typical patterns:

```bash
git --version
git rev-parse --show-toplevel
git branch --show-current
git rev-parse HEAD
git remote -v
git config user.name
git config user.email
command -v gh
gh auth status
```

### Proxy-aware GitHub connectivity test

Do not assume a previously documented proxy port is still available. Before any
networked Git operation, scan the current machine for candidate proxy ports:

1. inspect proxy-related environment variables and Git `http.proxy` / `https.proxy`
   values without printing credentials;
2. inspect local TCP listeners and identify plausible proxy services (for
   example Clash, Mihomo, Surge, sing-box, V2Ray, or a user-configured SOCKS
   listener); and
3. test each candidate before use. A listening port alone is not evidence that
   it can reach GitHub.

Prefer GitHub's SSH-over-443 endpoint and test with a non-mutating command.
For a candidate `<proxy-port>`, use a short timeout and SOCKS5 proxy command:

```bash
git -c core.sshCommand='ssh -o BatchMode=yes -o HostName=ssh.github.com -o ConnectTimeout=15 -o ProxyCommand="nc -x 127.0.0.1:<proxy-port> -X 5 ssh.github.com 443" -p 443' ls-remote origin HEAD
```

If it succeeds, record the port, SOCKS5 protocol, endpoint
`ssh.github.com:443`, timestamp, and successful read-only test in `ENV.md`.
Use that temporary SSH command for later `fetch`, `pull`, or `push` only while
the port continues to pass the test. If it fails, try the next candidate; do
not alter the remote URL, global Git configuration, SSH configuration, or
credentials merely to make discovery pass. Never write proxy authentication
material to `ENV.md`.

Do not expose credentials or token values.

Do not perform destructive pushes or repository mutations as part of discovery.

## ENV.md template

Use a structure similar to:

```markdown
# Research Environment

Generated at: <YYYY-MM-DD HH:MM:SS+08:00>

## Machine

- Host:
- OS:
- Architecture:
- CPU:
- RAM:
- Available disk:

## UV

- Available:
- Version:
- Python:
- Project pyproject:
- Project lock:
- Active environment:

## Conda

- Available:
- Version:

### Environments

- ...

## Compute

### Accelerators

- Type:
- Count:
- Model:
- Memory:
- Driver:
- Runtime:
- Current availability:

## Git

- Version:
- Repository root:
- Current branch:
- HEAD:
- User name:
- User email:

## GitHub

- Origin:
- Transport:
- GitHub CLI:
- Authentication status:
- Push capability:
- API/CLI capability:
- Proxy discovery: scanned / not available
- Selected proxy route: SOCKS5 127.0.0.1:<port> or none
- Read-only connectivity test: command class, endpoint, timestamp, outcome

## Notes

- ...
```

## Updating ENV.md

Update `ENV.md` when:

- moving to a new machine;
- accelerator availability changes materially;
- environment tooling changes;
- repository communication method changes;
- a previously recorded capability is no longer valid.

Do not update it merely because repository code changed.
