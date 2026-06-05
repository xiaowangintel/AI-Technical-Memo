# test_block_fp8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/quant/test_block_fp8.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `block fp8` scenario in `test/manual/quant`. It uses SGLang's shared test infrastructure to configure models or services and verify expected accuracy and functional behavior. / 该手动测试模块覆盖 `test/manual/quant` 中的 `block fp8` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的精度与功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-29: Scenario logic / 场景逻辑
```python
import itertools
import unittest
from functools import lru_cache

import torch

from sglang.srt.layers.activation import SiluAndMul
from sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe import fused_moe
from sglang.srt.layers.moe.topk import TopKConfig, select_experts
from sglang.srt.layers.quantization.fp8_kernel import (
    per_tensor_quant_mla_fp8,
    per_token_group_quant_fp8,
    per_token_group_quant_mla_deep_gemm_masked_fp8,
    static_quant_fp8,
    w8a8_block_fp8_matmul,
)
from sglang.srt.layers.quantization.fp8_utils import (
    input_to_float8,
    mxfp8_group_quantize,
    triton_mxfp8_blockscaled_linear,
)
from sglang.srt.utils import is_sm100_supported, is_sm120_supported
from sglang.test.test_utils import CustomTestCase

_is_cuda = torch.cuda.is_available() and torch.version.cuda


# For test
@lru_cache(maxsize=1)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `is_available` and `lru_cache`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 30-37: Helper routines around _get_triton_mxfp8_upcast / 辅助例程
```python
def _get_triton_mxfp8_upcast():
    try:
        from triton_kernels.numerics_details.mxfp import upcast_from_mxfp_torch
    except Exception as err:
        raise RuntimeError(
            "MXFP8 dequantization requires triton_kernels with MXFP8 support."
        ) from err
    return upcast_from_mxfp_torch
```
**EN:** This range implements helper routine(s) `_get_triton_mxfp8_upcast` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `RuntimeError`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 38-40: Scenario logic / 场景逻辑
```python


# For test
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 41-65: Helper routines around native_per_token_group_quant_fp8 / 辅助例程
```python
def native_per_token_group_quant_fp8(
    x, group_size, eps=1e-10, dtype=torch.float8_e4m3fn
):
    """Function to perform per-token-group quantization on an input tensor `x` using native torch.

    It converts the tensor values into float8 values and returns the
    quantized tensor along with the scaling factor used for quantization.
    Note that only `torch.float8_e4m3fn` is supported for now.
    """
    assert (
        x.shape[-1] % group_size == 0
    ), "the last dimension of `x` cannot be divisible by `group_size`"
    assert x.is_contiguous(), "`x` is not contiguous"

    finfo = torch.finfo(dtype)
    fp8_min = finfo.min
    fp8_max = finfo.max

    x_ = x.reshape(x.numel() // group_size, group_size)
    amax = x_.abs().max(dim=-1, keepdim=True)[0].clamp(min=eps).to(torch.float32)
    x_s = amax / fp8_max
    x_q = (x_ / x_s).clamp(min=fp8_min, max=fp8_max).to(dtype)
    x_q = x_q.reshape(x.shape)
    x_s = x_s.reshape(x.shape[:-1] + (x.shape[-1] // group_size,))
```
**EN:** This range implements helper routine(s) `native_per_token_group_quant_fp8` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `assert`, `is_contiguous`, `finfo` and `reshape`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 66-66: Scenario logic / 场景逻辑
```python
    return x_q, x_s
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 69-76: Class definition for TestPerTokenGroupQuantFP8 / 类定义
```python
class TestPerTokenGroupQuantFP8(CustomTestCase):
    DTYPES = [torch.half, torch.bfloat16, torch.float32]
    NUM_TOKENS = [7, 83, 2048]
    D = [512, 4096, 5120, 13824]
    GROUP_SIZE = [64, 128, 256, 512]
    SEEDS = [0]

    @classmethod
```
**EN:** This range declares `TestPerTokenGroupQuantFP8`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 77-80: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        if not torch.cuda.is_available():
            raise unittest.SkipTest("CUDA is not available")
        torch.set_default_device("cuda")
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `is_available`, `SkipTest` and `set_default_device`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 82-93: Helper routines around _per_token_group_quant_fp8 / 辅助例程
```python
    def _per_token_group_quant_fp8(self, num_tokens, d, dtype, group_size, seed):
        torch.manual_seed(seed)

        x = torch.rand(num_tokens, d, dtype=dtype)

        with torch.inference_mode():
            ref_out, ref_scale = native_per_token_group_quant_fp8(x, group_size)
            out, scale = per_token_group_quant_fp8(x, group_size)

        self.assertTrue(
            torch.allclose(out.to(torch.float32), ref_out.to(torch.float32), rtol=0.20)
        )
