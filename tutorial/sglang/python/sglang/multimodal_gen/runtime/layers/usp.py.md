# usp.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/usp.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `_maybe_wait`, `_usp_all_to_all_single`, and `_usp_input_all_to_all`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `_maybe_wait`、`_usp_all_to_all_single` 和 `_usp_input_all_to_all` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 3-14: module setup and imports / 模块初始化与导入
```python
import logging
from typing import TYPE_CHECKING

import torch
import torch.distributed._functional_collectives as ft_c
from torch.distributed.tensor.experimental._attention import _cp_options

from sglang.multimodal_gen.runtime.distributed.parallel_state import (
    get_sp_group,
    get_ulysses_parallel_world_size,
)
from sglang.srt.utils.common import torch_release
```
**EN:** This block establishes the module context and imports `logging`, `typing`, `torch`, `torch.distributed._functional_collectives`, `torch.distributed.tensor.experimental._attention`, and `sglang.multimodal_gen.runtime.distributed.parallel_state`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `logging`、`typing`、`torch`、`torch.distributed._functional_collectives`、`torch.distributed.tensor.experimental._attention` 和 `sglang.multimodal_gen.runtime.distributed.parallel_state`。这些依赖为后续实现提供所需符号。

### Lines 16-23: supporting statements / 辅助语句
```python
_cp_options.enable_load_balance = False

if TYPE_CHECKING:
    from sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend import (
        AttentionImpl,
    )

logger = logging.getLogger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `logging.getLogger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `logging.getLogger` 协同工作。

### Lines 26-33: `_maybe_wait` implementation / `_maybe_wait` 实现
```python
def _maybe_wait(tensor: torch.Tensor) -> torch.Tensor:
    """
    When tracing the code, the result tensor is not an AsyncCollectiveTensor,
    so we cannot call ``wait()``.
    """
    if isinstance(tensor, ft_c.AsyncCollectiveTensor):
        return tensor.wait()
    return tensor
```
**EN:** This block defines function `_maybe_wait`. When tracing the code, the result tensor is not an AsyncCollectiveTensor, so we cannot call ``wait()``. Key calls include `isinstance`, and `tensor.wait`. The implementation branches on conditions. Parameters such as `tensor` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_maybe_wait`。 它用于处理 maybe wait 相关逻辑。 关键调用包括 `isinstance` 和 `tensor.wait`。 实现中包含条件分支。 本段逻辑主要由 `tensor` 等参数驱动。

### Lines 36-45: `_usp_all_to_all_single` implementation / `_usp_all_to_all_single` 实现
```python
def _usp_all_to_all_single(x: torch.Tensor) -> torch.Tensor:
    ulysses_pg = get_sp_group().ulysses_group
    assert ulysses_pg is not None, "Ulysses process group is not initialized."
    x_shape = x.shape
    x = x.flatten().contiguous()
    output = torch.empty_like(x)
    # USP calls this collective many times per denoising step and waits
    # immediately, so avoid the extra wrapper overhead of functional collectives.
    torch.distributed.all_to_all_single(output, x, group=ulysses_pg)
    return output.reshape(x_shape)
```
**EN:** This block defines function `_usp_all_to_all_single`. It handles usp all to all single logic. Key calls include `x.flatten.contiguous`, `torch.empty_like`, `torch.distributed.all_to_all_single`, `output.reshape`, and `get_sp_group`. Parameters such as `x` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_usp_all_to_all_single`。 它用于处理 usp all to all single 相关逻辑。 关键调用包括 `x.flatten.contiguous`、`torch.empty_like`、`torch.distributed.all_to_all_single`、`output.reshape` 和 `get_sp_group`。 本段逻辑主要由 `x` 等参数驱动。

