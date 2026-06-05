# test_topk.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/srt/cpu/test_topk.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates topk behavior in SGLang's srt / cpu area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 SRT / CPU 领域中与 topk 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: module imports and dependencies / 模块导入与依赖
```python
import unittest

import torch

from sglang.srt.layers.moe.topk import (
    biased_grouped_topk_impl as native_biased_grouped_topk,
)
from sglang.srt.layers.moe.topk import fused_topk_torch_native as native_fused_topk
from sglang.srt.layers.moe.topk import grouped_topk_gpu as native_grouped_topk
from sglang.srt.models.llama4 import Llama4MoE
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `torch`, `sglang.srt.layers.moe.topk`, `sglang.srt.models.llama4`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `torch`, `sglang.srt.layers.moe.topk`, `sglang.srt.models.llama4`。

### Lines 13-13: supporting statements / 辅助语句
```python
torch.manual_seed(1234)
```
**EN:** This block performs supporting work through calls such as `manual_seed`, preparing state for nearby definitions.
**CN:** 该代码块通过 `manual_seed` 等调用完成辅助性工作，为相邻定义准备状态。

### Lines 14-16: supporting source context / 辅助源码上下文
```python


# This is used by the Deepseek-V2 model
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 17-17: class TestGroupedTopK declaration / 类 TestGroupedTopK 声明
```python
class TestGroupedTopK(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 18-51: method run single test / 方法 run single test
```python
    def _run_single_test(self, M, E, G, topk, topk_group, renormalize, dtype):
        torch.manual_seed(1234)

        # expand gating_output by M, otherwise bfloat16 fall into same value aftering truncating
        hidden_states = torch.randn(M, 100, dtype=dtype)
        gating_output = torch.randn(M, E, dtype=dtype) * 2 * M

        ref_topk_weights, ref_topk_ids = native_grouped_topk(
            hidden_states.float(),
            gating_output.float(),
            topk,
            renormalize,
            G,
            topk_group,
        )

        # fused version
        topk_weights, topk_ids = torch.ops.sgl_kernel.grouped_topk_cpu(
            hidden_states,
            gating_output,
            topk,
            renormalize,
            G,
            topk_group,
            0,
            None,
            None,
        )

        res = torch.zeros(M, E, dtype=torch.float)
        ref = torch.zeros(M, E, dtype=torch.float)
        res.scatter_(1, topk_ids.long(), topk_weights)
        ref.scatter_(1, ref_topk_ids.long(), ref_topk_weights)
        torch.testing.assert_close(res, ref)
```
**EN:** This block implements `_run_single_test` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_single_test`，承担模块行为中的一个聚焦逻辑片段。

### Lines 53-61: test case grouped topk / 测试用例 grouped topk
```python
    def test_grouped_topk(self):
        for renormalize in [True, False]:
            self._run_single_test(123, 8, 2, 2, 1, renormalize, torch.bfloat16)
            self._run_single_test(123, 16, 4, 3, 2, renormalize, torch.bfloat16)
            self._run_single_test(123, 32, 4, 3, 2, renormalize, torch.bfloat16)
            self._run_single_test(1123, 32, 4, 3, 2, renormalize, torch.bfloat16)
            self._run_single_test(123, 64, 1, 6, 1, renormalize, torch.bfloat16)
            self._run_single_test(123, 256, 8, 4, 8, renormalize, torch.bfloat16)
            self._run_single_test(123, 160, 8, 6, 2, renormalize, torch.bfloat16)
```
**EN:** This test exercises `test_grouped_topk` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_grouped_topk`。

### Lines 62-64: supporting source context / 辅助源码上下文
```python


