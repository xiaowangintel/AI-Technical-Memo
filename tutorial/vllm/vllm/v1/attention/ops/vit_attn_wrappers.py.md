# vit_attn_wrappers.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/ops/vit_attn_wrappers.py`
- **Repository**: vllm-project/vllm
- **Purpose**: This file contains ops for ViT attention to be compatible with torch.compile as there are operations here not supported by torch.compile (for instance, `.item()` in flash attention) Using these ops and wrapping vision blocks with `torch.compile` can speed up throughput in vision models by ~5% relative on H100, and improve token latencies by ~7% (see qwen2_5_vl for example usage) To use these ops, you must have a recent version of PyTorch installed (>= 2.4.0) / 该模块位于 `attention/ops` 子系统，主要围绕 `flash_attn_maxseqlen_wrapper`, `flash_attn_maxseqlen_wrapper_fake`, `vit_flash_attn_wrapper` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""
This file contains ops for ViT attention to be compatible with torch.compile
as there are operations here not supported by torch.compile (for instance,
`.item()` in flash attention)

Using these ops and wrapping vision blocks with `torch.compile` can speed up
throughput in vision models by ~5% relative on H100, and improve token
latencies by ~7% (see qwen2_5_vl for example usage)

To use these ops, you must have a recent version of PyTorch installed (>= 2.4.0)
"""

import einops
import torch
import torch.nn.functional as F

from vllm.platforms import current_platform
from vllm.utils.torch_utils import direct_register_custom_op
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。

### `flash_attn_maxseqlen_wrapper` function / `flash_attn_maxseqlen_wrapper` 函数
```python
def flash_attn_maxseqlen_wrapper(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    batch_size: int,
    is_rocm_aiter: bool,
    fa_version: int | None,
    scale: float | None = None,
    cu_seqlens: torch.Tensor | None = None,
    max_seqlen: torch.Tensor | None = None,
) -> torch.Tensor:
    kwargs = {}
    if is_rocm_aiter:
        from aiter import flash_attn_varlen_func
    else:
        from vllm.v1.attention.backends.fa_utils import flash_attn_varlen_func

        if not current_platform.is_rocm() and fa_version is not None:
            kwargs["fa_version"] = fa_version

    q_len = q.size(1)
    if cu_seqlens is None:
        cu_seqlens = torch.arange(
            0, (batch_size + 1) * q_len, step=q_len, dtype=torch.int32, device=q.device
        )
    max_seqlen = q_len if max_seqlen is None else max_seqlen.item()

    q, k, v = (einops.rearrange(x, "b s ... -> (b s) ...") for x in [q, k, v])
    output = flash_attn_varlen_func(
        q,
        k,
        v,
        cu_seqlens_q=cu_seqlens,
        cu_seqlens_k=cu_seqlens,
        max_seqlen_q=max_seqlen,
        max_seqlen_k=max_seqlen,
        dropout_p=0.0,
        causal=False,
        softmax_scale=scale,
        **kwargs,
    )
    context_layer = einops.rearrange(output, "(b s) h d -> b s h d", b=batch_size)
    return context_layer
```
**EN:** This function implements `flash_attn_maxseqlen_wrapper` within the module. Key calls include `size`, `flash_attn_varlen_func`, `rearrange`, `arange`, `item`, `is_rocm`. The control flow contains 4 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `flash_attn_maxseqlen_wrapper`，其作用域位于the module。 关键调用包括 `size`, `flash_attn_varlen_func`, `rearrange`, `arange`, `item`, `is_rocm`。 控制流包含 4 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `flash_attn_maxseqlen_wrapper_fake` function / `flash_attn_maxseqlen_wrapper_fake` 函数
```python
def flash_attn_maxseqlen_wrapper_fake(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    batch_size: int,
    is_rocm_aiter: bool,
    fa_version: int | None,
    scale: float | None = None,
    cu_seqlens: torch.Tensor | None = None,
    max_seqlen: torch.Tensor | None = None,
) -> torch.Tensor:
    return torch.empty_like(q)
```
**EN:** This function implements `flash_attn_maxseqlen_wrapper_fake` within the module. Key calls include `empty_like`.
**CN:** 该函数会实现 `flash_attn_maxseqlen_wrapper_fake`，其作用域位于the module。 关键调用包括 `empty_like`。

### Module setup / 模块初始化
```python
direct_register_custom_op(
    op_name="flash_attn_maxseqlen_wrapper",
    op_func=flash_attn_maxseqlen_wrapper,
    fake_impl=flash_attn_maxseqlen_wrapper_fake,
)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。

### `vit_flash_attn_wrapper` function / `vit_flash_attn_wrapper` 函数
```python
def vit_flash_attn_wrapper(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    batch_size: int,
    is_rocm_aiter: bool,
    fa_version: int | None,
    scale: float | None = None,
    cu_seqlens: torch.Tensor | None = None,
    max_seqlen: torch.Tensor | None = None,
) -> torch.Tensor:
    return torch.ops.vllm.flash_attn_maxseqlen_wrapper(
        q,
        k,
        v,
        batch_size,
        is_rocm_aiter,
        fa_version,
        scale,
        cu_seqlens,
        max_seqlen,
    )
```
**EN:** This function implements `vit_flash_attn_wrapper` within the module. Key calls include `flash_attn_maxseqlen_wrapper`.
**CN:** 该函数会实现 `vit_flash_attn_wrapper`，其作用域位于the module。 关键调用包括 `flash_attn_maxseqlen_wrapper`。

### `triton_attn_wrapper` function / `triton_attn_wrapper` 函数
```python
def triton_attn_wrapper(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    batch_size: int,
    scale: float | None = None,
    cu_seqlens: torch.Tensor | None = None,
    max_seqlen: torch.Tensor | None = None,
) -> torch.Tensor:
    from vllm.v1.attention.ops.triton_prefill_attention import context_attention_fwd

    q_len = q.size(1)
    if cu_seqlens is None:
        cu_seqlens = torch.arange(
            0, (batch_size + 1) * q_len, step=q_len, dtype=torch.int32, device=q.device
        )
    max_seqlen = q_len if max_seqlen is None else max_seqlen.item()

    q, k, v = (einops.rearrange(x, "b s ... -> (b s) ...") for x in [q, k, v])
    output = torch.empty_like(q)
    context_attention_fwd(
        q,
        k,
        v,
        output,
        b_start_loc=cu_seqlens[:-1],
        b_seq_len=cu_seqlens[1:] - cu_seqlens[:-1],
        max_input_len=max_seqlen,
        is_causal=False,
        sliding_window_q=None,
        sliding_window_k=None,
        softmax_scale=scale,
    )

    context_layer = einops.rearrange(output, "(b s) h d -> b s h d", b=batch_size)
    return context_layer
```
**EN:** This function implements `triton_attn_wrapper` within the module. Key calls include `size`, `empty_like`, `context_attention_fwd`, `rearrange`, `arange`, `item`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `triton_attn_wrapper`，其作用域位于the module。 关键调用包括 `size`, `empty_like`, `context_attention_fwd`, `rearrange`, `arange`, `item`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `triton_attn_wrapper_fake` function / `triton_attn_wrapper_fake` 函数
```python
def triton_attn_wrapper_fake(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    batch_size: int,
    scale: float | None = None,
    cu_seqlens: torch.Tensor | None = None,
    max_seqlen: torch.Tensor | None = None,
) -> torch.Tensor:
    return torch.empty_like(q)
```
**EN:** This function implements `triton_attn_wrapper_fake` within the module. Key calls include `empty_like`.
**CN:** 该函数会实现 `triton_attn_wrapper_fake`，其作用域位于the module。 关键调用包括 `empty_like`。

### Module setup / 模块初始化
```python
direct_register_custom_op(
    op_name="triton_attn_wrapper",
    op_func=triton_attn_wrapper,
    fake_impl=triton_attn_wrapper_fake,
)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。

### `vit_triton_attn_wrapper` function / `vit_triton_attn_wrapper` 函数
```python
def vit_triton_attn_wrapper(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    batch_size: int,
    scale: float | None = None,
    cu_seqlens: torch.Tensor | None = None,
    max_seqlen: torch.Tensor | None = None,
) -> torch.Tensor:
    return torch.ops.vllm.triton_attn_wrapper(
        q,
        k,
        v,
        batch_size,
        scale,
        cu_seqlens,
        max_seqlen,
    )
```
**EN:** This function implements `vit_triton_attn_wrapper` within the module. Key calls include `triton_attn_wrapper`.
**CN:** 该函数会实现 `vit_triton_attn_wrapper`，其作用域位于the module。 关键调用包括 `triton_attn_wrapper`。

### `apply_sdpa` function / `apply_sdpa` 函数
```python
def apply_sdpa(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    scale: float | None = None,
    enable_gqa: bool = False,
) -> torch.Tensor:
    """
    Input shape:
    (batch_size x seq_len x num_heads x head_size)
    """
    q, k, v = (einops.rearrange(x, "b s h d -> b h s d") for x in [q, k, v])
    output = F.scaled_dot_product_attention(
        q, k, v, dropout_p=0.0, scale=scale, enable_gqa=enable_gqa
    )
    output = einops.rearrange(output, "b h s d -> b s h d ")
    return output
```
**EN:** This function implements `apply_sdpa` within the module. The docstring frames it as: Input shape: (batch_size x seq_len x num_heads x head_size) Key calls include `scaled_dot_product_attention`, `rearrange`.
**CN:** 该函数会实现 `apply_sdpa`，其作用域位于the module。 关键调用包括 `scaled_dot_product_attention`, `rearrange`。

### `torch_sdpa_wrapper` function / `torch_sdpa_wrapper` 函数
```python
def torch_sdpa_wrapper(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    scale: float | None = None,
    cu_seqlens: torch.Tensor | None = None,
    enable_gqa: bool = False,
) -> torch.Tensor:
    # Never remove the contiguous logic for ROCm
    # Without it, hallucinations occur with the backend
    if current_platform.is_rocm():
        q = q.contiguous()
        k = k.contiguous()
        v = v.contiguous()

    if cu_seqlens is None:
        return apply_sdpa(q, k, v, scale=scale, enable_gqa=enable_gqa)

    outputs = []

    lens = (cu_seqlens[1:] - cu_seqlens[:-1]).tolist()
    q_chunks = torch.split(q, lens, dim=1)
    k_chunks = torch.split(k, lens, dim=1)
    v_chunks = torch.split(v, lens, dim=1)
    for q_i, k_i, v_i in zip(q_chunks, k_chunks, v_chunks):
        output_i = apply_sdpa(q_i, k_i, v_i, scale=scale, enable_gqa=enable_gqa)
        outputs.append(output_i)
    context_layer = torch.cat(outputs, dim=1)
    return context_layer
```
**EN:** This function implements `torch_sdpa_wrapper` within the module. Key calls include `is_rocm`, `tolist`, `split`, `zip`, `cat`, `contiguous`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `torch_sdpa_wrapper`，其作用域位于the module。 关键调用包括 `is_rocm`, `tolist`, `split`, `zip`, `cat`, `contiguous`。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `torch_sdpa_wrapper_fake` function / `torch_sdpa_wrapper_fake` 函数
```python
def torch_sdpa_wrapper_fake(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    scale: float | None,
    cu_seqlens: torch.Tensor | None,
    enable_gqa: bool = False,
) -> torch.Tensor:
    return torch.empty_like(q)
```
**EN:** This function implements `torch_sdpa_wrapper_fake` within the module. Key calls include `empty_like`.
**CN:** 该函数会实现 `torch_sdpa_wrapper_fake`，其作用域位于the module。 关键调用包括 `empty_like`。

### Module setup / 模块初始化
```python
direct_register_custom_op(
    op_name="torch_sdpa_wrapper",
    op_func=torch_sdpa_wrapper,
    fake_impl=torch_sdpa_wrapper_fake,
)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。

### `vit_torch_sdpa_wrapper` function / `vit_torch_sdpa_wrapper` 函数
```python
def vit_torch_sdpa_wrapper(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    scale: float | None = None,
    cu_seqlens: torch.Tensor | None = None,
    enable_gqa: bool = False,
) -> torch.Tensor:
    return torch.ops.vllm.torch_sdpa_wrapper(
        q, k, v, scale, cu_seqlens, enable_gqa=enable_gqa
    )
```
**EN:** This function implements `vit_torch_sdpa_wrapper` within the module. Key calls include `torch_sdpa_wrapper`.
**CN:** 该函数会实现 `vit_torch_sdpa_wrapper`，其作用域位于the module。 关键调用包括 `torch_sdpa_wrapper`。

### `flashinfer_wrapper` function / `flashinfer_wrapper` 函数
```python
def flashinfer_wrapper(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    scale: float,
    workspace_buffer: torch.Tensor,
    cu_seqlens: torch.Tensor | None = None,
    max_seqlen: torch.Tensor | None = None,
    sequence_lengths: torch.Tensor | None = None,
    q_scale: torch.Tensor | None = None,
    k_scale: torch.Tensor | None = None,
    v_scale: torch.Tensor | None = None,
    o_data_type: torch.dtype | None = None,
) -> torch.Tensor:
    from flashinfer.prefill import cudnn_batch_prefill_with_kv_cache

    is_reshaped = q.dim() == 4

    if is_reshaped:
        reshape_batch_size = q.shape[0]
        q, k, v = (einops.rearrange(x, "b s ... -> (b s) ...") for x in [q, k, v])
    # cuDNN <= 9.10.2.21 requires q, k to be contiguous
    # this comes with no cost for ViTs with RoPE because
    # RoPE has already made q and k contiguous.
    q, k = q.contiguous(), k.contiguous()

    assert cu_seqlens is not None
    assert max_seqlen is not None
    assert sequence_lengths is not None
    assert len(cu_seqlens) % 2 == 0, "cu_seqlens must be divisible by 2"
    cu_seqlength = len(cu_seqlens) // 2
    batch_offsets_qko = cu_seqlens[:cu_seqlength].view(-1, 1, 1, 1)
    batch_offsets_v = cu_seqlens[cu_seqlength:].view(-1, 1, 1, 1)
    sequence_lengths = sequence_lengths.view(-1, 1, 1, 1)
    max_seqlen = max_seqlen.item()

    output, _ = cudnn_batch_prefill_with_kv_cache(
        q,
        k,
        v,
        scale,
        workspace_buffer,
        max_token_per_sequence=max_seqlen,
        max_sequence_kv=max_seqlen,
        actual_seq_lens_q=sequence_lengths,
        actual_seq_lens_kv=sequence_lengths,
        causal=False,
        return_lse=False,
        batch_offsets_q=batch_offsets_qko,
        batch_offsets_k=batch_offsets_qko,
        batch_offsets_v=batch_offsets_v,
        batch_offsets_o=batch_offsets_qko,
        q_scale=q_scale,
        k_scale=k_scale,
        v_scale=v_scale,
        o_data_type=o_data_type,
    )

    if is_reshaped:
        output = einops.rearrange(output, "(b s) h d -> b s h d", b=reshape_batch_size)

    return output
```
**EN:** This function implements `flashinfer_wrapper` within the module. Key calls include `view`, `item`, `cudnn_batch_prefill_with_kv_cache`, `dim`, `contiguous`, `len`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `flashinfer_wrapper`，其作用域位于the module。 关键调用包括 `view`, `item`, `cudnn_batch_prefill_with_kv_cache`, `dim`, `contiguous`, `len`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `vit_flashinfer_wrapper_fake` function / `vit_flashinfer_wrapper_fake` 函数
```python
def vit_flashinfer_wrapper_fake(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    scale: float,
    workspace_buffer: torch.Tensor,
    cu_seqlens: torch.Tensor | None = None,
    max_seqlen: torch.Tensor | None = None,
    sequence_lengths: torch.Tensor | None = None,
    q_scale: torch.Tensor | None = None,
    k_scale: torch.Tensor | None = None,
    v_scale: torch.Tensor | None = None,
    o_data_type: torch.dtype | None = None,
) -> torch.Tensor:
    return torch.empty_like(q, dtype=o_data_type or q.dtype)
```
**EN:** This function implements `vit_flashinfer_wrapper_fake` within the module. Key calls include `empty_like`.
**CN:** 该函数会实现 `vit_flashinfer_wrapper_fake`，其作用域位于the module。 关键调用包括 `empty_like`。

### Module setup / 模块初始化
```python
direct_register_custom_op(
    op_name="flashinfer_wrapper",
    op_func=flashinfer_wrapper,
    fake_impl=vit_flashinfer_wrapper_fake,
)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。

### `vit_flashinfer_wrapper` function / `vit_flashinfer_wrapper` 函数
```python
def vit_flashinfer_wrapper(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    scale: float,
    workspace_buffer: torch.Tensor,
    cu_seqlens: torch.Tensor | None = None,
    max_seqlen: torch.Tensor | None = None,
    sequence_lengths: torch.Tensor | None = None,
    q_scale: torch.Tensor | None = None,
    k_scale: torch.Tensor | None = None,
    v_scale: torch.Tensor | None = None,
    o_data_type: torch.dtype | None = None,
) -> torch.Tensor:
    return torch.ops.vllm.flashinfer_wrapper(
        q,
        k,
        v,
        scale,
        workspace_buffer,
        cu_seqlens,
        max_seqlen,
        sequence_lengths,
        q_scale,
        k_scale,
        v_scale,
        o_data_type,
    )
```
**EN:** This function implements `vit_flashinfer_wrapper` within the module. Key calls include `flashinfer_wrapper`.
**CN:** 该函数会实现 `vit_flashinfer_wrapper`，其作用域位于the module。 关键调用包括 `flashinfer_wrapper`。

## Key Concepts / 关键概念
- `flash_attn_maxseqlen_wrapper`: top-level helper or orchestration entry point. / `flash_attn_maxseqlen_wrapper`：顶层辅助函数或编排入口。
- `flash_attn_maxseqlen_wrapper_fake`: top-level helper or orchestration entry point. / `flash_attn_maxseqlen_wrapper_fake`：顶层辅助函数或编排入口。
- `vit_flash_attn_wrapper`: top-level helper or orchestration entry point. / `vit_flash_attn_wrapper`：顶层辅助函数或编排入口。
- `triton_attn_wrapper`: top-level helper or orchestration entry point. / `triton_attn_wrapper`：顶层辅助函数或编排入口。
- `triton_attn_wrapper_fake`: top-level helper or orchestration entry point. / `triton_attn_wrapper_fake`：顶层辅助函数或编排入口。
- `vit_triton_attn_wrapper`: top-level helper or orchestration entry point. / `vit_triton_attn_wrapper`：顶层辅助函数或编排入口。
- `apply_sdpa`: top-level helper or orchestration entry point. / `apply_sdpa`：顶层辅助函数或编排入口。
- `torch_sdpa_wrapper`: top-level helper or orchestration entry point. / `torch_sdpa_wrapper`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- External / 外部依赖: `einops`, `torch`, `aiter`, `flashinfer`
- Internal vLLM / 内部依赖: `vllm.platforms`, `vllm.utils.torch_utils`, `vllm.v1.attention.backends.fa_utils`, `vllm.v1.attention.ops.triton_prefill_attention`
