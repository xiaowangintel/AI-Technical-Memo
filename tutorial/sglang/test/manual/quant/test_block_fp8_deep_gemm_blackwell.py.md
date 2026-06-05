# test_block_fp8_deep_gemm_blackwell.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/quant/test_block_fp8_deep_gemm_blackwell.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `block fp8 deep gemm blackwell` scenario in `test/manual/quant`. It uses SGLang's shared test infrastructure to configure models or services and verify expected functional behavior. / 该手动测试模块覆盖 `test/manual/quant` 中的 `block fp8 deep gemm blackwell` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的功能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Imports and shared helpers / 导入与共享辅助项
```python
import itertools
import unittest
from typing import List, Tuple

import torch
from deep_gemm import fp8_gemm_nt

from sglang.test.test_utils import CustomTestCase

_is_cuda = torch.cuda.is_available() and torch.version.cuda


# Modify form DeepGEMM Blackwell
```
**EN:** This range imports `itertools`, `unittest`, `typing` and `torch`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file. Representative call sites include `is_available`.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 14-15: Helper routines around ceil_div / 辅助例程
```python
def ceil_div(x: int, y: int) -> int:
    return (x + y - 1) // y
```
**EN:** This range implements helper routine(s) `ceil_div` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 18-19: Helper routines around align / 辅助例程
```python
def align(x: int, y: int) -> int:
    return ceil_div(x, y) * y
```
**EN:** This range implements helper routine(s) `align` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `ceil_div`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 22-28: Helper routines around per_token_group_quant_fp8 / 辅助例程
```python
def per_token_group_quant_fp8(x: torch.Tensor) -> Tuple[torch.Tensor, torch.Tensor]:
    assert x.dim() == 2 and x.size(1) % 128 == 0
    m, n = x.shape
    x_view = x.view(m, -1, 128)
    x_amax = x_view.abs().float().amax(dim=2).view(m, -1).clamp(1e-4)
    sf = x_amax / 448.0
    return (x_view * (1.0 / sf.unsqueeze(2))).to(torch.float8_e4m3fn).view(m, n), sf
```
**EN:** This range implements helper routine(s) `per_token_group_quant_fp8` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `dim`, `size`, `view` and `abs`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 31-44: Helper routines around per_block_quant_fp8 / 辅助例程
```python
def per_block_quant_fp8(x: torch.Tensor) -> Tuple[torch.Tensor, torch.Tensor]:
    assert x.dim() == 2
    m, n = x.shape
    x_padded = torch.zeros(
        (align(m, 128), align(n, 128)), dtype=x.dtype, device=x.device
    )
    x_padded[:m, :n] = x
    x_view = x_padded.view(-1, 128, x_padded.size(1) // 128, 128)
    x_amax = x_view.abs().float().amax(dim=(1, 3), keepdim=True).clamp(1e-4)
    sf = x_amax / 448.0
    x_scaled = (x_view * (1.0 / sf)).to(torch.float8_e4m3fn)
    return x_scaled.view_as(x_padded)[:m, :n].contiguous(), sf.view(
        x_view.size(0), x_view.size(2)
    )
```
**EN:** This range implements helper routine(s) `per_block_quant_fp8` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `dim`, `zeros`, `align` and `view`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 47-49: Helper routines around ceil_to_ue8m0 / 辅助例程
```python
def ceil_to_ue8m0(x: torch.Tensor):
    assert x.view(-1).amax().item() > 0
    return torch.pow(2.0, torch.ceil(torch.log2(x.abs())))
```
**EN:** This range implements helper routine(s) `ceil_to_ue8m0` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `view`, `amax`, `item` and `pow`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 52-58: Helper routines around per_token_group_quant_mxfp8 / 辅助例程
```python
def per_token_group_quant_mxfp8(x: torch.Tensor) -> Tuple[torch.Tensor, torch.Tensor]:
    assert x.dim() == 2 and x.size(1) % 128 == 0
    m, n = x.shape
    x_view = x.view(m, -1, 128)
    x_amax = x_view.abs().float().amax(dim=2).view(m, -1).clamp(1e-4)
    sf = ceil_to_ue8m0(x_amax / 448.0)
    return (x_view * (1.0 / sf.unsqueeze(2))).to(torch.float8_e4m3fn).view(m, n), sf
```
**EN:** This range implements helper routine(s) `per_token_group_quant_mxfp8` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `dim`, `size`, `view` and `abs`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 61-74: Helper routines around per_block_quant_mxfp8 / 辅助例程
```python
def per_block_quant_mxfp8(x: torch.Tensor) -> Tuple[torch.Tensor, torch.Tensor]:
    assert x.dim() == 2
    m, n = x.shape
    x_padded = torch.zeros(
        (align(m, 128), align(n, 128)), dtype=x.dtype, device=x.device
    )
    x_padded[:m, :n] = x
    x_view = x_padded.view(-1, 128, x_padded.size(1) // 128, 128)
    x_amax = x_view.abs().float().amax(dim=(1, 3), keepdim=True).clamp(1e-4)
    sf = ceil_to_ue8m0(x_amax / 448.0)
    x_scaled = (x_view * (1.0 / sf)).to(torch.float8_e4m3fn)
    return x_scaled.view_as(x_padded)[:m, :n].contiguous(), sf.view(
        x_view.size(0), x_view.size(2)
    )
```
**EN:** This range implements helper routine(s) `per_block_quant_mxfp8` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `dim`, `zeros`, `align` and `view`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 75-77: Scenario logic / 场景逻辑
```python


# For test
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 78-102: Helper routines around native_w8a8_block_fp8_matmul / 辅助例程
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

### Lines 103-127: Scenario logic / 场景逻辑
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

### Lines 128-130: Scenario logic / 场景逻辑
```python

    C = C.reshape(origin_C_shape).to(output_dtype)
    return C
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `reshape` and `to`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 133-157: Helper routines around block_quant_dequant / 辅助例程
```python
def block_quant_dequant(
    x_q_block: torch.Tensor,
    x_s: torch.Tensor,
    block_size: List[int],
    dtype: torch.dtype,
) -> torch.Tensor:
    """This function converts block-wise quantization to unquantized.
    The inputs are block-wise quantization tensor `x_q_block`, block-wise quantization scale
    and the block size.
    The output is an unquantized tensor with dtype.
    """
    block_n, block_k = block_size[0], block_size[1]
    n, k = x_q_block.shape
    n_tiles = (n + block_n - 1) // block_n
    k_tiles = (k + block_k - 1) // block_k
    assert n_tiles == x_s.shape[0]
    assert k_tiles == x_s.shape[1]

    x_dq_block = torch.empty_like(x_q_block, dtype=dtype)

    for j in range(n_tiles):
        for i in range(k_tiles):
            x_q_block_tile = x_q_block[
                j * block_n : min((j + 1) * block_n, n),
                i * block_k : min((i + 1) * block_k, k),
```
**EN:** This range implements helper routine(s) `block_quant_dequant` so setup, transformation, or validation logic can be reused cleanly. Assertions in this block enforce the intended outcome. Representative call sites include `empty_like`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 158-165: Scenario logic / 场景逻辑
```python
            ]
            x_dq_block_tile = x_dq_block[
                j * block_n : min((j + 1) * block_n, n),
                i * block_k : min((i + 1) * block_k, k),
            ]
            x_dq_block_tile[:, :] = x_q_block_tile.to(torch.float32) * x_s[j][i]

    return x_dq_block
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `to`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 168-192: Class definition for TestDeepGemmBlackwell / 类定义
```python
class TestDeepGemmBlackwell(CustomTestCase):

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
            # (3072, 1536),
            # (24576, 7168),
            # (4096, 512),
            # (7168, 2048),
            # (4608, 7168),
