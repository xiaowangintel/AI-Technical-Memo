# bench_other.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/tree_of_thought_deep/bench_other.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on tree of thought deep other. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 tree of thought deep other 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Imports and setup / 导入与初始化
```python
import argparse
import ast
import json
import re
import time
from collections import Counter
from concurrent.futures import ThreadPoolExecutor

import numpy as np
from tqdm import tqdm

from sglang.test.test_utils import add_common_other_args_and_parse, get_call_generate
from sglang.utils import dump_state_text, read_jsonl

INVALID = -9999999
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, coordinates asynchronous or parallel execution, and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、协调异步或并行执行、加载、筛选或序列化基准测试数据集。

### Lines 18-26: Function `get_answer_value` / 函数 `get_answer_value`
```python
def get_answer_value(answer_str):
    answer_str = answer_str.replace(",", "")
    numbers = re.findall(r"\d+", answer_str)
    if len(numbers) < 1:
        return INVALID
    try:
        return ast.literal_eval(numbers[-1])
    except SyntaxError:
        return INVALID
```
**EN:** `get_answer_value` is a function that computes evaluation scores and aggregate statistics. It returns `INVALID` to the caller. Notable calls include `answer_str.replace`, `re.findall`, `len`.
**CN:** `get_answer_value` 是一个函数，用于计算评测分数与聚合统计结果。它会向调用方返回 `INVALID`。其中较关键的调用包括 `answer_str.replace`, `re.findall`, `len`。

### Lines 29-35: Function `most_frequent_number` / 函数 `most_frequent_number`
```python
def most_frequent_number(numbers):
    if not numbers:
        return None

    frequency = Counter(numbers)
    most_frequent = max(frequency, key=frequency.get)
    return most_frequent
```
**EN:** `most_frequent_number` is a function that implements the core logic for this scope. It returns `most_frequent` to the caller. Notable calls include `Counter`, `max`.
**CN:** `most_frequent_number` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `most_frequent`。其中较关键的调用包括 `Counter`, `max`。

### Lines 38-44: Top-level execution logic / 顶层执行逻辑
```python
USER_PREFIX = "[INST] "
USER_SUFFIX = " [/INST]"
ASSISTANT_PREFIX = ""
ASSISTANT_SUFFIX = " </s><s>"

# Use a low temp to make the results more deterministic and the comparison more fair.
temp = 0.001
```
**EN:** This block performs top-level coordination such as preparing inputs, selecting modes, or invoking previously defined helpers.
**CN:** 该代码块负责顶层协调，例如准备输入、选择模式或调用前面定义的辅助函数。

### Lines 47-59: Function `propose_plan` / 函数 `propose_plan`
```python
def propose_plan(s, question, num_branches, call_generate):
    s += (
        USER_PREFIX
        + """Please generate a high-level plan for solving the following question. As the first step, just say what method and idea you will use to solve the question. You can reorganize the information in the question. Do not do the actual calculation. Keep your response concise and within 80 words. Question: """
        + question
        + USER_SUFFIX
    )

    s += ASSISTANT_PREFIX
    comps = call_generate(
        s, max_tokens=256, temperature=temp, stop=None, n=num_branches
    )
    return [s + comp + ASSISTANT_SUFFIX for comp in comps]
```
**EN:** `propose_plan` is a function that processes tokenized prompts or decoded outputs. It returns `[s + comp + ASSISTANT_SUFFIX for comp in comps]` to the caller. Notable calls include `call_generate`.
**CN:** `propose_plan` 是一个函数，用于处理分词后的提示词或解码后的输出。它会向调用方返回 `[s + comp + ASSISTANT_SUFFIX for comp in comps]`。其中较关键的调用包括 `call_generate`。

### Lines 62-72: Function `execute_plan` / 函数 `execute_plan`
```python
def execute_plan(s, num_branches, call_generate):
    s += (
        USER_PREFIX
        + """The plan looks good! Now, use real numbers and do the calculation. Please solve the question step-by-step according to the high-level plan. Give me the final answer. Make your response short."""
        + USER_SUFFIX
    )
    s += ASSISTANT_PREFIX
    comps = call_generate(
        s, max_tokens=256, temperature=temp, stop=None, n=num_branches
    )
    return [s + comp + ASSISTANT_SUFFIX for comp in comps]