```
**EN:** This range implements helper routine(s) `_per_token_group_quant_fp8` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `manual_seed`, `rand`, `inference_mode` and `native_per_token_group_quant_fp8`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 94-94: Assertions and result checks / 断言与结果检查
```python
        self.assertTrue(torch.allclose(scale, ref_scale))
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertTrue` and `allclose`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 96-111: Test routines around test_per_token_group_quant_fp8 / 测试例程
```python
    def test_per_token_group_quant_fp8(self):
        for params in itertools.product(
            self.NUM_TOKENS,
            self.D,
            self.DTYPES,
            self.GROUP_SIZE,
            self.SEEDS,
        ):
            with self.subTest(
                num_tokens=params[0],
                d=params[1],
                dtype=params[2],
                group_size=params[3],
                seed=params[4],
            ):
                self._per_token_group_quant_fp8(*params)
```
**EN:** This range defines concrete test routine(s) `test_per_token_group_quant_fp8`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `product`, `subTest` and `_per_token_group_quant_fp8`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 112-114: Scenario logic / 场景逻辑
```python


# For test
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 115-133: Helper routines around native_static_quant_fp8 / 辅助例程
```python
def native_static_quant_fp8(x, x_s, dtype=torch.float8_e4m3fn):
    """Function to perform static quantization on an input tensor `x` using native torch.

    It converts the tensor values into float8 values and returns the
    quantized tensor along with the scaling factor used for quantization.
    """
    assert x.is_contiguous(), "`x` is not contiguous"
    assert x_s.numel() == 1, "only supports per-tensor scale"

    finfo = torch.finfo(dtype)
    fp8_min = finfo.min
    fp8_max = finfo.max

    x_ = x.reshape(x.numel() // x.shape[-1], x.shape[-1])
    x_s_inv = 1.0 / x_s
    x_q = (x_ * x_s_inv).clamp(min=fp8_min, max=fp8_max).to(dtype)
    x_q = x_q.reshape(x.shape)

    return x_q, x_s
```
**EN:** This range implements helper routine(s) `native_static_quant_fp8` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `is_contiguous`, `numel`, `finfo` and `reshape`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 136-142: Class definition for TestStaticQuantFP8 / 类定义
```python
class TestStaticQuantFP8(CustomTestCase):
    DTYPES = [torch.half, torch.bfloat16, torch.float32]
    NUM_TOKENS = [7, 83, 2048]
    D = [512, 4096, 5120, 13824]
    SEEDS = [0]

    @classmethod
```
**EN:** This range declares `TestStaticQuantFP8`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 143-146: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        if not torch.cuda.is_available():
            raise unittest.SkipTest("CUDA is not available")
        torch.set_default_device("cuda")
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `is_available`, `SkipTest` and `set_default_device`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 148-160: Helper routines around _static_quant_fp8 / 辅助例程
```python
    def _static_quant_fp8(self, num_tokens, d, dtype, seed):
        torch.manual_seed(seed)

        x = torch.rand(num_tokens, d, dtype=dtype)
        fp8_max = torch.finfo(torch.float8_e4m3fn).max
        x_s = x.max() / fp8_max

        with torch.inference_mode():
            ref_out, _ = native_static_quant_fp8(x, x_s)
            out, _ = static_quant_fp8(x, x_s, repeat_scale=True)

        self.assertTrue(
            torch.allclose(out.to(torch.float32), ref_out.to(torch.float32), rtol=0.50)
```
**EN:** This range implements helper routine(s) `_static_quant_fp8` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `manual_seed`, `rand`, `finfo` and `inference_mode`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 161-161: Scenario logic / 场景逻辑
```python
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 163-176: Test routines around test_static_quant_fp8 / 测试例程
```python
    def test_static_quant_fp8(self):
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
                self._static_quant_fp8(*params)
```
**EN:** This range defines concrete test routine(s) `test_static_quant_fp8`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `product`, `subTest` and `_static_quant_fp8`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 179-187: Class definition for TestPerTensorQuantMlaFP8 / 类定义
```python
class TestPerTensorQuantMlaFP8(CustomTestCase):
    DTYPES = [torch.half, torch.bfloat16, torch.float32]
    NUM_TOKENS = [7, 83, 2048]
    D = [512, 4096, 5120, 13824]
    LAST_D_EXT = [1024, 0]
    LAST_D = [512]
    SEEDS = [0]

    @classmethod
