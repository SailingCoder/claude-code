[English](README.md) | **中文**

# Claude Code 源代码快照（安全研究用途）

> 本仓库镜像了一份**公开暴露的 Claude Code 源代码快照**，该快照于 **2026年3月31日** 通过 npm 分发包中的 source map 泄露被公开访问。本仓库用于**教育、防御性安全研究和软件供应链分析**。

---

## 研究背景

本仓库由一名**大学生**维护，研究方向包括：

- 软件供应链暴露与构建产物泄露
- 安全软件工程实践
- AI 代理开发工具架构
- 真实 CLI 系统的防御性分析

本档案旨在支持：

- 教育学习
- 安全研究实践
- 架构审查
- 讨论打包和发布流程中的安全隐患

本仓库**不**声称对原始代码拥有所有权，也不应被视为 Anthropic 官方仓库。

---

## 源代码如何被公开访问

[Chaofan Shou (@Fried_rice)](https://x.com/Fried_rice) 公开指出，Claude Code 的源代码可以通过 npm 包中暴露的 `.map` 文件获取：

> **"Claude code 的源代码通过 npm registry 中的 map 文件泄露了！"**
>
> — [@Fried_rice, 2026年3月31日](https://x.com/Fried_rice/status/2038894956459290963)

已发布的 source map 引用了托管在 Anthropic R2 存储桶中的未混淆 TypeScript 源代码，使得 `src/` 快照可被公开下载。

---

## 仓库范围

Claude Code 是 Anthropic 的 CLI 工具，用于在终端中与 Claude 交互，执行软件工程任务，包括编辑文件、运行命令、搜索代码库和协调工作流。

本仓库包含用于研究和分析的 `src/` 目录镜像快照。

- **公开暴露时间**：2026-03-31
- **编程语言**：TypeScript
- **运行时**：Bun
- **终端 UI**：React + [Ink](https://github.com/vadimdemedes/ink)
- **规模**：约 1,900 个文件，512,000+ 行代码

---

## 目录结构

```text
src/
├── main.tsx                 # 入口编排（基于 Commander.js 的 CLI 路径）
├── commands.ts              # 命令注册表
├── tools.ts                 # 工具注册表
├── Tool.ts                  # 工具类型定义
├── QueryEngine.ts           # LLM 查询引擎
├── context.ts               # 系统/用户上下文收集
├── cost-tracker.ts          # Token 用量追踪
│
├── commands/                # 斜杠命令实现（约 50 个）
├── tools/                   # Agent 工具实现（约 40 个）
├── components/              # Ink UI 组件（约 140 个）
├── hooks/                   # React Hooks
├── services/                # 外部服务集成
├── screens/                 # 全屏 UI（诊断、REPL、恢复）
├── types/                   # TypeScript 类型定义
├── utils/                   # 工具函数
│
├── bridge/                  # IDE 和远程控制桥接层
├── coordinator/             # 多 Agent 协调器
├── plugins/                 # 插件系统
├── skills/                  # 技能系统
├── keybindings/             # 快捷键配置
├── vim/                     # Vim 模式
├── voice/                   # 语音输入
├── remote/                  # 远程会话
├── server/                  # 服务器模式
├── memdir/                  # 持久化记忆目录
├── tasks/                   # 任务管理
├── state/                   # 状态管理
├── migrations/              # 配置迁移
├── schemas/                 # 配置 Schema（Zod）
├── entrypoints/             # 初始化逻辑
├── ink/                     # Ink 渲染器封装
├── buddy/                   # 伴侣精灵
├── native-ts/               # 原生 TypeScript 工具
├── outputStyles/            # 输出样式
├── query/                   # 查询管道
└── upstreamproxy/           # 代理配置
```

---

## 架构概览

### 1. 工具系统 (`src/tools/`)

Claude Code 可调用的每个工具都以独立模块实现。每个工具定义了输入 Schema、权限模型和执行逻辑。

| 工具 | 描述 |
|---|---|
| `BashTool` | Shell 命令执行 |
| `FileReadTool` | 文件读取（图片、PDF、Notebook） |
| `FileWriteTool` | 文件创建/覆盖 |
| `FileEditTool` | 文件局部修改（字符串替换） |
| `GlobTool` | 文件模式匹配搜索 |
| `GrepTool` | 基于 ripgrep 的内容搜索 |
| `WebFetchTool` | 获取 URL 内容 |
| `WebSearchTool` | 网络搜索 |
| `AgentTool` | 子 Agent 生成 |
| `SkillTool` | 技能执行 |
| `MCPTool` | MCP 服务器工具调用 |
| `LSPTool` | 语言服务器协议集成 |
| `NotebookEditTool` | Jupyter Notebook 编辑 |
| `TaskCreateTool` / `TaskUpdateTool` | 任务创建与管理 |
| `SendMessageTool` | Agent 间消息传递 |
| `TeamCreateTool` / `TeamDeleteTool` | 团队 Agent 管理 |
| `EnterPlanModeTool` / `ExitPlanModeTool` | 计划模式切换 |
| `EnterWorktreeTool` / `ExitWorktreeTool` | Git Worktree 隔离 |
| `ToolSearchTool` | 延迟工具发现 |
| `CronCreateTool` | 定时触发器创建 |
| `RemoteTriggerTool` | 远程触发器 |
| `SleepTool` | 主动模式等待 |
| `SyntheticOutputTool` | 结构化输出生成 |

### 2. 命令系统 (`src/commands/`)

用户通过 `/` 前缀调用的斜杠命令。

| 命令 | 描述 |
|---|---|
| `/commit` | 创建 Git 提交 |
| `/review` | 代码审查 |
| `/compact` | 上下文压缩 |
| `/mcp` | MCP 服务器管理 |
| `/config` | 设置管理 |
| `/doctor` | 环境诊断 |
| `/login` / `/logout` | 身份认证 |
| `/memory` | 持久化记忆管理 |
| `/skills` | 技能管理 |
| `/tasks` | 任务管理 |
| `/vim` | Vim 模式切换 |
| `/diff` | 查看变更 |
| `/cost` | 查看使用费用 |
| `/theme` | 切换主题 |
| `/context` | 上下文可视化 |
| `/pr_comments` | 查看 PR 评论 |
| `/resume` | 恢复上次会话 |
| `/share` | 分享会话 |
| `/desktop` | 桌面端切换 |
| `/mobile` | 移动端切换 |

### 3. 服务层 (`src/services/`)

| 服务 | 描述 |
|---|---|
| `api/` | Anthropic API 客户端、文件 API、启动引导 |
| `mcp/` | Model Context Protocol 服务器连接与管理 |
| `oauth/` | OAuth 2.0 认证流程 |
| `lsp/` | 语言服务器协议管理器 |
| `analytics/` | 基于 GrowthBook 的功能开关与分析 |
| `plugins/` | 插件加载器 |
| `compact/` | 会话上下文压缩 |
| `policyLimits/` | 组织策略限制 |
| `remoteManagedSettings/` | 远程托管设置 |
| `extractMemories/` | 自动记忆提取 |
| `tokenEstimation.ts` | Token 数量估算 |
| `teamMemorySync/` | 团队记忆同步 |

### 4. 桥接系统 (`src/bridge/`)

连接 IDE 扩展（VS Code、JetBrains）与 Claude Code CLI 的双向通信层。

- `bridgeMain.ts` — 桥接主循环
- `bridgeMessaging.ts` — 消息协议
- `bridgePermissionCallbacks.ts` — 权限回调
- `replBridge.ts` — REPL 会话桥接
- `jwtUtils.ts` — 基于 JWT 的认证
- `sessionRunner.ts` — 会话执行管理

### 5. 权限系统 (`src/hooks/toolPermission/`)

在每次工具调用时检查权限。根据配置的权限模式（`default`、`plan`、`bypassPermissions`、`auto` 等）自动解析或提示用户批准/拒绝。

### 6. 功能开关

通过 Bun 的 `bun:bundle` 功能开关实现死代码消除：

```typescript
import { feature } from 'bun:bundle'

// 未激活的代码在构建时会被完全移除
const voiceCommand = feature('VOICE_MODE')
  ? require('./commands/voice/index.js').default
  : null
```

主要开关：`PROACTIVE`、`KAIROS`、`BRIDGE_MODE`、`DAEMON`、`VOICE_MODE`、`AGENT_TRIGGERS`、`MONITOR_TOOL`

---

## 核心文件详解

### `QueryEngine.ts`（约 46K 行）

LLM API 调用的核心引擎。处理流式响应、工具调用循环、思考模式、重试逻辑和 Token 计数。

### `Tool.ts`（约 29K 行）

定义所有工具的基础类型和接口 — 输入 Schema、权限模型和进度状态类型。

### `commands.ts`（约 25K 行）

管理所有斜杠命令的注册和执行。使用条件导入按环境加载不同的命令集。

### `main.tsx`

基于 Commander.js 的 CLI 解析器和 React/Ink 渲染器初始化。启动时并行执行 MDM 设置、Keychain 预读取和 GrowthBook 初始化以加快启动速度。

---

## 技术栈

| 类别 | 技术 |
|---|---|
| 运行时 | [Bun](https://bun.sh) |
| 编程语言 | TypeScript（严格模式） |
| 终端 UI | [React](https://react.dev) + [Ink](https://github.com/vadimdemedes/ink) |
| CLI 解析 | [Commander.js](https://github.com/tj/commander.js)（extra-typings） |
| Schema 校验 | [Zod v4](https://zod.dev) |
| 代码搜索 | [ripgrep](https://github.com/BurntSushi/ripgrep) |
| 协议 | [MCP SDK](https://modelcontextprotocol.io)、LSP |
| API | [Anthropic SDK](https://docs.anthropic.com) |
| 遥测 | OpenTelemetry + gRPC |
| 功能开关 | GrowthBook |
| 认证 | OAuth 2.0、JWT、macOS Keychain |

---

## 设计亮点

### 并行预读取

通过在加载重量级模块之前并行预读取 MDM 设置、Keychain 和 API 预连接来优化启动时间。

```typescript
// main.tsx — 在其他导入之前作为副作用触发
startMdmRawRead()
startKeychainPrefetch()
```

### 延迟加载

重量级模块（OpenTelemetry、gRPC、分析工具及部分功能开关子系统）通过动态 `import()` 延迟到实际需要时再加载。

### Agent 集群

子 Agent 通过 `AgentTool` 生成，`coordinator/` 负责多 Agent 编排。`TeamCreateTool` 支持团队级别的并行工作。

### 技能系统

定义在 `skills/` 中的可复用工作流通过 `SkillTool` 执行。用户可添加自定义技能。

### 插件架构

内置和第三方插件通过 `plugins/` 子系统加载。

---

## 研究声明 / 所有权免责

- 本仓库是一份**教育与防御性安全研究档案**，由一名大学生维护。
- 旨在研究源代码暴露、打包发布失误以及现代 AI Agent CLI 系统的架构。
- Claude Code 原始源代码的所有权属于 **Anthropic**。
- 本仓库**与 Anthropic 无关，未获得其认可或维护**。
