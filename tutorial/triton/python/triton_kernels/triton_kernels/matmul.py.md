# matmul.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/triton_kernels/matmul.py`
- **Purpose / 用途:** Implementation module for matmul; it defines the runtime logic, helper types, and GPU-facing utilities used by the Triton kernels package via symbols FusedActivation, Epilogue, FnName, FusedComm, can_overflow_int32. / 用于 matmul 的实现模块；它定义了 Triton kernels 包使用的运行时逻辑、辅助类型以及面向 GPU 的工具；核心符号包括 FusedActivation、Epilogue、FnName、FusedComm、can_overflow_int32。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-2 (module)
```python
1| # isort: off
2| # fmt: off
```
**EN:** This preamble contains file-level comments or formatting directives that keep imports and generated code stable.

**CN:** 这一前导部分包含文件级注释或格式化指令，用于保持导入顺序和生成代码稳定。

### Block 2 — Lines 3-39 (module)
```python
 3| from dataclasses import dataclass
 4| import itertools
 5| import torch
 6| import triton
 7| from enum import Enum, auto
 8| import math
 9| from typing import Callable
10| # utilities
11| from triton_kernels import target_info
12| from triton_kernels.meta import Closure
13| from triton_kernels.numerics import InFlexData, OutFlexData
14| from triton_kernels.target_info import is_cuda
15| from triton_kernels.tensor_details.layout_details.hopper_scale import HopperMXScaleLayout
16| # details
17| from .matmul_details._matmul import _matmul
18| from .matmul_details._p_matmul import _p_matmul, get_per_device_per_stream_alloc_fn
19| from .numerics_details.mxfp import MXFP_BLOCK_SIZE
20| from .numerics_details.mxfp_details._downcast_to_mxfp import NVFP_BLOCK_SIZE
21| from .tensor_details.layout_details.strided import StridedLayout
22| from .tensor_details.layout_details.blackwell_scale import BlackwellActMXScaleLayout, SWIZZLE_SIZE_OUTER
23| from .tensor_details.layout_details.blackwell_value_shuffled import BlackwellMX4ValueShuffledLayout
24| from .matmul_details.opt_flags import (
25|     InapplicableConstraint,
26|     OptFlags as OptFlags,
27|     make_opt_flags,
28|     scoped_opt_flags as scoped_opt_flags,
29|     scoped_opt_flags_constraints as scoped_opt_flags_constraints,
30|     update_opt_flags_constraints,
31| )
32| from .matmul_details.opt_flags_details import opt_flags_nvidia
33| from .specialize import FnSpecs, SpecializationModule, ClosureArg
34| from .tensor import Storage, Tensor, UINT8, FP4, wrap_torch_tensor, RaggedTensorMetadata, is_tma_compliant, make_tma, convert_layout
35| from .tensor import dtype_to_torch_dtype, torch_dtype_to_dtype
36| from .reduce import reduce
37| from .reduce import PostprocessFn as ReducePostprocessFn
38| from .tensor_details.ragged_tensor import ragged_metadata_fields
39| 
```
**EN:** This block imports `dataclasses (dataclass)`, `itertools`, `torch`, `triton`, `enum (Enum, auto)`, `math`, `typing (Callable)`, `triton_kernels (target_info)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `dataclasses (dataclass)`, `itertools`, `torch`, `triton`, `enum (Enum, auto)`, `math`, `typing (Callable)`, `triton_kernels (target_info)` ，为模块提供所需的外部库和内部辅助工具。

### Block 3 — Lines 40-41 (FusedActivation)
```python
40| @dataclass(frozen=True)
41| class FusedActivation:
```
**EN:** Defines class `FusedActivation` with decorators `dataclass(frozen=True)` to organize related behavior. Key fields include `specs`, `fn_args`.

**CN:** 定义类 `FusedActivation`，带有装饰器 `dataclass(frozen=True)`，用于组织相关行为。关键字段包括 `specs`, `fn_args`.

### Block 4 — Lines 42-42 (FusedActivation)
```python
42|     specs: FnSpecs = FnSpecs.default()
```
**EN:** Annotated assignment stores `specs` and calls `FnSpecs.default`.

**CN:** 带类型注解的赋值保存 `specs`，并调用 `FnSpecs.default`.

### Block 5 — Lines 43-45 (FusedActivation)
```python
43|     fn_args: tuple[object, ...] = tuple()
44| 
45| 
```
**EN:** Annotated assignment stores `fn_args` and calls `tuple`.

**CN:** 带类型注解的赋值保存 `fn_args`，并调用 `tuple`.

### Block 6 — Lines 46-47 (Epilogue)
```python
46| @dataclass(frozen=True)
47| class Epilogue:
```
**EN:** Defines class `Epilogue` with decorators `dataclass(frozen=True)` to organize related behavior. Key fields include `specs`, `fn_arg_values_matmul`, `fn_arg_values_finalize`, `effective_itemsize`.

**CN:** 定义类 `Epilogue`，带有装饰器 `dataclass(frozen=True)`，用于组织相关行为。关键字段包括 `specs`, `fn_arg_values_matmul`, `fn_arg_values_finalize`, `effective_itemsize`.

### Block 7 — Lines 48-48 (Epilogue)
```python
48|     specs: FnSpecs = FnSpecs.default()
```
**EN:** Annotated assignment stores `specs` and calls `FnSpecs.default`.

**CN:** 带类型注解的赋值保存 `specs`，并调用 `FnSpecs.default`.

### Block 8 — Lines 49-49 (Epilogue)
```python
49|     fn_arg_values_matmul: tuple[object, ...] = tuple()
```
**EN:** Annotated assignment stores `fn_arg_values_matmul` and calls `tuple`.

**CN:** 带类型注解的赋值保存 `fn_arg_values_matmul`，并调用 `tuple`.

### Block 9 — Lines 50-50 (Epilogue)
```python
50|     fn_arg_values_finalize: tuple[object, ...] = tuple()
```
**EN:** Annotated assignment stores `fn_arg_values_finalize` and calls `tuple`.

**CN:** 带类型注解的赋值保存 `fn_arg_values_finalize`，并调用 `tuple`.

### Block 10 — Lines 51-52 (Epilogue)
```python
51|     effective_itemsize: float | None = None
52| 
```
**EN:** Annotated assignment stores `effective_itemsize` and stores constant `None`.

**CN:** 带类型注解的赋值保存 `effective_itemsize` and 保存常量 `None`.

### Block 11 — Lines 53-53 (FnName)
```python
53| class FnName(Enum):
```
**EN:** Defines class `FnName` inheriting from `Enum` to organize related behavior. Key fields include `QUANTIZE_MXFP8`, `QUANTIZE_MXFP4`, `QUANTIZE_NVFP4`.

**CN:** 定义类 `FnName`，继承自 `Enum`，用于组织相关行为。关键字段包括 `QUANTIZE_MXFP8`, `QUANTIZE_MXFP4`, `QUANTIZE_NVFP4`.

### Block 12 — Lines 54-54 (FnName)
```python
54|     QUANTIZE_MXFP8 = auto()
```
**EN:** Assigns `QUANTIZE_MXFP8` and calls `auto`.

**CN:** 将 `QUANTIZE_MXFP8`，并调用 `auto`.

### Block 13 — Lines 55-55 (FnName)
```python
55|     QUANTIZE_MXFP4 = auto()
```
**EN:** Assigns `QUANTIZE_MXFP4` and calls `auto`.

**CN:** 将 `QUANTIZE_MXFP4`，并调用 `auto`.

### Block 14 — Lines 56-58 (FnName)
```python
56|     QUANTIZE_NVFP4 = auto()
57| 
58| 
```
**EN:** Assigns `QUANTIZE_NVFP4` and calls `auto`.

**CN:** 将 `QUANTIZE_NVFP4`，并调用 `auto`.

### Block 15 — Lines 59-60 (FusedComm)
```python
59| @dataclass(frozen=True)
60| class FusedComm:
```
**EN:** Defines class `FusedComm` with decorators `dataclass(frozen=True)` to organize related behavior. Key fields include `out_handles`, `map_dst_coord`, `all_writes_issued`, `reduce_rank`, `n_reduce_shards`.

**CN:** 定义类 `FusedComm`，带有装饰器 `dataclass(frozen=True)`，用于组织相关行为。关键字段包括 `out_handles`, `map_dst_coord`, `all_writes_issued`, `reduce_rank`, `n_reduce_shards`.

### Block 16 — Lines 61-75 (FusedComm)
```python
61|     out_handles: torch.Tensor
62|     # Map from the kernel output coord to the destination shard idx and coord.
63|     # Used like:
64|     #  dst_shard_idx, dst_y_m, dst_y_n = map_dst_coord.fn(base_off_m, offs_m, base_off_n, offs_n, *map_dst_coord.closure)
65|     # Arguments:
66|     #   base_off_m: int | None     the base offset of offs_m; None if the rows are scattered
67|     #   offs_m: BLOCK_M(int)       the output row offsets
68|     #   base_off_n: int            the base offset of offs_n
69|     #   offs_n: BLOCK_N(int)       the output column offsets
70|     #   ...closure: tuple          additional arguments bound to the map_dst_coord function
71|     # Returns:
72|     #   dst_shard_idx: int | BLOCK_Mx1(int) | 1xBLOCK_N(int) | BLOCK_MxBLOCK_N(int)
73|     #                              the destination shard index or indices
74|     #   dst_y_m: BLOCK_M(int)      the destination row offsets
75|     #   dst_y_n: BLOCK_N(int)      the destination column offsets
```
**EN:** Annotated assignment stores `out_handles` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `out_handles` and 声明该带注解的字段.

### Block 17 — Lines 76-76 (FusedComm)
```python
76|     map_dst_coord: Closure
```
**EN:** Annotated assignment stores `map_dst_coord` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `map_dst_coord` and 声明该带注解的字段.

### Block 18 — Lines 77-77 (FusedComm)
```python
77|     all_writes_issued: Closure
```
**EN:** Annotated assignment stores `all_writes_issued` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `all_writes_issued` and 声明该带注解的字段.

### Block 19 — Lines 78-78 (FusedComm)
```python
78|     reduce_rank: int = 0
```
**EN:** Annotated assignment stores `reduce_rank` and stores constant `0`.

**CN:** 带类型注解的赋值保存 `reduce_rank` and 保存常量 `0`.

### Block 20 — Lines 79-80 (FusedComm)
```python
79|     n_reduce_shards: int = 1
80| 
```
**EN:** Annotated assignment stores `n_reduce_shards` and stores constant `1`.

**CN:** 带类型注解的赋值保存 `n_reduce_shards` and 保存常量 `1`.

### Block 21 — Lines 81-92 (module)
```python
81| specializations = SpecializationModule("matmul",
82|     kernels=[("_matmul", _matmul), ("_p_matmul", _p_matmul)],
83|     closure_args={
84|         "epilogue": ClosureArg("EPILOGUE_FN", "epilogue_fn_args"), #
85|         "activation": ClosureArg("ACTIVATION_FN", "activation_fn_args"), #
86|     },
87| )
88| # -----------------------------------------------------------------------------
89| #                    Matrix Multiplication + Outer Gather/Scatter
90| # -----------------------------------------------------------------------------
91| 
92| 
```
**EN:** Assigns `specializations` and calls `SpecializationModule`.

**CN:** 将 `specializations`，并调用 `SpecializationModule`.

### Block 22 — Lines 93-93 (can_overflow_int32)
```python
93| def can_overflow_int32(tensor: torch.Tensor):
```
**EN:** Defines function `can_overflow_int32(tensor)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `range`, `isinstance`, `tensor.storage.data.stride`, `tensor.stride` to implement its workflow.

