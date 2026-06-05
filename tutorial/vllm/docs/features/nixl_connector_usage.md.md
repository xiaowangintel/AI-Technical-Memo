# nixl_connector_usage.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/nixl_connector_usage.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** This guide presents NixlConnector as a high-performance, fully asynchronous KV cache transfer connector for disaggregated prefilling. It covers installation across Nvidia, ROCm, and non-CUDA environments, including `uv pip install nixl`, ROCm/RIXL notes, and a source-install script for other platforms.
- **CN:** 本指南将 NixlConnector 定位为面向解耦 prefill 的高性能、全异步 KV cache 传输连接器。它覆盖了 Nvidia、ROCm 与非 CUDA 环境下的安装方式，包括 `uv pip install nixl`、ROCm / RIXL 说明，以及其他平台的源码安装脚本。
- **EN:** Transport configuration is a major theme. The doc recommends UCX environment variables such as `UCX_TLS` and `UCX_NET_DEVICES`, warns that NCCL variables do not apply, and shows how to choose alternate NIXL backends like `LIBFABRIC` through `kv_connector_extra_config.backends`.
- **CN:** 传输配置是文档重点。原文建议使用 `UCX_TLS`、`UCX_NET_DEVICES` 等 UCX 环境变量，提醒用户不要沿用 NCCL 变量，并展示了如何通过 `kv_connector_extra_config.backends` 选择 `LIBFABRIC` 等替代后端。
- **EN:** The usage path is concrete: start prefiller and decoder instances with side-channel ports, place a proxy in front, and optionally scale to multi-host setups. Important operational nuances include lease/TTL settings, the note that `kv_role` is effectively a placeholder for NixlConnector, failure-policy trade-offs (`fail` vs `recompute`), and experimental features like heterogeneous KV layout or cross-layer blocks.
- **CN:** 使用流程很具体：为 prefiller 和 decoder 启动带 side-channel 端口的实例，在前面放置代理，并可扩展到多机部署。文档中的关键运维细节包括 lease / TTL 配置、`kv_role` 对 NixlConnector 实际上只是占位符的说明、失败策略（`fail` vs `recompute`）的取舍，以及异构 KV 布局、cross-layer blocks 等实验特性。

## Key Concepts / 关键概念
- **EN:** NixlConnector focuses on asynchronous KV transfer for disaggregated prefill/decode.  
  **CN:** NixlConnector 专注于解耦 prefill / decode 场景下的异步 KV 传输。
- **EN:** UCX configuration matters more than NCCL variables for transport setup.  
  **CN:** 在传输配置中，UCX 变量比 NCCL 变量更关键。
- **EN:** `VLLM_NIXL_SIDE_CHANNEL_PORT` is required for handshake coordination on both sides.  
  **CN:** `VLLM_NIXL_SIDE_CHANNEL_PORT` 是两端握手协同所必需的关键配置。
- **EN:** Failure handling and experimental layout options can materially affect production behavior.  
  **CN:** 失败处理策略与实验性布局选项会显著影响生产环境行为。
