---
layout: ../../layouts/PostLayout.astro
title: "Agent 时代下的锚点"
description: "从 Java 后端与企业知识检索实践出发，梳理 Agent 从 Demo 走向生产环境时不会轻易过时的八类工程能力。"
date: 2026-08-30
author: Mat
tags:
  - AI Agent
  - RAG
  - 系统工程
  - Java
cover: "/assets/images/agent-anchor-cover.png"
coverAlt: "海面上的技术变化与海面下的工程锚点"
coverCaption: "框架和热词像海面的浪，真正支撑 Agent 进入生产环境的，是海面下相对稳定的工程能力。"
draft: false
ai_assisted: true
---

> **AI 使用说明：** 本文基于作者提供的技术经历与学习计划，由生成式 AI 协助撰写、整理和制作配图；公开发布前，涉及个人经历、技术判断与引用的内容仍需作者最终确认。

> **追框架，不如追不变量。**

过去一年，Agent 相关的框架、协议和产品形态变化很快。今天讨论的是某种 runtime、planner 或 skills system，明天又可能出现新的名字。

但从 Java 后端、企业知识检索和 AI 助手项目的实践出发，我越来越确信：**变化的是封装方式，不变的是系统约束。**

一个 Agent 真正要解决的，不只是如何调用模型，而是如何把概率性的模型放进一个可控、可靠、可恢复、可评估的工程系统。本文尝试回答三个问题：

1. 哪些东西会快速变化？
2. 哪些工程问题长期存在？
3. 后端工程师应该把时间投入在哪里？

---

## 一、框架会变，系统问题不会

很多流行概念都可以映射到更底层的系统能力：

| 常见概念 | 背后长期存在的能力 |
| --- | --- |
| MCP、Tool Calling | 工具协议、能力发现与执行边界 |
| Prompt、RAG、Memory | 上下文组装与信息预算 |
| Planner、ReAct、Reflection | 执行策略与控制循环 |
| Session、Checkpoint | 持久状态与失败恢复 |
| Guardrail、Approval | 权限、风险控制与人工介入 |
| Trace、Eval、Replay | 可观测、归因与回归测试 |

[MCP 的官方架构说明][mcp-architecture]本身也体现了这种分层：协议负责连接 host、client 与 server，并暴露 resources、prompts 和 tools；权限策略、上下文聚合和用户授权仍由宿主应用负责。

因此，评价一个 Agent 系统时，我更愿意先问下面几个问题：

- 上下文如何组装，信息从哪里来？
- 工具调用如何校验、审批和控制副作用？
- 任务中断后能否恢复，重试会不会重复执行？
- 结果出错时，能否定位是检索、推理还是工具执行的问题？
- 高风险步骤何时必须交还给人？

为了帮助自己思考，我把 Agent 简化成一个非严格的工程心智模型：

```text
Agent = Model + Context + State + Tools + Environment + Policy + Loop + Feedback
```

模型很重要，但生产级能力通常由等号右侧的整个系统共同决定。

---

## 二、Agent 工程的八个锚点

### 1. Context Engineering：上下文工程

Prompt 只是上下文的一部分。一个成熟的 Agent 在调用模型前，往往还要处理对话历史、检索结果、长期记忆、工具返回、状态快照、权限信息和 token 预算。

真正的问题不是“能否把信息塞进窗口”，而是：

> **能否在正确的时间，把正确的信息，以可验证的形式交给模型。**

从这个视角看，RAG 不会消失，但它更可能成为 Context Runtime 中负责检索和引用的一部分，而不是完整系统本身。

### 2. Action Runtime：工具调用只是入口

协议能够让模型发现和调用工具，但从一次调用到可靠执行，中间仍有很长的工程链路：

- schema 与参数校验
- 超时、取消和重试
- 幂等与去重
- 风险分级和审批
- 结果结构化
- 审计与回放

真正有价值的不是“模型会调工具”，而是让一个不确定的模型能够安全地操作确定性的业务系统。

### 3. Durable Execution：可持续、可恢复的执行

十秒钟的问答很难暴露状态问题，持续十分钟甚至更久的任务却会立即遇到中断、重试、恢复和重复副作用。

因此，长任务需要 checkpoint、resume、idempotency、event log 和明确的状态机。[LangGraph 的文档][langgraph-durable]将 durable execution 与 human-in-the-loop 列为长运行 Agent 的基础能力；[OpenAI Agents SDK][openai-running]也提供了面向长任务的持久化执行集成。这些实现会变化，但“失败后从哪里继续”不会变。

### 4. Authority Model：权限不只是 RBAC

传统企业系统常用 RBAC 回答“谁能访问什么”。Agent 还要继续回答：

> 谁可以让哪个 Agent，在什么上下文中，对什么资源执行什么能力；风险上限是多少；是否需要审批；事后如何审计？

这会涉及身份、能力、资源边界、风险等级、审批和审计。权限不是上线前补的一层开关，而应进入工具设计和运行时状态。

### 5. Environment & Sandbox：执行环境与隔离

Chatbot 主要输出文本，Agent 则可能修改文件、访问网络、调用内部 API，甚至执行系统命令。一旦系统能够改变外部状态，沙箱、秘密信息保护、资源配额和网络边界就成为基础设施问题。

具体沙箱产品可能更替，但 Execution Isolation 这一需求不会消失。

### 6. Trajectory & Observability：可追踪、可归因、可回放

知识库问答给出错误答案时，原因可能出现在多个阶段：没有召回、排序错误、上下文截断、工具结果被误解，或者模型生成错误。如果系统只保留最终答案，就无法知道应该优化哪一层。

