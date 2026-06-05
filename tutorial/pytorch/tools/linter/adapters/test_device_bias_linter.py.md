# test_device_bias_linter.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/linter/adapters/test_device_bias_linter.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements lint adapters and validation helpers that enforce repository-wide code-quality rules.
- **Purpose (CN)**: 实现 lint 适配器与校验辅助逻辑，用于执行仓库范围的代码质量规则。
## Line-by-Line Analysis / 逐行分析

### Lines 1-10
```python
#!/usr/bin/env python3
"""
This lint verifies that every Python test file (file that matches test_*.py or
*_test.py in the test folder) has a cuda hard code in `requires_gpu()` or
`requires_triton()` decorated function or `if HAS_GPU:` guarded main section,
to ensure that the test not fail on other GPU devices.
"""

from __future__ import annotations
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as __future__. This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 __future__。 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 11-20
```python
import argparse
import ast
import json
import multiprocessing as mp
from enum import Enum
from typing import NamedTuple


LINTER_CODE = "TEST_DEVICE_BIAS"
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as argparse, ast, json, and 3 more. Configuration constants such as LINTER_CODE centralize defaults so later functions share the same policy knobs. Argument-parsing logic turns command-line inputs into structured parameters for the tool core. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 argparse、ast、json 等共 6 项。 LINTER_CODE 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。

### Lines 21-29
```python

class LintSeverity(str, Enum):
    ERROR = "error"
    WARNING = "warning"
    ADVICE = "advice"
    DISABLED = "disabled"


class LintMessage(NamedTuple):
```
- **EN**: It introduces classes such as LintSeverity, LintMessage, which package state and behavior for this tooling task. This chunk continues `LintMessage` and expands its internal control flow or data movement. Configuration constants such as ERROR, WARNING, ADVICE, and 1 more centralize defaults so later functions share the same policy knobs.
- **CN**: 它引入了 LintSeverity、LintMessage 等类，用来封装该工具任务所需的状态与行为。 这一段延续了 `LintMessage`，进一步展开其内部控制流或数据流转。 ERROR、WARNING、ADVICE 等共 4 项 等配置常量集中定义默认值，使后续函数共享同一套策略开关。

### Lines 30-39
```python
    path: str | None
    line: int | None
    char: int | None
    code: str
    severity: LintSeverity
    name: str
    original: str | None
    replacement: str | None
    description: str | None
```
- **EN**: This chunk continues `LintMessage` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `LintMessage`，进一步展开其内部控制流或数据流转。

