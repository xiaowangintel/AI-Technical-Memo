# disagg_prefill.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/disagg_prefill.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This page introduces experimental disaggregated prefilling, where prefill and decode run in different vLLM instances. The source stresses two motivations: tuning TTFT and ITL separately, and controlling tail ITL by avoiding prefill work interrupting decode.
- **CN:** 本页介绍实验性的解耦式 prefilling：prefill 与 decode 在不同 vLLM 实例中运行。原文强调两个动机：分别调优 TTFT 与 ITL，以及避免 prefill 插入 decode 过程从而控制尾部 ITL。
- **EN:** A key caveat is explicit in the doc: disaggregated prefill does **not** improve throughput. The usage section is connector-focused and lists ExampleConnector, LMCacheConnectorV1, NixlConnector, P2pNcclConnector, MooncakeConnector, MultiConnector, OffloadingConnector, and FlexKVConnectorV1, with example `--kv-transfer-config` snippets.
- **CN:** 文档明确给出一个重要限制：解耦 prefill **不会**提升吞吐。使用部分高度围绕连接器展开，列出了 ExampleConnector、LMCacheConnectorV1、NixlConnector、P2pNcclConnector、MooncakeConnector、MultiConnector、OffloadingConnector 和 FlexKVConnectorV1，并给出了 `--kv-transfer-config` 示例。
- **EN:** The development section grounds the workflow in three abstractions—`Connector`, `LookupBuffer`, and `Pipe`—plus scheduler/worker connectors. It also notes that third-party connectors are the recommended path for production-level disaggregated prefilling.
- **CN:** 开发部分把工作流落到三个核心抽象：`Connector`、`LookupBuffer` 与 `Pipe`，并补充了 scheduler / worker connector 的组织方式。文档还指出，生产级解耦 prefill 更依赖第三方连接器实现。

## Key Concepts / 关键概念
- **EN:** Prefill and decode are split so TTFT and ITL can be tuned independently.  
  **CN:** prefill 与 decode 被拆分，以便独立调优 TTFT 和 ITL。
- **EN:** The doc explicitly positions tail-latency control—not throughput gain—as the main benefit.  
  **CN:** 文档明确将“控制尾延迟”而非“提升吞吐”作为主要收益。
- **EN:** `LookupBuffer` exposes `insert` and blocking `drop_select` semantics similar to SQL.  
  **CN:** `LookupBuffer` 提供类似 SQL 语义的 `insert` 和阻塞式 `drop_select` 接口。
- **EN:** Production deployments are expected to rely on connector ecosystems beyond the built-in abstractions.  
  **CN:** 生产部署通常需要依赖这些抽象之外的第三方连接器生态。
