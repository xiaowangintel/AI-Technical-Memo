# quant_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/quant_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Support module for kernel tests in kernels / quant_utils; it provides reference logic, fixtures, or utilities reused by multiple scenarios. / kernels / quant_utils 对应的内核测试支撑模块；它提供会被多个场景复用的参考逻辑、fixture 或工具函数。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 5-13)
```python
import torch

from vllm.model_executor.layers.quantization.utils.quant_utils import (
    get_fp8_min_max,
    group_broadcast,
)
from vllm.platforms import current_platform
from vllm.utils.deep_gemm import _ceil_to_ue8m0, is_deep_gemm_e8m0_used
from vllm.utils.math_utils import round_up
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as torch; and vLLM components like vllm.model_executor.layers.quantization.utils.quant_utils, vllm.platforms, vllm.utils.deep_gemm, vllm.utils.math_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 torch；vLLM 内部组件，例如 vllm.model_executor.layers.quantization.utils.quant_utils、vllm.platforms、vllm.utils.deep_gemm、vllm.utils.math_utils。

### Constants and module state (lines 15-15)
```python
FP8_DTYPE = current_platform.fp8_dtype()
```
**EN:** This block centralizes shared constants and parameter grids, including FP8_DTYPE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 FP8_DTYPE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `as_float32_tensor` (lines 18-19)
```python
def as_float32_tensor(x: float | torch.Tensor) -> torch.Tensor:
    return torch.as_tensor(x, dtype=torch.float32, device="cuda")
```
**EN:** This helper function implements the shared logic for as float32 tensor. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 as float32 tensor 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `ref_dynamic_per_token_quant` (lines 22-63)
```python
def ref_dynamic_per_token_quant(
    x: torch.Tensor, quant_dtype: torch.dtype, scale_ub: torch.Tensor | None = None
) -> tuple[torch.Tensor, torch.Tensor]:
    assert quant_dtype in [torch.int8, FP8_DTYPE]
    if scale_ub is not None:
        assert quant_dtype == FP8_DTYPE

    if quant_dtype == torch.int8:
        qtype_traits = torch.iinfo(quant_dtype)
        qtype_traits_min = qtype_traits.min
        qtype_traits_max = qtype_traits.max
    else:
        qtype_traits_min, qtype_traits_max = get_fp8_min_max()
    qtype_max = as_float32_tensor(qtype_traits_max)
    s_1 = as_float32_tensor(1.0)
    s_512 = as_float32_tensor(512.0)

    # For fp8, in order to match the cuda kernel output, we have to do exactly
    # the same operations as in the corresponding fp8 kernel to prevent
    # rounding errors.

    # Compute scales
    x_token_max, _ = x.abs().max(dim=-1)
    x_token_max = as_float32_tensor(x_token_max)
    if scale_ub is not None:
        x_token_max = x_token_max.clamp(max=scale_ub)
    scales = (x_token_max / qtype_max)[:, None]

    # Quant
    if quant_dtype == torch.int8:
        iscales = as_float32_tensor(s_1 / scales)
        torch_out = as_float32_tensor(x) * iscales
        torch_out = torch_out.round()
        torch_out = torch_out.clamp(qtype_traits_min, qtype_traits_max).to(quant_dtype)
    else:
        assert quant_dtype == FP8_DTYPE
        min_scaling_factor = s_1 / (qtype_max * s_512)
        scales = scales.clamp(min=min_scaling_factor)
        torch_out = as_float32_tensor(x) / scales
        torch_out = torch_out.clamp(qtype_traits_min, qtype_traits_max).to(quant_dtype)

    return torch_out, scales
```
**EN:** This helper acts as a reference implementation for dynamic per token quant. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数为 dynamic per token quant 提供参考实现。 结尾处的断言会固定预期行为或计算图形态。

### Function `ref_dynamic_per_tensor_fp8_quant` (lines 69-88)
```python
def ref_dynamic_per_tensor_fp8_quant(
    x: torch.Tensor,
) -> tuple[torch.Tensor, torch.Tensor]:
    fp8_traits_min, fp8_traits_max = get_fp8_min_max()
    fp8_max = as_float32_tensor(fp8_traits_max)
    one = as_float32_tensor(1.0)

    # For fp8, in order to match the cuda kernel output, we have to do exactly
    # the same operations as in the corresponding fp8 kernel to prevent
    # rounding errors.

    x_max = as_float32_tensor(x.abs().max())
    ref_scale = x_max / fp8_max
    ref_iscale = one / ref_scale
    ref_out = (
        (as_float32_tensor(x) * ref_iscale)
        .clamp(fp8_traits_min, fp8_traits_max)
        .to(FP8_DTYPE)
    )
    return ref_out, ref_scale.view(1)
