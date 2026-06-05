# _matmul.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/triton_kernels/matmul_details/_matmul.py`
- **Purpose / 用途:** Implementation module for matmul; it defines the runtime logic, helper types, and GPU-facing utilities used by the Triton kernels package via symbols round_f32_to_tf32, _matmul. / 用于 matmul 的实现模块；它定义了 Triton kernels 包使用的运行时逻辑、辅助类型以及面向 GPU 的工具；核心符号包括 round_f32_to_tf32、_matmul。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-2 (module)
```python
1| # isort: off
2| # fmt: off
```
**EN:** This preamble contains file-level comments or formatting directives that keep imports and generated code stable.

**CN:** 这一前导部分包含文件级注释或格式化指令，用于保持导入顺序和生成代码稳定。

### Block 2 — Lines 3-25 (module)
```python
 3| import triton
 4| import triton.language as tl
 5| from triton_kernels.tensor_details.layout_details.blackwell_scale import (
 6|     unswizzle_mx_scale_bw,
 7| )
 8| from triton_kernels.tensor_details.layout_details.hopper_scale import unswizzle_mxfp4_scale_hopper
 9| from triton_kernels.tensor_details.layout_details.hopper_value import mxfp4_to_bf16_triton
10| from triton_kernels.tensor_details.layout_details.cdna4_scale import unswizzle_mx_scale_cdna4
11| from triton_kernels.tensor_details.layout_details.gfx1250_scale import unswizzle_mx_scale_gfx1250
12| from triton_kernels.numerics_details.flexpoint import float_to_flex, load_scale
13| from triton_kernels.numerics_details.mxfp_details._downcast_to_mxfp import MXFP_BLOCK_SIZE, NVFP_BLOCK_SIZE
14| from triton_kernels.numerics_details.mxfp_details._upcast_from_mxfp import upcast_mxfp4_tile
15| from triton_kernels.target_info import cuda_capability_geq
16| from ._common import (
17|     compute_offsets,
18|     get_scaled_dot_format_string,
19|     make_matmul_repr,
20|     matmul_launch_metadata,
21|     compute_pids,
22|     output_mx_scale_store_ptr,
23| )
24| 
25| 
```
**EN:** This block imports `triton`, `triton.language`, `triton_kernels.tensor_details.layout_details.blackwell_scale (unswizzle_mx_scale_bw)`, `triton_kernels.tensor_details.layout_details.hopper_scale (unswizzle_mxfp4_scale_hopper)`, `triton_kernels.tensor_details.layout_details.hopper_value (mxfp4_to_bf16_triton)`, `triton_kernels.tensor_details.layout_details.cdna4_scale (unswizzle_mx_scale_cdna4)`, `triton_kernels.tensor_details.layout_details.gfx1250_scale (unswizzle_mx_scale_gfx1250)`, `triton_kernels.numerics_details.flexpoint (float_to_flex, load_scale)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `triton`, `triton.language`, `triton_kernels.tensor_details.layout_details.blackwell_scale (unswizzle_mx_scale_bw)`, `triton_kernels.tensor_details.layout_details.hopper_scale (unswizzle_mxfp4_scale_hopper)`, `triton_kernels.tensor_details.layout_details.hopper_value (mxfp4_to_bf16_triton)`, `triton_kernels.tensor_details.layout_details.cdna4_scale (unswizzle_mx_scale_cdna4)`, `triton_kernels.tensor_details.layout_details.gfx1250_scale (unswizzle_mx_scale_gfx1250)`, `triton_kernels.numerics_details.flexpoint (float_to_flex, load_scale)` ，为模块提供所需的外部库和内部辅助工具。

### Block 3 — Lines 26-28 (round_f32_to_tf32)
```python
26| @triton.jit
27| def round_f32_to_tf32(x: tl.tensor):
28|     # use cvt.rn on Hopper+ to match the rounding of TMA.
```
**EN:** Defines function `round_f32_to_tf32(x)` with decorators `triton.jit` for this module. The body mainly prepares intermediate values; returns the computed result. It uses calls such as `tl.inline_asm_elementwise`, `cuda_capability_geq` to implement its workflow.

**CN:** 定义函数 `round_f32_to_tf32(x)`，带有装饰器 `triton.jit`，供本模块使用. 主体主要准备中间值; 返回计算结果. 其中会调用 `tl.inline_asm_elementwise`, `cuda_capability_geq` 来实现其工作流程.

### Block 4 — Lines 29-29 (round_f32_to_tf32)
```python
29|     ASM: tl.constexpr = "cvt.rn.tf32.f32 $0, $1;" if cuda_capability_geq(9, 0) else "cvt.rna.tf32.f32 $0, $1;"
```
**EN:** Annotated assignment stores `ASM` and uses conditional expression `'cvt.rn.tf32.f32 $0, $1;' if cuda_capability_geq(9, 0) el...`.

**CN:** 带类型注解的赋值保存 `ASM` and 使用条件表达式 `'cvt.rn.tf32.f32 $0, $1;' if cuda_capability_geq(9, 0) el...`.

### Block 5 — Lines 30-31 (round_f32_to_tf32)
```python
30|     return tl.inline_asm_elementwise(ASM, "=r, r", [x], dtype=tl.float32, is_pure=True, pack=1)
31| 
```
**EN:** Returns `tl.inline_asm_elementwise(ASM, '=r, r', [x], dtype=tl.float32, is_pure=True, ...`.

**CN:** 返回 `tl.inline_asm_elementwise(ASM, '=r, r', [x], dtype=tl.float32, is_pure=True, ...`.

### Block 6 — Lines 32-32 (module)
```python
32| _matmul_repr = make_matmul_repr("_matmul", [0, 1, 2])
```
**EN:** Assigns `_matmul_repr` and calls `make_matmul_repr`.

**CN:** 将 `_matmul_repr`，并调用 `make_matmul_repr`.

### Block 7 — Lines 33-100 (_matmul)
```python
 33| @triton.jit(do_not_specialize=["TOKENS_PER_EXPT_FOR_ANNOTATION"],
 34|             repr=_matmul_repr, launch_metadata=matmul_launch_metadata)
 35| def _matmul(
 36|              Y, YPtr, stride_y_k, stride_y_z, stride_y_m, stride_y_n,
 37|              YExpectedScale, YActualScale, YChecksumScale,
 38|              stride_y_mx_k, stride_y_mx_z, stride_y_mx_m, stride_y_mx_n,
 39|              X, XPtr, stride_x_z, stride_x_m, stride_x_k, X_TRANSPOSE: tl.constexpr,
 40|              XScale,
 41|              XMxScale, stride_x_mx_z, stride_x_mx_m, stride_x_mx_k,
 42|              W, WPtr, stride_w_e, stride_w_k, stride_w_n, W_TRANSPOSE: tl.constexpr,
 43|              WScale,
 44|              WMxScale, stride_w_mx_e, stride_w_mx_k, stride_w_mx_n,
 45|              OutAcc, stride_acc_z, stride_acc_m, stride_acc_n,
 46|              OutAccScale, Y_ACC_IS_Y: tl.constexpr,
 47|              B, stride_b_e, # Bias
 48|              M, N, K, K_W, # shapes
 49|              # expt data
 50|              Betas, Gammas,
 51|              GatherIndx,
 52|              WriteBackIndx, writeback_size,
 53|              RAGGED_DIMENSION: tl.constexpr,
 54|              XSliceSizes, XSliceOffs, XBlockOffs, XBlockSchedule, X_EXPECTED_SLICE_SIZE: tl.constexpr, X_SLICE_SIZES_DIVISIBILITY: tl.constexpr,
 55|              XOutputScaleBlockOffs,
 56|              WSliceSizes, WSliceOffs, WBlockOffs, WBlockSchedule, W_EXPECTED_SLICE_SIZE: tl.constexpr, _W_SLICE_SIZES_DIVISIBILITY: tl.constexpr,
 57|              # true grid size
 58|              batch_size, grid_m, grid_n,
 59|              # Out scale
 60|              out_alpha,
 61|              # fused activation function
 62|              ACTIVATION_FN: tl.constexpr, activation_fn_args, ACTIVATION_REDUCTION_N: tl.constexpr,
 63|              # epilogue transform
 64|              EPILOGUE_FN: tl.constexpr, epilogue_fn_args,
 65|              # MoE config
 66|              N_EXPTS_TOT: tl.constexpr,
 67|              # precision config
 68|              MAX_NUM_IMPRECISE_ACC: tl.constexpr, ALLOW_TF32: tl.constexpr,
 69|              FLEXPOINT_SATURATE_INF: tl.constexpr,
 70|              PER_BATCH_W_SCALE: tl.constexpr,
 71|              PER_BATCH_OUT_SCALE: tl.constexpr,
 72|              PER_BATCH_ACC_SCALE: tl.constexpr,
 73|              # optimization config
 74|              BLOCK_M: tl.constexpr, BLOCK_N: tl.constexpr, BLOCK_K: tl.constexpr,
 75|              GROUP_M: tl.constexpr, XCD_SWIZZLE: tl.constexpr,
 76|              # One of ["HOPPER", "BLACKWELL", None]
 77|              SWIZZLE_MX_VALUE: tl.constexpr,
 78|              # One of ["HOPPER", "BLACKWELL", None]
 79|              SWIZZLE_MX_SCALE: tl.constexpr,
 80|              MX_BLOCK_SIZE: tl.constexpr,
 81|              EPILOGUE_SUBTILE: tl.constexpr,
 82|              EVEN_K: tl.constexpr, SPLIT_K: tl.constexpr,
 83|              W_CACHE_MODIFIER: tl.constexpr,
 84|              NUM_SMS: tl.constexpr,
 85|              X_TMA_MODE: tl.constexpr,
 86|              Y_TMA_MODE: tl.constexpr,
 87|              Y_MX_SCALE_LAYOUT: tl.constexpr = None,
 88|              OUT_N_TILE_ALIGNED: tl.constexpr = False,
 89|              TOKENS_PER_EXPT_FOR_ANNOTATION=None,
 90|              UPCAST_INDICES: tl.constexpr = False,
 91|              SWAP_XW: tl.constexpr = False,
 92|              IS_EPILOGUE_QUANT_MX: tl.constexpr = False,
 93|              Y_VALUE_PACK_FACTOR: tl.constexpr = 1,
 94|              FLATTEN_LOOPS: tl.constexpr = True, # Only relevant to persistent kernel
 95|              pYPtrs=None,
 96|              map_dst_coord=None,
 97|              all_writes_issued=None,
 98|              reduce_rank = 0,
 99|              n_reduce_shards: tl.constexpr = 1,
100|              ):
```
**EN:** Defines function `_matmul(Y, YPtr, stride_y_k, stride_y_z, stride_y_m, stride_y_n, YExpectedScale, YActualScale, YChecksumScale, stride_y_mx_k, stride_y_mx_z, stride_y_mx_m, stride_y_mx_n, X, XPtr, stride_x_z, stride_x_m, stride_x_k, X_TRANSPOSE, XScale, XMxScale, stride_x_mx_z, stride_x_mx_m, stride_x_mx_k, W, WPtr, stride_w_e, stride_w_k, stride_w_n, W_TRANSPOSE, WScale, WMxScale, stride_w_mx_e, stride_w_mx_k, stride_w_mx_n, OutAcc, stride_acc_z, stride_acc_m, stride_acc_n, OutAccScale, Y_ACC_IS_Y, B, stride_b_e, M, N, K, K_W, Betas, Gammas, GatherIndx, WriteBackIndx, writeback_size, RAGGED_DIMENSION, XSliceSizes, XSliceOffs, XBlockOffs, XBlockSchedule, X_EXPECTED_SLICE_SIZE, X_SLICE_SIZES_DIVISIBILITY, XOutputScaleBlockOffs, WSliceSizes, WSliceOffs, WBlockOffs, WBlockSchedule, W_EXPECTED_SLICE_SIZE, _W_SLICE_SIZES_DIVISIBILITY, batch_size, grid_m, grid_n, out_alpha, ACTIVATION_FN, activation_fn_args, ACTIVATION_REDUCTION_N, EPILOGUE_FN, epilogue_fn_args, N_EXPTS_TOT, MAX_NUM_IMPRECISE_ACC, ALLOW_TF32, FLEXPOINT_SATURATE_INF, PER_BATCH_W_SCALE, PER_BATCH_OUT_SCALE, PER_BATCH_ACC_SCALE, BLOCK_M, BLOCK_N, BLOCK_K, GROUP_M, XCD_SWIZZLE, SWIZZLE_MX_VALUE, SWIZZLE_MX_SCALE, MX_BLOCK_SIZE, EPILOGUE_SUBTILE, EVEN_K, SPLIT_K, W_CACHE_MODIFIER, NUM_SMS, X_TMA_MODE, Y_TMA_MODE, Y_MX_SCALE_LAYOUT, OUT_N_TILE_ALIGNED, TOKENS_PER_EXPT_FOR_ANNOTATION, UPCAST_INDICES, SWAP_XW, IS_EPILOGUE_QUANT_MX, Y_VALUE_PACK_FACTOR, FLATTEN_LOOPS, pYPtrs, map_dst_coord, all_writes_issued, reduce_rank, n_reduce_shards)` with decorators `triton.jit(do_not_specialize=['TOKENS...` for this module. The body mainly invokes `tl.assume`; invokes `tl.assume`; invokes `tl.assume`. It uses calls such as `triton.jit`, `tl.assume`, `tl.program_id`, `compute_pids`, `compute_offsets` to implement its workflow.

**CN:** 定义函数 `_matmul(Y, YPtr, stride_y_k, stride_y_z, stride_y_m, stride_y_n, YExpectedScale, YActualScale, Y检查umScale, stride_y_mx_k, stride_y_mx_z, stride_y_mx_m, stride_y_mx_n, X, XPtr, stride_x_z, stride_x_m, stride_x_k, X_TRANSPOSE, XScale, XMxScale, stride_x_mx_z, stride_x_mx_m, stride_x_mx_k, W, WPtr, stride_w_e, stride_w_k, stride_w_n, W_TRANSPOSE, WScale, WMxScale, stride_w_mx_e, stride_w_mx_k, stride_w_mx_n, OutAcc, stride_acc_z, stride_acc_m, stride_acc_n, OutAccScale, Y_ACC_IS_Y, B, stride_b_e, M, N, K, K_W, Betas, Gammas, GatherIndx, WriteBackIndx, writeback_size, RAGGED_DIMENSION, XSliceSizes, XSliceOffs, XBlockOffs, XBlockSchedule, X_EXPECTED_SLICE_SIZE, X_SLICE_SIZES_DIVISIBILITY, XOutputScaleBlockOffs, WSliceSizes, WSliceOffs, WBlockOffs, WBlockSchedule, W_EXPECTED_SLICE_SIZE, _W_SLICE_SIZES_DIVISIBILITY, batch_size, grid_m, grid_n, out_alpha, ACTIVATION_FN, activation_fn_args, ACTIVATION_REDUCTION_N, EPILOGUE_FN, epilogue_fn_args, N_EXPTS_TOT, MAX_NUM_IMPRECISE_ACC, ALLOW_TF32, FLEXPOINT_SATURATE_INF, PER_BATCH_W_SCALE, PER_BATCH_OUT_SCALE, PER_BATCH_ACC_SCALE, BLOCK_M, BLOCK_N, BLOCK_K, GROUP_M, XCD_SWIZZLE, SWIZZLE_MX_VALUE, SWIZZLE_MX_SCALE, MX_BLOCK_SIZE, EPILOGUE_SUBTILE, EVEN_K, SPLIT_K, W_CACHE_MODIFIER, NUM_SMS, X_TMA_MODE, Y_TMA_MODE, Y_MX_SCALE_LAYOUT, OUT_N_TILE_ALIGNED, TOKENS_PER_EXPT_FOR_ANNOTATION, UPCAST_INDICES, SWAP_XW, IS_EPILOGUE_QUANT_MX, Y_VALUE_PACK_FACTOR, FLATTEN_LOOPS, pYPtrs, map_dst_coord, all_writes_issued, reduce_rank, n_reduce_shards)`，带有装饰器 `triton.jit(do_not_specialize=['TOKENS...`，供本模块使用. 主体主要invokes `tl.assume`; invokes `tl.assume`; invokes `tl.assume`. 其中会调用 `triton.jit`, `tl.assume`, `tl.program_id`, `compute_pids`, `compute_offsets` 来实现其工作流程.

### Block 8 — Lines 101-101 (_matmul)
```python
101|     tl.assume(stride_y_k >= 0)
```
**EN:** Calls `tl.assume` for side effects, registration, or validation.

**CN:** 调用 `tl.assume` ，用于副作用、注册或校验。

### Block 9 — Lines 102-102 (_matmul)
```python
102|     tl.assume(stride_y_z >= 0)
```
**EN:** Calls `tl.assume` for side effects, registration, or validation.

**CN:** 调用 `tl.assume` ，用于副作用、注册或校验。

### Block 10 — Lines 103-103 (_matmul)
```python
103|     tl.assume(stride_y_m >= 0)
```
**EN:** Calls `tl.assume` for side effects, registration, or validation.

**CN:** 调用 `tl.assume` ，用于副作用、注册或校验。

### Block 11 — Lines 104-104 (_matmul)
```python
104|     tl.assume(stride_y_n >= 0)
```
**EN:** Calls `tl.assume` for side effects, registration, or validation.

**CN:** 调用 `tl.assume` ，用于副作用、注册或校验。

### Block 12 — Lines 105-105 (_matmul)
```python
105|     tl.assume(stride_x_z >= 0)
```
**EN:** Calls `tl.assume` for side effects, registration, or validation.

**CN:** 调用 `tl.assume` ，用于副作用、注册或校验。

### Block 13 — Lines 106-106 (_matmul)
```python
106|     tl.assume(stride_x_m >= 0)
```
**EN:** Calls `tl.assume` for side effects, registration, or validation.

**CN:** 调用 `tl.assume` ，用于副作用、注册或校验。

### Block 14 — Lines 107-107 (_matmul)
```python
107|     tl.assume(stride_x_k >= 0)
```
**EN:** Calls `tl.assume` for side effects, registration, or validation.

**CN:** 调用 `tl.assume` ，用于副作用、注册或校验。

### Block 15 — Lines 108-108 (_matmul)
```python
108|     tl.assume(stride_w_e >= 0)
```
**EN:** Calls `tl.assume` for side effects, registration, or validation.

**CN:** 调用 `tl.assume` ，用于副作用、注册或校验。

### Block 16 — Lines 109-109 (_matmul)
```python
109|     tl.assume(stride_w_k >= 0)
```
**EN:** Calls `tl.assume` for side effects, registration, or validation.

**CN:** 调用 `tl.assume` ，用于副作用、注册或校验。

### Block 17 — Lines 110-110 (_matmul)
```python
110|     tl.assume(stride_w_n >= 0)
```
**EN:** Calls `tl.assume` for side effects, registration, or validation.

**CN:** 调用 `tl.assume` ，用于副作用、注册或校验。

### Block 18 — Lines 111-112 (_matmul)
```python
111|     if stride_w_mx_e is not None:
112|         tl.assume(stride_w_mx_e >= 0)
```
**EN:** Checks `stride_w_mx_e is not None`. The true branch mainly invokes `tl.assume`.

**CN:** 检查 `stride_w_mx_e is not None`. 真分支主要invokes `tl.assume`.

### Block 19 — Lines 113-114 (_matmul)
```python
113|     if stride_w_mx_k is not None:
114|         tl.assume(stride_w_mx_k >= 0)
```
**EN:** Checks `stride_w_mx_k is not None`. The true branch mainly invokes `tl.assume`.

**CN:** 检查 `stride_w_mx_k is not None`. 真分支主要invokes `tl.assume`.

### Block 20 — Lines 115-116 (_matmul)
```python
115|     if stride_w_mx_n is not None:
116|         tl.assume(stride_w_mx_n >= 0)
```
**EN:** Checks `stride_w_mx_n is not None`. The true branch mainly invokes `tl.assume`.

**CN:** 检查 `stride_w_mx_n is not None`. 真分支主要invokes `tl.assume`.

### Block 21 — Lines 117-118 (_matmul)
```python
117|     if B is not None:
118|         tl.assume(stride_b_e >= 0)
```
**EN:** Checks `B is not None`. The true branch mainly invokes `tl.assume`.

**CN:** 检查 `B is not None`. 真分支主要invokes `tl.assume`.

### Block 22 — Lines 119-119 (_matmul)
```python
119|     tl.assume(batch_size >= 0)
```
**EN:** Calls `tl.assume` for side effects, registration, or validation.

**CN:** 调用 `tl.assume` ，用于副作用、注册或校验。

### Block 23 — Lines 120-120 (_matmul)
```python
120|     tl.assume(grid_m >= 0)
```
**EN:** Calls `tl.assume` for side effects, registration, or validation.

**CN:** 调用 `tl.assume` ，用于副作用、注册或校验。

### Block 24 — Lines 121-123 (_matmul)
```python
121|     tl.assume(grid_n >= 0)
122| 
123| 
```
**EN:** Calls `tl.assume` for side effects, registration, or validation.

**CN:** 调用 `tl.assume` ，用于副作用、注册或校验。

### Block 25 — Lines 124-124 (_matmul)
```python
124|     w_type: tl.constexpr = W.dtype.element_ty
```
**EN:** Annotated assignment stores `w_type` and references `W.dtype.element_ty`.

**CN:** 带类型注解的赋值保存 `w_type` and 引用 `W.dtype.element_ty`.

### Block 26 — Lines 125-125 (_matmul)
```python
125|     is_x_microscaled: tl.constexpr = XMxScale is not None
```
**EN:** Annotated assignment stores `is_x_microscaled` and evaluates `XMxScale is not None`.

**CN:** 带类型注解的赋值保存 `is_x_microscaled` and 计算 `XMxScale is not None`.

### Block 27 — Lines 126-126 (_matmul)
```python
126|     is_w_microscaled: tl.constexpr = WMxScale is not None
```
**EN:** Annotated assignment stores `is_w_microscaled` and evaluates `WMxScale is not None`.

**CN:** 带类型注解的赋值保存 `is_w_microscaled` and 计算 `WMxScale is not None`.

### Block 28 — Lines 127-127 (_matmul)
```python
127|     is_w_mxfp4: tl.constexpr = w_type == tl.uint8 and is_w_microscaled
```
**EN:** Annotated assignment stores `is_w_mxfp4` and evaluates `w_type == tl.uint8 and is_w_microscaled`.

**CN:** 带类型注解的赋值保存 `is_w_mxfp4` and 计算 `w_type == tl.uint8 and is_w_microscaled`.

### Block 29 — Lines 128-128 (_matmul)
```python
128|     MX_PACK_DIVISOR: tl.constexpr = MX_BLOCK_SIZE
```
**EN:** Annotated assignment stores `MX_PACK_DIVISOR` and references `MX_BLOCK_SIZE`.

**CN:** 带类型注解的赋值保存 `MX_PACK_DIVISOR` and 引用 `MX_BLOCK_SIZE`.

### Block 30 — Lines 129-131 (_matmul)
```python
129|     if is_x_microscaled or is_w_microscaled:
130|         MX_SCALE_BLOCK_K: tl.constexpr = BLOCK_K // MX_PACK_DIVISOR
131| 
```
**EN:** Checks `is_x_microscaled or is_w_microscaled`. The true branch mainly prepares intermediate values.

**CN:** 检查 `is_x_microscaled or is_w_microscaled`. 真分支主要准备中间值.

### Block 31 — Lines 132-176 (_matmul)
```python
132|     if is_w_microscaled:
133|         tl.static_assert(MX_BLOCK_SIZE == NVFP_BLOCK_SIZE or MX_BLOCK_SIZE == MXFP_BLOCK_SIZE,
134|                          "Unsupported microscale factor")
135|         tl.static_assert(w_type == tl.uint8 or (w_type == tl.float8e4nv or w_type == tl.float8e5),
136|                          "mx_weight_ptr must be uint8 or fp8")
137|         # NOTE: uint8 scale means OCP E8M0 here. Direct NVFP-style scales stay float8e4nv.
138|         tl.static_assert(
139|             WMxScale.dtype.element_ty == tl.uint8
140|             or WMxScale.dtype.element_ty == tl.float8e4nv,
141|             "mx_scale_ptr must be uint8 or float8e4nv",
142|         )
143|         tl.static_assert(BLOCK_K % MX_PACK_DIVISOR == 0, f"{BLOCK_K=} must be a multiple of {MX_PACK_DIVISOR=}")
144|         tl.static_assert(SWIZZLE_MX_VALUE == "HOPPER_VALUE" or SWIZZLE_MX_VALUE == "STRIDED", "Only Hopper swizzling is supported for values")
145| 
146|         if SWIZZLE_MX_VALUE == "HOPPER_VALUE":
147|             tl.static_assert(is_w_mxfp4, "Only mxfp4 is supported for HOPPER swizzling")
148|             # We have pack 2 fp4 values in a byte but we divide the dimension by 2
149|             # when swizzling
150|             W_K_DIVISOR: tl.constexpr = 1
151|             W_K_MULTIPLIER: tl.constexpr = 2
152|             W_N_DIVISOR: tl.constexpr = 4
153|         else:
154|             # We have pack 2 fp4 values in a  byte
155|             W_K_DIVISOR: tl.constexpr = 2 if is_w_mxfp4 else 1
156|             W_K_MULTIPLIER: tl.constexpr = 1
157|             W_N_DIVISOR: tl.constexpr = 1
158| 
159|         if W_TRANSPOSE:
160|             # When weight is transposed, 2 fp4 values are packed per Byte along
161|             # the contiguous dimension, K.
162|             PACKED_BLOCK_K_W: tl.constexpr = (BLOCK_K // W_K_DIVISOR) * W_K_MULTIPLIER
163|             PACKED_BLOCK_N_W: tl.constexpr = BLOCK_N // W_N_DIVISOR
164|         else:
165|             # When weight is not transposed, fp4 values are *not* packed along
166|             # the contiguous dimension, N.
167|             PACKED_BLOCK_K_W: tl.constexpr = BLOCK_K
168|             PACKED_BLOCK_N_W: tl.constexpr = BLOCK_N // W_K_DIVISOR
169|     else:
170|         W_K_DIVISOR: tl.constexpr = 1
171|         W_K_MULTIPLIER: tl.constexpr = 1
172|         W_N_DIVISOR: tl.constexpr = 1
173|         PACKED_BLOCK_K_W: tl.constexpr = BLOCK_K
174|         PACKED_BLOCK_N_W: tl.constexpr = BLOCK_N
175|         tl.static_assert(SWIZZLE_MX_VALUE == "STRIDED")
176|         tl.static_assert(SWIZZLE_MX_SCALE == "STRIDED")
```
**EN:** Checks `is_w_microscaled`. The true branch mainly invokes `tl.static_assert`; invokes `tl.static_assert`, while the else branch prepares intermediate values; prepares intermediate values.

**CN:** 检查 `is_w_microscaled`. 真分支主要invokes `tl.static_assert`; invokes `tl.static_assert`；而 else 分支准备中间值; 准备中间值.

### Block 32 — Lines 177-189 (_matmul)
```python
177|     if is_x_microscaled:
178|         x_type: tl.constexpr = X.dtype.element_ty
179|         is_x_fp4: tl.constexpr = x_type == tl.uint8
180|         tl.static_assert(x_type == tl.float8e4nv or x_type == tl.uint8, "mx_act_ptr must be float8e4nv or uint8")
181|         # NOTE: uint8 scale means OCP E8M0 here. Direct NVFP-style scales stay float8e4nv.
182|         tl.static_assert(
183|             XMxScale.dtype.element_ty == tl.uint8
184|             or XMxScale.dtype.element_ty == tl.float8e4nv,
185|             "mx_scale_ptr must be uint8 or float8e4nv",
186|         )
187|         tl.static_assert(BLOCK_K % MX_PACK_DIVISOR == 0, "BLOCK_K must be a multiple of MX_PACK_DIVISOR")
188|     else:
189|         is_x_fp4: tl.constexpr = False
```
**EN:** Checks `is_x_microscaled`. The true branch mainly prepares intermediate values; prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `is_x_microscaled`. 真分支主要准备中间值; 准备中间值；而 else 分支准备中间值.

### Block 33 — Lines 190-190 (_matmul)
```python
190|     is_out_microscaled: tl.constexpr = stride_y_mx_z is not None
```
**EN:** Annotated assignment stores `is_out_microscaled` and evaluates `stride_y_mx_z is not None`.

**CN:** 带类型注解的赋值保存 `is_out_microscaled` and 计算 `stride_y_mx_z is not None`.

### Block 34 — Lines 191-192 (_matmul)
```python
191|     is_out_fp4: tl.constexpr = is_out_microscaled and Y_VALUE_PACK_FACTOR == 2
192| 
```
**EN:** Annotated assignment stores `is_out_fp4` and evaluates `is_out_microscaled and Y_VALUE_PACK_FACTOR == 2`.

**CN:** 带类型注解的赋值保存 `is_out_fp4` and 计算 `is_out_microscaled and Y_VALUE_PACK_FACTOR == 2`.

### Block 35 — Lines 193-200 (_matmul)
```python
193|     if _W_SLICE_SIZES_DIVISIBILITY is None:
194|         W_SLICE_SIZES_DIVISIBILITY: tl.constexpr = 1
195|     else:
196|         if PACKED_BLOCK_K_W > BLOCK_K:
197|             W_SLICE_SIZES_DIVISIBILITY: tl.constexpr =  _W_SLICE_SIZES_DIVISIBILITY * (PACKED_BLOCK_K_W // BLOCK_K)
198|         else:
199|             W_SLICE_SIZES_DIVISIBILITY: tl.constexpr =  _W_SLICE_SIZES_DIVISIBILITY // (BLOCK_K // PACKED_BLOCK_K_W)
200| 
```
**EN:** Checks `_W_SLICE_SIZES_DIVISIBILITY is None`. The true branch mainly prepares intermediate values, while the else branch branches on runtime conditions.

**CN:** 检查 `_W_SLICE_SIZES_DIVISIBILITY is None`. 真分支主要准备中间值；而 else 分支根据运行时条件分支.

### Block 36 — Lines 201-201 (_matmul)
```python
201|     OUT_BLOCK_N: tl.constexpr = BLOCK_N // ACTIVATION_REDUCTION_N
```
**EN:** Annotated assignment stores `OUT_BLOCK_N` and evaluates `BLOCK_N // ACTIVATION_REDUCTION_N`.

**CN:** 带类型注解的赋值保存 `OUT_BLOCK_N` and 计算 `BLOCK_N // ACTIVATION_REDUCTION_N`.

### Block 37 — Lines 202-203 (_matmul)
```python
202|     yN = N // ACTIVATION_REDUCTION_N
203| 
```
**EN:** Assigns `yN` and evaluates `N // ACTIVATION_REDUCTION_N`.

**CN:** 将 `yN` and 计算 `N // ACTIVATION_REDUCTION_N`.

### Block 38 — Lines 204-204 (_matmul)
```python
204|     pid = tl.program_id(0)
```
**EN:** Assigns `pid` and calls `tl.program_id`.

**CN:** 将 `pid`，并调用 `tl.program_id`.

### Block 39 — Lines 205-209 (_matmul)
```python
205|     if RAGGED_DIMENSION == "M":
206|         padding_m = grid_m - tl.load(XBlockOffs + N_EXPTS_TOT)
207|     else:
208|         padding_m: tl.constexpr = 0
209| 
```
**EN:** Checks `RAGGED_DIMENSION == 'M'`. The true branch mainly prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `RAGGED_DIMENSION == 'M'`. 真分支主要准备中间值；而 else 分支准备中间值.

### Block 40 — Lines 210-211 (_matmul)
```python
210|     index_type: tl.constexpr = tl.int64 if UPCAST_INDICES else tl.int32
211| 
```
**EN:** Annotated assignment stores `index_type` and uses conditional expression `tl.int64 if UPCAST_INDICES else tl.int32`.

**CN:** 带类型注解的赋值保存 `index_type` and 使用条件表达式 `tl.int64 if UPCAST_INDICES else tl.int32`.

### Block 41 — Lines 212-212 (_matmul)
```python
212|     unpadded_m = grid_m - padding_m
```
**EN:** Assigns `unpadded_m` and evaluates `grid_m - padding_m`.

**CN:** 将 `unpadded_m` and 计算 `grid_m - padding_m`.

### Block 42 — Lines 213-213 (_matmul)
```python
213|     tl.assume(unpadded_m >= 0)
```
**EN:** Calls `tl.assume` for side effects, registration, or validation.

**CN:** 调用 `tl.assume` ，用于副作用、注册或校验。

### Block 43 — Lines 214-215 (_matmul)
```python
214|     total_actual_tiles = batch_size * unpadded_m * grid_n * SPLIT_K
215| 
```
**EN:** Assigns `total_actual_tiles` and evaluates `batch_size * unpadded_m * grid_n * SPLIT_K`.

**CN:** 将 `total_actual_tiles` and 计算 `batch_size * unpadded_m * grid_n * SPLIT_K`.

### Block 44 — Lines 216-220 (_matmul)
```python
216|     if padding_m > 0 and pid >= total_actual_tiles:
217|         if pYPtrs is not None:
218|             all_writes_issued.fn(*all_writes_issued.captured)
219|         return
220| 
```
**EN:** Checks `padding_m > 0 and pid >= total_actual_tiles`. The true branch mainly branches on runtime conditions; returns the computed result.

**CN:** 检查 `padding_m > 0 and pid >= total_actual_tiles`. 真分支主要根据运行时条件分支; 返回计算结果.

### Block 45 — Lines 221-222 (_matmul)
```python
221|     pid_s, pid_m, pid_n, pid_k = compute_pids(pid, unpadded_m, grid_n, total_actual_tiles, XCD_SWIZZLE, GROUP_M, SPLIT_K)
222| 
```
**EN:** Assigns `pid_s`, `pid_m`, `pid_n`, `pid_k` and calls `compute_pids`.

**CN:** 将 `pid_s`, `pid_m`, `pid_n`, `pid_k`，并调用 `compute_pids`.

### Block 46 — Lines 223-233 (_matmul)
```python
223|     (
224|         expt_id, start_z, start_z_out,
225|         start_m, slice_block_off_m, off_m,
226|         off_k_x, off_k_w
227|     ) = compute_offsets(
228|             pid_s, pid_m, pid_k,
229|             XBlockSchedule, XSliceOffs, XBlockOffs, X_SLICE_SIZES_DIVISIBILITY,
230|             WBlockSchedule, WSliceOffs, W_SLICE_SIZES_DIVISIBILITY,
231|             RAGGED_DIMENSION,
232|             BLOCK_M, BLOCK_K, PACKED_BLOCK_K_W, SPLIT_K
233|         )
```
**EN:** Assigns `expt_id`, `start_z`, `start_z_out`, `start_m`, `slice_block_off_m`, `off_m` and calls `compute_offsets`.

**CN:** 将 `expt_id`, `start_z`, `start_z_out`, `start_m`, `slice_block_off_m`, `off_m`，并调用 `compute_offsets`.

### Block 47 — Lines 234-235 (_matmul)
```python
234|     if X_SLICE_SIZES_DIVISIBILITY is not None:
235|         off_k_x = off_k_x // X_SLICE_SIZES_DIVISIBILITY * X_SLICE_SIZES_DIVISIBILITY
```
**EN:** Checks `X_SLICE_SIZES_DIVISIBILITY is not None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `X_SLICE_SIZES_DIVISIBILITY is not None`. 真分支主要准备中间值.

### Block 48 — Lines 236-239 (_matmul)
```python
236|     if W_SLICE_SIZES_DIVISIBILITY is not None:
237|         off_k_w = off_k_w // W_SLICE_SIZES_DIVISIBILITY * W_SLICE_SIZES_DIVISIBILITY
238| 
239| 
```
**EN:** Checks `W_SLICE_SIZES_DIVISIBILITY is not None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `W_SLICE_SIZES_DIVISIBILITY is not None`. 真分支主要准备中间值.

### Block 49 — Lines 240-245 (_matmul)
```python
240|     if RAGGED_DIMENSION == "M":
241|         eM = tl.multiple_of(tl.load(XSliceSizes + expt_id), X_SLICE_SIZES_DIVISIBILITY)
242|     else:
243|         eM = M
244| 
245| 
```
**EN:** Checks `RAGGED_DIMENSION == 'M'`. The true branch mainly prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `RAGGED_DIMENSION == 'M'`. 真分支主要准备中间值；而 else 分支准备中间值.

### Block 50 — Lines 246-255 (_matmul)
```python
246|     if RAGGED_DIMENSION == "K":
247|         K_W = tl.multiple_of(tl.load(WSliceOffs + pid_s + 1), W_SLICE_SIZES_DIVISIBILITY)
248|         K_X = tl.multiple_of(tl.load(XSliceOffs + pid_s + 1), X_SLICE_SIZES_DIVISIBILITY)
249|     else:
250|         if SWIZZLE_MX_VALUE == "HOPPER_VALUE":
251|             # Hopper value swizzling physically pads K to a full 64-row tile.
252|             K_W = tl.cdiv(K, 64) * 64
253|         else:
254|             K_W = K
255|         K_X = K
```
**EN:** Checks `RAGGED_DIMENSION == 'K'`. The true branch mainly prepares intermediate values; prepares intermediate values, while the else branch branches on runtime conditions; prepares intermediate values.

**CN:** 检查 `RAGGED_DIMENSION == 'K'`. 真分支主要准备中间值; 准备中间值；而 else 分支根据运行时条件分支; 准备中间值.

### Block 51 — Lines 256-260 (_matmul)
```python
256|     if PACKED_BLOCK_K_W > BLOCK_K:
257|         K_W = K_W * (PACKED_BLOCK_K_W // BLOCK_K)
258|     else:
259|         K_W = K_W // (BLOCK_K // PACKED_BLOCK_K_W)
260| 
```
**EN:** Checks `PACKED_BLOCK_K_W > BLOCK_K`. The true branch mainly prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `PACKED_BLOCK_K_W > BLOCK_K`. 真分支主要准备中间值；而 else 分支准备中间值.

### Block 52 — Lines 261-261 (_matmul)
```python
261|     loop_k = tl.multiple_of(tl.load(XSliceSizes + pid_s), X_SLICE_SIZES_DIVISIBILITY) if RAGGED_DIMENSION == "K" else K - off_k_x
```
**EN:** Assigns `loop_k` and uses conditional expression `tl.multiple_of(tl.load(XSliceSizes + pid_s), X_SLICE_SIZE...`.

**CN:** 将 `loop_k` and 使用条件表达式 `tl.multiple_of(tl.load(XSliceSizes + pid_s), X_SLICE_SIZE...`.

### Block 53 — Lines 262-264 (_matmul)
```python
262|     k_tiles = tl.cdiv(loop_k, BLOCK_K * SPLIT_K)
263| 
264|     # For split-k, advance to the output k slice
```
**EN:** Assigns `k_tiles` and calls `tl.cdiv`.

**CN:** 将 `k_tiles`，并调用 `tl.cdiv`.

### Block 54 — Lines 265-269 (_matmul)
```python
265|     if SPLIT_K > 1:
266|         Y += pid_k.to( index_type) * stride_y_k
267|         if is_out_microscaled:
268|             YActualScale += pid_k.to(index_type) * stride_y_mx_k
269| 
```
**EN:** Checks `SPLIT_K > 1`. The true branch mainly branches on runtime conditions.

**CN:** 检查 `SPLIT_K > 1`. 真分支主要根据运行时条件分支.

### Block 55 — Lines 270-270 (_matmul)
```python
270|     expt_id, off_m = expt_id.to(index_type), off_m.to(index_type)
```
**EN:** Assigns `expt_id`, `off_m` and builds a tuple.

**CN:** 将 `expt_id`, `off_m` and 构造一个元组.

### Block 56 — Lines 271-271 (_matmul)
```python
271|     start_m, start_z = start_m.to(index_type), start_z.to(index_type)
```
**EN:** Assigns `start_m`, `start_z` and builds a tuple.

**CN:** 将 `start_m`, `start_z` and 构造一个元组.

### Block 57 — Lines 272-273 (_matmul)
```python
272|     pid_n, pid_k = pid_n.to(index_type), pid_k.to(index_type)
273|     # A pointers
```
**EN:** Assigns `pid_n`, `pid_k` and builds a tuple.

**CN:** 将 `pid_n`, `pid_k` and 构造一个元组.

### Block 58 — Lines 274-274 (_matmul)
```python
274|     offs_x_m = off_m + tl.arange(0, BLOCK_M)
```
**EN:** Assigns `offs_x_m` and evaluates `off_m + tl.arange(0, BLOCK_M)`.

**CN:** 将 `offs_x_m` and 计算 `off_m + tl.arange(0, BLOCK_M)`.

### Block 59 — Lines 275-275 (_matmul)
```python
275|     offs_x_m = tl.max_contiguous(tl.multiple_of(offs_x_m % eM, BLOCK_M), BLOCK_M)
```
**EN:** Assigns `offs_x_m` and calls `tl.max_contiguous`.

**CN:** 将 `offs_x_m`，并调用 `tl.max_contiguous`.

### Block 60 — Lines 276-276 (_matmul)
```python
276|     X += start_z * stride_x_z
```
**EN:** Updates `X` with operator `Add` using `start_z * stride_x_z`.

**CN:** 更新 `X`，使用运算符 `Add`，并使用 `start_z * stride_x_z`.

### Block 61 — Lines 277-282 (_matmul)
```python
277|     if GatherIndx is None:
278|         X += start_m * stride_x_m
279|     else:
280|         GatherIndx += start_m
281|         # no needs to bounds-check here because `offs_x_m` wraps around M dim
282|         offs_x_m = tl.load(GatherIndx + offs_x_m)
```
**EN:** Checks `GatherIndx is None`., while the else branch prepares intermediate values.

**CN:** 检查 `GatherIndx is None`.；而 else 分支准备中间值.

### Block 62 — Lines 283-286 (_matmul)
```python
283|     if is_x_fp4:
284|         offs_k_x = off_k_x // 2 + tl.arange(0, BLOCK_K // 2)
285|     else:
286|         offs_k_x = off_k_x + tl.arange(0, BLOCK_K)
```
**EN:** Checks `is_x_fp4`. The true branch mainly prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `is_x_fp4`. 真分支主要准备中间值；而 else 分支准备中间值.

### Block 63 — Lines 287-290 (_matmul)
```python
287|     XPtrs = X + offs_x_m.to(index_type)[:, None] * stride_x_m + offs_k_x.to(index_type)[None, :] * stride_x_k
288| 
289| 
290|     # TODO: refactor if/else when triton front end improves
```
**EN:** Assigns `XPtrs` and evaluates `X + offs_x_m.to(index_type)[:, None] * stride_x_m + offs_k_x.to(ind...`.

**CN:** 将 `XPtrs` and 计算 `X + offs_x_m.to(index_type)[:, None] * stride_x_m + offs_k_x.to(ind...`.

### Block 64 — Lines 291-339 (_matmul)
```python
291|     if is_w_microscaled:
292|         WMxScale += expt_id * stride_w_mx_e
293| 
294|         if SWIZZLE_MX_SCALE == "BLACKWELL_SCALE":
295|             # TODO: support non W_TRANSPOSE with blackwell swizzling
296|             tl.static_assert(W_TRANSPOSE)
297|             tl.static_assert(BLOCK_N % 128 == 0)
298|             tl.static_assert(MX_SCALE_BLOCK_K % 4 == 0)
299|             PACKED_MX_BLOCK: tl.constexpr = (MX_SCALE_BLOCK_K // 4) * 32 * 4 * 4
300|             SCALE_BLOCK_N: tl.constexpr = BLOCK_N // 128
301|             stride_scale_k: tl.constexpr = 1
302|         elif SWIZZLE_MX_SCALE == "HOPPER_SCALE":
303|             # TODO: support non W_TRANSPOSE with Hopper swizzling
304|             tl.static_assert(W_TRANSPOSE)
305|             n_warps: tl.constexpr = tl.extra.cuda.num_warps()
306|             tl.static_assert(n_warps == 8 or n_warps == 4)
307|             tl.static_assert(BLOCK_N % (2 * n_warps * 2 * 8) == 0)
308|             tl.static_assert(MX_SCALE_BLOCK_K % 2 == 0)
309|             PACKED_MX_BLOCK: tl.constexpr = MX_SCALE_BLOCK_K * 32
310|             SCALE_BLOCK_N: tl.constexpr = BLOCK_N // 32
311|             stride_scale_k = stride_w_mx_k
312|         elif SWIZZLE_MX_SCALE == "CDNA4_SCALE":
313|             tl.static_assert(stride_w_mx_k is not None)
314|             tl.static_assert(stride_w_mx_n is not None)
315|             NON_K_PRESHUFFLE_BLOCK_SIZE: tl.constexpr = 32
316|             PACKED_MX_BLOCK: tl.constexpr = MX_SCALE_BLOCK_K * NON_K_PRESHUFFLE_BLOCK_SIZE
317|             SCALE_BLOCK_N: tl.constexpr = BLOCK_N // NON_K_PRESHUFFLE_BLOCK_SIZE
318|             stride_scale_k = stride_w_mx_k
319|         elif SWIZZLE_MX_SCALE == "GFX1250_SCALE":
320|             tl.static_assert(stride_w_mx_k is not None)
321|             tl.static_assert(stride_w_mx_n is not None)
322|             NON_K_PRESHUFFLE_BLOCK_SIZE: tl.constexpr = 128
323|             PACKED_MX_BLOCK: tl.constexpr = MX_SCALE_BLOCK_K * NON_K_PRESHUFFLE_BLOCK_SIZE
324|             SCALE_BLOCK_N: tl.constexpr = BLOCK_N // NON_K_PRESHUFFLE_BLOCK_SIZE
325|             stride_scale_k = stride_w_mx_k
326|         else:
327|             PACKED_MX_BLOCK: tl.constexpr = MX_SCALE_BLOCK_K
328|             SCALE_BLOCK_N: tl.constexpr = BLOCK_N
329|             stride_scale_k = stride_w_mx_k
330|         offs_n_scale = (pid_n * SCALE_BLOCK_N + tl.arange(0, SCALE_BLOCK_N)) % N
331|         offs_n_scale = tl.max_contiguous(tl.multiple_of(offs_n_scale, SCALE_BLOCK_N), SCALE_BLOCK_N)
332|         # K dimension must be the last dimension for the scales
333|         offs_k_scale = off_k_w // PACKED_BLOCK_K_W * PACKED_MX_BLOCK + tl.arange(0, PACKED_MX_BLOCK)
334|         WMxScalePtrs = WMxScale + offs_k_scale.to(index_type)[None, :] * stride_scale_k + offs_n_scale.to(index_type)[:, None] * stride_w_mx_n
335|     else:
336|         WMxScalePtrs = None
337|         offs_k_scale = None
338| 
339|     # B pointers
```
**EN:** Checks `is_w_microscaled`. The true branch mainly branches on runtime conditions; prepares intermediate values, while the else branch prepares intermediate values; prepares intermediate values.

**CN:** 检查 `is_w_microscaled`. 真分支主要根据运行时条件分支; 准备中间值；而 else 分支准备中间值; 准备中间值.

### Block 65 — Lines 340-340 (_matmul)
```python
340|     offs_w_n = pid_n * PACKED_BLOCK_N_W + tl.arange(0, PACKED_BLOCK_N_W)
```
**EN:** Assigns `offs_w_n` and evaluates `pid_n * PACKED_BLOCK_N_W + tl.arange(0, PACKED_BLOCK_N_W)`.

**CN:** 将 `offs_w_n` and 计算 `pid_n * PACKED_BLOCK_N_W + tl.arange(0, PACKED_BLOCK_N_W)`.

### Block 66 — Lines 341-341 (_matmul)
```python
341|     N_W = N
```
**EN:** Assigns `N_W` and references `N`.

**CN:** 将 `N_W` and 引用 `N`.

### Block 67 — Lines 342-343 (_matmul)
```python
342|     if SWIZZLE_MX_VALUE == "HOPPER_VALUE":
343|         N_W = tl.cdiv(N_W, 64) * 64
```
**EN:** Checks `SWIZZLE_MX_VALUE == 'HOPPER_VALUE'`. The true branch mainly prepares intermediate values.

**CN:** 检查 `SWIZZLE_MX_VALUE == 'HOPPER_VALUE'`. 真分支主要准备中间值.

### Block 68 — Lines 344-345 (_matmul)
```python
344|     offs_w_n = tl.max_contiguous(tl.multiple_of(offs_w_n % (N_W // W_N_DIVISOR), PACKED_BLOCK_N_W), PACKED_BLOCK_N_W)
345| 
```
**EN:** Assigns `offs_w_n` and calls `tl.max_contiguous`.

**CN:** 将 `offs_w_n`，并调用 `tl.max_contiguous`.

### Block 69 — Lines 346-354 (_matmul)
```python
346|     if is_x_microscaled:
347|         XMxScale += start_z.to(index_type) * stride_x_mx_z
348|         if GatherIndx is None:
349|             XMxScale += start_m * stride_x_mx_m
350|         offs_x_k_scale = off_k_x // MX_BLOCK_SIZE + tl.arange(0, MX_SCALE_BLOCK_K)
351|         XMxScalePtrs = XMxScale + offs_x_m.to(index_type)[:, None] * stride_x_mx_m + offs_x_k_scale.to(index_type)[None, :] * stride_x_mx_k
352|     else:
353|         XMxScalePtrs = None
354| 
```
**EN:** Checks `is_x_microscaled`. The true branch mainly branches on runtime conditions; prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `is_x_microscaled`. 真分支主要根据运行时条件分支; 准备中间值；而 else 分支准备中间值.

### Block 70 — Lines 355-355 (_matmul)
```python
355|     offs_w_k = off_k_w + tl.arange(0, PACKED_BLOCK_K_W)
```
**EN:** Assigns `offs_w_k` and evaluates `off_k_w + tl.arange(0, PACKED_BLOCK_K_W)`.

**CN:** 将 `offs_w_k` and 计算 `off_k_w + tl.arange(0, PACKED_BLOCK_K_W)`.

### Block 71 — Lines 356-356 (_matmul)
```python
356|     W += expt_id * stride_w_e
```
**EN:** Updates `W` with operator `Add` using `expt_id * stride_w_e`.

**CN:** 更新 `W`，使用运算符 `Add`，并使用 `expt_id * stride_w_e`.

### Block 72 — Lines 357-358 (_matmul)
```python
357|     WPtrs = W + (offs_w_k.to(index_type)[:, None] * stride_w_k + offs_w_n.to(index_type)[None, :] * stride_w_n)
358|     # compute output
```
**EN:** Assigns `WPtrs` and evaluates `W + (offs_w_k.to(index_type)[:, None] * stride_w_k + offs_w_n.to(in...`.

**CN:** 将 `WPtrs` and 计算 `W + (offs_w_k.to(index_type)[:, None] * stride_w_k + offs_w_n.to(in...`.

### Block 73 — Lines 359-359 (_matmul)
```python
359|     acc = tl.zeros((BLOCK_N, BLOCK_M) if SWAP_XW else (BLOCK_M, BLOCK_N), dtype=tl.float32)
```
**EN:** Assigns `acc` and calls `tl.zeros`.

**CN:** 将 `acc`，并调用 `tl.zeros`.

### Block 74 — Lines 360-360 (_matmul)
```python
360|     x_k_limit = K_X + BLOCK_K * SPLIT_K
```
**EN:** Assigns `x_k_limit` and evaluates `K_X + BLOCK_K * SPLIT_K`.

**CN:** 将 `x_k_limit` and 计算 `K_X + BLOCK_K * SPLIT_K`.

### Block 75 — Lines 361-362 (_matmul)
```python
361|     w_k_limit = K_W + PACKED_BLOCK_K_W * SPLIT_K
362| 
```
**EN:** Assigns `w_k_limit` and evaluates `K_W + PACKED_BLOCK_K_W * SPLIT_K`.

**CN:** 将 `w_k_limit` and 计算 `K_W + PACKED_BLOCK_K_W * SPLIT_K`.

### Block 76 — Lines 363-472 (_matmul)
```python
363|     for ki in range(k_tiles):
364|         x_k_limit -= BLOCK_K * SPLIT_K
365|         w_k_limit -= PACKED_BLOCK_K_W * SPLIT_K
366|         if EVEN_K:
367|             if is_x_fp4:
368|                 mask_k_x = tl.full([BLOCK_K // 2], True, dtype=tl.int1)
369|             else:
370|                 mask_k_x = tl.full([BLOCK_K], True, dtype=tl.int1)
371|             mask_k_w = tl.full([PACKED_BLOCK_K_W], True, dtype=tl.int1)
372|             if is_w_microscaled and SWIZZLE_MX_SCALE == "STRIDED":
373|                 mask_k_scale = tl.full([PACKED_MX_BLOCK], True, dtype=tl.int1)
374|             if is_x_microscaled:
375|                 mask_x_k_scale = tl.full([MX_SCALE_BLOCK_K], True, dtype=tl.int1)
376|         else:
377|             if is_x_fp4:
378|                 mask_k_x = offs_k_x * 2 < x_k_limit
379|             else:
380|                 mask_k_x = offs_k_x < x_k_limit
381|             mask_k_w = offs_w_k < w_k_limit
382|             if is_w_microscaled and SWIZZLE_MX_SCALE == "STRIDED":
383|                 # dividing by W_K_DIVISOR because w_k_limit is also already
384|                 # divided by W_K_DIVISOR (2 for mxfp4 wehre 2 fp4 values are
385|                 # packed per Byte along K)
386|                 mask_k_scale = offs_k_scale * (MX_PACK_DIVISOR // W_K_DIVISOR) < w_k_limit
387|             if is_x_microscaled:
388|                 mask_x_k_scale = offs_x_k_scale * MX_PACK_DIVISOR < x_k_limit
389| 
390|         x = tl.load(XPtrs, mask=mask_k_x[None, :], other=0.0)
391|         w = tl.load(WPtrs, mask=mask_k_w[:, None], other=0.0, cache_modifier=W_CACHE_MODIFIER)
392|         if cuda_capability_geq(8, 0):
393|             if x.dtype == tl.float32 and ALLOW_TF32:
394|                 x = round_f32_to_tf32(x)
395|             if w.dtype == tl.float32 and ALLOW_TF32:
396|                 w = round_f32_to_tf32(w)
397|         if is_x_microscaled or is_w_microscaled:
398|             x_format: tl.constexpr = get_scaled_dot_format_string(x.dtype)
399|             w_format: tl.constexpr = get_scaled_dot_format_string(w.dtype)
400| 
401|             if is_x_microscaled:
402|                 x_scales = tl.load(XMxScalePtrs, mask=mask_x_k_scale[None, :], other=0.0)
403|             elif x_format == "fp16" or x_format == "bf16":
404|                 x_scales: tl.constexpr = None
405|             else:
406|                 if WMxScale.dtype.element_ty == tl.uint8:
407|                     x_scales = tl.full((BLOCK_M, MX_SCALE_BLOCK_K), 127, dtype=tl.uint8)
408|                 else:
409|                     x_scales = tl.full((BLOCK_M, MX_SCALE_BLOCK_K), 1.0, dtype=tl.float8e4nv)
410| 
411|             if is_w_microscaled:
412|                 if SWIZZLE_MX_SCALE == "BLACKWELL_SCALE":
413|                     w_scales = unswizzle_mx_scale_bw(tl.load(WMxScalePtrs))
414|                 elif SWIZZLE_MX_SCALE == "HOPPER_SCALE":
415|                     # Handshake with the swizzling code
416|                     num_warps: tl.constexpr = tl.extra.cuda.num_warps()
417|                     tl.static_assert(num_warps == 8 or num_warps == 4)
418|                     w_scales = unswizzle_mxfp4_scale_hopper(tl.load(WMxScalePtrs), mx_axis=1, num_warps=num_warps)
419|                     mask_k_scale = off_k_x + tl.arange(0, MX_SCALE_BLOCK_K) * MX_PACK_DIVISOR < x_k_limit
420|                     scale_zero = tl.full(w_scales.shape, 0, dtype=w_scales.dtype)
421|                     w_scales = tl.where(mask_k_scale[None, :], w_scales, scale_zero)
422|                 elif SWIZZLE_MX_SCALE == "CDNA4_SCALE":
423|                     w_scales = unswizzle_mx_scale_cdna4(tl.load(WMxScalePtrs), BLOCK_N, MX_SCALE_BLOCK_K)
424|                 elif SWIZZLE_MX_SCALE == "GFX1250_SCALE":
425|                     w_scales = unswizzle_mx_scale_gfx1250(tl.load(WMxScalePtrs), BLOCK_N, MX_SCALE_BLOCK_K)
426|                 else:
427|                     w_scales = tl.load(WMxScalePtrs, mask=mask_k_scale[None, :], other=0.0)
428|             else:
429|                 w_scales: tl.constexpr = None
430| 
431|             if is_x_fp4 and not is_w_microscaled and not cuda_capability_geq(10, 0):
432|                 tl.static_assert(w_format == "fp16" or w_format == "bf16")
433|                 x_dense = upcast_mxfp4_tile(x, x_scales, w.dtype)
434|                 if SWAP_XW:
435|                     acc = tl.dot(w.T, x_dense.T, acc, max_num_imprecise_acc=MAX_NUM_IMPRECISE_ACC, allow_tf32=ALLOW_TF32)
436|                 else:
437|                     acc = tl.dot(x_dense, w, acc, max_num_imprecise_acc=MAX_NUM_IMPRECISE_ACC, allow_tf32=ALLOW_TF32)
438|             elif SWIZZLE_MX_VALUE == "HOPPER_VALUE":
439|                 # Handshake with the swizzling code
440|                 tl.static_assert(w_format == "e2m1")
441|                 tl.static_assert(SWAP_XW)
442|                 wT = mxfp4_to_bf16_triton(w.T, w_scales, mx_axis=1)
443|                 tl.static_assert(wT.dtype == tl.bfloat16)
444|                 if is_x_microscaled:
445|                     acc = tl.dot_scaled(wT, None, "bf16", x.T, x_scales, x_format, acc=acc, fast_math=True)
446|                 else:
447|                     tl.static_assert(x_format == "bf16")
448|                     acc = tl.dot(wT, x.T, acc, max_num_imprecise_acc=MAX_NUM_IMPRECISE_ACC, allow_tf32=ALLOW_TF32)
449|             else:
450|                 rhs_k_pack: tl.constexpr = W_TRANSPOSE or not is_w_microscaled or W_K_DIVISOR != 2
451|                 if SWAP_XW:
452|                     acc = tl.dot_scaled(w.T, w_scales, w_format, x.T, x_scales, x_format, acc=acc, fast_math=True)
453|                 else:
454|                     acc = tl.dot_scaled(x, x_scales, x_format, w, w_scales, w_format, acc=acc, fast_math=True, rhs_k_pack=rhs_k_pack)
455|             if is_w_microscaled:
456|                 if SWIZZLE_MX_SCALE == "BLACKWELL_SCALE":
457|                     WMxScalePtrs += (MX_SCALE_BLOCK_K // 4 * SPLIT_K) * stride_w_mx_k
458|                 else:
459|                     WMxScalePtrs += (PACKED_MX_BLOCK * SPLIT_K) * stride_w_mx_k
460|             if is_x_microscaled:
461|                 XMxScalePtrs += (MX_SCALE_BLOCK_K * SPLIT_K) * stride_x_mx_k
462|         else:
463|             if SWAP_XW:
464|                 acc = tl.dot(w.T, x.T, acc, max_num_imprecise_acc=MAX_NUM_IMPRECISE_ACC, allow_tf32=ALLOW_TF32)
465|             else:
466|                 acc = tl.dot(x, w, acc, max_num_imprecise_acc=MAX_NUM_IMPRECISE_ACC, allow_tf32=ALLOW_TF32)
467|         if is_x_fp4:
468|             XPtrs += ((BLOCK_K // 2) * SPLIT_K) * stride_x_k
469|         else:
470|             XPtrs += (BLOCK_K * SPLIT_K) * stride_x_k
471|         WPtrs += (PACKED_BLOCK_K_W * SPLIT_K) * stride_w_k
472|     # bias + scale
```
**EN:** Loops over `range(k_tiles)` with target `ki`. The loop body mainly branches on runtime conditions; prepares intermediate values.

**CN:** 遍历 `range(k_tiles)` ，目标变量为 `ki`. 循环体主要根据运行时条件分支; 准备中间值.

### Block 77 — Lines 473-473 (_matmul)
```python
473|     offs_m = off_m + tl.arange(0, BLOCK_M)
```
**EN:** Assigns `offs_m` and evaluates `off_m + tl.arange(0, BLOCK_M)`.

**CN:** 将 `offs_m` and 计算 `off_m + tl.arange(0, BLOCK_M)`.

### Block 78 — Lines 474-474 (_matmul)
```python
474|     offs_y_n = BLOCK_N * pid_n + tl.arange(0, BLOCK_N)
```
**EN:** Assigns `offs_y_n` and evaluates `BLOCK_N * pid_n + tl.arange(0, BLOCK_N)`.

**CN:** 将 `offs_y_n` and 计算 `BLOCK_N * pid_n + tl.arange(0, BLOCK_N)`.

### Block 79 — Lines 475-475 (_matmul)
```python
475|     mask_m = offs_m < eM
```
**EN:** Assigns `mask_m` and evaluates `offs_m < eM`.

**CN:** 将 `mask_m` and 计算 `offs_m < eM`.

### Block 80 — Lines 476-476 (_matmul)
```python
476|     mask_n = offs_y_n < N
```
**EN:** Assigns `mask_n` and evaluates `offs_y_n < N`.

**CN:** 将 `mask_n` and 计算 `offs_y_n < N`.

### Block 81 — Lines 477-484 (_matmul)
```python
477|     if B is not None:
478|         BPtrs = B + expt_id * stride_b_e + offs_y_n
479|         if pid_k == 0:
480|             bias = tl.load(BPtrs, mask=mask_n, other=0)
481|         else:
482|             bias = tl.full([BLOCK_N], 0, dtype=tl.float32)
483|     else:
484|         bias = tl.full([BLOCK_N], 0, dtype=tl.float32)
```
**EN:** Checks `B is not None`. The true branch mainly prepares intermediate values; branches on runtime conditions, while the else branch prepares intermediate values.

**CN:** 检查 `B is not None`. 真分支主要准备中间值; 根据运行时条件分支；而 else 分支准备中间值.

### Block 82 — Lines 485-488 (_matmul)
```python
485|     if Betas is not None:
486|         betas = tl.load(Betas + start_m + offs_m, mask=mask_m, other=0.0)
487|     else:
488|         betas = tl.full([BLOCK_M], 1, dtype=tl.float32)
```
**EN:** Checks `Betas is not None`. The true branch mainly prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `Betas is not None`. 真分支主要准备中间值；而 else 分支准备中间值.

### Block 83 — Lines 489-493 (_matmul)
```python
489|     if Gammas is not None:
490|         gammas = tl.load(Gammas + start_m + offs_m, mask=mask_m, other=0.0)
491|     else:
492|         gammas = tl.full([BLOCK_M], 1, dtype=tl.float32)
493|     # flexpoint
```
**EN:** Checks `Gammas is not None`. The true branch mainly prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `Gammas is not None`. 真分支主要准备中间值；而 else 分支准备中间值.

### Block 84 — Lines 494-494 (_matmul)
```python
494|     x_scale = load_scale(XScale)
```
**EN:** Assigns `x_scale` and calls `load_scale`.

**CN:** 将 `x_scale`，并调用 `load_scale`.

### Block 85 — Lines 495-498 (_matmul)
```python
495|     if PER_BATCH_W_SCALE:
496|         w_scale = load_scale(WScale + expt_id)
497|     else:
498|         w_scale = load_scale(WScale)
```
**EN:** Checks `PER_BATCH_W_SCALE`. The true branch mainly prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `PER_BATCH_W_SCALE`. 真分支主要准备中间值；而 else 分支准备中间值.

### Block 86 — Lines 499-500 (_matmul)
```python
499|     if SWAP_XW:
500|         acc = acc.trans()
```
**EN:** Checks `SWAP_XW`. The true branch mainly prepares intermediate values.

**CN:** 检查 `SWAP_XW`. 真分支主要准备中间值.

### Block 87 — Lines 501-501 (_matmul)
```python
501|     acc *= x_scale * w_scale
```
**EN:** Updates `acc` with operator `Mult` using `x_scale * w_scale`.

**CN:** 更新 `acc`，使用运算符 `Mult`，并使用 `x_scale * w_scale`.

### Block 88 — Lines 502-502 (_matmul)
```python
502|     acc = acc + bias[None, :] * betas[:, None]
```
**EN:** Assigns `acc` and evaluates `acc + bias[None, :] * betas[:, None]`.

**CN:** 将 `acc` and 计算 `acc + bias[None, :] * betas[:, None]`.

### Block 89 — Lines 503-504 (_matmul)
```python
503|     if out_alpha is not None:
504|         acc *= out_alpha
```
**EN:** Checks `out_alpha is not None`..

**CN:** 检查 `out_alpha is not None`..

### Block 90 — Lines 505-512 (_matmul)
```python
505|     if ACTIVATION_FN is not None:
506|         out = ACTIVATION_FN(acc, *activation_fn_args)
507|         tl.static_assert(out.shape[1] == OUT_BLOCK_N, f"Activation fn out.shape[1] ({out.shape[1]}) doesn't match computed OUT_BLOCK_N ({OUT_BLOCK_N})")
508|         offs_y_n = OUT_BLOCK_N * pid_n + tl.arange(0, OUT_BLOCK_N)
509|         mask_n = offs_y_n < yN
510|     else:
511|         tl.static_assert(ACTIVATION_REDUCTION_N == 1, "Activation reduction must be 1 if no activation fn is provided")
512|         out = acc
```
**EN:** Checks `ACTIVATION_FN is not None`. The true branch mainly prepares intermediate values; invokes `tl.static_assert`, while the else branch invokes `tl.static_assert`; prepares intermediate values.

**CN:** 检查 `ACTIVATION_FN is not None`. 真分支主要准备中间值; invokes `tl.static_assert`；而 else 分支invokes `tl.static_assert`; 准备中间值.

### Block 91 — Lines 513-514 (_matmul)
```python
513|     out *= gammas[:, None]
514|     # write-back
```
**EN:** Updates `out` with operator `Mult` using `gammas[:, None]`.

**CN:** 更新 `out`，使用运算符 `Mult`，并使用 `gammas[:, None]`.

### Block 92 — Lines 515-515 (_matmul)
```python
515|     Y += start_z_out.to(index_type) * stride_y_z
```
**EN:** Updates `Y` with operator `Add` using `start_z_out.to(index_type) * stride_y_z`.

**CN:** 更新 `Y`，使用运算符 `Add`，并使用 `start_z_out.to(index_type) * stride_y_z`.

### Block 93 — Lines 516-524 (_matmul)
```python
516|     if WriteBackIndx is not None:
517|         WriteBackIndx += start_m
518|         dst_idx = tl.load(WriteBackIndx + offs_m, mask=start_m + offs_m < writeback_size, other=-1)
519|         mask_m = mask_m & (dst_idx != -1)
520|         offs_y_m = dst_idx
521|     else:
522|         Y += start_m * stride_y_m
523|         offs_y_m = offs_m
524| 
```
**EN:** Checks `WriteBackIndx is not None`. The true branch mainly prepares intermediate values; prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `WriteBackIndx is not None`. 真分支主要准备中间值; 准备中间值；而 else 分支准备中间值.

### Block 94 — Lines 525-532 (_matmul)
```python
525|     if is_out_fp4:
526|         tl.static_assert(Y_TMA_MODE is None, "FP4 outputs are only supported without output TMA")
527|         offs_y_n_store = pid_n * (OUT_BLOCK_N // 2) + tl.arange(0, OUT_BLOCK_N // 2)
528|         YPtrs = Y + offs_y_m.to(index_type)[:, None] * stride_y_m + offs_y_n_store.to(index_type)[None, :] * stride_y_n
529|         mask_store = mask_m[:, None] if OUT_N_TILE_ALIGNED else mask_m[:, None] & (offs_y_n_store[None, :] < tl.cdiv(yN, 2))
530|     else:
531|         YPtrs = Y + offs_y_m.to(index_type)[:, None] * stride_y_m + offs_y_n.to(index_type)[None, :] * stride_y_n
532|         mask_store = mask_m[:, None] if OUT_N_TILE_ALIGNED else mask_m[:, None] & mask_n[None, :]
```
**EN:** Checks `is_out_fp4`. The true branch mainly invokes `tl.static_assert`; prepares intermediate values, while the else branch prepares intermediate values; prepares intermediate values.

**CN:** 检查 `is_out_fp4`. 真分支主要invokes `tl.static_assert`; 准备中间值；而 else 分支准备中间值; 准备中间值.

### Block 95 — Lines 533-534 (_matmul)
```python
533|     mask = mask_m[:, None] if OUT_N_TILE_ALIGNED else mask_m[:, None] & mask_n[None, :]
534| 
```
**EN:** Assigns `mask` and uses conditional expression `mask_m[:, None] if OUT_N_TILE_ALIGNED else mask_m[:, None...`.

**CN:** 将 `mask` and 使用条件表达式 `mask_m[:, None] if OUT_N_TILE_ALIGNED else mask_m[:, None...`.

### Block 96 — Lines 535-546 (_matmul)
```python
535|     if OutAcc is not None:
536|         if PER_BATCH_ACC_SCALE:
537|             ScalePtr = OutAccScale + start_z_out
538|         else:
539|             ScalePtr = OutAccScale
540| 
541|         if Y_ACC_IS_Y:
542|             AccPtrs = YPtrs
543|         else:
544|             AccPtrs = OutAcc + start_z_out.to(index_type) * stride_acc_z + offs_y_m.to(index_type)[:, None] * stride_acc_m + offs_y_n.to(index_type)[None, :] * stride_acc_n
545|         out += tl.load(AccPtrs, mask=mask, other=0.0) * load_scale(ScalePtr)
546| 
```
**EN:** Checks `OutAcc is not None`. The true branch mainly branches on runtime conditions; branches on runtime conditions.

**CN:** 检查 `OutAcc is not None`. 真分支主要根据运行时条件分支; 根据运行时条件分支.

### Block 97 — Lines 547-595 (_matmul)
```python
547|     if is_out_microscaled:
548|         MX_SCALE_BLOCK_N: tl.constexpr = OUT_BLOCK_N // MX_BLOCK_SIZE
549|         N_MX_BLOCK = tl.cdiv(N, MX_BLOCK_SIZE)
550|         tl.static_assert(EPILOGUE_FN is not None)
551|         if PER_BATCH_OUT_SCALE:
552|             YExpectedScale = YExpectedScale + start_z_out
553|         # OCP MX outputs leave YExpectedScale unset, so this is an identity there.
554|         # NVFP4 uses YExpectedScale to precondition the dense output before the
555|         # microscaling epilogue writes direct e4m3 block scales.
556|         out = float_to_flex(out, YExpectedScale, None, None, mask, Y, False)
557|         out, out_scale = EPILOGUE_FN(out, mask, *epilogue_fn_args)
558|         tl.static_assert(BLOCK_N % MX_SCALE_BLOCK_N == 0, "")
559|         offs_y_n_scale = MX_SCALE_BLOCK_N * pid_n + tl.arange(0, MX_SCALE_BLOCK_N)
560|         output_scale_rows = offs_y_m if WriteBackIndx is not None else start_m + offs_y_m
561|         YActualScalePtrs = output_mx_scale_store_ptr(
562|             YActualScale,
563|             offs_m,
564|             output_scale_rows,
565|             offs_y_n_scale,
566|             start_z_out,
567|             start_m,
568|             M,
569|             N_MX_BLOCK,
570|             XOutputScaleBlockOffs,
571|             expt_id,
572|             pid_k - pid_k,
573|             pid_k - pid_k,
574|             batch_size,
575|             stride_y_mx_k,
576|             stride_y_mx_z,
577|             stride_y_mx_m,
578|             stride_y_mx_n,
579|             WriteBackIndx is not None,
580|             False,
581|             Y_TMA_MODE,
582|             RAGGED_DIMENSION,
583|             Y_MX_SCALE_LAYOUT,
584|             INDEX_TYPE=index_type,
585|         )
586|         mask_n_scale = offs_y_n_scale < N_MX_BLOCK
587|         scale_store_mask = mask_m[:, None] if OUT_N_TILE_ALIGNED else mask_m[:, None] & mask_n_scale[None, :]
588|         tl.store(YActualScalePtrs, out_scale, mask=scale_store_mask)
589|     else:
590|         if PER_BATCH_OUT_SCALE:
591|             YExpectedScale = YExpectedScale + start_z_out
592|             YActualScale = YActualScale + start_z_out
593|         out = float_to_flex(out, YExpectedScale, YActualScale, YChecksumScale, mask, Y, FLEXPOINT_SATURATE_INF)
594|         if EPILOGUE_FN is not None and not IS_EPILOGUE_QUANT_MX:
595|             out = EPILOGUE_FN(out, *epilogue_fn_args, target_dtype=YPtrs.dtype.element_ty)
```
**EN:** Checks `is_out_microscaled`. The true branch mainly prepares intermediate values; prepares intermediate values, while the else branch branches on runtime conditions; prepares intermediate values.

**CN:** 检查 `is_out_microscaled`. 真分支主要准备中间值; 准备中间值；而 else 分支根据运行时条件分支; 准备中间值.

### Block 98 — Lines 596-621 (_matmul)
```python
596|     if pYPtrs is None:
597|         tl.store(YPtrs, out, mask=mask_store)
598|     else:
599|         tl.static_assert(not is_out_fp4, "FP4 outputs are not supported with fused comms")
600|         tl.static_assert(Y_TMA_MODE is None, "TMA is not supported with fused comms")
601|         dst_shard_idx, dst_y_m, dst_y_n = map_dst_coord.fn(
602|             off_m if WriteBackIndx is None else None, offs_y_m,
603|             OUT_BLOCK_N * pid_n, offs_y_n,
604|             *map_dst_coord.captured)
605|         offs_mn = (
606|             dst_y_m.to(index_type)[:, None] * stride_y_m * n_reduce_shards + reduce_rank * stride_y_m +
607|             dst_y_n[None, :] * stride_y_n
608|         )
609|         for i in tl.static_range(n_reduce_shards):
610|             if dst_shard_idx is not None:
611|                 peer = dst_shard_idx * n_reduce_shards + (reduce_rank + i) % n_reduce_shards
612|             else:
613|                 peer = (reduce_rank + i) % n_reduce_shards
614|             peer_Y_ptr = tl.load(pYPtrs + peer).to(tl.pointer_type(YPtr.type.element_ty))
615|             if len(peer_Y_ptr.shape) == 0:
616|                 tl.multiple_of(peer_Y_ptr, 16)
617|             else:
618|                 tl.multiple_of(peer_Y_ptr, [16, 16])
619|             tl.store(peer_Y_ptr + offs_mn, out, mask=mask)
620| 
621| 
```
**EN:** Checks `pYPtrs is None`. The true branch mainly invokes `tl.store`, while the else branch invokes `tl.static_assert`; invokes `tl.static_assert`.

**CN:** 检查 `pYPtrs is None`. 真分支主要invokes `tl.store`；而 else 分支invokes `tl.static_assert`; invokes `tl.static_assert`.

### Block 99 — Lines 622-623 (_matmul)
```python
622|     if pYPtrs is not None:
623|         all_writes_issued.fn(*all_writes_issued.captured)
```
**EN:** Checks `pYPtrs is not None`. The true branch mainly invokes `all_writes_issued.fn`.

**CN:** 检查 `pYPtrs is not None`. 真分支主要invokes `all_writes_issued.fn`.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `round_f32_to_tf32`, `_matmul`.
  **CN:** 主要符号：`round_f32_to_tf32`, `_matmul`。
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
- **EN:** External modules: `triton`, `triton.language`.
  **CN:** 外部模块：`triton`, `triton.language`。
- **EN:** Internal modules: `triton_kernels.tensor_details.layout_details.blackwell_scale (unswizzle_mx_scale_bw)`, `triton_kernels.tensor_details.layout_details.hopper_scale (unswizzle_mxfp4_scale_hopper)`, `triton_kernels.tensor_details.layout_details.hopper_value (mxfp4_to_bf16_triton)`, `triton_kernels.tensor_details.layout_details.cdna4_scale (unswizzle_mx_scale_cdna4)`, `triton_kernels.tensor_details.layout_details.gfx1250_scale (unswizzle_mx_scale_gfx1250)`, `triton_kernels.numerics_details.flexpoint (float_to_flex, load_scale)`, `triton_kernels.numerics_details.mxfp_details._downcast_to_mxfp (MXFP_BLOCK_SIZE, NVFP_BLOCK_SIZE)`, `triton_kernels.numerics_details.mxfp_details._upcast_from_mxfp (upcast_mxfp4_tile)`, `triton_kernels.target_info (cuda_capability_geq)`, `._common (compute_offsets, get_scaled_dot_format_string, make_matmul_repr, matmul_launch_metadata, compute_pids, output_mx_scale_store_ptr)`.
  **CN:** 内部模块：`triton_kernels.tensor_details.layout_details.blackwell_scale (unswizzle_mx_scale_bw)`, `triton_kernels.tensor_details.layout_details.hopper_scale (unswizzle_mxfp4_scale_hopper)`, `triton_kernels.tensor_details.layout_details.hopper_value (mxfp4_to_bf16_triton)`, `triton_kernels.tensor_details.layout_details.cdna4_scale (unswizzle_mx_scale_cdna4)`, `triton_kernels.tensor_details.layout_details.gfx1250_scale (unswizzle_mx_scale_gfx1250)`, `triton_kernels.numerics_details.flexpoint (float_to_flex, load_scale)`, `triton_kernels.numerics_details.mxfp_details._downcast_to_mxfp (MXFP_BLOCK_SIZE, NVFP_BLOCK_SIZE)`, `triton_kernels.numerics_details.mxfp_details._upcast_from_mxfp (upcast_mxfp4_tile)`, `triton_kernels.target_info (cuda_capability_geq)`, `._common (compute_offsets, get_scaled_dot_format_string, make_matmul_repr, matmul_launch_metadata, compute_pids, output_mx_scale_store_ptr)`。
