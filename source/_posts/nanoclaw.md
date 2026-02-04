---
title: NanoClaw 代码深度分析
date: 2026-02-04 10:00:00
tags:
  - nanoclaw
  - code-analysis
  - typescript
categories:
  - 技术分享
description: 深入分析 NanoClaw 项目架构、核心模块设计、安全机制与最佳实践
---

# NanoClaw 代码深度分析

## 1. 项目整体架构和概述

### 1.1 项目简介

**NanoClaw** 是一个轻量级、安全的个人 Claude AI 助手，通过 WhatsApp 提供访问接口。它是一个极简主义的替代方案，与 OpenClaw 相比，专注于以下核心特性：

- **单进程架构**：一个 Node.js 进程处理所有功能
- **容器隔离**：AI 代理在 Apple Container（或 Docker）中运行，提供真正的操作系统级隔离
- **简洁易懂**：代码库足够小，可以在短时间内完全理解
- **AI 原生设计**：通过 Claude Code 进行设置和调试，无需复杂的配置界面

### 1.2 架构概览

```
┌─────────────────────────────────────────────────────────────────────┐
│                        HOST (macOS/Linux)                            │
│                   (Main Node.js Process)                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌──────────────┐                     ┌────────────────────┐        │
│  │  WhatsApp    │────────────────────▶│   SQLite Database  │        │
│  │  (baileys)   │◀────────────────────│   (messages.db)    │        │
│  └──────────────┘   store/send        └─────────┬──────────┘        │
│                                                  │                   │
│         ┌────────────────────────────────────────┘                   │
│         │                                                            │
│         ▼                                                            │
│  ┌──────────────────┐    ┌──────────────────┐    ┌───────────────┐  │
│  │  Message Loop    │    │  Scheduler Loop  │    │  IPC Watcher  │  │
│  │  (polls SQLite)  │    │  (checks tasks)  │    │  (file-based) │  │
│  └────────┬─────────┘    └────────┬─────────┘    └───────────────┘  │
│           │                       │                                  │
│           └───────────┬───────────┘                                  │
│                       │ spawns container                             │
│                       ▼                                              │
├─────────────────────────────────────────────────────────────────────┤
│                  CONTAINER (Apple Container/Docker)                  │
│                     (Isolated Linux VM)                              │
├─────────────────────────────────────────────────────────────────────┤
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │                    AGENT RUNNER                               │   │
│  │              (Claude Agent SDK)                               │   │
│  │                                                                │   │
│  │  Working directory: /workspace/group                           │   │
│  │  Tools: Bash, Read, Write, Edit, WebSearch, agent-browser     │   │
│  │  MCP: nanoclaw (scheduler, messaging)                         │   │
│  └──────────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────────┘
```

### 1.3 目录结构

```
nanoclaw/
├── src/                          # 主程序源代码（9个TypeScript文件）
│   ├── index.ts                  # 主应用：WhatsApp连接、消息路由、IPC
│   ├── config.ts                 # 配置常量和路径
│   ├── types.ts                  # TypeScript接口和类型定义
│   ├── db.ts                     # SQLite数据库操作
│   ├── container-runner.ts       # 生成代理容器
│   ├── task-scheduler.ts         # 定时任务调度
│   ├── mount-security.ts         # 容器挂载安全验证
│   ├── logger.ts                 # Pino日志配置
│   ├── utils.ts                  # JSON加载/保存工具
│   └── whatsapp-auth.ts          # WhatsApp认证工具
│
├── container/                    # 容器配置
│   ├── Dockerfile                # 代理容器镜像定义
│   ├── build.sh                  # 容器构建脚本
│   ├── agent-runner/             # 在容器内运行的代码
│   │   ├── src/
│   │   │   ├── index.ts          # 容器入口点
│   │   │   └── ipc-mcp.ts        # 主机通信MCP服务器
│   │   ├── package.json
│   │   └── tsconfig.json
│   └── skills/
│       └── agent-browser.md      # 浏览器自动化技能
│
├── groups/                       # 按群组隔离的文件和记忆
│   ├── main/                     # 主控制频道（自聊）
│   │   ├── CLAUDE.md             # 主频道记忆和指令
│   │   └── logs/                 # 执行日志
│   └── global/                   # 所有群组可访问的全局记忆
│       └── CLAUDE.md             # 共享上下文和偏好
│
├── .claude/skills/               # Claude Code技能
│   ├── setup/SKILL.md            # 初始安装和设置
│   ├── customize/SKILL.md        # 添加频道和修改行为
│   ├── debug/SKILL.md            # 故障排除和诊断
│   └── x-integration/            # X（Twitter）集成
│
├── docs/                         # 文档
│   ├── SPEC.md                   # 完整技术规范
│   ├── REQUIREMENTS.md           # 架构决策和理念
│   └── SECURITY.md               # 安全模型和信任边界
│
├── store/                        # SQLite数据和WhatsApp认证
├── data/                         # 应用状态（会话、注册组、IPC）
└── launchd/                      # macOS服务配置
```

