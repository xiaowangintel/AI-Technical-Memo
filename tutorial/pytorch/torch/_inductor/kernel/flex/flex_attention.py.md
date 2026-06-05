# flex_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/kernel/flex/flex_attention.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It defines classes such as `JointOutputResult`. It exposes functions such as `_sanitize_kernel_options_for_triton`, `flex_attention_grid`, `get_float32_precision`, `flex_attention`, `flex_attention_backward_grid`, `validate_joint_graph`, and `...+3`. Module note: Triton Implementation of the flex_attention Kernel
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。其中定义了 `JointOutputResult` 等类。同时提供 `_sanitize_kernel_options_for_triton`、`flex_attention_grid`、`get_float32_precision`、`flex_attention`、`flex_attention_backward_grid`、`validate_joint_graph`、`另有3项` 等函数。模块文档字符串给出了额外背景说明。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
````python
# mypy: allow-untyped-defs
"""Triton Implementation of the flex_attention Kernel"""

from __future__ import annotations

import logging
import math
import warnings
from collections.abc import Sequence
from dataclasses import dataclass
from typing import Any, cast, TYPE_CHECKING

import sympy

import torch
from torch._inductor.virtualized import V
from torch.nn.attention.flex_attention import _Backend
from torch.utils._sympy.functions import FloorDiv, Mod

from ...ir import ComputedBuffer, ExternKernel, FixedLayout, TensorBox
from ...lowering import empty, empty_strided, lowerings, register_lowering, to_dtype
from ...select_algorithm import (
    autotune_select_algorithm,
    SymbolicGridFn,
    TritonTemplate,
)
from ...utils import can_use_tma
from .common import (
````
- **EN**: Imports dependencies such as `__future__`, `logging`, `math`, `warnings`, `collections.abc`, `dataclasses`, and `...+11` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution.
- **CN**: 这里导入了 `__future__`、`logging`、`math`、`warnings`、`collections.abc`、`dataclasses`、`另有11项` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。借助符号表达式来推理索引、形状或代数性质。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。

### Lines 29-56 / 第 29-56 行
````python
    build_subgraph_buffer,
    create_indices_fake,
    create_num_blocks_fake_generator,
    create_placeholder,
    freeze_irnodes,
    get_fwd_subgraph_outputs,
    infer_dense_strides,
    load_flex_template,
    maybe_realize,
    set_head_dim_values,
    SubgraphResults,
)
from .flex_cpu import lower_cpu
from .flex_decoding import _use_flex_decoding, create_flex_decoding_kernel
from .flex_flash_attention import (
    _use_flex_flash_attention,
    _use_flex_flash_attention_backward,
    create_flex_flash_attention_backward_kernel,
    create_flex_flash_attention_kernel,
    is_trivial_mask_graph,
    is_trivial_score_graph,
)


if TYPE_CHECKING:
    from ...template_heuristics.triton import FlexBwDConfig, FlexConfig


````
- **EN**: Imports dependencies such as `.flex_cpu`, `.flex_decoding`, `.flex_flash_attention`, and `...template_heuristics.triton` for the logic in this range. Keeps some imports or declarations behind `TYPE_CHECKING` so they are used only for static analysis. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `.flex_cpu`、`.flex_decoding`、`.flex_flash_attention`、`...template_heuristics.triton` 等依赖，为后续逻辑提供基础能力。通过 `TYPE_CHECKING` 包裹部分导入或声明，避免它们在运行时生效，只服务于静态类型检查。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含分支、循环或上下文管理等控制流。

### Lines 57-84 / 第 57-84 行
````python
log = logging.getLogger(__name__)
aten = torch.ops.aten
prims = torch.ops.prims
Expr = sympy.Expr


def _sanitize_kernel_options_for_triton(
    kernel_options: dict[str, Any],
) -> tuple[dict[str, Any], _Backend]:
    """We always strip quotes around str values, we only need this in lowering, so we pop it here
    to avoid passing to triton constexpr dict
    """
    sanitized = dict(kernel_options)
    backend = cast(_Backend, sanitized.pop("BACKEND", "AUTO"))
    return sanitized, backend


@SymbolicGridFn
def flex_attention_grid(batch_size, q_heads, num_queries, d_model, meta, *, cdiv):
    """How is this kernel parallelized?
    We create a grid of (ceil_div(n_queries, query_block_size), batch_size, num_heads)
    Each block is responsible for iterating over blocks of keys and values calculating
    the final attention output.
    """
    return (cdiv(num_queries, meta["BLOCK_M"]), batch_size, q_heads)


def get_float32_precision():
````
- **EN**: Introduces function `_sanitize_kernel_options_for_triton`, function `flex_attention_grid`, function `get_float32_precision`. Applies decorators to register behavior or alter how the following definition is constructed. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution.
- **CN**: 这里定义了函数`_sanitize_kernel_options_for_triton`、函数`flex_attention_grid`、函数`get_float32_precision`。使用装饰器来注册行为，或改变后续定义的构造方式。借助符号表达式来推理索引、形状或代数性质。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。

### Lines 85-112 / 第 85-112 行
````python
    if (
        (
            torch.backends.cuda.matmul.fp32_precision == "ieee"
            if torch.backends.cuda.matmul.fp32_precision != "none"
            else torch.get_float32_matmul_precision() == "highest"
        )
        or torch.version.hip
        or torch.mtia.is_available()
    ):
        return "'ieee'"
    else:
        return "'tf32'"


flex_attention_template = TritonTemplate(
    name="flex_attention",
    grid=flex_attention_grid,
    source=load_flex_template("flex_attention")
    + load_flex_template("utilities")
    + load_flex_template("common"),
    always_freeze_layout=True,
)


@register_lowering(torch.ops.higher_order.flex_attention, type_promotion_kind=None)
def flex_attention(
    query,
    key,
````
- **EN**: Introduces function `flex_attention`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了函数`flex_attention`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 113-140 / 第 113-140 行
````python
    value,
    subgraph,
    block_mask,
    scale,
    kernel_options: dict[str, Any],
    score_mod_other_buffers,
    mask_mod_other_buffers,
):
    """The main lowering for the flex_attention hop
    This can currently lower to one of 3 templates:
    1. Base Triton Template
    2. Flex Decode Triton Template
    3. Cpu specific CPP template
    """
    if query.get_device().type == "cpu":
        return lower_cpu(
            query,
            key,
            value,
            subgraph,
            block_mask,
            scale,
            kernel_options,
            score_mod_other_buffers,
            mask_mod_other_buffers,
        )
    # below is cuda path if device is not cpu
    # tl.dot does not support embedding size less than 16
