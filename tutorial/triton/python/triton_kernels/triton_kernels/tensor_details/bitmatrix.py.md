# bitmatrix.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/triton_kernels/tensor_details/bitmatrix.py`
- **Purpose / 用途:** Implementation module for bitmatrix; it defines the runtime logic, helper types, and GPU-facing utilities used by the Triton kernels package via symbols BitmatrixMetadata, _keyed_add, _bitmatrix_metadata_compute_stage2, _bitmatrix_metadata_compute_stage1, cdiv. / 用于 bitmatrix 的实现模块；它定义了 Triton kernels 包使用的运行时逻辑、辅助类型以及面向 GPU 的工具；核心符号包括 BitmatrixMetadata、_keyed_add、_bitmatrix_metadata_compute_stage2、_bitmatrix_metadata_compute_stage1、cdiv。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-7 (module)
```python
1| from dataclasses import dataclass
2| import triton
3| import triton.language as tl
4| import torch
5| from .bitmatrix_details.sum_bitmatrix_rows import sum_bitmatrix_rows
6| 
7| 
```
**EN:** This block imports `dataclasses (dataclass)`, `triton`, `triton.language`, `torch`, `.bitmatrix_details.sum_bitmatrix_rows (sum_bitmatrix_rows)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `dataclasses (dataclass)`, `triton`, `triton.language`, `torch`, `.bitmatrix_details.sum_bitmatrix_rows (sum_bitmatrix_rows)` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 8-9 (BitmatrixMetadata)
```python
8| @dataclass
9| class BitmatrixMetadata:
```
**EN:** Defines class `BitmatrixMetadata` with decorators `dataclass` to organize related behavior. Key fields include `col_sum`, `col_sorted_indx`, `row_sorted_indx`.

**CN:** 定义类 `BitmatrixMetadata`，带有装饰器 `dataclass`，用于组织相关行为。关键字段包括 `col_sum`, `col_sorted_indx`, `row_sorted_indx`.

### Block 3 — Lines 10-20 (BitmatrixMetadata)
```python
10|     """
11|     Example:
12|     `bitmatrix` = [0 0 1 0 1 1 0
13|                    0 1 0 0 0 1 0
14|                    1 1 1 0 0 0 1
15|                    0 0 1 0 1 0 0]
16|     `col_sum` = [1 2 3 0 2 2 1]
17|     `col_sorted_indx` = cat([5], [3 6], [0 7], [], [9 1 10], [2 4], [8])
18|     `row_sorted_indx` = cat([3 6 8], [1 9], [0 2 4 10], [5 7])
19|     """
20|     # the number of entries equal to 1 in each column
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 4 — Lines 21-22 (BitmatrixMetadata)
```python
21|     col_sum: torch.Tensor
22|     # indices of nonzero values numbered row-major, grouped by cols, concatenated
```
**EN:** Annotated assignment stores `col_sum` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `col_sum` and 声明该带注解的字段.

### Block 5 — Lines 23-24 (BitmatrixMetadata)
```python
23|     col_sorted_indx: torch.Tensor
24|     # indices of nonzero values numbered col-major, grouped by rows, concatenated
```
**EN:** Annotated assignment stores `col_sorted_indx` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `col_sorted_indx` and 声明该带注解的字段.

### Block 6 — Lines 25-31 (BitmatrixMetadata)
```python
25|     row_sorted_indx: torch.Tensor
26| 
27| 
28| # `make_bitmatrix_metadata`: entry point for optimized implementation
29| # ---------------------------------------------------------------------------- #
30| 
31| 
```
**EN:** Annotated assignment stores `row_sorted_indx` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `row_sorted_indx` and 声明该带注解的字段.

### Block 7 — Lines 32-34 (_keyed_add)
```python
32| @triton.jit
33| def _keyed_add(x, y):
34|     # we keep the key in the upper 16 bits of a uint32:
```
**EN:** Defines function `_keyed_add(x, y)` with decorators `triton.jit` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `tl.where` to implement its workflow.

**CN:** 定义函数 `_keyed_add(x, y)`，带有装饰器 `triton.jit`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `tl.where` 来实现其工作流程.

### Block 8 — Lines 35-36 (_keyed_add)
```python
35|     key_mask: tl.constexpr = 0xffff0000
36| 
```
**EN:** Annotated assignment stores `key_mask` and stores constant `4294901760`.

