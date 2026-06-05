# multi_process_fuzzer.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/experimental/torchfuzz/multi_process_fuzzer.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements experimental fuzzing helpers used to stress PyTorch operators or APIs.
- **Purpose (CN)**: 实现实验性模糊测试辅助逻辑，用于压测 PyTorch 算子或 API。
## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```python
#!/usr/bin/env python3
"""
Multi-process fuzzer library that uses worker processes to execute fuzzer.py with different seeds.
"""

import multiprocessing as mp
import re
import subprocess
import sys
import time
from collections import defaultdict
from dataclasses import dataclass
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as multiprocessing, re, subprocess, and 4 more. This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. Subprocess orchestration bridges this script to external build tools, linters, or system commands.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 multiprocessing、re、subprocess 等共 7 项。 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。

### Lines 14-25
```python

try:
    from tqdm import tqdm

    HAS_TQDM = True
except ImportError:
    HAS_TQDM = False

    # Create a mock tqdm class for type safety
    class MockTqdm:
        @staticmethod
        def write(msg, file=None):
```
- **EN**: This block assembles the Python-side dependencies, importing external packages such as tqdm. It introduces classes such as MockTqdm, which package state and behavior for this tooling task. This chunk defines `write`, which generates derived source text, templates, or metadata outputs. Configuration constants such as HAS_TQDM, HAS_TQDM centralize defaults so later functions share the same policy knobs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段组织 Python 侧依赖，引入了外部依赖包，如 tqdm。 它引入了 MockTqdm 等类，用来封装该工具任务所需的状态与行为。 这一段定义了 `write`，其作用是生成派生源码文本、模板或元数据输出。 HAS_TQDM、HAS_TQDM 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 26-41
```python
            print(msg, file=file, flush=True)

    tqdm = MockTqdm()


def persist_print(msg):
    """Print messages that persist with tqdm progress bars."""
    try:
        if HAS_TQDM and hasattr(tqdm, "write"):
            # Keep prints on the same stream as the bar
            tqdm.write(msg, file=sys.stderr)
        else:
            print(msg, file=sys.stderr, flush=True)
    except BrokenPipeError:
        import os
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as os. This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `persist_print`, which implements a focused step inside the torch fuzzing pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 os。 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `persist_print`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 42-55
```python
        os.makedirs("/tmp/torchfuzz", exist_ok=True)
        with open("/tmp/torchfuzz/crash.log", "a") as f:
            f.write(f"BrokenPipeError: {msg}\n")


# List of regex patterns for ignore bucket
IGNORE_PATTERNS: list[re.Pattern] = [
    re.compile(
        r"torch\._inductor\.exc\.InductorError: AssertionError: -1"
    ),  # https://github.com/pytorch/pytorch/issues/167937
    # Add more patterns here as needed, e.g.:
    # re.compile(r"Some other error message"),
]
```
- **EN**: This chunk continues `persist_print` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `persist_print`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 56-67
```python

@dataclass
class FuzzerResult:
    seed: int
    success: bool
    output: str
    duration: float
    ignored_pattern_idx: int
    operation_stats: dict[str, int]  # New field for operation statistics


