# bench_other.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/llm_judge/bench_other.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on llm judge other. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 llm judge other 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: Imports and setup / 导入与初始化
```python
import argparse
import json
import time
from concurrent.futures import ThreadPoolExecutor
from functools import partial

from tqdm import tqdm

from sglang.test.test_utils import add_common_other_args_and_parse, get_call_generate
from sglang.utils import dump_state_text, read_jsonl

system_prompt = "Please serve as an impartial judge and rigorously evaluate the quality of the following article. Apply the most stringent standards possible, showing no leniency."

dimension_prompts = [
    "Content: This refers to the essences of the essay. The substance should be well researched, accurate, relevant to the topic and should show a thorough understanding of the subject. The essay should also reflect a clear goal or purpose.",
    "Organization and Structure: An essay needs to be properly structured with a clear introduction, body, and conclusion. The essay should flow naturally, with one paragraph leading seamlessly into the next.",
    "Argument and Analysis: The argument made in the essay should be logical, coherent and clearly articulated. Each point made should be backed up by solid evidence and thorough analysis.",
    "Clarity and Precision: The essay should be written in a clear and concise manner. The points made should be easily understood by the reader. The language used should also be precise and unambiguous.",
    "Grammar and Punctuation: Proper use of grammar and punctuation is vital in an academic essay. Errors in grammar and punctuation not only distract the reader but can also negatively impact the meaning and interpretation of the content.",
    "Referencing and Citation: An essay should contain proper citations and references for all sources used. This not only prevents accusations of plagiarism but also gives credit to the authors of the works that have contributed to the essay. The citation should adhere to a specific format as required by the academic institution or specified by the professor.",
]
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, coordinates asynchronous or parallel execution, and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、协调异步或并行执行、加载、筛选或序列化基准测试数据集。

### Lines 24-49: Function `multi_dimension_judge` / 函数 `multi_dimension_judge`
```python
def multi_dimension_judge(article, generate):
    s = system_prompt
    s += "\n```\n" + article + "\n```\n\n"

    judges = []
    for i in range(len(dimension_prompts)):
        comp = generate(
            s
            + "USER: Please judge the quality based on the following metric. "
            + dimension_prompts[i]
            + " Please provide a single-paragraph judgement. "
            + "Focus on the provided metric and do not say other things. "
            'End your judgement paragraph with the word "END"\nJUDGE:',
            max_tokens=256,
            stop="END",
        )
        judges.append(comp)

    s += "I will judge the quality based on the following metrics.\n"
    for i in range(len(dimension_prompts)):
        s += dimension_prompts[i].split(":")[0] + ": " + judges[i].strip() + "\n"

    s += "In summary, on a scale of 1 to 10, I would give the article a score of"
    s += generate(s, max_tokens=2, stop=None)

    return s
```
**EN:** `multi_dimension_judge` is a function that processes tokenized prompts or decoded outputs and computes evaluation scores and aggregate statistics. It returns `s` to the caller. Notable calls include `range`, `generate`, `len`.
**CN:** `multi_dimension_judge` 是一个函数，用于处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。它会向调用方返回 `s`。其中较关键的调用包括 `range`, `generate`, `len`。

### Lines 52-77: Async function `multi_dimension_judge_async` / 异步函数 `multi_dimension_judge_async`
```python
async def multi_dimension_judge_async(article, generate):
    s = system_prompt
    s += "\n```\n" + article + "\n```\n\n"

    judges = []
    for i in range(len(dimension_prompts)):
        comp = await generate(
            s
            + "USER: Please judge the quality based on the following metric. "
            + dimension_prompts[i]
            + " Please provide a single-paragraph judgement. "
            + "Focus on the provided metric and do not say other things. "
            'End your judgement paragraph with the word "END"\nJUDGE:',
            max_tokens=256,
            stop="END",
        )
        judges.append(comp)

    s += "I will judge the quality based on the following metrics.\n"
    for i in range(len(dimension_prompts)):
        s += dimension_prompts[i].split(":")[0] + ": " + judges[i].strip() + "\n"

    s += "In summary, on a scale of 1 to 10, I would give the article a score of"
    s += await generate(s, max_tokens=2, stop=None)

    return s
```
**EN:** `multi_dimension_judge_async` is a async function that coordinates asynchronous or parallel execution, processes tokenized prompts or decoded outputs, and computes evaluation scores and aggregate statistics. It returns `s` to the caller. Notable calls include `range`, `len`, `judges.append`.
**CN:** `multi_dimension_judge_async` 是一个异步函数，用于协调异步或并行执行、处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。它会向调用方返回 `s`。其中较关键的调用包括 `range`, `len`, `judges.append`。

### Lines 80-143: Function `main` / 函数 `main`
```python
def main(args):
    lines = read_jsonl(args.data_path)[: args.num_questions]
    states = [None] * len(lines)

    # Select backend
    call_generate = partial(get_call_generate(args), temperature=0)

    # Run requests
    tic = time.perf_counter()

    if args.backend != "lmql":

        def get_one_answer(i):
            states[i] = multi_dimension_judge(lines[i], call_generate)

        if args.parallel == 1:
            for i in tqdm(range(len(lines))):
                get_one_answer(i)
        else:
            with ThreadPoolExecutor(args.parallel) as executor:
                list(
                    tqdm(
                        executor.map(get_one_answer, list(range(len(lines)))),
                        total=len(lines),
                    )
                )

    else:
        import asyncio

        async def get_one_answer_async(i):
            states[i] = await multi_dimension_judge_async(lines[i], call_generate)

        batches = []
        for i in range(0, len(lines), args.parallel):
            batches.append(list(range(i, min(i + args.parallel, len(lines)))))

        loop = asyncio.get_event_loop()
        for bt in tqdm(batches):
            loop.run_until_complete(
                asyncio.gather(*[get_one_answer_async(i) for i in bt])
            )

    latency = time.perf_counter() - tic

    # Compute accuracy
    print(f"Latency: {latency:.3f}")

    # Write results
    dump_state_text(f"tmp_output_{args.backend}.txt", states)

    with open(args.result_file, "a") as fout:
        value = {
            "task": "llm_judge",
            "backend": args.backend,
            "num_gpus": 1,
            "latency": round(latency, 3),
            "num_requests": args.num_questions,
            "other": {
                "num_questions": args.num_questions,
                "parallel": args.parallel,
            },
        }
        fout.write(json.dumps(value) + "\n")
```
**EN:** `main` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. Notable calls include `partial`, `time.perf_counter`, `print`.
**CN:** `main` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。其中较关键的调用包括 `partial`, `time.perf_counter`, `print`。

### Lines 146-151: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--data-path", type=str, default="articles.jsonl")
    parser.add_argument("--num-questions", type=int, default=20)
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
- **Standard library / 标准库**: `argparse`, `json`, `time`, `concurrent.futures`, `functools`, `asyncio`
- **Third-party / 第三方依赖**: `tqdm`
- **Internal / 项目内部依赖**: `sglang.test.test_utils`, `sglang.utils`