**CN:** 带类型注解的赋值保存 `key_mask` and 保存常量 `4294901760`.

### Block 9 — Lines 37-37 (_keyed_add)
```python
37|     kx = x & key_mask
```
**EN:** Assigns `kx` and evaluates `x & key_mask`.

**CN:** 将 `kx` and 计算 `x & key_mask`.

### Block 10 — Lines 38-38 (_keyed_add)
```python
38|     ky = y & key_mask
```
**EN:** Assigns `ky` and evaluates `y & key_mask`.

**CN:** 将 `ky` and 计算 `y & key_mask`.

### Block 11 — Lines 39-39 (_keyed_add)
```python
39|     z = tl.where(kx == ky, x + y - kx, y)
```
**EN:** Assigns `z` and calls `tl.where`.

**CN:** 将 `z`，并调用 `tl.where`.

### Block 12 — Lines 40-42 (_keyed_add)
```python
40|     return z
41| 
42| 
```
**EN:** Returns `z`.

**CN:** 返回 `z`.

### Block 13 — Lines 43-45 (_bitmatrix_metadata_compute_stage2)
```python
43| @triton.jit
44| def _bitmatrix_metadata_compute_stage2(ColSortedIndx, RowSortedIndx, NonzeroIndx, n_tokens, ColPartialSum, stride_pm,
45|                                        stride_pn, ColOffs, TOKS_PER_ROW: tl.constexpr, BLOCK_PER_TOK: tl.constexpr):
```
**EN:** Defines function `_bitmatrix_metadata_compute_stage2(ColSortedIndx, RowSortedIndx, NonzeroIndx, n_tokens, ColPartialSum, stride_pm, stride_pn, ColOffs, TOKS_PER_ROW, BLOCK_PER_TOK)` with decorators `triton.jit` for this module. The body mainly prepares intermediate values; invokes `tl.static_assert`; branches on runtime conditions. It uses calls such as `tl.static_assert`, `tl.program_id`, `tl.arange`, `tl.load(NonzeroIndx + offs_global, ma...`, `(col_indx << 16 | offs_local).to` to implement its workflow.

**CN:** 定义函数 `_bitmatrix_metadata_compute_stage2(ColSortedIndx, RowSortedIndx, NonzeroIndx, n_tokens, ColPartialSum, stride_pm, stride_pn, ColOffs, TOKS_PER_ROW, BLOCK_PER_TOK)`，带有装饰器 `triton.jit`，供本模块使用. 主体主要准备中间值; invokes `tl.static_assert`; 根据运行时条件分支. 其中会调用 `tl.static_assert`, `tl.program_id`, `tl.arange`, `tl.load(NonzeroIndx + offs_global, ma...`, `(col_indx << 16 | offs_local).to` 来实现其工作流程.

### Block 14 — Lines 46-46 (_bitmatrix_metadata_compute_stage2)
```python
46|     BLOCK_SIZE: tl.constexpr = BLOCK_PER_TOK * TOKS_PER_ROW
```
**EN:** Annotated assignment stores `BLOCK_SIZE` and evaluates `BLOCK_PER_TOK * TOKS_PER_ROW`.

**CN:** 带类型注解的赋值保存 `BLOCK_SIZE` and 计算 `BLOCK_PER_TOK * TOKS_PER_ROW`.

### Block 15 — Lines 47-47 (_bitmatrix_metadata_compute_stage2)
```python
47|     tl.static_assert(BLOCK_SIZE <= 32768)
```
**EN:** Calls `tl.static_assert` for side effects, registration, or validation.

**CN:** 调用 `tl.static_assert` ，用于副作用、注册或校验。

### Block 16 — Lines 48-49 (_bitmatrix_metadata_compute_stage2)
```python
48|     if isinstance(n_tokens, tl.tensor) and n_tokens.dtype.is_ptr():
49|         n_tokens = tl.load(n_tokens)
```
**EN:** Checks `isinstance(n_tokens, tl.tensor) and n_tokens.dtype.is_ptr()`. The true branch mainly prepares intermediate values.

**CN:** 检查 `isinstance(n_tokens, tl.tensor) and n_tokens.dtype.is_ptr()`. 真分支主要准备中间值.