def is_ignored_output(output: str) -> int:
```
- **EN**: It introduces classes such as FuzzerResult, which package state and behavior for this tooling task. This chunk defines `is_ignored_output`, which implements a focused step inside the torch fuzzing pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 它引入了 FuzzerResult 等类，用来封装该工具任务所需的状态与行为。 这一段定义了 `is_ignored_output`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 68-81
```python
    """
    Check if the output matches any ignore pattern.

    Args:
        output: The combined stdout/stderr string.

    Returns:
        Index of the matched ignore pattern, or -1 if none matched.
    """
    for idx, pattern in enumerate(IGNORE_PATTERNS):
        if pattern.search(output):
            return idx
    return -1
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `is_ignored_output` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `is_ignored_output`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 82-95
```python

def run_fuzzer_with_seed(
    seed: int,
    template: str = "default",
    supported_ops: str | None = None,
) -> FuzzerResult:
    """
    Run fuzzer.py with a specific seed.

    Args:
        seed: The seed value to pass to fuzzer.py
        template: The template to use for code generation
        supported_ops: Comma-separated ops string with optional weights
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `run_fuzzer_with_seed`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `run_fuzzer_with_seed`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 96-112
```python
    Returns:
        FuzzerResult dataclass instance
    """
    start_time = time.time()

    try:
        # Run fuzzer.py with the specified seed and template
        cmd = [
            sys.executable,
            "fuzzer.py",
            "--single",
            "--seed",
            str(seed),
            "--template",
            template,
        ]
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `run_fuzzer_with_seed` and expands its internal control flow or data movement. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `run_fuzzer_with_seed`，进一步展开其内部控制流或数据流转。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 113-126
```python
        # Append supported ops if provided
        if supported_ops:
            cmd.extend(["--supported-ops", supported_ops])

        result = subprocess.run(
            cmd,
            capture_output=True,
            text=True,
            timeout=300,  # 5 minute timeout per seed
        )

        duration = time.time() - start_time
        success = result.returncode == 0
```
- **EN**: This chunk continues `run_fuzzer_with_seed` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `run_fuzzer_with_seed`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 127-140
```python
        # Combine stdout and stderr for output
        output = ""
        if result.stdout:
            output += f"STDOUT:\n{result.stdout}\n"
        if result.stderr:
            output += f"STDERR:\n{result.stderr}\n"
        output += f"Return code: {result.returncode}"

        # Parse operation statistics from the output
        operation_stats = {}
        if result.stdout:
            lines = result.stdout.split("\n")
            in_stats_section = False
            for line in lines:
```
- **EN**: This chunk continues `run_fuzzer_with_seed` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `run_fuzzer_with_seed`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 141-158
```python
                if line.strip() == "OPERATION_STATS:":
                    in_stats_section = True
                    continue
                elif in_stats_section:
                    if line.startswith("  ") and ":" in line:
                        # Parse line like "  torch.add: 3"
                        op_line = line.strip()
                        if ": " in op_line:
                            op_name, count_str = op_line.split(": ", 1)
                            try:
                                count = int(count_str)
                                operation_stats[op_name] = count
                            except ValueError:
                                pass  # Skip malformed lines
                    else:
                        # End of stats section
                        in_stats_section = False
```
- **EN**: This chunk continues `run_fuzzer_with_seed` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `run_fuzzer_with_seed`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 159-174
```python
        # Check if output should be ignored and which pattern matched
        ignored_pattern_idx = is_ignored_output(output)
        if ignored_pattern_idx != -1:
            # Mark as ignored (could also return a special flag if needed)
            output = "[IGNORED] " + output

        return FuzzerResult(
            seed, success, output, duration, ignored_pattern_idx, operation_stats
        )

    except subprocess.TimeoutExpired:
        duration = time.time() - start_time
        return FuzzerResult(
            seed, False, "Process timed out after 300 seconds", duration, -1, {}
        )
```
- **EN**: This chunk continues `run_fuzzer_with_seed` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `run_fuzzer_with_seed`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 175-187
```python
    except Exception as e:
        duration = time.time() - start_time
        return FuzzerResult(
            seed, False, f"Exception occurred: {str(e)}", duration, -1, {}
        )


def print_output_lines(output: str, write_func):
    """Helper to print non-empty lines of output using the provided write_func."""
    for line in output.split("\n"):
        if line.strip():
            write_func(f"   {line}")
    if hasattr(write_func, "__self__") and hasattr(write_func.__self__, "write"):
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `print_output_lines`, which implements a focused step inside the torch fuzzing pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `print_output_lines`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 188-205
```python
        # For tqdm.write, add an empty line for separation
        write_func("")


def handle_result_output(
    *,
    label: str,
    seed: int,
    duration: float,
    output: str,
    ignored: bool,
    verbose: bool,
    write_func,
):
    """Unified handler for result output, reducing code repetition."""
    ignored_text = " [IGNORED]" if ignored else ""
    write_func(f"{label} - Seed {seed} (duration: {duration:.2f}s){ignored_text}")
    if output.strip() or label.startswith("❌") or verbose:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `handle_result_output`, which implements a focused step inside the torch fuzzing pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `handle_result_output`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 206-219
```python
        print_output_lines(output, write_func)


def run_multi_process_fuzzer(
    num_processes: int | None = None,
    seed_start: int = 0,
    seed_count: int = 100,
    verbose: bool = False,
    template: str = "default",
    supported_ops: str | None = None,
) -> None:
    """
    Run the multi-process fuzzer.
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `run_multi_process_fuzzer`, which orchestrates command-line execution and forwards parsed arguments into the core logic.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `run_multi_process_fuzzer`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。

