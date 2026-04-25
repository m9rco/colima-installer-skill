---
name: colima-installer
description: >
  Install a lightweight Docker environment on macOS using Colima, with
  pre-configured China-region registry mirrors. Use when the user asks to
  install Docker, set up a Docker alternative, avoid Docker Desktop, or needs
  a CLI-only/headless container runtime on Mac. Trigger phrases: 安装Docker,
  轻量docker, Colima, docker替代, Docker Desktop替代, 无GUI docker, 命令行docker.
allowed-tools:
  - Bash(brew *)
  - Bash(colima *)
  - Bash(docker *)
  - Bash(which *)
  - Bash(tee *)
  - Bash(cat *)
---

# colima-installer

Install Colima + docker CLI on macOS, pre-configured with China-region registry
mirrors. Pure CLI, no GUI, no Docker Desktop bloat.

**Repo:** https://github.com/m9rco/colima-installer-skill

## When to use

Trigger this skill whenever the user wants to:

- Install Docker on macOS without Docker Desktop
- Set up a lightweight container runtime (no GUI)
- Replace Docker Desktop / OrbStack with something smaller
- Get Docker working quickly with Chinese mirror acceleration

## Environment check

!`sw_vers -productName 2>/dev/null | head -1`
!`which brew 2>/dev/null && brew --version | head -1 || echo "❌ Homebrew required"`
!`which colima 2>/dev/null && colima version 2>/dev/null | head -1 || echo "ℹ️ Colima not installed yet"`
!`which docker 2>/dev/null && docker version --format 'Client: {{.Client.Version}}' 2>/dev/null || echo "ℹ️ Docker CLI not installed yet"`
!`colima status 2>&1 | grep -E "running|not running" | head -1 || true`

## Step-by-step installation

### 1. Install Colima and docker CLI

```bash
brew install colima docker
```

This installs Colima (~10MB) and the `docker` CLI client (~28MB).
**Not** Docker Desktop — just the client binary.

### 2. Clean stale Docker config

macOS may have a stale `credsStore: osxkeychain` entry from previous
Docker Desktop installs. If present, `docker run` will fail with
`docker-credential-osxkeychain not found`.

```bash
tee ~/.docker/config.json > /dev/null <<'EOF'
{
  "auths": {},
  "currentContext": "colima"
}
EOF
```

### 3. Start Colima

Recommended parameters for Apple Silicon:

```bash
colima start \
  --cpu 2 \
  --memory 2 \
  --disk 20 \
  --vm-type=vz \
  --vz-rosetta \
  --mount-type=virtiofs
```

- `--vm-type=vz`: macOS native Virtualization.Framework (fastest)
- `--vz-rosetta`: run x86_64 containers on Apple Silicon
- `--mount-type=virtiofs`: high-performance shared filesystem

For Intel Mac, omit `--vz-rosetta`. For macOS < 13, use `--vm-type=qemu`.

### 4. Configure China-region registry mirrors

> ⚠️ **Critical gotcha**: `~/.docker/daemon.json` on the host is **not** picked
> up by the dockerd running inside Colima's VM. You must write the config
> **inside the VM** via `colima ssh`.

```bash
colima ssh -- sudo tee /etc/docker/daemon.json > /dev/null <<'EOF'
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "https://mirror.ccs.tencentyun.com",
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ]
}
EOF

colima ssh -- sudo systemctl restart docker
```

### 5. Verify

```bash
colima status
docker version
docker info | grep -A 4 "Registry Mirrors"
docker run --rm hello-world
```

Expected: `Hello from Docker!` within 60 seconds.

## Common commands

| Task | Command |
|------|---------|
| Start | `colima start` |
| Stop | `colima stop` |
| Status | `colima status` |
| Restart | `colima restart` |
| Enter VM | `colima ssh` |
| Delete instance | `colima delete` |
| Auto-start on login | `brew services start colima` |

## Resource tuning

```bash
colima stop
colima start --cpu 4 --memory 4    # bump specs
```

## Troubleshooting

### `docker-credential-osxkeychain not found`

Stale config from Docker Desktop. Run step 2 above.

### `Cannot connect to the Docker daemon`

```bash
colima status              # check if running
colima start               # start if stopped
docker context use colima  # switch context
```

### Slow image pulls

- Verify mirrors are configured **inside the VM**: `colima ssh -- cat /etc/docker/daemon.json`
- If host-side `~/.docker/daemon.json` has `registry-mirrors`, that's wrong
  (ineffective for Colima). Move the config into the VM per step 4.

### VM fails to start

- macOS < 13: replace `--vm-type=vz` with `--vm-type=qemu`
- Apple Silicon without Rosetta: remove `--vz-rosetta`
- Clean reset: `colima delete` then `colima start ...`

## Verified versions

- macOS 14+ on Apple Silicon
- Homebrew 5.x
- Colima 0.10+
- Docker Engine 29.x

## Links

- Colima: https://github.com/abiosoft/colima
- WorkBuddy docs: https://www.codebuddy.cn/docs/workbuddy/Overview