### Block 17 — Lines 50-50 (_bitmatrix_metadata_compute_stage2)
```python
50|     nonzero_indx_size = n_tokens * TOKS_PER_ROW
```
**EN:** Assigns `nonzero_indx_size` and evaluates `n_tokens * TOKS_PER_ROW`.

**CN:** 将 `nonzero_indx_size` and 计算 `n_tokens * TOKS_PER_ROW`.

### Block 18 — Lines 51-52 (_bitmatrix_metadata_compute_stage2)
```python
51|     pid_m = tl.program_id(0)
52|     # load column indices
```
**EN:** Assigns `pid_m` and calls `tl.program_id`.

**CN:** 将 `pid_m`，并调用 `tl.program_id`.

### Block 19 — Lines 53-53 (_bitmatrix_metadata_compute_stage2)
```python
53|     offs_local = tl.arange(0, BLOCK_SIZE)
```
**EN:** Assigns `offs_local` and calls `tl.arange`.

**CN:** 将 `offs_local`，并调用 `tl.arange`.

### Block 20 — Lines 54-54 (_bitmatrix_metadata_compute_stage2)
```python
54|     offs_global = pid_m * BLOCK_SIZE + offs_local
```
**EN:** Assigns `offs_global` and evaluates `pid_m * BLOCK_SIZE + offs_local`.

**CN:** 将 `offs_global` and 计算 `pid_m * BLOCK_SIZE + offs_local`.

### Block 21 — Lines 55-55 (_bitmatrix_metadata_compute_stage2)
```python
55|     mask = offs_global < nonzero_indx_size
```
**EN:** Assigns `mask` and evaluates `offs_global < nonzero_indx_size`.

**CN:** 将 `mask` and 计算 `offs_global < nonzero_indx_size`.

### Block 22 — Lines 56-57 (_bitmatrix_metadata_compute_stage2)
```python
56|     col_indx = tl.load(NonzeroIndx + offs_global, mask=mask, other=-1).to(tl.uint32)
57|     # stable-sort by columns index
```
**EN:** Assigns `col_indx` and calls `tl.load(NonzeroIndx + offs_global, mask=mask, o...`.

**CN:** 将 `col_indx`，并调用 `tl.load(NonzeroIndx + offs_global, mask=mask, o...`.

### Block 23 — Lines 58-58 (_bitmatrix_metadata_compute_stage2)
```python
58|     kv_pairs = ((col_indx << 16) | offs_local).to(tl.uint32)
```
**EN:** Assigns `kv_pairs` and calls `(col_indx << 16 | offs_local).to`.

**CN:** 将 `kv_pairs`，并调用 `(col_indx << 16 | offs_local).to`.

### Block 24 — Lines 59-59 (_bitmatrix_metadata_compute_stage2)
```python
59|     kv_pairs = tl.sort(kv_pairs, 0)
```
**EN:** Assigns `kv_pairs` and calls `tl.sort`.

**CN:** 将 `kv_pairs`，并调用 `tl.sort`.

### Block 25 — Lines 60-60 (_bitmatrix_metadata_compute_stage2)
```python
60|     col_indx = kv_pairs >> 16
```
**EN:** Assigns `col_indx` and evaluates `kv_pairs >> 16`.

**CN:** 将 `col_indx` and 计算 `kv_pairs >> 16`.

### Block 26 — Lines 61-61 (_bitmatrix_metadata_compute_stage2)
```python
61|     offs_global = pid_m * BLOCK_SIZE + (kv_pairs & 0xffff)
```
**EN:** Assigns `offs_global` and evaluates `pid_m * BLOCK_SIZE + (kv_pairs & 65535)`.

**CN:** 将 `offs_global` and 计算 `pid_m * BLOCK_SIZE + (kv_pairs & 65535)`.

### Block 27 — Lines 62-63 (_bitmatrix_metadata_compute_stage2)
```python
62|     mask = col_indx != 0xffff
63|     # compute run lengths in column-sorted order:
```
**EN:** Assigns `mask` and evaluates `col_indx != 65535`.

**CN:** 将 `mask` and 计算 `col_indx != 65535`.

