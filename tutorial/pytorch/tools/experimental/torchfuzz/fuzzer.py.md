# fuzzer.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/experimental/torchfuzz/fuzzer.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements experimental fuzzing helpers used to stress PyTorch operators or APIs.
- **Purpose (CN)**: 实现实验性模糊测试辅助逻辑，用于压测 PyTorch 算子或 API。
## Line-by-Line Analysis / 逐行分析

### Lines 1-12
```python
# mypy: ignore-errors
import logging
import multiprocessing as mp
import os
import random
import sys


# Add parent directory to path so we can import torchfuzz as a module
current_dir = os.path.dirname(os.path.abspath(__file__))
parent_dir = os.path.dirname(current_dir)
if parent_dir not in sys.path:
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as logging, multiprocessing, os, and 2 more. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 logging、multiprocessing、os 等共 5 项。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 13-21
```python
    sys.path.insert(0, parent_dir)

import torch
from torchfuzz.codegen import convert_graph_to_python_code, create_program_file
from torchfuzz.ops_fuzzer import fuzz_operation_graph, fuzz_spec
from torchfuzz.runner import ProgramRunner
from torchfuzz.visualize_graph import visualize_operation_graph
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as torch; external packages such as torchfuzz.codegen, torchfuzz.ops_fuzzer, torchfuzz.runner, and 1 more.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 torch；外部依赖包，如 torchfuzz.codegen、torchfuzz.ops_fuzzer、torchfuzz.runner 等共 4 项。

### Lines 22-31
```python
def _parse_supported_ops_with_weights(spec: str) -> tuple[list[str], dict[str, float]]:
    """Parse --supported-ops string.

    Format: comma-separated fully-qualified torch ops, each optionally with =weight.
    Example: "torch.matmul=5,torch.nn.functional.rms_norm=5,torch.add"
    Returns (ops_list, weights_dict)
    """
    ops: list[str] = []
    weights: dict[str, float] = {}
    if not spec:
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `_parse_supported_ops_with_weights`, which parses or loads structured input into tool-friendly data structures. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `_parse_supported_ops_with_weights`，其作用是把结构化输入解析或加载为工具可处理的数据结构。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 32-40
```python
        return ops, weights
    for entry in spec.split(","):
        entry = entry.strip()
        if not entry:
            continue
        if "=" in entry:
            name, w = entry.split("=", 1)
            name = name.strip()
            try:
```
- **EN**: This chunk continues `_parse_supported_ops_with_weights` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `_parse_supported_ops_with_weights`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 41-49
```python
                weight = float(w.strip())
            except ValueError:
                continue
            ops.append(name)
            weights[name] = weight
        else:
            ops.append(entry)
    return ops, weights
```
- **EN**: This chunk continues `_parse_supported_ops_with_weights` and expands its internal control flow or data movement. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `_parse_supported_ops_with_weights`，进一步展开其内部控制流或数据流转。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 50-61
```python

