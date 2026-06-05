# test_conv_layer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/layers/test_conv_layer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates conv layer behavior in SGLang's unit / layers area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 层 领域中与 conv layer 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: module imports and dependencies / 模块导入与依赖
```python
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.test.ci.ci_register`。

### Lines 3-4: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=7, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=7, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 6-11: module imports and dependencies / 模块导入与依赖
```python
import unittest

import torch
import torch.nn as nn

from sglang.srt.layers.conv import Conv2dLayer, Conv3dLayer
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `torch`, `torch.nn`, `sglang.srt.layers.conv`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `torch`, `torch.nn`, `sglang.srt.layers.conv`。

### Lines 14-19: function copy weights / 函数 copy weights
```python
def _copy_weights(src, dst_nn):
    """Copy weights from Conv*dLayer to nn.Conv*d for comparison."""
    with torch.no_grad():
        dst_nn.weight.copy_(src.weight)
        if src.bias is not None:
            dst_nn.bias.copy_(src.bias)
```
**EN:** Copy weights from Conv*dLayer to nn.Conv*d for comparison. This block implements `_copy_weights` and captures one focused piece of the module's behavior.
**CN:** Copy weights from Conv*dLayer to nn.Conv*d for comparison. 该代码块实现 `_copy_weights`，承担模块行为中的一个聚焦逻辑片段。

### Lines 22-23: class TestConv2dLayer declaration / 类 TestConv2dLayer 声明
```python
class TestConv2dLayer(unittest.TestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 24-33: test case basic patch embedding / 测试用例 basic patch embedding
```python
    def test_basic_patch_embedding(self):
        layer = Conv2dLayer(3, 768, kernel_size=14, stride=14, bias=False)
        ref = nn.Conv2d(3, 768, kernel_size=14, stride=14, bias=False)
        self.assertFalse(layer.enable_linear)
        _copy_weights(layer, ref)
        x = torch.randn(2, 3, 224, 224)
        with torch.no_grad():
            torch.testing.assert_close(
                layer.forward_native(x), ref(x), rtol=1e-4, atol=1e-4
            )
```
**EN:** This test exercises `test_basic_patch_embedding` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_basic_patch_embedding`。

### Lines 35-46: test case enable linear / 测试用例 enable linear
```python
    def test_enable_linear(self):
        layer = Conv2dLayer(
            3, 768, kernel_size=14, stride=14, bias=True, disable_linear=False
        )
        ref = nn.Conv2d(3, 768, kernel_size=14, stride=14, bias=True)
        self.assertTrue(layer.enable_linear)
        _copy_weights(layer, ref)
        x = torch.randn(1, 3, 224, 224)
        with torch.no_grad():
            torch.testing.assert_close(
                layer.forward_native(x), ref(x), rtol=1e-4, atol=1e-4
            )
```
**EN:** This test exercises `test_enable_linear` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_enable_linear`。

### Lines 48-51: test case padding valid / 测试用例 padding valid
```python
    def test_padding_valid(self):
        layer = Conv2dLayer(3, 768, kernel_size=14, stride=14, padding="valid")
        self.assertFalse(layer.enable_linear)
        self.assertEqual(layer.padding, (0, 0))
```
**EN:** This test exercises `test_padding_valid` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_padding_valid`。

### Lines 53-55: test case padding same disables linear / 测试用例 padding same disables linear
```python
    def test_padding_same_disables_linear(self):
        layer = Conv2dLayer(3, 64, kernel_size=3, stride=1, padding="same")
        self.assertFalse(layer.enable_linear)
```
**EN:** This test exercises `test_padding_same_disables_linear` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_padding_same_disables_linear`。

### Lines 57-59: test case non matching stride disables linear / 测试用例 non matching stride disables linear
```python
    def test_non_matching_stride_disables_linear(self):
        layer = Conv2dLayer(3, 64, kernel_size=3, stride=1, padding=1)
        self.assertFalse(layer.enable_linear)
```
**EN:** This test exercises `test_non_matching_stride_disables_linear` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_non_matching_stride_disables_linear`。

### Lines 61-63: test case groups disable linear / 测试用例 groups disable linear
```python
    def test_groups_disable_linear(self):
        layer = Conv2dLayer(4, 8, kernel_size=2, stride=2, groups=2)
        self.assertFalse(layer.enable_linear)
```
**EN:** This test exercises `test_groups_disable_linear` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_groups_disable_linear`。

### Lines 65-67: test case default disables linear / 测试用例 default disables linear
```python
    def test_default_disables_linear(self):
        layer = Conv2dLayer(3, 768, kernel_size=14, stride=14)
        self.assertFalse(layer.enable_linear)
```
**EN:** This test exercises `test_default_disables_linear` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_disables_linear`。

### Lines 69-71: test case dilation disables linear / 测试用例 dilation disables linear
```python
    def test_dilation_disables_linear(self):
        layer = Conv2dLayer(3, 64, kernel_size=3, stride=3, dilation=2)
        self.assertFalse(layer.enable_linear)
```
**EN:** This test exercises `test_dilation_disables_linear` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dilation_disables_linear`。

### Lines 73-86: test case padding mode reflect / 测试用例 padding mode reflect
```python
    def test_padding_mode_reflect(self):
        layer = Conv2dLayer(
            3, 64, kernel_size=3, stride=1, padding=1, padding_mode="reflect", bias=True
        )
        ref = nn.Conv2d(
            3, 64, kernel_size=3, stride=1, padding=1, padding_mode="reflect", bias=True
        )
        self.assertFalse(layer.enable_linear)
        _copy_weights(layer, ref)
        x = torch.randn(1, 3, 16, 16)
        with torch.no_grad():
            torch.testing.assert_close(
                layer.forward_native(x), ref(x), rtol=1e-4, atol=1e-4
            )
```
**EN:** This test exercises `test_padding_mode_reflect` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_padding_mode_reflect`。

### Lines 88-96: test case conv path with padding / 测试用例 conv path with padding
```python
    def test_conv_path_with_padding(self):
        layer = Conv2dLayer(3, 64, kernel_size=3, stride=1, padding=1, bias=True)
        ref = nn.Conv2d(3, 64, kernel_size=3, stride=1, padding=1, bias=True)
        _copy_weights(layer, ref)
        x = torch.randn(1, 3, 32, 32)
        with torch.no_grad():
            torch.testing.assert_close(
                layer.forward_native(x), ref(x), rtol=1e-4, atol=1e-4
            )
```
**EN:** This test exercises `test_conv_path_with_padding` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_conv_path_with_padding`。

### Lines 98-110: test case mulmat matches conv / 测试用例 mulmat matches conv
```python
    def test_mulmat_matches_conv(self):
        layer = Conv2dLayer(
            3, 768, kernel_size=14, stride=14, bias=True, disable_linear=False
        )
        self.assertTrue(layer.enable_linear)
        x = torch.randn(2, 3, 224, 224)
        with torch.no_grad():
            torch.testing.assert_close(
                layer._forward_mulmat(x),
                layer._forward_conv(x),
                rtol=1e-4,
                atol=1e-4,
            )
```
**EN:** This test exercises `test_mulmat_matches_conv` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mulmat_matches_conv`。

### Lines 112-119: test case forward cuda uses mulmat when enabled / 测试用例 forward cuda uses mulmat when enabled
```python
    def test_forward_cuda_uses_mulmat_when_enabled(self):
        layer = Conv2dLayer(
            3, 64, kernel_size=4, stride=4, bias=False, disable_linear=False
        )
        self.assertTrue(layer.enable_linear)
        x = torch.randn(1, 3, 16, 16)
        with torch.no_grad():
            torch.testing.assert_close(layer.forward_cuda(x), layer._forward_mulmat(x))
```
**EN:** This test exercises `test_forward_cuda_uses_mulmat_when_enabled` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_forward_cuda_uses_mulmat_when_enabled`。

### Lines 121-126: test case forward cuda uses conv when not eligible / 测试用例 forward cuda uses conv when not eligible
```python
    def test_forward_cuda_uses_conv_when_not_eligible(self):
        layer = Conv2dLayer(3, 64, kernel_size=3, stride=1, padding=1, bias=False)
        self.assertFalse(layer.enable_linear)
        x = torch.randn(1, 3, 16, 16)
        with torch.no_grad():
            torch.testing.assert_close(layer.forward_cuda(x), layer._forward_conv(x))
```
**EN:** This test exercises `test_forward_cuda_uses_conv_when_not_eligible` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_forward_cuda_uses_conv_when_not_eligible`。

### Lines 128-144: test case tuple kernel size / 测试用例 tuple kernel size
```python
    def test_tuple_kernel_size(self):
        layer = Conv2dLayer(
            3,
            768,
            kernel_size=(14, 14),
            stride=(14, 14),
            bias=False,
            disable_linear=False,
        )
        self.assertTrue(layer.enable_linear)
        ref = nn.Conv2d(3, 768, kernel_size=(14, 14), stride=(14, 14), bias=False)
        _copy_weights(layer, ref)
        x = torch.randn(1, 3, 224, 224)
        with torch.no_grad():
            torch.testing.assert_close(
                layer.forward_native(x), ref(x), rtol=1e-4, atol=1e-4
            )
```
**EN:** This test exercises `test_tuple_kernel_size` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tuple_kernel_size`。

### Lines 146-150: test case output shape / 测试用例 output shape
```python
    def test_output_shape(self):
        layer = Conv2dLayer(3, 768, kernel_size=16, stride=16, bias=False)
        x = torch.randn(4, 3, 224, 224)
        out = layer.forward_native(x)
        self.assertEqual(out.shape, (4, 768, 14, 14))
```
**EN:** This test exercises `test_output_shape` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_output_shape`。

### Lines 152-154: test case no bias parameter / 测试用例 no bias parameter
```python
    def test_no_bias_parameter(self):
        layer = Conv2dLayer(3, 64, kernel_size=4, stride=4, bias=False)
        self.assertIsNone(layer.bias)
```
**EN:** This test exercises `test_no_bias_parameter` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_bias_parameter`。

### Lines 157-158: class TestConvValidation declaration / 类 TestConvValidation 声明
```python
class TestConvValidation(unittest.TestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 159-161: test case in channels not divisible by groups / 测试用例 in channels not divisible by groups
```python
    def test_in_channels_not_divisible_by_groups(self):
        with self.assertRaises(ValueError):
            Conv2dLayer(3, 64, kernel_size=3, stride=1, groups=2)
```
**EN:** This test exercises `test_in_channels_not_divisible_by_groups` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_in_channels_not_divisible_by_groups`。

### Lines 163-165: test case out channels not divisible by groups / 测试用例 out channels not divisible by groups
```python
    def test_out_channels_not_divisible_by_groups(self):
        with self.assertRaises(ValueError):
            Conv2dLayer(4, 6, kernel_size=3, stride=1, groups=4)
```
**EN:** This test exercises `test_out_channels_not_divisible_by_groups` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_out_channels_not_divisible_by_groups`。

### Lines 167-169: test case invalid padding string / 测试用例 invalid padding string
```python
    def test_invalid_padding_string(self):
        with self.assertRaises(ValueError):
            Conv2dLayer(3, 64, kernel_size=3, stride=1, padding="full")
```
**EN:** This test exercises `test_invalid_padding_string` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_invalid_padding_string`。

### Lines 171-173: test case padding same with stride / 测试用例 padding same with stride
```python
    def test_padding_same_with_stride(self):
        with self.assertRaises(ValueError):
            Conv2dLayer(3, 64, kernel_size=3, stride=2, padding="same")
```
**EN:** This test exercises `test_padding_same_with_stride` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_padding_same_with_stride`。

### Lines 175-200: test case padding same with non zeros padding mode / 测试用例 padding same with non zeros padding mode
```python
    def test_padding_same_with_non_zeros_padding_mode(self):
        layer = Conv2dLayer(
            3,
            64,
            kernel_size=3,
            stride=1,
            padding="same",
            padding_mode="reflect",
            bias=True,
        )
        ref = nn.Conv2d(
            3,
            64,
            kernel_size=3,
            stride=1,
            padding="same",
            padding_mode="reflect",
            bias=True,
        )
        self.assertFalse(layer.enable_linear)
        _copy_weights(layer, ref)
        x = torch.randn(1, 3, 16, 16)
        with torch.no_grad():
            torch.testing.assert_close(
                layer.forward_native(x), ref(x), rtol=1e-4, atol=1e-4
            )
```
**EN:** This test exercises `test_padding_same_with_non_zeros_padding_mode` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_padding_same_with_non_zeros_padding_mode`。

### Lines 202-204: test case invalid padding mode / 测试用例 invalid padding mode
```python
    def test_invalid_padding_mode(self):
        with self.assertRaises(ValueError):
            Conv3dLayer(3, 64, kernel_size=3, stride=1, padding_mode="invalid")
```
**EN:** This test exercises `test_invalid_padding_mode` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_invalid_padding_mode`。

### Lines 206-208: test case conv3d in channels not divisible by groups / 测试用例 conv3d in channels not divisible by groups
```python
    def test_conv3d_in_channels_not_divisible_by_groups(self):
        with self.assertRaises(ValueError):
            Conv3dLayer(3, 64, kernel_size=3, stride=1, groups=2)
```
**EN:** This test exercises `test_conv3d_in_channels_not_divisible_by_groups` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_conv3d_in_channels_not_divisible_by_groups`。

### Lines 211-212: class TestConv3dLayer declaration / 类 TestConv3dLayer 声明
```python
class TestConv3dLayer(unittest.TestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 213-226: test case basic temporal patch embedding / 测试用例 basic temporal patch embedding
```python
    def test_basic_temporal_patch_embedding(self):
        layer = Conv3dLayer(
            3, 1152, kernel_size=[2, 14, 14], stride=[2, 14, 14], bias=False
        )
        ref = nn.Conv3d(
            3, 1152, kernel_size=[2, 14, 14], stride=[2, 14, 14], bias=False
        )
        self.assertTrue(layer.enable_linear)
        _copy_weights(layer, ref)
        x = torch.randn(1, 3, 2, 14, 14)
        with torch.no_grad():
            torch.testing.assert_close(
                layer.forward_native(x), ref(x), rtol=1e-4, atol=1e-4
            )
```
**EN:** This test exercises `test_basic_temporal_patch_embedding` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_basic_temporal_patch_embedding`。

### Lines 228-239: test case with bias / 测试用例 with bias
```python
    def test_with_bias(self):
        layer = Conv3dLayer(
            3, 1536, kernel_size=[2, 14, 14], stride=[2, 14, 14], bias=True
        )
        ref = nn.Conv3d(3, 1536, kernel_size=[2, 14, 14], stride=[2, 14, 14], bias=True)
        self.assertTrue(layer.enable_linear)
        _copy_weights(layer, ref)
        x = torch.randn(4, 3, 2, 14, 14)
        with torch.no_grad():
            torch.testing.assert_close(
                layer.forward_native(x), ref(x), rtol=1e-4, atol=1e-4
            )
```
**EN:** This test exercises `test_with_bias` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_with_bias`。

### Lines 241-253: test case mulmat matches conv / 测试用例 mulmat matches conv
```python
    def test_mulmat_matches_conv(self):
        layer = Conv3dLayer(
            3, 1152, kernel_size=[2, 14, 14], stride=[2, 14, 14], bias=True
        )
        self.assertTrue(layer.enable_linear)
        x = torch.randn(2, 3, 2, 14, 14)
        with torch.no_grad():
            torch.testing.assert_close(
                layer._forward_mulmat(x),
                layer._forward_conv(x),
                rtol=1e-4,
                atol=1e-4,
            )
```
**EN:** This test exercises `test_mulmat_matches_conv` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mulmat_matches_conv`。

### Lines 255-257: test case non matching stride disables linear / 测试用例 non matching stride disables linear
```python
    def test_non_matching_stride_disables_linear(self):
        layer = Conv3dLayer(3, 64, kernel_size=3, stride=1, padding=1)
        self.assertFalse(layer.enable_linear)
```
**EN:** This test exercises `test_non_matching_stride_disables_linear` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_non_matching_stride_disables_linear`。

### Lines 259-261: test case dilation disables linear / 测试用例 dilation disables linear
```python
    def test_dilation_disables_linear(self):
        layer = Conv3dLayer(3, 64, kernel_size=3, stride=3, dilation=2)
        self.assertFalse(layer.enable_linear)
```
**EN:** This test exercises `test_dilation_disables_linear` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dilation_disables_linear`。

### Lines 263-281: test case disable linear / 测试用例 disable linear
```python
    def test_disable_linear(self):
        layer = Conv3dLayer(
            3,
            1152,
            kernel_size=[2, 14, 14],
            stride=[2, 14, 14],
            bias=False,
            disable_linear=True,
        )
        self.assertFalse(layer.enable_linear)
        ref = nn.Conv3d(
            3, 1152, kernel_size=[2, 14, 14], stride=[2, 14, 14], bias=False
        )
        _copy_weights(layer, ref)
        x = torch.randn(1, 3, 2, 14, 14)
        with torch.no_grad():
            torch.testing.assert_close(
                layer.forward_native(x), ref(x), rtol=1e-4, atol=1e-4
            )
```
**EN:** This test exercises `test_disable_linear` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_disable_linear`。

### Lines 283-291: test case conv path with padding / 测试用例 conv path with padding
```python
    def test_conv_path_with_padding(self):
        layer = Conv3dLayer(3, 64, kernel_size=3, stride=1, padding=1, bias=True)
        ref = nn.Conv3d(3, 64, kernel_size=3, stride=1, padding=1, bias=True)
        _copy_weights(layer, ref)
        x = torch.randn(1, 3, 4, 8, 8)
        with torch.no_grad():
            torch.testing.assert_close(
                layer.forward_native(x), ref(x), rtol=1e-4, atol=1e-4
            )
```
**EN:** This test exercises `test_conv_path_with_padding` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_conv_path_with_padding`。

### Lines 293-299: test case output shape / 测试用例 output shape
```python
    def test_output_shape(self):
        layer = Conv3dLayer(
            3, 1152, kernel_size=[2, 14, 14], stride=[2, 14, 14], bias=False
        )
        x = torch.randn(1, 3, 2, 14, 14)
        out = layer.forward_native(x)
        self.assertEqual(out.shape, (1, 1152, 1, 1, 1))
```
**EN:** This test exercises `test_output_shape` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_output_shape`。

### Lines 301-311: test case batch processing / 测试用例 batch processing
```python
    def test_batch_processing(self):
        layer = Conv3dLayer(
            3, 1536, kernel_size=[2, 14, 14], stride=[2, 14, 14], bias=True
        )
        ref = nn.Conv3d(3, 1536, kernel_size=[2, 14, 14], stride=[2, 14, 14], bias=True)
        _copy_weights(layer, ref)
        x = torch.randn(8, 3, 2, 14, 14)
        with torch.no_grad():
            torch.testing.assert_close(
                layer.forward_native(x), ref(x), rtol=1e-4, atol=1e-4
            )
```
**EN:** This test exercises `test_batch_processing` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_batch_processing`。

### Lines 313-320: test case forward native uses mulmat when eligible / 测试用例 forward native uses mulmat when eligible
```python
    def test_forward_native_uses_mulmat_when_eligible(self):
        layer = Conv3dLayer(3, 128, kernel_size=[2, 4, 4], stride=[2, 4, 4], bias=True)
        self.assertTrue(layer.enable_linear)
        x = torch.randn(1, 3, 2, 4, 4)
        with torch.no_grad():
            torch.testing.assert_close(
                layer.forward_native(x), layer._forward_mulmat(x)
            )
```
**EN:** This test exercises `test_forward_native_uses_mulmat_when_eligible` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_forward_native_uses_mulmat_when_eligible`。

### Lines 322-327: test case padding valid / 测试用例 padding valid
```python
    def test_padding_valid(self):
        layer = Conv3dLayer(
            3, 64, kernel_size=[2, 4, 4], stride=[2, 4, 4], padding="valid"
        )
        self.assertTrue(layer.enable_linear)
        self.assertEqual(layer.padding, (0, 0, 0))
```
**EN:** This test exercises `test_padding_valid` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_padding_valid`。

### Lines 329-333: test case weight shape / 测试用例 weight shape
```python
    def test_weight_shape(self):
        layer = Conv3dLayer(
            3, 1152, kernel_size=[2, 14, 14], stride=[2, 14, 14], bias=False
        )
        self.assertEqual(layer.weight.shape, (1152, 3, 2, 14, 14))
```
**EN:** This test exercises `test_weight_shape` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_weight_shape`。

### Lines 335-364: test case glm4v workflow / 测试用例 glm4v workflow
```python
    def test_glm4v_workflow(self):
        """GLM4V-style: 2D input -> reshape to 5D -> Conv3dLayer -> flatten."""
        in_channels, temporal_patch_size, patch_size = 3, 2, 14
        hidden_size = 1536
        layer = Conv3dLayer(
            in_channels,
            hidden_size,
            kernel_size=[temporal_patch_size, patch_size, patch_size],
            stride=[temporal_patch_size, patch_size, patch_size],
            bias=True,
        )
        ref = nn.Conv3d(
            in_channels,
            hidden_size,
            kernel_size=[temporal_patch_size, patch_size, patch_size],
            stride=[temporal_patch_size, patch_size, patch_size],
            bias=True,
        )
        _copy_weights(layer, ref)
        num_patches = 4
        flat_dim = in_channels * temporal_patch_size * patch_size * patch_size
        x_2d = torch.randn(num_patches, flat_dim)
        x_5d = x_2d.view(-1, in_channels, temporal_patch_size, patch_size, patch_size)
        with torch.no_grad():
            torch.testing.assert_close(
                layer.forward_native(x_5d).view(-1, hidden_size),
                ref(x_5d).view(-1, hidden_size),
                rtol=1e-4,
                atol=1e-4,
            )
```
**EN:** GLM4V-style: 2D input -> reshape to 5D -> Conv3dLayer -> flatten. This test exercises `test_glm4v_workflow` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** GLM4V-style: 2D input -> reshape to 5D -> Conv3dLayer -> flatten. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_glm4v_workflow`。

### Lines 367-368: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_copy_weights`: Copy weights from Conv*dLayer to nn.Conv*d for comparison. / 该代码块实现 `_copy_weights`，承担模块行为中的一个聚焦逻辑片段。
- `TestConv2dLayer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestConvValidation`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestConv3dLayer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestConv2dLayer.test_basic_patch_embedding`: This test exercises `test_basic_patch_embedding` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_basic_patch_embedding`。
- `TestConv2dLayer.test_enable_linear`: This test exercises `test_enable_linear` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_enable_linear`。
- `TestConv2dLayer.test_padding_valid`: This test exercises `test_padding_valid` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_padding_valid`。
- `TestConv2dLayer.test_padding_same_disables_linear`: This test exercises `test_padding_same_disables_linear` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_padding_same_disables_linear`。
- `TestConv2dLayer.test_non_matching_stride_disables_linear`: This test exercises `test_non_matching_stride_disables_linear` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_non_matching_stride_disables_linear`。
- `TestConv2dLayer.test_groups_disable_linear`: This test exercises `test_groups_disable_linear` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_groups_disable_linear`。
- `TestConv2dLayer.test_default_disables_linear`: This test exercises `test_default_disables_linear` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_default_disables_linear`。
- `TestConv2dLayer.test_dilation_disables_linear`: This test exercises `test_dilation_disables_linear` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dilation_disables_linear`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `torch`, `torch.nn`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.srt.layers.conv`

- **Total lines / 总行数**: 368
