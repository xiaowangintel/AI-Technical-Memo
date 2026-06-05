# benchmark_prefix_caching.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/benchmark_prefix_caching.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, prefix-caching experiments, centered around `PROMPT`, `test_prefix`, `Request`, `sample_tokens`. / 实现与基准测试编排、前缀缓存实验相关的逻辑，核心符号包括 `PROMPT`, `test_prefix`, `Request`, `sample_tokens`。

## Line-by-Line Analysis / 逐行分析
### Module overview (lines 3-29)
```python
"""
Benchmark the efficiency of prefix caching.

This script allows you to benchmark the performance of
a model with and without prefix caching using either fixed prompts
or prompts sampled from the ShareGPT dataset.

Fixed example usage:
    python benchmark_prefix_caching.py \
        --model meta-llama/Llama-2-7b-chat-hf \
        --enable-prefix-caching \
        --num-prompts 1 \
        --repeat-count 100 \
        --input-length-range 128:256

ShareGPT example usage:
    # This command samples 20 prompts with input lengths
    # between 128 and 256 tokens from the ShareGPT dataset,
    # then replicates each prompt 5 times.
    python benchmark_prefix_caching.py \
        --model meta-llama/Llama-2-7b-chat-hf \
        --dataset-path /path/to/ShareGPT_V3_unfiltered_cleaned_split.json \
        --enable-prefix-caching \
        --num-prompts 20 \
        --repeat-count 5 \
        --input-length-range 128:256
"""
```
**EN:** The module docstring introduces Benchmark the efficiency of prefix caching. It gives readers the context needed before the implementation details start.
**CN:** 模块文档字符串先说明了 Benchmark the efficiency of prefix caching 的背景与用途，在进入实现细节前先建立阅读上下文。

### Imports (lines 31-40)
```python
import dataclasses
import json
import random
import time

from transformers import PreTrainedTokenizerBase

from vllm import LLM, SamplingParams
from vllm.engine.arg_utils import EngineArgs
from vllm.utils.argparse_utils import FlexibleArgumentParser
```
**EN:** This block gathers standard-library helpers such as `dataclasses`, `json`, `random`, `time`; third-party packages such as `transformers`; project-local modules such as `vllm`, `vllm.engine.arg_utils`, `vllm.utils.argparse_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `dataclasses`, `json`, `random`, `time`；第三方依赖，如 `transformers`；项目内部模块，如 `vllm`, `vllm.engine.arg_utils`, `vllm.utils.argparse_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 42-47)
```python
try:
    from vllm.tokenizers import get_tokenizer
except ImportError:
    from backend_request_func import get_tokenizer

PROMPT = "You are a helpful assistant in recognizes the content of tables in markdown format. Here is a table as fellows. You need to answer my question about the table.\n# Table\n|Opening|Opening|Sl. No.|Film|Cast|Director|Music Director|Notes|\n|----|----|----|----|----|----|----|----|\n|J A N|9|1|Agni Pushpam|Jayabharathi, Kamalahasan|Jeassy|M. K. Arjunan||\n|J A N|16|2|Priyamvada|Mohan Sharma, Lakshmi, KPAC Lalitha|K. S. Sethumadhavan|V. Dakshinamoorthy||\n|J A N|23|3|Yakshagaanam|Madhu, Sheela|Sheela|M. S. Viswanathan||\n|J A N|30|4|Paalkkadal|Sheela, Sharada|T. K. Prasad|A. T. Ummer||\n|F E B|5|5|Amma|Madhu, Srividya|M. Krishnan Nair|M. K. Arjunan||\n|F E B|13|6|Appooppan|Thikkurissi Sukumaran Nair, Kamal Haasan|P. Bhaskaran|M. S. Baburaj||\n|F E B|20|7|Srishti|Chowalloor Krishnankutty, Ravi Alummoodu|K. T. Muhammad|M. S. Baburaj||\n|F E B|20|8|Vanadevatha|Prem Nazir, Madhubala|Yusufali Kechery|G. Devarajan||\n|F E B|27|9|Samasya|Madhu, Kamalahaasan|K. Thankappan|Shyam||\n|F E B|27|10|Yudhabhoomi|K. P. Ummer, Vidhubala|Crossbelt Mani|R. K. Shekhar||\n|M A R|5|11|Seemantha Puthran|Prem Nazir, Jayabharathi|A. B. Raj|M. K. Arjunan||\n|M A R|12|12|Swapnadanam|Rani Chandra, Dr. Mohandas|K. G. George|Bhaskar Chandavarkar||\n|M A R|19|13|Thulavarsham|Prem Nazir, sreedevi, Sudheer|N. Sankaran Nair|V. Dakshinamoorthy||\n|M A R|20|14|Aruthu|Kaviyoor Ponnamma, Kamalahasan|Ravi|G. Devarajan||\n|M A R|26|15|Swimming Pool|Kamal Haasan, M. G. Soman|J. Sasikumar|M. K. Arjunan||\n\n# Question\nWhat' s the content in the (1,1) cells\n"  # noqa: E501
```
**EN:** This top-level block prepares shared state such as `PROMPT`. It uses direct expressions to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `PROMPT`。它借助 direct expressions 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `test_prefix` (lines 50-56)
```python
def test_prefix(llm=None, sampling_params=None, prompts=None):
    start_time = time.time()

    llm.generate(prompts, sampling_params=sampling_params)

    end_time = time.time()
    print(f"cost time {end_time - start_time}")
```
**EN:** `test_prefix` coordinates or measures prefix-caching experiments. It mainly works with `llm`, `sampling_params`, `prompts` and relies on `time.time`, `llm.generate`, `print` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `test_prefix` 负责协调或测量前缀缓存实验。 它主要处理 `llm`, `sampling_params`, `prompts`，并结合 `time.time`, `llm.generate`, `print` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Class `Request` (lines 60-63)
```python
class Request:
    prompt: str
    prompt_len: int
    output_len: int
```
**EN:** Class `Request` packages shared state and related operations for this benchmark module. It extends `object` and exposes methods such as its methods.
**CN:** 类 `Request` 用于封装该基准模块的共享状态与相关操作。它继承自 `object`，并提供 its methods 等方法。

