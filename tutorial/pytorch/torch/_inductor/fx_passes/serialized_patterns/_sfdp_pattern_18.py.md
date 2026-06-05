# _sfdp_pattern_18.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/fx_passes/serialized_patterns/_sfdp_pattern_18.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements FX graph transformation passes.
- **用途（中文）**: 该模块实现 FX 图变换 pass。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
# mypy: ignore-errors

# noqa: F401, E501
# This is an auto-generated file. Please do not modify it by hand.
# To re-generate, run:
# cd ~/pytorch && python torchgen/fuse/gen_patterns.py

import torch
import torch._inductor
import operator

aten = torch.ops.aten
prims = torch.ops.prims

from torch._inductor.pattern_matcher import (
   Arg,
   CallFunction,
   CallFunctionVarArgs,
   CallMethod,
   CallMethodVarArgs,
````
- **EN**: Imports dependencies such as `torch`, `torch._inductor`, `operator`, and `torch._inductor.pattern_matcher` for the logic in this range. Initializes or updates values such as `aten`, and `prims`.
- **CN**: 这里导入了 `torch`、`torch._inductor`、`operator`、`torch._inductor.pattern_matcher` 等依赖，为后续逻辑提供基础能力。初始化或更新了 `aten`、`prims` 等值。

### Lines 21-40 / 第 21-40 行
````python
   CallModule,
   CallModuleVarArgs,
   ExclusiveKeywordArg,
   Ignored,
   KeywordArg,
   ListOf,
   MultiOutputPattern,
   PatternExpr,
   RepeatedExpr,
   _TargetArgsExpr,
   _TargetExpr,
   _TargetExprVarArgs,
)
rand_default = CallFunction(aten.rand.default, Ignored(), dtype=Ignored(), device=Ignored(), pin_memory=False)
gt_Scalar = CallFunction(aten.gt.Scalar, rand_default, KeywordArg('dropout_p'), _users=2)
permute_default = CallFunction(aten.permute.default, KeywordArg('query'), Ignored())
expand_default = CallFunction(aten.expand.default, permute_default, Ignored())
clone_default = CallFunction(aten.clone.default, expand_default, memory_format=torch.contiguous_format)
view_default = CallFunction(aten.view.default, clone_default, Ignored(), _users=2)
permute_default_1 = CallFunction(aten.permute.default, KeywordArg('key'), Ignored(), _users=2)
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `rand_default`, `gt_Scalar`, `permute_default`, `expand_default`, `clone_default`, `view_default`, and `...+1`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `rand_default`、`gt_Scalar`、`permute_default`、`expand_default`、`clone_default`、`view_default`、`另有1项` 等值。

### Lines 41-60 / 第 41-60 行
````python
permute_default_2 = CallFunction(aten.permute.default, permute_default_1, Ignored())
expand_default_1 = CallFunction(aten.expand.default, permute_default_2, Ignored())
clone_default_1 = CallFunction(aten.clone.default, expand_default_1, memory_format=torch.contiguous_format)
view_default_1 = CallFunction(aten.view.default, clone_default_1, Ignored(), _users=2)
bmm_default = CallFunction(aten.bmm.default, view_default, view_default_1)
view_default_2 = CallFunction(aten.view.default, bmm_default, Ignored())
full_default = CallFunction(aten.full.default, [], KeywordArg('inv_scale'), dtype=Ignored(), device=Ignored(), pin_memory=False, _users=2)
div_Tensor = CallFunction(aten.div.Tensor, view_default_2, full_default)
full_default_1 = CallFunction(aten.full.default, [], Ignored(), dtype=Ignored(), device=Ignored(), pin_memory=False)
where_self = CallFunction(aten.where.self, KeywordArg('causal_mask'), div_Tensor, full_default_1, _users=2)
amax_default = CallFunction(aten.amax.default, where_self, Ignored(), True)
sub_Tensor = CallFunction(aten.sub.Tensor, where_self, amax_default)
exp_default = CallFunction(aten.exp.default, sub_Tensor, _users=2)
sum_dim_IntList = CallFunction(aten.sum.dim_IntList, exp_default, Ignored(), True)
div_Tensor_1 = CallFunction(aten.div.Tensor, exp_default, sum_dim_IntList, _users=3)
mul_Tensor = CallFunction(aten.mul.Tensor, gt_Scalar, div_Tensor_1)
mul_Tensor_1 = CallFunction(aten.mul.Tensor, mul_Tensor, Ignored())
expand_default_2 = CallFunction(aten.expand.default, mul_Tensor_1, Ignored())
view_default_3 = CallFunction(aten.view.default, expand_default_2, Ignored(), _users=2)
permute_default_3 = CallFunction(aten.permute.default, KeywordArg('value'), Ignored(), _users=2)
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `permute_default_2`, `expand_default_1`, `clone_default_1`, `view_default_1`, `bmm_default`, `view_default_2`, and `...+14`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `permute_default_2`、`expand_default_1`、`clone_default_1`、`view_default_1`、`bmm_default`、`view_default_2`、`另有14项` 等值。

