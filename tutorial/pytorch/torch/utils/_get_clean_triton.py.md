# _get_clean_triton.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/utils/_get_clean_triton.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements utility logic that supports broader PyTorch execution, packaging, or developer workflows. This specific file centers on `_get_clean_triton.py`. Runtime configuration, environment overrides, or feature gating are important in this module.
- **Purpose (CN) / 用途（中文）**: 实现支撑更广泛 PyTorch 执行、打包或开发者工作流的辅助逻辑。 该文件具体围绕 `_get_clean_triton.py` 展开。 运行时配置、环境变量覆盖或特性开关是此模块的重要主题。
## Line-by-Line Analysis / 逐行分析

### Lines 1-13 / 第 1-13 行
```python
# mypy: allow-untyped-defs
import argparse
import os
import re
import subprocess
import sys
from pathlib import Path


def remove_triton_function_declaration(source_code: str) -> str:
    remove_head = re.sub(r"(\n.+\s\'\'\'\n)", "\n", source_code)
    remove_tail = re.sub(r"(\'\'\'\,.+)", "\n", remove_head)
    return remove_tail
```
- **EN**: This block establishes the module dependencies, pulling in standard-library helpers such as argparse, os, re, subprocess. Key callable entry points in this range include `remove_triton_function_declaration`. They package a focused unit of behavior behind named helpers or APIs. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段建立模块依赖，引入了标准库辅助模块，如 argparse, os, re, subprocess。 这一段的重要可调用入口包括 `remove_triton_function_declaration`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 16-29 / 第 16-29 行
```python
def remove_async_compile(source_code: str) -> str:
    remove_top_level = str.replace(source_code, "async_compile = AsyncCompile()", "")
    remove_compile = str.replace(remove_top_level, "async_compile.wait(globals())", "")
    remove_del = str.replace(remove_compile, "del async_compile", "")
    return remove_del


def rename_kernels(source_code: str) -> str:
    pattern = r"(\w+)\s*=\s*async_compile\.triton\('triton_',\s"
    triton_kernel_decl = "def triton_"
    matches = [
        (match.end(), match.group(1))
        for match in re.finditer(pattern, source_code, re.DOTALL)
    ]
```
- **EN**: Key callable entry points in this range include `remove_async_compile`, `rename_kernels`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `remove_async_compile`, `rename_kernels`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 31-46 / 第 31-46 行
```python
    # Starting from the last match to avoid issues with shifting indices after replacements
    for end_index, captured_string in reversed(matches):
        # Find the index of the next "B" after the current match
        index_of_B = source_code.find(triton_kernel_decl, end_index)
        if index_of_B != -1:
            # Replace the triton_kernel_decl with the captured string
            source_code = (
                source_code[:index_of_B]
                + f"def {captured_string}"
                + source_code[index_of_B + len(triton_kernel_decl) :]
            )
        else:
            # If triton_kernel_decl is not found after the current match, continue to the next
            continue

    return source_code
```
- **EN**: Key callable entry points in this range include `rename_kernels`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `rename_kernels`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 49-60 / 第 49-60 行
```python
def merge_params(original_params: list[str], new_params: list[str]) -> list[str]:
    for idx in range(len(new_params)):
        if new_params[idx] == "T":
            new_params[idx] = original_params[idx]
    return new_params


def add_launch_params(
    original: str, kernel_to_params: dict[str, tuple[str, str]]
) -> str:
    # Regex to match the function call in the original string
    pattern = r"(\w+)\.run\((.*)\)"
```
- **EN**: Key callable entry points in this range include `merge_params`, `add_launch_params`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `merge_params`, `add_launch_params`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 62-73 / 第 62-73 行
```python
    def replace(match) -> str:
        # Extract parts from the regex match
        func_name = match.group(1)
        params = match.group(2)
        new_params, grid = kernel_to_params[func_name]
        new_params = merge_params(params.split(", "), new_params.split(", "))

        # Format the new function call
        new_string = f"{func_name}[{grid}]({', '.join(new_params)})"
        return new_string

    transformed = re.sub(pattern, replace, original)
```
- **EN**: Key callable entry points in this range include `add_launch_params`. They package a focused unit of behavior behind named helpers or APIs. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `add_launch_params`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 75-89 / 第 75-89 行
```python
    remove_inductor_wrappers = re.sub(
        r"@triton_heuristics[^@]*@triton.jit",
        r"@triton.jit",
        transformed,
        flags=re.DOTALL,
    )

    return remove_inductor_wrappers


def process_file(
    input_filename: str, output_filename: str, auto_generate_params: bool = True
) -> str:
    with open(input_filename) as file:
        source_code = file.read()
```
- **EN**: Key callable entry points in this range include `add_launch_params`, `process_file`. They package a focused unit of behavior behind named helpers or APIs. Context-manager usage makes resource, mode, or temporary-state boundaries explicit. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `add_launch_params`, `process_file`，它们把聚焦的行为封装成具名辅助函数或 API。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 91-108 / 第 91-108 行
```python
    transformed_code = source_code
    if "def triton_(" in source_code:
        raise RuntimeError(
            "Need to run original Pytorch code generating kernels with TORCHINDUCTOR_UNIQUE_KERNEL_NAMES=1"
        )
    # transformed_code = rename_kernels(transformed_code)
    transformed_code = remove_triton_function_declaration(transformed_code)
    transformed_code = remove_async_compile(transformed_code)

    launch_params_filename = f"{input_filename}.launch_params"

    # Auto-generate launch_params if they don't exist and auto_generate_params is True
    if not os.path.exists(launch_params_filename) and auto_generate_params:
        print(f"Launch params file {launch_params_filename} not found. Generating...")
        try:
            # Set environment variable and run the input file
            env = os.environ.copy()
            env["TORCHINDUCTOR_DUMP_LAUNCH_PARAMS"] = "1"
```
- **EN**: Key callable entry points in this range include `process_file`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Structured exception handling keeps failure modes explicit instead of letting partial state escape silently. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code.
- **CN**: 这一段的重要可调用入口包括 `process_file`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 结构化异常处理让失败模式保持显式，而不是让部分状态悄悄泄漏出去。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。

