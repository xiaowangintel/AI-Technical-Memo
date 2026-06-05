# test_streaming_session_swa.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/sessions/test_streaming_session_swa.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates streaming session swa behavior in SGLang's sessions area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 sessions 领域中与 streaming session swa 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.test.ci.ci_register import register_cuda_ci
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
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.test.ci.ci_register`, `sglang.test.kits.streaming_session_kit`, `sglang.test.server_fixtures.streaming_session_fixture`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.test.ci.ci_register`, `sglang.test.kits.streaming_session_kit`, `sglang.test.server_fixtures.streaming_session_fixture`。

### Lines 15-25: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=519, stage="base-b", runner_config="1-gpu-large")


SWA_MODEL = "openai/gpt-oss-20b"

# Common gpt-oss-20b launch args. Matches TestSessionLatency/TestSWARadixCacheKL.
SWA_COMMON_ARGS = [
    "--mem-fraction-static",
    "0.70",
    "--disable-piecewise-cuda-graph",
]
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 28-28: class TestStreamingSessionSWA declaration / 类 TestStreamingSessionSWA 声明
```python
class TestStreamingSessionSWA(StreamingSessionServerBase, StreamingSessionKitMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `StreamingSessionServerBase`, `StreamingSessionKitMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `StreamingSessionServerBase`, `StreamingSessionKitMixin`。

### Lines 29-32: class-level constants and configuration for `TestStreamingSessionSWA` / 类级常量与配置
```python
    """Baseline streaming session on a hybrid-SWA model."""

    model = SWA_MODEL
    extra_args = ["--chunked-prefill-size", "512", *SWA_COMMON_ARGS]
```
**EN:** This block defines shared names such as `model`, `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 35-37: class TestStreamingSessionSWARetractLargePage declaration / 类 TestStreamingSessionSWARetractLargePage 声明
```python
class TestStreamingSessionSWARetractLargePage(
    StreamingSessionServerBase, StreamingSessionKitMixin
):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `StreamingSessionServerBase`, `StreamingSessionKitMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `StreamingSessionServerBase`, `StreamingSessionKitMixin`。

### Lines 38-48: class-level constants and configuration for `TestStreamingSessionSWARetractLargePage` / 类级常量与配置
```python
    """SWA under retract decode with page=256."""

    model = SWA_MODEL
    extra_args = [
        "--chunked-prefill-size",
        "4096",
        "--page-size",
        "256",
        *SWA_COMMON_ARGS,
    ]
    env_overrides = [("SGLANG_TEST_RETRACT", True)]
```
**EN:** This block defines shared names such as `model`, `extra_args`, `env_overrides`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `extra_args`, `env_overrides` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 51-53: class TestStreamingSessionSWARetractMixedChunk declaration / 类 TestStreamingSessionSWARetractMixedChunk 声明
```python
class TestStreamingSessionSWARetractMixedChunk(
    StreamingSessionServerBase, StreamingSessionKitMixin
):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `StreamingSessionServerBase`, `StreamingSessionKitMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `StreamingSessionServerBase`, `StreamingSessionKitMixin`。

### Lines 54-63: class-level constants and configuration for `TestStreamingSessionSWARetractMixedChunk` / 类级常量与配置
```python
    """SWA under retract decode with --enable-mixed-chunk."""

    model = SWA_MODEL
    extra_args = [
        "--chunked-prefill-size",
        "128",
        "--enable-mixed-chunk",
        *SWA_COMMON_ARGS,
    ]
    env_overrides = [("SGLANG_TEST_RETRACT", True)]
```
**EN:** This block defines shared names such as `model`, `extra_args`, `env_overrides`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `extra_args`, `env_overrides` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 66-68: class TestStreamingSessionSWAAbortLeakRepro declaration / 类 TestStreamingSessionSWAAbortLeakRepro 声明
```python
class TestStreamingSessionSWAAbortLeakRepro(
    StreamingSessionServerBase, AbortLeakReproKitMixin
):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `StreamingSessionServerBase`, `AbortLeakReproKitMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `StreamingSessionServerBase`, `AbortLeakReproKitMixin`。

### Lines 69-84: class-level constants and configuration for `TestStreamingSessionSWAAbortLeakRepro` / 类级常量与配置
```python
    """SWA abort-heavy chunked prefill leak repro."""

    model = SWA_MODEL
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
        *SWA_COMMON_ARGS,
    ]
```
**EN:** This block defines shared names such as `model`, `extra_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `extra_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 87-88: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestStreamingSessionSWA`: Baseline streaming session on a hybrid-SWA model. / 用于组织相关测试、夹具或辅助方法。
- `TestStreamingSessionSWARetractLargePage`: SWA under retract decode with page=256. / 用于组织相关测试、夹具或辅助方法。
- `TestStreamingSessionSWARetractMixedChunk`: SWA under retract decode with --enable-mixed-chunk. / 用于组织相关测试、夹具或辅助方法。
- `TestStreamingSessionSWAAbortLeakRepro`: SWA abort-heavy chunked prefill leak repro. / 用于组织相关测试、夹具或辅助方法。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.kits.streaming_session_kit`, `sglang.test.server_fixtures.streaming_session_fixture`

- **Total lines / 总行数**: 88