# DeepSeek V2/V3/R1 uses biased_grouped_top
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 65-65: class TestBiasedGroupedTopK declaration / 类 TestBiasedGroupedTopK 声明
```python
class TestBiasedGroupedTopK(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 66-117: method run single test / 方法 run single test
```python
    def _run_single_test(
        self,
        M,
        E,
        G,
        topk,
        topk_group,
        renormalize,
        gating_dtype,
        bias_dtype,
        routed_scaling_factor,
    ):
        torch.manual_seed(1024)

        # expand gating_output by M, otherwise bfloat16 fall into same value aftering truncating
        hidden_states = torch.randn(M, 100, dtype=torch.bfloat16)
        gating_output = torch.randn(M, E, dtype=gating_dtype) * 2 * M
        correction_bias = torch.randn(E, dtype=bias_dtype)

        ref_topk_weights, ref_topk_ids = native_biased_grouped_topk(
            hidden_states.float(),
            gating_output.float(),
            correction_bias.float(),
            topk,
            renormalize,
            G,
            topk_group,
        )
        ref_topk_weights = (
            ref_topk_weights * routed_scaling_factor
            if routed_scaling_factor is not None
            else ref_topk_weights
        )
        # fused version
        topk_weights, topk_ids = torch.ops.sgl_kernel.biased_grouped_topk_cpu(
            hidden_states,
            gating_output,
            correction_bias,
            topk,
            renormalize,
            G,
            topk_group,
            0,
            routed_scaling_factor,
            None,
        )

        res = torch.zeros(M, E, dtype=torch.float)
        ref = torch.zeros(M, E, dtype=torch.float)
        res.scatter_(1, topk_ids.long(), topk_weights)
        ref.scatter_(1, ref_topk_ids.long(), ref_topk_weights)
        torch.testing.assert_close(res, ref)
```
**EN:** This block implements `_run_single_test` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_single_test`，承担模块行为中的一个聚焦逻辑片段。

