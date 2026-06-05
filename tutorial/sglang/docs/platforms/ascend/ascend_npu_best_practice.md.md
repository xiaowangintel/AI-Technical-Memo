# ascend_npu_best_practice.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/platforms/ascend/ascend_npu_best_practice.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This section describes the best practice data of mainstream LLM models such as DeepSeek and Qwen on the Ascend NPU. If you encounter issues or have any questions, please open an issue. / 该文档围绕 Best Practice on Ascend NPU 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** This section describes the best practice data of mainstream LLM models such as DeepSeek and Qwen on the Ascend NPU. If you encounter issues or have any questions, please open an issue.
**CN:** 本节围绕 Overview 展开，概述了 LLM, Qwen, DeepSeek, NPU 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Low Latency
**EN:** This section provides a comparison table for Low Latency, covering columns such as Model, Hardware, Cards and examples such as Deepseek-R1, Deepseek-R1, Deepseek-R1, Deepseek-R1.
**CN:** 本节围绕 Low Latency 展开，概述了 Atlas, Disaggregation, Deepseek-R1, Configuration 等要点，并说明相关配置、流程、示例或限制条件。

### Section: High Throughput
**EN:** This section provides a comparison table for High Throughput, covering columns such as Model, Hardware, Cards and examples such as Deepseek-R1, Deepseek-R1, Deepseek-R1, Deepseek-R1.
**CN:** 本节围绕 High Throughput 展开，概述了 Atlas, Deepseek-R1, Configuration, Disaggregation 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Low Latency
**EN:** This section provides a comparison table for Low Latency, covering columns such as Model, Hardware, Cards and examples such as Qwen3-235B-A22B, Qwen3-32B, Qwen3-32B, Qwen3-32B.
**CN:** 本节围绕 Low Latency 展开，概述了 Atlas, Mixed, Configuration, Optimal 等要点，并说明相关配置、流程、示例或限制条件。

### Section: High Throughput
**EN:** This section provides a comparison table for High Throughput, covering columns such as Model, Hardware, Cards and examples such as Qwen3-235B-A22B, Qwen3-235B-A22B, Qwen3-235B-A22B, Qwen3-235B-A22B.
**CN:** 本节围绕 High Throughput 展开，概述了 Atlas, Mixed, Configuration, W8A8 等要点，并说明相关配置、流程、示例或限制条件。

