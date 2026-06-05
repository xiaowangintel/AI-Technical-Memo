# test_run.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/testing/test_run.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides shared testing infrastructure, test-selection helpers, or execution utilities for PyTorch CI.
- **Purpose (CN)**: 提供 PyTorch CI 使用的共享测试基础设施、测试选择辅助逻辑或执行工具。
## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```python
from __future__ import annotations

from copy import copy
from functools import total_ordering
from typing import Any, TYPE_CHECKING


if TYPE_CHECKING:
    from collections.abc import Iterable
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__, copy, functools, and 2 more. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__、copy、functools 等共 5 项。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 11-22
```python

class TestRun:
    """
    TestRun defines the set of tests that should be run together in a single pytest invocation.
    It'll either be a whole test file or a subset of a test file.

    This class assumes that we won't always know the full set of TestClasses in a test file.
    So it's designed to include or exclude explicitly requested TestClasses, while having accepting
    that there will be an ambiguous set of "unknown" test classes that are not expliclty called out.
    Those manifest as tests that haven't been explicitly excluded.
    """
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. It introduces classes such as TestRun, which package state and behavior for this tooling task. This chunk continues `TestRun` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 它引入了 TestRun 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `TestRun`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 23-32
```python
    test_file: str
    _excluded: frozenset[str]  # Tests that should be excluded from this test run
    _included: frozenset[
        str
    ]  # If non-empy, only these tests should be run in this test run

    # NB: Also the class is called TestRun, it's not a test class, so having this field set
    # will allow pytest to ignore this accordingly
    __test__ = False
```
- **EN**: This chunk continues `TestRun` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `TestRun`，进一步展开其内部控制流或数据流转。

### Lines 33-41
```python
    def __init__(
        self,
        name: str,
        excluded: Iterable[str] | None = None,
        included: Iterable[str] | None = None,
    ) -> None:
        if excluded and included:
            raise ValueError("Can't specify both included and excluded")
```
- **EN**: This chunk defines `__init__`, which implements a focused step inside the testing infrastructure pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段定义了 `__init__`，其作用是实现测试基础设施流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 42-54
```python
        ins = set(included or [])
        exs = set(excluded or [])

        if "::" in name:
            if included or excluded:
                raise AssertionError(
                    "Can't specify included or excluded tests when specifying a test class in the file name"
                )
            self.test_file, test_class = name.split("::")
            ins.add(test_class)
        else:
            self.test_file = name
```
- **EN**: This chunk continues `__init__` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow.
- **CN**: 这一段延续了 `__init__`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。

### Lines 55-66
```python
        self._excluded = frozenset(exs)
        self._included = frozenset(ins)

    @staticmethod
    def empty() -> TestRun:
        return TestRun("")

    def is_empty(self) -> bool:
        # Lack of a test_file means that this is an empty run,
        # which means there is nothing to run. It's the zero.
        return not self.test_file
```
- **EN**: This chunk defines `is_empty`, which implements a focused step inside the testing infrastructure pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `is_empty`，其作用是实现测试基础设施流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 67-75
```python
    def is_full_file(self) -> bool:
        return not self._included and not self._excluded

    def included(self) -> frozenset[str]:
        return self._included

    def excluded(self) -> frozenset[str]:
        return self._excluded
```
- **EN**: This chunk defines `excluded`, which implements a focused step inside the testing infrastructure pipeline. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `excluded`，其作用是实现测试基础设施流水线中的一个关键步骤。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 76-84
```python
    def get_pytest_filter(self) -> str:
        if self._included:
            return " or ".join(sorted(self._included))
        elif self._excluded:
            return f"not ({' or '.join(sorted(self._excluded))})"
        else:
            return ""

    def contains(self, test: TestRun) -> bool:
```
- **EN**: This chunk defines `contains`, which implements a focused step inside the testing infrastructure pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `contains`，其作用是实现测试基础设施流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 85-93
```python
        if self.test_file != test.test_file:
            return False

        if self.is_full_file():
            return True  # self contains all tests

        if test.is_full_file():
            return False  # test contains all tests, but self doesn't
```
- **EN**: This chunk continues `contains` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `contains`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 94-105
```python
        # Does self exclude a subset of what test excludes?
        if test._excluded:
            return test._excluded.issubset(self._excluded)

        # Does self include everything test includes?
        if self._included:
            return test._included.issubset(self._included)

        # Getting to here means that test includes and self excludes
        # Does self exclude anything test includes? If not, we're good
        return not self._excluded.intersection(test._included)
```
- **EN**: This chunk continues `contains` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `contains`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 106-118
```python
    def __copy__(self) -> TestRun:
        return TestRun(self.test_file, excluded=self._excluded, included=self._included)

    def __bool__(self) -> bool:
        return not self.is_empty()

    def __repr__(self) -> str:
        r: str = f"RunTest({self.test_file}"
        r += f", included: {self._included}" if self._included else ""
        r += f", excluded: {self._excluded}" if self._excluded else ""
        r += ")"
        return r