````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 141-168 / 第 141-168 行
````python
    small_dqk = V.graph.sizevars.evaluate_expr(sympy.Lt(query.get_size()[-1], 16))
    small_dv = V.graph.sizevars.evaluate_expr(sympy.Lt(value.get_size()[-1], 16))
    if small_dqk or small_dv:
        raise NotImplementedError(
            f"NYI: embedding dimension of the query, key, and value must be "
            f"at least 16 but got E={query.get_size()[-1]} and Ev={value.get_size()[-1]}"
        )

    (
        _,  # q_length
        _,  # kv_length
        kv_num_blocks,
        kv_indices,
        full_kv_num_blocks,
        full_kv_indices,
        q_num_blocks,
        q_indices,
        full_q_num_blocks,
        full_q_indices,
        SPARSE_Q_BLOCK_SIZE,
        SPARSE_KV_BLOCK_SIZE,
        mask_graph,
    ) = block_mask

    kernel_options, backend = _sanitize_kernel_options_for_triton(kernel_options)

    # Early check for FLASH backend: detect unsupported captured scalars before
    # building subgraph buffers (which can trigger unbacked_bindings errors)
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `small_dqk`, and `small_dv`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `small_dqk`、`small_dv` 等值。

### Lines 169-196 / 第 169-196 行
````python
    if backend == "FLASH":
        from .flex_flash_attention import _has_unsupported_captured_scalars

        if _has_unsupported_captured_scalars(
            score_mod_other_buffers, mask_mod_other_buffers
        ):
            raise RuntimeError(
                "BACKEND='FLASH' but flash attention cannot be used: "
                "NYI: score_mod or mask_mod captures a dynamic scalar (SymInt/SymFloat). "
                "The FLASH backend cannot inline symbolic values into the CuteDSL template. "
                "Workarounds: use BACKEND='TRITON', compile with dynamic=False, or pass the "
                "value as a tensor on device instead of capturing a Python scalar."
            )

    placeholder_inps = [
        create_placeholder(name, dtype, query.get_device())
        for name, dtype in [
            ("score", query.get_dtype()),
            ("b", torch.int32),
            ("h", torch.int32),
            ("m", torch.int32),
            ("n", torch.int32),
        ]
    ]
    subgraph_buffer = build_subgraph_buffer(
        placeholder_inps + list(score_mod_other_buffers), subgraph
    )
    freeze_irnodes(subgraph_buffer)
````
- **EN**: Imports dependencies such as `.flex_flash_attention` for the logic in this range. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `.flex_flash_attention` 等依赖，为后续逻辑提供基础能力。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 197-224 / 第 197-224 行
````python

    mask_graph_placeholder_inps = [
        create_placeholder(name, dtype, query.get_device())
        for name, dtype in [
            ("b", torch.int32),
            ("h", torch.int32),
            ("m", torch.int32),
            ("n", torch.int32),
        ]
    ]
    mask_graph_buffer = build_subgraph_buffer(
        mask_graph_placeholder_inps + list(mask_mod_other_buffers), mask_graph
    )
    freeze_irnodes(mask_graph_buffer)
    # Mark symbols in custom kernel options as static shapes and add guards.
    kernel_options = {
        k: V.graph.sizevars.guard_int(v) if isinstance(v, sympy.Symbol) else v
        for k, v in kernel_options.items()
    }
    kernel_options.setdefault("FLOAT32_PRECISION", get_float32_precision())
    enable_gqa = V.graph.sizevars.evaluate_expr(
        sympy.Ne(query.get_size()[1], key.get_size()[1]),
    )

    can_use_decode = _use_flex_decoding(
        query, kv_indices, value, kernel_options, enable_gqa
    )
    use_decode = (backend == "TRITON_DECODE") or (backend == "AUTO" and can_use_decode)
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `mask_graph_placeholder_inps`, `mask_graph_buffer`, `kernel_options`, `k`, `enable_gqa`, `can_use_decode`, and `...+1`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `mask_graph_placeholder_inps`、`mask_graph_buffer`、`kernel_options`、`k`、`enable_gqa`、`can_use_decode`、`另有1项` 等值。

### Lines 225-252 / 第 225-252 行
````python

    if backend == "TRITON_DECODE" and not can_use_decode:
        raise RuntimeError(
            "BACKEND='TRITON_DECODE' was specified but flex_decoding cannot be used for this input. "
            "flex_decoding is only available for short sequence lengths with specific configurations."
        )

    if use_decode:
        return create_flex_decoding_kernel(
            query,
            key,
            value,
            block_mask,
            scale,
            kernel_options,
            subgraph_buffer,
            mask_graph_buffer,
            score_mod_other_buffers,
            mask_mod_other_buffers,
        )

    (
        query,
        key,
        value,
        kv_num_blocks,
        kv_indices,
        full_kv_num_blocks,
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `flex_attention`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。这一段延续了函数`flex_attention` 的具体实现。

### Lines 253-280 / 第 253-280 行
````python
        full_kv_indices,
        q_num_blocks,
        q_indices,
        full_q_num_blocks,
        full_q_indices,
    ) = maybe_realize(
        [
            query,
            key,
            value,
            kv_num_blocks,
            kv_indices,
            full_kv_num_blocks,
            full_kv_indices,
            q_num_blocks,
            q_indices,
            full_q_num_blocks,
            full_q_indices,
        ]
    )

    if _use_flex_flash_attention(
        subgraph,
        mask_graph,
        kernel_options,
        num_score_mod_placeholders=len(placeholder_inps),
        backend=backend,
    ):
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `num_score_mod_placeholders`, and `backend`. This range continues the implementation of function `flex_attention`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `num_score_mod_placeholders`、`backend` 等值。这一段延续了函数`flex_attention` 的具体实现。

### Lines 281-308 / 第 281-308 行
````python
        return create_flex_flash_attention_kernel(
            query,
            key,
            value,
            block_mask,
            scale,
            kernel_options,
            subgraph_buffer,
            mask_graph_buffer,
            score_mod_other_buffers,
            mask_mod_other_buffers,
            kv_num_blocks,
            kv_indices,
            full_kv_num_blocks,
            full_kv_indices,
            SPARSE_Q_BLOCK_SIZE,
            SPARSE_KV_BLOCK_SIZE,
            mask_graph=mask_graph,
            subgraph=subgraph,
        )

    score_mod_other_buffers = maybe_realize(score_mod_other_buffers)
    mask_mod_other_buffers = maybe_realize(mask_mod_other_buffers)

    freeze_irnodes(score_mod_other_buffers)
    freeze_irnodes(mask_mod_other_buffers)

    Bq, Hq, seq_len_q, qk_head_dim = query.get_size()
