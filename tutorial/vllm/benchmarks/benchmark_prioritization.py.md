# benchmark_prioritization.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/benchmark_prioritization.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, scheduler prioritization benchmarks, centered around `get_random_flag`, `sample_requests`, `run_vllm`, `main`. / 实现与基准测试编排、调度优先级基准相关的逻辑，核心符号包括 `get_random_flag`, `sample_requests`, `run_vllm`, `main`。

## Line-by-Line Analysis / 逐行分析
### Module overview (lines 3-3)
```python
"""Benchmark offline prioritization."""
```
**EN:** The module docstring introduces Benchmark offline prioritization. It gives readers the context needed before the implementation details start.
**CN:** 模块文档字符串先说明了 Benchmark offline prioritization 的背景与用途，在进入实现细节前先建立阅读上下文。

### Imports (lines 5-13)
```python
import argparse
import json
import random
import time

from transformers import AutoTokenizer, PreTrainedTokenizerBase

from vllm.engine.arg_utils import EngineArgs
from vllm.utils.argparse_utils import FlexibleArgumentParser
```
**EN:** This block gathers standard-library helpers such as `argparse`, `json`, `random`, `time`; third-party packages such as `transformers`; project-local modules such as `vllm.engine.arg_utils`, `vllm.utils.argparse_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `argparse`, `json`, `random`, `time`；第三方依赖，如 `transformers`；项目内部模块，如 `vllm.engine.arg_utils`, `vllm.utils.argparse_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Function `get_random_flag` (lines 17-18)
```python
def get_random_flag():
    return 0 if random.random() < 0.5 else 1
```
**EN:** `get_random_flag` implements a helper used by `benchmark_prioritization.py`. It mainly works with no explicit parameters and relies on `random.random` plus value production to move data through this part of the benchmark pipeline.
**CN:** `get_random_flag` 负责实现 `benchmark_prioritization.py` 使用的辅助逻辑。 它主要处理 无显式参数，并结合 `random.random` 以及 结果返回 来完成这一段基准测试流程。

### Function `sample_requests` (lines 21-70)
```python
def sample_requests(
    dataset_path: str,
    num_requests: int,
    tokenizer: PreTrainedTokenizerBase,
    fixed_output_len: int | None,
) -> list[tuple[str, int, int, int]]:
    if fixed_output_len is not None and fixed_output_len < 4:
        raise ValueError("output_len too small")

    # Load the dataset.
    with open(dataset_path) as f:
        dataset = json.load(f)
    # Filter out the conversations with less than 2 turns.
    dataset = [data for data in dataset if len(data["conversations"]) >= 2]
    # Only keep the first two turns of each conversation.
    dataset = [
        (data["conversations"][0]["value"], data["conversations"][1]["value"])
        for data in dataset
    ]

    # Shuffle the dataset.
    random.shuffle(dataset)

    # Filter out sequences that are too long or too short
    filtered_dataset: list[tuple[str, int, int]] = []
    for i in range(len(dataset)):
        if len(filtered_dataset) == num_requests:
            break

        # Tokenize the prompts and completions.
        prompt = dataset[i][0]
        prompt_token_ids = tokenizer(prompt).input_ids
        completion = dataset[i][1]
        completion_token_ids = tokenizer(completion).input_ids
    # ... omitted for brevity ...
            # Prune too long sequences.
            continue

        priority = get_random_flag()

        filtered_dataset.append((prompt, prompt_len, output_len, priority))

    return filtered_dataset
```
**EN:** `sample_requests` implements backend request helpers. It mainly works with `dataset_path`, `num_requests`, `tokenizer`, `fixed_output_len` and relies on `ValueError`, `open`, `json.load`, `len`, `random.shuffle`, `range` plus iteration, branching and context management to move data through this part of the benchmark pipeline.
**CN:** `sample_requests` 负责实现后端请求辅助逻辑。 它主要处理 `dataset_path`, `num_requests`, `tokenizer`, `fixed_output_len`，并结合 `ValueError`, `open`, `json.load`, `len`, `random.shuffle`, `range` 以及 循环迭代、条件分支、上下文管理 来完成这一段基准测试流程。

### Function `run_vllm` (lines 73-112)
```python
def run_vllm(
    requests: list[tuple[str, int, int]],
    n: int,
    engine_args: EngineArgs,
    disable_detokenize: bool = False,
) -> float:
    from vllm import LLM, SamplingParams

    llm = LLM.from_engine_args(engine_args)

    assert all(
        llm.llm_engine.model_config.max_model_len >= (request[1] + request[2])
        for request in requests
    ), (
        "Please ensure that max_model_len is greater than the sum of"
        " input_len and output_len for all requests."
    )

    # Add the requests to the engine.
    prompts = []
    sampling_params = []
    priority = []
    for prompt, _, output_len, _priority in requests:
        prompts.append(prompt)
        priority.append(_priority)
        sampling_params.append(
            SamplingParams(
                n=n,
                temperature=1.0,
                top_p=1.0,
                ignore_eos=True,
                max_tokens=output_len,
                detokenize=not disable_detokenize,
            )
        )

    start = time.perf_counter()
    llm.generate(prompts, sampling_params, priority=priority, use_tqdm=True)
    end = time.perf_counter()
    return end - start
```
**EN:** `run_vllm` implements a helper used by `benchmark_prioritization.py`. It mainly works with `requests`, `n`, `engine_args`, `disable_detokenize` and relies on `LLM.from_engine_args`, `all`, `prompts.append`, `priority.append`, `sampling_params.append`, `SamplingParams` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `run_vllm` 负责实现 `benchmark_prioritization.py` 使用的辅助逻辑。 它主要处理 `requests`, `n`, `engine_args`, `disable_detokenize`，并结合 `LLM.from_engine_args`, `all`, `prompts.append`, `priority.append`, `sampling_params.append`, `SamplingParams` 以及 循环迭代 来完成这一段基准测试流程。