---

## 2. 核心模块分析

### 2.1 主应用模块 (`src/index.ts`)

**功能职责：**
- WhatsApp Web 连接管理（使用 Baileys 库）
- 消息接收和存储到 SQLite
- 消息路由到已注册群组
- 容器生成和生命周期管理
- 基于文件的 IPC 通信
- 状态管理（会话、时间戳、已注册群组）

**关键函数分析：**

```typescript
// 消息处理流程
async function processMessage(msg: NewMessage): Promise<void>
```
- 仅处理已注册群组的消息
- 主群组响应所有消息；其他群组需要触发词前缀
- 获取自上次代理交互以来的所有消息以提供完整上下文
- 使用 XML 格式构建对话历史提示词

```typescript
// 代理执行流程
async function runAgent(
  group: RegisteredGroup,
  prompt: string,
  chatJid: string,
): Promise<string | null>
```
- 为容器准备任务快照和可用群组快照
- 调用 `runContainerAgent` 在隔离容器中执行 Claude
- 处理会话 ID 的保存和恢复

```typescript
// IPC 监控流程
function startIpcWatcher(): void
```
- 扫描每个群组的 IPC 目录
- 处理消息发送请求（带授权验证）
- 处理任务管理操作（schedule_task、pause_task、resume_task、cancel_task）
- 验证群组身份以防止跨群组权限提升

### 2.2 容器运行器 (`src/container-runner.ts`)

**功能职责：**
- 为每个群组构建卷挂载配置
- 使用 Apple Container 生成隔离的代理执行环境
- 通过 JSON over stdin/stdout 处理容器输入/输出
- 管理每个群组的会话目录
- 写入任务和群组快照供容器读取

**安全特性：**

```typescript
// 卷挂载构建（行 57-163）
function buildVolumeMounts(group: RegisteredGroup, isMain: boolean): VolumeMount[]
```

| 挂载路径 | 主群组 | 其他群组 | 用途 |
|---------|--------|----------|------|
| `/workspace/project` | 读写 | 无 | 项目根目录访问 |
| `/workspace/group` | 读写 | 读写 | 群组文件夹 |
| `/workspace/global` | 隐式 | 只读 | 全局记忆 |
| `/home/node/.claude` | 读写 | 读写 | 会话隔离 |
| `/workspace/ipc` | 读写 | 读写 | IPC命名空间隔离 |
| `/workspace/env-dir` | 只读 | 只读 | 过滤后的环境变量 |
| `/workspace/extra/*` | 可配置 | 可配置（只读） | 额外挂载 |

**关键安全设计：**
- **IPC 命名空间隔离**：每个群组有自己的 IPC 目录，防止跨群组权限提升
- **凭证过滤**：仅从 `.env` 中提取 `CLAUDE_CODE_OAUTH_TOKEN` 和 `ANTHROPIC_API_KEY`
- **外部允许列表**：额外挂载通过 `~/.config/nanoclaw/mount-allowlist.json` 验证

### 2.3 数据库模块 (`src/db.ts`)

**数据库架构：**

```sql
-- 聊天表：聊天元数据
CREATE TABLE chats (
  jid TEXT PRIMARY KEY,           -- WhatsApp JID
  name TEXT,                      -- 群组/联系人名称
  last_message_time TEXT          -- 最后活动时间
);

-- 消息表：完整消息历史
CREATE TABLE messages (
  id TEXT,
  chat_jid TEXT,
  sender TEXT,
  sender_name TEXT,
  content TEXT,
  timestamp TEXT,
  is_from_me INTEGER,
  PRIMARY KEY (id, chat_jid)
);

-- 定时任务表
CREATE TABLE scheduled_tasks (
  id TEXT PRIMARY KEY,
  group_folder TEXT NOT NULL,
  chat_jid TEXT NOT NULL,
  prompt TEXT NOT NULL,
  schedule_type TEXT NOT NULL,    -- 'cron' | 'interval' | 'once'
  schedule_value TEXT NOT NULL,
  next_run TEXT,
  last_run TEXT,
  last_result TEXT,
  status TEXT DEFAULT 'active',
  created_at TEXT NOT NULL
);

-- 任务运行日志表
CREATE TABLE task_run_logs (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  task_id TEXT NOT NULL,
  run_at TEXT NOT NULL,
  duration_ms INTEGER NOT NULL,
  status TEXT NOT NULL,
  result TEXT,
  error TEXT
);
```

