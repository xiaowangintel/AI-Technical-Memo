# torch_utils.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/triton_kernels/tensor_details/layout_details/torch_utils.py`
- **Purpose / 用途:** Implementation module for torch utils; it defines the runtime logic, helper types, and GPU-facing utilities used by the Triton kernels package via symbols repack. / 用于 torch utils 的实现模块；它定义了 Triton kernels 包使用的运行时逻辑、辅助类型以及面向 GPU 的工具；核心符号包括 repack。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-43 (module)
```python
 1| import torch
 2| 
 3| # def unpack(data: torch.Tensor, dim: int, is_fp4: bool):
 4| #     if not is_fp4:
 5| #         return data
 6| #     if data.shape[dim] == 1:
 7| #         return data
 8| #     ret_shape = list(data.shape)
 9| #     ret_shape[dim] *= 2
10| #     ret = torch.empty(ret_shape, dtype=data.dtype, device=data.device)
11| #     idx_lo = [slice(None)] * data.ndim
12| #     idx_hi = [slice(None)] * data.ndim
13| #     idx_lo[dim] = slice(0, data.shape[dim]*2, 2)
14| #     idx_hi[dim] = slice(1, data.shape[dim]*2, 2)
15| #     ret[tuple(idx_lo)] = data & 0x0F
16| #     ret[tuple(idx_hi)] = data & 0xF0
17| #     ret[tuple(idx_hi)] >>= 4
18| #     return ret
19| 
20| # def pack(data: torch.Tensor, dim: int, is_fp4: bool):
21| #     if not is_fp4:
22| #         return data
23| #     if data.shape[dim] == 1:
24| #         return data
25| #     size = data.shape[dim] // 2
26| #     idx_lo = [slice(None)] * data.ndim
27| #     idx_hi = [slice(None)] * data.ndim
28| #     idx_lo[dim] = slice(0, size*2, 2)
29| #     idx_hi[dim] = slice(1, size*2, 2)
30| #     out = (data[tuple(idx_hi)] << 4)
31| #     out |= data[tuple(idx_lo)]
32| #     return out
33| 
34| # def repack(data: torch.Tensor, old_dim: int, new_dim: int, is_fp4: bool):
35| #     old_dim %= data.ndim
36| #     new_dim %= data.ndim
37| #     if not is_fp4 or old_dim == new_dim:
38| #         return data
39| #     tmp = unpack(data, old_dim, is_fp4)
40| #     ret = pack(tmp, new_dim, is_fp4)
41| #     return ret
42| 
43| 
```
**EN:** This block imports `torch` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `torch` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 44-44 (repack)
```python
44| def repack(data: torch.Tensor, old_dim: int, new_dim: int, is_fp4: bool, out=None) -> torch.Tensor:
```
**EN:** Defines function `repack(data, old_dim, new_dim, is_fp4, out)` for this module. The body mainly branches on runtime conditions; branches on runtime conditions; prepares intermediate values. It uses calls such as `list`, `_idx`, `out_odd.copy_`, `out_odd.bitwise_and_`, `out_even.copy_` to implement its workflow.

**CN:** 定义函数 `repack(data, old_dim, new_dim, is_fp4, out)`，供本模块使用. 主体主要根据运行时条件分支; 根据运行时条件分支; 准备中间值. 其中会调用 `list`, `_idx`, `out_odd.copy_`, `out_odd.bitwise_and_`, `out_even.copy_` 来实现其工作流程.

### Block 3 — Lines 45-45 (repack)
```python
45|     old_dim %= data.ndim
```
**EN:** Updates `old_dim` with operator `Mod` using `data.ndim`.

**CN:** 更新 `old_dim`，使用运算符 `Mod`，并使用 `data.ndim`.

### Block 4 — Lines 46-46 (repack)
```python
46|     new_dim %= data.ndim
```
**EN:** Updates `new_dim` with operator `Mod` using `data.ndim`.

**CN:** 更新 `new_dim`，使用运算符 `Mod`，并使用 `data.ndim`.

### Block 5 — Lines 47-51 (repack)
```python
47|     if (not is_fp4) or (old_dim == new_dim):
48|         if out is not None:
49|             out.copy_(data)
50|             return out
51|         return data
```
**EN:** Checks `not is_fp4 or old_dim == new_dim`. The true branch mainly branches on runtime conditions; returns the computed result.