### Lines 61-80 / 第 61-80 行
````python
expand_default_3 = CallFunction(aten.expand.default, permute_default_3, Ignored())
clone_default_2 = CallFunction(aten.clone.default, expand_default_3, memory_format=torch.contiguous_format)
view_default_4 = CallFunction(aten.view.default, clone_default_2, Ignored(), _users=2)
bmm_default_1 = CallFunction(aten.bmm.default, view_default_3, view_default_4)
view_default_5 = CallFunction(aten.view.default, bmm_default_1, Ignored())
neg_default = CallFunction(aten.neg.default, div_Tensor_1)
view_default_6 = CallFunction(aten.view.default, KeywordArg('tangents_1'), Ignored(), _users=2)
permute_default_4 = CallFunction(aten.permute.default, view_default_4, Ignored())
bmm_default_2 = CallFunction(aten.bmm.default, view_default_6, permute_default_4)
view_default_7 = CallFunction(aten.view.default, bmm_default_2, Ignored())
convert_element_type_default = CallFunction(prims.convert_element_type.default, gt_Scalar, Ignored())
mul_Tensor_2 = CallFunction(aten.mul.Tensor, convert_element_type_default, Ignored())
mul_Tensor_3 = CallFunction(aten.mul.Tensor, view_default_7, mul_Tensor_2)
mul_Tensor_4 = CallFunction(aten.mul.Tensor, mul_Tensor_3, div_Tensor_1, _users=2)
sum_dim_IntList_1 = CallFunction(aten.sum.dim_IntList, mul_Tensor_4, Ignored(), True)
fma_default = CallFunction(prims.fma.default, neg_default, sum_dim_IntList_1, mul_Tensor_4)
scalar_tensor_default = CallFunction(aten.scalar_tensor.default, Ignored(), dtype=Ignored(), layout=torch.strided, device=Ignored())
where_self_1 = CallFunction(aten.where.self, KeywordArg('causal_mask'), fma_default, scalar_tensor_default)
div_Tensor_2 = CallFunction(aten.div.Tensor, where_self_1, full_default)
view_default_8 = CallFunction(aten.view.default, div_Tensor_2, Ignored(), _users=2)
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `expand_default_3`, `clone_default_2`, `view_default_4`, `bmm_default_1`, `view_default_5`, `neg_default`, and `...+14`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `expand_default_3`、`clone_default_2`、`view_default_4`、`bmm_default_1`、`view_default_5`、`neg_default`、`另有14项` 等值。

### Lines 81-100 / 第 81-100 行
````python
permute_default_5 = CallFunction(aten.permute.default, view_default_1, Ignored())
bmm_default_3 = CallFunction(aten.bmm.default, view_default_8, permute_default_5)
view_default_9 = CallFunction(aten.view.default, bmm_default_3, Ignored())
permute_default_6 = CallFunction(aten.permute.default, view_default_9, Ignored())
permute_default_7 = CallFunction(aten.permute.default, view_default, Ignored())
bmm_default_4 = CallFunction(aten.bmm.default, permute_default_7, view_default_8)
view_default_10 = CallFunction(aten.view.default, bmm_default_4, Ignored())
permute_default_8 = CallFunction(aten.permute.default, view_default_10, Ignored())
permute_default_9 = CallFunction(aten.permute.default, permute_default_8, Ignored())
permute_default_10 = CallFunction(aten.permute.default, view_default_3, Ignored())
bmm_default_5 = CallFunction(aten.bmm.default, permute_default_10, view_default_6)
view_default_11 = CallFunction(aten.view.default, bmm_default_5, Ignored())
permute_default_11 = CallFunction(aten.permute.default, view_default_11, Ignored())
_sfdp_pattern_18_training = MultiOutputPattern([view_default_5,
  permute_default_1,
  permute_default_3,
  permute_default_6,
  permute_default_9,
  permute_default_11,
  None,
````
- **EN**: Initializes or updates values such as `permute_default_5`, `bmm_default_3`, `view_default_9`, `permute_default_6`, `permute_default_7`, `bmm_default_4`, and `...+8`.
- **CN**: 初始化或更新了 `permute_default_5`、`bmm_default_3`、`view_default_9`、`permute_default_6`、`permute_default_7`、`bmm_default_4`、`另有8项` 等值。

### Lines 101-120 / 第 101-120 行
````python
  None,
  None
])


permute_default = CallFunction(aten.permute.default, KeywordArg('query'), Ignored())
expand_default = CallFunction(aten.expand.default, permute_default, Ignored())
clone_default = CallFunction(aten.clone.default, expand_default, memory_format=torch.contiguous_format)
view_default = CallFunction(aten.view.default, clone_default, Ignored())
permute_default_1 = CallFunction(aten.permute.default, KeywordArg('key'), Ignored(), _users=2)
permute_default_2 = CallFunction(aten.permute.default, permute_default_1, Ignored())
expand_default_1 = CallFunction(aten.expand.default, permute_default_2, Ignored())
clone_default_1 = CallFunction(aten.clone.default, expand_default_1, memory_format=torch.contiguous_format)
view_default_1 = CallFunction(aten.view.default, clone_default_1, Ignored())
bmm_default = CallFunction(aten.bmm.default, view_default, view_default_1)
view_default_2 = CallFunction(aten.view.default, bmm_default, Ignored())
full_default = CallFunction(aten.full.default, [], KeywordArg('inv_scale'), dtype=Ignored(), device=Ignored(), pin_memory=False)
div_Tensor = CallFunction(aten.div.Tensor, view_default_2, full_default)
full_default_1 = CallFunction(aten.full.default, [], Ignored(), dtype=Ignored(), device=Ignored(), pin_memory=False)
where_self = CallFunction(aten.where.self, KeywordArg('causal_mask'), div_Tensor, full_default_1, _users=2)
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `permute_default`, `expand_default`, `clone_default`, `view_default`, `permute_default_1`, `permute_default_2`, and `...+9`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `permute_default`、`expand_default`、`clone_default`、`view_default`、`permute_default_1`、`permute_default_2`、`另有9项` 等值。

### Lines 121-140 / 第 121-140 行
````python
amax_default = CallFunction(aten.amax.default, where_self, Ignored(), True)
sub_Tensor = CallFunction(aten.sub.Tensor, where_self, amax_default)
exp_default = CallFunction(aten.exp.default, sub_Tensor, _users=2)
sum_dim_IntList = CallFunction(aten.sum.dim_IntList, exp_default, Ignored(), True)
div_Tensor_1 = CallFunction(aten.div.Tensor, exp_default, sum_dim_IntList)
expand_default_2 = CallFunction(aten.expand.default, div_Tensor_1, Ignored())
view_default_3 = CallFunction(aten.view.default, expand_default_2, Ignored())
permute_default_3 = CallFunction(aten.permute.default, KeywordArg('value'), Ignored(), _users=2)
expand_default_3 = CallFunction(aten.expand.default, permute_default_3, Ignored())
clone_default_2 = CallFunction(aten.clone.default, expand_default_3, memory_format=torch.contiguous_format)
view_default_4 = CallFunction(aten.view.default, clone_default_2, Ignored())
bmm_default_1 = CallFunction(aten.bmm.default, view_default_3, view_default_4)
view_default_5 = CallFunction(aten.view.default, bmm_default_1, Ignored())
_sfdp_pattern_18_inference = MultiOutputPattern([view_default_5,
  permute_default_1,
  permute_default_3
])


