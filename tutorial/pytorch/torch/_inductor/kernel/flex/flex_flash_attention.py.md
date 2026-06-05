# flex_flash_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/kernel/flex/flex_flash_attention.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It defines classes such as `FlexFlashConfig`, and `HierarchicalIndex`. It exposes functions such as `_get_flex_flash_fwd_configs`, `_get_flex_flash_bwd_configs`, `ensure_flash_available`, `_hierarchical_indexer_cute`, `patch_fixed_layout_indexer_for_cutedsl`, `wrap_choice_render_with_cutedsl_indexer`, and `...+11`. Module note: Call into flash-attention 4 for flexattention
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。其中定义了 `FlexFlashConfig`、`HierarchicalIndex` 等类。同时提供 `_get_flex_flash_fwd_configs`、`_get_flex_flash_bwd_configs`、`ensure_flash_available`、`_hierarchical_indexer_cute`、`patch_fixed_layout_indexer_for_cutedsl`、`wrap_choice_render_with_cutedsl_indexer`、`另有11项` 等函数。模块文档字符串给出了额外背景说明。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
"""Call into flash-attention 4 for flexattention"""

import dataclasses
import functools
import importlib
from collections.abc import Callable, Sequence
from contextlib import contextmanager
from typing import Any, cast, Literal

import sympy
from sympy import Expr, Integer

import torch
from torch.fx import GraphModule

from ...ir import FixedLayout, ShapeAsConstantBuffer, Subgraph, TensorBox
from ...lowering import empty_strided
from ...select_algorithm import autotune_select_algorithm
from ...virtualized import V
````
- **EN**: Imports dependencies such as `dataclasses`, `functools`, `importlib`, `collections.abc`, `contextlib`, `typing`, and `...+7` for the logic in this range. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里导入了 `dataclasses`、`functools`、`importlib`、`collections.abc`、`contextlib`、`typing`、`另有7项` 等依赖，为后续逻辑提供基础能力。借助符号表达式来推理索引、形状或代数性质。

### Lines 21-40 / 第 21-40 行
````python
from .common import (
    create_indices_fake,
    create_num_blocks_fake_generator,
    infer_dense_strides,
    load_flex_template,
    SubgraphResults,
)


@dataclasses.dataclass
class FlexFlashConfig:
    """Autotuning configuration for CuteDSL flex flash attention kernels.

    score_mod_vec_size: Number of elements processed per thread in the score_mod
        application loop. Maps to score_mod.__vec_size__ in CuTe flash attention.
        None uses the kernel default. Only effective for forward; backward does
        not currently support vectorized score_mod.
    """

    score_mod_vec_size: int | None = None
````
- **EN**: Imports dependencies such as `.common` for the logic in this range. Introduces class `FlexFlashConfig`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields.
- **CN**: 这里导入了 `.common` 等依赖，为后续逻辑提供基础能力。这里定义了类`FlexFlashConfig`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。

### Lines 41-60 / 第 41-60 行
````python


def _get_flex_flash_fwd_configs(
    has_score_mod: bool,
    has_aux_tensors: bool,
) -> list[FlexFlashConfig]:
    if not has_score_mod or not torch._inductor.config.max_autotune:
        return [FlexFlashConfig()]
    if has_aux_tensors:
        return [FlexFlashConfig(score_mod_vec_size=1)]
    return [
        FlexFlashConfig(score_mod_vec_size=v) for v in (1, 2, 4, 8, 16, 32, 64, 128)
    ]


def _get_flex_flash_bwd_configs() -> list[FlexFlashConfig]:
    return [FlexFlashConfig()]


aten = torch.ops.aten
````
- **EN**: Introduces function `_get_flex_flash_fwd_configs`, function `_get_flex_flash_bwd_configs`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `has_score_mod`, `has_aux_tensors`, and `aten`.
- **CN**: 这里定义了函数`_get_flex_flash_fwd_configs`、函数`_get_flex_flash_bwd_configs`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `has_score_mod`、`has_aux_tensors`、`aten` 等值。

### Lines 61-80 / 第 61-80 行
````python
prims = torch.ops.prims


@functools.lru_cache(maxsize=1)
def ensure_flash_available() -> bool:
    """Check if flash-attn is importable; cache the result for reuse.

    Call ensure_flash_available.cache_clear() after installing flash-attn
    in the same interpreter to retry the import.
    """
    try:
        return importlib.util.find_spec("flash_attn.cute") is not None  # type: ignore[attr-defined]
    except ImportError:
        return False


from ...codegen.cutedsl.cutedsl_template import CuteDSLTemplate


