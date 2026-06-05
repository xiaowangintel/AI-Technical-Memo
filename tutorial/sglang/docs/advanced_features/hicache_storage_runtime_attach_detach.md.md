# hicache_storage_runtime_attach_detach.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/hicache_storage_runtime_attach_detach.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This document explains how to **dynamically attach/detach the HiCache L3 storage backend at runtime** (e.g., mooncake / hf3fs / nixl / file / aibrix / eic) while **SGLang is already running and serving traffic**, without restarting the process. / 该文档围绕 Runtime Attach/Detach Hi缓存 Storage Backend (No Restart) 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This document explains how to **dynamically attach/detach the HiCache L3 storage backend at runtime** (e.g., mooncake / hf3fs / nixl / file / aibrix / eic) while **SGLang is already running and serving traffic**, without restarting the process.
**CN:** 本节围绕 Overview 展开，概述了 eic, API, nixl, idle 等要点，并说明相关配置、流程、示例或限制条件。

### Section: 1.1 Architecture / control path
**EN:** The control path is: 1. **HTTP Server** (python/sglang/srt/entrypoints/http_server.py) - Exposes PUT /hicache/storage-backend, DELETE /hicache/storage-backend, GET /hicache/storage-backend 2.
**CN:** 本节围绕 1.1 Architecture / control path 展开，概述了 Calls, Scheduler, detach_storage_backend, backend 等要点，并说明相关配置、流程、示例或限制条件。

### Section: 2. Idle-state requirement (strict)
**EN:** > Tip: before switching, drain upstream traffic and wait for the server to become idle, then call attach/detach.
**CN:** 本节围绕 2. Idle-state requirement (strict) 展开，概述了 Tip, DLLM, idle, Reject 等要点，并说明相关配置、流程、示例或限制条件。

### Section: 2.1 DP (data parallel) semantics
**EN:** Operationally: - Prefer to keep backend config identical across ranks - If attach fails, immediately call detach (best-effort/idempotent), fix config, then retry attach
**CN:** 本节围绕 2.1 DP (data parallel) semantics 展开，概述了 ranks, success, see, TODO 等要点，并说明相关配置、流程、示例或限制条件。

### Section: 3. How to use (HTTP Admin API)
**EN:** The examples below assume your SGLang HTTP server is at http://127.0.0.1:30000.
**CN:** 本节围绕 3. How to use (HTTP Admin API) 展开，概述了 below, assume, server, SGLang HTTP 等要点，并说明相关配置、流程、示例或限制条件。

### Section: 3.1 Query current storage backend status
**EN:** ``bash curl -s http://127.0.0.1:30000/hicache/storage-backend ` Example response: `json " } ``
**CN:** 本节围绕 3.1 Query current storage backend status 展开，概述了 curl, response, mooncake, 

Example response:

 等要点，并说明相关配置、流程、示例或限制条件。

### Section: 3.2 Attach (enable) a storage backend
**EN:** This content focuses on 3.2 Attach (enable) a storage backend and highlights PUT, Content-Type, prefetch_threshold, curl.
**CN:** 本节围绕 3.2 Attach (enable) a storage backend 展开，概述了 PUT, Content-Type, prefetch_threshold, curl 等要点，并说明相关配置、流程、示例或限制条件。

### Section: 3.3 Detach (disable) the storage backend
**EN:** ``bash curl -s -X DELETE http://127.0.0.1:30000/hicache/storage-backend `` Notes: - Detach only makes SGLang **stop using** the L3 storage backend and stops prefetch/backup threads - It **does not automatically delete** data stored in Mooncake/HF3FS (or other remote backends)
**CN:** 本节围绕 3.3 Detach (disable) the storage backend 展开，概述了 Notes, DELETE, Detach, curl 等要点，并说明相关配置、流程、示例或限制条件。

### Section: 4. Behavior and caveats
**EN:** This content focuses on 4. Behavior and caveats and highlights attach, HiRadixCache, Must, Host.
**CN:** 本节围绕 4. Behavior and caveats 展开，概述了 attach, HiRadixCache, Must, Host 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** backend / **CN:** backend
- **EN:** storage / **CN:** storage
- **EN:** PUT / **CN:** PUT
- **EN:** idle / **CN:** idle
- **EN:** attach / **CN:** attach
- **EN:** ranks / **CN:** ranks
- **EN:** HiCache / **CN:** Hi缓存
- **EN:** mooncake / **CN:** mooncake

## Dependencies / 依赖关系
- `python/sglang/srt/entrypoints/http_server.py`
- `python/sglang/srt/managers/tokenizer_control_mixin.py`
- `python/sglang/srt/managers/scheduler.py`
- `python/sglang/srt/mem_cache/hiradix_cache.py`
- `python/sglang/srt/managers/cache_controller.py`
