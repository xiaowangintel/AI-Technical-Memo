# opt_flags.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/triton_kernels/matmul_details/opt_flags.py`
- **Purpose / 用途:** Implementation module for opt flags; it defines the runtime logic, helper types, and GPU-facing utilities used by the Triton kernels package via symbols OptFlags, max_allowable_mn, all_constraints_satisfied, make_default_opt_flags_amd, make_default_opt_flags_nvidia. / 用于 opt flags 的实现模块；它定义了 Triton kernels 包使用的运行时逻辑、辅助类型以及面向 GPU 的工具；核心符号包括 OptFlags、max_allowable_mn、all_constraints_satisfied、make_default_opt_flags_amd、make_default_opt_flags_nvidia。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-2 (module)
```python
1| # isort: off
2| # fmt: off
```
**EN:** This preamble contains file-level comments or formatting directives that keep imports and generated code stable.

**CN:** 这一前导部分包含文件级注释或格式化指令，用于保持导入顺序和生成代码稳定。

### Block 2 — Lines 3-17 (module)
```python
 3| from contextlib import contextmanager
 4| from contextvars import ContextVar
 5| from dataclasses import dataclass
 6| 
 7| import triton
 8| from triton_kernels import target_info
 9| from triton_kernels.target_info import get_cdna_version, get_rdna_version, cuda_capability_geq
10| from triton_kernels.tensor import FP4, FP32, Tensor, torch_dtype_to_dtype
11| import torch
12| from triton_kernels.tensor_details.layout_details.hopper_scale import HopperMXScaleLayout
13| from triton_kernels.tensor_details.layout_details.strided import StridedLayout
14| from triton_kernels.tensor_details.layout_details.base import Layout
15| from triton_kernels.tensor_details.layout_details.blackwell_value_shuffled import BlackwellMX4ValueShuffledLayout
16| from .opt_flags_details import opt_flags_amd, opt_flags_nvidia
17| 
```
**EN:** This block imports `contextlib (contextmanager)`, `contextvars (ContextVar)`, `dataclasses (dataclass)`, `triton`, `triton_kernels (target_info)`, `triton_kernels.target_info (get_cdna_version, get_rdna_version, cuda_capability_geq)`, `triton_kernels.tensor (FP4, FP32, Tensor, torch_dtype_to_dtype)`, `torch` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `contextlib (contextmanager)`, `contextvars (ContextVar)`, `dataclasses (dataclass)`, `triton`, `triton_kernels (target_info)`, `triton_kernels.target_info (get_cdna_version, get_rdna_version, cuda_capability_geq)`, `triton_kernels.tensor (FP4, FP32, Tensor, torch_dtype_to_dtype)`, `torch` ，为模块提供所需的外部库和内部辅助工具。

### Block 3 — Lines 18-19 (OptFlags)
```python
18| @dataclass
19| class OptFlags:
```
**EN:** Defines class `OptFlags` with decorators `dataclass` to organize related behavior. Key fields include `block_m`, `block_n`, `block_k`, `num_warps`, `num_stages`, `group_m`, `xcd_swizzle`, `w_cache_modifier`.

**CN:** 定义类 `OptFlags`，带有装饰器 `dataclass`，用于组织相关行为。关键字段包括 `block_m`, `block_n`, `block_k`, `num_warps`, `num_stages`, `group_m`, `xcd_swizzle`, `w_cache_modifier`.

### Block 4 — Lines 20-20 (OptFlags)
```python
20|     block_m: int
```
**EN:** Annotated assignment stores `block_m` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `block_m` and 声明该带注解的字段.

### Block 5 — Lines 21-21 (OptFlags)
```python
21|     block_n: int
```
**EN:** Annotated assignment stores `block_n` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `block_n` and 声明该带注解的字段.

### Block 6 — Lines 22-22 (OptFlags)
```python
22|     block_k: int
```
**EN:** Annotated assignment stores `block_k` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `block_k` and 声明该带注解的字段.

### Block 7 — Lines 23-23 (OptFlags)
```python
23|     num_warps: int
```
**EN:** Annotated assignment stores `num_warps` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `num_warps` and 声明该带注解的字段.

### Block 8 — Lines 24-24 (OptFlags)
```python
24|     num_stages: int
```
**EN:** Annotated assignment stores `num_stages` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `num_stages` and 声明该带注解的字段.

### Block 9 — Lines 25-25 (OptFlags)
```python
25|     group_m: int
```
**EN:** Annotated assignment stores `group_m` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `group_m` and 声明该带注解的字段.

### Block 10 — Lines 26-26 (OptFlags)
```python
26|     xcd_swizzle: int
```
**EN:** Annotated assignment stores `xcd_swizzle` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `xcd_swizzle` and 声明该带注解的字段.

### Block 11 — Lines 27-27 (OptFlags)
```python
27|     w_cache_modifier: str
```
**EN:** Annotated assignment stores `w_cache_modifier` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `w_cache_modifier` and 声明该带注解的字段.

### Block 12 — Lines 28-28 (OptFlags)
```python
28|     split_k: int
```
**EN:** Annotated assignment stores `split_k` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `split_k` and 声明该带注解的字段.

### Block 13 — Lines 29-29 (OptFlags)
```python
29|     is_persistent: bool
```
**EN:** Annotated assignment stores `is_persistent` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `is_persistent` and 声明该带注解的字段.

### Block 14 — Lines 30-30 (OptFlags)
```python
30|     idle_sms: int
```
**EN:** Annotated assignment stores `idle_sms` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `idle_sms` and 声明该带注解的字段.

### Block 15 — Lines 31-31 (OptFlags)
```python
31|     epilogue_subtile: int | None
```
**EN:** Annotated assignment stores `epilogue_subtile` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `epilogue_subtile` and 声明该带注解的字段.

### Block 16 — Lines 32-32 (OptFlags)
```python
32|     arch: str
```
**EN:** Annotated assignment stores `arch` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `arch` and 声明该带注解的字段.

### Block 17 — Lines 33-33 (OptFlags)
```python
33|     occupancy_target: int
```
**EN:** Annotated assignment stores `occupancy_target` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `occupancy_target` and 声明该带注解的字段.

### Block 18 — Lines 34-36 (OptFlags)
```python
34|     target_kernel_kwargs: dict
35| 
36| 
```
**EN:** Annotated assignment stores `target_kernel_kwargs` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `target_kernel_kwargs` and 声明该带注解的字段.

### Block 19 — Lines 37-42 (max_allowable_mn)
```python
37| def max_allowable_mn(
38|     max_mn: int,
39|     m: int,
40|     n: int,
41|     split_k: int,
42|     ) -> int:
```
**EN:** Defines function `max_allowable_mn(max_mn, m, n, split_k)` for this module. The body mainly returns the computed result.

**CN:** 定义函数 `max_allowable_mn(max_mn, m, n, split_k)`，供本模块使用. 主体主要返回计算结果.

### Block 20 — Lines 43-45 (max_allowable_mn)
```python
43|     return 1 if m * n >= max_mn else split_k
44| 
45| 
```
**EN:** Returns `1 if m * n >= max_mn else split_k`.

**CN:** 返回 `1 if m * n >= max_mn else split_k`.

### Block 21 — Lines 46-46 (all_constraints_satisfied)
```python
46| def all_constraints_satisfied(opt_flags: OptFlags, constraints: dict) -> bool:
```
**EN:** Defines function `all_constraints_satisfied(opt_flags, constraints)` for this module. The body mainly prepares intermediate values; checks invariants; branches on runtime conditions. It uses calls such as `all`, `constraints.get`, `getattr`, `constraints.items` to implement its workflow.

**CN:** 定义函数 `all_constraints_satisfied(opt_flags, constraints)`，供本模块使用. 主体主要准备中间值; 检查不变量; 根据运行时条件分支. 其中会调用 `all`, `constraints.get`, `getattr`, `constraints.items` 来实现其工作流程.

### Block 22 — Lines 47-47 (all_constraints_satisfied)
```python
47|     _split_k_constraints = ['split_k', 'max_allowable_mn', 'disable_mx4_block_swap']
```
**EN:** Assigns `_split_k_constraints` and builds a list.

**CN:** 将 `_split_k_constraints` and 构造一个列表.

### Block 23 — Lines 48-48 (all_constraints_satisfied)
```python
48|     assert all(getattr(opt_flags, ck) == cv for ck, cv in constraints.items() if cv is not None and ck not in _split_k_constraints)
```
**EN:** Asserts `all((getattr(opt_flags, ck) == cv for ck, cv in constraints.items() if cv is not None a...` to enforce invariants.

**CN:** 断言 `all((getattr(opt_flags, ck) == cv for ck, cv in constraints.items() if cv is not None a...` 以确保不变量成立。

### Block 24 — Lines 49-52 (all_constraints_satisfied)
```python
49|     if constraints.get('split_k') and not constraints.get('max_allowable_mn'):
50|         assert opt_flags.split_k == constraints['split_k']
51| 
52| 
```
**EN:** Checks `constraints.get('split_k') and (not constraints.get('max_allowable_mn'))`. The true branch mainly checks invariants.

**CN:** 检查 `constraints.get('split_k') and (not constraints.get('max_allowable_mn'))`. 真分支主要检查不变量.

