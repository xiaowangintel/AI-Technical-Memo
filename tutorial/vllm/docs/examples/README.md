# README.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/examples/README.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This file is the top-level index for vLLM's example directory. It organises examples into 14 thematic categories, each linking to a subdirectory under `examples/`. The categories span from minimal "hello world" usage all the way to production observability and reinforcement learning.

**CN:** 本文件是 vLLM 示例目录的顶层索引，将示例组织为 14 个主题分类，每个分类指向 `examples/` 下的对应子目录。分类范围从最简单的入门用法延伸至生产级可观测性和强化学习。

### Structure / 结构

**EN:** A flat bullet list maps each category name (with hyperlink) to a one-line description. No code is shown; the file serves purely as navigation.

**CN:** 文件采用扁平项目列表结构，每个分类名称附带超链接和单行描述，不包含代码，纯粹用于导航。

### Categories Covered / 涵盖分类

**EN:** `basic`, `generate`, `pooling`, `speech_to_text`, `features`, `reasoning`, `tool_calling`, `applications`, `rl`, `deployment`, `ray_serving`, `disaggregated`, `observability`.

**CN:** 基础推理、文本生成、向量池化、语音转文字、特性演示、推理能力、工具调用、应用示例、强化学习、生产部署、Ray 服务、解耦推理（分离预填充与解码）、可观测性。

## Key Concepts / 关键概念

- **EN:** Offline inference vs. online serving — `basic/` demonstrates both modes. **CN:** 离线推理与在线服务——`basic/` 目录同时演示两种模式。
- **EN:** Disaggregated serving — `disaggregated/` covers separate prefill/decode instances, multiple KV-cache connectors (LMCache, Mooncake, FlexKV, P2P NCCL), and failure recovery. **CN:** 解耦服务——`disaggregated/` 涵盖独立预填充/解码实例、多种 KV 缓存连接器及故障恢复。
- **EN:** Observability — `observability/` provides metrics, logging, OpenTelemetry tracing, and Grafana/Perses dashboards. **CN:** 可观测性——`observability/` 提供指标采集、日志、OpenTelemetry 追踪及 Grafana/Perses 仪表盘。
- **EN:** Feature demonstrations — `features/` targets individual vLLM capabilities such as APC, LoRA, speculative decoding, and structured outputs. **CN:** 特性演示——`features/` 聚焦 APC、LoRA、投机解码、结构化输出等单项功能。
- **EN:** RL integration — `rl/` shows how vLLM can be used as a rollout engine in reinforcement learning pipelines. **CN:** 强化学习集成——`rl/` 展示如何将 vLLM 用作强化学习流水线中的推演引擎。