```
**EN:** `execute_plan` is a function that processes tokenized prompts or decoded outputs and computes evaluation scores and aggregate statistics. It returns `[s + comp + ASSISTANT_SUFFIX for comp in comps]` to the caller. Notable calls include `call_generate`.
**CN:** `execute_plan` 是一个函数，用于处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。它会向调用方返回 `[s + comp + ASSISTANT_SUFFIX for comp in comps]`。其中较关键的调用包括 `call_generate`。

### Lines 75-85: Function `reflect_solution` / 函数 `reflect_solution`
```python
def reflect_solution(s, num_branches, call_generate):
    s += (
        USER_PREFIX
        + """Okay. Now, evaluate your own solution and give it a score on a scale of 1 to 5. Please do rigorous check of the correctness."""
        + USER_SUFFIX
    )
    s += ASSISTANT_PREFIX
    comps = call_generate(
        s, max_tokens=256, temperature=temp, stop=None, n=num_branches
    )
    return [s + comp + ASSISTANT_SUFFIX for comp in comps]
```
**EN:** `reflect_solution` is a function that processes tokenized prompts or decoded outputs and computes evaluation scores and aggregate statistics. It returns `[s + comp + ASSISTANT_SUFFIX for comp in comps]` to the caller. Notable calls include `call_generate`.
**CN:** `reflect_solution` 是一个函数，用于处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。它会向调用方返回 `[s + comp + ASSISTANT_SUFFIX for comp in comps]`。其中较关键的调用包括 `call_generate`。

### Lines 88-98: Function `get_final_answer` / 函数 `get_final_answer`
```python
def get_final_answer(s, num_branches, call_generate):
    s += (
        USER_PREFIX
        + """Based on your reflection, do you change your mind? Now, give me the final answer after careful consideration."""
        + USER_SUFFIX
    )
    s += ASSISTANT_PREFIX
    comps = call_generate(
        s, max_tokens=256, temperature=temp, stop=None, n=num_branches
    )
    return [s + comp + ASSISTANT_SUFFIX for comp in comps]
```
**EN:** `get_final_answer` is a function that processes tokenized prompts or decoded outputs and computes evaluation scores and aggregate statistics. It returns `[s + comp + ASSISTANT_SUFFIX for comp in comps]` to the caller. Notable calls include `call_generate`.
**CN:** `get_final_answer` 是一个函数，用于处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。它会向调用方返回 `[s + comp + ASSISTANT_SUFFIX for comp in comps]`。其中较关键的调用包括 `call_generate`。

### Lines 101-119: Function `tree_search` / 函数 `tree_search`
```python
def tree_search(question, num_branches, call_generate):
    plan_forks = propose_plan("", question, num_branches, call_generate)

    sol_states = []
    for plan in plan_forks:
        forks = execute_plan(plan, num_branches, call_generate)
        sol_states.extend(forks)

    ref_states = []
    for sol in sol_states:
        forks = reflect_solution(sol, num_branches, call_generate)
        ref_states.extend(forks)

    solutions = []
    for sol in ref_states:
        ans = get_final_answer(sol, num_branches, call_generate)
        solutions.append(ans)

    return solutions
