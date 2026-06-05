# test_quantization.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/hardware_backend/mlx/test_quantization.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates quantization behavior in SGLang's unit / hardware backend / mlx area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / hardware backend / mlx 领域中与 quantization 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: supporting statements / 辅助语句
```python
"""Unit tests for MLX backend on-the-fly quantization.

Covers:
  - mlx_q4 / mlx_q8 quantize fp16 weights to QuantizedLinear in-place
  - active-memory drops after quantization
  - smoke /generate still works post-quantize
  - pre-quantized HF repos still load (regression guard for mlx_lm passthrough)
  - mlx_q4 flag on an already-quantized model is a no-op (skip + log)

Skips on non-Apple-Silicon platforms and when ``mlx`` / ``mlx_lm`` are missing.
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 13-21: module imports and dependencies / 模块导入与依赖
```python
from __future__ import annotations

import gc
import importlib.util
import platform
import unittest

from sglang.srt.layers.quantization.mlx import MlxQuantizationConfig
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `__future__`, `gc`, `importlib.util`, `platform`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `__future__`, `gc`, `importlib.util`, `platform`。

### Lines 22-26: supporting source context / 辅助源码上下文
```python

# Registered with the CPU suite (runtime no-op marker, parsed via AST).
# On non-Apple-Silicon CI runners the entire TestCase class skips via the
# @skipUnless guard below, so this registration is the harmless "yes this
# test exists" signal the registry requires.
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 27-39: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=10, suite="base-a-test-cpu")

_IS_APPLE_SILICON = platform.system() == "Darwin" and platform.machine() == "arm64"
_HAS_MLX = (
    importlib.util.find_spec("mlx") is not None
    and importlib.util.find_spec("mlx_lm") is not None
)

_SKIP_REASON = "Apple-Silicon-only test (requires Darwin/arm64 + mlx + mlx_lm)"

# Tiny model used across tests; ~0.6B fp16 = ~1.1 GB on disk after first download.
_TEST_MODEL = "Qwen/Qwen3-0.6B"
_TEST_MODEL_PREQUANT = "mlx-community/Qwen3-0.6B-4bit"
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci, system, machine.
**CN:** 该代码块通过 register_cpu_ci, system, machine 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 42-43: class TestMlxQuantization declaration / 类 TestMlxQuantization 声明
```python
@unittest.skipUnless(_IS_APPLE_SILICON and _HAS_MLX, _SKIP_REASON)
class TestMlxQuantization(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 44-44: supporting statements / 辅助语句
```python
    """Smoke tests for --quantization mlx_q4 / mlx_q8 in MlxModelRunner."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 45-47: supporting source context / 辅助源码上下文
```python

    # ---------- helpers ----------

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 48-57: method module counts / 方法 module counts
```python
    @staticmethod
    def _module_counts(model) -> tuple[int, int]:
        n_quant, n_linear = 0, 0
        for _, m in model.named_modules():
            cls = type(m).__name__
            if cls == "QuantizedLinear":
                n_quant += 1
            elif cls == "Linear":
                n_linear += 1
        return n_quant, n_linear
```
**EN:** This block implements `_module_counts` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_module_counts`，承担模块行为中的一个聚焦逻辑片段。

### Lines 59-64: method reset mlx memory / 方法 reset mlx memory
```python
    @staticmethod
    def _reset_mlx_memory() -> None:
        import mlx.core as mx

        gc.collect()
        mx.clear_cache()
```
**EN:** This block implements `_reset_mlx_memory` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_reset_mlx_memory`，承担模块行为中的一个聚焦逻辑片段。

### Lines 66-73: method build runner / 方法 build runner
```python
    def _build_runner(self, model_path: str, quantization: str | None):
        from sglang.srt.hardware_backend.mlx.model_runner import MlxModelRunner

        return MlxModelRunner(
            model_path=model_path,
            quantization=quantization,
            pool_size=1024,  # small pool — these tests don't drive generation depth
        )
```
**EN:** This block implements `_build_runner` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_build_runner`，承担模块行为中的一个聚焦逻辑片段。

### Lines 74-76: supporting source context / 辅助源码上下文
```python

    # ---------- tests ----------

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 77-93: test case mlx q4 creates quantized linear modules / 测试用例 mlx q4 creates quantized linear modules
```python
    def test_mlx_q4_creates_quantized_linear_modules(self):
        """All Linear modules should be QuantizedLinear after mlx_q4 load."""
        self._reset_mlx_memory()
        runner = self._build_runner(_TEST_MODEL, "mlx_q4")
        try:
            n_quant, n_linear = self._module_counts(runner.model)
            self.assertGreater(
                n_quant, 0, "expected at least one QuantizedLinear module"
            )
            self.assertEqual(
                n_linear,
                0,
                f"all Linear modules should have been quantized, got {n_linear} remaining",
            )
        finally:
            del runner
            self._reset_mlx_memory()
```
**EN:** All Linear modules should be QuantizedLinear after mlx_q4 load. This test exercises `test_mlx_q4_creates_quantized_linear_modules` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** All Linear modules should be QuantizedLinear after mlx_q4 load. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mlx_q4_creates_quantized_linear_modules`。

### Lines 95-120: test case mlx q4 reduces memory vs fp16 / 测试用例 mlx q4 reduces memory vs fp16
```python
    def test_mlx_q4_reduces_memory_vs_fp16(self):
        """mlx_q4 should use meaningfully less memory than the fp16 baseline."""
        import mlx.core as mx

        self._reset_mlx_memory()
        runner_fp = self._build_runner(_TEST_MODEL, None)
        mx.eval(runner_fp.model.parameters())
        mem_fp = mx.get_active_memory()
        del runner_fp
        self._reset_mlx_memory()

        runner_q4 = self._build_runner(_TEST_MODEL, "mlx_q4")
        mx.eval(runner_q4.model.parameters())
        mem_q4 = mx.get_active_memory()
        del runner_q4
        self._reset_mlx_memory()

        # Conservative: expect at least 40% reduction. On Qwen3-0.6B we measured ~72%;
        # 40% leaves headroom for different mlx_lm versions, model shapes, etc.
        reduction = 1 - (mem_q4 / max(mem_fp, 1))
        self.assertGreater(
            reduction,
            0.40,
            f"expected >40% memory reduction with mlx_q4, got {reduction*100:.1f}% "
            f"(fp16={mem_fp/1024**3:.2f} GB, q4={mem_q4/1024**3:.2f} GB)",
        )
```
**EN:** mlx_q4 should use meaningfully less memory than the fp16 baseline. This test exercises `test_mlx_q4_reduces_memory_vs_fp16` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** mlx_q4 should use meaningfully less memory than the fp16 baseline. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mlx_q4_reduces_memory_vs_fp16`。

### Lines 122-132: test case mlx q8 creates quantized linear modules / 测试用例 mlx q8 creates quantized linear modules
```python
    def test_mlx_q8_creates_quantized_linear_modules(self):
        """Same check for the 8-bit variant."""
        self._reset_mlx_memory()
        runner = self._build_runner(_TEST_MODEL, "mlx_q8")
        try:
            n_quant, n_linear = self._module_counts(runner.model)
            self.assertGreater(n_quant, 0)
            self.assertEqual(n_linear, 0)
        finally:
            del runner
            self._reset_mlx_memory()
```
**EN:** Same check for the 8-bit variant. This test exercises `test_mlx_q8_creates_quantized_linear_modules` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Same check for the 8-bit variant. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mlx_q8_creates_quantized_linear_modules`。

### Lines 134-156: test case mlx q4 generates text / 测试用例 mlx q4 generates text
```python
    def test_mlx_q4_generates_text(self):
        """After on-the-fly quantization the model must still generate non-empty text."""
        from mlx_lm import generate
        from transformers import AutoTokenizer

        self._reset_mlx_memory()
        runner = self._build_runner(_TEST_MODEL, "mlx_q4")
        try:
            tok = AutoTokenizer.from_pretrained(_TEST_MODEL)
            output = generate(
                runner.model,
                tok,
                prompt="The capital of France is",
                max_tokens=5,
                verbose=False,
            )
            self.assertIsInstance(output, str)
            self.assertGreater(
                len(output.strip()), 0, "generation returned empty string"
            )
        finally:
            del runner
            self._reset_mlx_memory()
```
**EN:** After on-the-fly quantization the model must still generate non-empty text. This test exercises `test_mlx_q4_generates_text` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** After on-the-fly quantization the model must still generate non-empty text. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mlx_q4_generates_text`。

### Lines 158-173: test case pre quantized hf repo passthrough / 测试用例 pre quantized hf repo passthrough
```python
    def test_pre_quantized_hf_repo_passthrough(self):
        """Loading mlx-community/<model>-4bit must still work (mlx_lm passthrough,
        regression guard for the no-quantization-flag path).
        """
        self._reset_mlx_memory()
        runner = self._build_runner(_TEST_MODEL_PREQUANT, quantization=None)
        try:
            n_quant, n_linear = self._module_counts(runner.model)
            self.assertGreater(
                n_quant,
                0,
                "pre-quantized HF repo should load as QuantizedLinear without --quantization",
            )
        finally:
            del runner
            self._reset_mlx_memory()
```
**EN:** Loading mlx-community/<model>-4bit must still work (mlx_lm passthrough, regression guard for the no-quantization-flag path). This test exercises `test_pre_quantized_hf_repo_passthrough` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Loading mlx-community/<model>-4bit must still work (mlx_lm passthrough, regression guard for the no-quantization-flag path). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_pre_quantized_hf_repo_passthrough`。

### Lines 175-187: test case quantize flag on already quantized model is noop / 测试用例 quantize flag on already quantized model is noop
```python
    def test_quantize_flag_on_already_quantized_model_is_noop(self):
        """Passing --quantization mlx_q4 on a pre-quantized repo should NOT double-quantize."""
        self._reset_mlx_memory()
        # Using mlx_q4 against an already-q4 repo. The runner logs the skip and leaves
        # the existing QuantizedLinear modules untouched.
        runner = self._build_runner(_TEST_MODEL_PREQUANT, "mlx_q4")
        try:
            n_quant, n_linear = self._module_counts(runner.model)
            self.assertGreater(n_quant, 0)
            self.assertEqual(n_linear, 0)
        finally:
            del runner
            self._reset_mlx_memory()
```
**EN:** Passing --quantization mlx_q4 on a pre-quantized repo should NOT double-quantize. This test exercises `test_quantize_flag_on_already_quantized_model_is_noop` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Passing --quantization mlx_q4 on a pre-quantized repo should NOT double-quantize. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_quantize_flag_on_already_quantized_model_is_noop`。

### Lines 190-190: class TestMlxQuantizationOverride declaration / 类 TestMlxQuantizationOverride 声明
```python
class TestMlxQuantizationOverride(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 191-195: supporting statements / 辅助语句
```python
    """Pure-logic tests for ``MlxQuantizationConfig.override_quantization_method``.

    The override is a classmethod over a dict; no mlx / Apple Silicon
    dependency. Runs on every CI platform and guards #25119 from regression.
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 197-202: test case mlx q4 dict config autodetect / 测试用例 mlx q4 dict config autodetect
```python
    def test_mlx_q4_dict_config_autodetect(self):
        """Bare {group_size, bits=4} dict maps to mlx_q4."""
        result = MlxQuantizationConfig.override_quantization_method(
            {"group_size": 64, "bits": 4}, None
        )
        self.assertEqual(result, "mlx_q4")
```
**EN:** Bare {group_size, bits=4} dict maps to mlx_q4. This test exercises `test_mlx_q4_dict_config_autodetect` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Bare {group_size, bits=4} dict maps to mlx_q4. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mlx_q4_dict_config_autodetect`。

### Lines 204-209: test case mlx q8 dict config autodetect / 测试用例 mlx q8 dict config autodetect
```python
    def test_mlx_q8_dict_config_autodetect(self):
        """Bare {group_size, bits=8} dict maps to mlx_q8."""
        result = MlxQuantizationConfig.override_quantization_method(
            {"group_size": 32, "bits": 8}, None
        )
        self.assertEqual(result, "mlx_q8")
```
**EN:** Bare {group_size, bits=8} dict maps to mlx_q8. This test exercises `test_mlx_q8_dict_config_autodetect` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Bare {group_size, bits=8} dict maps to mlx_q8. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mlx_q8_dict_config_autodetect`。

### Lines 211-224: test case non mlx dict not matched / 测试用例 non mlx dict not matched
```python
    def test_non_mlx_dict_not_matched(self):
        """Dicts with an explicit quant_method belong to that method, not ours."""
        # modelopt-style: explicit quant_method takes priority.
        self.assertIsNone(
            MlxQuantizationConfig.override_quantization_method(
                {"quant_method": "modelopt", "bits": 4, "group_size": 64}, None
            )
        )
        # gptq-style: same.
        self.assertIsNone(
            MlxQuantizationConfig.override_quantization_method(
                {"quant_method": "gptq", "bits": 4, "group_size": 128}, None
            )
        )
```
**EN:** Dicts with an explicit quant_method belong to that method, not ours. This test exercises `test_non_mlx_dict_not_matched` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Dicts with an explicit quant_method belong to that method, not ours. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_non_mlx_dict_not_matched`。

### Lines 226-253: test case non dict not matched / 测试用例 non dict not matched
```python
    def test_non_dict_not_matched(self):
        """Non-dict inputs and malformed dicts return None."""
        # None / string inputs.
        self.assertIsNone(
            MlxQuantizationConfig.override_quantization_method(None, None)
        )
        self.assertIsNone(
            MlxQuantizationConfig.override_quantization_method("mlx_q4", None)
        )
        # Missing keys.
        self.assertIsNone(
            MlxQuantizationConfig.override_quantization_method({"bits": 4}, None)
        )
        self.assertIsNone(
            MlxQuantizationConfig.override_quantization_method({"group_size": 64}, None)
        )
        # Non-integer values.
        self.assertIsNone(
            MlxQuantizationConfig.override_quantization_method(
                {"bits": "4", "group_size": 64}, None
            )
        )
        # Unsupported bit-width.
        self.assertIsNone(
            MlxQuantizationConfig.override_quantization_method(
                {"bits": 2, "group_size": 64}, None
            )
        )
```
**EN:** Non-dict inputs and malformed dicts return None. This test exercises `test_non_dict_not_matched` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Non-dict inputs and malformed dicts return None. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_non_dict_not_matched`。

### Lines 255-274: test case user quant explicit defers to user / 测试用例 user quant explicit defers to user
```python
    def test_user_quant_explicit_defers_to_user(self):
        """When the user passes --quantization explicitly, defer to that choice."""
        # User chose mlx_q8 explicitly, even though config dict shape suggests q4
        self.assertIsNone(
            MlxQuantizationConfig.override_quantization_method(
                {"group_size": 64, "bits": 4}, "mlx_q8"
            )
        )
        # User chose mlx_q4 explicitly with matching config
        self.assertIsNone(
            MlxQuantizationConfig.override_quantization_method(
                {"group_size": 64, "bits": 4}, "mlx_q4"
            )
        )
        # User chose something completely different
        self.assertIsNone(
            MlxQuantizationConfig.override_quantization_method(
                {"group_size": 64, "bits": 4}, "fp8"
            )
        )
```
**EN:** When the user passes --quantization explicitly, defer to that choice. This test exercises `test_user_quant_explicit_defers_to_user` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** When the user passes --quantization explicitly, defer to that choice. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_user_quant_explicit_defers_to_user`。

### Lines 277-278: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestMlxQuantization`: Smoke tests for --quantization mlx_q4 / mlx_q8 in MlxModelRunner. / 用于组织相关测试、夹具或辅助方法。
- `TestMlxQuantizationOverride`: Pure-logic tests for ``MlxQuantizationConfig.override_quantization_method``. / 用于组织相关测试、夹具或辅助方法。
- `TestMlxQuantization._module_counts`: This block implements `_module_counts` and captures one focused piece of the module's behavior. / 该代码块实现 `_module_counts`，承担模块行为中的一个聚焦逻辑片段。
- `TestMlxQuantization._reset_mlx_memory`: This block implements `_reset_mlx_memory` and captures one focused piece of the module's behavior. / 该代码块实现 `_reset_mlx_memory`，承担模块行为中的一个聚焦逻辑片段。
- `TestMlxQuantization._build_runner`: This block implements `_build_runner` and captures one focused piece of the module's behavior. / 该代码块实现 `_build_runner`，承担模块行为中的一个聚焦逻辑片段。
- `TestMlxQuantization.test_mlx_q4_creates_quantized_linear_modules`: All Linear modules should be QuantizedLinear after mlx_q4 load. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mlx_q4_creates_quantized_linear_modules`。
- `TestMlxQuantization.test_mlx_q4_reduces_memory_vs_fp16`: mlx_q4 should use meaningfully less memory than the fp16 baseline. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mlx_q4_reduces_memory_vs_fp16`。
- `TestMlxQuantization.test_mlx_q8_creates_quantized_linear_modules`: Same check for the 8-bit variant. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mlx_q8_creates_quantized_linear_modules`。
- `TestMlxQuantization.test_mlx_q4_generates_text`: After on-the-fly quantization the model must still generate non-empty text. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mlx_q4_generates_text`。
- `TestMlxQuantization.test_pre_quantized_hf_repo_passthrough`: Loading mlx-community/<model>-4bit must still work (mlx_lm passthrough, regression guard for the no-quantization-flag path). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_pre_quantized_hf_repo_passthrough`。
- `TestMlxQuantization.test_quantize_flag_on_already_quantized_model_is_noop`: Passing --quantization mlx_q4 on a pre-quantized repo should NOT double-quantize. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_quantize_flag_on_already_quantized_model_is_noop`。
- `TestMlxQuantizationOverride.test_mlx_q4_dict_config_autodetect`: Bare {group_size, bits=4} dict maps to mlx_q4. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mlx_q4_dict_config_autodetect`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `gc`, `importlib.util`, `platform`, `unittest`
- **Internal modules / 内部模块**: `sglang.srt.layers.quantization.mlx`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 278
