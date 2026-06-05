# test_streaming_session.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/sessions/test_streaming_session.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates streaming session behavior in SGLang's sessions area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 sessions 领域中与 streaming session 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: supporting statements / 辅助语句
```python
"""Per-commit streaming-session tests.

Default config + EagleV2RetractLargePage + abort-leak repro stay per-commit.
Other variants (Retract / Eagle / EagleV2 / EagleRetractLargePage) live in
test_streaming_session_extra.py.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 8-24: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.kits.streaming_session_kit import (
    AbortLeakReproKitMixin,
    StreamingSessionKitMixin,
)
from sglang.test.server_fixtures.streaming_session_fixture import (
    ABORT_REPRO_CHUNKED_PREFILL_SIZE,
    ABORT_REPRO_CONTEXT_LEN,
    ABORT_REPRO_PAGE_SIZE,
    StreamingSessionServerBase,
)
from sglang.test.test_utils import (
    DEFAULT_DRAFT_MODEL_EAGLE3,
    DEFAULT_TARGET_MODEL_EAGLE3,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.test.ci.ci_register`, `sglang.test.kits.streaming_session_kit`, `sglang.test.server_fixtures.streaming_session_fixture`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.test.ci.ci_register`, `sglang.test.kits.streaming_session_kit`, `sglang.test.server_fixtures.streaming_session_fixture`。

### Lines 26-27: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=691, stage="base-b", runner_config="1-gpu-large")
register_amd_ci(est_time=691, suite="stage-b-test-1-gpu-large-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 30-30: class TestStreamingSession declaration / 类 TestStreamingSession 声明
```python
class TestStreamingSession(StreamingSessionServerBase, StreamingSessionKitMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `StreamingSessionServerBase`, `StreamingSessionKitMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `StreamingSessionServerBase`, `StreamingSessionKitMixin`。

### Lines 31-33: class-level constants and configuration for `TestStreamingSession` / 类级常量与配置
```python
    """Default streaming-session config (small model, no spec)."""

    extra_args = ["--chunked-prefill-size", "512"]
```
**EN:** This block defines shared names such as `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 36-36: class TestStreamingSessionEagleV2RetractLargePage declaration / 类 TestStreamingSessionEagleV2RetractLargePage 声明
```python
class TestStreamingSessionEagleV2RetractLargePage(TestStreamingSession):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `TestStreamingSession`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `TestStreamingSession`。

### Lines 37-63: class-level constants and configuration for `TestStreamingSessionEagleV2RetractLargePage` / 类级常量与配置
```python
    """EAGLE3 spec v2 + retract + page=256."""

    model = DEFAULT_TARGET_MODEL_EAGLE3
    extra_args = [
        "--chunked-prefill-size",
        "4096",
        "--dtype=float16",
        "--speculative-algorithm",
        "EAGLE3",
        "--speculative-draft-model",
        DEFAULT_DRAFT_MODEL_EAGLE3,
        "--speculative-num-steps",
        "3",
        "--speculative-eagle-topk",
        "1",
        "--speculative-num-draft-tokens",
        "4",
        "--mem-fraction-static",
        "0.7",
        "--page-size",
        "256",
    ]
    env_overrides = [
        ("SGLANG_ENABLE_SPEC_V2", True),
        ("SGLANG_TEST_RETRACT", True),
        ("SGLANG_ALLOW_OVERWRITE_LONGER_CONTEXT_LEN", True),
    ]
```
**EN:** This block defines shared names such as `model`, `extra_args`, `env_overrides`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `extra_args`, `env_overrides` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 66-68: class TestStreamingSessionAbortLeakRepro declaration / 类 TestStreamingSessionAbortLeakRepro 声明
```python
class TestStreamingSessionAbortLeakRepro(
    StreamingSessionServerBase, AbortLeakReproKitMixin
):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `StreamingSessionServerBase`, `AbortLeakReproKitMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `StreamingSessionServerBase`, `AbortLeakReproKitMixin`。

### Lines 69-80: class-level constants and configuration for `TestStreamingSessionAbortLeakRepro` / 类级常量与配置
```python
    extra_args = [
        "--chunked-prefill-size",
        str(ABORT_REPRO_CHUNKED_PREFILL_SIZE),
        "--context-length",
        str(ABORT_REPRO_CONTEXT_LEN),
        "--page-size",
        str(ABORT_REPRO_PAGE_SIZE),
        "--max-running-requests",
        "32",
        "--log-level",
        "info",
    ]
```
**EN:** This block defines shared names such as `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 83-84: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestStreamingSession`: Default streaming-session config (small model, no spec). / 用于组织相关测试、夹具或辅助方法。
- `TestStreamingSessionEagleV2RetractLargePage`: EAGLE3 spec v2 + retract + page=256. / 用于组织相关测试、夹具或辅助方法。
- `TestStreamingSessionAbortLeakRepro`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.kits.streaming_session_kit`, `sglang.test.server_fixtures.streaming_session_fixture`, `sglang.test.test_utils`

- **Total lines / 总行数**: 84
