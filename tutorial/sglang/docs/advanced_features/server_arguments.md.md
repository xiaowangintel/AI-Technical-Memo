# server_arguments.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/server_arguments.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This page provides a list of server arguments used in the command line to configure the behavior and performance of the language model server during deployment. / 该文档围绕 服务端 Arguments 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This page provides a list of server arguments used in the command line to configure the behavior and performance of the language model server during deployment.
**CN:** 本节围绕 Overview 展开，概述了 server, arguments, These, model 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Common launch commands
**EN:** To use a configuration file, create a YAML file with your server arguments and specify it with --config. CLI arguments will override config file values.
**CN:** 本节围绕 Common launch commands 展开，概述了 add, model-path, Meta-Llama-3-8B-Instruct, meta-llama/Meta-Llama-3-8B-Instruct 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Model and tokenizer
**EN:** This section provides a comparison table for Model and tokenizer, covering columns such as Argument, Description, Defaults and examples such as --model-path<br>--model, --tokenizer-path, --tokenizer-mode, --tokenizer-worker-num.
**CN:** 本节围绕 模型 and 令牌izer 展开，概述了 model, Type, auto, weights 等要点，并说明相关配置、流程、示例或限制条件。

### Section: HTTP server
**EN:** This section provides a comparison table for HTTP server, covering columns such as Argument, Description, Defaults and examples such as --host, --port, --fastapi-root-path, --grpc-mode.
**CN:** 本节围绕 HTTP 服务端 展开，概述了 Type, server, port, Defaults 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Quantization and data type
**EN:** This section provides a comparison table for Quantization and data type, covering columns such as Argument, Description, Defaults and examples such as --dtype, --quantization, --quantization-param-path, --kv-cache-dtype.
**CN:** 本节围绕 量化 and data type 展开，概述了 Type, auto, Path, quantization 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Memory and scheduling
**EN:** This section provides a comparison table for Memory and scheduling, covering columns such as Argument, Description, Defaults and examples such as --mem-fraction-static, --max-running-requests, --max-queued-requests, --max-total-tokens.
**CN:** 本节围绕 Memory and scheduling 展开，概述了 Type, prefill, int, float 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Runtime options
**EN:** This section provides a comparison table for Runtime options, covering columns such as Argument, Description, Defaults and examples such as --device, --tensor-parallel-size<br>--tp-size, --pipeline-parallel-size<br>--pp-size, --attention-context-parallel-size<br>--attn-cp-size.
**CN:** 本节围绕 Runtime options 展开，概述了 Type, int,  | Type: int |
| , size 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Logging
**EN:** This section provides a comparison table for Logging, covering columns such as Argument, Description, Defaults and examples such as --log-level, --log-level-http, --log-requests, --log-requests-level.
**CN:** 本节围绕 Logging 展开，概述了 Type, List, str, buckets 等要点，并说明相关配置、流程、示例或限制条件。

### Section: RequestMetricsExporter configuration
**EN:** This section provides a comparison table for RequestMetricsExporter configuration, covering columns such as Argument, Description, Defaults and examples such as --export-metrics-to-file, --export-metrics-to-file-dir.
**CN:** 本节围绕 Request指标Exporter 配置 展开，概述了 --export-metrics-to-file, Type, Export, Options 等要点，并说明相关配置、流程、示例或限制条件。

### Section: API related
**EN:** This section provides a comparison table for API related, covering columns such as Argument, Description, Defaults and examples such as --api-key, --admin-api-key, --served-model-name, --weight-version.
**CN:** 本节围绕 API related 展开，概述了 Type, model, str, API 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Data parallelism
**EN:** This section provides a comparison table for Data parallelism, covering columns such as Argument, Description, Defaults and examples such as --data-parallel-size<br>--dp-size, --load-balance-method.
**CN:** 本节围绕 Data 并行策略 展开，概述了 auto, load, total_tokens, Type 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Multi-node distributed serving
**EN:** This section provides a comparison table for Multi-node distributed serving, covering columns such as Argument, Description, Defaults and examples such as --dist-init-addr<br>--nccl-init-addr, --nnodes, --node-rank.
**CN:** 本节围绕 Multi-node distributed serving 展开，概述了 Type, int, Options, --nnodes 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Model override args
**EN:** This section provides a comparison table for Model override args, covering columns such as Argument, Description, Defaults and examples such as --json-model-override-args, --preferred-sampling-params.
**CN:** 本节围绕 模型 override args 展开，概述了 Type, str, Options, Argument 等要点，并说明相关配置、流程、示例或限制条件。

