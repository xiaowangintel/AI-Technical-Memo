# _topk_forward.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/triton_kernels/topk_details/_topk_forward.py`
- **Purpose / 用途:** Implementation module for topk forward; it defines the runtime logic, helper types, and GPU-facing utilities used by the Triton kernels package via symbols get_topmask_and_fullmask, fpval_to_key, key_to_fpval, indx_to_key, key_to_indx. / 用于 topk forward 的实现模块；它定义了 Triton kernels 包使用的运行时逻辑、辅助类型以及面向 GPU 的工具；核心符号包括 get_topmask_and_fullmask、fpval_to_key、key_to_fpval、indx_to_key、key_to_indx。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-4 (module)
```python
1| import triton
2| import triton.language as tl
3| 
4| 
```
**EN:** This block imports `triton`, `triton.language` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `triton`, `triton.language` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 5-6 (get_topmask_and_fullmask)
```python
5| @triton.jit
6| def get_topmask_and_fullmask(x):
```
**EN:** Defines function `get_topmask_and_fullmask(x)` with decorators `triton.jit` for this module. The body mainly invokes `tl.static_assert`; prepares intermediate values; prepares intermediate values. It uses calls such as `tl.static_assert`, `tl.full`, `x.dtype.is_int_unsigned` to implement its workflow.

**CN:** 定义函数 `get_topmask_and_fullmask(x)`，带有装饰器 `triton.jit`，供本模块使用. 主体主要invokes `tl.static_assert`; 准备中间值; 准备中间值. 其中会调用 `tl.static_assert`, `tl.full`, `x.dtype.is_int_unsigned` 来实现其工作流程.

### Block 3 — Lines 7-7 (get_topmask_and_fullmask)
```python
7|     tl.static_assert(x.dtype.is_int_unsigned(), "floating-point value must be passed as bits")
```
**EN:** Calls `tl.static_assert` for side effects, registration, or validation.

**CN:** 调用 `tl.static_assert` ，用于副作用、注册或校验。

### Block 4 — Lines 8-8 (get_topmask_and_fullmask)
```python
8|     tm: tl.constexpr = 1 << (-1 + x.dtype.primitive_bitwidth)
```
**EN:** Annotated assignment stores `tm` and evaluates `1 << -1 + x.dtype.primitive_bitwidth`.

**CN:** 带类型注解的赋值保存 `tm` and 计算 `1 << -1 + x.dtype.primitive_bitwidth`.

### Block 5 — Lines 9-9 (get_topmask_and_fullmask)
```python
9|     fm: tl.constexpr = (1 << x.dtype.primitive_bitwidth) - 1
```
**EN:** Annotated assignment stores `fm` and evaluates `(1 << x.dtype.primitive_bitwidth) - 1`.

**CN:** 带类型注解的赋值保存 `fm` and 计算 `(1 << x.dtype.primitive_bitwidth) - 1`.

### Block 6 — Lines 10-10 (get_topmask_and_fullmask)
```python
10|     tm_arr = tl.full(x.shape, tm, dtype=x.dtype)
```
**EN:** Assigns `tm_arr` and calls `tl.full`.

**CN:** 将 `tm_arr`，并调用 `tl.full`.

### Block 7 — Lines 11-11 (get_topmask_and_fullmask)
```python
11|     fm_arr = tl.full(x.shape, fm, dtype=x.dtype)
```
**EN:** Assigns `fm_arr` and calls `tl.full`.

**CN:** 将 `fm_arr`，并调用 `tl.full`.

### Block 8 — Lines 12-14 (get_topmask_and_fullmask)
```python
12|     return tm_arr, fm_arr
13| 
14| 
```
**EN:** Returns `(tm_arr, fm_arr)`.

**CN:** 返回 `(tm_arr, fm_arr)`.

### Block 9 — Lines 15-16 (fpval_to_key)
```python
15| @triton.jit
16| def fpval_to_key(x):
```
**EN:** Defines function `fpval_to_key(x)` with decorators `triton.jit` for this module. The body mainly prepares intermediate values; returns the computed result. It uses calls such as `get_topmask_and_fullmask`, `tl.where` to implement its workflow.