flash_attention_cutedsl_template = CuteDSLTemplate(
````
- **EN**: Imports dependencies such as `...codegen.cutedsl.cutedsl_template` for the logic in this range. Introduces function `ensure_flash_available`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly.
- **CN**: 这里导入了 `...codegen.cutedsl.cutedsl_template` 等依赖，为后续逻辑提供基础能力。这里定义了函数`ensure_flash_available`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。

### Lines 81-100 / 第 81-100 行
````python
    name="flash_attention_cutedsl", source=load_flex_template("flash_attention")
)
flash_attention_backward_cutedsl_template = CuteDSLTemplate(
    name="flash_attention_backward_cutedsl",
    source=load_flex_template("flash_attention_backward"),
)


class HierarchicalIndex(sympy.Function):
    """
    Inert wrapper to carry an N-D index tuple through Inductor's SymPy-based IR.

    Inductor generally represents a tensor index as a single `sympy.Expr` (often a
    flattened linear offset in memory). CuteDSL, however, wants structured coordinates so it
    can emit `tensor[i, j, ...]` and handle strides internally. We therefore wrap
    the per-dimension indices in a `sympy.Function` node: this keeps the value a
    `sympy.Expr` for existing substitution/CSE machinery, while letting CuteDSL
    codegen pattern-match and unpack the coordinates via `index.args`.

    `eval()` returns None to keep the node inert (no simplification/flattening).
````
- **EN**: Introduces class `HierarchicalIndex`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Initializes or updates values such as `name`, `flash_attention_backward_cutedsl_template`, and `source`.
- **CN**: 这里定义了类`HierarchicalIndex`。借助符号表达式来推理索引、形状或代数性质。初始化或更新了 `name`、`flash_attention_backward_cutedsl_template`、`source` 等值。

### Lines 101-120 / 第 101-120 行
````python

    These nodes are intended to be short-lived wrappers and are only interpreted by
    CuteDSL codegen (see `ModificationWrapperCuteDSL.load` in
    `torch/_inductor/codegen/cutedsl/cutedsl_kernel.py`).
    """

    @classmethod
    def eval(cls, *args):
        return None


def _hierarchical_indexer_cute(
    size: Sequence[int],
    stride: Sequence[int] | None = None,
    offset: Expr = Integer(0),
) -> Callable[[Sequence[Expr]], Expr]:
    """Return an indexer that preserves multi-dimensional indices for CuteDSL."""

    def indexer(indices: Sequence[Expr]) -> Expr:
        assert offset == Integer(0), "Offset not supported for hierarchical indexing"
````
- **EN**: Introduces function `eval`, function `_hierarchical_indexer_cute`, function `indexer`. Applies decorators to register behavior or alter how the following definition is constructed. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`eval`、函数`_hierarchical_indexer_cute`、函数`indexer`。使用装饰器来注册行为，或改变后续定义的构造方式。借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 121-140 / 第 121-140 行
````python
        assert len(indices) == len(size), (
            f"Rank mismatch: got {len(indices)} indices for tensor of rank {len(size)}"
        )
        if not indices:
            return Integer(0)
        if len(indices) == 1:
            return indices[0]
        return HierarchicalIndex(*indices)

    return indexer


@contextmanager
def patch_fixed_layout_indexer_for_cutedsl():
    """
    Temporarily swap FixedLayout.make_indexer so CuteDSL sees hierarchical indexing.

    Note [CuteDSL indexer patch]:
    Flex flash attention only supports a limited set of IR ops (pointwise, reads, no stores),
    so temporarily changing the indexing behavior is safe for the kernels we emit today.
````
- **EN**: Introduces function `patch_fixed_layout_indexer_for_cutedsl`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`patch_fixed_layout_indexer_for_cutedsl`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 141-160 / 第 141-160 行
````python
    TODO(dynamic shapes): Reconfirm once flex flash attention supports dynamic shapes.
    """
    original_make_indexer = FixedLayout.make_indexer

    def cutedsl_make_indexer(self):
        return _hierarchical_indexer_cute(self.size, self.stride, self.offset)

    FixedLayout.make_indexer = cutedsl_make_indexer  # type: ignore[assignment]
    try:
        yield
    finally:
        FixedLayout.make_indexer = original_make_indexer  # type: ignore[assignment]


def wrap_choice_render_with_cutedsl_indexer(choice: Any) -> None:
    """
    Wrap a template choice's kernel render to apply CuteDSL indexer patching.

    See Note [CuteDSL indexer patch]:
    CuteDSL handles tensor strides internally, so template rendering must use
````
- **EN**: Introduces function `cutedsl_make_indexer`, function `wrap_choice_render_with_cutedsl_indexer`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `original_make_indexer`, `try`, and `finally`.
- **CN**: 这里定义了函数`cutedsl_make_indexer`、函数`wrap_choice_render_with_cutedsl_indexer`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `original_make_indexer`、`try`、`finally` 等值。

### Lines 161-180 / 第 161-180 行
````python
    hierarchical indexing.
    """
    original_make_kernel_render = choice.make_kernel_render

    def make_kernel_render_with_patch(*args, **kwargs):
        render_kernel, render = original_make_kernel_render(*args, **kwargs)

        def render_with_patch():
            with patch_fixed_layout_indexer_for_cutedsl():
                return render()

        return render_kernel, render_with_patch

    choice.make_kernel_render = make_kernel_render_with_patch