**CN:** 检查 `not is_fp4 or old_dim == new_dim`. 真分支主要根据运行时条件分支; 返回计算结果.

### Block 6 — Lines 52-53 (repack)
```python
52|     if data.dtype.is_floating_point:
53|         raise TypeError(f"Expected integer dtype for bitwise ops, got {data.dtype}")
```
**EN:** Checks `data.dtype.is_floating_point`..

**CN:** 检查 `data.dtype.is_floating_point`..

### Block 7 — Lines 54-54 (repack)
```python
54|     out_shape = list(data.shape)
```
**EN:** Assigns `out_shape` and calls `list`.

**CN:** 将 `out_shape`，并调用 `list`.

### Block 8 — Lines 55-55 (repack)
```python
55|     out_shape[old_dim] *= 2
```
**EN:** Updates `out_shape[old_dim]` with operator `Mult` using `2`.

**CN:** 更新 `out_shape[old_dim]`，使用运算符 `Mult`，并使用 `2`.

### Block 9 — Lines 56-56 (repack)
```python
56|     out_shape[new_dim] //= 2
```
**EN:** Updates `out_shape[new_dim]` with operator `FloorDiv` using `2`.

**CN:** 更新 `out_shape[new_dim]`，使用运算符 `FloorDiv`，并使用 `2`.

### Block 10 — Lines 57-59 (repack)
```python
57|     if out is None:
58|         out = torch.empty(out_shape, dtype=data.dtype, device=data.device)
59| 
```
**EN:** Checks `out is None`. The true branch mainly prepares intermediate values.

**CN:** 检查 `out is None`. 真分支主要准备中间值.

### Block 11 — Lines 60-60 (_idx)
```python
60|     def _idx(ndim: int, dim: int, sl: slice):
```
**EN:** Defines function `_idx(ndim, dim, sl)` for this module. The body mainly prepares intermediate values; prepares intermediate values; returns the computed result. It uses calls such as `tuple`, `slice` to implement its workflow.

**CN:** 定义函数 `_idx(ndim, dim, sl)`，供本模块使用. 主体主要准备中间值; 准备中间值; 返回计算结果. 其中会调用 `tuple`, `slice` 来实现其工作流程.

### Block 12 — Lines 61-61 (_idx)
```python
61|         idx = [slice(None)] * ndim
```
**EN:** Assigns `idx` and evaluates `[slice(None)] * ndim`.

**CN:** 将 `idx` and 计算 `[slice(None)] * ndim`.

### Block 13 — Lines 62-62 (_idx)
```python
62|         idx[dim] = sl
```
**EN:** Assigns `idx[dim]` and references `sl`.

**CN:** 将 `idx[dim]` and 引用 `sl`.

### Block 14 — Lines 63-65 (_idx)
```python
63|         return tuple(idx)
64| 
65|     # data slices along new_dim (pairwise)
```
**EN:** Returns `tuple(idx)`.

**CN:** 返回 `tuple(idx)`.

### Block 15 — Lines 66-66 (repack)
```python
66|     d_even = _idx(data.ndim, new_dim, slice(0, None, 2))
```
**EN:** Assigns `d_even` and calls `_idx`.

**CN:** 将 `d_even`，并调用 `_idx`.

### Block 16 — Lines 67-68 (repack)
```python
67|     d_odd = _idx(data.ndim, new_dim, slice(1, None, 2))
68|     # out slices along old_dim (interleave into even/odd positions)
```
**EN:** Assigns `d_odd` and calls `_idx`.

**CN:** 将 `d_odd`，并调用 `_idx`.

### Block 17 — Lines 69-69 (repack)
```python
69|     r_even = _idx(out.ndim, old_dim, slice(0, None, 2))
```
**EN:** Assigns `r_even` and calls `_idx`.

**CN:** 将 `r_even`，并调用 `_idx`.

### Block 18 — Lines 70-71 (repack)
```python
70|     r_odd = _idx(out.ndim, old_dim, slice(1, None, 2))
71|     #
```
**EN:** Assigns `r_odd` and calls `_idx`.

**CN:** 将 `r_odd`，并调用 `_idx`.

### Block 19 — Lines 72-72 (repack)
```python
72|     out_even = out[r_even]
```
**EN:** Assigns `out_even` and evaluates `out[r_even]`.