### Block 28 — Lines 64-64 (_bitmatrix_metadata_compute_stage2)
```python
64|     x = (kv_pairs & 0xffff0000 | 0x00000001)
```
**EN:** Assigns `x` and evaluates `kv_pairs & 4294901760 | 1`.

**CN:** 将 `x` and 计算 `kv_pairs & 4294901760 | 1`.

### Block 29 — Lines 65-65 (_bitmatrix_metadata_compute_stage2)
```python
65|     cols_and_inclusive_run_lengths = tl.associative_scan(x, 0, _keyed_add)
```
**EN:** Assigns `cols_and_inclusive_run_lengths` and calls `tl.associative_scan`.

**CN:** 将 `cols_and_inclusive_run_lengths`，并调用 `tl.associative_scan`.

### Block 30 — Lines 66-67 (_bitmatrix_metadata_compute_stage2)
```python
66|     exclusive_run_lengths = (cols_and_inclusive_run_lengths - 1) & 0xffff
67|     # compute output
```
**EN:** Assigns `exclusive_run_lengths` and evaluates `cols_and_inclusive_run_lengths - 1 & 65535`.

**CN:** 将 `exclusive_run_lengths` and 计算 `cols_and_inclusive_run_lengths - 1 & 65535`.

### Block 31 — Lines 68-68 (_bitmatrix_metadata_compute_stage2)
```python
68|     row_sorted_indx = tl.load(ColPartialSum + pid_m * stride_pm + col_indx * stride_pn, mask=mask)
```
**EN:** Assigns `row_sorted_indx` and calls `tl.load`.

**CN:** 将 `row_sorted_indx`，并调用 `tl.load`.

### Block 32 — Lines 69-69 (_bitmatrix_metadata_compute_stage2)
```python
69|     row_sorted_indx += tl.load(ColOffs + col_indx, mask=mask)
```
**EN:** Updates `row_sorted_indx` with operator `Add` using `tl.load(ColOffs + col_indx, mask=mask)`.

**CN:** 更新 `row_sorted_indx`，使用运算符 `Add`，并使用 `tl.load(ColOffs + col_indx, mask=mask)`.

### Block 33 — Lines 70-71 (_bitmatrix_metadata_compute_stage2)
```python
70|     row_sorted_indx += exclusive_run_lengths
71|     # write back output
```
**EN:** Updates `row_sorted_indx` with operator `Add` using `exclusive_run_lengths`.

**CN:** 更新 `row_sorted_indx`，使用运算符 `Add`，并使用 `exclusive_run_lengths`.

### Block 34 — Lines 72-72 (_bitmatrix_metadata_compute_stage2)
```python
72|     tl.store(RowSortedIndx + offs_global, row_sorted_indx, mask=mask)
```
**EN:** Calls `tl.store` for side effects, registration, or validation.

**CN:** 调用 `tl.store` ，用于副作用、注册或校验。

### Block 35 — Lines 73-75 (_bitmatrix_metadata_compute_stage2)
```python
73|     tl.store(ColSortedIndx + row_sorted_indx, offs_global, mask=mask)
74| 
75| 
```
**EN:** Calls `tl.store` for side effects, registration, or validation.

**CN:** 调用 `tl.store` ，用于副作用、注册或校验。

### Block 36 — Lines 76-79 (_bitmatrix_metadata_compute_stage1)
```python
76| @triton.jit
77| def _bitmatrix_metadata_compute_stage1(CombinedIndx, n_combined_indx, sentinel, BLOCK: tl.constexpr, ColSum, ColOffs,
78|                                        n_cols, PartialColSum, shape_pm, stride_pm, stride_pn, BLOCK_M: tl.constexpr,
79|                                        BLOCK_N: tl.constexpr):
```
**EN:** Defines function `_bitmatrix_metadata_compute_stage1(CombinedIndx, n_combined_indx, sentinel, BLOCK, ColSum, ColOffs, n_cols, PartialColSum, shape_pm, stride_pm, stride_pn, BLOCK_M, BLOCK_N)` with decorators `triton.jit` for this module. The body mainly prepares intermediate values; branches on runtime conditions. It uses calls such as `tl.program_id`, `range`, `tl.load`, `tl.sum`, `tl.store` to implement its workflow.