```
**EN:** This range declares `TestDeepGemmBlackwell`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 193-200: Constants and scenario settings / 常量与场景配置
```python
            # (512, 7168),
            # (7168, 2304),
            # (7168, 512),
        ]
        BLOCK_SIZE = [[128, 128]]
        SEEDS = [0]

    @classmethod
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。

### Lines 201-204: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        if not torch.cuda.is_available():
            raise unittest.SkipTest("CUDA is not available")
        torch.set_default_device("cuda")
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. Representative call sites include `is_available`, `SkipTest` and `set_default_device`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 206-217: Helper routines around _test_deep_gemm_blackwell / 辅助例程
```python
    def _test_deep_gemm_blackwell(self, M, NK, block_size, out_dtype, seed):
        N, K = NK
        torch.manual_seed(seed)

        A = torch.empty((M, K), dtype=torch.bfloat16).normal_(0, 0.2)
        B = torch.empty((N, K), dtype=torch.bfloat16).normal_(0, 0.2)

        A_q, A_s = per_token_group_quant_fp8(A)
        B_q, B_s = per_block_quant_fp8(B)

        A_dq = block_quant_dequant(A_q, A_s, [1, block_size[1]], out_dtype)
        B_dq = block_quant_dequant(B_q, B_s, block_size, out_dtype)
```
**EN:** This range implements helper routine(s) `_test_deep_gemm_blackwell` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `manual_seed`, `empty`, `normal_` and `per_token_group_quant_fp8`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 218-230: Assertions and result checks / 断言与结果检查
```python

        A_qu = per_token_group_quant_mxfp8(A_dq)
        B_qu = per_block_quant_mxfp8(B_dq)
        out = None

        with torch.inference_mode():
            ref_out = native_w8a8_block_fp8_matmul(
                A_qu[0], B_qu[0], A_qu[1], B_qu[1], block_size, out_dtype
            )
            out = torch.empty_like(ref_out)
            fp8_gemm_nt(A_qu, B_qu, out)

        torch.testing.assert_close(out, ref_out, atol=1e-1, rtol=1e-2)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `per_token_group_quant_mxfp8`, `per_block_quant_mxfp8`, `inference_mode` and `native_w8a8_block_fp8_matmul`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 232-242: Test routines around test_deep_gemm_blackwell / 测试例程
```python
    def test_deep_gemm_blackwell(self):
        for params in itertools.product(
            self.M,
            self.NKs,
            self.BLOCK_SIZE,
            self.OUT_DTYPES,
            self.SEEDS,
        ):
            with self.subTest(
                M=params[0],
                NKs=params[1],
```
**EN:** This range defines concrete test routine(s) `test_deep_gemm_blackwell`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `product` and `subTest`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 243-247: Scenario logic / 场景逻辑
```python
                block_size=params[2],
                out_dtype=params[3],
                seed=params[4],
            ):
                self._test_deep_gemm_blackwell(*params)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `_test_deep_gemm_blackwell`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 248-251: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main(verbosity=2)
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置
- Token-level inspection / Token 级分析

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `itertools`, `typing`, `unittest`
- **Third-party / 第三方库**: `deep_gemm`, `torch`
- **Project Modules / 项目模块**: `sglang.test.test_utils`