rand_default = CallFunction(aten.rand.default, Ignored(), dtype=Ignored(), device=Ignored(), pin_memory=False)
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `amax_default`, `sub_Tensor`, `exp_default`, `sum_dim_IntList`, `div_Tensor_1`, `expand_default_2`, and `...+9`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `amax_default`、`sub_Tensor`、`exp_default`、`sum_dim_IntList`、`div_Tensor_1`、`expand_default_2`、`另有9项` 等值。

### Lines 141-160 / 第 141-160 行
````python
gt_Scalar = CallFunction(aten.gt.Scalar, rand_default, KeywordArg('dropout_p'), _users=2)
permute_default = CallFunction(aten.permute.default, KeywordArg('query'), Ignored())
expand_default = CallFunction(aten.expand.default, permute_default, Ignored())
view_default = CallFunction(aten.view.default, expand_default, Ignored(), _users=2)
permute_default_1 = CallFunction(aten.permute.default, KeywordArg('key'), Ignored(), _users=2)
permute_default_2 = CallFunction(aten.permute.default, permute_default_1, Ignored())
expand_default_1 = CallFunction(aten.expand.default, permute_default_2, Ignored())
view_default_1 = CallFunction(aten.view.default, expand_default_1, Ignored(), _users=2)
bmm_default = CallFunction(aten.bmm.default, view_default, view_default_1)
view_default_2 = CallFunction(aten.view.default, bmm_default, Ignored())
full_default = CallFunction(aten.full.default, [], KeywordArg('inv_scale'), dtype=Ignored(), device=Ignored(), pin_memory=False, _users=2)
div_Tensor = CallFunction(aten.div.Tensor, view_default_2, full_default)
full_default_1 = CallFunction(aten.full.default, [], Ignored(), dtype=Ignored(), device=Ignored(), pin_memory=False)
where_self = CallFunction(aten.where.self, KeywordArg('causal_mask'), div_Tensor, full_default_1, _users=2)
amax_default = CallFunction(aten.amax.default, where_self, Ignored(), True)
sub_Tensor = CallFunction(aten.sub.Tensor, where_self, amax_default)
exp_default = CallFunction(aten.exp.default, sub_Tensor, _users=2)
sum_dim_IntList = CallFunction(aten.sum.dim_IntList, exp_default, Ignored(), True)
div_Tensor_1 = CallFunction(aten.div.Tensor, exp_default, sum_dim_IntList, _users=3)
mul_Tensor = CallFunction(aten.mul.Tensor, gt_Scalar, div_Tensor_1)
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `gt_Scalar`, `permute_default`, `expand_default`, `view_default`, `permute_default_1`, `permute_default_2`, and `...+14`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `gt_Scalar`、`permute_default`、`expand_default`、`view_default`、`permute_default_1`、`permute_default_2`、`另有14项` 等值。

### Lines 161-180 / 第 161-180 行
````python
mul_Tensor_1 = CallFunction(aten.mul.Tensor, mul_Tensor, Ignored())
expand_default_2 = CallFunction(aten.expand.default, mul_Tensor_1, Ignored())
view_default_3 = CallFunction(aten.view.default, expand_default_2, Ignored(), _users=2)
permute_default_3 = CallFunction(aten.permute.default, KeywordArg('value'), Ignored(), _users=2)
expand_default_3 = CallFunction(aten.expand.default, permute_default_3, Ignored())
view_default_4 = CallFunction(aten.view.default, expand_default_3, Ignored(), _users=2)
bmm_default_1 = CallFunction(aten.bmm.default, view_default_3, view_default_4)
view_default_5 = CallFunction(aten.view.default, bmm_default_1, Ignored())
neg_default = CallFunction(aten.neg.default, div_Tensor_1)
view_default_6 = CallFunction(aten.view.default, KeywordArg('tangents_1'), Ignored(), _users=2)
permute_default_4 = CallFunction(aten.permute.default, view_default_4, Ignored())
bmm_default_2 = CallFunction(aten.bmm.default, view_default_6, permute_default_4)
view_default_7 = CallFunction(aten.view.default, bmm_default_2, Ignored())
convert_element_type_default = CallFunction(prims.convert_element_type.default, gt_Scalar, Ignored())
mul_Tensor_2 = CallFunction(aten.mul.Tensor, convert_element_type_default, Ignored())
mul_Tensor_3 = CallFunction(aten.mul.Tensor, view_default_7, mul_Tensor_2)
mul_Tensor_4 = CallFunction(aten.mul.Tensor, mul_Tensor_3, div_Tensor_1, _users=2)
sum_dim_IntList_1 = CallFunction(aten.sum.dim_IntList, mul_Tensor_4, Ignored(), True)
fma_default = CallFunction(prims.fma.default, neg_default, sum_dim_IntList_1, mul_Tensor_4)
scalar_tensor_default = CallFunction(aten.scalar_tensor.default, Ignored(), dtype=Ignored(), layout=torch.strided, device=Ignored())
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `mul_Tensor_1`, `expand_default_2`, `view_default_3`, `permute_default_3`, `expand_default_3`, `view_default_4`, and `...+14`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `mul_Tensor_1`、`expand_default_2`、`view_default_3`、`permute_default_3`、`expand_default_3`、`view_default_4`、`另有14项` 等值。