**CN:** 定义函数 `fpval_to_key(x)`，带有装饰器 `triton.jit`，供本模块使用. 主体主要准备中间值; 返回计算结果. 其中会调用 `get_topmask_and_fullmask`, `tl.where` 来实现其工作流程.

### Block 10 — Lines 17-17 (fpval_to_key)
```python
17|     tm, fm = get_topmask_and_fullmask(x)
```
**EN:** Assigns `tm`, `fm` and calls `get_topmask_and_fullmask`.

**CN:** 将 `tm`, `fm`，并调用 `get_topmask_and_fullmask`.

### Block 11 — Lines 18-20 (fpval_to_key)
```python
18|     return x ^ tl.where((x & tm) != 0, fm, tm)
19| 
20| 
```
**EN:** Returns `x ^ tl.where(x & tm != 0, fm, tm)`.

**CN:** 返回 `x ^ tl.where(x & tm != 0, fm, tm)`.

### Block 12 — Lines 21-22 (key_to_fpval)
```python
21| @triton.jit
22| def key_to_fpval(x):
```
**EN:** Defines function `key_to_fpval(x)` with decorators `triton.jit` for this module. The body mainly prepares intermediate values; returns the computed result. It uses calls such as `get_topmask_and_fullmask`, `tl.where` to implement its workflow.

**CN:** 定义函数 `key_to_fpval(x)`，带有装饰器 `triton.jit`，供本模块使用. 主体主要准备中间值; 返回计算结果. 其中会调用 `get_topmask_and_fullmask`, `tl.where` 来实现其工作流程.

### Block 13 — Lines 23-23 (key_to_fpval)
```python
23|     tm, fm = get_topmask_and_fullmask(x)
```
**EN:** Assigns `tm`, `fm` and calls `get_topmask_and_fullmask`.

**CN:** 将 `tm`, `fm`，并调用 `get_topmask_and_fullmask`.

### Block 14 — Lines 24-27 (key_to_fpval)
```python
24|     return x ^ tl.where((x & tm) == 0, fm, tm)
25| 
26| 
27| # stable top-k tie-breaks to value with smaller index
```
**EN:** Returns `x ^ tl.where(x & tm == 0, fm, tm)`.

**CN:** 返回 `x ^ tl.where(x & tm == 0, fm, tm)`.

### Block 15 — Lines 28-29 (indx_to_key)
```python
28| @triton.jit
29| def indx_to_key(indx, N_EXPTS_PAD: tl.constexpr):
```
**EN:** Defines function `indx_to_key(indx, N_EXPTS_PAD)` with decorators `triton.jit` for this module. The body mainly returns the computed result.

**CN:** 定义函数 `indx_to_key(indx, N_EXPTS_PAD)`，带有装饰器 `triton.jit`，供本模块使用. 主体主要返回计算结果.

### Block 16 — Lines 30-32 (indx_to_key)
```python
30|     return N_EXPTS_PAD - indx
31| 
32| 
```
**EN:** Returns `N_EXPTS_PAD - indx`.

**CN:** 返回 `N_EXPTS_PAD - indx`.

### Block 17 — Lines 33-34 (key_to_indx)
```python
33| @triton.jit
34| def key_to_indx(indx, N_EXPTS_PAD: tl.constexpr):
```
**EN:** Defines function `key_to_indx(indx, N_EXPTS_PAD)` with decorators `triton.jit` for this module. The body mainly returns the computed result.

**CN:** 定义函数 `key_to_indx(indx, N_EXPTS_PAD)`，带有装饰器 `triton.jit`，供本模块使用. 主体主要返回计算结果.

### Block 18 — Lines 35-37 (key_to_indx)
```python
35|     return N_EXPTS_PAD - indx
36| 
37| 
```
**EN:** Returns `N_EXPTS_PAD - indx`.

**CN:** 返回 `N_EXPTS_PAD - indx`.