### Function `sample_tokens` (lines 66-74)
```python
def sample_tokens(tokenizer: PreTrainedTokenizerBase, length: int) -> list[int]:
    vocab = tokenizer.get_vocab()
    all_special_ids = set(tokenizer.all_special_ids)

    # Remove the special tokens.
    return random.choices(
        [v for v in vocab.values() if v not in all_special_ids],
        k=length,
    )
```
**EN:** `sample_tokens` implements a helper used by `benchmark_prefix_caching.py`. It mainly works with `tokenizer`, `length` and relies on `tokenizer.get_vocab`, `set`, `random.choices`, `vocab.values` plus value production to move data through this part of the benchmark pipeline.
**CN:** `sample_tokens` 负责实现 `benchmark_prefix_caching.py` 使用的辅助逻辑。 它主要处理 `tokenizer`, `length`，并结合 `tokenizer.get_vocab`, `set`, `random.choices`, `vocab.values` 以及 结果返回 来完成这一段基准测试流程。

### Function `sample_requests_from_dataset` (lines 77-123)
```python
def sample_requests_from_dataset(
    dataset_path: str,
    num_requests: int,
    tokenizer: PreTrainedTokenizerBase,
    input_length_range: tuple[int, int],
    fixed_output_len: int | None,
) -> list[Request]:
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

    min_len, max_len = input_length_range
    assert min_len >= 0 and max_len >= min_len, "input_length_range too small"

    # Filter out sequences that are too long or too short
    filtered_requests: list[Request] = []

    for i in range(len(dataset)):
        if len(filtered_requests) == num_requests:
            break

        # Tokenize the prompts and completions.
        prompt_token_ids = tokenizer(dataset[i][0]).input_ids
        prompt = tokenizer.decode(prompt_token_ids)
        completion = dataset[i][1]
        completion_token_ids = tokenizer(completion).input_ids
        prompt_len = len(prompt_token_ids)
        output_len = (
            len(completion_token_ids) if fixed_output_len is None else fixed_output_len
        )
        if min_len <= prompt_len <= max_len:
            filtered_requests.append(Request(prompt, prompt_len, output_len))

    return filtered_requests
```
**EN:** `sample_requests_from_dataset` implements dataset preparation logic. It mainly works with `dataset_path`, `num_requests`, `tokenizer`, `input_length_range`, `fixed_output_len` and relies on `ValueError`, `open`, `json.load`, `len`, `random.shuffle`, `range` plus iteration, branching and context management to move data through this part of the benchmark pipeline.
**CN:** `sample_requests_from_dataset` 负责实现数据集准备逻辑。 它主要处理 `dataset_path`, `num_requests`, `tokenizer`, `input_length_range`, `fixed_output_len`，并结合 `ValueError`, `open`, `json.load`, `len`, `random.shuffle`, `range` 以及 循环迭代、条件分支、上下文管理 来完成这一段基准测试流程。