### Lines 110-126 / 第 110-126 行
```python
            result = subprocess.run(
                [sys.executable, input_filename],
                env=env,
                capture_output=True,
                text=True,
                cwd=os.path.dirname(input_filename) or ".",
            )

            if result.returncode != 0:
                print(f"Error running {input_filename}:")
                print(f"stdout: {result.stdout}")
                print(f"stderr: {result.stderr}")
                raise RuntimeError(
                    f"Failed to generate launch params. Command failed with return code {result.returncode}"
                )

            print(f"Successfully generated {launch_params_filename}")
```
- **EN**: Key callable entry points in this range include `process_file`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `process_file`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 128-143 / 第 128-143 行
```python
        except Exception as e:
            raise RuntimeError(
                f"Failed to generate launch params by running {input_filename}: {str(e)}"
            ) from e

    if not os.path.exists(launch_params_filename):
        raise RuntimeError(
            f"Missing {launch_params_filename}. Run `TORCHINDUCTOR_DUMP_LAUNCH_PARAMS=1 python {input_filename}` first."
        )

    with open(launch_params_filename) as f:
        launch_params_meta = f.readlines()

    split_params = [i.split("|") for i in launch_params_meta]
    kernel_args_grid = {a.strip(): (b.strip(), c.strip()) for a, b, c in split_params}
    transformed_code = add_launch_params(transformed_code, kernel_args_grid)
```
- **EN**: Key callable entry points in this range include `process_file`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Context-manager usage makes resource, mode, or temporary-state boundaries explicit. Structured exception handling keeps failure modes explicit instead of letting partial state escape silently. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code.
- **CN**: 这一段的重要可调用入口包括 `process_file`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。 结构化异常处理让失败模式保持显式，而不是让部分状态悄悄泄漏出去。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。