````
- **EN**: Includes returns or checks that define the contract of this code path. Initializes or updates values such as `mask_graph`, `subgraph`, `score_mod_other_buffers`, and `mask_mod_other_buffers`. This range continues the implementation of function `flex_attention`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `mask_graph`、`subgraph`、`score_mod_other_buffers`、`mask_mod_other_buffers` 等值。这一段延续了函数`flex_attention` 的具体实现。

### Lines 309-336 / 第 309-336 行
````python
    Bkv, Hkv, seq_len_kv, v_head_dim = value.get_size()
    assert V.graph.sizevars.evaluate_expr(sympy.Eq(Bq, Bkv) | sympy.Eq(Bkv, 1)), (
        f"Bq and Bkv must broadcastable. Got Bq={Bq} and Bkv={Bkv}"
    )
    assert V.graph.sizevars.evaluate_expr(sympy.Gt(seq_len_q, 0)), (
        "Query length must be greater than 0"
    )
    assert V.graph.sizevars.evaluate_expr(sympy.Gt(seq_len_kv, 0)), (
        "Key length must be greater than 0"
    )

    B = Bq

    seq_q_divisible = V.graph.sizevars.statically_known_true(
        sympy.Eq(Mod(seq_len_q, 128), 0)
    )
    seq_kv_divisible = V.graph.sizevars.statically_known_true(
        sympy.Eq(Mod(seq_len_kv, 128), 0)
    )
    if seq_q_divisible and seq_kv_divisible:
        kernel_options.setdefault("IS_DIVISIBLE", True)
    else:
        kernel_options.setdefault("IS_DIVISIBLE", False)

    # NB it is okay that the v_head_dim is different
    # We are using these to match fill order of the output.
    q_strides = query.get_stride()
    # Construct output layout with strides matching the query.
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `B`, `seq_q_divisible`, `seq_kv_divisible`, `else`, and `q_strides`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `B`、`seq_q_divisible`、`seq_kv_divisible`、`else`、`q_strides` 等值。

### Lines 337-364 / 第 337-364 行
````python
    out_size = [B, Hq, seq_len_q, v_head_dim]
    out_strides = infer_dense_strides(out_size, q_strides)

    layout = FixedLayout(
        query.get_device(),
        query.get_dtype(),
        [B, Hq, seq_len_q, v_head_dim],
        stride=[sympy.sympify(s) for s in out_strides],
    )
    # see NOTE:[TritonTemplates with multiple outputs]
    logsumexp_shape = [B, Hq, seq_len_q]
    logsumexp = empty_strided(
        logsumexp_shape,
        None,
        dtype=torch.float32,  # The logsumexp is always stored in fp32 regardless of the input dtype
        device=query.get_device(),
    )
    max_scores = empty_strided(
        logsumexp_shape,  # Same shape as logsumexp
        None,
        dtype=torch.float32,  # The max scores are always stored in fp32 regardless of the input dtype
        device=query.get_device(),
    )
    kernel_options.setdefault("SM_SCALE", scale)

    # Determine GQA broadcast factor.
    gqa_shared_heads = FloorDiv(Hq, Hkv)
    kernel_options.setdefault("GQA_SHARED_HEADS", gqa_shared_heads)
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `out_size`, `out_strides`, `layout`, `stride`, `logsumexp_shape`, `logsumexp`, and `...+4`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `out_size`、`out_strides`、`layout`、`stride`、`logsumexp_shape`、`logsumexp`、`另有4项` 等值。

### Lines 365-392 / 第 365-392 行
````python

    # Inside of Triton kernel, only apply partial masking if partial blocks are computed.
    # full_kv_num_blocks is None if partial blocks are not computed
    has_full_blocks = full_kv_num_blocks is not None
    kernel_options.setdefault("HAS_FULL_BLOCKS", has_full_blocks)
    if not has_full_blocks:
        full_kv_num_blocks, full_kv_indices = (
            empty(0, device=query.get_device()) for _ in range(2)
        )

    set_head_dim_values(kernel_options, qk_head_dim, v_head_dim, V.graph.sizevars)

    choices: list[Any] = []

    dtype = query.get_dtype()
    head_dim = V.graph.sizevars.guard_int(query.get_size()[-1])
    configs: list[FlexConfig] = V.choices.get_flex_attention_fwd_configs(
        head_dim, dtype, query.get_device().type
    )

    # Mark SPARSE_KV_BLOCK_SIZE & SPARSE_Q_BLOCK_SIZE as static shapes and add guards.
    SPARSE_KV_BLOCK_SIZE = V.graph.sizevars.guard_int(SPARSE_KV_BLOCK_SIZE)
    SPARSE_Q_BLOCK_SIZE = V.graph.sizevars.guard_int(SPARSE_Q_BLOCK_SIZE)

    # Note, we don't need to pass in the captured buffers explicitly
    # because they're implicitly added by the score_mod function
    # We do need to explicitly pass it in for autotuning though.
    original_kernel_options = kernel_options.copy()
````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。

### Lines 393-420 / 第 393-420 行
````python
    # Default config for warp specialization
    num_consumer_groups, num_buffers_warp_spec = 0, 0

    for conf in configs:
        cur_kernel_options = original_kernel_options.copy()
        # Performance tuning
        # Triton parameters
        # Remove prefix for forward kernels options and delete backward kernel options.
        for k in list(cur_kernel_options.keys()):
            if k.startswith("fwd_"):
                v = cur_kernel_options.pop(k)
                cur_kernel_options[k[4:]] = v
            if k.startswith("bwd_"):
                cur_kernel_options.pop(k)
        cur_kernel_options.setdefault("num_stages", conf.num_stages)
        cur_kernel_options.setdefault("num_warps", conf.num_warps)
        if cur_kernel_options.get("num_consumer_groups", False):
            cur_kernel_options.setdefault("num_consumer_groups", num_consumer_groups)
            cur_kernel_options.setdefault(
                "num_buffers_warp_spec", num_buffers_warp_spec
            )

        # Intel GPU enables TMA by default
        cur_kernel_options.setdefault("USE_TMA", bool(torch.xpu.is_available()))

        if cur_kernel_options["USE_TMA"] and not can_use_tma(query, key, value):
            cur_kernel_options["USE_TMA"] = False

````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `cur_kernel_options`, and `v`.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `cur_kernel_options`、`v` 等值。

