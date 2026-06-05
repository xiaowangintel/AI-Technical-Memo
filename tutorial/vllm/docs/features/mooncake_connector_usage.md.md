# mooncake_connector_usage.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/mooncake_connector_usage.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This guide explains MooncakeConnector as a KV-transfer backend for disaggregated serving. It begins with Mooncake’s purpose—improving LLM inference efficiency, especially with slow object storage, using multi-level caching over DRAM/SSD plus RDMA-based zero-copy transfer.
- **CN:** 本指南将 MooncakeConnector 说明为解耦服务中的 KV 传输后端。开头先介绍 Mooncake 本身的目标：在慢速对象存储环境下，通过基于 DRAM / SSD 的多级缓存与 RDMA 零拷贝传输提升 LLM 推理效率。
- **EN:** The usage section is deployment-oriented: a prefiller node runs `kv_producer`, a decoder node runs `kv_consumer`, and a proxy process routes requests between them. The examples use ports 8010/8020 and a dedicated `mooncake_connector_proxy.py` script.
- **CN:** 使用部分完全以部署为中心：prefiller 节点运行 `kv_producer`，decoder 节点运行 `kv_consumer`，再由代理进程在两者间转发请求。文档示例使用 8010 / 8020 端口和专门的 `mooncake_connector_proxy.py` 脚本。
- **EN:** The integration details include operational knobs such as `VLLM_MOONCAKE_BOOTSTRAP_PORT`, abort timeout handling, KV role choices, and extra config for worker count and protocol (`rdma` by default).
- **CN:** 集成细节包含若干运维参数，例如 `VLLM_MOONCAKE_BOOTSTRAP_PORT`、请求中止后的超时释放机制、KV 角色选项，以及用于配置 worker 数量和协议（默认 `rdma`）的扩展参数。

## Key Concepts / 关键概念
- **EN:** MooncakeConnector links prefill and decode through transferred KV caches.  
  **CN:** MooncakeConnector 通过传输 KV cache 连接 prefill 与 decode。
- **EN:** The documented setup requires three parts: prefiller, decoder, and proxy.  
  **CN:** 文档给出的标准部署包含三部分：prefiller、decoder 和 proxy。
- **EN:** Bootstrap port handling is an explicit requirement for prefiller-side coordination.  
  **CN:** bootstrap 端口配置是 prefiller 侧协同启动中的明确要求。
- **EN:** `kv_connector_extra_config` mainly tunes worker concurrency and protocol selection.  
  **CN:** `kv_connector_extra_config` 主要用于调节 worker 并发和协议选择。