**CN:** 定义函数 `_bitmatrix_metadata_compute_stage1(CombinedIndx, n_combined_indx, sentinel, BLOCK, ColSum, ColOffs, n_cols, PartialColSum, shape_pm, stride_pm, stride_pn, BLOCK_M, BLOCK_N)`，带有装饰器 `triton.jit`，供本模块使用. 主体主要准备中间值; 根据运行时条件分支. 其中会调用 `tl.program_id`, `range`, `tl.load`, `tl.sum`, `tl.store` 来实现其工作流程.

### Block 37 — Lines 80-81 (_bitmatrix_metadata_compute_stage1)
```python
80|     pid = tl.program_id(0)
81|     # compute col_partial_sums
```
**EN:** Assigns `pid` and calls `tl.program_id`.

**CN:** 将 `pid`，并调用 `tl.program_id`.

### Block 38 — Lines 82-105 (_bitmatrix_metadata_compute_stage1)
```python
 82|     if pid < n_cols:
 83|         PartialColSum += pid * stride_pn
 84|         curr_sum = 0
 85|         for start in range(0, shape_pm, BLOCK_M):
 86|             offs = start + tl.arange(0, BLOCK_M) * stride_pm
 87|             partial_col_sum = tl.load(PartialColSum + offs, mask=offs < shape_pm)
 88|             out = tl.cumsum(partial_col_sum, 0) - partial_col_sum + curr_sum
 89|             curr_sum += tl.sum(partial_col_sum, 0)
 90|             tl.store(PartialColSum + offs, out, mask=offs < shape_pm)
 91|     # compute col_offs
 92|     elif pid == n_cols:
 93|         curr_sum = 0
 94|         for start in range(0, n_cols, BLOCK_N):
 95|             offs = start + tl.arange(0, BLOCK_N)
 96|             col_sum = tl.load(ColSum + offs, mask=offs < n_cols)
 97|             col_offs = tl.cumsum(col_sum, 0) - col_sum + curr_sum
 98|             curr_sum += tl.sum(col_sum, 0)
 99|             tl.store(ColOffs + offs, col_offs, mask=offs < n_cols)
100|     # memset `combined_indx` to `sentinel`
101|     else:
102|         offs = (pid - n_cols - 1) * BLOCK + tl.arange(0, BLOCK)
103|         tl.store(CombinedIndx + offs, sentinel, mask=offs < n_combined_indx)
104| 
105| 
```
**EN:** Checks `pid < n_cols`. The true branch mainly prepares intermediate values; iterates over cases or dimensions, while the else branch branches on runtime conditions.

**CN:** 检查 `pid < n_cols`. 真分支主要准备中间值; 遍历测试用例或维度；而 else 分支根据运行时条件分支.

### Block 39 — Lines 106-106 (cdiv)
```python
106| def cdiv(x, y):
```
**EN:** Defines function `cdiv(x, y)` for this module. The body mainly returns the computed result.

**CN:** 定义函数 `cdiv(x, y)`，供本模块使用. 主体主要返回计算结果.

### Block 40 — Lines 107-109 (cdiv)
```python
107|     return (x + y - 1) // y
108| 
109| 
```
**EN:** Returns `(x + y - 1) // y`.

**CN:** 返回 `(x + y - 1) // y`.

### Block 41 — Lines 110-110 (make_bitmatrix_metadata)
```python
110| def make_bitmatrix_metadata(nonzero_indx, bitmatrix):
```
**EN:** Defines function `make_bitmatrix_metadata(nonzero_indx, bitmatrix)` for this module. The body mainly checks invariants; prepares intermediate values; prepares intermediate values. It uses calls such as `sum_bitmatrix_rows`, `nonzero_indx.numel`, `torch.empty`, `_bitmatrix_metadata_compute_stage1[me...`, `_bitmatrix_metadata_compute_stage2[co...` to implement its workflow.

**CN:** 定义函数 `make_bitmatrix_metadata(nonzero_indx, bitmatrix)`，供本模块使用. 主体主要检查不变量; 准备中间值; 准备中间值. 其中会调用 `sum_bitmatrix_rows`, `nonzero_indx.numel`, `torch.empty`, `_bitmatrix_metadata_compute_stage1[me...`, `_bitmatrix_metadata_compute_stage2[co...` 来实现其工作流程.

