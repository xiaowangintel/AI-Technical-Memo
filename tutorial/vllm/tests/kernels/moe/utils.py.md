# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Support module for kernel tests in kernels / moe / utils; it provides reference logic, fixtures, or utilities reused by multiple scenarios. / kernels / moe / utils 对应的内核测试支撑模块；它提供会被多个场景复用的参考逻辑、fixture 或工具函数。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-37)
```python
import torch

import vllm._custom_ops as ops
from tests.kernels.quant_utils import per_block_cast_to_int8
from tests.kernels.quantization.nvfp4_utils import FLOAT4_E2M1_MAX, FLOAT8_E4M3_MAX
from vllm.model_executor.layers.activation import SiluAndMul
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.all2all_utils import (
    maybe_make_prepare_finalize,
)
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEParallelConfig,
    FusedMoEQuantConfig,
    RoutingMethodType,
)
from vllm.model_executor.layers.fused_moe.experts.fused_batched_moe import (
    BatchedTritonExperts,
    NaiveBatchedExperts,
)
from vllm.model_executor.layers.fused_moe.experts.triton_moe import (
    TritonExperts,
)
from vllm.model_executor.layers.fused_moe.fused_moe import (
    fused_experts,
)
from vllm.model_executor.layers.fused_moe.modular_kernel import FusedMoEKernel
from vllm.model_executor.layers.fused_moe.prepare_finalize.batched import (
    BatchedPrepareAndFinalize,
)
from vllm.model_executor.layers.fused_moe.router.fused_topk_router import fused_topk
from vllm.model_executor.layers.fused_moe.utils import moe_kernel_quantize_input
from vllm.utils.deep_gemm import per_block_cast_to_fp8
from vllm.utils.math_utils import round_up
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as torch; shared test helpers from tests.kernels.quant_utils, tests.kernels.quantization.nvfp4_utils; and vLLM components like vllm._custom_ops, vllm.model_executor.layers.activation, vllm.model_executor.layers.fused_moe.activation, vllm.model_executor.layers.fused_moe.all2all_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 torch；共享测试辅助模块，例如 tests.kernels.quant_utils、tests.kernels.quantization.nvfp4_utils；vLLM 内部组件，例如 vllm._custom_ops、vllm.model_executor.layers.activation、vllm.model_executor.layers.fused_moe.activation、vllm.model_executor.layers.fused_moe.all2all_utils。

### Function `shuffle_weight` (lines 40-47)
```python
def shuffle_weight(w: torch.Tensor) -> torch.Tensor:
    """Fold weights to adjacent locations for Triton MoE / SwiGLU kernel layout."""
    shape = w.shape
    n = shape[-1]
    first = w[..., : n // 2]
    second = w[..., n // 2 :]
    stacked = torch.stack((first, second), dim=-1)
    return stacked.reshape(shape)
```
**EN:** This helper function implements the shared logic for shuffle weight. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 shuffle weight 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `make_dummy_moe_config` (lines 50-77)
```python
def make_dummy_moe_config(
    num_experts: int = 1,
    experts_per_token: int = 1,
    hidden_dim: int = 1,
    intermediate_size_per_partition: int = 1,
    in_dtype: torch.dtype = torch.bfloat16,
) -> FusedMoEConfig:
    """
    This is a dummy config for the mk constructor interface
    as most kernels like DeepGEMM, CUTLASSFp4, Triton, MARLIN
    do not actually use this config.

    CUTLASSFp8 needs to set some params for workshapes.
    """
    return FusedMoEConfig(
        num_experts=num_experts,
        experts_per_token=experts_per_token,
        hidden_dim=hidden_dim,
        intermediate_size_per_partition=intermediate_size_per_partition,
        num_local_experts=num_experts,
        num_logical_experts=num_experts,
        moe_parallel_config=FusedMoEParallelConfig.make_no_parallel(),
        activation=MoEActivation.SILU,
        in_dtype=in_dtype,
        device="cuda",
        routing_method=RoutingMethodType.TopK,
        max_num_tokens=512,
    )
```
**EN:** This helper function implements the shared logic for dummy MoE config. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 dummy MoE config 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `triton_moe` (lines 80-104)
```python
def triton_moe(
    a: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    topk_weight: torch.Tensor,
    topk_ids: torch.Tensor,
    w1_scale: torch.Tensor | None = None,
    w2_scale: torch.Tensor | None = None,
    a1_scale: torch.Tensor | None = None,
    a2_scale: torch.Tensor | None = None,
    quant_dtype: torch.dtype | None = None,
    per_act_token_quant=False,
    block_shape: list[int] | None = None,
) -> torch.Tensor:
    quant_config = FusedMoEQuantConfig.make(
        quant_dtype,
        per_act_token_quant=per_act_token_quant,
        block_shape=block_shape,
        w1_scale=w1_scale,
        w2_scale=w2_scale,
        a1_scale=a1_scale,
        a2_scale=a2_scale,
    )

    return fused_experts(a, w1, w2, topk_weight, topk_ids, quant_config=quant_config)
```
**EN:** This helper function implements the shared logic for triton MoE. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 triton MoE 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `batched_moe` (lines 107-158)
```python
def batched_moe(
    a: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    topk_weight: torch.Tensor,
    topk_ids: torch.Tensor,
    w1_scale: torch.Tensor | None = None,
    w2_scale: torch.Tensor | None = None,
    a1_scale: torch.Tensor | None = None,
    a2_scale: torch.Tensor | None = None,
    quant_dtype: torch.dtype | None = None,
    per_act_token_quant: bool = False,
    block_shape: list[int] | None = None,
) -> torch.Tensor:
    max_num_tokens = round_up(a.shape[0], 64)

    quant_config = FusedMoEQuantConfig.make(
        quant_dtype,
        per_act_token_quant=per_act_token_quant,
        block_shape=block_shape,
        w1_scale=w1_scale,
        w2_scale=w2_scale,
        a1_scale=a1_scale,
        a2_scale=a2_scale,
    )

    moe_config = make_dummy_moe_config()

    fused_experts = FusedMoEKernel(
        BatchedPrepareAndFinalize(
            max_num_tokens, num_dispatchers=1, num_local_experts=w1.shape[0], rank=0
        ),
        BatchedTritonExperts(
            max_num_tokens=max_num_tokens,
            num_dispatchers=1,
            quant_config=quant_config,
            moe_config=moe_config,
        ),
        inplace=False,
    )

    return fused_experts.apply(
        a,
        w1,
        w2,
        topk_weight,
        topk_ids,
        global_num_experts=w1.shape[0],
        activation=moe_config.activation,
        apply_router_weight_on_input=False,
        expert_map=None,
    )
```
**EN:** This helper function implements the shared logic for batched MoE. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 batched MoE 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `naive_batched_moe` (lines 161-211)
```python
def naive_batched_moe(
    a: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    topk_weight: torch.Tensor,
    topk_ids: torch.Tensor,
    w1_scale: torch.Tensor | None = None,
    w2_scale: torch.Tensor | None = None,
    a1_scale: torch.Tensor | None = None,
    a2_scale: torch.Tensor | None = None,
    quant_dtype: torch.dtype | None = None,
    per_act_token_quant: bool = False,
    block_shape: list[int] | None = None,
) -> torch.Tensor:
    max_num_tokens = round_up(a.shape[0], 64)

    quant_config = FusedMoEQuantConfig.make(
        quant_dtype,
        per_act_token_quant=per_act_token_quant,
        block_shape=block_shape,
        w1_scale=w1_scale,
        w2_scale=w2_scale,
        a1_scale=a1_scale,
        a2_scale=a2_scale,
    )
    moe_config = make_dummy_moe_config()

    fused_experts = FusedMoEKernel(
        BatchedPrepareAndFinalize(
            max_num_tokens, num_dispatchers=1, num_local_experts=w1.shape[0], rank=0
        ),
        NaiveBatchedExperts(
            max_num_tokens=max_num_tokens,
            num_dispatchers=1,
            quant_config=quant_config,
            moe_config=moe_config,
        ),
        inplace=False,
    )

    return fused_experts.apply(
        a,
        w1,
        w2,
        topk_weight,
        topk_ids,
        global_num_experts=w1.shape[0],
        activation=moe_config.activation,
        apply_router_weight_on_input=False,
        expert_map=None,
    )
```
**EN:** This helper function implements the shared logic for naive batched MoE. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 naive batched MoE 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `chunk_scales` (lines 214-222)
```python
def chunk_scales(
    scales: torch.Tensor | None, start: int, end: int
) -> torch.Tensor | None:
    if scales is not None:
        if scales.numel() == 1:
            return scales
        else:
            return scales[start:end]
    return None
```
**EN:** This helper function implements the shared logic for chunk scales. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 chunk scales 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `make_quantized_test_activations` (lines 225-253)
```python
def make_quantized_test_activations(
    E: int,
    m: int,
    k: int,
    in_dtype: torch.dtype,
    quant_dtype: torch.dtype | None = None,
    block_shape: list[int] | None = None,
    per_act_token_quant: bool = False,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor | None]:
    a = torch.randn((E, m, k), device="cuda", dtype=in_dtype) / 10
    a_q = a
    a_scale = None

    if quant_dtype is not None:
        assert quant_dtype == torch.float8_e4m3fn or quant_dtype == torch.int8, (
            "only fp8/int8 supported"
        )
        a_q = torch.zeros_like(a, dtype=quant_dtype)
        a_scale_l = [None] * E
        for e in range(E):
            a_q[e], a_scale_l[e] = moe_kernel_quantize_input(
                a[e], None, quant_dtype, per_act_token_quant, block_shape
            )
        a_scale = torch.stack(a_scale_l)

        if not per_act_token_quant and block_shape is None:
            a_scale = a_scale.view(E, 1, 1)

    return a, a_q, a_scale
```
**EN:** This helper function implements the shared logic for quantized test activations. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 quantized test activations 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `moe_quantize_weights_2d` (lines 256-298)
```python
def moe_quantize_weights_2d(
    w: torch.Tensor,
    w_s: torch.Tensor | None,
    quant_dtype: torch.dtype | str | None,
    per_token_quant: bool,
    block_shape: list[int] | None,
) -> tuple[torch.Tensor, torch.Tensor | None, torch.Tensor | None]:
    assert (
        quant_dtype == torch.float8_e4m3fn
        or quant_dtype == torch.int8
        or quant_dtype == "nvfp4"
    ), "only fp8/int8/nvfp4 supported"

    w_gs = None

    if block_shape is not None:
        assert not per_token_quant
        if quant_dtype == torch.int8:
            w, w_s = per_block_cast_to_int8(w, block_shape)
        elif quant_dtype == torch.float8_e4m3fn:
            w, w_s = per_block_cast_to_fp8(w, block_shape)
        elif quant_dtype == "nvfp4":
            raise RuntimeError("blocked quantization not supported for nvfp4")
        else:
            raise RuntimeError(f"Unsupported quant type {quant_dtype}")
    else:
        if quant_dtype == torch.int8:
            w, w_s = ops.scaled_int8_quant(
                w, w_s, use_per_token_if_dynamic=per_token_quant
            )
        elif quant_dtype == torch.float8_e4m3fn:
            w, w_s = ops.scaled_fp8_quant(
                w, w_s, use_per_token_if_dynamic=per_token_quant
            )
        elif quant_dtype == "nvfp4":
            assert not per_token_quant
            w_amax = torch.abs(w).max().to(torch.float32)
            w_gs = FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX / w_amax
            w, w_s = ops.scaled_fp4_quant(w, w_gs)
        else:
            raise RuntimeError(f"Unsupported quant type {quant_dtype}")

    return w, w_s, w_gs
```
**EN:** This helper function implements the shared logic for MoE quantize weights 2d. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 MoE quantize weights 2d 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `moe_quantize_weights` (lines 301-332)
```python
def moe_quantize_weights(
    w: torch.Tensor,
    w_s: torch.Tensor | None,
    quant_dtype: torch.dtype | str | None,
    per_token_quant: bool,
    block_shape: list[int] | None,
) -> tuple[torch.Tensor, torch.Tensor | None, torch.Tensor | None]:
    assert w.dim() == 3
    e, rows, cols = w.shape
    w_l = [None] * e
    w_s_l = [None] * e
    w_gs_l = [None] * e
    for idx in range(e):
        w_l[idx], w_s_l[idx], w_gs_l[idx] = moe_quantize_weights_2d(
            w[idx], None, quant_dtype, per_token_quant, block_shape
        )

    w = torch.stack(w_l)
    w_s = torch.stack(w_s_l)
    w_gs = torch.stack(w_gs_l) if e > 0 and w_gs_l[0] is not None else None

    if w_s.ndim == 2:
        assert w_s.shape[-1] == 1
        w_s = w_s.view(-1, 1, 1)

    if block_shape is not None:
        block_n, block_k = block_shape
        n_tiles = (rows + block_n - 1) // block_n
        k_tiles = (cols + block_k - 1) // block_k
        assert w_s.shape == (e, n_tiles, k_tiles)

    return w, w_s, w_gs
```
**EN:** This helper function implements the shared logic for MoE quantize weights. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 MoE quantize weights 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `make_test_weight` (lines 335-355)
```python
def make_test_weight(
    e: int,
    rows: int,
    cols: int,
    in_dtype: torch.dtype = torch.bfloat16,
    quant_dtype: torch.dtype | str | None = None,
    block_shape: list[int] | None = None,
    per_out_ch_quant: bool = False,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor | None, torch.Tensor | None]:
    w_16 = torch.randn((e, rows, cols), device="cuda", dtype=in_dtype) / 15

    if quant_dtype is not None:
        w, w_s, w_gs = moe_quantize_weights(
            w_16, None, quant_dtype, per_out_ch_quant, block_shape
        )
    else:
        w = w_16
        w_s = None
        w_gs = None

    return w_16, w, w_s, w_gs
```
**EN:** This helper function implements the shared logic for test weight. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 test weight 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `make_test_weights` (lines 358-382)
```python
def make_test_weights(
    e: int,
    n: int,
    k: int,
    in_dtype: torch.dtype = torch.bfloat16,
    quant_dtype: torch.dtype | str | None = None,
    block_shape: list[int] | None = None,
    per_out_ch_quant: bool = False,
    make_gate: bool = True,
) -> tuple[
    tuple[torch.Tensor, torch.Tensor, torch.Tensor | None, torch.Tensor | None],
    tuple[torch.Tensor, torch.Tensor, torch.Tensor | None, torch.Tensor | None],
]:
    return (
        make_test_weight(
            e,
            (2 if make_gate else 1) * n,
            k,
            in_dtype,
            quant_dtype,
            block_shape,
            per_out_ch_quant,
        ),
        make_test_weight(e, k, n, in_dtype, quant_dtype, block_shape, per_out_ch_quant),
    )
```
**EN:** This helper function implements the shared logic for test weights. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 test weights 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `per_token_cast_to_fp8` (lines 385-395)
```python
def per_token_cast_to_fp8(
    x: torch.Tensor, block_size: int = 128
) -> tuple[torch.Tensor, torch.Tensor]:
    assert x.dim() == 2
    m, n = x.shape
    pad_size = (block_size - (n % block_size)) % block_size
    x = torch.nn.functional.pad(x, (0, pad_size), value=0) if pad_size > 0 else x
    x_view = x.view(m, -1, block_size)
    x_amax = x_view.abs().float().amax(dim=2).view(m, -1).clamp(1e-4)
    fp8_data = (x_view * (448.0 / x_amax.unsqueeze(2))).to(torch.float8_e4m3fn)
    return fp8_data.view(m, n + pad_size)[:, :n], (x_amax / 448.0).view(m, -1)
```
**EN:** This helper function implements the shared logic for per token cast to FP8. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 per token cast to FP8 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `make_test_quant_config` (lines 398-450)
```python
def make_test_quant_config(
    e: int,
    n: int,
    k: int,
    in_dtype: torch.dtype,
    quant_dtype: torch.dtype | str | None = None,
    per_act_token_quant: bool = False,
    block_shape: list[int] | None = None,
    make_gate: bool = True,
    is_scale_swizzled: bool = True,
) -> tuple[torch.Tensor, torch.Tensor, FusedMoEQuantConfig]:
    (_, w1, w1_s, w1_gs), (_, w2, w2_s, w2_gs) = make_test_weights(
        e,
        n,
        k,
        in_dtype,
        quant_dtype,
        per_out_ch_quant=per_act_token_quant,
        block_shape=block_shape,
        make_gate=make_gate,
    )

    # Hacky/trivial scales for nvfp4.
    a1_gscale: torch.Tensor | None = None
    a2_gscale: torch.Tensor | None = None
    if quant_dtype == "nvfp4":
        a1_gscale = torch.ones((e,), device="cuda", dtype=torch.float32)
        a2_gscale = torch.ones((e,), device="cuda", dtype=torch.float32)
        a1_scale = a1_gscale
        a2_scale = a2_gscale
    else:
        a1_scale = None
        a2_scale = None

    return (
        w1,
        w2,
        FusedMoEQuantConfig.make(
            quant_dtype,
            per_act_token_quant=per_act_token_quant,
            block_shape=block_shape,
            w1_scale=w1_s,
            w2_scale=w2_s,
            a1_gscale=a1_gscale,
            a2_gscale=a2_gscale,
            a1_scale=a1_scale,
            a2_scale=a2_scale,
            # TODO: make sure this is handled properly
            g1_alphas=(1 / w1_gs) if w1_gs is not None else None,
            g2_alphas=(1 / w2_gs) if w2_gs is not None else None,
            is_scale_swizzled=is_scale_swizzled,
        ),
    )
```
**EN:** This helper function implements the shared logic for test quant config. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 test quant config 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `fused_moe` (lines 453-476)
```python
def fused_moe(
    hidden_states: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    score: torch.Tensor,
    topk: int,
    renormalize: bool = False,
    quant_config: FusedMoEQuantConfig | None = None,
    global_num_experts: int = -1,
    expert_map: torch.Tensor | None = None,
) -> torch.Tensor:
    topk_weights, topk_ids, _ = fused_topk(
        hidden_states, score.float(), topk, renormalize
    )
    return fused_experts(
        hidden_states,
        w1,
        w2,
        topk_weights,
        topk_ids,
        global_num_experts=global_num_experts,
        expert_map=expert_map,
        quant_config=quant_config,
    )
```
**EN:** This helper function implements the shared logic for fused MoE. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 fused MoE 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `BaselineMM` (lines 479-479)
```python
class BaselineMM(torch.nn.Module):
```
**EN:** This helper class groups the state and behavior needed for BaselineMM. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 BaselineMM 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `BaselineMM.__init__` (lines 480-487)
```python
    def __init__(
        self,
        b: torch.Tensor,
        out_dtype: torch.dtype,
    ):
        super().__init__()
        self.b = torch.nn.Parameter(b.to(dtype=torch.float32))
        self.out_dtype = out_dtype
```
**EN:** This method implements the initialization for `BaselineMM`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `BaselineMM` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `BaselineMM.forward` (lines 489-490)
```python
    def forward(self, a: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor | None]:
        return torch.mm(a.to(dtype=torch.float32), self.b).to(self.out_dtype), None
```
**EN:** This method on `BaselineMM` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `BaselineMM` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `BaselineSiluAndMul` (lines 493-493)
```python
class BaselineSiluAndMul(torch.nn.Module):
```
**EN:** This helper class groups the state and behavior needed for BaselineSiluAndMul. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 BaselineSiluAndMul 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `BaselineSiluAndMul.__init__` (lines 494-495)
```python
    def __init__(self):
        super().__init__()
```
**EN:** This method implements the initialization for `BaselineSiluAndMul`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `BaselineSiluAndMul` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `BaselineSiluAndMul.forward` (lines 497-499)
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        d = x.shape[-1] // 2
        return torch.nn.functional.silu(x[..., :d]) * x[..., d:]
```
**EN:** This method on `BaselineSiluAndMul` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `BaselineSiluAndMul` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `TestMLP` (lines 502-502)
```python
class TestMLP(torch.nn.Module):
```
**EN:** This helper class groups the state and behavior needed for TestMLP. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestMLP 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestMLP.__init__` (lines 503-512)
```python
    def __init__(
        self,
        w1: torch.Tensor,
        w2: torch.Tensor,
        out_dtype: torch.dtype,
    ):
        super().__init__()
        self.gate_up_proj = BaselineMM(w1, out_dtype)
        self.down_proj = BaselineMM(w2, out_dtype)
        self.act_fn = BaselineSiluAndMul()
```
**EN:** This method implements the initialization for `TestMLP`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `TestMLP` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `TestMLP.forward` (lines 514-518)
```python
    def forward(self, x):
        x, _ = self.gate_up_proj(x)
        x = self.act_fn(x)
        x, _ = self.down_proj(x)
        return x
```
**EN:** This method on `TestMLP` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestMLP` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `make_naive_shared_experts` (lines 521-528)
```python
def make_naive_shared_experts(
    N: int,
    K: int,
    in_dtype: torch.dtype = torch.bfloat16,
) -> torch.nn.Module:
    w1 = torch.randn((K, N * 2), device="cuda", dtype=in_dtype) / 15
    w2 = torch.randn((N, K), device="cuda", dtype=in_dtype) / 15
    return TestMLP(w1, w2, out_dtype=in_dtype)
```
**EN:** This helper function implements the shared logic for naive shared experts. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 naive shared experts 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Class `RealMLP` (lines 531-531)
```python
class RealMLP(torch.nn.Module):
```
**EN:** This helper class groups the state and behavior needed for RealMLP. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 RealMLP 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `RealMLP.__init__` (lines 532-588)
```python
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        w1: torch.Tensor,
        w2: torch.Tensor,
        hidden_act: str = "silu",
        quant_config=None,
        reduce_results: bool = True,
        prefix: str = "",
        w1_s: torch.Tensor | None = None,
        w2_s: torch.Tensor | None = None,
    ) -> None:
        from vllm.model_executor.layers.linear import (
            MergedColumnParallelLinear,
            RowParallelLinear,
        )

        super().__init__()
        self.gate_up_proj = MergedColumnParallelLinear(
            hidden_size,
            [intermediate_size] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.gate_up_proj",
        )
        self.gate_up_proj.register_parameter(
            "weight", torch.nn.Parameter(w1, requires_grad=False)
        )
        self.gate_up_proj.register_parameter(
            "weight_scale", torch.nn.Parameter(w1_s, requires_grad=False)
        )
        self.gate_up_proj.register_parameter(
            "input_scale", None
        )  # torch.nn.Parameter(None, requires_grad=False))
        self.down_proj = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=False,
            quant_config=quant_config,
            reduce_results=reduce_results,
            prefix=f"{prefix}.down_proj",
        )
        self.down_proj.register_parameter(
            "weight", torch.nn.Parameter(w2, requires_grad=False)
        )
        self.down_proj.register_parameter(
            "weight_scale", torch.nn.Parameter(w2_s, requires_grad=False)
        )
        self.down_proj.register_parameter(
            "input_scale", None
        )  # torch.nn.Parameter(None, requires_grad=False))
        if hidden_act != "silu":
            raise ValueError(
                f"Unsupported activation: {hidden_act}. Only silu is supported for now."
            )
        self.act_fn = SiluAndMul()
```
**EN:** This method implements the initialization for `RealMLP`. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 这个方法实现了 `RealMLP` 的初始化逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Method `RealMLP.forward` (lines 590-594)
```python
    def forward(self, x):
        gate_up, _ = self.gate_up_proj(x)
        x = self.act_fn(gate_up)
        x, _ = self.down_proj(x)
        return x
```
**EN:** This method on `RealMLP` implements forward. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `RealMLP` 中的这个方法实现了 forward。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `make_shared_experts_with_weights` (lines 597-619)
```python
def make_shared_experts_with_weights(
    N: int,
    K: int,
    in_dtype: torch.dtype,
    w1: torch.Tensor,
    w2: torch.Tensor,
    w1_s: torch.Tensor | None = None,
    w2_s: torch.Tensor | None = None,
    quant_dtype: torch.dtype | str | None = None,
) -> torch.nn.Module:
    old_dtype = torch.get_default_dtype()
    try:
        torch.set_default_dtype(in_dtype)
        if quant_dtype == torch.float8_e4m3fn:
            from vllm.model_executor.layers.quantization.fp8 import Fp8Config

            quant_config = Fp8Config(True)
        else:
            quant_config = None

        return RealMLP(K, N, w1, w2, "silu", quant_config, w1_s=w1_s, w2_s=w2_s)
    finally:
        torch.set_default_dtype(old_dtype)
```
**EN:** This helper function implements the shared logic for shared experts with weights. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 shared experts with weights 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `modular_triton_fused_moe` (lines 622-635)
```python
def modular_triton_fused_moe(
    moe_config: FusedMoEConfig,
    quant_config: FusedMoEQuantConfig,
) -> FusedMoEKernel:
    return FusedMoEKernel(
        maybe_make_prepare_finalize(
            moe=moe_config,
            quant_config=quant_config,
            allow_new_interface=True,
            use_monolithic=False,
        ),
        TritonExperts(moe_config, quant_config),
        inplace=False,
    )
```
**EN:** This helper function implements the shared logic for modular triton fused MoE. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 modular triton fused MoE 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `make_shared_experts` (lines 638-654)
```python
def make_shared_experts(
    N: int,
    K: int,
    in_dtype: torch.dtype = torch.bfloat16,
    quant_dtype: torch.dtype | str | None = None,
) -> torch.nn.Module:
    (_, w1, w1_s, _), (_, w2, w2_s, _) = make_test_weights(
        1,
        N,
        K,
        in_dtype=in_dtype,
        quant_dtype=quant_dtype,
    )

    return make_shared_experts_with_weights(
        N, K, in_dtype, w1, w2, w1_s=w1_s, w2_s=w2_s, quant_dtype=quant_dtype
    )
```
**EN:** This helper function implements the shared logic for shared experts. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 shared experts 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

## Key Concepts / 关键概念
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。
- **Shared test utilities / 共享测试工具:** The module factors repeated setup or reference logic out of the individual test files. / 该模块把重复的初始化或参考逻辑从单个测试文件中抽离出来复用。

## Dependencies / 依赖关系
- `torch`
- `vllm._custom_ops`
- `tests.kernels.quant_utils -> per_block_cast_to_int8`
- `tests.kernels.quantization.nvfp4_utils -> FLOAT4_E2M1_MAX, FLOAT8_E4M3_MAX`
- `vllm.model_executor.layers.activation -> SiluAndMul`
- `vllm.model_executor.layers.fused_moe.activation -> MoEActivation`
- `vllm.model_executor.layers.fused_moe.all2all_utils -> maybe_make_prepare_finalize`
- `vllm.model_executor.layers.fused_moe.config -> FusedMoEConfig, FusedMoEParallelConfig, FusedMoEQuantConfig, RoutingMethodType`
- `vllm.model_executor.layers.fused_moe.experts.fused_batched_moe -> BatchedTritonExperts, NaiveBatchedExperts`
- `vllm.model_executor.layers.fused_moe.experts.triton_moe -> TritonExperts`
- `vllm.model_executor.layers.fused_moe.fused_moe -> fused_experts`
- `vllm.model_executor.layers.fused_moe.modular_kernel -> FusedMoEKernel`
- `vllm.model_executor.layers.fused_moe.prepare_finalize.batched -> BatchedPrepareAndFinalize`
- `vllm.model_executor.layers.fused_moe.router.fused_topk_router -> fused_topk`
- `vllm.model_executor.layers.fused_moe.utils -> moe_kernel_quantize_input`
- `vllm.utils.deep_gemm -> per_block_cast_to_fp8`
