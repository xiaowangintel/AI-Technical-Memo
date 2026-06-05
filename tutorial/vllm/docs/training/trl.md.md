# trl.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/training/trl.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This page describes how Hugging Face TRL integrates with vLLM for online trainers that need fast completion generation. The main operational choice is whether vLLM runs as a separate server or colocated inside the trainer process.
- **CN:** 本文说明 Hugging Face TRL 如何与 vLLM 集成，以支持需要高速生成补全结果的在线训练器。主要的运行选择在于：vLLM 作为独立服务运行，还是与 trainer 共置于同一进程。

## Key Concepts / 关键概念
- **EN:** TRL supports multiple online trainers with vLLM, including GRPO, Online DPO, RLOO, Nash-MD, and XPO.
  **CN:** TRL 支持多个结合 vLLM 的在线训练器，包括 GRPO、Online DPO、RLOO、Nash-MD 和 XPO。
- **EN:** Users enable the integration by setting `use_vllm=True` in trainer configuration.
  **CN:** 用户可在训练配置中设置 `use_vllm=True` 来启用该集成。
- **EN:** Server mode isolates generation on dedicated GPUs and communicates over HTTP.
  **CN:** Server 模式会将生成任务隔离到专用 GPU 上，并通过 HTTP 通信。
- **EN:** Colocate mode shares trainer resources for better utilization but may increase memory contention; some trainers also support vLLM sleep mode.
  **CN:** Colocate 模式共享训练资源以提升利用率，但也可能增加显存争用；部分训练器还支持 vLLM sleep mode。
