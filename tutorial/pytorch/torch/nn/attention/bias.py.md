# bias.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/nn/attention/bias.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Implements attention-specific abstractions and helpers for modern sequence models.
- **Purpose (CN)**: 实现现代序列模型中的注意力相关抽象与辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-26
```python
# mypy: allow-untyped-defs
"""Defines bias subclasses that work with scaled_dot_product_attention"""

from enum import auto, IntEnum
from warnings import warn

import torch
import torch.nn.functional as F
from torch.backends.cuda import (
    can_use_efficient_attention,
    can_use_flash_attention,
    is_flash_attention_available,
    SDPAParams,
)
from torch.nn.attention import _raise_kernel_warnings
from torch.nn.attention._utils import (
    _calculate_scale,
    _input_requires_grad,
    _postprocess_flash_output,
    _validate_sdpa_input,
)


__all__ = ["causal_upper_left", "causal_lower_right", "CausalVariant", "CausalBias"]
```
- **EN**: This assignment curates the public export surface that downstream imports are expected to use.
- **CN**: 这一赋值语句整理了公开导出的接口集合，供下游导入方使用。

### Lines 27-53
```python
torch._dynamo.allow_in_graph(is_flash_attention_available)
torch._dynamo.allow_in_graph(can_use_flash_attention)
torch._dynamo.allow_in_graph(can_use_efficient_attention)
torch._dynamo.allow_in_graph(SDPAParams)


class CausalVariant(IntEnum):
    r"""
    Enum for causal variants used in attention mechanisms.

    Defines two types of causal biases:

    ``UPPER_LEFT``: Represents upper-left triangular bias for standard causal attention.
    The equivalent pytorch code for constructing this bias is:

    .. code-block:: python

        torch.tril(torch.ones(size, dtype=torch.bool))

    For instance, with ``shape=(3,4)``, the materialized bias tensor will be:

    .. code-block:: text

        [[1, 0, 0, 0],
         [1, 1, 0, 0],
         [1, 1, 1, 0]]
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。

### Lines 54-78
```python

    ``LOWER_RIGHT``: Represents lower-right triangular bias, the include values are aligned to the lower
    right corner of the matrix.

    The equivalent pytorch code for constructing this bias is:

    .. code-block:: python

        diagonal_offset = size[1] - size[0]
        torch.tril(
            torch.ones(size, dtype=torch.bool),
            diagonal=diagonal_offset,
        )

    For instance, with ``shape=(3,4)``, the materialized bias tensor will be:

    .. code-block:: text

        [[1, 1, 0, 0],
         [1, 1, 1, 0],
         [1, 1, 1, 1]]

    Note that these variants are equivalent to each other when the sequence lengths of the query and key/value
    tensors are equal since the triangular matrix is square.
```
- **EN**: This block continues `CausalVariant` and works to track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 该代码块继续实现 `CausalVariant`，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 79-103
```python
    .. warning:: This enum is a prototype and subject to change.
    """

    UPPER_LEFT = auto()
    LOWER_RIGHT = auto()


class CausalBias(torch.Tensor):
    """
    A bias representing causal attention patterns. For an overview of the bias structure, see the :class:`CausalVariant` enum.

    This class is used for defining causal (triangular) attention biases. For construing the bias, there exist
    two factory functions: :func:`causal_upper_left` and :func:`causal_lower_right`.

    Example:

    .. code-block:: python

        from torch.nn.attention.bias import causal_lower_right

        bsz, num_heads, seqlen_q, seqlen_kv, head_dim = 32, 8, 4, 12, 8

        # Create a lower-right causal bias
        attn_bias = causal_lower_right(seqlen_q, seqlen_kv)
```
- **EN**: This module-level block helps hand work to a compiler/backend pipeline.
- **CN**: 这个模块级代码块用于将工作移交给编译器或后端流水线。

### Lines 104-127
```python
        q = torch.randn(
            bsz, num_heads, seqlen_q, head_dim, device="cuda", dtype=torch.float16
        )
        k = torch.randn(
            bsz, num_heads, seqlen_kv, head_dim, device="cuda", dtype=torch.float16
        )
        v = torch.randn(
            bsz, num_heads, seqlen_kv, head_dim, device="cuda", dtype=torch.float16
        )

        out = F.scaled_dot_product_attention(q, k, v, attn_bias)

    .. warning:: This class is a prototype and subject to change.
    """

    def __init__(self, variant: CausalVariant, seq_len_q: int, seq_len_kv: int) -> None:
        """
        Initializes the CausalBias instance with a specified variant and sequence lengths.

        Args:
            variant (CausalVariant): The type of causal bias to use (either UPPER_LEFT or LOWER_RIGHT).
            seq_len_q (int): The sequence length of the query tensor.
            seq_len_kv (int): The sequence length of the key/value tensor.