### Lines 421-448 / 第 421-448 行
````python
        cur_kernel_options.setdefault("BLOCK_M", conf.block_m)
        cur_kernel_options.setdefault("BLOCK_N", conf.block_n)
        # Blocksparse options
        cur_kernel_options.setdefault("SPARSE_Q_BLOCK_SIZE", SPARSE_Q_BLOCK_SIZE)
        cur_kernel_options.setdefault("SPARSE_KV_BLOCK_SIZE", SPARSE_KV_BLOCK_SIZE)

        if (
            cur_kernel_options["SPARSE_KV_BLOCK_SIZE"] % cur_kernel_options["BLOCK_N"]
            != 0
            or cur_kernel_options["SPARSE_Q_BLOCK_SIZE"] % cur_kernel_options["BLOCK_M"]
            != 0
        ):
            if len(configs) == 1:
                raise ValueError(
                    f"Q and KV block size must be divisible by BLOCK_M and BLOCK_N. We "
                    f"got Q_BLOCK_SIZE={cur_kernel_options['SPARSE_Q_BLOCK_SIZE']} and "
                    f"KV_BLOCK_SIZE={cur_kernel_options['SPARSE_KV_BLOCK_SIZE']}."
                )
            continue

        # ROCm specific kernargs
        for attrib in ["kpack", "matrix_instr_nonkdim", "waves_per_eu"]:
            if hasattr(conf, attrib):
                cur_kernel_options[attrib] = getattr(conf, attrib)

        error = flex_attention_template.maybe_append_choice(
            choices=choices,
            input_nodes=[
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `error`, `choices`, and `input_nodes`. This range continues the implementation of function `flex_attention`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `error`、`choices`、`input_nodes` 等值。这一段延续了函数`flex_attention` 的具体实现。

### Lines 449-476 / 第 449-476 行
````python
                query,
                key,
                value,
                logsumexp,
                max_scores,
                kv_num_blocks,
                kv_indices,
                full_kv_num_blocks,
                full_kv_indices,
            ],
            layout=layout,
            subgraphs=[
                subgraph_buffer,
                mask_graph_buffer,
            ],
            mutated_inputs=[
                logsumexp,
                max_scores,
            ],
            call_sizes=query.get_size(),
            **cur_kernel_options,
        )
        if error is not None and len(configs) == 1:
            raise error
    inputs_for_autotuning = (
        [
            query,
            key,
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `layout`, `subgraphs`, `mutated_inputs`, `call_sizes`, and `inputs_for_autotuning`. This range continues the implementation of function `flex_attention`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `layout`、`subgraphs`、`mutated_inputs`、`call_sizes`、`inputs_for_autotuning` 等值。这一段延续了函数`flex_attention` 的具体实现。

### Lines 477-504 / 第 477-504 行
````python
            value,
            logsumexp,
            max_scores,
            kv_num_blocks,
            kv_indices,
            full_kv_num_blocks,
            full_kv_indices,
        ]
        + list(score_mod_other_buffers)
        + list(mask_mod_other_buffers)
    )
    input_gen_fns = {
        5: create_num_blocks_fake_generator(kv_indices),
        6: create_indices_fake,
        7: create_num_blocks_fake_generator(full_kv_indices),
        8: create_indices_fake,
    }

    out, _ = autotune_select_algorithm(
        "flex_attention",
        choices,
        # Need to filter out symbols since there is an invariant
        # that all input_nodes are of type IRNode
        [x for x in inputs_for_autotuning if isinstance(x, torch._inductor.ir.IRNode)],
        layout,
        input_gen_fns=input_gen_fns,
    )

````
- **EN**: Initializes or updates values such as `input_gen_fns`. This range continues the implementation of function `flex_attention`.
- **CN**: 初始化或更新了 `input_gen_fns` 等值。这一段延续了函数`flex_attention` 的具体实现。

### Lines 505-532 / 第 505-532 行
````python
    # need subgraph inputs and outputs to analyze all symints used in flex attention
    out.data.data.subgraph_inps = list(score_mod_other_buffers) + list(
        mask_mod_other_buffers
    )
    out.data.data.subgraph_outs = get_fwd_subgraph_outputs(
        subgraph_buffer, mask_graph_buffer
    )

    return (out, logsumexp, max_scores)


# ---------------------------- Backward HOP Implementation ----------------------------