**关键设计决策：**
- 使用 `better-sqlite3` 进行同步 SQLite 操作（比异步更简单、更快）
- 消息内容仅对注册群组存储（隐私保护）
- 所有聊天元数据存储以实现群组发现
- 数据库迁移通过 try-catch 模式处理（行 65-79）

### 2.4 挂载安全模块 (`src/mount-security.ts`)

**安全功能：**

```typescript
// 默认阻止的模式
const DEFAULT_BLOCKED_PATTERNS = [
  '.ssh', '.gnupg', '.gpg', '.aws', '.azure', '.gcloud',
  '.kube', '.docker', 'credentials', '.env', '.netrc',
  '.npmrc', '.pypirc', 'id_rsa', 'id_ed25519',
  'private_key', '.secret',
];
```

**验证流程：**

1. **路径扩展**：将 `~` 扩展为家目录
2. **符号链接解析**：使用 `fs.realpathSync` 防止遍历攻击
3. **阻止模式检查**：路径组件匹配阻止列表
4. **允许根检查**：验证路径是否在允许的根目录下
5. **只读强制执行**：非主群组强制只读

**允许列表配置示例：**

```json
{
  "allowedRoots": [
    { "path": "~/projects", "allowReadWrite": true },
    { "path": "~/Documents/work", "allowReadWrite": false }
  ],
  "blockedPatterns": ["password", "secret", "token"],
  "nonMainReadOnly": true
}
```

### 2.5 定时任务调度器 (`src/task-scheduler.ts`)

**功能特性：**
- 每 60 秒轮询检查到期任务
- 支持三种调度类型：cron 表达式、间隔（毫秒）、一次性
- 任务在容器上下文中执行，具有完整代理能力
- 支持两种上下文模式：
  - `group`：使用群组的当前会话（有对话历史）
  - `isolated`：新会话（独立任务）

### 2.6 容器内代理运行器 (`container/agent-runner/src/index.ts`)

**功能职责：**
- 通过 stdin 接收配置 JSON
- 使用 Claude Agent SDK 执行代理
- 处理会话恢复和归档
- 在压缩前归档对话
- 通过 stdout 返回结果（带标记）

**会话归档功能：**

```typescript
// 在压缩前自动归档对话（行 87-127）
function createPreCompactHook(): HookCallback
```

- 解析转录文件提取消息
- 生成带日期和摘要的文件名
- 保存到 `conversations/` 目录
- 保留对话历史供将来参考

### 2.7 IPC MCP 服务器 (`container/agent-runner/src/ipc-mcp.ts`)

**可用工具：**

| 工具 | 描述 | 权限 |
|------|------|------|
| `send_message` | 发送 WhatsApp 消息 | 所有群组 |
| `schedule_task` | 创建定时任务 | 主群组可为任何群组创建；其他仅为自己 |
| `list_tasks` | 列出定时任务 | 主群组查看所有；其他仅查看自己的 |
| `pause_task` | 暂停任务 | 仅自己的任务 |
| `resume_task` | 恢复任务 | 仅自己的任务 |
| `cancel_task` | 取消任务 | 仅自己的任务 |
| `register_group` | 注册新群组 | 仅主群组 |

**IPC 机制：**
- 通过文件系统写入 JSON 文件
- 原子写入：临时文件后重命名
- 主机进程轮询处理

---

## 3. 关键代码实现细节

### 3.1 消息轮询和处理流程

```typescript
// src/index.ts:747-777
async function startMessageLoop(): Promise<void> {
  while (true) {
    try {
      const jids = Object.keys(registeredGroups);
      const { messages } = getNewMessages(jids, lastTimestamp, ASSISTANT_NAME);

      for (const msg of messages) {
        try {
          await processMessage(msg);
          // 仅在成功处理后推进时间戳 - 至少一次交付保证
          lastTimestamp = msg.timestamp;
          saveState();
        } catch (err) {
          // 停止处理此批次 - 失败的消息将在下次循环重试
          break;
        }
      }
    } catch (err) {
      logger.error({ err }, 'Error in message loop');
    }
    await new Promise((resolve) => setTimeout(resolve, POLL_INTERVAL));
  }
}
```

**关键设计：**
- **至少一次交付**：仅在成功处理后推进时间戳
- **错误隔离**：一条消息失败不会阻止其他消息
- **批量处理**：每次迭代处理所有待处理消息

