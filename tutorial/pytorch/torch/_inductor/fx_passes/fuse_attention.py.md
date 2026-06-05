# fuse_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/fuse_attention.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes. It exposes functions such as `_sfdp_pattern_1`, `_sfdp_replacement_1`, `_sfdp_pattern_2`, `_sfdp_replacement_2`, `_sfdp_pattern_3`, `_sfdp_replacement_3`, and `...+56`.
- **用途（中文）**: 该模块实现 FX 图变换 pass。同时提供 `_sfdp_pattern_1`、`_sfdp_replacement_1`、`_sfdp_pattern_2`、`_sfdp_replacement_2`、`_sfdp_pattern_3`、`_sfdp_replacement_3`、`另有56项` 等函数。

## Line-by-Line Analysis / 逐行分析
### Lines 1-28 / 第 1-28 行
````python
# mypy: allow-untyped-defs
import functools
import inspect
import logging
import warnings

import torch

from ..._dynamo.utils import counters
from ..pattern_matcher import (
    filter_nodes,
    fwd_only,
    gen_register_replacement,
    joint_fwd_bwd,
)


log = logging.getLogger(__name__)
aten = torch.ops.aten

_scaled_dot_product_attention = aten.scaled_dot_product_attention


def _sfdp_pattern_1(query, key, value, inv_scale):
    return (
        torch.matmul(query, key.transpose(-2, -1))
        .div(inv_scale)
        .softmax(dim=-1)
````
- **EN**: Imports dependencies such as `functools`, `inspect`, `logging`, `warnings`, `torch`, `..._dynamo.utils`, and `...+1` for the logic in this range. Introduces function `_sfdp_pattern_1`. Builds or updates a registry/mapping so later code can dispatch by name or capability. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `functools`、`inspect`、`logging`、`warnings`、`torch`、`..._dynamo.utils`、`另有1项` 等依赖，为后续逻辑提供基础能力。这里定义了函数`_sfdp_pattern_1`。构建或更新注册表/映射表，方便后续代码按名称或能力进行派发。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 29-56 / 第 29-56 行
````python
        .matmul(value)
    )


def _sfdp_replacement_1(query, key, value, inv_scale):
    counters["inductor"]["fuse_attention"] += 1
    return _scaled_dot_product_attention(
        query,
        key,
        value,
        attn_mask=None,
        dropout_p=0.0,
        is_causal=False,
        scale=1.0 / inv_scale,
    )


def _sfdp_pattern_2(query, key, value, scale_factor):
    return (
        torch.matmul(query, key.transpose(-2, -1))
        .mul(scale_factor)
        .softmax(dim=-1)
        .matmul(value)
    )


def _sfdp_replacement_2(query, key, value, scale_factor):
    counters["inductor"]["fuse_attention"] += 1
````
- **EN**: Introduces function `_sfdp_replacement_1`, function `_sfdp_pattern_2`, function `_sfdp_replacement_2`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `attn_mask`, `dropout_p`, `is_causal`, and `scale`.
- **CN**: 这里定义了函数`_sfdp_replacement_1`、函数`_sfdp_pattern_2`、函数`_sfdp_replacement_2`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `attn_mask`、`dropout_p`、`is_causal`、`scale` 等值。

### Lines 57-84 / 第 57-84 行
````python
    return _scaled_dot_product_attention(
        query,
        key,
        value,
        attn_mask=None,
        dropout_p=0.0,
        is_causal=False,
        scale=scale_factor,
    )


def _sfdp_pattern_3(query, key, value, inv_scale_factor, dropout_p):
    return torch.nn.functional.dropout(
        torch.matmul(query, key.transpose(-2, -1))
        .div(inv_scale_factor)
        .softmax(dim=-1),
        p=dropout_p,
    ).matmul(value)


def _sfdp_replacement_3(query, key, value, inv_scale_factor, dropout_p):
    counters["inductor"]["fuse_attention"] += 1
    return _scaled_dot_product_attention(
        query,
        key,
        value,
        attn_mask=None,
        dropout_p=dropout_p,
````
- **EN**: Introduces function `_sfdp_pattern_3`, function `_sfdp_replacement_3`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `attn_mask`, `dropout_p`, `is_causal`, `scale`, and `p`.
- **CN**: 这里定义了函数`_sfdp_pattern_3`、函数`_sfdp_replacement_3`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `attn_mask`、`dropout_p`、`is_causal`、`scale`、`p` 等值。

### Lines 85-112 / 第 85-112 行
````python
        is_causal=False,
        scale=1.0 / inv_scale_factor,
    )


def _sfdp_pattern_4(query, key, value, scale_factor, dropout_p):
    return torch.nn.functional.dropout(
        torch.matmul(query, key.transpose(-2, -1)).mul(scale_factor).softmax(dim=-1),
        p=dropout_p,
    ).matmul(value)


def _sfdp_replacement_4(query, key, value, scale_factor, dropout_p):
    counters["inductor"]["fuse_attention"] += 1
    return _scaled_dot_product_attention(
        query,
        key,
        value,
        attn_mask=None,
        dropout_p=dropout_p,
        is_causal=False,
        scale=scale_factor,
    )


def _sfdp_pattern_5(query, key, value, attn_mask, inv_scale):
    attn_weight = torch.softmax(
        (query @ key.transpose(-2, -1) / (inv_scale)) + attn_mask, dim=-1
````
- **EN**: Introduces function `_sfdp_pattern_4`, function `_sfdp_replacement_4`, function `_sfdp_pattern_5`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `is_causal`, `scale`, `p`, `attn_mask`, `dropout_p`, and `attn_weight`.
- **CN**: 这里定义了函数`_sfdp_pattern_4`、函数`_sfdp_replacement_4`、函数`_sfdp_pattern_5`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `is_causal`、`scale`、`p`、`attn_mask`、`dropout_p`、`attn_weight` 等值。

### Lines 113-140 / 第 113-140 行
````python
    )
    # attn_weight = torch.dropout(attn_weight, dropout_p)
    return attn_weight @ value


def _sfdp_replacement_5(query, key, value, attn_mask, inv_scale):
    counters["inductor"]["fuse_attention"] += 1
    return _scaled_dot_product_attention(
        query,
        key,
        value,
        attn_mask=attn_mask.to(dtype=query.dtype),
        dropout_p=0.0,
        scale=1.0 / inv_scale,
        is_causal=False,
    )


def _sfdp_pattern_6(query, key, value, attn_mask, inv_scale, dropout_p):
    attn_weight = torch.softmax(
        (query @ key.transpose(-2, -1) / inv_scale) + attn_mask, dim=-1
    )
    attn_weight = torch.dropout(attn_weight, dropout_p, True)
    return attn_weight @ value


def _sfdp_replacement_6(query, key, value, attn_mask, inv_scale, dropout_p):
    counters["inductor"]["fuse_attention"] += 1
````
- **EN**: Introduces function `_sfdp_replacement_5`, function `_sfdp_pattern_6`, function `_sfdp_replacement_6`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `attn_mask`, `dropout_p`, `scale`, `is_causal`, and `attn_weight`.
- **CN**: 这里定义了函数`_sfdp_replacement_5`、函数`_sfdp_pattern_6`、函数`_sfdp_replacement_6`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `attn_mask`、`dropout_p`、`scale`、`is_causal`、`attn_weight` 等值。

### Lines 141-168 / 第 141-168 行
````python
    return _scaled_dot_product_attention(
        query,
        key,
        value,
        attn_mask=attn_mask.to(dtype=query.dtype),
        dropout_p=dropout_p,
        scale=1.0 / inv_scale,
        is_causal=False,
    )


def _sfdp_pattern_7(query, key, value, inv_scale, dropout_p):
    # in real workloads inputs to matmul are permuted
    # causing matmul to expand to a series of expand and clone calls
    # we want the same to happen during pattern tracing
    q = query.permute(0, 2, 1, 3)
    k = key.permute(0, 2, 1, 3)
    v = value.permute(0, 2, 1, 3)
    div = q @ k.transpose(-2, -1) / inv_scale
    div = div.to(torch.float32)
    attn_weight = torch.softmax(div, dim=-1)
    attn_weight = torch.dropout(attn_weight, dropout_p, True)
    attn_weight = attn_weight.to(torch.float16)
    v = v.to(attn_weight.dtype)
    return attn_weight @ v


