# flex_decoding.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/kernel/flex/flex_decoding.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It exposes functions such as `_use_flex_decoding`, `flex_decoding_grid`, `get_split_k`, and `create_flex_decoding_kernel`. Module note: Triton Implementation of the flex_attention Kernel for short query length (FlexDecoding)
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。同时提供 `_use_flex_decoding`、`flex_decoding_grid`、`get_split_k`、`create_flex_decoding_kernel` 等函数。模块文档字符串给出了额外背景说明。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: allow-untyped-defs
"""Triton Implementation of the flex_attention Kernel for short query length (FlexDecoding)"""

import logging
from typing import Any

import sympy

import torch
from torch._inductor.virtualized import V
from torch.utils._sympy.functions import FloorDiv, Mod

from ... import ir
from ...ir import FixedLayout, FlexibleLayout
from ...lowering import empty, empty_strided, lowerings
from ...runtime.runtime_utils import ceildiv, is_power_of_2, next_power_of_2
from ...select_algorithm import (
    autotune_select_algorithm,
    SymbolicGridFn,
    TritonTemplate,
````
- **EN**: Imports dependencies such as `logging`, `typing`, `sympy`, `torch`, `torch._inductor.virtualized`, `torch.utils._sympy.functions`, and `...+5` for the logic in this range. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution.
- **CN**: 这里导入了 `logging`、`typing`、`sympy`、`torch`、`torch._inductor.virtualized`、`torch.utils._sympy.functions`、`另有5项` 等依赖，为后续逻辑提供基础能力。借助符号表达式来推理索引、形状或代数性质。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。

### Lines 21-40 / 第 21-40 行
````python
)
from ...utils import can_use_tma
from .common import (
    create_indices_fake,
    create_num_blocks_fake_generator,
    freeze_irnodes,
    get_fwd_subgraph_outputs,
    load_flex_template,
    maybe_realize,
    set_head_dim_values,
)


aten = torch.ops.aten
prims = torch.ops.prims

log = logging.getLogger(__name__)