### 3.2 LID 到电话号码映射

```typescript
// src/index.ts:54-69
let lidToPhoneMap: Record<string, string> = {};

function translateJid(jid: string): string {
  if (!jid.endsWith('@lid')) return jid;
  const lidUser = jid.split('@')[0].split(':')[0];
  const phoneJid = lidToPhoneMap[lidUser];
  if (phoneJid) {
    return phoneJid;
  }
  return jid;
}
```

WhatsApp 现在为自聊发送 LID JID，此映射确保正确处理。

### 3.3 容器输出解析

```typescript
// src/container-runner.ts:368-384
const startIdx = stdout.indexOf(OUTPUT_START_MARKER);
const endIdx = stdout.indexOf(OUTPUT_END_MARKER);

let jsonLine: string;
if (startIdx !== -1 && endIdx !== -1 && endIdx > startIdx) {
  jsonLine = stdout
    .slice(startIdx + OUTPUT_START_MARKER.length, endIdx)
    .trim();
} else {
  // 回退：最后一行非空行（向后兼容）
  const lines = stdout.trim().split('\n');
  jsonLine = lines[lines.length - 1];
}

const output: ContainerOutput = JSON.parse(jsonLine);
```

使用标记器进行稳健的 JSON 解析，处理代理可能产生的额外输出。

### 3.4 IPC 授权验证

```typescript
// src/index.ts:326-347
// 授权：验证此群组是否可以发送到此 chatJid
const targetGroup = registeredGroups[data.chatJid];
if (
  isMain ||
  (targetGroup && targetGroup.folder === sourceGroup)
) {
  await sendMessage(data.chatJid, `${ASSISTANT_NAME}: ${data.text}`);
} else {
  logger.warn(
    { chatJid: data.chatJid, sourceGroup },
    'Unauthorized IPC message attempt blocked',
  );
}
```

关键安全控制：仅允许主群组或消息发送者发送到其自己的聊天。

---

## 4. 代码设计亮点和最佳实践

### 4.1 安全设计模式

| 模式 | 实现 | 优点 |
|------|------|------|
| **纵深防御** | 容器隔离 + 挂载验证 + IPC 授权 | 多层保护 |
| **最小权限** | 非主群组只读挂载 | 限制潜在损害 |
| **外部配置** | 允许列表在项目根之外 | 代理无法修改安全策略 |
| **身份验证** | IPC 目录路径决定群组身份 | 无法伪造 |

### 4.2 错误处理模式

```typescript
// 优雅降级（src/utils.ts:4-13）
export function loadJson<T>(filePath: string, defaultValue: T): T {
  try {
    if (fs.existsSync(filePath)) {
      return JSON.parse(fs.readFileSync(filePath, 'utf-8'));
    }
  } catch {
    // 出错返回默认值
  }
  return defaultValue;
}

// 数据库迁移（src/db.ts:65-79）
try {
  db.exec(`ALTER TABLE messages ADD COLUMN sender_name TEXT`);
} catch {
  /* 列已存在 */
}
```

### 4.3 日志记录最佳实践

```typescript
// 结构化日志与上下文（使用 Pino）
logger.info(
  { group: group.name, messageCount: missedMessages.length },
  'Processing message',
);

logger.error(
  { group: group.name, error: output.error },
  'Container agent error',
);
```

### 4.4 TypeScript 类型安全

```typescript
// 全面的类型定义（src/types.ts）
export interface ScheduledTask {
  id: string;
  group_folder: string;
  chat_jid: string;
  prompt: string;
  schedule_type: 'cron' | 'interval' | 'once';
  schedule_value: string;
  context_mode: 'group' | 'isolated';
  next_run: string | null;
  last_run: string | null;
  last_result: string | null;
  status: 'active' | 'paused' | 'completed';
  created_at: string;
}
```

### 4.5 代码简洁性

整个代码库遵循极简主义：
- 无过度工程化
- 无不必要的抽象
- 依赖清晰的代码而非大量注释
- 同步 SQLite 操作简化逻辑
- 基于文件的 IPC 避免消息队列复杂性

---

## 5. 技术栈和依赖分析

### 5.1 核心技术栈

