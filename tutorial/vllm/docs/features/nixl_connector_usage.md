# nixl_connector_usage.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/nixl_connector_usage.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This guide describes how to install, configure, and run the *NixlConnector* — vLLM's high-performance, fully asynchronous KV cache transfer connector for disaggregated prefilling. It covers installation on NVIDIA, ROCm, and non-CUDA platforms, transport configuration, basic single-host setup, environment variables, multi-machine deployments, and tensor/data parallel configurations.

**CN:** 本指南介绍如何安装、配置和运行 *NixlConnector*——vLLM 用于解耦预填充的高性能、完全异步 KV 缓存传输连接器。内容涵盖在 NVIDIA、ROCm 和非 CUDA 平台上的安装、传输配置、单机基础部署、环境变量、多机部署，以及张量/数据并行配置。

### Installation / 安装

**EN:** NVIDIA: `uv pip install nixl`. ROCm: RIXL is pre-included in the ROCm Docker image. Non-CUDA: build UCX from source via `python tools/install_nixl_from_source_ubuntu.py`.

**CN:** NVIDIA：`uv pip install nixl`。ROCm：RIXL 已预置在 ROCm Docker 镜像中。非 CUDA：通过 `python tools/install_nixl_from_source_ubuntu.py` 从源码构建 UCX。

### Basic Single-Host Setup / 单机基础部署

**EN:** Launch prefiller on GPU 0 with `VLLM_NIXL_SIDE_CHANNEL_PORT=5600` and `kv_role: "kv_both"`, decoder on GPU 1 with port 5601. Use a proxy server to route requests between them.

**CN:** 在 GPU 0 上以 `VLLM_NIXL_SIDE_CHANNEL_PORT=5600` 和 `kv_role: "kv_both"` 启动预填充实例，在 GPU 1 上以端口 5601 启动解码实例，通过代理服务器在两者之间路由请求。

### Key Environment Variables / 关键环境变量

**EN:** `VLLM_NIXL_SIDE_CHANNEL_PORT` (default 5600) — required on both P and D; unique per host. In DP deployments each worker uses `base_port + dp_rank`. `VLLM_NIXL_SIDE_CHANNEL_HOST` — set when P and D are on different machines.

**CN:** `VLLM_NIXL_SIDE_CHANNEL_PORT`（默认 5600）——P 和 D 均需要设置，同一主机上须唯一。DP 部署中每个 worker 使用 `base_port + dp_rank`。`VLLM_NIXL_SIDE_CHANNEL_HOST`——P 和 D 在不同机器上时设置。

### Lease and TTL Configuration / 租约与 TTL 配置

**EN:** `kv_lease_duration` (default 30 s) — how long prefiller holds KV blocks for a finished request while awaiting the decoder; extended automatically by heartbeats. `decoder_kv_blocks_ttl` (default 480 s) — TTL for KV blocks cached on decoder in bidirectional mode.

**CN:** `kv_lease_duration`（默认 30 秒）——预填充实例为已完成请求保留 KV 块等待解码器的时长，通过心跳自动延续。`decoder_kv_blocks_ttl`（默认 480 秒）——双向模式下解码端缓存 KV 块的生存时间。

### Backend Selection / 后端选择

**EN:** Default backend is UCX. Select alternatives (LIBFABRIC, GDS, etc.) via `kv_connector_extra_config.backends`. Backend availability depends on NIXL build configuration.

**CN:** 默认后端为 UCX。通过 `kv_connector_extra_config.backends` 选择其他后端（LIBFABRIC、GDS 等），可用性取决于 NIXL 的构建配置。

## Key Concepts / 关键概念

- **EN:** NIXL — asynchronous KV cache transfer library with pluggable transport backends (UCX, GDS, LIBFABRIC). **CN:** NIXL——具有可插拔传输后端（UCX、GDS、LIBFABRIC）的异步 KV 缓存传输库。
- **EN:** Side channel port — used for initial P↔D handshake; separate from the main vLLM HTTP port. **CN:** 旁路通道端口——用于 P↔D 初始握手，与 vLLM 主 HTTP 端口分离。
- **EN:** `kv_role` — `kv_producer` (prefiller), `kv_consumer` (decoder), or `kv_both` for flexible bidirectional setups. **CN:** `kv_role`——`kv_producer`（预填充）、`kv_consumer`（解码）或 `kv_both`（灵活双向部署）。
- **EN:** `kv_load_failure_policy: "fail"` — ensures requests fail explicitly if KV transfer fails, rather than silently falling back. **CN:** `kv_load_failure_policy: "fail"`——确保 KV 传输失败时请求显式报错，而非静默回退。
- **EN:** UCX vs NCCL — for NixlConnector, configure UCX environment variables (not NCCL variables) for network device and transport selection. **CN:** UCX 与 NCCL——使用 NixlConnector 时应配置 UCX 环境变量（而非 NCCL 变量）来选择网络设备和传输方式。
