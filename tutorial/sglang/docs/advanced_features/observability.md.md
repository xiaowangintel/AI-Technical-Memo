# observability.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/observability.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Production Metrics SGLang exposes the following metrics via Prometheus. You can enable them by adding --enable-metrics when launching the server. / 该文档围绕 可观测性 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Production Metrics
**EN:** SGLang exposes the following metrics via Prometheus. You can enable them by adding --enable-metrics when launching the server.
**CN:** 本节围绕 Production 指标 展开，概述了 See, Production, --enable-metrics, via 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Logging
**EN:** By default, SGLang does not log any request contents. You can log them by using --log-requests.
**CN:** 本节围绕 Logging 展开，概述了 See, log, Logging, --log-requests 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Request Dump and Replay
**EN:** You can dump all requests and replay them later for benchmarking or other purposes.
**CN:** 本节围绕 Request Dump and Replay 展开，概述了 dump, requests, replay, server 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Crash Dump and Replay
**EN:** Sometimes the server might crash, and you may want to debug the cause of the crash. SGLang supports crash dumping, which will dump all requests from the 5 minutes before the crash, allowing you to replay the requests and debug the reason later.
**CN:** 本节围绕 Crash Dump and Replay 展开，概述了 crash, dump, debug, replay 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** crash / **CN:** crash
- **EN:** dump / **CN:** dump
- **EN:** requests / **CN:** requests
- **EN:** See / **CN:** See
- **EN:** server / **CN:** 服务端
- **EN:** replay / **CN:** replay
- **EN:** Logging / **CN:** Logging
- **EN:** scripts/playground/replay_request_dump.py / **CN:** scripts/playground/replay_request_dump.py

## Dependencies / 依赖关系
- `../references/production_metrics.md`
- `../references/production_request_trace.md`
- `server_arguments.md`
- `scripts/playground/replay_request_dump.py`
