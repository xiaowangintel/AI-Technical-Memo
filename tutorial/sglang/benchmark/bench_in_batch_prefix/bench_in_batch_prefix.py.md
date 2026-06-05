# bench_in_batch_prefix.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/bench_in_batch_prefix/bench_in_batch_prefix.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on in batch prefix. It primarily sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and prepares tensors and invokes GPU kernels. / 该 Python 模块聚焦于 in batch prefix 相关流程。它主要用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、准备张量并调用 GPU 内核。

## Line-by-Line Analysis / 逐行分析
### Lines 6-15: Imports and setup / 导入与初始化
```python
import random
import string
import time

from tqdm import tqdm
from transformers import AutoTokenizer

import sglang as sgl
from sglang import set_default_backend
from sglang.lang.backend.runtime_endpoint import RuntimeEndpoint
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it sends requests to serving or OpenAI-compatible APIs and processes tokenized prompts or decoded outputs.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会向服务端或 OpenAI 兼容 API 发送请求、处理分词后的提示词或解码后的输出。

### Lines 18-32: Function `generate_random_string` / 函数 `generate_random_string`
```python
def generate_random_string(token_length: int) -> str:
    random_string = "".join(
        random.choices(string.ascii_letters + string.digits, k=token_length * 100)
    )
    tokenized_output = tokenizer.encode(random_string, add_special_tokens=False)[
        :token_length
    ]

    if len(tokenized_output) < token_length:
        tokenized_output = tokenized_output + [tokenizer.pad_token_id] * (
            token_length - len(tokenized_output)
        )

    decoded_string = tokenizer.decode(tokenized_output, skip_special_tokens=False)
    return decoded_string
```
**EN:** `generate_random_string` is a function that prepares tensors and invokes GPU kernels and processes tokenized prompts or decoded outputs. It returns `decoded_string` to the caller. Notable calls include `''.join`, `tokenizer.decode`, `random.choices`.
**CN:** `generate_random_string` 是一个函数，用于准备张量并调用 GPU 内核、处理分词后的提示词或解码后的输出。它会向调用方返回 `decoded_string`。其中较关键的调用包括 `''.join`, `tokenizer.decode`, `random.choices`。

### Lines 35-36: Function `generate_unique_prefix` / 函数 `generate_unique_prefix`
```python
def generate_unique_prefix(base_text, index):
    return str(index) + base_text[len(str(index)) :]
```
**EN:** `generate_unique_prefix` is a function that implements the core logic for this scope. It returns `str(index) + base_text[len(str(index)):]` to the caller. Notable calls include `str`, `len`.
**CN:** `generate_unique_prefix` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `str(index) + base_text[len(str(index)):]`。其中较关键的调用包括 `str`, `len`。

### Lines 40-42: Function `text_qa` / 函数 `text_qa`
```python
def text_qa(s, question, gen_len):
    s += "Q: " + question + "\n"
    s += "A:" + sgl.gen("answer", stop="\n", temperature=0, max_tokens=gen_len)
```
**EN:** `text_qa` is a function that processes tokenized prompts or decoded outputs and computes evaluation scores and aggregate statistics. Notable calls include `sgl.gen`.
**CN:** `text_qa` 是一个函数，用于处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。其中较关键的调用包括 `sgl.gen`。

### Lines 45-59: Function `prepare_prompts` / 函数 `prepare_prompts`
```python
def prepare_prompts(num_prefix, num_samples_per_prefix, prefix_length, suffix_length):
    base_prefix = generate_random_string(prefix_length)

    tot_input_len = 0
    all_prompts = []
    for i in tqdm(range(num_prefix), desc="prepare prompts"):
        unique_prefix = generate_unique_prefix(base_prefix, i)
        prompt_list = []
        for j in range(num_samples_per_prefix):
            suffix = generate_random_string(suffix_length)
            prompt = unique_prefix + suffix
            prompt_list.append(prompt)
            tot_input_len += len(tokenizer.encode(prompt))
        all_prompts.append(prompt_list)
    return all_prompts, tot_input_len
