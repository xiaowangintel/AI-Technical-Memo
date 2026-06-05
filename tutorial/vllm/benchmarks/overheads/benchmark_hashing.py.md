# benchmark_hashing.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/overheads/benchmark_hashing.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, hashing and cache-key experiments, centered around `LONG_PROMPT`, `LONG_PROMPT`, `main`. / 实现与基准测试编排、哈希与缓存键实验相关的逻辑，核心符号包括 `LONG_PROMPT`, `LONG_PROMPT`, `main`。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-8)
```python
import cProfile
import pstats

from vllm import LLM, SamplingParams
from vllm.utils.argparse_utils import FlexibleArgumentParser
```
**EN:** This block gathers standard-library helpers such as `cProfile`, `pstats`; project-local modules such as `vllm`, `vllm.utils.argparse_utils`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `cProfile`, `pstats`；项目内部模块，如 `vllm`, `vllm.utils.argparse_utils`。这些依赖构成了后续基准测试逻辑的基础。

### Top-level setup (lines 11-12)
```python
LONG_PROMPT = ["You are an expert in large language models, aren't you?"] * 1000
LONG_PROMPT = " ".join(LONG_PROMPT)
```
**EN:** This top-level block prepares shared state such as `LONG_PROMPT`. It uses `join` to establish defaults, constants, or reusable objects before the benchmark entry points run.
**CN:** 该顶层代码块负责准备共享状态，例如 `LONG_PROMPT`。它借助 `join` 建立默认值、常量或可复用对象，为后续基准入口做准备。

### Function `main` (lines 15-49)
```python
def main(args):
    llm = LLM(
        model=args.model,
        enforce_eager=True,
        enable_prefix_caching=True,
        tensor_parallel_size=args.tensor_parallel_size,
    )

    sampling_params = SamplingParams(temperature=0, max_tokens=args.output_len)
    profiler = cProfile.Profile()

    print("------warm up------")
    for i in range(3):
        output = llm.generate(LONG_PROMPT, sampling_params)
        print(output[0].outputs[0].text)

    print("------start generating------")
    for i in range(3):
        profiler.runctx(
            "llm.generate(LONG_PROMPT, sampling_params)", globals(), locals()
        )

    # analyze the runtime of hashing function
    stats = pstats.Stats(profiler)
    stats.sort_stats("cumulative")
    total_time = 0
    total_calls = 0
    for func in stats.stats:
        if "hash_of_block" in func[2]:
            total_time = stats.stats[func][3]
            total_calls = stats.stats[func][0]
    percentage = (total_time / stats.total_tt) * 100
    print(
        f"Hashing took {total_time:.2f} seconds,{percentage:.2f}% of the total runtime."
    )
```
**EN:** `main` serves as the main entry point for this script. It mainly works with `args` and relies on `LLM`, `SamplingParams`, `cProfile.Profile`, `print`, `range`, `llm.generate` plus iteration and branching to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 `args`，并结合 `LLM`, `SamplingParams`, `cProfile.Profile`, `print`, `range`, `llm.generate` 以及 循环迭代、条件分支 来完成这一段基准测试流程。

### Entry point (lines 52-64)
```python
if __name__ == "__main__":
    parser = FlexibleArgumentParser(
        description="Benchmark the performance of hashing function in"
        "automatic prefix caching."
    )
    parser.add_argument("--model", type=str, default="lmsys/longchat-7b-16k")
    parser.add_argument("--tensor-parallel-size", "-tp", type=int, default=1)
    parser.add_argument("--output-len", type=int, default=10)
    parser.add_argument(
        "--enable-prefix-caching", action="store_true", help="enable prefix caching"
    )
    args = parser.parse_args()
    main(args)
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `FlexibleArgumentParser`, `parser.add_argument`, `parser.parse_args`, `main` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `FlexibleArgumentParser`, `parser.add_argument`, `parser.parse_args`, `main` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Dataset handling: benchmark inputs are loaded, normalized, or synthesized before execution.
- **CN:** 数据集处理：基准输入会在执行前被加载、规范化或合成。

## Dependencies / 依赖关系
- **EN:** Standard library: `cProfile`, `pstats`.
- **CN:** 标准库依赖：`cProfile`, `pstats`。
- **EN:** Third-party packages: none.
- **CN:** 第三方依赖：无。
- **EN:** Internal modules: `vllm`, `vllm.utils.argparse_utils`.
- **CN:** 内部模块：`vllm`, `vllm.utils.argparse_utils`。