### Block 19 — Lines 38-40 (streaming_topk)
```python
38| @triton.jit
39| def streaming_topk(X, stride_xm, n_expts_tot, offs_m, mask_m, N_EXPTS_PAD: tl.constexpr, N_EXPTS_ACT: tl.constexpr,
40|                    BLOCK_N: tl.constexpr):
```
**EN:** Defines function `streaming_topk(X, stride_xm, n_expts_tot, offs_m, mask_m, N_EXPTS_PAD, N_EXPTS_ACT, BLOCK_N)` with decorators `triton.jit` for this module. The body mainly prepares intermediate values; prepares intermediate values; branches on runtime conditions. It uses calls such as `tl.dtype`, `tl.load`, `fpval_to_key`, `tl.topk`, `tl.static_range if loop_iterations <=...` to implement its workflow.

**CN:** 定义函数 `streaming_topk(X, stride_xm, n_expts_tot, offs_m, mask_m, N_EXPTS_PAD, N_EXPTS_ACT, BLOCK_N)`，带有装饰器 `triton.jit`，供本模块使用. 主体主要准备中间值; 准备中间值; 根据运行时条件分支. 其中会调用 `tl.dtype`, `tl.load`, `fpval_to_key`, `tl.topk`, `tl.static_range if loop_iterations <=...` 来实现其工作流程.

### Block 20 — Lines 41-41 (streaming_topk)
```python
41|     x_nbits: tl.constexpr = X.dtype.element_ty.primitive_bitwidth
```
**EN:** Annotated assignment stores `x_nbits` and references `X.dtype.element_ty.primitive_bitwidth`.

**CN:** 带类型注解的赋值保存 `x_nbits` and 引用 `X.dtype.element_ty.primitive_bitwidth`.

### Block 21 — Lines 42-42 (streaming_topk)
```python
42|     x_utype: tl.constexpr = tl.dtype(f"uint{x_nbits}")
```
**EN:** Annotated assignment stores `x_utype` and calls `tl.dtype`.

**CN:** 带类型注解的赋值保存 `x_utype`，并调用 `tl.dtype`.

### Block 22 — Lines 43-48 (streaming_topk)
```python
43|     if x_nbits < 16:
44|         # this ensures that we leave at least 16 bits for expert index
45|         # even if the input dtype is smaller than 16 bits:
46|         y_nbits: tl.constexpr = 32
47|     else:
48|         y_nbits: tl.constexpr = x_nbits * 2
```
**EN:** Checks `x_nbits < 16`. The true branch mainly prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `x_nbits < 16`. 真分支主要准备中间值；而 else 分支准备中间值.

### Block 23 — Lines 49-49 (streaming_topk)
```python
49|     x_ultype: tl.constexpr = tl.dtype(f"uint{y_nbits}")
```
**EN:** Annotated assignment stores `x_ultype` and calls `tl.dtype`.

**CN:** 带类型注解的赋值保存 `x_ultype`，并调用 `tl.dtype`.

### Block 24 — Lines 50-52 (streaming_topk)
```python
50|     x_dtype: tl.constexpr = X.dtype.element_ty
51| 
52|     # subtract 1 from loop iterations because we peel the first (masked) iteration:
```
**EN:** Annotated assignment stores `x_dtype` and references `X.dtype.element_ty`.

**CN:** 带类型注解的赋值保存 `x_dtype` and 引用 `X.dtype.element_ty`.

### Block 25 — Lines 53-53 (streaming_topk)
```python
53|     loop_iterations: tl.constexpr = N_EXPTS_PAD // BLOCK_N - 1
```
**EN:** Annotated assignment stores `loop_iterations` and evaluates `N_EXPTS_PAD // BLOCK_N - 1`.

**CN:** 带类型注解的赋值保存 `loop_iterations` and 计算 `N_EXPTS_PAD // BLOCK_N - 1`.

### Block 26 — Lines 54-54 (streaming_topk)
```python
54|     offs_x_n = loop_iterations * BLOCK_N + tl.arange(0, BLOCK_N)
```
**EN:** Assigns `offs_x_n` and evaluates `loop_iterations * BLOCK_N + tl.arange(0, BLOCK_N)`.

**CN:** 将 `offs_x_n` and 计算 `loop_iterations * BLOCK_N + tl.arange(0, BLOCK_N)`.

### Block 27 — Lines 55-57 (streaming_topk)
```python
55|     mask_n = offs_x_n[None, :] < n_expts_tot
56| 
57|     # first iteration:
```
**EN:** Assigns `mask_n` and evaluates `offs_x_n[None, :] < n_expts_tot`.

