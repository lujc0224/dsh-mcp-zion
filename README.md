# dsh-mcp-zion

Zion BaaS 的 DeepSeek Harness (DSH) 插件：把 `zion-mcp@2.3.0`（**插件全功能版**）以 stdio MCP 方式桥进 DSH,session 内出现 `mcp__zion__*` 42 个工具。

从 Claude Code 环境迁移而来，替代 Claude 侧 zion-nocode 插件。已端到端实测：项目上下文 / 项目列表 / 资源查询 / runtime GraphQL / schema 快照全部打通。

**内置官方 `zion-platform` skill**(476K,自 zion-nocode 插件原样捆绑)：入口做项目识别与 `typeSystem` 分流，再路由到 `pre/` 或 `post/` 版本的领域指导(database / actionflow / permissions / payments / assets / ai-agent / static-site / runtime-logs 等)—— 该 skill 正是为 `zion-mcp@2.3.0` 编写的，与本插件桥接的二进制严格同版本。安装本插件即同时获得工具与说明书。

## 两个版本，别搞混

| 版本 | 来源 | 工具数 | 能力 |
|---|---|---|---|
| `~/.npm-global/bin/zion-mcp` **v1.0.8** | 项目级 `.mcp.json` 用的 MCP | 7 | **基本只读**:get_projects / get_project_schema / 灌 mock |
| `zion-mcp@2.3.0`（本插件用的） | zion-nocode 插件实际 spawn 的（wrapper = `npx -y zion-mcp@2.3.0`） | **42** | **全能力**:database / actionflow / permissions / tpa / zai / **sync_backend 发布** / **deploy_static_site** / schema undo-redo / runtime_graphql |

本插件 spawn 与插件版完全相同的命令（`npx -y zion-mcp@2.3.0 mcp`）,工具面与 Claude 侧 zion-nocode 一字不差。

## 安装

```sh
dsh plugin --profile web add github:lujc0224/dsh-mcp-zion
```

或本地路径安装：

```sh
dsh plugin --profile web add /path/to/dsh-mcp-zion
```

重启 profile（`dsh web`）后生效。验证：新 session 里出现 `mcp__zion__*` 工具，可跑 `get_current_project` 确认项目绑定。

## 设计决策

| 决策 | 选择 | 理由 |
|---|---|---|
| 版本 | `npx -y zion-mcp@2.3.0 mcp` | 与 zion-nocode 插件同命令同版本；v1.0.8 只读版不够用 |
| cwd | 固定 Zion 项目根 | zion-mcp 从 cwd 向上找 `.zion-mcp/credentials.json`，固定即复用现有登录态与 project-context |
| 超时 | 180s | sync_backend（发布）/ schema session 超过默认 60s |
| 启动容错 | `failOnStartupError` 默认 false | npx 缺失时降级为「无 zion 工具」，不阻塞 profile 启动 |

## 使用前提

- Node + npm 可用（npx 拉 zion-mcp@2.3.0,首次几秒、之后走缓存）
- Zion 项目目录下已有 `.zion-mcp/`（含 credentials.json;首次需在项目内登录一次生成）
- **换机器/换项目**：改 `cordis.patch.yml` 里两处绝对路径（`cwd` 与 `env.WORKSPACE_ROOT`）

## 注意

- `delete_project` / `reset_project` 等不可逆操作自带原生 GUI 确认弹窗，DSH 侧调用同样会触发
- 桥接件 `@deepseek-ai/dsh-mcp-client` 由 dsh 部署自身提供（与 zhipu MCP 行同源），无需在此声明