def _use_flex_decoding(query, kv_indices, value, kernel_options, enable_gqa) -> bool:
````
- **EN**: Imports dependencies such as `...utils`, and `.common` for the logic in this range. Introduces function `_use_flex_decoding`. Initializes or updates values such as `aten`, `prims`, and `log`.
- **CN**: 这里导入了 `...utils`、`.common` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_use_flex_decoding`。初始化或更新了 `aten`、`prims`、`log` 等值。

### Lines 41-60 / 第 41-60 行
````python
    """Decide which kernel to use, return true if use flex decoding kernel.
    Note:
       Since the number of splits is calculated based of the number of batch and head dims
       we need to ensure that the batch and head dims are statically known. Otherwise we just
       use the main flex_attention kernel.
    """
    force_flex = kernel_options.get("FORCE_USE_FLEX_ATTENTION", False)

    short_query_length = V.graph.sizevars.evaluate_expr(
        sympy.Lt(query.get_size()[-2], 128)
    )
    non_zero_length = V.graph.sizevars.evaluate_expr(sympy.Gt(query.get_size()[-2], 0))
    static_batch = isinstance(query.get_size()[0], (int, sympy.Integer))
    static_num_heads = isinstance(query.get_size()[1], (int, sympy.Integer))
    if enable_gqa:
        # in the current flex decoding triton kernel, grouped query heads for the
        # same kv head are handled by the same block. So it's hard to support different
        # kv num blocks for grouped query heads. We just fall back to main flex_attention
        # kernel where each query head is handled by a separate block.
        valid_block_mask_num_heads = V.graph.sizevars.evaluate_expr(
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 61-80 / 第 61-80 行
````python
            sympy.Eq(kv_indices.get_size()[1], 1)
        )
    else:
        valid_block_mask_num_heads = V.graph.sizevars.evaluate_expr(
            sympy.Or(
                sympy.Eq(kv_indices.get_size()[1], 1),
                sympy.Eq(kv_indices.get_size()[1], query.get_size()[1]),
            )
        )

    Hq = query.get_size()[1]
    Hkv = value.get_size()[1]
    ratio = FloorDiv(Hq, Hkv)

    pw_of_two = V.graph.sizevars.guard_or_false(
        sympy.And(sympy.Gt(ratio, 0), sympy.Eq(ratio & (ratio - 1), 0))
    )

    out = (
        not force_flex
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `else`, `valid_block_mask_num_heads`, `Hq`, `Hkv`, `ratio`, `pw_of_two`, and `...+1`. This range continues the implementation of function `_use_flex_decoding`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含分支、循环或上下文管理等控制流。初始化或更新了 `else`、`valid_block_mask_num_heads`、`Hq`、`Hkv`、`ratio`、`pw_of_two`、`另有1项` 等值。这一段延续了函数`_use_flex_decoding` 的具体实现。

### Lines 81-100 / 第 81-100 行
````python
        and not kernel_options.get("OUTPUT_MAX", False)
        and short_query_length
        and static_batch
        and static_num_heads
        and non_zero_length
        and valid_block_mask_num_heads
        and pw_of_two
    )
    log.debug(
        "Use flex decoding %s, force_flex_attention=%s, short_query_length=%s, static_batch=%s, static_num_heads=%s",
        out,
        force_flex,
        short_query_length,
        static_batch,
        static_num_heads,
    )
    return out


@SymbolicGridFn
````
- **EN**: Applies decorators to register behavior or alter how the following definition is constructed. Includes returns or checks that define the contract of this code path. This range continues the implementation of function `_use_flex_decoding`.
- **CN**: 使用装饰器来注册行为，或改变后续定义的构造方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。这一段延续了函数`_use_flex_decoding` 的具体实现。

### Lines 101-120 / 第 101-120 行
````python
def flex_decoding_grid(batch_size, kv_heads, gqa_group_size, seq_len_q, d_model, meta):
    """How is this kernel parallelized?
    We create a grid of (batch_size * kv_heads, SPLIT_KV, 1)
    Each block is responsible for iterating over blocks of keys and values calculating
    the local output for their tile of keys and values over all full length of query.
    groups of SPLIT_KV blocks then combine their output to produce the final result.
    """

    BLOCK_M = meta["BLOCK_M"]
    num_block_m = ceildiv(seq_len_q * gqa_group_size, BLOCK_M)
    return (num_block_m, batch_size * kv_heads, meta["SPLIT_KV"])


flex_decoding_template = TritonTemplate(
    name="flex_decoding",
    grid=flex_decoding_grid,
    source=load_flex_template("flex_decode")
    + load_flex_template("utilities")
    + load_flex_template("common"),
    always_freeze_layout=True,
````
- **EN**: Introduces function `flex_decoding_grid`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `BLOCK_M`, `num_block_m`, `flex_decoding_template`, `name`, `grid`, `source`, and `...+1`.
- **CN**: 这里定义了函数`flex_decoding_grid`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `BLOCK_M`、`num_block_m`、`flex_decoding_template`、`name`、`grid`、`source`、`另有1项` 等值。

### Lines 121-140 / 第 121-140 行
````python
)


def get_split_k(B: int, H: int, Mk: int) -> int:
    if torch.xpu.is_available():
        num_SM = torch.xpu.get_device_properties("xpu").gpu_subslice_count
    else:
        num_SM = torch.cuda.get_device_properties("cuda").multi_processor_count
    bh = max(B * H, 1)  # NOTE: Handle B*h=0 case
    assert isinstance(bh, (int, sympy.Integer)), "B and H must be concrete integers"
    split_k = num_SM // bh * 2  # Each SM should at least get one block.
    # TODO: workload evening at runtime for splits fully masked out.
    # Before we have runtime workload evening, assign 2 splits per SM.
    split_k = max(split_k, 1)

    return split_k


def create_flex_decoding_kernel(*args, **kwargs):
    """Flex decode lowering that is optimized for small Q_LEN and GQA packing"""
````
- **EN**: Introduces function `get_split_k`, function `create_flex_decoding_kernel`. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`get_split_k`、函数`create_flex_decoding_kernel`。借助符号表达式来推理索引、形状或代数性质。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 141-160 / 第 141-160 行
````python
    (
        query,
        key,
        value,
        block_mask,
        scale,
        kernel_options,
        score_mod_subgraph,
        mask_mod_subgraph,
        score_mod_other_buffers,
        mask_mod_other_buffers,
    ) = args
    (
        _,  # q_length
        _,  # kv_length
        kv_num_blocks,
        kv_indices,
        full_kv_num_blocks,  # full_kv_num_blocks,
        full_kv_indices,  # full_kv_indices,
        _,  # q_num_blocks
````
- **EN**: This range continues the implementation of function `create_flex_decoding_kernel`.
- **CN**: 这一段延续了函数`create_flex_decoding_kernel` 的具体实现。

### Lines 161-180 / 第 161-180 行
````python
        _,  # q_indices
        _,  # full_q_num_blocks,
        _,  # full_q_indices,
        SPARSE_Q_BLOCK_SIZE,
        SPARSE_KV_BLOCK_SIZE,
        _,
    ) = block_mask

    Bq, Hq, seq_len_q, qk_head_dim = query.get_size()
    Bkv, Hkv, seq_len_kv, v_head_dim = value.get_size()

    assert V.graph.sizevars.evaluate_expr(sympy.Eq(Bq, Bkv) | sympy.Eq(Bkv, 1)), (
        f"Bq and Bkv must broadcastable. Got Bq={Bq} and Bkv={Bkv}"
    )

    B = Bq
    kernel_options = dict(kernel_options)
    # Mark symbols in custom kernel options as static shapes and add guards.
    kernel_options = {
        k: V.graph.sizevars.guard_int(v) if isinstance(v, sympy.Symbol) else v
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `B`, `kernel_options`, and `k`. This range continues the implementation of function `create_flex_decoding_kernel`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `B`、`kernel_options`、`k` 等值。这一段延续了函数`create_flex_decoding_kernel` 的具体实现。

### Lines 181-200 / 第 181-200 行
````python
        for k, v in kernel_options.items()
    }

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

    # Calculate GQA head sharing
    gqa_shared_heads = FloorDiv(Hq, Hkv)
    if not is_power_of_2(gqa_shared_heads):
        raise ValueError(
            "Number of shared query heads sharing the same KV head must be power of 2. "
        )
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `seq_q_divisible`, `seq_kv_divisible`, `else`, and `gqa_shared_heads`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `seq_q_divisible`、`seq_kv_divisible`、`else`、`gqa_shared_heads` 等值。

### Lines 201-220 / 第 201-220 行
````python
    kernel_options.setdefault("GQA_SHARED_HEADS", gqa_shared_heads)

    # Determine if there are "full" blocks where we only need to apply score_mod, and can skip mask_mod
    has_full_blocks = full_kv_num_blocks is not None
    kernel_options.setdefault("HAS_FULL_BLOCKS", has_full_blocks)
    if not has_full_blocks:
        # Create a plackeholder full block list in case it is empty
        full_kv_num_blocks, full_kv_indices = (
            empty(0, device=query.get_device()) for _ in range(2)
        )

    (
        query,
        key,
        value,
        kv_num_blocks,
        kv_indices,
        full_kv_num_blocks,
        full_kv_indices,
    ) = maybe_realize(
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `has_full_blocks`. This range continues the implementation of function `create_flex_decoding_kernel`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `has_full_blocks` 等值。这一段延续了函数`create_flex_decoding_kernel` 的具体实现。

### Lines 221-240 / 第 221-240 行
````python
        [
            query,
            key,
            value,
            kv_num_blocks,
            kv_indices,
            full_kv_num_blocks,
            full_kv_indices,
        ]
    )
    score_mod_other_buffers = maybe_realize(score_mod_other_buffers)
    mask_mod_other_buffers = maybe_realize(mask_mod_other_buffers)

    freeze_irnodes(score_mod_other_buffers)
    freeze_irnodes(mask_mod_other_buffers)

    choices: list[Any] = []
    dtype = key.get_dtype()
    head_dim = V.graph.sizevars.guard_int(key.get_size()[-1])
    configs = V.choices.get_flex_decode_configs(
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `score_mod_other_buffers`, `mask_mod_other_buffers`, `choices`, `dtype`, `head_dim`, and `configs`. This range continues the implementation of function `create_flex_decoding_kernel`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `score_mod_other_buffers`、`mask_mod_other_buffers`、`choices`、`dtype`、`head_dim`、`configs` 等值。这一段延续了函数`create_flex_decoding_kernel` 的具体实现。

### Lines 241-260 / 第 241-260 行
````python
        head_dim, dtype, query.get_device().type
    )

    # TODO: fix autotuning.

    kernel_options.setdefault("SM_SCALE", scale)
    kernel_options.setdefault("SPLIT_KV", get_split_k(B, Hkv, seq_len_kv))
    MAX_SPLIT_KV = kernel_options["SPLIT_KV"]

    # create config dependent intermediate buffers
    buf_ACC_shape = [B, MAX_SPLIT_KV, Hq, seq_len_q, v_head_dim]
    buf_ML_shape = buf_ACC_shape[:-1]
    buf_M = empty_strided(
        buf_ML_shape,
        None,
        dtype=torch.float32,  # The rowmax is always stored in fp32 regardless of the input dtype
        device=query.get_device(),
    )
    buf_L = empty_strided(
        buf_ML_shape,
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `MAX_SPLIT_KV`, `buf_ACC_shape`, `buf_ML_shape`, `buf_M`, `dtype`, `device`, and `...+1`. This range continues the implementation of function `create_flex_decoding_kernel`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `MAX_SPLIT_KV`、`buf_ACC_shape`、`buf_ML_shape`、`buf_M`、`dtype`、`device`、`另有1项` 等值。这一段延续了函数`create_flex_decoding_kernel` 的具体实现。

### Lines 261-280 / 第 261-280 行
````python
        None,
        dtype=torch.float32,  # The intermediate sumexp is always stored in fp32 regardless of the input dtype
        device=query.get_device(),
    )

    layout_acc = FixedLayout(
        query.get_device(),
        torch.float32,
        buf_ACC_shape,
        FlexibleLayout.contiguous_strides(buf_ACC_shape),
    )

    set_head_dim_values(kernel_options, qk_head_dim, v_head_dim, V.graph.sizevars)

    kernel_options.setdefault(
        "BLOCK_M",
        (
            # m
            # if V.graph.sizevars.evaluate_expr(sympy.Lt(query.get_size()[-2], 0))
            # else  # Always use a BLOCK_M > 16 before Triton fix https://github.com/triton-lang/triton/pull/4061 is in pin
````
- **EN**: Preserves reference comments that point to design notes, specifications, or vendor datasheets. Uses symbolic expressions to reason about indices, shapes, or algebraic properties. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 保留了指向设计说明、规范或厂商数据手册的注释引用。借助符号表达式来推理索引、形状或代数性质。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。

### Lines 281-300 / 第 281-300 行
````python
            max(
                next_power_of_2(
                    V.graph.sizevars.optimization_hint(seq_len_q) * gqa_shared_heads
                ),
                1 if torch.xpu.is_available() else 16,
            )
        ),
    )

    query = ir.ExternKernel.realize_input(query)
    stride_b, stride_hq, stride_seq_len_q, stride_qk_head_dim = query.get_stride()

    # Reshape query for GQA: [B, Hq, Mq, D] -> [B, Hkv, G, Mq, D]
    gqa_query_shape = (B, Hkv, gqa_shared_heads, seq_len_q, qk_head_dim)
    gqa_query_stride = (
        stride_b,
        stride_hq * gqa_shared_heads,
        stride_hq,
        stride_seq_len_q,
        stride_qk_head_dim,
````
- **EN**: Initializes or updates values such as `query`, `gqa_query_shape`, and `gqa_query_stride`. This range continues the implementation of function `create_flex_decoding_kernel`.
- **CN**: 初始化或更新了 `query`、`gqa_query_shape`、`gqa_query_stride` 等值。这一段延续了函数`create_flex_decoding_kernel` 的具体实现。

### Lines 301-320 / 第 301-320 行
````python
    )
    query = lowerings[aten.as_strided](query, gqa_query_shape, gqa_query_stride)

    kernel_options.setdefault(
        "SAFE_M_BOUNDARY",
        Mod(seq_len_q * gqa_shared_heads, kernel_options["BLOCK_M"]) == 0,
    )
    # TODO: This feels sketchy
    kernel_options.setdefault("SAFE_N_BOUNDARY", True)
    # Mark SPARSE_KV_BLOCK_SIZE as static shapes and add guards.
    SPARSE_Q_BLOCK_SIZE = V.graph.sizevars.guard_int(SPARSE_Q_BLOCK_SIZE)
    SPARSE_KV_BLOCK_SIZE = V.graph.sizevars.guard_int(SPARSE_KV_BLOCK_SIZE)

    original_kernel_options = kernel_options.copy()
    # Note, we don't need to pass in the captured buffers explicitly
    # because they're implicitly added by the score_mod function
    # We do need to explicitly pass it in for autotuning though.

    # Default config for warp specialization
    num_consumer_groups, num_buffers_warp_spec = 0, 0
````
- **EN**: Initializes or updates values such as `query`, `SPARSE_Q_BLOCK_SIZE`, `SPARSE_KV_BLOCK_SIZE`, and `original_kernel_options`. This range continues the implementation of function `create_flex_decoding_kernel`.
- **CN**: 初始化或更新了 `query`、`SPARSE_Q_BLOCK_SIZE`、`SPARSE_KV_BLOCK_SIZE`、`original_kernel_options` 等值。这一段延续了函数`create_flex_decoding_kernel` 的具体实现。

### Lines 321-340 / 第 321-340 行
````python

    for conf in configs:
        if conf.block_n > SPARSE_KV_BLOCK_SIZE:
            conf.block_n = SPARSE_KV_BLOCK_SIZE

        if SPARSE_Q_BLOCK_SIZE % kernel_options["BLOCK_M"] != 0:
            continue

        if SPARSE_KV_BLOCK_SIZE % conf.block_n != 0:
            continue

        cur_kernel_options = original_kernel_options.copy()
        # Remove prefix for forward kernels options and delete backward kernel options.
        for k in list(cur_kernel_options.keys()):
            if k.startswith("fwd_"):
                v = cur_kernel_options.pop(k)
                cur_kernel_options[k[4:]] = v
            if k.startswith("bwd_"):
                cur_kernel_options.pop(k)
        # Performance tuning
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `cur_kernel_options`, and `v`. This range continues the implementation of function `create_flex_decoding_kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `cur_kernel_options`、`v` 等值。这一段延续了函数`create_flex_decoding_kernel` 的具体实现。

