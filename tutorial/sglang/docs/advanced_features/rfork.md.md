# rfork.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/rfork.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: R-Fork (Tensor Remote Fork) is a novel weight loading methodology that leverages efficient inter-node GPU-to-GPU data transfer path to load tensors from a running SGLang instance to a new instance with zero-copy. / 该文档围绕 R-Fork 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** R-Fork (Tensor Remote Fork) is a novel weight loading methodology that leverages efficient inter-node GPU-to-GPU data transfer path to load tensors from a running SGLang instance to a new instance with zero-copy.
**CN:** 本节围绕 Overview 展开，概述了 R-Fork, instance, loading, GPU-to-GPU 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Usage
**EN:** This section provides a comparison table for Usage, covering columns such as Argument, Usage and examples such as load-format, remote-instance-weight-loader-backend, remote-instance-weight-loader-seed-instance-ip, remote-instance-weight-loader-seed-instance-service-port.
**CN:** 本节围绕 Usage 展开，概述了 nccl, transfer_engine, seed, backend 等要点，并说明相关配置、流程、示例或限制条件。

### Section: NCCL as backend
**EN:** This content focuses on NCCL as backend and highlights args, shell, instance, sglang.launch_server.
**CN:** 本节围绕 NCCL as backend 展开，概述了 args, shell, instance, sglang.launch_server 等要点，并说明相关配置、流程、示例或限制条件。

### Section: TransferEngine as backend
**EN:** This content focuses on TransferEngine as backend and highlights args, shell, sglang.launch_server, seed.
**CN:** 本节围绕 TransferEngine as backend 展开，概述了 args, shell, sglang.launch_server, seed 等要点，并说明相关配置、流程、示例或限制条件。

### Section: ModelExpress as backend
**EN:** ModelExpress is a coordination service that manages P2P weight transfer metadata. It removes the need for direct seed IP/port configuration by providing a centralized registry that seeds publish to and clients discover from.
**CN:** 本节围绕 模型Express as backend 展开，概述了 ModelExpress, seed, model_name, TransferEngine 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** ModelExpress / **CN:** 模型Express
- **EN:** seed / **CN:** seed
- **EN:** instance / **CN:** instance
- **EN:** nccl / **CN:** nccl
- **EN:** transfer_engine / **CN:** transfer_engine
- **EN:** R-Fork / **CN:** R-Fork
- **EN:** TransferEngine / **CN:** TransferEngine
- **EN:** backend / **CN:** backend

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
