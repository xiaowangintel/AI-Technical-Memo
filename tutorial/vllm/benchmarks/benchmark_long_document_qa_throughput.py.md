# benchmark_long_document_qa_throughput.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/benchmark_long_document_qa_throughput.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, throughput measurement flows, centered around `test_long_document_qa`, `repeat_prompts`, `main`, `create_argument_parser`. / 实现与基准测试编排、吞吐量测量流程相关的逻辑，核心符号包括 `test_long_document_qa`, `repeat_prompts`, `main`, `create_argument_parser`。

## Line-by-Line Analysis / 逐行分析
### Module overview (lines 3-41)
```python
"""
Offline benchmark to test the long document QA throughput.

Example usage:
    # This workload samples 8 different prompts with a default input
    # length of 20000 tokens, then replicates each prompt 2 times 
    # in random order.
    python benchmark_long_document_qa_throughput.py \
        --model meta-llama/Llama-2-7b-chat-hf \
        --enable-prefix-caching \
        --num-documents 8 \
        --repeat-count 2 

Commandline arguments:
    --num-documents: The number of documents to sample prompts from.

    --document-length: The length of each document in tokens. 
                       (Optional, default: 20000)

    --output-len: The number of tokens to generate for each prompt.
                  (Optional, default: 10)

    --repeat-count: The number of times to repeat each prompt.
                    (Optional, default: 2)

    --repeat-mode: The mode to repeat prompts. The supported modes are:
        - 'random': shuffle the prompts randomly. (Default)
        - 'tile': the entire prompt list is repeated in sequence. (Potentially
                  lowest cache hit)
        - 'interleave': each prompt is repeated consecutively before 
                        moving to the next element. (Highest cache hit)
    
    --shuffle-seed: Random seed when the repeat mode is "random".
                    (Optional, default: 0)

In the meantime, it also supports all the vLLM engine args to initialize the 
LLM engine. You can refer to the `vllm.engine.arg_utils.EngineArgs` for more
details.
"""
```
**EN:** The module docstring introduces Offline benchmark to test the long document QA throughput. It gives readers the context needed before the implementation details start.
**CN:** 模块文档字符串先说明了 Offline benchmark to test the long document QA throughput 的背景与用途，在进入实现细节前先建立阅读上下文。