### Lines 119-135: test case biased grouped topk / 测试用例 biased grouped topk
```python
    def test_biased_grouped_topk(self):
        for renormalize in [False]:
            for bias_dtype in [torch.float32, torch.bfloat16]:
                for gating_dtype in [torch.float32, torch.bfloat16]:
                    for routed_scaling_factor in [None, 1.125]:
                        for E_num in [128, 192, 256, 384]:
                            self._run_single_test(
                                34,
                                E_num,
                                8,
                                8,
                                2,
                                renormalize,
                                gating_dtype,
                                bias_dtype,
                                routed_scaling_factor,
                            )
```
**EN:** This test exercises `test_biased_grouped_topk` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_biased_grouped_topk`。

### Lines 138-138: class TestTopK declaration / 类 TestTopK 声明
```python
class TestTopK(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 139-162: method run single test / 方法 run single test
```python
    def _run_single_test(self, M, E, topk, renormalize, dtype):
        torch.manual_seed(1998)

        # expand gating_output by M, otherwise bfloat16 fall into same value aftering truncating
        hidden_states = torch.randn(M, 100, dtype=dtype)
        gating_output = torch.randn(M, E, dtype=dtype) * 2 * M

        ref_topk_weights, ref_topk_ids = native_fused_topk(
            hidden_states.float(),
            gating_output.float(),
            topk,
            renormalize,
        )

        # fused version
        topk_weights, topk_ids = torch.ops.sgl_kernel.topk_softmax_cpu(
            hidden_states, gating_output, topk, renormalize
        )

        res = torch.zeros(M, E, dtype=torch.float)
        ref = torch.zeros(M, E, dtype=torch.float)
        res.scatter_(1, topk_ids.long(), topk_weights)
        ref.scatter_(1, ref_topk_ids.long(), ref_topk_weights)
        torch.testing.assert_close(res, ref)
```
**EN:** This block implements `_run_single_test` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_single_test`，承担模块行为中的一个聚焦逻辑片段。

### Lines 164-172: test case topk / 测试用例 topk
```python
    def test_topk(self):
        for renormalize in [True, False]:
            self._run_single_test(123, 8, 2, renormalize, torch.bfloat16)
            self._run_single_test(123, 16, 3, renormalize, torch.bfloat16)
            self._run_single_test(123, 32, 3, renormalize, torch.bfloat16)
            self._run_single_test(123, 32, 3, renormalize, torch.bfloat16)
            self._run_single_test(123, 64, 6, renormalize, torch.bfloat16)
            self._run_single_test(123, 256, 4, renormalize, torch.bfloat16)
            self._run_single_test(123, 160, 6, renormalize, torch.bfloat16)
```
**EN:** This test exercises `test_topk` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_topk`。

### Lines 175-175: class TestCustomTopK declaration / 类 TestCustomTopK 声明
```python
class TestCustomTopK(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 176-201: method run single test / 方法 run single test
```python
    def _run_single_test(
        self, M, E, topk, renormalize, dtype, native_custom_f, fused_custom_f
    ):
        torch.manual_seed(16)

        # expand gating_output by M, otherwise bfloat16 fall into same value aftering truncating
        hidden_states = torch.randn(M, 100, dtype=dtype)
        gating_output = torch.randn(M, E, dtype=dtype) * 2 * M

        ref_topk_weights, ref_topk_ids = native_custom_f(
            hidden_states.float(),
            gating_output.float(),
            topk,
            renormalize,
        )

        # fused version
        topk_weights, topk_ids = fused_custom_f(
            hidden_states, gating_output, topk, renormalize
        )

        res = torch.zeros(M, E, dtype=torch.float)
        ref = torch.zeros(M, E, dtype=torch.float)
        res.scatter_(1, topk_ids.long(), topk_weights)
        ref.scatter_(1, ref_topk_ids.long(), ref_topk_weights)
        torch.testing.assert_close(res, ref)
```
**EN:** This block implements `_run_single_test` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_single_test`，承担模块行为中的一个聚焦逻辑片段。

### Lines 203-216: test case custom topk / 测试用例 custom topk
```python
    def test_custom_topk(self):
        test_custom_functions = [
            (Llama4MoE.custom_routing_function, torch.ops.sgl_kernel.topk_sigmoid_cpu)
        ]
        for native_custom_f, fused_custom_f in test_custom_functions:
            self._run_single_test(
                123, 8, 1, False, torch.bfloat16, native_custom_f, fused_custom_f
            )
            self._run_single_test(
                123, 16, 1, False, torch.bfloat16, native_custom_f, fused_custom_f
            )
            self._run_single_test(
                123, 32, 1, False, torch.bfloat16, native_custom_f, fused_custom_f
            )
```
**EN:** This test exercises `test_custom_topk` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_custom_topk`。

### Lines 219-220: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestGroupedTopK`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestBiasedGroupedTopK`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestTopK`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestCustomTopK`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestGroupedTopK._run_single_test`: This block implements `_run_single_test` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_single_test`，承担模块行为中的一个聚焦逻辑片段。
- `TestGroupedTopK.test_grouped_topk`: This test exercises `test_grouped_topk` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_grouped_topk`。
- `TestBiasedGroupedTopK._run_single_test`: This block implements `_run_single_test` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_single_test`，承担模块行为中的一个聚焦逻辑片段。
- `TestBiasedGroupedTopK.test_biased_grouped_topk`: This test exercises `test_biased_grouped_topk` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_biased_grouped_topk`。
- `TestTopK._run_single_test`: This block implements `_run_single_test` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_single_test`，承担模块行为中的一个聚焦逻辑片段。
- `TestTopK.test_topk`: This test exercises `test_topk` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_topk`。
- `TestCustomTopK._run_single_test`: This block implements `_run_single_test` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_single_test`，承担模块行为中的一个聚焦逻辑片段。
- `TestCustomTopK.test_custom_topk`: This test exercises `test_custom_topk` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_custom_topk`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.layers.moe.topk`, `sglang.srt.models.llama4`, `sglang.test.test_utils`

- **Total lines / 总行数**: 220