### Lines 341-360 / 第 341-360 行
````python
        cur_kernel_options.setdefault("BLOCK_N", conf.block_n)
        cur_kernel_options.setdefault("SPARSE_Q_BLOCK_SIZE", SPARSE_Q_BLOCK_SIZE)
        cur_kernel_options.setdefault("SPARSE_KV_BLOCK_SIZE", SPARSE_KV_BLOCK_SIZE)
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

        # Add ROCm-specific parameters if they exist in the config
        for attrib in ["kpack", "matrix_instr_nonkdim", "waves_per_eu"]:
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. This range continues the implementation of function `create_flex_decoding_kernel`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。这一段延续了函数`create_flex_decoding_kernel` 的具体实现。

### Lines 361-380 / 第 361-380 行
````python
            if hasattr(conf, attrib):
                cur_kernel_options[attrib] = getattr(conf, attrib)

        flex_decoding_template.maybe_append_choice(
            choices=choices,
            input_nodes=[
                query,
                key,
                value,
                buf_M,
                buf_L,
                kv_num_blocks,
                kv_indices,
                full_kv_num_blocks,
                full_kv_indices,
            ],
            layout=layout_acc,
            subgraphs=[
                score_mod_subgraph,
                mask_mod_subgraph,
````
- **EN**: Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `choices`, `input_nodes`, `layout`, and `subgraphs`. This range continues the implementation of function `create_flex_decoding_kernel`.
- **CN**: 包含分支、循环或上下文管理等控制流。初始化或更新了 `choices`、`input_nodes`、`layout`、`subgraphs` 等值。这一段延续了函数`create_flex_decoding_kernel` 的具体实现。

### Lines 381-400 / 第 381-400 行
````python
            ],
            mutated_inputs=[buf_M, buf_L],
            call_sizes=query.get_size(),
            **cur_kernel_options,
        )

    filtered_score_mod_buffers = [
        buf for buf in score_mod_other_buffers if not isinstance(buf, sympy.Symbol)
    ]
    filtered_mask_mod_buffers = [
        buf for buf in mask_mod_other_buffers if not isinstance(buf, sympy.Symbol)
    ]

    inputs_for_flex_decoding = (
        [
            query,
            key,
            value,
            buf_M,
            buf_L,
````
- **EN**: Uses symbolic expressions to reason about indices, shapes, or algebraic properties. Initializes or updates values such as `mutated_inputs`, `call_sizes`, `filtered_score_mod_buffers`, `filtered_mask_mod_buffers`, and `inputs_for_flex_decoding`. This range continues the implementation of function `create_flex_decoding_kernel`.
- **CN**: 借助符号表达式来推理索引、形状或代数性质。初始化或更新了 `mutated_inputs`、`call_sizes`、`filtered_score_mod_buffers`、`filtered_mask_mod_buffers`、`inputs_for_flex_decoding` 等值。这一段延续了函数`create_flex_decoding_kernel` 的具体实现。

### Lines 401-420 / 第 401-420 行
````python
            kv_num_blocks,
            kv_indices,
            full_kv_num_blocks,
            full_kv_indices,
        ]
        + filtered_score_mod_buffers
        + filtered_mask_mod_buffers
    )

    input_gen_fns = {
        5: create_num_blocks_fake_generator(kv_indices),
        6: create_indices_fake,
        7: create_num_blocks_fake_generator(full_kv_indices),
        8: create_indices_fake,
    }

    buf_ACC, _ = autotune_select_algorithm(
        "flex_decoding",
        choices,
        inputs_for_flex_decoding,
````
- **EN**: Initializes or updates values such as `input_gen_fns`. This range continues the implementation of function `create_flex_decoding_kernel`.
- **CN**: 初始化或更新了 `input_gen_fns` 等值。这一段延续了函数`create_flex_decoding_kernel` 的具体实现。

### Lines 421-440 / 第 421-440 行
````python
        layout_acc,
        input_gen_fns=input_gen_fns,
    )

    # need subgraph inputs and outputs to analyze all symints used in flex attention
    buf_ACC.data.data.subgraph_inps = list(score_mod_other_buffers) + list(
        mask_mod_other_buffers
    )
    buf_ACC.data.data.subgraph_outs = get_fwd_subgraph_outputs(
        score_mod_subgraph, mask_mod_subgraph
    )

    # Reduction

    g_M = lowerings[aten.max](buf_M, dim=1, keepdim=True)[0]
    # See [Note] Handle fully masked out rows:
    # g_M Is the global max among split kv blocks.
    masked_rows = lowerings[aten.eq](g_M, -float("inf"))
    adj_M = lowerings[aten.sub](buf_M, g_M)
    adj_M = lowerings[aten.where](masked_rows, 0, adj_M)
