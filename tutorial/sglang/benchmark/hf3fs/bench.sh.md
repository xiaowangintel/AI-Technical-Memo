# bench.sh — Script Analysis / 脚本分析

## Source / 来源
- **File**: `benchmark/hf3fs/bench.sh`
- **Repository**: sgl-project/sglang
- **Purpose**: This shell script targets hf3fs. It primarily sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. / 该 shell 脚本面向 hf3fs。它主要用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

## Line-by-Line Analysis / 逐行分析
### Lines 1-2: Benchmark commands / 基准测试命令
```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib/python3.12/dist-packages:/usr/local/lib/python3.12/dist-packages/torch/lib
python3 benchmark/hf3fs/bench_client.py
```
**EN:** This block launches Python-based benchmark or serving commands. The main commands are `export`, `python3`. Concretely it sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels.
**CN:** 该代码块启动基于 Python 的基准测试或服务命令。 主要命令包括 `export`, `python3`。 具体来说，它会向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

### Lines 4-6: Benchmark commands / 基准测试命令
```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib/python3.12/dist-packages:/usr/local/lib/python3.12/dist-packages/torch/lib
SGLANG_HICACHE_HF3FS_CONFIG_PATH=/sgl-workspace/sglang/benchmark/hf3fs/hf3fs.json \
python3 benchmark/hf3fs/bench_storage.py
```
**EN:** This block launches Python-based benchmark or serving commands. The main commands are `export`, `SGLANG_HICACHE_HF3FS_CONFIG_PATH=/sgl-workspace/sglang/benchmark/hf3fs/hf3fs.json`, `python3`. Concretely it measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块启动基于 Python 的基准测试或服务命令。 主要命令包括 `export`, `SGLANG_HICACHE_HF3FS_CONFIG_PATH=/sgl-workspace/sglang/benchmark/hf3fs/hf3fs.json`, `python3`。 具体来说，它会测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、加载、筛选或序列化基准测试数据集。

### Lines 8-12: Benchmark commands / 基准测试命令
```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib/python3.12/dist-packages:/usr/local/lib/python3.12/dist-packages/torch/lib
export SGLANG_HICACHE_HF3FS_CONFIG_PATH=/sgl-workspace/sglang/benchmark/hf3fs/hf3fs.json
echo '{"file_path_prefix": "/data/hf3fs-test-0", "file_size": 1099511627776, "numjobs": 16, "entries": 8}' > \
${SGLANG_HICACHE_HF3FS_CONFIG_PATH}
python3 benchmark/hf3fs/bench_zerocopy.py
```
**EN:** This block launches Python-based benchmark or serving commands. The main commands are `export`, `echo`, `${SGLANG_HICACHE_HF3FS_CONFIG_PATH}`, `python3`. Concretely it measures runtime latency, throughput, or other benchmark metrics, prepares tensors and invokes GPU kernels, and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块启动基于 Python 的基准测试或服务命令。 主要命令包括 `export`, `echo`, `${SGLANG_HICACHE_HF3FS_CONFIG_PATH}`, `python3`。 具体来说，它会测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核、加载、筛选或序列化基准测试数据集。

### Lines 14-14: Commented instructions / 注释说明
```bash
####################################################################################################
```
**EN:** This block documents how the benchmark should be launched or interpreted before any commands run.
**CN:** 该代码块以注释形式说明了基准测试的启动方式或结果解读方式。

### Lines 16-24: Benchmark commands / 基准测试命令
```bash
rm -rf nohup.out && \
nohup python3 -m sglang.launch_server \
    --model-path /code/models/Qwen3-32B/ \
    --host 0.0.0.0 --port 33301 \
    --page-size 64 \
    --enable-hierarchical-cache \
    --hicache-ratio 2 --hicache-size 0 \
    --hicache-write-policy write_through \
    --hicache-storage-backend hf3fs &
```
**EN:** This block launches Python-based benchmark or serving commands. The main commands are `rm`, `python3`, `--model-path`, `--host`. Concretely it orchestrates external commands and log collection.
**CN:** 该代码块启动基于 Python 的基准测试或服务命令。 主要命令包括 `rm`, `python3`, `--model-path`, `--host`。 具体来说，它会编排外部命令并收集日志。