### Lines 40-57
```python

DEVICE_BIAS = ["cuda", "xpu", "mps"]
GPU_RELATED_DECORATORS = {"requires_gpu", "requires_triton"}


def is_main_has_gpu(tree: ast.AST) -> bool:
    def _contains_has_gpu(node: ast.AST) -> bool:
        if isinstance(node, ast.Name) and node.id in ["HAS_GPU", "RUN_GPU"]:
            return True
        elif isinstance(node, ast.BoolOp):
            return any(_contains_has_gpu(value) for value in node.values)
        elif isinstance(node, ast.UnaryOp):
            return _contains_has_gpu(node.operand)
        elif isinstance(node, ast.Compare):
            return _contains_has_gpu(node.left) or any(
                _contains_has_gpu(comp) for comp in node.comparators
            )
        elif isinstance(node, (ast.IfExp, ast.Call)):
```
- **EN**: This chunk defines `_contains_has_gpu`, which implements a focused step inside the lint tooling pipeline. Configuration constants such as DEVICE_BIAS, GPU_RELATED_DECORATORS centralize defaults so later functions share the same policy knobs. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `_contains_has_gpu`，其作用是实现Lint 工具链流水线中的一个关键步骤。 DEVICE_BIAS、GPU_RELATED_DECORATORS 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 58-69
```python
            return False
        return False

    for node in ast.walk(tree):
        # Detect if __name__ == "__main__":
        if isinstance(node, ast.If):
            if (
                isinstance(node.test, ast.Compare)
                and isinstance(node.test.left, ast.Name)
                and node.test.left.id == "__name__"
            ):
                if any(
```
- **EN**: This chunk continues `_contains_has_gpu` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `_contains_has_gpu`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 70-79
```python
                    isinstance(comp, ast.Constant) and comp.value == "__main__"
                    for comp in node.test.comparators
                ):
                    for inner_node in node.body:
                        if isinstance(inner_node, ast.If) and _contains_has_gpu(
                            inner_node.test
                        ):
                            return True
    return False
```
- **EN**: This chunk continues `_contains_has_gpu` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `_contains_has_gpu`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 80-88
```python

class DeviceBiasVisitor(ast.NodeVisitor):
    def __init__(self, filename: str, is_gpu_test_suite: bool) -> None:
        self.filename = filename
        self.lint_messages: list[LintMessage] = []
        self.is_gpu_test_suite = is_gpu_test_suite

    def _has_proper_decorator(self, node: ast.FunctionDef) -> bool:
        for d in node.decorator_list:
```
- **EN**: It introduces classes such as DeviceBiasVisitor, which package state and behavior for this tooling task. This chunk defines `_has_proper_decorator`, which implements a focused step inside the lint tooling pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 它引入了 DeviceBiasVisitor 等类，用来封装该工具任务所需的状态与行为。 这一段定义了 `_has_proper_decorator`，其作用是实现Lint 工具链流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 89-98
```python
            if isinstance(d, ast.Name) and d.id in GPU_RELATED_DECORATORS:
                return True
            if (
                isinstance(d, ast.Call)
                and isinstance(d.func, ast.Name)
                and d.func.id in GPU_RELATED_DECORATORS
            ):
                return True
        return False
```
- **EN**: This chunk continues `_has_proper_decorator` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `_has_proper_decorator`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 99-107
```python
    # check device = "cuda" or torch.device("cuda")
    def _check_keyword_device(self, subnode: ast.keyword, msg_prefix: str) -> None:
        if subnode.arg != "device":
            return
        val = subnode.value
        if isinstance(val, ast.Constant) and any(
            # pyrefly: ignore [not-iterable, unsupported-operation]
            bias in val.value
            for bias in DEVICE_BIAS
```
- **EN**: This chunk defines `_check_keyword_device`, which validates invariants and reports policy violations early. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `_check_keyword_device`，其作用是校验不变量，并尽早报告策略违规。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 108-125
```python
        ):
            self.record(
                subnode,
                f"{msg_prefix} device='{val.value}', suggest to use device=GPU_TYPE",
            )
        elif isinstance(val, ast.Call):
            if (
                isinstance(val.func, ast.Attribute)
                and val.func.attr == "device"
                and len(val.args) > 0
                and isinstance(val.args[0], ast.Constant)
                # pyrefly: ignore [not-iterable, unsupported-operation]
                and any(bias in val.args[0].value for bias in DEVICE_BIAS)
            ):
                self.record(
                    val,
                    f"{msg_prefix} torch.device('{val.args[0].value}'), suggest to use torch.device(GPU_TYPE)",
                )
```
- **EN**: This chunk continues `_check_keyword_device` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `_check_keyword_device`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 126-140
```python

    # check .cuda() or .to("cuda")
    def _check_device_methods(self, subnode: ast.Call, msg_prefix: str) -> None:
        func = subnode.func
        if not isinstance(func, ast.Attribute):
            return
        method_name = func.attr
        if method_name in DEVICE_BIAS:
            self.record(
                subnode,
                f"{msg_prefix} .{method_name}(), suggest to use .to(GPU_TYPE)",
            )
        elif method_name == "to" and subnode.args:
            arg = subnode.args[0]
            if isinstance(arg, ast.Constant) and any(
```
- **EN**: This chunk defines `_check_device_methods`, which validates invariants and reports policy violations early. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `_check_device_methods`，其作用是校验不变量，并尽早报告策略违规。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 141-149
```python
                # pyrefly: ignore [not-iterable, unsupported-operation]
                bias in arg.value
                for bias in DEVICE_BIAS
            ):
                self.record(
                    subnode,
                    f"{msg_prefix} .to('{arg.value}'), suggest to use .to(GPU_TYPE)",
                )
```
- **EN**: This chunk continues `_check_device_methods` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `_check_device_methods`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 150-167
```python
    def _check_with_statement(self, node: ast.With, msg_prefix: str) -> None:
        for item in node.items:
            ctx_expr = item.context_expr
            if isinstance(ctx_expr, ast.Call):
                func = ctx_expr.func
                if (
                    isinstance(func, ast.Attribute)
                    and func.attr == "device"
                    and isinstance(func.value, ast.Name)
                    and func.value.id == "torch"
                    and ctx_expr.args
                    and isinstance(ctx_expr.args[0], ast.Constant)
                    # pyrefly: ignore [not-iterable, unsupported-operation]
                    and any(bias in ctx_expr.args[0].value for bias in DEVICE_BIAS)
                ):
                    self.record(
                        ctx_expr,
                        f"{msg_prefix} `with torch.device('{ctx_expr.args[0].value}')`, suggest to use torch.device(GPU_TYPE)",
```
- **EN**: This chunk defines `_check_with_statement`, which validates invariants and reports policy violations early. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `_check_with_statement`，其作用是校验不变量，并尽早报告策略违规。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 168-180
```python
                    )

    def _check_node(self, node: ast.AST, msg_prefix: str) -> None:
        for subnode in ast.walk(node):
            if isinstance(subnode, ast.keyword):
                self._check_keyword_device(subnode, msg_prefix)
            elif isinstance(subnode, ast.Call) and isinstance(
                subnode.func, ast.Attribute
            ):
                self._check_device_methods(subnode, msg_prefix)
            elif isinstance(subnode, ast.With):
                self._check_with_statement(subnode, msg_prefix)
```
- **EN**: This chunk defines `_check_node`, which validates invariants and reports policy violations early. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `_check_node`，其作用是校验不变量，并尽早报告策略违规。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 181-192
```python
    def visit_FunctionDef(self, node: ast.FunctionDef) -> None:
        if self._has_proper_decorator(node):
            msg_prefix = (
                "`@requires_gpu` or `@requires_triton` function should not hardcode"
            )
            self._check_node(node, msg_prefix)
        elif self.is_gpu_test_suite:
            # If the function is guarded by HAS_GPU in main(), we still need to check for device bias
            msg_prefix = "The test suites is shared amount GPUS, should not hardcode"
            self._check_node(node, msg_prefix)
        self.generic_visit(node)
```
- **EN**: This chunk defines `visit_FunctionDef`, which implements a focused step inside the lint tooling pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段定义了 `visit_FunctionDef`，其作用是实现Lint 工具链流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 193-207
```python
    def record(self, node: ast.AST, message: str) -> None:
        self.lint_messages.append(
            LintMessage(
                path=self.filename,
                line=getattr(node, "lineno", None),
                char=None,
                code=LINTER_CODE,
                severity=LintSeverity.ERROR,
                name="[device-bias]",
                original=None,
                replacement=None,
                description=message,
            )
        )
```
- **EN**: This chunk defines `record`, which implements a focused step inside the lint tooling pipeline.
- **CN**: 这一段定义了 `record`，其作用是实现Lint 工具链流水线中的一个关键步骤。

