# ascend_npu_support_features.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/platforms/ascend/ascend_npu_support_features.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This section describes the basic functions and features supported by the Ascend NPU.If you encounter issues or have any questions, please open an issue. If you want to know the meaning and usage of each parameter, click Server Arguments. / 该文档围绕 Support Features on Ascend NPU 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This section describes the basic functions and features supported by the Ascend NPU.If you encounter issues or have any questions, please open an issue. If you want to know the meaning and usage of each parameter, click Server Arguments.
**CN:** 本节围绕 Overview 展开，概述了 open, want, know, basic 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Model and tokenizer
**EN:** This section provides a comparison table for Model and tokenizer, covering columns such as Argument, Defaults, Options and examples such as --model-path<br/>--model, --tokenizer-path, --tokenizer-mode, --tokenizer-worker-num.
**CN:** 本节围绕 模型 and 令牌izer 展开，概述了 auto, Type, str, bool 等要点，并说明相关配置、流程、示例或限制条件。

### Section: HTTP server
**EN:** This section provides a comparison table for HTTP server, covering columns such as Argument, Defaults, Options and examples such as --host, --port, --skip-server-warmup, --warmups.
**CN:** 本节围绕 HTTP 服务端 展开，概述了 Type, str, int, --host 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Quantization and data type
**EN:** This section provides a comparison table for Quantization and data type, covering columns such as Argument, Defaults, Options and examples such as --dtype, --quantization, --quantization-param-path, --kv-cache-dtype.
**CN:** 本节围绕 量化 and data type 展开，概述了 Type, auto, GPU, Special 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Memory and scheduling
**EN:** This section provides a comparison table for Memory and scheduling, covering columns such as Argument, Defaults, Options and examples such as --mem-fraction-static, --max-running-requests, --prefill-max-requests, --max-queued-requests.
**CN:** 本节围绕 Memory and scheduling 展开，概述了 Type, br/, int, bool 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Runtime options
**EN:** This section provides a comparison table for Runtime options, covering columns such as Argument, Defaults, Options and examples such as --device, --tensor-parallel-size<br/>--tp-size, --pipeline-parallel-size<br/>--pp-size, --attention-context-parallel-size<br/>--attn-cp-size.
**CN:** 本节围绕 Runtime options 展开，概述了 Type, int, br/, Experimental 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Logging
**EN:** This section provides a comparison table for Logging, covering columns such as Argument, Defaults, Options and examples such as --log-level, --log-level-http, --log-requests, --log-requests-level.
**CN:** 本节围绕 Logging 展开，概述了 Type, br/, List, str 等要点，并说明相关配置、流程、示例或限制条件。

### Section: RequestMetricsExporter configuration
**EN:** This section provides a comparison table for RequestMetricsExporter configuration, covering columns such as Argument, Defaults, Options and examples such as --export-metrics-to-<br/>file, --export-metrics-to-<br/>file-dir.
**CN:** 本节围绕 Request指标Exporter 配置 展开，概述了 --export-metrics-to-, br/, Type, Server 等要点，并说明相关配置、流程、示例或限制条件。

### Section: API related
**EN:** This section provides a comparison table for API related, covering columns such as Argument, Defaults, Options and examples such as --api-key, --admin-api-key, --served-model-name, --weight-version.
**CN:** 本节围绕 API related 展开，概述了 Type, br/, str, model 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Data parallelism
**EN:** This section provides a comparison table for Data parallelism, covering columns such as Argument, Defaults, Options and examples such as --data-parallel-size<br/>--dp-size, --load-balance-method.
**CN:** 本节围绕 Data 并行策略 展开，概述了 br/, auto, Type, Server 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Multi-node distributed serving
**EN:** This section provides a comparison table for Multi-node distributed serving, covering columns such as Argument, Defaults, Options and examples such as --dist-init-addr<br/>--nccl-init-addr, --nnodes, --node-rank.
**CN:** 本节围绕 Multi-node distributed serving 展开，概述了 Type, int, Server, Options 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Model override args
**EN:** This section provides a comparison table for Model override args, covering columns such as Argument, Defaults, Options and examples such as --json-model-override-<br/>args, --preferred-sampling-<br/>params.
**CN:** 本节围绕 模型 override args 展开，概述了 Type, br/, str, args 等要点，并说明相关配置、流程、示例或限制条件。