**CN:** 将 `mask_n` and 计算 `offs_x_n[None, :] < n_expts_tot`.

### Block 28 — Lines 58-58 (streaming_topk)
```python
58|     X_ptrs = X + offs_m[:, None] * stride_xm + offs_x_n[None, :]
```
**EN:** Assigns `X_ptrs` and evaluates `X + offs_m[:, None] * stride_xm + offs_x_n[None, :]`.

**CN:** 将 `X_ptrs` and 计算 `X + offs_m[:, None] * stride_xm + offs_x_n[None, :]`.

### Block 29 — Lines 59-59 (streaming_topk)
```python
59|     x = tl.load(X_ptrs, mask=(mask_m & mask_n), other=float("-inf"))
```
**EN:** Assigns `x` and calls `tl.load`.

**CN:** 将 `x`，并调用 `tl.load`.

### Block 30 — Lines 60-60 (streaming_topk)
```python
60|     x = fpval_to_key(x.to(x_utype, bitcast=True))
```
**EN:** Assigns `x` and calls `fpval_to_key`.

**CN:** 将 `x`，并调用 `fpval_to_key`.

### Block 31 — Lines 61-61 (streaming_topk)
```python
61|     x = (x.to(x_ultype) << 16) | indx_to_key(offs_x_n, N_EXPTS_PAD)[None, :]
```
**EN:** Assigns `x` and evaluates `x.to(x_ultype) << 16 | indx_to_key(offs_x_n, N_EXPTS_PAD)[None, :]`.

**CN:** 将 `x` and 计算 `x.to(x_ultype) << 16 | indx_to_key(offs_x_n, N_EXPTS_PAD)[None, :]`.

### Block 32 — Lines 62-64 (streaming_topk)
```python
62|     acc = tl.topk(x, N_EXPTS_ACT, dim=1)
63| 
64|     # subsequent iterations:
```
**EN:** Assigns `acc` and calls `tl.topk`.

**CN:** 将 `acc`，并调用 `tl.topk`.

### Block 33 — Lines 65-75 (streaming_topk)
```python
65|     for _i in (tl.static_range if loop_iterations <= 4 else range)(loop_iterations):
66|         acc = tl.bitonic_merge(acc)  # ensure sorted ascending for the merge
67|         X_ptrs -= BLOCK_N
68|         offs_x_n -= BLOCK_N
69|         x = tl.load(X_ptrs, mask=mask_m, other=float("-inf"))
70|         x = fpval_to_key(x.to(x_utype, bitcast=True))
71|         x = (x.to(x_ultype) << 16) | indx_to_key(offs_x_n, N_EXPTS_PAD)[None, :]
72|         acc = tl.maximum(acc, tl.topk(x, N_EXPTS_ACT, dim=1))
73| 
74|     # sort packed (value_key, index_key) descending:
75|     # this keeps outputs ordered by gate value and uses smaller expert index for ties
```
**EN:** Loops over `(tl.static_range if loop_iterations <= 4 else range)(loop...` with target `_i`. The loop body mainly prepares intermediate values; prepares intermediate values.

**CN:** 遍历 `(tl.static_range if loop_iterations <= 4 else range)(loop...` ，目标变量为 `_i`. 循环体主要准备中间值; 准备中间值.

### Block 34 — Lines 76-77 (streaming_topk)
```python
76|     acc = tl.sort(acc, dim=1, descending=True)
77|     # 0000vvvvvvvviiii --> 0000iiii:
```
**EN:** Assigns `acc` and calls `tl.sort`.

**CN:** 将 `acc`，并调用 `tl.sort`.

### Block 35 — Lines 78-78 (streaming_topk)
```python
78|     y_indices_raw = (acc & 0xFFFF).to(tl.uint32)
```
**EN:** Assigns `y_indices_raw` and calls `(acc & 65535).to`.

**CN:** 将 `y_indices_raw`，并调用 `(acc & 65535).to`.

### Block 36 — Lines 79-80 (streaming_topk)
```python
79|     y_indices = key_to_indx(y_indices_raw, N_EXPTS_PAD)
80|     # 0000vvvvvvvviiii --> vvvvvvvv:
```
**EN:** Assigns `y_indices` and calls `key_to_indx`.