```
**EN:** `tree_search` is a function that computes evaluation scores and aggregate statistics. It returns `solutions` to the caller. Notable calls include `propose_plan`, `execute_plan`, `sol_states.extend`.
**CN:** `tree_search` 是一个函数，用于计算评测分数与聚合统计结果。它会向调用方返回 `solutions`。其中较关键的调用包括 `propose_plan`, `execute_plan`, `sol_states.extend`。

### Lines 122-214: Function `main` / 函数 `main`
```python
def main(args):
    lines = read_jsonl(args.data_path)

    # Construct prompts
    num_branches = 2
    questions = []
    labels = []
    for i in range(len(lines[: args.num_questions])):
        questions.append(lines[i]["question"])
        labels.append(get_answer_value(lines[i]["answer"]))
    assert all(l != INVALID for l in labels)
    arguments = [{"question": q, "num_branches": num_branches} for q in questions]

    # Select backend
    call_generate = get_call_generate(args)

    # Run requests
    states = [None] * len(questions)

    tic = time.perf_counter()
    if args.backend != "lmql":

        def get_one_answer(i):
            states[i] = tree_search(**arguments[i], call_generate=call_generate)

        if args.parallel == 1:
            for i in tqdm(range(len(questions))):
                get_one_answer(i)
        else:
            with ThreadPoolExecutor(args.parallel) as executor:
                list(
                    tqdm(
                        executor.map(get_one_answer, list(range(len(questions)))),
                        total=len(questions),
                    )
                )

    else:
        import asyncio

        from lmql_funcs import tree_search_async

        async def get_one_answer_async(i):
            states[i] = await tree_search_async(
                **arguments[i], call_generate=call_generate
            )

        batches = [
            [] for _ in range((len(questions) + args.parallel - 1) // args.parallel)
        ]
        for i in range(len(questions)):
            batches[i // args.parallel].append(i)

        loop = asyncio.get_event_loop()
        for bt in tqdm(batches):
            tasks = [get_one_answer_async(k) for k in bt]
            loop.run_until_complete(asyncio.gather(*tasks))

    latency = time.perf_counter() - tic

    answers_text = []
    for s in states:
        answers_text.append([x for xs in s for x in xs])

    preds = []
    for i in range(len(states)):
        answers = [get_answer_value(v) for v in answers_text[i]]
        preds.append(most_frequent_number(answers))

    # Compute accuracy
    acc = np.mean(np.array(preds) == np.array(labels))
    invalid = np.mean(np.array(preds) == INVALID)
    print(f"Latency: {latency:.3f}")
    print(f"Invalid: {invalid:.3f}")
    print(f"Accuracy: {acc:.3f}")

    # Write results
    dump_state_text(f"tmp_output_{args.backend}.txt", answers_text)

    with open(args.result_file, "a") as fout:
        value = {
            "task": "tree_of_thought_gsm8k",
            "backend": args.backend,
            "num_gpus": 1,
            "latency": round(latency, 3),
            "accuracy": round(acc, 3),
            "num_requests": args.num_questions,
            "other": {
                "num_questions": args.num_questions,
                "parallel": args.parallel,
            },
        }
        fout.write(json.dumps(value) + "\n")
```
**EN:** `main` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. Notable calls include `read_jsonl`, `range`, `all`.
**CN:** `main` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。其中较关键的调用包括 `read_jsonl`, `range`, `all`。

### Lines 217-222: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--data-path", type=str, default="test.jsonl")
    parser.add_argument("--num-questions", type=int, default=200)
    args = add_common_other_args_and_parse(parser)
    main(args)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置、加载、筛选或序列化基准测试数据集。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Concurrency / 并发**: Coordinates multiple requests or workers at the same time. / 同时协调多个请求或工作单元。
- **Serving APIs / 服务接口**: Talks to HTTP or OpenAI-compatible model-serving endpoints. / 与 HTTP 或 OpenAI 兼容模型服务端点交互。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `ast`, `json`, `re`, `time`, `collections`, `concurrent.futures`, `asyncio`
- **Third-party / 第三方依赖**: `numpy`, `tqdm`, `lmql_funcs`
- **Internal / 项目内部依赖**: `sglang.test.test_utils`, `sglang.utils`
