---
name: mall-code-review
description: Review current git changes for Java backend projects using structured code review checks plus team code standards, project architecture constraints, and CR output requirements. Use when the user asks for code review, CR output, git diff review, or wants review followed by fixes.
---

# Mall Code Review

## Overview

对当前 git 变更做结构化代码评审，再叠加团队 Java 后端规范、项目 `ARCHITECTURE.md` 约束、以及项目根目录 `CR-STANDARD.md` 的补充规则。评审结果必须写入当前仓库 `CR/` 目录，文件名为 `{yyyyMMdd}_CR.md`。

默认先输出 review 结果；如果用户明确要求“review 并修复”或在 review 后确认修复，则继续按 findings 实施修复。

## 何时使用

当用户出现以下诉求时使用本 skill：

- review 当前 git changes
- review 当前分支与指定分支/提交的差异
- 输出 CR markdown 文档
- 按团队 Java 后端规范做代码评审
- review 后继续修复问题

典型触发语句：

- `review 当前 git changes`
- `按 CR 规范帮我做一次 code review`
- `把 review 结果写到 CR 目录`
- `review 后直接修复`

## 不适用场景

以下场景不应直接套用本 skill：

- 纯前端页面/样式评审
- 与当前 git 变更无关的泛泛代码质量讨论
- 只想做架构方案讨论、不涉及具体代码 diff
- 需要直接大范围重构而不是先做 CR 的任务

## 产出物

本 skill 的标准产出物有两类：

1. 终端中的评审结论摘要
2. 仓库根目录 `CR/{yyyyMMdd}_CR.md` 评审文档

如果用户要求进入修复模式，还会额外产出：

- 本地代码修复结果
- 修复说明与未修复项说明

## Workflow

### 1. 确认评审范围

- 运行 `git status -sb`、`git diff --stat`、`git diff`，必要时再看 `git diff --cached`。
- 如果没有变更，确认用户是要 review staged changes，还是某个 commit / branch range。
- 如果 diff 很大，先按模块或功能分组总结，再分批 review。

### 2. 先读取项目事实，再下结论

- 如果仓库根目录存在 `ARCHITECTURE.md`，必须先读，把它视为项目事实源。
- 如果仓库根目录存在 `CR-STANDARD.md`，必须在出 findings 前读取，把它视为项目级补充约束。
- 如果仓库根目录不存在 `CR-STANDARD.md`，则按“无项目附加规则”继续，不要把缺失本身当成问题。
- 如果 `ARCHITECTURE.md` 或源码没有确认某件事，不要脑补架构、目录放置规则或运行方式。

对于类似当前 mall 系列的 Java 后端仓库，重点关注：

- 模块边界
- 包路径放置
- controller / service / mapper 分层
- shared / common 包滥用
- auth、payment、order、message、数据写路径等关键链路

### 3. 执行通用评审

按需加载这些参考文件：

- `references/core-review-checklist.md`
- `references/severity-standard.md`
- `references/team-standard-checklist.md`
- `references/output-template.md`

至少覆盖以下维度：

- 架构与 SOLID 坏味道
- 安全与可靠性风险
- 错误处理与边界条件
- 性能与数据访问风险
- 死代码与可删除代码

### 4. 执行团队与项目规范检查

除了通用 review 外，必须显式检查 `references/team-standard-checklist.md` 与当前仓库 `CR-STANDARD.md`。

优先检查明确、可执行的代码规则，例如：

- 代码放置是否遵循现有模块和包结构
- 类、接口方法、重要参数/DTO 是否按团队标准具备注释
- 需要可替换实现的地方是否做到了面向抽象编程
- 异常是否沿用既有业务异常模式，而不是随意抛错或吞错
- Redis 是否被误当成持久化业务数据来源
- 易变配置是否被硬编码进本地 properties 或代码
- SQL / 查询逻辑是否存在高风险全表扫描
- 日志是否存在大对象、大 JSON、字符串拼接等问题
- 返回 List 字段时，是否符合项目“空列表优于 null”的约定

如果 `CR-STANDARD.md` 的规则更严格，以项目根目录 `CR-STANDARD.md` 为准。

### 5. 生成 CR 文档

- 将 review 结果写入 `<repo-root>/CR/{yyyyMMdd}_CR.md`
- 如果 `CR/` 目录不存在，先创建
- 输出结构遵循 `references/output-template.md`
- 同名文件已存在时，默认覆盖为本次最新评审结果，不做追加拼接

输出内容必须包含核心评审结构，并额外追加：

- `新增/修改的类`
- `代码修改主要目的总结`

- `新增/修改的类`：列出本次 diff 涉及的类，并标明“新增”或“修改”
- `代码修改主要目的总结`：按模块或功能总结这批代码改动的主要目的

### 6. 修复模式

在 CR 文档写完后：

- 如果用户只要求 review，则在输出 findings 和 CR 文件路径后结束
- 如果用户一开始就要求“review 并修复”，则写完报告后直接进入修复
- 如果用户在看到报告后再要求修复，则按优先级实施修复

默认修复顺序：

1. P0
2. P1
3. P2

默认修复全部可明确落地的问题，但只修改本地文件，不做 `git commit`、`git push`、`git merge` 等 git 提交动作。

修复时不要顺手做大范围重构。除非用户明确扩大范围，否则只修本次 findings 覆盖的问题。

## 完成定义

只有同时满足以下条件，才算本 skill 执行完成：

- 已确认 review 范围
- 已读取 `ARCHITECTURE.md`
- 已按存在情况处理 `CR-STANDARD.md`
- 已输出结构化 findings
- 已写出 `CR/{yyyyMMdd}_CR.md`
- 若进入修复模式，已说明修复了哪些问题、哪些问题未修

## 输出规则

- Findings 必须按 `references/severity-standard.md` 的严重级别排序
- 先看正确性、安全性、数据风险、事务风险、行为回归
- 不要把纯风格问题夸大成阻塞问题
- 明确区分：
  - 必须现在修
  - 建议本次修
  - 可以后续跟进
- 每条 finding 尽量写清楚：
  - 问题位置
  - 问题现象
  - 风险影响
  - 建议修法
- 如果没有发现问题，也必须写清楚：
  - 检查了什么
  - 没有验证什么
  - 剩余风险是什么

## Notes

- 本 skill 针对 Java 后端 CR 流程优化，尤其适合 Spring Boot + MyBatis 风格仓库。
- 所有结论必须落在真实 diff 和真实仓库规则上，不要输出脱离代码的泛泛教材式建议。
- 如果用户要求“只 review 不改代码”，即使已经识别出明确修法，也不要直接修改文件。
- 优先做“高信号、可执行、可落地”的评审，不输出大段空泛理论。
