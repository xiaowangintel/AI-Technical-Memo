# mooncake_connector_usage.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/mooncake_connector_usage.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This guide describes how to configure and use the *MooncakeConnector* for disaggregated prefill-decode serving in vLLM. Mooncake is a multi-level KV cache system that uses RDMA (including GPUDirect RDMA) for zero-copy, high-bandwidth data transfer between prefiller and decoder nodes.

**CN:** 本指南介绍如何在 vLLM 中配置和使用 *MooncakeConnector* 进行解耦预填充-解码服务。Mooncake 是一个多级 KV 缓存系统，使用 RDMA（包括 GPUDirect RDMA）在预填充和解码节点之间进行零拷贝、高带宽的数据传输。

### Setup / 安装部署

**EN:** Install via `uv pip install mooncake-transfer-engine`. Start prefiller with `kv_role: "kv_producer"` and decoder with `kv_role: "kv_consumer"`, then run a proxy script to route requests.

**CN:** 通过 `uv pip install mooncake-transfer-engine` 安装。以 `kv_role: "kv_producer"` 启动预填充节点，以 `kv_role: "kv_consumer"` 启动解码节点，然后运行代理脚本路由请求。

### Environment Variables / 环境变量

**EN:** `VLLM_MOONCAKE_BOOTSTRAP_PORT` (default 8998, required on prefiller, must be unique per host) controls the Mooncake bootstrap server port. `VLLM_MOONCAKE_ABORT_REQUEST_TIMEOUT` (default 480 s) sets the timeout before KV blocks are released for aborted requests.

**CN:** `VLLM_MOONCAKE_BOOTSTRAP_PORT`（默认 8998，预填充端必填，同一主机上须唯一）控制 Mooncake 引导服务端口。`VLLM_MOONCAKE_ABORT_REQUEST_TIMEOUT`（默认 480 秒）设置请求中断后 KV 块释放前的超时时长。

### KV Transfer Config / KV 传输配置

**EN:** `kv_role` options: `kv_producer`, `kv_consumer`, `kv_both`. Extra config: `num_workers` (thread pool size per worker, default 10) and `mooncake_protocol` (default `"rdma"`).

**CN:** `kv_role` 选项：`kv_producer`、`kv_consumer`、`kv_both`。额外配置：`num_workers`（每 worker 线程池大小，默认 10）和 `mooncake_protocol`（默认 `"rdma"`）。

### Key Differentiator vs MooncakeStoreConnector / 与 MooncakeStoreConnector 的主要区别

**EN:** MooncakeConnector performs direct point-to-point KV transfer between a specific prefiller and decoder pair, while MooncakeStoreConnector offloads KV caches to a shared distributed store enabling cross-instance prefix cache sharing.

**CN:** MooncakeConnector 在特定预填充-解码对之间执行直接点对点 KV 传输，而 MooncakeStoreConnector 将 KV 缓存卸载到共享分布式存储，支持跨实例前缀缓存共享。

## Key Concepts / 关键概念

- **EN:** Mooncake — multi-level KV cache pool on DRAM/SSD with GPUDirect RDMA for zero-copy transfer. **CN:** Mooncake——基于 DRAM/SSD 的多级 KV 缓存池，利用 GPUDirect RDMA 实现零拷贝传输。
- **EN:** `kv_producer` / `kv_consumer` — role assignment for prefiller and decoder respectively. **CN:** `kv_producer` / `kv_consumer`——分别分配给预填充节点和解码节点的角色。
- **EN:** Bootstrap port — used for Mooncake's initial handshake; each host-level instance requires a unique port. **CN:** 引导端口——用于 Mooncake 的初始握手，每个主机级实例需要唯一端口。
- **EN:** Proxy server — routes client requests to the appropriate prefiller or decoder endpoint. **CN:** 代理服务器——将客户端请求路由到合适的预填充或解码端点。
- **EN:** `kv_both` — symmetric role allowing a single instance to act as both producer and consumer; useful for experimental setups. **CN:** `kv_both`——对称角色，允许单个实例同时充当生产者和消费者，适用于实验性部署。