### Lines 181-200 / 第 181-200 行
````python
where_self_1 = CallFunction(aten.where.self, KeywordArg('causal_mask'), fma_default, scalar_tensor_default)
div_Tensor_2 = CallFunction(aten.div.Tensor, where_self_1, full_default)
view_default_8 = CallFunction(aten.view.default, div_Tensor_2, Ignored(), _users=2)
permute_default_5 = CallFunction(aten.permute.default, view_default_1, Ignored())
bmm_default_3 = CallFunction(aten.bmm.default, view_default_8, permute_default_5)
view_default_9 = CallFunction(aten.view.default, bmm_default_3, Ignored())
permute_default_6 = CallFunction(aten.permute.default, view_default_9, Ignored())
permute_default_7 = CallFunction(aten.permute.default, view_default, Ignored())
bmm_default_4 = CallFunction(aten.bmm.default, permute_default_7, view_default_8)
view_default_10 = CallFunction(aten.view.default, bmm_default_4, Ignored())
permute_default_8 = CallFunction(aten.permute.default, view_default_10, Ignored())
permute_default_9 = CallFunction(aten.permute.default, permute_default_8, Ignored())
permute_default_10 = CallFunction(aten.permute.default, view_default_3, Ignored())
bmm_default_5 = CallFunction(aten.bmm.default, permute_default_10, view_default_6)
view_default_11 = CallFunction(aten.view.default, bmm_default_5, Ignored())
permute_default_11 = CallFunction(aten.permute.default, view_default_11, Ignored())
_sfdp_pattern_18_bs1_training = MultiOutputPattern([view_default_5,
  permute_default_1,
  permute_default_3,
  permute_default_6,
````
- **EN**: Initializes or updates values such as `where_self_1`, `div_Tensor_2`, `view_default_8`, `permute_default_5`, `bmm_default_3`, `view_default_9`, and `...+11`.
- **CN**: 初始化或更新了 `where_self_1`、`div_Tensor_2`、`view_default_8`、`permute_default_5`、`bmm_default_3`、`view_default_9`、`另有11项` 等值。

### Lines 201-220 / 第 201-220 行
````python
  permute_default_9,
  permute_default_11,
  None,
  None,
  None
])


permute_default = CallFunction(aten.permute.default, KeywordArg('query'), Ignored())
expand_default = CallFunction(aten.expand.default, permute_default, Ignored())
view_default = CallFunction(aten.view.default, expand_default, Ignored())
permute_default_1 = CallFunction(aten.permute.default, KeywordArg('key'), Ignored(), _users=2)
permute_default_2 = CallFunction(aten.permute.default, permute_default_1, Ignored())
expand_default_1 = CallFunction(aten.expand.default, permute_default_2, Ignored())
view_default_1 = CallFunction(aten.view.default, expand_default_1, Ignored())
bmm_default = CallFunction(aten.bmm.default, view_default, view_default_1)
view_default_2 = CallFunction(aten.view.default, bmm_default, Ignored())
full_default = CallFunction(aten.full.default, [], KeywordArg('inv_scale'), dtype=Ignored(), device=Ignored(), pin_memory=False)
div_Tensor = CallFunction(aten.div.Tensor, view_default_2, full_default)
full_default_1 = CallFunction(aten.full.default, [], Ignored(), dtype=Ignored(), device=Ignored(), pin_memory=False)
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `permute_default`, `expand_default`, `view_default`, `permute_default_1`, `permute_default_2`, `expand_default_1`, and `...+6`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `permute_default`、`expand_default`、`view_default`、`permute_default_1`、`permute_default_2`、`expand_default_1`、`另有6项` 等值。

### Lines 221-240 / 第 221-240 行
````python
where_self = CallFunction(aten.where.self, KeywordArg('causal_mask'), div_Tensor, full_default_1, _users=2)
amax_default = CallFunction(aten.amax.default, where_self, Ignored(), True)
sub_Tensor = CallFunction(aten.sub.Tensor, where_self, amax_default)
exp_default = CallFunction(aten.exp.default, sub_Tensor, _users=2)
sum_dim_IntList = CallFunction(aten.sum.dim_IntList, exp_default, Ignored(), True)
div_Tensor_1 = CallFunction(aten.div.Tensor, exp_default, sum_dim_IntList)
expand_default_2 = CallFunction(aten.expand.default, div_Tensor_1, Ignored())
view_default_3 = CallFunction(aten.view.default, expand_default_2, Ignored())
permute_default_3 = CallFunction(aten.permute.default, KeywordArg('value'), Ignored(), _users=2)
expand_default_3 = CallFunction(aten.expand.default, permute_default_3, Ignored())
view_default_4 = CallFunction(aten.view.default, expand_default_3, Ignored())
bmm_default_1 = CallFunction(aten.bmm.default, view_default_3, view_default_4)
view_default_5 = CallFunction(aten.view.default, bmm_default_1, Ignored())
_sfdp_pattern_18_bs1_inference = MultiOutputPattern([view_default_5,
  permute_default_1,
  permute_default_3
])


rand_default = CallFunction(aten.rand.default, Ignored(), dtype=Ignored(), device=Ignored(), pin_memory=False)
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `where_self`, `amax_default`, `sub_Tensor`, `exp_default`, `sum_dim_IntList`, `div_Tensor_1`, and `...+9`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `where_self`、`amax_default`、`sub_Tensor`、`exp_default`、`sum_dim_IntList`、`div_Tensor_1`、`另有9项` 等值。