### Lines 145-156 / 第 145-156 行
```python
    with open(output_filename, "w") as file:
        file.write(transformed_code)
    print(f"Successfully generated {output_filename}")
    return transformed_code


def get_clean_triton(
    input_path: Path,
    output_path: Path = Path("triton_only_repro.py"),
    auto_generate_params: bool = True,
):
    """Run experiments and output results to file
```
- **EN**: Key callable entry points in this range include `process_file`, `get_clean_triton`. They package a focused unit of behavior behind named helpers or APIs. Context-manager usage makes resource, mode, or temporary-state boundaries explicit. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `process_file`, `get_clean_triton`，它们把聚焦的行为封装成具名辅助函数或 API。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 158-169 / 第 158-169 行
```python
    Args:
        input_path (Path): Path to inductor generated output codede
        output_path (Path): Path to write out the new python file
        auto_generate_params (bool): Whether to automatically generate launch_params if missing
    """
    return process_file(str(input_path), str(output_path), auto_generate_params)


if __name__ == "__main__":
    """Sample usage:
    # Running sweep
    python _get_clean_triton.py output_code.py
```
- **EN**: Key callable entry points in this range include `get_clean_triton`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 这一段的重要可调用入口包括 `get_clean_triton`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 171-188 / 第 171-188 行
```python
    # To disable auto-generation of launch params:
    python _get_clean_triton.py output_code.py --no-auto-generate
    """
    parser = argparse.ArgumentParser(
        description="Clean Inductor generated code to remove Inductor dependencies"
    )

    # Add the arguments
    parser.add_argument(
        "input_path", type=Path, help="Path to inductor generated output code"
    )
    parser.add_argument(
        "--output_path",
        type=Path,
        default=Path("triton_only_repro.py"),
        help="Path to write out the clean triton output",
    )
    parser.add_argument(
```
- **EN**: This chunk continues the implementation of `get_clean_triton`, filling in the details of its control flow or data handling.
- **CN**: 这一段延续了 `get_clean_triton` 的实现，继续补充其控制流或数据处理细节。

### Lines 189-200 / 第 189-200 行
```python
        "--no-auto-generate",
        action="store_true",
        help="Disable automatic generation of launch_params file",
    )

    # Parse the arguments
    args = parser.parse_args()

    # Call the function with parsed arguments
    result = get_clean_triton(
        args.input_path, args.output_path, not args.no_auto_generate
    )
```
- **EN**: This chunk continues the implementation of `get_clean_triton`, filling in the details of its control flow or data handling.
- **CN**: 这一段延续了 `get_clean_triton` 的实现，继续补充其控制流或数据处理细节。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements utility logic that supports broader PyTorch execution, packaging, or developer workflows.
  - CN: 实现支撑更广泛 PyTorch 执行、打包或开发者工作流的辅助逻辑。
- **Graph rewriting**
  - EN: Rewrites module or FX graph structure so later execution follows a transformed pipeline.
  - CN: 改写模块或 FX 图结构，使后续执行遵循转换后的流水线。
- **Configuration management**
  - EN: Centralizes feature flags, environment overrides, and runtime-config wiring.
  - CN: 集中管理特性开关、环境变量覆盖以及运行时配置接线。
- **Serialization safety**
  - EN: Keeps persisted data exchange bounded, predictable, and easier to validate.
  - CN: 让持久化数据交换保持有界、可预测且更易验证。
- **remove_triton_function_declaration**
  - EN: `remove_triton_function_declaration` is a representative function that exposes or coordinates an important action in this module.
  - CN: `remove_triton_function_declaration` 是一个具有代表性的函数，用于暴露或协调此模块中的关键动作。
- **remove_async_compile**
  - EN: `remove_async_compile` is a representative function that exposes or coordinates an important action in this module.
  - CN: `remove_async_compile` 是一个具有代表性的函数，用于暴露或协调此模块中的关键动作。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `argparse`, `os`, `re`, `subprocess`, `sys`, `pathlib:Path`
- **Primary symbols / 核心符号**: `remove_triton_function_declaration`, `remove_async_compile`, `rename_kernels`, `merge_params`, `add_launch_params`, `process_file`, `get_clean_triton`