### Lines 26-32: Benchmark commands / 基准测试命令
```bash
rm -rf bench_multiturn.out && \
nohup python3 benchmark/hicache/bench_multiturn.py \
    --model-path /code/models/Qwen3-32B \
    --dataset-path /code/models/ShareGPT_V3_unfiltered_cleaned_split.json \
    --port 33301 \
    --request-length 2048 --num-clients 512 --num-rounds 3 --max-parallel 8 \
    > bench_multiturn.out &
```
**EN:** This block launches Python-based benchmark or serving commands. The main commands are `rm`, `python3`, `--model-path`, `--dataset-path`. Concretely it measures runtime latency, throughput, or other benchmark metrics, loads, filters, or serializes benchmark datasets, and orchestrates external commands and log collection.
**CN:** 该代码块启动基于 Python 的基准测试或服务命令。 主要命令包括 `rm`, `python3`, `--model-path`, `--dataset-path`。 具体来说，它会测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集、编排外部命令并收集日志。

### Lines 34-34: Commented instructions / 注释说明
```bash
####################################################################################################
```
**EN:** This block documents how the benchmark should be launched or interpreted before any commands run.
**CN:** 该代码块以注释形式说明了基准测试的启动方式或结果解读方式。

### Lines 36-46: Benchmark commands / 基准测试命令
```bash
rm -rf nohup.out && \
nohup python3 -m sglang.launch_server \
    --model-path /code/models/DeepSeek-R1/ \
    --tp 16 --nnodes 2 --node-rank 0 \
    --dist-init-addr 10.74.249.153:5000 \
    --host 0.0.0.0 --port 33301 \
    --page-size 64 \
    --enable-hierarchical-cache \
    --hicache-ratio 2 --hicache-size 60 \
    --hicache-write-policy write_through \
    --hicache-storage-backend hf3fs &
```
**EN:** This block launches Python-based benchmark or serving commands. The main commands are `rm`, `python3`, `--model-path`, `--tp`. Concretely it orchestrates external commands and log collection.
**CN:** 该代码块启动基于 Python 的基准测试或服务命令。 主要命令包括 `rm`, `python3`, `--model-path`, `--tp`。 具体来说，它会编排外部命令并收集日志。

### Lines 48-54: Benchmark commands / 基准测试命令
```bash
rm -rf bench_multiturn.out && \
nohup python3 benchmark/hicache/bench_multiturn.py \
    --model-path /code/models/Qwen3-32B \
    --dataset-path /code/models/ShareGPT_V3_unfiltered_cleaned_split.json \
    --port 33301 \
    --request-length 2048 --num-clients 1024 --num-rounds 3 --max-parallel 8 \
    > bench_multiturn.out &
```
**EN:** This block launches Python-based benchmark or serving commands. The main commands are `rm`, `python3`, `--model-path`, `--dataset-path`. Concretely it measures runtime latency, throughput, or other benchmark metrics, loads, filters, or serializes benchmark datasets, and orchestrates external commands and log collection.
**CN:** 该代码块启动基于 Python 的基准测试或服务命令。 主要命令包括 `rm`, `python3`, `--model-path`, `--dataset-path`。 具体来说，它会测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集、编排外部命令并收集日志。

### Lines 56-56: Commented instructions / 注释说明
```bash
####################################################################################################
```
**EN:** This block documents how the benchmark should be launched or interpreted before any commands run.
**CN:** 该代码块以注释形式说明了基准测试的启动方式或结果解读方式。

### Lines 58-59: Benchmark commands / 基准测试命令
```bash
ps aux | grep "sglang.launch_server" | grep -v grep | awk '{print $2}' | xargs kill -9
ps aux | grep "bench_multiturn.py" | grep -v grep | awk '{print $2}' | xargs kill -9
```
**EN:** This block launches Python-based benchmark or serving commands. The main commands are `ps`. Concretely it orchestrates external commands and log collection.
**CN:** 该代码块启动基于 Python 的基准测试或服务命令。 主要命令包括 `ps`。 具体来说，它会编排外部命令并收集日志。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Orchestration / 编排**: Launches tools, captures logs, or stitches multi-step experiments together. / 启动工具、抓取日志或拼接多步实验流程。

## Dependencies / 依赖关系
- **Commands / 命令**: `export`, `python3`, `SGLANG_HICACHE_HF3FS_CONFIG_PATH=/sgl-workspace/sglang/benchmark/hf3fs/hf3fs.json`, `echo`, `${SGLANG_HICACHE_HF3FS_CONFIG_PATH}`, `rm`, `--model-path`, `--host`, `--page-size`, `--enable-hierarchical-cache`, `--hicache-ratio`, `--hicache-write-policy`, `--hicache-storage-backend`, `--dataset-path`, `--port`, `--request-length`, `>`, `--tp`, `--dist-init-addr`, `ps`