### Section: LoRA
**EN:** This section provides a comparison table for LoRA, covering columns such as Argument, Defaults, Options and examples such as --enable-lora, --enable-lora-overlap-loading, --max-lora-rank, --lora-target-modules.
**CN:** 本节围绕 LoRA 展开，概述了 br/, Type, lru, Bool 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Kernel Backends (Attention, Sampling, Grammar, GEMM)
**EN:** This section provides a comparison table for Kernel Backends (Attention, Sampling, Grammar, GEMM), covering columns such as Argument, Defaults, Options and examples such as --attention-backend, --prefill-attention-backend, --decode-attention-backend, --sampling-backend.
**CN:** 本节围绕 Kernel Backends (Attention, Sampling, Grammar, GEMM) 展开，概述了 br/, GPU, ascend, Special 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Speculative decoding
**EN:** This section provides a comparison table for Speculative decoding, covering columns such as Argument, Defaults, Options and examples such as --speculative-algorithm, --speculative-draft-model-path<br/>--speculative-draft-model, --speculative-draft-model-<br/>revision, --speculative-draft-load-format.
**CN:** 本节围绕 投机解码 展开，概述了 Type, br/, auto, GPU 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Ngram speculative decoding
**EN:** This section provides a comparison table for Ngram speculative decoding, covering columns such as Argument, Defaults, Options and examples such as --speculative-ngram-<br/>min-match-window-size, --speculative-ngram-<br/>max-match-window-size, --speculative-ngram-<br/>min-bfs-breadth, --speculative-ngram-<br/>max-bfs-breadth.
**CN:** 本节围绕 Ngram 投机解码 展开，概述了 Experimental, Type, br/, --speculative-ngram- 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Expert parallelism
**EN:** This section provides a comparison table for Expert parallelism, covering columns such as Argument, Defaults, Options and examples such as --expert-parallel-size<br/>--ep-size<br/>--ep, --moe-a2a-backend, --moe-runner-backend, --flashinfer-mxfp4-<br/>moe-precision.
**CN:** 本节围绕 Expert 并行策略 展开，概述了 br/, Type, GPU, auto 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Mamba Cache
**EN:** This section provides a comparison table for Mamba Cache, covering columns such as Argument, Defaults, Options and examples such as --max-mamba-cache-size, --mamba-ssm-dtype, --mamba-full-memory-ratio, --mamba-scheduler-strategy.
**CN:** 本节围绕 Mamba 缓存 展开，概述了 Type, br/, auto, float32 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Hierarchical cache
**EN:** This section provides a comparison table for Hierarchical cache, covering columns such as Argument, Defaults, Options and examples such as --enable-hierarchical-<br/>cache, --hicache-ratio, --hicache-size, --hicache-write-policy.
**CN:** 本节围绕 Hierarchical 缓存 展开，概述了 br/, Type, --hicache-storage-, GPU 等要点，并说明相关配置、流程、示例或限制条件。

### Section: LMCache
**EN:** This section provides a comparison table for LMCache, covering columns such as Argument, Defaults, Options and examples such as --enable-lmcache.
**CN:** 本节围绕 LM缓存 展开，概述了 GPU, Server, Options, Special 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Offloading (must be used with --disable-cuda-graph)
**EN:** This section provides a comparison table for Offloading (must be used with --disable-cuda-graph), covering columns such as Argument, Defaults, Options and examples such as --cpu-offload-gb, --offload-group-size, --offload-num-in-group, --offload-prefetch-step.
**CN:** 本节围绕 Offloading (must be used with --disable-cuda-graph) 展开，概述了 DeepSeek, Type, int, cpu 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Args for multi-item scoring
**EN:** This section provides a comparison table for Args for multi-item scoring, covering columns such as Argument, Defaults, Options and examples such as --multi-item-scoring-delimiter.
**CN:** 本节围绕 Args for multi-item scoring 展开，概述了 Type, Server, Options, Argument 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Optimization/debug options
**EN:** This section provides a comparison table for Optimization/debug options, covering columns such as Argument, Defaults, Options and examples such as --disable-radix-cache, --cuda-graph-max-bs, --cuda-graph-bs, --disable-cuda-graph.
**CN:** 本节围绕 Optimization/debug options 展开，概述了 br/, bool, flag, enable 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Dynamic batch tokenizer
**EN:** This section provides a comparison table for Dynamic batch tokenizer, covering columns such as Argument, Defaults, Options and examples such as --enable-dynamic-<br/>batch-tokenizer, --dynamic-batch-<br/>tokenizer-batch-size, --dynamic-batch-<br/>tokenizer-batch-timeout.
**CN:** 本节围绕 Dynamic batch 令牌izer 展开，概述了 br/, Type, --dynamic-batch-, Server 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Debug tensor dumps
**EN:** This section provides a comparison table for Debug tensor dumps, covering columns such as Argument, Defaults, Options and examples such as --debug-tensor-dump-<br/>output-folder, --debug-tensor-dump-<br/>layers, --debug-tensor-dump-<br/>input-file.
**CN:** 本节围绕 Debug tensor dumps 展开，概述了 --debug-tensor-dump-, Type, br/, debug-tensor-dump- 等要点，并说明相关配置、流程、示例或限制条件。