### Lines 48-101: `_usp_input_all_to_all` implementation / `_usp_input_all_to_all` 实现
```python
def _usp_input_all_to_all(x: torch.Tensor, head_dim: int = 1) -> torch.Tensor:
    """
    Perform Ulysses-style input all-to-all over the head dimension.

    Default layout expects heads at dim=1 and sequence at dim=2:
        [b, h, s_local, d] -> [b, h_local, s_global, d]

    If heads are at dim=2 (input is [b, s_local, h, d]), set head_dim=2, and the
    function returns [b, s_global, h_local, d], preserving the original
    head/sequence dim ordering.

    Args:
        x: A 4D tensor with layout [b, *, *, d] where '*' are sequence and heads
        head_dim: Which dimension index corresponds to heads (1 or 2)

    Returns:
        Tensor with the same dim order as input, with heads sharded and sequence gathered.
    """
    world_size = get_ulysses_parallel_world_size()
    if world_size <= 1:
        return x

    assert x.ndim == 4, f"x must have 4 dimensions, got {x.ndim}"
    assert head_dim in (1, 2), f"head_dim must be 1 or 2, got {head_dim}"

    # Move the dimension to be split (h_global) to dim 0 for all_to_all_single
    if head_dim == 1:
        b, h_global, s_local, d = x.shape
        # Shape transition: [b, h_global, s_local, d] -> [h_global, b, s_local, d]
        permute_order = (1, 0, 2, 3)
    else:  # head_dim == 2
        b, s_local, h_global, d = x.shape
        # Shape transition: [b, s_local, h_global, d] -> [h_global, b, s_local, d]
        permute_order = (2, 0, 1, 3)

    assert (
        h_global % world_size == 0
    ), f"h_global ({h_global}) must be divisible by world_size ({world_size})"

    h_local, s_global = h_global // world_size, s_local * world_size

    x = x.permute(permute_order).contiguous()
    x = _usp_all_to_all_single(x)
    x = x.reshape(world_size, h_local, b, s_local, d)

    # Reorder dims to place 'world_size' adjacent to 's_local' to merge them into 's_global'
    if head_dim == 1:
        # Shape transition: [world_size, h_local, b, s_local, d] -> [b, h_local, world_size, s_local, d]
        x = x.permute(2, 1, 0, 3, 4).contiguous().reshape(b, h_local, s_global, d)
    else:  # head_dim == 2
        # Shape transition: [world_size, h_local, b, s_local, d] -> [b, world_size, s_local, h_local, d]
        x = x.permute(2, 0, 3, 1, 4).contiguous().reshape(b, s_global, h_local, d)

    return x
```
**EN:** This block defines function `_usp_input_all_to_all`. Perform Ulysses-style input all-to-all over the head dimension. Default layout expects heads at dim=1 and sequence at dim=2: [b, h, s_local, d] -> [b, h_local, s_global, d] If heads are at dim=2 (input is [b, s_local, h, d]), set head_dim=2, and the function returns [b, s_global, h_local, d], preserving the original head/sequence dim ordering. Key calls include `get_ulysses_parallel_world_size`, `x.permute.contiguous`, `_usp_all_to_all_single`, `x.reshape`, and `x.permute.contiguous.reshape`. The implementation branches on conditions. Parameters such as `x`, and `head_dim` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_usp_input_all_to_all`。 它用于处理 usp input all to all 相关逻辑。 关键调用包括 `get_ulysses_parallel_world_size`、`x.permute.contiguous`、`_usp_all_to_all_single`、`x.reshape` 和 `x.permute.contiguous.reshape`。 实现中包含条件分支。 本段逻辑主要由 `x` 和 `head_dim` 等参数驱动。

### Lines 104-157: `_usp_output_all_to_all` implementation / `_usp_output_all_to_all` 实现
```python
def _usp_output_all_to_all(x: torch.Tensor, head_dim: int = 1) -> torch.Tensor:
    """
    Perform Ulysses-style output all-to-all over the head dimension (inverse of input).

    Default layout expects heads at dim=1 and sequence at dim=2:
        [b, h_local, s, d] -> [b, h, s_local, d]

    If heads are at dim=2 (input is [b, s_global, h // world_size, d]), set head_dim=2,
    and the function returns [b, s_local, h, d], preserving the original head/sequence
    dim ordering.

    Args:
        x: A 4D tensor with layout [b, *, *, d] where '*' are sequence and heads
        head_dim: Which dimension index corresponds to heads (1 or 2)

    Returns:
        Tensor with the same dim order as input, with heads gathered and sequence sharded.
    """
    world_size = get_ulysses_parallel_world_size()
    if world_size <= 1:
        return x

    assert x.ndim == 4, f"x must have 4 dimensions, got {x.ndim}"
    assert head_dim in (1, 2), f"head_dim must be 1 or 2, got {head_dim}"

    # Move the dimension to be split (s_global) to dim 0 for all_to_all_single
    if head_dim == 1:
        b, h_local, s_global, d = x.shape
        # Shape transition: [b, h_local, s_global, d] -> [s_global, b, h_local, d]
        permute_order = (2, 0, 1, 3)
    else:  # head_dim == 2
        b, s_global, h_local, d = x.shape
        # Shape transition: [b, s_global, h_local, d] -> [s_global, b, h_local, d]
        permute_order = (1, 0, 2, 3)

    assert (
        s_global % world_size == 0
    ), f"s_global ({s_global}) must be divisible by world_size ({world_size})"

    s_local, h_global = s_global // world_size, h_local * world_size

    x = x.permute(permute_order).contiguous()
    x = _usp_all_to_all_single(x)
    x = x.reshape(world_size, s_local, b, h_local, d)

    # Reorder dims to place 'world_size' adjacent to 'h_local' to merge them into 'h_global'
    if head_dim == 1:
        # Shape transition: [world_size, s_local, b, h_local, d] -> [b, world_size, h_local, s_local, d]
        x = x.permute(2, 0, 3, 1, 4).contiguous().reshape(b, h_global, s_local, d)
    else:  # head_dim == 2
        # Shape transition: [world_size, s_local, b, h_local, d] -> [b, s_local, world_size, h_local, d]
        x = x.permute(2, 1, 0, 3, 4).contiguous().reshape(b, s_local, h_global, d)

    return x