```
**EN:** This helper acts as a reference implementation for dynamic per tensor FP8 quant. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数为 dynamic per tensor FP8 quant 提供参考实现。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `native_w8a8_block_matmul` (lines 91-154)
```python
def native_w8a8_block_matmul(
    A: torch.Tensor,
    B: torch.Tensor,
    As: torch.Tensor,
    Bs: torch.Tensor,
    block_size: list[int],
    output_dtype: torch.dtype,
    compute_type: torch.dtype = torch.float32,
) -> torch.Tensor:
    """This function performs matrix multiplication with block-wise
    quantization using native torch.
    It is agnostic to the input data type and can be used for both int8 and
    fp8 data types.

    It takes two input tensors `A` and `B` (int8) with scales `As` and
    `Bs` (float32).
    The output is returned in the specified `output_dtype`.
    """
    A = A.to(compute_type)
    B = B.to(compute_type)
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
    assert n_tiles == Bs.shape[0], f"{n_tiles} == {Bs.shape[0]}"
    assert k_tiles == Bs.shape[1], f"{k_tiles} == {Bs.shape[1]}"

    C_shape = (M, N)
    C = torch.zeros(C_shape, dtype=compute_type, device=A.device)

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

    C = C.reshape(origin_C_shape).to(output_dtype)
    return C
