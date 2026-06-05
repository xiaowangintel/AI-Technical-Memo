# bench_speculative.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/playground/bench_speculative.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `bench_speculative` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Python 模块用于支撑 SGLang 中的 `bench_speculative` 流程，主要负责CI 编排、测试执行。它属于 `playground` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""
Usage:
# single GPU
python3 bench_speculative.py --model-path meta-llama/Llama-2-7b-chat-hf --speculative-draft-model-path lmsys/sglang-EAGLE-llama2-chat-7B

# multiple GPU
python3 bench_speculative.py --model-path deepseek-ai/DeepSeek-V3 --speculative-draft-model-path lmsys/DeepSeek-V3-NextN --tp-size 8 --trust-remote-code --batch-size 1 4 8 16 32 --steps 0 1 2 --topk 0 1 2 4 --num_draft_tokens 0 2 4 8
"""
```
**EN:** Usage: # single GPU python3 bench_speculative.py --model-path meta-llama/Llama-2-7b-chat-hf --speculative-draft-model-path lmsys/sglang-EAGLE-llama2-chat-7B # multiple GPU python3 bench_speculative.py --model-path deepseek-ai/DeepSeek-V3 --speculative-draft-model-path lmsys/DeepSeek-V3-NextN --tp-size 8 --trust-remote-code --batch-size 1 4 8 16 32 --steps 0 1 2 --topk 0 1 2 4 --num_draft_tokens 0 2 4 8
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 10-30: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import argparse
import asyncio
import json
import os
import time
from types import SimpleNamespace
from typing import List

import numpy as np
import requests
from transformers import AutoTokenizer

from sglang.bench_serving import benchmark, set_global_args
from sglang.benchmark.datasets import DatasetRow
from sglang.benchmark.datasets.mmmu import sample_mmmu_requests
from sglang.srt.server_args import ServerArgs
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    kill_process_tree,
    popen_launch_server,
)
```
**EN:** This block loads argparse, asyncio, json, os, time, types, typing, numpy. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 argparse, asyncio, json, os, time, types, typing, numpy。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 33-35: Defines the `node0_print` routine / 定义 `node0_print` 例程
```python
def node0_print(msg):
    if server_args.node_rank == 0:
        print(msg)
```
**EN:** This block defines `node0_print`. It accepts 1 parameter(s): msg. Internally it emits status messages.
**CN:** 该代码块定义了 `node0_print`。它接收 1 个参数：msg。内部会输出状态信息。

### Lines 38-47: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
prompts = [
    "Human: Give me a fully functional FastAPI server. Show the full, long python code without stop.\n\nAssistant:",
    "Human: Imagine you are an experienced Ethereum developer tasked with creating a smart contract for a blockchain messenger. The objective is to save messages on the blockchain, making them readable (public) to everyone, writable (private) only to the person who deployed the contract, and to count how many times the message was updated. Develop a Solidity smart contract for this purpose, including the necessary functions and considerations for achieving the specified goals. Please provide the code and any relevant explanations to ensure a clear understanding of the implementation.\n\nAssistant:",
    "Human: Write a travel blog post to Hawaii.\n\nAssistant:",
    "Human: I want you to act as an English translator, spelling corrector and improver. I will speak to you in any language and you will detect the language, translate it and answer in the corrected and improved version of my text, in English. I want you to replace my simplified A0-level words and sentences with more beautiful and elegant, upper level English words and sentences. Keep the meaning same, but make them more literary. My first sentence is 'istanbulu cok seviyom burada olmak cok guzel'. Answer in more than 5000 words.\n\nAssistant:",
    "Human: I want you to act as a storyteller. You will come up with entertaining stories that are engaging, imaginative and captivating for the audience. It can be fairy tales, educational stories or any other type of stories which has the potential to capture people's attention and imagination. Depending on the target audience, you may choose specific themes or topics for your storytelling session e.g., if it’s children then you can talk about animals; If it’s adults then history-based tales might engage them better etc. Answer in more than 5000 words. My first request is 'I need an interesting story on perseverance.'\n\nAssistant:",
    "Human: Solve x^2 = -1. Think step-by-step. Give me a long detailed explanation. \n\nAssistant:",
    "Human: Tell me about the president of the USA in wikipedia style.\n\nAssistant:",
    "Human: Hello? Who are you? Write code, math, and poem to explanin yourself.\n\nAssistant:",
]
```
**EN:** This section defines shared constants, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 共享常量，用于把可复用的默认值集中在模块顶部。

### Lines 50-52: Declares the `FakeTokenizer` class / 声明 `FakeTokenizer` 类
```python
class FakeTokenizer:
    def encode(self, text: str, add_special_tokens: bool = False):
        return []
```
**EN:** This block declares the `FakeTokenizer` class. The class groups 1 method(s), including encode.
**CN:** 该代码块声明了 `FakeTokenizer` 类。该类集中定义了 1 个方法，包括 encode。

### Lines 55-134: Defines the `send_one_batch` routine / 定义 `send_one_batch` 例程
```python
def send_one_batch(base_url, num_prompts, batch_size, processor, is_multimodal):
    # format: (prompt, input_len, output len). We set input_len as a dummy value 0.
    if is_multimodal:
        backend = "sglang-oai-chat"
        api_url = f"{base_url}/v1/chat/completions"
        input_requests = sample_mmmu_requests(
            num_prompts,
            processor,
            backend=backend,
            fixed_output_len=512,
        )
        tokenizer = processor.tokenizer
    else:
        padded_prompts = (prompts * ((num_prompts + len(prompts) - 1) // len(prompts)))[
            :num_prompts
        ]
        input_requests: List[DatasetRow] = [
            DatasetRow(p, 0, 512) for p in padded_prompts
        ]
        backend = "sglang"
        api_url = f"{base_url}/generate"
        tokenizer = processor

    # We need to set some dummy values in order to call `benchmark` below.
    args = SimpleNamespace(
        disable_ignore_eos=False,
        disable_stream=False,
        return_logprob=False,
        return_routed_experts=False,
        plot_throughput=False,
        backend=backend,
        dataset_name="custom",
        num_prompts=None,
        sharegpt_output_len=None,
        random_input_len=None,
        random_output_len=None,
        random_range_ratio=None,
        output_file=None,
        warmup_requests=1,
        output_details=False,
    )
    set_global_args(args)

    # Run benchmark
    results = asyncio.run(
        benchmark(
            backend=backend,
            api_url=api_url,
            base_url=base_url,
            model_id="default",
            tokenizer=tokenizer,
            input_requests=input_requests,
            request_rate=float("inf"),
            max_concurrency=batch_size,
            disable_tqdm=False,
            lora_names=None,
            lora_request_distribution=None,
            lora_zipf_alpha=None,
            extra_request_body={},
            profile=None,
        )
    )

    assert results["completed"] == len(input_requests)
    acc_length = results["accept_length"] or 1.0
    avg_output_token = results["total_output_tokens"] / results["completed"]

    server_info = requests.get(base_url + "/server_info").json()
    # We use 20% percentile instead of median on purpose
    step_time = np.percentile(
        server_info["internal_states"][0]["step_time_dict"][str(batch_size)], 20
    )
    speed = 1 / step_time * acc_length

    return (
        round(acc_length, 3),
        round(step_time, 5),
        round(speed, 3),
        avg_output_token,
    )
```
**EN:** This block defines `send_one_batch`. It accepts 5 parameter(s): base_url, num_prompts, batch_size, processor, is_multimodal. Internally it executes external commands.
**CN:** 该代码块定义了 `send_one_batch`。它接收 5 个参数：base_url, num_prompts, batch_size, processor, is_multimodal。内部会执行外部命令。

### Lines 137-279: Defines the `main` routine / 定义 `main` 例程
```python
def main(args, server_args):
    base_url = "http://127.0.0.1:20000"

    configs = []
    for batch_size in args.batch_size:
        for steps in args.steps:
            for topk in args.topk:
                for num_draft_tokens in args.num_draft_tokens:
                    if steps * topk + 1 < num_draft_tokens:
                        continue

                    if (steps == 0 or topk == 0 or num_draft_tokens == 0) and (
                        steps + topk + num_draft_tokens != 0
                    ):
                        # steps == 0 and topk == 0 and num_draft_tokens == 0 is a special case for non-speculative decoding.
                        continue

                    configs.append((batch_size, steps, topk, num_draft_tokens))

    for i in range(args.start, args.end or len(configs)):
        batch_size, steps, topk, num_draft_tokens = configs[i]

        node0_print(
            f"Start {i=}: {batch_size=}, {steps=}, {topk=}, {num_draft_tokens=}"
        )

        # Create an LLM.
        if steps == 0:
            other_args = []
        else:
            other_args = [
                "--speculative-num-steps",
                steps,
                "--speculative-eagle-topk",
                topk,
                "--speculative-num-draft-tokens",
                num_draft_tokens,
            ]
            if server_args.speculative_draft_model_path is not None:
                other_args.extend(
                    [
                        "--speculative-draft-model-path",
                        server_args.speculative_draft_model_path,
                        "--speculative-algorithm",
                        server_args.speculative_algorithm,
                    ]
                )

        other_args.extend(
            [
                "--cuda-graph-max-bs",
                batch_size,
                "--mem-fraction-static",
                server_args.mem_fraction_static,
                "--tp-size",
                server_args.tp_size,
                "--max-running-requests",
                batch_size,
            ]
        )

        if server_args.trust_remote_code:
            other_args.extend(
                [
                    "--trust-remote-code",
                ]
            )

        if server_args.attention_backend:
            other_args.extend(
                [
                    "--attention-backend",
                    server_args.attention_backend,
                ]
            )

        if server_args.quantization:
            other_args.extend(
                [
                    "--quantization",
                    server_args.quantization,
                ]
            )

        process = popen_launch_server(
            args.model_path,
            base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=other_args,
            env={
                "SGLANG_RECORD_STEP_TIME": "1",
                **os.environ,
            },
        )

        if args.is_multimodal:
            from transformers import AutoProcessor

            processor = AutoProcessor.from_pretrained(
                args.model_path, trust_remote_code=server_args.trust_remote_code
            )
        else:
            processor = AutoTokenizer.from_pretrained(
                args.model_path, trust_remote_code=server_args.trust_remote_code
            )

        try:
            # Warmup
            send_one_batch(
                base_url, batch_size, batch_size, processor, args.is_multimodal
            )

            # Benchmark
            acc_length, step_time, speed, completion_tokens = send_one_batch(
                base_url,
                max(args.num_prompts, batch_size),
                batch_size,
                processor,
                args.is_multimodal,
            )
        finally:
            kill_process_tree(process.pid)

        node0_print(
            f"Finish {i=}: {batch_size=}, {steps=}, {topk=}, {num_draft_tokens=}, {speed=:.2f} token/s, step_time={step_time * 1000:.2f} ms"
        )

        record = {
            "batch_size": batch_size,
            "steps": steps,
            "topk": topk,
            "num_draft_tokens": num_draft_tokens,
            "acc_length": acc_length,
            "step_time": step_time,
            "speed": speed,
            "completion_tokens": completion_tokens,
        }

        with open(args.output, "a") as fout:
            fout.write(json.dumps(record) + "\n")

        # Wait for the server to shutdown
        time.sleep(5)
```
**EN:** This block defines `main`. It accepts 2 parameter(s): args, server_args. Internally it reads or writes files.
**CN:** 该代码块定义了 `main`。它接收 2 个参数：args, server_args。内部会读写文件。

### Lines 284-319: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    ServerArgs.add_cli_args(parser)
    parser.add_argument(
        "--batch-size",
        type=int,
        nargs="+",
        default=(1, 2, 4, 8, 16),
    )
    parser.add_argument(
        "--steps",
        type=int,
        nargs="+",
        default=(0, 1, 3, 5, 7),  # use (0, 1, 2, 3, 4) for large batch size
    )
    parser.add_argument(
        "--topk",
        type=int,
        nargs="+",
        default=(0, 1, 2, 4, 8),
    )
    parser.add_argument(
        "--num_draft_tokens",
        type=int,
        nargs="+",
        default=(0, 2, 4, 8, 16, 32),  # use (0, 2, 4, 8) for large batch size
    )
    parser.add_argument("--num-prompts", type=int, default=16)
    parser.add_argument("--start", type=int, default=0)
    parser.add_argument("--end", type=int)
    parser.add_argument("--output", type=str, default="output.jsonl")
    parser.add_argument("--is-multimodal", action="store_true", default=False)
    args = parser.parse_args()
    server_args: ServerArgs = ServerArgs.from_cli_args(args)

    main(args, server_args)
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It parses CLI arguments.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会解析命令行参数。

## Key Concepts / 关键概念
- **Subprocess orchestration** / 子进程编排
- **CLI parsing** / 命令行解析
- **Environment management** / 环境管理
- **Process control** / 进程控制
- **Structured data handling** / 结构化数据处理

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `asyncio`, `json`, `os`, `time`, `types`, `typing`
- **Third-party modules / 第三方模块**: `numpy`, `requests`, `transformers`
- **Repository-local imports / 仓库内导入**: `sglang.bench_serving`, `sglang.benchmark.datasets`, `sglang.benchmark.datasets.mmmu`, `sglang.srt.server_args`, `sglang.test.test_utils`
