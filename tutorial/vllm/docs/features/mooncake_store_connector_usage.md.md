# mooncake_store_connector_usage.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/mooncake_store_connector_usage.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This doc presents MooncakeStoreConnector as a store-backed KV cache connector using MooncakeDistributedStore. Unlike direct `MooncakeConnector` transfer, it adds shared-pool semantics for CPU/disk offloading, cross-instance prefix caching, and both single-node and multi-node deployments.
- **CN:** 本文档将 MooncakeStoreConnector 定位为基于 MooncakeDistributedStore 的存储型 KV cache 连接器。与直接点对点传输的 `MooncakeConnector` 不同，它提供共享缓存池语义，支持 CPU / 磁盘 offload、跨实例前缀缓存，以及单机 / 多机部署。
- **EN:** The setup path is more involved: install Mooncake, start `mooncake_master`, provide a JSON config via `MOONCAKE_CONFIG_PATH`, and choose between `embedded` and `standalone-store` modes. The doc also explains how `global_segment_size`, `local_buffer_size`, `protocol`, and `enable_offload` affect deployment.
- **CN:** 其配置流程更复杂：先安装 Mooncake，启动 `mooncake_master`，通过 `MOONCAKE_CONFIG_PATH` 提供 JSON 配置，并在 `embedded` 与 `standalone-store` 模式间做选择。文档还解释了 `global_segment_size`、`local_buffer_size`、`protocol` 与 `enable_offload` 对部署的影响。
- **EN:** A strong integration point is the XpYd example: `MultiConnector` combines `MooncakeConnector` for direct P2P transfer with `MooncakeStoreConnector` for shared prefix-cache storage. The doc additionally stresses end-to-end alignment for disk offloading and notes that cross-DP prefix hits need a fixed `PYTHONHASHSEED`.
- **CN:** 一个关键集成点是 XpYd 示例：`MultiConnector` 同时组合 `MooncakeConnector`（负责直接 P2P 传输）与 `MooncakeStoreConnector`（负责共享前缀缓存存储）。文档还强调磁盘 offload 需要端到端配置一致，并指出跨 DP 的前缀缓存命中需要固定 `PYTHONHASHSEED`。

## Key Concepts / 关键概念
- **EN:** MooncakeStoreConnector extends KV transfer into a shared external cache/store layer.  
  **CN:** MooncakeStoreConnector 将 KV 传输能力扩展到共享的外部缓存 / 存储层。
- **EN:** `embedded` and `standalone-store` modes define whether storage ownership is in-process or external.  
  **CN:** `embedded` 与 `standalone-store` 模式决定存储由进程内还是外部组件持有。
- **EN:** `MultiConnector` is the documented way to combine direct transfer with store-backed sharing.  
  **CN:** `MultiConnector` 是文档推荐的“直连传输 + 存储共享”组合方式。
- **EN:** Disk offload only works when master, client, and vLLM-side settings are aligned.  
  **CN:** 只有当 master、client 和 vLLM 侧配置一致时，磁盘 offload 才能真正生效。
