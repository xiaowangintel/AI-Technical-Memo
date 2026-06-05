# test_streaming_session_extra.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/sessions/test_streaming_session_extra.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates streaming session extra behavior in SGLang's sessions area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 sessions 领域中与 streaming session extra 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.kits.streaming_session_kit import StreamingSessionKitMixin
from sglang.test.server_fixtures.streaming_session_fixture import (
    StreamingSessionServerBase,
)
from sglang.test.test_utils import (
    DEFAULT_DRAFT_MODEL_EAGLE3,
    DEFAULT_TARGET_MODEL_EAGLE3,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.test.ci.ci_register`, `sglang.test.kits.streaming_session_kit`, `sglang.test.server_fixtures.streaming_session_fixture`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.test.ci.ci_register`, `sglang.test.kits.streaming_session_kit`, `sglang.test.server_fixtures.streaming_session_fixture`。

### Lines 13-13: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=691, stage="extra-a", runner_config="1-gpu-large")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 16-18: class TestStreamingSessionRetractMixedChunk declaration / 类 TestStreamingSessionRetractMixedChunk 声明
```python
class TestStreamingSessionRetractMixedChunk(
    StreamingSessionServerBase, StreamingSessionKitMixin
):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `StreamingSessionServerBase`, `StreamingSessionKitMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `StreamingSessionServerBase`, `StreamingSessionKitMixin`。

### Lines 19-22: class-level constants and configuration for `TestStreamingSessionRetractMixedChunk` / 类级常量与配置
```python
    """Retract + --enable-mixed-chunk."""

    extra_args = ["--chunked-prefill-size", "128", "--enable-mixed-chunk"]
    env_overrides = [("SGLANG_TEST_RETRACT", True)]
```
**EN:** This block defines shared names such as `extra_args`, `env_overrides`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `extra_args`, `env_overrides` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 25-27: class TestStreamingSessionRetractLargePage declaration / 类 TestStreamingSessionRetractLargePage 声明
```python
class TestStreamingSessionRetractLargePage(
    StreamingSessionServerBase, StreamingSessionKitMixin
):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `StreamingSessionServerBase`, `StreamingSessionKitMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `StreamingSessionServerBase`, `StreamingSessionKitMixin`。

### Lines 28-32: class-level constants and configuration for `TestStreamingSessionRetractLargePage` / 类级常量与配置
```python
    """Retract + page=256: exercises page-aligned `_free_tail`. Partial-page
    free would corrupt pages still holding committed tokens."""

    extra_args = ["--chunked-prefill-size", "4096", "--page-size", "256"]
    env_overrides = [("SGLANG_TEST_RETRACT", True)]
```
**EN:** This block defines shared names such as `extra_args`, `env_overrides`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `extra_args`, `env_overrides` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 33-35: supporting source context / 辅助源码上下文
```python


# Common EAGLE3 spec args; reused by Eagle/EagleV2/EagleRetractLargePage variants.
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 36-50: module-level constants and configuration / 模块级常量与配置
```python
_EAGLE3_SPEC_ARGS = [
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
]
```
**EN:** This block defines shared names such as `_EAGLE3_SPEC_ARGS`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `_EAGLE3_SPEC_ARGS` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 53-53: class TestStreamingSessionEagle declaration / 类 TestStreamingSessionEagle 声明
```python
class TestStreamingSessionEagle(StreamingSessionServerBase, StreamingSessionKitMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `StreamingSessionServerBase`, `StreamingSessionKitMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `StreamingSessionServerBase`, `StreamingSessionKitMixin`。

### Lines 54-64: class-level constants and configuration for `TestStreamingSessionEagle` / 类级常量与配置
```python
    """EAGLE3 spec v1 (overlap disabled); offset=-1 — see kit's note."""

    kv_inherit_offset = -1
    model = DEFAULT_TARGET_MODEL_EAGLE3
    extra_args = [
        "--disable-overlap-schedule",
        "--chunked-prefill-size",
        "512",
        *_EAGLE3_SPEC_ARGS,
    ]
    env_overrides = [("SGLANG_ALLOW_OVERWRITE_LONGER_CONTEXT_LEN", True)]
```
**EN:** This block defines shared names such as `kv_inherit_offset`, `model`, `extra_args`, `env_overrides`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `kv_inherit_offset`, `model`, `extra_args`, `env_overrides` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 67-67: class TestStreamingSessionEagleV2 declaration / 类 TestStreamingSessionEagleV2 声明
```python
class TestStreamingSessionEagleV2(StreamingSessionServerBase, StreamingSessionKitMixin):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `StreamingSessionServerBase`, `StreamingSessionKitMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `StreamingSessionServerBase`, `StreamingSessionKitMixin`。

### Lines 68-79: class-level constants and configuration for `TestStreamingSessionEagleV2` / 类级常量与配置
```python
    """EAGLE3 spec v2 (overlap on)."""

    model = DEFAULT_TARGET_MODEL_EAGLE3
    extra_args = [
        "--chunked-prefill-size",
        "512",
        *_EAGLE3_SPEC_ARGS,
    ]
    env_overrides = [
        ("SGLANG_ENABLE_SPEC_V2", True),
        ("SGLANG_ALLOW_OVERWRITE_LONGER_CONTEXT_LEN", True),
    ]
```
**EN:** This block defines shared names such as `model`, `extra_args`, `env_overrides`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `model`, `extra_args`, `env_overrides` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 82-84: class TestStreamingSessionEagleRetractLargePage declaration / 类 TestStreamingSessionEagleRetractLargePage 声明
```python
class TestStreamingSessionEagleRetractLargePage(
    StreamingSessionServerBase, StreamingSessionKitMixin
):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `StreamingSessionServerBase`, `StreamingSessionKitMixin`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `StreamingSessionServerBase`, `StreamingSessionKitMixin`。

### Lines 85-101: class-level constants and configuration for `TestStreamingSessionEagleRetractLargePage` / 类级常量与配置
```python
    """EAGLE3 spec v1 + retract + page=256: max-pressure on `_free_tail`
    (spec tail + retract alloc-commit gap + page alignment)."""

    kv_inherit_offset = -1
    model = DEFAULT_TARGET_MODEL_EAGLE3
    extra_args = [
        "--disable-overlap-schedule",
        "--chunked-prefill-size",
        "4096",
        *_EAGLE3_SPEC_ARGS,
        "--page-size",
        "256",
    ]
    env_overrides = [
        ("SGLANG_TEST_RETRACT", True),
        ("SGLANG_ALLOW_OVERWRITE_LONGER_CONTEXT_LEN", True),
    ]
```
**EN:** This block defines shared names such as `kv_inherit_offset`, `model`, `extra_args`, `env_overrides`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `kv_inherit_offset`, `model`, `extra_args`, `env_overrides` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 104-105: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestStreamingSessionRetractMixedChunk`: Retract + --enable-mixed-chunk. / 用于组织相关测试、夹具或辅助方法。
- `TestStreamingSessionRetractLargePage`: Retract + page=256: exercises page-aligned `_free_tail`. / 用于组织相关测试、夹具或辅助方法。
- `TestStreamingSessionEagle`: EAGLE3 spec v1 (overlap disabled); offset=-1 — see kit's note. / 用于组织相关测试、夹具或辅助方法。
- `TestStreamingSessionEagleV2`: EAGLE3 spec v2 (overlap on). / 用于组织相关测试、夹具或辅助方法。
- `TestStreamingSessionEagleRetractLargePage`: EAGLE3 spec v1 + retract + page=256: max-pressure on `_free_tail` (spec tail + retract alloc-commit gap + page alignment). / 用于组织相关测试、夹具或辅助方法。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.kits.streaming_session_kit`, `sglang.test.server_fixtures.streaming_session_fixture`, `sglang.test.test_utils`

- **Total lines / 总行数**: 105