def _sfdp_replacement_7(query, key, value, inv_scale, dropout_p):
````
- **EN**: Introduces function `_sfdp_pattern_7`, function `_sfdp_replacement_7`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `attn_mask`, `dropout_p`, `scale`, `is_causal`, `q`, `k`, and `...+3`.
- **CN**: 这里定义了函数`_sfdp_pattern_7`、函数`_sfdp_replacement_7`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `attn_mask`、`dropout_p`、`scale`、`is_causal`、`q`、`k`、`另有3项` 等值。

### Lines 169-196 / 第 169-196 行
````python
    # sdpa prefers inputs in permuted format
    # it makes a copy to put them in this format
    # if they aren't already
    # to make replacement efficient ensure that inputs to sdpa
    # are in required order
    counters["inductor"]["fuse_attention"] += 1
    q = query.permute(0, 2, 1, 3)
    k = key.permute(0, 2, 1, 3)
    v = value.permute(0, 2, 1, 3)
    return _scaled_dot_product_attention(
        q,
        k,
        v,
        attn_mask=None,  # attn_mask,
        dropout_p=dropout_p,
        scale=1.0 / inv_scale,
        is_causal=False,
    )


def _sfdp_pattern_8(query, key, value, inv_scale):
    # no dropout version of pattern 7
    q = query.permute(0, 2, 1, 3)
    k = key.permute(0, 2, 1, 3)
    v = value.permute(0, 2, 1, 3)
    div = q @ k.transpose(-2, -1) / inv_scale
    div = div.to(torch.float32)
    attn_weight = torch.softmax(div, dim=-1)
````
- **EN**: Introduces function `_sfdp_pattern_8`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `q`, `k`, `v`, `attn_mask`, `dropout_p`, `scale`, and `...+3`.
- **CN**: 这里定义了函数`_sfdp_pattern_8`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `q`、`k`、`v`、`attn_mask`、`dropout_p`、`scale`、`另有3项` 等值。

### Lines 197-224 / 第 197-224 行
````python
    attn_weight = attn_weight.to(torch.float16)
    v = v.to(attn_weight.dtype)
    return attn_weight @ v


def _sfdp_replacement_8(query, key, value, inv_scale):
    counters["inductor"]["fuse_attention"] += 1
    q = query.permute(0, 2, 1, 3)
    k = key.permute(0, 2, 1, 3)
    v = value.permute(0, 2, 1, 3)
    return _scaled_dot_product_attention(
        q,
        k,
        v,
        attn_mask=None,  # attn_mask,
        dropout_p=0.0,
        scale=1.0 / inv_scale,
        is_causal=False,
    )


def _sfdp_pattern_9(query, key, value, inv_scale, dropout_p):
    q = query.permute(0, 2, 1, 3)
    k = key.permute(0, 2, 1, 3)
    v = value.permute(0, 2, 1, 3)
    q = q / inv_scale
    div = q @ k.transpose(-2, -1)
    div = div.to(torch.float32)
````
- **EN**: Introduces function `_sfdp_replacement_8`, function `_sfdp_pattern_9`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `attn_weight`, `v`, `q`, `k`, `attn_mask`, `dropout_p`, and `...+3`.
- **CN**: 这里定义了函数`_sfdp_replacement_8`、函数`_sfdp_pattern_9`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `attn_weight`、`v`、`q`、`k`、`attn_mask`、`dropout_p`、`另有3项` 等值。

### Lines 225-252 / 第 225-252 行
````python
    attn_weight = torch.softmax(div, dim=-1)
    attn_weight = torch.dropout(attn_weight, dropout_p, True)
    attn_weight = attn_weight.to(torch.float16)
    v = v.to(attn_weight.dtype)
    return attn_weight @ v


def _sfdp_replacement_9(query, key, value, inv_scale, dropout_p):
    counters["inductor"]["fuse_attention"] += 1
    q = query.permute(0, 2, 1, 3)
    k = key.permute(0, 2, 1, 3)
    v = value.permute(0, 2, 1, 3)
    return _scaled_dot_product_attention(
        q,
        k,
        v,
        attn_mask=None,  # attn_mask,
        dropout_p=dropout_p,
        scale=1.0 / inv_scale,
        is_causal=False,
    )


def _sfdp_pattern_10(query, key, value, inv_scale):
    # no dropout version of 9
    q = query.permute(0, 2, 1, 3)
    k = key.permute(0, 2, 1, 3)
    v = value.permute(0, 2, 1, 3)
````
- **EN**: Introduces function `_sfdp_replacement_9`, function `_sfdp_pattern_10`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `attn_weight`, `v`, `q`, `k`, `attn_mask`, `dropout_p`, and `...+2`.
- **CN**: 这里定义了函数`_sfdp_replacement_9`、函数`_sfdp_pattern_10`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `attn_weight`、`v`、`q`、`k`、`attn_mask`、`dropout_p`、`另有2项` 等值。

### Lines 253-280 / 第 253-280 行
````python
    q = q / inv_scale
    div = q @ k.transpose(-2, -1)
    div = div.to(torch.float32)
    attn_weight = torch.softmax(div, dim=-1)
    attn_weight = attn_weight.to(torch.float16)
    v = v.to(attn_weight.dtype)
    return attn_weight @ v


def _sfdp_replacement_10(query, key, value, inv_scale):
    counters["inductor"]["fuse_attention"] += 1
    q = query.permute(0, 2, 1, 3)
    k = key.permute(0, 2, 1, 3)
    v = value.permute(0, 2, 1, 3)
    return _scaled_dot_product_attention(
        q,
        k,
        v,
        attn_mask=None,  # attn_mask,
        dropout_p=0.0,
        scale=1.0 / inv_scale,
        is_causal=False,
    )


def _sfdp_pattern_11(query, key, value, inv_scale):
    # Mainly for huggingface models
    q = query.permute(0, 2, 1, 3)
````
- **EN**: Introduces function `_sfdp_replacement_10`, function `_sfdp_pattern_11`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `q`, `div`, `attn_weight`, `v`, `k`, `attn_mask`, and `...+3`.
- **CN**: 这里定义了函数`_sfdp_replacement_10`、函数`_sfdp_pattern_11`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `q`、`div`、`attn_weight`、`v`、`k`、`attn_mask`、`另有3项` 等值。

### Lines 281-308 / 第 281-308 行
````python
    k = key.permute(0, 2, 1, 3)
    v = value.permute(0, 2, 1, 3)
    return torch.matmul(q, k.transpose(-2, -1)).div(inv_scale).softmax(dim=-1).matmul(v)


def _sfdp_replacement_11(query, key, value, inv_scale):
    counters["inductor"]["fuse_attention"] += 1
    return _scaled_dot_product_attention(
        query.transpose(1, 2),
        key.transpose(1, 2),
        value.transpose(1, 2),
        attn_mask=None,
        dropout_p=0.0,
        is_causal=False,
        scale=1.0 / inv_scale,
    )


def _sfdp_pattern_12(query, key, value, inv_scale_factor, dropout_p):
    q = query.permute(0, 2, 1, 3)
    k = key.permute(0, 2, 1, 3)
    v = value.permute(0, 2, 1, 3)
    return torch.nn.functional.dropout(
        torch.matmul(q, k.transpose(-2, -1)).div(inv_scale_factor).softmax(dim=-1),
        p=dropout_p,
    ).matmul(v)


````
- **EN**: Introduces function `_sfdp_replacement_11`, function `_sfdp_pattern_12`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `k`, `v`, `attn_mask`, `dropout_p`, `is_causal`, `scale`, and `...+2`.
- **CN**: 这里定义了函数`_sfdp_replacement_11`、函数`_sfdp_pattern_12`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `k`、`v`、`attn_mask`、`dropout_p`、`is_causal`、`scale`、`另有2项` 等值。