**CN:** 定义函数 `can_overflow_int32(tensor)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `range`, `isinstance`, `tensor.storage.data.stride`, `tensor.stride` 来实现其工作流程.

### Block 23 — Lines 94-94 (can_overflow_int32)
```python
94|     max_int32 = (1 << 31) - 1
```
**EN:** Assigns `max_int32` and evaluates `(1 << 31) - 1`.

**CN:** 将 `max_int32` and 计算 `(1 << 31) - 1`.

### Block 24 — Lines 95-96 (can_overflow_int32)
```python
95|     offset = 0
96|     # TODO: this should always be tensor
```
**EN:** Assigns `offset` and stores constant `0`.

**CN:** 将 `offset` and 保存常量 `0`.

### Block 25 — Lines 97-97 (can_overflow_int32)
```python
97|     ndim = tensor.storage.data.ndim if isinstance(tensor, Tensor) else tensor.ndim
```
**EN:** Assigns `ndim` and uses conditional expression `tensor.storage.data.ndim if isinstance(tensor, Tensor) el...`.

**CN:** 将 `ndim` and 使用条件表达式 `tensor.storage.data.ndim if isinstance(tensor, Tensor) el...`.

### Block 26 — Lines 98-98 (can_overflow_int32)
```python
98|     shape = tensor.storage.data.shape if isinstance(tensor, Tensor) else tensor.shape
```
**EN:** Assigns `shape` and uses conditional expression `tensor.storage.data.shape if isinstance(tensor, Tensor) e...`.

**CN:** 将 `shape` and 使用条件表达式 `tensor.storage.data.shape if isinstance(tensor, Tensor) e...`.

### Block 27 — Lines 99-99 (can_overflow_int32)
```python
99|     strides = tensor.storage.data.stride() if isinstance(tensor, Tensor) else tensor.stride()
```
**EN:** Assigns `strides` and uses conditional expression `tensor.storage.data.stride() if isinstance(tensor, Tensor...`.

**CN:** 将 `strides` and 使用条件表达式 `tensor.storage.data.stride() if isinstance(tensor, Tensor...`.

### Block 28 — Lines 100-101 (can_overflow_int32)
```python
100|     for i in range(ndim):
101|         offset += (shape[i] - 1) * strides[i]
```
**EN:** Loops over `range(ndim)` with target `i`.

**CN:** 遍历 `range(ndim)` ，目标变量为 `i`.

### Block 29 — Lines 102-104 (can_overflow_int32)
```python
102|     return offset > max_int32
103| 
104| 
```
**EN:** Returns `offset > max_int32`.

**CN:** 返回 `offset > max_int32`.

### Block 30 — Lines 105-105 (should_upcast_indices)
```python
105| def should_upcast_indices(*args):
```
**EN:** Defines function `should_upcast_indices(*args)` for this module. The body mainly returns the computed result. It uses calls such as `any`, `can_overflow_int32` to implement its workflow.

**CN:** 定义函数 `should_upcast_indices(*args)`，供本模块使用. 主体主要返回计算结果. 其中会调用 `any`, `can_overflow_int32` 来实现其工作流程.

### Block 31 — Lines 106-114 (should_upcast_indices)
```python
106|     return any(tensor is not None and can_overflow_int32(tensor) for tensor in args)
107| 
108| 
109| # ---------------------
110| # Numerics
111| # ---------------------
112| 
113| # fmt: off
114| 
```
**EN:** Returns `any((tensor is not None and can_overflow_int32(tensor) for tensor in args))`.

**CN:** 返回 `any((tensor is not None and can_overflow_int32(tensor) for tensor in args))`.

### Block 32 — Lines 115-116 (FlexCtx)
```python
115| @dataclass(frozen=True)
116| class FlexCtx:
```
**EN:** Defines class `FlexCtx` with decorators `dataclass(frozen=True)` to organize related behavior. Key fields include `lhs_data`, `rhs_data`, `out_data`, `acc_data`.

**CN:** 定义类 `FlexCtx`，带有装饰器 `dataclass(frozen=True)`，用于组织相关行为。关键字段包括 `lhs_data`, `rhs_data`, `out_data`, `acc_data`.

### Block 33 — Lines 117-117 (FlexCtx)
```python
117|     lhs_data: InFlexData = InFlexData()
```
**EN:** Annotated assignment stores `lhs_data` and calls `InFlexData`.

**CN:** 带类型注解的赋值保存 `lhs_data`，并调用 `InFlexData`.

### Block 34 — Lines 118-118 (FlexCtx)
```python
118|     rhs_data: InFlexData = InFlexData()
```
**EN:** Annotated assignment stores `rhs_data` and calls `InFlexData`.

**CN:** 带类型注解的赋值保存 `rhs_data`，并调用 `InFlexData`.

### Block 35 — Lines 119-119 (FlexCtx)
```python
119|     out_data: OutFlexData = OutFlexData()
```
**EN:** Annotated assignment stores `out_data` and calls `OutFlexData`.

**CN:** 带类型注解的赋值保存 `out_data`，并调用 `OutFlexData`.

### Block 36 — Lines 120-121 (FlexCtx)
```python
120|     acc_data: InFlexData = InFlexData()
121| 
```
**EN:** Annotated assignment stores `acc_data` and calls `InFlexData`.

**CN:** 带类型注解的赋值保存 `acc_data`，并调用 `InFlexData`.

### Block 37 — Lines 122-123 (PrecisionConfig)
```python
122| @dataclass
123| class PrecisionConfig:
```
**EN:** Defines class `PrecisionConfig` with decorators `dataclass` to organize related behavior. Key fields include `max_num_imprecise_acc`, `allow_tf32`, `flex_ctx`, `acc_scale`, `flexpoint_saturate_inf`, `report_quantization_err_fn`, `a_mx_scale`, `a_microblock_size`.

**CN:** 定义类 `PrecisionConfig`，带有装饰器 `dataclass`，用于组织相关行为。关键字段包括 `max_num_imprecise_acc`, `allow_tf32`, `flex_ctx`, `acc_scale`, `flexpoint_saturate_inf`, `report_quantization_err_fn`, `a_mx_scale`, `a_microblock_size`.

### Block 38 — Lines 124-124 (PrecisionConfig)
```python
124|     max_num_imprecise_acc: int | None = None
```
**EN:** Annotated assignment stores `max_num_imprecise_acc` and stores constant `None`.

**CN:** 带类型注解的赋值保存 `max_num_imprecise_acc` and 保存常量 `None`.

### Block 39 — Lines 125-125 (PrecisionConfig)
```python
125|     allow_tf32: bool = True
```
**EN:** Annotated assignment stores `allow_tf32` and stores constant `True`.

**CN:** 带类型注解的赋值保存 `allow_tf32` and 保存常量 `True`.

### Block 40 — Lines 126-126 (PrecisionConfig)
```python
126|     flex_ctx: FlexCtx = FlexCtx()
```
**EN:** Annotated assignment stores `flex_ctx` and calls `FlexCtx`.

**CN:** 带类型注解的赋值保存 `flex_ctx`，并调用 `FlexCtx`.

### Block 41 — Lines 127-127 (PrecisionConfig)
```python
127|     acc_scale: float = 1.0
```
**EN:** Annotated assignment stores `acc_scale` and stores constant `1.0`.

**CN:** 带类型注解的赋值保存 `acc_scale` and 保存常量 `1.0`.

### Block 42 — Lines 128-128 (PrecisionConfig)
```python
128|     flexpoint_saturate_inf: bool = False
```
**EN:** Annotated assignment stores `flexpoint_saturate_inf` and stores constant `False`.

**CN:** 带类型注解的赋值保存 `flexpoint_saturate_inf` and 保存常量 `False`.

### Block 43 — Lines 129-129 (PrecisionConfig)
```python
129|     report_quantization_err_fn: Callable | None = None
```
**EN:** Annotated assignment stores `report_quantization_err_fn` and stores constant `None`.

**CN:** 带类型注解的赋值保存 `report_quantization_err_fn` and 保存常量 `None`.

### Block 44 — Lines 130-130 (PrecisionConfig)
```python
130|     a_mx_scale: torch.Tensor | Tensor | None = None
```
**EN:** Annotated assignment stores `a_mx_scale` and stores constant `None`.

**CN:** 带类型注解的赋值保存 `a_mx_scale` and 保存常量 `None`.

### Block 45 — Lines 131-131 (PrecisionConfig)
```python
131|     a_microblock_size: int | None = None
```
**EN:** Annotated assignment stores `a_microblock_size` and stores constant `None`.

**CN:** 带类型注解的赋值保存 `a_microblock_size` and 保存常量 `None`.

### Block 46 — Lines 132-132 (PrecisionConfig)
```python
132|     b_mx_scale: torch.Tensor | Tensor | None = None
```
**EN:** Annotated assignment stores `b_mx_scale` and stores constant `None`.

**CN:** 带类型注解的赋值保存 `b_mx_scale` and 保存常量 `None`.

### Block 47 — Lines 133-133 (PrecisionConfig)
```python
133|     b_microblock_size: int | None = None
```
**EN:** Annotated assignment stores `b_microblock_size` and stores constant `None`.

**CN:** 带类型注解的赋值保存 `b_microblock_size` and 保存常量 `None`.

### Block 48 — Lines 134-134 (PrecisionConfig)
```python
134|     c_mx_scale: torch.Tensor | Tensor | None = None
```
**EN:** Annotated assignment stores `c_mx_scale` and stores constant `None`.

**CN:** 带类型注解的赋值保存 `c_mx_scale` and 保存常量 `None`.

### Block 49 — Lines 135-135 (PrecisionConfig)
```python
135|     c_microblock_size: int | None = None
```
**EN:** Annotated assignment stores `c_microblock_size` and stores constant `None`.

**CN:** 带类型注解的赋值保存 `c_microblock_size` and 保存常量 `None`.

### Block 50 — Lines 136-136 (PrecisionConfig)
```python
136|     c_value_pack_factor: int = 1
```
**EN:** Annotated assignment stores `c_value_pack_factor` and stores constant `1`.

**CN:** 带类型注解的赋值保存 `c_value_pack_factor` and 保存常量 `1`.

### Block 51 — Lines 137-137 (PrecisionConfig)
```python
137|     out_dtype: torch.dtype | None = None
```
**EN:** Annotated assignment stores `out_dtype` and stores constant `None`.

**CN:** 带类型注解的赋值保存 `out_dtype` and 保存常量 `None`.

### Block 52 — Lines 138-138 (PrecisionConfig)
```python
138|     intermediate_out_dtype: torch.dtype = torch.float32
```
**EN:** Annotated assignment stores `intermediate_out_dtype` and references `torch.float32`.

**CN:** 带类型注解的赋值保存 `intermediate_out_dtype` and 引用 `torch.float32`.

### Block 53 — Lines 139-141 (PrecisionConfig)
```python
139|     enforce_bitwise_invariance: bool = False
140| 
141| # TODO: merge in opt_flags
```
**EN:** Annotated assignment stores `enforce_bitwise_invariance` and stores constant `False`.

**CN:** 带类型注解的赋值保存 `enforce_bitwise_invariance` and 保存常量 `False`.

### Block 54 — Lines 142-142 (get_swap_xw)
```python
142| def get_swap_xw(precision_config, opt_flags, lhs_dtype, rhs_dtype):
```
**EN:** Defines function `get_swap_xw(precision_config, opt_flags, lhs_dtype, rhs_dtype)` for this module. The body mainly branches on runtime conditions; returns the computed result. It uses calls such as `opt_flags_nvidia.compute_swap_xw`, `triton.runtime.driver.active.get_curr...` to implement its workflow.

**CN:** 定义函数 `get_swap_xw(precision_config, opt_flags, lhs_dtype, rhs_dtype)`，供本模块使用. 主体主要根据运行时条件分支; 返回计算结果. 其中会调用 `opt_flags_nvidia.compute_swap_xw`, `triton.runtime.driver.active.get_curr...` 来实现其工作流程.

### Block 55 — Lines 143-144 (get_swap_xw)
```python
143|     if triton.runtime.driver.active.get_current_target().backend != "cuda":
144|         return False
```
**EN:** Checks `triton.runtime.driver.active.get_current_target().backend != 'cuda'`. The true branch mainly returns the computed result.

**CN:** 检查 `triton.runtime.driver.active.get_current_target().backend != 'cuda'`. 真分支主要返回计算结果.

### Block 56 — Lines 145-150 (get_swap_xw)
```python
145|     return opt_flags_nvidia.compute_swap_xw(precision_config, opt_flags.block_m, opt_flags.is_persistent, lhs_dtype, rhs_dtype)
146| 
147| # ---------------------
148| # Allocation
149| # ---------------------
150| 
```
**EN:** Returns `opt_flags_nvidia.compute_swap_xw(precision_config, opt_flags.block_m, opt_fla...`.

**CN:** 返回 `opt_flags_nvidia.compute_swap_xw(precision_config, opt_flags.block_m, opt_fla...`.

### Block 57 — Lines 151-152 (MatmulAllocation)
```python
151| @dataclass
152| class MatmulAllocation:
```
**EN:** Defines class `MatmulAllocation` with decorators `dataclass` to organize related behavior. Key fields include `device`, `output`, `scratchpads`.

**CN:** 定义类 `MatmulAllocation`，带有装饰器 `dataclass`，用于组织相关行为。关键字段包括 `device`, `output`, `scratchpads`.

### Block 58 — Lines 153-153 (MatmulAllocation)
```python
153|     device: str
```
**EN:** Annotated assignment stores `device` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `device` and 声明该带注解的字段.

### Block 59 — Lines 154-154 (MatmulAllocation)
```python
154|     output: tuple[tuple[int], torch.dtype]
```
**EN:** Annotated assignment stores `output` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `output` and 声明该带注解的字段.

### Block 60 — Lines 155-156 (MatmulAllocation)
```python
155|     scratchpads: dict[str, tuple]
156| 
```
**EN:** Annotated assignment stores `scratchpads` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `scratchpads` and 声明该带注解的字段.

### Block 61 — Lines 157-160 (init_allocation)
```python
157| def init_allocation(x, w, precision_config, fused_activation,
158|                     gather_indx, scatter_indx, batch_dim,
159|                     n_reduce_shards, opt_flags):
160|     # ---- output ------
```
**EN:** Defines function `init_allocation(x, w, precision_config, fused_activation, gather_indx, scatter_indx, batch_dim, n_reduce_shards, opt_flags)` for this module. The body mainly prepares intermediate values; prepares intermediate values; branches on runtime conditions. It uses calls such as `dict`, `MatmulAllocation`, `dtype_to_torch_dtype`, `isinstance`, `triton.cdiv` to implement its workflow.

**CN:** 定义函数 `init_allocation(x, w, precision_config, fused_activation, gather_indx, scatter_indx, batch_dim, n_reduce_shards, opt_flags)`，供本模块使用. 主体主要准备中间值; 准备中间值; 根据运行时条件分支. 其中会调用 `dict`, `MatmulAllocation`, `dtype_to_torch_dtype`, `isinstance`, `triton.cdiv` 来实现其工作流程.

### Block 62 — Lines 161-162 (init_allocation)
```python
161|     N = w.shape[-1]
162|     # by default - M is number of rows in the activations
```
**EN:** Assigns `N` and evaluates `w.shape[-1]`.

**CN:** 将 `N` and 计算 `w.shape[-1]`.

### Block 63 — Lines 163-164 (init_allocation)
```python
163|     M = x.shape[-2]
164|     # if the activations are gathered, then M is number of gather indices
```
**EN:** Assigns `M` and evaluates `x.shape[-2]`.

**CN:** 将 `M` and 计算 `x.shape[-2]`.

### Block 64 — Lines 165-166 (init_allocation)
```python
165|     if gather_indx is not None:
166|         M = gather_indx.shape[0]
```
**EN:** Checks `gather_indx is not None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `gather_indx is not None`. 真分支主要准备中间值.

### Block 65 — Lines 167-168 (init_allocation)
```python
167|     if scatter_indx is not None:
168|         M = scatter_indx.shape[0]
```
**EN:** Checks `scatter_indx is not None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `scatter_indx is not None`. 真分支主要准备中间值.

### Block 66 — Lines 169-169 (init_allocation)
```python
169|     y_rows = M
```
**EN:** Assigns `y_rows` and references `M`.

**CN:** 将 `y_rows` and 引用 `M`.

### Block 67 — Lines 170-170 (init_allocation)
```python
170|     y_rows *= n_reduce_shards
```
**EN:** Updates `y_rows` with operator `Mult` using `n_reduce_shards`.

**CN:** 更新 `y_rows`，使用运算符 `Mult`，并使用 `n_reduce_shards`.

### Block 68 — Lines 171-171 (init_allocation)
```python
171|     out_shape = (batch_dim, y_rows, N // fused_activation.specs.reduction_n)
```
**EN:** Assigns `out_shape` and builds a tuple.

**CN:** 将 `out_shape` and 构造一个元组.

### Block 69 — Lines 172-172 (init_allocation)
```python
172|     out_dtype = precision_config.out_dtype or x.dtype
```
**EN:** Assigns `out_dtype` and evaluates `precision_config.out_dtype or x.dtype`.

**CN:** 将 `out_dtype` and 计算 `precision_config.out_dtype or x.dtype`.

### Block 70 — Lines 173-173 (init_allocation)
```python
173|     out_shape = out_shape[:-1] + (out_shape[-1] // precision_config.c_value_pack_factor, )
```
**EN:** Assigns `out_shape` and evaluates `out_shape[:-1] + (out_shape[-1] // precision_config.c_value_pack_fa...`.

**CN:** 将 `out_shape` and 计算 `out_shape[:-1] + (out_shape[-1] // precision_config.c_value_pack_fa...`.

### Block 71 — Lines 174-175 (init_allocation)
```python
174|     output = (out_shape, out_dtype)
175|     # ---- scratchpad -----#
```
**EN:** Assigns `output` and builds a tuple.

**CN:** 将 `output` and 构造一个元组.

### Block 72 — Lines 176-176 (init_allocation)
```python
176|     scratchpad = dict()
```
**EN:** Assigns `scratchpad` and calls `dict`.

**CN:** 将 `scratchpad`，并调用 `dict`.

### Block 73 — Lines 177-177 (init_allocation)
```python
177|     N_scratch = N // fused_activation.specs.reduction_n if opt_flags.split_k == 1 else N
```
**EN:** Assigns `N_scratch` and uses conditional expression `N // fused_activation.specs.reduction_n if opt_flags.spli...`.

**CN:** 将 `N_scratch` and 使用条件表达式 `N // fused_activation.specs.reduction_n if opt_flags.spli...`.

### Block 74 — Lines 178-180 (init_allocation)
```python
178|     if opt_flags.split_k > 1:
179|         scratch_out_dtype = dtype_to_torch_dtype(precision_config.intermediate_out_dtype)
180|         scratchpad["matmul"] = ((opt_flags.split_k, batch_dim, M, N_scratch), scratch_out_dtype)
```
**EN:** Checks `opt_flags.split_k > 1`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `opt_flags.split_k > 1`. 真分支主要准备中间值; 准备中间值.

### Block 75 — Lines 181-187 (init_allocation)
```python
181|     if "matmul" in scratchpad and precision_config.c_mx_scale is not None:
182|         assert batch_dim == 1, "batch_dim > 1 not supported yet"
183|         scale_dtype = precision_config.c_mx_scale.storage.data.dtype if isinstance(precision_config.c_mx_scale, Tensor) else precision_config.c_mx_scale.dtype
184|         scratchpad["mx_c_mx_scale"] = (
185|             (opt_flags.split_k, 1, M, triton.cdiv(N_scratch, precision_config.c_microblock_size)),
186|             scale_dtype,
187|         )
```
**EN:** Checks `'matmul' in scratchpad and precision_config.c_mx_scale is not None`. The true branch mainly checks invariants; prepares intermediate values.

**CN:** 检查 `'matmul' in scratchpad and precision_config.c_mx_scale is not None`. 真分支主要检查不变量; 准备中间值.

### Block 76 — Lines 188-189 (init_allocation)
```python
188|     return MatmulAllocation(x.device, output, scratchpad)
189| 
```
**EN:** Returns `MatmulAllocation(x.device, output, scratchpad)`.

**CN:** 返回 `MatmulAllocation(x.device, output, scratchpad)`.

### Block 77 — Lines 190-190 (apply_allocation)
```python
190| def apply_allocation(allocation: MatmulAllocation, output):
```
**EN:** Defines function `apply_allocation(allocation, output)` for this module. The body mainly prepares intermediate values; prepares intermediate values; branches on runtime conditions. It uses calls such as `dtype_to_torch_dtype`, `dict`, `torch.empty`, `allocation.scratchpads.items` to implement its workflow.

**CN:** 定义函数 `apply_allocation(allocation, output)`，供本模块使用. 主体主要准备中间值; 准备中间值; 根据运行时条件分支. 其中会调用 `dtype_to_torch_dtype`, `dict`, `torch.empty`, `allocation.scratchpads.items` 来实现其工作流程.

### Block 78 — Lines 191-191 (apply_allocation)
```python
191|     dtype = dtype_to_torch_dtype(allocation.output[1])
```
**EN:** Assigns `dtype` and calls `dtype_to_torch_dtype`.

**CN:** 将 `dtype`，并调用 `dtype_to_torch_dtype`.

### Block 79 — Lines 192-192 (apply_allocation)
```python
192|     ret = dict()
```
**EN:** Assigns `ret` and calls `dict`.

**CN:** 将 `ret`，并调用 `dict`.

### Block 80 — Lines 193-198 (apply_allocation)
```python
193|     if output is None:
194|         output = torch.empty(allocation.output[0], device=allocation.device, dtype=dtype)
195|     else:
196|         if output.ndim == 2:
197|             output = output[None, :, :]
198|         assert output.shape == allocation.output[0]
```
**EN:** Checks `output is None`. The true branch mainly prepares intermediate values, while the else branch branches on runtime conditions; checks invariants.

**CN:** 检查 `output is None`. 真分支主要准备中间值；而 else 分支根据运行时条件分支; 检查不变量.

### Block 81 — Lines 199-199 (apply_allocation)
```python
199|     ret["output"] = output[None, :, :]
```
**EN:** Assigns `ret['output']` and evaluates `output[None, :, :]`.

**CN:** 将 `ret['output']` and 计算 `output[None, :, :]`.

### Block 82 — Lines 200-203 (apply_allocation)
```python
200|     ret["scratchpad"] = {
201|         k: torch.empty(v[0], device=allocation.device, dtype=v[1])
202|             for k, v in allocation.scratchpads.items()
203|     }
```
**EN:** Assigns `ret['scratchpad']` and evaluates `{k: torch.empty(v[0], device=allocation.device, dtype=v[1]) for k, ...`.

**CN:** 将 `ret['scratchpad']` and 计算 `{k: torch.empty(v[0], device=allocation.device, dtype=v[1]) for k, ...`.

### Block 83 — Lines 204-211 (apply_allocation)
```python
204|     return ret
205| 
206| # -----------------------------------------------------------------------------
207| # Canonicalize
208| # -----------------------------------------------------------------------------
209| # the `matmul` kernel can operate on 2D or 3D inputs depending on the mode being used
210| # we can canonicalize storages to make the implementation more uniform
211| 
```
**EN:** Returns `ret`.

**CN:** 返回 `ret`.

### Block 84 — Lines 212-212 (_canonicalize_storage)
```python
212| def _canonicalize_storage(storage, out_ndim, flex_data):
```
**EN:** Defines function `_canonicalize_storage(storage, out_ndim, flex_data)` for this module. The body mainly checks invariants; prepares intermediate values; prepares intermediate values. It uses calls such as `storage.data.as_strided`, `Storage`, `list`, `flex_data.reinterpret`, `storage.data.stride` to implement its workflow.

**CN:** 定义函数 `_canonicalize_storage(storage, out_ndim, flex_data)`，供本模块使用. 主体主要检查不变量; 准备中间值; 准备中间值. 其中会调用 `storage.data.as_strided`, `Storage`, `list`, `flex_data.reinterpret`, `storage.data.stride` 来实现其工作流程.

### Block 85 — Lines 213-221 (_canonicalize_storage)
```python
213|     assert out_ndim >= storage.data.ndim
214|     # Need to use as_strided instead of view because for a tensor with
215|     # shape[-2] == 1 can have ambuiguity related to col-wise. Fo example,
216|     # > t = torch.randn(2, 5, 1).mT
217|     # > t_view = t.view(t.shape)
218|     # > t.stride(), t_view.stride()
219|     # ((5, 1, 1), (5, 5, 1))
220|     # Our check t_view is col-wise fails since t_view.stride(-2) != 1
221|     # This case is covered by (m, n, k) == (1000, 700, 2) in test_matmul.py
```
**EN:** Asserts `out_ndim >= storage.data.ndim` to enforce invariants.

**CN:** 断言 `out_ndim >= storage.data.ndim` 以确保不变量成立。

### Block 86 — Lines 222-222 (_canonicalize_storage)
```python
222|     new_storage_shape = [1] * (out_ndim - storage.data.ndim) + list(storage.data.shape)
```
**EN:** Assigns `new_storage_shape` and evaluates `[1] * (out_ndim - storage.data.ndim) + list(storage.data.shape)`.

**CN:** 将 `new_storage_shape` and 计算 `[1] * (out_ndim - storage.data.ndim) + list(storage.data.shape)`.

### Block 87 — Lines 223-223 (_canonicalize_storage)
```python
223|     new_storage_stride = [0] * (out_ndim - storage.data.ndim) + list(storage.data.stride())
```
**EN:** Assigns `new_storage_stride` and evaluates `[0] * (out_ndim - storage.data.ndim) + list(storage.data.stride())`.

**CN:** 将 `new_storage_stride` and 计算 `[0] * (out_ndim - storage.data.ndim) + list(storage.data.stride())`.

### Block 88 — Lines 224-224 (_canonicalize_storage)
```python
224|     new_storage_data = storage.data.as_strided(new_storage_shape, new_storage_stride)
```
**EN:** Assigns `new_storage_data` and calls `storage.data.as_strided`.

**CN:** 将 `new_storage_data`，并调用 `storage.data.as_strided`.

### Block 89 — Lines 225-226 (_canonicalize_storage)
```python
225|     if flex_data is not None:
226|         new_storage_data = flex_data.reinterpret(new_storage_data)
```
**EN:** Checks `flex_data is not None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `flex_data is not None`. 真分支主要准备中间值.

### Block 90 — Lines 227-233 (_canonicalize_storage)
```python
227|     return Storage(new_storage_data, storage.layout)
228| 
229| 
230| # -----------------------------------------------------------------------------
231| # Triton Implementation
232| # -----------------------------------------------------------------------------
233| 
```
**EN:** Returns `Storage(new_storage_data, storage.layout)`.

**CN:** 返回 `Storage(new_storage_data, storage.layout)`.

### Block 91 — Lines 234-234 (matmul_set_idle_sms)
```python
234| def matmul_set_idle_sms(num_idle_sms):
```
**EN:** Defines function `matmul_set_idle_sms(num_idle_sms)` for this module. The body mainly invokes `update_opt_flags_constraints`. It uses calls such as `update_opt_flags_constraints` to implement its workflow.

**CN:** 定义函数 `matmul_set_idle_sms(num_idle_sms)`，供本模块使用. 主体主要invokes `update_opt_flags_constraints`. 其中会调用 `update_opt_flags_constraints` 来实现其工作流程.

### Block 92 — Lines 235-237 (matmul_set_idle_sms)
```python
235|     """
236|     persistent kernels will leave `num_idle_sms` idle
237|     """
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 93 — Lines 238-239 (matmul_set_idle_sms)
```python
238|     update_opt_flags_constraints({"idle_sms": num_idle_sms})
239| 
```
**EN:** Calls `update_opt_flags_constraints` for side effects, registration, or validation.

**CN:** 调用 `update_opt_flags_constraints` ，用于副作用、注册或校验。

### Block 94 — Lines 240-254 (matmul)
```python
240| def matmul(a, b, bias,
241|     a_ragged_metadata: RaggedTensorMetadata | None = None,
242|     b_ragged_metadata: RaggedTensorMetadata | None = None,
243|     gather_indx: torch.Tensor | None = None,
244|     scatter_indx: torch.Tensor | None = None,
245|     precision_config: PrecisionConfig | None = None,
246|     betas: torch.Tensor | None = None,
247|     gammas: torch.Tensor | None = None,
248|     out_alpha: float | None = None,
249|     c: torch.Tensor | None = None,
250|     fused_comm: FusedComm | None = None,
251|     fused_activation: FusedActivation | None = None,
252|     epilogue: Epilogue | None = None,
253|     c_acc_in: torch.Tensor | None = None,
254| ):
```
**EN:** Defines function `matmul(a, b, bias, a_ragged_metadata, b_ragged_metadata, gather_indx, scatter_indx, precision_config, betas, gammas, out_alpha, c, fused_comm, fused_activation, epilogue, c_acc_in)` for this module. The body mainly prepares intermediate values; branches on runtime conditions; branches on runtime conditions. It uses calls such as `isinstance`, `all`, `torch_dtype_to_dtype`, `make_opt_flags`, `FusedActivation` to implement its workflow.

**CN:** 定义函数 `matmul(a, b, bias, a_ragged_metadata, b_ragged_metadata, gather_indx, scatter_indx, precision_config, betas, gammas, out_alpha, c, fused_comm, fused_activation, epilogue, c_acc_in)`，供本模块使用. 主体主要准备中间值; 根据运行时条件分支; 根据运行时条件分支. 其中会调用 `isinstance`, `all`, `torch_dtype_to_dtype`, `make_opt_flags`, `FusedActivation` 来实现其工作流程.

### Block 95 — Lines 255-265 (matmul)
```python
255|     """
256|     Y[:, :] = 0.
257|     for e in num_experts:
258|         Y[idxs_y_m(e), :] += matmul(X[idxs_x_m(e), :], W[e, :, :])
259| 
260|     matmul can be optionally fused with all gather or scatter at the end for the output. When fused_comm is specified, the m-th row of the output will be stored to (m * n_reduce_shards + reduce_rank) -th row
261|     of each rank id in range [scatter_shard_indx[m] * n_reduce_shards, (scatter_shard_indx[m] + 1) * n_reduce_shards) if scatter_shard_indx is not None, otherwise the output will be all gathered across all reduce ranks.
262|     When scatter_shard_indx is specified, the caller should ensure that the indices of different shards do not conflict.
263| 
264|     The output buffer for fused comm should be pre-allocated and passed in via fused_comm.out_handles, which contains ipc handles to the output tensors, each with shape (n_rows * n_reduce_shards, n_cols).
265|     """
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 96 — Lines 266-266 (matmul)
```python
266|     is_input_batched = a.ndim == 3
```
**EN:** Assigns `is_input_batched` and evaluates `a.ndim == 3`.

**CN:** 将 `is_input_batched` and 计算 `a.ndim == 3`.

### Block 97 — Lines 267-273 (matmul)
```python
267|     if is_input_batched:
268|         assert gather_indx is None, "gather not supported in batched mode"
269|         assert scatter_indx is None, "scatter not supported in batched mode"
270|         assert b_ragged_metadata is None, "w cannot be ragged in batched mode"
271|         assert a_ragged_metadata is None, "x cannot be ragged in batched mode"
272|         assert fused_comm is None, "fused comm is not supported in batched mode"
273|         assert b.ndim == 3 and b.shape[0] == a.shape[0]
```
**EN:** Checks `is_input_batched`. The true branch mainly checks invariants; checks invariants.

**CN:** 检查 `is_input_batched`. 真分支主要检查不变量; 检查不变量.

### Block 98 — Lines 274-277 (matmul)
```python
274|     if b_ragged_metadata is not None:
275|         assert gather_indx is None
276|         assert scatter_indx is None
277|     # canonicalize inputs
```
**EN:** Checks `b_ragged_metadata is not None`. The true branch mainly checks invariants; checks invariants.

**CN:** 检查 `b_ragged_metadata is not None`. 真分支主要检查不变量; 检查不变量.

### Block 99 — Lines 278-279 (matmul)
```python
278|     if precision_config is None:
279|         precision_config = PrecisionConfig()
```
**EN:** Checks `precision_config is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `precision_config is None`. 真分支主要准备中间值.

### Block 100 — Lines 280-281 (matmul)
```python
280|     if fused_activation is None:
281|         fused_activation = FusedActivation(FnSpecs.default(), tuple())
```
**EN:** Checks `fused_activation is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `fused_activation is None`. 真分支主要准备中间值.

### Block 101 — Lines 282-283 (matmul)
```python
282|     if epilogue is None:
283|         epilogue = Epilogue(FnSpecs.default(), tuple(), tuple(), False)
```
**EN:** Checks `epilogue is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `epilogue is None`. 真分支主要准备中间值.

### Block 102 — Lines 284-285 (matmul)
```python
284|     if fused_comm is not None and precision_config.c_mx_scale is not None:
285|         raise NotImplementedError("fused comm with output MX scales is not supported")
```
**EN:** Checks `fused_comm is not None and precision_config.c_mx_scale is not None`..

**CN:** 检查 `fused_comm is not None and precision_config.c_mx_scale is not None`..

### Block 103 — Lines 286-287 (matmul)
```python
286|     n_slices = max(1, b.shape[0]) if a_ragged_metadata is None else a_ragged_metadata.n_slices
287|     # unpack b scale
```
**EN:** Assigns `n_slices` and uses conditional expression `max(1, b.shape[0]) if a_ragged_metadata is None else a_ra...`.

**CN:** 将 `n_slices` and 使用条件表达式 `max(1, b.shape[0]) if a_ragged_metadata is None else a_ra...`.

### Block 104 — Lines 288-288 (matmul)
```python
288|     b_scale = precision_config.b_mx_scale
```
**EN:** Assigns `b_scale` and references `precision_config.b_mx_scale`.

**CN:** 将 `b_scale` and 引用 `precision_config.b_mx_scale`.

### Block 105 — Lines 289-289 (matmul)
```python
289|     b_has_mx = b_scale is not None
```
**EN:** Assigns `b_has_mx` and evaluates `b_scale is not None`.

**CN:** 将 `b_has_mx` and 计算 `b_scale is not None`.

### Block 106 — Lines 290-292 (matmul)
```python
290|     if not isinstance(b, Tensor):
291|         dtype = FP4 if b.dtype == torch.uint8 else None
292|         b = wrap_torch_tensor(b, dtype=dtype)
```
**EN:** Checks `not isinstance(b, Tensor)`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `not isinstance(b, Tensor)`. 真分支主要准备中间值; 准备中间值.

### Block 107 — Lines 293-293 (matmul)
```python
293|     b_is_shuffled = isinstance(b.storage.layout, BlackwellMX4ValueShuffledLayout)
```
**EN:** Assigns `b_is_shuffled` and calls `isinstance`.

**CN:** 将 `b_is_shuffled`，并调用 `isinstance`.

### Block 108 — Lines 294-295 (matmul)
```python
294|     if b_scale is not None and not isinstance(b_scale, Tensor):
295|         b_scale = wrap_torch_tensor(b_scale)
```
**EN:** Checks `b_scale is not None and (not isinstance(b_scale, Tensor))`. The true branch mainly prepares intermediate values.

**CN:** 检查 `b_scale is not None and (not isinstance(b_scale, Tensor))`. 真分支主要准备中间值.

### Block 109 — Lines 296-296 (matmul)
```python
296|     b_scale_layout = None if b_scale is None else b_scale.storage.layout
```
**EN:** Assigns `b_scale_layout` and uses conditional expression `None if b_scale is None else b_scale.storage.layout`.

**CN:** 将 `b_scale_layout` and 使用条件表达式 `None if b_scale is None else b_scale.storage.layout`.

### Block 110 — Lines 297-301 (matmul)
```python
297|     if b_has_mx and (
298|             b.storage.layout is not None and not isinstance(b.storage.layout, StridedLayout)
299|             or isinstance(b_scale_layout, HopperMXScaleLayout)):
300|         if not b_is_shuffled:
301|             assert b.stride(-2) == 1, "`w` must be column-major with Hopper-swizzled MX scales or non-strided MX value layouts"
```
**EN:** Checks `b_has_mx and (b.storage.layout is not None and (not isinstance(b.storage.layout, Stride...`. The true branch mainly branches on runtime conditions.

**CN:** 检查 `b_has_mx and (b.storage.layout is not None and (not isinstance(b.storage.layout, Stride...`. 真分支主要根据运行时条件分支.

### Block 111 — Lines 302-302 (matmul)
```python
302|     is_hopper_fp8 = is_cuda() and not target_info.cuda_capability_geq(10, 0) and b.dtype.bitwidth == 8
```
**EN:** Assigns `is_hopper_fp8` and evaluates `is_cuda() and (not target_info.cuda_capability_geq(10, 0)) and (b.d...`.

**CN:** 将 `is_hopper_fp8` and 计算 `is_cuda() and (not target_info.cuda_capability_geq(10, 0)) and (b.d...`.

### Block 112 — Lines 303-304 (matmul)
```python
303|     if is_hopper_fp8: assert b.stride(-2) == 1, "`w` must be column-major when it has data-type FP8 on capability < 10"
304|     # unpack a scale
```
**EN:** Checks `is_hopper_fp8`. The true branch mainly checks invariants.

**CN:** 检查 `is_hopper_fp8`. 真分支主要检查不变量.

### Block 113 — Lines 305-305 (matmul)
```python
305|     a_scale = precision_config.a_mx_scale
```
**EN:** Assigns `a_scale` and references `precision_config.a_mx_scale`.

**CN:** 将 `a_scale` and 引用 `precision_config.a_mx_scale`.

### Block 114 — Lines 306-306 (matmul)
```python
306|     a_has_mx = a_scale is not None
```
**EN:** Assigns `a_has_mx` and evaluates `a_scale is not None`.

**CN:** 将 `a_has_mx` and 计算 `a_scale is not None`.

### Block 115 — Lines 307-307 (matmul)
```python
307|     if a_has_mx: assert a.stride(-1) == 1, "'x' must be row-major when it has data-type mxfp"
```
**EN:** Checks `a_has_mx`. The true branch mainly checks invariants.

**CN:** 检查 `a_has_mx`. 真分支主要检查不变量.

### Block 116 — Lines 308-309 (matmul)
```python
308|     if a_scale is not None and not isinstance(a_scale, Tensor):
309|         a_scale = wrap_torch_tensor(a_scale)
```
**EN:** Checks `a_scale is not None and (not isinstance(a_scale, Tensor))`. The true branch mainly prepares intermediate values.

**CN:** 检查 `a_scale is not None and (not isinstance(a_scale, Tensor))`. 真分支主要准备中间值.

### Block 117 — Lines 310-312 (matmul)
```python
310|     if not isinstance(a, Tensor):
311|         dtype = FP4 if a_has_mx and a.dtype == torch.uint8 else None
312|         a = wrap_torch_tensor(a, dtype=dtype)
```
**EN:** Checks `not isinstance(a, Tensor)`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `not isinstance(a, Tensor)`. 真分支主要准备中间值; 准备中间值.

### Block 118 — Lines 313-313 (matmul)
```python
313|     a_scale_dtype = None if a_scale is None else a_scale.storage.data.dtype
```
**EN:** Assigns `a_scale_dtype` and uses conditional expression `None if a_scale is None else a_scale.storage.data.dtype`.

**CN:** 将 `a_scale_dtype` and 使用条件表达式 `None if a_scale is None else a_scale.storage.data.dtype`.

### Block 119 — Lines 314-315 (matmul)
```python
314|     b_scale_dtype = None if b_scale is None else b_scale.storage.data.dtype
315|     # NOTE: uint8 scale means OCP E8M0 here. Direct NVFP-style scales stay float8_e4m3fn.
```
**EN:** Assigns `b_scale_dtype` and uses conditional expression `None if b_scale is None else b_scale.storage.data.dtype`.

**CN:** 将 `b_scale_dtype` and 使用条件表达式 `None if b_scale is None else b_scale.storage.data.dtype`.

### Block 120 — Lines 316-319 (matmul)
```python
316|     if a_scale_dtype is not None:
317|         assert a_scale_dtype == torch.uint8 or a_scale_dtype == torch.float8_e4m3fn, (
318|             f"Unsupported microscale dtype {a_scale_dtype}"
319|         )
```
**EN:** Checks `a_scale_dtype is not None`. The true branch mainly checks invariants.

**CN:** 检查 `a_scale_dtype is not None`. 真分支主要检查不变量.

### Block 121 — Lines 320-320 (matmul)
```python
320|     a_microblock_size = precision_config.a_microblock_size
```
**EN:** Assigns `a_microblock_size` and references `precision_config.a_microblock_size`.

**CN:** 将 `a_microblock_size` and 引用 `precision_config.a_microblock_size`.

### Block 122 — Lines 321-324 (matmul)
```python
321|     if a_microblock_size is not None:
322|         assert a_microblock_size == int(MXFP_BLOCK_SIZE) or a_microblock_size == int(NVFP_BLOCK_SIZE), (
323|             f"Unsupported microscale block size {a_microblock_size}"
324|         )
```
**EN:** Checks `a_microblock_size is not None`. The true branch mainly checks invariants.

**CN:** 检查 `a_microblock_size is not None`. 真分支主要检查不变量.

### Block 123 — Lines 325-327 (matmul)
```python
325|     assert a_scale is None or a_microblock_size is not None, (
326|         "precision_config.a_microblock_size is required when precision_config.a_mx_scale is set"
327|     )
```
**EN:** Asserts `a_scale is None or a_microblock_size is not None` to enforce invariants.

**CN:** 断言 `a_scale is None or a_microblock_size is not None` 以确保不变量成立。

### Block 124 — Lines 328-331 (matmul)
```python
328|     if b_scale_dtype is not None:
329|         assert b_scale_dtype == torch.uint8 or b_scale_dtype == torch.float8_e4m3fn, (
330|             f"Unsupported microscale dtype {b_scale_dtype}"
331|         )
```
**EN:** Checks `b_scale_dtype is not None`. The true branch mainly checks invariants.

**CN:** 检查 `b_scale_dtype is not None`. 真分支主要检查不变量.

### Block 125 — Lines 332-332 (matmul)
```python
332|     b_microblock_size = precision_config.b_microblock_size
```
**EN:** Assigns `b_microblock_size` and references `precision_config.b_microblock_size`.

**CN:** 将 `b_microblock_size` and 引用 `precision_config.b_microblock_size`.

### Block 126 — Lines 333-336 (matmul)
```python
333|     if b_microblock_size is not None:
334|         assert b_microblock_size == int(MXFP_BLOCK_SIZE) or b_microblock_size == int(NVFP_BLOCK_SIZE), (
335|             f"Unsupported microscale block size {b_microblock_size}"
336|         )
```
**EN:** Checks `b_microblock_size is not None`. The true branch mainly checks invariants.

**CN:** 检查 `b_microblock_size is not None`. 真分支主要检查不变量.

### Block 127 — Lines 337-339 (matmul)
```python
337|     assert b_scale is None or b_microblock_size is not None, (
338|         "precision_config.b_microblock_size is required when precision_config.b_mx_scale is set"
339|     )
```
**EN:** Asserts `b_scale is None or b_microblock_size is not None` to enforce invariants.

**CN:** 断言 `b_scale is None or b_microblock_size is not None` 以确保不变量成立。

### Block 128 — Lines 340-343 (matmul)
```python
340|     if a_microblock_size is not None and b_microblock_size is not None:
341|         assert a_microblock_size == b_microblock_size, (
342|             f"Microscaled operands must share a block size. Got {a_microblock_size} and {b_microblock_size}"
343|         )
```
**EN:** Checks `a_microblock_size is not None and b_microblock_size is not None`. The true branch mainly checks invariants.

**CN:** 检查 `a_microblock_size is not None and b_microblock_size is not None`. 真分支主要检查不变量.

### Block 129 — Lines 344-346 (matmul)
```python
344|     mx_block_size = (
345|         a_microblock_size or b_microblock_size or precision_config.c_microblock_size or int(MXFP_BLOCK_SIZE)
346|     )
```
**EN:** Assigns `mx_block_size` and evaluates `a_microblock_size or b_microblock_size or precision_config.c_microb...`.

**CN:** 将 `mx_block_size` and 计算 `a_microblock_size or b_microblock_size or precision_config.c_microb...`.

### Block 130 — Lines 347-353 (matmul)
```python
347|     assert all(
348|         size is None or size == mx_block_size
349|         for size in (a_microblock_size, b_microblock_size, precision_config.c_microblock_size)
350|     ), (
351|         "Microscaled operands/output must share a block size. "
352|         f"Got a={a_microblock_size}, b={b_microblock_size}, c={precision_config.c_microblock_size}"
353|     )
```
**EN:** Asserts `all((size is None or size == mx_block_size for size in (a_microblock_size, b_microblock...` to enforce invariants.

**CN:** 断言 `all((size is None or size == mx_block_size for size in (a_microblock_size, b_microblock...` 以确保不变量成立。

### Block 131 — Lines 354-355 (matmul)
```python
354|     if precision_config.c_mx_scale is not None and precision_config.c_microblock_size is None:
355|         precision_config.c_microblock_size = mx_block_size
```
**EN:** Checks `precision_config.c_mx_scale is not None and precision_config.c_microblock_size is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `precision_config.c_mx_scale is not None and precision_config.c_microblock_size is None`. 真分支主要准备中间值.

### Block 132 — Lines 356-359 (matmul)
```python
356|     precision_config.c_value_pack_factor = 2 if precision_config.c_mx_scale is not None and epilogue.specs.name in (
357|         FnName.QUANTIZE_MXFP4.name,
358|         FnName.QUANTIZE_NVFP4.name,
359|     ) else 1
```
**EN:** Assigns `precision_config.c_value_pack_factor` and uses conditional expression `2 if precision_config.c_mx_scale is not None and epilogue...`.

**CN:** 将 `precision_config.c_value_pack_factor` and 使用条件表达式 `2 if precision_config.c_mx_scale is not None and epilogue...`.

### Block 133 — Lines 360-361 (matmul)
```python
360|     a_transpose = a.stride(-1) != 1
361|     # determine shapes
```
**EN:** Assigns `a_transpose` and evaluates `a.stride(-1) != 1`.

**CN:** 将 `a_transpose` and 计算 `a.stride(-1) != 1`.

### Block 134 — Lines 362-362 (matmul)
```python
362|     has_gather = gather_indx is not None
```
**EN:** Assigns `has_gather` and evaluates `gather_indx is not None`.

**CN:** 将 `has_gather` and 计算 `gather_indx is not None`.

### Block 135 — Lines 363-363 (matmul)
```python
363|     has_scatter = scatter_indx is not None
```
**EN:** Assigns `has_scatter` and evaluates `scatter_indx is not None`.

**CN:** 将 `has_scatter` and 计算 `scatter_indx is not None`.

### Block 136 — Lines 364-364 (matmul)
```python
364|     is_a_ragged = a_ragged_metadata is not None
```
**EN:** Assigns `is_a_ragged` and evaluates `a_ragged_metadata is not None`.

**CN:** 将 `is_a_ragged` and 计算 `a_ragged_metadata is not None`.

### Block 137 — Lines 365-365 (matmul)
```python
365|     is_b_ragged = b_ragged_metadata is not None
```
**EN:** Assigns `is_b_ragged` and evaluates `b_ragged_metadata is not None`.

**CN:** 将 `is_b_ragged` and 计算 `b_ragged_metadata is not None`.

### Block 138 — Lines 366-366 (matmul)
```python
366|     is_c_ragged = is_a_ragged and b_ragged_metadata is None
```
**EN:** Assigns `is_c_ragged` and evaluates `is_a_ragged and b_ragged_metadata is None`.

**CN:** 将 `is_c_ragged` and 计算 `is_a_ragged and b_ragged_metadata is None`.

### Block 139 — Lines 367-367 (matmul)
```python
367|     ragged_dimension = "K" if is_b_ragged else "M" if is_a_ragged else None
```
**EN:** Assigns `ragged_dimension` and uses conditional expression `'K' if is_b_ragged else 'M' if is_a_ragged else None`.

**CN:** 将 `ragged_dimension` and 使用条件表达式 `'K' if is_b_ragged else 'M' if is_a_ragged else None`.

### Block 140 — Lines 368-368 (matmul)
```python
368|     M = a.shape[-2] if gather_indx is None else gather_indx.shape[0]
```
**EN:** Assigns `M` and uses conditional expression `a.shape[-2] if gather_indx is None else gather_indx.shape[0]`.

**CN:** 将 `M` and 使用条件表达式 `a.shape[-2] if gather_indx is None else gather_indx.shape[0]`.

### Block 141 — Lines 369-374 (matmul)
```python
369|     if ragged_dimension == "K":
370|         batch_size = b_ragged_metadata.n_slices
371|     elif ragged_dimension is None and b.ndim == 3:
372|         batch_size = b.shape[0]
373|     else:
374|         batch_size = 1
```
**EN:** Checks `ragged_dimension == 'K'`. The true branch mainly prepares intermediate values, while the else branch branches on runtime conditions.

**CN:** 检查 `ragged_dimension == 'K'`. 真分支主要准备中间值；而 else 分支根据运行时条件分支.

### Block 142 — Lines 375-378 (matmul)
```python
375|     if c_acc_in is not None:
376|         c_acc_is_c = c_acc_in.data_ptr() == c.data_ptr() and c_acc_in.stride() == c.stride()
377|     else:
378|         c_acc_is_c = None
```
**EN:** Checks `c_acc_in is not None`. The true branch mainly prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `c_acc_in is not None`. 真分支主要准备中间值；而 else 分支准备中间值.

### Block 143 — Lines 379-379 (matmul)
```python
379|     K = a.shape[-1]
```
**EN:** Assigns `K` and evaluates `a.shape[-1]`.

**CN:** 将 `K` and 计算 `a.shape[-1]`.

### Block 144 — Lines 380-380 (matmul)
```python
380|     K_W, N = b.shape[-2:]
```
**EN:** Assigns `K_W`, `N` and evaluates `b.shape[-2:]`.

**CN:** 将 `K_W`, `N` and 计算 `b.shape[-2:]`.

### Block 145 — Lines 381-383 (matmul)
```python
381|     if a.ndim == 3 and b.ndim == 3:
382|         assert a.shape[0] == b.shape[0]
383|     # compute optimization flags
```
**EN:** Checks `a.ndim == 3 and b.ndim == 3`. The true branch mainly checks invariants.

**CN:** 检查 `a.ndim == 3 and b.ndim == 3`. 真分支主要检查不变量.

### Block 146 — Lines 384-384 (matmul)
```python
384|     out_dtype = precision_config.out_dtype or a.dtype
```
**EN:** Assigns `out_dtype` and evaluates `precision_config.out_dtype or a.dtype`.

**CN:** 将 `out_dtype` and 计算 `precision_config.out_dtype or a.dtype`.

### Block 147 — Lines 385-385 (matmul)
```python
385|     out_dtype = torch_dtype_to_dtype(out_dtype)
```
**EN:** Assigns `out_dtype` and calls `torch_dtype_to_dtype`.

**CN:** 将 `out_dtype`，并调用 `torch_dtype_to_dtype`.

### Block 148 — Lines 386-387 (matmul)
```python
386|     if out_dtype == UINT8 and precision_config.c_mx_scale is not None:
387|         out_dtype = FP4
```
**EN:** Checks `out_dtype == UINT8 and precision_config.c_mx_scale is not None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `out_dtype == UINT8 and precision_config.c_mx_scale is not None`. 真分支主要准备中间值.

### Block 149 — Lines 388-398 (matmul)
```python
388|     can_use_tma = (
389|         a.numel() > 0 and is_tma_compliant(a) and
390|         b.numel() > 0 and is_tma_compliant(b) and
391|         (b_scale is None or is_tma_compliant(b_scale)) and
392|         (ragged_dimension != "M" or a.stride(-1) == 1) and
393|         # Currently we don't support tma if y is column major; may revisit later if this becomes an issue.
394|         (c is None or c.stride(-1) == 1) and
395|         (c_acc_in is None or c_acc_is_c) and
396|         # if ragged dimension is K, w must be either padded or row major to ensure alignment
397|         (ragged_dimension != "K" or b.stride(-1) == 1 or b_ragged_metadata.slice_sizes_divisibility is not None)
398|     )
```
**EN:** Assigns `can_use_tma` and evaluates `a.numel() > 0 and is_tma_compliant(a) and (b.numel() > 0) and is_tm...`.

**CN:** 将 `can_use_tma` and 计算 `a.numel() > 0 and is_tma_compliant(a) and (b.numel() > 0) and is_tm...`.

### Block 150 — Lines 399-404 (matmul)
```python
399|     if b_scale is not None and isinstance(b_scale.storage.layout, StridedLayout) and b_scale.storage.data.stride()[-1] != 1:
400|         # In this case, we need to transpose b_scale. Then the reduction dim
401|         # becomes the last dim that will be divided by 32. This to be a multiple
402|         # of 16 to be TMA-compliant requires block_k to be a multiple of 512,
403|         # which is too big.
404|         can_use_tma = False
```
**EN:** Checks `b_scale is not None and isinstance(b_scale.storage.layout, StridedLayout) and (b_scale....`. The true branch mainly prepares intermediate values.

**CN:** 检查 `b_scale is not None and isinstance(b_scale.storage.layout, StridedLayout) and (b_scale....`. 真分支主要准备中间值.

### Block 151 — Lines 405-405 (matmul)
```python
405|     has_gather_tma = has_gather and target_info.has_tma_gather()
```
**EN:** Assigns `has_gather_tma` and evaluates `has_gather and target_info.has_tma_gather()`.

**CN:** 将 `has_gather_tma` and 计算 `has_gather and target_info.has_tma_gather()`.

### Block 152 — Lines 406-406 (matmul)
```python
406|     is_ragged_mx = (a_has_mx or b_has_mx) and (is_a_ragged or is_b_ragged)
```
**EN:** Assigns `is_ragged_mx` and evaluates `(a_has_mx or b_has_mx) and (is_a_ragged or is_b_ragged)`.

**CN:** 将 `is_ragged_mx` and 计算 `(a_has_mx or b_has_mx) and (is_a_ragged or is_b_ragged)`.

### Block 153 — Lines 407-407 (matmul)
```python
407|     can_use_split_k = scatter_indx is None and not is_ragged_mx and ragged_dimension != "K" and c_acc_in is None and precision_config.c_mx_scale is None
```
**EN:** Assigns `can_use_split_k` and evaluates `scatter_indx is None and (not is_ragged_mx) and (ragged_dimension !...`.

**CN:** 将 `can_use_split_k` and 计算 `scatter_indx is None and (not is_ragged_mx) and (ragged_dimension !...`.

### Block 154 — Lines 408-408 (matmul)
```python
408|     block_k = None
```
**EN:** Assigns `block_k` and stores constant `None`.

**CN:** 将 `block_k` and 保存常量 `None`.

### Block 155 — Lines 409-413 (matmul)
```python
409|     if ragged_dimension == "K":
410|         block_k = a_ragged_metadata.slice_sizes_divisibility or b_ragged_metadata.slice_sizes_divisibility
411|         a_uses_tma_when_persistent = a.stride(-1) != 1 or (a_ragged_metadata.slice_sizes_divisibility is not None)
412|     else:
413|         a_uses_tma_when_persistent = has_gather_tma or not has_gather
```
**EN:** Checks `ragged_dimension == 'K'`. The true branch mainly prepares intermediate values; prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `ragged_dimension == 'K'`. 真分支主要准备中间值; 准备中间值；而 else 分支准备中间值.

### Block 156 — Lines 414-423 (matmul)
```python
414|     opt_flags = make_opt_flags(out_dtype, a.dtype, b.dtype, precision_config,
415|         batch_size, M, N, b.shape[-2], a_ragged_metadata,
416|         can_use_tma, can_use_split_k, epilogue.effective_itemsize,
417|         a_transpose, c_acc_in is not None,
418|         block_k = block_k,
419|         mx_block_size = mx_block_size,
420|         x_uses_tma_when_persistent = a_uses_tma_when_persistent,
421|         rhs_layout=b.storage.layout,
422|         epilogue_reduction_n=fused_activation.specs.reduction_n,
423|     )
```
**EN:** Assigns `opt_flags` and calls `make_opt_flags`.

**CN:** 将 `opt_flags`，并调用 `make_opt_flags`.

### Block 157 — Lines 424-438 (matmul)
```python
424|     if b_is_shuffled:
425|         if b.dtype.bitwidth != 4:
426|             raise ValueError("Shuffled weights are only supported for mxfp4 values")
427|         if not opt_flags.is_persistent:
428|             raise InapplicableConstraint("Shuffled weights require the persistent TMA kernel")
429|         b_layout = b.storage.layout
430|         if b_layout.block_k != opt_flags.block_k or b_layout.block_n != opt_flags.block_n:
431|             raise ValueError(
432|                 f"Shuffled weight layout uses block_k={b_layout.block_k} and "
433|                 f"block_n={b_layout.block_n}, but kernel selected "
434|                 f"block_k={opt_flags.block_k}, block_n={opt_flags.block_n}. "
435|                 f"Use disable_mx4_block_swap constraint to match the layout."
436|             )
437|     # there seems to be a bug on A100
438|     # pytest -vs test_matmul.py::test_op[False-False-False-False-pad_b-16-768-512-1024-ragged-float16-float16-10-1-False-None-False-False-False-True-None]
```
**EN:** Checks `b_is_shuffled`. The true branch mainly branches on runtime conditions; branches on runtime conditions.

**CN:** 检查 `b_is_shuffled`. 真分支主要根据运行时条件分支; 根据运行时条件分支.

### Block 158 — Lines 439-440 (matmul)
```python
439|     if ragged_dimension == "K" and torch.cuda.get_device_capability()[0] < 9:
440|         opt_flags.num_stages = 1
```
**EN:** Checks `ragged_dimension == 'K' and torch.cuda.get_device_capability()[0] < 9`. The true branch mainly prepares intermediate values.

**CN:** 检查 `ragged_dimension == 'K' and torch.cuda.get_device_capability()[0] < 9`. 真分支主要准备中间值.

### Block 159 — Lines 441-454 (matmul)
```python
441|     if ragged_dimension == "K":
442|         a_has_tma = opt_flags.is_persistent and (a.stride(-1) != 1 or (a_ragged_metadata.slice_sizes_divisibility is not None))
443|         # If TMA is used, limit is handled automatically, so we can pretend K is "even".
444|         # (For unpadded input, we assume that the first block_k unused rows are zero-filled,
445|         # when routing_data.expt_hist.sum() is less than K or K_W.)
446|         if opt_flags.is_persistent:
447|             even_K = a_has_tma or (a_ragged_metadata.slice_sizes_divisibility is not None)
448|         else:
449|             even_K = a_ragged_metadata.slice_sizes_divisibility is not None and b_ragged_metadata.slice_sizes_divisibility is not None
450|     else:
451|         batch_size = b.shape[0] if a_ragged_metadata is None and b.ndim == 3 else 1
452|         assert K == K_W
453|         a_has_tma = opt_flags.is_persistent and (has_gather_tma or not has_gather)
454|         even_K = (K % opt_flags.block_k == 0)
```
**EN:** Checks `ragged_dimension == 'K'`. The true branch mainly prepares intermediate values; branches on runtime conditions, while the else branch prepares intermediate values; checks invariants.

**CN:** 检查 `ragged_dimension == 'K'`. 真分支主要准备中间值; 根据运行时条件分支；而 else 分支准备中间值; 检查不变量.

### Block 160 — Lines 455-457 (matmul)
```python
455|     if b_scale is not None and b_scale.storage.layout.name != "STRIDED" and not opt_flags.is_persistent and target_info.has_native_mxfp():
456|         raise NotImplementedError("Must use persistent kernel and be TMA-compliant for native MXFP")
457|     # fused activation
```
**EN:** Checks `b_scale is not None and b_scale.storage.layout.name != 'STRIDED' and (not opt_flags.is_...`..

**CN:** 检查 `b_scale is not None and b_scale.storage.layout.name != 'STRIDED' and (not opt_flags.is_...`..

### Block 161 — Lines 458-458 (matmul)
```python
458|     matmul_fused_activation = fused_activation
```
**EN:** Assigns `matmul_fused_activation` and references `fused_activation`.

**CN:** 将 `matmul_fused_activation` and 引用 `fused_activation`.

### Block 162 — Lines 459-459 (matmul)
```python
459|     reduce_fused_activation = FusedActivation()
```
**EN:** Assigns `reduce_fused_activation` and calls `FusedActivation`.

**CN:** 将 `reduce_fused_activation`，并调用 `FusedActivation`.

### Block 163 — Lines 460-462 (matmul)
```python
460|     if opt_flags.split_k > 1:
461|         matmul_fused_activation, reduce_fused_activation = reduce_fused_activation, matmul_fused_activation
462|     # allocate output/scratchpad memory
```
**EN:** Checks `opt_flags.split_k > 1`. The true branch mainly prepares intermediate values.

**CN:** 检查 `opt_flags.split_k > 1`. 真分支主要准备中间值.

### Block 164 — Lines 463-466 (matmul)
```python
463|     allocation = init_allocation(a, b, precision_config, fused_activation,
464|                                  gather_indx, scatter_indx, batch_size,
465|                                  fused_comm.n_reduce_shards if fused_comm is not None else 1,
466|                                  opt_flags)
```
**EN:** Assigns `allocation` and calls `init_allocation`.

**CN:** 将 `allocation`，并调用 `init_allocation`.

### Block 165 — Lines 467-468 (matmul)
```python
467|     memory = apply_allocation(allocation, c)
468|     # early exit
```
**EN:** Assigns `memory` and calls `apply_allocation`.

**CN:** 将 `memory`，并调用 `apply_allocation`.

### Block 166 — Lines 469-474 (matmul)
```python
469|     if batch_size * M * N == 0:
470|         ret = memory["output"].squeeze(0)
471|         if not is_input_batched:
472|             ret = ret.squeeze(0)
473|         return ret
474|     # TMA descriptors require a global memory allocation
```
**EN:** Checks `batch_size * M * N == 0`. The true branch mainly prepares intermediate values; branches on runtime conditions.

**CN:** 检查 `batch_size * M * N == 0`. 真分支主要准备中间值; 根据运行时条件分支.

### Block 167 — Lines 475-477 (matmul)
```python
475|     if opt_flags.is_persistent:
476|         triton.set_allocator(get_per_device_per_stream_alloc_fn(a.device))
477|     # Intermediate tensors and postprocess kernels for each situation
```
**EN:** Checks `opt_flags.is_persistent`. The true branch mainly invokes `triton.set_allocator`.

**CN:** 检查 `opt_flags.is_persistent`. 真分支主要invokes `triton.set_allocator`.

### Block 168 — Lines 478-479 (matmul)
```python
478|     has_scratchpad = "matmul" in memory["scratchpad"]
479|     # Canonical output tensor (matmul scratchpad if present, otherwise final output tensor)
```
**EN:** Assigns `has_scratchpad` and evaluates `'matmul' in memory['scratchpad']`.

**CN:** 将 `has_scratchpad` and 计算 `'matmul' in memory['scratchpad']`.

### Block 169 — Lines 480-480 (matmul)
```python
480|     out_matmul = memory["scratchpad"].get("matmul", memory["output"])
```
**EN:** Assigns `out_matmul` and calls `memory['scratchpad'].get`.

**CN:** 将 `out_matmul`，并调用 `memory['scratchpad'].get`.

### Block 170 — Lines 481-482 (matmul)
```python
481|     out_matmul_flex = OutFlexData() if has_scratchpad or out_matmul.dtype == torch.float32 else precision_config.flex_ctx.out_data
482|     # Unified mx-scale pointer; when scratchpad exists, prefer its mx buffer
```
**EN:** Assigns `out_matmul_flex` and uses conditional expression `OutFlexData() if has_scratchpad or out_matmul.dtype == to...`.

**CN:** 将 `out_matmul_flex` and 使用条件表达式 `OutFlexData() if has_scratchpad or out_matmul.dtype == to...`.

### Block 171 — Lines 483-483 (matmul)
```python
483|     out_matmul_scale = precision_config.c_mx_scale
```
**EN:** Assigns `out_matmul_scale` and references `precision_config.c_mx_scale`.

**CN:** 将 `out_matmul_scale` and 引用 `precision_config.c_mx_scale`.

### Block 172 — Lines 484-487 (matmul)
```python
484|     if out_matmul_scale is not None:
485|         out_matmul_scale = out_matmul_scale if isinstance(out_matmul_scale, Tensor) else wrap_torch_tensor(out_matmul_scale)
486|         if has_scratchpad and "mx_c_mx_scale" in memory["scratchpad"]:
487|             out_matmul_scale = wrap_torch_tensor(memory["scratchpad"]["mx_c_mx_scale"])
```
**EN:** Checks `out_matmul_scale is not None`. The true branch mainly prepares intermediate values; branches on runtime conditions.

**CN:** 检查 `out_matmul_scale is not None`. 真分支主要准备中间值; 根据运行时条件分支.

### Block 173 — Lines 488-489 (matmul)
```python
488|     out_matmul_has_mx = out_matmul_scale is not None and out_matmul.element_size() == 1
489|     # matrix multiplication
```
**EN:** Assigns `out_matmul_has_mx` and evaluates `out_matmul_scale is not None and out_matmul.element_size() == 1`.

**CN:** 将 `out_matmul_has_mx` and 计算 `out_matmul_scale is not None and out_matmul.element_size() == 1`.

### Block 174 — Lines 490-490 (matmul)
```python
490|     flex = precision_config.flex_ctx
```
**EN:** Assigns `flex` and references `precision_config.flex_ctx`.

**CN:** 将 `flex` and 引用 `precision_config.flex_ctx`.

### Block 175 — Lines 491-492 (matmul)
```python
491|     bias_stride = None if bias is None else bias.stride(0)
492|     # moe metadata
```
**EN:** Assigns `bias_stride` and uses conditional expression `None if bias is None else bias.stride(0)`.

**CN:** 将 `bias_stride` and 使用条件表达式 `None if bias is None else bias.stride(0)`.

### Block 176 — Lines 493-493 (matmul)
```python
493|     expt_data_w = tuple([None] * 6) if ragged_dimension != "K" else ragged_metadata_fields(b_ragged_metadata, opt_flags.block_k)
```
**EN:** Assigns `expt_data_w` and uses conditional expression `tuple([None] * 6) if ragged_dimension != 'K' else ragged_...`.

**CN:** 将 `expt_data_w` and 使用条件表达式 `tuple([None] * 6) if ragged_dimension != 'K' else ragged_...`.

### Block 177 — Lines 494-495 (matmul)
```python
494|     expt_data_x = tuple([None] * 6) if ragged_dimension is None else ragged_metadata_fields(a_ragged_metadata, opt_flags.block_m if ragged_dimension == "M" else opt_flags.block_k)
495|     # spmd grid
```
**EN:** Assigns `expt_data_x` and uses conditional expression `tuple([None] * 6) if ragged_dimension is None else ragged...`.

**CN:** 将 `expt_data_x` and 使用条件表达式 `tuple([None] * 6) if ragged_dimension is None else ragged...`.

### Block 178 — Lines 496-496 (matmul)
```python
496|     grid_m = triton.cdiv(M, opt_flags.block_m)
```
**EN:** Assigns `grid_m` and calls `triton.cdiv`.

**CN:** 将 `grid_m`，并调用 `triton.cdiv`.

### Block 179 — Lines 497-498 (matmul)
```python
497|     if ragged_dimension == "M":
498|         grid_m = a_ragged_metadata.n_blocks(a_ragged_metadata.n_slices, M, opt_flags.block_m)
```
**EN:** Checks `ragged_dimension == 'M'`. The true branch mainly prepares intermediate values.

**CN:** 检查 `ragged_dimension == 'M'`. 真分支主要准备中间值.

### Block 180 — Lines 499-499 (matmul)
```python
499|     grid_n = triton.cdiv(N, opt_flags.block_n)
```
**EN:** Assigns `grid_n` and calls `triton.cdiv`.

**CN:** 将 `grid_n`，并调用 `triton.cdiv`.

### Block 181 — Lines 500-500 (matmul)
```python
500|     grid = batch_size * grid_m * grid_n * opt_flags.split_k
```
**EN:** Assigns `grid` and evaluates `batch_size * grid_m * grid_n * opt_flags.split_k`.

**CN:** 将 `grid` and 计算 `batch_size * grid_m * grid_n * opt_flags.split_k`.

### Block 182 — Lines 501-504 (matmul)
```python
501|     if opt_flags.is_persistent:
502|         available_sms = target_info.num_sms() - opt_flags.idle_sms
503|         grid = min(opt_flags.occupancy_target * available_sms, grid)
504|     # canonicalize storage
```
**EN:** Checks `opt_flags.is_persistent`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `opt_flags.is_persistent`. 真分支主要准备中间值; 准备中间值.

### Block 183 — Lines 505-505 (matmul)
```python
505|     has_scatter_tma = scatter_indx is not None and target_info.has_tma_gather()
```
**EN:** Assigns `has_scatter_tma` and evaluates `scatter_indx is not None and target_info.has_tma_gather()`.

**CN:** 将 `has_scatter_tma` and 计算 `scatter_indx is not None and target_info.has_tma_gather()`.

### Block 184 — Lines 506-506 (matmul)
```python
506|     c = wrap_torch_tensor(out_matmul.view(math.prod(out_matmul.shape[:-1]), out_matmul.shape[-1]) if has_scatter else out_matmul.view(math.prod(out_matmul.shape[:-2]), *out_matmul.shape[-2:]))
```
**EN:** Assigns `c` and calls `wrap_torch_tensor`.

**CN:** 将 `c`，并调用 `wrap_torch_tensor`.

### Block 185 — Lines 507-507 (matmul)
```python
507|     a = Tensor(_canonicalize_storage(a.storage, 2 if has_gather_tma else 3, flex.lhs_data), dtype=a.dtype, shape=a.shape, shape_max=a.shape_max)
```
**EN:** Assigns `a` and calls `Tensor`.

**CN:** 将 `a`，并调用 `Tensor`.

### Block 186 — Lines 508-508 (matmul)
```python
508|     b_storage_ndim = 5 if b_is_shuffled else 3
```
**EN:** Assigns `b_storage_ndim` and uses conditional expression `5 if b_is_shuffled else 3`.

**CN:** 将 `b_storage_ndim` and 使用条件表达式 `5 if b_is_shuffled else 3`.

### Block 187 — Lines 509-509 (matmul)
```python
509|     b = Tensor(_canonicalize_storage(b.storage, b_storage_ndim, flex.rhs_data), dtype=b.dtype, shape=b.shape, shape_max=b.shape_max)
```
**EN:** Assigns `b` and calls `Tensor`.

**CN:** 将 `b`，并调用 `Tensor`.

### Block 188 — Lines 510-511 (matmul)
```python
510|     c = Tensor(_canonicalize_storage(c.storage, 2 if has_scatter_tma else 3, out_matmul_flex), dtype=c.dtype, shape=c.shape, shape_max=c.shape_max)
511|     # create tma descriptor for x
```
**EN:** Assigns `c` and calls `Tensor`.

**CN:** 将 `c`，并调用 `Tensor`.

### Block 189 — Lines 512-521 (matmul)
```python
512|     if c_acc_in is not None:
513|         assert opt_flags.split_k == 1, "c_acc_in + split_k is not supported."
514|         assert scatter_indx is None, "c_acc_in + scatter is not supported."
515|         if c_acc_in.ndim == 2:
516|             c_acc_in = c_acc_in.unsqueeze(0)
517|         assert c_acc_in.shape == out_matmul.shape[-3:]
518|         c_acc_strides = c_acc_in.stride()
519|     else:
520|         c_acc_strides = (None, None, None)
521| 
```
**EN:** Checks `c_acc_in is not None`. The true branch mainly checks invariants; checks invariants, while the else branch prepares intermediate values.

**CN:** 检查 `c_acc_in is not None`. 真分支主要检查不变量; 检查不变量；而 else 分支准备中间值.

### Block 190 — Lines 522-522 (matmul)
```python
522|     a_tma_block_size = [1, opt_flags.block_k] if has_gather_tma else [1, opt_flags.block_m, opt_flags.block_k]
```
**EN:** Assigns `a_tma_block_size` and uses conditional expression `[1, opt_flags.block_k] if has_gather_tma else [1, opt_fla...`.

**CN:** 将 `a_tma_block_size` and 使用条件表达式 `[1, opt_flags.block_k] if has_gather_tma else [1, opt_fla...`.

### Block 191 — Lines 523-523 (matmul)
```python
523|     a_tma_mode = None if not a_has_tma else "ragged" if ragged_dimension == "M" and not has_gather_tma else "dense"
```
**EN:** Assigns `a_tma_mode` and uses conditional expression `None if not a_has_tma else 'ragged' if ragged_dimension =...`.

**CN:** 将 `a_tma_mode` and 使用条件表达式 `None if not a_has_tma else 'ragged' if ragged_dimension =...`.

### Block 192 — Lines 524-524 (matmul)
```python
524|     a_tensor_or_tma = make_tma(a, a_tma_block_size, a_tma_mode) if a_has_tma else a.storage.data
```
**EN:** Assigns `a_tensor_or_tma` and uses conditional expression `make_tma(a, a_tma_block_size, a_tma_mode) if a_has_tma el...`.

**CN:** 将 `a_tensor_or_tma` and 使用条件表达式 `make_tma(a, a_tma_block_size, a_tma_mode) if a_has_tma el...`.

### Block 193 — Lines 525-527 (matmul)
```python
525|     if a_has_tma and precision_config.allow_tf32 and a.storage.data.dtype == torch.float32:
526|         a_tensor_or_tma.round_f32_to_tf32 = True
527|     # create tma descriptor for y
```
**EN:** Checks `a_has_tma and precision_config.allow_tf32 and (a.storage.data.dtype == torch.float32)`. The true branch mainly prepares intermediate values.

**CN:** 检查 `a_has_tma and precision_config.allow_tf32 and (a.storage.data.dtype == torch.float32)`. 真分支主要准备中间值.

### Block 194 — Lines 528-537 (matmul)
```python
528|     c_has_tma = (
529|         opt_flags.is_persistent and (scatter_indx is None or has_scatter_tma)
530|         and is_tma_compliant(c)
531|         and (c_acc_in is None or c_acc_is_c)
532|         and fused_comm is None
533|         and (
534|             precision_config.c_value_pack_factor == 1
535|             or matmul_fused_activation.specs.reduction_n == 1
536|         )
537|     )
```
**EN:** Assigns `c_has_tma` and evaluates `opt_flags.is_persistent and (scatter_indx is None or has_scatter_tm...`.

**CN:** 将 `c_has_tma` and 计算 `opt_flags.is_persistent and (scatter_indx is None or has_scatter_tm...`.

### Block 195 — Lines 538-538 (matmul)
```python
538|     c_logical_block_n = opt_flags.block_n // opt_flags.epilogue_subtile // matmul_fused_activation.specs.reduction_n
```
**EN:** Assigns `c_logical_block_n` and evaluates `opt_flags.block_n // opt_flags.epilogue_subtile // matmul_fused_act...`.

**CN:** 将 `c_logical_block_n` and 计算 `opt_flags.block_n // opt_flags.epilogue_subtile // matmul_fused_act...`.

### Block 196 — Lines 539-539 (matmul)
```python
539|     c_tma_block_n = c_logical_block_n // precision_config.c_value_pack_factor
```
**EN:** Assigns `c_tma_block_n` and evaluates `c_logical_block_n // precision_config.c_value_pack_factor`.

**CN:** 将 `c_tma_block_n` and 计算 `c_logical_block_n // precision_config.c_value_pack_factor`.

### Block 197 — Lines 540-540 (matmul)
```python
540|     out_tile_n = opt_flags.block_n // matmul_fused_activation.specs.reduction_n
```
**EN:** Assigns `out_tile_n` and evaluates `opt_flags.block_n // matmul_fused_activation.specs.reduction_n`.

**CN:** 将 `out_tile_n` and 计算 `opt_flags.block_n // matmul_fused_activation.specs.reduction_n`.

### Block 198 — Lines 541-541 (matmul)
```python
541|     c_tma_block_size = [1, c_tma_block_n] if has_scatter_tma else [1, opt_flags.block_m, c_tma_block_n]
```
**EN:** Assigns `c_tma_block_size` and uses conditional expression `[1, c_tma_block_n] if has_scatter_tma else [1, opt_flags....`.

**CN:** 将 `c_tma_block_size` and 使用条件表达式 `[1, c_tma_block_n] if has_scatter_tma else [1, opt_flags....`.

### Block 199 — Lines 542-542 (matmul)
```python
542|     c_tma_mode = None if not c_has_tma else "ragged" if is_c_ragged and not has_scatter_tma else "dense"
```
**EN:** Assigns `c_tma_mode` and uses conditional expression `None if not c_has_tma else 'ragged' if is_c_ragged and (n...`.

**CN:** 将 `c_tma_mode` and 使用条件表达式 `None if not c_has_tma else 'ragged' if is_c_ragged and (n...`.

### Block 200 — Lines 543-544 (matmul)
```python
543|     c_tensor_or_tma = make_tma(c, c_tma_block_size, c_tma_mode) if c_has_tma else c.storage.data
544|     # create tma descriptor for w
```
**EN:** Assigns `c_tensor_or_tma` and uses conditional expression `make_tma(c, c_tma_block_size, c_tma_mode) if c_has_tma el...`.

**CN:** 将 `c_tensor_or_tma` and 使用条件表达式 `make_tma(c, c_tma_block_size, c_tma_mode) if c_has_tma el...`.

### Block 201 — Lines 545-545 (matmul)
```python
545|     b_has_tma = opt_flags.is_persistent
```
**EN:** Assigns `b_has_tma` and references `opt_flags.is_persistent`.

**CN:** 将 `b_has_tma` and 引用 `opt_flags.is_persistent`.

### Block 202 — Lines 546-546 (matmul)
```python
546|     b_tensor_or_tma = make_tma(b, [1, opt_flags.block_k, opt_flags.block_n], "dense") if b_has_tma else b.storage.data
```
**EN:** Assigns `b_tensor_or_tma` and uses conditional expression `make_tma(b, [1, opt_flags.block_k, opt_flags.block_n], 'd...`.

**CN:** 将 `b_tensor_or_tma` and 使用条件表达式 `make_tma(b, [1, opt_flags.block_k, opt_flags.block_n], 'd...`.

### Block 203 — Lines 547-549 (matmul)
```python
547|     if b_has_tma and precision_config.allow_tf32 and b.storage.data.dtype == torch.float32:
548|         b_tensor_or_tma.round_f32_to_tf32 = True
549|     # create tma descriptor for w_scale
```
**EN:** Checks `b_has_tma and precision_config.allow_tf32 and (b.storage.data.dtype == torch.float32)`. The true branch mainly prepares intermediate values.

**CN:** 检查 `b_has_tma and precision_config.allow_tf32 and (b.storage.data.dtype == torch.float32)`. 真分支主要准备中间值.

### Block 204 — Lines 550-550 (matmul)
```python
550|     b_scale_has_tma = opt_flags.is_persistent and b_scale is not None
```
**EN:** Assigns `b_scale_has_tma` and evaluates `opt_flags.is_persistent and b_scale is not None`.

**CN:** 将 `b_scale_has_tma` and 计算 `opt_flags.is_persistent and b_scale is not None`.

### Block 205 — Lines 551-551 (matmul)
```python
551|     b_transpose = b_is_shuffled or b.storage.data.stride()[-2] == 1
```
**EN:** Assigns `b_transpose` and evaluates `b_is_shuffled or b.storage.data.stride()[-2] == 1`.

**CN:** 将 `b_transpose` and 计算 `b_is_shuffled or b.storage.data.stride()[-2] == 1`.

### Block 206 — Lines 552-573 (matmul)
```python
552|     if b_scale_has_tma:
553|         scale_block_k = opt_flags.block_k // mx_block_size
554|         b_scale_storage = b_scale.storage
555|         b_scale_tma_block_size = [scale_block_k, opt_flags.block_n]
556|         if isinstance(b_scale_storage.layout, (StridedLayout, HopperMXScaleLayout)):
557|             b_scale_tma_block_size = [1] + b_scale_tma_block_size
558|             b_scale_tensor_or_tma = make_tma(
559|                 Tensor(
560|                     _canonicalize_storage(b_scale.storage, 3, None),
561|                     dtype=b_scale.dtype,
562|                     shape=b_scale.shape,
563|                     shape_max=b_scale.shape_max,
564|                 ),
565|                 b_scale_tma_block_size,
566|                 "dense",
567|                 is_scale=True,
568|             )
569|         else:
570|             b_scale_tensor_or_tma = make_tma(b_scale, b_scale_tma_block_size, "dense", is_scale=True)
571|     else:
572|         b_scale_tensor_or_tma = None if b_scale is None else b_scale.storage.data
573|     # create tma descriptor for x_scale
```
**EN:** Checks `b_scale_has_tma`. The true branch mainly prepares intermediate values; prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `b_scale_has_tma`. 真分支主要准备中间值; 准备中间值；而 else 分支准备中间值.

### Block 207 — Lines 574-574 (matmul)
```python
574|     a_scale_has_tma = False
```
**EN:** Assigns `a_scale_has_tma` and stores constant `False`.

**CN:** 将 `a_scale_has_tma` and 保存常量 `False`.

### Block 208 — Lines 575-579 (matmul)
```python
575|     if a_has_mx and isinstance(a_scale.storage.layout, BlackwellActMXScaleLayout):
576|         # check if we can use tma for x scale
577|         assert opt_flags.is_persistent, "swizzled x scale is only supported for persistent case"
578|         assert opt_flags.block_m == 128 and opt_flags.block_k >= 128, "block_m and block_k must be at least 128 if x scale is swizzled"
579|         a_scale_has_tma = True
```
**EN:** Checks `a_has_mx and isinstance(a_scale.storage.layout, BlackwellActMXScaleLayout)`. The true branch mainly checks invariants; checks invariants.

**CN:** 检查 `a_has_mx and isinstance(a_scale.storage.layout, BlackwellActMXScaleLayout)`. 真分支主要检查不变量; 检查不变量.

### Block 209 — Lines 580-586 (matmul)
```python
580|     if a_scale_has_tma:
581|         scale_block_k = opt_flags.block_k // mx_block_size
582|         a_scale_tma_block_size = [opt_flags.block_m, scale_block_k]
583|         a_scale_tensor_or_tma = make_tma(a_scale, a_scale_tma_block_size, "dense", is_scale=True)
584|     else:
585|         a_scale_tensor_or_tma = None if a_scale is None else a_scale.storage.data
586|     # canonicalize strides
```
**EN:** Checks `a_scale_has_tma`. The true branch mainly prepares intermediate values; prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `a_scale_has_tma`. 真分支主要准备中间值; 准备中间值；而 else 分支准备中间值.

### Block 210 — Lines 587-587 (matmul)
```python
587|     a_strides = [0]*(3 - a.storage.data.ndim) + list(a.storage.data.stride())
```
**EN:** Assigns `a_strides` and evaluates `[0] * (3 - a.storage.data.ndim) + list(a.storage.data.stride())`.

**CN:** 将 `a_strides` and 计算 `[0] * (3 - a.storage.data.ndim) + list(a.storage.data.stride())`.

### Block 211 — Lines 588-588 (matmul)
```python
588|     a_scale_strides = a_scale.stride() if a_has_mx and not a_scale_has_tma else (None, None, None)
```
**EN:** Assigns `a_scale_strides` and uses conditional expression `a_scale.stride() if a_has_mx and (not a_scale_has_tma) el...`.

**CN:** 将 `a_scale_strides` and 使用条件表达式 `a_scale.stride() if a_has_mx and (not a_scale_has_tma) el...`.

### Block 212 — Lines 589-589 (matmul)
```python
589|     a_scale_strides = (0, ) * (3 - len(a_scale_strides)) + a_scale_strides
```
**EN:** Assigns `a_scale_strides` and evaluates `(0,) * (3 - len(a_scale_strides)) + a_scale_strides`.

**CN:** 将 `a_scale_strides` and 计算 `(0,) * (3 - len(a_scale_strides)) + a_scale_strides`.

### Block 213 — Lines 590-590 (matmul)
```python
590|     b_scale_strides = b_scale.stride() if b_has_mx and not b_scale_has_tma else (None, None, None)
```
**EN:** Assigns `b_scale_strides` and uses conditional expression `b_scale.stride() if b_has_mx and (not b_scale_has_tma) el...`.

**CN:** 将 `b_scale_strides` and 使用条件表达式 `b_scale.stride() if b_has_mx and (not b_scale_has_tma) el...`.

### Block 214 — Lines 591-592 (matmul)
```python
591|     b_scale_strides = (0, ) * (3 - len(b_scale_strides)) + b_scale_strides
592| 
```
**EN:** Assigns `b_scale_strides` and evaluates `(0,) * (3 - len(b_scale_strides)) + b_scale_strides`.

**CN:** 将 `b_scale_strides` and 计算 `(0,) * (3 - len(b_scale_strides)) + b_scale_strides`.

### Block 215 — Lines 593-593 (matmul)
```python
593|     out_matmul_scale_strides = out_matmul_scale.storage.data.stride() if out_matmul_has_mx else (None, None, None, None)
```
**EN:** Assigns `out_matmul_scale_strides` and uses conditional expression `out_matmul_scale.storage.data.stride() if out_matmul_has_...`.

**CN:** 将 `out_matmul_scale_strides` and 使用条件表达式 `out_matmul_scale.storage.data.stride() if out_matmul_has_...`.

### Block 216 — Lines 594-594 (matmul)
```python
594|     out_matmul_scale_strides = (0, ) * (4 - len(out_matmul_scale_strides)) + out_matmul_scale_strides
```
**EN:** Assigns `out_matmul_scale_strides` and evaluates `(0,) * (4 - len(out_matmul_scale_strides)) + out_matmul_scale_strides`.

**CN:** 将 `out_matmul_scale_strides` and 计算 `(0,) * (4 - len(out_matmul_scale_strides)) + out_matmul_scale_strides`.

### Block 217 — Lines 595-595 (matmul)
```python
595|     out_matmul_scale_layout = None if out_matmul_scale is None else out_matmul_scale.storage.layout.name
```
**EN:** Assigns `out_matmul_scale_layout` and uses conditional expression `None if out_matmul_scale is None else out_matmul_scale.st...`.

**CN:** 将 `out_matmul_scale_layout` and 使用条件表达式 `None if out_matmul_scale is None else out_matmul_scale.st...`.

### Block 218 — Lines 596-602 (matmul)
```python
596|     if (
597|         scatter_indx is not None
598|         and out_matmul_scale is not None
599|         and isinstance(out_matmul_scale.storage.layout, BlackwellActMXScaleLayout)
600|         and out_matmul_scale.storage.layout.ragged_metadata is not None
601|     ):
602|         raise NotImplementedError("scatter with ragged Blackwell ACT output MX scales is not supported")
```
**EN:** Checks `scatter_indx is not None and out_matmul_scale is not None and isinstance(out_matmul_sca...`..

**CN:** 检查 `scatter_indx is not None and out_matmul_scale is not None and isinstance(out_matmul_sca...`..

### Block 219 — Lines 603-603 (matmul)
```python
603|     output_scale_block_offs = None
```
**EN:** Assigns `output_scale_block_offs` and stores constant `None`.

**CN:** 将 `output_scale_block_offs` and 保存常量 `None`.

### Block 220 — Lines 604-606 (matmul)
```python
604|     if ragged_dimension == "M" and out_matmul_scale_layout == "BLACKWELL_ACT_SCALE":
605|         output_scale_block_offs = a_ragged_metadata.block_offs(int(SWIZZLE_SIZE_OUTER))
606|     # launch kernel
```
**EN:** Checks `ragged_dimension == 'M' and out_matmul_scale_layout == 'BLACKWELL_ACT_SCALE'`. The true branch mainly prepares intermediate values.

**CN:** 检查 `ragged_dimension == 'M' and out_matmul_scale_layout == 'BLACKWELL_ACT_SCALE'`. 真分支主要准备中间值.

### Block 221 — Lines 607-611 (matmul)
```python
607|     kernels = specializations.get(epilogue=epilogue.specs, activation=matmul_fused_activation.specs)
608|     # When stride(-2) == stride(-1) == 1, it's ambiguous whether W is transposed
609|     # (i.e. col-wise). Since this matters when w_has_mx is True and w_transpose
610|     # is True the fast code path, stride(-2) == 1 takes precedence, e.g., vs.
611|     # w_transpose = w_storage.data.stride()[-1] != 1
```
**EN:** Assigns `kernels` and calls `specializations.get`.

**CN:** 将 `kernels`，并调用 `specializations.get`.

### Block 222 — Lines 612-618 (matmul)
```python
612|     fused_comm_kwargs = {
613|         "pYPtrs": fused_comm.out_handles,
614|         "map_dst_coord": fused_comm.map_dst_coord,
615|         "all_writes_issued": fused_comm.all_writes_issued,
616|         "reduce_rank": fused_comm.reduce_rank,
617|         "n_reduce_shards": fused_comm.n_reduce_shards,
618|     } if fused_comm is not None else {}
```
**EN:** Assigns `fused_comm_kwargs` and uses conditional expression `{'pYPtrs': fused_comm.out_handles, 'map_dst_coord': fused...`.

**CN:** 将 `fused_comm_kwargs` and 使用条件表达式 `{'pYPtrs': fused_comm.out_handles, 'map_dst_coord': fused...`.

### Block 223 — Lines 619-619 (matmul)
```python
619|     b_strides = b.storage.data.stride()[:3] if b_is_shuffled else b.storage.data.stride()
```
**EN:** Assigns `b_strides` and uses conditional expression `b.storage.data.stride()[:3] if b_is_shuffled else b.stora...`.

**CN:** 将 `b_strides` and 使用条件表达式 `b.storage.data.stride()[:3] if b_is_shuffled else b.stora...`.

### Block 224 — Lines 620-620 (matmul)
```python
620|     extra_kernel_kwargs = {"W_SHUFFLED": b_is_shuffled} if opt_flags.is_persistent else {}
```
**EN:** Assigns `extra_kernel_kwargs` and uses conditional expression `{'W_SHUFFLED': b_is_shuffled} if opt_flags.is_persistent ...`.

**CN:** 将 `extra_kernel_kwargs` and 使用条件表达式 `{'W_SHUFFLED': b_is_shuffled} if opt_flags.is_persistent ...`.

### Block 225 — Lines 621-621 (matmul)
```python
621|     n_valid_slices = b_tensor_or_tma.shape[0] if ragged_dimension == "M" else n_slices
```
**EN:** Assigns `n_valid_slices` and uses conditional expression `b_tensor_or_tma.shape[0] if ragged_dimension == 'M' else ...`.

**CN:** 将 `n_valid_slices` and 使用条件表达式 `b_tensor_or_tma.shape[0] if ragged_dimension == 'M' else ...`.

### Block 226 — Lines 622-688 (matmul)
```python
622|     (kernels._p_matmul if opt_flags.is_persistent else kernels._matmul)[(grid,)](
623|                    c_tensor_or_tma, c.storage.data, *out_matmul.stride(),
624|                    *((out_matmul_flex.expected_scale, out_matmul_scale.storage.data, None) if out_matmul_has_mx else out_matmul_flex),
625|                    *out_matmul_scale_strides[-4:],
626|                    a_tensor_or_tma, a.storage.data, *a_strides, a_transpose,
627|                    flex.lhs_data.scale,
628|                    a_scale_tensor_or_tma, *a_scale_strides,
629|                    b_tensor_or_tma, b.storage.data, *b_strides, b_transpose,
630|                    flex.rhs_data.scale,
631|                    b_scale_tensor_or_tma, *b_scale_strides,
632|                    flex.acc_data.reinterpret(c_acc_in), *c_acc_strides,
633|                    flex.acc_data.scale, c_acc_is_c,
634|                    bias, bias_stride,
635|                    None if ragged_dimension == "M" else a.shape[-2],
636|                    N, K, K_W,
637|                    betas, gammas,
638|                    gather_indx,
639|                    scatter_indx,
640|                    None if scatter_indx is None else scatter_indx.shape[0],
641|                    ragged_dimension,
642|                    *expt_data_x,
643|                    output_scale_block_offs,
644|                    *expt_data_w,
645|                    batch_size, grid_m, grid_n,
646|                    out_alpha,
647|                    *matmul_fused_activation.fn_args, matmul_fused_activation.specs.reduction_n,
648|                    *epilogue.fn_arg_values_matmul,
649|                    n_valid_slices,
650|                    precision_config.max_num_imprecise_acc,
651|                    precision_config.allow_tf32,
652|                    precision_config.flexpoint_saturate_inf,
653|                    flex.rhs_data.is_per_batch,
654|                    out_matmul_flex.is_per_batch,
655|                    flex.acc_data.is_per_batch,
656|                    opt_flags.block_m,
657|                    opt_flags.block_n,
658|                    opt_flags.block_k,
659|                    opt_flags.group_m,
660|                    XCD_SWIZZLE=opt_flags.xcd_swizzle,
661|                    SWIZZLE_MX_VALUE=b.storage.layout.name,
662|                    SWIZZLE_MX_SCALE="STRIDED" if b_scale is None else b_scale.storage.layout.name,
663|                    MX_BLOCK_SIZE=mx_block_size,
664|                    EPILOGUE_SUBTILE=opt_flags.epilogue_subtile,
665|                    SPLIT_K=opt_flags.split_k,
666|                    EVEN_K=even_K,
667|                    W_CACHE_MODIFIER=opt_flags.w_cache_modifier,
668|                    TOKENS_PER_EXPT_FOR_ANNOTATION=None if a_ragged_metadata is None else a_ragged_metadata.expected_slice_size,
669|                    num_warps=opt_flags.num_warps,
670|                    num_stages=opt_flags.num_stages,
671|                    arch=opt_flags.arch,
672|                    UPCAST_INDICES=should_upcast_indices(a, b, out_matmul),
673|                    X_TMA_MODE=a_tma_mode,
674|                    Y_TMA_MODE=c_tma_mode,
675|                    Y_MX_SCALE_LAYOUT=out_matmul_scale_layout,
676|                    OUT_N_TILE_ALIGNED=(N // matmul_fused_activation.specs.reduction_n) % out_tile_n == 0,
677|                    SWAP_XW=get_swap_xw(precision_config, opt_flags, a.dtype, b.dtype),
678|                    IS_EPILOGUE_QUANT_MX=epilogue.specs.name in (
679|                        FnName.QUANTIZE_MXFP8.name,
680|                        FnName.QUANTIZE_MXFP4.name,
681|                        FnName.QUANTIZE_NVFP4.name,
682|                    ),
683|                    Y_VALUE_PACK_FACTOR=precision_config.c_value_pack_factor,
684|                    NUM_SMS = grid if opt_flags.is_persistent else 0,
685|                    **fused_comm_kwargs,
686|                    **opt_flags.target_kernel_kwargs,
687|                    **extra_kernel_kwargs)
688| 
```
**EN:** Calls `(kernels._p_matmul if opt_flags.is_persistent else kernels._matmul)[grid,]` for side effects, registration, or validation.

**CN:** 调用 `(kernels._p_matmul if opt_flags.is_persistent else kernels._matmul)[grid,]` ，用于副作用、注册或校验。

### Block 227 — Lines 689-689 (matmul)
```python
689|     assert not (opt_flags.split_k > 1 and scatter_indx is not None)
```
**EN:** Asserts `not (opt_flags.split_k > 1 and scatter_indx is not None)` to enforce invariants.

**CN:** 断言 `not (opt_flags.split_k > 1 and scatter_indx is not None)` 以确保不变量成立。

### Block 228 — Lines 690-690 (matmul)
```python
690|     out_final_mx_scale = None
```
**EN:** Assigns `out_final_mx_scale` and stores constant `None`.

**CN:** 将 `out_final_mx_scale` and 保存常量 `None`.

### Block 229 — Lines 691-722 (matmul)
```python
691|     if opt_flags.split_k > 1:
692|         assert not out_matmul_has_mx
693|         postprocess_fn1 = ReducePostprocessFn(specs=reduce_fused_activation.specs, fn_args=reduce_fused_activation.fn_args)
694|         postprocess_fn2 = ReducePostprocessFn(specs=epilogue.specs, fn_args=epilogue.fn_arg_values_finalize)
695|         c, y_mx_scale = reduce(
696|             x = out_matmul.view(out_matmul.shape[0], -1, out_matmul.shape[-1]),
697|             dim = 0,
698|             # output data/metadata
699|             y = memory["output"].view(-1, memory["output"].shape[-1]),
700|             y_dtype = memory["output"].dtype,
701|             y_flex = precision_config.flex_ctx.out_data,
702|             y_flex_saturate_inf = precision_config.flexpoint_saturate_inf,
703|             y_has_mx = precision_config.c_mx_scale is not None,
704|             y_mx_scale_dtype = None if precision_config.c_mx_scale is None else precision_config.c_mx_scale.storage.data.dtype if isinstance(precision_config.c_mx_scale, Tensor) else precision_config.c_mx_scale.dtype,
705|             y_microblock_size = precision_config.c_microblock_size,
706|             y_value_pack_factor = precision_config.c_value_pack_factor,
707|             # fused functions
708|             postprocess_fn1 = postprocess_fn1,
709|             postprocess_fn2 = postprocess_fn2,
710|         )
711|         logical_out_n = out_matmul.shape[-1] // reduce_fused_activation.specs.reduction_n
712|         out_final = c.view(*memory["output"].shape[1:])
713|         if y_mx_scale is not None:
714|             out_final_mx_scale = y_mx_scale.view(*memory["output"].shape[1:-1], triton.cdiv(logical_out_n, precision_config.c_microblock_size))
715|     else:
716|         out_final = out_matmul.squeeze(0)
717|         out_final_mx_scale = (
718|             None if out_matmul_scale is None
719|             else out_matmul_scale if isinstance(precision_config.c_mx_scale, Tensor)
720|             else out_matmul_scale.storage.data
721|         )
722| 
```
**EN:** Checks `opt_flags.split_k > 1`. The true branch mainly checks invariants; prepares intermediate values, while the else branch prepares intermediate values; prepares intermediate values.

**CN:** 检查 `opt_flags.split_k > 1`. 真分支主要检查不变量; 准备中间值；而 else 分支准备中间值; 准备中间值.

### Block 230 — Lines 723-724 (matmul)
```python
723|     if not (is_input_batched or b_ragged_metadata is not None):
724|         out_final = out_final.squeeze(0)
```
**EN:** Checks `not (is_input_batched or b_ragged_metadata is not None)`. The true branch mainly prepares intermediate values.

**CN:** 检查 `not (is_input_batched or b_ragged_metadata is not None)`. 真分支主要准备中间值.

### Block 231 — Lines 725-726 (matmul)
```python
725|     if out_final_mx_scale is not None:
726|         precision_config.c_mx_scale = out_final_mx_scale
```
**EN:** Checks `out_final_mx_scale is not None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `out_final_mx_scale is not None`. 真分支主要准备中间值.

### Block 232 — Lines 727-732 (matmul)
```python
727|     return out_final
728| 
729| # -----------------------------------------------------------------------------
730| # Reference Implementation
731| # -----------------------------------------------------------------------------
732| 
```
**EN:** Returns `out_final`.

**CN:** 返回 `out_final`.

### Block 233 — Lines 733-733 (apply_precision)
```python
733| def apply_precision(x_tri, w_tri, precision_config):
```
**EN:** Defines function `apply_precision(x_tri, w_tri, precision_config)` for this module. The body mainly prepares intermediate values; branches on runtime conditions; branches on runtime conditions. It uses calls such as `layout.StridedLayout`, `convert_layout`, `upcast_from_mxfp`, `apply`, `x.clone` to implement its workflow.

**CN:** 定义函数 `apply_precision(x_tri, w_tri, precision_config)`，供本模块使用. 主体主要准备中间值; 根据运行时条件分支; 根据运行时条件分支. 其中会调用 `layout.StridedLayout`, `convert_layout`, `upcast_from_mxfp`, `apply`, `x.clone` 来实现其工作流程.

### Block 234 — Lines 734-736 (apply_precision)
```python
734|     from .tensor_details import layout
735|     from .numerics_details.mxfp import upcast_from_mxfp
736| 
```
**EN:** This block imports `.tensor_details (layout)`, `.numerics_details.mxfp (upcast_from_mxfp)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `.tensor_details (layout)`, `.numerics_details.mxfp (upcast_from_mxfp)` ，为模块提供所需的外部库和内部辅助工具。

### Block 235 — Lines 737-738 (apply_precision)
```python
737|     flex_ctx = precision_config.flex_ctx
738| 
```
**EN:** Assigns `flex_ctx` and references `precision_config.flex_ctx`.

**CN:** 将 `flex_ctx` and 引用 `precision_config.flex_ctx`.

### Block 236 — Lines 739-739 (apply)
```python
739|     def apply(x, scale):
```
**EN:** Defines function `apply(x, scale)` for this module. The body mainly branches on runtime conditions; returns the computed result. It uses calls such as `x.clone`, `x.float` to implement its workflow.

**CN:** 定义函数 `apply(x, scale)`，供本模块使用. 主体主要根据运行时条件分支; 返回计算结果. 其中会调用 `x.clone`, `x.float` 来实现其工作流程.

### Block 237 — Lines 740-741 (apply)
```python
740|         if scale is None:
741|             return x.clone()
```
**EN:** Checks `scale is None`. The true branch mainly returns the computed result.

**CN:** 检查 `scale is None`. 真分支主要返回计算结果.

### Block 238 — Lines 742-743 (apply)
```python
742|         return x.float() * scale
743| 
```
**EN:** Returns `x.float() * scale`.

**CN:** 返回 `x.float() * scale`.

### Block 239 — Lines 744-753 (apply_precision)
```python
744|     if precision_config.a_mx_scale is not None:
745|         a_scale = precision_config.a_mx_scale
746|         mx_axis = x_tri.ndim - 1
747|         canonical_layout = layout.StridedLayout(major_dim=mx_axis)
748|         x_tri = convert_layout(x_tri, canonical_layout)
749|         x_tri_scale = convert_layout(a_scale, canonical_layout)
750|         x_ref = upcast_from_mxfp(x_tri.storage.data, x_tri_scale.storage.data, torch.bfloat16, axis=mx_axis)
751|     else:
752|         x_ref = apply(x_tri, flex_ctx.lhs_data.scale)
753| 
```
**EN:** Checks `precision_config.a_mx_scale is not None`. The true branch mainly prepares intermediate values; prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `precision_config.a_mx_scale is not None`. 真分支主要准备中间值; 准备中间值；而 else 分支准备中间值.

### Block 240 — Lines 754-763 (apply_precision)
```python
754|     if precision_config.b_mx_scale is not None:
755|         b_scale = precision_config.b_mx_scale
756|         mx_axis = w_tri.ndim - 2
757|         canonical_layout = layout.StridedLayout(major_dim=mx_axis)
758|         w_tri = convert_layout(w_tri, canonical_layout)
759|         w_tri_scale = convert_layout(b_scale, canonical_layout)
760|         w_ref = upcast_from_mxfp(w_tri.storage.data, w_tri_scale.storage.data, torch.bfloat16, axis=mx_axis)
761|     else:
762|         w_ref = apply(w_tri, flex_ctx.rhs_data.scale)
763| 
```
**EN:** Checks `precision_config.b_mx_scale is not None`. The true branch mainly prepares intermediate values; prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `precision_config.b_mx_scale is not None`. 真分支主要准备中间值; 准备中间值；而 else 分支准备中间值.

### Block 241 — Lines 764-768 (apply_precision)
```python
764|     return (
765|         x_ref, w_ref,
766|     )
767| 
768| 
```
**EN:** Returns `(x_ref, w_ref)`.

**CN:** 返回 `(x_ref, w_ref)`.

### Block 242 — Lines 769-769 (scale)
```python
769| def scale(val, scal):
```
**EN:** Defines function `scale(val, scal)` for this module. The body mainly branches on runtime conditions. It uses calls such as `scal.numel` to implement its workflow.

**CN:** 定义函数 `scale(val, scal)`，供本模块使用. 主体主要根据运行时条件分支. 其中会调用 `scal.numel` 来实现其工作流程.

### Block 243 — Lines 770-777 (scale)
```python
770|     if scal is None:
771|         return val
772|     elif scal.numel() == 1:
773|         return val / scal
774|     else:
775|         assert val.ndim == 3
776|         return val / scal[:, None, None]
777| 
```
**EN:** Checks `scal is None`. The true branch mainly returns the computed result, while the else branch branches on runtime conditions.

**CN:** 检查 `scal is None`. 真分支主要返回计算结果；而 else 分支根据运行时条件分支.

### Block 244 — Lines 778-778 (compute_actual_scale)
```python
778| def compute_actual_scale(x, dtype, per_batch_scale=False):
```
**EN:** Defines function `compute_actual_scale(x, dtype, per_batch_scale)` for this module. The body mainly prepares intermediate values; prepares intermediate values; returns the computed result. It uses calls such as `x.abs().amax`, `x.abs().max`, `x.abs`, `tuple`, `range` to implement its workflow.

**CN:** 定义函数 `compute_actual_scale(x, dtype, per_batch_scale)`，供本模块使用. 主体主要准备中间值; 准备中间值; 返回计算结果. 其中会调用 `x.abs().amax`, `x.abs().max`, `x.abs`, `tuple`, `range` 来实现其工作流程.

### Block 245 — Lines 779-779 (compute_actual_scale)
```python
779|     from triton_kernels.numerics import MAX_FINITE_FLOAT8E4B8, MAX_FINITE_FLOAT8E4NV, MAX_FINITE_FLOAT8E5
```
**EN:** This block imports `triton_kernels.numerics (MAX_FINITE_FLOAT8E4B8, MAX_FINITE_FLOAT8E4NV, MAX_FINITE_FLOAT8E5)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `triton_kernels.numerics (MAX_FINITE_FLOAT8E4B8, MAX_FINITE_FLOAT8E4NV, MAX_FINITE_FLOAT8E5)` ，为模块提供所需的外部库和内部辅助工具。

### Block 246 — Lines 780-784 (compute_actual_scale)
```python
780|     max_finite = {
781|         torch.float8_e5m2: MAX_FINITE_FLOAT8E5,
782|         torch.float8_e4m3fn: MAX_FINITE_FLOAT8E4NV,
783|         torch.float8_e4m3fnuz: MAX_FINITE_FLOAT8E4B8,
784|     }[dtype]
```
**EN:** Assigns `max_finite` and evaluates `{torch.float8_e5m2: MAX_FINITE_FLOAT8E5, torch.float8_e4m3fn: MAX_F...`.

**CN:** 将 `max_finite` and 计算 `{torch.float8_e5m2: MAX_FINITE_FLOAT8E5, torch.float8_e4m3fn: MAX_F...`.

### Block 247 — Lines 785-785 (compute_actual_scale)
```python
785|     maxvals = x.abs().amax(dim=tuple(range(1, x.ndim))) if per_batch_scale else x.abs().max()
```
**EN:** Assigns `maxvals` and uses conditional expression `x.abs().amax(dim=tuple(range(1, x.ndim))) if per_batch_sc...`.

**CN:** 将 `maxvals` and 使用条件表达式 `x.abs().amax(dim=tuple(range(1, x.ndim))) if per_batch_sc...`.

### Block 248 — Lines 786-788 (compute_actual_scale)
```python
786|     return maxvals / max_finite
787| 
788| 
```
**EN:** Returns `maxvals / max_finite`.

**CN:** 返回 `maxvals / max_finite`.

### Block 249 — Lines 789-798 (matmul_torch)
```python
789| def matmul_torch(a, b, bias,
790|                  a_ragged_metadata: RaggedTensorMetadata | None = None,
791|                  b_ragged_metadata: RaggedTensorMetadata | None = None,
792|                  gather_indx: torch.Tensor = None,
793|                  scatter_indx: torch.Tensor = None,
794|                  precision_config: PrecisionConfig = None,
795|                  betas = None,
796|                  gammas = None,
797|                  round_x = None, round_y = None,
798|                  ):
```
**EN:** Defines function `matmul_torch(a, b, bias, a_ragged_metadata, b_ragged_metadata, gather_indx, scatter_indx, precision_config, betas, gammas, round_x, round_y)` for this module. The body mainly prepares intermediate values; branches on runtime conditions; prepares intermediate values. It uses calls such as `apply_precision`, `torch.zeros`, `enumerate`, `scale`, `range` to implement its workflow.

**CN:** 定义函数 `matmul_torch(a, b, bias, a_ragged_metadata, b_ragged_metadata, gather_indx, scatter_indx, precision_config, betas, gammas, round_x, round_y)`，供本模块使用. 主体主要准备中间值; 根据运行时条件分支; 准备中间值. 其中会调用 `apply_precision`, `torch.zeros`, `enumerate`, `scale`, `range` 来实现其工作流程.

### Block 250 — Lines 799-800 (matmul_torch)
```python
799|     a, b = apply_precision(a, b, precision_config)
800| 
```
**EN:** Assigns `a`, `b` and calls `apply_precision`.

**CN:** 将 `a`, `b`，并调用 `apply_precision`.

### Block 251 — Lines 801-826 (matmul_torch)
```python
801|     if b_ragged_metadata is not None:
802|         n_expts_tot = b_ragged_metadata.slice_sizes.shape[0]
803|         m, n = a.shape[-2], b.shape[-1]
804|         out = torch.zeros((n_expts_tot, m, n), dtype=torch.float32, device=a.device)
805|         x_slice_offs = a_ragged_metadata.slice_offs
806|         w_slice_offs = b_ragged_metadata.slice_offs
807|         for expt in range(n_expts_tot):
808|             k = int(b_ragged_metadata.slice_sizes[expt].item())
809|             if k == 0:
810|                 continue
811|             x_start = int(x_slice_offs[expt].item())
812|             w_start = int(w_slice_offs[expt].item())
813|             x_slice = a[:, x_start:x_start + k]
814|             w_slice = b[w_start:w_start + k, :]
815|             out_expt = matmul_torch(
816|                 x_slice, w_slice, None, None,
817|                 None, None, None, PrecisionConfig(),
818|                 betas, gammas,
819|                 round_x, round_y,
820|             )
821|             out[expt] = out_expt.to(out.dtype)
822|         actual_scale = precision_config.flex_ctx.out_data.actual_scale
823|         if actual_scale is not None:
824|             actual_scale.copy_(compute_actual_scale(out, precision_config.out_dtype))
825|         return scale(out, precision_config.flex_ctx.out_data.expected_scale)
826| 
```
**EN:** Checks `b_ragged_metadata is not None`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `b_ragged_metadata is not None`. 真分支主要准备中间值; 准备中间值.

### Block 252 — Lines 827-827 (matmul_torch)
```python
827|     is_input_batched = a.ndim == 3
```
**EN:** Assigns `is_input_batched` and evaluates `a.ndim == 3`.

**CN:** 将 `is_input_batched` and 计算 `a.ndim == 3`.

### Block 253 — Lines 828-828 (matmul_torch)
```python
828|     assert a.dtype.itemsize > 1
```
**EN:** Asserts `a.dtype.itemsize > 1` to enforce invariants.

**CN:** 断言 `a.dtype.itemsize > 1` 以确保不变量成立。

### Block 254 — Lines 829-829 (matmul_torch)
```python
829|     assert b.dtype.itemsize > 1
```
**EN:** Asserts `b.dtype.itemsize > 1` to enforce invariants.

**CN:** 断言 `b.dtype.itemsize > 1` 以确保不变量成立。

### Block 255 — Lines 830-833 (matmul_torch)
```python
830|     if is_input_batched:
831|         assert gather_indx is None, "gather not supported in batched mode"
832|         assert scatter_indx is None, "scatter not supported in batched mode"
833|         assert b.ndim == 3 and b.shape[0] == a.shape[0]
```
**EN:** Checks `is_input_batched`. The true branch mainly checks invariants; checks invariants.

**CN:** 检查 `is_input_batched`. 真分支主要检查不变量; 检查不变量.

### Block 256 — Lines 834-835 (matmul_torch)
```python
834|     if round_x is None:
835|         round_x = lambda x, idx: x
```
**EN:** Checks `round_x is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `round_x is None`. 真分支主要准备中间值.

### Block 257 — Lines 836-837 (matmul_torch)
```python
836|     if round_y is None:
837|         round_y = lambda x: x
```
**EN:** Checks `round_y is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `round_y is None`. 真分支主要准备中间值.

### Block 258 — Lines 838-839 (matmul_torch)
```python
838|     if bias is not None and bias.ndim == 1:
839|         bias = bias.view(1, *bias.shape)
```
**EN:** Checks `bias is not None and bias.ndim == 1`. The true branch mainly prepares intermediate values.

**CN:** 检查 `bias is not None and bias.ndim == 1`. 真分支主要准备中间值.

### Block 259 — Lines 840-841 (matmul_torch)
```python
840|     if b.ndim == 2:
841|         b = b.view(1, *b.shape)
```
**EN:** Checks `b.ndim == 2`. The true branch mainly prepares intermediate values.

**CN:** 检查 `b.ndim == 2`. 真分支主要准备中间值.

### Block 260 — Lines 842-844 (matmul_torch)
```python
842|     if a.ndim == 2:
843|         a = a.view(1, *a.shape)
844|     # memory offsets
```
**EN:** Checks `a.ndim == 2`. The true branch mainly prepares intermediate values.

**CN:** 检查 `a.ndim == 2`. 真分支主要准备中间值.

### Block 261 — Lines 845-852 (matmul_torch)
```python
845|     if a_ragged_metadata is not None and not is_input_batched:
846|         sizes = a_ragged_metadata.slice_sizes
847|         off = torch.zeros(sizes.shape[0] + 1, dtype=torch.int32)
848|         off[1:] = torch.cumsum(sizes, 0)
849|         offs = list(itertools.pairwise(off))
850|     else:
851|         offs = [[0, a.shape[1]] for _ in range(b.shape[0])]
852|     # compute
```
**EN:** Checks `a_ragged_metadata is not None and (not is_input_batched)`. The true branch mainly prepares intermediate values; prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `a_ragged_metadata is not None and (not is_input_batched)`. 真分支主要准备中间值; 准备中间值；而 else 分支准备中间值.

### Block 262 — Lines 853-853 (matmul_torch)
```python
853|     n_rows = a.shape[1] if gather_indx is None else gather_indx.shape[0]
```
**EN:** Assigns `n_rows` and uses conditional expression `a.shape[1] if gather_indx is None else gather_indx.shape[0]`.

**CN:** 将 `n_rows` and 使用条件表达式 `a.shape[1] if gather_indx is None else gather_indx.shape[0]`.

### Block 263 — Lines 854-854 (matmul_torch)
```python
854|     y = torch.zeros((a.shape[0], n_rows, b.shape[-1]), device=a.device, dtype=a.dtype)
```
**EN:** Assigns `y` and calls `torch.zeros`.

**CN:** 将 `y`，并调用 `torch.zeros`.

### Block 264 — Lines 855-867 (matmul_torch)
```python
855|     for i, (lo, hi) in enumerate(offs):
856|         if gather_indx is None:
857|             idx = torch.arange(lo, hi, device=a.device)
858|         else:
859|             idx = gather_indx[lo:hi]
860|         batch = i if is_input_batched else 0
861|         out = torch.matmul(round_x(a[batch, idx, :], torch.arange(lo, hi, device="cuda")).float(),
862|                            b[i].float())
863|         if bias is not None:
864|             out += bias[i, :] if betas is None else bias[i, :] * betas[lo:hi, None]
865|         if gammas is not None:
866|             out *= gammas[lo:hi, None]
867|         y[batch, lo:hi, :] = round_y(out)
```
**EN:** Loops over `enumerate(offs)` with target `(i, (lo, hi))`. The loop body mainly branches on runtime conditions; prepares intermediate values.

**CN:** 遍历 `enumerate(offs)` ，目标变量为 `(i, (lo, hi))`. 循环体主要根据运行时条件分支; 准备中间值.

### Block 265 — Lines 868-869 (matmul_torch)
```python
868|     if not is_input_batched:
869|         y = y.view(y.shape[1], y.shape[2])
```
**EN:** Checks `not is_input_batched`. The true branch mainly prepares intermediate values.

**CN:** 检查 `not is_input_batched`. 真分支主要准备中间值.

### Block 266 — Lines 870-875 (matmul_torch)
```python
870|     if scatter_indx is None:
871|         out = y
872|     else:
873|         out = torch.zeros((scatter_indx.shape[0], y.shape[-1]), dtype=y.dtype, device=a.device)
874|         msk = scatter_indx != -1
875|         out[scatter_indx[msk], :] = y[msk, :]
```
**EN:** Checks `scatter_indx is None`. The true branch mainly prepares intermediate values, while the else branch prepares intermediate values; prepares intermediate values.

**CN:** 检查 `scatter_indx is None`. 真分支主要准备中间值；而 else 分支准备中间值; 准备中间值.

### Block 267 — Lines 876-876 (matmul_torch)
```python
876|     actual_scale = precision_config.flex_ctx.out_data.actual_scale
```
**EN:** Assigns `actual_scale` and references `precision_config.flex_ctx.out_data.actual_scale`.

**CN:** 将 `actual_scale` and 引用 `precision_config.flex_ctx.out_data.actual_scale`.

### Block 268 — Lines 877-878 (matmul_torch)
```python
877|     if actual_scale is not None:
878|         actual_scale.copy_(compute_actual_scale(out, precision_config.out_dtype))
```
**EN:** Checks `actual_scale is not None`. The true branch mainly invokes `actual_scale.copy_`.

**CN:** 检查 `actual_scale is not None`. 真分支主要invokes `actual_scale.copy_`.

### Block 269 — Lines 879-881 (matmul_torch)
```python
879|     return scale(out, precision_config.flex_ctx.out_data.expected_scale)
880| 
881| 
```
**EN:** Returns `scale(out, precision_config.flex_ctx.out_data.expected_scale)`.

**CN:** 返回 `scale(out, precision_config.flex_ctx.out_data.expected_scale)`.

### Block 270 — Lines 882-885 (post_matmul_comm_torch)
```python
882| def post_matmul_comm_torch(y: torch.Tensor, rank: int, n_reduce_shards: int,
883|                            world_size: int,
884|                            scatter_shard_indx: torch.Tensor | None = None,
885| ):
```
**EN:** Defines function `post_matmul_comm_torch(y, rank, n_reduce_shards, world_size, scatter_shard_indx)` for this module. The body mainly prepares intermediate values; invokes `dist.all_gather`; prepares intermediate values. It uses calls such as `dist.all_gather`, `torch.empty_like`, `torch.cat(ys, dim=-1).reshape`, `torch.empty`, `torch.zeros` to implement its workflow.

**CN:** 定义函数 `post_matmul_comm_torch(y, rank, n_reduce_shards, world_size, scatter_shard_indx)`，供本模块使用. 主体主要准备中间值; invokes `dist.all_gather`; 准备中间值. 其中会调用 `dist.all_gather`, `torch.empty_like`, `torch.cat(ys, dim=-1).reshape`, `torch.empty`, `torch.zeros` 来实现其工作流程.

### Block 271 — Lines 886-898 (post_matmul_comm_torch)
```python
886|     """
887|     Reference implementation of post matmul communication.
888| 
889|     y: the local matmul output
890|     rank: the global rank
891|     n_reduce_shards: the number of reduce shards
892|     world_size: the world size
893|     scatter_shard_indx: the shard indices for the scatter. None if all gather.
894| 
895|     Output shape:
896|     (batch_size, n_rows, n_cols) -> (batch_size, n_rows * n_reduce_shards, n_cols) if batched, otherwise
897|     (n_rows, n_cols) -> (n_rows * n_reduce_shards, n_cols)
898|     """
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 272 — Lines 899-902 (post_matmul_comm_torch)
```python
899|     from torch import distributed as dist
900|     # if n_reduce_shards == 1:
901|     #     return y
902| 
```
**EN:** This block imports `torch (distributed)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `torch (distributed)` ，为模块提供所需的外部库和内部辅助工具。

### Block 273 — Lines 903-903 (post_matmul_comm_torch)
```python
903|     ys = [torch.empty_like(y) for _ in range(world_size)]
```
**EN:** Assigns `ys` and evaluates `[torch.empty_like(y) for _ in range(world_size)]`.

**CN:** 将 `ys` and 计算 `[torch.empty_like(y) for _ in range(world_size)]`.

### Block 274 — Lines 904-904 (post_matmul_comm_torch)
```python
904|     dist.all_gather(ys, y)
```
**EN:** Calls `dist.all_gather` for side effects, registration, or validation.

**CN:** 调用 `dist.all_gather` ，用于副作用、注册或校验。

### Block 275 — Lines 905-906 (post_matmul_comm_torch)
```python
905|     out_shape = (*y.shape[:-2], y.shape[-2] * n_reduce_shards, y.shape[-1])
906| 
```
**EN:** Assigns `out_shape` and builds a tuple.

**CN:** 将 `out_shape` and 构造一个元组.

### Block 276 — Lines 907-929 (post_matmul_comm_torch)
```python
907|     if scatter_shard_indx is None:
908|         # all gather
909|         assert n_reduce_shards == world_size
910|         return torch.cat(ys, dim=-1).reshape(out_shape)
911|     else:
912|         # Note: when multiple ranks scatter to the same destination, the result is undefined.
913|         scatter_shard_indx_global = torch.empty((world_size, *scatter_shard_indx.shape), device=scatter_shard_indx.device, dtype=scatter_shard_indx.dtype)
914|         dist.all_gather([scatter_shard_indx_global[i] for i in range(world_size)], scatter_shard_indx)
915| 
916|         assert len(out_shape) == 2, "batched mode not supported"
917|         result = torch.zeros(out_shape, device=y.device, dtype=y.dtype)
918|         reduce_shard_id = rank // n_reduce_shards
919| 
920|         for i in range(world_size // n_reduce_shards):
921|             scatter_mask = scatter_shard_indx_global[i * n_reduce_shards, :] == reduce_shard_id
922|             for j in range(n_reduce_shards):
923|                 out_slice = result.as_strided(
924|                     (result.shape[0] // n_reduce_shards, result.shape[1]),
925|                     (result.stride(0) * n_reduce_shards, result.stride(1)),
926|                     storage_offset=j * result.stride(0),
927|                 )
928|                 out_slice[scatter_mask, :] = ys[i * n_reduce_shards + j][scatter_mask, :]
929|         return result
```
**EN:** Checks `scatter_shard_indx is None`. The true branch mainly checks invariants; returns the computed result, while the else branch prepares intermediate values; invokes `dist.all_gather`.

**CN:** 检查 `scatter_shard_indx is None`. 真分支主要检查不变量; 返回计算结果；而 else 分支准备中间值; invokes `dist.all_gather`.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `FusedActivation`, `Epilogue`, `FnName`, `FusedComm`, `can_overflow_int32`, `should_upcast_indices`, `FlexCtx`, `PrecisionConfig`.
  **CN:** 主要符号：`FusedActivation`, `Epilogue`, `FnName`, `FusedComm`, `can_overflow_int32`, `should_upcast_indices`, `FlexCtx`, `PrecisionConfig`。
- **EN:** PyTorch tensors are the main data container and host-side orchestration layer.
  **CN:** PyTorch 张量是主要的数据容器和主机端编排层。
- **EN:** Triton is used to JIT-compile or launch GPU kernels and to express tile-level execution details.
  **CN:** Triton 用于 JIT 编译或启动 GPU 内核，并表达分块级执行细节。
- **EN:** Pytest markers, fixtures, and parametrization expand coverage across hardware and shape combinations.
  **CN:** Pytest 的标记、fixture 和参数化用于覆盖不同硬件与张量形状组合。
- **EN:** Mixed-precision MXFP/NVFP formats and scale tensors are part of the numeric pipeline.
  **CN:** 混合精度 MXFP/NVFP 格式及其缩放张量是数值流程的一部分。
- **EN:** Flex-point style metadata or scaling is used to control quantized arithmetic.
  **CN:** 使用 Flex-point 风格的元数据或缩放来控制量化计算。
- **EN:** Custom layout objects describe how logical tensors are packed or swizzled in memory.
  **CN:** 自定义 layout 对象描述逻辑张量如何在内存中打包或重排。
- **EN:** Distributed execution concepts such as meshes, ranks, or shard mapping appear in this file.
  **CN:** 该文件涉及 mesh、rank 或分片映射等分布式执行概念。

## Dependencies / 依赖关系
- **EN:** External modules: `dataclasses (dataclass)`, `itertools`, `torch`, `triton`, `enum (Enum, auto)`, `math`, `typing (Callable)`.
  **CN:** 外部模块：`dataclasses (dataclass)`, `itertools`, `torch`, `triton`, `enum (Enum, auto)`, `math`, `typing (Callable)`。
- **EN:** Internal modules: `triton_kernels (target_info)`, `triton_kernels.meta (Closure)`, `triton_kernels.numerics (InFlexData, OutFlexData)`, `triton_kernels.target_info (is_cuda)`, `triton_kernels.tensor_details.layout_details.hopper_scale (HopperMXScaleLayout)`, `.matmul_details._matmul (_matmul)`, `.matmul_details._p_matmul (_p_matmul, get_per_device_per_stream_alloc_fn)`, `.numerics_details.mxfp (MXFP_BLOCK_SIZE)`, `.numerics_details.mxfp_details._downcast_to_mxfp (NVFP_BLOCK_SIZE)`, `.tensor_details.layout_details.strided (StridedLayout)`, `.tensor_details.layout_details.blackwell_scale (BlackwellActMXScaleLayout, SWIZZLE_SIZE_OUTER)`, `.tensor_details.layout_details.blackwell_value_shuffled (BlackwellMX4ValueShuffledLayout)`.
  **CN:** 内部模块：`triton_kernels (target_info)`, `triton_kernels.meta (Closure)`, `triton_kernels.numerics (InFlexData, OutFlexData)`, `triton_kernels.target_info (is_cuda)`, `triton_kernels.tensor_details.layout_details.hopper_scale (HopperMXScaleLayout)`, `.matmul_details._matmul (_matmul)`, `.matmul_details._p_matmul (_p_matmul, get_per_device_per_stream_alloc_fn)`, `.numerics_details.mxfp (MXFP_BLOCK_SIZE)`, `.numerics_details.mxfp_details._downcast_to_mxfp (NVFP_BLOCK_SIZE)`, `.tensor_details.layout_details.strided (StridedLayout)`, `.tensor_details.layout_details.blackwell_scale (BlackwellActMXScaleLayout, SWIZZLE_SIZE_OUTER)`, `.tensor_details.layout_details.blackwell_value_shuffled (BlackwellMX4ValueShuffledLayout)`。