### Lines 241-260 / 第 241-260 行
````python
gt_Scalar = CallFunction(aten.gt.Scalar, rand_default, KeywordArg('dropout_p'), _users=2)
permute_default = CallFunction(aten.permute.default, KeywordArg('query'), Ignored())
expand_default = CallFunction(aten.expand.default, permute_default, Ignored())
clone_default = CallFunction(aten.clone.default, expand_default, memory_format=torch.contiguous_format)
view_default = CallFunction(aten.view.default, clone_default, Ignored(), _users=2)
permute_default_1 = CallFunction(aten.permute.default, KeywordArg('key'), Ignored(), _users=2)
permute_default_2 = CallFunction(aten.permute.default, permute_default_1, Ignored())
expand_default_1 = CallFunction(aten.expand.default, permute_default_2, Ignored())
clone_default_1 = CallFunction(aten.clone.default, expand_default_1, memory_format=torch.contiguous_format)
view_default_1 = CallFunction(aten.view.default, clone_default_1, Ignored(), _users=2)
bmm_default = CallFunction(aten.bmm.default, view_default, view_default_1)
view_default_2 = CallFunction(aten.view.default, bmm_default, Ignored())
full_default = CallFunction(aten.full.default, [], KeywordArg('inv_scale'), dtype=Ignored(), device=Ignored(), pin_memory=False, _users=2)
div_Tensor = CallFunction(aten.div.Tensor, view_default_2, full_default)
full_default_1 = CallFunction(aten.full.default, [], Ignored(), dtype=Ignored(), device=Ignored(), pin_memory=False)
where_self = CallFunction(aten.where.self, KeywordArg('causal_mask'), div_Tensor, full_default_1)
convert_element_type_default = CallFunction(prims.convert_element_type.default, where_self, Ignored(), _users=2)
amax_default = CallFunction(aten.amax.default, convert_element_type_default, Ignored(), True)
sub_Tensor = CallFunction(aten.sub.Tensor, convert_element_type_default, amax_default)
exp_default = CallFunction(aten.exp.default, sub_Tensor, _users=2)
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `gt_Scalar`, `permute_default`, `expand_default`, `clone_default`, `view_default`, `permute_default_1`, and `...+14`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `gt_Scalar`、`permute_default`、`expand_default`、`clone_default`、`view_default`、`permute_default_1`、`另有14项` 等值。

### Lines 261-280 / 第 261-280 行
````python
sum_dim_IntList = CallFunction(aten.sum.dim_IntList, exp_default, Ignored(), True)
div_Tensor_1 = CallFunction(aten.div.Tensor, exp_default, sum_dim_IntList)
convert_element_type_default_1 = CallFunction(prims.convert_element_type.default, div_Tensor_1, Ignored(), _users=2)
mul_Tensor = CallFunction(aten.mul.Tensor, gt_Scalar, convert_element_type_default_1)
mul_Tensor_1 = CallFunction(aten.mul.Tensor, mul_Tensor, Ignored())
expand_default_2 = CallFunction(aten.expand.default, mul_Tensor_1, Ignored())
view_default_3 = CallFunction(aten.view.default, expand_default_2, Ignored(), _users=2)
permute_default_3 = CallFunction(aten.permute.default, KeywordArg('value'), Ignored(), _users=2)
expand_default_3 = CallFunction(aten.expand.default, permute_default_3, Ignored())
clone_default_2 = CallFunction(aten.clone.default, expand_default_3, memory_format=torch.contiguous_format)
view_default_4 = CallFunction(aten.view.default, clone_default_2, Ignored(), _users=2)
bmm_default_1 = CallFunction(aten.bmm.default, view_default_3, view_default_4)
view_default_5 = CallFunction(aten.view.default, bmm_default_1, Ignored())
convert_element_type_default_2 = CallFunction(prims.convert_element_type.default, convert_element_type_default_1, Ignored(), _users=2)
neg_default = CallFunction(aten.neg.default, convert_element_type_default_2)
view_default_6 = CallFunction(aten.view.default, KeywordArg('tangents_1'), Ignored(), _users=2)
permute_default_4 = CallFunction(aten.permute.default, view_default_4, Ignored())
bmm_default_2 = CallFunction(aten.bmm.default, view_default_6, permute_default_4)
view_default_7 = CallFunction(aten.view.default, bmm_default_2, Ignored())
convert_element_type_default_3 = CallFunction(prims.convert_element_type.default, gt_Scalar, Ignored())
````
- **EN**: Initializes or updates values such as `sum_dim_IntList`, `div_Tensor_1`, `convert_element_type_default_1`, `mul_Tensor`, `mul_Tensor_1`, `expand_default_2`, and `...+14`.
- **CN**: 初始化或更新了 `sum_dim_IntList`、`div_Tensor_1`、`convert_element_type_default_1`、`mul_Tensor`、`mul_Tensor_1`、`expand_default_2`、`另有14项` 等值。

### Lines 281-300 / 第 281-300 行
````python
mul_Tensor_2 = CallFunction(aten.mul.Tensor, convert_element_type_default_3, Ignored())
mul_Tensor_3 = CallFunction(aten.mul.Tensor, view_default_7, mul_Tensor_2)
convert_element_type_default_4 = CallFunction(prims.convert_element_type.default, mul_Tensor_3, Ignored())
mul_Tensor_4 = CallFunction(aten.mul.Tensor, convert_element_type_default_4, convert_element_type_default_2, _users=2)
sum_dim_IntList_1 = CallFunction(aten.sum.dim_IntList, mul_Tensor_4, Ignored(), True)
fma_default = CallFunction(prims.fma.default, neg_default, sum_dim_IntList_1, mul_Tensor_4)
convert_element_type_default_5 = CallFunction(prims.convert_element_type.default, fma_default, Ignored())
scalar_tensor_default = CallFunction(aten.scalar_tensor.default, Ignored(), dtype=Ignored(), layout=torch.strided, device=Ignored())
where_self_1 = CallFunction(aten.where.self, KeywordArg('causal_mask'), convert_element_type_default_5, scalar_tensor_default)
div_Tensor_2 = CallFunction(aten.div.Tensor, where_self_1, full_default)
view_default_8 = CallFunction(aten.view.default, div_Tensor_2, Ignored(), _users=2)
permute_default_5 = CallFunction(aten.permute.default, view_default_1, Ignored())
bmm_default_3 = CallFunction(aten.bmm.default, view_default_8, permute_default_5)
view_default_9 = CallFunction(aten.view.default, bmm_default_3, Ignored())
permute_default_6 = CallFunction(aten.permute.default, view_default_9, Ignored())
permute_default_7 = CallFunction(aten.permute.default, view_default, Ignored())
bmm_default_4 = CallFunction(aten.bmm.default, permute_default_7, view_default_8)
view_default_10 = CallFunction(aten.view.default, bmm_default_4, Ignored())
permute_default_8 = CallFunction(aten.permute.default, view_default_10, Ignored())
permute_default_9 = CallFunction(aten.permute.default, permute_default_8, Ignored())
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `mul_Tensor_2`, `mul_Tensor_3`, `convert_element_type_default_4`, `mul_Tensor_4`, `sum_dim_IntList_1`, `fma_default`, and `...+14`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `mul_Tensor_2`、`mul_Tensor_3`、`convert_element_type_default_4`、`mul_Tensor_4`、`sum_dim_IntList_1`、`fma_default`、`另有14项` 等值。