**CN:** 将 `y_indices`，并调用 `key_to_indx`.

### Block 37 — Lines 81-81 (streaming_topk)
```python
81|     y_values_raw = (acc >> 16).to(x_utype)
```
**EN:** Assigns `y_values_raw` and calls `(acc >> 16).to`.

**CN:** 将 `y_values_raw`，并调用 `(acc >> 16).to`.

### Block 38 — Lines 82-83 (streaming_topk)
```python
82|     y_values = key_to_fpval(y_values_raw).to(x_dtype, bitcast=True)
83| 
```
**EN:** Assigns `y_values` and calls `key_to_fpval(y_values_raw).to`.

**CN:** 将 `y_values`，并调用 `key_to_fpval(y_values_raw).to`.

### Block 39 — Lines 84-86 (streaming_topk)
```python
84|     return y_values, y_indices
85| 
86| 
```
**EN:** Returns `(y_values, y_indices)`.

**CN:** 返回 `(y_values, y_indices)`.

### Block 40 — Lines 87-95 (_topk_forward)
```python
87| @triton.jit
88| def _topk_forward(X, stride_xm,  # inputs
89|                   PeerYvs, PeerYis, stride_ym,  # topk values/indices
90|                   USE_PROVIDED_INDX: tl.constexpr, PeerBits, stride_rm: tl.constexpr,
91|                   stride_rn: tl.constexpr,  # bitmatrix
92|                   n_rows, n_expts_tot,  # shape
93|                   dst_offs_m, APPLY_SOFTMAX: tl.constexpr,  # constant
94|                   BLOCK_M: tl.constexpr, N_EXPTS_PAD: tl.constexpr, N_EXPTS_ACT: tl.constexpr, BLOCK_N: tl.constexpr):
95| 
```
**EN:** Defines function `_topk_forward(X, stride_xm, PeerYvs, PeerYis, stride_ym, USE_PROVIDED_INDX, PeerBits, stride_rm, stride_rn, n_rows, n_expts_tot, dst_offs_m, APPLY_SOFTMAX, BLOCK_M, N_EXPTS_PAD, N_EXPTS_ACT, BLOCK_N)` with decorators `triton.jit` for this module. The body mainly prepares intermediate values; prepares intermediate values; branches on runtime conditions. It uses calls such as `len`, `tl.program_id`, `tl.static_assert`, `tl.arange`, `tl.static_range` to implement its workflow.

**CN:** 定义函数 `_topk_forward(X, stride_xm, PeerYvs, PeerYis, stride_ym, USE_PROVIDED_INDX, PeerBits, stride_rm, stride_rn, n_rows, n_expts_tot, dst_offs_m, APPLY_SOFTMAX, BLOCK_M, N_EXPTS_PAD, N_EXPTS_ACT, BLOCK_N)`，带有装饰器 `triton.jit`，供本模块使用. 主体主要准备中间值; 准备中间值; 根据运行时条件分支. 其中会调用 `len`, `tl.program_id`, `tl.static_assert`, `tl.arange`, `tl.static_range` 来实现其工作流程.

### Block 41 — Lines 96-97 (_topk_forward)
```python
96|     N_PEERS: tl.constexpr = len(PeerYvs)
97| 
```
**EN:** Annotated assignment stores `N_PEERS` and calls `len`.

**CN:** 带类型注解的赋值保存 `N_PEERS`，并调用 `len`.

### Block 42 — Lines 98-98 (_topk_forward)
```python
98|     pid = tl.program_id(0)
```
**EN:** Assigns `pid` and calls `tl.program_id`.

**CN:** 将 `pid`，并调用 `tl.program_id`.

### Block 43 — Lines 99-101 (_topk_forward)
```python
 99|     if isinstance(n_rows, tl.tensor) and n_rows.dtype.is_ptr():
100|         n_rows = tl.load(n_rows)
101| 
```
**EN:** Checks `isinstance(n_rows, tl.tensor) and n_rows.dtype.is_ptr()`. The true branch mainly prepares intermediate values.

**CN:** 检查 `isinstance(n_rows, tl.tensor) and n_rows.dtype.is_ptr()`. 真分支主要准备中间值.

