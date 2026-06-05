# test_linear_attn_model_registry.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/configs/test_linear_attn_model_registry.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates linear attn model registry behavior in SGLang's unit / configs area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / configs 领域中与 linear attn model registry 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for srt/configs/linear_attn_model_registry.py"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3-14: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.srt.configs.linear_attn_model_registry import (
    _LINEAR_ATTN_MODEL_REGISTRY,
    LinearAttnModelSpec,
    get_linear_attn_config,
    get_linear_attn_spec_by_arch,
    import_backend_class,
    register_linear_attn_model,
)
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.srt.configs.linear_attn_model_registry`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.srt.configs.linear_attn_model_registry`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`。

### Lines 16-16: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=7, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 17-19: supporting source context / 辅助源码上下文
```python


# Dummy config classes for testing
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 20-20: class FakeLinearAttnConfig declaration / 类 FakeLinearAttnConfig 声明
```python
class FakeLinearAttnConfig:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 21-21: class-level constants and configuration for `FakeLinearAttnConfig` / 类级常量与配置
```python
    full_attention_layer_ids = [0, 2, 4]
```
**EN:** This block defines shared names such as `full_attention_layer_ids`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `full_attention_layer_ids` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 24-24: class FakeVLMWrapperConfig declaration / 类 FakeVLMWrapperConfig 声明
```python
class FakeVLMWrapperConfig:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 25-25: supporting statements / 辅助语句
```python
    """Simulates a VLM wrapper that has get_text_config()."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 27-28: method init / 方法 init
```python
    def __init__(self):
        self._text_config = FakeLinearAttnConfig()
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。

### Lines 30-31: helper routine get text config / 辅助流程 get text config
```python
    def get_text_config(self):
        return self._text_config
```
**EN:** This helper encapsulates `get_text_config` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `get_text_config`，以便周围测试复用准备、执行或校验逻辑。

### Lines 34-34: class AnotherConfig declaration / 类 AnotherConfig 声明
```python
class AnotherConfig:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 35-35: supporting statements / 辅助语句
```python
    pass
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 38-38: class TestLinearAttnModelRegistry declaration / 类 TestLinearAttnModelRegistry 声明
```python
class TestLinearAttnModelRegistry(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 39-42: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        # Save and clear the global registry between tests
        self._saved_registry = list(_LINEAR_ATTN_MODEL_REGISTRY)
        _LINEAR_ATTN_MODEL_REGISTRY.clear()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 44-46: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        _LINEAR_ATTN_MODEL_REGISTRY.clear()
        _LINEAR_ATTN_MODEL_REGISTRY.extend(self._saved_registry)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 48-55: method make spec / 方法 make spec
```python
    def _make_spec(self, **overrides):
        defaults = dict(
            config_class=FakeLinearAttnConfig,
            backend_class_name="sglang.srt.layers.attention.triton_backend.TritonAttnBackend",
            arch_names=["FakeModelForCausalLM"],
        )
        defaults.update(overrides)
        return LinearAttnModelSpec(**defaults)
```
**EN:** This block implements `_make_spec` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_spec`，承担模块行为中的一个聚焦逻辑片段。

### Lines 57-65: test case register and lookup by config / 测试用例 register and lookup by config
```python
    def test_register_and_lookup_by_config(self):
        spec = self._make_spec()
        register_linear_attn_model(spec)

        hf_config = FakeLinearAttnConfig()
        result = get_linear_attn_config(hf_config)
        self.assertIsNotNone(result)
        self.assertIs(result[0], spec)
        self.assertIs(result[1], hf_config)
```
**EN:** This test exercises `test_register_and_lookup_by_config` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_register_and_lookup_by_config`。

### Lines 67-72: test case lookup no match / 测试用例 lookup no match
```python
    def test_lookup_no_match(self):
        spec = self._make_spec()
        register_linear_attn_model(spec)

        result = get_linear_attn_config(AnotherConfig())
        self.assertIsNone(result)
```
**EN:** This test exercises `test_lookup_no_match` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lookup_no_match`。

### Lines 74-76: test case lookup empty registry / 测试用例 lookup empty registry
```python
    def test_lookup_empty_registry(self):
        result = get_linear_attn_config(FakeLinearAttnConfig())
        self.assertIsNone(result)
```
**EN:** This test exercises `test_lookup_empty_registry` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lookup_empty_registry`。

### Lines 78-88: test case unwrap text config / 测试用例 unwrap text config
```python
    def test_unwrap_text_config(self):
        spec = self._make_spec(unwrap_text_config=True)
        register_linear_attn_model(spec)

        vlm_config = FakeVLMWrapperConfig()
        result = get_linear_attn_config(vlm_config)
        self.assertIsNotNone(result)
        self.assertIs(result[0], spec)
        # The resolved config should be the inner text config
        self.assertIsInstance(result[1], FakeLinearAttnConfig)
        self.assertIs(result[1], vlm_config._text_config)
```
**EN:** This test exercises `test_unwrap_text_config` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unwrap_text_config`。

