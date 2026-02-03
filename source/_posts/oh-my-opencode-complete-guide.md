---
title: Oh-My-OpenCode 完全指南：多代理协作编程新范式
date: 2026-02-03 18:00:00
tags:
  - Oh-My-OpenCode
  - OpenCode
  - AI 编程
  - 多代理协作
  - 开发工具
categories:
  - 技术分享
description: 深入解析 Oh-My-OpenCode 3.2.1 的核心功能和使用方法，包括 11 个专化代理、Ultrawork 全自动模式、Prometheus 精密规划模式，帮助你构建高效的 AI 编程团队。
---

## 前言

如果说 Claude Code 是单个 AI 编程助手的巅峰之作，那么 **Oh-My-OpenCode（OMO）** 就是将 AI 编程推向全新维度的革命性插件。它将单个 AI 代理升级为**多代理协作团队**，让 11 个专业代理并行工作，像一支训练有素的开发团队一样协作编码。

本文基于 **OMO v3.2.1** 版本（最新版，包含 Hephaestus 代理和多项性能优化），从零基础开始，带你全面了解这个强大的多代理编程框架。

---

## 一、什么是 Oh-My-OpenCode？

### 核心定位

**Oh-My-OpenCode** 是 [OpenCode](https://opencode.ai/) 的顶级插件。OpenCode 本身是一个开源 AI 编码代理（类似 Claude Code / Cursor 的开源替代），而 OMO 在其基础上添加了**编排层**，让多个专业代理能够像"小团队"一样协作完成任务。

### 核心理念对比

| 维度 | 传统 AI 编码助手 | Oh-My-OpenCode |
|------|-----------------|----------------|
| **工作模式** | 单代理串行处理 | 多代理并行协作 |
| **任务分配** | 所有工作一个代理做 | 专业代理各司其职 |
| **规划能力** | 边做边想 | 先规划后执行 |
| **执行效率** | 线性处理 | 多线程并行 |
| **适用场景** | 简单到中等复杂度 | 简单到超复杂项目 |

### 为什么需要多代理？

想象一个真实的开发团队：
- **架构师**负责设计整体方案
- **研究员**查找最佳实践和开源实现
- **开发工程师**编写核心代码
- **代码审查员**检查质量和安全性
- **测试工程师**验证功能正确性

OMO 就是为 AI 编码复制了这种**专业化分工**模式。每个代理专注自己擅长的领域，通过协调者统一调度，整体效率远超单代理。

---

## 二、两大核心工作模式

OMO 提供两种截然不同的工作模式，适应不同场景需求：

### 2.1 Ultrawork 全自动模式（ulw）

**关键词**: `ulw` 或 `ultrawork`

这是最简单的使用方式——**脑放空，全自动**。你只需要描述目标，代理团队会自主完成所有工作。

```bash
ulw 在我的 Next.js 项目中添加用户认证功能
```

OMO 会自动：
1. **探索代码库** - Explore 代理分析项目结构
2. **研究最佳实践** - Librarian 代理查找相关文档和示例
3. **设计架构** - Oracle 代理审查设计方案
4. **实施代码** - Hephaestus 代理编写高质量代码
5. **测试验证** - 自动运行测试并修复问题

**适用场景**:
- ✅ 快速原型开发
- ✅ 修复已知 Bug
- ✅ 添加标准功能（如认证、CRUD）
- ✅ 代码重构和优化

**不适合**:
- ❌ 需要深度架构设计的复杂系统
- ❌ 跨多会话的长期项目

### 2.2 Prometheus + Atlas 精密规划模式

**进入方式**: 按 `Tab` 键进入 Prometheus 模式

这是 OMO 的**精密规划模式**，适合复杂/多会话任务。

**工作流程**:

1. **按 Tab 进入 Prometheus 模式**
2. **描述你的任务** - 例如"重构用户模块，将单体架构改为微服务"
3. **Prometheus 提问澄清** - 它会问细节问题，确保理解需求
4. **审阅生成的计划** - 计划保存在 `.sisyphus/plans/*.md`
5. **输入 `/start-work` 启动执行** - Atlas 代理按规划执行

**核心优势**:
- 复杂任务先规划，避免返工
- 计划文件可保存，支持跨会话继续
- 多步骤任务有清晰的执行路径

**适用场景**:
- ✅ 大型重构项目
- ✅ 多文件改动的新功能
- ✅ 需要多轮迭代的复杂任务
- ✅ 跨会话的长期项目

---

## 三、11 个专化代理详解

OMO 的核心竞争力在于其**专业化代理团队**。每个代理都有明确的职责和推荐的 AI 模型：

### 核心编排代理

| 代理名 | 推荐模型 | 核心职责 |
|--------|----------|----------|
| **Sisyphus** | Claude Opus 4.5 | **主编排者**，Todo 驱动，全局协调并行执行 |
| **Hephaestus** | GPT-5.2 Codex | **深度工作者**，目标导向，先探索后行动，精炼代码 |

### 专业审查代理

| 代理名 | 推荐模型 | 核心职责 |
|--------|----------|----------|
| **Oracle** | GPT-5.2 | **架构师**，负责设计、代码审阅、调试（只读，不修改代码） |
| **Momus** | GPT-5.2 | **计划审阅者**，确保计划清晰、可验证 |

### 研究探索代理

| 代理名 | 推荐模型 | 核心职责 |
|--------|----------|----------|
| **Librarian** | GLM-4.7 | **研究员**，多仓库分析、文档检索、开源实现示例查找 |
| **Explore** | Claude Haiku 4.5 | **探索者**，快速代码库探索、模式匹配 |
| **Metis** | Claude Opus 4.5 | **分析者**，计划前分析，识别隐藏意图和风险 |

### 规划与多模态代理

| 代理名 | 推荐模型 | 核心职责 |
|--------|----------|----------|
| **Prometheus** | Claude Opus 4.5 | **规划者**，通过访谈生成详细工作计划 |
| **Multimodal-looker** | Gemini-3-flash | **视觉分析师**，分析图片、PDF、设计图 |

### 手动调用代理示例

```bash
# 架构审查
@oracle 审阅这个微服务架构设计

# 查找开源实现
@librarian 用户权限管理在开源项目中是怎么实现的？

# 探索代码库
@explore 搜索项目中所有 TODO 和 FIXME
```

### 模型自动回退机制

OMO 智能的模型选择策略：
- **原生订阅** → 使用官方 API（最优质量）
- **Copilot 订阅** → 使用 GitHub Copilot 内置模型
- **Zen / Z.ai** → 使用第三方代理服务

---

## 四、安装与配置

### 4.1 前置要求

- **OpenCode** ≥ 1.0.150
- **Bun** 或 **Node.js** ≥ 22.x

### 4.2 安装步骤

**步骤 1：安装 OpenCode**

```bash
curl -fsSL https://opencode.ai/install | bash

# 验证版本
opencode --version  # 需 ≥ 1.0.150
```

**步骤 2：安装 Oh-My-OpenCode**

推荐方式（互动式安装）：

```bash
# 使用 Bun（推荐）
bunx oh-my-opencode install

# 或使用 npx
npx oh-my-opencode install
```

安装程序会询问你的订阅情况（Claude Pro/Max、ChatGPT Plus、Gemini、GitHub Copilot 等），自动生成最佳配置。

**步骤 3：认证模型提供商**

```bash
opencode auth login
```

按提示选择：
- **Anthropic（Claude）** → Claude Pro/Max OAuth
- **Google（Gemini）** → Antigravity OAuth（支持多账号负载均衡）
- **OpenAI / GitHub Copilot** → 对应认证流程

**步骤 4：验证安装**

```bash
# 检查配置
cat ~/.config/opencode/opencode.json  # 应包含 "oh-my-opencode"

# 查看可用模型
opencode models
```

### 4.3 卸载

```bash
# 编辑配置移除插件
# ~/.config/opencode/opencode.json

# 删除配置文件
rm -f ~/.config/opencode/oh-my-opencode.json
rm -f .opencode/oh-my-opencode.json
```

---

## 五、配置自定义（进阶）

### 5.1 配置文件位置

```bash
~/.config/opencode/oh-my-opencode.json  # 支持 JSONC 注释
```

### 5.2 常用自定义示例

**代理模型自定义**:

```json
{
  "agents": {
    "oracle": { "model": "openai/gpt-5.2" },
    "explore": { 
      "model": "anthropic/claude-haiku-4-5",
      "temperature": 0.3
    },
    "multimodal-looker": { "disable": true }
  }
}
```

**类别（Categories）配置**:

用于 `delegate_task` 时指定领域模型：

```json
{
  "categories": {
    "visual-engineering": { 
      "model": "google/gemini-3-pro-preview" 
    },
    "ultrabrain": { 
      "model": "openai/gpt-5.2-codex",
      "variant": "xhigh"
    }
  }
}
```

视觉任务会自动使用 Gemini，深度编码任务使用 GPT-5.2 Codex。

**后台任务并发配置**:

```json
{
  "background_task": {
    "defaultConcurrency": 5,
    "providerConcurrency": { "anthropic": 3 }
  }
}
```

**启用 tmux 可视化**:

```json
{
  "tmux": { "enabled": true }
}
```

在 tmux 中可以看到并行代理的执行状态，非常直观。

---

## 六、高级功能

### 6.1 钩子（Hooks）系统

OMO 内置 25+ 钩子，可以精细控制代理行为：

```json
{
  "disabled_hooks": ["comment-checker"]
}
```

**重要钩子说明**：

| 钩子名 | 作用 |
|--------|------|
| `todo-continuation-enforcer` | 强制完成 TODO，不允许遗漏 |
| `ralph-loop` | 防止无限循环，检测重复模式 |
| `context-window-monitor` | 上下文窗口管理，防止超出限制 |

### 6.2 技能（Skills）系统

自定义技能支持浏览器自动化（Playwright 或 agent-browser）：

```markdown
# skill: web-analyzer

使用 Playwright 分析网页性能

## 触发条件
当用户要求分析网页加载性能时

## 执行步骤
1. 使用 Playwright 打开目标网页
2. 收集 Performance API 数据
3. 分析关键指标（FCP, LCP, TTI）
4. 生成优化建议报告
```

### 6.3 MCP（Model Context Protocol）

内置 MCP 服务器：
- **websearch** - 网页搜索
- **context7** - 代码库语义搜索
- **grep_app** - 代码片段查找

可以禁用不需要的 MCP：

```json
{
  "disabled_mcp": ["websearch"]
}
```

### 6.4 LSP 支持

添加语言服务器获得更智能的代码分析：

```json
{
  "lsp": {
    "typescript-language-server": {
      "command": ["typescript-language-server", "--stdio"]
    },
    "rust-analyzer": {
      "command": ["rust-analyzer"]
    }
  }
}
```

---

## 七、最佳实践与技巧

### 7.1 任务模式选择指南

```
小任务（< 10 分钟）→ 直接用 ulw
│
大任务（> 30 分钟）→ 必用 Prometheus 规划模式
│
多会话项目 → 计划文件自动保存，/start-work 继续
│
紧急修复 → ulw + 明确目标
│
架构重构 → Prometheus + Oracle 审阅
```

### 7.2 提示词技巧

**高效提示公式**:

```
[上下文] + [具体目标] + [约束条件] + [ulw 可选]

示例：
"在这个 Express.js 项目中 [上下文]，
添加 JWT 认证中间件 [目标]，
使用 passport-jwt 库，不要改动现有路由 [约束]，
ulw [全自动模式]"
```

### 7.3 模型选择建议

| 任务类型 | 推荐模型 | 原因 |
|----------|----------|------|
| 快速查询 | Claude Haiku 4.5 | 最快最便宜 |
| 日常开发 | Claude Sonnet 4.5 | 性价比平衡 |
| 架构设计 | Claude Opus 4.5 | 最高质量 |
| 深度编码 | GPT-5.2 Codex | 代码生成最强 |
| 视觉分析 | Gemini-3 Pro | 多模态能力 |

### 7.4 性能优化

1. **限制并发数** - 根据 API 配额调整 `defaultConcurrency`
2. **禁用不常用代理** - 如不用图片分析可禁用 `multimodal-looker`
3. **使用本地模型** - 简单查询可用 Ollama 本地模型
4. **合理配置钩子** - 只启用必要的钩子减少开销

---

## 八、故障排除

### 常见问题解决

| 问题 | 解决方案 |
|------|----------|
| 配置不生效 | 检查 OpenCode 版本（>1.0.132），删除旧配置重装 |
| 模型不可用 | 运行 `opencode models` 检查，重新 `auth login` |
| 并发问题 | 查看后台任务日志，降低 `defaultConcurrency` |
| 卡死/无响应 | 检查 tmux 状态，查看后台任务是否超时 |
| 代理不执行 | 确认代理未被禁用，检查模型配置是否正确 |

### 调试技巧

```bash
# 查看详细日志
opencode --verbose

# 检查代理状态
cat .sisyphus/state.json

# 查看计划文件
ls -la .sisyphus/plans/

# 手动测试代理
@oracle 分析当前项目架构
```

---

## 九、版本更新

OMO 会自动检查更新，也可手动更新：

```bash
bunx oh-my-opencode install
```

查看最新版本：[GitHub Releases](https://github.com/code-yeongyu/oh-my-opencode/releases)

### v3.2.1 新特性

- ✅ 修复后台代理并发槽泄漏问题
- ✅ 支持 GitHub Copilot Gemini 模型预览
- ✅ Hephaestus 代理已稳定（v3.2.0 引入）

---

## 十、与其他工具对比

| 工具 | 工作模式 | 优势 | 劣势 |
|------|----------|------|------|
| **OMO** | 多代理协作 | 专业化分工，并行高效 | 配置较复杂 |
| **Claude Code** | 单代理 | 简单易用，开箱即用 | 复杂任务效率较低 |
| **Cursor** | 单代理+IDE | 深度 IDE 集成 | 仅限编辑器内使用 |
| **GitHub Copilot** | 代码补全 | 实时补全，低延迟 | 非完整代理 |

### 选择建议

- **OMO 适合**: 复杂项目、需要多步骤协调、追求极致效率的开发者
- **Claude Code 适合**: 快速原型、简单任务、不想配置的用户
- **Cursor 适合**: 习惯在 IDE 内工作、重视代码补全的开发者

---

## 总结

Oh-My-OpenCode 代表了 AI 编程的**新范式**——从单兵作战到团队协作。11 个专业代理各司其职，Prometheus 负责规划，Sisyphus 负责编排，Hephaestus 负责深度编码，Oracle 负责审查...这种分工模式让 AI 能够处理越来越复杂的软件开发任务。

**核心价值**：
- 🚀 **效率倍增** - 并行代理同时处理不同子任务
- 🎯 **专业化** - 每个代理专注自己擅长的领域
- 📋 **可规划** - 复杂任务先规划后执行，避免返工
- 🔧 **可定制** - 丰富的配置选项，适应不同工作流

**下一步行动**：
1. 安装 OpenCode 和 OMO
2. 从简单的 `ulw` 任务开始体验
3. 逐步尝试 Prometheus 规划模式
4. 根据需求自定义代理和配置

让代理为你编码，享受真正的"Ultrawork"！

---

## 参考资料

1. **Oh-My-OpenCode GitHub**: [github.com/code-yeongyu/oh-my-opencode](https://github.com/code-yeongyu/oh-my-opencode)
2. **OpenCode 官网**: [opencode.ai](https://opencode.ai/)
3. **本文参考的微信文章**: 《Oh-My-OpenCode 3.2.1从新手到专家完整操作手册》by 码农不器

---

*本文基于 OMO v3.2.1 版本整理，如有更新请以官方文档为准。*