```
**EN:** `prepare_prompts` is a function that loads, filters, or serializes benchmark datasets and processes tokenized prompts or decoded outputs. It returns `(all_prompts, tot_input_len)` to the caller. Notable calls include `generate_random_string`, `tqdm`, `range`.
**CN:** `prepare_prompts` 是一个函数，用于加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出。它会向调用方返回 `(all_prompts, tot_input_len)`。其中较关键的调用包括 `generate_random_string`, `tqdm`, `range`。

### Lines 62-73: Function `test_batch_by_batch` / 函数 `test_batch_by_batch`
```python
def test_batch_by_batch(all_prompts, gen_len):
    backend.flush_cache()

    tot_time = 0
    for i in range(len(all_prompts)):
        tic = time.perf_counter()
        text_qa.run_batch(
            list(zip(all_prompts[i], [gen_len] * len(all_prompts[i]))),
        )
        tot_time += time.perf_counter() - tic

    return tot_time
```
**EN:** `test_batch_by_batch` is a function that measures runtime latency, throughput, or other benchmark metrics. It returns `tot_time` to the caller. Notable calls include `backend.flush_cache`, `range`, `len`.
**CN:** `test_batch_by_batch` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标。它会向调用方返回 `tot_time`。其中较关键的调用包括 `backend.flush_cache`, `range`, `len`。

### Lines 76-89: Function `test_batch_by_batch_with_hint` / 函数 `test_batch_by_batch_with_hint`
```python
def test_batch_by_batch_with_hint(all_prompts, gen_len):
    backend.flush_cache()

    tot_time = 0
    for i in range(len(all_prompts)):
        tic = time.perf_counter()
        # Send a hint to cache the prefix
        text_qa.run_batch(list(zip(all_prompts[i][:1], [gen_len])))
        # Send the batch
        text_qa.run_batch(list(zip(all_prompts[i], [gen_len] * len(all_prompts[i]))))

        tot_time += time.perf_counter() - tic

    return tot_time
```
**EN:** `test_batch_by_batch_with_hint` is a function that measures runtime latency, throughput, or other benchmark metrics. It returns `tot_time` to the caller. Notable calls include `backend.flush_cache`, `range`, `len`.
**CN:** `test_batch_by_batch_with_hint` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标。它会向调用方返回 `tot_time`。其中较关键的调用包括 `backend.flush_cache`, `range`, `len`。

### Lines 92-103: Function `test_send_all` / 函数 `test_send_all`
```python
def test_send_all(all_prompts, gen_len):
    backend.flush_cache()

    all_prompts = [x for prompt_list in all_prompts for x in prompt_list]

    tic = time.perf_counter()
    text_qa.run_batch(
        list(zip(all_prompts, [gen_len] * len(all_prompts))),
    )
    tot_time = time.perf_counter() - tic

    return tot_time
```
**EN:** `test_send_all` is a function that measures runtime latency, throughput, or other benchmark metrics. It returns `tot_time` to the caller. Notable calls include `backend.flush_cache`, `time.perf_counter`, `text_qa.run_batch`.
**CN:** `test_send_all` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标。它会向调用方返回 `tot_time`。其中较关键的调用包括 `backend.flush_cache`, `time.perf_counter`, `text_qa.run_batch`。

### Lines 106-130: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    tokenizer = AutoTokenizer.from_pretrained("hf-internal-testing/llama-tokenizer")
    backend = RuntimeEndpoint("http://127.0.0.1:30000")
    set_default_backend(backend)

    random.seed(0)
    num_prefix = 10
    num_samples_per_prefix = 32
    prefix_length = 1024
    suffix_length = 128
    gen_len = 1
    all_prompts, tot_input_len = prepare_prompts(
        num_prefix, num_samples_per_prefix, prefix_length, suffix_length
    )

    print(f"Total input token length: {tot_input_len}\n")

    cost = test_batch_by_batch(all_prompts, gen_len)
    print(f"Latency of test_batch_by_batch          : {cost:.4f} s\n")

    cost = test_batch_by_batch_with_hint(all_prompts, gen_len)
    print(f"Latency of test_batch_by_batch_with_hint: {cost:.4f} s\n")

    cost = test_send_all(all_prompts, gen_len)
    print(f"Latency of test_send_all                : {cost:.4f} s\n")
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Serving APIs / 服务接口**: Talks to HTTP or OpenAI-compatible model-serving endpoints. / 与 HTTP 或 OpenAI 兼容模型服务端点交互。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。
- **Orchestration / 编排**: Launches tools, captures logs, or stitches multi-step experiments together. / 启动工具、抓取日志或拼接多步实验流程。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `random`, `string`, `time`
- **Third-party / 第三方依赖**: `tqdm`, `transformers`
- **Internal / 项目内部依赖**: `sglang`, `sglang.lang.backend.runtime_endpoint`