### Lines 90-98: test case unwrap text config no match / 测试用例 unwrap text config no match
```python
    def test_unwrap_text_config_no_match(self):
        """unwrap_text_config=False should not call get_text_config()."""
        spec = self._make_spec(unwrap_text_config=False)
        register_linear_attn_model(spec)

        vlm_config = FakeVLMWrapperConfig()
        # VLM wrapper itself is not a FakeLinearAttnConfig, so no match
        result = get_linear_attn_config(vlm_config)
        self.assertIsNone(result)
```
**EN:** unwrap_text_config=False should not call get_text_config(). This test exercises `test_unwrap_text_config_no_match` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** unwrap_text_config=False should not call get_text_config(). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unwrap_text_config_no_match`。

### Lines 100-106: test case lookup by arch / 测试用例 lookup by arch
```python
    def test_lookup_by_arch(self):
        spec = self._make_spec(arch_names=["AlphaForCausalLM", "BetaForCausalLM"])
        register_linear_attn_model(spec)

        self.assertIs(get_linear_attn_spec_by_arch("AlphaForCausalLM"), spec)
        self.assertIs(get_linear_attn_spec_by_arch("BetaForCausalLM"), spec)
        self.assertIsNone(get_linear_attn_spec_by_arch("GammaForCausalLM"))
```
**EN:** This test exercises `test_lookup_by_arch` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lookup_by_arch`。

### Lines 108-109: test case lookup by arch empty registry / 测试用例 lookup by arch empty registry
```python
    def test_lookup_by_arch_empty_registry(self):
        self.assertIsNone(get_linear_attn_spec_by_arch("AnyArch"))
```
**EN:** This test exercises `test_lookup_by_arch_empty_registry` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lookup_by_arch_empty_registry`。

### Lines 111-129: test case multiple registrations / 测试用例 multiple registrations
```python
    def test_multiple_registrations(self):
        spec_a = self._make_spec(
            config_class=FakeLinearAttnConfig,
            arch_names=["AlphaForCausalLM"],
        )
        spec_b = self._make_spec(
            config_class=AnotherConfig,
            arch_names=["BetaForCausalLM"],
        )
        register_linear_attn_model(spec_a)
        register_linear_attn_model(spec_b)

        # Config-based lookup
        self.assertIs(get_linear_attn_config(FakeLinearAttnConfig())[0], spec_a)
        self.assertIs(get_linear_attn_config(AnotherConfig())[0], spec_b)

        # Arch-based lookup
        self.assertIs(get_linear_attn_spec_by_arch("AlphaForCausalLM"), spec_a)
        self.assertIs(get_linear_attn_spec_by_arch("BetaForCausalLM"), spec_b)
```
**EN:** This test exercises `test_multiple_registrations` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_registrations`。

### Lines 131-139: test case first match wins / 测试用例 first match wins
```python
    def test_first_match_wins(self):
        """When two specs match the same config class, the first registered wins."""
        spec1 = self._make_spec(backend_class_name="pkg.Backend1")
        spec2 = self._make_spec(backend_class_name="pkg.Backend2")
        register_linear_attn_model(spec1)
        register_linear_attn_model(spec2)

        result = get_linear_attn_config(FakeLinearAttnConfig())
        self.assertIs(result[0], spec1)
```
**EN:** When two specs match the same config class, the first registered wins. This test exercises `test_first_match_wins` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When two specs match the same config class, the first registered wins. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_first_match_wins`。

### Lines 141-146: test case import backend class / 测试用例 import backend class
```python
    def test_import_backend_class(self):
        # Import a real stdlib class to verify the mechanism
        cls = import_backend_class("collections.OrderedDict")
        from collections import OrderedDict

        self.assertIs(cls, OrderedDict)
```
**EN:** This test exercises `test_import_backend_class` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_import_backend_class`。

### Lines 148-157: test case spec defaults / 测试用例 spec defaults
```python
    def test_spec_defaults(self):
        spec = LinearAttnModelSpec(
            config_class=FakeLinearAttnConfig,
            backend_class_name="pkg.mod.Cls",
        )
        self.assertEqual(spec.arch_names, [])
        self.assertTrue(spec.uses_mamba_radix_cache)
        self.assertTrue(spec.support_mamba_cache)
        self.assertFalse(spec.support_mamba_cache_extra_buffer)
        self.assertFalse(spec.unwrap_text_config)
```
**EN:** This test exercises `test_spec_defaults` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_spec_defaults`。

### Lines 160-161: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `FakeLinearAttnConfig`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `FakeVLMWrapperConfig`: Simulates a VLM wrapper that has get_text_config(). / 用于组织相关测试、夹具或辅助方法。
- `AnotherConfig`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestLinearAttnModelRegistry`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `FakeVLMWrapperConfig.__init__`: This block implements `__init__` and captures one focused piece of the module's behavior. / 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。
- `FakeVLMWrapperConfig.get_text_config`: This helper encapsulates `get_text_config` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `get_text_config`，以便周围测试复用准备、执行或校验逻辑。
- `TestLinearAttnModelRegistry.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestLinearAttnModelRegistry.tearDown`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestLinearAttnModelRegistry._make_spec`: This block implements `_make_spec` and captures one focused piece of the module's behavior. / 该代码块实现 `_make_spec`，承担模块行为中的一个聚焦逻辑片段。
- `TestLinearAttnModelRegistry.test_register_and_lookup_by_config`: This test exercises `test_register_and_lookup_by_config` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_register_and_lookup_by_config`。
- `TestLinearAttnModelRegistry.test_lookup_no_match`: This test exercises `test_lookup_no_match` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lookup_no_match`。
- `TestLinearAttnModelRegistry.test_lookup_empty_registry`: This test exercises `test_lookup_empty_registry` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lookup_empty_registry`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.srt.configs.linear_attn_model_registry`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 161
