# mooncake_store_connector_usage.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/mooncake_store_connector_usage.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This guide covers *MooncakeStoreConnector*, a KV cache connector that uses MooncakeDistributedStore as a shared pool. Unlike point-to-point MooncakeConnector, it enables CPU/disk offloading and cross-instance prefix cache sharing via hash-based deduplication.

**CN:** 本指南介绍 *MooncakeStoreConnector*，一种使用 MooncakeDistributedStore 作为共享池的 KV 缓存连接器。与点对点的 MooncakeConnector 不同，它支持 CPU/磁盘卸载，以及通过基于哈希的去重实现跨实例前缀缓存共享。

### Prerequisites / 前置条件

**EN:** Install `mooncake-transfer-engine`. Start a Mooncake master server (`mooncake_master --port 50051`) before launching vLLM. Multiple vLLM instances can share one master. Set `MOONCAKE_CONFIG_PATH` pointing to a JSON config file.

**CN:** 安装 `mooncake-transfer-engine`。在启动 vLLM 之前先运行 Mooncake 主服务（`mooncake_master --port 50051`）。多个 vLLM 实例可共享一个主服务。设置 `MOONCAKE_CONFIG_PATH` 指向 JSON 配置文件。

### Configuration File Fields / 配置文件字段

**EN:** `mode` (`"embedded"` or `"standalone-store"`), `protocol` (`"rdma"` or `"tcp"`), `global_segment_size` (CPU memory contributed per GPU), `local_buffer_size` (private per-node buffer), `enable_offload` (enables SSD-write staging buffer for large prefills).

**CN:** `mode`（`"embedded"` 或 `"standalone-store"`）、`protocol`（`"rdma"` 或 `"tcp"`）、`global_segment_size`（每 GPU 贡献的 CPU 内存）、`local_buffer_size`（节点私有缓冲区）、`enable_offload`（为大型预填充启用 SSD 写入暂存区）。

### Use Cases / 使用场景

**EN:** (1) Single-node KV offloading to extend cache capacity beyond GPU VRAM. (2) Disaggregated prefill-decode (XpYd) — combine with `MooncakeConnector` via `MultiConnector` for both P2P transfer and cross-instance prefix sharing.

**CN:** (1) 单节点 KV 卸载——将缓存容量扩展至 GPU 显存之外。(2) 解耦预填充-解码（XpYd）——通过 `MultiConnector` 与 `MooncakeConnector` 组合，同时支持 P2P 传输和跨实例前缀缓存共享。

### Deployment Modes / 部署模式

**EN:** `embedded` — each vLLM rank contributes `global_segment_size` to the pool in-process. `standalone-store` — ranks are pure requesters; an external `mooncake_client` process owns the CPU pool and optional SSD tier.

**CN:** `embedded`——每个 vLLM rank 在进程内向池中贡献 `global_segment_size` 内存。`standalone-store`——rank 仅作为请求方，外部 `mooncake_client` 进程拥有 CPU 池及可选 SSD 层。

## Key Concepts / 关键概念

- **EN:** Shared KV store — multiple vLLM instances access a common distributed KV pool, enabling cross-instance prefix cache hits. **CN:** 共享 KV 存储——多个 vLLM 实例访问共同的分布式 KV 池，实现跨实例前缀缓存命中。
- **EN:** Hash-based deduplication — identical KV blocks (same prefix) are stored once and reused by any instance. **CN:** 基于哈希的去重——相同的 KV 块（相同前缀）只存储一次，可被任意实例复用。
- **EN:** `enable_offload` — extends effective KV capacity by staging large prefill blocks to SSD; requires matching flags on master and client. **CN:** `enable_offload`——通过将大型预填充块暂存到 SSD 扩展有效 KV 容量，需在主服务和客户端同步开启。
- **EN:** `MultiConnector` — combines MooncakeConnector (P2P) and MooncakeStoreConnector (shared pool) for full disaggregated serving. **CN:** `MultiConnector`——将 MooncakeConnector（P2P）和 MooncakeStoreConnector（共享池）结合，实现完整的解耦服务。
- **EN:** Mooncake master — the metadata coordination server; must be started before any vLLM instance using this connector. **CN:** Mooncake 主服务——元数据协调服务器，必须在任何使用此连接器的 vLLM 实例启动前先运行。
