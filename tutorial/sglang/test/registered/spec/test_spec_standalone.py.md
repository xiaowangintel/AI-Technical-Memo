# test_spec_standalone.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/spec/test_spec_standalone.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates spec standalone behavior in SGLang's spec area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 推测解码 领域中与 spec standalone 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

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

# V2 standalone speculative decoding tests (FA3, Triton, FlashInfer backends).
# Non-V2 backends moved to test_spec_standalone_extra.py.
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 9-9: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=406, stage="base-b", runner_config="1-gpu-large")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 12-12: class TestStandaloneV2SpeculativeDecodingBase declaration / 类 TestStandaloneV2SpeculativeDecodingBase 声明
```python
class TestStandaloneV2SpeculativeDecodingBase(StandaloneServerBase, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `StandaloneServerBase`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `StandaloneServerBase`, `CustomTestCase`。

### Lines 13-13: class-level constants and configuration for `TestStandaloneV2SpeculativeDecodingBase` / 类级常量与配置
```python
    attention_backend = "fa3"
```
**EN:** This block defines shared names such as `attention_backend`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `attention_backend` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 16-16: class TestStandaloneV2SpeculativeDecodingTriton declaration / 类 TestStandaloneV2SpeculativeDecodingTriton 声明
```python
class TestStandaloneV2SpeculativeDecodingTriton(StandaloneServerBase, CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `StandaloneServerBase`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `StandaloneServerBase`, `CustomTestCase`。

### Lines 17-17: class-level constants and configuration for `TestStandaloneV2SpeculativeDecodingTriton` / 类级常量与配置
```python
    attention_backend = "triton"
```
**EN:** This block defines shared names such as `attention_backend`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `attention_backend` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 20-22: class TestStandaloneV2SpeculativeDecodingFlashinfer declaration / 类 TestStandaloneV2SpeculativeDecodingFlashinfer 声明
```python
class TestStandaloneV2SpeculativeDecodingFlashinfer(
    StandaloneServerBase, CustomTestCase
):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `StandaloneServerBase`, `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `StandaloneServerBase`, `CustomTestCase`。

### Lines 23-23: class-level constants and configuration for `TestStandaloneV2SpeculativeDecodingFlashinfer` / 类级常量与配置
```python
    attention_backend = "flashinfer"
```
**EN:** This block defines shared names such as `attention_backend`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `attention_backend` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 26-27: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestStandaloneV2SpeculativeDecodingBase`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestStandaloneV2SpeculativeDecodingTriton`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestStandaloneV2SpeculativeDecodingFlashinfer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.server_fixtures.standalone_fixture`, `sglang.test.test_utils`

- **Total lines / 总行数**: 27