### Block 42 — Lines 111-111 (make_bitmatrix_metadata)
```python
111|     assert nonzero_indx.ndim == 2
```
**EN:** Asserts `nonzero_indx.ndim == 2` to enforce invariants.

**CN:** 断言 `nonzero_indx.ndim == 2` 以确保不变量成立。

### Block 43 — Lines 112-112 (make_bitmatrix_metadata)
```python
112|     PARTIAL_BLOCK_M = 32
```
**EN:** Assigns `PARTIAL_BLOCK_M` and stores constant `32`.

**CN:** 将 `PARTIAL_BLOCK_M` and 保存常量 `32`.

### Block 44 — Lines 113-114 (make_bitmatrix_metadata)
```python
113|     col_sum, col_partial_sum = sum_bitmatrix_rows(bitmatrix, partials_block_size=PARTIAL_BLOCK_M)
114|     # allocate memory
```
**EN:** Assigns `col_sum`, `col_partial_sum` and calls `sum_bitmatrix_rows`.

**CN:** 将 `col_sum`, `col_partial_sum`，并调用 `sum_bitmatrix_rows`.

### Block 45 — Lines 115-115 (make_bitmatrix_metadata)
```python
115|     device = bitmatrix.device
```
**EN:** Assigns `device` and references `bitmatrix.device`.

**CN:** 将 `device` and 引用 `bitmatrix.device`.

### Block 46 — Lines 116-116 (make_bitmatrix_metadata)
```python
116|     n_indx = nonzero_indx.numel()
```
**EN:** Assigns `n_indx` and calls `nonzero_indx.numel`.

**CN:** 将 `n_indx`，并调用 `nonzero_indx.numel`.

### Block 47 — Lines 117-117 (make_bitmatrix_metadata)
```python
117|     n_cols = bitmatrix.shape[1]
```
**EN:** Assigns `n_cols` and evaluates `bitmatrix.shape[1]`.

**CN:** 将 `n_cols` and 计算 `bitmatrix.shape[1]`.

### Block 48 — Lines 118-118 (make_bitmatrix_metadata)
```python
118|     col_offs = torch.empty(n_cols, dtype=torch.int32, device=device)
```
**EN:** Assigns `col_offs` and calls `torch.empty`.

**CN:** 将 `col_offs`，并调用 `torch.empty`.

### Block 49 — Lines 119-119 (make_bitmatrix_metadata)
```python
119|     combined_indx = torch.empty(n_indx * 2, dtype=torch.int32, device=device)
```
**EN:** Assigns `combined_indx` and calls `torch.empty`.

**CN:** 将 `combined_indx`，并调用 `torch.empty`.

### Block 50 — Lines 120-120 (make_bitmatrix_metadata)
```python
120|     col_sorted_indx = combined_indx[:n_indx]
```
**EN:** Assigns `col_sorted_indx` and evaluates `combined_indx[:n_indx]`.

**CN:** 将 `col_sorted_indx` and 计算 `combined_indx[:n_indx]`.

### Block 51 — Lines 121-125 (make_bitmatrix_metadata)
```python
121|     row_sorted_indx = combined_indx[n_indx:]
122|     # this kernel:
123|     # - initializes `{row,col}_sorted_indx` to `sentinel`
124|     # - computes col_offs; necessary for computing `{row,col}_sorted_indx`
125|     # - computes col_partial_sums; necessary for computing `{row,col}_sorted_indx`
```
**EN:** Assigns `row_sorted_indx` and evaluates `combined_indx[n_indx:]`.

**CN:** 将 `row_sorted_indx` and 计算 `combined_indx[n_indx:]`.

### Block 52 — Lines 126-126 (make_bitmatrix_metadata)
```python
126|     MEMSET_BLOCK = 1024
```
**EN:** Assigns `MEMSET_BLOCK` and stores constant `1024`.

**CN:** 将 `MEMSET_BLOCK` and 保存常量 `1024`.

### Block 53 — Lines 127-127 (make_bitmatrix_metadata)
```python
127|     memset_grid = (cdiv(n_indx * 2, MEMSET_BLOCK) + n_cols + 1, )
```
**EN:** Assigns `memset_grid` and builds a tuple.

**CN:** 将 `memset_grid` and 构造一个元组.

