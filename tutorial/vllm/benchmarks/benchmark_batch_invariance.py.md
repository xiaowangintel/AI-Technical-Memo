# benchmark_batch_invariance.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmarks/benchmark_batch_invariance.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements benchmark orchestration, batch behavior benchmarks, centered around `_random_prompt`, `run_benchmark_with_batch_invariant`, `main`. / 实现与基准测试编排、批处理行为基准相关的逻辑，核心符号包括 `_random_prompt`, `run_benchmark_with_batch_invariant`, `main`。

## Line-by-Line Analysis / 逐行分析
### Module overview (lines 4-37)
```python
"""
Benchmark to measure the performance overhead of VLLM_BATCH_INVARIANT mode.

This benchmark runs the same workload twice:
1. With VLLM_BATCH_INVARIANT=0 (baseline)
2. With VLLM_BATCH_INVARIANT=1 (batch invariant mode)

And reports the timing and throughput metrics for comparison.

Environment variables:
    VLLM_BENCH_MODEL: Model to benchmark (default: "Qwen/Qwen3-1.7B")
    VLLM_BENCH_TP_SIZE: Tensor parallel size (default: 1, use 8 for deepseek)
    VLLM_BENCH_BATCH_SIZE: Max batch size (default: 128)
    VLLM_BENCH_NUM_TRIALS: Number of trials to run (default: 5)
    VLLM_BENCH_MIN_PROMPT: Min prompt length in words (default: 1024)
    VLLM_BENCH_MAX_PROMPT: Max prompt length in words (default: 2048)
    VLLM_BENCH_MAX_TOKENS: Max tokens to generate (default: 128)
    VLLM_BENCH_TEMPERATURE: Temperature for sampling (default: 0.0)
    VLLM_BENCH_GPU_MEMORY_UTILIZATION: GPU memory utilization (default: 0.4)
    VLLM_BENCH_MAX_MODEL_LEN: Max model length (default: 5120)
    VLLM_BENCH_BACKEND: Attention backend (default: FLASH_ATTN)

Example usage:
    # Benchmark qwen3 (default)
    python benchmarks/benchmark_batch_invariance.py

    # Benchmark deepseek with 8 GPUs
    VLLM_BENCH_MODEL="deepseek-ai/DeepSeek-V3" VLLM_BENCH_TP_SIZE=8 \\
        python benchmarks/benchmark_batch_invariance.py

    # Quick test with fewer trials
    VLLM_BENCH_NUM_TRIALS=2 VLLM_BENCH_BATCH_SIZE=32 \\
        python benchmarks/benchmark_batch_invariance.py
"""
```
**EN:** The module docstring introduces Benchmark to measure the performance overhead of VLLM_BATCH_INVARIANT mode. It gives readers the context needed before the implementation details start.
**CN:** 模块文档字符串先说明了 Benchmark to measure the performance overhead of VLLM_BATCH_INVARIANT mode 的背景与用途，在进入实现细节前先建立阅读上下文。

### Imports (lines 39-45)
```python
import contextlib
import os
import random
import time

from vllm import LLM, SamplingParams
from vllm.platforms import current_platform
```
**EN:** This block gathers standard-library helpers such as `contextlib`, `os`, `random`, `time`; project-local modules such as `vllm`, `vllm.platforms`. It defines the dependency surface used by the later benchmark logic.
**CN:** 该代码块集中导入了标准库辅助模块，如 `contextlib`, `os`, `random`, `time`；项目内部模块，如 `vllm`, `vllm.platforms`。这些依赖构成了后续基准测试逻辑的基础。

