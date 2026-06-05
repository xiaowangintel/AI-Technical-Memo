# torch.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/kernels/mhc/torch.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the PyTorch MHC kernel path and wrapper logic. / 实现 PyTorch 的 MHC 内核路径与包装逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-3)
```python
import torch
```
**EN:** This import block loads `torch`, establishing the tensor libraries, backend hooks, and vLLM helpers needed by the rest of `torch.py`.
**CN:** 该导入代码块加载了 `torch`，为 `torch.py` 后续逻辑准备张量库、后端钩子以及 vLLM 辅助工具。

### Function `mhc_pre_torch` (lines 6-91)
```python
def mhc_pre_torch(
    residual: torch.Tensor,
    fn: torch.Tensor,
    hc_scale: torch.Tensor,
    hc_base: torch.Tensor,
    rms_eps: float,
    hc_pre_eps: float,
    hc_sinkhorn_eps: float,
    hc_post_mult_value: float,
    sinkhorn_repeat: int,
    n_splits: int = 1,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    """
    Forward pass for mHC pre block.

    Args:
        residual: shape (..., hc_mult, hidden_size), dtype torch.bfloat16
        fn: shape (hc_mult3, hc_mult * hidden_size), dtype torch.float32
        hc_scale: shape (3,), dtype torch.float32
        hc_base: shape (hc_mult3,), dtype torch.float32
        rms_eps: RMS normalization epsilon
        hc_pre_eps: pre-mix epsilon
        hc_sinkhorn_eps: sinkhorn epsilon
        hc_post_mult_value: post-mix multiplier value
        sinkhorn_repeat: number of sinkhorn iterations
        n_splits: split-k factor;

    Returns:
        post_mix: shape (..., hc_mult), dtype torch.float32
        comb_mix: shape (..., hc_mult, hc_mult), dtype torch.float32
        layer_input: shape (..., hidden_size), dtype torch.bfloat16
    """

    # Validate shapes
    assert residual.dtype == torch.bfloat16
    assert fn.dtype == torch.float32
    assert hc_scale.dtype == torch.float32
    assert hc_base.dtype == torch.float32

    hc_mult = residual.shape[-2]
    hidden_size = residual.shape[-1]
    hc_mult2 = hc_mult * hc_mult
    hc_mult3 = hc_mult * 2 + hc_mult2

    hc_hidden_size = hc_mult * hidden_size
    assert fn.shape[0] == hc_mult3
    assert fn.shape[1] == hc_hidden_size
    assert hc_scale.shape == (3,)
    assert hc_base.shape == (hc_mult3,)

    outer_shape = residual.shape[:-2]

    residual_flat = residual.view(-1, hc_mult, hidden_size)
    num_tokens = residual_flat.shape[0]
    fn_flat = fn

    x = residual_flat.view(num_tokens, hc_mult * hidden_size).to(torch.float32)
    mixes = torch.matmul(x, fn_flat.t())
    sqrsum = x.square().sum(dim=-1, keepdim=True)
    mixes = mixes * torch.rsqrt(sqrsum / (hc_mult * hidden_size) + rms_eps)

    pre_logits = mixes[:, :hc_mult] * hc_scale[0] + hc_base[:hc_mult]
    pre_mix = torch.sigmoid(pre_logits) + hc_pre_eps

    post_logits = (
        mixes[:, hc_mult : 2 * hc_mult] * hc_scale[1] + hc_base[hc_mult : 2 * hc_mult]
    )
    post_mix = torch.sigmoid(post_logits) * hc_post_mult_value

    comb_logits = mixes[:, 2 * hc_mult :].view(num_tokens, hc_mult, hc_mult) * hc_scale[
        2
    ] + hc_base[2 * hc_mult :].view(1, hc_mult, hc_mult)
    comb_mix = torch.softmax(comb_logits, dim=-1) + hc_sinkhorn_eps
    comb_mix = comb_mix / (comb_mix.sum(dim=-2, keepdim=True) + hc_sinkhorn_eps)
    for _ in range(sinkhorn_repeat - 1):
        comb_mix = comb_mix / (comb_mix.sum(dim=-1, keepdim=True) + hc_sinkhorn_eps)
        comb_mix = comb_mix / (comb_mix.sum(dim=-2, keepdim=True) + hc_sinkhorn_eps)

    layer_input = torch.sum(
        pre_mix.unsqueeze(-1) * residual_flat.to(torch.float32), dim=1
    ).to(torch.bfloat16)
    return (
        post_mix.view(*outer_shape, hc_mult, 1),
        comb_mix.view(*outer_shape, hc_mult, hc_mult),
        layer_input.view(*outer_shape, hidden_size),
    )
```
**EN:** This helper implements `mhc_pre_torch`. Forward pass for mHC pre block. Internally it relies on calls such as `view`, `to`, `matmul`, `sum`.
**CN:** 该函数 `mhc_pre_torch` 封装了此模块中的一段关键运行时逻辑，重点处理 mhc pre torch 相关工作。 它内部会调用 `view`, `to`, `matmul`, `sum` 等例程。

### Function `mhc_post_torch` (lines 94-106)
```python
def mhc_post_torch(
    x: torch.Tensor,
    residual: torch.Tensor,
    post_layer_mix: torch.Tensor,
    comb_res_mix: torch.Tensor,
) -> torch.Tensor:
    mixed_residual = torch.einsum(
        "...ij,...ih->...jh",
        comb_res_mix.to(torch.float32),
        residual.to(torch.float32),
    )
    post_term = post_layer_mix.to(torch.float32) * x.unsqueeze(-2).to(torch.float32)
    return (mixed_residual + post_term).to(residual.dtype)
```
**EN:** This helper implements `mhc_post_torch`, encapsulating a focused unit of runtime logic for the module. Internally it relies on calls such as `einsum`, `to`, `unsqueeze`.
**CN:** 该函数 `mhc_post_torch` 封装了此模块中的一段关键运行时逻辑，重点处理 mhc post torch 相关工作。 它内部会调用 `einsum`, `to`, `unsqueeze` 等例程。

## Key Concepts / 关键概念
- **MHC kernels / MHC 内核**
  - **EN:** The implementation provides backend-specific MHC kernels and wrappers.
  - **CN:** 该实现提供面向不同后端的 MHC 内核与包装层。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: None
- **External / 外部依赖**: `torch`
