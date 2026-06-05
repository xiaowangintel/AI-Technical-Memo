# bench_other.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/json_jump_forward/bench_other.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on json jump forward other. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and measures runtime latency, throughput, or other benchmark metrics. / 该 Python 模块聚焦于 json jump forward other 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标。

## Line-by-Line Analysis / 逐行分析
### Lines 1-41: Imports and setup / 导入与初始化
```python
import argparse
import json
import time
from concurrent.futures import ThreadPoolExecutor
from functools import partial

import guidance
from tqdm import tqdm

from sglang.test.test_utils import add_common_other_args_and_parse, get_call_generate
from sglang.utils import dump_state_text, read_jsonl

# there are some FSM bugs with json regex converted from pydantic model
# here use a string regex instead
# regex_string = build_regex_from_object(HarryPoterRole)
character_regex = (
    r"""\{\n"""
    + r"""    "name": "[\w\d\s]{1,16}",\n"""
    + r"""    "house": "(Gryffindor|Slytherin|Ravenclaw|Hufflepuff)",\n"""
    + r"""    "blood status": "(Pure-blood|Half-blood|Muggle-born)",\n"""
    + r"""    "occupation": "(student|teacher|auror|ministry of magic|death eater|order of the phoenix)",\n"""
    + r"""    "wand": \{\n"""
    + r"""        "wood": "[\w\d\s]{1,16}",\n"""
    + r"""        "core": "[\w\d\s]{1,16}",\n"""
    + r"""        "length": [0-9]{1,2}\.[0-9]{0,2}\n"""
    + r"""    \},\n"""
    + r"""    "alive": "(Alive|Deceased)",\n"""
    + r"""    "patronus": "[\w\d\s]{1,16}",\n"""
    + r"""    "bogart": "[\w\d\s]{1,16}"\n"""
    + r"""\}"""
)

city_regex = (
    r"""\{\n"""
    + r"""  "name": "[\w\d\s]{1,16}",\n"""
    + r"""  "country": "[\w\d\s]{1,16}",\n"""
    + r"""  "latitude": [-+]?[0-9]*\.?[0-9]{0,2},\n"""
    + r"""  "population": [-+]?[0-9]{1,9},\n"""
    + r"""  "top 3 landmarks": \["[\w\d\s]{1,16}", "[\w\d\s]{1,16}", "[\w\d\s]{1,16}"\]\n"""
    + r"""\}"""
)
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration, coordinates asynchronous or parallel execution, and loads, filters, or serializes benchmark datasets.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置、协调异步或并行执行、加载、筛选或序列化基准测试数据集。

### Lines 44-47: Function `character_gen` / 函数 `character_gen`
```python
def character_gen(name, generate):
    s = name + " is a character in Harry Potter. Please fill in the following information about this character.\n"
    s += generate(s, max_tokens=256, regex=character_regex)
    return s
```
**EN:** `character_gen` is a function that processes tokenized prompts or decoded outputs. It returns `s` to the caller. Notable calls include `generate`.
**CN:** `character_gen` 是一个函数，用于处理分词后的提示词或解码后的输出。它会向调用方返回 `s`。其中较关键的调用包括 `generate`。

### Lines 51-56: Function `city_gen` / 函数 `city_gen`
```python
def city_gen(document, generate):
    s = "Please extract the information of a city from the following wikipedia page.\n"
    s += "Page begin.\n" + document + "Page end.\n"
    s += "Here is the name, country, and symbol of the city in JSON format.\n"
    s += generate(s, max_tokens=256, regex=city_regex)
    return s
```
**EN:** `city_gen` is a function that loads, filters, or serializes benchmark datasets and processes tokenized prompts or decoded outputs. It returns `s` to the caller. Notable calls include `generate`.
**CN:** `city_gen` 是一个函数，用于加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出。它会向调用方返回 `s`。其中较关键的调用包括 `generate`。

### Lines 61-82: Function `character_maker` / 函数 `character_maker`
```python
def character_maker(lm, name):
    regex_str_no_quote = r"[\w\d\s]+"
    regex_float = r"[0-9]+\.[0-9]+"
    lm += f"""\
    {name} is a character in Harry Potter. Please fill in the following information about this character.
    {{
        "name": "{guidance.gen("name", max_tokens=16, regex=regex_str_no_quote)}",
        "house": "{guidance.select(options=['Gryffindor', 'Slytherin', 'Ravenclaw', 'Hufflepuff'], name='house')}",
        "blood status": "{guidance.select(options=['Pure-blood', 'Half-blood', 'Muggle-born'], name='blood status')}",
        "occupation": "{guidance.select(options=['student', 'teacher', 'auror', 'ministry of magic', 'death eater', 'order of the phoenix'], name='occupation')}",
        "wand": {{
            "wood": "{guidance.gen("wood", max_tokens=16, regex=regex_str_no_quote)}",
            "core": "{guidance.gen('core', max_tokens=16, regex=regex_str_no_quote)}",
            "length": {guidance.gen('length', max_tokens=10, regex=regex_float)}
        }},
        "alive": "{guidance.select(options=['Alive', 'Deceased'], name='alive')}",
        "patronus": "{guidance.gen('patronus', max_tokens=16, regex=regex_str_no_quote)}",
        "bogart": "{guidance.gen('bogart', max_tokens=16, regex=regex_str_no_quote)}"
    }}
    """

    return lm
