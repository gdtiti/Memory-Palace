# Dashboard 与鉴权流

## 先记一个结论

前端并不是把 API Key 写死在构建产物里，而是优先走**运行时注入**，
其次才回退到浏览器本地存储。

## 页面壳层入口

主入口：`frontend/src/App.jsx`

这里决定了三件事：

1. 页面路由
   - `/memory`
   - `/review`
   - `/maintenance`
   - `/observability`
2. 顶部导航与 API Key 操作按钮
3. 把鉴权状态传给整个应用壳层

如果你只看一个前端文件，优先先看这个。

## API Key 从哪里来

核心文件：`frontend/src/lib/api.js`

鉴权状态读取顺序：

1. `window.__MEMORY_PALACE_RUNTIME__`
2. `window.__MCP_RUNTIME_CONFIG__`
3. `localStorage` 中的 `memory-palace.dashboardAuth`

规范化逻辑由这些函数负责：

- `normalizeMaintenanceAuth()`
- `getMaintenanceAuthState()`
- `saveStoredMaintenanceAuth()`
- `clearStoredMaintenanceAuth()`

支持两种传输方式：

- `header`
  - 注入 `X-MCP-API-Key`
- `bearer`
  - 注入 `Authorization: Bearer ...`

## 哪些请求会自动带鉴权

还是在 `frontend/src/lib/api.js`：

- `isProtectedPath()`
- `isProtectedApiRequest()`
- axios request interceptor

当前被视为受保护的路径有：

- `/browse/`
- `/review/`
- `/maintenance/`

也就是说，Dashboard 的核心数据请求默认都需要鉴权。

## 四个主页面各干什么

### Memory

文件：`frontend/src/features/memory/MemoryBrowser.jsx`

职责：

- 浏览树状记忆
- 查看 gist / breadcrumb / alias
- 创建、编辑、删除节点

对应的 API 连接点：

- `getMemoryNode`
- `createMemoryNode`
- `updateMemoryNode`
- `deleteMemoryNode`

### Review

文件：`frontend/src/features/review/ReviewPage.jsx`

职责：

- 查看 sessions / snapshots / diff
- rollback / integrate

对应的 API 连接点：

- `getSessions`
- `getSnapshots`
- `getDiff`
- `rollbackResource`
- `approveSnapshot`

### Maintenance

文件：`frontend/src/features/maintenance/MaintenancePage.jsx`

职责：

- orphan 清理
- vitality cleanup 候选查询、prepare、confirm
- 手动触发衰减

对应的 API 连接点：

- `listOrphanMemories`
- `getOrphanMemoryDetail`
- `deleteOrphanMemory`
- `triggerVitalityDecay`
- `queryVitalityCleanupCandidates`
- `prepareVitalityCleanup`
- `confirmVitalityCleanup`

### Observability

文件：`frontend/src/features/observability/ObservabilityPage.jsx`

职责：

- 搜索观测
- 运行态概览
- 索引 worker / 检索侧状态观察

主要还是调用 `frontend/src/lib/api.js` 中的 observability 与 summary 接口。

## 常见排障入口

### 页面 401

先看：

1. `frontend/src/lib/api.js`
2. `frontend/src/App.jsx`
3. `backend/api/maintenance.py`

### 页面看得到壳层，但数据空白

先看：

1. 对应 `features/<page>/`
2. `frontend/src/lib/api.js`
3. 后端对应的 `backend/api/*.py`

### 页面按钮点击后提示 write guard blocked

先看：

1. `frontend/src/features/memory/MemoryBrowser.jsx`
2. `backend/api/browse.py`
3. `backend/db/sqlite_client.py`

## 部署时需要记住的前端事实

- 前端默认 `baseURL` 是 `/api`
- 是否自动带 key，取决于运行时注入或本地存储是否存在
- Docker / 代理层可以把 key 处理放在服务端，不一定非要让用户手动点按钮

## 推荐下一跳

- 想看后端如何验证这些请求：
  - 读 `backend-runtime-and-api.md`
- 想看部署和脚本怎么把前后端串起来：
  - 读 `deployment-and-client-integration.md`