```
- **EN**: Declares `CausalBias(torch.Tensor)`; this class packages state and methods that hand work to a compiler/backend pipeline.
- **CN**: 声明 `CausalBias(torch.Tensor)`；该类封装了状态与方法，用于将工作移交给编译器或后端流水线。

### Lines 128-149
```python
        Raises a warning if the LOWER_RIGHT variant is used with seq_len_q > seq_len_kv, as it may produce NaNs.
        """
        if not isinstance(variant, CausalVariant):
            raise AssertionError(
                f"variant must be a CausalVariant, got {type(variant).__name__}"
            )
        super().__init__()
        self.variant = variant
        self.seq_len_q = seq_len_q
        self.seq_len_kv = seq_len_kv
        if seq_len_q > seq_len_kv and variant == CausalVariant.LOWER_RIGHT:
            warn(
                "Lower right causal bias will produce NaNs in the output when seq_len_q > seq_len_kv!",
                stacklevel=2,
            )

    def _upper_left(self, device: torch.device) -> torch.Tensor:
        """Upper left causal bias"""
        return torch.tril(
            torch.ones(self.seq_len_q, self.seq_len_kv, device=device, dtype=torch.bool)
        )
```
- **EN**: Declares `CausalBias(torch.Tensor)`; this class packages state and methods that hand work to a compiler/backend pipeline.
- **CN**: 声明 `CausalBias(torch.Tensor)`；该类封装了状态与方法，用于将工作移交给编译器或后端流水线。

### Lines 150-170
```python
    def _lower_right(self, device: torch.device) -> torch.Tensor:
        """Lower right causal bias"""
        diagonal_offset = self.seq_len_kv - self.seq_len_q
        return torch.tril(
            torch.ones(
                self.seq_len_q, self.seq_len_kv, device=device, dtype=torch.bool
            ),
            diagonal=diagonal_offset,
        )

    # pyrefly: ignore [bad-return]
    def _materialize(self, device: torch.device | None = None) -> torch.Tensor:
        """
        Materializes the causal bias into a tensor form.

        Depending on the variant, this method generates either an upper-left or lower-right
        triangular matrix to represent the causal bias.

        Args:
            device (Optional[torch.device]): The device on which to create the tensor. Defaults to CPU.
```
- **EN**: Declares `CausalBias(torch.Tensor)`; this class packages state and methods that hand work to a compiler/backend pipeline.
- **CN**: 声明 `CausalBias(torch.Tensor)`；该类封装了状态与方法，用于将工作移交给编译器或后端流水线。

### Lines 171-194
```python
        Returns:
            torch.Tensor: The materialized bias tensor.
        """
        if device is None:
            device = torch.device("cpu")
        if self.variant == CausalVariant.UPPER_LEFT:
            return self._upper_left(device)
        elif self.variant == CausalVariant.LOWER_RIGHT:
            return self._lower_right(device)

    @staticmethod
    def _dispatch(
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        attn_mask: "CausalBias",
        dropout_p: float = 0.0,
        is_causal: bool = False,
        scale: float | None = None,
        enable_gqa: bool = False,
    ) -> torch.Tensor:
        r"""
        Handles the logic for computing attention with the specified causal bias.
