---
name: colima-installer
description: |
  在 macOS 上安装轻量级 Docker 环境（Colima + docker CLI），预配国内镜像加速器。
  触发词：安装Docker、安装docker、轻量级docker、轻量docker、Colima、colima、docker替代、无GUI docker、命令行docker、Docker Desktop替代、Mac docker。
  适用于 macOS 用户需要纯命令行的 Docker 环境，比 Docker Desktop 和 OrbStack 更轻量，无 GUI。
---

# Colima Installer Skill

## 概述

**Colima**（Containers on Lima）是 macOS 上最轻量的 Docker 方案之一：
- 纯 CLI，无 GUI（不需要就不装 GUI 层）
- 底层 Lima + macOS Virtualization.Framework（原生虚拟化）
- 资源占用低（默认 2 核 2G，可调）
- 完全兼容 `docker` 命令
- 支持 Rosetta 跨架构（Apple Silicon 上跑 x86_64 镜像）

## 技术栈对比

| 方案 | GUI | VM内核 | 启动 | 资源占用 |
|------|-----|--------|------|---------|
| Docker Desktop | ✅ | 自带 | 慢 | 重 |
| OrbStack | ✅ | 自研轻量VM | 2-3s | 中 |
| **Colima** | ❌ | Lima + VZ | 5-10s | **轻** |

## 执行步骤

### 前置检查

```bash
which brew && brew --version
docker version 2>/dev/null || echo "Docker 未安装"
colima status 2>/dev/null || echo "Colima 未运行"
```

### 1. 安装 Colima 和 docker CLI

```bash
brew install colima docker
```

注意：这里的 `docker` 只是 CLI 客户端（约 28MB），不是 Docker Desktop。

### 2. 清理可能存在的旧配置

如果之前装过 Docker Desktop 或其他方案，`~/.docker/config.json` 可能含 `credsStore: osxkeychain` 会导致 `docker run` 失败。清理掉：

```bash
tee ~/.docker/config.json > /dev/null <<'EOF'
{
  "auths": {},
  "currentContext": "colima"
}
EOF
```

### 3. 启动 Colima

```bash
# 推荐参数：2核 / 2G / 20G / VZ虚拟化 / Rosetta跨架构 / virtiofs高性能挂载
colima start --cpu 2 --memory 2 --disk 20 --vm-type=vz --vz-rosetta --mount-type=virtiofs
```

首次启动约 30-60 秒（Colima 内置了 Alpine 镜像，不需要额外下载 Ubuntu）。

### 4. 配置国内镜像加速器

> ⚠️ 重点：`~/.docker/daemon.json` 是 **host 侧配置，不会传到 VM 内**。必须配到 VM 内的 `/etc/docker/daemon.json`。

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

# 重启 Docker daemon 使配置生效
colima ssh -- sudo systemctl restart docker
sleep 3
```

### 5. 验证

```bash
# 状态
colima status
docker version

# 镜像加速器生效
docker info | grep -A 4 "Registry Mirrors"
# 应输出 4 条 mirror URL

# 端到端测试
docker run --rm hello-world
```

## 常用命令

| 操作 | 命令 |
|------|------|
| 启动 | `colima start` |
| 停止 | `colima stop` |
| 重启 | `colima restart` |
| 状态 | `colima status` |
| 进入 VM | `colima ssh` |
| 删除实例 | `colima delete` |
| 开机自启 | `brew services start colima` |

## 资源调优

```bash
# 修改 CPU/内存后需要重启 VM
colima stop
colima start --cpu 4 --memory 4
```

## 故障排查

### `docker-credential-osxkeychain not found`
见步骤 2，清理 `~/.docker/config.json`。

### 镜像拉取慢或失败
检查镜像加速器是否真的配到 VM 内（`colima ssh -- cat /etc/docker/daemon.json`），而不是 host 的 `~/.docker/daemon.json`。

### VM 启动失败
- 确保 macOS >= 13（VZ 虚拟化要求）
- 如果 `--vm-type=vz` 失败，改用 `--vm-type=qemu`
- Apple Silicon 不用 Rosetta 跨架构可去掉 `--vz-rosetta`

## 参考资料

- 官网：https://github.com/abiosoft/colima
- docs：https://github.com/abiosoft/colima/blob/main/docs/FAQ.md
- 国内镜像加速：中科大 / 腾讯云 / 网易 / 百度云
