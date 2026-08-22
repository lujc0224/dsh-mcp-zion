# dsh-mcp-zion

Zion BaaS 的 DSH 插件：把 `zion-mcp@2.3.0`（**插件全功能版**）以 stdio MCP 方式桥进 DeepSeek Harness，session 内出现 `mcp__zion__*` 42 个工具。

从 Claude Code 环境清单（CLAUDE-ENV-INVENTORY.md）迁移而来，替代 Claude 侧 zion-nocode 插件。

## 两个版本，别搞混

| 版本 | 来源 | 工具数 | 能力 |
|---|---|---|---|
| `~/.npm-global/bin/zion-mcp` **v1.0.8** | 项目级 `.mcp.json` 用的 MCP | 7 | **基本只读**：get_projects / get_project_schema / 灌 mock |
| `zion-mcp@2.3.0`（本插件用的） | zion-nocode 插件实际 spawn 的（wrapper = `npx -y zion-mcp@2.3.0`） | **42** | **全能力**：database / actionflow / permissions / tpa / zai / **sync_backend 发布** / **deploy_static_site** / schema undo-redo / runtime_graphql |

本插件 spawn 与插件版完全相同的命令（`npx -y zion-mcp@2.3.0 mcp`），工具面与 Claude 侧 zion-nocode 一字不差（已实测 tools/list 对比）。

## 安装

```sh
dsh plugin --profile web add /Users/lujuncheng/project/09-dsh-plugins/packages/dsh-mcp-zion
```

发布到 GitHub 后可改用：

```sh
dsh plugin --profile web add github:lujc0224-stack/dsh-plugins#packages/dsh-mcp-zion
```

重启 profile（`dsh web`）后生效，验证方式：新 session 里应看到 `mcp__zion__*` 工具（42 个）。

## 设计决策

| 决策 | 选择 | 理由 |
|---|---|---|
| 版本 | `npx -y zion-mcp@2.3.0 mcp` | 与 zion-nocode 插件同命令同版本；v1.0.8 只读版不够用 |
| cwd | 固定 `~/project/01-mesonCloud` | zion-mcp 从 cwd 向上找 `.zion-mcp/credentials.json`，固定即复用现有登录态与 project-context |
| 超时 | 180s | sync_backend（发布）/ schema session 超过默认 60s |
| 启动容错 | `failOnStartupError` 默认 false | npx 缺失时降级为「无 zion 工具」，不阻塞 profile 启动 |

## 注意

- `delete_project` / `reset_project` 等不可逆操作自带原生 GUI 确认弹窗，DSH 侧调用同样会触发
- 首次 npx 解析有几秒开销（之后走 npm 缓存）
- 依赖：Node + npm 可用；mesonCloud 项目目录下已有 `.zion-mcp/`（含 credentials.json）
- 桥接件 `@deepseek-ai/dsh-mcp-client` 由 dsh 部署自身提供（与 profile 内 zhipu MCP 行同源），无需在此声明