```
**EN:** This block defines function `_usp_output_all_to_all`. Perform Ulysses-style output all-to-all over the head dimension (inverse of input). Default layout expects heads at dim=1 and sequence at dim=2: [b, h_local, s, d] -> [b, h, s_local, d] If heads are at dim=2 (input is [b, s_global, h // world_size, d]), set head_dim=2, and the function returns [b, s_local, h, d], preserving the original head/sequence dim ordering. Key calls include `get_ulysses_parallel_world_size`, `x.permute.contiguous`, `_usp_all_to_all_single`, `x.reshape`, and `x.permute.contiguous.reshape`. The implementation branches on conditions. Parameters such as `x`, and `head_dim` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_usp_output_all_to_all`。 它用于处理 usp output all to all 相关逻辑。 关键调用包括 `get_ulysses_parallel_world_size`、`x.permute.contiguous`、`_usp_all_to_all_single`、`x.reshape` 和 `x.permute.contiguous.reshape`。 实现中包含条件分支。 本段逻辑主要由 `x` 和 `head_dim` 等参数驱动。

### Lines 160-251: `ring_attn` implementation / `ring_attn` 实现
```python
def ring_attn(
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    attn_impl: "AttentionImpl",
    is_causal: bool = False,
    dropout_p: float = 0.0,
):
    """
    Ring Attention implementation.

    This function implements Ring Attention, a strategy for distributed attention
    computation that reduces peak memory usage. It accepts a generic attention
    implementation (`attn_impl`) which is called by the underlying PyTorch
    distributed attention primitive.

    Args:
        query, key, value: The input tensors for attention.
        attn_impl: An instance of an attention implementation backend
                   (e.g., FlashAttentionImpl) whose `forward` method will be
                   used as the computational kernel.
        is_causal: Whether to apply causal masking.
        dropout_p: Dropout probability.
    """
    # torch.distributed.tensor.experimental._attention is not a public API,
    from torch.distributed.tensor.experimental._attention import (
        _templated_ring_attention,
    )

    ring_pg = get_sp_group().ring_group
    assert ring_pg is not None, "Ring process group is not initialized."

    # Ring attention primitives expect tensors in [B, H, S, D] layout.
    # We permute the inputs here.
    query = torch.permute(query, [0, 2, 1, 3]).contiguous()
    key = torch.permute(key, [0, 2, 1, 3]).contiguous()
    value = torch.permute(value, [0, 2, 1, 3]).contiguous()

    # Create an adapter function that matches the signature expected by
    # _templated_ring_attention. The `attn_impl` already has dropout and
    # causal settings configured during its initialization.

    # Note: Please be aware that Attention Backend and Ring Attention may require different QKV tensor shapes.
    # For example, FlashAttention expects the format to be BSHD.
    def attn_callable_adapter(q, k, v, *args, **kwargs):
        # We ignore the dropout_p and is_causal passed by _templated_ring_attention
        # and rely on the pre-configured attn_impl.
        # The `attn_metadata` is not available here, so we pass None.
        # This is a limitation we must accept when using this experimental API.
        q = torch.permute(q, [0, 2, 1, 3])
        k = torch.permute(k, [0, 2, 1, 3])
        v = torch.permute(v, [0, 2, 1, 3])
        # logger.warning(f"Warning: return_softmax_lse is only supported for FlashAttentionImpl")
        output, softmax_lse, *rest = attn_impl.forward(
            q,
            k,
            v,
            attn_metadata=None,
            return_softmax_lse=True,
        )
        output = torch.permute(output, [0, 2, 1, 3])
        return output, softmax_lse, *rest

    # Starting from torch 2.6.0, _templated_ring_attention expects an integer
    # segment_id for the attention function.
    use_segment_id = torch_release >= (2, 6)

    attn_kwargs = dict(
        op=attn_callable_adapter,
        dropout_p=dropout_p,
        is_causal=is_causal,
        query=query,
        key=key,
        value=value,
        group=ring_pg,  # https://github.com/pytorch/pytorch/blob/c907c778f42ba2fdaf25b733dd25baf9779c6a12/torch/distributed/tensor/experimental/_context_parallel/_attention.py#L309
    )

    if use_segment_id:
        # For torch >= 2.6, segment_id is required. The value '1' is a placeholder
        # as we are not using complex segmentation features.
        out, *_ = _templated_ring_attention(
            seq_dim=1,  # segment_id
            **attn_kwargs,
        )
    else:
        out, *_ = _templated_ring_attention(
            **attn_kwargs,
        )

    # Permute the output back to [B, S, H, D] layout.
    output = torch.permute(out, [0, 2, 1, 3])
    return output
```
**EN:** This block defines function `ring_attn`. Ring Attention implementation. This function implements Ring Attention, a strategy for distributed attention computation that reduces peak memory usage. Key calls include `torch.permute.contiguous`, `dict`, `torch.permute`, `get_sp_group`, and `attn_impl.forward`. The implementation branches on conditions. Parameters such as `query`, `key`, `value`, `attn_impl`, and `is_causal` drive the behavior in this section.
**CN:** 该代码块定义了函数 `ring_attn`。 它用于处理 ring attn 相关逻辑。 关键调用包括 `torch.permute.contiguous`、`dict`、`torch.permute`、`get_sp_group` 和 `attn_impl.forward`。 实现中包含条件分支。 本段逻辑主要由 `query`、`key`、`value`、`attn_impl` 和 `is_causal` 等参数驱动。

## Key Concepts / 关键概念
- `_maybe_wait`: When tracing the code, the result tensor is not an AsyncCollectiveTensor, so we cannot call ``wait()``. / 顶层函数，用于处理 maybe wait 相关逻辑。
- `_usp_all_to_all_single`: Top-level function that handles usp all to all single logic. / 顶层函数，用于处理 usp all to all single 相关逻辑。
- `_usp_input_all_to_all`: Perform Ulysses-style input all-to-all over the head dimension. / 顶层函数，用于处理 usp input all to all 相关逻辑。
- `_usp_output_all_to_all`: Perform Ulysses-style output all-to-all over the head dimension (inverse of input). / 顶层函数，用于处理 usp output all to all 相关逻辑。
- `ring_attn`: Ring Attention implementation. / 顶层函数，用于处理 ring attn 相关逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `typing`
- **Third-party / 第三方依赖**: `torch`, `torch.distributed._functional_collectives`, `torch.distributed.tensor.experimental._attention`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.distributed.parallel_state`, `sglang.srt.utils.common`, `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`

- **Total lines / 总行数**: 251