def input_buffers_require_grads(graph_module, num_score_mod_placeholders: int):
    """Check if any of the input buffers (beyond the score mod placeholders) require gradients."""
    inputs = []
    for node in graph_module.graph.nodes:
````
- **EN**: Introduces function `make_kernel_render_with_patch`, function `render_with_patch`, function `input_buffers_require_grads`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `original_make_kernel_render`, and `inputs`.
- **CN**: 这里定义了函数`make_kernel_render_with_patch`、函数`render_with_patch`、函数`input_buffers_require_grads`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `original_make_kernel_render`、`inputs` 等值。

### Lines 181-200 / 第 181-200 行
````python
        if node.op == "placeholder":
            inputs.append(node)
    if len(inputs) <= num_score_mod_placeholders:
        return False

    def requires_grad(n):
        tensor_meta = n.meta.get("tensor_meta")
        return tensor_meta.requires_grad if tensor_meta is not None else False

    return any(requires_grad(n) for n in inputs[num_score_mod_placeholders:])


def is_trivial_score_graph(graph_module: GraphModule) -> bool:
    """Backwards currently doesn't support score_mods, match against identity"""
    graph = graph_module.graph
    nodes = list(graph.nodes)
    placeholders = [n for n in nodes if n.op == "placeholder"]
    output = [n for n in nodes if n.op == "output"]
    assert len(output) == 1, "Got graph w/ multiple outputs"
    output_val = output[0].args[0]
````
- **EN**: Introduces function `requires_grad`, function `is_trivial_score_graph`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `tensor_meta`, `graph`, `nodes`, `placeholders`, `output`, and `output_val`.
- **CN**: 这里定义了函数`requires_grad`、函数`is_trivial_score_graph`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `tensor_meta`、`graph`、`nodes`、`placeholders`、`output`、`output_val` 等值。

### Lines 201-220 / 第 201-220 行
````python
    # The identity graph just sends the score straight through
    return output_val == placeholders[0]


def is_trivial_mask_graph(graph_module: GraphModule) -> bool:
    """Mask graph is trivial when it only gates via the default full op."""
    graph = graph_module.graph
    nodes = list(graph.nodes)
    placeholders = [n for n in nodes if n.op == "placeholder"]
    output = [n for n in nodes if n.op == "output"]
    assert len(output) == 1, "Got graph w/ multiple outputs"
    output_val = output[0].args[0]

    # mask mod graph is empty if we have 4 inputs and full_default output
    return len(placeholders) == 4 and output_val.target is torch.ops.aten.full.default


@functools.lru_cache(maxsize=1)
def _is_symbol_from_tensor_shape(symbol: sympy.Symbol, shape_env: Any) -> bool:
    """Check if a symbol originates from a tensor size/stride (TensorPropertySource)."""
````
- **EN**: Introduces function `is_trivial_mask_graph`, function `_is_symbol_from_tensor_shape`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Uses symbolic expressions to reason about indices, shapes, or algebraic properties.
- **CN**: 这里定义了函数`is_trivial_mask_graph`、函数`_is_symbol_from_tensor_shape`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。借助符号表达式来推理索引、形状或代数性质。

### Lines 221-240 / 第 221-240 行
````python
    from torch._dynamo.source import TensorPropertySource

    sources = shape_env.var_to_sources.get(symbol, [])
    return any(isinstance(s, TensorPropertySource) for s in sources)


def _has_unsupported_captured_scalars(
    score_mod_other_buffers: Sequence[Any],
    mask_mod_other_buffers: Sequence[Any],
) -> bool:
    """Check if any captured buffers are dynamic scalars that cannot be inlined.

    When compiling with dynamic=True, captured Python scalars in score_mod or
    mask_mod may become:
    - sympy symbols from LocalSource (captured ints) - NOT from tensor shapes
    - 0-dim CPU tensors (captured floats)

    Symbols from TensorPropertySource (tensor size/stride) are fine because they
    get resolved at runtime.

````
- **EN**: Imports dependencies such as `torch._dynamo.source` for the logic in this range. Introduces function `_has_unsupported_captured_scalars`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里导入了 `torch._dynamo.source` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_has_unsupported_captured_scalars`。借助符号表达式来推理索引、形状或代数性质。处理与设备相关的行为、能力信息或派发决策。

### Lines 241-260 / 第 241-260 行
````python
    The FLASH backend cannot inline captured scalar symbolic values into the CuteDSL template.
    """
    from torch._inductor.virtualized import V

    shape_env = V.graph.sizevars.shape_env

    for buf in list(score_mod_other_buffers) + list(mask_mod_other_buffers):
        # Captured int becomes sympy.Symbol - check if it's NOT from a tensor shape
        if isinstance(buf, sympy.Expr):
            for symbol in buf.free_symbols:
                if not _is_symbol_from_tensor_shape(symbol, shape_env):
                    return True
        # Captured float becomes 0-dim TensorBox on CPU
        if isinstance(buf, TensorBox):
            device = buf.get_device()
            size = buf.get_size()
            if device is not None and device.type == "cpu" and len(size) == 0:
                # 0-dimensional CPU tensor (scalar) - can't be inlined into CUDA kernel
                return True
    return False