### Lines 301-320 / 第 301-320 行
````python
permute_default_10 = CallFunction(aten.permute.default, view_default_3, Ignored())
bmm_default_5 = CallFunction(aten.bmm.default, permute_default_10, view_default_6)
view_default_11 = CallFunction(aten.view.default, bmm_default_5, Ignored())
permute_default_11 = CallFunction(aten.permute.default, view_default_11, Ignored())
_sfdp_pattern_18_half_training = MultiOutputPattern([view_default_5,
  permute_default_1,
  permute_default_3,
  permute_default_6,
  permute_default_9,
  permute_default_11,
  None,
  None,
  None
])


permute_default = CallFunction(aten.permute.default, KeywordArg('query'), Ignored())
expand_default = CallFunction(aten.expand.default, permute_default, Ignored())
clone_default = CallFunction(aten.clone.default, expand_default, memory_format=torch.contiguous_format)
view_default = CallFunction(aten.view.default, clone_default, Ignored())
````
- **EN**: Initializes or updates values such as `permute_default_10`, `bmm_default_5`, `view_default_11`, `permute_default_11`, `_sfdp_pattern_18_half_training`, `permute_default`, and `...+3`.
- **CN**: 初始化或更新了 `permute_default_10`、`bmm_default_5`、`view_default_11`、`permute_default_11`、`_sfdp_pattern_18_half_training`、`permute_default`、`另有3项` 等值。

### Lines 321-340 / 第 321-340 行
````python
permute_default_1 = CallFunction(aten.permute.default, KeywordArg('key'), Ignored(), _users=2)
permute_default_2 = CallFunction(aten.permute.default, permute_default_1, Ignored())
expand_default_1 = CallFunction(aten.expand.default, permute_default_2, Ignored())
clone_default_1 = CallFunction(aten.clone.default, expand_default_1, memory_format=torch.contiguous_format)
view_default_1 = CallFunction(aten.view.default, clone_default_1, Ignored())
bmm_default = CallFunction(aten.bmm.default, view_default, view_default_1)
view_default_2 = CallFunction(aten.view.default, bmm_default, Ignored())
full_default = CallFunction(aten.full.default, [], KeywordArg('inv_scale'), dtype=Ignored(), device=Ignored(), pin_memory=False)
div_Tensor = CallFunction(aten.div.Tensor, view_default_2, full_default)
full_default_1 = CallFunction(aten.full.default, [], Ignored(), dtype=Ignored(), device=Ignored(), pin_memory=False)
where_self = CallFunction(aten.where.self, KeywordArg('causal_mask'), div_Tensor, full_default_1)
convert_element_type_default = CallFunction(prims.convert_element_type.default, where_self, Ignored(), _users=2)
amax_default = CallFunction(aten.amax.default, convert_element_type_default, Ignored(), True)
sub_Tensor = CallFunction(aten.sub.Tensor, convert_element_type_default, amax_default)
exp_default = CallFunction(aten.exp.default, sub_Tensor, _users=2)
sum_dim_IntList = CallFunction(aten.sum.dim_IntList, exp_default, Ignored(), True)
div_Tensor_1 = CallFunction(aten.div.Tensor, exp_default, sum_dim_IntList)
convert_element_type_default_1 = CallFunction(prims.convert_element_type.default, div_Tensor_1, Ignored())
expand_default_2 = CallFunction(aten.expand.default, convert_element_type_default_1, Ignored())
view_default_3 = CallFunction(aten.view.default, expand_default_2, Ignored())
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `permute_default_1`, `permute_default_2`, `expand_default_1`, `clone_default_1`, `view_default_1`, `bmm_default`, and `...+14`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `permute_default_1`、`permute_default_2`、`expand_default_1`、`clone_default_1`、`view_default_1`、`bmm_default`、`另有14项` 等值。

### Lines 341-360 / 第 341-360 行
````python
permute_default_3 = CallFunction(aten.permute.default, KeywordArg('value'), Ignored(), _users=2)
expand_default_3 = CallFunction(aten.expand.default, permute_default_3, Ignored())
clone_default_2 = CallFunction(aten.clone.default, expand_default_3, memory_format=torch.contiguous_format)
view_default_4 = CallFunction(aten.view.default, clone_default_2, Ignored())
bmm_default_1 = CallFunction(aten.bmm.default, view_default_3, view_default_4)
view_default_5 = CallFunction(aten.view.default, bmm_default_1, Ignored())
_sfdp_pattern_18_half_inference = MultiOutputPattern([view_default_5,
  permute_default_1,
  permute_default_3
])


