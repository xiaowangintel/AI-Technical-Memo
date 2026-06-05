# test_triton_w4a16.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/quantization/test_triton_w4a16.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / quantization / test_triton_w4a16, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / quantization / test_triton_w4a16 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 4-7)
```python
"""Tests for the ROCm Triton W4A16 GEMM kernel.

Run `pytest tests/kernels/quantization/test_triton_w4a16.py`.
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 9-15)
```python
import importlib

import pytest
import torch

from vllm.platforms import current_platform
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as importlib, pytest, torch; and vLLM components like vllm.platforms, vllm.utils.torch_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 importlib、pytest、torch；vLLM 内部组件，例如 vllm.platforms、vllm.utils.torch_utils。

### Top-level block starting at line 19 (lines 19-20)
```python
if not current_platform.is_rocm():
    pytest.skip("ROCm only", allow_module_level=True)
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Top-level block starting at line 22 (lines 22-22)
```python
pytest.importorskip("triton")
```
**EN:** This top-level `Expr` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `Expr` 代码块执行周边测试所依赖的辅助逻辑。

### Constants and module state (lines 24-30)
```python
device = "cuda"

triton_w4a16_module = importlib.import_module(
    "vllm.model_executor.kernels.linear.mixed_precision.triton_w4a16"
)
triton_w4a16_gemm = triton_w4a16_module.triton_w4a16_gemm
TritonW4A16LinearKernel = triton_w4a16_module.TritonW4A16LinearKernel
```
**EN:** This block prepares module-level state that later helpers and tests reuse.
**CN:** 该代码块准备了后续辅助函数和测试会复用的模块级状态。

### Function `_pack_int4_along_n` (lines 33-43)
```python
def _pack_int4_along_n(w_int4_kn: torch.Tensor) -> torch.Tensor:
    """Pack int4 values along N: [K, N] -> [K, N//8] int32."""
    assert w_int4_kn.dtype == torch.int32
    K, N = w_int4_kn.shape
    assert N % 8 == 0
    shifts = torch.arange(8, device=w_int4_kn.device, dtype=torch.int32) * 4
    return torch.sum(
        (w_int4_kn.view(K, N // 8, 8) & 0xF) << shifts,
        dim=2,
        dtype=torch.int32,
    ).contiguous()
```
**EN:** This helper function implements the shared logic for pack int4 along n. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 pack int4 along n 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `_unpack_int4_along_n` (lines 46-52)
```python
def _unpack_int4_along_n(w_packed_kn8: torch.Tensor) -> torch.Tensor:
    """Unpack int4 values along N: [K, N//8] -> [K, N] int32."""
    assert w_packed_kn8.dtype == torch.int32
    K, N8 = w_packed_kn8.shape
    shifts = torch.arange(8, device=w_packed_kn8.device, dtype=torch.int32) * 4
    nibbles = (w_packed_kn8.unsqueeze(-1) >> shifts) & 0xF
    return nibbles.reshape(K, N8 * 8)
```
**EN:** This helper function implements the shared logic for unpack int4 along n. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 unpack int4 along n 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `_pack_int4_along_k_to_ckpt` (lines 55-63)
```python
def _pack_int4_along_k_to_ckpt(w_int4_kn: torch.Tensor) -> torch.Tensor:
    """Pack int4 values along K into CT checkpoint layout: [K,N] -> [N, K//8]."""
    assert w_int4_kn.dtype == torch.int32
    K, N = w_int4_kn.shape
    assert K % 8 == 0
    out = torch.zeros((N, K // 8), dtype=torch.int32, device=w_int4_kn.device)
    for i in range(8):
        out |= (w_int4_kn[i::8, :].t() & 0xF) << (i * 4)
    return out.contiguous()
```
**EN:** This helper function implements the shared logic for pack int4 along k to ckpt. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 pack int4 along k to ckpt 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `_w4a16_reference` (lines 66-107)
```python
def _w4a16_reference(
    a_mk: torch.Tensor,
    b_packed_kn8: torch.Tensor,
    scales_gn: torch.Tensor,
    *,
    group_size: int,
    qzeros_gn8: torch.Tensor | None,
    zp_bias: int,
) -> torch.Tensor:
    """Reference implementation for W4A16.

    a_mk: [M,K] fp16/bf16
    b_packed_kn8: [K, N//8] int32, N-packed int4 weights
    scales_gn: [K//G, N] fp16/bf16
    qzeros_gn8: [K//G, N//8] int32, N-packed int4 zeros, or None
    """
    assert a_mk.dtype in (torch.float16, torch.bfloat16)
    assert b_packed_kn8.dtype == torch.int32
    assert scales_gn.dtype == a_mk.dtype

    M, K = a_mk.shape
    N = b_packed_kn8.shape[1] * 8
    assert b_packed_kn8.shape[0] == K

    assert group_size > 0 and K % group_size == 0
    G = group_size
    num_groups = K // G
    assert scales_gn.shape == (num_groups, N)

    w_int4 = _unpack_int4_along_n(b_packed_kn8)  # [K,N]
    if qzeros_gn8 is None:
        z_full = torch.full((K, N), zp_bias, dtype=torch.int32, device=a_mk.device)
    else:
        assert qzeros_gn8.shape == (num_groups, N // 8)
        z_gn = _unpack_int4_along_n(qzeros_gn8)  # [G,N] in groups
        z_full = z_gn.repeat_interleave(G, dim=0)  # [K,N]

    s_full = scales_gn.repeat_interleave(G, dim=0).to(torch.float32)  # [K,N]
    w_fp = (w_int4 - z_full).to(torch.float32) * s_full  # [K,N]

    out = a_mk.to(torch.float32) @ w_fp  # [M,N]
    return out.to(a_mk.dtype)
```
**EN:** This helper function implements the shared logic for w4a16 reference. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 w4a16 reference 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `test_triton_w4a16_gemm_matches_reference` (lines 110-162)
```python
@pytest.mark.skipif(not current_platform.is_rocm(), reason="ROCm only")
@pytest.mark.parametrize("dtype", [torch.float16, torch.bfloat16])
@pytest.mark.parametrize(
    "M,K,N,G,has_zp",
    [
        (1, 256, 256, 32, False),
        (17, 256, 512, 32, False),
        (32, 512, 256, 64, False),
        (33, 512, 512, 128, False),
        (64, 1024, 256, 256, False),
        (128, 256, 1024, 32, True),
        (64, 512, 512, 64, True),
    ],
)
def test_triton_w4a16_gemm_matches_reference(dtype, M, K, N, G, has_zp):
    if not torch.cuda.is_available():
        pytest.skip("CUDA/HIP device not available")
    if N % 8 != 0 or K % G != 0:
        pytest.skip("Invalid test shape")

    set_random_seed(0)

    a = (0.25 * torch.randn((M, K), device=device, dtype=torch.float32)).to(dtype)
    w_int4 = torch.randint(0, 16, (K, N), device=device, dtype=torch.int32)
    b_packed = _pack_int4_along_n(w_int4)

    scales = (0.05 * torch.rand((K // G, N), device=device, dtype=torch.float32)).to(
        dtype
    )

    qzeros = None
    if has_zp:
        zeros_int4 = torch.randint(0, 16, (K // G, N), device=device, dtype=torch.int32)
        qzeros = _pack_int4_along_n(zeros_int4)

    out = triton_w4a16_gemm(
        a=a,
        b_q=b_packed,
        scales=scales,
        qzeros=qzeros,
        group_size=G,
        zp_bias=8,
    )
    ref = _w4a16_reference(
        a,
        b_packed,
        scales,
        group_size=G,
        qzeros_gn8=qzeros,
        zp_bias=8,
    )

    torch.testing.assert_close(out, ref, rtol=1e-2, atol=1e-2)
```
**EN:** This pytest case verifies triton w4a16 gemm matches reference. It is parameterized across 2 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as dtype, M, K, N. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 triton w4a16 gemm matches reference 的行为。 它通过 2 组参数化输入覆盖多种场景；它会使用诸如 dtype、M、K、N 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

### Function `test_triton_w4a16_gemm_requires_contiguous_inputs` (lines 165-185)
```python
@pytest.mark.skipif(not current_platform.is_rocm(), reason="ROCm only")
def test_triton_w4a16_gemm_requires_contiguous_inputs():
    if not torch.cuda.is_available():
        pytest.skip("CUDA/HIP device not available")

    set_random_seed(0)
    M, K, N, G = 32, 256, 256, 32
    a = torch.randn((K, M), device=device, dtype=torch.float16).t()  # non-contiguous
    w_int4 = torch.randint(0, 16, (K, N), device=device, dtype=torch.int32)
    b_packed = _pack_int4_along_n(w_int4)
    scales = torch.rand((K // G, N), device=device, dtype=torch.float16)

    with pytest.raises(AssertionError):
        triton_w4a16_gemm(
            a=a,
            b_q=b_packed,
            scales=scales,
            qzeros=None,
            group_size=G,
            zp_bias=8,
        )
```
**EN:** This pytest case verifies triton w4a16 gemm requires contiguous inputs. unsupported hardware, backend, or configuration combinations are skipped early. the expected failure path is asserted explicitly. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 triton w4a16 gemm requires contiguous inputs 的行为。 不支持的硬件、后端或配置组合会被提前跳过；代码会显式断言预期的失败路径；代码会控制随机性以保证场景可复现。

### Function `test_triton_w4a16_process_weights_after_loading_repacks_layout` (lines 188-304)
```python
@pytest.mark.skipif(not current_platform.is_rocm(), reason="ROCm only")
def test_triton_w4a16_process_weights_after_loading_repacks_layout():
    if not torch.cuda.is_available():
        pytest.skip("CUDA/HIP device not available")

    from vllm.config import VllmConfig, set_current_vllm_config
    from vllm.distributed import (
        ensure_model_parallel_initialized,
        init_distributed_environment,
    )
    from vllm.model_executor.kernels.linear.mixed_precision.MPLinearKernel import (
        MPLinearLayerConfig,
    )
    from vllm.model_executor.parameter import (
        GroupQuantScaleParameter,
        PackedColumnParameter,
        PackedvLLMParameter,
    )
    from vllm.scalar_type import scalar_types

    with set_current_vllm_config(VllmConfig()):
        init_distributed_environment(
            world_size=1,
            rank=0,
            distributed_init_method="tcp://127.0.0.1:0",
            local_rank=0,
        )
        ensure_model_parallel_initialized(1, 1)

    set_random_seed(0)

    # Small-but-nontrivial shapes.
    K, N = 256, 256
    G = 32
    assert K % 8 == 0 and N % 8 == 0 and K % G == 0

    # Build a canonical int4 weight grid then pack into the CT checkpoint layout.
    w_int4_kn = torch.randint(0, 16, (K, N), device=device, dtype=torch.int32)
    w_ckpt_nk8 = _pack_int4_along_k_to_ckpt(w_int4_kn)  # [N, K//8]

# ... excerpt ...
            output_dim=0,
            packed_factor=8,
            packed_dim=1,
        ),
    )
    layer.register_parameter(
        "weight_scale",
        GroupQuantScaleParameter(
            data=scales_ckpt_nkg,
            weight_loader=weight_loader,
            input_dim=1,
            output_dim=0,
        ),
    )
    layer.register_parameter(
        "weight_zero_point",
        PackedColumnParameter(
            data=zeros_ckpt_n8kg,
            weight_loader=weight_loader,
            output_dim=0,
            packed_factor=8,
            packed_dim=0,
        ),
    )

    kernel.process_weights_after_loading(layer)

    # Expected transformed layouts.
    expected_w_kn8 = _pack_int4_along_n(w_int4_kn)  # [K, N//8]
    expected_scales_gn = scales_ckpt_nkg.t().contiguous()  # [K//G, N]
    expected_zeros_gn8 = zeros_ckpt_n8kg.t().contiguous()  # [K//G, N//8]

    assert tuple(layer.weight_packed.shape) == (K, N // 8)
    assert tuple(layer.weight_scale.shape) == (K // G, N)
    assert tuple(layer.weight_zero_point.shape) == (K // G, N // 8)

    torch.testing.assert_close(layer.weight_packed, expected_w_kn8)
    torch.testing.assert_close(layer.weight_scale, expected_scales_gn)
    torch.testing.assert_close(layer.weight_zero_point, expected_zeros_gn8)
```
**EN:** This pytest case verifies triton w4a16 process weights after loading repacks layout. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible. the code exercises an end-to-end vLLM execution path instead of only isolated tensor math. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 triton w4a16 process weights after loading repacks layout 的行为。 不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现；这里测试的是端到端 vLLM 执行路径，而不只是孤立的张量计算。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `importlib`
- `pytest`
- `torch`
- `vllm.platforms -> current_platform`
- `vllm.utils.torch_utils -> set_random_seed`