```
**EN:** This range declares `TestPerTensorQuantMlaFP8`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 188-191: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        if not torch.cuda.is_available():
            raise unittest.SkipTest("CUDA is not available")
        torch.set_default_device("cuda")
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `is_available`, `SkipTest` and `set_default_device`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 193-203: Helper routines around _per_tensor_quant_mla_fp8 / 辅助例程
```python
    def _per_tensor_quant_mla_fp8(self, num_tokens, d, last_d_ext, last_d, dtype, seed):
        torch.manual_seed(seed)

        x = torch.rand(
            (num_tokens, d // last_d, last_d + last_d_ext),
            dtype=dtype,
        )
        x_sub, _ = x.split([last_d, last_d_ext], dim=-1)

        with torch.inference_mode():
            ref_out, ref_s = input_to_float8(x_sub.transpose(0, 1))
```
**EN:** This range implements helper routine(s) `_per_tensor_quant_mla_fp8` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `manual_seed`, `rand`, `split` and `inference_mode`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 204-212: Assertions and result checks / 断言与结果检查
```python
            out, out_s = per_tensor_quant_mla_fp8(x_sub.transpose(0, 1))

        self.assertTrue(out.is_contiguous())
        self.assertTrue(
            torch.allclose(out.to(torch.float32), ref_out.to(torch.float32), rtol=0.50)
        )
        self.assertTrue(
            torch.allclose(out_s.to(torch.float32), ref_s.to(torch.float32))
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `per_tensor_quant_mla_fp8`, `transpose`, `assertTrue` and `is_contiguous`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 214-228: Test routines around test_per_tensor_quant_mla_fp8 / 测试例程
```python
    def test_per_tensor_quant_mla_fp8(self):
        for params in itertools.product(
            self.NUM_TOKENS,
            self.D,
            self.LAST_D_EXT,
            self.LAST_D,
            self.DTYPES,
            self.SEEDS,
        ):
            with self.subTest(
                num_tokens=params[0],
                d=params[1],
                last_d_ext=params[2],
                last_d=params[3],
                dtype=params[4],
```
**EN:** This range defines concrete test routine(s) `test_per_tensor_quant_mla_fp8`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `product` and `subTest`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 229-231: Scenario logic / 场景逻辑
```python
                seed=params[5],
            ):
                self._per_tensor_quant_mla_fp8(*params)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `_per_tensor_quant_mla_fp8`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 234-242: Class definition for TestPerTokenGroupQuantMlaDeepGemmMaskedFP8 / 类定义
```python
class TestPerTokenGroupQuantMlaDeepGemmMaskedFP8(CustomTestCase):
    DTYPES = [torch.half, torch.bfloat16, torch.float32]
    B = [128]
    NUM_TOKENS = [7, 83, 2048, 1024 * 16]
    D = [512, 128]
    GROUP_SIZE = [128]
    SEEDS = [0]

    @classmethod
```
**EN:** This range declares `TestPerTokenGroupQuantMlaDeepGemmMaskedFP8`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 243-246: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        if not torch.cuda.is_available():
            raise unittest.SkipTest("CUDA is not available")
        torch.set_default_device("cuda")
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `is_available`, `SkipTest` and `set_default_device`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 248-258: Helper routines around _per_token_group_quant_mla_deep_gemm_masked_fp8 / 辅助例程
```python
    def _per_token_group_quant_mla_deep_gemm_masked_fp8(
        self, b, num_tokens, d, dtype, group_size, seed
    ):
        torch.manual_seed(seed)

        x = torch.rand(b, num_tokens, d, dtype=dtype)

        with torch.inference_mode():
            ref_out, ref_scale = native_per_token_group_quant_fp8(x, group_size, 1e-12)
            out, scale, _, _, _ = per_token_group_quant_mla_deep_gemm_masked_fp8(
                x, group_size
```
**EN:** This range implements helper routine(s) `_per_token_group_quant_mla_deep_gemm_masked_fp8` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `manual_seed`, `rand`, `inference_mode` and `native_per_token_group_quant_fp8`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 259-268: Assertions and result checks / 断言与结果检查
```python
            )
            out = out[:, :num_tokens, :]
            scale = scale[:, :num_tokens, :]

        self.assertTrue(
            torch.allclose(
                out.to(torch.float32), ref_out.to(torch.float32), rtol=0.20, atol=1e-2
            )
        )
        self.assertTrue(torch.allclose(scale, ref_scale))
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertTrue`, `allclose` and `to`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 270-283: Test routines around test_per_token_group_quant_mla_deep_gemm_masked_fp8 / 测试例程
```python
    def test_per_token_group_quant_mla_deep_gemm_masked_fp8(self):
        for params in itertools.product(
            self.B,
            self.NUM_TOKENS,
            self.D,
            self.DTYPES,
            self.GROUP_SIZE,
            self.SEEDS,
        ):
            with self.subTest(
                b=params[0],
                num_tokens=params[1],
                d=params[2],
                dtype=params[3],
```
**EN:** This range defines concrete test routine(s) `test_per_token_group_quant_mla_deep_gemm_masked_fp8`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `product` and `subTest`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 284-287: Scenario logic / 场景逻辑
```python
                group_size=params[4],
                seed=params[5],
            ):
                self._per_token_group_quant_mla_deep_gemm_masked_fp8(*params)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `_per_token_group_quant_mla_deep_gemm_masked_fp8`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 288-290: Scenario logic / 场景逻辑
```python


# For test
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 291-315: Helper routines around native_w8a8_block_fp8_matmul / 辅助例程
```python
def native_w8a8_block_fp8_matmul(A, B, As, Bs, block_size, output_dtype=torch.float16):
    """This function performs matrix multiplication with block-wise quantization using native torch.

    It takes two input tensors `A` and `B` with scales `As` and `Bs`.
    The output is returned in the specified `output_dtype`.
    """

    A = A.to(torch.float32)
    B = B.to(torch.float32)
    assert A.shape[-1] == B.shape[-1]
    assert B.ndim == 2 and B.is_contiguous() and Bs.ndim == 2
    assert len(block_size) == 2
    block_n, block_k = block_size[0], block_size[1]
    assert (A.shape[-1] + block_k - 1) // block_k == As.shape[-1]
    assert A.shape[:-1] == As.shape[:-1]

    M = A.numel() // A.shape[-1]
    N, K = B.shape
    origin_C_shape = A.shape[:-1] + (N,)
    A = A.reshape(M, A.shape[-1])
    As = As.reshape(M, As.shape[-1])
    n_tiles = (N + block_n - 1) // block_n
    k_tiles = (K + block_k - 1) // block_k
    assert n_tiles == Bs.shape[0]
    assert k_tiles == Bs.shape[1]
```
**EN:** This range implements helper routine(s) `native_w8a8_block_fp8_matmul` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `to`, `is_contiguous`, `assert` and `numel`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 316-340: Scenario logic / 场景逻辑
```python

    C_shape = (M, N)
    C = torch.zeros(C_shape, dtype=torch.float32, device=A.device)

    A_tiles = [A[:, i * block_k : min((i + 1) * block_k, K)] for i in range(k_tiles)]
    B_tiles = [
        [
            B[
                j * block_n : min((j + 1) * block_n, N),
                i * block_k : min((i + 1) * block_k, K),
            ]
            for i in range(k_tiles)
        ]
        for j in range(n_tiles)
    ]
    C_tiles = [C[:, j * block_n : min((j + 1) * block_n, N)] for j in range(n_tiles)]
    As_tiles = [As[:, i : i + 1] for i in range(k_tiles)]

    for i in range(k_tiles):
        for j in range(n_tiles):
            a = A_tiles[i]
            b = B_tiles[j][i]
            c = C_tiles[j]
            s = As_tiles[i] * Bs[j][i]
            c[:, :] += torch.matmul(a, b.t()) * s
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `zeros`, `matmul` and `t`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 341-343: Scenario logic / 场景逻辑
```python

    C = C.reshape(origin_C_shape).to(output_dtype)
    return C
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `reshape` and `to`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 346-370: Class definition for TestW8A8BlockFP8Matmul / 类定义
```python
class TestW8A8BlockFP8Matmul(CustomTestCase):

    if not _is_cuda:
        OUT_DTYPES = [torch.float32, torch.half, torch.bfloat16]
        M = [1, 7, 83, 512, 2048]
        NKs = [
            (N, K)
            for N in [128, 512, 1024, 4096, 7748, 13824]
            for K in [256, 4096, 5120, 3884, 13824]
        ]
        # BLOCK_SIZE = [[64, 64], [64, 128], [128, 64], [128, 128]]
        BLOCK_SIZE = [[128, 128]]
        SEEDS = [0]
    else:
        # use practical shape in DeepSeek V3 for test
        OUT_DTYPES = [torch.bfloat16]
        M = [64, 128, 512, 1024, 4096]
        NKs = [
            (2112, 7168),
            (1536, 7168),
            (3072, 1536),
            (24576, 7168),
            (4096, 512),
            (7168, 2048),
            (4608, 7168),
```
**EN:** This range declares `TestW8A8BlockFP8Matmul`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 371-378: Constants and scenario settings / 常量与场景配置
```python
            (512, 7168),
            (7168, 2304),
            (7168, 512),
        ]
        BLOCK_SIZE = [[128, 128]]
        SEEDS = [0]

    @classmethod
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。

### Lines 379-382: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        if not torch.cuda.is_available():
            raise unittest.SkipTest("CUDA is not available")
        torch.set_default_device("cuda")
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `is_available`, `SkipTest` and `set_default_device`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 384-395: Helper routines around _w8a8_block_fp8_matmul / 辅助例程
```python
    def _w8a8_block_fp8_matmul(self, M, NK, block_size, out_dtype, seed):
        N, K = NK
        torch.manual_seed(seed)
        # NOTE(HandH1998): to avoid overflow when out_dtype = torch.half
        factor_for_scale = 1e-2
        fp8_info = torch.finfo(torch.float8_e4m3fn)
        fp8_max, fp8_min = fp8_info.max, fp8_info.min

        A_fp32 = (torch.rand(M, K, dtype=torch.float32) - 0.5) * 2 * fp8_max
        A_fp8 = A_fp32.clamp(min=fp8_min, max=fp8_max).to(torch.float8_e4m3fn)

        B_fp32 = (torch.rand(N, K, dtype=torch.float32) - 0.5) * 2 * fp8_max
```
**EN:** This range implements helper routine(s) `_w8a8_block_fp8_matmul` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `manual_seed`, `NOTE`, `finfo` and `rand`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 396-408: Scenario logic / 场景逻辑
```python
        B_fp8 = B_fp32.clamp(min=fp8_min, max=fp8_max).to(torch.float8_e4m3fn)

        block_n, block_k = block_size[0], block_size[1]
        n_tiles = (N + block_n - 1) // block_n
        k_tiles = (K + block_k - 1) // block_k

        As = torch.rand(M, k_tiles, dtype=torch.float32) * factor_for_scale
        Bs = torch.rand(n_tiles, k_tiles, dtype=torch.float32) * factor_for_scale

        with torch.inference_mode():
            ref_out = native_w8a8_block_fp8_matmul(
                A_fp8, B_fp8, As, Bs, block_size, out_dtype
            )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `clamp`, `to`, `rand` and `inference_mode`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 409-415: Assertions and result checks / 断言与结果检查
```python
            out = w8a8_block_fp8_matmul(A_fp8, B_fp8, As, Bs, block_size, out_dtype)

        self.assertTrue(
            torch.mean(torch.abs(out.to(torch.float32) - ref_out.to(torch.float32)))
            / torch.mean(torch.abs(ref_out.to(torch.float32)))
            < 0.001
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `w8a8_block_fp8_matmul`, `assertTrue`, `mean` and `abs`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 417-420: Test routines around test_w8a8_block_fp8_matmul / 测试例程
```python
    def test_w8a8_block_fp8_matmul(self):
        for params in itertools.product(
            self.M,
            self.NKs,
```
**EN:** This range defines concrete test routine(s) `test_w8a8_block_fp8_matmul`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `product`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 421-432: Scenario logic / 场景逻辑
```python
            self.BLOCK_SIZE,
            self.OUT_DTYPES,
            self.SEEDS,
        ):
            with self.subTest(
                M=params[0],
                NKs=params[1],
                block_size=params[2],
                out_dtype=params[3],
                seed=params[4],
            ):
                self._w8a8_block_fp8_matmul(*params)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `subTest` and `_w8a8_block_fp8_matmul`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 435-437: Helper routines around _mxfp8_group_dequant / 辅助例程
```python
def _mxfp8_group_dequant(q: torch.Tensor, scale_u8: torch.Tensor) -> torch.Tensor:
    upcast_from_mxfp_torch = _get_triton_mxfp8_upcast()
    return upcast_from_mxfp_torch(q, scale_u8, torch.float32, axis=1)
```
**EN:** This range implements helper routine(s) `_mxfp8_group_dequant` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `_get_triton_mxfp8_upcast` and `upcast_from_mxfp_torch`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 440-451: Class definition for TestMXFP8DenseLinear / 类定义
```python
class TestMXFP8DenseLinear(CustomTestCase):
    DTYPES = [torch.bfloat16]
    M = [1, 127, 128, 129, 255, 256]
    NKs = [
        (256, 512),
        (384, 1024),
        (512, 2048),
        (768, 1024),
    ]
    SEEDS = [0]

    @classmethod
```
**EN:** This range declares `TestMXFP8DenseLinear`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 452-457: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        if not torch.cuda.is_available():
            raise unittest.SkipTest("CUDA is not available")
        if not (is_sm100_supported() or is_sm120_supported()):
            raise unittest.SkipTest("MXFP8 requires Blackwell (SM100/SM120)")
        torch.set_default_device("cuda")
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `is_available`, `SkipTest`, `not` and `is_sm100_supported`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 459-464: Helper routines around _mxfp8_dense_linear / 辅助例程
```python
    def _mxfp8_dense_linear(self, M, NK, dtype, seed):
        N, K = NK
        torch.manual_seed(seed)

        input_fp32 = torch.randn((M, K), dtype=torch.float32) / 4
        input_fp16 = input_fp32.to(dtype)
```
**EN:** This range implements helper routine(s) `_mxfp8_dense_linear` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `manual_seed`, `randn` and `to`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 465-483: Scenario logic / 场景逻辑
```python

        weight_fp32 = torch.randn((N, K), dtype=torch.float32) / 4
        weight_q, weight_scale_u8 = mxfp8_group_quantize(weight_fp32)

        with torch.inference_mode():
            q_input, input_scale_u8 = mxfp8_group_quantize(input_fp16.to(torch.float32))
            a_dq = _mxfp8_group_dequant(q_input, input_scale_u8)
            b_dq = _mxfp8_group_dequant(weight_q, weight_scale_u8)
            ref_out = torch.matmul(a_dq, b_dq.t()).to(dtype)

            out = triton_mxfp8_blockscaled_linear(
                input=input_fp16,
                weight=weight_q,
                weight_scale=weight_scale_u8,
            )
            out_prequant = triton_mxfp8_blockscaled_linear(
                input=q_input,
                weight=weight_q,
                weight_scale=weight_scale_u8,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn`, `mxfp8_group_quantize`, `inference_mode` and `to`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 484-489: Assertions and result checks / 断言与结果检查
```python
                input_scale=input_scale_u8,
                output_dtype=dtype,
            )

        self.assertTrue(
            torch.mean(torch.abs(out.to(torch.float32) - ref_out.to(torch.float32)))
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertTrue`, `mean`, `abs` and `to`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 490-499: Assertions and result checks / 断言与结果检查
```python
            / torch.mean(torch.abs(ref_out.to(torch.float32)))
            < 0.02
        )
        self.assertTrue(
            torch.mean(
                torch.abs(out_prequant.to(torch.float32) - ref_out.to(torch.float32))
            )
            / torch.mean(torch.abs(ref_out.to(torch.float32)))
            < 0.02
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `mean`, `abs`, `to` and `assertTrue`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 501-514: Test routines around test_mxfp8_dense_linear / 测试例程
```python
    def test_mxfp8_dense_linear(self):
        for params in itertools.product(
            self.M,
            self.NKs,
            self.DTYPES,
            self.SEEDS,
        ):
            with self.subTest(
                M=params[0],
                NKs=params[1],
                dtype=params[2],
                seed=params[3],
            ):
                self._mxfp8_dense_linear(*params)
```
**EN:** This range defines concrete test routine(s) `test_mxfp8_dense_linear`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `product`, `subTest` and `_mxfp8_dense_linear`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 515-517: Scenario logic / 场景逻辑
```python


# For test
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 518-542: Helper routines around torch_w8a8_block_fp8_moe / 辅助例程
```python
def torch_w8a8_block_fp8_moe(a, w1, w2, w1_s, w2_s, score, topk, block_shape):
    """This function performs fused moe with block-wise quantization using native torch."""

    B, D = a.shape
    a = a.view(B, -1, D).repeat(1, topk, 1).reshape(-1, D)
    out = torch.zeros(B * topk, w2.shape[1], dtype=a.dtype, device=a.device)
    score = torch.softmax(score, dim=-1, dtype=torch.float32)
    topk_weight, topk_ids = torch.topk(score, topk)
    topk_weight = topk_weight.view(-1)
    topk_ids = topk_ids.view(-1)

    _, block_k = block_shape[0], block_shape[1]
    a_q, a_s = native_per_token_group_quant_fp8(a, block_k)
    # NOTE(HandH1998): Since "index_cuda" not implemented for 'Float8_e4m3fn', we need to cast `float8`` to `float32``.
    a_q = a_q.to(torch.float32)
    for i in range(w1.shape[0]):
        mask = topk_ids == i
        if mask.sum():
            inter_out = native_w8a8_block_fp8_matmul(
                a_q[mask], w1[i], a_s[mask], w1_s[i], block_shape, output_dtype=a.dtype
            )
            act_out = SiluAndMul().forward_native(inter_out)
            act_out_q, act_out_s = native_per_token_group_quant_fp8(act_out, block_k)
            act_out = act_out.to(torch.float32)
            out[mask] = native_w8a8_block_fp8_matmul(
```
**EN:** This range implements helper routine(s) `torch_w8a8_block_fp8_moe` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `view`, `repeat`, `reshape` and `zeros`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 543-547: Scenario logic / 场景逻辑
```python
                act_out_q, w2[i], act_out_s, w2_s[i], block_shape, output_dtype=a.dtype
            )
    return (
        out.view(B, -1, w2.shape[1]) * topk_weight.view(B, -1, 1).to(out.dtype)
    ).sum(dim=1)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `view` and `to`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 550-561: Class definition for TestW8A8BlockFP8FusedMoE / 类定义
```python
class TestW8A8BlockFP8FusedMoE(CustomTestCase):
    DTYPES = [torch.float32, torch.half, torch.bfloat16]
    M = [1, 33, 64, 222, 1024 * 128]
    N = [128, 1024, 2048]
    K = [256, 4096, 5120]
    E = [8, 24]
    TOP_KS = [2, 6]
    BLOCK_SIZE = [[64, 64], [64, 128], [128, 64], [128, 128]]
    # BLOCK_SIZE = [[128, 128]]
    SEEDS = [0]

    @classmethod
```
**EN:** This range declares `TestW8A8BlockFP8FusedMoE`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 562-565: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        if not torch.cuda.is_available():
            raise unittest.SkipTest("CUDA is not available")
        torch.set_default_device("cuda")
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `is_available`, `SkipTest` and `set_default_device`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 567-574: Helper routines around _w8a8_block_fp8_fused_moe / 辅助例程
```python
    def _w8a8_block_fp8_fused_moe(self, M, N, K, E, topk, block_size, dtype, seed):
        torch.manual_seed(seed)
        # NOTE(HandH1998): to avoid overflow when out_dtype = torch.half
        factor_for_scale = 1e-2
        fp8_info = torch.finfo(torch.float8_e4m3fn)
        fp8_max, fp8_min = fp8_info.max, fp8_info.min

        a = torch.randn((M, K), dtype=dtype) / 10
```
**EN:** This range implements helper routine(s) `_w8a8_block_fp8_fused_moe` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `manual_seed`, `NOTE`, `finfo` and `randn`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 575-591: Scenario logic / 场景逻辑
```python

        w1_fp32 = (torch.rand((E, 2 * N, K), dtype=torch.float32) - 0.5) * 2 * fp8_max
        w1 = w1_fp32.clamp(min=fp8_min, max=fp8_max).to(torch.float8_e4m3fn)

        w2_fp32 = (torch.rand((E, K, N), dtype=torch.float32) - 0.5) * 2 * fp8_max
        w2 = w2_fp32.clamp(min=fp8_min, max=fp8_max).to(torch.float8_e4m3fn)

        block_n, block_k = block_size[0], block_size[1]
        n_tiles_w1 = (2 * N + block_n - 1) // block_n
        n_tiles_w2 = (K + block_n - 1) // block_n
        k_tiles_w1 = (K + block_k - 1) // block_k
        k_tiles_w2 = (N + block_k - 1) // block_k

        w1_s = (
            torch.rand((E, n_tiles_w1, k_tiles_w1), dtype=torch.float32)
            * factor_for_scale
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `rand`, `clamp` and `to`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 592-599: Scenario logic / 场景逻辑
```python
        w2_s = (
            torch.rand((E, n_tiles_w2, k_tiles_w2), dtype=torch.float32)
            * factor_for_scale
        )

        score = torch.randn((M, E), dtype=dtype)

        with torch.inference_mode():
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `rand`, `randn` and `inference_mode`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 600-616: Scenario logic / 场景逻辑
```python
            ref_out = torch_w8a8_block_fp8_moe(
                a, w1, w2, w1_s, w2_s, score, topk, block_size
            )
            topk_output = select_experts(
                hidden_states=a,
                router_logits=score,
                topk_config=TopKConfig(top_k=topk, renormalize=False),
            )
            out = fused_moe(
                a,
                w1,
                w2,
                topk_output,
                use_fp8_w8a8=True,
                w1_scale=w1_s,
                w2_scale=w2_s,
                block_shape=block_size,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `torch_w8a8_block_fp8_moe`, `select_experts`, `TopKConfig` and `fused_moe`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 617-623: Assertions and result checks / 断言与结果检查
```python
            )

        self.assertTrue(
            torch.mean(torch.abs(out.to(torch.float32) - ref_out.to(torch.float32)))
            / torch.mean(torch.abs(ref_out.to(torch.float32)))
            < 0.02
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertTrue`, `mean`, `abs` and `to`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 625-646: Test routines around test_w8a8_block_fp8_fused_moe / 测试例程
```python
    def test_w8a8_block_fp8_fused_moe(self):
        for params in itertools.product(
            self.M,
            self.N,
            self.K,
            self.E,
            self.TOP_KS,
            self.BLOCK_SIZE,
            self.DTYPES,
            self.SEEDS,
        ):
            with self.subTest(
                M=params[0],
                N=params[1],
                K=params[2],
                E=params[3],
                topk=params[4],
                block_size=params[5],
                dtype=params[6],
                seed=params[7],
            ):
                self._w8a8_block_fp8_fused_moe(*params)
```
**EN:** This range defines concrete test routine(s) `test_w8a8_block_fp8_fused_moe`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `product`, `subTest` and `_w8a8_block_fp8_fused_moe`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 647-649: Scenario logic / 场景逻辑
```python


# For test
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 650-662: Helper routines around torch_w8a8_block_fp8_bmm / 辅助例程
```python
def torch_w8a8_block_fp8_bmm(a, a_s, w, w_s, block_shape, out_dtype):
    """This function performs bmm with block-wise quantization using native torch."""

    B, N, _ = w.shape
    _, M, _ = a.shape
    out = torch.empty((B, M, N), dtype=out_dtype, device=a.device)

    for i in range(B):
        out[i] = native_w8a8_block_fp8_matmul(
            a[i], w[i], a_s[i], w_s[i], block_shape, output_dtype=out_dtype
        )

    return out
```
**EN:** This range implements helper routine(s) `torch_w8a8_block_fp8_bmm` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `empty` and `native_w8a8_block_fp8_matmul`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 665-674: Class definition for TestW8A8BlockFP8BatchedDeepGemm / 类定义
```python
class TestW8A8BlockFP8BatchedDeepGemm(CustomTestCase):
    DTYPES = [torch.bfloat16]
    M = [1, 33, 64, 222, 8192]
    N = [128, 512]
    K = [128, 512]
    BATCH = [128]
    BLOCK_SIZE = [[128, 128]]
    SEEDS = [0]

    @classmethod
```
**EN:** This range declares `TestW8A8BlockFP8BatchedDeepGemm`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 675-682: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        if not torch.cuda.is_available():
            raise unittest.SkipTest("CUDA is not available")
        try:
            import deep_gemm  # noqa: F401
        except ImportError:
            raise unittest.SkipTest("DeepGEMM is not available")
        torch.set_default_device("cuda")
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `is_available`, `SkipTest` and `set_default_device`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 684-689: Helper routines around _w8a8_block_fp8_batched_deep_gemm / 辅助例程
```python
    def _w8a8_block_fp8_batched_deep_gemm(self, M, N, K, B, block_size, dtype, seed):
        torch.manual_seed(seed)
        factor_for_scale = 1e-2
        fp8_info = torch.finfo(torch.float8_e4m3fn)
        fp8_max, fp8_min = fp8_info.max, fp8_info.min
```
**EN:** This range implements helper routine(s) `_w8a8_block_fp8_batched_deep_gemm` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `manual_seed` and `finfo`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 690-708: Scenario logic / 场景逻辑
```python
        a_fp32 = torch.randn((B, M, K), dtype=torch.float32) / 10
        a = a_fp32.clamp(min=fp8_min, max=fp8_max).to(torch.float8_e4m3fn)

        w_fp32 = (torch.rand((B, N, K), dtype=torch.float32) - 0.5) * 2 * fp8_max
        w = w_fp32.clamp(min=fp8_min, max=fp8_max).to(torch.float8_e4m3fn)

        block_n, block_k = block_size[0], block_size[1]
        n_tiles_w = (N + block_n - 1) // block_n
        k_tiles_w = (K + block_k - 1) // block_k

        w_s = (
            torch.rand((B, n_tiles_w, k_tiles_w), dtype=torch.float32)
            * factor_for_scale
        )
        a_s = torch.rand((B, M, k_tiles_w), dtype=torch.float32) * factor_for_scale

        ae = a.new_empty(B, (M + 255) // 256 * 256, K)
        ae_s = a_s.new_empty(B, (M + 255) // 256 * 256, k_tiles_w)
        oe = torch.empty((B, (M + 255) // 256 * 256, N), dtype=dtype)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn`, `clamp`, `to` and `rand`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 709-714: Scenario logic / 场景逻辑
```python
        ae[:, :M, :] = a
        ae_s[:, :M, :] = a_s

        masked_m = torch.full((B,), M, dtype=torch.int)
        expected_m = M
        lhs = (
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `full`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 715-733: Assertions and result checks / 断言与结果检查
```python
            ae,
            ae_s,
        )
        rhs = (
            w,
            w_s,
        )

        from deep_gemm import fp8_m_grouped_gemm_nt_masked

        with torch.inference_mode():
            ref_out = torch_w8a8_block_fp8_bmm(a, a_s, w, w_s, block_size, dtype)
            fp8_m_grouped_gemm_nt_masked(lhs, rhs, oe, masked_m, expected_m)
            out = oe[:, :M, :]

        self.assertTrue(
            torch.mean(torch.abs(out.to(torch.float32) - ref_out.to(torch.float32)))
            / torch.mean(torch.abs(ref_out.to(torch.float32)))
            < 0.0001
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `inference_mode`, `torch_w8a8_block_fp8_bmm`, `fp8_m_grouped_gemm_nt_masked` and `assertTrue`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 734-734: Scenario logic / 场景逻辑
```python
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 736-739: Test routines around test_w8a8_block_fp8_batched_deep_gemm / 测试例程
```python
    def test_w8a8_block_fp8_batched_deep_gemm(self):

        for params in itertools.product(
            self.M,
```
**EN:** This range defines concrete test routine(s) `test_w8a8_block_fp8_batched_deep_gemm`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `product`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 740-756: Scenario logic / 场景逻辑
```python
            self.N,
            self.K,
            self.BATCH,
            self.BLOCK_SIZE,
            self.DTYPES,
            self.SEEDS,
        ):
            with self.subTest(
                M=params[0],
                N=params[1],
                K=params[2],
                B=params[3],
                block_size=params[4],
                dtype=params[5],
                seed=params[6],
            ):
                self._w8a8_block_fp8_batched_deep_gemm(*params)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `subTest` and `_w8a8_block_fp8_batched_deep_gemm`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 757-760: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main(verbosity=2)
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Accuracy evaluation / 精度评测
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置
- Token-level inspection / Token 级分析
- Disaggregated serving / 解耦式服务

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `functools`, `itertools`, `unittest`
- **Third-party / 第三方库**: `deep_gemm`, `torch`, `triton_kernels.numerics_details.mxfp`
- **Project Modules / 项目模块**: `sglang.srt.layers.activation`, `sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe`, `sglang.srt.layers.moe.topk`, `sglang.srt.layers.quantization.fp8_kernel`, `sglang.srt.layers.quantization.fp8_utils`, `sglang.srt.utils`, `sglang.test.test_utils`
