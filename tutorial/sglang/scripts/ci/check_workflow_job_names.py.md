# check_workflow_job_names.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/check_workflow_job_names.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `check_workflow_job_names` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Python 模块用于支撑 SGLang 中的 `check_workflow_job_names` 流程，主要负责CI 编排、测试执行。它属于 `ci` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: File header and usage notes / 文件头与使用说明
```python
#!/usr/bin/env python3
```
**EN:** These opening lines establish the file context, such as the shebang, encoding note, or top-level comments that tell operators how the script should be used.
**CN:** 这些起始行用于建立文件上下文，例如 shebang、编码说明或顶层注释，帮助使用者理解脚本的执行方式。

### Lines 2-11: Module overview and inline documentation / 模块概览与内嵌说明
```python
"""Check that required status check job names are unique across workflows.

Duplicate job names on the same commit allow a passing job in one workflow
to satisfy a required status check meant for a different workflow, bypassing
branch protection.

See: https://github.com/sgl-project/sglang/pull/20208 for an example where
pr-test-npu.yml's "pr-test-finish" job (which passed) caused GitHub to treat
the required "pr-test-finish" check (from pr-test.yml, which failed) as met.
"""
```
**EN:** Check that required status check job names are unique across workflows.
**CN:** 该代码块提供模块级说明，帮助读者快速理解文件目标及使用约束。

### Lines 13-17: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import glob
import sys
from collections import defaultdict

import yaml
```
**EN:** This block loads glob, sys, collections, yaml. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 glob, sys, collections, yaml。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 21-24: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
PROTECTED_JOB_NAMES = {
    "pr-test-finish",
    "lint",
}
```
**EN:** This section defines PROTECTED_JOB_NAMES, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 PROTECTED_JOB_NAMES，用于把可复用的默认值集中在模块顶部。

### Lines 27-54: Defines the `main` routine / 定义 `main` 例程
```python
def main() -> int:
    workflows = sorted(glob.glob(".github/workflows/*.yml"))
    job_to_files: dict[str, list[str]] = defaultdict(list)

    for wf in workflows:
        with open(wf, encoding="utf-8") as f:
            data = yaml.safe_load(f)
        if not data or "jobs" not in data:
            continue
        for job in data["jobs"]:
            if job in PROTECTED_JOB_NAMES:
                job_to_files[job].append(wf)

    duplicates = {job: files for job, files in job_to_files.items() if len(files) > 1}

    if not duplicates:
        return 0

    print("ERROR: Required status check job names must be unique across workflows.")
    print("Duplicates allow branch protection bypass via auto-merge.\n")
    for job, files in sorted(duplicates.items()):
        print(f"  Job '{job}' appears in:")
        for f in files:
            print(f"    - {f}")
        print()

    print("Fix: rename the job in non-primary workflows to avoid collision.")
    return 1
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it reads or writes files, emits status messages, loads structured configuration.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会读写文件、输出状态信息、加载结构化配置。

### Lines 57-58: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    sys.exit(main())
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It controls process exit status.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会控制进程退出状态。

## Key Concepts / 关键概念
- **Environment management** / 环境管理
- **Process control** / 进程控制
- **YAML configuration** / YAML 配置
- **Git state inspection** / Git 状态检查

## Dependencies / 依赖关系
- **Standard library / 标准库**: `collections`, `glob`, `sys`
- **Third-party modules / 第三方模块**: `yaml`
