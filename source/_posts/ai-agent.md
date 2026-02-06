---
title: AI Agent 记忆系统深度解析
date: 2026-02-06 15:46:55
tags:
  - agent
  - 记忆系统深度解析
categories:
  - 技术分享
description: 详解 AI Agent 中的短期记忆与长期记忆架构、上下文工程策略及开源记忆系统对比
---

# AI Agent 记忆系统深度解析

> **作者**: 柳遵飞（翼严）  
> **来源**: 阿里云开发者  
> **原文链接**: https://mp.weixin.qq.com/s/mftM6jr0YiFxRATeNvm5Qg

---

## 前言

随着 AI Agent 应用的快速发展，智能体需要处理越来越复杂的任务和更长的对话历史。然而，LLM 的上下文窗口限制、不断增长的 token 成本，以及如何让 AI"记住"用户偏好和历史交互，都成为了构建实用 AI Agent 系统面临的核心挑战。

记忆系统（Memory System）正是为了解决这些问题而诞生的关键技术。记忆系统使 AI Agent 能够像人类一样，在单次对话中保持上下文连贯性（短期记忆），同时能够跨会话记住用户偏好、历史交互和领域知识（长期记忆）。这不仅提升了用户体验的连续性和个性化程度，也为构建更智能、更实用的 AI 应用奠定了基础。

---

## 一、Memory 基础概念

### 1.1 记忆的定义与分类

对于 AI Agent 而言，记忆至关重要，因为它使它们能够记住之前的互动、从反馈中学习，并适应用户的偏好。对"记忆"的定义有两个层面：

**会话级记忆**：用户和智能体 Agent 在一个会话中的多轮交互（user-query & response）

**跨会话记忆**：从用户和智能体 Agent 的多个会话中抽取的通用信息，可以跨会话辅助 Agent 推理

### 1.2 各 Agent 框架的定义差异

各个 Agent 框架对记忆的概念命名各有不同，但共同的是都遵循上一节中介绍的两个不同层面的划分：会话级和跨会话级。

| 框架 | 会话级记忆 | 跨会话级记忆 |
|------|-----------|-------------|
| Google ADK | Session | Memory（长期知识库） |
| LangChain | Short-term memory | Long-term memory（个人知识库外挂） |
| AgentScope | memory | long_term_memory |

习惯上，可以将会话级别的历史消息称为短期记忆，把可以跨会话共享的信息称为长期记忆，但本质上两者并不是通过简单的时间维度进行的划分，从实践层面上以是否跨 Session 会话来进行区分。长期记忆的信息从短期记忆中抽取提炼而来，根据短期记忆中的信息实时地更新迭代，而其信息又会参与到短期记忆中辅助模型进行个性化推理。

---

## 二、Agent 框架集成记忆系统的架构

### 2.1 Agent 框架集成记忆的通用模式

各 Agent 框架集成记忆系统通常遵循以下通用模式：

1. **Step1：推理前加载** - 根据当前 user-query 从长期记忆中加载相关信息
2. **Step2：上下文注入** - 从长期记忆中检索的信息加入当前短期记忆中辅助模型推理
3. **Step3：记忆更新** - 短期记忆在推理完成后加入到长期记忆中
4. **Step4：信息处理** - 长期记忆模块中结合 LLM+向量化模型进行信息提取和检索

### 2.2 短期记忆（Session 会话）

短期记忆存储会话中产生的各类消息，包括用户输入、模型回复、工具调用及其结果等。这些消息直接参与模型推理，实时更新，并受模型的 maxToken 限制。当消息累积导致上下文窗口超出限制时，需要通过上下文工程策略（压缩、卸载、摘要等）进行处理，这也是上下文工程主要处理的部分。

**核心特点**：
- 存储会话中的所有交互消息（用户输入、模型回复、工具调用等）
- 直接参与模型推理，作为 LLM 的输入上下文
- 实时更新，每次交互都会新增消息
- 受模型 maxToken 限制，需要上下文工程策略进行优化

### 2.3 长期记忆（跨会话）

长期记忆与短期记忆形成双向交互：

**Record（写入）**：从短期记忆的会话消息中提取有效信息，通过LLM进行语义理解和抽取，存储到长期记忆中

**Retrieve（检索）**：根据当前用户查询，从长期记忆中检索相关信息，注入到短期记忆中作为上下文，辅助模型推理

**常见组件**：Mem0、Zep、Memos、ReMe 等

**信息组织维度**：
- **用户维度（个人记忆）**：个人知识库、用户画像、个性化推荐
- **业务领域维度**：领域经验、工具使用经验、可沉淀至知识库

---

## 三、短期记忆的上下文工程策略

### 3.1 核心策略

#### 上下文缩减（Context Reduction）

通过减少上下文中的信息量来降低 token 消耗：

1. **保留预览内容**：对于大块内容，只保留前 N 个字符或关键片段作为预览
2. **总结摘要**：使用 LLM 对整段内容进行总结摘要，保留关键信息

#### 上下文卸载（Context Offloading）

当内容被缩减后，原始完整内容被卸载到外部存储，消息中只保留引用。当需要完整内容时，可以通过引用重新加载。