### Lines 220-238
```python
    Args:
        num_processes: Number of worker processes to use
        seed_start: Starting seed value (inclusive)
        seed_count: Number of seeds to run
        verbose: Whether to print detailed output
        template: The template to use for code generation
        supported_ops: Comma-separated ops string with optional weights
    """
    seeds = list(range(seed_start, seed_start + seed_count))

    persist_print(f"🚀 Starting multi-process fuzzer with {num_processes} processes")
    persist_print(
        f"📊 Processing seeds {seed_start} to {seed_start + seed_count - 1} ({len(seeds)} total)"
    )
    persist_print(
        f"🔧 Command template: python fuzzer.py --seed {{seed}} --template {template}"
    )
    persist_print("=" * 60)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `run_multi_process_fuzzer` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `run_multi_process_fuzzer`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 239-254
```python
    start_time = time.time()
    results: list[FuzzerResult] = []
    successful_count = 0
    failed_count = 0
    ignored_count = 0
    ignored_seeds = []
    ignored_pattern_counts: dict[int, int] = dict.fromkeys(
        range(len(IGNORE_PATTERNS)), 0
    )

    try:
        # Use multiprocessing Pool to distribute work
        with mp.Pool(processes=num_processes) as pool:
            # Submit all seeds to the process pool
            future_results = []
            for seed in seeds:
```
- **EN**: This chunk continues `run_multi_process_fuzzer` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `run_multi_process_fuzzer`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 255-274
```python
                future = pool.apply_async(
                    run_fuzzer_with_seed, (seed, template, supported_ops)
                )
                future_results.append(future)

            # Set up progress bar
            if HAS_TQDM:
                from tqdm import tqdm  # Import the real tqdm here

                pbar = tqdm(
                    total=len(seeds),
                    desc="Processing seeds",
                    file=sys.stdout,
                    bar_format="{l_bar}{bar}| {n_fmt}/{total_fmt} [{elapsed}<{remaining}] ✅/❌/❓={postfix}",
                    dynamic_ncols=True,
                )
                pbar.set_postfix_str(
                    f"{successful_count}/{failed_count}/{ignored_count} | throughput: 0.00 seeds/hr"
                )
```
- **EN**: This block assembles the Python-side dependencies, importing external packages such as tqdm. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段组织 Python 侧依赖，引入了外部依赖包，如 tqdm。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 275-288
```python
                def write_func(msg):
                    # pyrefly: ignore [missing-attribute]
                    pbar.write(msg)
            else:
                persist_print("Progress: (install tqdm for better progress bar)")
                pbar = None
                write_func = persist_print

            # Collect results as they complete
            for i, future in enumerate(future_results):
                try:
                    result: FuzzerResult = future.get()
                    results.append(result)
```
- **EN**: This chunk defines `write_func`, which generates derived source text, templates, or metadata outputs. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段定义了 `write_func`，其作用是生成派生源码文本、模板或元数据输出。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 289-302
```python
                    if result.ignored_pattern_idx != -1:
                        ignored_seeds.append(result.seed)
                        ignored_pattern_counts[result.ignored_pattern_idx] += 1
                        ignored_count += 1

                    # Only increment failed_count if not ignored
                    if result.success:
                        successful_count += 1
                    elif result.ignored_pattern_idx == -1:
                        failed_count += 1

                    elapsed = time.time() - start_time
                    throughput = (i + 1) / (elapsed / 3600)