```
- **EN**: These decorators register or transform the following definition so it can hand work to a compiler/backend pipeline.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将工作移交给编译器或后端流水线。

### Lines 195-218
```python
        Args:
            query (Tensor): Query tensor; shape :math:`(N, ..., L, E)`.
            key (Tensor): Key tensor; shape :math:`(N, ..., S, E)`.
            value (Tensor): Value tensor; shape :math:`(N, ..., S, Ev)`.
            attn_mask (CausalBias): The type of causal attention to apply.
                A boolean mask where a value of True indicates that the element *should* take part in attention.
                A float mask of the same type as query, key, value that is added to the attention score.
            dropout_p (float): Dropout probability; if greater than 0.0, dropout is applied
            is_causal (bool): If true, assumes upper left causal attention masking and errors if both attn_mask and is_causal
                are set.
            scale (optional float): Scaling factor applied prior to softmax. If None, the default value is set
                to :math:`\frac{1}{\sqrt{E}}`.
            enable_gqa (optional bool): If set to True, Grouped Query Attention (GQA) is enabled, by default it is set to False.

        Returns:
            output (Tensor): Attention output; shape :math:`(N, ..., L, Ev)`.

        Raises:
            ValueError: If the causal bias variant is not a CausalVariant type.

        """
        if is_causal:
            raise ValueError("CausalBias should not be used with causal=True")
```
- **EN**: This block continues `CausalBias._dispatch` and works to track symbolic shape constraints and shape-dependent reasoning. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `CausalBias._dispatch`，用于跟踪符号形状约束以及依赖形状的推理。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 219-236
```python
        if (
            attn_mask.seq_len_q == attn_mask.seq_len_kv
            or attn_mask.variant == CausalVariant.UPPER_LEFT
        ):
            return F.scaled_dot_product_attention(
                query,
                key,
                value,
                attn_mask=None,
                dropout_p=dropout_p,
                is_causal=True,
                scale=scale,
                enable_gqa=enable_gqa,
            )
        elif attn_mask.variant == CausalVariant.LOWER_RIGHT:
            _validate_sdpa_input(query, key, value, None, dropout_p, is_causal, scale)
            sdpa_params = SDPAParams(
                query, key, value, None, dropout_p, is_causal, enable_gqa
```
- **EN**: This block continues `CausalBias._dispatch` and works to hand work to a compiler/backend pipeline. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `CausalBias._dispatch`，用于将工作移交给编译器或后端流水线。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 237-254
```python
            )
            if can_use_flash_attention(sdpa_params):
                alignment = 64 if query.device.type == "xpu" else 8
                og_head_size = query.size(-1)
                og_scale = _calculate_scale(og_head_size, scale)
                needs_padding = og_head_size % alignment != 0
                if needs_padding:
                    pad_len = alignment - (og_head_size % alignment)
                    query = torch.nn.functional.pad(query, (0, pad_len))
                    key = torch.nn.functional.pad(key, (0, pad_len))
                    value = torch.nn.functional.pad(value, (0, pad_len))
                out = torch.ops.aten._scaled_dot_product_flash_attention(
                    query,
                    key,
                    value,
                    dropout_p,
                    is_causal=True,  # TODO: Flash accepts causal = True and for this particular op it means lower right
                    return_debug_mask=False,
```
- **EN**: This block continues `CausalBias._dispatch` and works to hand work to a compiler/backend pipeline. It also branches on runtime conditions.
- **CN**: 该代码块继续实现 `CausalBias._dispatch`，用于将工作移交给编译器或后端流水线。 同时它还会根据运行时条件分支处理。

### Lines 255-272
```python
                    scale=og_scale,
                )[0]
                return _postprocess_flash_output(out, og_head_size)
            if can_use_efficient_attention(sdpa_params):
                compute_log_sumexp = False
                if _input_requires_grad(query, key, value):
                    compute_log_sumexp = True
                return torch.ops.aten._efficient_attention_forward(
                    query.transpose(1, 2),
                    key.transpose(1, 2),
                    value.transpose(1, 2),
                    bias=None,
                    cu_seqlens_q=None,
                    cu_seqlens_k=None,
                    max_seqlen_q=None,
                    max_seqlen_k=None,
                    dropout_p=dropout_p,
                    custom_mask_type=int(attn_mask.variant),
```
- **EN**: This block continues `CausalBias._dispatch` and works to organize reusable module behavior and state. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `CausalBias._dispatch`，用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 273-294
```python
                    compute_log_sumexp=compute_log_sumexp,
                    scale=scale,
                    seqlen_k=None,
                )[0].transpose(1, 2)
            else:
                _raise_kernel_warnings(sdpa_params)
                # We can't use efficient attention the only support for lower right is via materialization
                return F.scaled_dot_product_attention(
                    query,
                    key,
                    value,
                    attn_mask=attn_mask._materialize(query.device),
                    dropout_p=dropout_p,
                    is_causal=False,
                    scale=scale,
                    enable_gqa=enable_gqa,
                )
        else:
            raise ValueError(
                f"CausalBias.variant must be a CausalVariant type, but found: {attn_mask.variant}"
            )
```
- **EN**: This block continues `CausalBias` and works to hand work to a compiler/backend pipeline. It also computes and returns an intermediate/result value; validates assumptions before proceeding.
- **CN**: 该代码块继续实现 `CausalBias`，用于将工作移交给编译器或后端流水线。 同时它还会计算并返回中间值或结果；在继续前校验关键假设。

### Lines 295-321
```python
    @classmethod
    def __torch_function__(cls, func, types, args=(), kwargs=None):
        """Defines the behavior of torch.nn.functional.scaled_dot_product_attention when the attn_bias is an AttnBias"""
        if kwargs is None:
            kwargs = {}
        if func is torch.nn.functional.scaled_dot_product_attention:
            return cls._dispatch(*args, **kwargs)
        return super().__torch_function__(func, types, args, kwargs)

    def __repr__(self) -> str:  # type:ignore[override]
        return self._materialize().__repr__()


def causal_upper_left(*size) -> CausalBias:
    """
    Creates an upper-left triangular causal bias.

    This function generates a upper-left triangular matrix to represent causal attention bias with a
    diagonal offset set so that the inclusive values are aligned to the upper left corner of the matrix.
    This equivalent to the `is_causal=True` argument in `scaled_dot_product_attention`.

    The equivalent pytorch code for constructing this bias is:

    .. code-block:: python

        torch.tril(torch.ones(size, dtype=torch.bool))
