# test_hybrid_attn_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/attention/test_hybrid_attn_backend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on attention hybrid attn backend in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 attention hybrid attn backend 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Import dependencies
```python
import unittest

from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.server_fixtures.hybrid_attn_backend_fixture import (
    TestHybridAttnBackendBase,
)
from sglang.test.test_utils import (
    DEFAULT_DRAFT_MODEL_EAGLE,
    DEFAULT_MODEL_NAME_FOR_TEST_MLA,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for CUDA CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 14-14: Register CI metadata
```python
register_cuda_ci(est_time=407, stage="extra-a", runner_config="1-gpu-large")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围。

### Lines 17-17: Define class TestHybridAttnBackendMLA
```python
class TestHybridAttnBackendMLA(TestHybridAttnBackendBase):
```
**EN:** This declaration introduces the `TestHybridAttnBackendMLA` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestHybridAttnBackendMLA` 测试类，并说明它通过继承承担的职责。

### Lines 18-19: Declare TestHybridAttnBackendMLA configuration
```python
    accuracy_threshold = 0.60
    model = DEFAULT_MODEL_NAME_FOR_TEST_MLA
```
**EN:** This block defines class-level settings that are shared across the `TestHybridAttnBackendMLA` test methods.
**CN:** 该代码块定义了 `TestHybridAttnBackendMLA` 各测试方法共享的类级配置。

### Lines 22-22: Define class TestHybridAttnBackendTorchCompile
```python
class TestHybridAttnBackendTorchCompile(TestHybridAttnBackendBase):
```
**EN:** This declaration introduces the `TestHybridAttnBackendTorchCompile` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestHybridAttnBackendTorchCompile` 测试类，并说明它通过继承承担的职责。

### Lines 23-24: Declare TestHybridAttnBackendTorchCompile configuration
```python
    accuracy_threshold = 0.65
    extra_args = ["--enable-torch-compile"]
```
**EN:** This block defines class-level settings that are shared across the `TestHybridAttnBackendTorchCompile` test methods.
**CN:** 该代码块定义了 `TestHybridAttnBackendTorchCompile` 各测试方法共享的类级配置。

### Lines 27-27: Define class TestHybridAttnBackendSpeculativeDecodingPrefillBackend
```python
class TestHybridAttnBackendSpeculativeDecodingPrefillBackend(TestHybridAttnBackendBase):
```
**EN:** This declaration introduces the `TestHybridAttnBackendSpeculativeDecodingPrefillBackend` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestHybridAttnBackendSpeculativeDecodingPrefillBackend` 测试类，并说明它通过继承承担的职责。

### Lines 28-44: Declare TestHybridAttnBackendSpeculativeDecodingPrefillBackend configuration
```python
    speculative_decode = True
    # This eagle test uses a very small model, so the accuracy is low.
    accuracy_threshold = 0.2
    extra_args = [
        "--speculative-algorithm",
        "EAGLE",
        "--speculative-draft-model-path",
        DEFAULT_DRAFT_MODEL_EAGLE,
        "--speculative-num-steps",
        "3",
        "--speculative-eagle-topk",
        "2",
        "--speculative-num-draft-tokens",
        "4",
        "--speculative-attention-mode",
        "prefill",
    ]
```
**EN:** This block defines class-level settings that are shared across the `TestHybridAttnBackendSpeculativeDecodingPrefillBackend` test methods.
**CN:** 该代码块定义了 `TestHybridAttnBackendSpeculativeDecodingPrefillBackend` 各测试方法共享的类级配置。

### Lines 47-47: Define class TestHybridAttnBackendSpeculativeDecodingDecodeBackend
```python
class TestHybridAttnBackendSpeculativeDecodingDecodeBackend(TestHybridAttnBackendBase):
```
**EN:** This declaration introduces the `TestHybridAttnBackendSpeculativeDecodingDecodeBackend` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestHybridAttnBackendSpeculativeDecodingDecodeBackend` 测试类，并说明它通过继承承担的职责。

### Lines 48-64: Declare TestHybridAttnBackendSpeculativeDecodingDecodeBackend configuration
```python
    speculative_decode = True
    # This eagle test uses a very small model, so the accuracy is low.
    accuracy_threshold = 0.2
    extra_args = [
        "--speculative-algorithm",
        "EAGLE",
        "--speculative-draft-model-path",
        DEFAULT_DRAFT_MODEL_EAGLE,
        "--speculative-num-steps",
        "3",
        "--speculative-eagle-topk",
        "2",
        "--speculative-num-draft-tokens",
        "4",
        "--speculative-attention-mode",
        "decode",
    ]
```
**EN:** This block defines class-level settings that are shared across the `TestHybridAttnBackendSpeculativeDecodingDecodeBackend` test methods.
**CN:** 该代码块定义了 `TestHybridAttnBackendSpeculativeDecodingDecodeBackend` 各测试方法共享的类级配置。

### Lines 67-68: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.test.ci.ci_register`, `sglang.test.server_fixtures.hybrid_attn_backend_fixture`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `unittest`
- Notable symbols / 关键符号: `register_cuda_ci`, `unittest.main`, `DEFAULT_MODEL_NAME_FOR_TEST`
