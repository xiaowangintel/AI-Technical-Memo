# compaction.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/triton_kernels/compaction.py`
- **Purpose / 用途:** Implementation module for compaction; it defines the runtime logic, helper types, and GPU-facing utilities used by the Triton kernels package via symbols compaction, compaction_torch. / 用于 compaction 的实现模块；它定义了 Triton kernels 包使用的运行时逻辑、辅助类型以及面向 GPU 的工具；核心符号包括 compaction、compaction_torch。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-5 (module)
```python
1| import torch
2| from .compaction_details._masked_compaction import _masked_compaction
3| from .tensor import Tensor
4| 
5| 
```
**EN:** This block imports `torch`, `.compaction_details._masked_compaction (_masked_compaction)`, `.tensor (Tensor)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `torch`, `.compaction_details._masked_compaction (_masked_compaction)`, `.tensor (Tensor)` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 6-6 (compaction)
```python
6| def compaction(yv, yi, bitmask, sentinel=-1):
```
**EN:** Defines function `compaction(yv, yi, bitmask, sentinel)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `torch.empty_like`, `isinstance`, `_masked_compaction[n_rows,]`, `bitmask.stride` to implement its workflow.

**CN:** 定义函数 `compaction(yv, yi, bitmask, sentinel)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `torch.empty_like`, `isinstance`, `_masked_compaction[n_rows,]`, `bitmask.stride` 来实现其工作流程.

### Block 3 — Lines 7-31 (compaction)
```python
 7|     """
 8|     Return compacted copies of *yv* and *yi* based on a per-row bitmask.
 9| 
10|     Only the elements whose index appears among the active bits of *bitmask*
11|     are kept; the rest are replaced by *sentinel*.  Kept elements preserve
12|     their original left-to-right order.
13| 
14|     Parameters
15|     ----------
16|     yv : torch.Tensor, shape (B, K)
17|         Values tensor.
18|     yi : torch.Tensor, shape (B, K), dtype torch.long
19|         Integer indices (0 ≤ index < 32) associated with *yv*.
20|     bitmask : torch.Tensor, shape (B,) **or** (B, 32)
21|         Per-row mask of active indices.  See the in-place version for details.
22|     sentinel : int, default -1
23|         Value written into dropped positions of the returned tensors.
24| 
25|     Returns
26|     -------
27|     (yv_out, yi_out) : Tuple[torch.Tensor, torch.Tensor], each shape (B, K)
28|         New tensors with the same dtype/device as the inputs.
29| 
30|     """
31| 
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 4 — Lines 32-32 (compaction)
```python
32|     n_rows, n_cols = yi.shape
```
**EN:** Assigns `n_rows`, `n_cols` and references `yi.shape`.

**CN:** 将 `n_rows`, `n_cols` and 引用 `yi.shape`.

### Block 5 — Lines 33-33 (compaction)
```python
33|     ret_yv = torch.empty_like(yv)
```
**EN:** Assigns `ret_yv` and calls `torch.empty_like`.

**CN:** 将 `ret_yv`，并调用 `torch.empty_like`.

### Block 6 — Lines 34-34 (compaction)
```python
34|     ret_yi = torch.empty_like(yi)
```
**EN:** Assigns `ret_yi` and calls `torch.empty_like`.

**CN:** 将 `ret_yi`，并调用 `torch.empty_like`.

### Block 7 — Lines 35-37 (compaction)
```python
35|     if isinstance(bitmask, Tensor):
36|         bitmask = bitmask.storage.data
37| 
```
**EN:** Checks `isinstance(bitmask, Tensor)`. The true branch mainly prepares intermediate values.

**CN:** 检查 `isinstance(bitmask, Tensor)`. 真分支主要准备中间值.

### Block 8 — Lines 38-43 (compaction)
```python
38|     _masked_compaction[(n_rows, )](
39|         yv, yi, bitmask, bitmask.stride(0), bitmask.stride(1),  # inputs
40|         ret_yv, ret_yi,  # outputs
41|         sentinel,  # sentinel
42|         K=n_cols  # constants
43|     )
```
**EN:** Calls `_masked_compaction[n_rows,]` for side effects, registration, or validation.

**CN:** 调用 `_masked_compaction[n_rows,]` ，用于副作用、注册或校验。

### Block 9 — Lines 44-46 (compaction)
```python
44|     return ret_yv, ret_yi
45| 
46| 
```
**EN:** Returns `(ret_yv, ret_yi)`.

**CN:** 返回 `(ret_yv, ret_yi)`.

### Block 10 — Lines 47-47 (compaction_torch)
```python
47| def compaction_torch(yv: torch.Tensor, yi: torch.Tensor, bitmask: torch.Tensor, sentinel=-1):
```
**EN:** Defines function `compaction_torch(yv, yi, bitmask, sentinel)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `bits.flatten`, `mask.gather`, `(~keep).to(torch.int).argsort`, `yi.gather`, `yv.gather` to implement its workflow.

**CN:** 定义函数 `compaction_torch(yv, yi, bitmask, sentinel)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `bits.flatten`, `mask.gather`, `(~keep).to(torch.int).argsort`, `yi.gather`, `yv.gather` 来实现其工作流程.

