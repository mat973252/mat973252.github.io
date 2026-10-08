---
layout: ../../layouts/PostLayout.astro
title: "ctxpack：让下一次会话接得上"
description: "把目标、决策与 Git 快照整理成一份可检查的交接文档。"
date: 2026-10-08
author: Mat
readingTime: "约 3 分钟"
tags: [ctxpack, 上下文交接, Coding Agent]
---

写代码的任务经常比一个会话长。换模型、换工具，或者重新打开终端后，下一位 Agent 需要知道：现在做什么、哪些方案失败过、哪些结果已经核验、下一步从哪里开始。

[ctxpack](https://github.com/mat973252/ctxpack) 把这些信息保存在仓库内的 `.ctxpack/`，再输出可交接的 Markdown。目标是让任务状态可检查、可携带。

## 一个最小流程

以下命令对应已发布的 **v0.1.0**，需要 Node.js 22 或更新版本。在 Git 仓库里执行：

```sh
npm install -g @mat973252/ctxpack@0.1.0
ctxpack init
```

先编辑 `.ctxpack/state.json`，填写目标、当前状态和下一步；在 `decisions.md`、`failures.md` 中记录决策与失败经验。比如“接口迁移进行中；配置已完成；下一步验证一次真实调用”，而不是只写“继续开发”。

```sh
ctxpack capture
ctxpack validate
```

`capture` 记录 Git 工作区快照。`validate` 核对必填信息和当前 Git 状态；未提交改动或快照变化都会使预检退出 1。复核并提交或清理改动、补齐必填信息，再重新采集并通过预检后，生成交接文档：

```sh
ctxpack handoff --to codex > HANDOFF.md
```

下一位 Agent 读取交接文档，核对关键证据，再接着执行。也可以用 `ctxpack ui` 在终端查看状态与交接预览。

## 校验通过，说明什么

交接与预检命令只读，不会自动执行下一步。干净且匹配的 Git 快照，只证明元数据一致；目标是否仍然有效、验证结果是否过期，仍需要接收者判断。

截至本文发布，公开安装版本为 v0.1.0。结构化诊断等后续改动仍在[草稿 PR #3](https://github.com/mat973252/ctxpack/pull/3)，未包含在这个安装示例里。当前也没有证据可支持“普遍提高恢复效率”的结论。

交接文档的价值，先从一个小要求开始：让接手者知道下一步是什么，以及为什么。