**优势**：上下文更干净、占用更小、信息不丢、随取随用

**适用场景**：网页搜索结果、超长工具输出、临时计划等

#### 上下文隔离（Context Isolation）

通过多智能体架构，将上下文拆分到不同的子智能体中。主智能体编写任务指令，发送给子智能体，子智能体完成任务后返回结果。

**优势**：上下文小、开销低、简单直接

### 3.2 各框架的实现方式

**Google ADK**：
```python
from google.adk.apps.app import App, EventsCompactionConfig

app = App(
    name='my-agent',
    root_agent=root_agent,
    events_compaction_config=EventsCompactionConfig(
        compaction_interval=3,  # 每3次新调用触发压缩
        overlap_size=1          # 包含前一个窗口的最后一次调用
    ),
)
```

**LangChain**：
```python
from langchain.agents import create_agent
from langchain.agents.middleware import SummarizationMiddleware

agent = create_agent(
    model="gpt-4o",
    tools=[...],
    middleware=[
        SummarizationMiddleware(
            model="gpt-4o-mini",
            max_tokens_before_summary=4000,  # 4000 tokens时触发摘要
            messages_to_keep=20,  # 摘要后保留最后20条消息
        ),
    ],
)
```

**AgentScope**：
```java
AutoContextMemory memory = new AutoContextMemory(
    AutoContextConfig.builder()
        .msgThreshold(100)
        .maxToken(128 * 1024)
        .tokenRatio(0.75)
        .build(),
    model);

ReActAgent agent = ReActAgent.builder()
    .name("Assistant")
    .model(model)
    .memory(memory)
    .build();
```

---

## 四、长期记忆技术架构

### 4.1 核心组件

1. **LLM 大模型**：提取短期记忆中的有效信息
2. **Embedder 向量化**：将文本转换为语义向量
3. **VectorStore 向量数据库**：持久化存储记忆向量
4. **GraphStore 图数据库**：存储实体-关系知识图谱
5. **Reranker 重排序器**：对检索结果按语义相关性重新排序
6. **SQLite**：记录所有记忆操作的审计日志

### 4.2 Record & Retrieve 流程

**Record（记录）**：
LLM 事实提取 → 信息向量化 → 向量存储 → 图数据库存储 → SQLite 操作日志

**Retrieve（检索）**：
User query 向量化 → 向量数据库语义检索 → 图数据库关系补充 → Reranker-LLM 排序 → 结果返回

### 4.3 长期记忆与 RAG 的区别

技术层面相似（向量化、相似性检索、上下文注入），但功能层面不同：
- **RAG**：面向静态知识库
- **长期记忆**：面向个性化、动态更新的用户记忆

### 4.4 关键挑战

1. **准确性**：记忆建模、管理、检索相关性
2. **安全和隐私**：数据加密、防恶意攻击、用户数据掌控权
3. **多模态记忆**：跨模态关联与检索、统一表示、毫秒级响应

### 4.5 Agent 框架集成

**集成 Mem0**：
```java
Mem0LongTermMemory mem0Memory = new Mem0LongTermMemory(
    Mem0Config.builder()
        .apiKey("your-mem0-api-key")
        .build());

ReActAgent agent = ReActAgent.builder()
    .name("Assistant")
    .model(model)
    .memory(memory)  // 短期记忆
    .longTermMemory(mem0Memory)  // 长期记忆
    .build();
```

**集成 ReMe**：
```java
ReMeLongTermMemory remeMemory = ReMeLongTermMemory.builder()
    .userId("user123")
    .apiBaseUrl("http://localhost:8002")
    .build();

ReActAgent agent = ReActAgent.builder()
    .name("Assistant")
    .model(model)
    .memory(memory)
    .longTermMemory(remeMemory)
    .longTermMemoryMode(LongTermMemoryMode.BOTH)
    .build();
```

---

## 五、行业趋势与产品对比

### 5.1 技术发展趋势

- **Memory-as-a-Service (MaaS)**：记忆即服务，成为 AI 应用基础设施
- **精细化记忆管理**：借鉴人脑机制，分层动态架构
- **多模态记忆系统**：支持文本、视觉、语音统一存储
- **参数化记忆**：在 Transformer 中引入可学习记忆单元

### 5.2 开源产品对比

目前 **Mem0** 是长期记忆产品的领先者，被各方作为评测基准。

---

## 结语

记忆系统作为 AI Agent 的核心基础设施，各框架内置的压缩、卸载、摘要策略已能解决 80-90% 的通用场景问题。长期记忆未来会更加贴近人脑的记忆演化模式，以云服务模式提供通用记忆服务，共同助力 Agent 迈向更高阶的智能。

---

## 参考文档

1. [FlowLLM Context Engineering](https://github.com/FlowLLM-AI/flowllm/tree/main/docs/zh/reading)
2. [Google ADK Memory](https://google.github.io/adk-docs/sessions/memory/)
3. [LangChain Memory](https://docs.langchain.com/oss/python/langchain/long-term-memory)
4. [AgentScope Memory](https://doc.agentscope.io/zh_CN/tutorial/task_memory.html)
5. [O-MEM](https://arxiv.org/abs/2511.13593)