```
- **EN**: This chunk defines `__repr__`, which implements a focused step inside the testing infrastructure pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `__repr__`，其作用是实现测试基础设施流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 119-127
```python
    def __str__(self) -> str:
        if self.is_empty():
            return "Empty"

        pytest_filter = self.get_pytest_filter()
        if pytest_filter:
            return self.test_file + ", " + pytest_filter
        return self.test_file
```
- **EN**: This chunk defines `__str__`, which implements a focused step inside the testing infrastructure pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `__str__`，其作用是实现测试基础设施流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 128-136
```python
    def __eq__(self, other: object) -> bool:
        if not isinstance(other, TestRun):
            return False

        ret = self.test_file == other.test_file
        ret = ret and self._included == other._included
        ret = ret and self._excluded == other._excluded
        return ret
```
- **EN**: This chunk defines `__eq__`, which implements a focused step inside the testing infrastructure pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `__eq__`，其作用是实现测试基础设施流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 137-146
```python
    def __hash__(self) -> int:
        return hash((self.test_file, self._included, self._excluded))

    def __or__(self, other: TestRun) -> TestRun:
        """
        To OR/Union test runs means to run all the tests that either of the two runs specify.
        """

        # Is any file empty?
        if self.is_empty():
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `__or__`, which implements a focused step inside the testing infrastructure pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `__or__`，其作用是实现测试基础设施流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 147-156
```python
            return other
        if other.is_empty():
            return copy(self)

        # If not, ensure we have the same file
        if self.test_file != other.test_file:
            raise AssertionError(
                f"Can't exclude {other} from {self} because they're not the same test file"
            )
```
- **EN**: This chunk continues `__or__` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `__or__`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 157-165
```python
        # 4 possible cases:

        # 1. Either file is the full file, so union is everything
        if self.is_full_file() or other.is_full_file():
            # The union is the whole file
            return TestRun(self.test_file)

        # 2. Both files only run what's in _included, so union is the union of the two sets
        if self._included and other._included:
```
- **EN**: This chunk continues `__or__` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `__or__`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 166-175
```python
            return TestRun(
                self.test_file, included=self._included.union(other._included)
            )

        # 3. Both files only exclude what's in _excluded, so union is the intersection of the two sets
        if self._excluded and other._excluded:
            return TestRun(
                self.test_file, excluded=self._excluded.intersection(other._excluded)
            )
```
- **EN**: This chunk continues `__or__` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `__or__`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 176-186
```python
        # 4. One file includes and the other excludes, so we then continue excluding the _excluded set minus
        #    whatever is in the _included set
        included = self._included | other._included
        excluded = self._excluded | other._excluded
        return TestRun(self.test_file, excluded=excluded - included)

    def __sub__(self, other: TestRun) -> TestRun:
        """
        To subtract test runs means to run all the tests in the first run except for what the second run specifies.
        """
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `__sub__`, which implements a focused step inside the testing infrastructure pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `__sub__`，其作用是实现测试基础设施流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 187-196
```python
        # Is any file empty?
        if self.is_empty():
            return TestRun.empty()
        if other.is_empty():
            return copy(self)

        # Are you trying to subtract tests that don't even exist in this test run?
        if self.test_file != other.test_file:
            return copy(self)
```
- **EN**: This chunk continues `__sub__` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `__sub__`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 197-205
```python
        # You're subtracting everything?
        if other.is_full_file():
            return TestRun.empty()

        def return_inclusions_or_empty(inclusions: frozenset[str]) -> TestRun:
            if inclusions:
                return TestRun(self.test_file, included=inclusions)
            return TestRun.empty()
```
- **EN**: This chunk defines `return_inclusions_or_empty`, which implements a focused step inside the testing infrastructure pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `return_inclusions_or_empty`，其作用是实现测试基础设施流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 206-214
```python
        if other._included:
            if self._included:
                return return_inclusions_or_empty(self._included - other._included)
            else:
                return TestRun(
                    self.test_file, excluded=self._excluded | other._included
                )
        else:
            if self._included:
```
- **EN**: This chunk continues `return_inclusions_or_empty` and expands its internal control flow or data movement. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `return_inclusions_or_empty`，进一步展开其内部控制流或数据流转。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 215-224
```python
                return return_inclusions_or_empty(self._included & other._excluded)
            else:
                return return_inclusions_or_empty(other._excluded - self._excluded)

    def __and__(self, other: TestRun) -> TestRun:
        if self.test_file != other.test_file:
            return TestRun.empty()

        return (self | other) - (self - other) - (other - self)
```
- **EN**: This chunk defines `__and__`, which implements a focused step inside the testing infrastructure pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `__and__`，其作用是实现测试基础设施流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 225-234
```python
    def to_json(self) -> dict[str, Any]:
        r: dict[str, Any] = {
            "test_file": self.test_file,
        }
        if self._included:
            r["included"] = list(self._included)
        if self._excluded:
            r["excluded"] = list(self._excluded)
        return r