### Block 44 — Lines 102-105 (_topk_forward)
```python
102|     if pid * BLOCK_M >= n_rows:
103|         # early exit:
104|         return
105| 
```
**EN:** Checks `pid * BLOCK_M >= n_rows`. The true branch mainly returns the computed result.

**CN:** 检查 `pid * BLOCK_M >= n_rows`. 真分支主要返回计算结果.

### Block 45 — Lines 106-106 (_topk_forward)
```python
106|     tl.static_assert(BLOCK_N % 32 == 0)
```
**EN:** Calls `tl.static_assert` for side effects, registration, or validation.

**CN:** 调用 `tl.static_assert` ，用于副作用、注册或校验。

### Block 46 — Lines 107-107 (_topk_forward)
```python
107|     tl.static_assert(N_EXPTS_PAD % BLOCK_N == 0)
```
**EN:** Calls `tl.static_assert` for side effects, registration, or validation.

**CN:** 调用 `tl.static_assert` ，用于副作用、注册或校验。

### Block 47 — Lines 108-110 (_topk_forward)
```python
108|     x_dtype: tl.constexpr = X.dtype.element_ty
109| 
110|     # load logits
```
**EN:** Annotated assignment stores `x_dtype` and references `X.dtype.element_ty`.

**CN:** 带类型注解的赋值保存 `x_dtype` and 引用 `X.dtype.element_ty`.

### Block 48 — Lines 111-111 (_topk_forward)
```python
111|     offs_m = pid * BLOCK_M + tl.arange(0, BLOCK_M)
```
**EN:** Assigns `offs_m` and evaluates `pid * BLOCK_M + tl.arange(0, BLOCK_M)`.

**CN:** 将 `offs_m` and 计算 `pid * BLOCK_M + tl.arange(0, BLOCK_M)`.

### Block 49 — Lines 112-112 (_topk_forward)
```python
112|     offs_y_n = tl.arange(0, N_EXPTS_ACT)
```
**EN:** Assigns `offs_y_n` and calls `tl.arange`.

**CN:** 将 `offs_y_n`，并调用 `tl.arange`.

### Block 50 — Lines 113-113 (_topk_forward)
```python
113|     mask_m = offs_m[:, None] < n_rows
```
**EN:** Assigns `mask_m` and evaluates `offs_m[:, None] < n_rows`.

**CN:** 将 `mask_m` and 计算 `offs_m[:, None] < n_rows`.

### Block 51 — Lines 114-124 (_topk_forward)
```python
114|     if USE_PROVIDED_INDX:
115|         tl.static_assert(len(PeerYis) == 1)
116|         Yi_ptrs = PeerYis[0] + (dst_offs_m + offs_m[:, None]) * stride_ym + offs_y_n[None, :]
117|         y_indices = tl.load(Yi_ptrs, mask=mask_m)
118|         Xv_ptrs = X + offs_m[:, None] * stride_xm + y_indices
119|         y_values = tl.load(Xv_ptrs, mask=mask_m)
120|     else:
121|         y_values, y_indices = streaming_topk(X, stride_xm, n_expts_tot, offs_m, mask_m,  #
122|                                              N_EXPTS_PAD, N_EXPTS_ACT, BLOCK_N)
123| 
124|     # normalize selected values
```
**EN:** Checks `USE_PROVIDED_INDX`. The true branch mainly invokes `tl.static_assert`; prepares intermediate values, while the else branch prepares intermediate values.

**CN:** 检查 `USE_PROVIDED_INDX`. 真分支主要invokes `tl.static_assert`; 准备中间值；而 else 分支准备中间值.

### Block 52 — Lines 125-128 (_topk_forward)
```python
125|     if APPLY_SOFTMAX:
126|         y_values = tl.softmax(y_values.to(tl.float32), dim=1, keep_dims=True).to(x_dtype)
127| 
128|     # write back
```
**EN:** Checks `APPLY_SOFTMAX`. The true branch mainly prepares intermediate values.

**CN:** 检查 `APPLY_SOFTMAX`. 真分支主要准备中间值.