```
**EN:** `character_maker` is a function that processes tokenized prompts or decoded outputs. It returns `lm` to the caller. Notable calls include `guidance.gen`, `guidance.select`.
**CN:** `character_maker` 是一个函数，用于处理分词后的提示词或解码后的输出。它会向调用方返回 `lm`。其中较关键的调用包括 `guidance.gen`, `guidance.select`。

### Lines 85-105: Async function `call_generate_lmql` / 异步函数 `call_generate_lmql`
```python
async def call_generate_lmql(
    prompt, temperature, max_tokens, regex, max_len=4096, model=None, **kwargs
):
    assert model is not None
    import lmql

    @lmql.query(model=model)
    async def program(question, max_tokens, regex):
        '''lmql
        """{question}[ANSWER]""" where len(TOKENS(ANSWER)) < max_tokens and REGEX(ANSWER, regex)
        return ANSWER
        '''

    return await program(
        question=prompt,
        temperature=temperature,
        max_tokens=max_tokens,
        max_len=max_len,
        regex=regex,
        **kwargs,
    )
```
**EN:** `call_generate_lmql` is a async function that coordinates asynchronous or parallel execution, processes tokenized prompts or decoded outputs, and computes evaluation scores and aggregate statistics. It returns `await program(question=prompt, temperature=temperature, max_tokens=max_tokens, max_len=max_len, r...` to the caller. Notable calls include `lmql.query`, `program`.
**CN:** `call_generate_lmql` 是一个异步函数，用于协调异步或并行执行、处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。它会向调用方返回 `await program(question=prompt, temperature=temperature, max_tokens=max_tokens, max_len=max_len, r...`。其中较关键的调用包括 `lmql.query`, `program`。

### Lines 109-129: Function `city_maker` / 函数 `city_maker`
```python
def city_maker(lm, document):
    regex_str_no_quote = r"[\w\d\s]+"
    regex_float = r"[0-9]+\.[0-9]+"
    lm += f"""\
    Please extract the information of a city from the following wikipedia page.
    Page begin.
    {document}
    Page end.
    Here is the name, country, and symbol of the city in JSON format.
    {{
        "name": "{guidance.gen("name", max_tokens=16, regex=regex_str_no_quote)}",
        "country": "{guidance.gen("country", max_tokens=16, regex=regex_str_no_quote)}",
        "latitude": {guidance.gen("latitude", max_tokens=10, regex=regex_float)},
        "population": {guidance.gen("population", max_tokens=10, regex=r"[0-9]+")},
        "top 3 landmarks": [
            "{guidance.gen("landmark1", max_tokens=16, regex=regex_str_no_quote)}", "{guidance.gen("landmark2", max_tokens=16, regex=regex_str_no_quote)}", "{guidance.gen("landmark3", max_tokens=16, regex=regex_str_no_quote)}"
        ]
    }}
    """

    return lm
```
**EN:** `city_maker` is a function that loads, filters, or serializes benchmark datasets and processes tokenized prompts or decoded outputs. It returns `lm` to the caller. Notable calls include `guidance.gen`.
**CN:** `city_maker` 是一个函数，用于加载、筛选或序列化基准测试数据集、处理分词后的提示词或解码后的输出。它会向调用方返回 `lm`。其中较关键的调用包括 `guidance.gen`。

### Lines 132-207: Function `bench_character` / 函数 `bench_character`
```python
def bench_character(args):
    arguments = []
    with open(args.data_path, "r") as f:
        for line in f:
            arguments.append({"name": line.strip()})
    arguments = arguments[: args.num_jsons]

    states = [None] * len(arguments)

    # Select backend
    if args.backend == "outlines":
        call_generate = partial(get_call_generate(args), temperature=0)

        def get_one_answer(i):
            states[i] = character_gen(**arguments[i], generate=call_generate)

    elif args.backend == "guidance":
        model = guidance.models.LlamaCpp(
            args.model_path,
            n_gpu_layers=-1,
            n_ctx=args.n_ctx,
        )

        def get_one_answer(i):
            lm = model + character_maker(**arguments[i])
            states[i] = lm

    elif args.backend == "lmql":
        import asyncio

        import lmql

        model = lmql.model(args.model_path, endpoint=f"{args.host}:{args.port}")
        call_generate = partial(
            call_generate_lmql,
            model=model,
            max_tokens=256,
            regex=character_regex,
        )

        async def get_one_answer_async(i):
            states[i] = await call_generate(prompt=arguments[i]["name"], temperature=0)

    else:
        raise ValueError(f"Invalid backend: {args.backend}")

    tic = time.perf_counter()

    if args.backend != "lmql":
        if args.parallel == 1:
            for i in tqdm(range(len(arguments))):
                get_one_answer(i)
        else:
            with ThreadPoolExecutor(args.parallel) as executor:
                rets = list(
                    tqdm(
                        executor.map(get_one_answer, list(range(len(arguments)))),
                        total=len(arguments),
                    )
                )
                for _ in rets:
                    pass
    else:
        batches = []
        for i in range(0, len(arguments), args.parallel):
            batches.append(list(range(i, min(i + args.parallel, len(arguments)))))
        loop = asyncio.get_event_loop()

        for bt in tqdm(batches):
            loop.run_until_complete(
                asyncio.gather(*[get_one_answer_async(i) for i in bt])
            )

    latency = time.perf_counter() - tic

    return states, latency
```
**EN:** `bench_character` is a function that sends requests to serving or OpenAI-compatible APIs, measures runtime latency, throughput, or other benchmark metrics, and coordinates asynchronous or parallel execution. It returns `(states, latency)` to the caller. Notable calls include `time.perf_counter`, `open`, `len`.
**CN:** `bench_character` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行。它会向调用方返回 `(states, latency)`。其中较关键的调用包括 `time.perf_counter`, `open`, `len`。

### Lines 210-251: Function `bench_city_doc` / 函数 `bench_city_doc`
```python
def bench_city_doc(args):
    arguments = []
    for line in read_jsonl(args.data_path):
        arguments.append({"document": line["document"]})
    arguments = arguments[: args.num_jsons]

    states = [None] * len(arguments)

    # Select backend
    if args.backend == "outlines":
        call_generate = partial(get_call_generate(args), temperature=0)

        def get_one_answer(i):
            states[i] = city_gen(**arguments[i], generate=call_generate)

    elif args.backend == "guidance":
        model = guidance.models.LlamaCpp(
            args.model_path,
            n_gpu_layers=-1,
            n_ctx=args.n_ctx,
        )

        def get_one_answer(i):
            lm = model + city_maker(**arguments[i])
            states[i] = lm

    else:
        raise ValueError(f"Invalid backend: {args.backend}")

    tic = time.perf_counter()
    if args.parallel == 1:
        for i in tqdm(range(len(arguments))):
            get_one_answer(i)
    else:
        with ThreadPoolExecutor(args.parallel) as executor:
            rets = executor.map(get_one_answer, list(range(len(arguments))))
            for _ in rets:
                pass

    latency = time.perf_counter() - tic

    return states, latency
```
**EN:** `bench_city_doc` is a function that measures runtime latency, throughput, or other benchmark metrics, coordinates asynchronous or parallel execution, and loads, filters, or serializes benchmark datasets. It returns `(states, latency)` to the caller. Notable calls include `read_jsonl`, `time.perf_counter`, `arguments.append`.
**CN:** `bench_city_doc` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、协调异步或并行执行、加载、筛选或序列化基准测试数据集。它会向调用方返回 `(states, latency)`。其中较关键的调用包括 `read_jsonl`, `time.perf_counter`, `arguments.append`。

### Lines 254-277: Function `main` / 函数 `main`
```python
def main(args):
    if args.mode == "character":
        args.data_path = "dataset.txt"
        states, latency = bench_character(args)
    elif args.mode == "city":
        args.data_path = "questions.jsonl"
        states, latency = bench_city_doc(args)

    # Compute accuracy
    print(f"Latency: {latency:.3f}")

    # Write results
    dump_state_text(f"tmp_output_{args.backend}_{args.mode}.txt", states)

    with open(args.result_file, "a") as fout:
        value = {
            "task": "json_jump_forward",
            "backend": args.backend,
            "latency": round(latency, 3),
            "num_jsons": args.num_jsons,
            "mode": args.mode,
            "parallel": args.parallel,
        }
        fout.write(json.dumps(value) + "\n")
```
**EN:** `main` is a function that measures runtime latency, throughput, or other benchmark metrics, loads, filters, or serializes benchmark datasets, and computes evaluation scores and aggregate statistics. Notable calls include `print`, `dump_state_text`, `bench_character`.
**CN:** `main` 是一个函数，用于测量运行时延迟、吞吐或其他基准指标、加载、筛选或序列化基准测试数据集、计算评测分数与聚合统计结果。其中较关键的调用包括 `print`, `dump_state_text`, `bench_character`。

### Lines 280-288: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--data-path", type=str)
    parser.add_argument("--num-jsons", type=int, default=50)
    parser.add_argument(
        "--mode", type=str, default="character", choices=["character", "city"]
    )
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
- **Third-party / 第三方依赖**: `guidance`, `tqdm`, `lmql`
- **Internal / 项目内部依赖**: `sglang.test.test_utils`, `sglang.utils`
