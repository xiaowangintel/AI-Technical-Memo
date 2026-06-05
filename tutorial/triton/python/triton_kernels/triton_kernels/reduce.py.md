# reduce.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/triton_kernels/reduce.py`
- **Purpose / 用途:** Implementation module for reduce; it defines the runtime logic, helper types, and GPU-facing utilities used by the Triton kernels package via symbols PostprocessFn, OptFlags, scoped_opt_flags, _get_strides, reduce_launch_metadata. / 用于 reduce 的实现模块；它定义了 Triton kernels 包使用的运行时逻辑、辅助类型以及面向 GPU 的工具；核心符号包括 PostprocessFn、OptFlags、scoped_opt_flags、_get_strides、reduce_launch_metadata。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-1 (module)
```python
1| # fmt: off
```
**EN:** This preamble contains file-level comments or formatting directives that keep imports and generated code stable.

**CN:** 这一前导部分包含文件级注释或格式化指令，用于保持导入顺序和生成代码稳定。

### Block 2 — Lines 2-14 (module)
```python
 2| from dataclasses import dataclass
 3| from contextlib import contextmanager
 4| from contextvars import ContextVar
 5| import torch
 6| import triton
 7| import triton.language as tl
 8| from triton_kernels.numerics_details.mxfp import MXFP_BLOCK_SIZE, quantize_mxfp4_fn, quantize_mxfp8_fn, quantize_nvfp4_fn
 9| from triton_kernels.numerics_details.flexpoint import float_to_flex, load_scale
10| from triton_kernels.numerics import InFlexData, OutFlexData, MAX_FINITE_FLOAT8E4B8, MAX_FINITE_FLOAT8E4NV, MAX_FINITE_FLOAT8E5
11| from triton_kernels import target_info
12| from typing import Optional
13| from .specialize import SpecializationModule, ClosureArg, FnSpecs
14| 
```
**EN:** This block imports `dataclasses (dataclass)`, `contextlib (contextmanager)`, `contextvars (ContextVar)`, `torch`, `triton`, `triton.language`, `triton_kernels.numerics_details.mxfp (MXFP_BLOCK_SIZE, quantize_mxfp4_fn, quantize_mxfp8_fn, quantize_nvfp4_fn)`, `triton_kernels.numerics_details.flexpoint (float_to_flex, load_scale)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `dataclasses (dataclass)`, `contextlib (contextmanager)`, `contextvars (ContextVar)`, `torch`, `triton`, `triton.language`, `triton_kernels.numerics_details.mxfp (MXFP_BLOCK_SIZE, quantize_mxfp4_fn, quantize_mxfp8_fn, quantize_nvfp4_fn)`, `triton_kernels.numerics_details.flexpoint (float_to_flex, load_scale)` ，为模块提供所需的外部库和内部辅助工具。

### Block 3 — Lines 15-20 (module)
```python
15| try:
16|     from triton.language.extra import libdevice
17| except ImportError:
18|     libdevice = None
19| 
20| 
```
**EN:** Wraps code with exception/finally handling.

**CN:** 包装代码 以及异常/收尾处理.

### Block 4 — Lines 21-22 (PostprocessFn)
```python
21| @dataclass(frozen=True)
22| class PostprocessFn:
```
**EN:** Defines class `PostprocessFn` with decorators `dataclass(frozen=True)` to organize related behavior. Key fields include `specs`, `fn_args`.

**CN:** 定义类 `PostprocessFn`，带有装饰器 `dataclass(frozen=True)`，用于组织相关行为。关键字段包括 `specs`, `fn_args`.

### Block 5 — Lines 23-23 (PostprocessFn)
```python
23|     specs: FnSpecs = FnSpecs.default()
```
**EN:** Annotated assignment stores `specs` and calls `FnSpecs.default`.

**CN:** 带类型注解的赋值保存 `specs`，并调用 `FnSpecs.default`.

### Block 6 — Lines 24-26 (PostprocessFn)
```python
24|     fn_args: tuple[object] = tuple()
25| 
26| 
```
**EN:** Annotated assignment stores `fn_args` and calls `tuple`.

**CN:** 带类型注解的赋值保存 `fn_args`，并调用 `tuple`.

### Block 7 — Lines 27-28 (OptFlags)
```python
27| @dataclass(frozen=True)
28| class OptFlags:
```
**EN:** Defines class `OptFlags` with decorators `dataclass(frozen=True)` to organize related behavior. Key fields include `block_s0`, `block_x_s1`, `block_y_s1`, `num_warps`, `use_static_loop`, `chain_factor`.

**CN:** 定义类 `OptFlags`，带有装饰器 `dataclass(frozen=True)`，用于组织相关行为。关键字段包括 `block_s0`, `block_x_s1`, `block_y_s1`, `num_warps`, `use_static_loop`, `chain_factor`.

### Block 8 — Lines 29-29 (OptFlags)
```python
29|     block_s0: int
```
**EN:** Annotated assignment stores `block_s0` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `block_s0` and 声明该带注解的字段.

### Block 9 — Lines 30-30 (OptFlags)
```python
30|     block_x_s1: int
```
**EN:** Annotated assignment stores `block_x_s1` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `block_x_s1` and 声明该带注解的字段.

### Block 10 — Lines 31-31 (OptFlags)
```python
31|     block_y_s1: int
```
**EN:** Annotated assignment stores `block_y_s1` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `block_y_s1` and 声明该带注解的字段.

### Block 11 — Lines 32-32 (OptFlags)
```python
32|     num_warps: int
```
**EN:** Annotated assignment stores `num_warps` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `num_warps` and 声明该带注解的字段.

### Block 12 — Lines 33-33 (OptFlags)
```python
33|     use_static_loop: bool
```
**EN:** Annotated assignment stores `use_static_loop` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `use_static_loop` and 声明该带注解的字段.

### Block 13 — Lines 34-36 (OptFlags)
```python
34|     chain_factor: int = 1
35| 
36| 
```
**EN:** Annotated assignment stores `chain_factor` and stores constant `1`.

**CN:** 带类型注解的赋值保存 `chain_factor` and 保存常量 `1`.

### Block 14 — Lines 37-39 (module)
```python
37| _opt_flags: ContextVar[OptFlags | None] = ContextVar("reduce_opt_flags", default=None)
38| 
39| 
```
**EN:** Annotated assignment stores `_opt_flags` and calls `ContextVar`.

**CN:** 带类型注解的赋值保存 `_opt_flags`，并调用 `ContextVar`.

### Block 15 — Lines 40-41 (scoped_opt_flags)
```python
40| @contextmanager
41| def scoped_opt_flags(opt_flags: OptFlags):
```
**EN:** Defines function `scoped_opt_flags(opt_flags)` with decorators `contextmanager` for this module. The body mainly prepares intermediate values; handles exceptional paths. It uses calls such as `_opt_flags.set`, `_opt_flags.reset` to implement its workflow.

**CN:** 定义函数 `scoped_opt_flags(opt_flags)`，带有装饰器 `contextmanager`，供本模块使用. 主体主要准备中间值; 处理异常路径. 其中会调用 `_opt_flags.set`, `_opt_flags.reset` 来实现其工作流程.

### Block 16 — Lines 42-42 (scoped_opt_flags)
```python
42|     token = _opt_flags.set(opt_flags)
```
**EN:** Assigns `token` and calls `_opt_flags.set`.

**CN:** 将 `token`，并调用 `_opt_flags.set`.

### Block 17 — Lines 43-49 (scoped_opt_flags)
```python
43|     try:
44|         yield
45|     finally:
46|         _opt_flags.reset(token)
47| 
48| 
49| # Return strides in this order: (reduction dim, non-reduction dim #0, non-reduction dim #1).
```
**EN:** Wraps code with exception/finally handling.

**CN:** 包装代码 以及异常/收尾处理.

### Block 18 — Lines 50-50 (_get_strides)
```python
50| def _get_strides(t, dim, strides=None):
```
**EN:** Defines function `_get_strides(t, dim, strides)` for this module. The body mainly branches on runtime conditions; checks invariants; checks invariants. It uses calls such as `tuple`, `t.stride` to implement its workflow.

**CN:** 定义函数 `_get_strides(t, dim, strides)`，供本模块使用. 主体主要根据运行时条件分支; 检查不变量; 检查不变量. 其中会调用 `tuple`, `t.stride` 来实现其工作流程.

### Block 19 — Lines 51-53 (_get_strides)
```python
51|     if t is None:
52|         return 0, 0, 0
53| 
```
**EN:** Checks `t is None`. The true branch mainly returns the computed result.

**CN:** 检查 `t is None`. 真分支主要返回计算结果.

### Block 20 — Lines 54-54 (_get_strides)
```python
54|     assert t.ndim == 3
```
**EN:** Asserts `t.ndim == 3` to enforce invariants.

**CN:** 断言 `t.ndim == 3` 以确保不变量成立。

### Block 21 — Lines 55-55 (_get_strides)
```python
55|     assert dim in (0, 1, 2)
```
**EN:** Asserts `dim in (0, 1, 2)` to enforce invariants.

**CN:** 断言 `dim in (0, 1, 2)` 以确保不变量成立。

### Block 22 — Lines 56-56 (_get_strides)
```python
56|     nonred = tuple(d for d in (0, 1, 2) if d != dim)
```
**EN:** Assigns `nonred` and calls `tuple`.

**CN:** 将 `nonred`，并调用 `tuple`.

### Block 23 — Lines 57-58 (_get_strides)
```python
57|     if strides is None:
58|         strides = t.stride()
```
**EN:** Checks `strides is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `strides is None`. 真分支主要准备中间值.

### Block 24 — Lines 59-61 (_get_strides)
```python
59|     return strides[dim], strides[nonred[0]], strides[nonred[1]]
60| 
61| 
```
**EN:** Returns `(strides[dim], strides[nonred[0]], strides[nonred[1]])`.

**CN:** 返回 `(strides[dim], strides[nonred[0]], strides[nonred[1]])`.

### Block 25 — Lines 62-62 (reduce_launch_metadata)
```python
62| def reduce_launch_metadata(grid, kernel, args):
```
**EN:** Defines function `reduce_launch_metadata(grid, kernel, args)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `dict`, `tuple`, `m.sum`, `(m.sum(dim=dim) - 1).clamp(min=0).sum`, `launch_metadata_allow_sync` to implement its workflow.

**CN:** 定义函数 `reduce_launch_metadata(grid, kernel, args)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `dict`, `tuple`, `m.sum`, `(m.sum(dim=dim) - 1).clamp(min=0).sum`, `launch_metadata_allow_sync` 来实现其工作流程.