### Block 54 — Lines 128-135 (make_bitmatrix_metadata)
```python
128|     _bitmatrix_metadata_compute_stage1[memset_grid](
129|         combined_indx, n_indx * 2, -1, MEMSET_BLOCK, col_sum,  #
130|         col_offs, col_sum.shape[0], col_partial_sum,  # inputs
131|         col_partial_sum.shape[0], col_partial_sum.stride(0), col_partial_sum.stride(1),  # outputs
132|         BLOCK_M=512, BLOCK_N=512,  # tunable parameters
133|     )
134|     # this kernel computes valid entries of `{row,col}_sorted_indx`
135|     # using `col_offs` and `col_partial_sums`
```
**EN:** Calls `_bitmatrix_metadata_compute_stage1[memset_grid]` for side effects, registration, or validation.

**CN:** 调用 `_bitmatrix_metadata_compute_stage1[memset_grid]` ，用于副作用、注册或校验。

### Block 55 — Lines 136-136 (make_bitmatrix_metadata)
```python
136|     n_indx = nonzero_indx.numel()
```
**EN:** Assigns `n_indx` and calls `nonzero_indx.numel`.

**CN:** 将 `n_indx`，并调用 `nonzero_indx.numel`.

### Block 56 — Lines 137-137 (make_bitmatrix_metadata)
```python
137|     toks_per_row = nonzero_indx.shape[-1]
```
**EN:** Assigns `toks_per_row` and evaluates `nonzero_indx.shape[-1]`.

**CN:** 将 `toks_per_row` and 计算 `nonzero_indx.shape[-1]`.

### Block 57 — Lines 138-138 (make_bitmatrix_metadata)
```python
138|     compute_grid = (cdiv(bitmatrix.shape_max[0], PARTIAL_BLOCK_M), )
```
**EN:** Assigns `compute_grid` and builds a tuple.

**CN:** 将 `compute_grid` and 构造一个元组.

### Block 58 — Lines 139-145 (make_bitmatrix_metadata)
```python
139|     _bitmatrix_metadata_compute_stage2[compute_grid](
140|         col_sorted_indx, row_sorted_indx,  # outputs
141|         nonzero_indx, bitmatrix.shape[0], col_partial_sum, col_partial_sum.stride(0),
142|         col_partial_sum.stride(1),  # inputs
143|         col_offs,  #
144|         TOKS_PER_ROW=toks_per_row, BLOCK_PER_TOK=PARTIAL_BLOCK_M,  #
145|     )
```
**EN:** Calls `_bitmatrix_metadata_compute_stage2[compute_grid]` for side effects, registration, or validation.

**CN:** 调用 `_bitmatrix_metadata_compute_stage2[compute_grid]` ，用于副作用、注册或校验。

### Block 59 — Lines 146-156 (make_bitmatrix_metadata)
```python
146|     return BitmatrixMetadata(
147|         col_sum=col_sum,
148|         col_sorted_indx=col_sorted_indx,
149|         row_sorted_indx=row_sorted_indx,
150|     )
151| 
152| 
153| # `make_bitmatrix_metadata_torch`: entry point for reference implementation
154| # ---------------------------------------------------------------------------- #
155| 
156| 
```
**EN:** Returns `BitmatrixMetadata(col_sum=col_sum, col_sorted_indx=col_sorted_indx, row_sorte...`.

**CN:** 返回 `BitmatrixMetadata(col_sum=col_sum, col_sorted_indx=col_sorted_indx, row_sorte...`.