def fuzz_and_execute(
    seed: int | None = None,
    max_depth: int | None = None,
    log_at_faluire: bool = False,
    template: str = "default",
    supported_ops: list[str] | None = None,
    op_weights: dict[str, float] | None = None,
) -> None:
    """
    Generate a fuzzed operation stack, convert it to Python code, and execute it.
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `fuzz_and_execute`, which constructs randomized inputs to probe edge cases and robustness boundaries.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `fuzz_and_execute`，其作用是构造随机输入以探测边界情况与健壮性极限。

### Lines 62-73
```python
    Args:
        seed: Random seed for reproducible generation. If None, uses a random seed.
        max_depth: Maximum depth for operation stack (1-10). If None, uses a random depth.

    This function:
    1. Generates a random target specification
    2. Creates a stack of operations to produce that target
    3. Converts the stack into executable Python code
    4. Executes the generated Python code
    5. Validates the final result matches the target spec
    """
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `fuzz_and_execute` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `fuzz_and_execute`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 74-85
```python
    # Generate seed if not provided
    if seed is None:
        seed = random.randint(0, 2**31 - 1)

    # Generate max_depth if not provided (range 3-12)
    if max_depth is None:
        random.seed(seed + 999)  # Use seed offset for consistent depth selection
        max_depth = random.randint(2, 4)
    else:
        # Clamp max_depth to valid range
        max_depth = max(1, max_depth)
```
- **EN**: This chunk continues `fuzz_and_execute` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `fuzz_and_execute`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 86-94
```python
    print(f"Using seed: {seed}, max_depth: {max_depth}")

    # Set seed for reproducible generation
    random.seed(seed)
    torch.manual_seed(seed)
    operation_stack = None
    python_code = None
    target_spec = None
```
- **EN**: This chunk continues `fuzz_and_execute` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `fuzz_and_execute`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 95-106
```python
    def log(success: bool) -> None:
        import os
        import time

        # Create a unique folder for this iteration
        timestamp = int(time.time() * 1000)  # milliseconds
        folder_name = (
            f"fuzzing_seed_{seed}_{timestamp}_{'success' if success else 'failed'}"
        )
        iteration_folder = os.path.join("/tmp", folder_name)
        os.makedirs(iteration_folder, exist_ok=True)
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as os, time. This chunk defines `log`, which implements a focused step inside the torch fuzzing pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 os、time。 这一段定义了 `log`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 107-116
```python
        # Write summary file
        summary_path = os.path.join(iteration_folder, "summary.txt")
        with open(summary_path, "w") as f:
            f.write("Fuzzing Session Summary\n")
            f.write("======================\n")
            f.write(f"Seed: {seed}\n")
            f.write(f"Max depth: {max_depth}\n")
            f.write(f"Success: {success}\n")
            f.write(f"Target specification: {target_spec}\n")
            if operation_stack:
```
- **EN**: This chunk continues `log` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `log`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 117-126
```python
                f.write(f"Operations count: {len(operation_stack)}\n")

        if operation_stack:
            # Write operation stack to file in iteration folder
            stack_file_path = os.path.join(iteration_folder, "operation_stack.txt")
            with open(stack_file_path, "w") as f:
                f.write(f"Target specification: {target_spec}\n")
                f.write(f"Generated {len(operation_stack)} operations in stack\n\n")
                f.write("Operation stack (in reverse order - dependencies first):\n")
                for i in range(len(operation_stack) - 1, -1, -1):
```
- **EN**: This chunk continues `log` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `log`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 127-136
```python
                    op = operation_stack[i]
                    f.write(
                        f"  {i}: {op.op_name} -> {op.output_spec} (depth {op.depth})\n"
                    )

            # Generate visualization in the iteration folder
            visualize_operation_graph(
                operation_graph, "Operation Graph", iteration_folder
            )
```
- **EN**: This chunk continues `log` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `log`，进一步展开其内部控制流或数据流转。

### Lines 137-146
```python
    import time

    try:
        logger = logging.getLogger(__name__)

        # Generate target specification first
        logger.debug("⏱️  Step 1: Generating target spec...")
        start_time = time.time()
        target_spec = fuzz_spec(template)
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as time. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 time。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 147-155
```python
        # Apply user-specified operator weights (if provided)
        if op_weights:
            from torchfuzz.operators import set_operator_weights

            set_operator_weights(op_weights)
        logger.debug(
            "   Completed in %.3fs - %s", time.time() - start_time, target_spec
        )
```
- **EN**: This block assembles the Python-side dependencies, importing external packages such as torchfuzz.operators. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段组织 Python 侧依赖，引入了外部依赖包，如 torchfuzz.operators。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 156-165
```python
        logger.debug("⏱️  Step 2: Generating operation graph...")
        start_time = time.time()
        operation_graph = fuzz_operation_graph(
            target_spec,
            max_depth=max_depth,
            seed=seed,
            template=template,
            supported_ops=supported_ops,
        )
```
- **EN**: This chunk continues `log` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `log`，进一步展开其内部控制流或数据流转。