### Section: LoRA
**EN:** This section provides a comparison table for LoRA, covering columns such as Argument, Description, Defaults and examples such as --enable-lora, --enable-lora-overlap-loading, --max-lora-rank, --lora-target-modules.
**CN:** 本节围绕 LoRA 展开，概述了 LoRA, Type, adapters, --lora-paths 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Kernel Backends (Attention, Sampling, Grammar, GEMM)
**EN:** This section provides a comparison table for Kernel Backends (Attention, Sampling, Grammar, GEMM), covering columns such as Argument, Description, Defaults and examples such as --attention-backend, --prefill-attention-backend, --decode-attention-backend, --sampling-backend.
**CN:** 本节围绕 Kernel Backends (Attention, Sampling, Grammar, GEMM) 展开，概述了 Choose, fa3, aiter, backend 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Speculative decoding
**EN:** This section provides a comparison table for Speculative decoding, covering columns such as Argument, Description, Defaults and examples such as --speculative-algorithm, --speculative-draft-model-path<br>--speculative-draft-model, --speculative-draft-model-revision, --speculative-draft-load-format.
**CN:** 本节围绕 投机解码 展开，概述了 Type, Same, draft, model 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Ngram speculative decoding
**EN:** This section provides a comparison table for Ngram speculative decoding, covering columns such as Argument, Description, Defaults and examples such as --speculative-ngram-min-bfs-breadth, --speculative-ngram-max-bfs-breadth, --speculative-ngram-match-type, --speculative-ngram-max-trie-depth.
**CN:** 本节围绕 Ngram 投机解码 展开，概述了 BFS, Type, PROB, ngram 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Multi-layer Eagle speculative decoding
**EN:** This section provides a comparison table for Multi-layer Eagle speculative decoding, covering columns such as Argument, Description, Defaults and examples such as --enable-multi-layer-eagle.
**CN:** 本节围绕 Multi-layer Eagle 投机解码 展开，概述了 Eagle, Enable, Options, Argument 等要点，并说明相关配置、流程、示例或限制条件。

### Section: MoE
**EN:** This section provides a comparison table for MoE, covering columns such as Argument, Description, Defaults and examples such as --expert-parallel-size<br>--ep-size<br>--ep, --moe-a2a-backend, --moe-runner-backend, --flashinfer-mxfp4-moe-precision.
**CN:** 本节围绕 MoE 展开，概述了 Type, auto, expert, low_latency 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Mamba Cache
**EN:** This section provides a comparison table for Mamba Cache, covering columns such as Argument, Description, Defaults and examples such as --max-mamba-cache-size, --mamba-ssm-dtype, --mamba-full-memory-ratio, --mamba-scheduler-strategy.
**CN:** 本节围绕 Mamba 缓存 展开，概述了 mamba, extra_buffer, auto, Type 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Hierarchical cache
**EN:** This section provides a comparison table for Hierarchical cache, covering columns such as Argument, Description, Defaults and examples such as --enable-hierarchical-cache, --hicache-ratio, --hicache-size, --hicache-write-policy.
**CN:** 本节围绕 Hierarchical 缓存 展开，概述了 cache, Type, backend, pool 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Hierarchical sparse attention
**EN:** This section provides a comparison table for Hierarchical sparse attention, covering columns such as Argument, Description, Defaults and examples such as --hierarchical-sparse-attention-extra-config.
**CN:** 本节围绕 Hierarchical sparse attention 展开，概述了 str, algorithm, Type, fields 等要点，并说明相关配置、流程、示例或限制条件。

### Section: LMCache
**EN:** This section provides a comparison table for LMCache, covering columns such as Argument, Description, Defaults and examples such as --enable-lmcache.
**CN:** 本节围绕 LM缓存 展开，概述了 Options, Argument, Defaults, Description 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Ktransformers
**EN:** This section provides a comparison table for Ktransformers, covering columns such as Argument, Description, Defaults and examples such as --kt-weight-path, --kt-method, --kt-cpuinfer, --kt-threadpool-count.
**CN:** 本节围绕 Ktransformers 展开，概述了 Type, parameter, ktransformers, CPU 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Diffusion LLM
**EN:** This section provides a comparison table for Diffusion LLM, covering columns such as Argument, Description, Defaults and examples such as --dllm-algorithm, --dllm-algorithm-config.
**CN:** 本节围绕 Diffusion LLM 展开，概述了 LLM, Type, str, Must 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Offloading
**EN:** This section provides a comparison table for Offloading, covering columns such as Argument, Description, Defaults and examples such as --cpu-offload-gb, --offload-group-size, --offload-num-in-group, --offload-prefetch-step.
**CN:** 本节围绕 Offloading 展开，概述了 Type, int, Number, offloading 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Args for multi-item scoring
**EN:** This section provides a comparison table for Args for multi-item scoring, covering columns such as Argument, Description, Defaults and examples such as --multi-item-scoring-delimiter.
**CN:** 本节围绕 Args for multi-item scoring 展开，概述了 Query, delimiter, Type, Items 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Optimization/debug options
**EN:** This section provides a comparison table for Optimization/debug options, covering columns such as Argument, Description, Defaults and examples such as --disable-radix-cache, --cuda-graph-max-bs, --cuda-graph-bs, --disable-cuda-graph.
**CN:** 本节围绕 Optimization/debug options 展开，概述了 bool, flag, enable, Type 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Dynamic batch tokenizer
**EN:** This section provides a comparison table for Dynamic batch tokenizer, covering columns such as Argument, Description, Defaults and examples such as --enable-dynamic-batch-tokenizer, --dynamic-batch-tokenizer-batch-size, --dynamic-batch-tokenizer-batch-timeout.
**CN:** 本节围绕 Dynamic batch 令牌izer 展开，概述了 Type, --enable-dynamic-batch-tokenizer, batch, enable-dynamic-batch-tokenizer 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Debug tensor dumps
**EN:** This section provides a comparison table for Debug tensor dumps, covering columns such as Argument, Description, Defaults and examples such as --debug-tensor-dump-output-folder, --debug-tensor-dump-layers, --debug-tensor-dump-input-file, --debug-tensor-dump-inject.
**CN:** 本节围绕 Debug tensor dumps 展开，概述了 Type, str, Dump, layer 等要点，并说明相关配置、流程、示例或限制条件。