````
- **EN**: Imports dependencies such as `torch._inductor.virtualized` for the logic in this range. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `torch._inductor.virtualized` 等依赖，为后续逻辑提供基础能力。借助符号表达式来推理索引、形状或代数性质。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 261-280 / 第 261-280 行
````python


def _can_use_flex_flash_attention(
    subgraph: Subgraph,
    mask_graph: Subgraph,
    num_score_mod_placeholders: int,
) -> tuple[bool, str]:
    """Check if flex flash attention can be used for the given inputs.

    Returns:
        tuple: (can_use, reason) where reason explains why it can't be used if can_use is False
    """
    if not ensure_flash_available():
        return False, "CUTE flash attention library is not available"

    if input_buffers_require_grads(subgraph.graph_module, num_score_mod_placeholders):
        return (
            False,
            "Input buffers require gradients (not supported by flash attention)",
        )
````
- **EN**: Introduces function `_can_use_flex_flash_attention`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `subgraph`, `mask_graph`, `num_score_mod_placeholders`, `Returns`, and `tuple`.
- **CN**: 这里定义了函数`_can_use_flex_flash_attention`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `subgraph`、`mask_graph`、`num_score_mod_placeholders`、`Returns`、`tuple` 等值。

### Lines 281-300 / 第 281-300 行
````python

    return True, ""


def _use_flex_flash_attention(
    subgraph: Subgraph,
    mask_graph: Subgraph,
    kernel_options: dict[str, Any],
    num_score_mod_placeholders: int,
    backend: Literal["AUTO", "TRITON", "FLASH", "TRITON_DECODE"],
) -> bool:
    """Determine if we should use flex flash attention for the given inputs.

    Args:
        subgraph: The score modification subgraph
        mask_graph: The mask modification subgraph
        kernel_options: Kernel configuration options
        num_score_mod_placeholders: Number of placeholders in score_mod
        backend: Implementation selector (AUTO, TRITON, FLASH, TRITON_DECODE)

````
- **EN**: Introduces function `_use_flex_flash_attention`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `subgraph`, `mask_graph`, `kernel_options`, `num_score_mod_placeholders`, `backend`, and `Args`.
- **CN**: 这里定义了函数`_use_flex_flash_attention`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `subgraph`、`mask_graph`、`kernel_options`、`num_score_mod_placeholders`、`backend`、`Args` 等值。

### Lines 301-320 / 第 301-320 行
````python
    Returns:
        True if flash attention should be used, False otherwise
    """
    # Flash is experimental and must be explicitly requested
    if backend != "FLASH":
        return False

    can_use, reason = _can_use_flex_flash_attention(
        subgraph,
        mask_graph,
        num_score_mod_placeholders,
    )

    if not can_use:
        raise RuntimeError(
            f"BACKEND='FLASH' but flash attention cannot be used: {reason}"
        )

    return True

````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `Returns`. This range continues the implementation of function `_use_flex_flash_attention`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `Returns` 等值。这一段延续了函数`_use_flex_flash_attention` 的具体实现。

### Lines 321-340 / 第 321-340 行
````python