### Block 25 — Lines 53-70 (make_default_opt_flags_amd)
```python
53| def make_default_opt_flags_amd(
54|     out_dtype,
55|     lhs_dtype,
56|     rhs_dtype,
57|     precision_config,
58|     batch_size,
59|     m,
60|     n,
61|     k,
62|     ragged_metadata,
63|     can_use_persistent_tma,
64|     can_use_split_k,
65|     enforce_bitwise_invariance,
66|     epilogue_effective_itemsize,
67|     x_transpose,
68|     has_y_acc_in,
69|     constraints,
70| ):
```
**EN:** Defines function `make_default_opt_flags_amd(out_dtype, lhs_dtype, rhs_dtype, precision_config, batch_size, m, n, k, ragged_metadata, can_use_persistent_tma, can_use_split_k, enforce_bitwise_invariance, epilogue_effective_itemsize, x_transpose, has_y_acc_in, constraints)` for this module. The body mainly prepares intermediate values; prepares intermediate values; checks invariants. It uses calls such as `constraints.get`, `opt_flags_amd.compute_block_nk`, `OptFlags`, `all_constraints_satisfied`, `set` to implement its workflow.

**CN:** 定义函数 `make_default_opt_flags_amd(out_dtype, lhs_dtype, rhs_dtype, precision_config, batch_size, m, n, k, ragged_metadata, can_use_persistent_tma, can_use_split_k, enforce_bitwise_invariance, epilogue_effective_itemsize, x_transpose, has_y_acc_in, constraints)`，供本模块使用. 主体主要准备中间值; 准备中间值; 检查不变量. 其中会调用 `constraints.get`, `opt_flags_amd.compute_block_nk`, `OptFlags`, `all_constraints_satisfied`, `set` 来实现其工作流程.

### Block 26 — Lines 71-71 (make_default_opt_flags_amd)
```python
71|     constraints_supported = {"block_m", "block_n", "block_k", "split_k", "is_persistent", "epilogue_subtile", "max_allowable_mn", "num_warps", "disable_mx4_block_swap"}
```
**EN:** Assigns `constraints_supported` and evaluates `{'block_m', 'block_n', 'block_k', 'split_k', 'is_persistent', 'epil...`.

**CN:** 将 `constraints_supported` and 计算 `{'block_m', 'block_n', 'block_k', 'split_k', 'is_persistent', 'epil...`.

### Block 27 — Lines 72-72 (make_default_opt_flags_amd)
```python
72|     unsupported = set(constraints.keys()) - constraints_supported
```
**EN:** Assigns `unsupported` and evaluates `set(constraints.keys()) - constraints_supported`.

**CN:** 将 `unsupported` and 计算 `set(constraints.keys()) - constraints_supported`.

### Block 28 — Lines 73-74 (make_default_opt_flags_amd)
```python
73|     assert not unsupported, f"Given unsupported constraint: {unsupported}"
74|     # tokens per slice
```
**EN:** Asserts `not unsupported` to enforce invariants.

**CN:** 断言 `not unsupported` 以确保不变量成立。

### Block 29 — Lines 75-81 (make_default_opt_flags_amd)
```python
75|     if ragged_metadata is None:
76|         slice_size = m
77|     elif ragged_metadata.expected_slice_size is None:
78|         slice_size = max(1, m // ragged_metadata.n_slices)
79|     else:
80|         slice_size = ragged_metadata.expected_slice_size
81| 
```
**EN:** Checks `ragged_metadata is None`. The true branch mainly prepares intermediate values, while the else branch branches on runtime conditions.

**CN:** 检查 `ragged_metadata is None`. 真分支主要准备中间值；而 else 分支根据运行时条件分支.

### Block 30 — Lines 82-83 (make_default_opt_flags_amd)
```python
82|     is_cdna4 = get_cdna_version() == 4
83|     # block_m
```
**EN:** Assigns `is_cdna4` and evaluates `get_cdna_version() == 4`.

**CN:** 将 `is_cdna4` and 计算 `get_cdna_version() == 4`.

### Block 31 — Lines 84-96 (make_default_opt_flags_amd)
```python
84|     if constraints.get("block_m", None):
85|         block_m = constraints["block_m"]
86|     elif enforce_bitwise_invariance:
87|         block_m = 256 if is_cdna4 else 128
88|     elif slice_size >= 512 and n >= 2048:
89|         block_m = 256 if is_cdna4 else 128
90|     elif is_cdna4 and m >= 512:
91|         block_m = 128
92|     elif get_rdna_version() in (3, 4) and m >= 512:
93|         block_m = 64
94|     else:
95|         block_m = max(32, min(triton.next_power_of_2(slice_size), 64))
96| 
```
**EN:** Checks `constraints.get('block_m', None)`. The true branch mainly prepares intermediate values, while the else branch branches on runtime conditions.

**CN:** 检查 `constraints.get('block_m', None)`. 真分支主要准备中间值；而 else 分支根据运行时条件分支.

### Block 32 — Lines 97-101 (make_default_opt_flags_amd)
```python
 97|     if ragged_metadata is not None:
 98|         grid_m = ragged_metadata.n_blocks(ragged_metadata.n_slices, m, block_m)
 99|     else:
100|         grid_m = triton.cdiv(m, block_m)
101|     # group_m:
```
**EN:** Checks `ragged_metadata is not None`. The true branch mainly prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `ragged_metadata is not None`. 真分支主要准备中间值；而 else 分支准备中间值.

### Block 33 — Lines 102-103 (make_default_opt_flags_amd)
```python
102|     group_m = 4
103|     # number of xcds
```
**EN:** Assigns `group_m` and stores constant `4`.

**CN:** 将 `group_m` and 保存常量 `4`.

### Block 34 — Lines 104-104 (make_default_opt_flags_amd)
```python
104|     num_xcds = 8
```
**EN:** Assigns `num_xcds` and stores constant `8`.

**CN:** 将 `num_xcds` and 保存常量 `8`.

### Block 35 — Lines 105-107 (make_default_opt_flags_amd)
```python
105|     xcd_swizzle = num_xcds
106|     # block_nk:
107|     # TODO: Does opt_flags_amd.compute_block_nk need to be refactored?
```
**EN:** Assigns `xcd_swizzle` and references `num_xcds`.

**CN:** 将 `xcd_swizzle` and 引用 `num_xcds`.

### Block 36 — Lines 108-110 (make_default_opt_flags_amd)
```python
108|     block_n, block_k = opt_flags_amd.compute_block_nk(
109|         n, block_m, grid_m, num_xcds, lhs_dtype, rhs_dtype, precision_config
110|     )
```
**EN:** Assigns `block_n`, `block_k` and calls `opt_flags_amd.compute_block_nk`.

**CN:** 将 `block_n`, `block_k`，并调用 `opt_flags_amd.compute_block_nk`.

### Block 37 — Lines 111-112 (make_default_opt_flags_amd)
```python
111|     is_persistent = constraints.get("is_persistent", False)
112|     # split_k:
```
**EN:** Assigns `is_persistent` and calls `constraints.get`.

**CN:** 将 `is_persistent`，并调用 `constraints.get`.

### Block 38 — Lines 113-113 (make_default_opt_flags_amd)
```python
113|     split_k = 1
```
**EN:** Assigns `split_k` and stores constant `1`.

**CN:** 将 `split_k` and 保存常量 `1`.

### Block 39 — Lines 114-122 (make_default_opt_flags_amd)
```python
114|     if constraints.get("max_allowable_mn", 0) > 0 and constraints.get("split_k") is not None:
115|         split_k = max_allowable_mn(constraints["max_allowable_mn"], m, n, constraints.get("split_k"))
116|     elif constraints.get("split_k", None) is not None:
117|         split_k = constraints["split_k"]
118|     elif can_use_split_k and not enforce_bitwise_invariance:
119|         grid_size = grid_m * ((n + block_n - 1) // block_n)
120|         n_cu = torch.cuda.get_device_properties(0).multi_processor_count
121|         split_k = max(1, n_cu // grid_size)
122|     # w_cache_modifier:
```
**EN:** Checks `constraints.get('max_allowable_mn', 0) > 0 and constraints.get('split_k') is not None`. The true branch mainly prepares intermediate values, while the else branch branches on runtime conditions.

**CN:** 检查 `constraints.get('max_allowable_mn', 0) > 0 and constraints.get('split_k') is not None`. 真分支主要准备中间值；而 else 分支根据运行时条件分支.

### Block 40 — Lines 123-124 (make_default_opt_flags_amd)
```python
123|     w_cache_modifier = ".cg" if block_m <= 32 else None
124|     # num_warps, num_stages
```
**EN:** Assigns `w_cache_modifier` and uses conditional expression `'.cg' if block_m <= 32 else None`.

**CN:** 将 `w_cache_modifier` and 使用条件表达式 `'.cg' if block_m <= 32 else None`.

### Block 41 — Lines 125-125 (make_default_opt_flags_amd)
```python
125|     num_warps = 2 if (m is not None and m <= 16) else 8
```
**EN:** Assigns `num_warps` and uses conditional expression `2 if m is not None and m <= 16 else 8`.

**CN:** 将 `num_warps` and 使用条件表达式 `2 if m is not None and m <= 16 else 8`.

### Block 42 — Lines 126-127 (make_default_opt_flags_amd)
```python
126|     num_stages = 2
127|     # AMD-specific
```
**EN:** Assigns `num_stages` and stores constant `2`.

