# _p_matmul.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/triton_kernels/matmul_details/_p_matmul.py`
- **Purpose / 用途:** Implementation module for p matmul; it defines the runtime logic, helper types, and GPU-facing utilities used by the Triton kernels package via symbols cuda_capability_geq, get_dtype, _load_writeback_idx_and_mask, round_f32_to_tf32, _p_matmul. / 用于 p matmul 的实现模块；它定义了 Triton kernels 包使用的运行时逻辑、辅助类型以及面向 GPU 的工具；核心符号包括 cuda_capability_geq、get_dtype、_load_writeback_idx_and_mask、round_f32_to_tf32、_p_matmul。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-2 (module)
```python
1| # isort: off
2| # fmt: off
```
**EN:** This preamble contains file-level comments or formatting directives that keep imports and generated code stable.

**CN:** 这一前导部分包含文件级注释或格式化指令，用于保持导入顺序和生成代码稳定。

### Block 2 — Lines 3-32 (module)
```python
 3| import collections
 4| import torch
 5| import triton
 6| import triton.language as tl
 7| from triton.tools.ragged_tma import load_ragged, store_ragged
 8| from triton_kernels import target_info
 9| from triton_kernels.tensor_details.layout_details.blackwell_scale import (
10|     unswizzle_mx_scale_bw,
11|     unswizzle_act_mx_scale_bw,
12| )
13| from triton_kernels.numerics_details.flexpoint import (
14|     float_to_flex,
15|     load_scale,
16|     nan_propagating_absmax_reduce,
17|     compute_scale,
18| )
19| from triton_kernels.numerics_details.mxfp_details._downcast_to_mxfp import MXFP_BLOCK_SIZE, NVFP_BLOCK_SIZE
20| from triton_kernels.numerics_details.mxfp_details._upcast_from_mxfp import upcast_mxfp4_tile
21| from triton_kernels.tensor_details.layout_details.hopper_scale import unswizzle_mxfp4_scale_hopper
22| from triton_kernels.tensor_details.layout_details.hopper_value import mxfp4_to_bf16_triton
23| from ._common import (
24|     compute_offsets,
25|     get_scaled_dot_format_string,
26|     make_matmul_repr,
27|     matmul_launch_metadata,
28|     compute_pids,
29|     output_mx_scale_store_ptr,
30| )
31| 
32| 
```
**EN:** This block imports `collections`, `torch`, `triton`, `triton.language`, `triton.tools.ragged_tma (load_ragged, store_ragged)`, `triton_kernels (target_info)`, `triton_kernels.tensor_details.layout_details.blackwell_scale (unswizzle_mx_scale_bw, unswizzle_act_mx_scale_bw)`, `triton_kernels.numerics_details.flexpoint (float_to_flex, load_scale, nan_propagating_absmax_reduce, compute_scale)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `collections`, `torch`, `triton`, `triton.language`, `triton.tools.ragged_tma (load_ragged, store_ragged)`, `triton_kernels (target_info)`, `triton_kernels.tensor_details.layout_details.blackwell_scale (unswizzle_mx_scale_bw, unswizzle_act_mx_scale_bw)`, `triton_kernels.numerics_details.flexpoint (float_to_flex, load_scale, nan_propagating_absmax_reduce, compute_scale)` ，为模块提供所需的外部库和内部辅助工具。

### Block 3 — Lines 33-34 (cuda_capability_geq)
```python
33| @triton.constexpr_function
34| def cuda_capability_geq(major, minor):
```
**EN:** Defines function `cuda_capability_geq(major, minor)` with decorators `triton.constexpr_function` for this module. The body mainly returns the computed result. It uses calls such as `target_info.cuda_capability_geq` to implement its workflow.

**CN:** 定义函数 `cuda_capability_geq(major, minor)`，带有装饰器 `triton.constexpr_function`，供本模块使用. 主体主要返回计算结果. 其中会调用 `target_info.cuda_capability_geq` 来实现其工作流程.

### Block 4 — Lines 35-36 (cuda_capability_geq)
```python
35|     return target_info.cuda_capability_geq(major, minor)
36| 
```
**EN:** Returns `target_info.cuda_capability_geq(major, minor)`.

**CN:** 返回 `target_info.cuda_capability_geq(major, minor)`.

### Block 5 — Lines 37-38 (get_dtype)
```python
37| @triton.constexpr_function
38| def get_dtype(tensor_or_desc: tl.tensor | tl.tensor_descriptor) -> tl.dtype:
```
**EN:** Defines function `get_dtype(tensor_or_desc)` with decorators `triton.constexpr_function` for this module. The body mainly branches on runtime conditions. It uses calls such as `isinstance`, `ValueError`, `type` to implement its workflow.

**CN:** 定义函数 `get_dtype(tensor_or_desc)`，带有装饰器 `triton.constexpr_function`，供本模块使用. 主体主要根据运行时条件分支. 其中会调用 `isinstance`, `ValueError`, `type` 来实现其工作流程.

### Block 6 — Lines 39-45 (get_dtype)
```python
39|     if isinstance(tensor_or_desc, tl.tensor):
40|         return tensor_or_desc.dtype.element_ty
41|     elif isinstance(tensor_or_desc, tl.tensor_descriptor):
42|         return tensor_or_desc.dtype
43|     else:
44|         raise ValueError(f"Invalid type: {type(tensor_or_desc)}")
45| 
```
**EN:** Checks `isinstance(tensor_or_desc, tl.tensor)`. The true branch mainly returns the computed result, while the else branch branches on runtime conditions.

**CN:** 检查 `isinstance(tensor_or_desc, tl.tensor)`. 真分支主要返回计算结果；而 else 分支根据运行时条件分支.

### Block 7 — Lines 46-47 (_load_writeback_idx_and_mask)
```python
46| @triton.jit
47| def _load_writeback_idx_and_mask(WriteBackIndx, writeback_size, offs, mask):
```
**EN:** Defines function `_load_writeback_idx_and_mask(WriteBackIndx, writeback_size, offs, mask)` with decorators `triton.jit` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `tl.load` to implement its workflow.

**CN:** 定义函数 `_load_writeback_idx_and_mask(WriteBackIndx, writeback_size, offs, mask)`，带有装饰器 `triton.jit`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `tl.load` 来实现其工作流程.

### Block 8 — Lines 48-48 (_load_writeback_idx_and_mask)
```python
48|     mask = mask & (offs < writeback_size)
```
**EN:** Assigns `mask` and evaluates `mask & (offs < writeback_size)`.

**CN:** 将 `mask` and 计算 `mask & (offs < writeback_size)`.

### Block 9 — Lines 49-49 (_load_writeback_idx_and_mask)
```python
49|     offs = tl.load(WriteBackIndx + offs, mask=mask, other=-1)
```
**EN:** Assigns `offs` and calls `tl.load`.

**CN:** 将 `offs`，并调用 `tl.load`.

### Block 10 — Lines 50-50 (_load_writeback_idx_and_mask)
```python
50|     mask = offs != -1
```
**EN:** Assigns `mask` and evaluates `offs != -1`.

**CN:** 将 `mask` and 计算 `offs != -1`.

### Block 11 — Lines 51-53 (_load_writeback_idx_and_mask)
```python
51|     return (offs, mask)
52| 
53| 
```
**EN:** Returns `(offs, mask)`.

**CN:** 返回 `(offs, mask)`.

### Block 12 — Lines 54-55 (round_f32_to_tf32)
```python
54| @triton.jit
55| def round_f32_to_tf32(x: tl.tensor):
```
**EN:** Defines function `round_f32_to_tf32(x)` with decorators `triton.jit` for this module. The body mainly prepares intermediate values; returns the computed result. It uses calls such as `tl.inline_asm_elementwise`, `cuda_capability_geq` to implement its workflow.

**CN:** 定义函数 `round_f32_to_tf32(x)`，带有装饰器 `triton.jit`，供本模块使用. 主体主要准备中间值; 返回计算结果. 其中会调用 `tl.inline_asm_elementwise`, `cuda_capability_geq` 来实现其工作流程.

### Block 13 — Lines 56-56 (round_f32_to_tf32)
```python
56|     ASM: tl.constexpr = "cvt.rn.tf32.f32 $0, $1;" if cuda_capability_geq(9, 0) else "cvt.rna.tf32.f32 $0, $1;"
```
**EN:** Annotated assignment stores `ASM` and uses conditional expression `'cvt.rn.tf32.f32 $0, $1;' if cuda_capability_geq(9, 0) el...`.

**CN:** 带类型注解的赋值保存 `ASM` and 使用条件表达式 `'cvt.rn.tf32.f32 $0, $1;' if cuda_capability_geq(9, 0) el...`.

### Block 14 — Lines 57-58 (round_f32_to_tf32)
```python
57|     return tl.inline_asm_elementwise(ASM, "=r, r", [x], dtype=tl.float32, is_pure=True, pack=1)
58| 
```
**EN:** Returns `tl.inline_asm_elementwise(ASM, '=r, r', [x], dtype=tl.float32, is_pure=True, ...`.

**CN:** 返回 `tl.inline_asm_elementwise(ASM, '=r, r', [x], dtype=tl.float32, is_pure=True, ...`.

### Block 15 — Lines 59-59 (module)
```python
59| _matmul_repr = make_matmul_repr("_p_matmul", [0, 1, 2])
```
**EN:** Assigns `_matmul_repr` and calls `make_matmul_repr`.

**CN:** 将 `_matmul_repr`，并调用 `make_matmul_repr`.

### Block 16 — Lines 60-131 (_p_matmul)
```python
 60| @triton.jit(do_not_specialize=["TOKENS_PER_EXPT_FOR_ANNOTATION"],
 61|             repr=_matmul_repr, launch_metadata=matmul_launch_metadata)
 62| def _p_matmul(
 63|              Y, YPtr, stride_y_k, stride_y_z, stride_y_m, stride_y_n,
 64|              YExpectedScale, YActualScale, YChecksumScale,
 65|              stride_y_mx_k, stride_y_mx_z, stride_y_mx_m, stride_y_mx_n,
 66|              X, XPtr, stride_x_z, stride_x_m, stride_x_k, X_TRANSPOSE: tl.constexpr,
 67|              XScale,
 68|              XMxScale, stride_x_mx_z, stride_x_mx_m, stride_x_mx_k,
 69|              W, WPtr, stride_w_e, stride_w_k, stride_w_n, W_TRANSPOSE: tl.constexpr,
 70|              WScale,
 71|              WMxScale, stride_w_mx_e, stride_w_mx_k, stride_w_mx_n,
 72|              OutAcc, stride_acc_z, stride_acc_m, stride_acc_n,
 73|              OutAccScale, Y_ACC_IS_Y: tl.constexpr,
 74|              B, stride_b_e, # Bias
 75|              M, N, K, K_W, # shapes
 76|              # expt data
 77|              Betas, Gammas,
 78|              GatherIndx,
 79|              WriteBackIndx, writeback_size,
 80|              RAGGED_DIMENSION: tl.constexpr,
 81|              XSliceSizes, XSliceOffs, XBlockOffs, XBlockSchedule, X_EXPECTED_SLICE_SIZE: tl.constexpr, X_SLICE_SIZES_DIVISIBILITY: tl.constexpr,
 82|              XOutputScaleBlockOffs,
 83|              WSliceSizes, WSliceOffs, WBlockOffs, WBlockSchedule, W_EXPECTED_SLICE_SIZE: tl.constexpr, W_SLICE_SIZES_DIVISIBILITY: tl.constexpr,
 84|              # true grid size
 85|              batch_size, grid_m, grid_n,
 86|              # Out scale
 87|              out_alpha,
 88|              # fused activation function
 89|              ACTIVATION_FN: tl.constexpr, activation_fn_args, ACTIVATION_REDUCTION_N: tl.constexpr,
 90|              # epilogue transform
 91|              EPILOGUE_FN: tl.constexpr, epilogue_fn_args,
 92|              # MoE config
 93|              N_SLICES: tl.constexpr,
 94|              # precision config
 95|              MAX_NUM_IMPRECISE_ACC: tl.constexpr, ALLOW_TF32: tl.constexpr,
 96|              FLEXPOINT_SATURATE_INF: tl.constexpr,
 97|              PER_BATCH_W_SCALE: tl.constexpr,
 98|              PER_BATCH_OUT_SCALE: tl.constexpr,
 99|              PER_BATCH_ACC_SCALE: tl.constexpr,
100|              # optimization config
101|              BLOCK_M: tl.constexpr, BLOCK_N: tl.constexpr, BLOCK_K: tl.constexpr,
102|              GROUP_M: tl.constexpr, XCD_SWIZZLE: tl.constexpr,
103|              # NYI: Must be None
104|              SWIZZLE_MX_VALUE: tl.constexpr,
105|              # One of ["BLACKWELL", None]
106|              SWIZZLE_MX_SCALE: tl.constexpr,
107|              MX_BLOCK_SIZE: tl.constexpr,
108|              EPILOGUE_SUBTILE: tl.constexpr,
109|              EVEN_K: tl.constexpr, SPLIT_K: tl.constexpr,
110|              W_CACHE_MODIFIER: tl.constexpr,
111|              NUM_SMS: tl.constexpr,
112|              X_TMA_MODE: tl.constexpr,
113|              Y_TMA_MODE: tl.constexpr,
114|              Y_MX_SCALE_LAYOUT: tl.constexpr = None,
115|              OUT_N_TILE_ALIGNED: tl.constexpr = False,
116|              TOKENS_PER_EXPT_FOR_ANNOTATION=None,
117|              UPCAST_INDICES: tl.constexpr=False,
118|              SWAP_XW: tl.constexpr = False,
119|              IS_EPILOGUE_QUANT_MX: tl.constexpr = False,
120|              Y_VALUE_PACK_FACTOR: tl.constexpr = 1,
121|              FLATTEN_LOOPS: tl.constexpr = True,
122|              W_SHUFFLED: tl.constexpr = False,
123|              pYPtrs=None,
124|              map_dst_coord=None,
125|              all_writes_issued=None,
126|              reduce_rank=0,
127|              n_reduce_shards: tl.constexpr = 1,
128|              ):
129|     # tl.static_assert(SWIZZLE_MX_VALUE is None, "NYI. Value swizzling")
130| 
131|     # why is this faster than using host-side tensor descriptor?!
```
**EN:** Defines function `_p_matmul(Y, YPtr, stride_y_k, stride_y_z, stride_y_m, stride_y_n, YExpectedScale, YActualScale, YChecksumScale, stride_y_mx_k, stride_y_mx_z, stride_y_mx_m, stride_y_mx_n, X, XPtr, stride_x_z, stride_x_m, stride_x_k, X_TRANSPOSE, XScale, XMxScale, stride_x_mx_z, stride_x_mx_m, stride_x_mx_k, W, WPtr, stride_w_e, stride_w_k, stride_w_n, W_TRANSPOSE, WScale, WMxScale, stride_w_mx_e, stride_w_mx_k, stride_w_mx_n, OutAcc, stride_acc_z, stride_acc_m, stride_acc_n, OutAccScale, Y_ACC_IS_Y, B, stride_b_e, M, N, K, K_W, Betas, Gammas, GatherIndx, WriteBackIndx, writeback_size, RAGGED_DIMENSION, XSliceSizes, XSliceOffs, XBlockOffs, XBlockSchedule, X_EXPECTED_SLICE_SIZE, X_SLICE_SIZES_DIVISIBILITY, XOutputScaleBlockOffs, WSliceSizes, WSliceOffs, WBlockOffs, WBlockSchedule, W_EXPECTED_SLICE_SIZE, W_SLICE_SIZES_DIVISIBILITY, batch_size, grid_m, grid_n, out_alpha, ACTIVATION_FN, activation_fn_args, ACTIVATION_REDUCTION_N, EPILOGUE_FN, epilogue_fn_args, N_SLICES, MAX_NUM_IMPRECISE_ACC, ALLOW_TF32, FLEXPOINT_SATURATE_INF, PER_BATCH_W_SCALE, PER_BATCH_OUT_SCALE, PER_BATCH_ACC_SCALE, BLOCK_M, BLOCK_N, BLOCK_K, GROUP_M, XCD_SWIZZLE, SWIZZLE_MX_VALUE, SWIZZLE_MX_SCALE, MX_BLOCK_SIZE, EPILOGUE_SUBTILE, EVEN_K, SPLIT_K, W_CACHE_MODIFIER, NUM_SMS, X_TMA_MODE, Y_TMA_MODE, Y_MX_SCALE_LAYOUT, OUT_N_TILE_ALIGNED, TOKENS_PER_EXPT_FOR_ANNOTATION, UPCAST_INDICES, SWAP_XW, IS_EPILOGUE_QUANT_MX, Y_VALUE_PACK_FACTOR, FLATTEN_LOOPS, W_SHUFFLED, pYPtrs, map_dst_coord, all_writes_issued, reduce_rank, n_reduce_shards)` with decorators `triton.jit(do_not_specialize=['TOKENS...` for this module. The body mainly branches on runtime conditions; prepares intermediate values; prepares intermediate values. It uses calls such as `triton.jit`, `get_dtype`, `tl.static_assert`, `cuda_capability_geq`, `tl.range` to implement its workflow.

**CN:** 定义函数 `_p_matmul(Y, YPtr, stride_y_k, stride_y_z, stride_y_m, stride_y_n, YExpectedScale, YActualScale, Y检查umScale, stride_y_mx_k, stride_y_mx_z, stride_y_mx_m, stride_y_mx_n, X, XPtr, stride_x_z, stride_x_m, stride_x_k, X_TRANSPOSE, XScale, XMxScale, stride_x_mx_z, stride_x_mx_m, stride_x_mx_k, W, WPtr, stride_w_e, stride_w_k, stride_w_n, W_TRANSPOSE, WScale, WMxScale, stride_w_mx_e, stride_w_mx_k, stride_w_mx_n, OutAcc, stride_acc_z, stride_acc_m, stride_acc_n, OutAccScale, Y_ACC_IS_Y, B, stride_b_e, M, N, K, K_W, Betas, Gammas, GatherIndx, WriteBackIndx, writeback_size, RAGGED_DIMENSION, XSliceSizes, XSliceOffs, XBlockOffs, XBlockSchedule, X_EXPECTED_SLICE_SIZE, X_SLICE_SIZES_DIVISIBILITY, XOutputScaleBlockOffs, WSliceSizes, WSliceOffs, WBlockOffs, WBlockSchedule, W_EXPECTED_SLICE_SIZE, W_SLICE_SIZES_DIVISIBILITY, batch_size, grid_m, grid_n, out_alpha, ACTIVATION_FN, activation_fn_args, ACTIVATION_REDUCTION_N, EPILOGUE_FN, epilogue_fn_args, N_SLICES, MAX_NUM_IMPRECISE_ACC, ALLOW_TF32, FLEXPOINT_SATURATE_INF, PER_BATCH_W_SCALE, PER_BATCH_OUT_SCALE, PER_BATCH_ACC_SCALE, BLOCK_M, BLOCK_N, BLOCK_K, GROUP_M, XCD_SWIZZLE, SWIZZLE_MX_VALUE, SWIZZLE_MX_SCALE, MX_BLOCK_SIZE, EPILOGUE_SUBTILE, EVEN_K, SPLIT_K, W_CACHE_MODIFIER, NUM_SMS, X_TMA_MODE, Y_TMA_MODE, Y_MX_SCALE_LAYOUT, OUT_N_TILE_ALIGNED, TOKENS_PER_EXPT_FOR_ANNOTATION, UPCAST_INDICES, SWAP_XW, IS_EPILOGUE_QUANT_MX, Y_VALUE_PACK_FACTOR, FLATTEN_LOOPS, W_SHUFFLED, pYPtrs, map_dst_coord, all_writes_issued, reduce_rank, n_reduce_shards)`，带有装饰器 `triton.jit(do_not_specialize=['TOKENS...`，供本模块使用. 主体主要根据运行时条件分支; 准备中间值; 准备中间值. 其中会调用 `triton.jit`, `get_dtype`, `tl.static_assert`, `cuda_capability_geq`, `tl.range` 来实现其工作流程.

### Block 17 — Lines 132-134 (_p_matmul)
```python
132|     if Y_TMA_MODE is not None:
133|         Y = tl.make_tensor_descriptor(YPtr, Y.shape, Y.strides[:-1] + (1,), Y.block_shape)
134| 
```
**EN:** Checks `Y_TMA_MODE is not None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `Y_TMA_MODE is not None`. 真分支主要准备中间值.

### Block 18 — Lines 135-135 (_p_matmul)
```python
135|     w_type: tl.constexpr = get_dtype(W)
```
**EN:** Annotated assignment stores `w_type` and calls `get_dtype`.

**CN:** 带类型注解的赋值保存 `w_type`，并调用 `get_dtype`.

### Block 19 — Lines 136-136 (_p_matmul)
```python
136|     is_w_microscaled: tl.constexpr = WMxScale is not None
```
**EN:** Annotated assignment stores `is_w_microscaled` and evaluates `WMxScale is not None`.

**CN:** 带类型注解的赋值保存 `is_w_microscaled` and 计算 `WMxScale is not None`.

### Block 20 — Lines 137-137 (_p_matmul)
```python
137|     is_x_microscaled: tl.constexpr = XMxScale is not None
```
**EN:** Annotated assignment stores `is_x_microscaled` and evaluates `XMxScale is not None`.

**CN:** 带类型注解的赋值保存 `is_x_microscaled` and 计算 `XMxScale is not None`.

### Block 21 — Lines 138-138 (_p_matmul)
```python
138|     is_w_mxfp4: tl.constexpr = w_type == tl.uint8 and is_w_microscaled
```
**EN:** Annotated assignment stores `is_w_mxfp4` and evaluates `w_type == tl.uint8 and is_w_microscaled`.

**CN:** 带类型注解的赋值保存 `is_w_mxfp4` and 计算 `w_type == tl.uint8 and is_w_microscaled`.

### Block 22 — Lines 139-139 (_p_matmul)
```python
139|     tl.static_assert(not is_w_mxfp4 or (W_TRANSPOSE or W_SHUFFLED), "NYI. Non-transposed mxfp4 weights")
```
**EN:** Calls `tl.static_assert` for side effects, registration, or validation.

**CN:** 调用 `tl.static_assert` ，用于副作用、注册或校验。

### Block 23 — Lines 140-140 (_p_matmul)
```python
140|     MX_PACK_DIVISOR: tl.constexpr = MX_BLOCK_SIZE
```
**EN:** Annotated assignment stores `MX_PACK_DIVISOR` and references `MX_BLOCK_SIZE`.

**CN:** 带类型注解的赋值保存 `MX_PACK_DIVISOR` and 引用 `MX_BLOCK_SIZE`.

### Block 24 — Lines 141-142 (_p_matmul)
```python
141|     if is_x_microscaled or is_w_microscaled:
142|         MX_SCALE_BLOCK_K: tl.constexpr = BLOCK_K // MX_PACK_DIVISOR
```
**EN:** Checks `is_x_microscaled or is_w_microscaled`. The true branch mainly prepares intermediate values.

**CN:** 检查 `is_x_microscaled or is_w_microscaled`. 真分支主要准备中间值.

### Block 25 — Lines 143-186 (_p_matmul)
```python
143|     if is_w_microscaled:
144|         tl.static_assert(MX_BLOCK_SIZE == NVFP_BLOCK_SIZE or MX_BLOCK_SIZE == MXFP_BLOCK_SIZE,
145|                          "Unsupported microscale factor")
146|         tl.static_assert(w_type == tl.uint8 or (w_type == tl.float8e4nv or w_type == tl.float8e5),
147|                          "mx_weight_ptr must be uint8 or fp8")
148|         # NOTE: uint8 scale means OCP E8M0 here. Direct NVFP-style scales stay float8e4nv.
149|         tl.static_assert(
150|             get_dtype(WMxScale) == tl.uint8
151|             or get_dtype(WMxScale) == tl.float8e4nv,
152|             "mx_scale_ptr must be uint8 or float8e4nv",
153|         )
154|         tl.static_assert(BLOCK_K % MX_PACK_DIVISOR == 0, "BLOCK_K must be a multiple of MX_PACK_DIVISOR")
155| 
156|         # We have pack 2 fp4 values in a byte
157|         if SWIZZLE_MX_VALUE == "HOPPER_VALUE":
158|             tl.static_assert(is_w_mxfp4, "Only mxfp4 is supported for HOPPER swizzling")
159|             # We have pack 2 fp4 values in a byte but we divide the dimension by 2
160|             # when swizzling
161|             W_K_DIVISOR: tl.constexpr = 1
162|             W_K_MULTIPLIER: tl.constexpr = 2
163|             W_N_DIVISOR: tl.constexpr = 4
164|         else:
165|             # We have pack 2 fp4 values in a byte
166|             W_K_DIVISOR: tl.constexpr = 2 if is_w_mxfp4 else 1
167|             W_K_MULTIPLIER: tl.constexpr = 1
168|             W_N_DIVISOR: tl.constexpr = 1
169| 
170|         if W_TRANSPOSE:
171|             # When weight is transposed, 2 fp4 values are packed per Byte along
172|             # the contiguous dimension, K.
173|             PACKED_BLOCK_K_W: tl.constexpr = (BLOCK_K // W_K_DIVISOR) * W_K_MULTIPLIER
174|             PACKED_BLOCK_N_W: tl.constexpr = BLOCK_N // W_N_DIVISOR
175|         else:
176|             # When weight is not transposed, fp4 values are *not* packed along
177|             # the contiguous dimension, N.
178|             PACKED_BLOCK_K_W: tl.constexpr = BLOCK_K
179|             PACKED_BLOCK_N_W: tl.constexpr = BLOCK_N // W_K_DIVISOR
180|     else:
181|         W_K_DIVISOR: tl.constexpr = 1
182|         W_K_MULTIPLIER: tl.constexpr = 1
183|         W_N_DIVISOR: tl.constexpr = 1
184|         PACKED_BLOCK_K_W: tl.constexpr = BLOCK_K
185|         PACKED_BLOCK_N_W: tl.constexpr = BLOCK_N
186|         tl.static_assert(SWIZZLE_MX_SCALE == "STRIDED")
```
**EN:** Checks `is_w_microscaled`. The true branch mainly invokes `tl.static_assert`; invokes `tl.static_assert`, while the else branch prepares intermediate values; prepares intermediate values.

**CN:** 检查 `is_w_microscaled`. 真分支主要invokes `tl.static_assert`; invokes `tl.static_assert`；而 else 分支准备中间值; 准备中间值.

### Block 26 — Lines 187-199 (_p_matmul)
```python
187|     if is_x_microscaled:
188|         x_type: tl.constexpr = get_dtype(X)
189|         is_x_fp4: tl.constexpr = x_type == tl.uint8
190|         tl.static_assert(x_type == tl.float8e4nv or x_type == tl.uint8, "mx_act_ptr must be float8e4nv or uint8")
191|         # NOTE: uint8 scale means OCP E8M0 here. Direct NVFP-style scales stay float8e4nv.
192|         tl.static_assert(
193|             get_dtype(XMxScale) == tl.uint8
194|             or get_dtype(XMxScale) == tl.float8e4nv,
195|             "mx_scale_ptr must be uint8 or float8e4nv",
196|         )
197|         tl.static_assert(BLOCK_K % MX_PACK_DIVISOR == 0, "BLOCK_K must be a multiple of MX_PACK_DIVISOR")
198|     else:
199|         is_x_fp4: tl.constexpr = False
```
**EN:** Checks `is_x_microscaled`. The true branch mainly prepares intermediate values; prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `is_x_microscaled`. 真分支主要准备中间值; 准备中间值；而 else 分支准备中间值.

### Block 27 — Lines 200-200 (_p_matmul)
```python
200|     is_out_microscaled: tl.constexpr = stride_y_mx_z is not None
```
**EN:** Annotated assignment stores `is_out_microscaled` and evaluates `stride_y_mx_z is not None`.

**CN:** 带类型注解的赋值保存 `is_out_microscaled` and 计算 `stride_y_mx_z is not None`.

### Block 28 — Lines 201-202 (_p_matmul)
```python
201|     is_out_fp4: tl.constexpr = is_out_microscaled and Y_VALUE_PACK_FACTOR == 2
202| 
```
**EN:** Annotated assignment stores `is_out_fp4` and evaluates `is_out_microscaled and Y_VALUE_PACK_FACTOR == 2`.

**CN:** 带类型注解的赋值保存 `is_out_fp4` and 计算 `is_out_microscaled and Y_VALUE_PACK_FACTOR == 2`.

### Block 29 — Lines 203-207 (_p_matmul)
```python
203|     if RAGGED_DIMENSION == "M":
204|         useful_grid_m = tl.load(XBlockOffs + N_SLICES)
205|     else:
206|         useful_grid_m = grid_m
207| 
```
**EN:** Checks `RAGGED_DIMENSION == 'M'`. The true branch mainly prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `RAGGED_DIMENSION == 'M'`. 真分支主要准备中间值；而 else 分支准备中间值.

### Block 30 — Lines 208-209 (_p_matmul)
```python
208|     index_type: tl.constexpr = tl.int64
209| 
```
**EN:** Annotated assignment stores `index_type` and references `tl.int64`.

**CN:** 带类型注解的赋值保存 `index_type` and 引用 `tl.int64`.

### Block 31 — Lines 210-210 (_p_matmul)
```python
210|     USE_FLEXPOINT_SCALE: tl.constexpr = YActualScale is not None or YChecksumScale is not None
```
**EN:** Annotated assignment stores `USE_FLEXPOINT_SCALE` and evaluates `YActualScale is not None or YChecksumScale is not None`.

**CN:** 带类型注解的赋值保存 `USE_FLEXPOINT_SCALE` and 计算 `YActualScale is not None or Y检查umScale is not None`.

### Block 32 — Lines 211-211 (_p_matmul)
```python
211|     HAS_SCATTER: tl.constexpr = WriteBackIndx is not None
```
**EN:** Annotated assignment stores `HAS_SCATTER` and evaluates `WriteBackIndx is not None`.

**CN:** 带类型注解的赋值保存 `HAS_SCATTER` and 计算 `WriteBackIndx is not None`.

### Block 33 — Lines 212-212 (_p_matmul)
```python
212|     HAS_GATHER: tl.constexpr = GatherIndx is not None
```
**EN:** Annotated assignment stores `HAS_GATHER` and evaluates `GatherIndx is not None`.

**CN:** 带类型注解的赋值保存 `HAS_GATHER` and 计算 `GatherIndx is not None`.

### Block 34 — Lines 213-213 (_p_matmul)
```python
213|     USE_GATHER_TMA: tl.constexpr = HAS_GATHER and X_TMA_MODE == "dense"
```
**EN:** Annotated assignment stores `USE_GATHER_TMA` and evaluates `HAS_GATHER and X_TMA_MODE == 'dense'`.

**CN:** 带类型注解的赋值保存 `USE_GATHER_TMA` and 计算 `HAS_GATHER and X_TMA_MODE == 'dense'`.

### Block 35 — Lines 214-215 (_p_matmul)
```python
214|     USE_SCATTER_TMA: tl.constexpr = HAS_SCATTER and Y_TMA_MODE == "dense"
215| 
```
**EN:** Annotated assignment stores `USE_SCATTER_TMA` and evaluates `HAS_SCATTER and Y_TMA_MODE == 'dense'`.

**CN:** 带类型注解的赋值保存 `USE_SCATTER_TMA` and 计算 `HAS_SCATTER and Y_TMA_MODE == 'dense'`.

### Block 36 — Lines 216-219 (_p_matmul)
```python
216|     if RAGGED_DIMENSION == "K":
217|         tl.static_assert((OutAcc is None) or Y_ACC_IS_Y, "Using differernt y_acc is not supported with TMA kernel.")
218|         tl.static_assert(not (HAS_SCATTER or USE_GATHER_TMA or USE_SCATTER_TMA), "Cannot be used with RAGGED_DIMENSION == 'K'")
219| 
```
**EN:** Checks `RAGGED_DIMENSION == 'K'`. The true branch mainly invokes `tl.static_assert`; invokes `tl.static_assert`.

**CN:** 检查 `RAGGED_DIMENSION == 'K'`. 真分支主要invokes `tl.static_assert`; invokes `tl.static_assert`.

### Block 37 — Lines 220-223 (_p_matmul)
```python
220|     if EPILOGUE_SUBTILE is None:
221|         SUBTILE_FACTOR: tl.constexpr = 1
222|     else:
223|         SUBTILE_FACTOR: tl.constexpr = EPILOGUE_SUBTILE
```
**EN:** Checks `EPILOGUE_SUBTILE is None`. The true branch mainly prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `EPILOGUE_SUBTILE is None`. 真分支主要准备中间值；而 else 分支准备中间值.

### Block 38 — Lines 224-224 (_p_matmul)
```python
224|     EPILOGUE_BLOCK_N: tl.constexpr = BLOCK_N // SUBTILE_FACTOR
```
**EN:** Annotated assignment stores `EPILOGUE_BLOCK_N` and evaluates `BLOCK_N // SUBTILE_FACTOR`.

**CN:** 带类型注解的赋值保存 `EPILOGUE_BLOCK_N` and 计算 `BLOCK_N // SUBTILE_FACTOR`.

### Block 39 — Lines 225-225 (_p_matmul)
```python
225|     OUT_BLOCK_N: tl.constexpr = EPILOGUE_BLOCK_N // ACTIVATION_REDUCTION_N
```
**EN:** Annotated assignment stores `OUT_BLOCK_N` and evaluates `EPILOGUE_BLOCK_N // ACTIVATION_REDUCTION_N`.

**CN:** 带类型注解的赋值保存 `OUT_BLOCK_N` and 计算 `EPILOGUE_BLOCK_N // ACTIVATION_REDUCTION_N`.

### Block 40 — Lines 226-227 (_p_matmul)
```python
226|     yN = N // ACTIVATION_REDUCTION_N
227| 
```
**EN:** Assigns `yN` and evaluates `N // ACTIVATION_REDUCTION_N`.

**CN:** 将 `yN` and 计算 `N // ACTIVATION_REDUCTION_N`.

### Block 41 — Lines 228-231 (_p_matmul)
```python
228|     num_blocks = batch_size * useful_grid_m * grid_n * SPLIT_K
229| 
230|     # If true, do not share loop-carried variables between the prologue and the
231|     # epilogue to enable better pipelining with mmav5
```
**EN:** Assigns `num_blocks` and evaluates `batch_size * useful_grid_m * grid_n * SPLIT_K`.

**CN:** 将 `num_blocks` and 计算 `batch_size * useful_grid_m * grid_n * SPLIT_K`.

### Block 42 — Lines 232-234 (_p_matmul)
```python
232|     INDEPENDENT_EPILOGUE: tl.constexpr = cuda_capability_geq(10, 0)
233| 
234|     # start negative; will be incremented at the top of the loop
```
**EN:** Annotated assignment stores `INDEPENDENT_EPILOGUE` and calls `cuda_capability_geq`.

**CN:** 带类型注解的赋值保存 `INDEPENDENT_EPILOGUE`，并调用 `cuda_capability_geq`.

### Block 43 — Lines 235-238 (_p_matmul)
```python
235|     if INDEPENDENT_EPILOGUE:
236|         tile_id1 = tl.program_id(0) - NUM_SMS
237| 
238|     # Keep track of local max for updating flexpoint scales.
```
**EN:** Checks `INDEPENDENT_EPILOGUE`. The true branch mainly prepares intermediate values.

**CN:** 检查 `INDEPENDENT_EPILOGUE`. 真分支主要准备中间值.

### Block 44 — Lines 239-239 (_p_matmul)
```python
239|     USE_LOCAL_ABSMAX: tl.constexpr = (YActualScale is not None) and (not PER_BATCH_OUT_SCALE) and (not is_out_microscaled) and (pYPtrs is None)
```
**EN:** Annotated assignment stores `USE_LOCAL_ABSMAX` and evaluates `YActualScale is not None and (not PER_BATCH_OUT_SCALE) and (not is_...`.

**CN:** 带类型注解的赋值保存 `USE_LOCAL_ABSMAX` and 计算 `YActualScale is not None and (not PER_BATCH_OUT_SCALE) and (not is_...`.

### Block 45 — Lines 240-243 (_p_matmul)
```python
240|     if USE_LOCAL_ABSMAX:
241|         THREADS_PER_BLOCK: tl.constexpr = tl.extra.cuda.num_threads()
242|         local_absmax = tl.full([THREADS_PER_BLOCK], 0.0, tl.uint32)
243| 
```
**EN:** Checks `USE_LOCAL_ABSMAX`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `USE_LOCAL_ABSMAX`. 真分支主要准备中间值; 准备中间值.

### Block 46 — Lines 244-245 (_p_matmul)
```python
244|     DISALLOW_ACC_MULTI_BUFFER: tl.constexpr = is_w_microscaled and BLOCK_M * BLOCK_N >= 128 * 256
245| 
```
**EN:** Annotated assignment stores `DISALLOW_ACC_MULTI_BUFFER` and evaluates `is_w_microscaled and BLOCK_M * BLOCK_N >= 128 * 256`.

**CN:** 带类型注解的赋值保存 `DISALLOW_ACC_MULTI_BUFFER` and 计算 `is_w_microscaled and BLOCK_M * BLOCK_N >= 128 * 256`.

### Block 47 — Lines 246-741 (_p_matmul)
```python
246|     for block_id in tl.range(
247|         tl.program_id(0), num_blocks, NUM_SMS,
248|         flatten=FLATTEN_LOOPS,
249|         disallow_acc_multi_buffer=DISALLOW_ACC_MULTI_BUFFER,
250|         # Workaround for compile error in hopper warp specialization
251|         warp_specialize=FLATTEN_LOOPS,
252|     ):
253| 
254|         pid_z, pid_m, pid_n, pid_k = compute_pids(block_id, useful_grid_m, grid_n, num_blocks, XCD_SWIZZLE, GROUP_M, SPLIT_K)
255| 
256|         # ------------------------------------------------------------
257|         # prologue
258|         # ------------------------------------------------------------
259|         off_w_z, off_x_z, off_y_z, slice_off_m, slice_block_off_m, off_m, off_k_x0, off_k_w0 = compute_offsets(
260|             pid_z, pid_m, pid_k,
261|             XBlockSchedule, XSliceOffs, XBlockOffs, X_SLICE_SIZES_DIVISIBILITY,
262|             WBlockSchedule, WSliceOffs, W_SLICE_SIZES_DIVISIBILITY,
263|             RAGGED_DIMENSION,
264|             BLOCK_M, BLOCK_K, PACKED_BLOCK_K_W, SPLIT_K
265|         )
266| 
267|         # TODO: if RAGGED_DIMENSION == "M"
268|         if RAGGED_DIMENSION == "M":
269|             shape_m = tl.load(XSliceSizes + off_w_z)
270|         else:
271|             shape_m = M
272|         off_n = BLOCK_N * pid_n
273|         off_w_n = PACKED_BLOCK_N_W * pid_n
274|         block_div: tl.constexpr = 2 if is_x_fp4 else 1
275| 
276|         # ---- offset x ------
277|         if USE_GATHER_TMA:
278|             offs_m = off_m + tl.arange(0, BLOCK_M)
279|             mask_m = offs_m < shape_m
280|             if XBlockSchedule is None:
281|                 offs_x_m = tl.load(GatherIndx + slice_off_m.to(index_type) + offs_m, mask=mask_m)
282|                 # Bump rows to account for the Z offset.
283|                 offs_x_m += off_x_z * (stride_x_z // stride_x_m)
284|                 offs_x_m = tl.where(mask_m, offs_x_m, -1)
285|             else:
286|                 offs_x_m = tl.load(GatherIndx + slice_off_m.to(index_type) + offs_m, mask=mask_m, other=-1)
287|         if X_TMA_MODE is None:
288|             XBase = X + off_x_z.to(index_type) * stride_x_z
289|             offs_m = off_m + tl.arange(0, BLOCK_M)
290|             offs_m = tl.max_contiguous(tl.multiple_of(offs_m % shape_m, BLOCK_M), BLOCK_M)
291|             # no needs to bounds-check here because `offs_m` wraps around M dim
292|             if GatherIndx is not None:
293|                 tl.static_assert(HAS_GATHER)
294|                 offs_m = tl.load(GatherIndx + slice_off_m.to(index_type) + offs_m)
295|             offs_x_m = offs_m.to(index_type)[:, None] * stride_x_m
296|             offs_x_k = (off_k_x0.to(index_type) // block_div + tl.arange(0, BLOCK_K // block_div))[None, :] * stride_x_k
297| 
298|         XMxScalePtrs = None
299|         if is_x_microscaled and stride_x_mx_z is not None: # x is mx but not using TMA
300|             offs_m = off_m + tl.arange(0, BLOCK_M)
301|             XMxScalePtrs = XMxScale + off_x_z.to(index_type) * stride_x_mx_z
302|             if GatherIndx is None:
303|                 XMxScalePtrs += slice_off_m * stride_x_mx_m
304|             offs_k_scale = off_k_x0 // MX_BLOCK_SIZE + tl.arange(0, MX_SCALE_BLOCK_K)
305|             XMxScalePtrs += (offs_x_m if USE_GATHER_TMA else offs_m).to(index_type)[:, None] * stride_x_mx_m
306|             XMxScalePtrs += offs_k_scale.to(index_type)[None, :] * stride_x_mx_k
307| 
308|         acc = tl.zeros((BLOCK_N, BLOCK_M) if SWAP_XW else (BLOCK_M, BLOCK_N), dtype=tl.float32)
309| 
310|         # ------------------------------------------------------------
311|         # inner loop
312|         # ------------------------------------------------------------
313|         loop_k = tl.load(XSliceSizes + pid_z) if RAGGED_DIMENSION == "K" else K - off_k_x0
314|         k_tiles = tl.cdiv(loop_k, BLOCK_K * SPLIT_K)
315|         loop_bound = tl.maximum(k_tiles, 1)
316|         tl.assume(loop_bound > 0)  # Currently necessary for the compiler to flatten the loop properly.
317|         for ki in tl.range(loop_bound, disallow_acc_multi_buffer=DISALLOW_ACC_MULTI_BUFFER):
318|             if RAGGED_DIMENSION == "K" and ki >= k_tiles:
319|                 # Tile #ki does not exist: use out-of-bound indices to mask all loads.
320|                 off_k_x = K
321|                 off_k_w = K_W
322|             else:
323|                 off_k_x = off_k_x0 + ki * BLOCK_K * SPLIT_K
324|                 off_k_w = off_k_w0 + ki * PACKED_BLOCK_K_W * SPLIT_K
325| 
326|             # --- load x ---
327|             if USE_GATHER_TMA:
328|                 x = X.gather(offs_x_m, off_k_x // block_div)
329|             elif X_TMA_MODE == "dense":
330|                 if X_TRANSPOSE:
331|                     x = X.load([off_x_z, off_k_x // block_div, slice_off_m + off_m])
332|                     x = x.reshape(BLOCK_K // block_div, BLOCK_M).T
333|                 else:
334|                     x = X.load([off_x_z, slice_off_m + off_m, off_k_x // block_div])
335|                     x = x.reshape(BLOCK_M, BLOCK_K // block_div)
336|             elif X_TMA_MODE == "ragged":
337|                 x = load_ragged(X, slice_off_m, shape_m, [off_x_z, off_m, off_k_x // block_div], ragged_dim=1)
338|                 x = x.reshape(BLOCK_M, BLOCK_K // block_div)
339|             else:
340|                 tl.static_assert(X_TMA_MODE is None)
341|                 XPtrs = XBase + offs_x_m + offs_x_k
342|                 XBase += (BLOCK_K // block_div) * SPLIT_K * stride_x_k
343|                 mask_k = tl.arange(0, BLOCK_K // block_div) * block_div < K - off_k_x
344|                 if EVEN_K:
345|                     if SPLIT_K > 1:
346|                         x = tl.load(XPtrs, mask=mask_k[None, :], other=0.0)
347|                     else:
348|                         x = tl.load(XPtrs)
349|                 else:
350|                     x = tl.load(XPtrs, mask=mask_k[None, :], other=0.0)
351|                 if x.dtype == tl.float32 and ALLOW_TF32:
352|                     # since data are not loaded from TMA we need to explicitly round to tf32.
353|                     x = round_f32_to_tf32(x)
354|             # --- load x_scale ---
355|             x_format: tl.constexpr = get_scaled_dot_format_string(x.dtype)
356|             if is_x_microscaled:
357|                 if XMxScalePtrs is not None: # not using TMA for x scale load
358|                     # dividing MX_PACK_DIVISOR by W_K_DIVISOR because off_k_w is
359|                     # already divided by W_K_DIVISOR (2 for mxfp4 where 2 fp4
360|                     # values are packed per Byte along K)
361|                     off_k_mx = off_k_w // (MX_PACK_DIVISOR // W_K_DIVISOR)
362|                     if EVEN_K and SPLIT_K == 1:
363|                         mask_k_scale = tl.full([MX_SCALE_BLOCK_K], True, dtype=tl.int1)
364|                     else:
365|                         mask_k_scale = off_k_mx + tl.arange(0, MX_SCALE_BLOCK_K) < tl.cdiv(K, MX_PACK_DIVISOR)
366|                     mask_m = off_m + tl.arange(0, BLOCK_M) < shape_m
367|                     x_scales = tl.load(XMxScalePtrs, mask=mask_k_scale[None, :] & mask_m[:, None], other=0.0)
368|                 else: # use TMA for x scale load - only cover batched case for now
369|                     if X_TMA_MODE == "dense":
370|                         off_m_scale = off_x_z * ((M + 127) // 128) + off_m // 128
371|                     else:
372|                         # slice_block_off_m points to the start of the current slice in the padded version
373|                         # + off_m points to the current block in the slice
374|                         off_m_scale = slice_block_off_m + off_m // 128
375|                     x_scales = XMxScale.load([0, off_m_scale, off_k_x // MX_PACK_DIVISOR // 4, 0, 0])
376|                     x_scales = unswizzle_act_mx_scale_bw(x_scales)
377|             elif x_format == "fp16" or x_format == "bf16":
378|                 x_scales: tl.constexpr = None
379|             else:
380|                 if not is_w_microscaled:
381|                     x_scales: tl.constexpr = None
382|                 elif get_dtype(WMxScale) == tl.uint8:
383|                     x_scales = tl.full((BLOCK_M, BLOCK_K // MX_PACK_DIVISOR), 127, dtype=tl.uint8)
384|                 else:
385|                     x_scales = tl.full((BLOCK_M, BLOCK_K // MX_PACK_DIVISOR), 1.0, dtype=tl.float8e4nv)
386| 
387|             # --- load w ---
388|             if W_SHUFFLED:
389|                 tile_k_idx = off_k_w // PACKED_BLOCK_K_W
390|                 tile_n_idx = off_n // BLOCK_N
391|                 w = tl.reshape(
392|                     W.load([off_w_z.to(tl.int32), tile_k_idx.to(tl.int32), tile_n_idx.to(tl.int32), 0, 0]),
393|                     (BLOCK_N, PACKED_BLOCK_K_W),
394|                 ).T
395|             elif W_TRANSPOSE:
396|                 w = tl.reshape(W.load([off_w_z, off_w_n, off_k_w]), W.block_shape[1:]).T
397|             else:
398|                 w = tl.reshape(W.load([off_w_z, off_k_w, off_w_n]), W.block_shape[1:])
399| 
400|             # --- load w_scale ---
401|             w_format: tl.constexpr = get_scaled_dot_format_string(w.dtype)
402|             if is_w_microscaled:
403|                 off_k_mx = off_k_w // (MX_PACK_DIVISOR // W_K_DIVISOR)
404|                 tl.static_assert(MX_PACK_DIVISOR % W_K_DIVISOR == 0)
405|                 if SWIZZLE_MX_SCALE == "BLACKWELL_SCALE":
406|                     flattened_expt_n_idx = off_w_z * ((N + 127) // 128) + (off_n // 128)
407|                     w_scales = WMxScale.load([0, flattened_expt_n_idx, off_k_mx // 4, 0, 0])
408|                     w_scales = w_scales.reshape((w_scales.shape[1], w_scales.shape[2] * w_scales.shape[-2] * w_scales.shape[-1]))
409|                     w_scales = unswizzle_mx_scale_bw(w_scales)
410|                 elif SWIZZLE_MX_SCALE == "HOPPER_SCALE":
411|                     # NYI: Hopper swizzling with non-transposed W
412|                     tl.static_assert(W_TRANSPOSE)
413|                     off_n_scale = pid_n * (BLOCK_N // 32)
414|                     off_k_scale = (off_k_w // PACKED_BLOCK_K_W) * MX_SCALE_BLOCK_K * 32
415|                     w_scales = WMxScale.load([off_w_z, off_n_scale, off_k_scale])
416|                     w_scales = tl.reshape(w_scales, *w_scales.shape[1:])
417|                     num_warps: tl.constexpr = tl.extra.cuda.num_warps()
418|                     w_scales = unswizzle_mxfp4_scale_hopper(w_scales, mx_axis=1, num_warps=num_warps)
419|                     mask_k_scale = off_k_x + tl.arange(0, MX_SCALE_BLOCK_K) * MX_PACK_DIVISOR < off_k_x0 + loop_k
420|                     scale_zero = tl.full(w_scales.shape, 0, dtype=w_scales.dtype)
421|                     w_scales = tl.where(mask_k_scale[None, :], w_scales, scale_zero)
422|                 else:
423|                     w_scales = WMxScale.load([off_w_z, off_k_mx, off_n])
424|                     w_scales = tl.reshape(w_scales, *w_scales.shape[1:]).T
425|             else:
426|                 w_scales: tl.constexpr = None
427| 
428|             # --- update accumulator ---
429|             if is_x_microscaled or is_w_microscaled:
430|                 if is_x_fp4 and not is_w_microscaled and not cuda_capability_geq(10, 0):
431|                     tl.static_assert(w_format == "fp16" or w_format == "bf16")
432|                     x_dense = upcast_mxfp4_tile(x, x_scales, w.dtype)
433|                     if SWAP_XW:
434|                         acc = tl.dot(w.T, x_dense.T, acc, max_num_imprecise_acc=MAX_NUM_IMPRECISE_ACC, allow_tf32=ALLOW_TF32)
435|                     else:
436|                         acc = tl.dot(x_dense, w, acc, max_num_imprecise_acc=MAX_NUM_IMPRECISE_ACC, allow_tf32=ALLOW_TF32)
437|                 elif SWIZZLE_MX_VALUE == "HOPPER_VALUE":
438|                     tl.static_assert(w_format == "e2m1")
439|                     tl.static_assert(SWAP_XW)
440|                     wT = mxfp4_to_bf16_triton(w.T, w_scales, mx_axis=1)
441|                     tl.static_assert(wT.dtype == tl.bfloat16)
442|                     if is_x_microscaled:
443|                         acc = tl.dot_scaled(wT, None, "bf16", x.T, x_scales, x_format, acc=acc, fast_math=True)
444|                     else:
445|                         tl.static_assert(x_format == "bf16")
446|                         acc = tl.dot(wT, x.T, acc, max_num_imprecise_acc=MAX_NUM_IMPRECISE_ACC, allow_tf32=ALLOW_TF32)
447|                 else:
448|                     if SWAP_XW:
449|                         acc = tl.dot_scaled(w.T, w_scales, w_format, x.T, x_scales, x_format, acc=acc, fast_math=True)
450|                     else:
451|                         acc = tl.dot_scaled(x, x_scales, x_format, w, w_scales, w_format, acc=acc, fast_math=True)
452|             else:
453|                 if SWAP_XW:
454|                     acc = tl.dot(w.T, x.T, acc, max_num_imprecise_acc=MAX_NUM_IMPRECISE_ACC, allow_tf32=ALLOW_TF32)
455|                 else:
456|                     acc = tl.dot(x, w, acc, max_num_imprecise_acc=MAX_NUM_IMPRECISE_ACC, allow_tf32=ALLOW_TF32)
457| 
458|             if is_x_microscaled and XMxScalePtrs is not None:
459|                 XMxScalePtrs += (MX_SCALE_BLOCK_K * SPLIT_K) * stride_x_mx_k
460| 
461|         # ------------------------------------------------------------
462|         # epilogue
463|         # ------------------------------------------------------------
464|         if INDEPENDENT_EPILOGUE:
465|             tile_id1 += NUM_SMS
466|             pid_s1, pid_m1, pid_n1, pid_k1 = compute_pids(tile_id1, useful_grid_m, grid_n, num_blocks, XCD_SWIZZLE, GROUP_M, SPLIT_K)
467|             expt_id1, _, start_z1, start_m1, slice_block_off_m1, off_m1, _, _ = compute_offsets(
468|                 pid_s1, pid_m1, pid_k1,
469|                 XBlockSchedule, XSliceOffs, XBlockOffs, X_SLICE_SIZES_DIVISIBILITY,
470|                 WBlockSchedule, WSliceOffs, W_SLICE_SIZES_DIVISIBILITY,
471|                 RAGGED_DIMENSION,
472|                 BLOCK_M, BLOCK_K, PACKED_BLOCK_K_W, SPLIT_K
473|             )
474|             off_n1 = pid_n1 * BLOCK_N
475|             if RAGGED_DIMENSION == "M":
476|                 eM1 = tl.load(XSliceSizes + expt_id1)
477|             else:
478|                 eM1 = M
479|         else:
480|             tile_id1, expt_id1, start_z1, start_m1, eM1 = block_id, off_w_z, off_y_z, slice_off_m, shape_m
481|             _, off_m1, off_n1, pid_k1 = slice_block_off_m, off_m, off_n, pid_k
482| 
483|         offs_m = off_m1 + tl.arange(0, BLOCK_M)
484|         mask_m = offs_m < eM1
485|         if USE_SCATTER_TMA:
486|             offs_y_m, mask_m = _load_writeback_idx_and_mask(WriteBackIndx, writeback_size, start_m1 + offs_m, mask_m)
487|             MASK_ACC: tl.constexpr = USE_FLEXPOINT_SCALE
488|             if SPLIT_K > 1:
489|                 # Compute the split k offset in number of rows, and add it to offs_y_m.
490|                 # This allows us to write to the correct slice in the output tensor while using
491|                 # a 2D TMA scatter.
492|                 tl.device_assert(stride_y_k // stride_y_m == tl.cdiv(stride_y_k, stride_y_m))
493|                 split_k_row_offs = pid_k1 * (stride_y_k // stride_y_m)
494|                 offs_y_m = tl.where(mask_m, offs_y_m + split_k_row_offs, offs_y_m)
495|         elif Y_TMA_MODE is None and HAS_SCATTER:
496|             offs_y_m, mask_m = _load_writeback_idx_and_mask(WriteBackIndx, writeback_size, start_m1 + offs_m, mask_m)
497|             MASK_ACC: tl.constexpr = USE_FLEXPOINT_SCALE
498|         else:
499|             offs_y_m = start_m1 + offs_m
500|             MASK_ACC = False if USE_GATHER_TMA else USE_FLEXPOINT_SCALE
501| 
502|         # bias + scale
503|         offs_y_n = off_n1 + tl.arange(0, BLOCK_N)
504|         mask_n = offs_y_n < N
505|         if B is not None:
506|             BPtrs = B + expt_id1 * stride_b_e + offs_y_n
507|             if pid_k1 == 0:
508|                 bias = tl.load(BPtrs, mask=mask_n, other=0)
509|             else:
510|                 bias = tl.full([BLOCK_N], 0, dtype=tl.float32)
511|         else:
512|             bias = tl.full([BLOCK_N], 0, dtype=tl.float32)
513|         if Betas is not None:
514|             betas = tl.load(Betas + start_m1 + offs_m, mask=mask_m, other=0.0)
515|         else:
516|             betas = tl.full([BLOCK_M], 1, dtype=tl.float32)
517|         if Gammas is not None:
518|             gammas = tl.load(Gammas + start_m1 + offs_m, mask=mask_m, other=0.0)
519|         else:
520|             gammas = tl.full([BLOCK_M], 1, dtype=tl.float32)
521|         x_scale = load_scale(XScale)
522|         if PER_BATCH_W_SCALE:
523|             w_scale = load_scale(WScale + expt_id1)
524|         else:
525|             w_scale = load_scale(WScale)
526| 
527|         accs = (acc,)
528|         biases = (bias,)
529| 
530|         if SUBTILE_FACTOR >= 2:
531|             if SWAP_XW:
532|                 acc = acc.reshape(2, BLOCK_N // 2, BLOCK_M).permute(1, 2, 0)
533|             else:
534|                 acc = acc.reshape(BLOCK_M, 2, BLOCK_N // 2).permute(0, 2, 1)
535|             acc0, acc1 = acc.split()
536|             accs = (acc0, acc1)
537|             bias0, bias1 = bias.reshape(2, BLOCK_N // 2).permute(1, 0).split()
538|             biases = (bias0, bias1)
539| 
540|         if SUBTILE_FACTOR >= 4:
541|             if SWAP_XW:
542|                 acc0 = acc0.reshape(2, BLOCK_N // 4, BLOCK_M).permute(1, 2, 0)
543|                 acc1 = acc1.reshape(2, BLOCK_N // 4, BLOCK_M).permute(1, 2, 0)
544|             else:
545|                 acc0 = acc0.reshape(BLOCK_M, 2, BLOCK_N // 4).permute(0, 2, 1)
546|                 acc1 = acc1.reshape(BLOCK_M, 2, BLOCK_N // 4).permute(0, 2, 1)
547|             acc00, acc01 = acc0.split()
548|             acc10, acc11 = acc1.split()
549|             accs = (acc00, acc01, acc10, acc11)
550|             bias00, bias01 = bias0.reshape(2, BLOCK_N // 4).permute(1, 0).split()
551|             bias10, bias11 = bias1.reshape(2, BLOCK_N // 4).permute(1, 0).split()
552|             biases = (bias00, bias01, bias10, bias11)
553| 
554|         tl.static_assert(EPILOGUE_BLOCK_N == BLOCK_N // SUBTILE_FACTOR)
555|         tl.static_assert(len(accs) == SUBTILE_FACTOR)
556| 
557|         if is_out_microscaled:
558|             MX_SCALE_BLOCK_N: tl.constexpr = OUT_BLOCK_N // MX_BLOCK_SIZE
559| 
560|         for a_i in tl.static_range(len(accs)):
561|             acc_tile = accs[a_i]
562|             acc_tile *= x_scale * w_scale
563| 
564|             if SWAP_XW:
565|                 acc_tile = acc_tile.T
566| 
567|             acc_tile = acc_tile + biases[a_i][None, :] * betas[:, None]
568|             if out_alpha is not None:
569|                 acc_tile *= out_alpha
570| 
571|             if ACTIVATION_FN is not None:
572|                 out = ACTIVATION_FN(acc_tile, *activation_fn_args)
573|                 tl.static_assert(out.shape[1] == OUT_BLOCK_N, f"Activation fn out.shape[1] ({out.shape[1]}) doesn't match computed OUT_BLOCK_N ({OUT_BLOCK_N})")
574|             else:
575|                 tl.static_assert(ACTIVATION_REDUCTION_N == 1, "Activation reduction must be 1 if no activation fn is provided")
576|                 out = acc_tile
577| 
578|             out *= gammas[:, None]
579| 
580|             if OutAcc is not None:
581|                 tl.static_assert(not USE_SCATTER_TMA)
582|                 out_off_n = off_n1 // ACTIVATION_REDUCTION_N + a_i * OUT_BLOCK_N
583|                 if PER_BATCH_ACC_SCALE:
584|                     ScalePtr = OutAccScale + start_z1
585|                 else:
586|                     ScalePtr = OutAccScale
587| 
588|                 tl.static_assert(Y_TMA_MODE == "dense" or Y_TMA_MODE is None)
589|                 if Y_TMA_MODE == "dense":
590|                     off_kz = pid_k * batch_size + start_z1
591|                     acc = Y.load([off_kz, off_m1, out_off_n])
592|                     acc = acc.reshape(out.shape)
593|                     out += acc * load_scale(ScalePtr)
594|                 else:
595|                     offs_y_n = out_off_n + tl.arange(0, OUT_BLOCK_N)
596|                     mask_n = offs_y_n < yN
597| 
598|                     AccPtrs = YPtr + pid_k1.to(index_type) * stride_y_k + start_z1.to(index_type) * stride_y_z + offs_y_m.to(index_type)[:, None] * stride_y_m + offs_y_n[None, :] * stride_y_n
599|                     mask = mask_m[:, None] if OUT_N_TILE_ALIGNED else mask_m[:, None] & mask_n[None, :]
600|                     acc = tl.load(AccPtrs, mask=mask, other=0.0)
601|                     out += acc * load_scale(ScalePtr)
602| 
603|             if MASK_ACC:
604|                 out = tl.where(mask_m[:, None], out, 0.0)
605| 
606|             out_off_n = off_n1 // ACTIVATION_REDUCTION_N + a_i * OUT_BLOCK_N
607|             if is_out_microscaled:
608|                 tl.static_assert(EPILOGUE_FN is not None)
609|                 offs_y_n = out_off_n + tl.arange(0, OUT_BLOCK_N)
610|                 mask_n = offs_y_n < yN
611|                 out_mask = mask_m[:, None] if OUT_N_TILE_ALIGNED else mask_m[:, None] & mask_n[None, :]
612|                 if PER_BATCH_OUT_SCALE:
613|                     ExpectedScale = YExpectedScale + start_z1
614|                 else:
615|                     ExpectedScale = YExpectedScale
616|                 # OCP MX outputs leave YExpectedScale unset, so this is an
617|                 # identity there. NVFP4 uses YExpectedScale to precondition the
618|                 # dense output before the microscaling epilogue writes direct
619|                 # e4m3 block scales.
620|                 out = float_to_flex(
621|                     out,
622|                     ExpectedScale,
623|                     None,
624|                     None,
625|                     out_mask,
626|                     YPtr,
627|                     False,
628|                 )
629|                 out, out_scale = EPILOGUE_FN(out, out_mask, *epilogue_fn_args)
630|                 tl.static_assert(BLOCK_N % MX_SCALE_BLOCK_N == 0, "")
631|                 offs_y_n_scale = off_n1 // ACTIVATION_REDUCTION_N // MX_BLOCK_SIZE + a_i * MX_SCALE_BLOCK_N + tl.arange(0, MX_SCALE_BLOCK_N)
632|                 n_mx_blocks = tl.cdiv(yN, MX_BLOCK_SIZE)
633|                 YActualScalePtrs = output_mx_scale_store_ptr(
634|                     YActualScale,
635|                     offs_m,
636|                     offs_y_m,
637|                     offs_y_n_scale,
638|                     start_z1,
639|                     start_m1,
640|                     M,
641|                     n_mx_blocks,
642|                     XOutputScaleBlockOffs,
643|                     expt_id1,
644|                     pid_k,
645|                     pid_k1,
646|                     batch_size,
647|                     stride_y_mx_k,
648|                     stride_y_mx_z,
649|                     stride_y_mx_m,
650|                     stride_y_mx_n,
651|                     HAS_SCATTER,
652|                     USE_SCATTER_TMA,
653|                     Y_TMA_MODE,
654|                     RAGGED_DIMENSION,
655|                     Y_MX_SCALE_LAYOUT,
656|                     INDEX_TYPE=index_type,
657|                 )
658|                 mask_n_scale = offs_y_n_scale < n_mx_blocks
659|                 scale_store_mask = mask_m[:, None] if OUT_N_TILE_ALIGNED else mask_m[:, None] & mask_n_scale[None, :]
660|                 tl.store(YActualScalePtrs, out_scale, mask=scale_store_mask)
661|             else:
662|                 # Flexpoint
663|                 if USE_LOCAL_ABSMAX:
664|                     out_view = tl.reshape(out, [out.numel // THREADS_PER_BLOCK, THREADS_PER_BLOCK], can_reorder=True)
665|                     local_absmax = tl.maximum(local_absmax, nan_propagating_absmax_reduce(out_view, axis=0))
666| 
667|                 if PER_BATCH_OUT_SCALE:
668|                     ExpectedScale = YExpectedScale + start_z1
669|                     ActualScale = YActualScale + start_z1
670|                 else:
671|                     ExpectedScale = YExpectedScale
672|                     ActualScale = None  # local absmax is tracked and updated after the loop
673| 
674|                 out = float_to_flex(
675|                     out, ExpectedScale, ActualScale, YChecksumScale,
676|                     None, # mask: out is manually masked to 0
677|                     YPtr, FLEXPOINT_SATURATE_INF
678|                 )
679|                 if EPILOGUE_FN is not None and not IS_EPILOGUE_QUANT_MX:
680|                     out = EPILOGUE_FN(out, *epilogue_fn_args, target_dtype=YPtr.dtype.element_ty, pid=len(accs)*tile_id1 + a_i)
681| 
682|             out = out.to(YPtr.dtype.element_ty)
683|             if is_out_fp4:
684|                 out_off_n = out_off_n // 2
685|                 offs_y_n = out_off_n + tl.arange(0, OUT_BLOCK_N // 2)
686|                 mask_n = offs_y_n < tl.cdiv(yN, 2)
687| 
688|             if pYPtrs is None:
689|                 if USE_SCATTER_TMA:
690|                     # Convert -1 offsets to INT_MAX. We do this by clearing the leading bit. Note that
691|                     # there shouldn't be any other negative values.
692|                     offs_y_m = (offs_y_m.to(tl.uint32, bitcast=True) & 0x7FFFFFFF).to(tl.int32, bitcast=True)
693|                     Y.scatter(out, offs_y_m, out_off_n)
694|                 elif Y_TMA_MODE == "dense":
695|                     out = tl.reshape(out, [1] + out.shape)
696|                     off_kz = pid_k * batch_size + start_z1
697|                     Y.store([off_kz, off_m1, out_off_n], out)
698|                 elif Y_TMA_MODE == "ragged":
699|                     out = tl.reshape(out, [1] + out.shape)
700|                     store_ragged(Y, start_m1, eM1, [pid_k, off_m1, out_off_n], out, ragged_dim=1)
701|                 else:
702|                     tl.static_assert(Y_TMA_MODE is None)
703|                     offs_y_n = out_off_n + tl.arange(0, OUT_BLOCK_N)
704|                     mask_n = offs_y_n < yN
705|                     if is_out_fp4:
706|                         offs_y_n = out_off_n + tl.arange(0, OUT_BLOCK_N // 2)
707|                         mask_n = offs_y_n < tl.cdiv(yN, 2)
708|                     mask = mask_m[:, None] if OUT_N_TILE_ALIGNED else mask_m[:, None] & mask_n[None, :]
709|                     offs_kzmn = pid_k1.to(index_type) * stride_y_k + start_z1.to(index_type) * stride_y_z + offs_y_m.to(index_type)[:, None] * stride_y_m + offs_y_n[None, :] * stride_y_n
710|                     tl.store(YPtr + offs_kzmn, out, mask=mask)
711|             else:
712|                 tl.static_assert(not is_out_fp4, "FP4 outputs are not supported with fused comms")
713|                 tl.static_assert(Y_TMA_MODE is None, "TMA is not supported with fused comms")
714|                 offs_y_n = out_off_n + tl.arange(0, OUT_BLOCK_N)
715|                 mask_n = offs_y_n < yN
716|                 mask = mask_m[:, None] if OUT_N_TILE_ALIGNED else mask_m[:, None] & mask_n[None, :]
717| 
718|                 dst_shard_idx, dst_y_m, dst_y_n = map_dst_coord.fn(
719|                     start_m1 + off_m1 if WriteBackIndx is None else None, offs_y_m,
720|                     out_off_n, offs_y_n,
721|                     *map_dst_coord.captured)
722|                 offs_kzmn = (
723|                     pid_k1.to(index_type) * stride_y_k +
724|                     start_z1.to(index_type) * stride_y_z +
725|                     dst_y_n[None, :] * stride_y_n +
726|                     dst_y_m.to(index_type)[:, None] * stride_y_m * n_reduce_shards + reduce_rank * stride_y_m
727|                 )
728|                 for i in tl.static_range(n_reduce_shards):
729|                     if dst_shard_idx is not None:
730|                         peer = dst_shard_idx * n_reduce_shards + (reduce_rank + i) % n_reduce_shards
731|                     else:
732|                         peer = (reduce_rank + i) % n_reduce_shards
733|                     peer_Y_ptr = tl.load(pYPtrs + peer).to(tl.pointer_type(YPtr.type.element_ty))
734|                     if len(peer_Y_ptr.shape) == 0:
735|                         tl.multiple_of(peer_Y_ptr, 16)
736|                     else:
737|                         tl.multiple_of(peer_Y_ptr, [16, 16])
738|                     tl.store(peer_Y_ptr + offs_kzmn, out, mask=mask)
739| 
740| 
741|     # Update the flexpoint scales
```
**EN:** Loops over `tl.range(tl.program_id(0), num_blocks, NUM_SMS, flatten=F...` with target `block_id`. The loop body mainly prepares intermediate values; prepares intermediate values.

**CN:** 遍历 `tl.range(tl.program_id(0), num_blocks, NUM_SMS, flatten=F...` ，目标变量为 `block_id`. 循环体主要准备中间值; 准备中间值.

### Block 48 — Lines 742-744 (_p_matmul)
```python
742|     if USE_LOCAL_ABSMAX:
743|         tl.atomic_max(YActualScale, compute_scale(local_absmax.to(tl.float32, bitcast=True), YPtr), sem="relaxed")
744| 
```
**EN:** Checks `USE_LOCAL_ABSMAX`. The true branch mainly invokes `tl.atomic_max`.

**CN:** 检查 `USE_LOCAL_ABSMAX`. 真分支主要invokes `tl.atomic_max`.

### Block 49 — Lines 745-748 (_p_matmul)
```python
745|     if pYPtrs is not None:
746|         all_writes_issued.fn(*all_writes_issued.captured)
747| 
748| 
```
**EN:** Checks `pYPtrs is not None`. The true branch mainly invokes `all_writes_issued.fn`.

**CN:** 检查 `pYPtrs is not None`. 真分支主要invokes `all_writes_issued.fn`.

### Block 50 — Lines 749-751 (module)
```python
749| _per_device_alloc_fns = {}
750| 
751| 
```
**EN:** Assigns `_per_device_alloc_fns` and builds a dictionary.

**CN:** 将 `_per_device_alloc_fns` and 构造一个字典.

### Block 51 — Lines 752-752 (get_per_device_per_stream_alloc_fn)
```python
752| def get_per_device_per_stream_alloc_fn(device):
```
**EN:** Defines function `get_per_device_per_stream_alloc_fn(device)` for this module. The body mainly branches on runtime conditions; returns the computed result. It uses calls such as `collections.defaultdict`, `tensors.append`, `tensors[-1].numel`, `torch.empty` to implement its workflow.

**CN:** 定义函数 `get_per_device_per_stream_alloc_fn(device)`，供本模块使用. 主体主要根据运行时条件分支; 返回计算结果. 其中会调用 `collections.defaultdict`, `tensors.append`, `tensors[-1].numel`, `torch.empty` 来实现其工作流程.

### Block 52 — Lines 753-764 (get_per_device_per_stream_alloc_fn)
```python
753|     if device not in _per_device_alloc_fns:
754|         _per_stream_tensors = collections.defaultdict(list)
755| 
756|         def alloc_fn(size: int, alignment: int, stream: int):
757|             assert alignment == 128
758|             tensors = _per_stream_tensors[stream]
759|             if not tensors or tensors[-1].numel() < size:
760|                 tensors.append(torch.empty(size, device=device, dtype=torch.int8))
761|                 tensors[-1].__hibernate__ = {"type": "ignore"}
762|             return tensors[-1]
763| 
764|         _per_device_alloc_fns[device] = alloc_fn
```
**EN:** Checks `device not in _per_device_alloc_fns`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `device not in _per_device_alloc_fns`. 真分支主要准备中间值; 准备中间值.

### Block 53 — Lines 765-765 (get_per_device_per_stream_alloc_fn)
```python
765|     return _per_device_alloc_fns[device]
```
**EN:** Returns `_per_device_alloc_fns[device]`.

**CN:** 返回 `_per_device_alloc_fns[device]`.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `cuda_capability_geq`, `get_dtype`, `_load_writeback_idx_and_mask`, `round_f32_to_tf32`, `_p_matmul`, `get_per_device_per_stream_alloc_fn`.
  **CN:** 主要符号：`cuda_capability_geq`, `get_dtype`, `_load_writeback_idx_and_mask`, `round_f32_to_tf32`, `_p_matmul`, `get_per_device_per_stream_alloc_fn`。
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
- **EN:** Reduction logic combines partial values across dimensions or shards.
  **CN:** 归约逻辑会沿着维度或分片合并部分结果。
- **EN:** Ragged tensor metadata is used when rows or slices have non-uniform sizes.
  **CN:** 当行或切片大小不一致时，会使用 ragged tensor 元数据。

## Dependencies / 依赖关系
- **EN:** External modules: `collections`, `torch`, `triton`, `triton.language`, `triton.tools.ragged_tma (load_ragged, store_ragged)`.
  **CN:** 外部模块：`collections`, `torch`, `triton`, `triton.language`, `triton.tools.ragged_tma (load_ragged, store_ragged)`。
- **EN:** Internal modules: `triton_kernels (target_info)`, `triton_kernels.tensor_details.layout_details.blackwell_scale (unswizzle_mx_scale_bw, unswizzle_act_mx_scale_bw)`, `triton_kernels.numerics_details.flexpoint (float_to_flex, load_scale, nan_propagating_absmax_reduce, compute_scale)`, `triton_kernels.numerics_details.mxfp_details._downcast_to_mxfp (MXFP_BLOCK_SIZE, NVFP_BLOCK_SIZE)`, `triton_kernels.numerics_details.mxfp_details._upcast_from_mxfp (upcast_mxfp4_tile)`, `triton_kernels.tensor_details.layout_details.hopper_scale (unswizzle_mxfp4_scale_hopper)`, `triton_kernels.tensor_details.layout_details.hopper_value (mxfp4_to_bf16_triton)`, `._common (compute_offsets, get_scaled_dot_format_string, make_matmul_repr, matmul_launch_metadata, compute_pids, output_mx_scale_store_ptr)`.
  **CN:** 内部模块：`triton_kernels (target_info)`, `triton_kernels.tensor_details.layout_details.blackwell_scale (unswizzle_mx_scale_bw, unswizzle_act_mx_scale_bw)`, `triton_kernels.numerics_details.flexpoint (float_to_flex, load_scale, nan_propagating_absmax_reduce, compute_scale)`, `triton_kernels.numerics_details.mxfp_details._downcast_to_mxfp (MXFP_BLOCK_SIZE, NVFP_BLOCK_SIZE)`, `triton_kernels.numerics_details.mxfp_details._upcast_from_mxfp (upcast_mxfp4_tile)`, `triton_kernels.tensor_details.layout_details.hopper_scale (unswizzle_mxfp4_scale_hopper)`, `triton_kernels.tensor_details.layout_details.hopper_value (mxfp4_to_bf16_triton)`, `._common (compute_offsets, get_scaled_dot_format_string, make_matmul_repr, matmul_launch_metadata, compute_pids, output_mx_scale_store_ptr)`。
