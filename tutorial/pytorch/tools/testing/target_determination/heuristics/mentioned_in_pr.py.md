# mentioned_in_pr.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/testing/target_determination/heuristics/mentioned_in_pr.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements target-determination heuristics used to decide which tests should run for a change.
- **Purpose (CN)**: 实现目标判定启发式逻辑，用于决定某次变更应运行哪些测试。
## Line-by-Line Analysis / 逐行分析

### Lines 1-5
```python
from __future__ import annotations

import re
from typing import Any
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__, re, typing.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__、re、typing。

### Lines 6-13
```python
from tools.testing.target_determination.heuristics.interface import (
    HeuristicInterface,
    TestPrioritizations,
)
from tools.testing.target_determination.heuristics.utils import (
    get_git_commit_info,
    get_issue_or_pr_body,
    get_pr_number,
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.testing.target_determination.heuristics.interface, tools.testing.target_determination.heuristics.utils.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.testing.target_determination.heuristics.interface、tools.testing.target_determination.heuristics.utils。

### Lines 14-17
```python
)
from tools.testing.test_run import TestRun
```
- **EN**: This block assembles the Python-side dependencies, importing internal PyTorch modules such as tools.testing.test_run.
- **CN**: 这一段组织 Python 侧依赖，引入了内部 PyTorch 模块，如 tools.testing.test_run。

### Lines 18-23
```python
# This heuristic searches the PR body and commit titles, as well as issues/PRs
# mentioned in the PR body/commit title for test names (search depth of 1) and
# gives the test a rating of 1.  For example, if I mention "test_foo" in the PR
# body, test_foo will be rated 1.  If I mention #123 in the PR body, and #123
# mentions "test_foo", test_foo will be rated 1.
class MentionedInPR(HeuristicInterface):
```
- **EN**: It introduces classes such as MentionedInPR, which package state and behavior for this tooling task. This chunk continues `MentionedInPR` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 它引入了 MentionedInPR 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `MentionedInPR`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 24-27
```python
    def __init__(self, **kwargs: Any) -> None:
        super().__init__(**kwargs)

    def _search_for_linked_issues(self, s: str) -> list[str]:
```
- **EN**: This chunk defines `_search_for_linked_issues`, which implements a focused step inside the testing infrastructure pipeline.
- **CN**: 这一段定义了 `_search_for_linked_issues`，其作用是实现测试基础设施流水线中的一个关键步骤。

### Lines 28-31
```python
        return re.findall(r"#(\d+)", s) + re.findall(r"/pytorch/pytorch/.*/(\d+)", s)

    def get_prediction_confidence(self, tests: list[str]) -> TestPrioritizations:
        try:
```
- **EN**: This chunk defines `get_prediction_confidence`, which implements a focused step inside the testing infrastructure pipeline. Pattern-matching logic extracts structured facts from loosely formatted text inputs. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_prediction_confidence`，其作用是实现测试基础设施流水线中的一个关键步骤。 模式匹配逻辑从格式较松散的文本输入中提取结构化事实。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 32-36
```python
            commit_messages = get_git_commit_info()
        except Exception as e:
            print(f"Can't get commit info due to {e}")
            commit_messages = ""
        try:
```
- **EN**: This chunk continues `get_prediction_confidence` and expands its internal control flow or data movement. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `get_prediction_confidence`，进一步展开其内部控制流或数据流转。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 37-44
```python
            pr_number = get_pr_number()
            if pr_number is not None:
                pr_body = get_issue_or_pr_body(pr_number)
            else:
                pr_body = ""
        except Exception as e:
            print(f"Can't get PR body due to {e}")
            pr_body = ""
```
- **EN**: This chunk continues `get_prediction_confidence` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `get_prediction_confidence`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 45-48
```python

        # Search for linked issues or PRs
        linked_issue_bodies: list[str] = []
        for issue in self._search_for_linked_issues(
```
- **EN**: This chunk continues `get_prediction_confidence` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `get_prediction_confidence`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 49-55
```python
            commit_messages
        ) + self._search_for_linked_issues(pr_body):
            try:
                linked_issue_bodies.append(get_issue_or_pr_body(int(issue)))
            except Exception:
                pass
```
- **EN**: This chunk continues `get_prediction_confidence` and expands its internal control flow or data movement. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段延续了 `get_prediction_confidence`，进一步展开其内部控制流或数据流转。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 56-63
```python
        mentioned = []
        for test in tests:
            if (
                test in commit_messages
                or test in pr_body
                or any(test in body for body in linked_issue_bodies)
            ):
                mentioned.append(test)
```
- **EN**: This chunk continues `get_prediction_confidence` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `get_prediction_confidence`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 64-65
```python

        return TestPrioritizations(tests, {TestRun(test): 1 for test in mentioned})
```
- **EN**: This chunk continues `get_prediction_confidence` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `get_prediction_confidence`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

## Key Concepts / 关键概念

- **Testing infrastructure**
  - EN: This file belongs to the testing infrastructure layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于测试基础设施层，应结合同一子目录中的相邻脚本一起理解。
- **Heuristic decision-making**
  - EN: The code uses lightweight rules to prioritize tests, files, or workflow actions.
  - CN: 代码使用轻量规则来排序测试、文件或工作流动作。
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
- **MentionedInPR**
  - EN: `MentionedInPR` is one of the main local symbols exposed or implemented here.
  - CN: `MentionedInPR` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Internal PyTorch Python modules / 内部 PyTorch Python 模块**: `tools.testing.target_determination.heuristics.interface`, `tools.testing.target_determination.heuristics.utils`, `tools.testing.test_run`
- **Python standard library / Python 标准库**: `__future__`, `re`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `MentionedInPR`