### Lines 309-336 / 第 309-336 行
````python
def _sfdp_replacement_12(query, key, value, inv_scale_factor, dropout_p):
    counters["inductor"]["fuse_attention"] += 1
    return _scaled_dot_product_attention(
        query.transpose(1, 2),
        key.transpose(1, 2),
        value.transpose(1, 2),
        attn_mask=None,
        dropout_p=dropout_p,
        is_causal=False,
        scale=1.0 / inv_scale_factor,
    )


def _sfdp_pattern_13(query, key, value, dropout_p):
    attn_weight = torch.bmm(query, key.transpose(1, 2)).softmax(dim=-1)
    attn_weight = torch.nn.functional.dropout(attn_weight, p=dropout_p)
    return torch.bmm(attn_weight, value)


def _sfdp_replacement_13(query, key, value, dropout_p):
    counters["inductor"]["fuse_attention"] += 1
    return _scaled_dot_product_attention(
        query.unsqueeze(0),
        key.unsqueeze(0),
        value.unsqueeze(0),
        dropout_p=dropout_p,
        scale=1.0,
    ).squeeze(0)
````
- **EN**: Introduces function `_sfdp_replacement_12`, function `_sfdp_pattern_13`, function `_sfdp_replacement_13`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `attn_mask`, `dropout_p`, `is_causal`, `scale`, and `attn_weight`.
- **CN**: 这里定义了函数`_sfdp_replacement_12`、函数`_sfdp_pattern_13`、函数`_sfdp_replacement_13`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `attn_mask`、`dropout_p`、`is_causal`、`scale`、`attn_weight` 等值。

### Lines 337-364 / 第 337-364 行
````python


def _sfdp_pattern_14(query, key, value, attn_mask, inv_scale):
    # for BertLarge
    # Permutations are needed to create clones in graph.
    q = query.permute([0, 2, 1, 3])
    k = key.permute([0, 2, 1, 3])
    v = value.permute([0, 2, 1, 3])
    return (
        (torch.matmul(q, k.transpose(-2, -1)).div(inv_scale) + attn_mask)
        .softmax(dim=-1)
        .matmul(v)
    )


def _sfdp_replacement_14(query, key, value, attn_mask, inv_scale):
    counters["inductor"]["fuse_attention"] += 1
    return _scaled_dot_product_attention(
        query.transpose(1, 2),
        key.transpose(1, 2),
        value.transpose(1, 2),
        attn_mask=attn_mask.to(dtype=query.dtype),
        dropout_p=0.0,
        is_causal=False,
        scale=1.0 / inv_scale,
    )


````
- **EN**: Introduces function `_sfdp_pattern_14`, function `_sfdp_replacement_14`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `q`, `k`, `v`, `attn_mask`, `dropout_p`, `is_causal`, and `...+1`.
- **CN**: 这里定义了函数`_sfdp_pattern_14`、函数`_sfdp_replacement_14`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `q`、`k`、`v`、`attn_mask`、`dropout_p`、`is_causal`、`另有1项` 等值。

### Lines 365-392 / 第 365-392 行
````python
def _sfdp_pattern_15(query, key, value, attn_mask, inv_scale):
    # for DistilBert
    # Permutations are needed to create clones in graph.
    # Ref: https://github.com/pytorch/pytorch/issues/119911
    q = query.permute([0, 2, 1, 3])
    k = key.permute([0, 2, 1, 3])
    v = value.permute([0, 2, 1, 3])
    bs = q.size(0)
    k_len = k.size(-2)
    scores = q @ k.transpose(-2, -1)
    scores = scores.div(inv_scale)
    fill_value = torch.full((), -float("inf"), dtype=query.dtype, device=query.device)
    attn_mask = (attn_mask == 0).view((bs, 1, 1, k_len)).expand_as(scores)
    return torch.softmax(scores.masked_fill(attn_mask, fill_value), dim=-1) @ v


def _sfdp_replacement_15(query, key, value, attn_mask, inv_scale):
    counters["inductor"]["fuse_attention"] += 1
    bs = query.size(0)
    n_head = query.size(2)
    q_len = query.size(1)
    k_len = key.size(1)
    # do attn_mask->logical_not() in _scaled_dot_product_attention
    attn_mask = (
        (attn_mask == 1).view((bs, 1, 1, k_len)).expand((bs, n_head, q_len, k_len))
    )
    return _scaled_dot_product_attention(
        query.transpose(1, 2),
````
- **EN**: Introduces function `_sfdp_pattern_15`, function `_sfdp_replacement_15`. Preserves reference comments that point to design notes, specifications, or vendor datasheets. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了函数`_sfdp_pattern_15`、函数`_sfdp_replacement_15`。保留了指向设计说明、规范或厂商数据手册的注释引用。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 393-420 / 第 393-420 行
````python
        key.transpose(1, 2),
        value.transpose(1, 2),
        attn_mask=attn_mask.to(dtype=torch.bool),
        dropout_p=0.0,
        is_causal=False,
        scale=1.0 / inv_scale,
    )


def _sfdp_pattern_16(query, key, value, attn_mask, inv_scale, dropout_p):
    # for BertLarge with dropout
    q = query.permute([0, 2, 1, 3])
    k = key.permute([0, 2, 1, 3])
    v = value.permute([0, 2, 1, 3])
    return (
        torch.nn.functional.dropout(
            (torch.matmul(q, k.transpose(-2, -1)).div(inv_scale) + attn_mask).softmax(
                dim=-1
            ),
            dropout_p,
        )
        .to(dtype=query.dtype)
        .matmul(v)
    )


def _sfdp_replacement_16(query, key, value, attn_mask, inv_scale, dropout_p):
    counters["inductor"]["fuse_attention"] += 1
````
- **EN**: Introduces function `_sfdp_pattern_16`, function `_sfdp_replacement_16`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `attn_mask`, `dropout_p`, `is_causal`, `scale`, `q`, `k`, and `...+2`.
- **CN**: 这里定义了函数`_sfdp_pattern_16`、函数`_sfdp_replacement_16`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `attn_mask`、`dropout_p`、`is_causal`、`scale`、`q`、`k`、`另有2项` 等值。

### Lines 421-448 / 第 421-448 行
````python
    return _scaled_dot_product_attention(
        query.transpose(1, 2),
        key.transpose(1, 2),
        value.transpose(1, 2),
        attn_mask=attn_mask.to(dtype=query.dtype),
        dropout_p=dropout_p,
        is_causal=False,
        scale=1.0 / inv_scale,
    )


def _sfdp_pattern_17(query, key, value, attn_mask, inv_scale, dropout_p):
    # for DistilBert with dropout
    q = query.permute([0, 2, 1, 3])
    k = key.permute([0, 2, 1, 3])
    v = value.permute([0, 2, 1, 3])
    bs = q.size(0)
    k_len = k.size(-2)
    scores = q @ k.transpose(-2, -1)
    scores = scores.div(inv_scale)
    fill_value = torch.full((), -float("inf"), dtype=query.dtype, device=query.device)
    attn_mask = (attn_mask == 0).view((bs, 1, 1, k_len)).expand_as(scores)
    return (
        torch.nn.functional.dropout(
            torch.softmax(scores.masked_fill(attn_mask, fill_value), dim=-1), dropout_p
        )
        @ v
    )
````
- **EN**: Introduces function `_sfdp_pattern_17`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了函数`_sfdp_pattern_17`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 449-476 / 第 449-476 行
````python


def _sfdp_replacement_17(query, key, value, attn_mask, inv_scale, dropout_p):
    counters["inductor"]["fuse_attention"] += 1
    bs = query.size(0)
    n_head = query.size(2)
    q_len = query.size(1)
    k_len = key.size(1)
    # do attn_mask->logical_not() in _scaled_dot_product_attention
    attn_mask = (
        (attn_mask == 1).view((bs, 1, 1, k_len)).expand((bs, n_head, q_len, k_len))
    )
    return _scaled_dot_product_attention(
        query.transpose(1, 2),
        key.transpose(1, 2),
        value.transpose(1, 2),
        attn_mask=attn_mask.to(dtype=torch.bool),
        dropout_p=dropout_p,
        is_causal=False,
        scale=1.0 / inv_scale,
    )