def create_flex_flash_attention_kernel(
    query: TensorBox,
    key: TensorBox,
    value: TensorBox,
    block_mask: tuple[Any, ...],
    scale: float,
    kernel_options: dict[str, Any],
    subgraph_buffer: SubgraphResults,
    mask_graph_buffer: SubgraphResults,
    score_mod_other_buffers: list[TensorBox],
    mask_mod_other_buffers: list[TensorBox],
    kv_num_blocks: TensorBox | None,
    kv_indices: TensorBox | None,
    full_kv_num_blocks: TensorBox | None,
    full_kv_indices: TensorBox | None,
    sparse_q_block_size: int,
    sparse_kv_block_size: int,
    mask_graph: Subgraph,
    subgraph: Subgraph | None = None,
````
- **EN**: Introduces function `create_flex_flash_attention_kernel`. Initializes or updates values such as `query`, `key`, `value`, `block_mask`, `scale`, `kernel_options`, and `...+12`.
- **CN**: 这里定义了函数`create_flex_flash_attention_kernel`。初始化或更新了 `query`、`key`、`value`、`block_mask`、`scale`、`kernel_options`、`另有12项` 等值。

### Lines 341-360 / 第 341-360 行
````python
) -> tuple[TensorBox, TensorBox]:
    """Create a flex flash attention kernel using CuteDSL template."""
    if query.dtype != key.dtype or query.dtype != value.dtype:
        raise ValueError(
            f"Mixed query, key, and value dtype is not supported on this platform, "
            f"got query.dtype: {query.dtype}, key.dtype: {key.dtype}, "
            f"and value.dtype: {value.dtype}."
        )
    if not ensure_flash_available():
        raise RuntimeError("CUTE flash attention not available")

    # Get dimensions
    batch_size, num_heads, seq_len_q, head_dim = query.get_size()
    v_head_dim = value.get_size()[-1]
    device = query.get_device()
    dtype = query.get_dtype()
    assert device is not None, "Device must be specified"

    # Match stride pattern from query tensor
    q_strides = query.get_stride()
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 361-380 / 第 361-380 行
````python
    out_size = [batch_size, num_heads, seq_len_q, v_head_dim]
    out_strides = infer_dense_strides(out_size, q_strides)

    output = empty_strided(
        size=out_size,
        stride=out_strides,
        dtype=dtype,
        device=device,
    )

    lse = empty_strided(
        size=[batch_size, num_heads, seq_len_q],
        stride=None,  # LSE can be contiguous
        dtype=torch.float32,  # LSE is always fp32
        device=device,
    )

    # Create layout for primary output
    output_layout = FixedLayout(
        device=device,
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `out_size`, `out_strides`, `output`, `size`, `stride`, `dtype`, and `...+3`. This range continues the implementation of function `create_flex_flash_attention_kernel`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `out_size`、`out_strides`、`output`、`size`、`stride`、`dtype`、`另有3项` 等值。这一段延续了函数`create_flex_flash_attention_kernel` 的具体实现。

### Lines 381-400 / 第 381-400 行
````python
        dtype=dtype,
        size=[batch_size, num_heads, seq_len_q, v_head_dim],
        stride=[sympy.sympify(s) for s in output.get_stride()],
    )

    sparse_q_block_size = V.graph.sizevars.guard_int(sparse_q_block_size)
    sparse_kv_block_size = V.graph.sizevars.guard_int(sparse_kv_block_size)

    mask_graph_is_trivial = is_trivial_mask_graph(mask_graph.graph_module)
    score_graph_is_trivial = subgraph is None or is_trivial_score_graph(
        subgraph.graph_module
    )

    needs_block_mask = not mask_graph_is_trivial
    has_score_mod = not score_graph_is_trivial
    has_full_blocks = full_kv_num_blocks is not None

    choices: list[Any] = []
    assert flash_attention_cutedsl_template is not None

````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `dtype`, `size`, `stride`, `sparse_q_block_size`, `sparse_kv_block_size`, `mask_graph_is_trivial`, and `...+5`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `dtype`、`size`、`stride`、`sparse_q_block_size`、`sparse_kv_block_size`、`mask_graph_is_trivial`、`另有5项` 等值。

### Lines 401-420 / 第 401-420 行
````python
    input_nodes = [query, key, value, lse]
    if has_full_blocks:
        input_nodes.extend(
            [kv_num_blocks, kv_indices, full_kv_num_blocks, full_kv_indices]
        )

    if needs_block_mask and not has_full_blocks:
        raise NotImplementedError(
            "Flash attention with block mask but without full blocks is not supported yet"
        )

    subgraphs = []
    if has_score_mod:
        subgraphs.append(subgraph_buffer)
    subgraphs.append(mask_graph_buffer)

    configs = _get_flex_flash_fwd_configs(
        has_score_mod, len(score_mod_other_buffers) > 0
    )

````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `input_nodes`, `subgraphs`, and `configs`. This range continues the implementation of function `create_flex_flash_attention_kernel`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `input_nodes`、`subgraphs`、`configs` 等值。这一段延续了函数`create_flex_flash_attention_kernel` 的具体实现。