### Block 11 — Lines 48-50 (compaction_torch)
```python
48|     """
49|     reference implementation of `masked_compact`
50|     """
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 12 — Lines 51-51 (compaction_torch)
```python
51|     B, K = yi.shape
```
**EN:** Assigns `B`, `K` and references `yi.shape`.

**CN:** 将 `B`, `K` and 引用 `yi.shape`.

### Block 13 — Lines 52-53 (compaction_torch)
```python
52|     device = yi.device
53|     # Expand bitmask to a boolean matrix of active bits  (B, 32)
```
**EN:** Assigns `device` and references `yi.device`.

**CN:** 将 `device` and 引用 `yi.device`.

### Block 14 — Lines 54-54 (compaction_torch)
```python
54|     w = (1 << torch.arange(32, device=device, dtype=bitmask.dtype))
```
**EN:** Assigns `w` and evaluates `1 << torch.arange(32, device=device, dtype=bitmask.dtype)`.

**CN:** 将 `w` and 计算 `1 << torch.arange(32, device=device, dtype=bitmask.dtype)`.

### Block 15 — Lines 55-55 (compaction_torch)
```python
55|     bits = (bitmask.unsqueeze(-1) & w) != 0
```
**EN:** Assigns `bits` and evaluates `bitmask.unsqueeze(-1) & w != 0`.

**CN:** 将 `bits` and 计算 `bitmask.unsqueeze(-1) & w != 0`.

### Block 16 — Lines 56-57 (compaction_torch)
```python
56|     mask = bits.flatten(start_dim=-2)  # or bits.reshape(B, -1)
57|     # For every yi element decide whether it should be kept
```
**EN:** Assigns `mask` and calls `bits.flatten`.

**CN:** 将 `mask`，并调用 `bits.flatten`.

### Block 17 — Lines 58-60 (compaction_torch)
```python
58|     keep = mask.gather(1, yi.long())
59|     # Build a stable permutation that brings all "keep" items forward
60|     #    False→0, True→1  ==> invert so kept==0, dropped==1, then argsort
```
**EN:** Assigns `keep` and calls `mask.gather`.

**CN:** 将 `keep`，并调用 `mask.gather`.

### Block 18 — Lines 61-62 (compaction_torch)
```python
61|     order = (~keep).to(torch.int).argsort(dim=1, stable=True)
62|     # Re‑order tensors according to above permutation
```
**EN:** Assigns `order` and calls `(~keep).to(torch.int).argsort`.

**CN:** 将 `order`，并调用 `(~keep).to(torch.int).argsort`.

### Block 19 — Lines 63-63 (compaction_torch)
```python
63|     yi_sorted = yi.gather(1, order)
```
**EN:** Assigns `yi_sorted` and calls `yi.gather`.

**CN:** 将 `yi_sorted`，并调用 `yi.gather`.

### Block 20 — Lines 64-65 (compaction_torch)
```python
64|     yv_sorted = yv.gather(1, order)
65|     # fill relevant positions with sentinel
```
**EN:** Assigns `yv_sorted` and calls `yv.gather`.

**CN:** 将 `yv_sorted`，并调用 `yv.gather`.

### Block 21 — Lines 66-66 (compaction_torch)
```python
66|     keep_sorted = keep.gather(1, order)
```
**EN:** Assigns `keep_sorted` and calls `keep.gather`.

**CN:** 将 `keep_sorted`，并调用 `keep.gather`.

### Block 22 — Lines 67-67 (compaction_torch)
```python
67|     yi_sorted[~keep_sorted] = sentinel
```
**EN:** Assigns `yi_sorted[~keep_sorted]` and references `sentinel`.

**CN:** 将 `yi_sorted[~keep_sorted]` and 引用 `sentinel`.

### Block 23 — Lines 68-68 (compaction_torch)
```python
68|     yv_sorted[~keep_sorted] = sentinel
```
**EN:** Assigns `yv_sorted[~keep_sorted]` and references `sentinel`.

**CN:** 将 `yv_sorted[~keep_sorted]` and 引用 `sentinel`.

### Block 24 — Lines 69-69 (compaction_torch)
```python
69|     return yv_sorted, yi_sorted
```
**EN:** Returns `(yv_sorted, yi_sorted)`.

**CN:** 返回 `(yv_sorted, yi_sorted)`.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `compaction`, `compaction_torch`.
  **CN:** 主要符号：`compaction`, `compaction_torch`。
- **EN:** PyTorch tensors are the main data container and host-side orchestration layer.
  **CN:** PyTorch 张量是主要的数据容器和主机端编排层。

## Dependencies / 依赖关系
- **EN:** External modules: `torch`.
  **CN:** 外部模块：`torch`。
- **EN:** Internal modules: `.compaction_details._masked_compaction (_masked_compaction)`, `.tensor (Tensor)`.
  **CN:** 内部模块：`.compaction_details._masked_compaction (_masked_compaction)`, `.tensor (Tensor)`。
