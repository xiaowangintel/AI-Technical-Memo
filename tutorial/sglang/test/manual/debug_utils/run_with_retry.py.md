# run_with_retry.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/debug_utils/run_with_retry.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `run with retry` scenario in `test/manual/debug_utils`. It uses SGLang's shared test infrastructure to configure models or services and verify expected functional behavior. / 该手动测试模块覆盖 `test/manual/debug_utils` 中的 `run with retry` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Scenario logic / 场景逻辑
```python
"""
Simple wrapper to run a test file with retry logic.

Usage:
    python3 -m sglang.test.ci.run_with_retry test_file.py [--max-attempts 2] [--retry-wait 60]
"""

import argparse
import sys

from sglang.test.ci.ci_utils import TestFile, run_unittest_files
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 14-38: Helper routines around main / 辅助例程
```python
def main():
    parser = argparse.ArgumentParser(description="Run a test file with retry logic")
    parser.add_argument("test_file", help="The test file to run")
    parser.add_argument(
        "--max-attempts",
        type=int,
        default=2,
        help="Maximum number of attempts (default: 2)",
    )
    parser.add_argument(
        "--retry-wait",
        type=int,
        default=60,
        help="Seconds to wait between retries (default: 60)",
    )
    parser.add_argument(
        "--timeout",
        type=int,
        default=1200,
        help="Timeout per attempt in seconds (default: 1200)",
    )
    args = parser.parse_args()

    # Create a TestFile with a reasonable estimated time
    test_file = TestFile(name=args.test_file, estimated_time=args.timeout)
```
**EN:** This range implements helper routine(s) `main` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `ArgumentParser`, `add_argument`, `attempts` and `retries`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 39-49: Scenario logic / 场景逻辑
```python

    exit_code = run_unittest_files(
        files=[test_file],
        timeout_per_file=args.timeout,
        continue_on_error=False,
        enable_retry=True,
        max_attempts=args.max_attempts,
        retry_wait_seconds=args.retry_wait,
    )

    sys.exit(exit_code)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `run_unittest_files` and `exit`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 50-53: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Scenario-specific test logic / 场景化测试逻辑

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `argparse`, `sys`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang.test.ci.ci_utils`
