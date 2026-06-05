# disagg_encoder.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/disagg_encoder.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document covers *disaggregated encoder* deployment, a vLLM pattern for multimodal LLMs where the vision-encoder stage runs in a separate vLLM process from the prefill/decode stage. It explains motivation, usage examples, key abstractions, and development guidance.

**CN:** 本文档介绍*解耦编码器*部署模式，即对多模态 LLM 将视觉编码器阶段与预填充/解码阶段分别运行在独立的 vLLM 进程中，内容包含动机、使用示例、关键抽象和开发指南。

### Benefits / 优点

**EN:** Three main benefits: (1) Independent fine-grained scaling — encoder fleet can scale separately from the LM; (2) Lower TTFT — language-only requests bypass the encoder entirely; (3) Cross-process reuse — a shared remote cache eliminates redundant encoding for repeated images.

**CN:** 三大优点：(1) 独立精细扩缩容——编码器集群可独立于语言模型扩缩容；(2) 更低 TTFT——纯文本请求完全跳过编码器；(3) 跨进程复用——共享远程缓存消除重复图像的冗余编码。

### Architecture / 架构

**EN:** Two process types: **Encoder instance** (vision encoding only) and **PD instance(s)** (language prefill + decode). A connector (`ECConnector`) transfers encoder-cache (EC) embeddings between them. EC transfer code lives in `vllm/distributed/ec_transfer`.

**CN:** 两种进程类型：**编码器实例**（仅负责视觉编码）和 **PD 实例**（语言预填充 + 解码）。连接器（`ECConnector`）在两者之间传输编码器缓存（EC）嵌入，相关代码位于 `vllm/distributed/ec_transfer`。

### Deployment Configurations / 部署配置

**EN:** Two reference scripts: `disagg_1e1pd_example.sh` (1 encoder + 1 PD) and `disagg_1e1p1d_example.sh` (1 encoder + 1 prefill + 1 decode, fully disaggregated).

**CN:** 两个参考脚本：`disagg_1e1pd_example.sh`（1 编码器 + 1 PD）和 `disagg_1e1p1d_example.sh`（1 编码器 + 1 预填充 + 1 解码，完全解耦）。

### ECConnector / EC 连接器

**EN:** The `ECConnector` interface has two roles: **Scheduler role** (checks cache existence, schedules loads) and **Worker role** (loads embeddings into memory). The reference implementation is `ExampleConnector`; production setups use `NixlConnector`.

**CN:** `ECConnector` 接口承担两个角色：**调度器角色**（检查缓存是否存在并调度加载）和 **Worker 角色**（将嵌入加载到内存中）。参考实现为 `ExampleConnector`，生产环境推荐使用 `NixlConnector`。

## Key Concepts / 关键概念

- **EN:** Disaggregated encoder — splits vision encoding and language inference into separate vLLM processes. **CN:** 解耦编码器——将视觉编码与语言推理拆分为独立的 vLLM 进程。
- **EN:** EC (Encoder Cache) embeddings — the output of the vision encoder that is transferred to the PD instance. **CN:** EC（编码器缓存）嵌入——视觉编码器的输出，传输至 PD 实例。
- **EN:** `ECConnector` — abstraction for retrieving EC caches; used by both scheduler and worker. **CN:** `ECConnector`——检索 EC 缓存的抽象接口，调度器和 Worker 均使用。
- **EN:** TTFT reduction — language-only requests avoid the encoder critical path entirely. **CN:** TTFT 降低——纯文本请求完全绕开编码器关键路径。
- **EN:** E→P→D pipeline — encoder feeds prefill instance which then KV-transfers to a decode instance, combining both disaggregated encoder and disaggregated prefill. **CN:** E→P→D 流水线——编码器为预填充实例提供数据，再 KV 转移至解码实例，将解耦编码器与解耦预填充结合。