```
- **EN**: These decorators register or transform the following definition so it can implement attention-specific transformations and bookkeeping.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够实现注意力相关的变换与簿记逻辑。

### Lines 322-348
```python
    For instance, with `shape=(3,4)`, the materialized bias tensor will be:

    .. code-block:: text

        [[1, 0, 0, 0],
         [1, 1, 0, 0],
         [1, 1, 1, 0]]

    Args:
        size: The size of the bias matrix.

    Returns:
        CausalBias: The UPPER_LEFT triangular causal bias variant.
    """
    if len(size) != 2:
        raise AssertionError("causal_upper_left only supports 2D tensors")
    seq_len_q, seq_len_kv = size
    return CausalBias(CausalVariant.UPPER_LEFT, seq_len_q, seq_len_kv)


def causal_lower_right(*size) -> CausalBias:
    """
    Creates a lower-right triangular causal bias.

    This function generates a lower-right triangular matrix to represent causal attention bias with a
    diagonal offset set so that the inclusive values are aligned to the lower right corner of the matrix.
```
- **EN**: This module-level block helps track symbolic shape constraints and shape-dependent reasoning. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于跟踪符号形状约束以及依赖形状的推理。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 349-369
```python
    The equivalent pytorch code for constructing this bias is:

    .. code-block:: python

        diagonal_offset = size[1] - size[0]
        torch.tril(
            torch.ones(size, dtype=torch.bool),
            diagonal=diagonal_offset,
        )

    For instance, with `shape=(3,4)`, the materialized bias tensor will be:

    .. code-block:: text

        [[1, 1, 0, 0],
         [1, 1, 1, 0],
         [1, 1, 1, 1]]

    Args:
        size: The size of the bias matrix.
```
- **EN**: This block continues `causal_lower_right` and works to track symbolic shape constraints and shape-dependent reasoning.
- **CN**: 该代码块继续实现 `causal_lower_right`，用于跟踪符号形状约束以及依赖形状的推理。

### Lines 370-376
```python
    Returns:
        CausalBias: The LOWER_RIGHT triangular causal bias variant.
    """
    if len(size) != 2:
        raise AssertionError("causal_lower_right only supports 2D tensors")
    seq_len_q, seq_len_kv = size
    return CausalBias(CausalVariant.LOWER_RIGHT, seq_len_q, seq_len_kv)
```
- **EN**: This block continues `causal_lower_right` and works to hand work to a compiler/backend pipeline. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 该代码块继续实现 `causal_lower_right`，用于将工作移交给编译器或后端流水线。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

## Key Concepts / 关键概念

- **Neural-network abstractions / 神经网络抽象**
  - EN: The file contributes to reusable neural-network layers, functionals, or support utilities.
  - CN: 该文件为可复用的神经网络层、函数式接口或辅助工具提供实现。
- **Symbolic shapes / 符号形状**
  - EN: The file reasons about tensor extents symbolically instead of requiring concrete integers everywhere.
  - CN: 该文件以符号方式推理张量形状，而不是处处要求具体整数。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Compiler handoff / 编译器交接**
  - EN: The implementation prepares captured work for backend compilation or execution.
  - CN: 该实现会把捕获到的工作整理后交给后端编译或执行。
- **Attention mechanics / 注意力机制**
  - EN: The logic specializes in attention-style data movement, masking, or projection patterns.
  - CN: 该逻辑专门处理注意力风格的数据搬运、掩码或投影模式。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.nn.functional`, `torch.backends.cuda`, `torch.nn.attention`, `torch.nn.attention._utils`
- **Standard library / 标准库**: `enum`, `warnings`
- **Primary symbols / 核心符号**: `__all__`, `CausalVariant`, `CausalBias`, `causal_upper_left`, `causal_lower_right`
- **Relationship summary / 关系总结**: Most dependencies here support neural-network operators, module composition, and tensor/state utilities. / 这里的大多数依赖用于支撑神经网络算子、模块组合以及张量/状态工具。