````
- **EN**: Initializes or updates values such as `input_gen_fns`, `g_M`, `masked_rows`, and `adj_M`. This range continues the implementation of function `create_flex_decoding_kernel`.
- **CN**: 初始化或更新了 `input_gen_fns`、`g_M`、`masked_rows`、`adj_M` 等值。这一段延续了函数`create_flex_decoding_kernel` 的具体实现。

### Lines 441-460 / 第 441-460 行
````python
    alpha = lowerings[aten.exp2](adj_M)

    buf_L = lowerings[aten.mul](buf_L, alpha)
    g_L = lowerings[aten.sum](buf_L, axis=1)
    masked_rows_squeezed = lowerings[aten.squeeze](masked_rows, dim=1)
    g_L = lowerings[aten.where](masked_rows_squeezed, 1.0, g_L)
    logsumexp = lowerings[aten.log2](g_L)
    logsumexp = lowerings[aten.add](logsumexp, lowerings[aten.squeeze](g_M, dim=1))

    alpha_unseq = lowerings[aten.unsqueeze](alpha, 4)
    buf_ACC = lowerings[aten.mul](buf_ACC, alpha_unseq)
    output = lowerings[aten.sum](buf_ACC, axis=1)
    L_unseq = lowerings[aten.unsqueeze](g_L, 3)
    output = lowerings[aten.div](output, L_unseq)
    output = lowerings[prims.convert_element_type](output, query.get_dtype())

    return (
        output,
        logsumexp,
    )
````
- **EN**: Includes returns or checks that define the contract of this code path. Initializes or updates values such as `alpha`, `buf_L`, `g_L`, `masked_rows_squeezed`, `logsumexp`, `alpha_unseq`, and `...+3`. This range continues the implementation of function `create_flex_decoding_kernel`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `alpha`、`buf_L`、`g_L`、`masked_rows_squeezed`、`logsumexp`、`alpha_unseq`、`另有3项` 等值。这一段延续了函数`create_flex_decoding_kernel` 的具体实现。

## Key Concepts / 关键概念
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
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `typing`
- **Third-party / 第三方**: `sympy`
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._inductor.virtualized`, `torch.utils._sympy.functions`, `...`, `...ir`, `...lowering`, `...runtime.runtime_utils`, `...select_algorithm`, `...utils`, `.common`