### Lines 421-440 / 第 421-440 行
````python
    error: NotImplementedError | None = None
    for conf in configs:
        with patch_fixed_layout_indexer_for_cutedsl():
            error = flash_attention_cutedsl_template.maybe_append_choice(
                choices,
                input_nodes=input_nodes,
                layout=output_layout,
                mutated_inputs=[lse],
                subgraphs=subgraphs,
                SM_SCALE=scale,
                HAS_SCORE_MOD=has_score_mod,
                SCORE_MOD_VEC_SIZE=conf.score_mod_vec_size,
                NEEDS_BLOCK_MASK=needs_block_mask,
                SPARSE_Q_BLOCK_SIZE=sparse_q_block_size,
                SPARSE_KV_BLOCK_SIZE=sparse_kv_block_size,
            )
        if error is not None and len(configs) == 1:
            raise RuntimeError(f"CuteDSL template failed: {error}")

    for choice in choices:
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `error`, `input_nodes`, `layout`, `mutated_inputs`, `subgraphs`, `SM_SCALE`, and `...+5`. This range continues the implementation of function `create_flex_flash_attention_kernel`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `error`、`input_nodes`、`layout`、`mutated_inputs`、`subgraphs`、`SM_SCALE`、`另有5项` 等值。这一段延续了函数`create_flex_flash_attention_kernel` 的具体实现。

### Lines 441-460 / 第 441-460 行
````python
        wrap_choice_render_with_cutedsl_indexer(choice)

    if not choices:
        raise RuntimeError(f"CuteDSL template failed: {error}")

    input_gen_fns: dict[int, Callable] | None = None
    if has_full_blocks:
        input_gen_fns = {
            4: create_num_blocks_fake_generator(kv_indices),
            5: create_indices_fake,
            6: create_num_blocks_fake_generator(full_kv_indices),
            7: create_indices_fake,
        }

    template_output, _ = autotune_select_algorithm(
        "flex_flash_attention",
        choices,
        input_nodes,
        output_layout,
        input_gen_fns=input_gen_fns,
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `input_gen_fns`. This range continues the implementation of function `create_flex_flash_attention_kernel`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `input_gen_fns` 等值。这一段延续了函数`create_flex_flash_attention_kernel` 的具体实现。

### Lines 461-480 / 第 461-480 行
````python
        return_multi_template=False,
    )

    return (template_output, lse)


def _can_use_flex_flash_attention_backward(
    fw_subgraph: Subgraph,
    mask_graph: Subgraph,
    joint_outputs: Any | None = None,
    score_mod_other_buffers: Sequence[TensorBox] | None = None,
    num_score_mod_placeholders: int = 5,
) -> tuple[bool, str]:
    if not ensure_flash_available():
        return False, "CUTE flash attention is not available"

    if input_buffers_require_grads(
        fw_subgraph.graph_module, num_score_mod_placeholders
    ):
        return (
````
- **EN**: Introduces function `_can_use_flex_flash_attention_backward`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `return_multi_template`, `fw_subgraph`, `mask_graph`, `joint_outputs`, `score_mod_other_buffers`, and `num_score_mod_placeholders`.
- **CN**: 这里定义了函数`_can_use_flex_flash_attention_backward`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `return_multi_template`、`fw_subgraph`、`mask_graph`、`joint_outputs`、`score_mod_other_buffers`、`num_score_mod_placeholders` 等值。

### Lines 481-500 / 第 481-500 行
````python
            False,
            "Input buffers require gradients (not supported by flash attention backward)",
        )

    if joint_outputs is not None:
        if joint_outputs.captured_grads_compute:
            return (
                False,
                "NYI: Flex Flash Attention bwd doesn't support captured grads yet.",
            )
        if joint_outputs.mutated_grads:
            return (
                False,
                "NYI: Flex Flash Attention bwd doesn't support mutated grads yet.",
            )

    return True, ""


