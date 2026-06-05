# disaggregation.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/diffusion/disaggregation.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Split a monolithic text-to-video/image pipeline into independent **Encoder**, **Denoiser**, and **Decoder** roles, each running on its own GPU(s). A central **DiffusionServer** routes requests through the pipeline. / 该文档围绕 Disaggregated Diffusion Pipeline 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** Split a monolithic text-to-video/image pipeline into independent **Encoder**, **Denoiser**, and **Decoder** roles, each running on its own GPU(s). A central **DiffusionServer** routes requests through the pipeline.
**CN:** 本节围绕 Overview 展开，概述了 GPU, Split, Encoder, Decoder 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Quick Start
**EN:** This section provides a comparison table for Quick Start, covering columns such as --disagg-role, What it runs and examples such as monolithic, encoder, denoiser, decoder.
**CN:** 本节围绕 Quick Start 展开，概述了 --disagg-role, monolithic, server, ENCODER 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Single-Machine Example (Verified)
**EN:** The following commands have been tested end-to-end on an 8×H200 machine with Wan-AI/Wan2.1-T2V-1.3B-Diffusers. Each role runs on a separate GPU via --base-gpu-id; the server head node requires no GPU.
**CN:** 本节围绕 Single-Machine Example (Verified) 展开，概述了 GPU, Terminal, --base-gpu-id, Wan-AI/Wan2.1-T2V-1.3B-Diffusers 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Multi-Machine Example
**EN:** This content focuses on Multi-Machine Example and highlights Machine, serve, Wan-AI, model-path.
**CN:** 本节围绕 Multi-Machine Example 展开，概述了 Machine, serve, Wan-AI, model-path 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Multiple Instances per Role
**EN:** Use semicolons in --*-urls to register multiple instances: ``bash # 2 encoders + 2 denoisers (4-GPU SP each) + 1 decoder sglang serve --model-path ...
**CN:** 本节围绕 Multiple Instances per Role 展开，概述了 GPU, urls, serve, server 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Port Convention
**EN:** This section provides a comparison table for Port Convention, covering columns such as Socket, Port and examples such as DS frontend (ROUTER), Encoder result (PULL), Denoiser result (PULL), Decoder result (PULL).
**CN:** 本节围绕 Port Convention 展开，概述了 PULL, scheduler_port, result, Port 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Transfer Mechanism
**EN:** Tensor data between roles (encoder→denoiser, denoiser→decoder) is transferred via a P2P transfer engine. The DiffusionServer only routes lightweight control messages (alloc/push/ready); actual tensor data flows directly between instances.
**CN:** 本节围绕 Transfer Mechanism 展开，概述了 data, P2P, RDMA, Tensor 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Transfer Flow
**EN:** **Sender** (encoder/denoiser) stages tensors: async copy to transfer buffer (GPU or CPU pinned, depending on GPUDirect support), overlapped with metadata JSON serialization.
**CN:** 本节围绕 Transfer Flow 展开，概述了 DiffusionServer, sends, Sender, Receiver 等要点，并说明相关配置、流程、示例或限制条件。

### Section: RDMA Flags
**EN:** This section provides a comparison table for RDMA Flags, covering columns such as Flag, Default, Description and examples such as --disagg-p2p-hostname, --disagg-ib-device, --disagg-transfer-pool-size.
**CN:** 本节围绕 RDMA Flags 展开，概述了 --disagg-ib-device, --disagg-p2p-hostname, MiB, Flag 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Per-Role Parallelism
**EN:** This section provides a comparison table for Per-Role Parallelism, covering columns such as Flag, Description and examples such as --encoder-tp, --denoiser-tp / --denoiser-sp / --denoiser-ulysses / --denoiser-ring, --decoder-tp.
**CN:** 本节围绕 Per-Role 并行策略 展开，概述了 parallelism, Flag, tensor, Encoder 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Other Options
**EN:** This section provides a comparison table for Other Options, covering columns such as Flag, Default, Description and examples such as --disagg-timeout, --disagg-dispatch-policy.
**CN:** 本节围绕 Other Options 展开，概述了 round_robin, Flag, Timeout, Description 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Python API
**EN:** This content focuses on Python API and highlights launch_pool_disagg_server, import, ServerArgs, server_args.
**CN:** 本节围绕 Python API 展开，概述了 launch_pool_disagg_server, import, ServerArgs, server_args 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Architecture
**EN:** `` Client ─── HTTP (port 30000) ──► FastAPI Server │ ▼ DiffusionServer (ROUTER, scheduler_port) ┌───────┼───────┐ PUSH work │ │ │ PUSH work ▼ │ ▼ Encoder[0..N] │ Decoder[0..K] │ │ ▲ P2P tensor │ │ │ P2P tensor transfer ▼ │ │ transfer Denoiser[0..M] ─────┘ │ PULL results ◄────┘ (decoder → DS → client) ``
**CN:** 本节围绕 Architecture 展开，概述了 P2P, PUSH, PULL, work 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Request State Machine
**EN:** `` PENDING → ENCODER_WAITING → ENCODER_RUNNING → ENCODER_DONE │ DENOISING_WAITING → DENOISING_RUNNING → DENOISING_DONE │ DECODER_WAITING → DECODER_RUNNING → DONE ` Any state can transition to FAILED or TIMED_OUT`.
**CN:** 本节围绕 Request State Machine 展开，概述了 DONE, FAILED, PENDING, TIMED_OUT 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** GPU / **CN:** GPU
- **EN:** DiffusionServer / **CN:** Diffusion服务端
- **EN:** Decoder / **CN:** decode 阶段r
- **EN:** Encoder / **CN:** Encoder
- **EN:** --disagg-role / **CN:** --disagg-role
- **EN:** Denoiser / **CN:** Denoiser
- **EN:** disagg-role / **CN:** disagg-role
- **EN:** Wan-AI / **CN:** Wan-AI

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