### Section: PD disaggregation
**EN:** This section provides a comparison table for PD disaggregation, covering columns such as Argument, Defaults, Options and examples such as --disaggregation-mode, --disaggregation-transfer-backend, --disaggregation-bootstrap-port, --disaggregation-ib-device.
**CN:** 本节围绕 PD disaggregation 展开，概述了 Type, br/, --disaggregation-decode-, int 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Encode prefill disaggregation
**EN:** This section provides a comparison table for Encode prefill disaggregation, covering columns such as Argument, Defaults, Options and examples such as --enable-adaptive-dispatch-to-encoder, --encoder-only, --language-only, --encoder-transfer-backend.
**CN:** 本节围绕 Encode prefill 阶段 disaggregation 展开，概述了 br/, List, zmq_to_scheduler, bool 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Custom weight loader
**EN:** This section provides a comparison table for Custom weight loader, covering columns such as Argument, Defaults, Options and examples such as --custom-weight-loader, --weight-loader-disable-<br/>mmap, --remote-instance-weight-<br/>loader-seed-instance-ip, --remote-instance-weight-<br/>loader-seed-instance-service-port.
**CN:** 本节围绕 Custom weight loader 展开，概述了 br/, --remote-instance-weight-, Type, remote-instance-weight- 等要点，并说明相关配置、流程、示例或限制条件。

### Section: For PD-Multiplexing
**EN:** This section provides a comparison table for For PD-Multiplexing, covering columns such as Argument, Defaults, Options and examples such as --enable-pdmux, --pdmux-config-path, --sm-group-num.
**CN:** 本节围绕 For PD-Multiplexing 展开，概述了 GPU, Special, Type, Server 等要点，并说明相关配置、流程、示例或限制条件。

### Section: For Multi-Modal
**EN:** This section provides a comparison table for For Multi-Modal, covering columns such as Argument, Defaults, Options and examples such as --enable-broadcast-mm-<br/>inputs-process, --mm-process-config, --mm-enable-dp-encoder, --limit-mm-data-per-request.
**CN:** 本节围绕 For Multi-Modal 展开，概述了 Type, Dict, br/, bool 等要点，并说明相关配置、流程、示例或限制条件。

### Section: For checkpoint decryption
**EN:** This section provides a comparison table for For checkpoint decryption, covering columns such as Argument, Defaults, Options and examples such as --decrypted-config-file, --decrypted-draft-config-file, --enable-prefix-mm-cache.
**CN:** 本节围绕 For checkpoint decryption 展开，概述了 Type, str, Server, Options 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Forward hooks
**EN:** This section provides a comparison table for Forward hooks, covering columns such as Argument, Defaults, Options and examples such as --forward-hooks.
**CN:** 本节围绕 Forward hooks 展开，概述了 Type, Server, Options, Argument 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Configuration file support
**EN:** This section provides a comparison table for Configuration file support, covering columns such as Argument, Defaults, Options and examples such as --config.
**CN:** 本节围绕 配置 file support 展开，概述了 Type, Server, Options, --config 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Other Params
**EN:** This section provides a comparison table for Other Params, covering columns such as Argument, Defaults, Options and examples such as --checkpoint-engine- <br/> wait-weights- <br/> before-ready, --kt-weight-path, --kt-method, --kt-cpuinfer.
**CN:** 本节围绕 Other Params 展开，概述了 Type, int, Options, Argument 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** Type / **CN:** Type
- **EN:** br/ / **CN:** br/
- **EN:** flag / **CN:** flag
- **EN:** bool / **CN:** bool
- **EN:** enable / **CN:** enable
- **EN:** GPU / **CN:** GPU
- **EN:** Special / **CN:** Special
- **EN:** int / **CN:** int

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
