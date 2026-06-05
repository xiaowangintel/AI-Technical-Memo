# opt_flags_nvidia.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/triton_kernels/matmul_details/opt_flags_details/opt_flags_nvidia.py`
- **Purpose / 用途:** Implementation module for opt flags nvidia; it defines the runtime logic, helper types, and GPU-facing utilities used by the Triton kernels package via symbols is_x_scale_swizzled, is_blackwell_mx_lhs_dense_rhs, compute_swap_xw, compute_grid_size, compute_block_n. / 用于 opt flags nvidia 的实现模块；它定义了 Triton kernels 包使用的运行时逻辑、辅助类型以及面向 GPU 的工具；核心符号包括 is_x_scale_swizzled、is_blackwell_mx_lhs_dense_rhs、compute_swap_xw、compute_grid_size、compute_block_n。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-9 (module)
```python
1| import torch
2| import triton
3| from triton_kernels import target_info
4| from triton_kernels.numerics_details.mxfp_details._downcast_to_mxfp import MXFP_BLOCK_SIZE
5| from triton_kernels.tensor import FP4, FP16, FP32, BF16, Tensor
6| from triton_kernels.tensor_details.layout import HopperMXScaleLayout
7| from triton_kernels.tensor_details.layout_details.blackwell_scale import BlackwellActMXScaleLayout, BlackwellMXScaleLayout
8| 
9| 
```
**EN:** This block imports `torch`, `triton`, `triton_kernels (target_info)`, `triton_kernels.numerics_details.mxfp_details._downcast_to_mxfp (MXFP_BLOCK_SIZE)`, `triton_kernels.tensor (FP4, FP16, FP32, BF16)`, `triton_kernels.tensor_details.layout (HopperMXScaleLayout)`, `triton_kernels.tensor_details.layout_details.blackwell_scale (BlackwellActMXScaleLayout, BlackwellMXScaleLayout)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `torch`, `triton`, `triton_kernels (target_info)`, `triton_kernels.numerics_details.mxfp_details._downcast_to_mxfp (MXFP_BLOCK_SIZE)`, `triton_kernels.tensor (FP4, FP16, FP32, BF16)`, `triton_kernels.tensor_details.layout (HopperMXScaleLayout)`, `triton_kernels.tensor_details.layout_details.blackwell_scale (BlackwellActMXScaleLayout, BlackwellMXScaleLayout)` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 10-10 (is_x_scale_swizzled)
```python
10| def is_x_scale_swizzled(precision_config):
```
**EN:** Defines function `is_x_scale_swizzled(precision_config)` for this module. The body mainly returns the computed result. It uses calls such as `isinstance` to implement its workflow.

**CN:** 定义函数 `is_x_scale_swizzled(precision_config)`，供本模块使用. 主体主要返回计算结果. 其中会调用 `isinstance` 来实现其工作流程.

### Block 3 — Lines 11-15 (is_x_scale_swizzled)
```python
11|     return (precision_config is not None and precision_config.a_mx_scale is not None
12|             and isinstance(precision_config.a_mx_scale, Tensor)
13|             and isinstance(precision_config.a_mx_scale.storage.layout, BlackwellActMXScaleLayout))
14| 
15| 
```
**EN:** Returns `precision_config is not None and precision_config.a_mx_scale is not None and ...`.

**CN:** 返回 `precision_config is not None and precision_config.a_mx_scale is not None and ...`.

### Block 4 — Lines 16-16 (is_blackwell_mx_lhs_dense_rhs)
```python
16| def is_blackwell_mx_lhs_dense_rhs(precision_config, lhs_dtype, rhs_dtype):
```
**EN:** Defines function `is_blackwell_mx_lhs_dense_rhs(precision_config, lhs_dtype, rhs_dtype)` for this module. The body mainly returns the computed result. It uses calls such as `target_info.cuda_capability_geq`, `int` to implement its workflow.

**CN:** 定义函数 `is_blackwell_mx_lhs_dense_rhs(precision_config, lhs_dtype, rhs_dtype)`，供本模块使用. 主体主要返回计算结果. 其中会调用 `target_info.cuda_capability_geq`, `int` 来实现其工作流程.

### Block 5 — Lines 17-22 (is_blackwell_mx_lhs_dense_rhs)
```python
17|     return (target_info.cuda_capability_geq(10, 0) and precision_config is not None
18|             and precision_config.a_mx_scale is not None and precision_config.a_microblock_size == int(MXFP_BLOCK_SIZE)
19|             and precision_config.b_mx_scale is None and precision_config.c_mx_scale is None and lhs_dtype.bitwidth <= 8
20|             and rhs_dtype in [FP16, BF16])
21| 
22| 
```
**EN:** Returns `target_info.cuda_capability_geq(10, 0) and precision_config is not None and (...`.

**CN:** 返回 `target_info.cuda_capability_geq(10, 0) and precision_config is not None and (...`.

### Block 6 — Lines 23-23 (compute_swap_xw)
```python
23| def compute_swap_xw(precision_config, block_m, is_persistent, lhs_dtype, rhs_dtype):
```
**EN:** Defines function `compute_swap_xw(precision_config, block_m, is_persistent, lhs_dtype, rhs_dtype)` for this module. The body mainly branches on runtime conditions; returns the computed result. It uses calls such as `target_info.cuda_capability_geq`, `isinstance` to implement its workflow.

**CN:** 定义函数 `compute_swap_xw(precision_config, block_m, is_persistent, lhs_dtype, rhs_dtype)`，供本模块使用. 主体主要根据运行时条件分支; 返回计算结果. 其中会调用 `target_info.cuda_capability_geq`, `isinstance` 来实现其工作流程.

### Block 7 — Lines 24-35 (compute_swap_xw)
```python
24|     if target_info.cuda_capability_geq(10, 0):
25|         if lhs_dtype == FP4 and rhs_dtype == FP4:
26|             return block_m <= 128 and is_persistent
27|         if precision_config.b_mx_scale is not None:
28|             return block_m <= 64 and is_persistent
29|         else:
30|             return block_m < 64 and is_persistent
31|     elif target_info.cuda_capability_geq(9, 0):
32|         layout = None if not isinstance(precision_config.b_mx_scale,
33|                                         Tensor) else precision_config.b_mx_scale.storage.layout
34|         return isinstance(layout, HopperMXScaleLayout)
35| 
```
**EN:** Checks `target_info.cuda_capability_geq(10, 0)`. The true branch mainly branches on runtime conditions; branches on runtime conditions, while the else branch branches on runtime conditions.

**CN:** 检查 `target_info.cuda_capability_geq(10, 0)`. 真分支主要根据运行时条件分支; 根据运行时条件分支；而 else 分支根据运行时条件分支.

### Block 8 — Lines 36-38 (compute_swap_xw)
```python
36|     return False
37| 
38| 
```
**EN:** Returns `False`.

**CN:** 返回 `False`.

### Block 9 — Lines 39-39 (compute_grid_size)
```python
39| def compute_grid_size(routing_data, batch_size, m, n, block_m, block_n):
```
**EN:** Defines function `compute_grid_size(routing_data, batch_size, m, n, block_m, block_n)` for this module. The body mainly branches on runtime conditions; prepares intermediate values; returns the computed result. It uses calls such as `routing_data.n_blocks`, `triton.cdiv` to implement its workflow.

**CN:** 定义函数 `compute_grid_size(routing_data, batch_size, m, n, block_m, block_n)`，供本模块使用. 主体主要根据运行时条件分支; 准备中间值; 返回计算结果. 其中会调用 `routing_data.n_blocks`, `triton.cdiv` 来实现其工作流程.

### Block 10 — Lines 40-43 (compute_grid_size)
```python
40|     if routing_data is not None and batch_size == 1:
41|         grid_m = routing_data.n_blocks(routing_data.n_slices, m, block_m)
42|     else:
43|         grid_m = triton.cdiv(m, block_m)
```
**EN:** Checks `routing_data is not None and batch_size == 1`. The true branch mainly prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `routing_data is not None and batch_size == 1`. 真分支主要准备中间值；而 else 分支准备中间值.

### Block 11 — Lines 44-44 (compute_grid_size)
```python
44|     grid_n = (n + block_n - 1) // block_n
```
**EN:** Assigns `grid_n` and evaluates `(n + block_n - 1) // block_n`.

**CN:** 将 `grid_n` and 计算 `(n + block_n - 1) // block_n`.

### Block 12 — Lines 45-47 (compute_grid_size)
```python
45|     return batch_size * grid_m * grid_n
46| 
47| 
```
**EN:** Returns `batch_size * grid_m * grid_n`.

**CN:** 返回 `batch_size * grid_m * grid_n`.

### Block 13 — Lines 48-49 (compute_block_n)
```python
48| def compute_block_n(n: int, arch, precision_config):
49|     # block_n:
```
**EN:** Defines function `compute_block_n(n, arch, precision_config)` for this module. The body mainly prepares intermediate values; branches on runtime conditions; branches on runtime conditions. It uses calls such as `isinstance`, `min`, `max`, `triton.next_power_of_2` to implement its workflow.

**CN:** 定义函数 `compute_block_n(n, arch, precision_config)`，供本模块使用. 主体主要准备中间值; 根据运行时条件分支; 根据运行时条件分支. 其中会调用 `isinstance`, `min`, `max`, `triton.next_power_of_2` 来实现其工作流程.

### Block 14 — Lines 50-50 (compute_block_n)
```python
50|     layout = None if not isinstance(precision_config.b_mx_scale, Tensor) else precision_config.b_mx_scale.storage.layout
```
**EN:** Assigns `layout` and uses conditional expression `None if not isinstance(precision_config.b_mx_scale, Tenso...`.

**CN:** 将 `layout` and 使用条件表达式 `None if not isinstance(precision_config.b_mx_scale, Tenso...`.

### Block 15 — Lines 51-54 (compute_block_n)
```python
51|     if isinstance(layout, HopperMXScaleLayout):
52|         # https://github.com/triton-lang/triton/blob/814b862166c756d9f33238844f4ac047e0243388/python/triton_kernels/triton_kernels/matmul_details/_matmul.py#L265
53|         block_n = 2 * layout.num_warps * 2 * 8
54|         return block_n, block_n
```
**EN:** Checks `isinstance(layout, HopperMXScaleLayout)`. The true branch mainly prepares intermediate values; returns the computed result.

**CN:** 检查 `isinstance(layout, HopperMXScaleLayout)`. 真分支主要准备中间值; 返回计算结果.

### Block 16 — Lines 55-59 (compute_block_n)
```python
55|     if precision_config.max_num_imprecise_acc is None and n > 128:
56|         block_n, block_n_tma = 256, 256
57|     else:
58|         target = min(128, triton.next_power_of_2(n))
59|         block_n, block_n_tma = max(8, target), max(16, target)
```
**EN:** Checks `precision_config.max_num_imprecise_acc is None and n > 128`. The true branch mainly prepares intermediate values, while the else branch prepares intermediate values; prepares intermediate values.

**CN:** 检查 `precision_config.max_num_imprecise_acc is None and n > 128`. 真分支主要准备中间值；而 else 分支准备中间值; 准备中间值.

### Block 17 — Lines 60-63 (compute_block_n)
```python
60|     if isinstance(layout, BlackwellMXScaleLayout):
61|         # Blackwell scale swizzle requires BLOCK_N to be a multiple of 128.
62|         block_n = max(128, block_n)
63|         block_n_tma = max(128, block_n_tma)
```
**EN:** Checks `isinstance(layout, BlackwellMXScaleLayout)`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `isinstance(layout, BlackwellMXScaleLayout)`. 真分支主要准备中间值; 准备中间值.

### Block 18 — Lines 64-66 (compute_block_n)
```python
64|     return block_n, block_n_tma
65| 
66| 
```
**EN:** Returns `(block_n, block_n_tma)`.

**CN:** 返回 `(block_n, block_n_tma)`.

### Block 19 — Lines 67-67 (compute_block_k)
```python
67| def compute_block_k(m: int, k: int | None, is_persistent: bool, lhs_dtype, rhs_dtype, precision_config, has_y_acc_in):
```
**EN:** Defines function `compute_block_k(m, k, is_persistent, lhs_dtype, rhs_dtype, precision_config, has_y_acc_in)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `int`, `target_info.cuda_capability_geq`, `is_blackwell_mx_lhs_dense_rhs`, `max`, `min` to implement its workflow.

**CN:** 定义函数 `compute_block_k(m, k, is_persistent, lhs_dtype, rhs_dtype, precision_config, has_y_acc_in)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `int`, `target_info.cuda_capability_geq`, `is_blackwell_mx_lhs_dense_rhs`, `max`, `min` 来实现其工作流程.

### Block 20 — Lines 68-68 (compute_block_k)
```python
68|     lhs_width = lhs_dtype.bitwidth
```
**EN:** Assigns `lhs_width` and references `lhs_dtype.bitwidth`.

**CN:** 将 `lhs_width` and 引用 `lhs_dtype.bitwidth`.

### Block 21 — Lines 69-70 (compute_block_k)
```python
69|     rhs_width = rhs_dtype.bitwidth
70|     # block_k needs to match the cacheline size (1024 bits)
```
**EN:** Assigns `rhs_width` and references `rhs_dtype.bitwidth`.

**CN:** 将 `rhs_width` and 引用 `rhs_dtype.bitwidth`.

### Block 22 — Lines 71-71 (compute_block_k)
```python
71|     block_k = int(1024 // min(lhs_width, rhs_width))
```
**EN:** Assigns `block_k` and calls `int`.

**CN:** 将 `block_k`，并调用 `int`.

### Block 23 — Lines 72-72 (compute_block_k)
```python
72|     has_native_mxfp = target_info.cuda_capability_geq(10, 0)
```
**EN:** Assigns `has_native_mxfp` and calls `target_info.cuda_capability_geq`.

**CN:** 将 `has_native_mxfp`，并调用 `target_info.cuda_capability_geq`.

### Block 24 — Lines 73-80 (compute_block_k)
```python
73|     if rhs_width == 4 and not has_native_mxfp:
74|         block_k = 128
75|     elif is_persistent and is_x_scale_swizzled(precision_config):
76|         # x scale has been swizzled to BlackwellActMXScaleLayout, enforce block_k to be multiple of 128
77|         block_k = max(block_k, 128)
78|     elif k is not None:  # cover small k case
79|         min_block_k = 32 if is_persistent or lhs_width != 16 or rhs_width != 16 else 16
80|         block_k = max(min_block_k, min(triton.next_power_of_2(k), block_k))
```
**EN:** Checks `rhs_width == 4 and (not has_native_mxfp)`. The true branch mainly prepares intermediate values, while the else branch branches on runtime conditions.

**CN:** 检查 `rhs_width == 4 and (not has_native_mxfp)`. 真分支主要准备中间值；而 else 分支根据运行时条件分支.

### Block 25 — Lines 81-83 (compute_block_k)
```python
81|     if (is_persistent and k is not None and k >= 256
82|             and is_blackwell_mx_lhs_dense_rhs(precision_config, lhs_dtype, rhs_dtype)):
83|         block_k = max(block_k, 256)
```
**EN:** Checks `is_persistent and k is not None and (k >= 256) and is_blackwell_mx_lhs_dense_rhs(precis...`. The true branch mainly prepares intermediate values.

**CN:** 检查 `is_persistent and k is not None and (k >= 256) and is_blackwell_mx_lhs_dense_rhs(precis...`. 真分支主要准备中间值.

### Block 26 — Lines 84-92 (compute_block_k)
```python
84|     if precision_config is not None and precision_config.b_mx_scale is not None:
85|         if has_native_mxfp and is_persistent:
86|             # If both inputs are fp4, allow larger block_k.
87|             max_block_k = 256 if lhs_dtype == FP4 and rhs_dtype == FP4 else 128
88|             block_k = min(block_k, max_block_k)
89|         if (isinstance(precision_config.b_mx_scale, Tensor)
90|                 and isinstance(precision_config.b_mx_scale.storage.layout, BlackwellMXScaleLayout)):
91|             # Blackwell scale swizzling loads four K-scale columns at a time.
92|             block_k = max(block_k, 4 * MXFP_BLOCK_SIZE.value)
```
**EN:** Checks `precision_config is not None and precision_config.b_mx_scale is not None`. The true branch mainly branches on runtime conditions; branches on runtime conditions.

**CN:** 检查 `precision_config is not None and precision_config.b_mx_scale is not None`. 真分支主要根据运行时条件分支; 根据运行时条件分支.

### Block 27 — Lines 93-94 (compute_block_k)
```python
93|     if has_y_acc_in and lhs_width == rhs_width == 16 and not target_info.cuda_capability_geq(10, 0):
94|         block_k = min(block_k, 32)
```
**EN:** Checks `has_y_acc_in and lhs_width == rhs_width == 16 and (not target_info.cuda_capability_geq(...`. The true branch mainly prepares intermediate values.

**CN:** 检查 `has_y_acc_in and lhs_width == rhs_width == 16 and (not target_info.cuda_capability_geq(...`. 真分支主要准备中间值.

### Block 28 — Lines 95-97 (compute_block_k)
```python
95|     return block_k
96| 
97| 
```
**EN:** Returns `block_k`.

**CN:** 返回 `block_k`.

### Block 29 — Lines 98-98 (compute_split_k)
```python
98| def compute_split_k(block_k: int, k: int | None, grid_size: int) -> int:
```
**EN:** Defines function `compute_split_k(block_k, k, grid_size)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `torch.cuda.get_device_properties`, `max`, `triton.cdiv`, `min` to implement its workflow.

**CN:** 定义函数 `compute_split_k(block_k, k, grid_size)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `torch.cuda.get_device_properties`, `max`, `triton.cdiv`, `min` 来实现其工作流程.

### Block 30 — Lines 99-99 (compute_split_k)
```python
99|     device_props = torch.cuda.get_device_properties(0)
```
**EN:** Assigns `device_props` and calls `torch.cuda.get_device_properties`.

**CN:** 将 `device_props`，并调用 `torch.cuda.get_device_properties`.

### Block 31 — Lines 100-100 (compute_split_k)
```python
100|     n_sms = device_props.multi_processor_count
```
**EN:** Assigns `n_sms` and references `device_props.multi_processor_count`.

**CN:** 将 `n_sms` and 引用 `device_props.multi_processor_count`.

### Block 32 — Lines 101-101 (compute_split_k)
```python
101|     split_k = n_sms // grid_size
```
**EN:** Assigns `split_k` and evaluates `n_sms // grid_size`.

**CN:** 将 `split_k` and 计算 `n_sms // grid_size`.

### Block 33 — Lines 102-105 (compute_split_k)
```python
102|     if k is not None:
103|         # avoid split_k for small k
104|         num_block_k = triton.cdiv(k, block_k)
105|         split_k = min(split_k, num_block_k // 4)
```
**EN:** Checks `k is not None`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `k is not None`. 真分支主要准备中间值; 准备中间值.

### Block 34 — Lines 106-106 (compute_split_k)
```python
106|     split_k = max(split_k, 1)
```
**EN:** Assigns `split_k` and calls `max`.

**CN:** 将 `split_k`，并调用 `max`.

### Block 35 — Lines 107-109 (compute_split_k)
```python
107|     return split_k
108| 
109| 
```
**EN:** Returns `split_k`.

**CN:** 返回 `split_k`.

### Block 36 — Lines 110-110 (compute_num_warps)
```python
110| def compute_num_warps(block_m, block_n, is_persistent: bool, precision_config, constraints):
```
**EN:** Defines function `compute_num_warps(block_m, block_n, is_persistent, precision_config, constraints)` for this module. The body mainly prepares intermediate values; branches on runtime conditions; prepares intermediate values. It uses calls such as `isinstance`, `constraints.get`, `max` to implement its workflow.

**CN:** 定义函数 `compute_num_warps(block_m, block_n, is_persistent, precision_config, constraints)`，供本模块使用. 主体主要准备中间值; 根据运行时条件分支; 准备中间值. 其中会调用 `isinstance`, `constraints.get`, `max` 来实现其工作流程.

### Block 37 — Lines 111-111 (compute_num_warps)
```python
111|     layout = None if not isinstance(precision_config.b_mx_scale, Tensor) else precision_config.b_mx_scale.storage.layout
```
**EN:** Assigns `layout` and uses conditional expression `None if not isinstance(precision_config.b_mx_scale, Tenso...`.

**CN:** 将 `layout` and 使用条件表达式 `None if not isinstance(precision_config.b_mx_scale, Tenso...`.

### Block 38 — Lines 112-113 (compute_num_warps)
```python
112|     if isinstance(layout, HopperMXScaleLayout):
113|         return layout.num_warps
```
**EN:** Checks `isinstance(layout, HopperMXScaleLayout)`. The true branch mainly returns the computed result.

**CN:** 检查 `isinstance(layout, HopperMXScaleLayout)`. 真分支主要返回计算结果.

### Block 39 — Lines 114-114 (compute_num_warps)
```python
114|     num_warps = constraints.get("num_warps", None)
```
**EN:** Assigns `num_warps` and calls `constraints.get`.

**CN:** 将 `num_warps`，并调用 `constraints.get`.

### Block 40 — Lines 115-116 (compute_num_warps)
```python
115|     if num_warps is not None:
116|         return num_warps
```
**EN:** Checks `num_warps is not None`. The true branch mainly returns the computed result.

**CN:** 检查 `num_warps is not None`. 真分支主要返回计算结果.

### Block 41 — Lines 117-119 (compute_num_warps)
```python
117|     return max(block_m * block_n // 4096, 4 if is_persistent else 1)
118| 
119| 
```
**EN:** Returns `max(block_m * block_n // 4096, 4 if is_persistent else 1)`.

**CN:** 返回 `max(block_m * block_n // 4096, 4 if is_persistent else 1)`.

### Block 42 — Lines 120-137 (compute_num_stages)
```python
120| def compute_num_stages(
121|     precision_config,
122|     is_persistent,
123|     block_m,
124|     block_n,
125|     block_k,
126|     out_dtype,
127|     lhs_dtype,
128|     rhs_dtype,
129|     x_transpose,
130|     epilogue_effective_itemsize,
131|     has_y_acc_in,
132|     mx_block_size=None,
133|     epilogue_reduction_n=1,
134|     *,
135|     epilogue_subtile,
136|     occupancy_target,
137| ):
```
**EN:** Defines function `compute_num_stages(precision_config, is_persistent, block_m, block_n, block_k, out_dtype, lhs_dtype, rhs_dtype, x_transpose, epilogue_effective_itemsize, has_y_acc_in, mx_block_size, epilogue_reduction_n, epilogue_subtile, occupancy_target)` for this module. The body mainly branches on runtime conditions; prepares intermediate values; prepares intermediate values. It uses calls such as `target_info.cuda_capability_geq`, `torch.cuda.get_device_properties`, `max`, `min`, `int` to implement its workflow.

**CN:** 定义函数 `compute_num_stages(precision_config, is_persistent, block_m, block_n, block_k, out_dtype, lhs_dtype, rhs_dtype, x_transpose, epilogue_effective_itemsize, has_y_acc_in, mx_block_size, epilogue_reduction_n, epilogue_subtile, occupancy_target)`，供本模块使用. 主体主要根据运行时条件分支; 准备中间值; 准备中间值. 其中会调用 `target_info.cuda_capability_geq`, `torch.cuda.get_device_properties`, `max`, `min`, `int` 来实现其工作流程.

### Block 43 — Lines 138-139 (compute_num_stages)
```python
138|     if precision_config.max_num_imprecise_acc is not None:
139|         return 3
```
**EN:** Checks `precision_config.max_num_imprecise_acc is not None`. The true branch mainly returns the computed result.

**CN:** 检查 `precision_config.max_num_imprecise_acc is not None`. 真分支主要返回计算结果.

### Block 44 — Lines 140-140 (compute_num_stages)
```python
140|     act_size = lhs_dtype.bitwidth / 8
```
**EN:** Assigns `act_size` and evaluates `lhs_dtype.bitwidth / 8`.

**CN:** 将 `act_size` and 计算 `lhs_dtype.bitwidth / 8`.

### Block 45 — Lines 141-141 (compute_num_stages)
```python
141|     weight_size = rhs_dtype.bitwidth / 8
```
**EN:** Assigns `weight_size` and evaluates `rhs_dtype.bitwidth / 8`.

**CN:** 将 `weight_size` and 计算 `rhs_dtype.bitwidth / 8`.

### Block 46 — Lines 142-142 (compute_num_stages)
```python
142|     has_native_mxfp = target_info.cuda_capability_geq(10, 0)
```
**EN:** Assigns `has_native_mxfp` and calls `target_info.cuda_capability_geq`.

**CN:** 将 `has_native_mxfp`，并调用 `target_info.cuda_capability_geq`.

### Block 47 — Lines 143-150 (compute_num_stages)
```python
143|     if has_native_mxfp and precision_config.b_mx_scale is not None and lhs_dtype in [FP16, BF16]:
144|         # For fp16/bf16 x mxfp, we upcast weight on the fly, so size
145|         # smem_capacity accordingly.
146|         # w/o this, gets the following error:
147|         # "triton.runtime.errors.OutOfResources: out of resource: shared memory, Required: 263356, Hardware limit: 232448. Reducing block sizes or `num_stages` may help"
148|         # for x.shape = [2048, >=4096] bf16 x [32, >=4096, >=4096] float8_e4m3fn
149|         # block_m=64, block_n=256, block_k=128, split_k=1, is_persistent=True -> leading to num_stages=4
150|         weight_size = 2
```
**EN:** Checks `has_native_mxfp and precision_config.b_mx_scale is not None and (lhs_dtype in [FP16, BF...`. The true branch mainly prepares intermediate values.

**CN:** 检查 `has_native_mxfp and precision_config.b_mx_scale is not None and (lhs_dtype in [FP16, BF...`. 真分支主要准备中间值.

### Block 48 — Lines 151-155 (compute_num_stages)
```python
151|     if has_native_mxfp and precision_config.a_mx_scale is not None and rhs_dtype in [FP16, BF16]:
152|         # For mxfp x fp16/bf16, we upcast activations on the fly, so size
153|         # smem_capacity accordingly.
154|         act_size = 2
155| 
```
**EN:** Checks `has_native_mxfp and precision_config.a_mx_scale is not None and (rhs_dtype in [FP16, BF...`. The true branch mainly prepares intermediate values.

**CN:** 检查 `has_native_mxfp and precision_config.a_mx_scale is not None and (rhs_dtype in [FP16, BF...`. 真分支主要准备中间值.

### Block 49 — Lines 156-156 (compute_num_stages)
```python
156|     stage_size = block_m * block_k * act_size + block_k * block_n * weight_size
```
**EN:** Assigns `stage_size` and evaluates `block_m * block_k * act_size + block_k * block_n * weight_size`.

**CN:** 将 `stage_size` and 计算 `block_m * block_k * act_size + block_k * block_n * weight_size`.

### Block 50 — Lines 157-157 (compute_num_stages)
```python
157|     device_props = torch.cuda.get_device_properties(0)
```
**EN:** Assigns `device_props` and calls `torch.cuda.get_device_properties`.

**CN:** 将 `device_props`，并调用 `torch.cuda.get_device_properties`.

### Block 51 — Lines 158-158 (compute_num_stages)
```python
158|     smem_capacity = device_props.shared_memory_per_block_optin
```
**EN:** Assigns `smem_capacity` and references `device_props.shared_memory_per_block_optin`.

**CN:** 将 `smem_capacity` and 引用 `device_props.shared_memory_per_block_optin`.

### Block 52 — Lines 159-159 (compute_num_stages)
```python
159|     smem_capacity //= occupancy_target
```
**EN:** Updates `smem_capacity` with operator `FloorDiv` using `occupancy_target`.

**CN:** 更新 `smem_capacity`，使用运算符 `FloorDiv`，并使用 `occupancy_target`.

### Block 53 — Lines 160-167 (compute_num_stages)
```python
160|     if has_native_mxfp:
161|         # 4-bit e2m1 operands are padded 2x
162|         # https://docs.nvidia.com/cuda/parallel-thread-execution/#packing-format-used-for-matrix-a-and-b-by-kind-mxf8f6f4-in-shared-memory
163|         if precision_config.a_mx_scale is not None and lhs_dtype == FP4 and rhs_dtype != FP4:
164|             stage_size += block_k * block_n * act_size
165|         if precision_config.b_mx_scale is not None and rhs_dtype == FP4 and lhs_dtype != FP4:
166|             stage_size += block_k * block_n * weight_size
167| 
```
**EN:** Checks `has_native_mxfp`. The true branch mainly branches on runtime conditions; branches on runtime conditions.

**CN:** 检查 `has_native_mxfp`. 真分支主要根据运行时条件分支; 根据运行时条件分支.

### Block 54 — Lines 168-171 (compute_num_stages)
```python
168|     if precision_config.a_mx_scale is not None:
169|         scale_block_size = mx_block_size or int(MXFP_BLOCK_SIZE)
170|         stage_size += block_m * (block_k // scale_block_size)
171| 
```
**EN:** Checks `precision_config.a_mx_scale is not None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `precision_config.a_mx_scale is not None`. 真分支主要准备中间值.

### Block 55 — Lines 172-176 (compute_num_stages)
```python
172|     if precision_config.b_mx_scale is not None:
173|         # mx scales
174|         scale_block_size = mx_block_size or int(MXFP_BLOCK_SIZE)
175|         stage_size += block_n * (block_k // scale_block_size)
176| 
```
**EN:** Checks `precision_config.b_mx_scale is not None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `precision_config.b_mx_scale is not None`. 真分支主要准备中间值.

### Block 56 — Lines 177-204 (compute_num_stages)
```python
177|     if is_persistent:
178|         # Per-stage wait barrier
179|         stage_size += 8
180|         out_itemsize = (out_dtype.bitwidth / 8) * (1.25 if has_y_acc_in else 1.0)
181|         if target_info.cuda_capability_geq(10, 0):
182|             acc_size = epilogue_effective_itemsize or out_itemsize
183|         else:
184|             acc_size = out_itemsize
185|         if target_info.cuda_capability_geq(10, 0) and epilogue_subtile is not None:
186|             acc_block_n = block_n // epilogue_subtile // epilogue_reduction_n
187|         else:
188|             acc_block_n = block_n // epilogue_reduction_n
189|         # pipelined TMA store local to global, or
190|         # pipelined layout conversion before store of the accumulator
191|         # note: layout conversion has some padding
192|         epilogue_smem = int((block_m + 4) * acc_block_n * acc_size)
193|         if compute_swap_xw(precision_config, block_m, is_persistent, lhs_dtype, rhs_dtype):
194|             # The fp32 accumulator stays in TMEM for the Blackwell SWAP_XW
195|             # persistent path. Fused reductions such as swiglu still need smem
196|             # for the unreduced output tile before the narrower TMA-store tile.
197|             if epilogue_reduction_n > 1 or epilogue_subtile > 1:
198|                 epilogue_smem += int(block_m * block_n * out_itemsize)
199|         smem_capacity -= epilogue_smem
200|         if x_transpose:
201|             smem_capacity -= int(block_m * block_k * act_size)
202| 
203|     # Persistent fp32 kernels need extra smem headroom (metadata/barriers/TMA state)
204|     # that is not fully captured by the simple stage_size model above.
```
**EN:** Checks `is_persistent`. The true branch mainly prepares intermediate values; branches on runtime conditions.

**CN:** 检查 `is_persistent`. 真分支主要准备中间值; 根据运行时条件分支.

### Block 57 — Lines 205-206 (compute_num_stages)
```python
205|     if is_persistent and (lhs_dtype == FP32 or rhs_dtype == FP32):
206|         smem_capacity -= 32 * 1024
```
**EN:** Checks `is_persistent and (lhs_dtype == FP32 or rhs_dtype == FP32)`..

**CN:** 检查 `is_persistent and (lhs_dtype == FP32 or rhs_dtype == FP32)`..

### Block 58 — Lines 207-210 (compute_num_stages)
```python
207|     if is_persistent and not has_native_mxfp and epilogue_reduction_n > 1:
208|         # Hopper fused reductions materialize an additional reduced-N output
209|         # tile in smem.
210|         smem_capacity -= int(block_m * acc_block_n * out_itemsize)
```
**EN:** Checks `is_persistent and (not has_native_mxfp) and (epilogue_reduction_n > 1)`..

**CN:** 检查 `is_persistent and (not has_native_mxfp) and (epilogue_reduction_n > 1)`..

### Block 59 — Lines 211-211 (compute_num_stages)
```python
211|     smem_capacity = max(smem_capacity, 0)
```
**EN:** Assigns `smem_capacity` and calls `max`.

**CN:** 将 `smem_capacity`，并调用 `max`.

### Block 60 — Lines 212-212 (compute_num_stages)
```python
212|     max_stages = 5 if rhs_dtype == FP4 else 4  # maybe 5 everywhere; just haven't tested
```
**EN:** Assigns `max_stages` and uses conditional expression `5 if rhs_dtype == FP4 else 4`.

**CN:** 将 `max_stages` and 使用条件表达式 `5 if rhs_dtype == FP4 else 4`.

### Block 61 — Lines 213-214 (compute_num_stages)
```python
213|     b_mx_scale_layout = None if not isinstance(precision_config.b_mx_scale,
214|                                                Tensor) else precision_config.b_mx_scale.storage.layout
```
**EN:** Assigns `b_mx_scale_layout` and uses conditional expression `None if not isinstance(precision_config.b_mx_scale, Tenso...`.

**CN:** 将 `b_mx_scale_layout` and 使用条件表达式 `None if not isinstance(precision_config.b_mx_scale, Tenso...`.

### Block 62 — Lines 215-220 (compute_num_stages)
```python
215|     if (is_persistent and rhs_dtype == FP4 and isinstance(b_mx_scale_layout, HopperMXScaleLayout)
216|             and precision_config.a_mx_scale is not None and precision_config.c_mx_scale is not None):
217|         # The Hopper-scale FP4 path with MX input and output needs enough
218|         # extra epilogue/scale smem that a 5-stage persistent kernel can
219|         # exceed H100's launch limit.
220|         max_stages = 4
```
**EN:** Checks `is_persistent and rhs_dtype == FP4 and isinstance(b_mx_scale_layout, HopperMXScaleLayou...`. The true branch mainly prepares intermediate values.

**CN:** 检查 `is_persistent and rhs_dtype == FP4 and isinstance(b_mx_scale_layout, HopperMXScaleLayou...`. 真分支主要准备中间值.

### Block 63 — Lines 221-222 (compute_num_stages)
```python
221|     num_stages = min(smem_capacity // int(stage_size), max_stages)
222|     # Keep one stage of headroom for persistent fp32 to avoid launch-time OOR.
```
**EN:** Assigns `num_stages` and calls `min`.

**CN:** 将 `num_stages`，并调用 `min`.

### Block 64 — Lines 223-224 (compute_num_stages)
```python
223|     if is_persistent and (lhs_dtype == FP32 or rhs_dtype == FP32):
224|         num_stages = min(num_stages, 3)
```
**EN:** Checks `is_persistent and (lhs_dtype == FP32 or rhs_dtype == FP32)`. The true branch mainly prepares intermediate values.

**CN:** 检查 `is_persistent and (lhs_dtype == FP32 or rhs_dtype == FP32)`. 真分支主要准备中间值.

### Block 65 — Lines 225-226 (compute_num_stages)
```python
225|     if num_stages == 0:
226|         num_stages = 1
```
**EN:** Checks `num_stages == 0`. The true branch mainly prepares intermediate values.

**CN:** 检查 `num_stages == 0`. 真分支主要准备中间值.

### Block 66 — Lines 227-227 (compute_num_stages)
```python
227|     return num_stages
```
**EN:** Returns `num_stages`.

**CN:** 返回 `num_stages`.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `is_x_scale_swizzled`, `is_blackwell_mx_lhs_dense_rhs`, `compute_swap_xw`, `compute_grid_size`, `compute_block_n`, `compute_block_k`, `compute_split_k`, `compute_num_warps`.
  **CN:** 主要符号：`is_x_scale_swizzled`, `is_blackwell_mx_lhs_dense_rhs`, `compute_swap_xw`, `compute_grid_size`, `compute_block_n`, `compute_block_k`, `compute_split_k`, `compute_num_warps`。
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
- **EN:** SwiGLU fusion or activation-specific handling is part of the computation flow.
  **CN:** 计算流程中包含 SwiGLU 融合或激活函数相关处理。

## Dependencies / 依赖关系
- **EN:** External modules: `torch`, `triton`.
  **CN:** 外部模块：`torch`, `triton`。
- **EN:** Internal modules: `triton_kernels (target_info)`, `triton_kernels.numerics_details.mxfp_details._downcast_to_mxfp (MXFP_BLOCK_SIZE)`, `triton_kernels.tensor (FP4, FP16, FP32, BF16, Tensor)`, `triton_kernels.tensor_details.layout (HopperMXScaleLayout)`, `triton_kernels.tensor_details.layout_details.blackwell_scale (BlackwellActMXScaleLayout, BlackwellMXScaleLayout)`.
  **CN:** 内部模块：`triton_kernels (target_info)`, `triton_kernels.numerics_details.mxfp_details._downcast_to_mxfp (MXFP_BLOCK_SIZE)`, `triton_kernels.tensor (FP4, FP16, FP32, BF16, Tensor)`, `triton_kernels.tensor_details.layout (HopperMXScaleLayout)`, `triton_kernels.tensor_details.layout_details.blackwell_scale (BlackwellActMXScaleLayout, BlackwellMXScaleLayout)`。
