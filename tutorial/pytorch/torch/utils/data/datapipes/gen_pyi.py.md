# gen_pyi.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/utils/data/datapipes/gen_pyi.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements data-loading infrastructure, datapipes, samplers, worker coordination, and dataset traversal utilities. This specific file centers on `gen_pyi.py`. Dataset iteration, batching, and worker orchestration are central concerns here.
- **Purpose (CN) / 用途（中文）**: 实现数据加载基础设施、datapipes、sampler、worker 协调以及数据集遍历工具。 该文件具体围绕 `gen_pyi.py` 展开。 数据集迭代、批处理以及 worker 协调是这里的核心关注点。
## Line-by-Line Analysis / 逐行分析

### Lines 1-22 / 第 1-22 行
```python
# mypy: allow-untyped-defs
import os
from collections import defaultdict
from pathlib import Path
from typing import Any
from typing_extensions import deprecated


try:
    from torchgen.api.python import format_function_signature
    from torchgen.utils import FileManager
except ImportError:
    import sys

    REPO_ROOT = Path(__file__).absolute().parents[4]
    sys.path.insert(0, str(REPO_ROOT))

    from torchgen.api.python import format_function_signature
    from torchgen.utils import FileManager

    if len(sys.path) > 0 and sys.path[0] == str(REPO_ROOT):
        del sys.path[0]
```
- **EN**: Named constants such as `REPO_ROOT` centralize shared configuration or sentinel values. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Structured exception handling keeps failure modes explicit instead of letting partial state escape silently. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: `REPO_ROOT` 等具名常量把共享配置或哨兵值集中定义在一起。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 结构化异常处理让失败模式保持显式，而不是让部分状态悄悄泄漏出去。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 25-43 / 第 25-43 行
```python
__all__: list[str] = []  # not intended to expose any symbols


def __dir__() -> list[str]:
    return []  # appease public API test


@deprecated(
    "`torch.utils.data.datapipes.gen_pyi.materialize_lines` is deprecated and will be removed in the future.",
    category=FutureWarning,
)
def materialize_lines(lines: list[str], indentation: int) -> str:
    output = ""
    new_line_with_indent = "\n" + " " * indentation
    for i, line in enumerate(lines):
        if i != 0:
            output += new_line_with_indent
        output += line.replace("\n", new_line_with_indent)
    return output
```
- **EN**: The `__all__` assignment defines the intended public surface that other modules are expected to import or re-export. Key callable entry points in this range include `__dir__`, `materialize_lines`. They package a focused unit of behavior behind named helpers or APIs. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: `__all__` 赋值定义了预期的公开接口，供其他模块导入或再次导出。 这一段的重要可调用入口包括 `__dir__`, `materialize_lines`，它们把聚焦的行为封装成具名辅助函数或 API。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 46-66 / 第 46-66 行
```python
@deprecated(
    "`torch.utils.data.datapipes.gen_pyi.gen_from_template` is deprecated and will be removed in the future.",
    category=FutureWarning,
)
def gen_from_template(
    dir: str,
    template_name: str,
    output_name: str,
    replacements: list[tuple[str, Any, int]],
) -> None:
    template_path = os.path.join(dir, template_name)
    output_path = os.path.join(dir, output_name)

    with open(template_path, encoding="utf-8") as f:
        content = f.read()
    for placeholder, lines, indentation in replacements:
        with open(output_path, "w", encoding="utf-8") as f:
            content = content.replace(
                placeholder, materialize_lines(lines, indentation)
            )
            f.write(content)
```
- **EN**: Key callable entry points in this range include `gen_from_template`. They package a focused unit of behavior behind named helpers or APIs. Decorators in this block alter call semantics, registration behavior, or metadata attached to the following definitions. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Context-manager usage makes resource, mode, or temporary-state boundaries explicit. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段的重要可调用入口包括 `gen_from_template`，它们把聚焦的行为封装成具名辅助函数或 API。 这一段中的装饰器会改变后续定义的调用语义、注册行为或附加元数据。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 69-83 / 第 69-83 行
```python
def find_file_paths(dir_paths: list[str], files_to_exclude: set[str]) -> set[str]:
    """
    When given a path to a directory, returns the paths to the relevant files within it.

    This function does NOT recursive traverse to subdirectories.
    """
    paths: set[str] = set()
    for dir_path in dir_paths:
        all_files = os.listdir(dir_path)
        python_files = {fname for fname in all_files if ".py" == fname[-3:]}
        filter_files = {
            fname for fname in python_files if fname not in files_to_exclude
        }
        paths.update({os.path.join(dir_path, fname) for fname in filter_files})
    return paths
```
- **EN**: Key callable entry points in this range include `find_file_paths`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段的重要可调用入口包括 `find_file_paths`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 86-105 / 第 86-105 行
```python
def extract_method_name(line: str) -> str:
    """Extract method name from decorator in the form of "@functional_datapipe({method_name})"."""
    if '("' in line:
        start_token, end_token = '("', '")'
    elif "('" in line:
        start_token, end_token = "('", "')"
    else:
        raise RuntimeError(
            f"Unable to find appropriate method name within line:\n{line}"
        )
    start, end = line.find(start_token) + len(start_token), line.find(end_token)
    return line[start:end]


def extract_class_name(line: str) -> str:
    """Extract class name from class definition in the form of "class {CLASS_NAME}({Type}):"."""
    start_token = "class "
    end_token = "("
    start, end = line.find(start_token) + len(start_token), line.find(end_token)
    return line[start:end]
```
- **EN**: Key callable entry points in this range include `extract_method_name`, `extract_class_name`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段的重要可调用入口包括 `extract_method_name`, `extract_class_name`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 108-131 / 第 108-131 行
```python
def parse_datapipe_file(
    file_path: str,
) -> tuple[dict[str, list[str]], dict[str, str], set[str], dict[str, list[str]]]:
    """Given a path to file, parses the file and returns a dictionary of method names to function signatures."""
    method_to_signature, method_to_class_name, special_output_type = {}, {}, set()
    doc_string_dict = defaultdict(list)
    with open(file_path, encoding="utf-8") as f:
        open_paren_count = 0
        method_name, class_name, signature = "", "", ""
        skip = False
        for line in f:
            if line.count('"""') % 2 == 1:
                skip = not skip
            if skip or '"""' in line:  # Saving docstrings
                doc_string_dict[method_name].append(line)
                continue
            if "@functional_datapipe" in line:
                method_name = extract_method_name(line)
                doc_string_dict[method_name] = []
                continue
            if method_name and "class " in line:
                class_name = extract_class_name(line)
                continue
            if method_name and ("def __init__(" in line or "def __new__(" in line):
```
- **EN**: Key callable entry points in this range include `parse_datapipe_file`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Context-manager usage makes resource, mode, or temporary-state boundaries explicit. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段的重要可调用入口包括 `parse_datapipe_file`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 上下文管理器的使用让资源、模式或临时状态的边界更加明确。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 132-155 / 第 132-155 行
```python
                if "def __new__(" in line:
                    special_output_type.add(method_name)
                open_paren_count += 1
                start = line.find("(") + len("(")
                line = line[start:]
            if open_paren_count > 0:
                open_paren_count += line.count("(")
                open_paren_count -= line.count(")")
                if open_paren_count == 0:
                    end = line.rfind(")")
                    signature += line[:end]
                    method_to_signature[method_name] = process_signature(signature)
                    method_to_class_name[method_name] = class_name
                    method_name, class_name, signature = "", "", ""
                elif open_paren_count < 0:
                    raise RuntimeError(
                        "open parenthesis count < 0. This shouldn't be possible."
                    )
                else:
                    signature += line.strip()
    return (
        method_to_signature,
        method_to_class_name,
        special_output_type,
```
- **EN**: Key callable entry points in this range include `parse_datapipe_file`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段的重要可调用入口包括 `parse_datapipe_file`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 156-179 / 第 156-179 行
```python
        doc_string_dict,
    )


def parse_datapipe_files(
    file_paths: set[str],
) -> tuple[dict[str, list[str]], dict[str, str], set[str], dict[str, list[str]]]:
    methods_and_signatures = {}
    methods_and_class_names = {}
    methods_with_special_output_types = set()
    methods_and_doc_strings = {}
    for path in file_paths:
        (
            method_to_signature,
            method_to_class_name,
            methods_needing_special_output_types,
            doc_string_dict,
        ) = parse_datapipe_file(path)
        methods_and_signatures.update(method_to_signature)
        methods_and_class_names.update(method_to_class_name)
        methods_with_special_output_types.update(methods_needing_special_output_types)
        methods_and_doc_strings.update(doc_string_dict)
    return (
        methods_and_signatures,
```
- **EN**: Key callable entry points in this range include `parse_datapipe_file`, `parse_datapipe_files`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段的重要可调用入口包括 `parse_datapipe_file`, `parse_datapipe_files`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 180-202 / 第 180-202 行
```python
        methods_and_class_names,
        methods_with_special_output_types,
        methods_and_doc_strings,
    )


def split_outside_bracket(line: str, delimiter: str = ",") -> list[str]:
    """Given a line of text, split it on comma unless the comma is within a bracket '[]'."""
    bracket_count = 0
    curr_token = ""
    res = []
    for char in line:
        if char == "[":
            bracket_count += 1
        elif char == "]":
            bracket_count -= 1
        elif char == delimiter and bracket_count == 0:
            res.append(curr_token)
            curr_token = ""
            continue
        curr_token += char
    res.append(curr_token)
    return res
```
- **EN**: Key callable entry points in this range include `parse_datapipe_files`, `split_outside_bracket`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段的重要可调用入口包括 `parse_datapipe_files`, `split_outside_bracket`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 205-224 / 第 205-224 行
```python
def process_signature(line: str) -> list[str]:
    """
    Clean up a given raw function signature.

    This includes removing the self-referential datapipe argument, default
    arguments of input functions, newlines, and spaces.
    """
    tokens: list[str] = split_outside_bracket(line)
    for i, token in enumerate(tokens):
        tokens[i] = token.strip(" ")
        if token == "cls":
            tokens[i] = "self"
        elif i > 0 and ("self" == tokens[i - 1]) and (tokens[i][0] != "*"):
            # Remove the datapipe after 'self' or 'cls' unless it has '*'
            tokens[i] = ""
        elif "Callable =" in token:  # Remove default argument if it is a function
            head = token.rpartition("=")[0]
            tokens[i] = head.strip(" ") + " = ..."
    tokens = [t for t in tokens if t != ""]
    return tokens
```
- **EN**: Key callable entry points in this range include `process_signature`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段的重要可调用入口包括 `process_signature`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 227-250 / 第 227-250 行
```python
def get_method_definitions(
    file_path: str | list[str],
    files_to_exclude: set[str],
    deprecated_files: set[str],
    default_output_type: str,
    method_to_special_output_type: dict[str, str],
    root: str = "",
) -> list[str]:
    """
    #.pyi generation for functional DataPipes Process.

    # 1. Find files that we want to process (exclude the ones who don't)
    # 2. Parse method name and signature
    # 3. Remove first argument after self (unless it is "*datapipes"), default args, and spaces
    """
    if root == "":
        root = str(Path(__file__).parent.resolve())
    file_path = [file_path] if isinstance(file_path, str) else file_path
    file_path = [os.path.join(root, path) for path in file_path]
    file_paths = find_file_paths(
        file_path, files_to_exclude=files_to_exclude.union(deprecated_files)
    )
    (
        methods_and_signatures,
```
- **EN**: Key callable entry points in this range include `get_method_definitions`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段的重要可调用入口包括 `get_method_definitions`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 251-274 / 第 251-274 行
```python
        methods_and_class_names,
        methods_w_special_output_types,
        methods_and_doc_strings,
    ) = parse_datapipe_files(file_paths)

    for fn_name in method_to_special_output_type:
        if fn_name not in methods_w_special_output_types:
            methods_w_special_output_types.add(fn_name)

    method_definitions = []
    for method_name, arguments in methods_and_signatures.items():
        class_name = methods_and_class_names[method_name]
        if method_name in methods_w_special_output_types:
            output_type = method_to_special_output_type[method_name]
        else:
            output_type = default_output_type
        doc_string = "".join(methods_and_doc_strings[method_name])
        if doc_string == "":
            doc_string = " ..."
        else:
            doc_string = "\n" + doc_string
        definition = format_function_signature(method_name, arguments, output_type)
        method_definitions.append(
            f"# Functional form of '{class_name}'\n"
```
- **EN**: Key callable entry points in this range include `get_method_definitions`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段的重要可调用入口包括 `get_method_definitions`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 275-297 / 第 275-297 行
```python
            + definition.removesuffix("...").rstrip()  # remove "..."
            + doc_string,
        )
    method_definitions.sort(
        key=lambda s: s.split("\n")[1]
    )  # sorting based on method_name

    return method_definitions


# Defined outside of main() so they can be imported by TorchData
iterDP_file_path: str = "iter"
iterDP_files_to_exclude: set[str] = {"__init__.py", "utils.py"}
iterDP_deprecated_files: set[str] = set()
iterDP_method_to_special_output_type: dict[str, str] = {
    "demux": "list[IterDataPipe]",
    "fork": "list[IterDataPipe]",
}

mapDP_file_path: str = "map"
mapDP_files_to_exclude: set[str] = {"__init__.py", "utils.py"}
mapDP_deprecated_files: set[str] = set()
mapDP_method_to_special_output_type: dict[str, str] = {"shuffle": "IterDataPipe"}
```
- **EN**: Key callable entry points in this range include `get_method_definitions`. They package a focused unit of behavior behind named helpers or APIs. The tail of the block returns computed state or forwards the next value to the caller. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段的重要可调用入口包括 `get_method_definitions`，它们把聚焦的行为封装成具名辅助函数或 API。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 300-321 / 第 300-321 行
```python
def main() -> None:
    """
    # Inject file into template datapipe.pyi.in.

    TODO: The current implementation of this script only generates interfaces for built-in methods. To generate
          interface for user-defined DataPipes, consider changing `IterDataPipe.register_datapipe_as_function`.
    """
    iter_method_definitions = get_method_definitions(
        iterDP_file_path,
        iterDP_files_to_exclude,
        iterDP_deprecated_files,
        "IterDataPipe",
        iterDP_method_to_special_output_type,
    )

    map_method_definitions = get_method_definitions(
        mapDP_file_path,
        mapDP_files_to_exclude,
        mapDP_deprecated_files,
        "MapDataPipe",
        mapDP_method_to_special_output_type,
    )
```
- **EN**: Key callable entry points in this range include `main`. They package a focused unit of behavior behind named helpers or APIs. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段的重要可调用入口包括 `main`，它们把聚焦的行为封装成具名辅助函数或 API。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