rand_default = CallFunction(aten.rand.default, Ignored(), dtype=Ignored(), device=Ignored(), pin_memory=False)
gt_Scalar = CallFunction(aten.gt.Scalar, rand_default, KeywordArg('dropout_p'), _users=2)
permute_default = CallFunction(aten.permute.default, KeywordArg('query'), Ignored())
expand_default = CallFunction(aten.expand.default, permute_default, Ignored())
view_default = CallFunction(aten.view.default, expand_default, Ignored(), _users=2)
permute_default_1 = CallFunction(aten.permute.default, KeywordArg('key'), Ignored(), _users=2)
permute_default_2 = CallFunction(aten.permute.default, permute_default_1, Ignored())
expand_default_1 = CallFunction(aten.expand.default, permute_default_2, Ignored())
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `permute_default_3`, `expand_default_3`, `clone_default_2`, `view_default_4`, `bmm_default_1`, `view_default_5`, and `...+9`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `permute_default_3`、`expand_default_3`、`clone_default_2`、`view_default_4`、`bmm_default_1`、`view_default_5`、`另有9项` 等值。

### Lines 361-380 / 第 361-380 行
````python
view_default_1 = CallFunction(aten.view.default, expand_default_1, Ignored(), _users=2)
bmm_default = CallFunction(aten.bmm.default, view_default, view_default_1)
view_default_2 = CallFunction(aten.view.default, bmm_default, Ignored())
full_default = CallFunction(aten.full.default, [], KeywordArg('inv_scale'), dtype=Ignored(), device=Ignored(), pin_memory=False, _users=2)
div_Tensor = CallFunction(aten.div.Tensor, view_default_2, full_default)
full_default_1 = CallFunction(aten.full.default, [], Ignored(), dtype=Ignored(), device=Ignored(), pin_memory=False)
where_self = CallFunction(aten.where.self, KeywordArg('causal_mask'), div_Tensor, full_default_1)
convert_element_type_default = CallFunction(prims.convert_element_type.default, where_self, Ignored(), _users=2)
amax_default = CallFunction(aten.amax.default, convert_element_type_default, Ignored(), True)
sub_Tensor = CallFunction(aten.sub.Tensor, convert_element_type_default, amax_default)
exp_default = CallFunction(aten.exp.default, sub_Tensor, _users=2)
sum_dim_IntList = CallFunction(aten.sum.dim_IntList, exp_default, Ignored(), True)
div_Tensor_1 = CallFunction(aten.div.Tensor, exp_default, sum_dim_IntList)
convert_element_type_default_1 = CallFunction(prims.convert_element_type.default, div_Tensor_1, Ignored(), _users=2)
mul_Tensor = CallFunction(aten.mul.Tensor, gt_Scalar, convert_element_type_default_1)
mul_Tensor_1 = CallFunction(aten.mul.Tensor, mul_Tensor, Ignored())
expand_default_2 = CallFunction(aten.expand.default, mul_Tensor_1, Ignored())
view_default_3 = CallFunction(aten.view.default, expand_default_2, Ignored(), _users=2)
permute_default_3 = CallFunction(aten.permute.default, KeywordArg('value'), Ignored(), _users=2)
expand_default_3 = CallFunction(aten.expand.default, permute_default_3, Ignored())
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `view_default_1`, `bmm_default`, `view_default_2`, `full_default`, `div_Tensor`, `full_default_1`, and `...+14`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `view_default_1`、`bmm_default`、`view_default_2`、`full_default`、`div_Tensor`、`full_default_1`、`另有14项` 等值。

### Lines 381-400 / 第 381-400 行
````python
view_default_4 = CallFunction(aten.view.default, expand_default_3, Ignored(), _users=2)
bmm_default_1 = CallFunction(aten.bmm.default, view_default_3, view_default_4)
view_default_5 = CallFunction(aten.view.default, bmm_default_1, Ignored())
convert_element_type_default_2 = CallFunction(prims.convert_element_type.default, convert_element_type_default_1, Ignored(), _users=2)
neg_default = CallFunction(aten.neg.default, convert_element_type_default_2)
view_default_6 = CallFunction(aten.view.default, KeywordArg('tangents_1'), Ignored(), _users=2)
permute_default_4 = CallFunction(aten.permute.default, view_default_4, Ignored())
bmm_default_2 = CallFunction(aten.bmm.default, view_default_6, permute_default_4)
view_default_7 = CallFunction(aten.view.default, bmm_default_2, Ignored())
convert_element_type_default_3 = CallFunction(prims.convert_element_type.default, gt_Scalar, Ignored())
mul_Tensor_2 = CallFunction(aten.mul.Tensor, convert_element_type_default_3, Ignored())
mul_Tensor_3 = CallFunction(aten.mul.Tensor, view_default_7, mul_Tensor_2)
convert_element_type_default_4 = CallFunction(prims.convert_element_type.default, mul_Tensor_3, Ignored())
mul_Tensor_4 = CallFunction(aten.mul.Tensor, convert_element_type_default_4, convert_element_type_default_2, _users=2)
sum_dim_IntList_1 = CallFunction(aten.sum.dim_IntList, mul_Tensor_4, Ignored(), True)
fma_default = CallFunction(prims.fma.default, neg_default, sum_dim_IntList_1, mul_Tensor_4)
convert_element_type_default_5 = CallFunction(prims.convert_element_type.default, fma_default, Ignored())
scalar_tensor_default = CallFunction(aten.scalar_tensor.default, Ignored(), dtype=Ignored(), layout=torch.strided, device=Ignored())
where_self_1 = CallFunction(aten.where.self, KeywordArg('causal_mask'), convert_element_type_default_5, scalar_tensor_default)
div_Tensor_2 = CallFunction(aten.div.Tensor, where_self_1, full_default)
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `view_default_4`, `bmm_default_1`, `view_default_5`, `convert_element_type_default_2`, `neg_default`, `view_default_6`, and `...+14`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `view_default_4`、`bmm_default_1`、`view_default_5`、`convert_element_type_default_2`、`neg_default`、`view_default_6`、`另有14项` 等值。