### Lines 208-217
```python

def check_file(filename: str) -> list[LintMessage]:
    with open(filename) as f:
        source = f.read()
        tree = ast.parse(source, filename=filename)
        is_gpu_test_suite = is_main_has_gpu(tree)
        checker = DeviceBiasVisitor(filename, is_gpu_test_suite)
        checker.visit(tree)
    return checker.lint_messages
```
- **EN**: This chunk defines `check_file`, which validates invariants and reports policy violations early. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段定义了 `check_file`，其作用是校验不变量，并尽早报告策略违规。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 218-230
```python

def main() -> None:
    parser = argparse.ArgumentParser(
        description="Detect Device bias in functions decorated with requires_gpu/requires_triton"
        " or guarded by HAS_GPU block in main() that may break other GPU devices.",
        fromfile_prefix_chars="@",
    )
    parser.add_argument(
        "filenames",
        nargs="+",
        help="paths to lint",
    )
```
- **EN**: This chunk defines `main`, which orchestrates command-line execution and forwards parsed arguments into the core logic. Argument-parsing logic turns command-line inputs into structured parameters for the tool core.
- **CN**: 这一段定义了 `main`，其作用是编排命令行执行，并把解析后的参数转入核心逻辑。 参数解析逻辑把命令行输入转换为工具核心可以直接消费的结构化参数。

### Lines 231-239
```python
    args = parser.parse_args()

    with mp.Pool(8) as pool:
        lint_messages = pool.map(check_file, args.filenames)

    flat_lint_messages = []
    for sublist in lint_messages:
        flat_lint_messages.extend(sublist)
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 240-245
```python
    for lint_message in flat_lint_messages:
        print(json.dumps(lint_message._asdict()), flush=True)


if __name__ == "__main__":
    main()
```
- **EN**: This chunk continues `main` and expands its internal control flow or data movement. Structured-data parsing keeps repository metadata or intermediate reports in machine-readable form. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `main`，进一步展开其内部控制流或数据流转。 结构化数据解析使仓库元数据或中间报告保持机器可读形式。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

## Key Concepts / 关键概念

- **Lint tooling**
  - EN: This file belongs to the lint tooling layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于Lint 工具链层，应结合同一子目录中的相邻脚本一起理解。
- **Lint enforcement**
  - EN: The code applies policy checks to source files and reports actionable diagnostics.
  - CN: 代码对源码施加策略检查，并产出可执行的诊断信息。
- **CLI interface**
  - EN: The implementation exposes a command-line entry point and converts arguments into tool actions.
  - CN: 该实现暴露命令行入口，并把参数转换为工具动作。
- **Type-shape surface**
  - EN: The file models public Python typing information, stubs, or developer-facing signatures.
  - CN: 该文件建模公共 Python 类型信息、类型桩或面向开发者的签名。
- **LINTER_CODE**
  - EN: `LINTER_CODE` is one of the main local symbols exposed or implemented here.
  - CN: `LINTER_CODE` 是此处暴露或实现的主要局部符号之一。
- **LintSeverity**
  - EN: `LintSeverity` is one of the main local symbols exposed or implemented here.
  - CN: `LintSeverity` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `__future__`, `argparse`, `ast`, `json`, `multiprocessing`, `enum`, `typing`
- **Primary symbols in this file / 本文件核心符号**: `LINTER_CODE`, `LintSeverity`, `LintMessage`, `DEVICE_BIAS`, `GPU_RELATED_DECORATORS`, `is_main_has_gpu`, `DeviceBiasVisitor`, `check_file`, `main`