### Imports (lines 43-48)
```python
import random
import time

from vllm import LLM, SamplingParams
from vllm.engine.arg_utils import EngineArgs
from vllm.utils.argparse_utils import FlexibleArgumentParser
```
**EN:** This block gathers standard-library helpers such as `random`, `time`; project-local modules such as `vllm`, `vllm.engine.arg_utils`, `vllm.utils.argparse_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `random`, `time`；项目内部模块，如 `vllm`, `vllm.engine.arg_utils`, `vllm.utils.argparse_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Function `test_long_document_qa` (lines 51-64)
```python
def test_long_document_qa(llm=None, sampling_params=None, prompts=None):
    """
    Test long document QA with the given prompts and sampling parameters.
    Print the time spent in processing all the prompts.

    Args:
        llm: The language model used for generating responses.
        sampling_params: Sampling parameter used to generate the response.
        prompts: A list of prompt strings to be processed by the LLM.
    """
    start_time = time.time()
    llm.generate(prompts, sampling_params=sampling_params)
    end_time = time.time()
    print(f"Time to execute all requests: {end_time - start_time:.4f} secs")
```
**EN:** `test_long_document_qa` Test long document QA with the given prompts and sampling parameters. It mainly works with `llm`, `sampling_params`, `prompts` and relies on `time.time`, `llm.generate`, `print` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `test_long_document_qa` 的职责是：Test long document QA with the given prompts and sampling parameters。 它主要处理 `llm`, `sampling_params`, `prompts`，并结合 `time.time`, `llm.generate`, `print` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Function `repeat_prompts` (lines 67-103)
```python
def repeat_prompts(prompts, repeat_count, mode: str):
    """
    Repeat each prompt in the list for a specified number of times.
    The order of prompts in the output list depends on the mode.

    Args:
        prompts: A list of prompts to be repeated.
        repeat_count: The number of times each prompt is repeated.
        mode: The mode of repetition. Supported modes are:
            - 'random': Shuffle the prompts randomly after repetition.
            - 'tile': Repeat the entire prompt list in sequence.
              Example: [1, 2, 3] -> [1, 2, 3, 1, 2, 3].
            - 'interleave': Repeat each prompt consecutively before moving to
              the next. Example: [1, 2, 3] -> [1, 1, 2, 2, 3, 3].

    Returns:
        A list of repeated prompts in the specified order.

    Raises:
        ValueError: If an invalid mode is provided.
    """
    print("Repeat mode: ", mode)
    if mode == "random":
        repeated_prompts = prompts * repeat_count
        random.shuffle(repeated_prompts)
        return repeated_prompts
    elif mode == "tile":
        return prompts * repeat_count
    elif mode == "interleave":
        repeated_prompts = []
        for prompt in prompts:
            repeated_prompts.extend([prompt] * repeat_count)
        return repeated_prompts
    else:
        raise ValueError(
            f"Invalid mode: {mode}, only support 'random', 'tile', 'interleave'"
        )
```
**EN:** `repeat_prompts` Repeat each prompt in the list for a specified number of times. It mainly works with `prompts`, `repeat_count`, `mode` and relies on `print`, `random.shuffle`, `repeated_prompts.extend`, `ValueError` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `repeat_prompts` 的职责是：Repeat each prompt in the list for a specified number of times。 它主要处理 `prompts`, `repeat_count`, `mode`，并结合 `print`, `random.shuffle`, `repeated_prompts.extend`, `ValueError` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Function `main` (lines 106-141)
```python
def main(args):
    random.seed(args.shuffle_seed)

    # Prepare the prompts:
    # we append the document id at the beginning to avoid any of the document
    # being the prefix of other documents
    prompts = [
        str(i) + " ".join(["hi"] * args.document_length)
        for i in range(args.num_documents)
    ]

    prompts = repeat_prompts(prompts, args.repeat_count, mode=args.repeat_mode)

    warmup_prompts = [
        "This is warm up request " + str(i) + " ".join(["hi"] * args.document_length)
        for i in range(args.num_documents)
    ]

    # Create the LLM engine
    engine_args = EngineArgs.from_cli_args(args)
    llm = LLM.from_engine_args(engine_args)
    sampling_params = SamplingParams(temperature=0, max_tokens=args.output_len)

    print("------warm up------")
    test_long_document_qa(
        llm=llm,
        prompts=warmup_prompts,
        sampling_params=sampling_params,
    )

    print("------start generating------")
    test_long_document_qa(
        llm=llm,
        prompts=prompts,
        sampling_params=sampling_params,
    )
```
**EN:** `main` serves as the main entry point for this script. It mainly works with `args` and relies on `random.seed`, `str`, `join`, `range`, `repeat_prompts`, `EngineArgs.from_cli_args` plus straight-line setup logic to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 `args`，并结合 `random.seed`, `str`, `join`, `range`, `repeat_prompts`, `EngineArgs.from_cli_args` 以及 顺序执行的初始化逻辑 来完成这一段基准测试流程。

### Function `create_argument_parser` (lines 144-195)
```python
def create_argument_parser():
    parser = FlexibleArgumentParser(
        description="Benchmark the performance with or "
        "without automatic prefix caching."
    )

    parser.add_argument(
        "--document-length",
        type=int,
        # Roughly the number of tokens for a system paper,
        # excluding images
        default=20000,
        help="Range of input lengths for sampling prompts, "
        'specified as "min:max" (e.g., "128:256").',
    )

    parser.add_argument(
        "--num-documents",
        type=int,
        default=8,
        help="Range of input lengths for sampling prompts, "
        'specified as "min:max" (e.g., "128:256").',
    )

    parser.add_argument("--output-len", type=int, default=10)

    parser.add_argument(
        "--repeat-count",
        type=int,
        default=2,
        help="Number of times to repeat each prompt",
    )

    parser.add_argument(
    # ... omitted for brevity ...
        type=int,
        default=0,
        help='Random seed when the repeat mode is "random"',
    )

    parser = EngineArgs.add_cli_args(parser)

    return parser
```
**EN:** `create_argument_parser` constructs shared runtime objects for the benchmark. It mainly works with no explicit parameters and relies on `FlexibleArgumentParser`, `parser.add_argument`, `EngineArgs.add_cli_args` plus value production to move data through this part of the benchmark pipeline.
**CN:** `create_argument_parser` 负责构建基准测试共享的运行时对象。 它主要处理 无显式参数，并结合 `FlexibleArgumentParser`, `parser.add_argument`, `EngineArgs.add_cli_args` 以及 结果返回 来完成这一段基准测试流程。

### Entry point (lines 198-201)
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
- **EN:** Standard library: `random`, `time`.
- **CN:** 标准库依赖：`random`, `time`。
- **EN:** Third-party packages: none.
- **CN:** 第三方依赖：无。
- **EN:** Internal modules: `vllm`, `vllm.engine.arg_utils`, `vllm.utils.argparse_utils`.
- **CN:** 内部模块：`vllm`, `vllm.engine.arg_utils`, `vllm.utils.argparse_utils`。