@SymbolicGridFn
def flex_attention_backward_grid(
    batch_size, q_heads, num_queries, d_model, kv_heads, num_key_value, meta, *, cdiv
):
    """How is this kernel parallelized?
    We create a grid of (ceil_div(n_queries, query_block_size) * heads_ratio + ceil_div(n_kv, kv_block_size), batch_size, kv_heads)
    Currently this is only parallelizing over batch* kv_heads, but we can, and want to
    parallelize over ceil_div(q_heads//kv_heads * num_key_value, key_value_block_size).
    To do this will either require atomic updates to some grad values or to have a two pass kernel design.
    """
    return (
        cdiv(num_queries, meta["BLOCK_M2"]) * (q_heads // kv_heads)
        + cdiv(num_key_value, meta["BLOCK_N1"]),
        batch_size,
````
- **EN**: Introduces function `flex_attention_backward_grid`. Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`flex_attention_backward_grid`。使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 533-560 / 第 533-560 行
````python
        kv_heads,
    )


flex_attention_backward_template = TritonTemplate(
    name="flex_attention_backward",
    grid=flex_attention_backward_grid,
    source=load_flex_template("flex_backwards") + load_flex_template("utilities"),
    always_freeze_layout=True,
)


def validate_joint_graph(joint_graph: torch.fx.Graph):
    """We do some pre lowering graph checks in order to raise nicer error messages"""
    for node in joint_graph.nodes:
        if (
            node.op == "call_function"
            and node.target is torch.ops.flex_lib.zeros_and_scatter.default
        ):
            for user in node.users:
                if user.op != "output":
                    raise NotImplementedError(
                        "Using multiple indexing operations on the same tensor that requires gradients "
                        "in a score_mod function is not currently supported. "
                        "This typically happens when indexing the same tensor multiple times, like:\n\n"
                        "    def score_mod(score, b, h, q_idx, kv_idx):\n"
                        "        return score + bias[q_idx] + bias[kv_idx]  # bias used twice!\n\n"
                        "A valid workaround is to clone() the tensors that will be indexed multiple times. For example:\n\n"
````
- **EN**: Introduces function `validate_joint_graph`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `flex_attention_backward_template`, `name`, `grid`, `source`, and `always_freeze_layout`.
- **CN**: 这里定义了函数`validate_joint_graph`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `flex_attention_backward_template`、`name`、`grid`、`source`、`always_freeze_layout` 等值。

### Lines 561-588 / 第 561-588 行
````python
                        "    bias1 = bias.clone()\n"
                        "    def score_mod(score, b, h, q_idx, kv_idx):\n"
                        "        return score + bias[q_idx] + bias1[kv_idx]\n\n"
                        "Note that this solution will use additional memory."
                    )
    return


@dataclass(frozen=True)
class JointOutputResult:
    """Results from processing joint outputs."""

    grad_input: ComputedBuffer
    captured_grads_compute: list[ComputedBuffer]
    captured_grads: list[TensorBox | None]
    mutated_grads: list[TensorBox]


def process_joint_outputs(
    all_joint_outputs: SubgraphResults, num_placeholders: int
) -> JointOutputResult:
    """Process joint outputs and extract various buffers needed for lowering

    Args:
        all_joint_outputs: List of all the outputs from build_subgraphs
        num_placeholders: The number of placeholder inputs, used to skip over unused backward compute buffers

    Returns:
````
- **EN**: Introduces class `JointOutputResult`, function `process_joint_outputs`. Applies decorators to register behavior or alter how the following definition is constructed. Uses `@dataclass` to model structured state with explicit fields. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了类`JointOutputResult`、函数`process_joint_outputs`。使用装饰器来注册行为，或改变后续定义的构造方式。使用 `@dataclass` 来描述带有显式字段的结构化状态。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 589-616 / 第 589-616 行
````python
        JointOutputResult containing processed buffers and gradients
    """
    assert isinstance(all_joint_outputs, list)
    assert all_joint_outputs[0] is not None, (
        "joint_subgraph_buffer is None - this is a bug!"
    )

    joint_buffer = all_joint_outputs[0]
    other_grads = all_joint_outputs[num_placeholders - 1 :]

    # outer_grads has the structure: Len(other_buffer_grads) if buffer doesn't require grad than it will be None
    # We only grab the buffers that require grad for inlining into kernel
    grads_compute = [buf for buf in other_grads if buf is not None]

    def get_out(buf):
        if buf is None:
            return None
        assert isinstance(buf, ComputedBuffer)
        assert buf.name is not None
        return TensorBox.create(V.graph.get_buffer(buf.name))

    grads_out = [get_out(x) for x in other_grads]
    mutated_grads = [buf for buf in grads_out if buf is not None]

    return JointOutputResult(
        grad_input=joint_buffer,
        captured_grads_compute=grads_compute,
        captured_grads=grads_out,
````
- **EN**: Introduces function `get_out`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `joint_buffer`, `other_grads`, `grads_compute`, `grads_out`, `mutated_grads`, `grad_input`, and `...+2`.
- **CN**: 这里定义了函数`get_out`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `joint_buffer`、`other_grads`、`grads_compute`、`grads_out`、`mutated_grads`、`grad_input`、`另有2项` 等值。

### Lines 617-644 / 第 617-644 行
````python
        mutated_grads=mutated_grads,
    )


# TODO: We probably also need a layout constraint?
@register_lowering(
    torch.ops.higher_order.flex_attention_backward, type_promotion_kind=None
)
def flex_attention_backward(*args, **kwargs):
    """Lowering for the flex_attention_backward op in triton"""
    (
        query,
        key,
        value,
        out,
        logsumexp,
        grad_out,
        grad_logsumexp,
        fw_graph,
        joint_graph,
        block_mask,
        scale,
        kernel_options,
        score_mod_other_buffers,
        mask_mod_other_buffers,
    ) = args
    (
        _,  # q_length
````
- **EN**: Introduces function `flex_attention_backward`. Applies decorators to register behavior or alter how the following definition is constructed. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Builds or updates a registry/mapping so later code can dispatch by name or capability.
- **CN**: 这里定义了函数`flex_attention_backward`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。

### Lines 645-672 / 第 645-672 行
````python
        _,  # kv_length
        kv_num_blocks,
        kv_indices,
        full_kv_num_blocks,
        full_kv_indices,
        q_num_blocks,
        q_indices,
        full_q_num_blocks,
        full_q_indices,
        SPARSE_Q_BLOCK_SIZE,
        SPARSE_KV_BLOCK_SIZE,
        mask_graph,
    ) = block_mask

    (
        query,
        key,
        value,
        logsumexp,
        grad_out,
        kv_num_blocks,
        kv_indices,
        full_kv_num_blocks,
        full_kv_indices,
        q_num_blocks,
        q_indices,
        full_q_num_blocks,
        full_q_indices,
````
- **EN**: This range continues the implementation of function `flex_attention_backward`.
- **CN**: 这一段延续了函数`flex_attention_backward` 的具体实现。

### Lines 673-700 / 第 673-700 行
````python
    ) = maybe_realize(
        [
            query,
            key,
            value,
            logsumexp,
            grad_out,
            kv_num_blocks,
            kv_indices,
            full_kv_num_blocks,
            full_kv_indices,
            q_num_blocks,
            q_indices,
            full_q_num_blocks,
            full_q_indices,
        ]
    )

    device = query.get_device()
    dtype = query.get_dtype()
    Bq, Hq, seq_len_q, qk_head_dim = query.get_size()
    Bkv, Hkv, seq_len_kv, v_head_dim = value.get_size()

    assert V.graph.sizevars.evaluate_expr(sympy.Eq(Bq, Bkv) | sympy.Eq(Bkv, 1)), (
        f"Bq and Bkv must broadcastable. Got Bq={Bq} and Bkv={Bkv}"
    )

    kernel_options, backend = _sanitize_kernel_options_for_triton(kernel_options)
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 701-728 / 第 701-728 行
````python
    # Add check for mixed dtypes
    if query.dtype != key.dtype or query.dtype != value.dtype:
        raise ValueError(
            f"Backward pass with mixed query, key, and value dtype is not supported, "
            f"got query.dtype={query.dtype}, key.dtype={key.dtype}, "
            f"and value.dtype={value.dtype}"
        )
    # Mark symbols in custom kernel options as static shapes and add guards.
    kernel_options = {
        k: V.graph.sizevars.guard_int(v) if isinstance(v, sympy.Symbol) else v
        for k, v in kernel_options.items()
    }
    kernel_options.setdefault("FLOAT32_PRECISION", get_float32_precision())
    kernel_options.setdefault("PRESCALE_QK", False)
    kernel_options.setdefault("ROWS_GUARANTEED_SAFE", False)
    kernel_options.setdefault("BLOCKS_ARE_CONTIGUOUS", False)
    kernel_options.setdefault("WRITE_DQ", True)
    seq_q_divisible = V.graph.sizevars.statically_known_true(
        sympy.Eq(Mod(seq_len_q, 128), 0)
    )
    seq_kv_divisible = V.graph.sizevars.statically_known_true(
        sympy.Eq(Mod(seq_len_kv, 128), 0)
    )
    if seq_q_divisible and seq_kv_divisible:
        kernel_options.setdefault("IS_DIVISIBLE", True)
    else:
        kernel_options.setdefault("IS_DIVISIBLE", False)

````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 729-756 / 第 729-756 行
````python
    fwd_placeholder_inps = [
        create_placeholder(name, dtype, device)
        for name, dtype in [
            ("score", dtype),
            ("b", torch.int32),
            ("h", torch.int32),
            ("m", torch.int32),
            ("n", torch.int32),
        ]
    ]
    fw_subgraph_buffer = build_subgraph_buffer(
        fwd_placeholder_inps + list(score_mod_other_buffers), fw_graph
    )
    freeze_irnodes(fw_subgraph_buffer)

    joint_placeholder_inps = fwd_placeholder_inps + [
        create_placeholder("grad_score_mod", dtype, device)
    ]
    # Sometimes we have weird unused nodes here
    joint_graph.graph_module.graph.eliminate_dead_code()

    # It is hard to raise nice errors for some joint graphs during subgraph lowering
    # This lets us do some checks before attempting to lower
    validate_joint_graph(joint_graph.graph_module.graph)

    all_joint_outputs = build_subgraph_buffer(
        joint_placeholder_inps + list(score_mod_other_buffers),
        joint_graph,
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 757-784 / 第 757-784 行
````python
    )

    freeze_irnodes(all_joint_outputs)

    joint_outputs = process_joint_outputs(
        all_joint_outputs, len(joint_placeholder_inps)
    )

    mask_graph_placeholder_inps = [
        create_placeholder(name, dtype, query.get_device())
        for name, dtype in [
            ("b", torch.int32),
            ("h", torch.int32),
            ("m", torch.int32),
            ("n", torch.int32),
        ]
    ]
    mask_graph_buffer = build_subgraph_buffer(
        mask_graph_placeholder_inps + list(mask_mod_other_buffers), mask_graph
    )
    freeze_irnodes(mask_graph_buffer)

    if _use_flex_flash_attention_backward(
        fw_graph,
        mask_graph,
        backend=backend,
        joint_outputs=joint_outputs,
        score_mod_other_buffers=score_mod_other_buffers,
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `joint_outputs`, `mask_graph_placeholder_inps`, `mask_graph_buffer`, `backend`, and `score_mod_other_buffers`. This range continues the implementation of function `flex_attention_backward`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `joint_outputs`、`mask_graph_placeholder_inps`、`mask_graph_buffer`、`backend`、`score_mod_other_buffers` 等值。这一段延续了函数`flex_attention_backward` 的具体实现。

### Lines 785-812 / 第 785-812 行
````python
    ):
        needs_block_mask = not is_trivial_mask_graph(mask_graph.graph_module)
        if (
            torch.are_deterministic_algorithms_enabled()
            and not torch.is_deterministic_algorithms_warn_only_enabled()
            and needs_block_mask
        ):
            raise NotImplementedError(
                "Deterministic backward for flex_attention with block_mask using the FLASH backend "
                "is not yet implemented. The TRITON backend supports deterministic backward."
            )
        if torch.is_deterministic_algorithms_warn_only_enabled() and needs_block_mask:
            warnings.warn(
                "Deterministic backward for flex_attention with block_mask using the FLASH backend "
                "is not yet implemented. Running non-deterministic backward.",
            )
        # TODO: Implement dLSE support in flash-attention backward by folding
        # grad_logsumexp into the dPsum preprocess step.
        if grad_logsumexp is not None:
            raise NotImplementedError(
                "FLASH backend backward does not support differentiating through "
                "logsumexp (dLSE). This happens when the loss depends on the LSE "
                "output of flex_attention. "
                "Use BACKEND='TRITON' or avoid differentiating through logsumexp."
            )
        score_is_trivial = is_trivial_score_graph(fw_graph.graph_module)
        return create_flex_flash_attention_backward_kernel(
            query,
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `needs_block_mask`, and `score_is_trivial`. This range continues the implementation of function `flex_attention_backward`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `needs_block_mask`、`score_is_trivial` 等值。这一段延续了函数`flex_attention_backward` 的具体实现。

### Lines 813-840 / 第 813-840 行
````python
            key,
            value,
            out,
            logsumexp,
            grad_out,
            scale,
            kernel_options,
            SPARSE_Q_BLOCK_SIZE,
            SPARSE_KV_BLOCK_SIZE,
            fw_subgraph_buffer=None if score_is_trivial else fw_subgraph_buffer,
            joint_subgraph_buffer=None
            if score_is_trivial
            else joint_outputs.grad_input,
            score_mod_other_buffers=list(score_mod_other_buffers),
            mask_graph_buffer=mask_graph_buffer if needs_block_mask else None,
            q_num_blocks=q_num_blocks if needs_block_mask else None,
            q_indices=q_indices if needs_block_mask else None,
            full_q_num_blocks=full_q_num_blocks if needs_block_mask else None,
            full_q_indices=full_q_indices if needs_block_mask else None,
        )

    # Construct layout with stride order matching K
    key_size = [Bq, Hkv, seq_len_kv, qk_head_dim]
    key_strides = infer_dense_strides(key_size, key.get_stride())

    layout_broadcasted_k = FixedLayout(
        key.get_device(),
        key.get_dtype(),
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `fw_subgraph_buffer`, `joint_subgraph_buffer`, `score_mod_other_buffers`, `mask_graph_buffer`, `q_num_blocks`, `q_indices`, and `...+5`. This range continues the implementation of function `flex_attention_backward`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `fw_subgraph_buffer`、`joint_subgraph_buffer`、`score_mod_other_buffers`、`mask_graph_buffer`、`q_num_blocks`、`q_indices`、`另有5项` 等值。这一段延续了函数`flex_attention_backward` 的具体实现。

### Lines 841-868 / 第 841-868 行
````python
        key_size,
        stride=[sympy.sympify(s) for s in key_strides],
    )

    # Create delta which will is needed for the bwd's kernel
    mul_delta = lowerings[aten.mul](out, grad_out)
    delta = lowerings[aten.sum](mul_delta, axis=-1)
    delta = lowerings[prims.convert_element_type](delta, torch.float32)
    if grad_logsumexp is not None:
        grad_lse_exp2 = lowerings[aten.mul](grad_logsumexp, 1 / math.log(2))
        grad_lse_exp2 = ExternKernel.require_contiguous(grad_lse_exp2)
        delta = lowerings[aten.sub](delta, grad_lse_exp2)
        delta = ExternKernel.require_contiguous(delta)
        delta, grad_lse_exp2 = maybe_realize([delta, grad_lse_exp2])
    else:
        delta = ExternKernel.require_contiguous(delta)
        (delta,) = maybe_realize([delta])

    # # see NOTE:[TritonTemplates with multiple outputs]
    query_size = [Bq, Hq, seq_len_q, qk_head_dim]
    grad_query_strides = infer_dense_strides(query_size, query.get_stride())
    grad_query = empty_strided(
        query_size,
        stride=[sympy.sympify(s) for s in grad_query_strides],
        dtype=query.get_dtype(),
        device=query.get_device(),
    )

````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。

### Lines 869-896 / 第 869-896 行
````python
    # Construct output layout with stride order matching value
    value_size = [Bq, Hkv, seq_len_kv, v_head_dim]
    value_strides = infer_dense_strides(value_size, value.get_stride())

    broadcasted_grad_value = empty_strided(
        value_size,
        stride=[sympy.sympify(s) for s in value_strides],
        dtype=value.get_dtype(),
        device=value.get_device(),
    )

    kernel_options.setdefault("SM_SCALE", scale)

    # Determine GQA factor
    gqa_shared_heads = FloorDiv(Hq, Hkv)
    kernel_options.setdefault("GQA_SHARED_HEADS", gqa_shared_heads)

    # Inside of Triton kernel, only apply partial masking if partial blocks are computed.
    # full_kv_num_blocks is torch.zeros([1, 1, 1]) if partial blocks are not computed.
    has_full_blocks = full_kv_num_blocks is not None
    kernel_options.setdefault("HAS_FULL_BLOCKS", has_full_blocks)
    if not has_full_blocks:
        full_kv_num_blocks, full_kv_indices, full_q_num_blocks, full_q_indices = (
            empty(0, device=query.get_device()) for _ in range(4)
        )

    set_head_dim_values(kernel_options, qk_head_dim, v_head_dim, V.graph.sizevars)

````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 897-924 / 第 897-924 行
````python
    SPARSE_Q_BLOCK_SIZE = V.graph.sizevars.guard_int(SPARSE_Q_BLOCK_SIZE)
    SPARSE_KV_BLOCK_SIZE = V.graph.sizevars.guard_int(SPARSE_KV_BLOCK_SIZE)

    choices: list[Any] = []

    dtype = query.get_dtype()
    head_dim = V.graph.sizevars.guard_int(query.get_size()[-1])
    configs: list[FlexBwDConfig] = V.choices.get_flex_attention_bwd_configs(
        head_dim, dtype, query.get_device().type
    )

    # Default config for warp specialization
    num_consumer_groups, num_buffers_warp_spec = 0, 0

    original_kernel_options = kernel_options.copy()

    for conf in configs:
        if (
            SPARSE_KV_BLOCK_SIZE % conf.block_n1 != 0
            or SPARSE_Q_BLOCK_SIZE % conf.block_m1 != 0
            or SPARSE_KV_BLOCK_SIZE % conf.block_n2 != 0
            or SPARSE_Q_BLOCK_SIZE % conf.block_m2 != 0
        ):
            continue

        # Performance tuning
        # Triton heuristics
        cur_kernel_options = original_kernel_options.copy()
````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `SPARSE_Q_BLOCK_SIZE`, `SPARSE_KV_BLOCK_SIZE`, `choices`, `dtype`, `head_dim`, `configs`, and `...+2`.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `SPARSE_Q_BLOCK_SIZE`、`SPARSE_KV_BLOCK_SIZE`、`choices`、`dtype`、`head_dim`、`configs`、`另有2项` 等值。

### Lines 925-952 / 第 925-952 行
````python
        # Remove prefix for backward kernels options and delete forward kernel options.
        for k in list(cur_kernel_options.keys()):
            if k.startswith("bwd_"):
                v = cur_kernel_options.pop(k)
                cur_kernel_options[k[4:]] = v
            if k.startswith("fwd_"):
                cur_kernel_options.pop(k)
        cur_kernel_options.setdefault("num_warps", conf.num_warps)
        cur_kernel_options.setdefault("num_stages", conf.num_stages)

        if cur_kernel_options.get("num_consumer_groups", False):
            cur_kernel_options.setdefault("num_consumer_groups", num_consumer_groups)
            cur_kernel_options.setdefault(
                "num_buffers_warp_spec", num_buffers_warp_spec
            )

        # Intel GPU enables TMA by default
        cur_kernel_options.setdefault("USE_TMA", bool(torch.xpu.is_available()))

        if cur_kernel_options["USE_TMA"] and not can_use_tma(query, key, value):
            cur_kernel_options["USE_TMA"] = False

        cur_kernel_options.setdefault("BLOCK_M1", conf.block_m1)
        cur_kernel_options.setdefault("BLOCK_N1", conf.block_n1)
        cur_kernel_options.setdefault("BLOCK_M2", conf.block_m2)
        cur_kernel_options.setdefault("BLOCK_N2", conf.block_n2)

        # Blocksparse options
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `v`. This range continues the implementation of function `flex_attention_backward`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `v` 等值。这一段延续了函数`flex_attention_backward` 的具体实现。

### Lines 953-980 / 第 953-980 行
````python
        cur_kernel_options.setdefault("SPARSE_Q_BLOCK_SIZE", SPARSE_Q_BLOCK_SIZE)
        cur_kernel_options.setdefault("SPARSE_KV_BLOCK_SIZE", SPARSE_KV_BLOCK_SIZE)

        # ROCm specific kernargs
        for attrib in ["kpack", "matrix_instr_nonkdim", "waves_per_eu"]:
            if hasattr(conf, attrib):
                cur_kernel_options[attrib] = getattr(conf, attrib)

        flex_attention_backward_template.maybe_append_choice(
            choices=choices,
            input_nodes=[
                query,
                key,
                value,
                logsumexp,
                delta,
                grad_out,
                grad_query,
                broadcasted_grad_value,
                kv_num_blocks,
                kv_indices,
                q_num_blocks,
                q_indices,
                full_kv_num_blocks,
                full_kv_indices,
                full_q_num_blocks,
                full_q_indices,
            ],
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `choices`, and `input_nodes`. This range continues the implementation of function `flex_attention_backward`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `choices`、`input_nodes` 等值。这一段延续了函数`flex_attention_backward` 的具体实现。

### Lines 981-1008 / 第 981-1008 行
````python
            layout=layout_broadcasted_k,  # We use store_output only for grad_key
            subgraphs=[
                fw_subgraph_buffer,
                joint_outputs.grad_input,
                mask_graph_buffer,
                joint_outputs.captured_grads_compute,
            ],
            mutated_inputs=[
                grad_query,
                broadcasted_grad_value,
                *joint_outputs.mutated_grads,
            ],
            call_sizes=query.get_size() + key.get_size()[1:3],
            **cur_kernel_options,
        )
    inputs_for_autotuning = (
        [
            query,
            key,
            value,
            logsumexp,
            delta,
            grad_out,
            grad_query,
            broadcasted_grad_value,
            kv_num_blocks,
            kv_indices,
            q_num_blocks,
````
- **EN**: Initializes or updates values such as `layout`, `subgraphs`, `mutated_inputs`, `call_sizes`, and `inputs_for_autotuning`. This range continues the implementation of function `flex_attention_backward`.
- **CN**: 初始化或更新了 `layout`、`subgraphs`、`mutated_inputs`、`call_sizes`、`inputs_for_autotuning` 等值。这一段延续了函数`flex_attention_backward` 的具体实现。

### Lines 1009-1036 / 第 1009-1036 行
````python
            q_indices,
            full_kv_num_blocks,
            full_kv_indices,
            full_q_num_blocks,
            full_q_indices,
        ]
        + list(score_mod_other_buffers)
        + list(mask_mod_other_buffers)
        + joint_outputs.mutated_grads
    )
    input_gen_fns = {
        8: create_num_blocks_fake_generator(kv_indices),  # kv_num_blocks
        9: create_indices_fake,
        10: create_num_blocks_fake_generator(q_indices),  # q_num_blocks
        11: create_indices_fake,
        12: create_num_blocks_fake_generator(full_kv_indices),  # full_kv_num_blocks
        13: create_indices_fake,
        14: create_num_blocks_fake_generator(full_q_indices),  # full_q_num_blocks
        15: create_indices_fake,
    }

    broadcasted_grad_key, _ = autotune_select_algorithm(
        "flex_attention_backward",
        choices,
        [x for x in inputs_for_autotuning if isinstance(x, torch._inductor.ir.IRNode)],
        layout_broadcasted_k,
        input_gen_fns=input_gen_fns,
    )  # [Bq, Hkv, seq_len_kv, k_head_dim]
````
- **EN**: Initializes or updates values such as `input_gen_fns`. This range continues the implementation of function `flex_attention_backward`.
- **CN**: 初始化或更新了 `input_gen_fns` 等值。这一段延续了函数`flex_attention_backward` 的具体实现。

### Lines 1037-1064 / 第 1037-1064 行
````python

    # need subgraph inputs and outputs to analyze all symints used in flex attention
    broadcasted_grad_key.data.data.subgraph_inps = list(score_mod_other_buffers) + list(
        mask_mod_other_buffers
    )
    broadcasted_grad_key.data.data.subgraph_outs = get_bwd_subgraph_outputs(
        fw_subgraph_buffer, mask_graph_buffer, joint_outputs
    )

    if V.graph.sizevars.evaluate_expr(sympy.Eq(Bq, Bkv)):
        grad_key = broadcasted_grad_key
        grad_value = broadcasted_grad_value
    else:
        assert V.graph.sizevars.evaluate_expr(sympy.Gt(Bq, 1) & sympy.Eq(Bkv, 1)), (
            f"Bq and Bkv must broadcastable. "
            f"Got Bq={V.graph.sizevars.evaluate_expr(Bq)} "
            f"and Bkv={V.graph.sizevars.evaluate_expr(Bkv)}"
        )
        grad_key = lowerings[aten.sum](broadcasted_grad_key, axis=0, keepdims=True)
        grad_value = lowerings[aten.sum](broadcasted_grad_value, axis=0, keepdims=True)

    # Cast captured grads to match original buffer dtypes. Gradients are accumulated
    # in fp32 for precision, then cast to the original dtype (e.g., bf16) here.
    captured_grads = tuple(
        to_dtype(g, orig.get_dtype())
        if g is not None and g.get_dtype() != orig.get_dtype()
        else g
        for g, orig in zip(joint_outputs.captured_grads, score_mod_other_buffers)
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1065-1090 / 第 1065-1090 行
````python
    )

    return (grad_query, grad_key, grad_value, captured_grads)


def get_bwd_subgraph_outputs(
    subgraph_buffer: SubgraphResults,
    mask_graph_buffer: SubgraphResults,
    joint_outputs: JointOutputResult,
) -> list[ComputedBuffer | TensorBox | None]:
    subgraph_buffer = (
        subgraph_buffer if isinstance(subgraph_buffer, Sequence) else [subgraph_buffer]
    )
    mask_graph_buffer = (
        mask_graph_buffer
        if isinstance(mask_graph_buffer, Sequence)
        else [mask_graph_buffer]
    )
    joint_output_buffers = [
        joint_outputs.grad_input,
        *joint_outputs.captured_grads_compute,
        *joint_outputs.captured_grads,
        *joint_outputs.mutated_grads,
    ]

    return [*subgraph_buffer, *mask_graph_buffer, *joint_output_buffers]
````
- **EN**: Introduces function `get_bwd_subgraph_outputs`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `subgraph_buffer`, `mask_graph_buffer`, `joint_outputs`, and `joint_output_buffers`.
- **CN**: 这里定义了函数`get_bwd_subgraph_outputs`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `subgraph_buffer`、`mask_graph_buffer`、`joint_outputs`、`joint_output_buffers` 等值。

## Key Concepts / 关键概念
- **EN**: Structured state containers  
  **CN**: 结构化状态容器
- **EN**: Symbolic shape or index reasoning  
  **CN**: 符号化形状或索引推理
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Operator lowering  
  **CN**: 算子下沉
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Autotuning support  
  **CN**: 自动调优支持
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `logging`, `math`, `warnings`, `collections.abc`, `dataclasses`, `typing`
- **Third-party / 第三方**: `sympy`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._inductor.virtualized`, `torch.nn.attention.flex_attention`, `torch.utils._sympy.functions`, `...ir`, `...lowering`, `...select_algorithm`, `...utils`, `.common`, `.flex_cpu`, `.flex_decoding`, `.flex_flash_attention`, `...template_heuristics.triton`
