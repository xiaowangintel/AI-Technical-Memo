# editing-agent-instructions.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/contributing/editing-agent-instructions.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This file is a maintenance guide for `AGENTS.md` and linked domain guides, with a strong emphasis on keeping instruction files short enough to stay useful to automated agents.
- **CN:** 本文是维护 `AGENTS.md` 及其关联领域指南的说明，核心目标是让这些指令文件保持精简，从而持续对自动化 agent 有效。
- **EN:** It encourages only high-signal, project-specific, non-obvious guidance and pushes contributors away from reactive rule sprawl, duplicated instructions, hardcoded paths, and untested documentation changes.
- **CN:** 它鼓励只添加高价值、项目特有、非显而易见的规则，并避免被动堆积规则、重复说明、硬编码路径和未经验证的文档改动。

## Key Concepts / 关键概念
- **EN:** Token budget — `AGENTS.md` and domain guides have explicit line-count budgets to control prompt size.
- **CN:** 令牌预算 — `AGENTS.md` 和领域指南都有明确的行数预算，用来控制提示上下文大小。
- **EN:** Content placement — Project-wide rules belong in `AGENTS.md`, while local conventions belong in domain guides.
- **CN:** 内容归位 — 全局规则应放入 `AGENTS.md`，局部约定应放到对应领域指南中。
- **EN:** Anti-patterns — The guide explicitly warns against reactive accumulation, copy-paste drift, imperative walls, and stale config snapshots.
- **CN:** 反模式 — 文档明确警告不要出现事件驱动式堆积、复制粘贴漂移、命令式长清单和过时配置快照。
- **EN:** Submission checklist — A final checklist forces authors to verify necessity, consistency, size, and real agent behavior.
- **CN:** 提交清单 — 最后的检查清单要求作者验证必要性、一致性、篇幅以及 agent 的实际行为。