```
- **EN**: This chunk defines `to_json`, which implements a focused step inside the testing infrastructure pipeline. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `to_json`，其作用是实现测试基础设施流水线中的一个关键步骤。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 235-243
```python
    @staticmethod
    def from_json(json: dict[str, Any]) -> TestRun:
        return TestRun(
            json["test_file"],
            included=json.get("included", []),
            excluded=json.get("excluded", []),
        )
```
- **EN**: This chunk defines `from_json`, which implements a focused step inside the testing infrastructure pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `from_json`，其作用是实现测试基础设施流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 244-258
```python
@total_ordering
class ShardedTest:
    test: TestRun
    shard: int
    num_shards: int
    time: float | None  # In seconds

    def __init__(
        self,
        test: TestRun | str,
        shard: int,
        num_shards: int,
        time: float | None = None,
    ) -> None:
        if isinstance(test, str):
```
- **EN**: It introduces classes such as ShardedTest, which package state and behavior for this tooling task. This chunk defines `__init__`, which implements a focused step inside the testing infrastructure pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 它引入了 ShardedTest 等类，用来封装该工具任务所需的状态与行为。 这一段定义了 `__init__`，其作用是实现测试基础设施流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 259-268
```python
            test = TestRun(test)
        self.test = test
        self.shard = shard
        self.num_shards = num_shards
        self.time = time

    @property
    def name(self) -> str:
        return self.test.test_file
```
- **EN**: This chunk defines `name`, which implements a focused step inside the testing infrastructure pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `name`，其作用是实现测试基础设施流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 269-278
```python
    def __eq__(self, other: object) -> bool:
        if not isinstance(other, ShardedTest):
            return False
        return (
            self.test == other.test
            and self.shard == other.shard
            and self.num_shards == other.num_shards
            and self.time == other.time
        )
```
- **EN**: This chunk defines `__eq__`, which implements a focused step inside the testing infrastructure pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `__eq__`，其作用是实现测试基础设施流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 279-287
```python
    def __repr__(self) -> str:
        ret = f"{self.test} {self.shard}/{self.num_shards}"
        if self.time:
            ret += f" ({self.time}s)"

        return ret

    def __lt__(self, other: object) -> bool:
        if not isinstance(other, ShardedTest):
```
- **EN**: This chunk defines `__lt__`, which implements a focused step inside the testing infrastructure pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `__lt__`，其作用是实现测试基础设施流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 288-297
```python
            raise NotImplementedError

        # This is how the list was implicitly sorted when it was a NamedTuple
        if self.name != other.name:
            return self.name < other.name
        if self.shard != other.shard:
            return self.shard < other.shard
        if self.num_shards != other.num_shards:
            return self.num_shards < other.num_shards
```
- **EN**: This chunk continues `__lt__` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. Assertions and explicit failures stop invalid states from propagating deeper into the workflow. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `__lt__`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 断言与显式失败逻辑可阻止非法状态继续向工作流更深层传播。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 298-307
```python
        # None is the smallest value
        if self.time is None:
            return True
        if other.time is None:
            return False
        return self.time < other.time

    def __str__(self) -> str:
        return f"{self.test} {self.shard}/{self.num_shards}"
```
- **EN**: This chunk defines `__str__`, which implements a focused step inside the testing infrastructure pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `__str__`，其作用是实现测试基础设施流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 308-315
```python
    def get_time(self, default: float = 0) -> float:
        return self.time if self.time is not None else default

    def get_pytest_args(self) -> list[str]:
        filter = self.test.get_pytest_filter()
        if filter:
            return ["-k", self.test.get_pytest_filter()]
        return []
```
- **EN**: This chunk defines `get_pytest_args`, which implements a focused step inside the testing infrastructure pipeline. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `get_pytest_args`，其作用是实现测试基础设施流水线中的一个关键步骤。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

## Key Concepts / 关键概念

- **Testing infrastructure**
  - EN: This file belongs to the testing infrastructure layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于测试基础设施层，应结合同一子目录中的相邻脚本一起理解。
- **Pytest integration**
  - EN: The file plugs into pytest collection, execution, or reporting behavior.
  - CN: 该文件接入 pytest 的收集、执行或报告行为。
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
- **TestRun**
  - EN: `TestRun` is one of the main local symbols exposed or implemented here.
  - CN: `TestRun` 是此处暴露或实现的主要局部符号之一。
- **ShardedTest**
  - EN: `ShardedTest` is one of the main local symbols exposed or implemented here.
  - CN: `ShardedTest` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `__future__`, `copy`, `functools`, `typing`, `collections.abc`
- **Primary symbols in this file / 本文件核心符号**: `TestRun`, `ShardedTest`
