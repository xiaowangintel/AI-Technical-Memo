# bench_mlp.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/bench/bench_mlp.py`
- **Purpose / 用途:** Benchmark driver for bench mlp; it measures performance, sweeps configurations, and reports results for Triton kernels. / 用于 bench mlp 的基准测试驱动；它会测量性能、扫描配置，并报告 Triton 内核的结果。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-23 (module)
```python
 1| from pathlib import Path
 2| from copy import deepcopy
 3| import os
 4| import triton.profiler as proton
 5| import torch
 6| import argparse
 7| import triton_kernels.roofline as roofline
 8| from triton_kernels.swiglu import swiglu_fn
 9| from triton_kernels.matmul import matmul, PrecisionConfig, FlexCtx, FnSpecs, FusedActivation
10| from triton_kernels.matmul_details.opt_flags import scoped_opt_flags_constraints
11| from triton_kernels.target_info import get_cdna_version
12| from triton_kernels.tensor_details import layout
13| from triton_kernels.reduce import reduce
14| from triton_kernels.topk import topk
15| from triton_kernels.tensor import make_ragged_tensor_metadata, remap_ragged_tensor_metadata  # ragged tensor
16| from triton_kernels.distributed import convert_dp_to_ep, convert_ep_to_dp, make_expt_dict_uniform, make_expt_assignment, SymmetricMemoryPool
17| from triton_kernels.distributed_details.mesh import Mesh
18| # quantization
19| from triton_kernels.tensor import convert_layout, wrap_torch_tensor, FP4
20| from triton_kernels.numerics import InFlexData
21| from triton_kernels.numerics_details.mxfp import MXFP_BLOCK_SIZE, downcast_to_mxfp
22| 
23| 
```
**EN:** This block imports `pathlib (Path)`, `copy (deepcopy)`, `os`, `triton.profiler`, `torch`, `argparse`, `triton_kernels.roofline`, `triton_kernels.swiglu (swiglu_fn)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `pathlib (Path)`, `copy (deepcopy)`, `os`, `triton.profiler`, `torch`, `argparse`, `triton_kernels.roofline`, `triton_kernels.swiglu (swiglu_fn)` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 24-24 (was_launched_with_torchrun)
```python
24| def was_launched_with_torchrun():
```
**EN:** Defines function `was_launched_with_torchrun()` for this module. The body mainly prepares intermediate values; returns the computed result. It uses calls such as `all` to implement its workflow.

**CN:** 定义函数 `was_launched_with_torchrun()`，供本模块使用. 主体主要准备中间值; 返回计算结果. 其中会调用 `all` 来实现其工作流程.

### Block 3 — Lines 25-25 (was_launched_with_torchrun)
```python
25|     required = ["RANK", "WORLD_SIZE", "LOCAL_RANK", "MASTER_ADDR", "MASTER_PORT"]
```
**EN:** Assigns `required` and builds a list.

**CN:** 将 `required` and 构造一个列表.

### Block 4 — Lines 26-28 (was_launched_with_torchrun)
```python
26|     return all(k in os.environ for k in required)
27| 
28| 
```
**EN:** Returns `all((k in os.environ for k in required))`.

**CN:** 返回 `all((k in os.environ for k in required))`.

### Block 5 — Lines 29-29 (parse_dtype)
```python
29| def parse_dtype(dtype):
```
**EN:** Defines function `parse_dtype(dtype)` for this module. The body mainly prepares intermediate values; branches on runtime conditions; returns the computed result. It uses calls such as `get_cdna_version` to implement its workflow.

**CN:** 定义函数 `parse_dtype(dtype)`，供本模块使用. 主体主要准备中间值; 根据运行时条件分支; 返回计算结果. 其中会调用 `get_cdna_version` 来实现其工作流程.

### Block 6 — Lines 30-30 (parse_dtype)
```python
30|     ret = {"fp16": torch.float16, "bf16": torch.bfloat16, "fp8": torch.float8_e4m3fn, "mx4": FP4}[dtype]
```
**EN:** Assigns `ret` and evaluates `{'fp16': torch.float16, 'bf16': torch.bfloat16, 'fp8': torch.float8...`.

**CN:** 将 `ret` and 计算 `{'fp16': torch.float16, 'bf16': torch.bfloat16, 'fp8': torch.float8...`.

### Block 7 — Lines 31-32 (parse_dtype)
```python
31|     if ret == torch.float8_e4m3fn and get_cdna_version() == 3:
32|         ret = torch.float8_e4m3fnuz
```
**EN:** Checks `ret == torch.float8_e4m3fn and get_cdna_version() == 3`. The true branch mainly prepares intermediate values.

**CN:** 检查 `ret == torch.float8_e4m3fn and get_cdna_version() == 3`. 真分支主要准备中间值.

### Block 8 — Lines 33-35 (parse_dtype)
```python
33|     return ret
34| 
35| 
```
**EN:** Returns `ret`.

**CN:** 返回 `ret`.

### Block 9 — Lines 36-36 (quantize_weight)
```python
36| def quantize_weight(w, dtype, **opt):
```
**EN:** Defines function `quantize_weight(w, dtype, **opt)` for this module. The body mainly branches on runtime conditions. It uses calls such as `w.to(torch.bfloat16).transpose(-1, -2...`, `InFlexData`, `w.to`, `wq.transpose(-1, -2).contiguous().tra...`, `downcast_to_mxfp` to implement its workflow.

**CN:** 定义函数 `quantize_weight(w, dtype, **opt)`，供本模块使用. 主体主要根据运行时条件分支. 其中会调用 `w.to(torch.bfloat16).transpose(-1, -2...`, `InFlexData`, `w.to`, `wq.transpose(-1, -2).contiguous().tra...`, `downcast_to_mxfp` 来实现其工作流程.

### Block 10 — Lines 37-59 (quantize_weight)
```python
37|     if dtype == torch.bfloat16:
38|         wq = w.to(torch.bfloat16).transpose(-1, -2).contiguous().transpose(-1, -2)
39|         return wq, InFlexData(), None
40|     elif dtype in [torch.float8_e4m3fn, torch.float8_e4m3fnuz]:
41|         fp8e4_dtype = torch.float8_e4m3fn if get_cdna_version() != 3 else torch.float8_e4m3fnuz
42|         wq = w.to(fp8e4_dtype)
43|         wq = wq.transpose(-1, -2).contiguous().transpose(-1, -2)
44|         return wq, InFlexData(dtype=wq.dtype, scale=w.abs().max().unsqueeze(0)), None
45|     else:
46|         assert dtype == FP4, f"{dtype=}"
47|         w, w_scale = downcast_to_mxfp(w.to(torch.bfloat16), torch.uint8, axis=1)
48|         if opt:
49|             w = wrap_torch_tensor(w, dtype=FP4)
50|             value_layout = opt.get("value_layout")
51|             if value_layout is not None:
52|                 w = convert_layout(w, value_layout)
53|             w_scale = wrap_torch_tensor(w_scale)
54|             scale_layout = opt.get("scale_layout")
55|             if scale_layout is not None:
56|                 w_scale = convert_layout(w_scale, scale_layout)
57|         return w, InFlexData(), w_scale
58| 
59| 
```
**EN:** Checks `dtype == torch.bfloat16`. The true branch mainly prepares intermediate values; returns the computed result, while the else branch branches on runtime conditions.

**CN:** 检查 `dtype == torch.bfloat16`. 真分支主要准备中间值; 返回计算结果；而 else 分支根据运行时条件分支.

### Block 11 — Lines 60-69 (run_mlp)
```python
60| def run_mlp(x_dp_local_bf16, x_dp_local_fp8,  # activations
61|             wg_global, bg_global, pcg,  # gate parameters / precision config
62|             w1_ep_local, b1_ep_local, pc1, act1,  # first matmul parameters / precision config / fused activation
63|             w2_ep_local, b2_ep_local, pc2,  # second matmul parameters / precision config
64|             n_expts_act, expt_assignment,  # expert assignment
65|             rank,  # distributed context
66|             symm_mem_pool,  # symmetric memory pool
67|             fc1_constraints=None, fc2_constraints=None,  # per-kernel opt_flags constraints
68|             ):
69|     # gate matrix multiplication
```
**EN:** Defines function `run_mlp(x_dp_local_bf16, x_dp_local_fp8, wg_global, bg_global, pcg, w1_ep_local, b1_ep_local, pc1, act1, w2_ep_local, b2_ep_local, pc2, n_expts_act, expt_assignment, rank, symm_mem_pool, fc1_constraints, fc2_constraints)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `matmul`, `topk`, `make_ragged_tensor_metadata`, `convert_dp_to_ep`, `remap_ragged_tensor_metadata` to implement its workflow.

**CN:** 定义函数 `run_mlp(x_dp_local_bf16, x_dp_local_fp8, wg_global, bg_global, pcg, w1_ep_local, b1_ep_local, pc1, act1, w2_ep_local, b2_ep_local, pc2, n_expts_act, expt_assignment, rank, symm_mem_pool, fc1_constraints, fc2_constraints)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `matmul`, `topk`, `make_ragged_tensor_metadata`, `convert_dp_to_ep`, `remap_ragged_tensor_metadata` 来实现其工作流程.

### Block 12 — Lines 70-71 (run_mlp)
```python
70|     l_dp_local = matmul(x_dp_local_bf16, wg_global, bg_global, precision_config=pcg)
71|     # active global logits (sparse)
```
**EN:** Assigns `l_dp_local` and calls `matmul`.

**CN:** 将 `l_dp_local`，并调用 `matmul`.

### Block 13 — Lines 72-73 (run_mlp)
```python
72|     l_global_active = topk(l_dp_local, n_expts_act, apply_softmax=True, all_gather=True, symm_mem_pool=symm_mem_pool)
73|     # expert histogram, dispatch/combine indx
```
**EN:** Assigns `l_global_active` and calls `topk`.

**CN:** 将 `l_global_active`，并调用 `topk`.

### Block 14 — Lines 74-74 (run_mlp)
```python
74|     active_indx = l_global_active.indx
```
**EN:** Assigns `active_indx` and references `l_global_active.indx`.

**CN:** 将 `active_indx` and 引用 `l_global_active.indx`.

### Block 15 — Lines 75-75 (run_mlp)
```python
75|     expt_sizes = l_global_active.mask_metadata.col_sum
```
**EN:** Assigns `expt_sizes` and references `l_global_active.mask_metadata.col_sum`.

**CN:** 将 `expt_sizes` and 引用 `l_global_active.mask_metadata.col_sum`.

### Block 16 — Lines 76-76 (run_mlp)
```python
76|     dispatch_indx = l_global_active.mask_metadata.row_sorted_indx
```
**EN:** Assigns `dispatch_indx` and references `l_global_active.mask_metadata.row_sorted_indx`.

**CN:** 将 `dispatch_indx` and 引用 `l_global_active.mask_metadata.row_sorted_indx`.

### Block 17 — Lines 77-78 (run_mlp)
```python
77|     combine_indx = l_global_active.mask_metadata.col_sorted_indx
78|     # ragged tensor metadata
```
**EN:** Assigns `combine_indx` and references `l_global_active.mask_metadata.col_sorted_indx`.

**CN:** 将 `combine_indx` and 引用 `l_global_active.mask_metadata.col_sorted_indx`.

### Block 18 — Lines 79-80 (run_mlp)
```python
79|     x_global_metadata = make_ragged_tensor_metadata(expt_sizes, dispatch_indx.shape[0])
80|     # convert x from dp-local to expert-sorted, ep-local
```
**EN:** Assigns `x_global_metadata` and calls `make_ragged_tensor_metadata`.

**CN:** 将 `x_global_metadata`，并调用 `make_ragged_tensor_metadata`.

### Block 19 — Lines 81-81 (run_mlp)
```python
81|     y_ep_local = convert_dp_to_ep(x_dp_local_fp8, expt_assignment, active_indx, dispatch_indx, symm_mem_pool)
```
**EN:** Assigns `y_ep_local` and calls `convert_dp_to_ep`.

**CN:** 将 `y_ep_local`，并调用 `convert_dp_to_ep`.

### Block 20 — Lines 82-83 (run_mlp)
```python
82|     y_ep_local_metadata = remap_ragged_tensor_metadata(x_global_metadata, expt_assignment.expt_map[rank, :])
83|     # first matmul + swiglu
```
**EN:** Assigns `y_ep_local_metadata` and calls `remap_ragged_tensor_metadata`.

**CN:** 将 `y_ep_local_metadata`，并调用 `remap_ragged_tensor_metadata`.

### Block 21 — Lines 84-87 (run_mlp)
```python
84|     with scoped_opt_flags_constraints(fc1_constraints or {}):
85|         y_ep_local = matmul(y_ep_local, w1_ep_local, b1_ep_local, a_ragged_metadata=y_ep_local_metadata,
86|                             precision_config=pc1, fused_activation=act1)
87|     # second matmul
```
**EN:** Uses context manager(s) `scoped_opt_flags_constraints(fc1_constrain...` around code that mainly prepares intermediate values.

**CN:** 使用上下文管理器 `scoped_opt_flags_constraints(fc1_constrain...` ，其中代码主要准备中间值.

### Block 22 — Lines 88-91 (run_mlp)
```python
88|     with scoped_opt_flags_constraints(fc2_constraints or {}):
89|         y_ep_local = matmul(y_ep_local, w2_ep_local, b2_ep_local, a_ragged_metadata=y_ep_local_metadata,
90|                             precision_config=pc2)
91|     # convert x from expert-sorted, ep-local to token-sorted, dp-local
```
**EN:** Uses context manager(s) `scoped_opt_flags_constraints(fc2_constrain...` around code that mainly prepares intermediate values.

**CN:** 使用上下文管理器 `scoped_opt_flags_constraints(fc2_constrain...` ，其中代码主要准备中间值.

### Block 23 — Lines 92-93 (run_mlp)
```python
92|     y_dp_local = convert_ep_to_dp(y_ep_local, expt_assignment, active_indx, combine_indx, symm_mem_pool)
93|     # weighted average of the output token from experts
```
**EN:** Assigns `y_dp_local` and calls `convert_ep_to_dp`.

**CN:** 将 `y_dp_local`，并调用 `convert_ep_to_dp`.

### Block 24 — Lines 94-94 (run_mlp)
```python
94|     y_dp_local = y_dp_local.view(-1, n_expts_act, y_dp_local.shape[-1])
```
**EN:** Assigns `y_dp_local` and calls `y_dp_local.view`.

**CN:** 将 `y_dp_local`，并调用 `y_dp_local.view`.

### Block 25 — Lines 95-95 (run_mlp)
```python
95|     z_dp_local, _ = reduce(y_dp_local, dim=1)
```
**EN:** Assigns `z_dp_local`, `_` and calls `reduce`.

**CN:** 将 `z_dp_local`, `_`，并调用 `reduce`.

### Block 26 — Lines 96-98 (run_mlp)
```python
96|     return z_dp_local
97| 
98| 
```
**EN:** Returns `z_dp_local`.

**CN:** 返回 `z_dp_local`.

### Block 27 — Lines 99-100 (bench_mlp)
```python
 99| def bench_mlp(batch_per_expt, dim1, dim2, n_expts_tot, n_expts_act, x_dtype, w_dtype, EP, shuffle_mx4=False,
100|               num_stages_fc1=None, num_stages_fc2=None, epilogue_subtile_fc1=None):
```
**EN:** Defines function `bench_mlp(batch_per_expt, dim1, dim2, n_expts_tot, n_expts_act, x_dtype, w_dtype, EP, shuffle_mx4, num_stages_fc1, num_stages_fc2, epilogue_subtile_fc1)` for this module. The body mainly checks invariants; prepares intermediate values; prepares intermediate values. It uses calls such as `torch.distributed.get_rank`, `torch.distributed.get_world_size`, `torch.cuda.current_device`, `SymmetricMemoryPool`, `symm_mem_pool.initialize_matmul` to implement its workflow.

**CN:** 定义函数 `bench_mlp(batch_per_expt, dim1, dim2, n_expts_tot, n_expts_act, x_dtype, w_dtype, EP, shuffle_mx4, num_stages_fc1, num_stages_fc2, epilogue_subtile_fc1)`，供本模块使用. 主体主要检查不变量; 准备中间值; 准备中间值. 其中会调用 `torch.distributed.get_rank`, `torch.distributed.get_world_size`, `torch.cuda.current_device`, `SymmetricMemoryPool`, `symm_mem_pool.initialize_matmul` 来实现其工作流程.

### Block 28 — Lines 101-101 (bench_mlp)
```python
101|     assert n_expts_tot % EP == 0
```
**EN:** Asserts `n_expts_tot % EP == 0` to enforce invariants.

**CN:** 断言 `n_expts_tot % EP == 0` 以确保不变量成立。

### Block 29 — Lines 102-102 (bench_mlp)
```python
102|     rank = torch.distributed.get_rank()
```
**EN:** Assigns `rank` and calls `torch.distributed.get_rank`.

**CN:** 将 `rank`，并调用 `torch.distributed.get_rank`.

### Block 30 — Lines 103-103 (bench_mlp)
```python
103|     n_ranks = torch.distributed.get_world_size()
```
**EN:** Assigns `n_ranks` and calls `torch.distributed.get_world_size`.

**CN:** 将 `n_ranks`，并调用 `torch.distributed.get_world_size`.

### Block 31 — Lines 104-104 (bench_mlp)
```python
104|     dev = torch.cuda.current_device()
```
**EN:** Assigns `dev` and calls `torch.cuda.current_device`.

**CN:** 将 `dev`，并调用 `torch.cuda.current_device`.

### Block 32 — Lines 105-105 (bench_mlp)
```python
105|     assert dev == rank, f"{torch.cuda.get_current_device()=}, {rank=}"
```
**EN:** Asserts `dev == rank` to enforce invariants.

**CN:** 断言 `dev == rank` 以确保不变量成立。

### Block 33 — Lines 106-106 (bench_mlp)
```python
106|     assert n_expts_tot % EP == 0, f"{n_expts_tot=}, {EP=}, n_expts_tot must be divisible by EP"
```
**EN:** Asserts `n_expts_tot % EP == 0` to enforce invariants.

**CN:** 断言 `n_expts_tot % EP == 0` 以确保不变量成立。

### Block 34 — Lines 107-107 (bench_mlp)
```python
107|     batch = batch_per_expt * n_expts_tot // n_expts_act
```
**EN:** Assigns `batch` and evaluates `batch_per_expt * n_expts_tot // n_expts_act`.

**CN:** 将 `batch` and 计算 `batch_per_expt * n_expts_tot // n_expts_act`.

### Block 35 — Lines 108-110 (bench_mlp)
```python
108|     assert EP == n_ranks, f"{EP=}, {n_ranks=}"
109| 
110|     #-- init memory pool --
```
**EN:** Asserts `EP == n_ranks` to enforce invariants.

**CN:** 断言 `EP == n_ranks` 以确保不变量成立。

### Block 36 — Lines 111-111 (bench_mlp)
```python
111|     symm_mem_pool = SymmetricMemoryPool(Mesh(torch.distributed.group.WORLD))
```
**EN:** Assigns `symm_mem_pool` and calls `SymmetricMemoryPool`.

**CN:** 将 `symm_mem_pool`，并调用 `SymmetricMemoryPool`.

### Block 37 — Lines 112-123 (bench_mlp)
```python
112|     symm_mem_pool.initialize_matmul(
113|         n_tokens_global=batch_per_expt * n_expts_tot // n_expts_act,
114|         d_input=dim1,
115|         d_model=dim2,
116|         n_expts_act=n_expts_act,
117|         n_expts_tot=n_expts_tot,
118|         dtype=x_dtype,
119|         device=torch.device(dev),
120|     )
121| 
122|     # -- init prameters --
123|     # weights
```
**EN:** Calls `symm_mem_pool.initialize_matmul` for side effects, registration, or validation.

**CN:** 调用 `symm_mem_pool.initialize_matmul` ，用于副作用、注册或校验。

### Block 38 — Lines 124-124 (bench_mlp)
```python
124|     wg_global = torch.randn((dim1, n_expts_tot), device=dev)
```
**EN:** Assigns `wg_global` and calls `torch.randn`.

**CN:** 将 `wg_global`，并调用 `torch.randn`.

### Block 39 — Lines 125-125 (bench_mlp)
```python
125|     torch.distributed.broadcast(wg_global, src=0)
```
**EN:** Calls `torch.distributed.broadcast` for side effects, registration, or validation.

**CN:** 调用 `torch.distributed.broadcast` ，用于副作用、注册或校验。

### Block 40 — Lines 126-126 (bench_mlp)
```python
126|     w1_ep_local = torch.randn((n_expts_tot // EP, dim1, dim2), device=dev)
```
**EN:** Assigns `w1_ep_local` and calls `torch.randn`.

**CN:** 将 `w1_ep_local`，并调用 `torch.randn`.

### Block 41 — Lines 127-128 (bench_mlp)
```python
127|     w2_ep_local = torch.randn((n_expts_tot // EP, dim2 // 2, dim1), device=dev)
128|     # biases
```
**EN:** Assigns `w2_ep_local` and calls `torch.randn`.

**CN:** 将 `w2_ep_local`，并调用 `torch.randn`.

### Block 42 — Lines 129-129 (bench_mlp)
```python
129|     bg_global = torch.randn((n_expts_tot, ), device=dev)
```
**EN:** Assigns `bg_global` and calls `torch.randn`.

**CN:** 将 `bg_global`，并调用 `torch.randn`.

### Block 43 — Lines 130-130 (bench_mlp)
```python
130|     torch.distributed.broadcast(bg_global, src=0)
```
**EN:** Calls `torch.distributed.broadcast` for side effects, registration, or validation.

**CN:** 调用 `torch.distributed.broadcast` ，用于副作用、注册或校验。

### Block 44 — Lines 131-131 (bench_mlp)
```python
131|     b1_ep_local = torch.randn((n_expts_tot // EP, dim2), device=dev)
```
**EN:** Assigns `b1_ep_local` and calls `torch.randn`.

**CN:** 将 `b1_ep_local`，并调用 `torch.randn`.

### Block 45 — Lines 132-132 (bench_mlp)
```python
132|     b2_ep_local = torch.randn((n_expts_tot // EP, dim1), device=dev)
```
**EN:** Assigns `b2_ep_local` and calls `torch.randn`.

**CN:** 将 `b2_ep_local`，并调用 `torch.randn`.

### Block 46 — Lines 133-134 (bench_mlp)
```python
133|     torch.distributed.barrier()
134|     # quantize
```
**EN:** Calls `torch.distributed.barrier` for side effects, registration, or validation.

**CN:** 调用 `torch.distributed.barrier` ，用于副作用、注册或校验。

### Block 47 — Lines 135-135 (bench_mlp)
```python
135|     opt1 = dict()
```
**EN:** Assigns `opt1` and calls `dict`.

**CN:** 将 `opt1`，并调用 `dict`.

### Block 48 — Lines 136-136 (bench_mlp)
```python
136|     opt2 = dict()
```
**EN:** Assigns `opt2` and calls `dict`.

**CN:** 将 `opt2`，并调用 `dict`.

### Block 49 — Lines 137-148 (bench_mlp)
```python
137|     if w_dtype == FP4:
138|         num_warps = 4 if batch <= 512 else 8
139|         value_layout = layout.make_default_matmul_mxfp4_w_layout(
140|             mx_axis=-2,
141|             allow_blackwell_value_shuffle=shuffle_mx4,
142|         )
143|         scale_layout = layout.make_default_matmul_mxfp4_w_scale_layout(mx_axis=-2, num_warps=num_warps)
144|         opt1 = {
145|             "value_layout": value_layout,
146|             "scale_layout": scale_layout,
147|         }
148|         opt2 = deepcopy(opt1)
```
**EN:** Checks `w_dtype == FP4`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `w_dtype == FP4`. 真分支主要准备中间值; 准备中间值.

### Block 50 — Lines 149-149 (bench_mlp)
```python
149|     wg_global, wg_flex, wg_scale = quantize_weight(wg_global, torch.bfloat16)
```
**EN:** Assigns `wg_global`, `wg_flex`, `wg_scale` and calls `quantize_weight`.

**CN:** 将 `wg_global`, `wg_flex`, `wg_scale`，并调用 `quantize_weight`.

### Block 51 — Lines 150-150 (bench_mlp)
```python
150|     w1_ep_local, w1_flex, w1_scale = quantize_weight(w1_ep_local, w_dtype, **opt1)
```
**EN:** Assigns `w1_ep_local`, `w1_flex`, `w1_scale` and calls `quantize_weight`.

**CN:** 将 `w1_ep_local`, `w1_flex`, `w1_scale`，并调用 `quantize_weight`.

### Block 52 — Lines 151-151 (bench_mlp)
```python
151|     w2_ep_local, w2_flex, w2_scale = quantize_weight(w2_ep_local, w_dtype, **opt2)
```
**EN:** Assigns `w2_ep_local`, `w2_flex`, `w2_scale` and calls `quantize_weight`.

**CN:** 将 `w2_ep_local`, `w2_flex`, `w2_scale`，并调用 `quantize_weight`.

### Block 53 — Lines 152-156 (bench_mlp)
```python
152|     pcg = PrecisionConfig(
153|         flex_ctx=FlexCtx(rhs_data=wg_flex),
154|         b_mx_scale=wg_scale,
155|         b_microblock_size=MXFP_BLOCK_SIZE.value,
156|     )
```
**EN:** Assigns `pcg` and calls `PrecisionConfig`.

**CN:** 将 `pcg`，并调用 `PrecisionConfig`.

### Block 54 — Lines 157-161 (bench_mlp)
```python
157|     pc1 = PrecisionConfig(
158|         flex_ctx=FlexCtx(rhs_data=w1_flex),
159|         b_mx_scale=w1_scale,
160|         b_microblock_size=MXFP_BLOCK_SIZE.value,
161|     )
```
**EN:** Assigns `pc1` and calls `PrecisionConfig`.

**CN:** 将 `pc1`，并调用 `PrecisionConfig`.

### Block 55 — Lines 162-168 (bench_mlp)
```python
162|     pc2 = PrecisionConfig(
163|         flex_ctx=FlexCtx(rhs_data=w2_flex),
164|         b_mx_scale=w2_scale,
165|         b_microblock_size=MXFP_BLOCK_SIZE.value,
166|     )
167| 
168|     # -- init activation --
```
**EN:** Assigns `pc2` and calls `PrecisionConfig`.

**CN:** 将 `pc2`，并调用 `PrecisionConfig`.

### Block 56 — Lines 169-169 (bench_mlp)
```python
169|     x_dp_local_fp8 = torch.randn((batch // n_ranks, dim1), device=dev).to(x_dtype)
```
**EN:** Assigns `x_dp_local_fp8` and calls `torch.randn((batch // n_ranks, dim1), device=de...`.

**CN:** 将 `x_dp_local_fp8`，并调用 `torch.randn((batch // n_ranks, dim1), device=de...`.

### Block 57 — Lines 170-172 (bench_mlp)
```python
170|     x_dp_local_bf16 = x_dp_local_fp8.to(torch.bfloat16)
171| 
172|     # -- matmul fusion options --
```
**EN:** Assigns `x_dp_local_bf16` and calls `x_dp_local_fp8.to`.

**CN:** 将 `x_dp_local_bf16`，并调用 `x_dp_local_fp8.to`.

### Block 58 — Lines 173-175 (bench_mlp)
```python
173|     act1 = FusedActivation(FnSpecs("swiglu", swiglu_fn, ("alpha", "limit"), reduction_n=2), (1.0, 1.0))
174| 
175|     # -- run benchmark --
```
**EN:** Assigns `act1` and calls `FusedActivation`.

**CN:** 将 `act1`，并调用 `FusedActivation`.

### Block 59 — Lines 176-176 (bench_mlp)
```python
176|     expt_dict = make_expt_dict_uniform(EP, n_expts_tot)
```
**EN:** Assigns `expt_dict` and calls `make_expt_dict_uniform`.

**CN:** 将 `expt_dict`，并调用 `make_expt_dict_uniform`.

### Block 60 — Lines 177-179 (bench_mlp)
```python
177|     expt_assignment = make_expt_assignment(EP, n_expts_tot, expt_dict, torch.device(dev))
178| 
179|     # Build per-kernel constraints
```
**EN:** Assigns `expt_assignment` and calls `make_expt_assignment`.

**CN:** 将 `expt_assignment`，并调用 `make_expt_assignment`.

### Block 61 — Lines 180-180 (bench_mlp)
```python
180|     fc1_constraints = {}
```
**EN:** Assigns `fc1_constraints` and builds a dictionary.

**CN:** 将 `fc1_constraints` and 构造一个字典.

### Block 62 — Lines 181-182 (bench_mlp)
```python
181|     if num_stages_fc1 is not None:
182|         fc1_constraints["num_stages"] = num_stages_fc1
```
**EN:** Checks `num_stages_fc1 is not None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `num_stages_fc1 is not None`. 真分支主要准备中间值.

### Block 63 — Lines 183-184 (bench_mlp)
```python
183|     if epilogue_subtile_fc1 is not None:
184|         fc1_constraints["epilogue_subtile"] = epilogue_subtile_fc1
```
**EN:** Checks `epilogue_subtile_fc1 is not None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `epilogue_subtile_fc1 is not None`. 真分支主要准备中间值.

### Block 64 — Lines 185-185 (bench_mlp)
```python
185|     fc2_constraints = {}
```
**EN:** Assigns `fc2_constraints` and builds a dictionary.

**CN:** 将 `fc2_constraints` and 构造一个字典.

### Block 65 — Lines 186-188 (bench_mlp)
```python
186|     if num_stages_fc2 is not None:
187|         fc2_constraints["num_stages"] = num_stages_fc2
188| 
```
**EN:** Checks `num_stages_fc2 is not None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `num_stages_fc2 is not None`. 真分支主要准备中间值.

### Block 66 — Lines 189-191 (bench_mlp)
```python
189|     fpath = Path(f"profile_{rank}")
190|     # Compile and warm up outside the profiler so subsequent profiled launches
191|     # retain launch metadata needed by roofline.parse_profile.
```
**EN:** Assigns `fpath` and calls `Path`.

**CN:** 将 `fpath`，并调用 `Path`.

### Block 67 — Lines 192-197 (bench_mlp)
```python
192|     run_mlp(x_dp_local_bf16, x_dp_local_fp8,  #
193|             wg_global, bg_global, pcg,  #
194|             w1_ep_local, b1_ep_local, pc1, act1,  #
195|             w2_ep_local, b2_ep_local, pc2,  #
196|             n_expts_act, expt_assignment, rank, symm_mem_pool, fc1_constraints=fc1_constraints,
197|             fc2_constraints=fc2_constraints)
```
**EN:** Calls `run_mlp` for side effects, registration, or validation.

**CN:** 调用 `run_mlp` ，用于副作用、注册或校验。

### Block 68 — Lines 198-198 (bench_mlp)
```python
198|     torch.cuda.synchronize()
```
**EN:** Calls `torch.cuda.synchronize` for side effects, registration, or validation.

**CN:** 调用 `torch.cuda.synchronize` ，用于副作用、注册或校验。

### Block 69 — Lines 199-199 (bench_mlp)
```python
199|     proton.start(str(fpath), hook="triton")
```
**EN:** Calls `proton.start` for side effects, registration, or validation.

**CN:** 调用 `proton.start` ，用于副作用、注册或校验。

### Block 70 — Lines 200-206 (bench_mlp)
```python
200|     for i in range(100):
201|         run_mlp(x_dp_local_bf16, x_dp_local_fp8,  #
202|                 wg_global, bg_global, pcg,  #
203|                 w1_ep_local, b1_ep_local, pc1, act1,  #
204|                 w2_ep_local, b2_ep_local, pc2,  #
205|                 n_expts_act, expt_assignment, rank, symm_mem_pool, fc1_constraints=fc1_constraints,
206|                 fc2_constraints=fc2_constraints)
```
**EN:** Loops over `range(100)` with target `i`. The loop body mainly invokes `run_mlp`.

**CN:** 遍历 `range(100)` ，目标变量为 `i`. 循环体主要invokes `run_mlp`.

### Block 71 — Lines 207-207 (bench_mlp)
```python
207|     torch.cuda.synchronize()
```
**EN:** Calls `torch.cuda.synchronize` for side effects, registration, or validation.

**CN:** 调用 `torch.cuda.synchronize` ，用于副作用、注册或校验。

### Block 72 — Lines 208-208 (bench_mlp)
```python
208|     torch.distributed.barrier()
```
**EN:** Calls `torch.distributed.barrier` for side effects, registration, or validation.

**CN:** 调用 `torch.distributed.barrier` ，用于副作用、注册或校验。

### Block 73 — Lines 209-209 (bench_mlp)
```python
209|     proton.finalize()
```
**EN:** Calls `proton.finalize` for side effects, registration, or validation.

**CN:** 调用 `proton.finalize` ，用于副作用、注册或校验。

### Block 74 — Lines 210-212 (bench_mlp)
```python
210|     return roofline.parse_profile(fpath.with_suffix(".hatchet"), useful_op_regex=".*matmul.*")
211| 
212| 
```
**EN:** Returns `roofline.parse_profile(fpath.with_suffix('.hatchet'), useful_op_regex='.*matm...`.

**CN:** 返回 `roofline.parse_profile(fpath.with_suffix('.hatchet'), useful_op_regex='.*matm...`.

### Block 75 — Lines 213-215 (roofline_mlp)
```python
213| def roofline_mlp(batch_sizes, dim1, dim2, n_expts_tot, n_expts_act, x_dtype, w_dtype, EP, \
214|                   name="", verbose=True, shuffle_mx4=False, num_stages_fc1=None,
215|                   num_stages_fc2=None, epilogue_subtile_fc1=None):
```
**EN:** Defines function `roofline_mlp(batch_sizes, dim1, dim2, n_expts_tot, n_expts_act, x_dtype, w_dtype, EP, name, verbose, shuffle_mx4, num_stages_fc1, num_stages_fc2, epilogue_subtile_fc1)` for this module. The body mainly prepares intermediate values; prepares intermediate values; invokes `out_path.mkdir`. It uses calls such as `Path`, `out_path.mkdir`, `torch.cuda.set_device`, `roofline.compute_roofline`, `roofline.plot_roofline` to implement its workflow.

**CN:** 定义函数 `roofline_mlp(batch_sizes, dim1, dim2, n_expts_tot, n_expts_act, x_dtype, w_dtype, EP, name, verbose, shuffle_mx4, num_stages_fc1, num_stages_fc2, epilogue_subtile_fc1)`，供本模块使用. 主体主要准备中间值; 准备中间值; invokes `out_path.mkdir`. 其中会调用 `Path`, `out_path.mkdir`, `torch.cuda.set_device`, `roofline.compute_roofline`, `roofline.plot_roofline` 来实现其工作流程.

### Block 76 — Lines 216-216 (roofline_mlp)
```python
216|     suffix = "-shuffled" if shuffle_mx4 else ""
```
**EN:** Assigns `suffix` and uses conditional expression `'-shuffled' if shuffle_mx4 else ''`.

**CN:** 将 `suffix` and 使用条件表达式 `'-shuffled' if shuffle_mx4 else ''`.

### Block 77 — Lines 217-217 (roofline_mlp)
```python
217|     suffix += f"-fc1stages{num_stages_fc1}" if num_stages_fc1 is not None else ""
```
**EN:** Updates `suffix` with operator `Add` using `f'-fc1stages{num_stages_fc1}' if num_stages_fc1 is not No...`.

**CN:** 更新 `suffix`，使用运算符 `Add`，并使用 `f'-fc1stages{num_stages_fc1}' if num_stages_fc1 is not No...`.

### Block 78 — Lines 218-218 (roofline_mlp)
```python
218|     suffix += f"-fc2stages{num_stages_fc2}" if num_stages_fc2 is not None else ""
```
**EN:** Updates `suffix` with operator `Add` using `f'-fc2stages{num_stages_fc2}' if num_stages_fc2 is not No...`.

**CN:** 更新 `suffix`，使用运算符 `Add`，并使用 `f'-fc2stages{num_stages_fc2}' if num_stages_fc2 is not No...`.

### Block 79 — Lines 219-219 (roofline_mlp)
```python
219|     suffix += f"-fc1subtile{epilogue_subtile_fc1}" if epilogue_subtile_fc1 is not None else ""
```
**EN:** Updates `suffix` with operator `Add` using `f'-fc1subtile{epilogue_subtile_fc1}' if epilogue_subtile_...`.

**CN:** 更新 `suffix`，使用运算符 `Add`，并使用 `f'-fc1subtile{epilogue_subtile_fc1}' if epilogue_subtile_...`.

### Block 80 — Lines 220-220 (roofline_mlp)
```python
220|     out_path = Path(f"logs/{name}/{x_dtype}x-{w_dtype}w-EP{EP}{suffix}/")
```
**EN:** Assigns `out_path` and calls `Path`.

**CN:** 将 `out_path`，并调用 `Path`.

### Block 81 — Lines 221-221 (roofline_mlp)
```python
221|     out_path.mkdir(parents=True, exist_ok=True)
```
**EN:** Calls `out_path.mkdir` for side effects, registration, or validation.

**CN:** 调用 `out_path.mkdir` ，用于副作用、注册或校验。

### Block 82 — Lines 222-222 (roofline_mlp)
```python
222|     torch.cuda.set_device(torch.distributed.get_rank())
```
**EN:** Calls `torch.cuda.set_device` for side effects, registration, or validation.

**CN:** 调用 `torch.cuda.set_device` ，用于副作用、注册或校验。

### Block 83 — Lines 223-233 (roofline_mlp)
```python
223|     csv_path = roofline.compute_roofline(dim1, dim2, n_expts_tot, n_expts_act, parse_dtype(x_dtype),
224|                                          parse_dtype(w_dtype), EP,  # fixed args
225|                                          shuffle_mx4=shuffle_mx4,  # weight shuffling option
226|                                          num_stages_fc1=num_stages_fc1,  # override num_stages for FC1 kernel
227|                                          num_stages_fc2=num_stages_fc2,  # override num_stages for FC2 kernel
228|                                          epilogue_subtile_fc1=epilogue_subtile_fc1,  # override epilogue subtile for FC1
229|                                          bench_fn=bench_mlp,  # function to benchmark
230|                                          intensity_proxy_name="batch_per_expt",  # intensity proxy name
231|                                          intensity_proxy_values=batch_sizes,  # intensity proxy values to sweep
232|                                          verbose=verbose,  # options
233|                                          out_path=out_path.with_suffix(".csv"))  # output path
```
**EN:** Assigns `csv_path` and calls `roofline.compute_roofline`.

**CN:** 将 `csv_path`，并调用 `roofline.compute_roofline`.

### Block 84 — Lines 234-238 (roofline_mlp)
```python
234|     png_path = roofline.plot_roofline(series=[csv_path],  # roofline data to plot
235|                                       flops_dtype=x_dtype,  # dtype to use for FLOPS roof
236|                                       xlabel="batch_per_expt", title=out_path,  # plot option
237|                                       out_path=out_path.with_suffix(".png"),  # output path
238|                                       max_tbps="memset", max_tflops="cublas")  # hardware limits
```
**EN:** Assigns `png_path` and calls `roofline.plot_roofline`.

**CN:** 将 `png_path`，并调用 `roofline.plot_roofline`.

### Block 85 — Lines 239-241 (roofline_mlp)
```python
239|     return png_path
240| 
241| 
```
**EN:** Returns `png_path`.

**CN:** 返回 `png_path`.

### Block 86 — Lines 242-283 (module)
```python
242| if __name__ == "__main__":
243|     # torchrun --nproc-per-node=2 ./bench_mlp.py --ep 2 --name gpt-oss-x2
244|     if not was_launched_with_torchrun():
245|         print("usage: torchrun --nproc-per-node=<EP> ./bench_mlp.py")
246|     has_native_mx4 = torch.cuda.get_device_capability(0)[0] >= 10 or get_cdna_version() == 4
247|     world_size = int(os.environ["WORLD_SIZE"])
248|     local_rank = int(os.environ["LOCAL_RANK"])
249|     torch.distributed.init_process_group(backend="nccl", world_size=world_size, device_id=torch.device(local_rank))
250|     parser = argparse.ArgumentParser()
251|     parser.add_argument("--quantized", action="store_true", default=True)
252|     args = parser.parse_args()
253|     # set dtypes
254|     dense_dtypes = ["fp8", "fp8"]
255|     quantized_dtypes = ["fp8", "mx4"] if has_native_mx4 else ["bf16", "mx4"]
256|     # set model type
257|     batch_sizes = [1, 2, 4, 8, 16, 32, 64]
258|     ep = torch.distributed.get_world_size()
259| 
260|     # Common args for all MoE scenarios
261|     moe_args = dict(dim1=5760, dim2=5760, n_expts_tot=128, n_expts_act=4, EP=ep, name="gpt-oss-x2")
262| 
263|     # Run 5 scenarios to isolate 3 optimization categories:
264|     #   Shuffling: scenario 2 → 3
265|     #   FC2 5stg:  scenario 3 → 4
266|     #   FC1 5stg:  scenario 4 → 5
267| 
268|     # 1. FP8 baseline
269|     roofline_mlp(batch_sizes, x_dtype=dense_dtypes[0], w_dtype=dense_dtypes[1], **moe_args)
270|     # 2. MX4 baseline
271|     roofline_mlp(batch_sizes, x_dtype=quantized_dtypes[0], w_dtype=quantized_dtypes[1], **moe_args)
272|     # 3. MX4 shuffled
273|     roofline_mlp(batch_sizes, x_dtype=quantized_dtypes[0], w_dtype=quantized_dtypes[1], shuffle_mx4=True, **moe_args)
274|     # 4. MX4 shuffled + FC2 5stg
275|     roofline_mlp(batch_sizes, x_dtype=quantized_dtypes[0], w_dtype=quantized_dtypes[1], shuffle_mx4=True,
276|                  num_stages_fc2=5, **moe_args)
277|     # 5. MX4 shuffled + FC1 subtile2+5stg + FC2 5stg
278|     #    block_k=128 (swap disabled) + subtile=2 frees enough smem for 5 stages
279|     roofline_mlp(batch_sizes, x_dtype=quantized_dtypes[0], w_dtype=quantized_dtypes[1], shuffle_mx4=True,
280|                  epilogue_subtile_fc1=2, num_stages_fc1=5, num_stages_fc2=5, **moe_args)
281| 
282|     torch.distributed.barrier()
283|     torch.distributed.destroy_process_group()
```
**EN:** Checks `__name__ == '__main__'`. The true branch mainly branches on runtime conditions; prepares intermediate values.

**CN:** 检查 `__name__ == '__main__'`. 真分支主要根据运行时条件分支; 准备中间值.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `was_launched_with_torchrun`, `parse_dtype`, `quantize_weight`, `run_mlp`, `bench_mlp`, `roofline_mlp`.
  **CN:** 主要符号：`was_launched_with_torchrun`, `parse_dtype`, `quantize_weight`, `run_mlp`, `bench_mlp`, `roofline_mlp`。
- **EN:** PyTorch tensors are the main data container and host-side orchestration layer.
  **CN:** PyTorch 张量是主要的数据容器和主机端编排层。
- **EN:** Triton is used to JIT-compile or launch GPU kernels and to express tile-level execution details.
  **CN:** Triton 用于 JIT 编译或启动 GPU 内核，并表达分块级执行细节。
- **EN:** Mixed-precision MXFP/NVFP formats and scale tensors are part of the numeric pipeline.
  **CN:** 混合精度 MXFP/NVFP 格式及其缩放张量是数值流程的一部分。
- **EN:** Flex-point style metadata or scaling is used to control quantized arithmetic.
  **CN:** 使用 Flex-point 风格的元数据或缩放来控制量化计算。
- **EN:** Custom layout objects describe how logical tensors are packed or swizzled in memory.
  **CN:** 自定义 layout 对象描述逻辑张量如何在内存中打包或重排。
- **EN:** Distributed execution concepts such as meshes, ranks, or shard mapping appear in this file.
  **CN:** 该文件涉及 mesh、rank 或分片映射等分布式执行概念。
- **EN:** Reduction logic combines partial values across dimensions or shards.
  **CN:** 归约逻辑会沿着维度或分片合并部分结果。

## Dependencies / 依赖关系
- **EN:** External modules: `pathlib (Path)`, `copy (deepcopy)`, `os`, `triton.profiler`, `torch`, `argparse`.
  **CN:** 外部模块：`pathlib (Path)`, `copy (deepcopy)`, `os`, `triton.profiler`, `torch`, `argparse`。
- **EN:** Internal modules: `triton_kernels.roofline`, `triton_kernels.swiglu (swiglu_fn)`, `triton_kernels.matmul (matmul, PrecisionConfig, FlexCtx, FnSpecs, FusedActivation)`, `triton_kernels.matmul_details.opt_flags (scoped_opt_flags_constraints)`, `triton_kernels.target_info (get_cdna_version)`, `triton_kernels.tensor_details (layout)`, `triton_kernels.reduce (reduce)`, `triton_kernels.topk (topk)`, `triton_kernels.tensor (make_ragged_tensor_metadata, remap_ragged_tensor_metadata)`, `triton_kernels.distributed (convert_dp_to_ep, convert_ep_to_dp, make_expt_dict_uniform, make_expt_assignment, SymmetricMemoryPool)`, `triton_kernels.distributed_details.mesh (Mesh)`, `triton_kernels.tensor (convert_layout, wrap_torch_tensor, FP4)`.
  **CN:** 内部模块：`triton_kernels.roofline`, `triton_kernels.swiglu (swiglu_fn)`, `triton_kernels.matmul (matmul, PrecisionConfig, FlexCtx, FnSpecs, FusedActivation)`, `triton_kernels.matmul_details.opt_flags (scoped_opt_flags_constraints)`, `triton_kernels.target_info (get_cdna_version)`, `triton_kernels.tensor_details (layout)`, `triton_kernels.reduce (reduce)`, `triton_kernels.topk (topk)`, `triton_kernels.tensor (make_ragged_tensor_metadata, remap_ragged_tensor_metadata)`, `triton_kernels.distributed (convert_dp_to_ep, convert_ep_to_dp, make_expt_dict_uniform, make_expt_assignment, SymmetricMemoryPool)`, `triton_kernels.distributed_details.mesh (Mesh)`, `triton_kernels.tensor (convert_layout, wrap_torch_tensor, FP4)`。