### Function `sample_requests_from_random` (lines 126-148)
```python
def sample_requests_from_random(
    num_requests: int,
    tokenizer: PreTrainedTokenizerBase,
    input_length_range: tuple[int, int],
    fixed_output_len: int | None,
    prefix_len: int,
) -> list[Request]:
    requests = []
    prefix_token_ids = sample_tokens(tokenizer, prefix_len)
    min_len, max_len = input_length_range

    for i in range(num_requests):
        unique_part_token_ids = sample_tokens(
            tokenizer, random.randint(min_len - prefix_len, max_len - prefix_len)
        )
        prompt_token_ids = prefix_token_ids + unique_part_token_ids
        prompt = tokenizer.decode(prompt_token_ids)
        prompt_len = len(prompt_token_ids)
        assert min_len <= prompt_len <= max_len, (
            f"prompt_len {prompt_len} out of range {min_len}:{max_len}"
        )
        requests.append(Request(prompt, prompt_len, fixed_output_len))
    return requests
```
**EN:** `sample_requests_from_random` implements backend request helpers. It mainly works with `num_requests`, `tokenizer`, `input_length_range`, `fixed_output_len`, `prefix_len` and relies on `sample_tokens`, `range`, `random.randint`, `tokenizer.decode`, `len`, `requests.append` plus iteration to move data through this part of the benchmark pipeline.
**CN:** `sample_requests_from_random` 负责实现后端请求辅助逻辑。 它主要处理 `num_requests`, `tokenizer`, `input_length_range`, `fixed_output_len`, `prefix_len`，并结合 `sample_tokens`, `range`, `random.randint`, `tokenizer.decode`, `len`, `requests.append` 以及 循环迭代 来完成这一段基准测试流程。

### Function `repeat_and_sort_requests` (lines 151-159)
```python
def repeat_and_sort_requests(
    requests: list[Request], repeat_count: int, sort: bool = False
) -> list[str]:
    repeated_requests = requests * repeat_count
    if sort:
        repeated_requests.sort(key=lambda x: x[1])
    else:
        random.shuffle(repeated_requests)
    return [req.prompt for req in repeated_requests]
```
**EN:** `repeat_and_sort_requests` implements backend request helpers. It mainly works with `requests`, `repeat_count`, `sort` and relies on `repeated_requests.sort`, `random.shuffle` plus branching to move data through this part of the benchmark pipeline.
**CN:** `repeat_and_sort_requests` 负责实现后端请求辅助逻辑。 它主要处理 `requests`, `repeat_count`, `sort`，并结合 `repeated_requests.sort`, `random.shuffle` 以及 条件分支 来完成这一段基准测试流程。