### Section: PD disaggregation
**EN:** This section provides a comparison table for PD disaggregation, covering columns such as Argument, Description, Defaults and examples such as --disaggregation-mode, --disaggregation-transfer-backend, --disaggregation-bootstrap-port, --disaggregation-ib-device.
**CN:** 本节围绕 PD disaggregation 展开，概述了 Type, decode, server, mooncake 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Encode prefill disaggregation
**EN:** This section provides a comparison table for Encode prefill disaggregation, covering columns such as Argument, Description, Defaults and examples such as --encoder-only, --language-only, --encoder-transfer-backend, --encoder-urls.
**CN:** 本节围绕 Encode prefill 阶段 disaggregation 展开，概述了 zmq_to_scheduler, encoder, List, Type 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Custom weight loader
**EN:** This section provides a comparison table for Custom weight loader, covering columns such as Argument, Description, Defaults and examples such as --custom-weight-loader, --weight-loader-disable-mmap, --weight-loader-prefetch-checkpoints, --weight-loader-prefetch-num-threads.
**CN:** 本节围绕 Custom weight loader 展开，概述了 Type, instance, nccl, loading 等要点，并说明相关配置、流程、示例或限制条件。

### Section: For PD-Multiplexing
**EN:** This section provides a comparison table for For PD-Multiplexing, covering columns such as Argument, Description, Defaults and examples such as --enable-pdmux, --pdmux-config-path, --sm-group-num.
**CN:** 本节围绕 For PD-Multiplexing 展开，概述了 Type, PD-Multiplexing, Number, Options 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Configuration file support
**EN:** This section provides a comparison table for Configuration file support, covering columns such as Argument, Description, Defaults and examples such as --config.
**CN:** 本节围绕 配置 file support 展开，概述了 Must, Type, config, Options 等要点，并说明相关配置、流程、示例或限制条件。

### Section: For Multi-Modal
**EN:** This section provides a comparison table for For Multi-Modal, covering columns such as Argument, Description, Defaults and examples such as --mm-max-concurrent-calls, --mm-per-request-timeout, --enable-broadcast-mm-inputs-process, --mm-process-config.
**CN:** 本节围绕 For Multi-Modal 展开，概述了 Type, Dict, bool, flag 等要点，并说明相关配置、流程、示例或限制条件。

### Section: For checkpoint decryption
**EN:** This section provides a comparison table for For checkpoint decryption, covering columns such as Argument, Description, Defaults and examples such as --decrypted-config-file, --decrypted-draft-config-file, --enable-prefix-mm-cache.
**CN:** 本节围绕 For checkpoint decryption 展开，概述了 Type, str, path, Enable 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Forward hooks
**EN:** This section provides a comparison table for Forward hooks, covering columns such as Argument, Description, Defaults and examples such as --forward-hooks.
**CN:** 本节围绕 Forward hooks 展开，概述了 list, name, dict, Type 等要点，并说明相关配置、流程、示例或限制条件。

### Section: For MindStudio-probe(msProbe) dump
**EN:** This section provides a comparison table for For MindStudio-probe(msProbe) dump, covering columns such as Argument, Description, Defaults and examples such as --msprobe-dump-config.
**CN:** 本节围绕 For MindStudio-probe(msProbe) dump 展开，概述了 Type, Options, msProbe, Argument 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Deprecated arguments
**EN:** This section provides a comparison table for Deprecated arguments, covering columns such as Argument, Description, Defaults and examples such as --enable-ep-moe, --enable-deepep-moe, --prefill-round-robin-balance, --enable-flashinfer-cutlass-moe.
**CN:** 本节围绕 Deprecated arguments 展开，概述了 NOTE, Please, --moe-runner-backend, N/A 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** Type / **CN:** Type
- **EN:** enable / **CN:** enable
- **EN:** flag / **CN:** flag
- **EN:** bool / **CN:** bool
- **EN:** Defaults / **CN:** Defaults
- **EN:** Options / **CN:** Options
- **EN:** str / **CN:** str
- **EN:** Argument / **CN:** Argument

## Dependencies / 依赖关系
- `../advanced_features/sgl_model_gateway.md`
- `hyperparameter_tuning.md`
- `../advanced_features/deterministic_inference.md`
- `../references/custom_chat_template.md`
- `config.yaml`
- `server_args.py`
- `//github.com/sgl-project/sglang/blob/main/python/sglang/srt/server_args.py`
- `warmup.py`