### Lines 401-420 / 第 401-420 行
````python
view_default_8 = CallFunction(aten.view.default, div_Tensor_2, Ignored(), _users=2)
permute_default_5 = CallFunction(aten.permute.default, view_default_1, Ignored())
bmm_default_3 = CallFunction(aten.bmm.default, view_default_8, permute_default_5)
view_default_9 = CallFunction(aten.view.default, bmm_default_3, Ignored())
permute_default_6 = CallFunction(aten.permute.default, view_default_9, Ignored())
permute_default_7 = CallFunction(aten.permute.default, view_default, Ignored())
bmm_default_4 = CallFunction(aten.bmm.default, permute_default_7, view_default_8)
view_default_10 = CallFunction(aten.view.default, bmm_default_4, Ignored())
permute_default_8 = CallFunction(aten.permute.default, view_default_10, Ignored())
permute_default_9 = CallFunction(aten.permute.default, permute_default_8, Ignored())
permute_default_10 = CallFunction(aten.permute.default, view_default_3, Ignored())
bmm_default_5 = CallFunction(aten.bmm.default, permute_default_10, view_default_6)
view_default_11 = CallFunction(aten.view.default, bmm_default_5, Ignored())
permute_default_11 = CallFunction(aten.permute.default, view_default_11, Ignored())
_sfdp_pattern_18_half_bs1_training = MultiOutputPattern([view_default_5,
  permute_default_1,
  permute_default_3,
  permute_default_6,
  permute_default_9,
  permute_default_11,
````
- **EN**: Initializes or updates values such as `view_default_8`, `permute_default_5`, `bmm_default_3`, `view_default_9`, `permute_default_6`, `permute_default_7`, and `...+9`.
- **CN**: 初始化或更新了 `view_default_8`、`permute_default_5`、`bmm_default_3`、`view_default_9`、`permute_default_6`、`permute_default_7`、`另有9项` 等值。

### Lines 421-440 / 第 421-440 行
````python
  None,
  None,
  None
])


permute_default = CallFunction(aten.permute.default, KeywordArg('query'), Ignored())
expand_default = CallFunction(aten.expand.default, permute_default, Ignored())
view_default = CallFunction(aten.view.default, expand_default, Ignored())
permute_default_1 = CallFunction(aten.permute.default, KeywordArg('key'), Ignored(), _users=2)
permute_default_2 = CallFunction(aten.permute.default, permute_default_1, Ignored())
expand_default_1 = CallFunction(aten.expand.default, permute_default_2, Ignored())
view_default_1 = CallFunction(aten.view.default, expand_default_1, Ignored())
bmm_default = CallFunction(aten.bmm.default, view_default, view_default_1)
view_default_2 = CallFunction(aten.view.default, bmm_default, Ignored())
full_default = CallFunction(aten.full.default, [], KeywordArg('inv_scale'), dtype=Ignored(), device=Ignored(), pin_memory=False)
div_Tensor = CallFunction(aten.div.Tensor, view_default_2, full_default)
full_default_1 = CallFunction(aten.full.default, [], Ignored(), dtype=Ignored(), device=Ignored(), pin_memory=False)
where_self = CallFunction(aten.where.self, KeywordArg('causal_mask'), div_Tensor, full_default_1)
convert_element_type_default = CallFunction(prims.convert_element_type.default, where_self, Ignored(), _users=2)
````
- **EN**: Handles device-specific behavior, capabilities, or dispatch decisions. Tracks dtype-specific behavior or constants that affect numeric execution. Initializes or updates values such as `permute_default`, `expand_default`, `view_default`, `permute_default_1`, `permute_default_2`, `expand_default_1`, and `...+8`.
- **CN**: 处理与设备相关的行为、能力信息或派发决策。追踪与 dtype 相关的行为或常量，它们会影响数值执行方式。初始化或更新了 `permute_default`、`expand_default`、`view_default`、`permute_default_1`、`permute_default_2`、`expand_default_1`、`另有8项` 等值。

### Lines 441-457 / 第 441-457 行
````python
amax_default = CallFunction(aten.amax.default, convert_element_type_default, Ignored(), True)
sub_Tensor = CallFunction(aten.sub.Tensor, convert_element_type_default, amax_default)
exp_default = CallFunction(aten.exp.default, sub_Tensor, _users=2)
sum_dim_IntList = CallFunction(aten.sum.dim_IntList, exp_default, Ignored(), True)
div_Tensor_1 = CallFunction(aten.div.Tensor, exp_default, sum_dim_IntList)
convert_element_type_default_1 = CallFunction(prims.convert_element_type.default, div_Tensor_1, Ignored())
expand_default_2 = CallFunction(aten.expand.default, convert_element_type_default_1, Ignored())
view_default_3 = CallFunction(aten.view.default, expand_default_2, Ignored())
permute_default_3 = CallFunction(aten.permute.default, KeywordArg('value'), Ignored(), _users=2)
expand_default_3 = CallFunction(aten.expand.default, permute_default_3, Ignored())
view_default_4 = CallFunction(aten.view.default, expand_default_3, Ignored())
bmm_default_1 = CallFunction(aten.bmm.default, view_default_3, view_default_4)
view_default_5 = CallFunction(aten.view.default, bmm_default_1, Ignored())
_sfdp_pattern_18_half_bs1_inference = MultiOutputPattern([view_default_5,
  permute_default_1,
  permute_default_3
])
````
- **EN**: Initializes or updates values such as `amax_default`, `sub_Tensor`, `exp_default`, `sum_dim_IntList`, `div_Tensor_1`, `convert_element_type_default_1`, and `...+8`.
- **CN**: 初始化或更新了 `amax_default`、`sub_Tensor`、`exp_default`、`sum_dim_IntList`、`div_Tensor_1`、`convert_element_type_default_1`、`另有8项` 等值。

## Key Concepts / 关键概念
- **EN**: Implements FX graph transformation passes  
  **CN**: 实现 FX 图变换 pass
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Dtype propagation or validation  
  **CN**: 数据类型传播或校验

## Dependencies / 依赖关系
- **Standard library / 标准库**: `operator`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch._inductor`, `torch._inductor.pattern_matcher`