### Lines 166-174
```python
        # Extract and print operation statistics
        operation_counts = {}
        for node in operation_graph.nodes.values():
            # Use the fully qualified torch operation name if available
            from torchfuzz.operators import get_operator

            # Try to get the fully qualified torch operation name
            torch_op_name = None
```
- **EN**: This block assembles the Python-side dependencies, importing external packages such as torchfuzz.operators. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段组织 Python 侧依赖，引入了外部依赖包，如 torchfuzz.operators。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 175-183
```python
            # Extract the base operation name (without arg_X suffixes)
            base_op_name = node.op_name
            if node.op_name.startswith("arg_"):
                # For arg operations, use just "arg" to look up in registry
                base_op_name = "arg"

            try:
                operator = get_operator(base_op_name)
                if (
```
- **EN**: This chunk continues `log` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `log`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 184-192
```python
                    operator
                    and hasattr(operator, "torch_op_name")
                    and operator.torch_op_name
                ):
                    torch_op_name = operator.torch_op_name
            except (KeyError, ValueError):
                # If the operator doesn't exist in registry, use the node's op_name
                pass
```
- **EN**: This chunk continues `log` and expands its internal control flow or data movement. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `log`，进一步展开其内部控制流或数据流转。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 193-201
```python
            # Use fully qualified name if available, otherwise use the node's op_name
            display_name = torch_op_name if torch_op_name else node.op_name
            operation_counts[display_name] = operation_counts.get(display_name, 0) + 1

        # Print operation statistics in a parseable format
        print("OPERATION_STATS:")
        for op_name, count in sorted(operation_counts.items()):
            print(f"  {op_name}: {count}")
```
- **EN**: This chunk continues `log` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `log`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 202-212
```python
        logger.debug("⏱️  Step 3: Converting to Python code...")
        start_time = time.time()
        python_code = convert_graph_to_python_code(
            operation_graph, seed=seed, template=template
        )
        logger.debug(
            "   Completed in %.3fs - %d chars",
            time.time() - start_time,
            len(python_code),
        )
```
- **EN**: This chunk continues `log` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `log`，进一步展开其内部控制流或数据流转。

### Lines 213-222
```python
        logger.debug("⏱️  Step 4: Executing Python code...")
        start_time = time.time()

        # Create program file and run with new runner
        program_path = create_program_file(python_code)
        runner = ProgramRunner()
        runner.run_program(program_path)

        logger.debug("   Completed in %.3fs", time.time() - start_time)
```
- **EN**: This chunk continues `log` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `log`，进一步展开其内部控制流或数据流转。

### Lines 223-232
```python
        # # Validate the result matches target specification
        if not log_at_faluire:
            log(True)

    except Exception as e:
        print(f"\n❌ Execution failed: {e}")
        # from visualize_stack import visualize_operation_stack
        log(False)
        import traceback
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as traceback. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 traceback。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 233-241
```python
        traceback.print_exc()
        error_message = str(e)
        print(f"Error: {error_message}")
        sys.exit(1)


