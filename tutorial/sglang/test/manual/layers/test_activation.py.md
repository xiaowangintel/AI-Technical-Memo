# test_activation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/layers/test_activation.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `activation` scenario in `test/manual/layers`. It uses SGLang's shared test infrastructure to configure models or services and verify expected functional behavior. / 该手动测试模块覆盖 `test/manual/layers` 中的 `activation` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Imports and shared helpers / 导入与共享辅助项
```python
import itertools
import unittest

import torch

from sglang.srt.layers.activation import GeluAndMul, QuickGELU
from sglang.srt.utils import is_hip
from sglang.test.test_utils import CustomTestCase

_is_hip = is_hip()
```
**EN:** This range imports `itertools`, `unittest`, `torch` and `sglang.srt.layers.activation`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. Representative call sites include `is_hip`.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 13-19: Class definition for TestGeluAndMul / 类定义
```python
class TestGeluAndMul(CustomTestCase):
    DTYPES = [torch.half, torch.bfloat16]
    NUM_TOKENS = [7, 83, 2048]
    D = [512, 4096, 5120, 13824]
    SEEDS = [0]

    @classmethod
```
**EN:** This range declares `TestGeluAndMul`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 20-23: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        if not torch.cuda.is_available():
            raise unittest.SkipTest("CUDA is not available")
        torch.set_default_device("cuda")
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `is_available`, `SkipTest` and `set_default_device`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 25-37: Helper routines around _run_gelu_and_mul_test / 辅助例程
```python
    def _run_gelu_and_mul_test(self, num_tokens, d, dtype, seed):
        torch.manual_seed(seed)

        layer = GeluAndMul().to(dtype=dtype)
        x = torch.randn(num_tokens, 2 * d, dtype=dtype)

        with torch.inference_mode():
            ref_out = layer.forward_native(x)
            out = layer.forward_cuda(x)

        if dtype == torch.bfloat16:
            atol = rtol = 1e-2
        else:
```
**EN:** This range implements helper routine(s) `_run_gelu_and_mul_test` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `manual_seed`, `GeluAndMul`, `to` and `randn`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 38-40: Assertions and result checks / 断言与结果检查
```python
            atol = rtol = 1e-3

        self.assertTrue(torch.allclose(out, ref_out, atol=atol, rtol=rtol))
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertTrue` and `allclose`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 42-55: Test routines around test_gelu_and_mul / 测试例程
```python
    def test_gelu_and_mul(self):
        for params in itertools.product(
            self.NUM_TOKENS,
            self.D,
            self.DTYPES,
            self.SEEDS,
        ):
            with self.subTest(
                num_tokens=params[0],
                d=params[1],
                dtype=params[2],
                seed=params[3],
            ):
                self._run_gelu_and_mul_test(*params)
```
**EN:** This range defines concrete test routine(s) `test_gelu_and_mul`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `product`, `subTest` and `_run_gelu_and_mul_test`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 58-64: Class definition for TestQuickGELU / 类定义
```python
class TestQuickGELU(CustomTestCase):
    DTYPES = [torch.half, torch.bfloat16]
    NUM_TOKENS = [7, 83, 2048]  # batch = sequence length
    DIMS = [512, 4096, 5120, 13824]  # all multiples of 16 bytes
    SEEDS = [0]

    @classmethod
```
**EN:** This range declares `TestQuickGELU`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 65-68: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        if not torch.cuda.is_available():
            raise unittest.SkipTest("CUDA is not available")
        torch.set_default_device("cuda")
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `is_available`, `SkipTest` and `set_default_device`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 70-82: Helper routines around _run_gelu_quick_test / 辅助例程
```python
    def _run_gelu_quick_test(self, n_tok: int, dim: int, dtype: torch.dtype, seed: int):
        torch.manual_seed(seed)

        layer = QuickGELU().to(dtype=dtype)

        x = torch.randn(n_tok, dim, dtype=dtype, device="cuda")

        with torch.inference_mode():
            ref = layer.forward_native(x)  # x * sigmoid(1.702 * x), fp32 math
            if _is_hip:
                out = layer.forward_hip(x)  # 128-bit vectorised kernel from sgl-kernel
            else:
                out = layer.forward_cuda(x)
```
**EN:** This range implements helper routine(s) `_run_gelu_quick_test` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `manual_seed`, `QuickGELU`, `to` and `randn`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 83-89: Assertions and result checks / 断言与结果检查
```python

        tol = 1e-2 if dtype is torch.bfloat16 else 1e-3
        self.assertTrue(
            torch.allclose(out, ref, atol=tol, rtol=tol),
            msg=f"Mismatch @ B={n_tok}, D={dim}, dtype={dtype}",
        )
        print(f"Match @ B={n_tok}, D={dim}, dtype={dtype}")
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertTrue` and `allclose`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 91-101: Test routines around test_quick_gelu / 测试例程
```python
    def test_quick_gelu(self):
        for params in itertools.product(
            self.NUM_TOKENS, self.DIMS, self.DTYPES, self.SEEDS
        ):
            with self.subTest(
                num_tokens=params[0],
                dim=params[1],
                dtype=params[2],
                seed=params[3],
            ):
                self._run_gelu_quick_test(*params)
```
**EN:** This range defines concrete test routine(s) `test_quick_gelu`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `product`, `subTest` and `_run_gelu_quick_test`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 102-105: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main(verbosity=2)
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Multi-GPU orchestration / 多 GPU 编排
- Token-level inspection / Token 级分析

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `itertools`, `unittest`
- **Third-party / 第三方库**: `torch`
- **Project Modules / 项目模块**: `sglang.srt.layers.activation`, `sglang.srt.utils`, `sglang.test.test_utils`