```
**EN:** This helper function implements the shared logic for native w8a8 block matmul. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 native w8a8 block matmul 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `native_per_token_group_quant_fp8` (lines 157-180)
```python
def native_per_token_group_quant_fp8(
    x, group_size, eps=1e-10, dtype=torch.float8_e4m3fn
):
    """Function to perform per-token-group quantization on an input tensor
    `x` using native torch."""
    assert x.shape[-1] % group_size == 0, (
        "the last dimension of `x` must be divisible by `group_size`"
    )
    assert x.is_contiguous(), "`x` is not contiguous"

    finfo = torch.finfo(dtype)
    fp8_min = finfo.min
    fp8_max = finfo.max

    x_ = x.reshape(x.numel() // group_size, group_size)
    amax = x_.abs().max(dim=-1, keepdim=True)[0].clamp(min=eps).to(torch.float32)
    x_s = amax / fp8_max
    if is_deep_gemm_e8m0_used():
        x_s = _ceil_to_ue8m0(x_s)
    x_q = (x_ / x_s).clamp(min=fp8_min, max=fp8_max).to(dtype)
    x_q = x_q.reshape(x.shape)
    x_s = x_s.reshape(x.shape[:-1] + (x.shape[-1] // group_size,))

    return x_q, x_s
```
**EN:** This helper function implements the shared logic for native per token group quant FP8. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 native per token group quant FP8 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `native_per_token_group_quant_int8` (lines 183-209)
```python
def native_per_token_group_quant_int8(x, group_size, eps=1e-10, dtype=torch.int8):
    """Function to perform per-token-group quantization on an input tensor
    `x` using native torch.

    It converts the tensor values into int8 values and returns the
    quantized tensor along with the scaling factor used for quantization.
    """
    assert x.shape[-1] % group_size == 0, (
        "the last dimension of `x` must be divisible by `group_size`"
    )
    assert x.is_contiguous(), "`x` is not contiguous"

    iinfo = torch.iinfo(dtype)
    int8_min = iinfo.min
    int8_max = iinfo.max

    x_ = x.reshape(x.numel() // group_size, group_size)
    # Use float32 for scale calculation for stability
    amax = x_.abs().max(dim=-1, keepdim=True)[0].clamp(min=eps).to(torch.float32)
    x_s = amax / int8_max
    x_q = (
        (x_.to(torch.float32) / x_s).round().clamp(min=int8_min, max=int8_max).to(dtype)
    )  # Round before clamping
    x_q = x_q.reshape(x.shape)
    x_s = x_s.reshape(x.shape[:-1] + (x.shape[-1] // group_size,))

    return x_q, x_s
```
**EN:** This helper function implements the shared logic for native per token group quant int8. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 native per token group quant int8 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Constants and module state (lines 212-212)
```python
DEFAULT_BLOCK_SHAPE = [128, 128]
```
**EN:** This block centralizes shared constants and parameter grids, including DEFAULT_BLOCK_SHAPE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 DEFAULT_BLOCK_SHAPE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `per_block_cast_to_int8` (lines 215-231)
```python
def per_block_cast_to_int8(
    x: torch.Tensor,
    block_shape: list[int] = DEFAULT_BLOCK_SHAPE,
) -> tuple[torch.Tensor, torch.Tensor]:
    block_m, block_n = block_shape
    assert x.dim() == 2
    m, n = x.shape
    x_padded = torch.zeros(
        (round_up(m, block_m), round_up(n, block_n)), dtype=x.dtype, device=x.device
    )
    x_padded[:m, :n] = x
    x_view = x_padded.view(-1, block_m, x_padded.size(1) // block_n, block_n)
    x_amax = x_view.abs().float().amax(dim=(1, 3), keepdim=True).clamp(1e-4)
    x_scaled = (x_view * (256.0 / x_amax)).to(torch.int8)
    x_scaled_sub = x_scaled.view_as(x_padded)[:m, :n].contiguous()
    scales = (x_amax / 256.0).view(x_view.size(0), x_view.size(2))
    return x_scaled_sub, scales
```
**EN:** This helper function implements the shared logic for per block cast to int8. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 per block cast to int8 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `dequant` (lines 234-248)
```python
def dequant(
    t: torch.Tensor,
    scale: torch.Tensor | None,
    block_shape: list[int] | None,
    per_act_token_quant: bool,
    out_dtype: torch.dtype | None = torch.float32,
) -> torch.Tensor:
    if scale is not None:
        f32 = torch.float32
        if per_act_token_quant or block_shape is None:
            return (t.to(f32) * scale).to(out_dtype)
        else:
            return (t.to(f32) * group_broadcast(scale, t.shape)).to(out_dtype)
    else:
        return t.to(out_dtype)
```
**EN:** This helper function implements the shared logic for dequant. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 dequant 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `batched_dequant` (lines 251-267)
```python
def batched_dequant(
    t: torch.Tensor,
    scale: torch.Tensor | None,
    block_shape: list[int] | None,
    per_act_token_quant: bool,
    out_dtype: torch.dtype | None = torch.float32,
) -> torch.Tensor:
    if scale is not None:
        assert t.shape[0] == scale.shape[0]
        out = torch.empty_like(t, dtype=out_dtype)
        for e in range(t.shape[0]):
            out[e] = dequant(
                t[e], scale[e], block_shape, per_act_token_quant, out_dtype
            )
        return out

    return t.to(out_dtype)
```
**EN:** This helper function implements the shared logic for batched dequant. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 batched dequant 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `native_batched_masked_quant_matmul` (lines 270-304)
```python
def native_batched_masked_quant_matmul(
    A: torch.Tensor,
    B: torch.Tensor,
    C: torch.Tensor,
    num_expert_tokens: torch.Tensor,
    A_scale: torch.Tensor | None = None,
    B_scale: torch.Tensor | None = None,
    block_shape: list[int] | None = None,
    per_act_token_quant: bool = False,
) -> torch.Tensor:
    num_expert_tokens_cpu = num_expert_tokens.clone()
    num_expert_tokens_cpu = num_expert_tokens_cpu.to(device="cpu")
    num_experts = num_expert_tokens.size(0)

    for e in range(num_experts):
        num_tokens = num_expert_tokens_cpu[e]
        if A.dtype.itemsize == 1 and block_shape is not None:
            assert A_scale is not None and B_scale is not None
            tmp = native_w8a8_block_matmul(
                A[e], B[e], A_scale[e], B_scale[e], block_shape, C.dtype
            )
            C[e, :num_tokens, :] = tmp[:num_tokens, :]
        elif A.dtype.itemsize == 1 and block_shape is None:
            assert A_scale is not None and B_scale is not None
            A_dq = dequant(A[e], A_scale[e], block_shape, per_act_token_quant)
            B_dq = dequant(B[e], B_scale[e], block_shape, per_act_token_quant)
            C[e, :num_tokens, :] = (A_dq[:num_tokens] @ B_dq.transpose(0, 1)).to(
                C.dtype
            )
        else:
            assert A_scale is None
            assert B_scale is None
            C[e, :num_tokens, :] = A[e, :num_tokens, :] @ B[e].transpose(0, 1)

    return C
```
**EN:** This helper function implements the shared logic for native batched masked quant matmul. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 native batched masked quant matmul 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

## Key Concepts / 关键概念
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。
- **Shared test utilities / 共享测试工具:** The module factors repeated setup or reference logic out of the individual test files. / 该模块把重复的初始化或参考逻辑从单个测试文件中抽离出来复用。

## Dependencies / 依赖关系
- `torch`
- `vllm.model_executor.layers.quantization.utils.quant_utils -> get_fp8_min_max, group_broadcast`
- `vllm.platforms -> current_platform`
- `vllm.utils.deep_gemm -> _ceil_to_ue8m0, is_deep_gemm_e8m0_used`
- `vllm.utils.math_utils -> round_up`
