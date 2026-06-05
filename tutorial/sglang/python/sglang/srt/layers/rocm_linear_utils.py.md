# rocm_linear_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/rocm_linear_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution with linear projections and GEMM backend integration for the SGLang SRT runtime. It exposes symbols such as `aiter_dsv3_router_gemm` and `get_dsv3_gemm_output_zero_allocator_size` and connects them to backend-specific paths such as `Triton`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行，并结合线性投影与 GEMM 后端集成。它提供了 `aiter_dsv3_router_gemm` 和 `get_dsv3_gemm_output_zero_allocator_size` 等符号，并把这些符号连接到 `Triton` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Imports, constants, and runtime setup
```python
import torch
from aiter.ops.triton.fused_kv_cache import fused_qk_rope_cat_and_cache_mla
from aiter.ops.triton.fused_qk_concat import fused_qk_rope_cat
from aiter.tuned_gemm import tgemm

__all__ = ["fused_qk_rope_cat", "fused_qk_rope_cat_and_cache_mla"]
```
**EN:** This section prepares the module namespace. It imports `torch`, `aiter.ops.triton.fused_kv_cache.fused_qk_rope_cat_and_cache_mla`, `aiter.ops.triton.fused_qk_concat.fused_qk_rope_cat`, and `aiter.tuned_gemm.tgemm`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `__all__` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `torch`、`aiter.ops.triton.fused_kv_cache.fused_qk_rope_cat_and_cache_mla`、`aiter.ops.triton.fused_qk_concat.fused_qk_rope_cat` 以及 `aiter.tuned_gemm.tgemm`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `__all__` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 9-16: Function `aiter_dsv3_router_gemm` and its core logic
```python
def aiter_dsv3_router_gemm(
    hidden_states: torch.Tensor,
    weight: torch.Tensor,
):
    """Use aiter tuned GEMM dispatcher (tgemm.mm) to automatically select the GEMM kernel."""
    return tgemm.mm(hidden_states, weight.detach(), otype=hidden_states.dtype)
```
**EN:** This block defines `aiter_dsv3_router_gemm` and contains the main logic for this step. It mainly invokes `tgemm.mm` and `weight.detach`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `aiter_dsv3_router_gemm`，并承载这一阶段的核心逻辑。 它主要调用 `tgemm.mm` 和 `weight.detach`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 17-25: `get_dsv3_gemm_output_zero_allocator_size` getter for dsv 3 GEMM output zero allocator size
```python
def get_dsv3_gemm_output_zero_allocator_size(
    n_routed_experts: int, num_moe_layers: int, allocate_size: int, embedding_dim: int
):
    if embedding_dim != 7168 or n_routed_experts != 256:
        return 0

    per_layer_size = 256 * (allocate_size + n_routed_experts)

    return num_moe_layers * per_layer_size
```
**EN:** This block defines `get_dsv3_gemm_output_zero_allocator_size` and contains the main logic for this step. Intermediate names such as `per_layer_size` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_dsv3_gemm_output_zero_allocator_size`，并承载这一阶段的核心逻辑。 像 `per_layer_size` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `aiter_dsv3_router_gemm` and `get_dsv3_gemm_output_zero_allocator_size`. / **主要符号**：核心入口包括 `aiter_dsv3_router_gemm` 和 `get_dsv3_gemm_output_zero_allocator_size`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Position encoding**: Describes how rotary embeddings or related position transforms are applied. / **位置编码**：说明如何应用旋转位置编码或相关位置变换。

## Dependencies / 依赖关系
- **Third-party**: `torch`, `aiter.ops.triton.fused_kv_cache.fused_qk_rope_cat_and_cache_mla`, `aiter.ops.triton.fused_qk_concat.fused_qk_rope_cat`, and `aiter.tuned_gemm.tgemm` / **第三方依赖**：`torch`、`aiter.ops.triton.fused_kv_cache.fused_qk_rope_cat_and_cache_mla`、`aiter.ops.triton.fused_qk_concat.fused_qk_rope_cat` 以及 `aiter.tuned_gemm.tgemm`