### Function `main` (lines 115-159)
```python
def main(args: argparse.Namespace):
    print(args)
    random.seed(args.seed)

    # Sample the requests.
    tokenizer = AutoTokenizer.from_pretrained(
        args.tokenizer, trust_remote_code=args.trust_remote_code
    )
    if args.dataset is None:
        # Synthesize a prompt with the given input length.
        prompt = "hi" * (args.input_len - 1)
        requests = [
            (prompt, args.input_len, args.output_len, get_random_flag())
            for _ in range(args.num_prompts)
        ]
    else:
        requests = sample_requests(
            args.dataset, args.num_prompts, tokenizer, args.output_len
        )

    if args.backend == "vllm":
        elapsed_time = run_vllm(
            requests, args.n, EngineArgs.from_cli_args(args), args.disable_detokenize
        )
    else:
        raise ValueError(f"Unknown backend: {args.backend}")
    total_num_tokens = sum(
        prompt_len + output_len for _, prompt_len, output_len, priority in requests
    )
    print(
        f"Throughput: {len(requests) / elapsed_time:.2f} requests/s, "
        f"{total_num_tokens / elapsed_time:.2f} tokens/s"
    )

    # Output JSON results if specified
    if args.output_json:
        results = {
            "elapsed_time": elapsed_time,
            "num_requests": len(requests),
            "total_num_tokens": total_num_tokens,
            "requests_per_second": len(requests) / elapsed_time,
            "tokens_per_second": total_num_tokens / elapsed_time,
        }
        with open(args.output_json, "w") as f:
            json.dump(results, f, indent=4)
```
**EN:** `main` serves as the main entry point for this script. It mainly works with `args` and relies on `print`, `random.seed`, `AutoTokenizer.from_pretrained`, `get_random_flag`, `range`, `sample_requests` plus branching and context management to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 `args`，并结合 `print`, `random.seed`, `AutoTokenizer.from_pretrained`, `get_random_flag`, `range`, `sample_requests` 以及 条件分支、上下文管理 来完成这一段基准测试流程。

### Function `create_argument_parser` (lines 162-206)
```python
def create_argument_parser():
    parser = FlexibleArgumentParser(description="Benchmark the throughput.")
    parser.add_argument(
        "--backend", type=str, choices=["vllm", "hf", "mii"], default="vllm"
    )
    parser.add_argument(
        "--dataset", type=str, default=None, help="Path to the dataset."
    )
    parser.add_argument(
        "--input-len",
        type=int,
        default=None,
        help="Input prompt length for each request",
    )
    parser.add_argument(
        "--output-len",
        type=int,
        default=None,
        help="Output length for each request. Overrides the "
        "output length from the dataset.",
    )
    parser.add_argument(
        "--n", type=int, default=1, help="Number of generated sequences per prompt."
    )
    parser.add_argument(
        "--num-prompts", type=int, default=200, help="Number of prompts to process."
    )
    parser.add_argument(
        "--output-json",
        type=str,
        default=None,
        help="Path to save the throughput results in JSON format.",
    )
    parser.add_argument(
        "--disable-detokenize",
        action="store_true",
        help=(
            "Do not detokenize responses (i.e. do not include "
            "detokenization time in the latency measurement)"
        ),
    )

    parser = EngineArgs.add_cli_args(parser)

    return parser
```
**EN:** `create_argument_parser` constructs shared runtime objects for the benchmark. It mainly works with no explicit parameters and relies on `FlexibleArgumentParser`, `parser.add_argument`, `EngineArgs.add_cli_args` plus value production to move data through this part of the benchmark pipeline.
**CN:** `create_argument_parser` 负责构建基准测试共享的运行时对象。 它主要处理 无显式参数，并结合 `FlexibleArgumentParser`, `parser.add_argument`, `EngineArgs.add_cli_args` 以及 结果返回 来完成这一段基准测试流程。

### Entry point (lines 209-220)
```python
if __name__ == "__main__":
    parser = create_argument_parser()
    args = parser.parse_args()
    if args.tokenizer is None:
        args.tokenizer = args.model
    if args.dataset is None:
        assert args.input_len is not None
        assert args.output_len is not None
    else:
        assert args.input_len is None

    main(args)
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `create_argument_parser`, `parser.parse_args`, `main` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `create_argument_parser`, `parser.parse_args`, `main` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Serving path benchmarking: the script interacts with HTTP services, servers, or request proxies.
- **CN:** 服务路径基准：脚本会与 HTTP 服务、服务器或请求代理交互。
- **EN:** Dataset handling: benchmark inputs are loaded, normalized, or synthesized before execution.
- **CN:** 数据集处理：基准输入会在执行前被加载、规范化或合成。
- **EN:** Performance metrics: the script emphasizes latency/throughput measurement rather than model quality.
- **CN:** 性能指标：该脚本更强调延迟/吞吐量测量，而不是模型质量。

## Dependencies / 依赖关系
- **EN:** Standard library: `argparse`, `json`, `random`, `time`.
- **CN:** 标准库依赖：`argparse`, `json`, `random`, `time`。
- **EN:** Third-party packages: `transformers`.
- **CN:** 第三方依赖：`transformers`。
- **EN:** Internal modules: `vllm.engine.arg_utils`, `vllm.utils.argparse_utils`.
- **CN:** 内部模块：`vllm.engine.arg_utils`, `vllm.utils.argparse_utils`。
