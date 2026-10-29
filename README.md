# workbuddy-godot-connector

WorkBuddy 连接器：让 WorkBuddy 直接操作本机正在运行的 **Godot 4.7** 编辑器——搭场景、写 GDScript、连信号、保存、运行并验证。

装好后工具列表里会有 **49 个** Godot 工具（Godot AI 的 47 个 + 本项目补的 2 个建项目/开项目工具）。

## 依赖

| 依赖 | 谁准备 | 说明 |
|---|---|---|
| **Godot 4.7+** | 你自己装 | 4.7.2 已验证，装哪都行 |
| **uv（`uvx`）** | 你自己装 | 后端是 PyPI 的 `godot-ai==4.2.3`，首次运行会下载约 73 个依赖，需联网 |
| **Node.js** | WorkBuddy 托管 | `.mcp.json` 已声明 `runtime: {type:"node", version:"22"}` |
| **WorkBuddy 5.0.0+** | — | 连接器市场版本要求 |

包内**不含 Godot 引擎**，只含 Godot AI 编辑器插件（`vendor/godot_ai`，约 2.1 MB）。

## 安装

### 方式 A：本地安装（本仓库）

```powershell
git clone https://github.com/banbixieyang/workbuddy-godot-connector
cd workbuddy-godot-connector
node install.mjs --godot "D:\Godot\Godot_v4.7.2-stable_mono_win64.exe"
```

`install.mjs` 会：把插件复制到 `~/.workbuddy/plugins/`、注册本地市场 `local-godot`、写 `enabledPlugins`，并把你给的 Godot 路径记进 `godot-path.txt`。改动前会备份三个 JSON。

然后重启 WorkBuddy，在「连接器」里启用。

### 方式 B：市场提交

按 [WorkBuddy 连接器规范](https://open.workbuddy.cn/docs/connector) 需要的 `connector-meta.json` 也在这个仓库里（`connector-meta.json`），可直接打包提交审核。

## 使用

1. 用 Godot 打开项目并保持开启（日志出现 `adopted external server` 即连接成功）。
2. 在 WorkBuddy 里对 AI 说：

> 新建一个 480×720 的 2D 躲避游戏，玩家可以左右移动，碰到障碍重生，走到终点显示胜利

它会调用 `godot_new_project` 建工程并打开编辑器，等会话出现后开始搭场景。

已经有的项目则说“打开 `D:\games\my-game`”，对应 `godot_open_project`。

## 工具

- **Godot AI 的 47 个**：`session_manage`、`session_activate`、`scene_get_hierarchy`、`node_create`、`node_set_property`、`script_create`、`script_patch`、`script_attach`、`scene_save`、`project_run`、`logs_read`、`editor_screenshot`、`ui_manage`、`signal_manage`、`resource_manage`、`batch_execute` ……
- **本项目补的 2 个**：
  - `godot_new_project(path, name?, width?, height?)` —— 建目录、写 `project.godot`、装 addon、确保后端、启动编辑器；
  - `godot_open_project(path)` —— 确保后端并打开已有项目。

## 它是怎么连上的

```
WorkBuddy ──stdio──> server.mjs ──确保后端──> godot-ai attach ──HTTP/WS──> Godot 编辑器
                        │                         │
                        └─ godot_new_project      └─ 127.0.0.1:8000 (MCP) + 9500 (编辑器 WS)
```

`server.mjs` 是个小代理：转发 Godot 的 47 个工具，自己实现 2 个项目工具，并在启动时确保 Godot AI 后端在跑（这一步同时绕开了 Windows 上 Godot 低完整性进程写不了 `%LOCALAPPDATA%`、自己起不了后端的问题）。

不直接连 `http://127.0.0.1:8000/mcp` 的原因：那个端点要求每次后端启动随机生成的 bearer capability，静态 `mcp.json` 带不了；`attach` 会自己读 capability 记录。

## 已知限制

- 一个连接器只配一个 MCP Server；多项目同时打开时靠 `session_activate` 选目标。
- 端口 8000 / 9500 被占用时会连不上（可用 `GODOT_AI_HTTP_PORT` / `GODOT_AI_WS_PORT` 覆盖）。
- 首次运行需要联网拉 `godot-ai` 依赖；离线环境需预置 uv 缓存。

## 许可

MIT。内置的 Godot AI Addon 亦为 MIT，见 `vendor/godot_ai/LICENSE`。