### Section: DeepSeek-R1 3_5K-1_5K 50ms on A3 32 Cards Disaggregation Mode
**EN:** This section provides a comparison table for DeepSeek-R1 3_5K-1_5K 50ms on A3 32 Cards Disaggregation Mode, covering columns such as if [[ "$LOCAL_HOST1" == "$ ", "$LOCAL_HOST2" == "$ " ]]; and examples such as if [[ "$LOCAL_HOST1" == "$ ".
**CN:** 本节围绕 DeepSeek-R1 3_5K-1_5K 50ms on A3 32 Cards Disaggregation Mode 展开，概述了 export, D_IP, P_IP, LOCAL_HOST1 等要点，并说明相关配置、流程、示例或限制条件。

### Section: DeepSeek-R1 2K-2K 50ms on A3 24 Cards Disaggregation Mode
**EN:** This section provides a comparison table for DeepSeek-R1 2K-2K 50ms on A3 24 Cards Disaggregation Mode, covering columns such as if [[ "$LOCAL_HOST1" == "$ ", "$LOCAL_HOST2" == "$ " ]]; and examples such as if [[ "$LOCAL_HOST1" == "$ ".
**CN:** 本节围绕 DeepSeek-R1 2K-2K 50ms on A3 24 Cards Disaggregation Mode 展开，概述了 export, D_IP, P_IP, LOCAL_HOST1 等要点，并说明相关配置、流程、示例或限制条件。

### Section: DeepSeek-R1 6K-1_6K 20ms on A3 32 Cards Disaggregation Mode
**EN:** This section provides a comparison table for DeepSeek-R1 6K-1_6K 20ms on A3 32 Cards Disaggregation Mode, covering columns such as if [[ "$LOCAL_HOST1" == "$ ", "$LOCAL_HOST2" == "$ " ]]; and examples such as if [[ "$LOCAL_HOST1" == "$ ".
**CN:** 本节围绕 DeepSeek-R1 6K-1_6K 20ms on A3 32 Cards Disaggregation Mode 展开，概述了 export, P_IP, D_IP, LOCAL_HOST1 等要点，并说明相关配置、流程、示例或限制条件。

### Section: DeepSeek-R1 3_9K-1K 19ms on A3 32 Cards Disaggregation Mode
**EN:** This section provides a comparison table for DeepSeek-R1 3_9K-1K 19ms on A3 32 Cards Disaggregation Mode, covering columns such as if [[ "$LOCAL_HOST1" == "$ ", "$LOCAL_HOST2" == "$ " ]]; and examples such as if [[ "$LOCAL_HOST1" == "$ ".
**CN:** 本节围绕 DeepSeek-R1 3_9K-1K 19ms on A3 32 Cards Disaggregation Mode 展开，概述了 export, P_IP, D_IP, LOCAL_HOST1 等要点，并说明相关配置、流程、示例或限制条件。

### Section: DeepSeek-R1 3_5K-1_5K 19ms on A3 32 Cards Disaggregation Mode
**EN:** Model: Deepseek R1 Hardware: Atlas 800I A3 32Card DeployMode: PD Disaggregation Dataset: random Input Output Length: 3.5K+1.5K TPOT: 19ms #### Model Deployment Please Turn to DeepSeek-R1 3_9K-1K 19ms on A3 32 Cards Disaggregation Mode #### Benchmark We tested it based on the RANDOM dataset.
**CN:** 本节围绕 DeepSeek-R1 3_5K-1_5K 19ms on A3 32 Cards Disaggregation Mode 展开，概述了 Model, RANDOM, TPOT, Atlas 等要点，并说明相关配置、流程、示例或限制条件。

### Section: DeepSeek-R1 3_5K-1K 19ms on A3 32 Cards Disaggregation Mode
**EN:** Model: Deepseek R1 Hardware: Atlas 800I A3 32Card DeployMode: PD Disaggregation Dataset: random Input Output Length: 3.5K+1K TPOT: 19ms #### Model Deployment Please Turn to DeepSeek-R1 3_9K-1K 19ms on A3 32 Cards Disaggregation Mode #### Benchmark We tested it based on the RANDOM dataset.
**CN:** 本节围绕 DeepSeek-R1 3_5K-1K 19ms on A3 32 Cards Disaggregation Mode 展开，概述了 Model, RANDOM, TPOT, Atlas 等要点，并说明相关配置、流程、示例或限制条件。

### Section: DeepSeek-R1 2K-2K 50ms on A3 8 Cards Mixed Mode
**EN:** `shell python -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 6699 --max-concurrency 352 --random-input-len 2048 --random-output-len 2048 --num-prompts 1408 --random-range-ratio 1 ``
**CN:** 本节围绕 DeepSeek-R1 2K-2K 50ms on A3 8 Cards Mixed Mode 展开，概述了 export, unset, PATH, MODEL_PATH 等要点，并说明相关配置、流程、示例或限制条件。

### Section: DeepSeek-R1 2K-2K 50ms on A3 16 Cards Disaggregation Mode
**EN:** This section provides a comparison table for DeepSeek-R1 2K-2K 50ms on A3 16 Cards Disaggregation Mode, covering columns such as if [[ "$LOCAL_HOST1" == "$ ", "$LOCAL_HOST2" == "$ " ]]; and examples such as if [[ "$LOCAL_HOST1" == "$ ".
**CN:** 本节围绕 DeepSeek-R1 2K-2K 50ms on A3 16 Cards Disaggregation Mode 展开，概述了 export, P_IP, D_IP, LOCAL_HOST1 等要点，并说明相关配置、流程、示例或限制条件。

### Section: DeepSeek-R1 3_5K-1_5K 50ms on A3 8 Cards Mixed Mode
**EN:** `shell python -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 6699 --max-concurrency 224 --random-input-len 3500 --random-output-len 1500 --num-prompts 896 --random-range-ratio 1 ``
**CN:** 本节围绕 DeepSeek-R1 3_5K-1_5K 50ms on A3 8 Cards Mixed Mode 展开，概述了 export, unset, PATH, MODEL_PATH 等要点，并说明相关配置、流程、示例或限制条件。

### Section: DeepSeek-R1 3_5K-1_5K 50ms on A3 16 Cards Disaggregation Mode
**EN:** This section provides a comparison table for DeepSeek-R1 3_5K-1_5K 50ms on A3 16 Cards Disaggregation Mode, covering columns such as if [[ "$LOCAL_HOST1" == "$ ", "$LOCAL_HOST2" == "$ " ]]; and examples such as if [[ "$LOCAL_HOST1" == "$ ".
**CN:** 本节围绕 DeepSeek-R1 3_5K-1_5K 50ms on A3 16 Cards Disaggregation Mode 展开，概述了 export, P_IP, D_IP, LOCAL_HOST1 等要点，并说明相关配置、流程、示例或限制条件。

### Section: DeepSeek-V3.2 128K-1K 26ms on A3 32 Cards Disaggregation Mode
**EN:** This section provides a comparison table for DeepSeek-V3.2 128K-1K 26ms on A3 32 Cards Disaggregation Mode, covering columns such as if [[ "$LOCAL_HOST1" == "$ ", "$LOCAL_HOST2" == "$ " ]]; and examples such as if [[ "$LOCAL_HOST1" == "$ ".
**CN:** 本节围绕 DeepSeek-V3.2 128K-1K 26ms on A3 32 Cards Disaggregation Mode 展开，概述了 export, P_IP, D_IP, LOCAL_HOST1 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-235B-A22B 3_5K-1_5K 50ms on A3 24 Cards Disaggregation Mode
**EN:** This section provides a comparison table for Qwen3-235B-A22B 3_5K-1_5K 50ms on A3 24 Cards Disaggregation Mode, covering columns such as if [[ "$LOCAL_HOST1" == "$ ", "$LOCAL_HOST2" == "$ " ]]; and examples such as if [[ "$LOCAL_HOST1" == "$ ".
**CN:** 本节围绕 Qwen3-235B-A22B 3_5K-1_5K 50ms on A3 24 Cards Disaggregation Mode 展开，概述了 export, P_IP, D_IP, LOCAL_HOST1 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-235B-A22B 3_5K-1_5K 50ms on A3 8 Cards Mixed Mode
**EN:** `shell python -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7439 --max-concurrency 272 --random-input-len 3500 --random-output-len 1500 --num-prompts 1088 --random-range-ratio 1 ``
**CN:** 本节围绕 Qwen3-235B-A22B 3_5K-1_5K 50ms on A3 8 Cards Mixed Mode 展开，概述了 export, unset, PATH, Ascend 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-235B-A22B 2K-2K 100ms on A3 8 Cards Mixed Mode
**EN:** `shell python -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7439 --max-concurrency 576 --random-input-len 2000 --random-output-len 2000 --num-prompts 576 --random-range-ratio 1 ``
**CN:** 本节围绕 Qwen3-235B-A22B 2K-2K 100ms on A3 8 Cards Mixed Mode 展开，概述了 export, unset, PATH, Ascend 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-235B-A22B 2K-2K 50ms on A3 8 Cards Mixed Mode
**EN:** `shell python -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7439 --max-concurrency 480 --random-input-len 2048 --random-output-len 2048 --num-prompts 480 --random-range-ratio 1 ``
**CN:** 本节围绕 Qwen3-235B-A22B 2K-2K 50ms on A3 8 Cards Mixed Mode 展开，概述了 export, unset, xxx, PATH 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-235B-A22B 2K-2K 50ms on A3 16 Cards Mixed Mode
**EN:** `shell python -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7439 --max-concurrency 768 --random-input-len 2000 --random-output-len 2000 --num-prompts 768 --random-range-ratio 1 ``
**CN:** 本节围绕 Qwen3-235B-A22B 2K-2K 50ms on A3 16 Cards Mixed Mode 展开，概述了 MIX_IP, export, LOCAL_HOST1, LOCAL_HOST2 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-235B-A22B 11K-1K 10ms on A3 8 Cards Mixed Mode
**EN:** `shell python -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7439 --max-concurrency 1 --random-input-len 11000 --random-output-len 1000 --num-prompts 1 --random-range-ratio 1 ``
**CN:** 本节围绕 Qwen3-235B-A22B 11K-1K 10ms on A3 8 Cards Mixed Mode 展开，概述了 export, unset, xxx, PATH 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-32B 6K-1_5K 18ms on A3 4 Cards Mixed Mode
**EN:** `shell python3 -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7439 --max-concurrency 32 --random-output-len 1500 --random-input-len 6000 --num-prompts 32 --random-range-ratio 1 ``
**CN:** 本节围绕 Qwen3-32B 6K-1_5K 18ms on A3 4 Cards Mixed Mode 展开，概述了 export, unset, xxx, Ascend 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-32B 4K-1_5K 11ms on A3 4 Cards Mixed Mode
**EN:** `shell python3 -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7239 --random-range-ratio 1 --max-concurrency 1 --random-output-len 1500 --random-input-len 4096 --num-prompts 4 ``
**CN:** 本节围绕 Qwen3-32B 4K-1_5K 11ms on A3 4 Cards Mixed Mode 展开，概述了 export, unset, Ascend, LOCAL_HOST1 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-32B 18K-4K 6ms on A3 8 Cards Mixed Mode
**EN:** `shell python3 -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7339 --random-range-ratio 1 --max-concurrency 1 --random-output-len 18000 --random-input-len 4000 --num-prompts 1 ``
**CN:** 本节围绕 Qwen3-32B 18K-4K 6ms on A3 8 Cards Mixed Mode 展开，概述了 export, unset, Ascend, LOCAL_HOST1 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-32B 3_5K-1_5K 50ms on A3 2 Cards Mixed Mode
**EN:** `shell python3 -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7239 --max-concurrency 78 --random-output-len 1500 --random-input-len 3500 --num-prompts 312 --random-range-ratio 1 ``
**CN:** 本节围绕 Qwen3-32B 3_5K-1_5K 50ms on A3 2 Cards Mixed Mode 展开，概述了 export, unset, Ascend, LOCAL_HOST1 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-32B 2K-2K 50ms on A3 2 Cards Mixed Mode
**EN:** `shell python3 -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7239 --max-concurrency 120 --random-output-len 2000 --random-input-len 2000 --num-prompts 480 --random-range-ratio 1 ``
**CN:** 本节围绕 Qwen3-32B 2K-2K 50ms on A3 2 Cards Mixed Mode 展开，概述了 export, unset, Ascend, LOCAL_HOST1 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-30B-A3B 3_5K-1_5K 50ms on A3 1 Card Mixed Mode
**EN:** `shell python -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7239 --max-concurrency 156 --random-input-len 3500 --random-output-len 1500 --num-prompts 624 --random-range-ratio 1 ``
**CN:** 本节围绕 Qwen3-30B-A3B 3_5K-1_5K 50ms on A3 1 Card Mixed Mode 展开，概述了 export, unset, Ascend, LOCAL_HOST2 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-Coder-480B-A35B-Instruct 3_5K-1_5K 50ms on A3 24 Cards Disaggregation Mode
**EN:** This section provides a comparison table for Qwen3-Coder-480B-A35B-Instruct 3_5K-1_5K 50ms on A3 24 Cards Disaggregation Mode, covering columns such as if [[ "$LOCAL_HOST1" == "$ ", "$LOCAL_HOST2" == "$ " ]]; and examples such as if [[ "$LOCAL_HOST1" == "$ ".
**CN:** 本节围绕 Qwen3-Coder-480B-A35B-Instruct 3_5K-1_5K 50ms on A3 24 Cards Disaggregation Mode 展开，概述了 export, P_IP, D_IP, LOCAL_HOST1 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-Coder-480B-A35B-Instruct 3_5K-1_5K 50ms on A3 16 Cards Mixed Mode
**EN:** `shell python -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7439 --max-concurrency 288 --random-input-len 3500 --random-output-len 1500 --num-prompts 1152 --random-range-ratio 1 --request-rate 20 ``
**CN:** 本节围绕 Qwen3-Coder-480B-A35B-Instruct 3_5K-1_5K 50ms on A3 16 Cards Mixed Mode 展开，概述了 MIX_IP, export, LOCAL_HOST1, LOCAL_HOST2 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-Coder-480B-A35B-Instruct 3_5K-1_5K 50ms on A3 8 Cards Mixed Mode
**EN:** `shell python -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7439 --max-concurrency 80 --random-input-len 3500 --random-output-len 1500 --num-prompts 320 --random-range-ratio 1 ``
**CN:** 本节围绕 Qwen3-Coder-480B-A35B-Instruct 3_5K-1_5K 50ms on A3 8 Cards Mixed Mode 展开，概述了 export, unset, PATH, Ascend 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-Next-80B-A3B-Instruct 3_5K-1_5K 50ms on A3 2 Cards Mixed Mode
**EN:** `shell python3 -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 6699 --max-concurrency 80 --random-output-len 1536 --random-input-len 3584 --num-prompts 160 --random-range-ratio 1 ``
**CN:** 本节围绕 Qwen3-Next-80B-A3B-Instruct 3_5K-1_5K 50ms on A3 2 Cards Mixed Mode 展开，概述了 export, source, cann_path, Model 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-32B 6K-1_5K 18ms on A2 8 Cards Mixed Mode
**EN:** `shell python3 -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7439 --max-concurrency 32 --random-output-len 1500 --random-input-len 6000 --num-prompts 32 --random-range-ratio 1 ``
**CN:** 本节围绕 Qwen3-32B 6K-1_5K 18ms on A2 8 Cards Mixed Mode 展开，概述了 export, unset, Ascend, LOCAL_HOST1 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-32B 4K-1_5K 11ms on A2 8 Cards Mixed Mode
**EN:** `shell python3 -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7339 --random-range-ratio 1 --max-concurrency 1 --random-output-len 1500 --random-input-len 4096 --num-prompts 4 ``
**CN:** 本节围绕 Qwen3-32B 4K-1_5K 11ms on A2 8 Cards Mixed Mode 展开，概述了 export, unset, PATH, Ascend 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-32B 1K-0_3K 12ms on A3 2 Cards Mixed Mode
**EN:** `shell python3 -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7339 --random-range-ratio 1 --max-concurrency 16 --random-output-len 300 --random-input-len 1024 --num-prompts 16 ``
**CN:** 本节围绕 Qwen3-32B 1K-0_3K 12ms on A3 2 Cards Mixed Mode 展开，概述了 export, unset, PATH, Ascend 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-32B 6K-1_5K 17ms on A3 2 Cards Mixed Mode
**EN:** `shell python3 -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7339 --random-range-ratio 1 --max-concurrency 16 --random-output-len 1500 --random-input-len 6144 --num-prompts 16 ``
**CN:** 本节围绕 Qwen3-32B 6K-1_5K 17ms on A3 2 Cards Mixed Mode 展开，概述了 export, unset, PATH, Ascend 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-8B 1K-0_3K 7ms on A3 1 Cards Mixed Mode
**EN:** `shell python3 -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7339 --random-range-ratio 1 --max-concurrency 16 --random-output-len 300 --random-input-len 1024 --num-prompts 16 ``
**CN:** 本节围绕 Qwen3-8B 1K-0_3K 7ms on A3 1 Cards Mixed Mode 展开，概述了 export, unset, PATH, Ascend 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-8B 6K-1_5K 12ms on A3 1 Cards Mixed Mode
**EN:** `shell python3 -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7339 --random-range-ratio 1 --max-concurrency 16 --random-output-len 1500 --random-input-len 6144 --num-prompts 16 ``
**CN:** 本节围绕 Qwen3-8B 6K-1_5K 12ms on A3 1 Cards Mixed Mode 展开，概述了 export, unset, PATH, Ascend 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-32B 3_5K-1_5K 50ms on A2 8 Cards Mixed Mode
**EN:** `shell python3 -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7239 --max-concurrency 78 --random-output-len 1500 --random-input-len 3500 --num-prompts 312 --random-range-ratio 1 ``
**CN:** 本节围绕 Qwen3-32B 3_5K-1_5K 50ms on A2 8 Cards Mixed Mode 展开，概述了 export, unset, Ascend, LOCAL_HOST1 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-32B 2K-2K 50ms on A2 8 Cards Mixed Mode
**EN:** `shell python3 -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7239 --max-concurrency 120 --random-output-len 2000 --random-input-len 2000 --num-prompts 120 --random-range-ratio 1 ``
**CN:** 本节围绕 Qwen3-32B 2K-2K 50ms on A2 8 Cards Mixed Mode 展开，概述了 export, unset, Ascend, LOCAL_HOST1 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-30B-A3B 6K-1_5K 10ms on A3 1 Cards Mixed Mode
**EN:** `shell python3 -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7339 --random-range-ratio 1 --max-concurrency 16 --random-output-len 1500 --random-input-len 6144 --num-prompts 16 ``
**CN:** 本节围绕 Qwen3-30B-A3B 6K-1_5K 10ms on A3 1 Cards Mixed Mode 展开，概述了 export, unset, LOCAL_HOST1, LOCAL_HOST2 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-30B-A3B 1K-0_3K 7ms on A3 1 Cards Mixed Mode
**EN:** `shell python3 -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7339 --random-range-ratio 1 --max-concurrency 8 --random-output-len 300 --random-input-len 1024 --num-prompts 8 ``
**CN:** 本节围绕 Qwen3-30B-A3B 1K-0_3K 7ms on A3 1 Cards Mixed Mode 展开，概述了 export, unset, LOCAL_HOST1, LOCAL_HOST2 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-Next 1K-0_3K 14_21ms on A3 2 Cards Mixed Mode
**EN:** `shell python3 -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 6699 --random-range-ratio 1 --max-concurrency 16 --random-output-len 300 --random-input-len 1024 --num-prompts 16 ``
**CN:** 本节围绕 Qwen3-Next 1K-0_3K 14_21ms on A3 2 Cards Mixed Mode 展开，概述了 export, unset, PATH, Ascend 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-Next 6K-1_5K 15_62ms on A3 2 Cards Mixed Mode
**EN:** `shell python3 -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 6699 --random-range-ratio 1 --max-concurrency 16 --random-output-len 1500 --random-input-len 6144 --num-prompts 16 ``
**CN:** 本节围绕 Qwen3-Next 6K-1_5K 15_62ms on A3 2 Cards Mixed Mode 展开，概述了 export, unset, PATH, Ascend 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-14B 3_5K-1_5K 9ms on A3 1 Cards Mixed Mode
**EN:** `shell python3 -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7239 --max-concurrency 1 --random-output-len 1500 --random-input-len 3500 --num-prompts 8 --random-range-ratio 1 ``
**CN:** 本节围绕 Qwen3-14B 3_5K-1_5K 9ms on A3 1 Cards Mixed Mode 展开，概述了 export, unset, Ascend, LOCAL_HOST1 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-14B 3_5K-1_5K 50ms on A3 1 Cards Mixed Mode
**EN:** `shell python3 -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7239 --max-concurrency 144 --random-output-len 1500 --random-input-len 3500 --num-prompts 576 --random-range-ratio 1 ``
**CN:** 本节围绕 Qwen3-14B 3_5K-1_5K 50ms on A3 1 Cards Mixed Mode 展开，概述了 export, unset, Ascend, LOCAL_HOST1 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-8B 3_5K-1_5K 50ms on A3 1 Cards Mixed Mode
**EN:** `shell python3 -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7239 --max-concurrency 64 --random-output-len 1500 --random-input-len 3500 --num-prompts 256 --random-range-ratio 1 ``
**CN:** 本节围绕 Qwen3-8B 3_5K-1_5K 50ms on A3 1 Cards Mixed Mode 展开，概述了 export, unset, Ascend, LOCAL_HOST1 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-8B 3_5K-1_5K 5ms on A3 1 Cards Mixed Mode
**EN:** `shell python3 -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 7239 --max-concurrency 1 --random-output-len 1500 --random-input-len 3500 --num-prompts 4 --random-range-ratio 1 ``
**CN:** 本节围绕 Qwen3-8B 3_5K-1_5K 5ms on A3 1 Cards Mixed Mode 展开，概述了 export, unset, Ascend, LOCAL_HOST1 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Qwen3-Next 3_5K-1_5K 20ms on A3 2 Cards Mixed Mode
**EN:** `shell python3 -m sglang.bench_serving --dataset-name random --backend sglang --host 127.0.0.1 --port 6699 --random-range-ratio 1 --max-concurrency 1 --random-output-len 1500 --random-input-len 3500 --num-prompts 1 ``
**CN:** 本节围绕 Qwen3-Next 3_5K-1_5K 20ms on A3 2 Cards Mixed Mode 展开，概述了 export, unset, PATH, Ascend 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** export / **CN:** export
- **EN:** LOCAL_HOST2 / **CN:** LOCAL_HOST2
- **EN:** LOCAL_HOST1 / **CN:** LOCAL_HOST1
- **EN:** unset / **CN:** unset
- **EN:** Atlas / **CN:** Atlas
- **EN:** MODEL_PATH / **CN:** 模型_PATH
- **EN:** Ascend / **CN:** Ascend
- **EN:** echo / **CN:** echo

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