因此，Agent 的可观测性至少应覆盖：

- 每一步输入、输出和耗时
- 工具参数、结果与异常
- 检索候选、排序和引用来源
- 状态变化与人工审批
- token、延迟和成本
- 可重放的 trajectory 与回归样本

### 7. Learning & Skills：把过程沉淀成能力

Memory 通常保存事实，Skill 更接近可复用的过程：什么条件下使用什么工具、怎样检查结果、失败后如何恢复。

```text
Experience -> Reflection -> Procedure -> Skill -> Better Execution
```

这条链路的价值不在于让 Agent “自动进化”，而在于把经过验证的做法沉淀为可检查、可版本化的操作规范。

### 8. Human-in-the-loop：为介入点做设计

高风险、信息不足或涉及外部副作用的任务，不应把“全自动”当成默认目标。成熟系统需要明确支持 interrupt、approve、reject、edit、resume 和 rollback。

[OpenAI Agents SDK 的 human-in-the-loop 说明][openai-hitl]展示了一个典型模式：敏感工具调用先暂停，将待审批状态持久化，再根据人的决定继续或拒绝。人工介入不是自动化失败，而是系统边界的一部分。

---

## 三、把抽象锚点落到知识检索实践

我的实践起点不是从零开始：我已经接触过企业知识检索与 AI 助手项目。接下来真正需要补齐的，不是再做一个“上传文档后可以聊天”的 Demo，而是把检索系统做成可评测、可归因、可持续优化的工程系统。

以一次回答错误为例，与其笼统地说“模型不够好”，不如把问题拆成下面几层：

1. **Index**：文档是否正确切分、更新和删除？
2. **Query Understanding**：查询是否需要改写或拆解？
3. **Retrieval**：相关内容是否进入候选集？
4. **Ranking**：真正相关的文档是否排在前面？
5. **Context**：重要证据是否在压缩或拼装时丢失？
6. **Generation**：回答是否忠于证据，并给出正确引用？

我准备按以下路径验证，而不是预设结果：

- 建立不少于 200 条带标注的 Golden Dataset；
- 对比 BM25、Dense、Hybrid 与 Reranker；
- 同时记录 Recall、MRR、nDCG、P95 延迟和成本；
- 为检索、排序、上下文和生成阶段保留 trace；
- 把失败样本沉淀为回归集，而不是只看一次演示效果。

这也是我理解的 Context Engineering：它不是 Prompt 技巧的扩大版，而是围绕信息来源、预算、权限、引用和评测建立完整链路。

---

## 四、我会长期投入的四条主线

### 1. Agent Runtime / Harness Engineering

重点研究工具注册、schema、执行控制、超时取消、幂等、审批、审计和沙箱。目标不是多接几个工具，而是理解模型如何安全地改变真实系统状态。

### 2. Context Runtime

把已有的搜索和 RAG 经验扩展到 context assembly、memory、compression、budgeting、caching 和 attribution，并用评测数据决定优化方向。

### 3. Reliable Execution

继续补齐 state machine、checkpoint、resume、scheduler、event log 和 recovery。这部分与 Java 后端的事务、状态和长流程工程经验最接近。

### 4. Eval / Replay / Regression

建立 trace、replay、failure analysis 和 regression suite。能跑通 Demo 只是起点，能够稳定复现问题、比较版本并防止退化，才是工程能力。

---

## 五、哪些方向我不会重仓

### 不把职业价值绑定到单一框架

框架需要会用，但不值得把长期能力押在某套 API 的细节上。更重要的是理解它解决了什么问题，又把什么责任留给了应用层。

### 不把 Prompt 技巧当成全部壁垒

Prompt 仍然重要，但很多浅层技巧会随着模型升级而失效。信息选择、状态、权限、评测和反馈闭环更值得长期积累。

### 不把 Multi-Agent 当作默认答案

[Anthropic 关于构建有效 Agent 的文章][anthropic-agents]建议从最简单、可满足需求的方案开始。多 Agent 是一种编排方式，不会自动解决状态、权限、成本和调试问题。

---

## 结语：从追风口到建锚点

Agent 时代真正稀缺的，不只是接入一个模型，而是把不确定的智能放进一个可以信任的工程系统。

对我来说，下一阶段的重点很明确：

- 用评测和 trace 把知识检索做实；
- 用状态、幂等和恢复机制把执行链路做稳；
- 用权限、审批和沙箱控制真实副作用；
- 用回放和回归测试持续积累工程经验。

框架还会继续变化。只要 Agent 需要进入真实业务，Context、State、Action、Policy、Observability 和 Human Control 就仍然是值得长期投入的锚点。

> **追框架，不如追不变量；追热词，不如建锚点。**

---

## 参考资料

1. [Model Context Protocol：Architecture][mcp-architecture]
2. [LangGraph：Durable execution][langgraph-durable]
3. [OpenAI Agents SDK：Running agents][openai-running]
4. [OpenAI Agents SDK：Human-in-the-loop][openai-hitl]
5. [Anthropic：Building effective agents][anthropic-agents]

[mcp-architecture]: https://modelcontextprotocol.io/specification/2025-06-18/architecture
[langgraph-durable]: https://docs.langchain.com/oss/python/langgraph/durable-execution
[openai-running]: https://openai.github.io/openai-agents-python/running_agents/
[openai-hitl]: https://openai.github.io/openai-agents-python/human_in_the_loop/
[anthropic-agents]: https://www.anthropic.com/engineering/building-effective-agents
