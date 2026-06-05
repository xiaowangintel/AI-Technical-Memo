# router.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/router.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `fused_moe_router_cudacore_kernel`, `fused_moe_router_cudacore`, `fused_moe_router_tensorcore_kernel`, and `fused_moe_router_tensorcore` and connects them to backend-specific paths such as `CUDA`, `Triton`, and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行。它提供了 `fused_moe_router_cudacore_kernel`、`fused_moe_router_cudacore`、`fused_moe_router_tensorcore_kernel` 以及 `fused_moe_router_tensorcore` 等符号，并把这些符号连接到 `CUDA`、`Triton` 以及 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Imports, constants, and runtime setup
```python
from typing import Optional, Tuple

import torch
import triton
import triton.language as tl

from sglang.srt.layers.moe.topk import fused_topk
from sglang.srt.utils import is_hip

_is_hip = is_hip()
```
**EN:** This section prepares the module namespace. It imports `typing.Optional`, `typing.Tuple`, `torch`, `triton`, `triton.language`, and `sglang.srt.layers.moe.topk.fused_topk`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_is_hip` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `typing.Optional`、`typing.Tuple`、`torch`、`triton`、`triton.language` 以及 `sglang.srt.layers.moe.topk.fused_topk`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_is_hip` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 13-116: Function `fused_moe_router_cudacore_kernel` and its core logic
```python
@triton.jit
def fused_moe_router_cudacore_kernel(
    input_ptr,  # input (bs, hidden_dim)
    moe_router_weight_ptr,  # input (num_experts, hidden_dim)
    topk_weights_ptr,  # output (bs, topk)
    topk_ids_ptr,  # output (bs, topk)
    correction_bias_ptr,
    is_correction_bias: tl.constexpr,
    num_experts: tl.constexpr,
    topk: tl.constexpr,
    moe_softcapping: tl.constexpr,
    moe_renormalize: tl.constexpr,  # not supported
    hidden_dim: tl.constexpr,
    BLOCK_SIZE: tl.constexpr,
):
    pid = tl.program_id(axis=0)

    offsets = tl.arange(0, BLOCK_SIZE)
    mask = offsets < hidden_dim

    # moe_router_weight is k major
    expert_offsets = tl.arange(0, num_experts)[:, None]
    router_mask = mask[None, :]
    w_router = tl.load(
        moe_router_weight_ptr + expert_offsets * hidden_dim + offsets[None, :],
        mask=router_mask,
        other=0.0,
    )

    x = tl.load(input_ptr + pid * hidden_dim + offsets, mask=mask, other=0.0)

    # todo: tl.dot?
    logits = tl.sum((w_router.to(tl.float32) * x[None, :].to(tl.float32)), axis=-1)

    # logit softcap
    if moe_softcapping == 0:
        logits_softcapped = logits
    else:
        logits_scaled = logits / moe_softcapping
        exped = tl.exp(2 * logits_scaled)
        top = exped - 1
        bottom = exped + 1
        logits_softcapped = top / bottom * moe_softcapping

    # Add bias after softcapping
    if is_correction_bias:
        bias = tl.load(correction_bias_ptr + tl.arange(0, num_experts))
        logits_softcapped = logits_softcapped + bias

    # topk
    # assert 1 <= topk <= num_experts

    # 5.38 us

    top1 = tl.argmax(logits_softcapped, axis=0)
    tl.store(topk_ids_ptr + pid * topk + 0, top1)  # 5.63 us

    top1_v = tl.max(logits_softcapped, axis=0)
    invsumexp = 1.0 / tl.sum(tl.exp(logits_softcapped - top1_v), axis=0)

    tl.store(
        topk_weights_ptr + pid * topk + 0,
        invsumexp,
    )  # 5.73 us

    if topk >= 2:
        top2 = tl.argmax(
            tl.where(
                tl.arange(0, num_experts) != top1, logits_softcapped, float("-inf")
            ),
            axis=0,
        )
        tl.store(topk_ids_ptr + pid * topk + 1, top2)
        top2_v = tl.sum(logits_softcapped * (tl.arange(0, num_experts) == top2), axis=0)
        tl.store(
            topk_weights_ptr + pid * topk + 1,
            tl.exp(top2_v - top1_v) * invsumexp,
        )  # 5.95us

    # probably slow
    if topk > 2:
        topk_mask = tl.full(logits_softcapped.shape, 1.0, dtype=logits_softcapped.dtype)
        topk_mask = tl.where(
            tl.arange(0, num_experts) != top1, topk_mask, float("-inf")
        )
        topk_mask = tl.where(
            tl.arange(0, num_experts) != top2, topk_mask, float("-inf")
        )
        for i in range(2, topk):
            topi = tl.argmax(logits_softcapped + topk_mask, axis=0)
            topk_mask = tl.where(
                tl.arange(0, num_experts) != topi, topk_mask, float("-inf")
            )
            tl.store(topk_ids_ptr + pid * topk + i, topi)
            topi_v = tl.sum(
                logits_softcapped * (tl.arange(0, num_experts) == topi), axis=0
            )
            tl.store(
                topk_weights_ptr + pid * topk + i,
                tl.exp(topi_v - top1_v) * invsumexp,
            )
    # assert not moe_renormalize, "moe weight renormalization not implemented"
```
**EN:** This block defines `fused_moe_router_cudacore_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.arange`, `tl.load`, `tl.sum`, and `tl.argmax`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid`, `offsets`, `mask`, `expert_offsets`, and `router_mask` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `fused_moe_router_cudacore_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.arange`、`tl.load`、`tl.sum` 以及 `tl.argmax`，说明该流程会编排底层辅助函数或计算内核。 像 `pid`、`offsets`、`mask`、`expert_offsets` 以及 `router_mask` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 117-158: Function `fused_moe_router_cudacore` and its core logic
```python
def fused_moe_router_cudacore(
    x: torch.Tensor,
    router_weight: torch.Tensor,
    topk: int,
    moe_softcapping: float,
    correction_bias: Optional[torch.Tensor] = None,
):
    assert len(x.shape) == 2 and x.shape[1] == router_weight.shape[1]
    bs, hidden_dim = x.shape
    num_experts = router_weight.shape[0]

    # router_logits = torch.empty((bs, num_experts), dtype=torch.float32, device=x.device)
    topk_weights = torch.empty((bs, topk), dtype=torch.float32, device=x.device)
    topk_ids = torch.empty((bs, topk), dtype=torch.int32, device=x.device)
    is_correction_bias = correction_bias is not None

    max_warps = 16 if _is_hip else 32
    config = {
        "BLOCK_SIZE": triton.next_power_of_2(hidden_dim),
        "num_warps": max(
            min(triton.next_power_of_2(triton.cdiv(hidden_dim, 256)), max_warps), 4
        ),
    }

    fused_moe_router_cudacore_kernel[(bs,)](
        x,
        router_weight,
        topk_weights,
        topk_ids,
        correction_bias,
        is_correction_bias=is_correction_bias,
        num_experts=num_experts,
        topk=topk,
        moe_softcapping=moe_softcapping,
        moe_renormalize=False,
        hidden_dim=hidden_dim,
        **config,
    )

    return topk_weights, topk_ids