### Function `_random_prompt` (lines 48-81)
```python
def _random_prompt(min_words: int = 1024, max_words: int = 1024 * 2) -> str:
    """Generate a random prompt for benchmarking."""
    prompt_templates = [
        "Question: What is the capital of France?\nAnswer: The capital of France is",
        "Q: How does photosynthesis work?\nA: Photosynthesis is the process by which",
        "User: Can you explain quantum mechanics?\nAssistant: Quantum mechanics is",
        "Once upon a time in a distant galaxy, there lived",
        "The old man walked slowly down the street, remembering",
        "In the year 2157, humanity finally discovered",
        "To implement a binary search tree in Python, first we need to",
        "The algorithm works by iterating through the array and",
        "Here's how to optimize database queries using indexing:",
        "The Renaissance was a period in European history that",
        "Climate change is caused by several factors including",
        "The human brain contains approximately 86 billion neurons which",
        "I've been thinking about getting a new laptop because",
        "Yesterday I went to the store and bought",
        "My favorite thing about summer is definitely",
    ]

    base_prompt = random.choice(prompt_templates)

    if max_words < min_words:
        max_words = min_words
    target_words = random.randint(min_words, max_words)

    if target_words > 50:
        padding_text = (
            " This is an interesting topic that deserves more explanation. "
            * (target_words // 50)
        )
        base_prompt = base_prompt + padding_text

    return base_prompt
```
**EN:** `_random_prompt` Generate a random prompt for benchmarking. It mainly works with `min_words`, `max_words` and relies on `random.choice`, `random.randint` plus branching to move data through this part of the benchmark pipeline.
**CN:** `_random_prompt` 的职责是：Generate a random prompt for benchmarking。 它主要处理 `min_words`, `max_words`，并结合 `random.choice`, `random.randint` 以及 条件分支 来完成这一段基准测试流程。

### Function `run_benchmark_with_batch_invariant` (lines 84-228)
```python
def run_benchmark_with_batch_invariant(
    model: str,
    tp_size: int,
    max_batch_size: int,
    num_trials: int,
    min_prompt: int,
    max_prompt: int,
    max_tokens: int,
    temperature: float,
    gpu_mem_util: float,
    max_model_len: int,
    backend: str,
    batch_invariant: bool,
    seed: int = 12345,
) -> dict:
    """
    Run the benchmark with the specified configuration.

    Returns a dict with timing and throughput metrics.
    """
    random.seed(seed)

    # Set environment variables
    if batch_invariant:
        os.environ["VLLM_BATCH_INVARIANT"] = "1"
    else:
        os.environ["VLLM_BATCH_INVARIANT"] = "0"

    print(f"\n{'=' * 80}")
    print(f"BENCHMARK: VLLM_BATCH_INVARIANT={int(batch_invariant)}")
    print(f"  Model: {model}")
    print(f"  TP Size: {tp_size}")
    print(f"  Backend: {backend}")
    print(f"  Max Batch Size: {max_batch_size}")
    # ... omitted for brevity ...
            "trial_times": trial_times,
        }

    finally:
        # Cleanup
        if llm is not None:
            with contextlib.suppress(Exception):
                llm.shutdown()
```
**EN:** `run_benchmark_with_batch_invariant` Run the benchmark with the specified configuration. It mainly works with `model`, `tp_size`, `max_batch_size`, `num_trials`, `min_prompt`, ... and relies on `random.seed`, `print`, `int`, `SamplingParams`, `time.perf_counter`, `LLM` plus iteration, branching, error handling and context management to move data through this part of the benchmark pipeline.
**CN:** `run_benchmark_with_batch_invariant` 的职责是：Run the benchmark with the specified configuration。 它主要处理 `model`, `tp_size`, `max_batch_size`, `num_trials`, `min_prompt`, ...，并结合 `random.seed`, `print`, `int`, `SamplingParams`, `time.perf_counter`, `LLM` 以及 循环迭代、条件分支、异常处理、上下文管理 来完成这一段基准测试流程。

