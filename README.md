# colima-installer

> A WorkBuddy / OpenClaw skill that installs a lightweight Docker environment on macOS using Colima, pre-configured with China-region registry mirrors.

一个 WorkBuddy / OpenClaw 技能包，用于在 macOS 上安装轻量级 Docker 环境（Colima + docker CLI），并预配好国内镜像加速器。

## 🎯 用途

当你在 macOS 上需要 Docker，又不想装笨重的 Docker Desktop 时，让 AI 助手帮你一键完成：

- 安装 `colima` 和 `docker` CLI（通过 Homebrew）
- 启动 Colima VM（VZ 虚拟化 + Rosetta 跨架构）
- 配置 4 个国内 Docker registry 镜像加速器
- 处理 macOS 常见坑（`docker-credential-osxkeychain`、zsh `no_clobber`、VM 内外配置差异）

## 📦 对比

| 方案 | GUI | 启动速度 | 资源占用 | 大小 |
|------|-----|----------|---------|------|
| Docker Desktop | ✅ | 慢 | 重（1.5-2GB） | ~1GB |
| OrbStack | ✅ | 2-3s | 中（300-500MB） | ~200MB |
| **Colima**（本 skill） | ❌ | 5-10s | **轻（按需）** | **10MB** |

## 🚀 如何使用这个 skill

### 方式 1：用 WorkBuddy 导入（推荐）

1. 打开 WorkBuddy 客户端
2. 进入 **技能市场** → **添加技能** → **上传技能**
3. 选择本仓库打包后的 zip 文件，或直接从 SkillHub 一键安装

### 方式 2：手动安装到本地

```bash
mkdir -p ~/.workbuddy/skills/colima-installer
curl -fsSL https://raw.githubusercontent.com/<YOUR_GITHUB>/colima-installer/main/SKILL.md \
  -o ~/.workbuddy/skills/colima-installer/SKILL.md
```

然后在 AI 对话里说"帮我安装 Colima / 安装轻量级 Docker"即可触发。

## ⚡ 触发词

- 安装 Docker / 安装 docker
- 轻量级 docker / 轻量 docker
- Colima / colima
- docker 替代 / Docker Desktop 替代
- 无 GUI docker / 命令行 docker
- Mac docker

## 🔧 skill 内容

核心文件 [`SKILL.md`](./SKILL.md) 包含：

- 前置环境检查步骤
- Homebrew 安装命令
- `~/.docker/config.json` 残留清理（避免 `docker-credential-osxkeychain` 报错）
- Colima 启动推荐参数（2 核 / 2G / 20G / VZ / Rosetta / virtiofs）
- **重点**：将 registry mirrors 写到 VM 内部的 `/etc/docker/daemon.json`（host 侧 `~/.docker/daemon.json` 对 Colima 无效）
- 端到端验证（`docker run hello-world`）
- 常用命令速查表
- 故障排查

## 🧪 验证通过的环境

- macOS 14+ (Apple Silicon M1/M2/M3/M4)
- Homebrew 5.x
- Colima 0.10+
- Docker Engine 29.x

## 📜 License

MIT License —— 详见 [LICENSE](./LICENSE)

## 🤝 贡献

欢迎提 Issue 或 PR：
- 发现新的 macOS 兼容性问题
- 补充其他国内镜像源
- 增加 `nerdctl` / Podman 的变体

## 🔗 相关资源

- [Colima 官方仓库](https://github.com/abiosoft/colima)
- [WorkBuddy 文档](https://www.codebuddy.cn/docs/workbuddy/Overview)
- [OpenClaw Skill 规范](https://clawhub.ai)
- [SkillHub 技能市场](https://skillhub.tencent.com)