```
**EN:** This block defines `fused_moe_router_cudacore` and contains the main logic for this step. It mainly invokes `torch.empty`, `fused_moe_router_cudacore_kernel`, `triton.next_power_of_2`, `max`, and `len`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `bs`, `hidden_dim`, `num_experts`, `topk_weights`, and `topk_ids` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `fused_moe_router_cudacore`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty`、`fused_moe_router_cudacore_kernel`、`triton.next_power_of_2`、`max` 以及 `len`，说明该流程会编排底层辅助函数或计算内核。 像 `bs`、`hidden_dim`、`num_experts`、`topk_weights` 以及 `topk_ids` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 159-285: Function `fused_moe_router_tensorcore_kernel` and its core logic
```python
@triton.jit
def fused_moe_router_tensorcore_kernel(
    a_ptr,  # input (bs, hidden_dim)
    b_ptr,  # input (num_experts, hidden_dim)
    topk_weights_ptr,  # output (bs, topk)
    topk_ids_ptr,  # output (bs, topk)
    bs,
    num_experts: tl.constexpr,
    topk: tl.constexpr,  # only support topk <= 2
    moe_softcapping: tl.constexpr,
    moe_renormalize: tl.constexpr,  # not supported
    correction_bias_ptr,
    is_correction_bias: tl.constexpr,
    K: tl.constexpr,
    BLOCK_SIZE_M: tl.constexpr,
    BLOCK_SIZE_N: tl.constexpr,
    BLOCK_SIZE_K: tl.constexpr,
    stride_am: tl.constexpr,
    stride_bn: tl.constexpr,
    dp_attn_workaround_flag: tl.constexpr,
):

    # 1. get block id
    pid = tl.program_id(axis=0)

    # 2. create pointers for the first block of A and B
    # 2.1. setup a_ptrs with offsets in m and k
    offs_m = pid * BLOCK_SIZE_M + tl.arange(0, BLOCK_SIZE_M)[:, None]
    bs_mask = offs_m < bs
    offs_k = tl.arange(0, BLOCK_SIZE_K)[None, :]
    a_ptrs = a_ptr + (offs_m * stride_am + offs_k)

    # 2.2. setup b_ptrs with offsets in k and n.
    #      Note: b matrix is k-major.
    offs_k = tl.arange(0, BLOCK_SIZE_K)[None, :]
    offs_n = tl.arange(0, BLOCK_SIZE_N)[:, None]
    expert_mask = offs_n < num_experts
    b_ptrs = b_ptr + (offs_n * stride_bn + offs_k)

    # 3. Create an accumulator of float32 of size [BLOCK_SIZE_M, BLOCK_SIZE_N]
    #    3.1. iterate in K dimension
    #    3.2. transpose tile B
    acc = tl.zeros((BLOCK_SIZE_M, BLOCK_SIZE_N), dtype=tl.float32)
    for k in range(0, K // BLOCK_SIZE_K):  # hidden_dim % BLOCK_SIZE_K == 0
        a = tl.load(
            a_ptrs,
            mask=bs_mask,
            other=0.0,
        ).to(tl.float32)
        b = tl.load(b_ptrs, mask=expert_mask, other=0.0).to(tl.float32).T
        acc += tl.dot(a, b)

        # Advance the ptrs to the next K block.
        a_ptrs += BLOCK_SIZE_K
        b_ptrs += BLOCK_SIZE_K

    # 4. logit softcap
    if moe_softcapping == 0:
        logits_softcapped = acc
    else:
        logits_scaled = acc / moe_softcapping
        exped = tl.exp(2 * logits_scaled)
        logits_softcapped = (exped - 1) / (exped + 1) * moe_softcapping

    # Add bias after softcapping
    if is_correction_bias:
        bias = tl.load(
            correction_bias_ptr + tl.arange(0, BLOCK_SIZE_N)[None, :],
            mask=expert_mask.T,
            other=0.0,
        )
        logits_softcapped = logits_softcapped + bias

    if dp_attn_workaround_flag:
        logits_softcapped = tl.where(
            logits_softcapped != logits_softcapped, -1e9, logits_softcapped
        )

    # 5. top1
    arange_block_size_n = tl.arange(0, BLOCK_SIZE_N)[None, :]
    cond_top1 = arange_block_size_n < num_experts
    top1 = tl.argmax(tl.where(cond_top1, logits_softcapped, float("-inf")), axis=1)
    top1_v = tl.max(
        tl.where(cond_top1, logits_softcapped, float("-inf")), axis=1, keep_dims=True
    )
    top1_invsumexp = 1.0 / tl.sum(
        tl.where(cond_top1, tl.exp(logits_softcapped - top1_v), 0.0), axis=1
    )

    # 6. store top1 to output
    offs_top1 = pid * topk * BLOCK_SIZE_M + topk * tl.arange(0, BLOCK_SIZE_M)
    top1_mask = offs_top1 < bs * topk
    tl.store(topk_ids_ptr + offs_top1, top1, mask=top1_mask)
    tl.store(
        topk_weights_ptr + offs_top1,
        top1_invsumexp,
        mask=top1_mask,
    )

    # 7. handle topk == 2
    if topk == 2:
        cond_top2 = (arange_block_size_n < num_experts) & (
            arange_block_size_n != top1[:, None]
        )
        top2 = tl.argmax(
            tl.where(cond_top2, logits_softcapped, float("-inf")),
            axis=1,
            keep_dims=True,
        )
        top2_v = tl.sum(
            logits_softcapped * (arange_block_size_n == top2), axis=1, keep_dims=True
        )
        top2_invsumexp = tl.exp(top2_v - top1_v) * top1_invsumexp[:, None]

        # store top2
        offs_top2 = (
            pid * topk * BLOCK_SIZE_M + topk * tl.arange(0, BLOCK_SIZE_M)[:, None] + 1
        )
        top2_mask = offs_top2 < bs * topk
        tl.store(topk_ids_ptr + offs_top2, top2, mask=top2_mask)
        tl.store(
            topk_weights_ptr + offs_top2,
            top2_invsumexp,
            mask=top2_mask,
        )
```
**EN:** This block defines `fused_moe_router_tensorcore_kernel` and contains the main logic for this step. Decorators like `triton.jit` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `tl.program_id`, `tl.zeros`, `range`, `tl.argmax`, and `tl.max`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `pid`, `offs_m`, `bs_mask`, `offs_k`, and `a_ptrs` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `fused_moe_router_tensorcore_kernel`，并承载这一阶段的核心逻辑。 像 `triton.jit` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `tl.program_id`、`tl.zeros`、`range`、`tl.argmax` 以及 `tl.max`，说明该流程会编排底层辅助函数或计算内核。 像 `pid`、`offs_m`、`bs_mask`、`offs_k` 以及 `a_ptrs` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 286-339: Function `fused_moe_router_tensorcore` and its core logic
```python
def fused_moe_router_tensorcore(
    x: torch.Tensor,
    router_weight: torch.Tensor,
    topk: int,
    moe_softcapping: float,
    BLOCK_SIZE_M: int,
    BLOCK_SIZE_N: int,
    BLOCK_SIZE_K: int,
    correction_bias: Optional[torch.Tensor] = None,
):
    assert len(x.shape) == 2 and x.shape[1] == router_weight.shape[1]
    bs, hidden_dim = x.shape
    num_experts = router_weight.shape[0]

    assert num_experts <= BLOCK_SIZE_N
    assert hidden_dim % BLOCK_SIZE_K == 0
    assert topk <= 2

    topk_weights = torch.empty((bs, topk), dtype=torch.float32, device=x.device)
    topk_ids = torch.empty((bs, topk), dtype=torch.int32, device=x.device)
    is_correction_bias = correction_bias is not None

    grid = (triton.cdiv(bs, BLOCK_SIZE_M) * triton.cdiv(num_experts, BLOCK_SIZE_N),)

    # TODO(ch-wan): temporary workaround for dp attention. We should support masked
    # router to skip padded tokens.
    from sglang.srt.layers.dp_attention import is_dp_attention_enabled

    dp_attn_workaround_flag = is_dp_attention_enabled()

    fused_moe_router_tensorcore_kernel[grid](
        a_ptr=x,
        b_ptr=router_weight,
        topk_weights_ptr=topk_weights,
        topk_ids_ptr=topk_ids,
        bs=bs,
        num_experts=num_experts,
        topk=topk,
        moe_softcapping=moe_softcapping,
        moe_renormalize=False,
        K=hidden_dim,
        correction_bias_ptr=correction_bias,
        is_correction_bias=is_correction_bias,
        BLOCK_SIZE_M=BLOCK_SIZE_M,
        BLOCK_SIZE_N=BLOCK_SIZE_N,
        BLOCK_SIZE_K=BLOCK_SIZE_K,
        stride_am=hidden_dim,
        stride_bn=hidden_dim,
        dp_attn_workaround_flag=dp_attn_workaround_flag,
    )

    return topk_weights, topk_ids
```
**EN:** This block defines `fused_moe_router_tensorcore` and contains the main logic for this step. It mainly invokes `torch.empty`, `is_dp_attention_enabled`, `fused_moe_router_tensorcore_kernel`, `len`, and `triton.cdiv`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `bs`, `hidden_dim`, `num_experts`, `topk_weights`, and `topk_ids` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `fused_moe_router_tensorcore`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty`、`is_dp_attention_enabled`、`fused_moe_router_tensorcore_kernel`、`len` 以及 `triton.cdiv`，说明该流程会编排底层辅助函数或计算内核。 像 `bs`、`hidden_dim`、`num_experts`、`topk_weights` 以及 `topk_ids` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 340-391: Function `fused_moe_router_shim` and its core logic
```python
def fused_moe_router_shim(
    moe_softcapping,
    hidden_states,
    gating_output,
    topk,
    renormalize,
    correction_bias: Optional[torch.Tensor] = None,
    enable_deterministic_inference: bool = False,
):
    assert not renormalize
    assert (
        len(hidden_states.shape) == 2
        and hidden_states.shape[1] == gating_output.shape[1]
    )
    bs, hidden_dim = hidden_states.shape
    num_experts = gating_output.shape[0]

    BLOCK_SIZE_M = 32

    BLOCK_SIZE_N = max(num_experts, 16)
    BLOCK_SIZE_K = (
        256 if num_experts < 256 else 64
    )  # if experts are large, need to use smaller k block or shared memory OOM

    if (
        (bs >= 512 or num_experts > 8)
        and hidden_dim % BLOCK_SIZE_K == 0
        # we keep using single kernel to avoid non-deterministic behavior
        and not enable_deterministic_inference
    ):
        # if large batch size or large expert, use kernel that uses tensorcore in matmul
        return fused_moe_router_tensorcore(
            x=hidden_states,
            router_weight=gating_output,
            topk=topk,
            moe_softcapping=moe_softcapping,
            BLOCK_SIZE_M=BLOCK_SIZE_M,
            BLOCK_SIZE_N=BLOCK_SIZE_N,
            BLOCK_SIZE_K=BLOCK_SIZE_K,
            correction_bias=correction_bias,
        )
    else:
        # if smaller, use kernel that does not use tensorcore in matmul
        return fused_moe_router_cudacore(
            x=hidden_states,
            router_weight=gating_output,
            topk=topk,
            moe_softcapping=moe_softcapping,
            correction_bias=correction_bias,
        )