def _sfdp_pattern_18(query, key, value, causal_mask, inv_scale, dropout_p):
    # for hf_GPT2 with dropout (introduces clone node) for inference
    # it also returns permuted key & value
    query = query.permute([0, 2, 1, 3])
    key = key.permute([0, 2, 1, 3])
````
- **EN**: Introduces function `_sfdp_replacement_17`, function `_sfdp_pattern_18`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `bs`, `n_head`, `q_len`, `k_len`, `attn_mask`, `dropout_p`, and `...+4`.
- **CN**: 这里定义了函数`_sfdp_replacement_17`、函数`_sfdp_pattern_18`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `bs`、`n_head`、`q_len`、`k_len`、`attn_mask`、`dropout_p`、`另有4项` 等值。

### Lines 477-504 / 第 477-504 行
````python
    value = value.permute([0, 2, 1, 3])
    attn_weights = torch.matmul(query, key.permute(0, 1, 3, 2))
    inv_scale = torch.full(
        [],
        inv_scale,
        dtype=attn_weights.dtype,
        device=attn_weights.device,
    )
    attn_weights = attn_weights.div(inv_scale)
    causal_mask_value = torch.full(
        (), torch.finfo(query.dtype).min, dtype=query.dtype, device=query.device
    )
    attn_weights = torch.where(causal_mask, attn_weights, causal_mask_value)
    return (
        (
            torch.nn.functional.dropout(attn_weights.softmax(dim=-1), dropout_p).matmul(
                value
            )
        ),
        key,
        value,
    )


def _sfdp_replacement_18(query, key, value, causal_mask, inv_scale, dropout_p):
    counters["inductor"]["fuse_attention"] += 1
    permuted_key = key.transpose(1, 2)
    permuted_value = value.transpose(1, 2)
````
- **EN**: Introduces function `_sfdp_replacement_18`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_sfdp_replacement_18`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 505-532 / 第 505-532 行
````python
    return (
        _scaled_dot_product_attention(
            query.transpose(1, 2),
            permuted_key,
            permuted_value,
            attn_mask=causal_mask,
            dropout_p=dropout_p,
            is_causal=False,
            scale=1.0 / inv_scale,
        ),
        permuted_key,
        permuted_value,
    )


def _sfdp_pattern_19(query, key, value, causal_mask, attn_mask, inv_scale, dropout_p):
    # for token-classification+gpt2 / text-generation+gpt2
    attn_weights = torch.matmul(query, key.permute(0, 1, 3, 2))
    inv_scale = torch.full(
        [],
        inv_scale,
        dtype=attn_weights.dtype,
        device=attn_weights.device,
    )
    attn_weights = attn_weights.div(inv_scale)
    causal_mask_value = torch.full(
        (), torch.finfo(query.dtype).min, dtype=query.dtype, device=query.device
    )
````
- **EN**: Introduces function `_sfdp_pattern_19`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_sfdp_pattern_19`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 533-560 / 第 533-560 行
````python
    attn_weights = torch.where(causal_mask, attn_weights, causal_mask_value)
    attn_weights = attn_weights + attn_mask
    attn_weights = attn_weights.softmax(dim=-1).type(value.dtype)
    return torch.nn.functional.dropout(attn_weights, dropout_p).matmul(value)


def _sfdp_replacement_19(
    query, key, value, causal_mask, attn_mask, inv_scale, dropout_p
):
    counters["inductor"]["fuse_attention"] += 1
    fill_value = torch.full((), -float("inf"), dtype=query.dtype, device=query.device)
    attn_mask = torch.where(causal_mask, attn_mask, fill_value)
    return _scaled_dot_product_attention(
        query,
        key,
        value,
        attn_mask=attn_mask,
        dropout_p=dropout_p,
        is_causal=False,
        scale=1.0 / inv_scale,
    )


def _sfdp_pattern_20(query, key, value, attn_mask, inv_scale, dropout_p):
    # for DistilBert with dropout transformers==4.44.2
    q = query.permute([0, 2, 1, 3])
    k = key.permute([0, 2, 1, 3])
    v = value.permute([0, 2, 1, 3])
````
- **EN**: Introduces function `_sfdp_replacement_19`, function `_sfdp_pattern_20`. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path.
- **CN**: 这里定义了函数`_sfdp_replacement_19`、函数`_sfdp_pattern_20`。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 561-588 / 第 561-588 行
````python
    bs = q.size(0)
    k_len = k.size(-2)
    q = q.div(inv_scale)
    scores = q @ k.transpose(-2, -1)
    fill_value = torch.full((), -float("inf"), dtype=query.dtype, device=query.device)
    attn_mask = (attn_mask == 0).view((bs, 1, 1, k_len)).expand_as(scores)
    return (
        torch.nn.functional.dropout(
            torch.softmax(scores.masked_fill(attn_mask, fill_value), dim=-1), dropout_p
        )
        @ v
    )


def _sfdp_replacement_20(query, key, value, attn_mask, inv_scale, dropout_p):
    counters["inductor"]["fuse_attention"] += 1
    bs = query.size(0)
    n_head = query.size(2)
    q_len = query.size(1)
    k_len = key.size(1)
    # do attn_mask->logical_not() in _scaled_dot_product_attention
    attn_mask = (
        (attn_mask == 1).view((bs, 1, 1, k_len)).expand((bs, n_head, q_len, k_len))
    )
    return _scaled_dot_product_attention(
        query.transpose(1, 2),
        key.transpose(1, 2),
        value.transpose(1, 2),
````
- **EN**: Introduces function `_sfdp_replacement_20`. Applies decorators to register behavior or alter how the following definition is constructed. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution.
- **CN**: 这里定义了函数`_sfdp_replacement_20`。使用装饰器来注册行为，或改变后续定义的构造方式。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。

### Lines 589-616 / 第 589-616 行
````python
        attn_mask=attn_mask.to(dtype=torch.bool),
        dropout_p=dropout_p,
        is_causal=False,
        scale=1.0 / inv_scale,
    )


def _sfdp_pattern_21(query, key, value, attn_mask):
    # for T5 with inplace add
    query = query.permute([0, 2, 1, 3])
    key = key.permute([0, 2, 1, 3])
    value = value.permute([0, 2, 1, 3])
    score = torch.matmul(query, key.permute(0, 1, 3, 2))
    masked_score = score + attn_mask
    score = masked_score.type_as(query)
    return score.float().softmax(dim=-1).type_as(query).matmul(value)


def _sfdp_replacement_21(query, key, value, attn_mask):
    counters["inductor"]["fuse_attention"] += 1
    query = query.permute(0, 2, 1, 3)
    key = key.permute(0, 2, 1, 3)
    value = value.permute(0, 2, 1, 3)
    return _scaled_dot_product_attention(
        query,
        key,
        value,
        attn_mask=attn_mask.to(dtype=query.dtype),
````
- **EN**: Introduces function `_sfdp_pattern_21`, function `_sfdp_replacement_21`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `attn_mask`, `dropout_p`, `is_causal`, `scale`, `query`, `key`, and `...+3`.
- **CN**: 这里定义了函数`_sfdp_pattern_21`、函数`_sfdp_replacement_21`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `attn_mask`、`dropout_p`、`is_causal`、`scale`、`query`、`key`、`另有3项` 等值。

### Lines 617-644 / 第 617-644 行
````python
        is_causal=False,
        scale=1.0,
    )


def _sfdp_pattern_22(query, key, value, attn_mask):
    # for T5 with inplace add and return key and value
    query = query.permute([0, 2, 1, 3])
    key = key.permute([0, 2, 1, 3])
    value = value.permute([0, 2, 1, 3])
    score = torch.matmul(query, key.permute(0, 1, 3, 2))
    masked_score = score + attn_mask
    score = masked_score.type_as(query)
    return score.float().softmax(dim=-1).type_as(query).matmul(value), key, value


