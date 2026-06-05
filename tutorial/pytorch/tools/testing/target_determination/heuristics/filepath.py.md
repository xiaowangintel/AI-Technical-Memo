# filepath.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/testing/target_determination/heuristics/filepath.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements target-determination heuristics used to decide which tests should run for a change.
- **Purpose (CN)**: 实现目标判定启发式逻辑，用于决定某次变更应运行哪些测试。
## Line-by-Line Analysis / 逐行分析

### Lines 1-8
```python
from __future__ import annotations

from collections import defaultdict
from functools import lru_cache
from pathlib import Path
from typing import Any, TYPE_CHECKING
from warnings import warn
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__, collections, functools, and 3 more. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__、collections、functools 等共 6 项。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。

### Lines 9-19
```python
from tools.testing.target_determination.heuristics.interface import (
    HeuristicInterface,
    TestPrioritizations,
)
from tools.testing.target_determination.heuristics.utils import (
    is_docs_only_change,
    normalize_ratings,
    query_changed_files,
)
from tools.testing.test_run import TestRun
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.testing.target_determination.heuristics.interface, tools.testing.target_determination.heuristics.utils, tools.testing.test_run.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.testing.target_determination.heuristics.interface、tools.testing.target_determination.heuristics.utils、tools.testing.test_run。

### Lines 20-26
```python

if TYPE_CHECKING:
    from collections.abc import Callable


REPO_ROOT = Path(__file__).parents[3]
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as collections.abc. Configuration constants such as REPO_ROOT centralize defaults so later functions share the same policy knobs. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 collections.abc。 REPO_ROOT 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 27-37
```python
keyword_synonyms: dict[str, list[str]] = {
    "amp": ["mixed_precision"],
    "quant": ["quantized", "quantization", "quantize"],
    "decomp": ["decomposition", "decompositions"],
    "numpy": ["torch_np", "numpy_tests"],
    "ops": ["opinfo"],
    "hop": ["higher_order_op"],
    "aot": ["flex_attention", "autograd"],
    "inductor": ["dynamo", "export"],  # not actually synonyms but they interact a lot
}
```
- **EN**: This chunk contributes a small but necessary piece of the testing infrastructure implementation, wiring local data and helper logic together.
- **CN**: 这一段补上了测试基础设施实现中的一小块但必要的基础逻辑，用于衔接局部数据与辅助实现。

### Lines 38-43
```python

custom_matchers: dict[str, Callable[[str], bool]] = {
    "nn": lambda x: "nn" in x.replace("onnx", "_"),
    "c10": lambda x: "c10" in x.replace("c10d", "_"),
}
```
- **EN**: This chunk contributes a small but necessary piece of the testing infrastructure implementation, wiring local data and helper logic together.
- **CN**: 这一段补上了测试基础设施实现中的一小块但必要的基础逻辑，用于衔接局部数据与辅助实现。

### Lines 44-55
```python

def is_valid_keyword(keyword: str) -> bool:
    not_keyword = [
        "torch",
        "test",
        "tests",
        "util",
        "utils",
        "func",
        "src",
        "c",
        "ns",
```
- **EN**: This chunk defines `is_valid_keyword`, which implements a focused step inside the testing infrastructure pipeline.
- **CN**: 这一段定义了 `is_valid_keyword`，其作用是实现测试基础设施流水线中的一个关键步骤。

### Lines 56-61
```python
        "tools",
        "internal",
    ]
    return keyword == "nn" or (keyword not in not_keyword and len(keyword) > 2)
```
- **EN**: This chunk continues `is_valid_keyword` and expands its internal control flow or data movement. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `is_valid_keyword`，进一步展开其内部控制流或数据流转。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 62-68
```python
@lru_cache(maxsize=1)
def get_keywords(file: str) -> list[str]:
    keywords = []
    for folder in Path(file).parts[:-1]:
        folder = sanitize_name(folder)
        keywords.append(folder)
```
- **EN**: This chunk defines `get_keywords`, which implements a focused step inside the testing infrastructure pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段定义了 `get_keywords`，其作用是实现测试基础设施流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 69-74
```python
    file_name = Path(file).stem.split("_")
    keywords.extend([sanitize_name(x) for x in file_name])
    return [kw for kw in keywords if is_valid_keyword(kw)]


def sanitize_name(folder_name: str) -> str:
```
- **EN**: This chunk defines `sanitize_name`, which implements a focused step inside the testing infrastructure pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `sanitize_name`，其作用是实现测试基础设施流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 75-80
```python
    folder_name = folder_name.removeprefix("_")

    for syn_rep, syns in keyword_synonyms.items():
        if folder_name in syns or folder_name == syn_rep:
            return syn_rep
```
- **EN**: This chunk continues `sanitize_name` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `sanitize_name`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 81-92
```python
    return folder_name


def file_matches_keyword(file: str, keyword: str) -> bool:
    keywords = get_keywords(file)
    return (
        keyword in keywords
        or any(
            syn in keywords or syn in file for syn in keyword_synonyms.get(keyword, [])
        )
        or custom_matchers.get(keyword, lambda x: keyword in x)(file)  # type: ignore[no-untyped-call]
    )
```
- **EN**: This chunk defines `file_matches_keyword`, which implements a focused step inside the testing infrastructure pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `file_matches_keyword`，其作用是实现测试基础设施流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 93-99
```python


