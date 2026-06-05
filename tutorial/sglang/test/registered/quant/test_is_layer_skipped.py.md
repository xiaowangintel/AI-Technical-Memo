# test_is_layer_skipped.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/quant/test_is_layer_skipped.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates is layer skipped behavior in SGLang's quant area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 quant 领域中与 is layer skipped 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: module imports and dependencies / 模块导入与依赖
```python
import unittest

from sglang.srt.layers.quantization.utils import is_layer_skipped
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `sglang.srt.layers.quantization.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `sglang.srt.layers.quantization.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`。

### Lines 7-16: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=5, suite="base-a-test-cpu")


# Qwen3-Next FP8 actually publishes the equivalent of this in
# packed_modules_mapping (qwen3_next.py:908-911). in_proj_ba / in_proj_qkvz
# are deliberately omitted because they are real, unified tensors.
QWEN3_NEXT_FUSED_MAPPING = {
    "qkv_proj": ["q_proj", "k_proj", "v_proj"],
    "gate_up_proj": ["gate_proj", "up_proj"],
}
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 19-23: function qwen3 next ignored layers / 函数 qwen3 next ignored layers
```python
def _qwen3_next_ignored_layers(layer_idx: int, name: str) -> list:
    # Mirrors the normalization in Fp8Config.from_config: each entry is kept in
    # both "model.<...>" and bare "<...>" forms.
    base = f"layers.{layer_idx}.linear_attn.{name}"
    return [base, f"model.{base}"]
```
**EN:** This block implements `_qwen3_next_ignored_layers` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_qwen3_next_ignored_layers`，承担模块行为中的一个聚焦逻辑片段。

### Lines 26-26: class TestIsLayerSkipped declaration / 类 TestIsLayerSkipped 声明
```python
class TestIsLayerSkipped(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 27-34: test case qwen3 next in proj ba is skipped / 测试用例 qwen3 next in proj ba is skipped
```python
    def test_qwen3_next_in_proj_ba_is_skipped(self):
        # Regression for #23467: in_proj_ba is a unified tensor in the FP8
        # checkpoint. modules_to_not_convert lists it explicitly, so it must
        # bypass FP8 quantization (otherwise validate_block_quant_shapes raises
        # on output_partition_size=8 vs block_n=128 at tp=4).
        prefix = "model.layers.0.linear_attn.in_proj_ba"
        ignored = _qwen3_next_ignored_layers(0, "in_proj_ba")
        self.assertTrue(is_layer_skipped(prefix, ignored, QWEN3_NEXT_FUSED_MAPPING))
```
**EN:** This test exercises `test_qwen3_next_in_proj_ba_is_skipped` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_qwen3_next_in_proj_ba_is_skipped`。

### Lines 36-39: test case qwen3 next in proj qkvz is skipped / 测试用例 qwen3 next in proj qkvz is skipped
```python
    def test_qwen3_next_in_proj_qkvz_is_skipped(self):
        prefix = "model.layers.5.linear_attn.in_proj_qkvz"
        ignored = _qwen3_next_ignored_layers(5, "in_proj_qkvz")
        self.assertTrue(is_layer_skipped(prefix, ignored, QWEN3_NEXT_FUSED_MAPPING))
```
**EN:** This test exercises `test_qwen3_next_in_proj_qkvz_is_skipped` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_qwen3_next_in_proj_qkvz_is_skipped`。

### Lines 41-48: test case mlp gate does not match gate up proj / 测试用例 mlp gate does not match gate up proj
```python
    def test_mlp_gate_does_not_match_gate_up_proj(self):
        # The motivation for #23467: an entry "mlp.gate" in
        # modules_to_not_convert must NOT skip a sibling "mlp.gate_up_proj".
        ignored = ["mlp.gate"]
        self.assertFalse(
            is_layer_skipped("model.layers.0.mlp.gate_up_proj", ignored, {})
        )
        self.assertTrue(is_layer_skipped("model.layers.0.mlp.gate", ignored, {}))
```
**EN:** This test exercises `test_mlp_gate_does_not_match_gate_up_proj` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mlp_gate_does_not_match_gate_up_proj`。

### Lines 51-52: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_qwen3_next_ignored_layers`: This block implements `_qwen3_next_ignored_layers` and captures one focused piece of the module's behavior. / 该代码块实现 `_qwen3_next_ignored_layers`，承担模块行为中的一个聚焦逻辑片段。
- `TestIsLayerSkipped`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestIsLayerSkipped.test_qwen3_next_in_proj_ba_is_skipped`: This test exercises `test_qwen3_next_in_proj_ba_is_skipped` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_qwen3_next_in_proj_ba_is_skipped`。
- `TestIsLayerSkipped.test_qwen3_next_in_proj_qkvz_is_skipped`: This test exercises `test_qwen3_next_in_proj_qkvz_is_skipped` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_qwen3_next_in_proj_qkvz_is_skipped`。
- `TestIsLayerSkipped.test_mlp_gate_does_not_match_gate_up_proj`: This test exercises `test_mlp_gate_does_not_match_gate_up_proj` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mlp_gate_does_not_match_gate_up_proj`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Internal modules / 内部模块**: `sglang.srt.layers.quantization.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 52
