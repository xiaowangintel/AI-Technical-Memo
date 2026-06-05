# README.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/training/weight_transfer/README.md`
- **Repository**: `vllm-project/vllm`

## Content Analysis / 内容分析
- **EN:** This overview presents weight transfer as the mechanism that keeps vLLM inference synchronized with a training process during RL workflows. It explains the common protocol, backend choices, configuration surface, and HTTP APIs.
- **CN:** 这份总览将权重传输描述为 RL 工作流中保持 vLLM 推理端与训练端同步的核心机制。它解释了通用协议、后端选择、配置方式以及 HTTP API。

## Key Concepts / 关键概念
- **EN:** The protocol has four phases: initialize, start, update, and finish.
  **CN:** 该协议包含四个阶段：初始化、开始更新、传输更新、完成更新。
- **EN:** Built-in backends map to distinct deployment patterns: NCCL for separate GPUs and IPC for colocated sharing.
  **CN:** 内置后端对应不同部署模式：NCCL 面向分离 GPU，IPC 面向共置共享。
- **EN:** Users can configure weight transfer in Python with `WeightTransferConfig` or in serving mode via CLI JSON.
  **CN:** 用户既可以在 Python 中通过 `WeightTransferConfig` 配置，也可以在服务模式中通过 CLI JSON 配置。
- **EN:** HTTP control endpoints for weight transfer are development-oriented and require `VLLM_SERVER_DEV_MODE=1`.
  **CN:** 与权重传输相关的 HTTP 控制端点偏向开发用途，需要启用 `VLLM_SERVER_DEV_MODE=1`。