```
- **EN**: This chunk continues `write_func` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `write_func`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 303-317
```python
                    # Update progress bar
                    if HAS_TQDM and pbar:
                        pbar.set_postfix_str(
                            f"{successful_count}/{failed_count}/{ignored_count} | throughput: {throughput:.2f} seeds/hr"
                        )
                        pbar.update(1)
                    else:
                        status_emoji = "✅" if result.success else "❌"
                        ignored_text = (
                            " (IGNORED)" if result.ignored_pattern_idx != -1 else ""
                        )
                        persist_print(
                            f"Completed {i + 1}/{len(seeds)} - Seed {result.seed}: {status_emoji}{ignored_text}"
                        )
```
- **EN**: This chunk continues `write_func` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `write_func`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 318-330
```python
                    # Unified output handling
                    if not result.success and result.ignored_pattern_idx == -1:
                        handle_result_output(
                            label="❌ FAILURE",
                            seed=result.seed,
                            duration=result.duration,
                            output=result.output,
                            ignored=False,
                            verbose=verbose,
                            write_func=write_func,
                        )
                    elif not result.success and result.ignored_pattern_idx != -1:
                        if verbose:
```
- **EN**: This chunk continues `write_func` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `write_func`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 331-350
```python
                            handle_result_output(
                                label="🚫 IGNORED",
                                seed=result.seed,
                                duration=result.duration,
                                output=result.output,
                                ignored=True,
                                verbose=verbose,
                                write_func=write_func,
                            )
                    elif verbose:
                        handle_result_output(
                            label="✅ SUCCESS",
                            seed=result.seed,
                            duration=result.duration,
                            output=result.output,
                            ignored=(result.ignored_pattern_idx != -1),
                            verbose=verbose,
                            write_func=write_func,
                        )
```
- **EN**: This chunk continues `write_func` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `write_func`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 351-367
```python
                except Exception as e:
                    failed_count += 1
                    if HAS_TQDM and pbar:
                        pbar.set_postfix_str(f"{successful_count}/{failed_count}")
                        pbar.update(1)
                        pbar.write(f"❌ POOL ERROR - Seed {seeds[i]}: {str(e)}")
                    else:
                        persist_print(
                            f"Completed {i + 1}/{len(seeds)} - Seed {seeds[i]}: ❌ POOL ERROR"
                        )
                        persist_print(f"❌ POOL ERROR - Seed {seeds[i]}: {str(e)}")
                    results.append(
                        FuzzerResult(
                            seeds[i], False, f"Pool error: {str(e)}", 0.0, -1, {}
                        )
                    )
```
- **EN**: This chunk continues `write_func` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `write_func`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 368-385
```python
            # Close progress bar
            if HAS_TQDM and pbar:
                pbar.close()
    except KeyboardInterrupt:
        persist_print("\n🛑 Interrupted by user (Ctrl+C)")
        # Print summary up to this point
        total_time = time.time() - start_time
        persist_print("=" * 60)
        persist_print("📈 SUMMARY (partial, interrupted)")
        persist_print("=" * 60)

        successful = [res for res in results if res.success]
        # Only count as failed if not ignored
        failed = [
            res for res in results if not res.success and res.ignored_pattern_idx == -1
        ]
        ignored = [res for res in results if res.ignored_pattern_idx != -1]
```
- **EN**: This chunk continues `write_func` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `write_func`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 386-399
```python
        persist_print(
            f"✅ Successful: {len(successful)}/{len(results)} ({(len(successful) / len(results) * 100 if results else 0):.1f}%)"
        )
        persist_print(
            f"❌ Failed:     {len(failed)}/{len(results)} ({(len(failed) / len(results) * 100 if results else 0):.1f}%)"
        )
        persist_print(f"⏱️  Total time: {total_time:.2f}s")
        if results:
            persist_print(
                f"⚡ Throughput: {(len(results) / (total_time / 3600)):.2f} seeds/hr"
                if total_time > 0
                else "⚡ Throughput: N/A"
            )
        if failed:
```
- **EN**: This chunk continues `write_func` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `write_func`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 400-416
```python
            persist_print(f"\n❌ Failed seeds: {[res.seed for res in failed]}")
        if successful:
            persist_print(f"✅ Successful seeds: {[res.seed for res in successful]}")
            avg_success_time = sum(res.duration for res in successful) / len(successful)
            persist_print(f"⚡ Avg time for successful runs: {avg_success_time:.2f}s")
        if ignored:
            persist_print(f"\n🚫 Ignored seeds: {[res.seed for res in ignored]}")
            # Print ignore pattern stats
            persist_print("\n🚫 Ignored pattern statistics:")
            total_ignored = len(ignored)
            for idx, pattern in enumerate(IGNORE_PATTERNS):
                count = ignored_pattern_counts[idx]
                percent = (count / total_ignored * 100) if total_ignored else 0
                persist_print(
                    f"  Pattern {idx}: {pattern.pattern!r} - {count} ({percent:.1f}%)"
                )
```
- **EN**: This chunk continues `write_func` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `write_func`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 417-428
```python
        # Aggregate and print operation distribution
        _print_operation_distribution(results)

        sys.exit(130)

    total_time = time.time() - start_time

    # Print summary
    persist_print("=" * 60)
    persist_print("📈 SUMMARY")
    persist_print("=" * 60)
```
- **EN**: This chunk continues `write_func` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `write_func`，进一步展开其内部控制流或数据流转。

### Lines 429-445
```python
    successful = [res for res in results if res.success]
    # Only count as failed if not ignored
    failed = [
        res for res in results if not res.success and res.ignored_pattern_idx == -1
    ]
    ignored = [res for res in results if res.ignored_pattern_idx != -1]

    persist_print(
        f"✅ Successful: {len(successful)}/{len(results)} ({len(successful) / len(results) * 100:.1f}%)"
    )
    persist_print(
        f"❌ Failed:     {len(failed)}/{len(results)} ({len(failed) / len(results) * 100:.1f}%)"
    )
    persist_print(f"⏱️  Total time: {total_time:.2f}s")
    persist_print(
        f"⚡ Throughput: {(len(results) / (total_time / 3600)):.2f} seeds/hr"
        if total_time > 0
```
- **EN**: This chunk continues `write_func` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `write_func`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 446-457
```python
        else "⚡ Throughput: N/A"
    )

    if failed:
        persist_print(f"\n❌ Failed seeds: {[res.seed for res in failed]}")

    if successful:
        persist_print(f"✅ Successful seeds: {[res.seed for res in successful]}")
        avg_success_time = sum(res.duration for res in successful) / len(successful)
        persist_print(f"⚡ Avg time for successful runs: {avg_success_time:.2f}s")

    if ignored:
```
- **EN**: This chunk continues `write_func` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `write_func`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 458-471
```python
        persist_print(f"\n🚫 Ignored seeds: {[res.seed for res in ignored]}")
        # Print ignore pattern stats
        persist_print("\n🚫 Ignored pattern statistics:")
        total_ignored = len(ignored)
        for idx, pattern in enumerate(IGNORE_PATTERNS):
            count = ignored_pattern_counts[idx]
            percent = (count / total_ignored * 100) if total_ignored else 0
            persist_print(
                f"  Pattern {idx}: {pattern.pattern!r} - {count} ({percent:.1f}%)"
            )

    # Aggregate and print operation distribution
    _print_operation_distribution(results)
```
- **EN**: This chunk continues `write_func` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `write_func`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 472-484
```python