def _sfdp_replacement_22(query, key, value, attn_mask):
    counters["inductor"]["fuse_attention"] += 1
    query = query.permute(0, 2, 1, 3)
    key = key.permute(0, 2, 1, 3)
    value = value.permute(0, 2, 1, 3)
    return (
        _scaled_dot_product_attention(
            query,
            key,
            value,
            attn_mask=attn_mask.to(dtype=query.dtype),
            is_causal=False,
````
- **EN**: Introduces function `_sfdp_pattern_22`, function `_sfdp_replacement_22`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `is_causal`, `scale`, `query`, `key`, `value`, `score`, and `...+2`.
- **CN**: 这里定义了函数`_sfdp_pattern_22`、函数`_sfdp_replacement_22`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `is_causal`、`scale`、`query`、`key`、`value`、`score`、`另有2项` 等值。

### Lines 645-672 / 第 645-672 行
````python
            scale=1.0,
        ),
        key,
        value,
    )


def _sfdp_pattern_23(query, key, value):
    # for T5 with inplace add and
    # return key and value and
    # attn_mask is generated by atem.full(..., 0)
    query = query.permute([0, 2, 1, 3])
    key = key.permute([0, 2, 1, 3])
    value = value.permute([0, 2, 1, 3])
    score = torch.matmul(query, key.permute(0, 1, 3, 2))
    fp32_score = score.float()
    score = fp32_score.type_as(query)
    return score.float().softmax(dim=-1).type_as(query).matmul(value), key, value


def _sfdp_replacement_23(query, key, value):
    counters["inductor"]["fuse_attention"] += 1
    query = query.permute(0, 2, 1, 3)
    key = key.permute(0, 2, 1, 3)
    value = value.permute(0, 2, 1, 3)
    return (
        _scaled_dot_product_attention(
            query,
````
- **EN**: Introduces function `_sfdp_pattern_23`, function `_sfdp_replacement_23`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `scale`, `query`, `key`, `value`, `score`, and `fp32_score`.
- **CN**: 这里定义了函数`_sfdp_pattern_23`、函数`_sfdp_replacement_23`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `scale`、`query`、`key`、`value`、`score`、`fp32_score` 等值。

### Lines 673-700 / 第 673-700 行
````python
            key,
            value,
            attn_mask=None,
            is_causal=False,
            scale=1.0,
        ),
        key,
        value,
    )


def _sfdp_pattern_24(query, key, value, attention_mask):
    """
    this pattern is for MBartForCausalLM/PLBartForCausalLM.
    attn_mask has a different dtype with QKV.
    there is no scale in sdpa.
    """
    bs = query.size(0)
    n_head = query.size(1)
    seq_len = query.size(2)
    head_size = query.size(3)
    q = query.view(bs * n_head, -1, head_size)
    k = key.reshape(bs * n_head, -1, head_size)
    v = value.reshape(bs * n_head, -1, head_size)
    attn_weights = torch.bmm(q, k.transpose(1, 2))
    attn_weights = attn_weights.view(bs, n_head, seq_len, -1) + attention_mask
    attn_weights = attn_weights.view(bs * n_head, seq_len, -1)
    attn_weights = torch.nn.functional.softmax(attn_weights, dim=-1)
````
- **EN**: Introduces function `_sfdp_pattern_24`. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `attn_mask`, `is_causal`, `scale`, `bs`, `n_head`, `seq_len`, and `...+5`.
- **CN**: 这里定义了函数`_sfdp_pattern_24`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `attn_mask`、`is_causal`、`scale`、`bs`、`n_head`、`seq_len`、`另有5项` 等值。

### Lines 701-728 / 第 701-728 行
````python
    if query.dtype == torch.half:
        attn_weights = attn_weights.to(torch.half)
    attn_output = torch.bmm(attn_weights, v)
    attn_output = attn_output.view(bs, n_head, seq_len, head_size)
    return attn_output


def _sfdp_replacement_24(query, key, value, attention_mask):
    counters["inductor"]["fuse_attention"] += 1
    return _scaled_dot_product_attention(
        query,
        key,
        value,
        attn_mask=attention_mask.to(dtype=query.dtype),
        is_causal=False,
        scale=1,
    )