| 组件 | 技术 | 版本 | 用途 |
|------|------|------|------|
| 运行时 | Node.js | 20+ | 主机进程执行环境 |
| 语言 | TypeScript | 5.7 | 类型安全的源代码 |
| WhatsApp | @whiskeysockets/baileys | 7.0.0-rc.9 | WhatsApp Web 连接 |
| 数据库 | better-sqlite3 | 11.8.1 | 同步 SQLite 操作 |
| 容器 | Apple Container / Docker | - | 代理隔离 |
| 代理 SDK | @anthropic-ai/claude-agent-sdk | 0.2.29 | Claude 代理执行 |
| 浏览器 | agent-browser + Chromium | - | 浏览器自动化 |
| 日志 | pino + pino-pretty | 9.6.0 | 结构化日志 |
| 任务调度 | cron-parser | 5.5.0 | Cron 表达式解析 |
| 验证 | zod | 4.3.6 | 模式验证 |

### 5.2 依赖分析

**生产依赖（8个）：**
- `@whiskeysockets/baileys`：WhatsApp Web 协议实现
- `better-sqlite3`：高性能同步 SQLite
- `cron-parser`：Cron 表达式解析
- `pino`/`pino-pretty`：快速结构化日志
- `qrcode-terminal`：终端 QR 码显示
- `zod`：运行时类型验证

**开发依赖（6个）：**
- `typescript`/`tsx`：TypeScript 编译和运行
- `prettier`：代码格式化
- `@types/*`：类型定义

**设计原则：**
- 最小依赖集
- 无框架（Express、Nest 等）
- 无 ORM（原始 SQL）
- 无消息队列（文件系统 IPC）

---

## 6. 改进建议

### 6.1 高优先级

#### 1. 凭证隔离增强
**当前问题：** 代理可以通过 Bash 或文件操作发现 Anthropic 凭证。

**建议：** 研究使用内核密钥环或专用认证代理，在容器外处理认证。

```typescript
// 当前（src/container-runner.ts:127-150）
const allowedVars = ['CLAUDE_CODE_OAUTH_TOKEN', 'ANTHROPIC_API_KEY'];
// 仅提取允许的变量，但仍对容器可见
```

#### 2. 消息重试机制
**当前问题：** 消息处理失败后重试，但无指数退避或最大重试限制。

**建议：** 实现带退避的重试计数器，防止无限循环。

```typescript
// 建议添加
interface MessageRetryState {
  messageId: string;
  retryCount: number;
  lastRetry: string;
}
```

#### 3. 健康检查端点
**当前问题：** 无运行状况监控方式。

**建议：** 添加简单的 HTTP 健康检查或状态文件写入。

### 6.2 中优先级

#### 4. 消息速率限制
**建议：** 为传入和传出消息实现速率限制，防止滥用。

```typescript
// 建议
interface RateLimitState {
  jid: string;
  messageCount: number;
  windowStart: string;
}
```

#### 5. 增强的日志轮转
**当前问题：** 容器日志无限增长（行 285-345）。

**建议：** 实施日志轮转和保留策略。

#### 6. 数据库连接池
**当前问题：** 每个查询使用单一数据库连接。

**建议：** 对于高吞吐量，考虑连接池。

### 6.3 低优先级

#### 7. 指标和监控
**建议：** 添加 Prometheus 指标或类似指标用于监控。

#### 8. 配置验证
**建议：** 在启动时验证所有配置，并明确错误。

```typescript
// 建议
function validateConfig(): ConfigValidationResult {
  // 验证路径、权限、容器可用性
}
```

#### 9. 测试覆盖
**建议：** 添加单元测试和集成测试。

#### 10. 文档改进
**建议：**
- API 文档（TypeDoc）
- 架构决策记录（ADR）
- 故障排除指南

### 6.4 安全加固建议

| 建议 | 优先级 | 实现复杂度 |
|------|--------|-----------|
| 凭证隔离增强 | 高 | 高 |
| 输入消毒 | 中 | 中 |
| 审计日志 | 中 | 低 |
| 消息签名验证 | 低 | 高 |
| 运行时安全扫描 | 低 | 中 |

---

## 7. 总结

NanoClaw 是一个设计精良、安全优先的个人 AI 助手。其主要优势：

1. **安全架构**：真正的容器隔离，而非应用级权限
2. **简洁性**：代码库足够小，可以完全理解
3. **实用主义**：没有不必要的抽象，专注于实际功能
4. **AI 原生**：设计为与 Claude Code 一起使用

代码展示了良好的软件工程实践：
- 清晰的模块边界
- 全面的类型安全
- 深思熟虑的错误处理
- 安全优先的设计
- 极简依赖策略

该项目作为 AI 驱动个人助手的参考实现，平衡了功能、安全性和可维护性。

---

*报告生成时间：2026-02-04*
*分析范围：完整代码库（~2,500 行 TypeScript）*
