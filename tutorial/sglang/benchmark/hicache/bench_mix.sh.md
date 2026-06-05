# bench_mix.sh — Script Analysis / 脚本分析

## Source / 来源
- **File**: `benchmark/hicache/bench_mix.sh`
- **Repository**: sgl-project/sglang
- **Purpose**: This shell script targets hicache mix. It primarily sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. / 该 shell 脚本面向 hicache mix。它主要用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Commented instructions / 注释说明
```bash
#!/bin/bash
```
**EN:** This block documents how the benchmark should be launched or interpreted before any commands run. Concretely it orchestrates external commands and log collection.
**CN:** 该代码块以注释形式说明了基准测试的启动方式或结果解读方式。 具体来说，它会编排外部命令并收集日志。

### Lines 3-18: Benchmark commands / 基准测试命令
```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib/python3.12/dist-packages:/usr/local/lib/python3.12/dist-packages/torch/lib
rm -rf nohup.out && \
nohup python3 -m sglang.launch_server \
    --attention-backend triton \
    --model-path /code/models/Qwen3-32B/ \
    --log-level info \
    --tp 4 --mem-frac 0.25 \
    --host 0.0.0.0 --port 33301 \
    --enable-metrics --enable-cache-report \
    --page-size 64 \
    --enable-hierarchical-cache \
    --hicache-ratio 2.5 --hicache-size 0 \
    --hicache-io-backend kernel \
    --hicache-mem-layout layer_first \
    --hicache-write-policy write_through \
    &
```
**EN:** This block launches Python-based benchmark or serving commands. The main commands are `export`, `rm`, `python3`, `--attention-backend`. Concretely it prepares tensors and invokes GPU kernels, computes evaluation scores and aggregate statistics, and orchestrates external commands and log collection.
**CN:** 该代码块启动基于 Python 的基准测试或服务命令。 主要命令包括 `export`, `rm`, `python3`, `--attention-backend`。 具体来说，它会准备张量并调用 GPU 内核、计算评测分数与聚合统计结果、编排外部命令并收集日志。

### Lines 20-20: Commented instructions / 注释说明
```bash
##################################################
```
**EN:** This block documents how the benchmark should be launched or interpreted before any commands run.
**CN:** 该代码块以注释形式说明了基准测试的启动方式或结果解读方式。

### Lines 22-22: Shell execution block / Shell 执行块
```bash
export CONFIG_PATH=/tmp/bench_mix_config.json
```
**EN:** This block groups related shell commands and environment variables for one experiment stage. Concretely it loads, filters, or serializes benchmark datasets.
**CN:** 该代码块把同一实验阶段相关的 shell 命令与环境变量组织在一起。 具体来说，它会加载、筛选或序列化基准测试数据集。

### Lines 24-34: Logging and redirection / 日志与重定向
```bash
# num_clients: Maximum number of concurrent client requests to be simulated
# round_ratios: Distribution of requests across rounds. Given sum(round_ratios) total requests,
#               round_ratios[i] denotes the number of requests that will execute for (i+1) rounds
echo '{
  "num_rounds": 10,
  "num_clients": 60,
  "round_ratios": [50, 25, 15, 15, 10, 10, 9, 8, 7, 6],
  "mean_new_tokens_per_round": [1000, 400, 350, 300, 280, 260, 240, 220, 210, 200],
  "mean_return_tokens_per_round": [100, 100, 100, 100, 100, 100, 100, 100, 100, 100],
  "mean_inter_round_interval": [30, 30, 30, 30, 30, 30, 30, 30, 30, 30]
}' > ${CONFIG_PATH}
```
**EN:** This block captures command output into files so later comparisons are reproducible. Concretely it sends requests to serving or OpenAI-compatible APIs, coordinates asynchronous or parallel execution, and processes tokenized prompts or decoded outputs.
**CN:** 该代码块把命令输出写入文件，便于后续复现与比较。 具体来说，它会向服务端或 OpenAI 兼容 API 发送请求、协调异步或并行执行、处理分词后的提示词或解码后的输出。

### Lines 36-42: Benchmark commands / 基准测试命令
```bash
rm -rf bench_mix.out && \
nohup python3 /sgl-workspace/sglang/benchmark/hicache/bench_mix.py \
    --model-path /code/models/Qwen3-32B/ \
    --dataset-path /code/models/ShareGPT_V3_unfiltered_cleaned_split.json \
    --port 33301 \
    --duration 600 \
> bench_mix.out &
```
**EN:** This block launches Python-based benchmark or serving commands. The main commands are `rm`, `python3`, `--model-path`, `--dataset-path`. Concretely it measures runtime latency, throughput, or other benchmark metrics, loads, filters, or serializes benchmark datasets, and orchestrates external commands and log collection.
**CN:** 该代码块启动基于 Python 的基准测试或服务命令。 主要命令包括 `rm`, `python3`, `--model-path`, `--dataset-path`。 具体来说，它会测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集、编排外部命令并收集日志。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Concurrency / 并发**: Coordinates multiple requests or workers at the same time. / 同时协调多个请求或工作单元。
- **Serving APIs / 服务接口**: Talks to HTTP or OpenAI-compatible model-serving endpoints. / 与 HTTP 或 OpenAI 兼容模型服务端点交互。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。

## Dependencies / 依赖关系
- **Commands / 命令**: `export`, `rm`, `python3`, `--attention-backend`, `--model-path`, `--log-level`, `--tp`, `--host`, `--enable-metrics`, `--page-size`, `--enable-hierarchical-cache`, `--hicache-ratio`, `--hicache-io-backend`, `--hicache-mem-layout`, `--hicache-write-policy`, `&`, `echo`, `"num_rounds":`, `"num_clients":`, `"round_ratios":`, `"mean_new_tokens_per_round":`, `"mean_return_tokens_per_round":`, `"mean_inter_round_interval":`, `}'`, `--dataset-path`, `--port`, `--duration`, `>`