def _sfdp_pattern_25(query, key, value, attn_mask, dropout_p):
    # for T5 with inplace add
    query = query.permute([0, 2, 1, 3])
    key = key.permute([0, 2, 1, 3])
    value = value.permute([0, 2, 1, 3])
    score = torch.matmul(query, key.permute(0, 1, 3, 2))
    masked_score = score + attn_mask
    return torch.nn.functional.dropout(
        masked_score.float().softmax(dim=-1).type_as(query), dropout_p
````
- **EN**: Introduces function `_sfdp_replacement_24`, function `_sfdp_pattern_25`. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_sfdp_replacement_24`、函数`_sfdp_pattern_25`。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 729-756 / 第 729-756 行
````python
    ).matmul(value)


def _sfdp_replacement_25(query, key, value, attn_mask, dropout_p):
    counters["inductor"]["fuse_attention"] += 1
    query = query.permute(0, 2, 1, 3)
    key = key.permute(0, 2, 1, 3)
    value = value.permute(0, 2, 1, 3)
    if attn_mask.device.type == "xpu":
        attn_mask = attn_mask.contiguous()
    return _scaled_dot_product_attention(
        query,
        key,
        value,
        attn_mask=attn_mask,
        dropout_p=dropout_p,
        is_causal=False,
        scale=1.0,
    )


def _sfdp_pattern_26(query, key, value, attn_mask, dropout_p):
    # for T5 with inplace add and return key and value
    query = query.permute([0, 2, 1, 3])
    key = key.permute([0, 2, 1, 3])
    value = value.permute([0, 2, 1, 3])
    score = torch.matmul(query, key.permute(0, 1, 3, 2))
    masked_score = score + attn_mask
````
- **EN**: Introduces function `_sfdp_replacement_25`, function `_sfdp_pattern_26`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_sfdp_replacement_25`、函数`_sfdp_pattern_26`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 757-784 / 第 757-784 行
````python
    return (
        torch.nn.functional.dropout(
            masked_score.float().softmax(dim=-1).type_as(query), dropout_p
        ).matmul(value),
        key,
        value,
    )


def _sfdp_replacement_26(query, key, value, attn_mask, dropout_p):
    counters["inductor"]["fuse_attention"] += 1
    query = query.permute(0, 2, 1, 3)
    key = key.permute(0, 2, 1, 3)
    value = value.permute(0, 2, 1, 3)
    if attn_mask.device.type == "xpu":
        attn_mask = attn_mask.contiguous()
    return (
        _scaled_dot_product_attention(
            query,
            key,
            value,
            attn_mask=attn_mask,
            dropout_p=dropout_p,
            is_causal=False,
            scale=1.0,
        ),
        key,
        value,
````
- **EN**: Introduces function `_sfdp_replacement_26`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_sfdp_replacement_26`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 785-812 / 第 785-812 行
````python
    )


def _sfdp_pattern_27(query, key, value, dropout_p):
    # for T5 with inplace add and
    # return key and value and
    # attn_mask is generated by atem.full(..., 0)
    query = query.permute([0, 2, 1, 3])
    key = key.permute([0, 2, 1, 3])
    value = value.permute([0, 2, 1, 3])
    score = torch.matmul(query, key.permute(0, 1, 3, 2))
    return (
        torch.nn.functional.dropout(
            score.float().softmax(dim=-1).type_as(query), dropout_p
        ).matmul(value),
        key,
        value,
    )


def _sfdp_replacement_27(query, key, value, dropout_p):
    counters["inductor"]["fuse_attention"] += 1
    query = query.permute(0, 2, 1, 3)
    key = key.permute(0, 2, 1, 3)
    value = value.permute(0, 2, 1, 3)
    return (
        _scaled_dot_product_attention(
            query,
````
- **EN**: Introduces function `_sfdp_pattern_27`, function `_sfdp_replacement_27`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `query`, `key`, `value`, and `score`.
- **CN**: 这里定义了函数`_sfdp_pattern_27`、函数`_sfdp_replacement_27`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `query`、`key`、`value`、`score` 等值。

### Lines 813-840 / 第 813-840 行
````python
            key,
            value,
            attn_mask=None,
            dropout_p=dropout_p,
            is_causal=False,
            scale=1.0,
        ),
        key,
        value,
    )


def _sfdp_pattern_28(query, key, value, scale_factor, dropout_p):
    # Visformer pattern
    # same as pattern 4 but non-contiguous q/k/v
    return _sfdp_pattern_4(query, key, value, scale_factor, dropout_p)


def _sfdp_replacement_28(query, key, value, scale_factor, dropout_p):
    counters["inductor"]["fuse_attention"] += 1
    return _scaled_dot_product_attention(
        query.contiguous(),
        key.contiguous(),
        value.contiguous(),
        attn_mask=None,
        dropout_p=dropout_p,
        is_causal=False,
        scale=scale_factor,
````
- **EN**: Introduces function `_sfdp_pattern_28`, function `_sfdp_replacement_28`. Includes returns or checks that define the contract of this code path. Initializes or updates values such as `attn_mask`, `dropout_p`, `is_causal`, and `scale`.
- **CN**: 这里定义了函数`_sfdp_pattern_28`、函数`_sfdp_replacement_28`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。初始化或更新了 `attn_mask`、`dropout_p`、`is_causal`、`scale` 等值。

### Lines 841-868 / 第 841-868 行
````python
    )


@functools.lru_cache(None)
def _warn_tf32_disabled() -> None:
    if (
        torch.cuda.is_available()
        and torch.backends.cuda.matmul.fp32_precision != "tf32"
        and torch.cuda.get_device_capability() >= (8, 0)
    ):
        warnings.warn(
            "TensorFloat32 tensor cores for float32 matrix multiplication available but not enabled. "
            "Skipping pattern matching to fused flash-attention. "
            "Consider setting `torch.set_float32_matmul_precision('high')` for better performance."
        )


def _sfdp_params_check(match):
    assert all(k in match.kwargs for k in ("query", "key", "value"))
    query = match.kwargs["query"].meta["val"]
    key = match.kwargs["key"].meta["val"]
    value = match.kwargs["value"].meta["val"]
    if not (query.dtype == key.dtype == value.dtype) or not (
        query.device == key.device == value.device
    ):
        return False
    # fused kernels use tf32
    if (
````
- **EN**: Introduces function `_warn_tf32_disabled`, function `_sfdp_params_check`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`_warn_tf32_disabled`、函数`_sfdp_params_check`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理与设备相关的行为、能力信息或派发决策。

### Lines 869-896 / 第 869-896 行
````python
        query.device.type == "cuda"
        and query.dtype == torch.float32
        and torch.backends.cuda.matmul.fp32_precision != "tf32"
    ):
        _warn_tf32_disabled()
        return False

    add_mask_node = filter_nodes(match.nodes, aten.add.Tensor)
    # Has attn_mask add.
    if len(add_mask_node) > 0:
        attn_mask_node = add_mask_node[0].args[1]
        # attn_mask_node may be a float/int number.
        if not hasattr(attn_mask_node, "meta"):
            return False
        attn_mask = attn_mask_node.meta["val"]  # type: ignore[union-attr]
        # Make sure attn_mask.dtype == query.dtype or attn_mask.dtype == torch.bool
        # attn_mask.dtype == torch.float for models like albert.
        if (
            not isinstance(attn_mask, torch.Tensor)
            or not (
                attn_mask.dtype == query.dtype
                or attn_mask.dtype == torch.bool
                or attn_mask.dtype == torch.float
            )
            or query.device != attn_mask.device
            # When we tensorify floats we end up turning floats
            # into 0d scalar tensors. It doesn't make any sense
            # to have a 0d scalar tensor attention mask so
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 897-924 / 第 897-924 行
````python
            # conveniently we can insert this check to get
            # tests that erroneously passing in a float
            # attention mask to fail as expected.
            or attn_mask.dim() == 0
        ):
            return False
    return True


def _sfdp_extra_check(scale_factor_op=None, disable_cuda=False):
    def fn(match):
        if (
            disable_cuda
            and "query" in match.kwargs
            and "cuda" in str(match.kwargs["query"].meta["val"].device)
        ):
            return False
        if scale_factor_op is not None:
            scale_factor_node = filter_nodes(match.nodes, scale_factor_op)[0]
            # Note: args[1] of the scale_factor_node is always the scale_factor for the current patterns.
            scale_factor = scale_factor_node.args[1]
            # make sure the scale_factor a float/int. SymInt?
            if not isinstance(scale_factor, (float, int)):
                return False
        return _sfdp_params_check(match)

    return fn

````
- **EN**: Introduces function `_sfdp_extra_check`, function `fn`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`_sfdp_extra_check`、函数`fn`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 925-952 / 第 925-952 行
````python

def partialize_and_update_signature(func, **kwargs):
    """
    Equivalent to functools.partial but also updates the signature on returned function
    """
    original_sig = inspect.signature(func)
    parameters = original_sig.parameters

    new_parameters = {
        key: value for key, value in parameters.items() if key not in kwargs
    }
    new_sig = inspect.Signature(parameters=list(new_parameters.values()))

    partial_func = functools.partial(func, **kwargs)

    def wrapper(*args, **kwargs):
        return partial_func(*args, **kwargs)

    wrapper.__signature__ = new_sig  # type: ignore[attr-defined]
    wrapper.__name__ = func.__name__

    return wrapper


def _get_sfdp_patterns(input_device: torch.device | None = None):
    from .joint_graph import patterns

    if input_device:
````
- **EN**: Imports dependencies such as `.joint_graph` for the logic in this range. Introduces function `partialize_and_update_signature`, function `wrapper`, function `_get_sfdp_patterns`. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path.
- **CN**: 这里导入了 `.joint_graph` 等依赖，为后续逻辑提供基础能力。这里定义了函数`partialize_and_update_signature`、函数`wrapper`、函数`_get_sfdp_patterns`。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。

### Lines 953-980 / 第 953-980 行
````python
        device = str(input_device)
    else:
        if torch.cuda.is_available():
            # workaround https://github.com/pytorch/pytorch/issues/97894
            device = "cuda"
        elif torch.xpu.is_available():
            device = "xpu"
        else:
            device = "cpu"

    # sizes/values don't actually matter for initial trace
    # once we get a possible match we re-trace with the actual values and verify the match still holds
    g_inp = functools.partial(
        torch.empty, (2, 4, 8, 16), device=device, requires_grad=True
    )
    # non-contiguous input to cover more patterns.
    gn_inp = functools.partial(
        torch.empty_strided,
        (2, 6, 16, 8),
        (2304, 128, 1, 16),
        device=device,
        requires_grad=True,
    )
    # attn_mask
    b_inp = functools.partial(torch.empty, (1, 1, 8, 8), device=device)
    m_inp = functools.partial(torch.empty, (2, 1, 1, 4), device=device)
    # need 2d attn_mask to generate patterns with view op
    m_inp_2d = functools.partial(torch.empty, (2, 4), device=device)
````
- **EN**: Preserves reference comments that point to design notes, specifications, or vendor datasheets. Handles device-specific behavior, capabilities, or dispatch decisions. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `device`, `else`, `g_inp`, `gn_inp`, `requires_grad`, `b_inp`, and `...+2`.
- **CN**: 保留了指向设计说明、规范或厂商数据手册的注释引用。处理与设备相关的行为、能力信息或派发决策。包含分支、循环或上下文管理等控制流。初始化或更新了 `device`、`else`、`g_inp`、`gn_inp`、`requires_grad`、`b_inp`、`另有2项` 等值。

### Lines 981-1008 / 第 981-1008 行
````python
    # inv_scale
    c_inp = functools.partial(torch.tensor, 2.0, device=device)
    # workaround https://github.com/pytorch/pytorch/issues/97894
    # 0.113377 is a "magic" value that lets us recover the lost input arg relationship
    d = {"dropout_p": 0.113377}
    s = {"inv_scale": 0.66666}
    sd = {"inv_scale": 0.66666, "dropout_p": 0.113377}

    # we could also generate all these patterns in 3d.. TODO
    g_3d_inp = functools.partial(
        torch.empty, (1024, 128, 128), device=device, requires_grad=True
    )

    # reshape in matmul decomposition generates a clone when batch_size>1 due to the memory layout change.
    # however when batch_size=1, reshape does not change the memory layout, so clone would not be generated.
    # here we need to trace with input of batch_size=1 to generate a pattern graph without clone.
    g_bs1_inp = functools.partial(
        torch.empty, (1, 4, 8, 16), device=device, requires_grad=True
    )
    m_bs1_inp = functools.partial(torch.empty, (1, 1, 1, 4), device=device)

    # softmax will generate a dtype conversion on inputs if they are in half,
    # but will not in float, so we generate a pattern for both
    for dtype in [torch.float, torch.half]:
        g = functools.partial(g_inp, dtype=dtype)
        gn = functools.partial(gn_inp, dtype=dtype)
        b = functools.partial(b_inp, dtype=dtype)
        b_float = functools.partial(b_inp, dtype=torch.float)
````
- **EN**: Preserves reference comments that point to design notes, specifications, or vendor datasheets. Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 保留了指向设计说明、规范或厂商数据手册的注释引用。处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。

### Lines 1009-1036 / 第 1009-1036 行
````python
        b_bool = functools.partial(b_inp, dtype=torch.bool)
        m = functools.partial(m_inp, dtype=dtype)
        m_float = functools.partial(m_inp, dtype=torch.float)
        m_bool = functools.partial(m_inp, dtype=torch.bool)
        m_2d = functools.partial(m_inp_2d, dtype=dtype)
        c = functools.partial(c_inp, dtype=dtype)
        g_3d = functools.partial(g_3d_inp, dtype=dtype)
        g_bs1 = functools.partial(g_bs1_inp, dtype=dtype)
        m_bs1 = functools.partial(m_bs1_inp, dtype=dtype)
        m_bs1_float = functools.partial(m_bs1_inp, dtype=torch.float)
        m_bs1_bool = functools.partial(m_bs1_inp, dtype=torch.bool)

        candidates = [
            (
                _sfdp_pattern_1,
                _sfdp_replacement_1,
                [g(), g(), g(), c()],
                {},
                _sfdp_extra_check(aten.div.Tensor),
            ),
            (
                _sfdp_pattern_2,
                _sfdp_replacement_2,
                [g(), g(), g(), c()],
                {},
                _sfdp_extra_check(aten.mul.Tensor),
            ),
            (
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `b_bool`, `m`, `m_float`, `m_bool`, `m_2d`, `c`, and `...+6`. This range continues the implementation of function `_get_sfdp_patterns`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `b_bool`、`m`、`m_float`、`m_bool`、`m_2d`、`c`、`另有6项` 等值。这一段延续了函数`_get_sfdp_patterns` 的具体实现。

### Lines 1037-1064 / 第 1037-1064 行
````python
                _sfdp_pattern_3,
                _sfdp_replacement_3,
                [g(), g(), g(), c()],
                d,
                _sfdp_extra_check(aten.div.Tensor),
            ),
            (
                _sfdp_pattern_4,
                _sfdp_replacement_4,
                [g(), g(), g(), c()],
                d,
                _sfdp_extra_check(aten.mul.Tensor),
            ),
            (
                _sfdp_pattern_5,
                _sfdp_replacement_5,
                [g(), g(), g(), b()],
                s,
                _sfdp_params_check,
            ),
            (
                _sfdp_pattern_6,
                _sfdp_replacement_6,
                [g(), g(), g(), b()],
                sd,
                _sfdp_params_check,
            ),
            (
````
- **EN**: This range continues the implementation of function `_get_sfdp_patterns`.
- **CN**: 这一段延续了函数`_get_sfdp_patterns` 的具体实现。

### Lines 1065-1092 / 第 1065-1092 行
````python
                _sfdp_pattern_7,
                _sfdp_replacement_7,
                [g(), g(), g()],
                sd,
                _sfdp_params_check,
            ),
            (
                _sfdp_pattern_8,
                _sfdp_replacement_8,
                [g(), g(), g()],
                s,
                _sfdp_params_check,
            ),
            (
                _sfdp_pattern_9,
                _sfdp_replacement_9,
                [g(), g(), g()],
                sd,
                _sfdp_params_check,
            ),
            (
                _sfdp_pattern_10,
                _sfdp_replacement_10,
                [g(), g(), g()],
                s,
                _sfdp_params_check,
            ),
            (
````
- **EN**: This range continues the implementation of function `_get_sfdp_patterns`.
- **CN**: 这一段延续了函数`_get_sfdp_patterns` 的具体实现。

### Lines 1093-1120 / 第 1093-1120 行
````python
                _sfdp_pattern_11,
                _sfdp_replacement_11,
                [g(), g(), g(), c()],
                {},
                _sfdp_extra_check(aten.div.Tensor),
            ),
            (
                _sfdp_pattern_12,
                _sfdp_replacement_12,
                [g(), g(), g(), c()],
                d,
                _sfdp_extra_check(aten.div.Tensor),
            ),
            (
                _sfdp_pattern_13,
                _sfdp_replacement_13,
                [g_3d(), g_3d(), g_3d()],
                d,
                _sfdp_params_check,
            ),
            (
                _sfdp_pattern_14,
                _sfdp_replacement_14,
                [g(), g(), g(), m(), c()],
                {},
                _sfdp_extra_check(aten.div.Tensor),
            ),
            (
````
- **EN**: This range continues the implementation of function `_get_sfdp_patterns`.
- **CN**: 这一段延续了函数`_get_sfdp_patterns` 的具体实现。

### Lines 1121-1148 / 第 1121-1148 行
````python
                _sfdp_pattern_15,
                _sfdp_replacement_15,
                [g(), g(), g(), m_2d(), c()],
                {},
                _sfdp_extra_check(aten.div.Tensor),
            ),
            # disable_cuda only for NVIDIA CUDA (not ROCm) due to Bert accuracy issue
            (
                _sfdp_pattern_16,
                _sfdp_replacement_16,
                [g(), g(), g(), m(), c()],
                d,
                _sfdp_extra_check(
                    aten.div.Tensor, disable_cuda=torch.version.hip is None
                ),
            ),
            (
                _sfdp_pattern_16,
                _sfdp_replacement_16,
                [g_bs1(), g_bs1(), g_bs1(), m_bs1(), c()],
                d,
                _sfdp_extra_check(
                    aten.div.Tensor, disable_cuda=torch.version.hip is None
                ),
            ),
            (
                _sfdp_pattern_17,
                _sfdp_replacement_17,
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. This range continues the implementation of function `_get_sfdp_patterns`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。这一段延续了函数`_get_sfdp_patterns` 的具体实现。

### Lines 1149-1176 / 第 1149-1176 行
````python
                [g(), g(), g(), m_2d(), c()],
                d,
                _sfdp_extra_check(aten.div.Tensor),
            ),
            (
                _sfdp_pattern_18,
                _sfdp_replacement_18,
                [g(), g(), g(), m_bool()],
                sd,
                _sfdp_params_check,
            ),
            (
                _sfdp_pattern_18,
                _sfdp_replacement_18,
                [g_bs1(), g_bs1(), g_bs1(), m_bs1_bool()],
                sd,
                _sfdp_params_check,
            ),
            (
                _sfdp_pattern_19,
                _sfdp_replacement_19,
                [g(), g(), g(), b_bool(), b_float()],
                sd,
                _sfdp_params_check,
            ),
            (
                _sfdp_pattern_20,
                _sfdp_replacement_20,
````
- **EN**: This range continues the implementation of function `_get_sfdp_patterns`.
- **CN**: 这一段延续了函数`_get_sfdp_patterns` 的具体实现。

### Lines 1177-1204 / 第 1177-1204 行
````python
                [g(), g(), g(), m_2d()],
                sd,
                _sfdp_params_check,
            ),
            (
                _sfdp_pattern_21,
                _sfdp_replacement_21,
                [g(), g(), g(), m_float()],
                {},
                _sfdp_params_check,
            ),
            (
                _sfdp_pattern_21,
                _sfdp_replacement_21,
                [g_bs1(), g_bs1(), g_bs1(), m_bs1_float()],
                {},
                _sfdp_params_check,
            ),
            (
                _sfdp_pattern_22,
                _sfdp_replacement_22,
                [g(), g(), g(), m_float()],
                {},
                _sfdp_params_check,
            ),
            (
                _sfdp_pattern_22,
                _sfdp_replacement_22,
````
- **EN**: This range continues the implementation of function `_get_sfdp_patterns`.
- **CN**: 这一段延续了函数`_get_sfdp_patterns` 的具体实现。

### Lines 1205-1232 / 第 1205-1232 行
````python
                [g_bs1(), g_bs1(), g_bs1(), m_bs1_float()],
                {},
                _sfdp_params_check,
            ),
            (
                _sfdp_pattern_23,
                _sfdp_replacement_23,
                [g(), g(), g()],
                {},
                _sfdp_params_check,
            ),
            (
                _sfdp_pattern_23,
                _sfdp_replacement_23,
                [g_bs1(), g_bs1(), g_bs1()],
                {},
                _sfdp_params_check,
            ),
            (
                _sfdp_pattern_24,
                _sfdp_replacement_24,
                [g(), g(), g(), b_float()],
                {},
                _sfdp_extra_check,
            ),
            (
                _sfdp_pattern_25,
                _sfdp_replacement_25,
````
- **EN**: This range continues the implementation of function `_get_sfdp_patterns`.
- **CN**: 这一段延续了函数`_get_sfdp_patterns` 的具体实现。

### Lines 1233-1260 / 第 1233-1260 行
````python
                [g(), g(), g(), m()],
                d,
                _sfdp_extra_check(disable_cuda=True),
            ),
            (
                _sfdp_pattern_25,
                _sfdp_replacement_25,
                [g_bs1(), g_bs1(), g_bs1(), m_bs1()],
                d,
                _sfdp_extra_check(disable_cuda=True),
            ),
            (
                _sfdp_pattern_26,
                _sfdp_replacement_26,
                [g(), g(), g(), m()],
                d,
                _sfdp_extra_check(disable_cuda=True),
            ),
            (
                _sfdp_pattern_26,
                _sfdp_replacement_26,
                [g_bs1(), g_bs1(), g_bs1(), m_bs1()],
                d,
                _sfdp_extra_check(disable_cuda=True),
            ),
            (
                _sfdp_pattern_27,
                _sfdp_replacement_27,
````
- **EN**: This range continues the implementation of function `_get_sfdp_patterns`.
- **CN**: 这一段延续了函数`_get_sfdp_patterns` 的具体实现。

### Lines 1261-1288 / 第 1261-1288 行
````python
                [g(), g(), g()],
                d,
                _sfdp_extra_check(disable_cuda=True),
            ),
            (
                _sfdp_pattern_27,
                _sfdp_replacement_27,
                [g_bs1(), g_bs1(), g_bs1()],
                d,
                _sfdp_extra_check(disable_cuda=True),
            ),
            (
                _sfdp_pattern_28,
                _sfdp_replacement_28,
                [gn(), gn(), gn(), c()],
                d,
                _sfdp_extra_check(aten.mul.Tensor),
            ),
        ]
        mask_fp32_patterns = ["pattern_16"]
        if dtype == torch.half:
            # Add inputs of bf16 q/k/v and fp32 mask, for models like albert.
            candidates.append(
                (
                    _sfdp_pattern_16,
                    _sfdp_replacement_16,
                    [g(), g(), g(), m_float(), c()],
                    d,
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `mask_fp32_patterns`. This range continues the implementation of function `_get_sfdp_patterns`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含分支、循环或上下文管理等控制流。初始化或更新了 `mask_fp32_patterns` 等值。这一段延续了函数`_get_sfdp_patterns` 的具体实现。

### Lines 1289-1316 / 第 1289-1316 行
````python
                    # disable_cuda only for NVIDIA CUDA (not ROCm) due to Bert accuracy issue
                    _sfdp_extra_check(
                        aten.div.Tensor, disable_cuda=torch.version.hip is None
                    ),
                )
            )
            candidates.append(
                (
                    _sfdp_pattern_16,
                    _sfdp_replacement_16,
                    [g_bs1(), g_bs1(), g_bs1(), m_bs1_float(), c()],
                    d,
                    # disable_cuda only for NVIDIA CUDA (not ROCm) due to Bert accuracy issue
                    _sfdp_extra_check(
                        aten.div.Tensor, disable_cuda=torch.version.hip is None
                    ),
                )
            )

        for pattern, replacement, args, workaround, extra_check in candidates:
            # XXX: when adding a new pattern, re-run `gen_attention_patterns` so the pattern
            # gets serialized to a python file and does not require tracing at runtime.
            assert isinstance(workaround, dict)
            name = pattern.__name__

            if dtype != torch.float:
                name += "_half"
                if (
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 1317-1344 / 第 1317-1344 行
````python
                    any(p in name for p in mask_fp32_patterns)
                    and args[3].dtype == torch.float32
                ):
                    name += "_mask_fp32"
            if args[0].size(0) == 1:
                name += "_bs1"

            training_name = name + "_training"
            yield (
                training_name,
                {
                    "search_fn": pattern,
                    "replace_fn": replacement,
                    "example_inputs": args,
                    "trace_fn": joint_fwd_bwd,
                    "pass_dicts": patterns,
                    "extra_check": extra_check,
                    "scalar_workaround": workaround,
                    "skip_duplicates": True,
                },
            )
            inference_workaround = {}
            if workaround:
                assert len(workaround) <= 2
                if "inv_scale" in workaround:
                    inference_workaround["inv_scale"] = workaround["inv_scale"]
                if "dropout_p" in workaround:
                    # functools.partial insufficient because we look at signature downstream
````
- **EN**: Tracks dtype-specific behavior or constants that affect numeric execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `training_name`, and `inference_workaround`.
- **CN**: 追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `training_name`、`inference_workaround` 等值。

### Lines 1345-1371 / 第 1345-1371 行
````python
                    pattern = partialize_and_update_signature(pattern, dropout_p=0.0)
                    replacement = partialize_and_update_signature(
                        replacement, dropout_p=0.0
                    )

            inference_name = name + "_inference"
            yield (
                inference_name,
                {
                    "search_fn": pattern,
                    "replace_fn": replacement,
                    "example_inputs": args,
                    "trace_fn": fwd_only,
                    "pass_dicts": patterns,
                    "extra_check": extra_check,
                    "scalar_workaround": inference_workaround,
                    # with dropout turned into clone, we end up with a number of
                    # semantically identical graphs
                    "skip_duplicates": True,
                },
            )


@functools.cache
def _sfdp_init(input_device: torch.device | None = None):
    for key, register_replacement_kwargs in _get_sfdp_patterns(input_device):
        gen_register_replacement(key, **register_replacement_kwargs)
````
- **EN**: Introduces function `_sfdp_init`. Applies decorators to register behavior or alter how the following definition is constructed. Manipulates cached state or cache paths to avoid rebuilding the same work repeatedly. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`_sfdp_init`。使用装饰器来注册行为，或改变后续定义的构造方式。这一段处理缓存状态或缓存路径，以避免重复构建相同工作。处理与设备相关的行为、能力信息或派发决策。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Compilation/runtime caching  
  **CN**: 编译或运行期缓存
- **EN**: Graph-level transformations  
  **CN**: 图级变换
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验
- **EN**: Primary functions: `_sfdp_pattern_1`, `_sfdp_replacement_1`, `_sfdp_pattern_2`, `_sfdp_replacement_2`, `_sfdp_pattern_3`, `_sfdp_replacement_3`, and `...+56`  
  **CN**: 主要函数：`_sfdp_pattern_1`、`_sfdp_replacement_1`、`_sfdp_pattern_2`、`_sfdp_replacement_2`、`_sfdp_pattern_3`、`_sfdp_replacement_3`、`另有56项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `functools`, `inspect`, `logging`, `warnings`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `..._dynamo.utils`, `..pattern_matcher`, `.joint_graph`