if __name__ == "__main__":
    import argparse
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as argparse. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 argparse。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 242-252
```python
    try:
        from multi_process_fuzzer import run_multi_process_fuzzer, run_until_failure
    except ImportError:
        # If importing as a module fails, import from the same directory
        import os
        import sys

        current_dir = os.path.dirname(os.path.abspath(__file__))
        sys.path.insert(0, current_dir)
        from multi_process_fuzzer import run_multi_process_fuzzer, run_until_failure
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as os, sys; external packages such as multi_process_fuzzer. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 os、sys；外部依赖包，如 multi_process_fuzzer。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 253-270
```python
    # Set up command-line argument parsing
    parser = argparse.ArgumentParser(
        description="PyTorch Fuzzer - Generate and test random PyTorch operations"
    )

    # Single seed execution arguments
    parser.add_argument("--seed", type=int, help="Random seed for single execution")
    parser.add_argument(
        "--max-depth", type=int, help="Maximum depth for operation stack (1-20)"
    )
    parser.add_argument(
        "--template",
        choices=["default", "dtensor", "dtensor_placements", "unbacked", "streams"],
        default="default",
        help="Template to use for code generation (default: default)",
    )
    parser.add_argument(
        "--supported-ops",
```
- **EN**: This chunk continues `log` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `log`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 271-288
```python
        type=str,
        help=(
            "Comma-separated fully-qualified torch ops to allow, each optionally with =weight. "
            "Examples: 'torch.matmul,torch.nn.functional.rms_norm' or "
            "'torch.matmul=5,torch.nn.functional.rms_norm=5'. Overrides template supported ops."
        ),
    )

    # Multi-process fuzzing arguments
    parser.add_argument(
        "--start", type=int, help="Starting seed value for multi-process fuzzing"
    )
    parser.add_argument(
        "--count", type=int, help="Number of seeds to run in multi-process fuzzing"
    )
    parser.add_argument(
        "--processes",
        "-p",
```
- **EN**: This chunk continues `log` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `log`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 289-303
```python
        type=int,
        help="Number of worker processes to use (default: auto-detected)",
    )
    parser.add_argument(
        "--verbose",
        "-v",
        action="store_true",
        help="Print detailed output for all runs (not just failures)",
    )
    parser.add_argument(
        "--stop-at-first-failure",
        action="store_true",
        help="Pick a random seed and keep iterating until finding a failure (exits with non-zero code)",
    )
```
- **EN**: This chunk continues `log` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `log`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 304-316
```python
    # Legacy arguments
    parser.add_argument(
        "--single",
        action="store_true",
        help="Run a single fuzz_and_execute (deprecated, use --seed)",
    )
    parser.add_argument(
        "--log-level",
        choices=["DEBUG", "INFO", "WARNING", "ERROR"],
        default="INFO",
        help="Set the logging level (default: INFO)",
    )
```
- **EN**: This chunk continues `log` and expands its internal control flow or data movement. Argument-parsing logic turns command-line inputs into structured parameters for the tool core.
- **CN**: 这一段延续了 `log`，进一步展开其内部控制流或数据流转。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。

### Lines 317-326
```python
    args = parser.parse_args()

    # Configure logging
    logging.basicConfig(
        level=getattr(logging, args.log_level), format="%(levelname)s: %(message)s"
    )
    logger = logging.getLogger(__name__)

    # Determine execution mode
    if args.seed is not None or args.single:
```
- **EN**: This chunk continues `log` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `log`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 327-336
```python
        # Single seed execution mode
        print("Running single fuzz_and_execute...")
        # Parse supported ops and any inline weights from that flag
        parsed_supported_ops: list[str] | None = None
        parsed_weights: dict[str, float] = {}
        if args.supported_ops:
            parsed_supported_ops, parsed_weights = _parse_supported_ops_with_weights(
                args.supported_ops
            )
```
- **EN**: This chunk continues `log` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `log`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 337-347
```python
        fuzz_and_execute(
            seed=args.seed,
            max_depth=args.max_depth,
            template=args.template,
            supported_ops=parsed_supported_ops,
            op_weights=(parsed_weights if parsed_weights else None),
        )
    elif args.stop_at_first_failure:
        # Stop-at-first-failure mode
        # Default number of processes
        if args.processes is None:
```
- **EN**: This chunk continues `log` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `log`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 348-365
```python
            cpu_count = mp.cpu_count()
            args.processes = max(1, min(16, int(cpu_count * 0.75)))

        if args.processes < 1:
            print("❌ Error: Number of processes must be at least 1")
            sys.exit(1)

        try:
            run_until_failure(
                num_processes=args.processes,
                verbose=args.verbose,
                template=args.template,
                supported_ops=args.supported_ops,
            )
        except Exception as e:
            print(f"❌ Unexpected error: {str(e)}")
            import traceback
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as traceback. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 traceback。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 366-376
```python
            traceback.print_exc()
            sys.exit(1)
    elif args.start is not None or args.count is not None:
        # Multi-process fuzzing mode
        if args.start is None:
            print("❌ Error: --start is required when --count is specified")
            sys.exit(1)
        if args.count is None:
            print("❌ Error: --count is required when --start is specified")
            sys.exit(1)
```
- **EN**: This chunk continues `log` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `log`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 377-386
```python
        # Validate arguments
        if args.count < 1:
            print("❌ Error: --count must be at least 1")
            sys.exit(1)

        # Default number of processes
        if args.processes is None:
            cpu_count = mp.cpu_count()
            args.processes = max(1, min(16, int(cpu_count * 0.75)))
```
- **EN**: This chunk continues `log` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `log`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 387-403
```python
        if args.processes < 1:
            print("❌ Error: Number of processes must be at least 1")
            sys.exit(1)

        try:
            run_multi_process_fuzzer(
                num_processes=args.processes,
                seed_start=args.start,
                seed_count=args.count,
                verbose=args.verbose,
                template=args.template,
                supported_ops=args.supported_ops,
            )
        except Exception as e:
            print(f"❌ Unexpected error: {str(e)}")
            import traceback
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as traceback. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 traceback。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 404-414
```python
            traceback.print_exc()
            sys.exit(1)
    else:
        # Show help when no arguments are provided
        parser.print_help()
        print("\nExamples:")
        print("  python fuzzer.py --seed 42                    # Run single seed")
        print(
            "  python fuzzer.py --start 0 --count 1000       # Run multi-process fuzzing"
        )
        print("  python fuzzer.py --start 100 --count 50 -p 8  # Use 8 processes")
```
- **EN**: This chunk continues `log` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `log`，进一步展开其内部控制流或数据流转。

## Key Concepts / 关键概念

- **Torch fuzzing**
  - EN: This file belongs to the torch fuzzing layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于Torch 模糊测试层，应结合同一子目录中的相邻脚本一起理解。
- **Template expansion**
  - EN: The implementation relies on placeholders or structured text expansion to generate source artifacts.
  - CN: 该实现依赖占位符或结构化文本展开来生成源码产物。
- **Statistics reporting**
  - EN: The implementation aggregates measurements and turns them into summaries or dashboards.
  - CN: 该实现聚合度量数据，并将其转化为摘要或仪表板。
- **Fuzz testing**
  - EN: The implementation creates randomized or adversarial inputs to probe API robustness.
  - CN: 该实现构造随机或对抗性输入，以探测 API 的健壮性。
- **_parse_supported_ops_with_weights**
  - EN: `_parse_supported_ops_with_weights` is one of the main local symbols exposed or implemented here.
  - CN: `_parse_supported_ops_with_weights` 是此处暴露或实现的主要局部符号之一。
- **fuzz_and_execute**
  - EN: `fuzz_and_execute` is one of the main local symbols exposed or implemented here.
  - CN: `fuzz_and_execute` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `torch`
- **Python standard library / Python 标准库**: `logging`, `multiprocessing`, `os`, `random`, `sys`, `time`, `traceback`, `argparse`
- **External packages / 外部依赖包**: `torchfuzz.codegen`, `torchfuzz.ops_fuzzer`, `torchfuzz.runner`, `torchfuzz.visualize_graph`, `torchfuzz.operators`, `multi_process_fuzzer`
- **Primary symbols in this file / 本文件核心符号**: `_parse_supported_ops_with_weights`, `fuzz_and_execute`