def _use_flex_flash_attention_backward(
````
- **EN**: Introduces function `_use_flex_flash_attention_backward`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_use_flex_flash_attention_backward`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 501-520 / 第 501-520 行
````python
    fw_subgraph: Subgraph,
    mask_graph: Subgraph,
    backend: Literal["AUTO", "TRITON", "FLASH", "TRITON_DECODE"],
    joint_outputs: Any | None = None,
    score_mod_other_buffers: Sequence[TensorBox] | None = None,
) -> bool:
    """Determine if we should use flex flash attention for the given inputs.

    Args:
        fw_subgraph: The forward score modification subgraph
        mask_graph: The mask modification subgraph
        backend: Implementation selector (AUTO, TRITON, FLASH, TRITON_DECODE)
        joint_outputs: Processed joint outputs (for PR1 constraint checking)
        score_mod_other_buffers: Additional buffers used by score_mod

    Returns:
        True if flash attention should be used, False otherwise
    """
    # Flash is experimental and must be explicitly requested
    if backend != "FLASH":
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `fw_subgraph`, `mask_graph`, `backend`, `joint_outputs`, `score_mod_other_buffers`, `Args`, and `...+1`. This range continues the implementation of function `_use_flex_flash_attention_backward`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `fw_subgraph`、`mask_graph`、`backend`、`joint_outputs`、`score_mod_other_buffers`、`Args`、`另有1项` 等值。这一段延续了函数`_use_flex_flash_attention_backward` 的具体实现。

### Lines 521-540 / 第 521-540 行
````python
        return False

    can_use, reason = _can_use_flex_flash_attention_backward(
        fw_subgraph,
        mask_graph,
        joint_outputs,
        score_mod_other_buffers,
    )

    if not can_use:
        raise RuntimeError(
            f"BACKEND='FLASH' but flash attention cannot be used: {reason}"
        )

    return True


def create_flex_flash_attention_backward_kernel(
    query: TensorBox,
    key: TensorBox,
````
- **EN**: Introduces function `create_flex_flash_attention_backward_kernel`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `query`, and `key`.
- **CN**: 这里定义了函数`create_flex_flash_attention_backward_kernel`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `query`、`key` 等值。

### Lines 541-560 / 第 541-560 行
````python
    value: TensorBox,
    out: TensorBox,
    logsumexp: TensorBox,
    grad_out: TensorBox,
    scale: float,
    kernel_options: dict[str, Any],
    sparse_q_block_size: int,
    sparse_kv_block_size: int,
    fw_subgraph_buffer: SubgraphResults | None = None,
    joint_subgraph_buffer: Any | None = None,
    score_mod_other_buffers: list[TensorBox] | None = None,
    mask_graph_buffer: SubgraphResults | None = None,
    q_num_blocks: TensorBox | None = None,
    q_indices: TensorBox | None = None,
    full_q_num_blocks: TensorBox | None = None,
    full_q_indices: TensorBox | None = None,
) -> tuple[TensorBox | ShapeAsConstantBuffer, TensorBox, TensorBox, tuple]:
    """Create a CuteDSL flash attention backward kernel for the default mod path."""
    if not ensure_flash_available():
        raise RuntimeError("CUTE flash attention not available")
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `value`, `out`, `logsumexp`, `grad_out`, `scale`, `kernel_options`, and `...+10`. This range continues the implementation of function `create_flex_flash_attention_backward_kernel`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `value`、`out`、`logsumexp`、`grad_out`、`scale`、`kernel_options`、`另有10项` 等值。这一段延续了函数`create_flex_flash_attention_backward_kernel` 的具体实现。

### Lines 561-580 / 第 561-580 行
````python

    batch_size, num_heads, seq_len_q, head_dim = query.get_size()
    _, num_heads_kv, seq_len_kv, v_head_dim = value.get_size()
    device = query.get_device()
    dtype = query.get_dtype()
    assert device is not None

    grad_query_strides = infer_dense_strides(
        [batch_size, num_heads, seq_len_q, head_dim], query.get_stride()
    )
    grad_query = empty_strided(
        size=[batch_size, num_heads, seq_len_q, head_dim],
        stride=grad_query_strides,
        dtype=dtype,
        device=device,
    )

    grad_key_strides = infer_dense_strides(
        [batch_size, num_heads_kv, seq_len_kv, head_dim], key.get_stride()
    )
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `device`, `dtype`, `grad_query_strides`, `grad_query`, `size`, `stride`, and `...+1`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `device`、`dtype`、`grad_query_strides`、`grad_query`、`size`、`stride`、`另有1项` 等值。

### Lines 581-600 / 第 581-600 行
````python
    grad_key = empty_strided(
        size=[batch_size, num_heads_kv, seq_len_kv, head_dim],
        stride=grad_key_strides,
        dtype=dtype,
        device=device,
    )

    grad_value_strides = infer_dense_strides(
        [batch_size, num_heads_kv, seq_len_kv, v_head_dim], value.get_stride()
    )
    grad_value = empty_strided(
        size=[batch_size, num_heads_kv, seq_len_kv, v_head_dim],
        stride=grad_value_strides,
        dtype=dtype,
        device=device,
    )

    # we use dq as the output layout
    output_layout = FixedLayout(
        device=device,
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `grad_key`, `size`, `stride`, `dtype`, `device`, `grad_value_strides`, and `...+2`. This range continues the implementation of function `create_flex_flash_attention_backward_kernel`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `grad_key`、`size`、`stride`、`dtype`、`device`、`grad_value_strides`、`另有2项` 等值。这一段延续了函数`create_flex_flash_attention_backward_kernel` 的具体实现。

### Lines 601-620 / 第 601-620 行
````python
        dtype=dtype,
        size=[batch_size, num_heads, seq_len_q, head_dim],
        stride=[sympy.sympify(s) for s in grad_query.get_stride()],
    )

    sparse_q_block_size = V.graph.sizevars.guard_int(sparse_q_block_size)
    sparse_kv_block_size = V.graph.sizevars.guard_int(sparse_kv_block_size)

    choices: list[Any] = []

    input_nodes: list[TensorBox] = [
        query,
        key,
        value,
        out,
        grad_out,
        logsumexp,
        grad_key,
        grad_value,
    ]
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `dtype`, `size`, `stride`, `sparse_q_block_size`, `sparse_kv_block_size`, `choices`, and `...+1`. This range continues the implementation of function `create_flex_flash_attention_backward_kernel`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `dtype`、`size`、`stride`、`sparse_q_block_size`、`sparse_kv_block_size`、`choices`、`另有1项` 等值。这一段延续了函数`create_flex_flash_attention_backward_kernel` 的具体实现。

### Lines 621-640 / 第 621-640 行
````python

    has_block_mask = mask_graph_buffer is not None
    if has_block_mask:
        assert q_indices is not None
        assert full_q_num_blocks is not None
        assert full_q_indices is not None
        input_nodes.extend(
            [
                cast(TensorBox, q_num_blocks),
                q_indices,
                full_q_num_blocks,
                full_q_indices,
            ]
        )

    has_score_mod = fw_subgraph_buffer is not None and joint_subgraph_buffer is not None
    subgraphs = []
    if has_score_mod:
        subgraphs.append(fw_subgraph_buffer)
        subgraphs.append(joint_subgraph_buffer)
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `has_block_mask`, `has_score_mod`, and `subgraphs`. This range continues the implementation of function `create_flex_flash_attention_backward_kernel`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `has_block_mask`、`has_score_mod`、`subgraphs` 等值。这一段延续了函数`create_flex_flash_attention_backward_kernel` 的具体实现。

### Lines 641-660 / 第 641-660 行
````python
    if has_block_mask:
        subgraphs.append(mask_graph_buffer)

    configs = _get_flex_flash_bwd_configs()

    error: NotImplementedError | None = None
    for conf in configs:
        with patch_fixed_layout_indexer_for_cutedsl():
            error = flash_attention_backward_cutedsl_template.maybe_append_choice(
                choices,
                input_nodes=input_nodes,
                layout=output_layout,
                mutated_inputs=[grad_key, grad_value],
                subgraphs=subgraphs or None,
                SM_SCALE=scale,
                HAS_SCORE_MOD=has_score_mod,
                SCORE_MOD_VEC_SIZE=conf.score_mod_vec_size,
                HAS_BLOCK_MASK=has_block_mask,
                SPARSE_Q_BLOCK_SIZE=sparse_q_block_size,
                SPARSE_KV_BLOCK_SIZE=sparse_kv_block_size,
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `configs`, `error`, `input_nodes`, `layout`, `mutated_inputs`, `subgraphs`, and `...+6`. This range continues the implementation of function `create_flex_flash_attention_backward_kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `configs`、`error`、`input_nodes`、`layout`、`mutated_inputs`、`subgraphs`、`另有6项` 等值。这一段延续了函数`create_flex_flash_attention_backward_kernel` 的具体实现。

### Lines 661-680 / 第 661-680 行
````python
            )
        if error is not None and len(configs) == 1:
            raise RuntimeError(f"CuteDSL template failed: {error}")

    for choice in choices:
        wrap_choice_render_with_cutedsl_indexer(choice)

    if not choices:
        raise RuntimeError(f"CuteDSL template failed: {error}")

    input_gen_fns: dict[int, Callable] | None = None
    if has_block_mask:
        input_gen_fns = {
            8: create_num_blocks_fake_generator(q_indices),
            9: create_indices_fake,
            10: create_num_blocks_fake_generator(full_q_indices),
            11: create_indices_fake,
        }

    template_output, _ = autotune_select_algorithm(
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `input_gen_fns`. This range continues the implementation of function `create_flex_flash_attention_backward_kernel`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `input_gen_fns` 等值。这一段延续了函数`create_flex_flash_attention_backward_kernel` 的具体实现。

### Lines 681-689 / 第 681-689 行
````python
        "flex_flash_attention_backward",
        choices,
        input_nodes,
        output_layout,
        input_gen_fns=input_gen_fns,
        return_multi_template=False,
    )

    return (template_output, grad_key, grad_value, tuple())
````
- **EN**: Includes returns or checks that define the contract of this code path. Initializes or updates values such as `input_gen_fns`, and `return_multi_template`. This range continues the implementation of function `create_flex_flash_attention_backward_kernel`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `input_gen_fns`、`return_multi_template` 等值。这一段延续了函数`create_flex_flash_attention_backward_kernel` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Autotuning support  
  **CN**: 自动调优支持

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `functools`, `importlib`, `collections.abc`, `contextlib`, `typing`
- **Third-party / 第三方**: `sympy`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.fx`, `...ir`, `...lowering`, `...select_algorithm`, `...virtualized`, `.common`, `...codegen.cutedsl.cutedsl_template`, `torch._dynamo.source`, `torch._inductor.virtualized`
