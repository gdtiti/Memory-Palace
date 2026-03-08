# 项目概览

## 产品目标

Memory Palace 为 AI Agent 提供持久化上下文，让跨会话协作不再依赖短期
上下文窗口。它把“记忆写入、检索、审查、治理、观察”串成一条完整链路，
目标是让 Agent 的长期记忆既可用，也可控、可追踪、可回滚。

## 主要能力

- 持久化记忆存储
  - 使用 SQLite 保存记忆、路径、快照与相关状态
- 可审计写入链路
  - 写入前经过 Write Guard，写入时生成快照，写入后异步重建索引
- 统一检索能力
  - 支持 `keyword`、`semantic`、`hybrid` 三种模式，并带意图分类
- 记忆治理
  - 支持 review / rollback、孤儿清理、vitality 衰减、sleep consolidation
- 多客户端接入
  - 通过 MCP 暴露能力，面向 Codex、Claude Code、Gemini CLI、OpenCode
- 可视化运维与观察
  - React 仪表盘提供 Memory、Review、Maintenance、Observability 页面

## 技术栈

### 后端

- Python 3.10+
- FastAPI
- SQLAlchemy + aiosqlite
- Pydantic
- MCP / FastMCP
- httpx / requests

### 前端

- React 18
- Vite 7
- Tailwind CSS 3
- Framer Motion
- React Router DOM
- Axios

### 部署与脚本

- Docker Compose
- `deploy/profiles/` 多档位配置
- `deploy/docker/` 容器镜像与入口脚本
- `scripts/` 一键部署、备份、技能同步、评测与发布前检查

## 运行时形态

### HTTP / Dashboard

- `backend/main.py` 启动 FastAPI 服务
- 公开 `/docs`、`/health`
- 注册三组核心路由：
  - `/browse`
  - `/review`
  - `/maintenance`

### MCP

- `backend/mcp_server.py` 提供核心 MCP 工具
- `backend/run_sse.py` 提供 SSE 传输层
- `backend/mcp_wrapper.py` 负责封装启动

### 前端页面

- `frontend/src/App.jsx` 负责路由壳层
- 四个主页面分别位于：
  - `frontend/src/features/memory/`
  - `frontend/src/features/review/`
  - `frontend/src/features/maintenance/`
  - `frontend/src/features/observability/`

## 部署说明

- 本仓库已提供 `docker-compose.yml`，默认包含：
  - `backend`
  - `sse`
  - `frontend`
- `deploy/profiles/` 下提供 macOS / Windows / Docker 多套环境模板
- `scripts/docker_one_click.ps1` 与 `scripts/docker_one_click.sh` 用于快速部署
- 文档口径偏保守：已给出推荐路径，但不同宿主环境仍需要按实际环境复核

## 推荐的项目认知顺序

1. 先读 `README.md` 或 `README_CN.md`
2. 再读 `docs/TECHNICAL_OVERVIEW.md`
3. 然后看 `llmdoc/architecture/retrieval-map.md`
4. 最后按任务场景跳转到具体源码与测试