### Function `main` (lines 162-217)
```python
def main(args):
    tokenizer = get_tokenizer(args.model, trust_remote_code=True)
    input_length_range = tuple(map(int, args.input_length_range.split(":")))
    random.seed(args.seed)
    if args.dataset_path is not None:
        if args.prefix_len > 0:
            raise ValueError(
                "prefix-len is not supported when dataset-path is provided."
            )
        print(f"Start to sample {args.num_prompts} prompts from {args.dataset_path}")
        filtered_requests = sample_requests_from_dataset(
            dataset_path=args.dataset_path,
            num_requests=args.num_prompts,
            tokenizer=tokenizer,
            input_length_range=input_length_range,
            fixed_output_len=args.output_len,
        )
    else:
        print(f"Start to sample {args.num_prompts} prompts from random")
        filtered_requests = sample_requests_from_random(
            num_requests=args.num_prompts,
            tokenizer=tokenizer,
            input_length_range=input_length_range,
            fixed_output_len=args.output_len,
            prefix_len=args.prefix_len,
        )

    # Print some helpful stats of the requests.
    print(f"Sampled {len(filtered_requests)} requests.")
    prompt_lens = [req.prompt_len for req in filtered_requests]
    print(f"Average input length: {sum(prompt_lens) / len(prompt_lens)}")
    print(f"P50 input length: {sorted(prompt_lens)[len(prompt_lens) // 2]}")
    print(f"Min Prompt Length: {min(prompt_lens)}")
    print(f"Max Prompt Length: {max(prompt_lens)}")
    # ... omitted for brevity ...
    )

    print("------start generating------")
    test_prefix(
        llm=llm,
        prompts=prompts,
        sampling_params=sampling_params,
    )
```
**EN:** `main` serves as the main entry point for this script. It mainly works with `args` and relies on `get_tokenizer`, `tuple`, `map`, `args.input_length_range.split`, `random.seed`, `ValueError` plus branching to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 `args`，并结合 `get_tokenizer`, `tuple`, `map`, `args.input_length_range.split`, `random.seed`, `ValueError` 以及 条件分支 来完成这一段基准测试流程。

### Function `create_argument_parser` (lines 220-271)
```python
def create_argument_parser():
    parser = FlexibleArgumentParser(
        description="Benchmark the performance with or without "
        "automatic prefix caching."
    )
    parser.add_argument(
        "--dataset-path", type=str, default=None, help="Path to the dataset."
    )
    parser.add_argument("--output-len", type=int, default=10)
    parser.add_argument(
        "--num-prompts",
        type=int,
        required=True,
        help="Number of the prompts sampled from dataset",
    )
    parser.add_argument(
        "--repeat-count",
        type=int,
        default=1,
        help="Number of times to repeat each prompt",
    )
    parser.add_argument(
        "--sort", action="store_true", help="Sort prompts by input length"
    )
    parser.add_argument(
        "--input-length-range",
        type=str,
        required=True,
        help="Range of input lengths for sampling prompts,"
        'specified as "min:max" (e.g., "128:256").',
    )
    parser.add_argument(
        "--prefix-len",
        type=int,
    # ... omitted for brevity ...
            "Do not detokenize responses (i.e. do not include "
            "detokenization time in the latency measurement)"
        ),
    )

    parser = EngineArgs.add_cli_args(parser)

    return parser
```
**EN:** `create_argument_parser` constructs shared runtime objects for the benchmark. It mainly works with no explicit parameters and relies on `FlexibleArgumentParser`, `parser.add_argument`, `EngineArgs.add_cli_args` plus value production to move data through this part of the benchmark pipeline.
**CN:** `create_argument_parser` 负责构建基准测试共享的运行时对象。 它主要处理 无显式参数，并结合 `FlexibleArgumentParser`, `parser.add_argument`, `EngineArgs.add_cli_args` 以及 结果返回 来完成这一段基准测试流程。

### Entry point (lines 274-277)
```python
if __name__ == "__main__":
    parser = create_argument_parser()
    args = parser.parse_args()
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
- **EN:** Standard library: `dataclasses`, `json`, `random`, `time`.
- **CN:** 标准库依赖：`dataclasses`, `json`, `random`, `time`。
- **EN:** Third-party packages: `transformers`.
- **CN:** 第三方依赖：`transformers`。
- **EN:** Internal modules: `vllm`, `vllm.engine.arg_utils`, `vllm.utils.argparse_utils`.
- **CN:** 内部模块：`vllm`, `vllm.engine.arg_utils`, `vllm.utils.argparse_utils`。
