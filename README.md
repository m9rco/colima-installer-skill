# colima-installer-skill

> An AI skill (Claude Code / CodeBuddy / WorkBuddy) that installs a lightweight Docker environment on macOS using Colima, with pre-configured China-region registry mirrors.

一个 AI 技能包，让你的 AI 助手一键为 macOS 安装轻量级 Docker（Colima + docker CLI），并预配好国内镜像加速器。

## ✨ 功能

- **一句话触发**：对 AI 说"帮我装轻量 Docker"即可
- **纯 CLI 无 GUI**：不装 Docker Desktop，仅 ~40MB
- **国内镜像加速**：预配 4 个国内镜像源（中科大 / 腾讯云 / 网易 / 百度云）
- **避开常见坑**：处理 macOS 上的 `docker-credential-osxkeychain`、zsh `no_clobber`、VM 内外 `daemon.json` 区别等

## 📊 对比

| 方案 | GUI | 启动速度 | 资源占用 | 总大小 |
|------|-----|----------|---------|--------|
| Docker Desktop | ✅ | 慢 | 重（1.5-2GB） | ~1GB |
| OrbStack | ✅ | 2-3s | 中（300-500MB） | ~200MB |
| **Colima**（本 skill） | ❌ | 5-10s | **轻（按需）** | **~40MB** |

## 🚀 安装

### 从 Release 安装（推荐）

用户级（跨项目生效）：

```bash
mkdir -p ~/.codebuddy/skills
curl -fsSL https://github.com/m9rco/colima-installer-skill/releases/latest/download/skill-colima-installer.zip \
  -o /tmp/skill-colima-installer.zip && \
  unzip -o /tmp/skill-colima-installer.zip -d ~/.codebuddy/skills/
```

项目级（仅当前项目生效）：

```bash
mkdir -p .codebuddy/skills
curl -fsSL https://github.com/m9rco/colima-installer-skill/releases/latest/download/skill-colima-installer.zip \
  -o /tmp/skill-colima-installer.zip && \
  unzip -o /tmp/skill-colima-installer.zip -d .codebuddy/skills/
```

对 Claude Code：把 `.codebuddy` 替换为 `.claude` 即可。

### 从 ClawHub / SkillHub 安装

```bash
clawhub install colima-installer
```

## 💬 使用

装好后，在 WorkBuddy / CodeBuddy / Claude Code 对话里直接说：

> "帮我安装轻量级 Docker"
> "Install Colima on my Mac"
> "I need Docker but don't want Docker Desktop"

AI 会自动触发本 skill，跑完整套安装 + 配置 + 验证流程。

## 🔧 Skill 内容

核心文件 [`.codebuddy/skills/colima-installer/SKILL.md`](./.codebuddy/skills/colima-installer/SKILL.md) 包含：

- 环境检测（macOS 版本 / Homebrew / Colima / Docker）
- `brew install colima docker`
- 清理残留 `~/.docker/config.json`（避免 osxkeychain 报错）
- Colima 启动推荐参数（VZ + Rosetta + virtiofs）
- **重点**：将 registry mirrors 写到 VM 内部的 `/etc/docker/daemon.json`
- 端到端验证（`docker run hello-world`）
- 常用命令速查表 & 故障排查

## 🧪 验证通过的环境

- macOS 14+（Apple Silicon M1/M2/M3/M4）
- Homebrew 5.x
- Colima 0.10+
- Docker Engine 29.x

## 🔄 自动发布

仓库每次 push main 分支都会自动构建 nightly release；打 tag `v*.*.*` 则发布正式版本到 GitHub Releases 和 ClawHub。

```bash
# 发版
git tag -a v1.0.0 -m "Release v1.0.0" && git push origin v1.0.0
```

## 📜 License

MIT

## 🔗 链接

- [Colima 官方仓库](https://github.com/abiosoft/colima)
- [WorkBuddy 文档](https://www.codebuddy.cn/docs/workbuddy/Overview)
- [ClawHub / SkillHub](https://skillhub.tencent.com)
- [姊妹项目：p4u-skill](https://github.com/m9rco/p4u-skill)
