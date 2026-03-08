# 部署、脚本与客户端接入

## 为什么这篇重要

这个仓库不只是一个后端服务，它同时包含：

- Web Dashboard
- MCP 服务
- Docker 部署入口
- 多 CLI / skill 接入文档与同步脚本

如果不把这几层分开，很容易把“项目功能”“部署方式”“客户端接入”混成一件事。

## Docker 运行形态

主入口：`docker-compose.yml`

默认三服务：

- `backend`
  - FastAPI 主服务，对外暴露后端端口
- `sse`
  - 复用 backend 镜像，以 `python run_sse.py` 运行 SSE 层
- `frontend`
  - 仪表盘前端服务，依赖 `backend` 与 `sse`

关键环境入口：

- `MEMORY_PALACE_DOCKER_ENV_FILE`
- `MEMORY_PALACE_BACKEND_PORT`
- `MEMORY_PALACE_FRONTEND_PORT`

## Profiles 的定位

核心文档：`docs/DEPLOYMENT_PROFILES.md`  
配置模板目录：`deploy/profiles/`

建议这样理解：

- A
  - 纯关键词，最低配置
- B
  - 默认起步档位，hash embedding，无外部服务依赖
- C
  - 强烈推荐的本地模型服务档位
- D
  - 远程 API / 云端模型服务档位

如果只是第一次跑通项目，优先 B；  
如果已经有 embedding/reranker 服务，优先看 C。

## 关键脚本

### 部署与环境

- `scripts/docker_one_click.sh`
- `scripts/docker_one_click.ps1`
- `scripts/apply_profile.sh`
- `scripts/apply_profile.ps1`
- `scripts/run_memory_palace_mcp_stdio.sh`

用途：

- 从 profile 生成本地运行时环境文件（模板参考 `.env.example`）
- 一键启动 Docker
- 统一不同平台的部署入口
- 为 CLI / MCP 注册提供稳定的 stdio 启动脚本

额外要点：

- `scripts/docker_one_click.sh`
  - 已包含 deployment lock，避免同一 checkout 并发部署互相踩踏

### 交付前检查

- `scripts/pre_publish_check.sh`

用途：

- 扫描本地产物
- 扫描敏感文件是否被跟踪
- 检查 `.env.example` 是否保留空占位

如果你要公开分享仓库或准备交付，这是很值得先跑的一步。

### Skill 同步与安装

- `scripts/sync_memory_palace_skill.py`
  - 把 canonical bundle 同步到 repo-local mirrors
- `scripts/install_skill.py`
  - 安装到 workspace 或 user scope，并可顺带注册 MCP 绑定

这两者的分工不同：

- `sync_*`
  - 更偏“当前仓库维护者”的镜像同步
- `install_*`
  - 更偏“把这套能力装到某个工作区/用户环境”

## Skills 文档与 canonical bundle

文档入口在 `docs/skills/`，建议按这个顺序读：

1. `docs/skills/GETTING_STARTED.md`
2. `docs/skills/SKILLS_QUICKSTART.md`
3. `docs/skills/MEMORY_PALACE_SKILLS.md`

真正的 canonical bundle 在：

- `docs/skills/memory-palace/`

里面最关键的是：

- `SKILL.md`
- `references/`
- `variants/`
- `agents/openai.yaml`

一句话理解：

```text
docs/skills/*.md
  = 给人读的使用说明

docs/skills/memory-palace/
  = 给模型和分发脚本使用的 canonical bundle
```

## 多客户端接入的认知边界

从 `docs/skills/SKILLS_QUICKSTART.md` 可归纳出当前口径：

- Claude / Codex / OpenCode / Gemini 都有文档化接入路径
- 但不同客户端的“skill 可见性”“MCP 配置位置”“workspace vs user-scope”
  不完全一样
- 尤其 Gemini 仍保留一定边界说明，不宜简单写成“完全无条件开箱即用”

因此排查接入问题时，先区分你卡在的是：

1. skill 镜像是否已同步
2. MCP server 是否已注册
3. 当前 CLI 看的是 workspace 配置还是 user 配置

## 推荐排障入口

### 服务能起，但页面打不开

先看：

1. `docker-compose.yml`
2. `deploy/docker/`
3. `docs/DEPLOYMENT_PROFILES.md`

### 本地 profile 不知道怎么选

先看：

1. `docs/DEPLOYMENT_PROFILES.md`
2. `deploy/profiles/`

### 技能同步后仍然触发不到

先看：

1. `docs/skills/SKILLS_QUICKSTART.md`
2. `scripts/sync_memory_palace_skill.py`
3. `scripts/install_skill.py`
4. `scripts/run_memory_palace_mcp_stdio.sh`

### 想知道“这份 skill 的权威源头在哪”

先看：

1. `docs/skills/README.md`
2. `docs/skills/memory-palace/`

## 推荐下一跳

- 想看后端运行时与 API：
  - 读 `backend-runtime-and-api.md`
- 想看前端与鉴权：
  - 读 `dashboard-and-auth-flow.md`
