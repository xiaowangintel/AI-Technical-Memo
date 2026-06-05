# test_spec_standalone_extra.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/spec/test_spec_standalone_extra.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates spec standalone extra behavior in SGLang's spec area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 推测解码 领域中与 spec standalone extra 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.server_fixtures.standalone_fixture import StandaloneServerBase
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.test.ci.ci_register`, `sglang.test.server_fixtures.standalone_fixture`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.test.ci.ci_register`, `sglang.test.server_fixtures.standalone_fixture`, `sglang.test.test_utils`。

### Lines 6-8: supporting source context / 辅助源码上下文
```python

# Non-V2 standalone speculative decoding tests (FA3, Triton, FlashInfer
# backends). Sibling V2 classes stay per-commit in test_spec_standalone.py.
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 9-9: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=406, stage="extra-a", runner_config="1-gpu-large")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 12-12: class TestStandaloneSpeculativeDecodingBase declaration / 类 TestStandaloneSpeculativeDecodingBase 声明
```python
class TestStandaloneSpeculativeDecodingBase(StandaloneServerBase, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `StandaloneServerBase`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `StandaloneServerBase`, `CustomTestCase`。

### Lines 13-16: class-level constants and configuration for `TestStandaloneSpeculativeDecodingBase` / 类级常量与配置
```python
    attention_backend = "fa3"
    speculative_eagle_topk = 2
    speculative_num_draft_tokens = 7
    enable_spec_v2 = False
```
**EN:** This block defines shared names such as `attention_backend`, `speculative_eagle_topk`, `speculative_num_draft_tokens`, `enable_spec_v2`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `attention_backend`, `speculative_eagle_topk`, `speculative_num_draft_tokens`, `enable_spec_v2` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 19-19: class TestStandaloneSpeculativeDecodingTriton declaration / 类 TestStandaloneSpeculativeDecodingTriton 声明
```python
class TestStandaloneSpeculativeDecodingTriton(StandaloneServerBase, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `StandaloneServerBase`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `StandaloneServerBase`, `CustomTestCase`。

### Lines 20-23: class-level constants and configuration for `TestStandaloneSpeculativeDecodingTriton` / 类级常量与配置
```python
    attention_backend = "triton"
    speculative_eagle_topk = 2
    speculative_num_draft_tokens = 7
    enable_spec_v2 = False
```
**EN:** This block defines shared names such as `attention_backend`, `speculative_eagle_topk`, `speculative_num_draft_tokens`, `enable_spec_v2`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `attention_backend`, `speculative_eagle_topk`, `speculative_num_draft_tokens`, `enable_spec_v2` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 26-26: class TestStandaloneSpeculativeDecodingFlashinfer declaration / 类 TestStandaloneSpeculativeDecodingFlashinfer 声明
```python
class TestStandaloneSpeculativeDecodingFlashinfer(StandaloneServerBase, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `StandaloneServerBase`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `StandaloneServerBase`, `CustomTestCase`。

### Lines 27-30: class-level constants and configuration for `TestStandaloneSpeculativeDecodingFlashinfer` / 类级常量与配置
```python
    attention_backend = "flashinfer"
    speculative_eagle_topk = 2
    speculative_num_draft_tokens = 7
    enable_spec_v2 = False
```
**EN:** This block defines shared names such as `attention_backend`, `speculative_eagle_topk`, `speculative_num_draft_tokens`, `enable_spec_v2`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `attention_backend`, `speculative_eagle_topk`, `speculative_num_draft_tokens`, `enable_spec_v2` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 33-34: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestStandaloneSpeculativeDecodingBase`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestStandaloneSpeculativeDecodingTriton`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestStandaloneSpeculativeDecodingFlashinfer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.server_fixtures.standalone_fixture`, `sglang.test.test_utils`

- **Total lines / 总行数**: 34