```
**EN:** This block defines `fused_moe_router_shim` and contains the main logic for this step. It mainly invokes `max`, `fused_moe_router_tensorcore`, `fused_moe_router_cudacore`, and `len`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `bs`, `hidden_dim`, `num_experts`, `BLOCK_SIZE_M`, and `BLOCK_SIZE_N` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `fused_moe_router_shim`，并承载这一阶段的核心逻辑。 它主要调用 `max`、`fused_moe_router_tensorcore`、`fused_moe_router_cudacore` 以及 `len`，说明该流程会编排底层辅助函数或计算内核。 像 `bs`、`hidden_dim`、`num_experts`、`BLOCK_SIZE_M` 以及 `BLOCK_SIZE_N` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 392-392: Class `FusedMoeRouter` declaration and shared state
```python
class FusedMoeRouter:
```
**EN:** This block introduces class `FusedMoeRouter` and the state shared by its methods.
**CN:** 该代码块引入类 `FusedMoeRouter`，并定义其方法共享的状态。

### Lines 393-397: `FusedMoeRouter` initialization and state setup
```python
    def __init__(self, router_linear, topk, moe_softcapping) -> None:
        self.router_linear = router_linear
        self.topk = topk
        self.moe_softcapping = moe_softcapping
```
**EN:** This block defines `FusedMoeRouter.__init__` and contains the main logic for this step. Intermediate names such as `self.router_linear`, `self.topk`, and `self.moe_softcapping` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `FusedMoeRouter.__init__`，并承载这一阶段的核心逻辑。 像 `self.router_linear`、`self.topk` 以及 `self.moe_softcapping` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 398-400: `FusedMoeRouter.__call__` callable execution entry
```python
    def __call__(self, *args, **kwargs):
        return self.forward(*args, **kwargs)