### Function `main` (lines 231-376)
```python
def main():
    # Check platform support
    if not (current_platform.is_cuda() and current_platform.has_device_capability(90)):
        print("ERROR: Requires CUDA and >= Hopper (SM90)")
        print(f"Current platform: {current_platform.device_type}")
        if current_platform.is_cuda():
            print(f"Device capability: {current_platform.get_device_capability()}")
        return 1

    # Read configuration from environment
    model = os.getenv("VLLM_BENCH_MODEL", "Qwen/Qwen3-1.7B")
    tp_size = int(os.getenv("VLLM_BENCH_TP_SIZE", "1"))
    max_batch_size = int(os.getenv("VLLM_BENCH_BATCH_SIZE", "128"))
    num_trials = int(os.getenv("VLLM_BENCH_NUM_TRIALS", "5"))
    min_prompt = int(os.getenv("VLLM_BENCH_MIN_PROMPT", "1024"))
    max_prompt = int(os.getenv("VLLM_BENCH_MAX_PROMPT", "2048"))
    max_tokens = int(os.getenv("VLLM_BENCH_MAX_TOKENS", "128"))
    temperature = float(os.getenv("VLLM_BENCH_TEMPERATURE", "0.0"))
    gpu_mem_util = float(os.getenv("VLLM_BENCH_GPU_MEMORY_UTILIZATION", "0.4"))
    max_model_len = int(os.getenv("VLLM_BENCH_MAX_MODEL_LEN", "5120"))
    backend = os.getenv("VLLM_BENCH_BACKEND", "FLASH_ATTN")

    print("\n" + "=" * 80)
    print("VLLM BATCH INVARIANCE BENCHMARK")
    print("=" * 80)
    print("\nConfiguration:")
    print(f"  Model: {model}")
    print(f"  Tensor Parallel Size: {tp_size}")
    print(f"  Attention Backend: {backend}")
    print(f"  Max Batch Size: {max_batch_size}")
    print(f"  Number of Trials: {num_trials}")
    print(f"  Prompt Length Range: {min_prompt}-{max_prompt} words")
    print(f"  Max Tokens to Generate: {max_tokens}")
    print(f"  Temperature: {temperature}")
    # ... omitted for brevity ...
        print(
            f"Throughput increased by {throughput_change_pct:.1f}% "
            "with batch invariance (unexpected!)"
        )

    print("=" * 80 + "\n")

    return 0
```
**EN:** `main` serves as the main entry point for this script. It mainly works with no explicit parameters and relies on `current_platform.is_cuda`, `current_platform.has_device_capability`, `print`, `current_platform.get_device_capability`, `os.getenv`, `int` plus branching to move data through this part of the benchmark pipeline.
**CN:** `main` 充当该脚本的主入口。 它主要处理 无显式参数，并结合 `current_platform.is_cuda`, `current_platform.has_device_capability`, `print`, `current_platform.get_device_capability`, `os.getenv`, `int` 以及 条件分支 来完成这一段基准测试流程。

### Entry point (lines 379-380)
```python
if __name__ == "__main__":
    exit(main())
```
**EN:** The `__main__` guard turns the file into an executable script. In practice it wires together `exit`, `main` so the module can be run directly from the command line.
**CN:** `__main__` 守卫让该文件可以直接作为脚本执行。实际运行时，它把 `exit`, `main` 等调用串起来，使模块能够从命令行直接启动。

## Key Concepts / 关键概念
- **EN:** CLI orchestration: the file can be launched as a command-line benchmark entry point.
- **CN:** 命令行编排：该文件可以作为命令行基准测试入口运行。
- **EN:** Tensor/kernel work: the module prepares tensors, invokes kernels, or measures accelerator behavior.
- **CN:** 张量/内核工作：该模块会准备张量、调用内核或测量加速器行为。
- **EN:** Dataset handling: benchmark inputs are loaded, normalized, or synthesized before execution.
- **CN:** 数据集处理：基准输入会在执行前被加载、规范化或合成。
- **EN:** Attention specialization: the code studies attention backends, cache layouts, or MLA variants.
- **CN:** 注意力专项：代码研究注意力后端、缓存布局或 MLA 变体。
- **EN:** Quantization paths: the file benchmarks low-precision math, scales, or quantized kernels.
- **CN:** 量化路径：该文件测试低精度计算、缩放参数或量化内核。

## Dependencies / 依赖关系
- **EN:** Standard library: `contextlib`, `os`, `random`, `time`.
- **CN:** 标准库依赖：`contextlib`, `os`, `random`, `time`。
- **EN:** Third-party packages: none.
- **CN:** 第三方依赖：无。
- **EN:** Internal modules: `vllm`, `vllm.platforms`.
- **CN:** 内部模块：`vllm`, `vllm.platforms`。
