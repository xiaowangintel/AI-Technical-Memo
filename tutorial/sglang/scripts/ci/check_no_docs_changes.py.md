# check_no_docs_changes.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/check_no_docs_changes.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `check_no_docs_changes` workflow in SGLang. It mainly handles CI orchestration, policy validation. / 该Python 模块用于支撑 SGLang 中的 `check_no_docs_changes` 流程，主要负责CI 编排、策略校验。它属于 `ci` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: File header and usage notes / 文件头与使用说明
```python
#!/usr/bin/env python3
```
**EN:** These opening lines establish the file context, such as the shebang, encoding note, or top-level comments that tell operators how the script should be used.
**CN:** 这些起始行用于建立文件上下文，例如 shebang、编码说明或顶层注释，帮助使用者理解脚本的执行方式。

### Lines 2-2: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""Reject staged changes under the legacy docs/ tree."""
```
**EN:** Reject staged changes under the legacy docs/ tree.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 4-7: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
from __future__ import annotations

import subprocess
import sys
```
**EN:** This block loads __future__, subprocess, sys. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 __future__, subprocess, sys。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 9-20: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
ERROR_MESSAGE = """\
Changes under the legacy docs/ directory are not allowed.

The documentation has been migrated. Please make documentation updates in the
corresponding location under docs_new/ instead.
"""

LEGACY_DOCS_ALLOWLIST = {
    "docs/_static/css/custom_log.css",
    "docs/_static/js/deprecation_banner.js",
    "docs/conf.py",
}
```
**EN:** This section defines ERROR_MESSAGE, LEGACY_DOCS_ALLOWLIST, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 ERROR_MESSAGE, LEGACY_DOCS_ALLOWLIST，用于把可复用的默认值集中在模块顶部。

### Lines 23-36: Defines the `staged_paths` routine / 定义 `staged_paths` 例程
```python
def staged_paths() -> list[str]:
    result = subprocess.run(
        [
            "git",
            "diff",
            "--cached",
            "--name-only",
            "--diff-filter=ACMRDTUXB",
        ],
        check=True,
        capture_output=True,
        text=True,
    )
    return [line.strip() for line in result.stdout.splitlines() if line.strip()]
```
**EN:** This block defines `staged_paths`. It takes no explicit parameters at the top level. Internally it executes external commands.
**CN:** 该代码块定义了 `staged_paths`。它在顶层定义中不接收显式参数。内部会执行外部命令。

### Lines 39-55: Defines the `main` routine / 定义 `main` 例程
```python
def main() -> int:
    paths = sys.argv[1:] or staged_paths()
    docs_paths = sorted(
        path
        for path in paths
        if (path == "docs" or path.startswith("docs/"))
        and path not in LEGACY_DOCS_ALLOWLIST
    )

    if not docs_paths:
        return 0

    print(ERROR_MESSAGE, file=sys.stderr)
    print("Detected legacy docs/ changes:", file=sys.stderr)
    for path in docs_paths:
        print(f"  - {path}", file=sys.stderr)
    return 1
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it emits status messages.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会输出状态信息。

### Lines 58-59: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    raise SystemExit(main())
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It hands control to the previously defined helpers.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会把控制流交给前面定义的辅助函数。

## Key Concepts / 关键概念
- **Subprocess orchestration** / 子进程编排
- **Process control** / 进程控制
- **Git state inspection** / Git 状态检查

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `subprocess`, `sys`
- **External commands inferred from code / 从代码推断的外部命令**: `git`
