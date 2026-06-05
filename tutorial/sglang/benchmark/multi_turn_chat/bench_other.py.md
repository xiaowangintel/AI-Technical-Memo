# bench_other.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/multi_turn_chat/bench_other.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on multi turn chat other. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 multi turn chat other 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Imports and setup / 导入与初始化
```python
import json
import time
from argparse import ArgumentParser
from concurrent.futures import ThreadPoolExecutor
from functools import partial

from data_gen import gen_arguments
from tqdm import tqdm
from vllm.transformers_utils.tokenizer import get_tokenizer

from sglang.test.test_utils import add_common_other_args_and_parse, get_call_generate
from sglang.utils import dump_state_text
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, coordinates asynchronous or parallel execution, and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、协调异步或并行执行、加载、筛选或序列化基准测试数据集。

### Lines 15-21: Function `multi_turns` / 函数 `multi_turns`
```python
def multi_turns(generate, qas):
    s = ""
    for qa in qas:
        s += qa["prompt"]
        s += generate(s, max_tokens=qa["new_tokens"])

    return s
```
**EN:** `multi_turns` is a function that processes tokenized prompts or decoded outputs. It returns `s` to the caller. Notable calls include `generate`.
**CN:** `multi_turns` 是一个函数，用于处理分词后的提示词或解码后的输出。它会向调用方返回 `s`。其中较关键的调用包括 `generate`。

### Lines 24-73: Function `main` / 函数 `main`
```python
def main(args):
    print(args)

    tokenizer = get_tokenizer(args.tokenizer, trust_remote_code=args.trust_remote_code)

    multi_qas = gen_arguments(args, tokenizer)

    states = [None] * args.num_qa

    call_generate = partial(get_call_generate(args), temperature=0)

    def get_one_answer(i):
        states[i] = multi_turns(generate=call_generate, **multi_qas[i])

    tic = time.perf_counter()
    if args.parallel == 1:
        for i in tqdm(range(len(multi_qas))):
            get_one_answer(i)
    else:
        with ThreadPoolExecutor(args.parallel) as executor:
            rets = list(
                tqdm(
                    executor.map(get_one_answer, list(range(len(multi_qas)))),
                    total=len(multi_qas),
                )
            )
            for _ in rets:
                pass

    latency = time.perf_counter() - tic

    # Compute accuracy
    print(f"Latency: {latency:.3f}")

    dump_state_text(f"tmp_output_{args.backend}.txt", states)

    with open(args.result_file, "a") as fout:
        value = {
            "task": "multi_turn_chat",
            "backend": args.backend,
            "num_gpus": 1,
            "latency": round(latency, 3),
            "num_requests": args.num_qa,
            "num_turns": args.turns,
            "other": {
                "parallel": args.parallel,
                "output_mode": "long" if args.long else "short",
            },
        }
        fout.write(json.dumps(value) + "\n")
```
**EN:** `main` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. Notable calls include `print`, `get_tokenizer`, `gen_arguments`.
**CN:** `main` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。其中较关键的调用包括 `print`, `get_tokenizer`, `gen_arguments`。

### Lines 76-93: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = ArgumentParser()
    parser.add_argument("--turns", type=int, default=4)
    parser.add_argument("--num-qa", type=int, default=20)
    parser.add_argument("--min-len-q", type=int, default=256)
    parser.add_argument("--max-len-q", type=int, default=512)
    parser.add_argument("--min-len-a", type=int, default=4)
    parser.add_argument("--max-len-a", type=int, default=8)
    parser.add_argument("--tokenizer", type=str, required=True)
    parser.add_argument("--trust-remote-code", action="store_true")
    parser.add_argument("--long", action="store_true")
    args = add_common_other_args_and_parse(parser)

    if args.long:
        args.min_len_a = 256
        args.max_len_a = 512
        args.num_qa = 20
    main(args)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration and processes tokenized prompts or decoded outputs.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置、处理分词后的提示词或解码后的输出。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Concurrency / 并发**: Coordinates multiple requests or workers at the same time. / 同时协调多个请求或工作单元。
- **Serving APIs / 服务接口**: Talks to HTTP or OpenAI-compatible model-serving endpoints. / 与 HTTP 或 OpenAI 兼容模型服务端点交互。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `time`, `argparse`, `concurrent.futures`, `functools`
- **Third-party / 第三方依赖**: `data_gen`, `tqdm`, `vllm.transformers_utils.tokenizer`
- **Internal / 项目内部依赖**: `sglang.test.test_utils`, `sglang.utils`