**CN:** 将 `num_stages` and 保存常量 `2`.

### Block 43 — Lines 128-128 (make_default_opt_flags_amd)
```python
128|     target_kernel_kwargs = {"waves_per_eu": 0, "matrix_instr_nonkdim": 16, "kpack": 1}
```
**EN:** Assigns `target_kernel_kwargs` and builds a dictionary.

**CN:** 将 `target_kernel_kwargs` and 构造一个字典.

### Block 44 — Lines 129-129 (make_default_opt_flags_amd)
```python
129|     epilogue_subtile = constraints.get('epilogue_subtile', None)
```
**EN:** Assigns `epilogue_subtile` and calls `constraints.get`.

**CN:** 将 `epilogue_subtile`，并调用 `constraints.get`.

### Block 45 — Lines 130-133 (make_default_opt_flags_amd)
```python
130|     if epilogue_subtile is None:
131|         epilogue_subtile = 1
132| 
133|     # prevents OutOfSharedMemoryError for mxfp8 on CDNA3
```
**EN:** Checks `epilogue_subtile is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `epilogue_subtile is None`. 真分支主要准备中间值.

### Block 46 — Lines 134-137 (make_default_opt_flags_amd)
```python
134|     if get_cdna_version() == 3 and rhs_dtype.bitwidth == 8 and precision_config.b_mx_scale is not None:
135|         num_stages = 1
136| 
137|     # specific configs for F16 x MXFP4 on CDNA4
```
**EN:** Checks `get_cdna_version() == 3 and rhs_dtype.bitwidth == 8 and (precision_config.b_mx_scale is...`. The true branch mainly prepares intermediate values.

**CN:** 检查 `get_cdna_version() == 3 and rhs_dtype.bitwidth == 8 and (precision_config.b_mx_scale is...`. 真分支主要准备中间值.

### Block 47 — Lines 138-151 (make_default_opt_flags_amd)
```python
138|     if is_cdna4 and lhs_dtype.bitwidth == 16 and rhs_dtype.bitwidth == 4 and precision_config.b_mx_scale is not None:
139|         split_k = 1
140|         if m <= 1024:
141|             target_kernel_kwargs["waves_per_eu"] = 3
142|             block_n = 128
143|             block_k = 128
144|             num_warps = 4
145|         else:
146|             target_kernel_kwargs["waves_per_eu"] = 0
147|             block_m = 64
148|             block_n = 512
149|             block_k = 256
150|             num_warps = 8
151| 
```
**EN:** Checks `is_cdna4 and lhs_dtype.bitwidth == 16 and (rhs_dtype.bitwidth == 4) and (precision_conf...`. The true branch mainly prepares intermediate values; branches on runtime conditions.

**CN:** 检查 `is_cdna4 and lhs_dtype.bitwidth == 16 and (rhs_dtype.bitwidth == 4) and (precision_conf...`. 真分支主要准备中间值; 根据运行时条件分支.

### Block 48 — Lines 152-152 (replace_with_valid_constraint)
```python
152|     def replace_with_valid_constraint(k: str, v):
```
**EN:** Defines function `replace_with_valid_constraint(k, v)` for this module. The body mainly branches on runtime conditions. It uses calls such as `constraints.get` to implement its workflow.

**CN:** 定义函数 `replace_with_valid_constraint(k, v)`，供本模块使用. 主体主要根据运行时条件分支. 其中会调用 `constraints.get` 来实现其工作流程.

### Block 49 — Lines 153-157 (replace_with_valid_constraint)
```python
153|         if constraints.get(k, None) is not None:
154|             return constraints[k]
155|         else:
156|             return v
157| 
```
**EN:** Checks `constraints.get(k, None) is not None`. The true branch mainly returns the computed result, while the else branch returns the computed result.

**CN:** 检查 `constraints.get(k, None) is not None`. 真分支主要返回计算结果；而 else 分支返回计算结果.

### Block 50 — Lines 158-175 (make_default_opt_flags_amd)
```python
158|     ret = OptFlags(
159|         block_m=replace_with_valid_constraint('block_m', block_m),
160|         block_n=replace_with_valid_constraint('block_n', block_n),
161|         block_k=replace_with_valid_constraint('block_k', block_k),
162|         num_warps=replace_with_valid_constraint('num_warps', num_warps),
163|         num_stages=num_stages,
164|         group_m=group_m,
165|         xcd_swizzle=xcd_swizzle,
166|         w_cache_modifier=w_cache_modifier,
167|         split_k=split_k,
168|         is_persistent=is_persistent,
169|         idle_sms=0,
170|         epilogue_subtile=epilogue_subtile,
171|         arch=None,
172|         target_kernel_kwargs=target_kernel_kwargs,
173|         occupancy_target=1,
174|     )
175|     # check constraints
```
**EN:** Assigns `ret` and calls `OptFlags`.

**CN:** 将 `ret`，并调用 `OptFlags`.

### Block 51 — Lines 176-176 (make_default_opt_flags_amd)
```python
176|     all_constraints_satisfied(ret, constraints)
```
**EN:** Calls `all_constraints_satisfied` for side effects, registration, or validation.

**CN:** 调用 `all_constraints_satisfied` ，用于副作用、注册或校验。

### Block 52 — Lines 177-178 (make_default_opt_flags_amd)
```python
177|     return ret
178| 
```
**EN:** Returns `ret`.

**CN:** 返回 `ret`.

### Block 53 — Lines 179-199 (make_default_opt_flags_nvidia)
```python
179| def make_default_opt_flags_nvidia(
180|     out_dtype,
181|     lhs_dtype,
182|     rhs_dtype,
183|     precision_config,
184|     batch_size,
185|     m,
186|     n,
187|     k,
188|     routing_data,
189|     can_use_persistent_tma,
190|     can_use_split_k,
191|     enforce_bitwise_invariance,
192|     epilogue_effective_itemsize,
193|     x_transpose,
194|     has_y_acc_in,
195|     constraints,
196|     x_uses_tma_when_persistent=True,
197|     mx_block_size=None,
198|     epilogue_reduction_n=1,
199| ):
```
**EN:** Defines function `make_default_opt_flags_nvidia(out_dtype, lhs_dtype, rhs_dtype, precision_config, batch_size, m, n, k, routing_data, can_use_persistent_tma, can_use_split_k, enforce_bitwise_invariance, epilogue_effective_itemsize, x_transpose, has_y_acc_in, constraints, x_uses_tma_when_persistent, mx_block_size, epilogue_reduction_n)` for this module. The body mainly prepares intermediate values; prepares intermediate values; checks invariants. It uses calls such as `constraints.get`, `opt_flags_nvidia.compute_grid_size`, `opt_flags_nvidia.compute_num_warps`, `isinstance`, `cuda_capability_geq` to implement its workflow.

**CN:** 定义函数 `make_default_opt_flags_nvidia(out_dtype, lhs_dtype, rhs_dtype, precision_config, batch_size, m, n, k, routing_data, can_use_persistent_tma, can_use_split_k, enforce_bitwise_invariance, epilogue_effective_itemsize, x_transpose, has_y_acc_in, constraints, x_uses_tma_when_persistent, mx_block_size, epilogue_reduction_n)`，供本模块使用. 主体主要准备中间值; 准备中间值; 检查不变量. 其中会调用 `constraints.get`, `opt_flags_nvidia.compute_grid_size`, `opt_flags_nvidia.compute_num_warps`, `isinstance`, `cuda_capability_geq` 来实现其工作流程.

### Block 54 — Lines 200-200 (make_default_opt_flags_nvidia)
```python
200|     constraints_supported = {"block_m", "block_n", "block_k", "split_k", "is_persistent", "epilogue_subtile", "num_stages", "idle_sms", "max_allowable_mn", "num_warps", "disable_mx4_block_swap"}
```
**EN:** Assigns `constraints_supported` and evaluates `{'block_m', 'block_n', 'block_k', 'split_k', 'is_persistent', 'epil...`.

**CN:** 将 `constraints_supported` and 计算 `{'block_m', 'block_n', 'block_k', 'split_k', 'is_persistent', 'epil...`.

### Block 55 — Lines 201-201 (make_default_opt_flags_nvidia)
```python
201|     unsupported = set(constraints.keys()) - constraints_supported
```
**EN:** Assigns `unsupported` and evaluates `set(constraints.keys()) - constraints_supported`.

**CN:** 将 `unsupported` and 计算 `set(constraints.keys()) - constraints_supported`.

### Block 56 — Lines 202-203 (make_default_opt_flags_nvidia)
```python
202|     assert not unsupported, f"Given unsupported constraint: {unsupported}"
203|     # tokens per expert
```
**EN:** Asserts `not unsupported` to enforce invariants.

**CN:** 断言 `not unsupported` 以确保不变量成立。

### Block 57 — Lines 204-210 (make_default_opt_flags_nvidia)
```python
204|     if routing_data is None or batch_size > 1:
205|         slice_size = m
206|     elif routing_data.expected_slice_size is None:
207|         slice_size = max(1, m // routing_data.n_slices)
208|     else:
209|         slice_size = routing_data.expected_slice_size
210|     # pid swizzling
```
**EN:** Checks `routing_data is None or batch_size > 1`. The true branch mainly prepares intermediate values, while the else branch branches on runtime conditions.

**CN:** 检查 `routing_data is None or batch_size > 1`. 真分支主要准备中间值；而 else 分支根据运行时条件分支.

### Block 58 — Lines 211-211 (make_default_opt_flags_nvidia)
```python
211|     group_m = 8
```
**EN:** Assigns `group_m` and stores constant `8`.

**CN:** 将 `group_m` and 保存常量 `8`.

### Block 59 — Lines 212-213 (make_default_opt_flags_nvidia)
```python
212|     if lhs_dtype == FP4 and rhs_dtype == FP4:
213|         group_m = 16
```
**EN:** Checks `lhs_dtype == FP4 and rhs_dtype == FP4`. The true branch mainly prepares intermediate values.

**CN:** 检查 `lhs_dtype == FP4 and rhs_dtype == FP4`. 真分支主要准备中间值.

### Block 60 — Lines 214-215 (make_default_opt_flags_nvidia)
```python
214|     xcd_swizzle = 1
215|     # block_m
```
**EN:** Assigns `xcd_swizzle` and stores constant `1`.

**CN:** 将 `xcd_swizzle` and 保存常量 `1`.

### Block 61 — Lines 216-237 (make_default_opt_flags_nvidia)
```python
216|     if constraints.get("block_m", None):
217|         block_m = constraints["block_m"]
218|     elif enforce_bitwise_invariance:
219|         block_m = 128
220|     else:
221|         if slice_size <= 64 and routing_data is not None and routing_data.slice_sizes is not None:
222|             # Ragged and likely memory bound; set the block size higher to minimize loading weights more than once.
223|             if (
224|                 lhs_dtype.bitwidth == 16
225|                 and rhs_dtype.bitwidth == 4
226|                 and slice_size >= 16
227|                 and torch.cuda.get_device_capability()[0] >= 10
228|             ):
229|                 block_m = max(16, min(triton.next_power_of_2(8 * slice_size), 128))
230|             else:
231|                 block_m = max(16, min(triton.next_power_of_2(2 * slice_size), 64))
232|             if block_m == 64 and precision_config.c_mx_scale is not None and rhs_dtype == FP4 and torch.cuda.get_device_capability()[0] >= 10:
233|                 # when having both fused_activation and mxfp8 downcast in epilogue, block_m=64 causing shared memory overflow
234|                 block_m = 128
235|         else:
236|             block_m = max(16, min(triton.next_power_of_2(slice_size), 128))
237|     # block n
```
**EN:** Checks `constraints.get('block_m', None)`. The true branch mainly prepares intermediate values, while the else branch branches on runtime conditions.

**CN:** 检查 `constraints.get('block_m', None)`. 真分支主要准备中间值；而 else 分支根据运行时条件分支.

### Block 62 — Lines 238-238 (make_default_opt_flags_nvidia)
```python
238|     arch = None
```
**EN:** Assigns `arch` and stores constant `None`.

**CN:** 将 `arch` and 保存常量 `None`.

### Block 63 — Lines 239-246 (make_default_opt_flags_nvidia)
```python
239|     if constraints.get("block_n", None) is not None:
240|         # A single block_n constraint should apply consistently to both the
241|         # regular and persistent/TMA paths.
242|         block_n = constraints["block_n"]
243|         block_n_tma = constraints["block_n"]
244|     else:
245|         block_n, block_n_tma = opt_flags_nvidia.compute_block_n(n, arch, precision_config)
246|     # is_persistent
```
**EN:** Checks `constraints.get('block_n', None) is not None`. The true branch mainly prepares intermediate values; prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `constraints.get('block_n', None) is not None`. 真分支主要准备中间值; 准备中间值；而 else 分支准备中间值.

### Block 64 — Lines 247-247 (make_default_opt_flags_nvidia)
```python
247|     grid_size_tma = opt_flags_nvidia.compute_grid_size(routing_data, batch_size, m, n, block_m, block_n_tma)
```
**EN:** Assigns `grid_size_tma` and calls `opt_flags_nvidia.compute_grid_size`.

**CN:** 将 `grid_size_tma`，并调用 `opt_flags_nvidia.compute_grid_size`.

### Block 65 — Lines 248-248 (make_default_opt_flags_nvidia)
```python
248|     n_sms = torch.cuda.get_device_properties(0).multi_processor_count
```
**EN:** Assigns `n_sms` and references `torch.cuda.get_device_properties(0).multi_processor_count`.

**CN:** 将 `n_sms` and 引用 `torch.cuda.get_device_properties(0).multi_processor_count`.

### Block 66 — Lines 249-249 (make_default_opt_flags_nvidia)
```python
249|     tiles_per_sm = grid_size_tma / n_sms
```
**EN:** Assigns `tiles_per_sm` and evaluates `grid_size_tma / n_sms`.

**CN:** 将 `tiles_per_sm` and 计算 `grid_size_tma / n_sms`.

### Block 67 — Lines 250-250 (make_default_opt_flags_nvidia)
```python
250|     supports_persistent = can_use_persistent_tma and (arch is None or int(arch[2:-1]) >= 9)
```
**EN:** Assigns `supports_persistent` and evaluates `can_use_persistent_tma and (arch is None or int(arch[2:-1]) >= 9)`.

**CN:** 将 `supports_persistent` and 计算 `can_use_persistent_tma and (arch is None or int(arch[2:-1]) >= 9)`.

### Block 68 — Lines 251-251 (make_default_opt_flags_nvidia)
```python
251|     a_mx_scale_layout = None if not isinstance(precision_config.a_mx_scale, Tensor) else precision_config.a_mx_scale.storage.layout
```
**EN:** Assigns `a_mx_scale_layout` and uses conditional expression `None if not isinstance(precision_config.a_mx_scale, Tenso...`.

**CN:** 将 `a_mx_scale_layout` and 使用条件表达式 `None if not isinstance(precision_config.a_mx_scale, Tenso...`.

### Block 69 — Lines 252-253 (make_default_opt_flags_nvidia)
```python
252|     b_mx_scale_layout = None if not isinstance(precision_config.b_mx_scale, Tensor) else precision_config.b_mx_scale.storage.layout
253| 
```
**EN:** Assigns `b_mx_scale_layout` and uses conditional expression `None if not isinstance(precision_config.b_mx_scale, Tenso...`.

**CN:** 将 `b_mx_scale_layout` and 使用条件表达式 `None if not isinstance(precision_config.b_mx_scale, Tenso...`.

### Block 70 — Lines 254-254 (_is_layout_strided)
```python
254|     def _is_layout_strided(layout: Layout | None) -> bool:
```
**EN:** Defines function `_is_layout_strided(layout)` for this module. The body mainly returns the computed result. It uses calls such as `isinstance` to implement its workflow.

**CN:** 定义函数 `_is_layout_strided(layout)`，供本模块使用. 主体主要返回计算结果. 其中会调用 `isinstance` 来实现其工作流程.

### Block 71 — Lines 255-256 (_is_layout_strided)
```python
255|         return layout is None or isinstance(layout, StridedLayout)
256| 
```
**EN:** Returns `layout is None or isinstance(layout, StridedLayout)`.

**CN:** 返回 `layout is None or isinstance(layout, StridedLayout)`.

### Block 72 — Lines 257-257 (make_default_opt_flags_nvidia)
```python
257|     requires_persistent = (not _is_layout_strided(a_mx_scale_layout) or not _is_layout_strided(b_mx_scale_layout)) and target_info.has_native_mxfp()
```
**EN:** Assigns `requires_persistent` and evaluates `(not _is_layout_strided(a_mx_scale_layout) or not _is_layout_stride...`.

**CN:** 将 `requires_persistent` and 计算 `(not _is_layout_strided(a_mx_scale_layout) or not _is_layout_stride...`.

### Block 73 — Lines 258-275 (make_default_opt_flags_nvidia)
```python
258|     if constraints.get("is_persistent", None) is not None:
259|         if requires_persistent and not constraints["is_persistent"]:
260|             raise InapplicableConstraint("cannot enforce `is_persistent=False` constraint because persistent kernel is required")
261|         is_persistent = constraints["is_persistent"]
262|     elif requires_persistent:
263|         assert supports_persistent, "persistent kernel required but not supported"
264|         is_persistent = True
265|     else:
266|         has_simple_epilogue = precision_config.max_num_imprecise_acc is None
267|         is_persistent = supports_persistent and has_simple_epilogue and (tiles_per_sm >= 2.0 or lhs_dtype.bitwidth <= 8) and (out_dtype.bitwidth < 32 or lhs_dtype.bitwidth == 32 or rhs_dtype.bitwidth == 32)
268|         # TMA is slower for batched matmuls with small m/n/k.
269|         if m * n * k < 131072:
270|             is_persistent = False
271|         if isinstance(b_mx_scale_layout, HopperMXScaleLayout):
272|             # TODO: persistent kernel is currently slower than non-persistent
273|             is_persistent = False
274| 
275|     # adjust block_n based on is_persistent signal
```
**EN:** Checks `constraints.get('is_persistent', None) is not None`. The true branch mainly branches on runtime conditions; prepares intermediate values, while the else branch branches on runtime conditions.

**CN:** 检查 `constraints.get('is_persistent', None) is not None`. 真分支主要根据运行时条件分支; 准备中间值；而 else 分支根据运行时条件分支.

### Block 74 — Lines 276-276 (make_default_opt_flags_nvidia)
```python
276|     block_n = block_n_tma if is_persistent else block_n
```
**EN:** Assigns `block_n` and uses conditional expression `block_n_tma if is_persistent else block_n`.

**CN:** 将 `block_n` and 使用条件表达式 `block_n_tma if is_persistent else block_n`.

### Block 75 — Lines 277-283 (make_default_opt_flags_nvidia)
```python
277|     if (is_persistent and constraints.get("block_n", None) is None
278|             and cuda_capability_geq(10, 0) and (lhs_dtype == FP32 or rhs_dtype == FP32)
279|             and not x_uses_tma_when_persistent):
280|         # Blackwell's fp32/tf32 persistent dot stages an operand in TMEM in
281|         # addition to the accumulator. A 128x256 accumulator already consumes
282|         # the full 512-column TMEM budget, so leave headroom for that operand.
283|         block_n = min(block_n, 128)
```
**EN:** Checks `is_persistent and constraints.get('block_n', None) is None and cuda_capability_geq(10, ...`. The true branch mainly prepares intermediate values.

**CN:** 检查 `is_persistent and constraints.get('block_n', None) is None and cuda_capability_geq(10, ...`. 真分支主要准备中间值.

### Block 76 — Lines 284-290 (make_default_opt_flags_nvidia)
```python
284|     if (is_persistent and constraints.get("block_n", None) is None
285|             and opt_flags_nvidia.is_blackwell_mx_lhs_dense_rhs(precision_config, lhs_dtype, rhs_dtype)):
286|         # Native Blackwell MX lhs + dense rhs persistent dots also stage an
287|         # expanded operand in TMEM, so keep the accumulator tile below the
288|         # 512-column budget.
289|         block_n = min(block_n, 128)
290|     # adjust block_m based on is_persistent signal
```
**EN:** Checks `is_persistent and constraints.get('block_n', None) is None and opt_flags_nvidia.is_blac...`. The true branch mainly prepares intermediate values.

**CN:** 检查 `is_persistent and constraints.get('block_n', None) is None and opt_flags_nvidia.is_blac...`. 真分支主要准备中间值.

### Block 77 — Lines 291-294 (make_default_opt_flags_nvidia)
```python
291|     if is_persistent and opt_flags_nvidia.is_x_scale_swizzled(precision_config):
292|         # a mx scale has been swizzled to BlackwellActMXScaleLayout, enforce block_m=128 to align with swizzling layout
293|         block_m = 128
294|     # block k
```
**EN:** Checks `is_persistent and opt_flags_nvidia.is_x_scale_swizzled(precision_config)`. The true branch mainly prepares intermediate values.

**CN:** 检查 `is_persistent and opt_flags_nvidia.is_x_scale_swizzled(precision_config)`. 真分支主要准备中间值.

### Block 78 — Lines 295-298 (make_default_opt_flags_nvidia)
```python
295|     if constraints.get("block_k", None) is not None:
296|         block_k = constraints["block_k"]
297|     else:
298|         block_k = opt_flags_nvidia.compute_block_k(m, k, is_persistent, lhs_dtype, rhs_dtype, precision_config, has_y_acc_in)
```
**EN:** Checks `constraints.get('block_k', None) is not None`. The true branch mainly prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `constraints.get('block_k', None) is not None`. 真分支主要准备中间值；而 else 分支准备中间值.

### Block 79 — Lines 299-303 (make_default_opt_flags_nvidia)
```python
299|     if block_n == 256 and block_k == 128 and block_m <= 64 and is_persistent and rhs_dtype == FP4 and k >= 4096 and slice_size > 1 and lhs_dtype != torch.bfloat16 and not constraints.get("disable_mx4_block_swap", False):
300|         # Swap block_n and block_k for mxfp4 weights so that block_k is a full cacheline, so long as K is sufficiently large.
301|         # TODO: swizzle the HBM layout of the weights instead
302|         block_n, block_k = block_k, block_n
303|     # split_k
```
**EN:** Checks `block_n == 256 and block_k == 128 and (block_m <= 64) and is_persistent and (rhs_dtype ...`. The true branch mainly prepares intermediate values.

**CN:** 检查 `block_n == 256 and block_k == 128 and (block_m <= 64) and is_persistent and (rhs_dtype ...`. 真分支主要准备中间值.

### Block 80 — Lines 304-304 (make_default_opt_flags_nvidia)
```python
304|     split_k = 1
```
**EN:** Assigns `split_k` and stores constant `1`.

**CN:** 将 `split_k` and 保存常量 `1`.

### Block 81 — Lines 305-311 (make_default_opt_flags_nvidia)
```python
305|     if constraints.get("max_allowable_mn", 0) > 0 and constraints.get("split_k") is not None:
306|         split_k = max_allowable_mn(constraints["max_allowable_mn"], m, n, constraints.get("split_k"))
307|     elif constraints.get("split_k", None) is not None:
308|         split_k = constraints["split_k"]
309|     elif can_use_split_k and not enforce_bitwise_invariance:
310|         estimated_actual_grid_size = opt_flags_nvidia.compute_grid_size(None, batch_size, m, n, block_m, block_n)
311|         split_k = opt_flags_nvidia.compute_split_k(block_k, k, estimated_actual_grid_size)
```
**EN:** Checks `constraints.get('max_allowable_mn', 0) > 0 and constraints.get('split_k') is not None`. The true branch mainly prepares intermediate values, while the else branch branches on runtime conditions.

**CN:** 检查 `constraints.get('max_allowable_mn', 0) > 0 and constraints.get('split_k') is not None`. 真分支主要准备中间值；而 else 分支根据运行时条件分支.

### Block 82 — Lines 312-315 (make_default_opt_flags_nvidia)
```python
312|     if split_k > 1:
313|         # Split-K writes full-N scratch and applies fused reductions in the
314|         # reduce kernel, not in the matmul epilogue.
315|         epilogue_reduction_n = 1
```
**EN:** Checks `split_k > 1`. The true branch mainly prepares intermediate values.

**CN:** 检查 `split_k > 1`. 真分支主要准备中间值.

### Block 83 — Lines 316-331 (make_default_opt_flags_nvidia)
```python
316|     compute_num_stages_args = (
317|         precision_config,
318|         is_persistent,
319|         block_m,
320|         block_n,
321|         block_k,
322|         torch_dtype_to_dtype(precision_config.intermediate_out_dtype) if split_k > 1 else out_dtype,
323|         lhs_dtype,
324|         rhs_dtype,
325|         x_transpose,
326|         epilogue_effective_itemsize,
327|         has_y_acc_in,
328|         mx_block_size,
329|         epilogue_reduction_n,
330|     )
331| 
```
**EN:** Assigns `compute_num_stages_args` and builds a tuple.

**CN:** 将 `compute_num_stages_args` and 构造一个元组.

### Block 84 — Lines 332-332 (make_default_opt_flags_nvidia)
```python
332|     num_warps = opt_flags_nvidia.compute_num_warps(block_m, block_n, is_persistent, precision_config, constraints)
```
**EN:** Assigns `num_warps` and calls `opt_flags_nvidia.compute_num_warps`.

**CN:** 将 `num_warps`，并调用 `opt_flags_nvidia.compute_num_warps`.

### Block 85 — Lines 333-340 (make_default_opt_flags_nvidia)
```python
333|     if (constraints.get("num_warps", None) is None
334|             and is_persistent
335|             and block_n <= 128
336|             and block_k >= 256
337|             and opt_flags_nvidia.is_blackwell_mx_lhs_dense_rhs(precision_config, lhs_dtype, rhs_dtype)):
338|         num_warps = max(num_warps, 8)
339| 
340|     # Occupancy target and maxnreg (for Hopper)
```
**EN:** Checks `constraints.get('num_warps', None) is None and is_persistent and (block_n <= 128) and (...`. The true branch mainly prepares intermediate values.

**CN:** 检查 `constraints.get('num_warps', None) is None and is_persistent and (block_n <= 128) and (...`. 真分支主要准备中间值.

### Block 86 — Lines 341-341 (make_default_opt_flags_nvidia)
```python
341|     occupancy_target = 1
```
**EN:** Assigns `occupancy_target` and stores constant `1`.

**CN:** 将 `occupancy_target` and 保存常量 `1`.

### Block 87 — Lines 342-342 (make_default_opt_flags_nvidia)
```python
342|     is_hopper_scale = isinstance(b_mx_scale_layout, HopperMXScaleLayout)
```
**EN:** Assigns `is_hopper_scale` and calls `isinstance`.

**CN:** 将 `is_hopper_scale`，并调用 `isinstance`.

### Block 88 — Lines 343-348 (make_default_opt_flags_nvidia)
```python
343|     if is_hopper_scale:
344|         occupancy_target = 16 // num_warps
345|         if precision_config.a_mx_scale is not None and precision_config.c_mx_scale is not None:
346|             # Hopper MXFP4 RHS plus MX input/output needs more than the
347|             # 128-register cap implied by the default occupancy target.
348|             occupancy_target = 1
```
**EN:** Checks `is_hopper_scale`. The true branch mainly prepares intermediate values; branches on runtime conditions.

**CN:** 检查 `is_hopper_scale`. 真分支主要准备中间值; 根据运行时条件分支.

### Block 89 — Lines 349-349 (make_default_opt_flags_nvidia)
```python
349|     threads_per_warp = 32
```
**EN:** Assigns `threads_per_warp` and stores constant `32`.

**CN:** 将 `threads_per_warp` and 保存常量 `32`.

### Block 90 — Lines 350-350 (make_default_opt_flags_nvidia)
```python
350|     reg_per_sm = 64 * 1024
```
**EN:** Assigns `reg_per_sm` and evaluates `64 * 1024`.

**CN:** 将 `reg_per_sm` and 计算 `64 * 1024`.

### Block 91 — Lines 351-351 (make_default_opt_flags_nvidia)
```python
351|     max_reg_per_thread = 256
```
**EN:** Assigns `max_reg_per_thread` and stores constant `256`.

**CN:** 将 `max_reg_per_thread` and 保存常量 `256`.

### Block 92 — Lines 352-352 (make_default_opt_flags_nvidia)
```python
352|     is_blackwell_or_newer = cuda_capability_geq(10, 0)
```
**EN:** Assigns `is_blackwell_or_newer` and calls `cuda_capability_geq`.

**CN:** 将 `is_blackwell_or_newer`，并调用 `cuda_capability_geq`.

### Block 93 — Lines 353-358 (make_default_opt_flags_nvidia)
```python
353|     if is_persistent and not is_blackwell_or_newer:
354|         maxnreg = reg_per_sm // (num_warps * threads_per_warp * occupancy_target)
355|         maxnreg = min(max_reg_per_thread, maxnreg)
356|     else:
357|         maxnreg = None
358| 
```
**EN:** Checks `is_persistent and (not is_blackwell_or_newer)`. The true branch mainly prepares intermediate values; prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `is_persistent and (not is_blackwell_or_newer)`. 真分支主要准备中间值; 准备中间值；而 else 分支准备中间值.

### Block 94 — Lines 359-362 (make_default_opt_flags_nvidia)
```python
359|     if constraints.get("epilogue_subtile", None) is not None:
360|         subtiles_to_check = [constraints["epilogue_subtile"]]
361|     else:
362|         subtiles_to_check = [1] if out_dtype == FP4 else [1, 2, 4]
```
**EN:** Checks `constraints.get('epilogue_subtile', None) is not None`. The true branch mainly prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `constraints.get('epilogue_subtile', None) is not None`. 真分支主要准备中间值；而 else 分支准备中间值.

### Block 95 — Lines 363-363 (make_default_opt_flags_nvidia)
```python
363|     num_stages = -1
```
**EN:** Assigns `num_stages` and evaluates `-1`.

**CN:** 将 `num_stages` and 计算 `-1`.

### Block 96 — Lines 364-369 (make_default_opt_flags_nvidia)
```python
364|     for ep in subtiles_to_check:
365|         ns = opt_flags_nvidia.compute_num_stages(*compute_num_stages_args, epilogue_subtile=ep,
366|                                                  occupancy_target=occupancy_target)
367|         if ns > num_stages:
368|             epilogue_subtile, num_stages = ep, ns
369| 
```
**EN:** Loops over `subtiles_to_check` with target `ep`. The loop body mainly prepares intermediate values; branches on runtime conditions.

**CN:** 遍历 `subtiles_to_check` ，目标变量为 `ep`. 循环体主要准备中间值; 根据运行时条件分支.

### Block 97 — Lines 370-371 (make_default_opt_flags_nvidia)
```python
370|     if constraints.get("num_stages", None):
371|         num_stages = constraints["num_stages"]
```
**EN:** Checks `constraints.get('num_stages', None)`. The true branch mainly prepares intermediate values.

**CN:** 检查 `constraints.get('num_stages', None)`. 真分支主要准备中间值.

### Block 98 — Lines 372-372 (make_default_opt_flags_nvidia)
```python
372|     assert num_stages >= 1
```
**EN:** Asserts `num_stages >= 1` to enforce invariants.

**CN:** 断言 `num_stages >= 1` 以确保不变量成立。

### Block 99 — Lines 373-394 (make_default_opt_flags_nvidia)
```python
373|     ret = OptFlags(
374|         block_m=block_m,
375|         block_n=block_n,
376|         block_k=block_k,
377|         num_warps=num_warps,
378|         num_stages=num_stages,
379|         group_m=group_m,
380|         xcd_swizzle=xcd_swizzle,
381|         w_cache_modifier=None,
382|         split_k=split_k,
383|         is_persistent=is_persistent,
384|         epilogue_subtile=epilogue_subtile,
385|         arch=arch,
386|         target_kernel_kwargs=dict(
387|             maxnreg=maxnreg,
388|             # For some reason, overlapping the epilogue is slower for hopper bf16 x mxfp4
389|             FLATTEN_LOOPS=not is_hopper_scale,
390|         ),
391|         idle_sms=constraints.get("idle_sms", 0),
392|         occupancy_target=occupancy_target,
393|     )
394|     # check constraints
```
**EN:** Assigns `ret` and calls `OptFlags`.

**CN:** 将 `ret`，并调用 `OptFlags`.

### Block 100 — Lines 395-395 (make_default_opt_flags_nvidia)
```python
395|     all_constraints_satisfied(ret, constraints)
```
**EN:** Calls `all_constraints_satisfied` for side effects, registration, or validation.

**CN:** 调用 `all_constraints_satisfied` ，用于副作用、注册或校验。

### Block 101 — Lines 396-401 (make_default_opt_flags_nvidia)
```python
396|     return ret
397| 
398| # --------------
399| # User Interface
400| # --------------
401| 
```
**EN:** Returns `ret`.

**CN:** 返回 `ret`.

### Block 102 — Lines 402-402 (module)
```python
402| _opt_flags_constraints: ContextVar[dict | None] = ContextVar("opt_flags_constraints", default=None)
```
**EN:** Annotated assignment stores `_opt_flags_constraints` and calls `ContextVar`.

**CN:** 带类型注解的赋值保存 `_opt_flags_constraints`，并调用 `ContextVar`.

### Block 103 — Lines 403-404 (module)
```python
403| _opt_flags: ContextVar[OptFlags | None] = ContextVar("opt_flags", default=None)
404| 
```
**EN:** Annotated assignment stores `_opt_flags` and calls `ContextVar`.

**CN:** 带类型注解的赋值保存 `_opt_flags`，并调用 `ContextVar`.

### Block 104 — Lines 405-405 (_get_opt_flags_constraints)
```python
405| def _get_opt_flags_constraints() -> dict:
```
**EN:** Defines function `_get_opt_flags_constraints()` for this module. The body mainly prepares intermediate values; returns the computed result. It uses calls such as `_opt_flags_constraints.get` to implement its workflow.

**CN:** 定义函数 `_get_opt_flags_constraints()`，供本模块使用. 主体主要准备中间值; 返回计算结果. 其中会调用 `_opt_flags_constraints.get` 来实现其工作流程.

### Block 105 — Lines 406-406 (_get_opt_flags_constraints)
```python
406|     constraints = _opt_flags_constraints.get()
```
**EN:** Assigns `constraints` and calls `_opt_flags_constraints.get`.

**CN:** 将 `constraints`，并调用 `_opt_flags_constraints.get`.

### Block 106 — Lines 407-408 (_get_opt_flags_constraints)
```python
407|     return {} if constraints is None else constraints
408| 
```
**EN:** Returns `{} if constraints is None else constraints`.

**CN:** 返回 `{} if constraints is None else constraints`.

### Block 107 — Lines 409-409 (update_opt_flags_constraints)
```python
409| def update_opt_flags_constraints(constraints: dict[str, int]):
```
**EN:** Defines function `update_opt_flags_constraints(constraints)` for this module. The body mainly prepares intermediate values; invokes `updated.update`; invokes `_opt_flags_constraints.set`. It uses calls such as `_get_opt_flags_constraints().copy`, `updated.update`, `_opt_flags_constraints.set`, `_get_opt_flags_constraints` to implement its workflow.

**CN:** 定义函数 `update_opt_flags_constraints(constraints)`，供本模块使用. 主体主要准备中间值; invokes `updated.update`; invokes `_opt_flags_constraints.set`. 其中会调用 `_get_opt_flags_constraints().copy`, `updated.update`, `_opt_flags_constraints.set`, `_get_opt_flags_constraints` 来实现其工作流程.

### Block 108 — Lines 410-410 (update_opt_flags_constraints)
```python
410|     updated = _get_opt_flags_constraints().copy()
```
**EN:** Assigns `updated` and calls `_get_opt_flags_constraints().copy`.

**CN:** 将 `updated`，并调用 `_get_opt_flags_constraints().copy`.

### Block 109 — Lines 411-411 (update_opt_flags_constraints)
```python
411|     updated.update(constraints)
```
**EN:** Calls `updated.update` for side effects, registration, or validation.

**CN:** 调用 `updated.update` ，用于副作用、注册或校验。

### Block 110 — Lines 412-413 (update_opt_flags_constraints)
```python
412|     _opt_flags_constraints.set(updated)
413| 
```
**EN:** Calls `_opt_flags_constraints.set` for side effects, registration, or validation.

**CN:** 调用 `_opt_flags_constraints.set` ，用于副作用、注册或校验。

### Block 111 — Lines 414-414 (reset_opt_flags_constraints)
```python
414| def reset_opt_flags_constraints():
```
**EN:** Defines function `reset_opt_flags_constraints()` for this module. The body mainly invokes `_opt_flags_constraints.set`. It uses calls such as `_opt_flags_constraints.set` to implement its workflow.

**CN:** 定义函数 `reset_opt_flags_constraints()`，供本模块使用. 主体主要invokes `_opt_flags_constraints.set`. 其中会调用 `_opt_flags_constraints.set` 来实现其工作流程.

### Block 112 — Lines 415-416 (reset_opt_flags_constraints)
```python
415|     _opt_flags_constraints.set(None)
416| 
```
**EN:** Calls `_opt_flags_constraints.set` for side effects, registration, or validation.

**CN:** 调用 `_opt_flags_constraints.set` ，用于副作用、注册或校验。

### Block 113 — Lines 417-418 (scoped_opt_flags_constraints)
```python
417| @contextmanager
418| def scoped_opt_flags_constraints(constraints):
```
**EN:** Defines function `scoped_opt_flags_constraints(constraints)` with decorators `contextmanager` for this module. The body mainly prepares intermediate values; invokes `updated.update`; prepares intermediate values. It uses calls such as `_get_opt_flags_constraints().copy`, `updated.update`, `_opt_flags_constraints.set`, `_opt_flags_constraints.reset`, `_get_opt_flags_constraints` to implement its workflow.

**CN:** 定义函数 `scoped_opt_flags_constraints(constraints)`，带有装饰器 `contextmanager`，供本模块使用. 主体主要准备中间值; invokes `updated.update`; 准备中间值. 其中会调用 `_get_opt_flags_constraints().copy`, `updated.update`, `_opt_flags_constraints.set`, `_opt_flags_constraints.reset`, `_get_opt_flags_constraints` 来实现其工作流程.

### Block 114 — Lines 419-419 (scoped_opt_flags_constraints)
```python
419|     updated = _get_opt_flags_constraints().copy()
```
**EN:** Assigns `updated` and calls `_get_opt_flags_constraints().copy`.

**CN:** 将 `updated`，并调用 `_get_opt_flags_constraints().copy`.

### Block 115 — Lines 420-420 (scoped_opt_flags_constraints)
```python
420|     updated.update(constraints)
```
**EN:** Calls `updated.update` for side effects, registration, or validation.

**CN:** 调用 `updated.update` ，用于副作用、注册或校验。

### Block 116 — Lines 421-421 (scoped_opt_flags_constraints)
```python
421|     token = _opt_flags_constraints.set(updated)
```
**EN:** Assigns `token` and calls `_opt_flags_constraints.set`.

**CN:** 将 `token`，并调用 `_opt_flags_constraints.set`.

### Block 117 — Lines 422-426 (scoped_opt_flags_constraints)
```python
422|     try:
423|         yield
424|     finally:
425|         _opt_flags_constraints.reset(token)
426| 
```
**EN:** Wraps code with exception/finally handling.

**CN:** 包装代码 以及异常/收尾处理.

### Block 118 — Lines 427-427 (reset_opt_flags)
```python
427| def reset_opt_flags():
```
**EN:** Defines function `reset_opt_flags()` for this module. The body mainly invokes `_opt_flags.set`. It uses calls such as `_opt_flags.set` to implement its workflow.

**CN:** 定义函数 `reset_opt_flags()`，供本模块使用. 主体主要invokes `_opt_flags.set`. 其中会调用 `_opt_flags.set` 来实现其工作流程.

### Block 119 — Lines 428-429 (reset_opt_flags)
```python
428|     _opt_flags.set(None)
429| 
```
**EN:** Calls `_opt_flags.set` for side effects, registration, or validation.

**CN:** 调用 `_opt_flags.set` ，用于副作用、注册或校验。

### Block 120 — Lines 430-430 (set_opt_flags)
```python
430| def set_opt_flags(opt_flags: OptFlags):
```
**EN:** Defines function `set_opt_flags(opt_flags)` for this module. The body mainly checks invariants; checks invariants; invokes `_opt_flags.set`. It uses calls such as `_opt_flags.set`, `_get_opt_flags_constraints`, `_opt_flags.get` to implement its workflow.

**CN:** 定义函数 `set_opt_flags(opt_flags)`，供本模块使用. 主体主要检查不变量; 检查不变量; invokes `_opt_flags.set`. 其中会调用 `_opt_flags.set`, `_get_opt_flags_constraints`, `_opt_flags.get` 来实现其工作流程.

### Block 121 — Lines 431-431 (set_opt_flags)
```python
431|     assert not _get_opt_flags_constraints(), "setting constraints is incompatible with manual flags override"
```
**EN:** Asserts `not _get_opt_flags_constraints()` to enforce invariants.

**CN:** 断言 `not _get_opt_flags_constraints()` 以确保不变量成立。

### Block 122 — Lines 432-432 (set_opt_flags)
```python
432|     assert _opt_flags.get() is None, "opt_flags already set; please reset to None first"
```
**EN:** Asserts `_opt_flags.get() is None` to enforce invariants.

**CN:** 断言 `_opt_flags.get() is None` 以确保不变量成立。

### Block 123 — Lines 433-434 (set_opt_flags)
```python
433|     _opt_flags.set(opt_flags)
434| 
```
**EN:** Calls `_opt_flags.set` for side effects, registration, or validation.

**CN:** 调用 `_opt_flags.set` ，用于副作用、注册或校验。

### Block 124 — Lines 435-436 (scoped_opt_flags)
```python
435| @contextmanager
436| def scoped_opt_flags(opt_flags: OptFlags):
```
**EN:** Defines function `scoped_opt_flags(opt_flags)` with decorators `contextmanager` for this module. The body mainly prepares intermediate values; handles exceptional paths. It uses calls such as `_opt_flags.set`, `_opt_flags.reset` to implement its workflow.

**CN:** 定义函数 `scoped_opt_flags(opt_flags)`，带有装饰器 `contextmanager`，供本模块使用. 主体主要准备中间值; 处理异常路径. 其中会调用 `_opt_flags.set`, `_opt_flags.reset` 来实现其工作流程.

### Block 125 — Lines 437-437 (scoped_opt_flags)
```python
437|     token = _opt_flags.set(opt_flags)
```
**EN:** Assigns `token` and calls `_opt_flags.set`.

**CN:** 将 `token`，并调用 `_opt_flags.set`.

### Block 126 — Lines 438-442 (scoped_opt_flags)
```python
438|     try:
439|         yield
440|     finally:
441|         _opt_flags.reset(token)
442| 
```
**EN:** Wraps code with exception/finally handling.

**CN:** 包装代码 以及异常/收尾处理.

### Block 127 — Lines 443-443 (InapplicableConstraint)
```python
443| class InapplicableConstraint(Exception):
```
**EN:** Defines class `InapplicableConstraint` inheriting from `Exception` to organize related behavior.

**CN:** 定义类 `InapplicableConstraint`，继承自 `Exception`，用于组织相关行为.

### Block 128 — Lines 444-445 (InapplicableConstraint)
```python
444|     pass
445| 
```
**EN:** This block contains only `pass` as a placeholder.

**CN:** This block 仅包含 `pass` 占位语句.

### Block 129 — Lines 446-466 (make_opt_flags)
```python
446| def make_opt_flags(
447|     out_dtype,
448|     lhs_dtype,
449|     rhs_dtype,
450|     precision_config,
451|     batch_size,
452|     m,
453|     n,
454|     k,
455|     ragged_metadata,
456|     can_use_persistent_tma,
457|     can_use_split_k,
458|     epilogue_effective_itemsize,
459|     x_transpose,
460|     has_y_acc_in,
461|     block_k,
462|     mx_block_size=None,
463|     x_uses_tma_when_persistent=True,
464|     rhs_layout=None,
465|     epilogue_reduction_n=1,
466| ):
```
**EN:** Defines function `make_opt_flags(out_dtype, lhs_dtype, rhs_dtype, precision_config, batch_size, m, n, k, ragged_metadata, can_use_persistent_tma, can_use_split_k, epilogue_effective_itemsize, x_transpose, has_y_acc_in, block_k, mx_block_size, x_uses_tma_when_persistent, rhs_layout, epilogue_reduction_n)` for this module. The body mainly prepares intermediate values; branches on runtime conditions; branches on runtime conditions. It uses calls such as `_get_opt_flags_constraints`, `opt_flags_constraints.get`, `_opt_flags.get`, `isinstance`, `InapplicableConstraint` to implement its workflow.

**CN:** 定义函数 `make_opt_flags(out_dtype, lhs_dtype, rhs_dtype, precision_config, batch_size, m, n, k, ragged_metadata, can_use_persistent_tma, can_use_split_k, epilogue_effective_itemsize, x_transpose, has_y_acc_in, block_k, mx_block_size, x_uses_tma_when_persistent, rhs_layout, epilogue_reduction_n)`，供本模块使用. 主体主要准备中间值; 根据运行时条件分支; 根据运行时条件分支. 其中会调用 `_get_opt_flags_constraints`, `opt_flags_constraints.get`, `_opt_flags.get`, `isinstance`, `InapplicableConstraint` 来实现其工作流程.

### Block 130 — Lines 467-467 (make_opt_flags)
```python
467|     opt_flags_constraints = _get_opt_flags_constraints()
```
**EN:** Assigns `opt_flags_constraints` and calls `_get_opt_flags_constraints`.

**CN:** 将 `opt_flags_constraints`，并调用 `_get_opt_flags_constraints`.

### Block 131 — Lines 468-469 (make_opt_flags)
```python
468|     if opt_flags_constraints.get("is_persistent", False) and not can_use_persistent_tma:
469|         raise InapplicableConstraint("cannot enforce `is_persistent=True` constraint")
```
**EN:** Checks `opt_flags_constraints.get('is_persistent', False) and (not can_use_persistent_tma)`..

**CN:** 检查 `opt_flags_constraints.get('is_persistent', False) and (not can_use_persistent_tma)`..

### Block 132 — Lines 470-471 (make_opt_flags)
```python
470|     if opt_flags_constraints.get("split_k") is not None and opt_flags_constraints.get("split_k") > 1 and not can_use_split_k:
471|         raise InapplicableConstraint("cannot enforce `split_k=True` constraint")
```
**EN:** Checks `opt_flags_constraints.get('split_k') is not None and opt_flags_constraints.get('split_k...`..

**CN:** 检查 `opt_flags_constraints.get('split_k') is not None and opt_flags_constraints.get('split_k...`..

### Block 133 — Lines 472-474 (make_opt_flags)
```python
472|     if opt_flags_constraints.get("max_allowable_mn"):
473|         if not opt_flags_constraints.get("split_k"):
474|             raise InapplicableConstraint("split_k also needs to be provided with max_allowable_mn")
```
**EN:** Checks `opt_flags_constraints.get('max_allowable_mn')`. The true branch mainly branches on runtime conditions.

**CN:** 检查 `opt_flags_constraints.get('max_allowable_mn')`. 真分支主要根据运行时条件分支.

### Block 134 — Lines 475-475 (make_opt_flags)
```python
475|     enforce_bitwise_invariance = precision_config.enforce_bitwise_invariance
```
**EN:** Assigns `enforce_bitwise_invariance` and references `precision_config.enforce_bitwise_invariance`.

**CN:** 将 `enforce_bitwise_invariance` and 引用 `precision_config.enforce_bitwise_invariance`.

### Block 135 — Lines 476-476 (make_opt_flags)
```python
476|     opt_flags = _opt_flags.get()
```
**EN:** Assigns `opt_flags` and calls `_opt_flags.get`.

**CN:** 将 `opt_flags`，并调用 `_opt_flags.get`.

### Block 136 — Lines 477-480 (make_opt_flags)
```python
477|     if opt_flags is not None:
478|         assert not opt_flags_constraints
479|         assert block_k is None
480|         return opt_flags
```
**EN:** Checks `opt_flags is not None`. The true branch mainly checks invariants; checks invariants.

**CN:** 检查 `opt_flags is not None`. 真分支主要检查不变量; 检查不变量.

### Block 137 — Lines 481-485 (make_opt_flags)
```python
481|     if isinstance(rhs_layout, BlackwellMX4ValueShuffledLayout):
482|         opt_flags_constraints = opt_flags_constraints.copy()
483|         opt_flags_constraints.setdefault("block_k", rhs_layout.block_k)
484|         opt_flags_constraints.setdefault("block_n", rhs_layout.block_n)
485|         opt_flags_constraints.setdefault("disable_mx4_block_swap", True)
```
**EN:** Checks `isinstance(rhs_layout, BlackwellMX4ValueShuffledLayout)`. The true branch mainly prepares intermediate values; invokes `opt_flags_constraints.setdefault`.

**CN:** 检查 `isinstance(rhs_layout, BlackwellMX4ValueShuffledLayout)`. 真分支主要准备中间值; invokes `opt_flags_constraints.setdefault`.

### Block 138 — Lines 486-488 (make_opt_flags)
```python
486|     if block_k is not None:
487|         opt_flags_constraints = opt_flags_constraints.copy()
488|         opt_flags_constraints.update(block_k=block_k, split_k=1)
```
**EN:** Checks `block_k is not None`. The true branch mainly prepares intermediate values; invokes `opt_flags_constraints.update`.

**CN:** 检查 `block_k is not None`. 真分支主要准备中间值; invokes `opt_flags_constraints.update`.

### Block 139 — Lines 489-492 (make_opt_flags)
```python
489|     args = [out_dtype, lhs_dtype, rhs_dtype, precision_config, batch_size, m, n, k,
490|             ragged_metadata, can_use_persistent_tma, can_use_split_k,
491|             enforce_bitwise_invariance, epilogue_effective_itemsize, x_transpose, has_y_acc_in,
492|             opt_flags_constraints]
```
**EN:** Assigns `args` and builds a list.

**CN:** 将 `args` and 构造一个列表.

### Block 140 — Lines 493-493 (make_opt_flags)
```python
493|     backend = triton.runtime.driver.active.get_current_target().backend
```
**EN:** Assigns `backend` and references `triton.runtime.driver.active.get_current_target().backend`.

**CN:** 将 `backend` and 引用 `triton.runtime.driver.active.get_current_target().backend`.

### Block 141 — Lines 494-495 (make_opt_flags)
```python
494|     if backend == "hip":
495|         return make_default_opt_flags_amd(*args)
```
**EN:** Checks `backend == 'hip'`. The true branch mainly returns the computed result.

**CN:** 检查 `backend == 'hip'`. 真分支主要返回计算结果.

### Block 142 — Lines 496-502 (make_opt_flags)
```python
496|     if backend == "cuda":
497|         return make_default_opt_flags_nvidia(
498|             *args,
499|             x_uses_tma_when_persistent=x_uses_tma_when_persistent,
500|             mx_block_size=mx_block_size,
501|             epilogue_reduction_n=epilogue_reduction_n,
502|         )
```
**EN:** Checks `backend == 'cuda'`. The true branch mainly returns the computed result.

**CN:** 检查 `backend == 'cuda'`. 真分支主要返回计算结果.

### Block 143 — Lines 503-503 (make_opt_flags)
```python
503|     assert False
```
**EN:** Asserts `False` to enforce invariants.

**CN:** 断言 `False` 以确保不变量成立。

## Key Concepts / 关键概念
- **EN:** Primary symbols: `OptFlags`, `max_allowable_mn`, `all_constraints_satisfied`, `make_default_opt_flags_amd`, `make_default_opt_flags_nvidia`, `_get_opt_flags_constraints`, `update_opt_flags_constraints`, `reset_opt_flags_constraints`.
  **CN:** 主要符号：`OptFlags`, `max_allowable_mn`, `all_constraints_satisfied`, `make_default_opt_flags_amd`, `make_default_opt_flags_nvidia`, `_get_opt_flags_constraints`, `update_opt_flags_constraints`, `reset_opt_flags_constraints`。
- **EN:** PyTorch tensors are the main data container and host-side orchestration layer.
  **CN:** PyTorch 张量是主要的数据容器和主机端编排层。
- **EN:** Triton is used to JIT-compile or launch GPU kernels and to express tile-level execution details.
  **CN:** Triton 用于 JIT 编译或启动 GPU 内核，并表达分块级执行细节。
- **EN:** Mixed-precision MXFP/NVFP formats and scale tensors are part of the numeric pipeline.
  **CN:** 混合精度 MXFP/NVFP 格式及其缩放张量是数值流程的一部分。
- **EN:** Custom layout objects describe how logical tensors are packed or swizzled in memory.
  **CN:** 自定义 layout 对象描述逻辑张量如何在内存中打包或重排。
- **EN:** Reduction logic combines partial values across dimensions or shards.
  **CN:** 归约逻辑会沿着维度或分片合并部分结果。
- **EN:** Ragged tensor metadata is used when rows or slices have non-uniform sizes.
  **CN:** 当行或切片大小不一致时，会使用 ragged tensor 元数据。

## Dependencies / 依赖关系
- **EN:** External modules: `contextlib (contextmanager)`, `contextvars (ContextVar)`, `dataclasses (dataclass)`, `triton`, `torch`.
  **CN:** 外部模块：`contextlib (contextmanager)`, `contextvars (ContextVar)`, `dataclasses (dataclass)`, `triton`, `torch`。
- **EN:** Internal modules: `triton_kernels (target_info)`, `triton_kernels.target_info (get_cdna_version, get_rdna_version, cuda_capability_geq)`, `triton_kernels.tensor (FP4, FP32, Tensor, torch_dtype_to_dtype)`, `triton_kernels.tensor_details.layout_details.hopper_scale (HopperMXScaleLayout)`, `triton_kernels.tensor_details.layout_details.strided (StridedLayout)`, `triton_kernels.tensor_details.layout_details.base (Layout)`, `triton_kernels.tensor_details.layout_details.blackwell_value_shuffled (BlackwellMX4ValueShuffledLayout)`, `.opt_flags_details (opt_flags_amd, opt_flags_nvidia)`.
  **CN:** 内部模块：`triton_kernels (target_info)`, `triton_kernels.target_info (get_cdna_version, get_rdna_version, cuda_capability_geq)`, `triton_kernels.tensor (FP4, FP32, Tensor, torch_dtype_to_dtype)`, `triton_kernels.tensor_details.layout_details.hopper_scale (HopperMXScaleLayout)`, `triton_kernels.tensor_details.layout_details.strided (StridedLayout)`, `triton_kernels.tensor_details.layout_details.base (Layout)`, `triton_kernels.tensor_details.layout_details.blackwell_value_shuffled (BlackwellMX4ValueShuffledLayout)`, `.opt_flags_details (opt_flags_amd, opt_flags_nvidia)`。
