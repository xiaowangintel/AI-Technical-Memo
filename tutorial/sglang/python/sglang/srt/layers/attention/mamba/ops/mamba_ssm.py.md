# mamba_ssm.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/mamba/ops/mamba_ssm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module collects mamba ssm helper functions used to prepare, validate, or dispatch attention operations in SGLang. / 该模块汇集 mamba ssm 辅助函数，用于在 SGLang 中准备、校验或分发注意力操作。
## Line-by-Line Analysis / 逐行分析
### Lines 9-12: imports
```python
import torch
import triton
import triton.language as tl
from packaging import version
```
**EN:** Imports the external and internal dependencies consumed by the code that follows.
**CN:** 导入后续代码所依赖的外部与内部模块。

### Lines 14-16: module constants
```python
PAD_SLOT_ID = -1

TRITON3 = version.parse(triton.__version__) >= version.parse("3.0.0")
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 18-30: conditional branch
```python
if TRITON3:

    @triton.jit
    def softplus(dt):
        dt = tl.where(dt <= 20.0, tl.math.log(tl.math.exp(dt) + 1), dt)
        return dt

else:

    @triton.jit
    def softplus(dt):
        dt = tl.where(dt <= 20.0, tl.math.log1p(tl.exp(dt)), dt)
        return dt
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 33-297: function _selective_scan_update_kernel
```python
@triton.heuristics({"HAS_DT_BIAS": lambda args: args["dt_bias_ptr"] is not None})
@triton.heuristics({"HAS_D": lambda args: args["D_ptr"] is not None})
@triton.heuristics({"HAS_Z": lambda args: args["z_ptr"] is not None})
@triton.heuristics(
    {
        "HAS_STATE_BATCH_INDICES": lambda args: args["state_batch_indices_ptr"]
        is not None
    }
)
@triton.heuristics(
    {"BLOCK_SIZE_DSTATE": lambda args: triton.next_power_of_2(args["dstate"])}
)
@triton.heuristics(
    {
        "CACHE_INTERMEDIATE_STATES": lambda args: args["intermediate_states_buffer"]
        is not None
    }
)
@triton.heuristics(
    {
        "HAS_EAGLE_TREE_CUSTOM_ATTN_MASK": lambda args: args[
            "retrieve_parent_token_ptr"
        ]
        is not None
    }
)
@triton.heuristics(
    {
        "HAS_INTERMEDIATE_STATE_INDICES": lambda args: args[
            "intermediate_state_indices_ptr"
        ]
        is not None
# ... omitted 221 lines ...
        current_step_idx += 1

        x_ptr += stride_x_T
        dt_ptr += stride_dt_T
        B_ptr += stride_B_T
        C_ptr += stride_C_T
        out_ptr += stride_out_T
        if HAS_Z:
            z_ptr += stride_z_T

    if not DISABLE_STATE_UPDATE:
        tl.store(state_ptrs, state.to(state_ptrs.dtype.element_ty), mask=mask)
```
**EN:** Implements the selective scan update kernel routine used by this attention module.
**CN:** 实现该注意力模块使用的 selective scan update kernel 例程。

### Lines 300-494: function selective_state_update
```python
def selective_state_update(
    state,
    x,
    dt,
    A,
    B,
    C,
    D=None,
    z=None,
    dt_bias=None,
    dt_softplus=False,
    state_batch_indices=None,
    pad_slot_id=PAD_SLOT_ID,
    out=None,
    disable_state_update=False,
    intermediate_states_buffer=None,
    cache_steps=None,
    retrieve_parent_token=None,
    intermediate_state_indices=None,
):
    """
    Argument:
        state: (batch, dim, dstate) or (batch, nheads, dim, dstate)
        x: (batch, dim) or (batch, nheads, dim) for single-token or (batch, T, nheads, dim) for multi-token
        dt: (batch, dim) or (batch, nheads, dim)
        A: (dim, dstate) or (nheads, dim, dstate)
        B: (batch, dstate) or (batch, ngroups, dstate) for single-token or (batch, T, ngroups, dstate) for multi-token
        C: (batch, dstate) or (batch, ngroups, dstate)
        D: (dim,) or (nheads, dim)
        z: (batch, dim) or (batch, nheads, dim)
        dt_bias: (dim,) or (nheads, dim)
        pad_slot_id: int
# ... omitted 151 lines ...
            out.stride(0),
            out.stride(1),
            out.stride(2),
            out.stride(3),
            retrieve_parent_token_strides[0],
            retrieve_parent_token_strides[1],
            dt_softplus,
            tie_hdim,
            BLOCK_SIZE_M,
            DISABLE_STATE_UPDATE=disable_state_update,
            num_warps=num_warps,
        )
```
**EN:** Implements the selective state update routine used by this attention module.
**CN:** 实现该注意力模块使用的 selective state update 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调
- **EN:** Speculative decoding support / **CN:** 投机解码支持

## Dependencies / 依赖关系
- `torch`
- `triton`
- `triton.language`
- `packaging.version`