### Block 53 — Lines 129-131 (_topk_forward)
```python
129|     for rank in tl.static_range(N_PEERS):
130|         Yv_ptrs = PeerYvs[rank] + (dst_offs_m + offs_m[:, None]) * stride_ym + offs_y_n[None, :]
131|         tl.store(Yv_ptrs, y_values, mask=mask_m)
```
**EN:** Loops over `tl.static_range(N_PEERS)` with target `rank`. The loop body mainly prepares intermediate values; invokes `tl.store`.

**CN:** 遍历 `tl.static_range(N_PEERS)` ，目标变量为 `rank`. 循环体主要准备中间值; invokes `tl.store`.

### Block 54 — Lines 132-137 (_topk_forward)
```python
132|     if not USE_PROVIDED_INDX:
133|         for rank in tl.static_range(N_PEERS):
134|             Yi_ptrs = PeerYis[rank] + (dst_offs_m + offs_m[:, None]) * stride_ym + offs_y_n[None, :]
135|             tl.store(Yi_ptrs, y_indices, mask=mask_m)
136| 
137|     # pack into bitmatrix
```
**EN:** Checks `not USE_PROVIDED_INDX`. The true branch mainly iterates over cases or dimensions.

**CN:** 检查 `not USE_PROVIDED_INDX`. 真分支主要遍历测试用例或维度.

### Block 55 — Lines 138-138 (_topk_forward)
```python
138|     y_div = y_indices // 32
```
**EN:** Assigns `y_div` and evaluates `y_indices // 32`.

**CN:** 将 `y_div` and 计算 `y_indices // 32`.

### Block 56 — Lines 139-139 (_topk_forward)
```python
139|     y_rem = y_indices % 32
```
**EN:** Assigns `y_rem` and evaluates `y_indices % 32`.

**CN:** 将 `y_rem` and 计算 `y_indices % 32`.

### Block 57 — Lines 140-140 (_topk_forward)
```python
140|     loop_iterations = N_EXPTS_PAD // BLOCK_N
```
**EN:** Assigns `loop_iterations` and evaluates `N_EXPTS_PAD // BLOCK_N`.

**CN:** 将 `loop_iterations` and 计算 `N_EXPTS_PAD // BLOCK_N`.

### Block 58 — Lines 141-147 (_topk_forward)
```python
141|     for i in range(loop_iterations):
142|         offs_r_n = tl.arange(0, BLOCK_N // 32) + i * (BLOCK_N // 32)
143|         y2 = tl.where(y_div[:, :, None] == offs_r_n[None, None, :], (1 << y_rem)[:, :, None], 0)
144|         r = tl.reduce_or(y2, axis=1)
145|         for rank in tl.static_range(N_PEERS):
146|             BitsPtrs = PeerBits[rank] + (dst_offs_m + offs_m[:, None]) * stride_rm + offs_r_n[None, :] * stride_rn
147|             tl.store(BitsPtrs, r, mask=mask_m)
```
**EN:** Loops over `range(loop_iterations)` with target `i`. The loop body mainly prepares intermediate values; prepares intermediate values.

**CN:** 遍历 `range(loop_iterations)` ，目标变量为 `i`. 循环体主要准备中间值; 准备中间值.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `get_topmask_and_fullmask`, `fpval_to_key`, `key_to_fpval`, `indx_to_key`, `key_to_indx`, `streaming_topk`, `_topk_forward`.
  **CN:** 主要符号：`get_topmask_and_fullmask`, `fpval_to_key`, `key_to_fpval`, `indx_to_key`, `key_to_indx`, `streaming_topk`, `_topk_forward`。
- **EN:** Triton is used to JIT-compile or launch GPU kernels and to express tile-level execution details.
  **CN:** Triton 用于 JIT 编译或启动 GPU 内核，并表达分块级执行细节。
- **EN:** Reduction logic combines partial values across dimensions or shards.
  **CN:** 归约逻辑会沿着维度或分片合并部分结果。
- **EN:** Top-k selection and, where relevant, gradient propagation are key operations here.
  **CN:** 这里的关键操作是 Top-k 选择以及相关的梯度传播。

## Dependencies / 依赖关系
- **EN:** External modules: `triton`, `triton.language`.
  **CN:** 外部模块：`triton`, `triton.language`。
- **EN:** Internal modules: none at the top level.
  **CN:** 顶层内部模块：无。