### Block 26 — Lines 63-63 (reduce_launch_metadata)
```python
63|     from .proton_opts import launch_metadata_allow_sync
```
**EN:** This block imports `.proton_opts (launch_metadata_allow_sync)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `.proton_opts (launch_metadata_allow_sync)` ，为模块提供所需的外部库和内部辅助工具。

### Block 27 — Lines 64-64 (reduce_launch_metadata)
```python
64|     ret = dict()
```
**EN:** Assigns `ret` and calls `dict`.

**CN:** 将 `ret`，并调用 `dict`.

### Block 28 — Lines 65-65 (reduce_launch_metadata)
```python
65|     X, Y, Mask, dim = args["X"], args["Y"], args["Mask"], args["DIM"]
```
**EN:** Assigns `X`, `Y`, `Mask`, `dim` and builds a tuple.

**CN:** 将 `X`, `Y`, `Mask`, `dim` and 构造一个元组.

### Block 29 — Lines 66-66 (reduce_launch_metadata)
```python
66|     unpadded_batch_size = args["UnpaddedBatchSize"]
```
**EN:** Assigns `unpadded_batch_size` and evaluates `args['UnpaddedBatchSize']`.

**CN:** 将 `unpadded_batch_size` and 计算 `args['UnpaddedBatchSize']`.

### Block 30 — Lines 67-67 (reduce_launch_metadata)
```python
67|     nbits = X.dtype.itemsize * 8
```
**EN:** Assigns `nbits` and evaluates `X.dtype.itemsize * 8`.

**CN:** 将 `nbits` and 计算 `X.dtype.itemsize * 8`.

### Block 31 — Lines 68-70 (reduce_launch_metadata)
```python
68|     ret["name"] = f"{kernel.name} {tuple(X.shape)}->{tuple(Y.shape)}"
69| 
70|     # TODO: Currently not counting scale or mx.
```
**EN:** Assigns `ret['name']` and evaluates `f'{kernel.name} {tuple(X.shape)}->{tuple(Y.shape)}'`.

**CN:** 将 `ret['name']` and 计算 `f'{kernel.name} {tuple(X.shape)}->{tuple(Y.shape)}'`.

### Block 32 — Lines 71-86 (reduce_launch_metadata)
```python
71|     if Mask is None:
72|         ret[f"flops{nbits}"] = X.numel() - Y.numel()
73|         ret["bytes"] = X.numel() * X.element_size() + Y.numel() * Y.element_size()
74|     else:
75|         m = (Mask != 0)
76|         dim0, dim1 = tuple(d for d in (0, 1, 2) if d != dim)
77|         if unpadded_batch_size is not None:
78|             m = m.narrow(dim0, 0, unpadded_batch_size.item())
79|         total_loads = m.sum()
80|         total_adds = (m.sum(dim=dim) - 1).clamp(min=0).sum()
81|         total_stores = m.shape[dim0] * Y.shape[1]
82|         if launch_metadata_allow_sync():
83|             total_loads = total_loads.item()
84|             total_adds = total_adds.item()
85|         ret[f"flops{nbits}"] = total_adds
86|         ret["bytes"] = total_loads * X.element_size() + total_stores * Y.element_size()
```
**EN:** Checks `Mask is None`. The true branch mainly prepares intermediate values; prepares intermediate values, while the else branch prepares intermediate values; prepares intermediate values.

**CN:** 检查 `Mask is None`. 真分支主要准备中间值; 准备中间值；而 else 分支准备中间值; 准备中间值.

### Block 33 — Lines 87-89 (reduce_launch_metadata)
```python
87|     return ret
88| 
89| 
```
**EN:** Returns `ret`.

**CN:** 返回 `ret`.

### Block 34 — Lines 90-98 (_select_reduce_forward_config)
```python
90| def _select_reduce_forward_config(
91|     x_dtype: torch.dtype,
92|     S0: int,
93|     Y_S1: int,
94|     reduction_n: int,
95|     K: int,
96|     has_mx: bool,
97|     mask_chainable: bool,  # Can we reuse mask along the S1 dim?
98| ) -> OptFlags:
```
**EN:** Defines function `_select_reduce_forward_config(x_dtype, S0, Y_S1, reduction_n, K, has_mx, mask_chainable)` for this module. The body mainly prepares intermediate values; branches on runtime conditions; prepares intermediate values. It uses calls such as `triton.cdiv`, `OptFlags`, `max`, `min`, `target_info.num_sms` to implement its workflow.

**CN:** 定义函数 `_select_reduce_forward_config(x_dtype, S0, Y_S1, reduction_n, K, has_mx, mask_chainable)`，供本模块使用. 主体主要准备中间值; 根据运行时条件分支; 准备中间值. 其中会调用 `triton.cdiv`, `OptFlags`, `max`, `min`, `target_info.num_sms` 来实现其工作流程.

### Block 35 — Lines 99-99 (_select_reduce_forward_config)
```python
99|     use_static_loop = K <= 8
```
**EN:** Assigns `use_static_loop` and evaluates `K <= 8`.

**CN:** 将 `use_static_loop` and 计算 `K <= 8`.

### Block 36 — Lines 100-128 (_select_reduce_forward_config)
```python
100|     if K in (2, 3, 4) and S0 <= 256 and Y_S1 >= 4096 and reduction_n == 1 and not has_mx:
101|         if K >= 3:
102|             # K>=3 does more loads than K=2, so keep its tile area
103|             # somewhat smaller.
104|             target_elems = 512
105|         else:
106|             if S0 <= 8:
107|                 target_elems = 512
108|             elif S0 <= 64:
109|                 target_elems = 1024
110|             elif S0 <= 128:
111|                 target_elems = 512
112|             else:
113|                 target_elems = 2048
114| 
115|         # A full-wave floor can force very narrow S1 tiles for tiny S0.
116|         # Keep this slightly below num_sms so we prefer contiguous S1 work
117|         # when a tile is already close to filling the device.
118|         min_programs = max(1, (3 * target_info.num_sms()) // 4)
119|         block_s1 = min(512, target_elems)
120|         while block_s1 >= 16:
121|             max_block_s0 = min(S0, max(1, target_elems // block_s1))
122|             min_s0_programs = triton.cdiv(min_programs, triton.cdiv(Y_S1, block_s1))
123|             if min_s0_programs <= S0:
124|                 max_occupancy_block_s0 = max(1, S0 // min_s0_programs)
125|                 block_s0 = min(max_block_s0, max_occupancy_block_s0)
126|                 return OptFlags(block_s0, block_s1, block_s1, 4, use_static_loop, 1)
127|             block_s1 //= 2
128| 
```
**EN:** Checks `K in (2, 3, 4) and S0 <= 256 and (Y_S1 >= 4096) and (reduction_n == 1) and (not has_mx)`. The true branch mainly branches on runtime conditions; prepares intermediate values.

**CN:** 检查 `K in (2, 3, 4) and S0 <= 256 and (Y_S1 >= 4096) and (reduction_n == 1) and (not has_mx)`. 真分支主要根据运行时条件分支; 准备中间值.

### Block 37 — Lines 129-129 (_select_reduce_forward_config)
```python
129|     block_s0 = 32
```
**EN:** Assigns `block_s0` and stores constant `32`.

**CN:** 将 `block_s0` and 保存常量 `32`.

### Block 38 — Lines 130-130 (_select_reduce_forward_config)
```python
130|     block_x_s1 = 128
```
**EN:** Assigns `block_x_s1` and stores constant `128`.

**CN:** 将 `block_x_s1` and 保存常量 `128`.

### Block 39 — Lines 131-131 (_select_reduce_forward_config)
```python
131|     block_y_s1 = block_x_s1 // reduction_n
```
**EN:** Assigns `block_y_s1` and evaluates `block_x_s1 // reduction_n`.

**CN:** 将 `block_y_s1` and 计算 `block_x_s1 // reduction_n`.

### Block 40 — Lines 132-132 (_select_reduce_forward_config)
```python
132|     grid_m = triton.cdiv(S0, block_s0)
```
**EN:** Assigns `grid_m` and calls `triton.cdiv`.

**CN:** 将 `grid_m`，并调用 `triton.cdiv`.

### Block 41 — Lines 133-135 (_select_reduce_forward_config)
```python
133|     grid_n = triton.cdiv(Y_S1, block_y_s1)
134|     # Enable chaining for large tensors (seems like it's neutral/negative for
135|     # fp8 and/or small tensors).
```
**EN:** Assigns `grid_n` and calls `triton.cdiv`.

**CN:** 将 `grid_n`，并调用 `triton.cdiv`.

### Block 42 — Lines 136-145 (_select_reduce_forward_config)
```python
136|     if mask_chainable and S0 >= 32768 and x_dtype.itemsize >= 2:
137|         chain_factor = (grid_m * grid_n) // (target_info.num_sms() * 4)
138|         chain_factor = min(max(1, chain_factor), grid_n)
139|         # Adjust chain_factor to divide along S1 axis as even as possible.
140|         n_divisor = triton.cdiv(grid_n, chain_factor)
141|         chain_factor = triton.cdiv(grid_n, n_divisor)
142|         if chain_factor > 1:
143|             use_static_loop = False
144|     else:
145|         chain_factor = 1
```
**EN:** Checks `mask_chainable and S0 >= 32768 and (x_dtype.itemsize >= 2)`. The true branch mainly prepares intermediate values; prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `mask_chainable and S0 >= 32768 and (x_dtype.itemsize >= 2)`. 真分支主要准备中间值; 准备中间值；而 else 分支准备中间值.

### Block 43 — Lines 146-147 (_select_reduce_forward_config)
```python
146|     num_warps = 8 if chain_factor > 1 else 4
147| 
```
**EN:** Assigns `num_warps` and uses conditional expression `8 if chain_factor > 1 else 4`.

**CN:** 将 `num_warps` and 使用条件表达式 `8 if chain_factor > 1 else 4`.

### Block 44 — Lines 148-159 (_select_reduce_forward_config)
```python
148|     return OptFlags(32, 128, 128 // reduction_n, num_warps, use_static_loop, chain_factor)
149| 
150| 
151| # Divide Mask into blocks of size (BLOCK_M, K).  In each block, compute
152| # ActiveInputCounts = Mask.sum(dim=1), and group rows based on ActiveInputCounts
153| # so that similar values are grouped together.
154| #
155| # The main kernel (_reduce_forward) has different code paths for (0 or 1 active
156| # inputs), (2 active inputs), (3 active inputs), and (more than 3 active
157| # inputs), so that's what we do here: we don't need a full argsort.  We "sort"
158| # rows in descending order so that the main kernel processes heavier rows (more
159| # active inputs) first.
```
**EN:** Returns `OptFlags(32, 128, 128 // reduction_n, num_warps, use_static_loop, chain_factor)`.

**CN:** 返回 `OptFlags(32, 128, 128 // reduction_n, num_warps, use_static_loop, chain_factor)`.

### Block 45 — Lines 160-169 (_create_row_idxs)
```python
160| @triton.jit
161| def _create_row_idxs(Mask, stride_mr, stride_m0,  #
162|                      ActiveInputCounts, RowIdxs,  #
163|                      S0,
164|                      UnpaddedBatchSize,
165|                      K: tl.constexpr,
166|                      BLOCK_M: tl.constexpr,
167|                      BLOCK_K: tl.constexpr,
168|                      BLOCK_S0: tl.constexpr,
169|                      ):
```
**EN:** Defines function `_create_row_idxs(Mask, stride_mr, stride_m0, ActiveInputCounts, RowIdxs, S0, UnpaddedBatchSize, K, BLOCK_M, BLOCK_K, BLOCK_S0)` with decorators `triton.jit` for this module. The body mainly invokes `tl.static_assert`; prepares intermediate values; branches on runtime conditions. It uses calls such as `tl.static_assert`, `idxs.ravel`, `tl.arange`, `tl.load`, `tl.sum` to implement its workflow.

**CN:** 定义函数 `_create_row_idxs(Mask, stride_mr, stride_m0, ActiveInputCounts, RowIdxs, S0, UnpaddedBatchSize, K, BLOCK_M, BLOCK_K, BLOCK_S0)`，带有装饰器 `triton.jit`，供本模块使用. 主体主要invokes `tl.static_assert`; 准备中间值; 根据运行时条件分支. 其中会调用 `tl.static_assert`, `idxs.ravel`, `tl.arange`, `tl.load`, `tl.sum` 来实现其工作流程.

### Block 46 — Lines 170-170 (_create_row_idxs)
```python
170|     tl.static_assert(BLOCK_M % BLOCK_S0 == 0)
```
**EN:** Calls `tl.static_assert` for side effects, registration, or validation.

**CN:** 调用 `tl.static_assert` ，用于副作用、注册或校验。

### Block 47 — Lines 171-172 (_create_row_idxs)
```python
171|     S0_BLK_CNT: tl.constexpr = BLOCK_M // BLOCK_S0
172| 
```
**EN:** Annotated assignment stores `S0_BLK_CNT` and evaluates `BLOCK_M // BLOCK_S0`.

**CN:** 带类型注解的赋值保存 `S0_BLK_CNT` and 计算 `BLOCK_M // BLOCK_S0`.

### Block 48 — Lines 173-179 (_create_row_idxs)
```python
173|     if UnpaddedBatchSize is not None:
174|         unpadded = tl.load(UnpaddedBatchSize).to(tl.int32)
175|         S0 = unpadded
176| 
177|     # E.g., if we have 5 CTA's, then CTA #0 reads S0 blocks (0, 5, 10, ...),
178|     # #1 reads S0 blocks (1, 6, 11, ...), and so on, so that heaviest rows from
179|     # each CTA ends up at the top of RowIdxs.
```
**EN:** Checks `UnpaddedBatchSize is not None`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `UnpaddedBatchSize is not None`. 真分支主要准备中间值; 准备中间值.

### Block 49 — Lines 180-180 (_create_row_idxs)
```python
180|     S0_blk_idxs = tl.arange(0, S0_BLK_CNT) * tl.num_programs(0) + tl.program_id(0)
```
**EN:** Assigns `S0_blk_idxs` and evaluates `tl.arange(0, S0_BLK_CNT) * tl.num_programs(0) + tl.program_id(0)`.

**CN:** 将 `S0_blk_idxs` and 计算 `tl.arange(0, S0_BLK_CNT) * tl.num_programs(0) + tl.program_id(0)`.

### Block 50 — Lines 181-181 (_create_row_idxs)
```python
181|     idxs = S0_blk_idxs[:, None] * BLOCK_S0 + tl.arange(0, BLOCK_S0)[None, :]
```
**EN:** Assigns `idxs` and evaluates `S0_blk_idxs[:, None] * BLOCK_S0 + tl.arange(0, BLOCK_S0)[None, :]`.

**CN:** 将 `idxs` and 计算 `S0_blk_idxs[:, None] * BLOCK_S0 + tl.arange(0, BLOCK_S0)[None, :]`.

### Block 51 — Lines 182-182 (_create_row_idxs)
```python
182|     idxs = idxs.ravel()
```
**EN:** Assigns `idxs` and calls `idxs.ravel`.

**CN:** 将 `idxs`，并调用 `idxs.ravel`.

### Block 52 — Lines 183-184 (_create_row_idxs)
```python
183|     valid = idxs < S0
184| 
```
**EN:** Assigns `valid` and evaluates `idxs < S0`.

**CN:** 将 `valid` and 计算 `idxs < S0`.

### Block 53 — Lines 185-185 (_create_row_idxs)
```python
185|     k_idxs = tl.arange(0, BLOCK_K)
```
**EN:** Assigns `k_idxs` and calls `tl.arange`.

**CN:** 将 `k_idxs`，并调用 `tl.arange`.

### Block 54 — Lines 186-190 (_create_row_idxs)
```python
186|     mask = tl.load(
187|         Mask + idxs[:, None] * stride_m0 + k_idxs[None, :] * stride_mr,
188|         mask=valid[:, None] & (k_idxs < K)[None, :],
189|         other=0,
190|     )
```
**EN:** Assigns `mask` and calls `tl.load`.

**CN:** 将 `mask`，并调用 `tl.load`.

### Block 55 — Lines 191-191 (_create_row_idxs)
```python
191|     n_actives = tl.sum((mask != 0).to(tl.int32), axis=1)
```
**EN:** Assigns `n_actives` and calls `tl.sum`.

**CN:** 将 `n_actives`，并调用 `tl.sum`.

### Block 56 — Lines 192-195 (_create_row_idxs)
```python
192|     tl.store(ActiveInputCounts + idxs, n_actives, mask=valid)
193| 
194|     # We're using to(tl.int32) because otherwise TRITON_DEBUG=1 complains about
195|     # unsigned underflow.
```
**EN:** Calls `tl.store` for side effects, registration, or validation.

**CN:** 调用 `tl.store` ，用于副作用、注册或校验。

### Block 57 — Lines 196-196 (_create_row_idxs)
```python
196|     has_4_or_more = valid & (n_actives >= 4)
```
**EN:** Assigns `has_4_or_more` and evaluates `valid & (n_actives >= 4)`.

**CN:** 将 `has_4_or_more` and 计算 `valid & (n_actives >= 4)`.

### Block 58 — Lines 197-197 (_create_row_idxs)
```python
197|     idxs4 = tl.maximum(has_4_or_more.to(tl.int32).cumsum(axis=0) - 1, 0)
```
**EN:** Assigns `idxs4` and calls `tl.maximum`.

**CN:** 将 `idxs4`，并调用 `tl.maximum`.

### Block 59 — Lines 198-198 (_create_row_idxs)
```python
198|     tl.store(RowIdxs + tl.gather(idxs, idxs4, axis=0), idxs, mask=has_4_or_more)
```
**EN:** Calls `tl.store` for side effects, registration, or validation.

**CN:** 调用 `tl.store` ，用于副作用、注册或校验。

### Block 60 — Lines 199-200 (_create_row_idxs)
```python
199|     offset = tl.sum(has_4_or_more)
200| 
```
**EN:** Assigns `offset` and calls `tl.sum`.

**CN:** 将 `offset`，并调用 `tl.sum`.

### Block 61 — Lines 201-201 (_create_row_idxs)
```python
201|     has_3 = valid & (n_actives == 3)
```
**EN:** Assigns `has_3` and evaluates `valid & (n_actives == 3)`.

**CN:** 将 `has_3` and 计算 `valid & (n_actives == 3)`.

### Block 62 — Lines 202-202 (_create_row_idxs)
```python
202|     idxs3 = tl.minimum(offset + tl.maximum(has_3.to(tl.int32).cumsum(axis=0) - 1, 0), BLOCK_M - 1)
```
**EN:** Assigns `idxs3` and calls `tl.minimum`.

**CN:** 将 `idxs3`，并调用 `tl.minimum`.

### Block 63 — Lines 203-203 (_create_row_idxs)
```python
203|     tl.store(RowIdxs + tl.gather(idxs, idxs3, axis=0), idxs, mask=has_3)
```
**EN:** Calls `tl.store` for side effects, registration, or validation.

**CN:** 调用 `tl.store` ，用于副作用、注册或校验。

### Block 64 — Lines 204-205 (_create_row_idxs)
```python
204|     offset += tl.sum(has_3)
205| 
```
**EN:** Updates `offset` with operator `Add` using `tl.sum(has_3)`.

**CN:** 更新 `offset`，使用运算符 `Add`，并使用 `tl.sum(has_3)`.

### Block 65 — Lines 206-206 (_create_row_idxs)
```python
206|     has_2 = valid & (n_actives == 2)
```
**EN:** Assigns `has_2` and evaluates `valid & (n_actives == 2)`.

**CN:** 将 `has_2` and 计算 `valid & (n_actives == 2)`.

### Block 66 — Lines 207-207 (_create_row_idxs)
```python
207|     idxs2 = tl.minimum(offset + tl.maximum(has_2.to(tl.int32).cumsum(axis=0) - 1, 0), BLOCK_M - 1)
```
**EN:** Assigns `idxs2` and calls `tl.minimum`.

**CN:** 将 `idxs2`，并调用 `tl.minimum`.

### Block 67 — Lines 208-208 (_create_row_idxs)
```python
208|     tl.store(RowIdxs + tl.gather(idxs, idxs2, axis=0), idxs, mask=has_2)
```
**EN:** Calls `tl.store` for side effects, registration, or validation.

**CN:** 调用 `tl.store` ，用于副作用、注册或校验。

### Block 68 — Lines 209-210 (_create_row_idxs)
```python
209|     offset += tl.sum(has_2)
210| 
```
**EN:** Updates `offset` with operator `Add` using `tl.sum(has_2)`.

**CN:** 更新 `offset`，使用运算符 `Add`，并使用 `tl.sum(has_2)`.

### Block 69 — Lines 211-211 (_create_row_idxs)
```python
211|     has_0_or_1 = valid & (n_actives <= 1)
```
**EN:** Assigns `has_0_or_1` and evaluates `valid & (n_actives <= 1)`.

**CN:** 将 `has_0_or_1` and 计算 `valid & (n_actives <= 1)`.

### Block 70 — Lines 212-212 (_create_row_idxs)
```python
212|     idxs1 = tl.minimum(offset + tl.maximum(has_0_or_1.to(tl.int32).cumsum(axis=0) - 1, 0), BLOCK_M - 1)
```
**EN:** Assigns `idxs1` and calls `tl.minimum`.

**CN:** 将 `idxs1`，并调用 `tl.minimum`.

### Block 71 — Lines 213-215 (_create_row_idxs)
```python
213|     tl.store(RowIdxs + tl.gather(idxs, idxs1, axis=0), idxs, mask=has_0_or_1)
214| 
215| 
```
**EN:** Calls `tl.store` for side effects, registration, or validation.

**CN:** 调用 `tl.store` ，用于副作用、注册或校验。

### Block 72 — Lines 216-250 (_reduce_forward_inner)
```python
216| @triton.jit(noinline=True)
217| def _reduce_forward_inner(X, stride_xr: tl.int64, stride_x0: tl.int64, stride_x1,  # x tensor (input)
218|                           XMx, stride_xmxr, stride_xmx0, stride_xmx1,  # x mx scale
219|                           Y, stride_y0: tl.int64, stride_y1,  # y tensor (output)
220|                           YMx, stride_ymx0, stride_ymx1,  # y mx scale
221|                           Mask, stride_mr, stride_m0, stride_m1,  # mask tensor
222|                           Scale, stride_sr, stride_s0, stride_s1,  # scale tensor
223|                           # shape (K = reduction dim; S0, IN_S1 = input dims, OUT_S1 = output dims)
224|                           K: tl.constexpr, S0, X_S1, Y_S1,  #
225|                           BLOCK_K: tl.constexpr,
226|                           POSTPROCESS_FN1: tl.constexpr, postprocess_fn1_args,  #
227|                           POSTPROCESS_FN2: tl.constexpr, postprocess_fn2_args,  #
228|                           POSTPROCESS_MX_FN: tl.constexpr, postprocess_mx_fn_args,  #
229|                           XFlex,  # TODO: remove this
230|                           XGlobalScale,  # x global scale
231|                           YFlexExpected, YFlexActual, YFlexChecksum,
232|                           Y_FLEX_SATURATE_INF: tl.constexpr,  # y flex (global) scale
233|                           ActiveInputCounts, RowIdxs,  #
234|                           BROADCAST_R: tl.constexpr,  #
235|                           BROADCAST_S0: tl.constexpr,  #
236|                           BROADCAST_S1: tl.constexpr,  #
237|                           IS_SCALE_NONE: tl.constexpr,  #
238|                           SCALE_BROADCAST_R: tl.constexpr,  #
239|                           SCALE_BROADCAST_S0: tl.constexpr,  #
240|                           SCALE_BROADCAST_S1: tl.constexpr,  #
241|                           BLOCK_S0: tl.constexpr,  #
242|                           BLOCK_X_S1: tl.constexpr,  #
243|                           BLOCK_Y_S1: tl.constexpr,  #
244|                           CHAIN_FACTOR: tl.constexpr,  #
245|                           Y_MX_BLOCK_SIZE: tl.constexpr,  #
246|                           Y_VALUE_PACK_FACTOR: tl.constexpr,  #
247|                           # Only the following two args are adjusted for the inner call.
248|                           USE_STATIC_LOOP: tl.constexpr,  #
249|                           LIMIT: tl.constexpr,
250|                           ):
```
**EN:** Defines function `_reduce_forward_inner(X, stride_xr, stride_x0, stride_x1, XMx, stride_xmxr, stride_xmx0, stride_xmx1, Y, stride_y0, stride_y1, YMx, stride_ymx0, stride_ymx1, Mask, stride_mr, stride_m0, stride_m1, Scale, stride_sr, stride_s0, stride_s1, K, S0, X_S1, Y_S1, BLOCK_K, POSTPROCESS_FN1, postprocess_fn1_args, POSTPROCESS_FN2, postprocess_fn2_args, POSTPROCESS_MX_FN, postprocess_mx_fn_args, XFlex, XGlobalScale, YFlexExpected, YFlexActual, YFlexChecksum, Y_FLEX_SATURATE_INF, ActiveInputCounts, RowIdxs, BROADCAST_R, BROADCAST_S0, BROADCAST_S1, IS_SCALE_NONE, SCALE_BROADCAST_R, SCALE_BROADCAST_S0, SCALE_BROADCAST_S1, BLOCK_S0, BLOCK_X_S1, BLOCK_Y_S1, CHAIN_FACTOR, Y_MX_BLOCK_SIZE, Y_VALUE_PACK_FACTOR, USE_STATIC_LOOP, LIMIT)` with decorators `triton.jit(noinline=True)` for this module. The body mainly prepares intermediate values; invokes `tl.static_assert`; prepares intermediate values. It uses calls such as `triton.jit`, `tl.program_id`, `tl.static_assert`, `tl.cdiv`, `load_scale` to implement its workflow.

**CN:** 定义函数 `_reduce_forward_inner(X, stride_xr, stride_x0, stride_x1, XMx, stride_xmxr, stride_xmx0, stride_xmx1, Y, stride_y0, stride_y1, YMx, stride_ymx0, stride_ymx1, Mask, stride_mr, stride_m0, stride_m1, Scale, stride_sr, stride_s0, stride_s1, K, S0, X_S1, Y_S1, BLOCK_K, POSTPROCESS_FN1, postprocess_fn1_args, POSTPROCESS_FN2, postprocess_fn2_args, POSTPROCESS_MX_FN, postprocess_mx_fn_args, XFlex, XGlobalScale, YFlexExpected, YFlexActual, YFlex检查um, Y_FLEX_SATURATE_INF, ActiveInputCounts, RowIdxs, BROADCAST_R, BROADCAST_S0, BROADCAST_S1, IS_SCALE_NONE, SCALE_BROADCAST_R, SCALE_BROADCAST_S0, SCALE_BROADCAST_S1, BLOCK_S0, BLOCK_X_S1, BLOCK_Y_S1, CHAIN_FACTOR, Y_MX_BLOCK_SIZE, Y_VALUE_PACK_FACTOR, USE_STATIC_LOOP, LIMIT)`，带有装饰器 `triton.jit(noinline=True)`，供本模块使用. 主体主要准备中间值; invokes `tl.static_assert`; 准备中间值. 其中会调用 `triton.jit`, `tl.program_id`, `tl.static_assert`, `tl.cdiv`, `load_scale` 来实现其工作流程.

### Block 73 — Lines 251-251 (_reduce_forward_inner)
```python
251|     pid_s0 = tl.program_id(0)
```
**EN:** Assigns `pid_s0` and calls `tl.program_id`.

**CN:** 将 `pid_s0`，并调用 `tl.program_id`.

### Block 74 — Lines 252-252 (_reduce_forward_inner)
```python
252|     tl.static_assert(XMx is None or BLOCK_X_S1 % 32 == 0)
```
**EN:** Calls `tl.static_assert` for side effects, registration, or validation.

**CN:** 调用 `tl.static_assert` ，用于副作用、注册或校验。

### Block 75 — Lines 253-253 (_reduce_forward_inner)
```python
253|     BLOCK_X_SMX1: tl.constexpr = tl.cdiv(BLOCK_X_S1, 32)
```
**EN:** Annotated assignment stores `BLOCK_X_SMX1` and calls `tl.cdiv`.

**CN:** 带类型注解的赋值保存 `BLOCK_X_SMX1`，并调用 `tl.cdiv`.

### Block 76 — Lines 254-254 (_reduce_forward_inner)
```python
254|     BLOCK_Y_SMX1: tl.constexpr = tl.cdiv(BLOCK_Y_S1, 1 if Y_MX_BLOCK_SIZE is None else Y_MX_BLOCK_SIZE)
```
**EN:** Annotated assignment stores `BLOCK_Y_SMX1` and calls `tl.cdiv`.

**CN:** 带类型注解的赋值保存 `BLOCK_Y_SMX1`，并调用 `tl.cdiv`.

### Block 77 — Lines 255-255 (_reduce_forward_inner)
```python
255|     offs_s0 = pid_s0 * BLOCK_S0 + tl.arange(0, BLOCK_S0)
```
**EN:** Assigns `offs_s0` and evaluates `pid_s0 * BLOCK_S0 + tl.arange(0, BLOCK_S0)`.

**CN:** 将 `offs_s0` and 计算 `pid_s0 * BLOCK_S0 + tl.arange(0, BLOCK_S0)`.

### Block 78 — Lines 256-256 (_reduce_forward_inner)
```python
256|     valid_s0 = offs_s0 < S0
```
**EN:** Assigns `valid_s0` and evaluates `offs_s0 < S0`.

**CN:** 将 `valid_s0` and 计算 `offs_s0 < S0`.

### Block 79 — Lines 257-258 (_reduce_forward_inner)
```python
257|     x_flex_scale = load_scale(XFlex)
258| 
```
**EN:** Assigns `x_flex_scale` and calls `load_scale`.

**CN:** 将 `x_flex_scale`，并调用 `load_scale`.

### Block 80 — Lines 259-259 (_reduce_forward_inner)
```python
259|     USE_BITMAP: tl.constexpr = ActiveInputCounts is not None
```
**EN:** Annotated assignment stores `USE_BITMAP` and evaluates `ActiveInputCounts is not None`.

**CN:** 带类型注解的赋值保存 `USE_BITMAP` and 计算 `ActiveInputCounts is not None`.

### Block 81 — Lines 260-276 (_reduce_forward_inner)
```python
260|     if USE_BITMAP:
261|         offs_s0 = tl.load(RowIdxs + offs_s0, mask=valid_s0, other=0)
262| 
263|         # Construct mask bitmap: we know each output row has at most LIMIT
264|         # active input rows, but they may be distributed over K (> LIMIT) rows.
265|         tl.static_assert(BROADCAST_S1)
266|         k_term = 0 if BROADCAST_R else (tl.arange(0, BLOCK_K)[None, :] * stride_mr)
267|         s0_term = 0 if BROADCAST_S0 else (offs_s0[:, None] * stride_m0)
268|         m_ptrs = Mask + k_term + s0_term
269|         valid_k = tl.arange(0, BLOCK_K) < K
270|         m = tl.load(m_ptrs, mask=valid_s0[:, None] & valid_k[None, :], other=0)
271| 
272|         # Compact each row into a bitmap.
273|         tl.static_assert(K <= 32)
274|         m = (m != 0).to(tl.uint32) << tl.arange(0, BLOCK_K)[None, :]
275|         bitmap = tl.sum(m, axis=1)
276| 
```
**EN:** Checks `USE_BITMAP`. The true branch mainly prepares intermediate values; invokes `tl.static_assert`.

**CN:** 检查 `USE_BITMAP`. 真分支主要准备中间值; invokes `tl.static_assert`.

### Block 82 — Lines 277-351 (_reduce_forward_inner)
```python
277|     for blk_s1 in tl.range(0, CHAIN_FACTOR):
278|         if not USE_STATIC_LOOP:
279|             y = tl.zeros((BLOCK_S0, BLOCK_X_S1), dtype=tl.float32)
280| 
281|         pid_s1 = tl.program_id(1) * CHAIN_FACTOR + blk_s1
282|         offs_x_s1 = pid_s1 * BLOCK_X_S1 + tl.arange(0, BLOCK_X_S1)
283|         offs_x_smx1 = pid_s1 * BLOCK_X_SMX1 + tl.arange(0, BLOCK_X_SMX1)
284|         valid_x_s1 = offs_x_s1 < X_S1
285|         valid_in_smx1 = offs_x_smx1 < tl.cdiv(X_S1, 32)
286|         if USE_BITMAP:
287|             bitmap1 = bitmap
288| 
289|         for k in (tl.static_range if USE_STATIC_LOOP else tl.range)(0, LIMIT):
290|             mask = valid_s0[:, None] & valid_x_s1[None, :]
291|             offs_r = k
292|             if USE_BITMAP:
293|                 ffs = libdevice.ffs(bitmap1.to(tl.int32))
294|                 # clear the lowest bit.
295|                 # (Equivalent to (bitmap1 & (bitmap1 - 1)), but the simpler
296|                 # expression triggers underflow check with TRITON_DEBUG=1.)
297|                 bitmap1 = bitmap1 & (bitmap1 - (bitmap1 != 0).to(tl.uint32))
298|                 mask &= (ffs != 0)[:, None]
299|                 offs_r = tl.maximum(ffs - 1, 0)[:, None]
300|             elif Mask is not None:
301|                 k_term = 0 if BROADCAST_R else (offs_r * stride_mr)
302|                 s0_term = 0 if BROADCAST_S0 else (offs_s0[:, None] * stride_m0)
303|                 s1_term = 0 if BROADCAST_S1 else (offs_x_s1[None, :] * stride_m1)
304|                 m_ptrs = Mask + k_term + s0_term + s1_term
305|                 m = tl.load(m_ptrs, mask=mask, other=1).to(tl.int1)
306|                 mask &= m
307|             x_ptrs = X + offs_r * stride_xr + offs_s0[:, None] * stride_x0 + offs_x_s1[None, :] * stride_x1
308|             x = tl.load(x_ptrs, mask=mask, other=0.0).to(tl.float32)
309|             if XMx is not None:
310|                 xmx_ptrs = XMx + offs_r * stride_xmxr + offs_s0[:, None] * stride_xmx0 + offs_x_smx1[None, :] * stride_xmx1
311|                 xmx = tl.load(xmx_ptrs, mask=valid_s0[:, None] & valid_in_smx1[None, :], other=0.0)
312|                 xmx = (xmx.to(tl.uint32) << 23).to(tl.float32, bitcast=True)
313|                 x = (xmx[:, :, None] * x.reshape([BLOCK_S0, BLOCK_X_S1 // 32, 32])).reshape([BLOCK_S0, BLOCK_X_S1])
314|             x = x * x_flex_scale
315|             if not IS_SCALE_NONE:
316|                 k_term_s = 0 if SCALE_BROADCAST_R else (offs_r * stride_sr)
317|                 s0_term_s = 0 if SCALE_BROADCAST_S0 else (offs_s0[:, None] * stride_s0)
318|                 s1_term_s = 0 if SCALE_BROADCAST_S1 else (offs_x_s1[None, :] * stride_s1)
319|                 s_ptrs = Scale + k_term_s + s0_term_s + s1_term_s
320|                 s = tl.load(s_ptrs, mask=mask, other=1)
321|                 x = tl.fma(x, s, 0.0)
322|             if USE_STATIC_LOOP and k == 0:
323|                 y = x
324|             else:
325|                 y += x
326|         if POSTPROCESS_FN1 is not None:
327|             y = POSTPROCESS_FN1(y, *postprocess_fn1_args)
328|         if XGlobalScale is not None:
329|             y *= tl.load(XGlobalScale)
330|         offs_y_s1 = pid_s1 * BLOCK_Y_S1 + tl.arange(0, BLOCK_Y_S1)
331|         offs_y_smx1 = pid_s1 * BLOCK_Y_SMX1 + tl.arange(0, BLOCK_Y_SMX1)
332|         valid_y_s1 = offs_y_s1 < Y_S1
333|         valid_y_smx1 = offs_y_smx1 < tl.cdiv(Y_S1, 1 if Y_MX_BLOCK_SIZE is None else Y_MX_BLOCK_SIZE)
334|         is_out_fp4: tl.constexpr = YMx is not None and Y_VALUE_PACK_FACTOR == 2
335|         if YMx is not None:
336|             y = float_to_flex(y, YFlexExpected, None, None, None, Y, False)
337|             y, y_scale = POSTPROCESS_MX_FN(y, valid_y_s1[None, :], *postprocess_mx_fn_args)
338|             y_mx_ptrs = YMx + offs_s0[:, None] * stride_ymx0 + offs_y_smx1[None, :] * stride_ymx1
339|             tl.store(y_mx_ptrs, y_scale, mask=valid_s0[:, None] & valid_y_smx1[None, :])
340|         else:
341|             y = float_to_flex(y, YFlexExpected, YFlexActual, YFlexChecksum, None, Y, Y_FLEX_SATURATE_INF)
342|             # TODO (phil): keeping for backward compatibility, but will remove !
343|             if POSTPROCESS_FN2 is not None:
344|                 y = POSTPROCESS_FN2(y, *postprocess_fn2_args, target_dtype=Y.dtype.element_ty)
345|         if is_out_fp4:
346|             offs_y_s1 = pid_s1 * (BLOCK_Y_S1 // 2) + tl.arange(0, BLOCK_Y_S1 // 2)
347|             valid_y_s1 = offs_y_s1 < tl.cdiv(Y_S1, 2)
348|         y_ptrs = Y + offs_s0[:, None] * stride_y0 + offs_y_s1[None, :] * stride_y1
349|         tl.store(y_ptrs, y, mask=valid_s0[:, None] & valid_y_s1[None, :])
350| 
351| 
```
**EN:** Loops over `tl.range(0, CHAIN_FACTOR)` with target `blk_s1`. The loop body mainly branches on runtime conditions; prepares intermediate values.

**CN:** 遍历 `tl.range(0, CHAIN_FACTOR)` ，目标变量为 `blk_s1`. 循环体主要根据运行时条件分支; 准备中间值.

### Block 83 — Lines 352-386 (_reduce_forward)
```python
352| @triton.jit(launch_metadata=reduce_launch_metadata)
353| def _reduce_forward(X, stride_xr: tl.int64, stride_x0: tl.int64, stride_x1,  # x tensor (input)
354|                     XMx, stride_xmxr, stride_xmx0, stride_xmx1,  # x mx scale
355|                     Y, stride_y0: tl.int64, stride_y1,  # y tensor (output)
356|                     YMx, stride_ymx0, stride_ymx1,  # y mx scale
357|                     Mask, stride_mr, stride_m0, stride_m1,  # mask tensor
358|                     Scale, stride_sr, stride_s0, stride_s1,  # scale tensor
359|                     UnpaddedBatchSize,  # optional scalar tensor
360|                     # shape (K = reduction dim; S0, IN_S1 = input dims, OUT_S1 = output dims)
361|                     K: tl.constexpr, S0, X_S1, Y_S1,  #
362|                     BLOCK_K: tl.constexpr,
363|                     POSTPROCESS_FN1: tl.constexpr, postprocess_fn1_args,  #
364|                     POSTPROCESS_FN2: tl.constexpr, postprocess_fn2_args,  #
365|                     POSTPROCESS_MX_FN: tl.constexpr, postprocess_mx_fn_args,  #
366|                     XFlex,  # TODO: remove this
367|                     XGlobalScale,  # x global scale
368|                     YFlexExpected, YFlexActual, YFlexChecksum,
369|                     Y_FLEX_SATURATE_INF: tl.constexpr,  # y flex (global) scale
370|                     ActiveInputCounts, RowIdxs,  #
371|                     BROADCAST_R: tl.constexpr,  #
372|                     BROADCAST_S0: tl.constexpr,  #
373|                     BROADCAST_S1: tl.constexpr,  #
374|                     IS_SCALE_NONE: tl.constexpr,  #
375|                     SCALE_BROADCAST_R: tl.constexpr,  #
376|                     SCALE_BROADCAST_S0: tl.constexpr,  #
377|                     SCALE_BROADCAST_S1: tl.constexpr,  #
378|                     USE_STATIC_LOOP: tl.constexpr,  #
379|                     BLOCK_S0: tl.constexpr,  #
380|                     BLOCK_X_S1: tl.constexpr,  #
381|                     BLOCK_Y_S1: tl.constexpr,  #
382|                     CHAIN_FACTOR: tl.constexpr,  #
383|                     Y_MX_BLOCK_SIZE: tl.constexpr,  #
384|                     Y_VALUE_PACK_FACTOR: tl.constexpr,  #
385|                     DIM,  # only used for launch_metadata
386|                     ):
```
**EN:** Defines function `_reduce_forward(X, stride_xr, stride_x0, stride_x1, XMx, stride_xmxr, stride_xmx0, stride_xmx1, Y, stride_y0, stride_y1, YMx, stride_ymx0, stride_ymx1, Mask, stride_mr, stride_m0, stride_m1, Scale, stride_sr, stride_s0, stride_s1, UnpaddedBatchSize, K, S0, X_S1, Y_S1, BLOCK_K, POSTPROCESS_FN1, postprocess_fn1_args, POSTPROCESS_FN2, postprocess_fn2_args, POSTPROCESS_MX_FN, postprocess_mx_fn_args, XFlex, XGlobalScale, YFlexExpected, YFlexActual, YFlexChecksum, Y_FLEX_SATURATE_INF, ActiveInputCounts, RowIdxs, BROADCAST_R, BROADCAST_S0, BROADCAST_S1, IS_SCALE_NONE, SCALE_BROADCAST_R, SCALE_BROADCAST_S0, SCALE_BROADCAST_S1, USE_STATIC_LOOP, BLOCK_S0, BLOCK_X_S1, BLOCK_Y_S1, CHAIN_FACTOR, Y_MX_BLOCK_SIZE, Y_VALUE_PACK_FACTOR, DIM)` with decorators `triton.jit(launch_metadata=reduce_lau...` for this module. The body mainly prepares intermediate values; prepares intermediate values; branches on runtime conditions. It uses calls such as `triton.jit`, `tl.program_id`, `_reduce_forward_inner`, `tl.arange`, `tl.load(UnpaddedBatchSize).to` to implement its workflow.

**CN:** 定义函数 `_reduce_forward(X, stride_xr, stride_x0, stride_x1, XMx, stride_xmxr, stride_xmx0, stride_xmx1, Y, stride_y0, stride_y1, YMx, stride_ymx0, stride_ymx1, Mask, stride_mr, stride_m0, stride_m1, Scale, stride_sr, stride_s0, stride_s1, UnpaddedBatchSize, K, S0, X_S1, Y_S1, BLOCK_K, POSTPROCESS_FN1, postprocess_fn1_args, POSTPROCESS_FN2, postprocess_fn2_args, POSTPROCESS_MX_FN, postprocess_mx_fn_args, XFlex, XGlobalScale, YFlexExpected, YFlexActual, YFlex检查um, Y_FLEX_SATURATE_INF, ActiveInputCounts, RowIdxs, BROADCAST_R, BROADCAST_S0, BROADCAST_S1, IS_SCALE_NONE, SCALE_BROADCAST_R, SCALE_BROADCAST_S0, SCALE_BROADCAST_S1, USE_STATIC_LOOP, BLOCK_S0, BLOCK_X_S1, BLOCK_Y_S1, CHAIN_FACTOR, Y_MX_BLOCK_SIZE, Y_VALUE_PACK_FACTOR, DIM)`，带有装饰器 `triton.jit(launch_metadata=reduce_lau...`，供本模块使用. 主体主要准备中间值; 准备中间值; 根据运行时条件分支. 其中会调用 `triton.jit`, `tl.program_id`, `_reduce_forward_inner`, `tl.arange`, `tl.load(UnpaddedBatchSize).to` 来实现其工作流程.

### Block 84 — Lines 387-387 (_reduce_forward)
```python
387|     pid_s0 = tl.program_id(0)
```
**EN:** Assigns `pid_s0` and calls `tl.program_id`.

**CN:** 将 `pid_s0`，并调用 `tl.program_id`.

### Block 85 — Lines 388-388 (_reduce_forward)
```python
388|     offs_s0 = pid_s0 * BLOCK_S0 + tl.arange(0, BLOCK_S0)
```
**EN:** Assigns `offs_s0` and evaluates `pid_s0 * BLOCK_S0 + tl.arange(0, BLOCK_S0)`.

**CN:** 将 `offs_s0` and 计算 `pid_s0 * BLOCK_S0 + tl.arange(0, BLOCK_S0)`.

### Block 86 — Lines 389-393 (_reduce_forward)
```python
389|     if UnpaddedBatchSize is not None:
390|         unpadded = tl.load(UnpaddedBatchSize).to(tl.int32)
391|         if pid_s0 * BLOCK_S0 >= unpadded:
392|             return
393|         S0 = unpadded
```
**EN:** Checks `UnpaddedBatchSize is not None`. The true branch mainly prepares intermediate values; branches on runtime conditions.

**CN:** 检查 `UnpaddedBatchSize is not None`. 真分支主要准备中间值; 根据运行时条件分支.

### Block 87 — Lines 394-395 (_reduce_forward)
```python
394|     valid_s0 = offs_s0 < S0
395| 
```
**EN:** Assigns `valid_s0` and evaluates `offs_s0 < S0`.

**CN:** 将 `valid_s0` and 计算 `offs_s0 < S0`.

### Block 88 — Lines 396-501 (_reduce_forward)
```python
396|     if ActiveInputCounts is not None:
397|         tl.static_assert(RowIdxs is not None)
398|         offs_s0 = tl.load(RowIdxs + offs_s0, mask=valid_s0, other=0)
399|         n_actives = tl.load(ActiveInputCounts + offs_s0, mask=valid_s0, other=0)
400|         max_actives = tl.max(n_actives)
401| 
402|         if max_actives <= 1:
403|             _reduce_forward_inner(
404|                 X, stride_xr, stride_x0, stride_x1,  #
405|                 XMx, stride_xmxr, stride_xmx0, stride_xmx1,  #
406|                 Y, stride_y0, stride_y1,  #
407|                 YMx, stride_ymx0, stride_ymx1,  #
408|                 Mask, stride_mr, stride_m0, stride_m1,  #
409|                 Scale, stride_sr, stride_s0, stride_s1,  #
410|                 K, S0, X_S1, Y_S1, BLOCK_K,  #
411|                 POSTPROCESS_FN1, postprocess_fn1_args,  #
412|                 POSTPROCESS_FN2, postprocess_fn2_args,  #
413|                 POSTPROCESS_MX_FN, postprocess_mx_fn_args,  #
414|                 XFlex, XGlobalScale,  #
415|                 YFlexExpected, YFlexActual, YFlexChecksum,  #
416|                 Y_FLEX_SATURATE_INF,  #
417|                 ActiveInputCounts, RowIdxs,  #
418|                 BROADCAST_R,  #
419|                 BROADCAST_S0,  #
420|                 BROADCAST_S1,  #
421|                 IS_SCALE_NONE,  #
422|                 SCALE_BROADCAST_R,  #
423|                 SCALE_BROADCAST_S0,  #
424|                 SCALE_BROADCAST_S1,  #
425|                 BLOCK_S0,  #
426|                 BLOCK_X_S1,  #
427|                 BLOCK_Y_S1,  #
428|                 CHAIN_FACTOR,  #
429|                 Y_MX_BLOCK_SIZE,  #
430|                 Y_VALUE_PACK_FACTOR,  #
431|                 USE_STATIC_LOOP=True,
432|                 LIMIT=1,
433|             )
434|             return
435|         elif max_actives == 2:
436|             _reduce_forward_inner(
437|                 X, stride_xr, stride_x0, stride_x1,  #
438|                 XMx, stride_xmxr, stride_xmx0, stride_xmx1,  #
439|                 Y, stride_y0, stride_y1,  #
440|                 YMx, stride_ymx0, stride_ymx1,  #
441|                 Mask, stride_mr, stride_m0, stride_m1,  #
442|                 Scale, stride_sr, stride_s0, stride_s1,  #
443|                 K, S0, X_S1, Y_S1, BLOCK_K,  #
444|                 POSTPROCESS_FN1, postprocess_fn1_args,  #
445|                 POSTPROCESS_FN2, postprocess_fn2_args,  #
446|                 POSTPROCESS_MX_FN, postprocess_mx_fn_args,  #
447|                 XFlex, XGlobalScale,  #
448|                 YFlexExpected, YFlexActual, YFlexChecksum,  #
449|                 Y_FLEX_SATURATE_INF,  #
450|                 ActiveInputCounts, RowIdxs,  #
451|                 BROADCAST_R,  #
452|                 BROADCAST_S0,  #
453|                 BROADCAST_S1,  #
454|                 IS_SCALE_NONE,  #
455|                 SCALE_BROADCAST_R,  #
456|                 SCALE_BROADCAST_S0,  #
457|                 SCALE_BROADCAST_S1,  #
458|                 BLOCK_S0,  #
459|                 BLOCK_X_S1,  #
460|                 BLOCK_Y_S1,  #
461|                 CHAIN_FACTOR,  #
462|                 Y_MX_BLOCK_SIZE,  #
463|                 Y_VALUE_PACK_FACTOR,  #
464|                 USE_STATIC_LOOP=True,
465|                 LIMIT=2,
466|             )
467|             return
468|         elif max_actives == 3:
469|             _reduce_forward_inner(
470|                 X, stride_xr, stride_x0, stride_x1,  #
471|                 XMx, stride_xmxr, stride_xmx0, stride_xmx1,  #
472|                 Y, stride_y0, stride_y1,  #
473|                 YMx, stride_ymx0, stride_ymx1,  #
474|                 Mask, stride_mr, stride_m0, stride_m1,  #
475|                 Scale, stride_sr, stride_s0, stride_s1,  #
476|                 K, S0, X_S1, Y_S1, BLOCK_K,  #
477|                 POSTPROCESS_FN1, postprocess_fn1_args,  #
478|                 POSTPROCESS_FN2, postprocess_fn2_args,  #
479|                 POSTPROCESS_MX_FN, postprocess_mx_fn_args,  #
480|                 XFlex, XGlobalScale,  #
481|                 YFlexExpected, YFlexActual, YFlexChecksum,  #
482|                 Y_FLEX_SATURATE_INF,  #
483|                 ActiveInputCounts, RowIdxs,  #
484|                 BROADCAST_R,  #
485|                 BROADCAST_S0,  #
486|                 BROADCAST_S1,  #
487|                 IS_SCALE_NONE,  #
488|                 SCALE_BROADCAST_R,  #
489|                 SCALE_BROADCAST_S0,  #
490|                 SCALE_BROADCAST_S1,  #
491|                 BLOCK_S0,  #
492|                 BLOCK_X_S1,  #
493|                 BLOCK_Y_S1,  #
494|                 CHAIN_FACTOR,  #
495|                 Y_MX_BLOCK_SIZE,  #
496|                 Y_VALUE_PACK_FACTOR,  #
497|                 USE_STATIC_LOOP=True,
498|                 LIMIT=3,
499|             )
500|             return
501| 
```
**EN:** Checks `ActiveInputCounts is not None`. The true branch mainly invokes `tl.static_assert`; prepares intermediate values.

**CN:** 检查 `ActiveInputCounts is not None`. 真分支主要invokes `tl.static_assert`; 准备中间值.

### Block 89 — Lines 502-534 (_reduce_forward)
```python
502|     _reduce_forward_inner(
503|         X, stride_xr, stride_x0, stride_x1,  #
504|         XMx, stride_xmxr, stride_xmx0, stride_xmx1,  #
505|         Y, stride_y0, stride_y1,  #
506|         YMx, stride_ymx0, stride_ymx1,  #
507|         Mask, stride_mr, stride_m0, stride_m1,  #
508|         Scale, stride_sr, stride_s0, stride_s1,  #
509|         K, S0, X_S1, Y_S1, BLOCK_K,  #
510|         POSTPROCESS_FN1, postprocess_fn1_args,  #
511|         POSTPROCESS_FN2, postprocess_fn2_args,  #
512|         POSTPROCESS_MX_FN, postprocess_mx_fn_args,  #
513|         XFlex, XGlobalScale,  #
514|         YFlexExpected, YFlexActual, YFlexChecksum,  #
515|         Y_FLEX_SATURATE_INF,  #
516|         ActiveInputCounts, RowIdxs,  #
517|         BROADCAST_R,  #
518|         BROADCAST_S0,  #
519|         BROADCAST_S1,  #
520|         IS_SCALE_NONE,  #
521|         SCALE_BROADCAST_R,  #
522|         SCALE_BROADCAST_S0,  #
523|         SCALE_BROADCAST_S1,  #
524|         BLOCK_S0,  #
525|         BLOCK_X_S1,  #
526|         BLOCK_Y_S1,  #
527|         CHAIN_FACTOR,  #
528|         Y_MX_BLOCK_SIZE,  #
529|         Y_VALUE_PACK_FACTOR,  #
530|         USE_STATIC_LOOP=USE_STATIC_LOOP,
531|         LIMIT=K,
532|     )
533| 
534| 
```
**EN:** Calls `_reduce_forward_inner` for side effects, registration, or validation.

**CN:** 调用 `_reduce_forward_inner` ，用于副作用、注册或校验。

### Block 90 — Lines 535-545 (module)
```python
535| forward_specializations = SpecializationModule(
536|     "reduce_forward",
537|     kernels=[("_reduce_forward", _reduce_forward)],
538|     closure_args={
539|         "postprocess_fn1": ClosureArg("POSTPROCESS_FN1", "postprocess_fn1_args"),
540|         "postprocess_fn2": ClosureArg("POSTPROCESS_FN2", "postprocess_fn2_args"),
541|         "postprocess_mx_fn": ClosureArg("POSTPROCESS_MX_FN", "postprocess_mx_fn_args"),
542|     },
543| )
544| 
545| 
```
**EN:** Assigns `forward_specializations` and calls `SpecializationModule`.

**CN:** 将 `forward_specializations`，并调用 `SpecializationModule`.

### Block 91 — Lines 546-566 (reduce_forward)
```python
546| def reduce_forward(
547|     x: torch.Tensor,
548|     dim: int,
549|     mask: Optional[torch.Tensor] = None,
550|     scale: Optional[torch.Tensor] = None,
551|     x_mxscale: Optional[torch.Tensor] = None,
552|     x_flex: Optional[InFlexData] = InFlexData(),
553|     x_global_scale: Optional[torch.Tensor] = None,
554|     y_dtype: Optional[torch.dtype] = None,
555|     y_flex: Optional[OutFlexData] = OutFlexData(),
556|     y_flex_saturate_inf: bool = False,
557|     y_has_mx: Optional[bool] = None,
558|     y_mx_scale_dtype: Optional[torch.dtype] = None,
559|     y_microblock_size: int = MXFP_BLOCK_SIZE.value,
560|     y_value_pack_factor: int = 1,
561|     y: Optional[torch.Tensor] = None,
562|     postprocess_fn1: Optional[PostprocessFn] = None,
563|     # TODO: keeping for backward compatibility, but will remove !
564|     postprocess_fn2: Optional[PostprocessFn] = None,
565|     unpadded_batch_size: Optional[torch.Tensor] = None,
566| ) -> tuple[torch.Tensor, Optional[torch.Tensor]]:
```
**EN:** Defines function `reduce_forward(x, dim, mask, scale, x_mxscale, x_flex, x_global_scale, y_dtype, y_flex, y_flex_saturate_inf, y_has_mx, y_mx_scale_dtype, y_microblock_size, y_value_pack_factor, y, postprocess_fn1, postprocess_fn2, unpadded_batch_size)` for this module. The body mainly branches on runtime conditions; branches on runtime conditions; branches on runtime conditions. It uses calls such as `InFlexData`, `OutFlexData`, `tuple`, `x.stride`, `_get_strides` to implement its workflow.

**CN:** 定义函数 `reduce_forward(x, dim, mask, scale, x_mxscale, x_flex, x_global_scale, y_dtype, y_flex, y_flex_saturate_inf, y_has_mx, y_mx_scale_dtype, y_microblock_size, y_value_pack_factor, y, postprocess_fn1, postprocess_fn2, unpadded_batch_size)`，供本模块使用. 主体主要根据运行时条件分支; 根据运行时条件分支; 根据运行时条件分支. 其中会调用 `InFlexData`, `OutFlexData`, `tuple`, `x.stride`, `_get_strides` 来实现其工作流程.

### Block 92 — Lines 567-592 (reduce_forward)
```python
567|     """
568|     Performs a reduction over the specified dimension of the input tensor,
569|     optionally multiplied by `scale` and ignoring masked elements.
570| 
571|     Arguments:
572|         - x: Tensor
573|           input tensor to reduce.
574|         - dim: int
575|           dimension along which `x` should be reduce.
576|         - mask: Optional[torch.Tensor]
577|           integer mask of the same shape as `x` (or broadcastable to it).
578|           entries that are `0` are ignored in the reduction.
579|           if `mask is None`, all elements are included.
580|         - scale: Optional[torch.Tensor]
581|           scale factors of the same shape as `x` (or broadcastable to it).
582|           the reduction is performed over `x * scale`. If `scale is None`,
583|           a value of 1 is used everywhere.
584|         - unpadded_batch_size: Optional[torch.Tensor]
585|           Optional single-element tensor specifying the number of entries to reduce along the first dimension.
586| 
587|     Returns:
588|         - output: torch.Tensor
589|           The reduced tensor with `dim` removed.
590|         - output_mxscale: Optional[torch.Tensor]
591|           The output mx scale if input is micro-scaled, else None.
592|     """
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 93 — Lines 593-594 (reduce_forward)
```python
593|     if x.ndim != 3:
594|         raise NotImplementedError("reduce only supports 3D inputs in this implementation")
```
**EN:** Checks `x.ndim != 3`..

**CN:** 检查 `x.ndim != 3`..

### Block 94 — Lines 595-596 (reduce_forward)
```python
595|     if dim < 0:
596|         dim += x.ndim
```
**EN:** Checks `dim < 0`..

**CN:** 检查 `dim < 0`..

### Block 95 — Lines 597-598 (reduce_forward)
```python
597|     if dim not in (0, 1, 2):
598|         raise ValueError("dim must be in {0,1,2}")
```
**EN:** Checks `dim not in (0, 1, 2)`..

**CN:** 检查 `dim not in (0, 1, 2)`..

### Block 96 — Lines 599-604 (reduce_forward)
```python
599|     if x_mxscale is not None:
600|         if dim == 2:
601|             raise ValueError("reduction over the micro-scaled dimension not supported")
602|         assert x.shape[:-1] == x_mxscale.shape[:-1]
603|         assert triton.cdiv(x.shape[-1], 32) * 32 == x_mxscale.shape[-1] * 32
604|     # assert not y_flex.is_per_batch
```
**EN:** Checks `x_mxscale is not None`. The true branch mainly branches on runtime conditions; checks invariants.

**CN:** 检查 `x_mxscale is not None`. 真分支主要根据运行时条件分支; 检查不变量.

### Block 97 — Lines 605-606 (reduce_forward)
```python
605|     if postprocess_fn1 is None:
606|         postprocess_fn1 = PostprocessFn()
```
**EN:** Checks `postprocess_fn1 is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `postprocess_fn1 is None`. 真分支主要准备中间值.

### Block 98 — Lines 607-608 (reduce_forward)
```python
607|     if postprocess_fn2 is None:
608|         postprocess_fn2 = PostprocessFn()
```
**EN:** Checks `postprocess_fn2 is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `postprocess_fn2 is None`. 真分支主要准备中间值.

### Block 99 — Lines 609-610 (reduce_forward)
```python
609|     if y_dtype is None:
610|         y_dtype = x.dtype
```
**EN:** Checks `y_dtype is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `y_dtype is None`. 真分支主要准备中间值.

### Block 100 — Lines 611-612 (reduce_forward)
```python
611|     if y_flex is None:
612|         y_flex = OutFlexData()
```
**EN:** Checks `y_flex is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `y_flex is None`. 真分支主要准备中间值.

### Block 101 — Lines 613-614 (reduce_forward)
```python
613|     if x_flex is None:
614|         x_flex = InFlexData()
```
**EN:** Checks `x_flex is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `x_flex is None`. 真分支主要准备中间值.

### Block 102 — Lines 615-616 (reduce_forward)
```python
615|     if y_has_mx is None:
616|         y_has_mx = x_mxscale is not None
```
**EN:** Checks `y_has_mx is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `y_has_mx is None`. 真分支主要准备中间值.

### Block 103 — Lines 617-619 (reduce_forward)
```python
617|     if y_mx_scale_dtype is None:
618|         y_mx_scale_dtype = torch.uint8
619|     # input shapes
```
**EN:** Checks `y_mx_scale_dtype is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `y_mx_scale_dtype is None`. 真分支主要准备中间值.

### Block 104 — Lines 620-620 (reduce_forward)
```python
620|     dims = (0, 1, 2)
```
**EN:** Assigns `dims` and builds a tuple.

**CN:** 将 `dims` and 构造一个元组.

### Block 105 — Lines 621-621 (reduce_forward)
```python
621|     nonred = tuple(d for d in dims if d != dim)
```
**EN:** Assigns `nonred` and calls `tuple`.

**CN:** 将 `nonred`，并调用 `tuple`.

### Block 106 — Lines 622-622 (reduce_forward)
```python
622|     S0, X_S1 = x.shape[nonred[0]], x.shape[nonred[1]]
```
**EN:** Assigns `S0`, `X_S1` and builds a tuple.

**CN:** 将 `S0`, `X_S1` and 构造一个元组.

### Block 107 — Lines 623-623 (reduce_forward)
```python
623|     Y_S1 = X_S1 // postprocess_fn1.specs.reduction_n
```
**EN:** Assigns `Y_S1` and evaluates `X_S1 // postprocess_fn1.specs.reduction_n`.

**CN:** 将 `Y_S1` and 计算 `X_S1 // postprocess_fn1.specs.reduction_n`.

### Block 108 — Lines 624-625 (reduce_forward)
```python
624|     if y is None:
625|         y = torch.empty((S0, Y_S1 // y_value_pack_factor), device=x.device, dtype=y_dtype)
```
**EN:** Checks `y is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `y is None`. 真分支主要准备中间值.

### Block 109 — Lines 626-626 (reduce_forward)
```python
626|     assert y.shape == (S0, Y_S1 // y_value_pack_factor), f"y.shape: {y.shape} != {(S0, Y_S1 // y_value_pack_factor)}"
```
**EN:** Asserts `y.shape == (S0, Y_S1 // y_value_pack_factor)` to enforce invariants.

**CN:** 断言 `y.shape == (S0, Y_S1 // y_value_pack_factor)` 以确保不变量成立。

### Block 110 — Lines 627-627 (reduce_forward)
```python
627|     y_mxscale = None
```
**EN:** Assigns `y_mxscale` and stores constant `None`.

**CN:** 将 `y_mxscale` and 保存常量 `None`.

### Block 111 — Lines 628-630 (reduce_forward)
```python
628|     if y_has_mx:
629|         y_mxscale = torch.empty((S0, triton.cdiv(Y_S1, y_microblock_size)), device=x.device, dtype=y_mx_scale_dtype)
630|     # Strides for X along reduced and non-reduced dims
```
**EN:** Checks `y_has_mx`. The true branch mainly prepares intermediate values.

**CN:** 检查 `y_has_mx`. 真分支主要准备中间值.

### Block 112 — Lines 631-631 (reduce_forward)
```python
631|     stride_xr = x.stride(dim)
```
**EN:** Assigns `stride_xr` and calls `x.stride`.

**CN:** 将 `stride_xr`，并调用 `x.stride`.

### Block 113 — Lines 632-632 (reduce_forward)
```python
632|     stride_x0 = x.stride(nonred[0])
```
**EN:** Assigns `stride_x0` and calls `x.stride`.

**CN:** 将 `stride_x0`，并调用 `x.stride`.

### Block 114 — Lines 633-634 (reduce_forward)
```python
633|     stride_x1 = x.stride(nonred[1])
634|     # Strides for X mx scales
```
**EN:** Assigns `stride_x1` and calls `x.stride`.

**CN:** 将 `stride_x1`，并调用 `x.stride`.

### Block 115 — Lines 635-635 (reduce_forward)
```python
635|     stride_xmxr = None if x_mxscale is None else x_mxscale.stride(dim)
```
**EN:** Assigns `stride_xmxr` and uses conditional expression `None if x_mxscale is None else x_mxscale.stride(dim)`.

**CN:** 将 `stride_xmxr` and 使用条件表达式 `None if x_mxscale is None else x_mxscale.stride(dim)`.

### Block 116 — Lines 636-636 (reduce_forward)
```python
636|     stride_xmx0 = None if x_mxscale is None else x_mxscale.stride(nonred[0])
```
**EN:** Assigns `stride_xmx0` and uses conditional expression `None if x_mxscale is None else x_mxscale.stride(nonred[0])`.

**CN:** 将 `stride_xmx0` and 使用条件表达式 `None if x_mxscale is None else x_mxscale.stride(nonred[0])`.

### Block 117 — Lines 637-638 (reduce_forward)
```python
637|     stride_xmx1 = None if x_mxscale is None else x_mxscale.stride(nonred[1])
638|     # Strides for Y mx scales
```
**EN:** Assigns `stride_xmx1` and uses conditional expression `None if x_mxscale is None else x_mxscale.stride(nonred[1])`.

**CN:** 将 `stride_xmx1` and 使用条件表达式 `None if x_mxscale is None else x_mxscale.stride(nonred[1])`.

### Block 118 — Lines 639-639 (reduce_forward)
```python
639|     stride_ymx0 = None if y_mxscale is None else y_mxscale.stride(0)
```
**EN:** Assigns `stride_ymx0` and uses conditional expression `None if y_mxscale is None else y_mxscale.stride(0)`.

**CN:** 将 `stride_ymx0` and 使用条件表达式 `None if y_mxscale is None else y_mxscale.stride(0)`.

### Block 119 — Lines 640-641 (reduce_forward)
```python
640|     stride_ymx1 = None if y_mxscale is None else y_mxscale.stride(1)
641|     # Mask strides (broadcast allowed via stride 0)
```
**EN:** Assigns `stride_ymx1` and uses conditional expression `None if y_mxscale is None else y_mxscale.stride(1)`.

**CN:** 将 `stride_ymx1` and 使用条件表达式 `None if y_mxscale is None else y_mxscale.stride(1)`.

### Block 120 — Lines 642-643 (reduce_forward)
```python
642|     stride_mr, stride_m0, stride_m1 = _get_strides(mask, dim)
643|     # Scale strides (broadcast allowed via stride 0)
```
**EN:** Assigns `stride_mr`, `stride_m0`, `stride_m1` and calls `_get_strides`.

**CN:** 将 `stride_mr`, `stride_m0`, `stride_m1`，并调用 `_get_strides`.

### Block 121 — Lines 644-644 (reduce_forward)
```python
644|     stride_sr, stride_s0, stride_s1 = _get_strides(scale, dim)
```
**EN:** Assigns `stride_sr`, `stride_s0`, `stride_s1` and calls `_get_strides`.

**CN:** 将 `stride_sr`, `stride_s0`, `stride_s1`，并调用 `_get_strides`.

### Block 122 — Lines 645-645 (reduce_forward)
```python
645|     K = x.shape[dim]
```
**EN:** Assigns `K` and evaluates `x.shape[dim]`.

**CN:** 将 `K` and 计算 `x.shape[dim]`.

### Block 123 — Lines 646-647 (reduce_forward)
```python
646|     mask_chainable = (mask is not None) and (stride_m1 == 0) and (scale is None)
647|     # Always use the 2D tiled kernel with constexpr metaprogramming for mask broadcasting
```
**EN:** Assigns `mask_chainable` and evaluates `mask is not None and stride_m1 == 0 and (scale is None)`.

**CN:** 将 `mask_chainable` and 计算 `mask is not None and stride_m1 == 0 and (scale is None)`.

### Block 124 — Lines 648-648 (reduce_forward)
```python
648|     opt_flags = _opt_flags.get()
```
**EN:** Assigns `opt_flags` and calls `_opt_flags.get`.

**CN:** 将 `opt_flags`，并调用 `_opt_flags.get`.

### Block 125 — Lines 649-655 (reduce_forward)
```python
649|     if opt_flags is None:
650|         opt_flags = _select_reduce_forward_config(
651|             x.dtype,
652|             S0, Y_S1, postprocess_fn1.specs.reduction_n, K,  #
653|             x_mxscale is not None or y_has_mx,
654|             mask_chainable,
655|         )
```
**EN:** Checks `opt_flags is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `opt_flags is None`. 真分支主要准备中间值.

### Block 126 — Lines 656-656 (reduce_forward)
```python
656|     block_y_s1_chain = opt_flags.block_y_s1 * opt_flags.chain_factor
```
**EN:** Assigns `block_y_s1_chain` and evaluates `opt_flags.block_y_s1 * opt_flags.chain_factor`.

**CN:** 将 `block_y_s1_chain` and 计算 `opt_flags.block_y_s1 * opt_flags.chain_factor`.

### Block 127 — Lines 657-657 (reduce_forward)
```python
657|     grid = (triton.cdiv(S0, opt_flags.block_s0), triton.cdiv(Y_S1, block_y_s1_chain))
```
**EN:** Assigns `grid` and builds a tuple.

**CN:** 将 `grid` and 构造一个元组.

### Block 128 — Lines 658-666 (reduce_forward)
```python
658|     if y_has_mx:
659|         if y_dtype == torch.float8_e4m3fn:
660|             postprocess_mx_fn = FnSpecs("quantize_mxfp8", quantize_mxfp8_fn, tuple(), tuple())
661|         elif y_mx_scale_dtype == torch.float8_e4m3fn:
662|             postprocess_mx_fn = FnSpecs("quantize_nvfp4", quantize_nvfp4_fn, tuple(), tuple())
663|         else:
664|             postprocess_mx_fn = FnSpecs("quantize_mxfp4", quantize_mxfp4_fn, tuple(), tuple())
665|     else:
666|         postprocess_mx_fn = FnSpecs.default()
```
**EN:** Checks `y_has_mx`. The true branch mainly branches on runtime conditions, while the else branch prepares intermediate values.

**CN:** 检查 `y_has_mx`. 真分支主要根据运行时条件分支；而 else 分支准备中间值.

### Block 129 — Lines 667-670 (reduce_forward)
```python
667|     reduce_kernel = forward_specializations.get(postprocess_fn1=postprocess_fn1.specs,
668|                                                 postprocess_fn2=postprocess_fn2.specs,
669|                                                 postprocess_mx_fn=postprocess_mx_fn)._reduce_forward
670| 
```
**EN:** Assigns `reduce_kernel` and references `forward_specializations.get(postprocess_fn1=postprocess_f...`.

**CN:** 将 `reduce_kernel` and 引用 `forward_specializations.get(postprocess_fn1=postprocess_f...`.

### Block 130 — Lines 671-703 (reduce_forward)
```python
671|     if (
672|         mask_chainable
673|         and K <= 32
674|         and target_info.is_cuda()
675|         and torch.cuda.get_device_capability()[0] >= 9
676|     ):
677|         # Optimize for the common case with per-row mask and most output rows
678|         # are expected to have one corresponding input row.
679|         # (We're using libdevice.ffs() so let's require is_cuda().)
680|         assert S0 < 2**32  # Just in case.
681|         assert stride_m1 == 0
682|         active_input_counts = torch.empty((S0,), dtype=torch.int16, device=x.device)
683|         row_idxs = torch.empty((S0,), dtype=torch.int32, device=x.device)
684|         block_m = opt_flags.block_s0
685|         num_warps = 8
686| 
687|         # Choose block_m as large as possible, while being able to load the mask.
688|         block_k = triton.next_power_of_2(K)
689|         while (block_m * 2) * block_k <= 128 * (32 * num_warps) and S0 > block_m * 20:
690|             block_m *= 2
691|         _create_row_idxs[(triton.cdiv(S0, block_m),)](
692|             mask, stride_mr, stride_m0,  #
693|             active_input_counts, row_idxs,  #
694|             S0, unpadded_batch_size,  #
695|             K,
696|             BLOCK_M=block_m,
697|             BLOCK_K=block_k,
698|             BLOCK_S0=opt_flags.block_s0,
699|             num_warps=num_warps,
700|         )
701|     else:
702|         active_input_counts = row_idxs = None
703| 
```
**EN:** Checks `mask_chainable and K <= 32 and target_info.is_cuda() and (torch.cuda.get_device_capabil...`. The true branch mainly checks invariants; checks invariants, while the else branch prepares intermediate values.

**CN:** 检查 `mask_chainable and K <= 32 and target_info.is_cuda() and (torch.cuda.get_device_capabil...`. 真分支主要检查不变量; 检查不变量；而 else 分支准备中间值.

### Block 131 — Lines 704-734 (reduce_forward)
```python
704|     reduce_kernel[grid](
705|         x_flex.reinterpret(x), stride_xr, stride_x0, stride_x1,  #
706|         x_mxscale, stride_xmxr, stride_xmx0, stride_xmx1,  #
707|         y_flex.reinterpret(y), y.stride(0), y.stride(1),  #
708|         y_mxscale, stride_ymx0, stride_ymx1,  #
709|         mask, stride_mr, stride_m0, stride_m1,  #
710|         scale, stride_sr, stride_s0, stride_s1,  #
711|         unpadded_batch_size,  #
712|         K, S0, X_S1, Y_S1,  #
713|         triton.next_power_of_2(K),  #
714|         *postprocess_fn1.fn_args, *postprocess_fn2.fn_args,  #
715|         x_flex.scale, x_global_scale, y_flex.expected_scale, y_flex.actual_scale, y_flex.checksum_scale,  #
716|         y_flex_saturate_inf,  #
717|         active_input_counts, row_idxs,  #
718|         BROADCAST_R=(stride_mr == 0),  #
719|         BROADCAST_S0=(stride_m0 == 0),  #
720|         BROADCAST_S1=(stride_m1 == 0),  #
721|         IS_SCALE_NONE=(scale is None),  #
722|         SCALE_BROADCAST_R=(stride_sr == 0),  #
723|         SCALE_BROADCAST_S0=(stride_s0 == 0),  #
724|         SCALE_BROADCAST_S1=(stride_s1 == 0),  #
725|         USE_STATIC_LOOP=opt_flags.use_static_loop,  #
726|         BLOCK_S0=opt_flags.block_s0,  #
727|         BLOCK_X_S1=opt_flags.block_x_s1,  #
728|         BLOCK_Y_S1=opt_flags.block_y_s1,  #
729|         CHAIN_FACTOR=opt_flags.chain_factor,
730|         Y_MX_BLOCK_SIZE=y_microblock_size,  #
731|         Y_VALUE_PACK_FACTOR=y_value_pack_factor,  #
732|         DIM=dim,  #
733|         num_warps=opt_flags.num_warps  #
734|     )
```
**EN:** Calls `reduce_kernel[grid]` for side effects, registration, or validation.

**CN:** 调用 `reduce_kernel[grid]` ，用于副作用、注册或校验。

### Block 132 — Lines 735-740 (reduce_forward)
```python
735|     return y, y_mxscale
736| 
737| 
738| # ------------------------------------------------------------
739| 
740| 
```
**EN:** Returns `(y, y_mxscale)`.

**CN:** 返回 `(y, y_mxscale)`.

### Block 133 — Lines 741-780 (_reduce_backward)
```python
741| @triton.jit
742| def _reduce_backward(
743|     dY,
744|     stride_y0: tl.int64,
745|     stride_y1,  # upstream grad (S0, Y_S1)
746|     dX,
747|     stride_xr: tl.int64,
748|     stride_x0: tl.int64,
749|     stride_x1,  # grad wrt X (K, S0, X_S1) in the chosen layout
750|     XMx,
751|     stride_xmxr,
752|     stride_xmx0,
753|     stride_xmx1,  # input micro-scales (optional)
754|     Mask,
755|     stride_mr,
756|     stride_m0,
757|     stride_m1,  # mask (optional)
758|     Scale,
759|     stride_sr,
760|     stride_s0,
761|     stride_s1,  # scale (optional)
762|     UnpaddedBatchSize,  # optional scalar tensor
763|     K,
764|     S0,
765|     X_S1,
766|     Y_S1,  # shapes
767|     XFlex,  # global input flex scale (scalar device buffer)
768|     IS_MASK_NONE: tl.constexpr,
769|     BROADCAST_R: tl.constexpr,
770|     BROADCAST_S0: tl.constexpr,
771|     BROADCAST_S1: tl.constexpr,
772|     IS_SCALE_NONE: tl.constexpr,
773|     SCALE_BROADCAST_R: tl.constexpr,
774|     SCALE_BROADCAST_S0: tl.constexpr,
775|     SCALE_BROADCAST_S1: tl.constexpr,
776|     REDUCTION_N: tl.constexpr,  # maps X_S1 -> Y_S1 (grouped sum in fwd)
777|     BLOCK_S0: tl.constexpr,
778|     BLOCK_X_S1: tl.constexpr,
779| ):
780|     # Tile over (S0, X_S1). We loop over the reduction K dimension.
```
**EN:** Defines function `_reduce_backward(dY, stride_y0, stride_y1, dX, stride_xr, stride_x0, stride_x1, XMx, stride_xmxr, stride_xmx0, stride_xmx1, Mask, stride_mr, stride_m0, stride_m1, Scale, stride_sr, stride_s0, stride_s1, UnpaddedBatchSize, K, S0, X_S1, Y_S1, XFlex, IS_MASK_NONE, BROADCAST_R, BROADCAST_S0, BROADCAST_S1, IS_SCALE_NONE, SCALE_BROADCAST_R, SCALE_BROADCAST_S0, SCALE_BROADCAST_S1, REDUCTION_N, BLOCK_S0, BLOCK_X_S1)` with decorators `triton.jit` for this module. The body mainly prepares intermediate values; prepares intermediate values; invokes `tl.static_assert`. It uses calls such as `tl.program_id`, `tl.static_assert`, `tl.load(dy_ptrs, mask=valid_s0[:, Non...`, `load_scale`, `tl.range` to implement its workflow.

**CN:** 定义函数 `_reduce_backward(dY, stride_y0, stride_y1, dX, stride_xr, stride_x0, stride_x1, XMx, stride_xmxr, stride_xmx0, stride_xmx1, Mask, stride_mr, stride_m0, stride_m1, Scale, stride_sr, stride_s0, stride_s1, UnpaddedBatchSize, K, S0, X_S1, Y_S1, XFlex, IS_MASK_NONE, BROADCAST_R, BROADCAST_S0, BROADCAST_S1, IS_SCALE_NONE, SCALE_BROADCAST_R, SCALE_BROADCAST_S0, SCALE_BROADCAST_S1, REDUCTION_N, BLOCK_S0, BLOCK_X_S1)`，带有装饰器 `triton.jit`，供本模块使用. 主体主要准备中间值; 准备中间值; invokes `tl.static_assert`. 其中会调用 `tl.program_id`, `tl.static_assert`, `tl.load(dy_ptrs, mask=valid_s0[:, Non...`, `load_scale`, `tl.range` 来实现其工作流程.

### Block 134 — Lines 781-781 (_reduce_backward)
```python
781|     pid_s0 = tl.program_id(0)
```
**EN:** Assigns `pid_s0` and calls `tl.program_id`.

**CN:** 将 `pid_s0`，并调用 `tl.program_id`.

### Block 135 — Lines 782-783 (_reduce_backward)
```python
782|     pid_s1 = tl.program_id(1)
783| 
```
**EN:** Assigns `pid_s1` and calls `tl.program_id`.

**CN:** 将 `pid_s1`，并调用 `tl.program_id`.

### Block 136 — Lines 784-784 (_reduce_backward)
```python
784|     tl.static_assert(BLOCK_X_S1 % 32 == 0)
```
**EN:** Calls `tl.static_assert` for side effects, registration, or validation.

**CN:** 调用 `tl.static_assert` ，用于副作用、注册或校验。

### Block 137 — Lines 785-786 (_reduce_backward)
```python
785|     BLOCK_X_SMX1: tl.constexpr = BLOCK_X_S1 // 32
786| 
```
**EN:** Annotated assignment stores `BLOCK_X_SMX1` and evaluates `BLOCK_X_S1 // 32`.

**CN:** 带类型注解的赋值保存 `BLOCK_X_SMX1` and 计算 `BLOCK_X_S1 // 32`.

### Block 138 — Lines 787-787 (_reduce_backward)
```python
787|     offs_s0 = pid_s0 * BLOCK_S0 + tl.arange(0, BLOCK_S0)
```
**EN:** Assigns `offs_s0` and evaluates `pid_s0 * BLOCK_S0 + tl.arange(0, BLOCK_S0)`.

**CN:** 将 `offs_s0` and 计算 `pid_s0 * BLOCK_S0 + tl.arange(0, BLOCK_S0)`.

### Block 139 — Lines 788-788 (_reduce_backward)
```python
788|     offs_x_s1 = pid_s1 * BLOCK_X_S1 + tl.arange(0, BLOCK_X_S1)
```
**EN:** Assigns `offs_x_s1` and evaluates `pid_s1 * BLOCK_X_S1 + tl.arange(0, BLOCK_X_S1)`.

**CN:** 将 `offs_x_s1` and 计算 `pid_s1 * BLOCK_X_S1 + tl.arange(0, BLOCK_X_S1)`.

### Block 140 — Lines 789-790 (_reduce_backward)
```python
789|     offs_x_smx1 = pid_s1 * BLOCK_X_SMX1 + tl.arange(0, BLOCK_X_SMX1)
790| 
```
**EN:** Assigns `offs_x_smx1` and evaluates `pid_s1 * BLOCK_X_SMX1 + tl.arange(0, BLOCK_X_SMX1)`.

**CN:** 将 `offs_x_smx1` and 计算 `pid_s1 * BLOCK_X_SMX1 + tl.arange(0, BLOCK_X_SMX1)`.

### Block 141 — Lines 791-797 (_reduce_backward)
```python
791|     if UnpaddedBatchSize is not None:
792|         unpadded = tl.load(UnpaddedBatchSize).to(tl.int32)
793|         if pid_s0 * BLOCK_S0 >= unpadded:
794|             return
795|         valid_s0 = offs_s0 < unpadded
796|     else:
797|         valid_s0 = offs_s0 < S0
```
**EN:** Checks `UnpaddedBatchSize is not None`. The true branch mainly prepares intermediate values; branches on runtime conditions, while the else branch prepares intermediate values.

**CN:** 检查 `UnpaddedBatchSize is not None`. 真分支主要准备中间值; 根据运行时条件分支；而 else 分支准备中间值.

### Block 142 — Lines 798-798 (_reduce_backward)
```python
798|     valid_x_s1 = offs_x_s1 < X_S1
```
**EN:** Assigns `valid_x_s1` and evaluates `offs_x_s1 < X_S1`.

**CN:** 将 `valid_x_s1` and 计算 `offs_x_s1 < X_S1`.

### Block 143 — Lines 799-801 (_reduce_backward)
```python
799|     valid_in_smx1 = offs_x_smx1 < tl.cdiv(X_S1, 32)
800| 
801|     # Map X_S1 positions to their Y_S1 group index (grouped-sum fwd)
```
**EN:** Assigns `valid_in_smx1` and evaluates `offs_x_smx1 < tl.cdiv(X_S1, 32)`.

**CN:** 将 `valid_in_smx1` and 计算 `offs_x_smx1 < tl.cdiv(X_S1, 32)`.

### Block 144 — Lines 802-802 (_reduce_backward)
```python
802|     offs_y_from_x = offs_x_s1 // REDUCTION_N
```
**EN:** Assigns `offs_y_from_x` and evaluates `offs_x_s1 // REDUCTION_N`.

**CN:** 将 `offs_y_from_x` and 计算 `offs_x_s1 // REDUCTION_N`.

### Block 145 — Lines 803-805 (_reduce_backward)
```python
803|     valid_y_from_x = offs_y_from_x < Y_S1
804| 
805|     # Load upstream grad; broadcasting over the REDUCTION_N group happens via indexing.
```
**EN:** Assigns `valid_y_from_x` and evaluates `offs_y_from_x < Y_S1`.

**CN:** 将 `valid_y_from_x` and 计算 `offs_y_from_x < Y_S1`.

### Block 146 — Lines 806-806 (_reduce_backward)
```python
806|     dy_ptrs = dY + offs_s0[:, None] * stride_y0 + offs_y_from_x[None, :] * stride_y1
```
**EN:** Assigns `dy_ptrs` and evaluates `dY + offs_s0[:, None] * stride_y0 + offs_y_from_x[None, :] * stride_y1`.

**CN:** 将 `dy_ptrs` and 计算 `dY + offs_s0[:, None] * stride_y0 + offs_y_from_x[None, :] * stride_y1`.

### Block 147 — Lines 807-809 (_reduce_backward)
```python
807|     dy = tl.load(dy_ptrs, mask=valid_s0[:, None] & valid_y_from_x[None, :], other=0.0).to(tl.float32)
808| 
809|     # Global flex scale (scalar)
```
**EN:** Assigns `dy` and calls `tl.load(dy_ptrs, mask=valid_s0[:, None] & valid...`.

**CN:** 将 `dy`，并调用 `tl.load(dy_ptrs, mask=valid_s0[:, None] & valid...`.

### Block 148 — Lines 810-812 (_reduce_backward)
```python
810|     x_flex_scale = load_scale(XFlex)
811| 
812|     # Loop over the reduced dimension
```
**EN:** Assigns `x_flex_scale` and calls `load_scale`.

**CN:** 将 `x_flex_scale`，并调用 `load_scale`.

### Block 149 — Lines 813-843 (_reduce_backward)
```python
813|     for k in tl.range(0, K, num_stages=2):
814|         g = dy
815|         # Multiply by input micro-scale per group of 32 lanes if present
816|         if XMx is not None:
817|             xmx_ptrs = XMx + k * stride_xmxr + offs_s0[:, None] * stride_xmx0 + offs_x_smx1[None, :] * stride_xmx1
818|             xmx = tl.load(xmx_ptrs, mask=valid_s0[:, None] & valid_in_smx1[None, :], other=0)
819|             xmx = (xmx.to(tl.uint32) << 23).to(tl.float32, bitcast=True)
820|             g = (g.reshape([BLOCK_S0, BLOCK_X_S1 // 32, 32]) * xmx[:, :, None]).reshape([BLOCK_S0, BLOCK_X_S1])
821|         # Multiply by global input flex scale
822|         g = g * x_flex_scale
823|         # Multiply by per-element Scale if provided
824|         if not IS_SCALE_NONE:
825|             k_term_s = 0 if SCALE_BROADCAST_R else (k * stride_sr)
826|             s0_term_s = 0 if SCALE_BROADCAST_S0 else (offs_s0[:, None] * stride_s0)
827|             s1_term_s = 0 if SCALE_BROADCAST_S1 else (offs_x_s1[None, :] * stride_s1)
828|             s_ptrs = Scale + k_term_s + s0_term_s + s1_term_s
829|             s = tl.load(s_ptrs, mask=valid_s0[:, None] & valid_x_s1[None, :], other=1)
830|             g = g * s
831|         # Apply mask if provided
832|         if not IS_MASK_NONE:
833|             k_term = 0 if BROADCAST_R else (k * stride_mr)
834|             s0_term = 0 if BROADCAST_S0 else (offs_s0[:, None] * stride_m0)
835|             s1_term = 0 if BROADCAST_S1 else (offs_x_s1[None, :] * stride_m1)
836|             m_ptrs = Mask + k_term + s0_term + s1_term
837|             m = tl.load(m_ptrs, mask=valid_s0[:, None] & valid_x_s1[None, :], other=1)
838|             g = tl.where(m != 0, g, 0.0)
839|         #
840|         dx_ptrs = dX + k * stride_xr + offs_s0[:, None] * stride_x0 + offs_x_s1[None, :] * stride_x1
841|         tl.store(dx_ptrs, g, mask=valid_s0[:, None] & valid_x_s1[None, :])
842| 
843| 
```
**EN:** Loops over `tl.range(0, K, num_stages=2)` with target `k`. The loop body mainly prepares intermediate values; branches on runtime conditions.

**CN:** 遍历 `tl.range(0, K, num_stages=2)` ，目标变量为 `k`. 循环体主要准备中间值; 根据运行时条件分支.

### Block 150 — Lines 844-861 (reduce_backward)
```python
844| def reduce_backward(
845|     dy: torch.Tensor,
846|     x_shape: tuple[int, int, int],
847|     dim: int,
848|     *,
849|     mask: Optional[torch.Tensor],
850|     scale: Optional[torch.Tensor],
851|     x_mxscale: Optional[torch.Tensor],
852|     x_flex: Optional[InFlexData],
853|     postprocess_fn1: Optional[PostprocessFn],
854|     x_strides: tuple[int, int, int],
855|     x_mx_strides: Optional[tuple[int, int, int]],
856|     mask_strides: Optional[tuple[int, int, int]],
857|     scale_strides: Optional[tuple[int, int, int]],
858|     dx: torch.Tensor,
859|     unpadded_batch_size: Optional[torch.Tensor] = None,
860| ):
861|     # Shapes/axes handling mirrors `reduce(...)`
```
**EN:** Defines function `reduce_backward(dy, x_shape, dim, mask, scale, x_mxscale, x_flex, postprocess_fn1, x_strides, x_mx_strides, mask_strides, scale_strides, dx, unpadded_batch_size)` for this module. The body mainly branches on runtime conditions; prepares intermediate values; prepares intermediate values. It uses calls such as `tuple`, `x_flex.reinterpret`, `_get_strides`, `_reduce_backward[grid]`, `triton.cdiv` to implement its workflow.

**CN:** 定义函数 `reduce_backward(dy, x_shape, dim, mask, scale, x_mxscale, x_flex, postprocess_fn1, x_strides, x_mx_strides, mask_strides, scale_strides, dx, unpadded_batch_size)`，供本模块使用. 主体主要根据运行时条件分支; 准备中间值; 准备中间值. 其中会调用 `tuple`, `x_flex.reinterpret`, `_get_strides`, `_reduce_backward[grid]`, `triton.cdiv` 来实现其工作流程.

### Block 151 — Lines 862-863 (reduce_backward)
```python
862|     if dim < 0:
863|         dim += 3
```
**EN:** Checks `dim < 0`..

**CN:** 检查 `dim < 0`..

### Block 152 — Lines 864-864 (reduce_backward)
```python
864|     dims = (0, 1, 2)
```
**EN:** Assigns `dims` and builds a tuple.

**CN:** 将 `dims` and 构造一个元组.

### Block 153 — Lines 865-866 (reduce_backward)
```python
865|     nonred = tuple(d for d in dims if d != dim)
866| 
```
**EN:** Assigns `nonred` and calls `tuple`.

**CN:** 将 `nonred`，并调用 `tuple`.

### Block 154 — Lines 867-867 (reduce_backward)
```python
867|     S0, X_S1 = x_shape[nonred[0]], x_shape[nonred[1]]
```
**EN:** Assigns `S0`, `X_S1` and builds a tuple.

**CN:** 将 `S0`, `X_S1` and 构造一个元组.

### Block 155 — Lines 868-870 (reduce_backward)
```python
868|     K = x_shape[dim]
869| 
870|     # Postprocess grouping (grouped sum). Default is identity (1).
```
**EN:** Assigns `K` and evaluates `x_shape[dim]`.

**CN:** 将 `K` and 计算 `x_shape[dim]`.

### Block 156 — Lines 871-871 (reduce_backward)
```python
871|     reduction_n = (postprocess_fn1.specs.reduction_n if postprocess_fn1 is not None else FnSpecs.default().reduction_n)
```
**EN:** Assigns `reduction_n` and uses conditional expression `postprocess_fn1.specs.reduction_n if postprocess_fn1 is n...`.

**CN:** 将 `reduction_n` and 使用条件表达式 `postprocess_fn1.specs.reduction_n if postprocess_fn1 is n...`.

### Block 157 — Lines 872-872 (reduce_backward)
```python
872|     Y_S1 = X_S1 // reduction_n
```
**EN:** Assigns `Y_S1` and evaluates `X_S1 // reduction_n`.

**CN:** 将 `Y_S1` and 计算 `X_S1 // reduction_n`.

### Block 158 — Lines 873-876 (reduce_backward)
```python
873|     assert dy.shape == (S0, Y_S1), f"dY shape {dy.shape} mismatch with (S0={S0}, Y_S1={Y_S1})"
874| 
875|     # Strides for dX must match the element size of the tensor passed to the kernel.
876|     # If we reinterpret the dtype (e.g., flex/float8), use the reinterpreted view's strides.
```
**EN:** Asserts `dy.shape == (S0, Y_S1)` to enforce invariants.

**CN:** 断言 `dy.shape == (S0, Y_S1)` 以确保不变量成立。

### Block 159 — Lines 877-877 (reduce_backward)
```python
877|     dx_view = x_flex.reinterpret(dx)
```
**EN:** Assigns `dx_view` and calls `x_flex.reinterpret`.

**CN:** 将 `dx_view`，并调用 `x_flex.reinterpret`.

### Block 160 — Lines 878-878 (reduce_backward)
```python
878|     stride_xr, stride_x0, stride_x1 = _get_strides(dx_view, dim)
```
**EN:** Assigns `stride_xr`, `stride_x0`, `stride_x1` and calls `_get_strides`.

**CN:** 将 `stride_xr`, `stride_x0`, `stride_x1`，并调用 `_get_strides`.

### Block 161 — Lines 879-879 (reduce_backward)
```python
879|     stride_xmxr = stride_xmx0 = stride_xmx1 = 0
```
**EN:** Assigns `stride_xmxr`, `stride_xmx0`, `stride_xmx1` and stores constant `0`.

**CN:** 将 `stride_xmxr`, `stride_xmx0`, `stride_xmx1` and 保存常量 `0`.

### Block 162 — Lines 880-882 (reduce_backward)
```python
880|     if x_mxscale is not None:
881|         stride_xmxr, stride_xmx0, stride_xmx1 = x_mx_strides
882| 
```
**EN:** Checks `x_mxscale is not None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `x_mxscale is not None`. 真分支主要准备中间值.

### Block 163 — Lines 883-883 (reduce_backward)
```python
883|     stride_mr, stride_m0, stride_m1 = _get_strides(mask, dim, mask_strides)
```
**EN:** Assigns `stride_mr`, `stride_m0`, `stride_m1` and calls `_get_strides`.

**CN:** 将 `stride_mr`, `stride_m0`, `stride_m1`，并调用 `_get_strides`.

### Block 164 — Lines 884-886 (reduce_backward)
```python
884|     stride_sr, stride_s0, stride_s1 = _get_strides(scale, dim, scale_strides)
885| 
886|     # Launch configuration mirrors forward (but we tile over X_S1, not Y_S1)
```
**EN:** Assigns `stride_sr`, `stride_s0`, `stride_s1` and calls `_get_strides`.

**CN:** 将 `stride_sr`, `stride_s0`, `stride_s1`，并调用 `_get_strides`.

### Block 165 — Lines 887-887 (reduce_backward)
```python
887|     BLOCK_S0 = 64
```
**EN:** Assigns `BLOCK_S0` and stores constant `64`.

**CN:** 将 `BLOCK_S0` and 保存常量 `64`.

### Block 166 — Lines 888-888 (reduce_backward)
```python
888|     BLOCK_X_S1 = 128
```
**EN:** Assigns `BLOCK_X_S1` and stores constant `128`.

**CN:** 将 `BLOCK_X_S1` and 保存常量 `128`.

### Block 167 — Lines 889-890 (reduce_backward)
```python
889|     grid = (triton.cdiv(S0, BLOCK_S0), triton.cdiv(X_S1, BLOCK_X_S1))
890| 
```
**EN:** Assigns `grid` and builds a tuple.

**CN:** 将 `grid` and 构造一个元组.

### Block 168 — Lines 891-933 (reduce_backward)
```python
891|     _reduce_backward[grid](
892|         dy,
893|         dy.stride(0),
894|         dy.stride(1),
895|         dx_view,
896|         stride_xr,
897|         stride_x0,
898|         stride_x1,
899|         x_mxscale,
900|         stride_xmxr,
901|         stride_xmx0,
902|         stride_xmx1,
903|         mask,
904|         stride_mr,
905|         stride_m0,
906|         stride_m1,
907|         scale,
908|         stride_sr,
909|         stride_s0,
910|         stride_s1,
911|         unpadded_batch_size,
912|         K,
913|         S0,
914|         X_S1,
915|         Y_S1,
916|         x_flex.scale,
917|         IS_MASK_NONE=(mask is None),
918|         BROADCAST_R=(stride_mr == 0),
919|         BROADCAST_S0=(stride_m0 == 0),
920|         BROADCAST_S1=(stride_m1 == 0),
921|         IS_SCALE_NONE=(scale is None),
922|         SCALE_BROADCAST_R=(stride_sr == 0),
923|         SCALE_BROADCAST_S0=(stride_s0 == 0),
924|         SCALE_BROADCAST_S1=(stride_s1 == 0),
925|         REDUCTION_N=reduction_n,
926|         BLOCK_S0=BLOCK_S0,
927|         BLOCK_X_S1=BLOCK_X_S1,
928|         num_warps=4,
929|     )
930| 
931| 
932| # ------------------------------------------------------------
933| 
```
**EN:** Calls `_reduce_backward[grid]` for side effects, registration, or validation.

**CN:** 调用 `_reduce_backward[grid]` ，用于副作用、注册或校验。

### Block 169 — Lines 934-944 (module)
```python
934| backward_specializations = SpecializationModule(
935|     "reduce_backward",
936|     kernels=[("_reduce_backward", _reduce_backward)],
937|     closure_args={
938|         "postprocess_fn1": ClosureArg("POSTPROCESS_FN1", "postprocess_fn1_args"),
939|         "postprocess_fn2": ClosureArg("POSTPROCESS_FN2", "postprocess_fn2_args"),
940|         "postprocess_mx_fn": ClosureArg("POSTPROCESS_MX_FN", "postprocess_mx_fn_args"),
941|     },
942| )
943| 
944| 
```
**EN:** Assigns `backward_specializations` and calls `SpecializationModule`.

**CN:** 将 `backward_specializations`，并调用 `SpecializationModule`.

### Block 170 — Lines 945-946 (_ReduceAutograd)
```python
945| class _ReduceAutograd(torch.autograd.Function):
946| 
```
**EN:** Defines class `_ReduceAutograd` inheriting from `torch.autograd.Function` to organize related behavior. Main methods are `forward`, `backward`.

**CN:** 定义类 `_ReduceAutograd`，继承自 `torch.autograd.Function`，用于组织相关行为。主要方法有 `forward`, `backward`.

### Block 171 — Lines 947-954 (forward)
```python
947|     @staticmethod
948|     def forward(ctx, x: torch.Tensor, dim: int, mask: Optional[torch.Tensor], scale: Optional[torch.Tensor],
949|                 x_mxscale: Optional[torch.Tensor], x_flex: Optional[InFlexData], x_global_scale: Optional[torch.Tensor],
950|                 y_dtype: Optional[torch.dtype], y_flex: Optional[OutFlexData], y_flex_saturate_inf: bool,
951|                 y_has_mx: Optional[bool], y_mx_scale_dtype: Optional[torch.dtype], y_microblock_size: int,
952|                 y_value_pack_factor: int, y: Optional[torch.Tensor], postprocess_fn1: Optional[PostprocessFn],
953|                 postprocess_fn2: Optional[PostprocessFn], unpadded_batch_size: Optional[torch.Tensor]):
954|         # Run your existing Triton forward
```
**EN:** Defines function `forward(ctx, x, dim, mask, scale, x_mxscale, x_flex, x_global_scale, y_dtype, y_flex, y_flex_saturate_inf, y_has_mx, y_mx_scale_dtype, y_microblock_size, y_value_pack_factor, y, postprocess_fn1, postprocess_fn2, unpadded_batch_size)` with decorators `staticmethod` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `reduce_forward`, `tuple`, `bool`, `InFlexData`, `PostprocessFn` to implement its workflow.

**CN:** 定义函数 `forward(ctx, x, dim, mask, scale, x_mxscale, x_flex, x_global_scale, y_dtype, y_flex, y_flex_saturate_inf, y_has_mx, y_mx_scale_dtype, y_microblock_size, y_value_pack_factor, y, postprocess_fn1, postprocess_fn2, unpadded_batch_size)`，带有装饰器 `staticmethod`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `reduce_forward`, `tuple`, `bool`, `InFlexData`, `PostprocessFn` 来实现其工作流程.

### Block 172 — Lines 955-976 (forward)
```python
955|         y, y_mx = reduce_forward(
956|             x=x,
957|             dim=dim,
958|             mask=mask,
959|             scale=scale,
960|             x_mxscale=x_mxscale,
961|             x_flex=x_flex,
962|             x_global_scale=x_global_scale,
963|             y_dtype=y_dtype,
964|             y_flex=y_flex,
965|             y_flex_saturate_inf=y_flex_saturate_inf,
966|             y_has_mx=y_has_mx,
967|             y_mx_scale_dtype=y_mx_scale_dtype,
968|             y_microblock_size=y_microblock_size,
969|             y_value_pack_factor=y_value_pack_factor,
970|             y=y,
971|             postprocess_fn1=postprocess_fn1,
972|             postprocess_fn2=postprocess_fn2,
973|             unpadded_batch_size=unpadded_batch_size,
974|         )
975| 
976|         # Save everything needed for backward (no tensors are modified)
```
**EN:** Assigns `y`, `y_mx` and calls `reduce_forward`.

**CN:** 将 `y`, `y_mx`，并调用 `reduce_forward`.

### Block 173 — Lines 977-977 (forward)
```python
977|         ctx.dim = dim
```
**EN:** Assigns `ctx.dim` and references `dim`.

**CN:** 将 `ctx.dim` and 引用 `dim`.

### Block 174 — Lines 978-978 (forward)
```python
978|         ctx.x_shape = tuple(x.shape)
```
**EN:** Assigns `ctx.x_shape` and calls `tuple`.

**CN:** 将 `ctx.x_shape`，并调用 `tuple`.

### Block 175 — Lines 979-979 (forward)
```python
979|         ctx.x_dtype = x.dtype
```
**EN:** Assigns `ctx.x_dtype` and references `x.dtype`.

**CN:** 将 `ctx.x_dtype` and 引用 `x.dtype`.

### Block 176 — Lines 980-980 (forward)
```python
980|         ctx.device = x.device
```
**EN:** Assigns `ctx.device` and references `x.device`.

**CN:** 将 `ctx.device` and 引用 `x.device`.

### Block 177 — Lines 981-981 (forward)
```python
981|         ctx.mask = mask
```
**EN:** Assigns `ctx.mask` and references `mask`.

**CN:** 将 `ctx.mask` and 引用 `mask`.

### Block 178 — Lines 982-982 (forward)
```python
982|         ctx.scale = scale
```
**EN:** Assigns `ctx.scale` and references `scale`.

**CN:** 将 `ctx.scale` and 引用 `scale`.

### Block 179 — Lines 983-983 (forward)
```python
983|         ctx.x_mxscale = x_mxscale
```
**EN:** Assigns `ctx.x_mxscale` and references `x_mxscale`.

**CN:** 将 `ctx.x_mxscale` and 引用 `x_mxscale`.

### Block 180 — Lines 984-984 (forward)
```python
984|         ctx.x_flex = x_flex if x_flex is not None else InFlexData()
```
**EN:** Assigns `ctx.x_flex` and uses conditional expression `x_flex if x_flex is not None else InFlexData()`.

**CN:** 将 `ctx.x_flex` and 使用条件表达式 `x_flex if x_flex is not None else InFlexData()`.

### Block 181 — Lines 985-985 (forward)
```python
985|         ctx.postprocess_fn1 = postprocess_fn1 if postprocess_fn1 is not None else PostprocessFn()
```
**EN:** Assigns `ctx.postprocess_fn1` and uses conditional expression `postprocess_fn1 if postprocess_fn1 is not None else Postp...`.

**CN:** 将 `ctx.postprocess_fn1` and 使用条件表达式 `postprocess_fn1 if postprocess_fn1 is not None else Postp...`.

### Block 182 — Lines 986-986 (forward)
```python
986|         ctx.x_strides = tuple(x.stride())
```
**EN:** Assigns `ctx.x_strides` and calls `tuple`.

**CN:** 将 `ctx.x_strides`，并调用 `tuple`.

### Block 183 — Lines 987-987 (forward)
```python
987|         ctx.x_mx_strides = tuple(x_mxscale.stride()) if x_mxscale is not None else None
```
**EN:** Assigns `ctx.x_mx_strides` and uses conditional expression `tuple(x_mxscale.stride()) if x_mxscale is not None else None`.

**CN:** 将 `ctx.x_mx_strides` and 使用条件表达式 `tuple(x_mxscale.stride()) if x_mxscale is not None else None`.

### Block 184 — Lines 988-988 (forward)
```python
988|         ctx.mask_strides = tuple(mask.stride()) if mask is not None else None
```
**EN:** Assigns `ctx.mask_strides` and uses conditional expression `tuple(mask.stride()) if mask is not None else None`.

**CN:** 将 `ctx.mask_strides` and 使用条件表达式 `tuple(mask.stride()) if mask is not None else None`.

### Block 185 — Lines 989-989 (forward)
```python
989|         ctx.scale_strides = tuple(scale.stride()) if scale is not None else None
```
**EN:** Assigns `ctx.scale_strides` and uses conditional expression `tuple(scale.stride()) if scale is not None else None`.

**CN:** 将 `ctx.scale_strides` and 使用条件表达式 `tuple(scale.stride()) if scale is not None else None`.

### Block 186 — Lines 990-990 (forward)
```python
990|         ctx.y_has_mx = bool(y_mx is not None)
```
**EN:** Assigns `ctx.y_has_mx` and calls `bool`.

**CN:** 将 `ctx.y_has_mx`，并调用 `bool`.

### Block 187 — Lines 991-992 (forward)
```python
991|         ctx.unpadded_batch_size = unpadded_batch_size
992| 
```
**EN:** Assigns `ctx.unpadded_batch_size` and references `unpadded_batch_size`.

**CN:** 将 `ctx.unpadded_batch_size` and 引用 `unpadded_batch_size`.

### Block 188 — Lines 993-994 (forward)
```python
993|         return y, y_mx
994| 
```
**EN:** Returns `(y, y_mx)`.

**CN:** 返回 `(y, y_mx)`.

### Block 189 — Lines 995-997 (backward)
```python
995|     @staticmethod
996|     def backward(ctx, grad_y: torch.Tensor, grad_y_mxscale: Optional[torch.Tensor] = None):
997|         # We do not support grads through MX-quantized outputs (no torch compute in bwd)
```
**EN:** Defines function `backward(ctx, grad_y, grad_y_mxscale)` with decorators `staticmethod` for this module. The body mainly branches on runtime conditions; prepares intermediate values; invokes `reduce_backward`. It uses calls such as `torch.empty`, `reduce_backward`, `NotImplementedError` to implement its workflow.

**CN:** 定义函数 `backward(ctx, grad_y, grad_y_mxscale)`，带有装饰器 `staticmethod`，供本模块使用. 主体主要根据运行时条件分支; 准备中间值; invokes `reduce_backward`. 其中会调用 `torch.empty`, `reduce_backward`, `NotImplementedError` 来实现其工作流程.

### Block 190 — Lines 998-1001 (backward)
```python
 998|         if ctx.y_has_mx:
 999|             raise NotImplementedError("Backward with y_mxscale (MX-quantized outputs) is not supported.")
1000| 
1001|         # Allocate grad for x; (no torch compute)
```
**EN:** Checks `ctx.y_has_mx`..

**CN:** 检查 `ctx.y_has_mx`..

### Block 191 — Lines 1002-1003 (backward)
```python
1002|         dx = torch.empty(ctx.x_shape, dtype=ctx.x_dtype, device=grad_y.device)
1003| 
```
**EN:** Assigns `dx` and calls `torch.empty`.

**CN:** 将 `dx`，并调用 `torch.empty`.

### Block 192 — Lines 1004-1019 (backward)
```python
1004|         reduce_backward(
1005|             dy=grad_y,
1006|             x_shape=ctx.x_shape,
1007|             dim=ctx.dim,
1008|             mask=ctx.mask,
1009|             scale=ctx.scale,
1010|             x_mxscale=ctx.x_mxscale,
1011|             x_flex=ctx.x_flex,
1012|             postprocess_fn1=ctx.postprocess_fn1,
1013|             x_strides=ctx.x_strides,
1014|             x_mx_strides=ctx.x_mx_strides,
1015|             mask_strides=ctx.mask_strides,
1016|             scale_strides=ctx.scale_strides,
1017|             dx=dx,
1018|             unpadded_batch_size=ctx.unpadded_batch_size,
1019|         )
```
**EN:** Calls `reduce_backward` for side effects, registration, or validation.

**CN:** 调用 `reduce_backward` ，用于副作用、注册或校验。

### Block 193 — Lines 1020-1022 (backward)
```python
1020|         return dx, None, None, None, None, None, None, None, None, None, None, None, None, None, None, None, None, None
1021| 
1022| 
```
**EN:** Returns `(dx, None, None, None, None, None, None, None, None, None, None, None, None, ...`.

**CN:** 返回 `(dx, None, None, None, None, None, None, None, None, None, None, None, None, ...`.

### Block 194 — Lines 1023-1042 (reduce)
```python
1023| def reduce(
1024|     x: torch.Tensor,
1025|     dim: int,
1026|     mask: Optional[torch.Tensor] = None,
1027|     scale: Optional[torch.Tensor] = None,
1028|     x_mxscale: Optional[torch.Tensor] = None,
1029|     x_flex: Optional[InFlexData] = InFlexData(),
1030|     x_global_scale: Optional[torch.Tensor] = None,
1031|     y: Optional[torch.Tensor] = None,
1032|     y_dtype: Optional[torch.dtype] = None,
1033|     y_flex: Optional[OutFlexData] = OutFlexData(),
1034|     y_flex_saturate_inf: bool = False,
1035|     y_has_mx: Optional[bool] = None,
1036|     y_mx_scale_dtype: Optional[torch.dtype] = None,
1037|     y_microblock_size: int = MXFP_BLOCK_SIZE.value,
1038|     y_value_pack_factor: int = 1,
1039|     postprocess_fn1: Optional[PostprocessFn] = None,
1040|     postprocess_fn2: Optional[PostprocessFn] = None,
1041|     unpadded_batch_size: Optional[torch.Tensor] = None,
1042| ):
```
**EN:** Defines function `reduce(x, dim, mask, scale, x_mxscale, x_flex, x_global_scale, y, y_dtype, y_flex, y_flex_saturate_inf, y_has_mx, y_mx_scale_dtype, y_microblock_size, y_value_pack_factor, postprocess_fn1, postprocess_fn2, unpadded_batch_size)` for this module. The body mainly returns the computed result. It uses calls such as `InFlexData`, `OutFlexData`, `_ReduceAutograd.apply` to implement its workflow.

**CN:** 定义函数 `reduce(x, dim, mask, scale, x_mxscale, x_flex, x_global_scale, y, y_dtype, y_flex, y_flex_saturate_inf, y_has_mx, y_mx_scale_dtype, y_microblock_size, y_value_pack_factor, postprocess_fn1, postprocess_fn2, unpadded_batch_size)`，供本模块使用. 主体主要返回计算结果. 其中会调用 `InFlexData`, `OutFlexData`, `_ReduceAutograd.apply` 来实现其工作流程.

### Block 195 — Lines 1043-1050 (reduce)
```python
1043|     return _ReduceAutograd.apply(x, dim, mask, scale, x_mxscale, x_flex, x_global_scale, y_dtype, y_flex,  #
1044|                                  y_flex_saturate_inf, y_has_mx, y_mx_scale_dtype, y_microblock_size,
1045|                                  y_value_pack_factor, y, postprocess_fn1, postprocess_fn2, unpadded_batch_size)
1046| 
1047| 
1048| # ------------------------------------------------------------
1049| 
1050| 
```
**EN:** Returns `_ReduceAutograd.apply(x, dim, mask, scale, x_mxscale, x_flex, x_global_scale,...`.

**CN:** 返回 `_ReduceAutograd.apply(x, dim, mask, scale, x_mxscale, x_flex, x_global_scale,...`.

### Block 196 — Lines 1051-1051 (compute_actual_scale)
```python
1051| def compute_actual_scale(x, dtype, per_batch_scale=False):
```
**EN:** Defines function `compute_actual_scale(x, dtype, per_batch_scale)` for this module. The body mainly prepares intermediate values; prepares intermediate values; returns the computed result. It uses calls such as `x.abs().amax`, `x.abs().max`, `x.abs`, `tuple`, `range` to implement its workflow.

**CN:** 定义函数 `compute_actual_scale(x, dtype, per_batch_scale)`，供本模块使用. 主体主要准备中间值; 准备中间值; 返回计算结果. 其中会调用 `x.abs().amax`, `x.abs().max`, `x.abs`, `tuple`, `range` 来实现其工作流程.

### Block 197 — Lines 1052-1056 (compute_actual_scale)
```python
1052|     max_finite = {
1053|         torch.float8_e5m2: MAX_FINITE_FLOAT8E5,
1054|         torch.float8_e4m3fn: MAX_FINITE_FLOAT8E4NV,
1055|         torch.float8_e4m3fnuz: MAX_FINITE_FLOAT8E4B8,
1056|     }[dtype]
```
**EN:** Assigns `max_finite` and evaluates `{torch.float8_e5m2: MAX_FINITE_FLOAT8E5, torch.float8_e4m3fn: MAX_F...`.

**CN:** 将 `max_finite` and 计算 `{torch.float8_e5m2: MAX_FINITE_FLOAT8E5, torch.float8_e4m3fn: MAX_F...`.

### Block 198 — Lines 1057-1057 (compute_actual_scale)
```python
1057|     maxvals = x.abs().amax(dim=tuple(range(1, x.ndim))) if per_batch_scale else x.abs().max()
```
**EN:** Assigns `maxvals` and uses conditional expression `x.abs().amax(dim=tuple(range(1, x.ndim))) if per_batch_sc...`.

**CN:** 将 `maxvals` and 使用条件表达式 `x.abs().amax(dim=tuple(range(1, x.ndim))) if per_batch_sc...`.

### Block 199 — Lines 1058-1060 (compute_actual_scale)
```python
1058|     return maxvals / max_finite
1059| 
1060| 
```
**EN:** Returns `maxvals / max_finite`.

**CN:** 返回 `maxvals / max_finite`.

### Block 200 — Lines 1061-1066 (reduce_torch)
```python
1061| def reduce_torch(x: torch.Tensor, dim: int, mask: Optional[torch.Tensor] = None,  #
1062|                  scale: Optional[torch.Tensor] = None,  #
1063|                  x_mxscale: Optional[torch.Tensor] = None,  #
1064|                  x_flex: Optional[InFlexData] = InFlexData(), y_flex: Optional[OutFlexData] = OutFlexData(),
1065|                  y_flex_saturate_inf: bool = False, postprocess_fn1: Optional[callable] = None,
1066|                  unpadded_batch_size: Optional[torch.Tensor] = None):
```
**EN:** Defines function `reduce_torch(x, dim, mask, scale, x_mxscale, x_flex, y_flex, y_flex_saturate_inf, postprocess_fn1, unpadded_batch_size)` for this module. The body mainly prepares intermediate values; branches on runtime conditions; prepares intermediate values. It uses calls such as `InFlexData`, `OutFlexData`, `x.to`, `scale.to`, `mask.to` to implement its workflow.

**CN:** 定义函数 `reduce_torch(x, dim, mask, scale, x_mxscale, x_flex, y_flex, y_flex_saturate_inf, postprocess_fn1, unpadded_batch_size)`，供本模块使用. 主体主要准备中间值; 根据运行时条件分支; 准备中间值. 其中会调用 `InFlexData`, `OutFlexData`, `x.to`, `scale.to`, `mask.to` 来实现其工作流程.

### Block 201 — Lines 1067-1067 (reduce_torch)
```python
1067|     from triton_kernels.numerics_details.mxfp import downcast_to_mxfp_torch, upcast_from_mxfp_torch
```
**EN:** This block imports `triton_kernels.numerics_details.mxfp (downcast_to_mxfp_torch, upcast_from_mxfp_torch)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `triton_kernels.numerics_details.mxfp (downcast_to_mxfp_torch, upcast_from_mxfp_torch)` ，为模块提供所需的外部库和内部辅助工具。

### Block 202 — Lines 1068-1069 (reduce_torch)
```python
1068|     x_dtype = x.dtype
1069|     # upcast input
```
**EN:** Assigns `x_dtype` and references `x.dtype`.

**CN:** 将 `x_dtype` and 引用 `x.dtype`.

### Block 203 — Lines 1070-1071 (reduce_torch)
```python
1070|     if x_mxscale is not None:
1071|         x = upcast_from_mxfp_torch(x, x_mxscale, torch.float32, axis=-1)
```
**EN:** Checks `x_mxscale is not None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `x_mxscale is not None`. 真分支主要准备中间值.

### Block 204 — Lines 1072-1072 (reduce_torch)
```python
1072|     x = x.to(torch.float32)
```
**EN:** Assigns `x` and calls `x.to`.

**CN:** 将 `x`，并调用 `x.to`.

### Block 205 — Lines 1073-1075 (reduce_torch)
```python
1073|     if x_flex is not None:
1074|         x *= x_flex.scale
1075|     # upcast scale
```
**EN:** Checks `x_flex is not None`..

**CN:** 检查 `x_flex is not None`..

### Block 206 — Lines 1076-1077 (reduce_torch)
```python
1076|     if scale is None:
1077|         scale = torch.ones(1, dtype=torch.float32, device=x.device)
```
**EN:** Checks `scale is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `scale is None`. 真分支主要准备中间值.

### Block 207 — Lines 1078-1079 (reduce_torch)
```python
1078|     scale = scale.to(torch.float32)
1079|     # initialize mask
```
**EN:** Assigns `scale` and calls `scale.to`.

**CN:** 将 `scale`，并调用 `scale.to`.

### Block 208 — Lines 1080-1081 (reduce_torch)
```python
1080|     if mask is None:
1081|         mask = torch.ones(1, dtype=torch.bool, device=x.device)
```
**EN:** Checks `mask is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `mask is None`. 真分支主要准备中间值.

### Block 209 — Lines 1082-1082 (reduce_torch)
```python
1082|     mask = mask.to(torch.bool)
```
**EN:** Assigns `mask` and calls `mask.to`.

**CN:** 将 `mask`，并调用 `mask.to`.

### Block 210 — Lines 1083-1083 (reduce_torch)
```python
1083|     ret = torch.where(mask, x * scale, 0).sum(dim=dim)
```
**EN:** Assigns `ret` and calls `torch.where(mask, x * scale, 0).sum`.

**CN:** 将 `ret`，并调用 `torch.where(mask, x * scale, 0).sum`.

### Block 211 — Lines 1084-1085 (reduce_torch)
```python
1084|     if postprocess_fn1 is not None:
1085|         ret = postprocess_fn1(ret)
```
**EN:** Checks `postprocess_fn1 is not None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `postprocess_fn1 is not None`. 真分支主要准备中间值.

### Block 212 — Lines 1086-1089 (reduce_torch)
```python
1086|     if y_flex is not None:
1087|         y_flex.actual_scale.copy_(compute_actual_scale(ret, x_dtype, y_flex.is_per_batch))
1088|         ret = (ret / y_flex.expected_scale).to(x_dtype)
1089|     # downcast output
```
**EN:** Checks `y_flex is not None`. The true branch mainly invokes `y_flex.actual_scale.copy_`; prepares intermediate values.

**CN:** 检查 `y_flex is not None`. 真分支主要invokes `y_flex.actual_scale.copy_`; 准备中间值.

### Block 213 — Lines 1090-1090 (reduce_torch)
```python
1090|     ret_mxscale = None
```
**EN:** Assigns `ret_mxscale` and stores constant `None`.

**CN:** 将 `ret_mxscale` and 保存常量 `None`.

### Block 214 — Lines 1091-1093 (reduce_torch)
```python
1091|     if x_mxscale is not None:
1092|         assert y_flex is None
1093|         ret, ret_mxscale = downcast_to_mxfp_torch(ret, torch.float8_e4m3fn, axis=-1)
```
**EN:** Checks `x_mxscale is not None`. The true branch mainly checks invariants; prepares intermediate values.

**CN:** 检查 `x_mxscale is not None`. 真分支主要检查不变量; 准备中间值.

### Block 215 — Lines 1094-1094 (reduce_torch)
```python
1094|     return ret.to(x_dtype), ret_mxscale
```
**EN:** Returns `(ret.to(x_dtype), ret_mxscale)`.

**CN:** 返回 `(ret.to(x_dtype), ret_mxscale)`.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `PostprocessFn`, `OptFlags`, `scoped_opt_flags`, `_get_strides`, `reduce_launch_metadata`, `_select_reduce_forward_config`, `_create_row_idxs`, `_reduce_forward_inner`.
  **CN:** 主要符号：`PostprocessFn`, `OptFlags`, `scoped_opt_flags`, `_get_strides`, `reduce_launch_metadata`, `_select_reduce_forward_config`, `_create_row_idxs`, `_reduce_forward_inner`。
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
- **EN:** External modules: `dataclasses (dataclass)`, `contextlib (contextmanager)`, `contextvars (ContextVar)`, `torch`, `triton`, `triton.language`, `typing (Optional)`.
  **CN:** 外部模块：`dataclasses (dataclass)`, `contextlib (contextmanager)`, `contextvars (ContextVar)`, `torch`, `triton`, `triton.language`, `typing (Optional)`。
- **EN:** Internal modules: `triton_kernels.numerics_details.mxfp (MXFP_BLOCK_SIZE, quantize_mxfp4_fn, quantize_mxfp8_fn, quantize_nvfp4_fn)`, `triton_kernels.numerics_details.flexpoint (float_to_flex, load_scale)`, `triton_kernels.numerics (InFlexData, OutFlexData, MAX_FINITE_FLOAT8E4B8, MAX_FINITE_FLOAT8E4NV, MAX_FINITE_FLOAT8E5)`, `triton_kernels (target_info)`, `.specialize (SpecializationModule, ClosureArg, FnSpecs)`.
  **CN:** 内部模块：`triton_kernels.numerics_details.mxfp (MXFP_BLOCK_SIZE, quantize_mxfp4_fn, quantize_mxfp8_fn, quantize_nvfp4_fn)`, `triton_kernels.numerics_details.flexpoint (float_to_flex, load_scale)`, `triton_kernels.numerics (InFlexData, OutFlexData, MAX_FINITE_FLOAT8E4B8, MAX_FINITE_FLOAT8E4NV, MAX_FINITE_FLOAT8E5)`, `triton_kernels (target_info)`, `.specialize (SpecializationModule, ClosureArg, FnSpecs)`。