def get_freq_dict(tests: list[str], changed_files: list[str]) -> dict[str, int]:
    keyword_frequency: dict[str, int] = defaultdict(int)
    for cf in changed_files:
        keywords = get_keywords(cf)
        for keyword in keywords:
```
- **EN**: This chunk defines `get_freq_dict`, which implements a focused step inside the testing infrastructure pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段定义了 `get_freq_dict`，其作用是实现测试基础设施流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 100-105
```python
            keyword_frequency[keyword] += 1

    test_ratings: dict[str, int] = defaultdict(int)

    for test in tests:
        for keyword, frequency in keyword_frequency.items():
```
- **EN**: This chunk continues `get_freq_dict` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `get_freq_dict`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 106-111
```python
            if file_matches_keyword(test, keyword):
                test_ratings[test] += frequency
    return test_ratings


class Filepath(HeuristicInterface):
```
- **EN**: It introduces classes such as Filepath, which package state and behavior for this tooling task. This chunk continues `Filepath` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 它引入了 Filepath 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `Filepath`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 112-117
```python
    # Heuristic based on folders in the file path.  Takes each folder of each
    # changed file and attempts to find matches based on those folders
    def __init__(self, **kwargs: dict[str, Any]) -> None:
        super().__init__(**kwargs)

    def get_prediction_confidence(self, tests: list[str]) -> TestPrioritizations:
```
- **EN**: This chunk defines `get_prediction_confidence`, which implements a focused step inside the testing infrastructure pipeline.
- **CN**: 这一段定义了 `get_prediction_confidence`，其作用是实现测试基础设施流水线中的一个关键步骤。

### Lines 118-123
```python
        try:
            changed_files = query_changed_files()
        except Exception as e:
            warn(f"Can't query changed test files due to {e}")
            changed_files = []
```
- **EN**: This chunk continues `get_prediction_confidence` and expands its internal control flow or data movement. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `get_prediction_confidence`，进一步展开其内部控制流或数据流转。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 124-130
```python
        # If only documentation files (.rst, .md) were modified, skip all tests
        if is_docs_only_change(changed_files):
            print("Only documentation files changed, skipping all tests")
            # Return negative scores to indicate all tests should be skipped
            skip_scores = {TestRun(test): -1.0 for test in tests}
            return TestPrioritizations(tests, skip_scores)
```
- **EN**: This chunk continues `get_prediction_confidence` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `get_prediction_confidence`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 131-138
```python
        test_ratings = get_freq_dict(tests, changed_files)
        test_ratings = {
            TestRun(k): float(v) for (k, v) in test_ratings.items() if k in tests
        }
        return TestPrioritizations(
            tests, normalize_ratings(test_ratings, 0.25, min_value=0.125)
        )
```
- **EN**: This chunk continues `get_prediction_confidence` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `get_prediction_confidence`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 139-144
```python

if __name__ == "__main__":
    # Quick thing so you can call the heuristic from the command line with a sha
    import os
    import sys
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as os, sys. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 os、sys。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 145-152
```python
    from tools.testing.discover_tests import TESTS

    git_diff = f"git diff --name-only {sys.argv[1]} {sys.argv[1]}^"
    changed_files = os.popen(git_diff).read().split("\n")
    freq_dict = get_freq_dict(
        TESTS, [x for x in changed_files if x != "" and not x.startswith("test")]
    )
    for k, v in sorted(freq_dict.items(), key=lambda x: x[1], reverse=False):
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.testing.discover_tests. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.testing.discover_tests。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 153-154
```python
        print(k, v)
    print(changed_files)
```
- **EN**: This chunk continues `get_prediction_confidence` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `get_prediction_confidence`，进一步展开其内部控制流或数据流转。

## Key Concepts / 关键概念

- **Testing infrastructure**
  - EN: This file belongs to the testing infrastructure layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于测试基础设施层，应结合同一子目录中的相邻脚本一起理解。
- **Autograd generation**
  - EN: The file participates in turning operator metadata into backward formulas, wrappers, or saved-state logic.
  - CN: 该文件参与把算子元数据转换为反向公式、包装层或保存状态逻辑。
- **Heuristic decision-making**
  - EN: The code uses lightweight rules to prioritize tests, files, or workflow actions.
  - CN: 代码使用轻量规则来排序测试、文件或工作流动作。
- **Filesystem coordination**
  - EN: The code spends effort on path normalization, file discovery, or directory layout management.
  - CN: 代码重点处理路径规范化、文件发现或目录布局管理。
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
- **REPO_ROOT**
  - EN: `REPO_ROOT` is one of the main local symbols exposed or implemented here.
  - CN: `REPO_ROOT` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `tools.testing.target_determination.heuristics.interface`, `tools.testing.target_determination.heuristics.utils`, `tools.testing.test_run`, `tools.testing.discover_tests`
- **Python standard library / Python 标准库**: `__future__`, `collections`, `functools`, `pathlib`, `typing`, `warnings`, `collections.abc`, `os`, `sys`
- **Primary symbols in this file / 本文件核心符号**: `REPO_ROOT`, `is_valid_keyword`, `get_keywords`, `sanitize_name`, `file_matches_keyword`, `get_freq_dict`, `Filepath`