### Block 60 — Lines 157-157 (make_bitmatrix_metadata_torch)
```python
157| def make_bitmatrix_metadata_torch(nonzero_indx, bitmatrix):
```
**EN:** Defines function `make_bitmatrix_metadata_torch(nonzero_indx, bitmatrix)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `nonzero_indx.reshape(-1).to`, `pad`, `torch.histc(nonzero_indx, bins=n_batc...`, `BitmatrixMetadata`, `torch.cat` to implement its workflow.

**CN:** 定义函数 `make_bitmatrix_metadata_torch(nonzero_indx, bitmatrix)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `nonzero_indx.reshape(-1).to`, `pad`, `torch.histc(nonzero_indx, bins=n_batc...`, `BitmatrixMetadata`, `torch.cat` 来实现其工作流程.

### Block 61 — Lines 158-158 (make_bitmatrix_metadata_torch)
```python
158|     n_batches = bitmatrix.shape[1]
```
**EN:** Assigns `n_batches` and evaluates `bitmatrix.shape[1]`.

**CN:** 将 `n_batches` and 计算 `bitmatrix.shape[1]`.

### Block 62 — Lines 159-159 (make_bitmatrix_metadata_torch)
```python
159|     nonzero_indx = nonzero_indx.reshape(-1).to(torch.int32)
```
**EN:** Assigns `nonzero_indx` and calls `nonzero_indx.reshape(-1).to`.

**CN:** 将 `nonzero_indx`，并调用 `nonzero_indx.reshape(-1).to`.

### Block 63 — Lines 160-160 (make_bitmatrix_metadata_torch)
```python
160|     pad = lambda x, total_size: torch.cat((x, torch.full((total_size - x.shape[0], ), -1, device=x.device)))
```
**EN:** Assigns `pad` and defines a lambda.

**CN:** 将 `pad` and 定义一个 lambda.

### Block 64 — Lines 161-161 (make_bitmatrix_metadata_torch)
```python
161|     col_sorted_indx = pad(torch.argsort(nonzero_indx[nonzero_indx != -1], stable=True), nonzero_indx.numel())
```
**EN:** Assigns `col_sorted_indx` and calls `pad`.

**CN:** 将 `col_sorted_indx`，并调用 `pad`.

### Block 65 — Lines 162-162 (make_bitmatrix_metadata_torch)
```python
162|     row_sorted_indx = pad(torch.argsort(col_sorted_indx[col_sorted_indx != -1], stable=True), nonzero_indx.numel())
```
**EN:** Assigns `row_sorted_indx` and calls `pad`.

**CN:** 将 `row_sorted_indx`，并调用 `pad`.

### Block 66 — Lines 163-163 (make_bitmatrix_metadata_torch)
```python
163|     col_sum = torch.histc(nonzero_indx, bins=n_batches, max=n_batches - 1).int()
```
**EN:** Assigns `col_sum` and calls `torch.histc(nonzero_indx, bins=n_batches, max=n...`.

**CN:** 将 `col_sum`，并调用 `torch.histc(nonzero_indx, bins=n_batches, max=n...`.

### Block 67 — Lines 164-168 (make_bitmatrix_metadata_torch)
```python
164|     return BitmatrixMetadata(
165|         col_sum=col_sum,
166|         col_sorted_indx=col_sorted_indx,
167|         row_sorted_indx=row_sorted_indx,
168|     )
```
**EN:** Returns `BitmatrixMetadata(col_sum=col_sum, col_sorted_indx=col_sorted_indx, row_sorte...`.

**CN:** 返回 `BitmatrixMetadata(col_sum=col_sum, col_sorted_indx=col_sorted_indx, row_sorte...`.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `BitmatrixMetadata`, `_keyed_add`, `_bitmatrix_metadata_compute_stage2`, `_bitmatrix_metadata_compute_stage1`, `cdiv`, `make_bitmatrix_metadata`, `make_bitmatrix_metadata_torch`.
  **CN:** 主要符号：`BitmatrixMetadata`, `_keyed_add`, `_bitmatrix_metadata_compute_stage2`, `_bitmatrix_metadata_compute_stage1`, `cdiv`, `make_bitmatrix_metadata`, `make_bitmatrix_metadata_torch`。
- **EN:** PyTorch tensors are the main data container and host-side orchestration layer.
  **CN:** PyTorch 张量是主要的数据容器和主机端编排层。
- **EN:** Triton is used to JIT-compile or launch GPU kernels and to express tile-level execution details.
  **CN:** Triton 用于 JIT 编译或启动 GPU 内核，并表达分块级执行细节。

## Dependencies / 依赖关系
- **EN:** External modules: `dataclasses (dataclass)`, `triton`, `triton.language`, `torch`.
  **CN:** 外部模块：`dataclasses (dataclass)`, `triton`, `triton.language`, `torch`。
- **EN:** Internal modules: `.bitmatrix_details.sum_bitmatrix_rows (sum_bitmatrix_rows)`.
  **CN:** 内部模块：`.bitmatrix_details.sum_bitmatrix_rows (sum_bitmatrix_rows)`。