### Lines 323-336 / 第 323-336 行
```python
    path = Path(__file__).absolute().parent
    fm = FileManager(install_dir=path, template_dir=path, dry_run=False)
    fm.write_with_template(
        "datapipe.pyi",
        "datapipe.pyi.in",
        lambda: {
            "IterDataPipeMethods": iter_method_definitions,
            "MapDataPipeMethods": map_method_definitions,
        },
    )


if __name__ == "__main__":
    main()
```
- **EN**: Key callable entry points in this range include `main`. They package a focused unit of behavior behind named helpers or APIs. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. This code shapes lazy data-pipeline composition rather than eagerly materializing intermediate results.
- **CN**: 这一段的重要可调用入口包括 `main`，它们把聚焦的行为封装成具名辅助函数或 API。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 这段代码塑造的是惰性数据流水线组合，而不是急切地产生中间结果。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements data-loading infrastructure, datapipes, samplers, worker coordination, and dataset traversal utilities.
  - CN: 实现数据加载基础设施、datapipes、sampler、worker 协调以及数据集遍历工具。
- **Graph rewriting**
  - EN: Rewrites module or FX graph structure so later execution follows a transformed pipeline.
  - CN: 改写模块或 FX 图结构，使后续执行遵循转换后的流水线。
- **DataPipes**
  - EN: Builds composable input pipelines that lazily transform or route dataset elements.
  - CN: 构建可组合的输入流水线，以惰性方式变换或路由数据集元素。
- **Serialization safety**
  - EN: Keeps persisted data exchange bounded, predictable, and easier to validate.
  - CN: 让持久化数据交换保持有界、可预测且更易验证。
- **__dir__**
  - EN: `__dir__` is a representative function that exposes or coordinates an important action in this module.
  - CN: `__dir__` 是一个具有代表性的函数，用于暴露或协调此模块中的关键动作。
- **materialize_lines**
  - EN: `materialize_lines` is a representative function that exposes or coordinates an important action in this module.
  - CN: `materialize_lines` 是一个具有代表性的函数，用于暴露或协调此模块中的关键动作。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `os`, `collections:defaultdict`, `pathlib:Path`, `typing:Any`
- **Third-party packages / 第三方包**: `typing_extensions:deprecated`
- **Primary symbols / 核心符号**: `__dir__`, `materialize_lines`, `gen_from_template`, `find_file_paths`, `extract_method_name`, `extract_class_name`, `parse_datapipe_file`, `parse_datapipe_files`, `split_outside_bracket`, `process_signature`, `get_method_definitions`, `main`