def _print_operation_distribution(results: list[FuzzerResult]) -> None:
    """Helper function to print operation distribution statistics."""
    total_operation_stats = defaultdict(int)
    total_operations = 0

    # Collect operation stats from all successful results
    for result in results:
        if result.success and result.operation_stats:
            for op_name, count in result.operation_stats.items():
                total_operation_stats[op_name] += count
                total_operations += count
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `_print_operation_distribution`, which implements a focused step inside the torch fuzzing pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `_print_operation_distribution`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 485-496
```python
    if total_operation_stats:
        persist_print("\n📊 OPERATION DISTRIBUTION")
        persist_print("=" * 60)
        persist_print(f"Total operations executed: {total_operations}")
        persist_print("")

        # Sort operations by count (descending) for better readability
        sorted_ops = sorted(
            total_operation_stats.items(), key=lambda x: x[1], reverse=True
        )

        for op_name, count in sorted_ops:
```
- **EN**: This chunk continues `_print_operation_distribution` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `_print_operation_distribution`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 497-513
```python
            percentage = (count / total_operations * 100) if total_operations > 0 else 0
            persist_print(f"  {op_name:<30} {count:>6} times ({percentage:>5.1f}%)")
    else:
        persist_print(
            "\n📊 No operation statistics collected (no successful runs with stats)"
        )


def run_until_failure(
    num_processes: int | None = None,
    verbose: bool = False,
    template: str = "default",
    supported_ops: str | None = None,
) -> None:
    """
    Run the multi-process fuzzer with a random starting seed, iterating until a failure is found.
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `run_until_failure`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `run_until_failure`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 514-527
```python
    Args:
        num_processes: Number of worker processes to use
        verbose: Whether to print detailed output
        template: The template to use for code generation
        supported_ops: Comma-separated ops string with optional weights

    Returns:
        Exits with non-zero code when a failure is found
    """
    import random

    # Pick a random seed to start from
    initial_seed = random.randint(0, 2**31 - 1)
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as random. This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 random。 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 528-543
```python
    persist_print(
        f"🎲 Starting continuous fuzzing with random initial seed: {initial_seed}"
    )
    persist_print(f"🚀 Using {num_processes} processes")
    persist_print(
        f"🔧 Command template: python fuzzer.py --seed {{seed}} --template {template}"
    )
    persist_print("🎯 Running until first failure is found...")
    persist_print("=" * 60)

    start_time = time.time()
    current_seed = initial_seed
    total_successful = 0
    total_ignored = 0
    batch_size = 100  # Process seeds in batches of 100
```
- **EN**: This chunk continues `run_until_failure` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `run_until_failure`，进一步展开其内部控制流或数据流转。

### Lines 544-556
```python
    try:
        while True:
            # Process a batch of seeds
            seeds = list(range(current_seed, current_seed + batch_size))

            with mp.Pool(processes=num_processes) as pool:
                future_results = []
                for seed in seeds:
                    future = pool.apply_async(
                        run_fuzzer_with_seed, (seed, template, supported_ops)
                    )
                    future_results.append((seed, future))
```
- **EN**: This chunk continues `run_until_failure` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `run_until_failure`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 557-569
```python
                # Set up progress bar for this batch
                if HAS_TQDM:
                    from tqdm import tqdm

                    pbar = tqdm(
                        total=len(seeds),
                        desc=f"Batch starting at seed {current_seed}",
                        file=sys.stdout,
                        bar_format="{l_bar}{bar}| {n_fmt}/{total_fmt} [{elapsed}] ✅/🚫={postfix}",
                        dynamic_ncols=True,
                    )
                    pbar.set_postfix_str(f"{total_successful}/{total_ignored}")
```
- **EN**: This block assembles the Python-side dependencies, importing external packages such as tqdm. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段组织 Python 侧依赖，引入了外部依赖包，如 tqdm。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 570-582
```python
                    def write_func(msg):
                        # pyrefly: ignore [missing-attribute]
                        pbar.write(msg)
                else:
                    pbar = None

                # Collect results as they complete
                for seed, future in future_results:
                    result: FuzzerResult = future.get()

                    if result.ignored_pattern_idx != -1:
                        total_ignored += 1
```
- **EN**: This chunk defines `write_func`, which generates derived source text, templates, or metadata outputs. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `write_func`，其作用是生成派生源码文本、模板或元数据输出。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 583-606
```python
                    if result.success:
                        total_successful += 1
                    elif result.ignored_pattern_idx == -1:
                        # Found a failure that is not ignored!
                        if HAS_TQDM and pbar:
                            pbar.close()

                        elapsed = time.time() - start_time
                        persist_print("\n" + "=" * 60)
                        persist_print("🎯 FAILURE FOUND!")
                        persist_print("=" * 60)
                        persist_print(f"❌ Failing seed: {result.seed}")
                        persist_print(
                            f"⏱️  Duration for this seed: {result.duration:.2f}s"
                        )
                        persist_print(f"⏱️  Total time elapsed: {elapsed:.2f}s")
                        persist_print(f"✅ Successful seeds tested: {total_successful}")
                        persist_print(f"🚫 Ignored seeds: {total_ignored}")
                        persist_print(
                            f"📊 Total seeds tested: {total_successful + total_ignored + 1}"
                        )
                        persist_print("\n💥 Failure output:")
                        persist_print("-" * 60)
                        print_output_lines(result.output, persist_print)
```
- **EN**: This chunk continues `write_func` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `write_func`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 607-622
```python
                        persist_print("-" * 60)
                        persist_print(
                            f"\n🔄 Reproduce with: python fuzzer.py --seed {result.seed} --template {template}"
                        )

                        # Exit with non-zero code
                        sys.exit(1)

                    # Update progress bar
                    if HAS_TQDM and pbar:
                        pbar.set_postfix_str(f"{total_successful}/{total_ignored}")
                        pbar.update(1)
                    elif verbose:
                        status_emoji = "✅" if result.success else "🚫"
                        persist_print(f"Seed {result.seed}: {status_emoji}")
```
- **EN**: This chunk continues `write_func` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `write_func`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 623-643
```python
                # Close progress bar for this batch
                if HAS_TQDM and pbar:
                    pbar.close()

            # Move to next batch
            current_seed += batch_size

    except KeyboardInterrupt:
        persist_print("\n🛑 Interrupted by user (Ctrl+C)")
        elapsed = time.time() - start_time
        persist_print("=" * 60)
        persist_print("📈 SUMMARY (interrupted)")
        persist_print("=" * 60)
        persist_print(f"⏱️  Total time: {elapsed:.2f}s")
        persist_print(f"✅ Successful seeds: {total_successful}")
        persist_print(f"🚫 Ignored seeds: {total_ignored}")
        persist_print(f"📊 Total seeds tested: {total_successful + total_ignored}")
        persist_print(
            f"⚡ Throughput: {((total_successful + total_ignored) / (elapsed / 3600)):.2f} seeds/hr"
        )
        sys.exit(130)
```
- **EN**: This chunk continues `write_func` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `write_func`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

## Key Concepts / 关键概念

- **Torch fuzzing**
  - EN: This file belongs to the torch fuzzing layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于Torch 模糊测试层，应结合同一子目录中的相邻脚本一起理解。
- **Template expansion**
  - EN: The implementation relies on placeholders or structured text expansion to generate source artifacts.
  - CN: 该实现依赖占位符或结构化文本展开来生成源码产物。
- **GitHub automation**
  - EN: The file integrates with pull requests, workflow metadata, or repository automation flows.
  - CN: 该文件与 Pull Request、工作流元数据或仓库自动化流程集成。
- **Statistics reporting**
  - EN: The implementation aggregates measurements and turns them into summaries or dashboards.
  - CN: 该实现聚合度量数据，并将其转化为摘要或仪表板。
- **Process orchestration**
  - EN: The file launches external commands and translates their results back into Python control flow.
  - CN: 该文件负责启动外部命令，并把结果重新映射回 Python 控制流。
- **Fuzz testing**
  - EN: The implementation creates randomized or adversarial inputs to probe API robustness.
  - CN: 该实现构造随机或对抗性输入，以探测 API 的健壮性。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `multiprocessing`, `re`, `subprocess`, `sys`, `time`, `collections`, `dataclasses`, `os`, `random`
- **External packages / 外部依赖包**: `tqdm`
- **Primary symbols in this file / 本文件核心符号**: `persist_print`, `FuzzerResult`, `is_ignored_output`, `run_fuzzer_with_seed`, `print_output_lines`, `handle_result_output`, `run_multi_process_fuzzer`, `_print_operation_distribution`, `run_until_failure`