```
**EN:** This block defines `FusedMoeRouter.__call__` and contains the main logic for this step. It mainly invokes `self.forward`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `FusedMoeRouter.__call__`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 401-408: `FusedMoeRouter.forward` main forward path
```python
    def forward(
        self, x: torch.Tensor, residual: torch.Tensor
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        if x.is_cuda:
            return self.forward_cuda(x, residual)
        else:
            return self.forward_vllm(x, residual)
```
**EN:** This block defines `FusedMoeRouter.forward` and contains the main logic for this step. It mainly invokes `self.forward_cuda` and `self.forward_vllm`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `FusedMoeRouter.forward`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward_cuda` 和 `self.forward_vllm`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 409-419: `FusedMoeRouter.forward_cuda` CUDA execution path
```python
    def forward_cuda(
        self, x: torch.Tensor, autotune=False
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        return fused_moe_router_shim(
            moe_softcapping=self.moe_softcapping,
            hidden_states=x,
            gating_output=self.router_linear.weight,
            topk=self.topk,
            renormalize=False,
        )
```
**EN:** This block defines `FusedMoeRouter.forward_cuda` and contains the main logic for this step. It mainly invokes `fused_moe_router_shim`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `FusedMoeRouter.forward_cuda`，并承载这一阶段的核心逻辑。 它主要调用 `fused_moe_router_shim`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 420-428: `FusedMoeRouter.forward_torch` TORCH execution path
```python
    def forward_torch(
        self,
        x: torch.Tensor,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        g = x.float() @ self.router_linear.weight.T.float()

        g = torch.tanh(g.float() / self.moe_softcapping) * self.moe_softcapping

        return fused_topk(x, g, self.topk, False)
```
**EN:** This block defines `FusedMoeRouter.forward_torch` and contains the main logic for this step. It mainly invokes `fused_topk`, `x.float`, `self.router_linear.weight.T.float`, `torch.tanh`, and `g.float`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `g` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `FusedMoeRouter.forward_torch`，并承载这一阶段的核心逻辑。 它主要调用 `fused_topk`、`x.float`、`self.router_linear.weight.T.float`、`torch.tanh` 以及 `g.float`，说明该流程会编排底层辅助函数或计算内核。 像 `g` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `fused_moe_router_cudacore_kernel`, `fused_moe_router_cudacore`, `fused_moe_router_tensorcore_kernel`, `fused_moe_router_tensorcore`, and `fused_moe_router_shim`. / **主要符号**：核心入口包括 `fused_moe_router_cudacore_kernel`、`fused_moe_router_cudacore`、`fused_moe_router_tensorcore_kernel`、`fused_moe_router_tensorcore` 以及 `fused_moe_router_shim`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Projection layers**: Focuses on matrix multiplication, weight layout, and projection-oriented wrappers. / **投影层**：关注矩阵乘法、权重布局与面向投影的包装层。

## Dependencies / 依赖关系
- **Standard library**: `typing.Optional` and `typing.Tuple` / **标准库**：`typing.Optional` 和 `typing.Tuple`
- **Third-party**: `torch`, `triton`, and `triton.language` / **第三方依赖**：`torch`、`triton` 以及 `triton.language`
- **Internal SGLang modules**: `sglang.srt.layers.moe.topk.fused_topk`, `sglang.srt.utils.is_hip`, and `sglang.srt.layers.dp_attention.is_dp_attention_enabled` / **SGLang 内部模块**：`sglang.srt.layers.moe.topk.fused_topk`、`sglang.srt.utils.is_hip` 以及 `sglang.srt.layers.dp_attention.is_dp_attention_enabled`
