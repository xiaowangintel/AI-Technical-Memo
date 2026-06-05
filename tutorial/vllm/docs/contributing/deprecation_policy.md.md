# deprecation_policy.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/contributing/deprecation_policy.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** Defines a release-based policy for deprecating public-facing behavior in vLLM, including CLI flags, environment variables, configuration files, API surfaces, and public Python APIs.
- **CN:** 本文定义了 vLLM 面向用户能力的弃用策略，覆盖 CLI 参数、环境变量、配置文件、API 接口以及公开 Python API。
- **EN:** The policy centers on a three-stage pipeline across minor releases so users receive warning, a temporary escape hatch, and only then full removal.
- **CN:** 该策略围绕跨多个次版本的三阶段流程展开，让用户先收到警告，再拥有临时回退开关，最后才进入彻底移除阶段。

## Key Concepts / 关键概念
- **EN:** Minor-release cadence — Deprecations and removals are tied to Y releases in semantic versioning rather than patch releases.
- **CN:** 次版本节奏 — 弃用与移除绑定在语义化版本中的 Y 版本，而不是补丁版本。
- **EN:** Three-stage pipeline — Features move from deprecated-on, to deprecated-off, to removed.
- **CN:** 三阶段流程 — 功能会从“默认开启但已弃用”演进到“默认关闭但已弃用”，最终被移除。
- **EN:** Communication channels — Warnings should surface in help text, logs, docs, release notes, and other user-facing places.
- **CN:** 沟通渠道 — 弃用提示应出现在帮助信息、日志、文档、发布说明等用户可见位置。
- **EN:** Patch-release safety — Patch releases must not remove deprecated functionality.
- **CN:** 补丁版本安全性 — 补丁版本禁止移除已经弃用的功能。