**CN:** 将 `out_even` and 计算 `out[r_even]`.

### Block 20 — Lines 73-73 (repack)
```python
73|     out_odd = out[r_odd]
```
**EN:** Assigns `out_odd` and evaluates `out[r_odd]`.

**CN:** 将 `out_odd` and 计算 `out[r_odd]`.

### Block 21 — Lines 74-74 (repack)
```python
74|     a = data[d_even]
```
**EN:** Assigns `a` and evaluates `data[d_even]`.

**CN:** 将 `a` and 计算 `data[d_even]`.

### Block 22 — Lines 75-77 (repack)
```python
75|     b = data[d_odd]
76| 
77|     # ---- build out_odd first, using out_even as scratch ----
```
**EN:** Assigns `b` and evaluates `data[d_odd]`.

**CN:** 将 `b` and 计算 `data[d_odd]`.

### Block 23 — Lines 78-78 (repack)
```python
78|     out_odd.copy_(b)
```
**EN:** Calls `out_odd.copy_` for side effects, registration, or validation.

**CN:** 调用 `out_odd.copy_` ，用于副作用、注册或校验。

### Block 24 — Lines 79-80 (repack)
```python
79|     out_odd.bitwise_and_(0xF0)  # out_odd = b & 0xF0
80| 
```
**EN:** Calls `out_odd.bitwise_and_` for side effects, registration, or validation.

**CN:** 调用 `out_odd.bitwise_and_` ，用于副作用、注册或校验。

### Block 25 — Lines 81-81 (repack)
```python
81|     out_even.copy_(a)
```
**EN:** Calls `out_even.copy_` for side effects, registration, or validation.

**CN:** 调用 `out_even.copy_` ，用于副作用、注册或校验。

### Block 26 — Lines 82-83 (repack)
```python
82|     out_even.bitwise_right_shift_(4)  # out_even (scratch) = a >> 4
83| 
```
**EN:** Calls `out_even.bitwise_right_shift_` for side effects, registration, or validation.

**CN:** 调用 `out_even.bitwise_right_shift_` ，用于副作用、注册或校验。

### Block 27 — Lines 84-86 (repack)
```python
84|     out_odd.bitwise_or_(out_even)  # out_odd = (a >> 4) | (b & 0xF0)
85| 
86|     # ---- now build out_even, no tmp by using add_(alpha=16) ----
```
**EN:** Calls `out_odd.bitwise_or_` for side effects, registration, or validation.

**CN:** 调用 `out_odd.bitwise_or_` ，用于副作用、注册或校验。

### Block 28 — Lines 87-87 (repack)
```python
87|     out_even.copy_(a)
```
**EN:** Calls `out_even.copy_` for side effects, registration, or validation.

**CN:** 调用 `out_even.copy_` ，用于副作用、注册或校验。

### Block 29 — Lines 88-88 (repack)
```python
88|     out_even.bitwise_and_(0x0F)  # out_even = a & 0x0F
```
**EN:** Calls `out_even.bitwise_and_` for side effects, registration, or validation.

**CN:** 调用 `out_even.bitwise_and_` ，用于副作用、注册或校验。

### Block 30 — Lines 89-90 (repack)
```python
89|     out_even.add_(b, alpha=16)  # out_even += 16*b  == (b << 4) | (a & 0x0F)
90| 
```
**EN:** Calls `out_even.add_` for side effects, registration, or validation.

**CN:** 调用 `out_even.add_` ，用于副作用、注册或校验。

### Block 31 — Lines 91-91 (repack)
```python
91|     return out
```
**EN:** Returns `out`.

**CN:** 返回 `out`.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `repack`.
  **CN:** 主要符号：`repack`。
- **EN:** PyTorch tensors are the main data container and host-side orchestration layer.
  **CN:** PyTorch 张量是主要的数据容器和主机端编排层。
- **EN:** Custom layout objects describe how logical tensors are packed or swizzled in memory.
  **CN:** 自定义 layout 对象描述逻辑张量如何在内存中打包或重排。

## Dependencies / 依赖关系
- **EN:** External modules: `torch`.
  **CN:** 外部模块：`torch`。
- **EN:** Internal modules: none at the top level.
  **CN:** 顶层内部模块：无。
