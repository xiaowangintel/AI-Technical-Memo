# strided.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/triton_kernels/tensor_details/layout_details/strided.py`
- **Purpose / 用途:** Implementation module for strided; it defines the runtime logic, helper types, and GPU-facing utilities used by the Triton kernels package via symbols StridedLayout, StridedLayoutTransformation. / 用于 strided 的实现模块；它定义了 Triton kernels 包使用的运行时逻辑、辅助类型以及面向 GPU 的工具；核心符号包括 StridedLayout、StridedLayoutTransformation。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-7 (module)
```python
1| from dataclasses import dataclass
2| from .base import Layout, LayoutTransformation
3| from .torch_utils import repack
4| import torch
5| 
6| 
7| # ------------------- Layout Definition -------------------
```
**EN:** This block imports `dataclasses (dataclass)`, `.base (Layout, LayoutTransformation)`, `.torch_utils (repack)`, `torch` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `dataclasses (dataclass)`, `.base (Layout, LayoutTransformation)`, `.torch_utils (repack)`, `torch` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 8-21 (StridedLayout)
```python
 8| @dataclass(frozen=True)
 9| class StridedLayout(Layout):
10| 
11|     # NOTE: We only encode the (logical) major dimension; the full dimension order is
12|     # derived from the tensor rank. This keeps the API minimal while still allowing
13|     # "which dim is contiguous/packed" to be expressed.
14|     #
15|     # For a tensor of rank `R`, the derived order is:
16|     #   base = list(reversed(range(R)))
17|     #   swap base[0] with base[index(major_dim)]
18|     #   order = base
19|     #
20|     # This matches the previous default `order=list(reversed(range(R)))` when
21|     # `major_dim == R - 1`.
```
**EN:** Defines class `StridedLayout` inheriting from `Layout` with decorators `dataclass(frozen=True)` to organize related behavior. Key fields include `major_dim`. Main methods are `__post_init__`, `make_transformation`, `name`, `swizzle_block_shape`, `order`.

**CN:** 定义类 `StridedLayout`，继承自 `Layout`，带有装饰器 `dataclass(frozen=True)`，用于组织相关行为。关键字段包括 `major_dim`。主要方法有 `__post_init__`, `make_transformation`, `name`, `swizzle_block_shape`, `order`.

### Block 3 — Lines 22-23 (StridedLayout)
```python
22|     major_dim: int = -1
23| 
```
**EN:** Annotated assignment stores `major_dim` and evaluates `-1`.

**CN:** 带类型注解的赋值保存 `major_dim` and 计算 `-1`.

### Block 4 — Lines 24-24 (__post_init__)
```python
24|     def __post_init__(self):
```
**EN:** Defines function `__post_init__(self)` for this module. The body mainly branches on runtime conditions. It uses calls such as `isinstance`, `TypeError`, `type` to implement its workflow.

**CN:** 定义函数 `__post_init__(self)`，供本模块使用. 主体主要根据运行时条件分支. 其中会调用 `isinstance`, `TypeError`, `type` 来实现其工作流程.

### Block 5 — Lines 25-27 (__post_init__)
```python
25|         if not isinstance(self.major_dim, int):
26|             raise TypeError(f"StridedLayout(major_dim=...) must be an int, got {type(self.major_dim)}")
27| 
```
**EN:** Checks `not isinstance(self.major_dim, int)`..

**CN:** 检查 `not isinstance(self.major_dim, int)`..

### Block 6 — Lines 28-28 (make_transformation)
```python
28|     def make_transformation(self, shape: list[int], is_fp4: bool) -> LayoutTransformation:
```
**EN:** Defines function `make_transformation(self, shape, is_fp4)` for this module. The body mainly returns the computed result. It uses calls such as `StridedLayoutTransformation`, `self.order`, `len` to implement its workflow.

**CN:** 定义函数 `make_transformation(self, shape, is_fp4)`，供本模块使用. 主体主要返回计算结果. 其中会调用 `StridedLayoutTransformation`, `self.order`, `len` 来实现其工作流程.

### Block 7 — Lines 29-30 (make_transformation)
```python
29|         return StridedLayoutTransformation(shape, is_fp4, self.order(len(shape)))
30| 
```
**EN:** Returns `StridedLayoutTransformation(shape, is_fp4, self.order(len(shape)))`.

**CN:** 返回 `StridedLayoutTransformation(shape, is_fp4, self.order(len(shape)))`.

### Block 8 — Lines 31-32 (name)
```python
31|     @property
32|     def name(self):
```
**EN:** Defines function `name(self)` with decorators `property` for this module. The body mainly returns the computed result.

**CN:** 定义函数 `name(self)`，带有装饰器 `property`，供本模块使用. 主体主要返回计算结果.

### Block 9 — Lines 33-34 (name)
```python
33|         return "STRIDED"
34| 
```
**EN:** Returns `'STRIDED'`.

**CN:** 返回 `'STRIDED'`.

### Block 10 — Lines 35-35 (swizzle_block_shape)
```python
35|     def swizzle_block_shape(self, block_shape):
```
**EN:** Defines function `swizzle_block_shape(self, block_shape)` for this module. The body mainly returns the computed result.

**CN:** 定义函数 `swizzle_block_shape(self, block_shape)`，供本模块使用. 主体主要返回计算结果.

### Block 11 — Lines 36-37 (swizzle_block_shape)
```python
36|         return block_shape
37| 
```
**EN:** Returns `block_shape`.

**CN:** 返回 `block_shape`.

### Block 12 — Lines 38-38 (order)
```python
38|     def order(self, rank: int) -> list[int]:
```
**EN:** Defines function `order(self, rank)` for this module. The body mainly branches on runtime conditions; branches on runtime conditions; prepares intermediate values. It uses calls such as `list`, `base.index`, `ValueError`, `reversed`, `range` to implement its workflow.

**CN:** 定义函数 `order(self, rank)`，供本模块使用. 主体主要根据运行时条件分支; 根据运行时条件分支; 准备中间值. 其中会调用 `list`, `base.index`, `ValueError`, `reversed`, `range` 来实现其工作流程.

### Block 13 — Lines 39-43 (order)
```python
39|         """
40|         Returns the minor->major dimension order for a given tensor rank.
41| 
42|         `self.major_dim` supports negative indexing (like Python).
43|         """
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 14 — Lines 44-45 (order)
```python
44|         if rank <= 0:
45|             return []
```
**EN:** Checks `rank <= 0`. The true branch mainly returns the computed result.

**CN:** 检查 `rank <= 0`. 真分支主要返回计算结果.

### Block 15 — Lines 46-47 (order)
```python
46|         if not (-rank <= self.major_dim < rank):
47|             raise ValueError(f"Invalid StridedLayout.major_dim={self.major_dim} for rank={rank}")
```
**EN:** Checks `not -rank <= self.major_dim < rank`..

**CN:** 检查 `not -rank <= self.major_dim < rank`..

### Block 16 — Lines 48-48 (order)
```python
48|         major_dim = self.major_dim if self.major_dim >= 0 else self.major_dim + rank
```
**EN:** Assigns `major_dim` and uses conditional expression `self.major_dim if self.major_dim >= 0 else self.major_dim...`.

**CN:** 将 `major_dim` and 使用条件表达式 `self.major_dim if self.major_dim >= 0 else self.major_dim...`.

### Block 17 — Lines 49-51 (order)
```python
49|         base = list(reversed(range(rank)))
50|         # Preserve the previous behavior: derive from canonical reversed order, then
51|         # swap the requested major dimension into position 0.
```
**EN:** Assigns `base` and calls `list`.

**CN:** 将 `base`，并调用 `list`.

### Block 18 — Lines 52-52 (order)
```python
52|         idx = base.index(major_dim)
```
**EN:** Assigns `idx` and calls `base.index`.

**CN:** 将 `idx`，并调用 `base.index`.

### Block 19 — Lines 53-53 (order)
```python
53|         base[0], base[idx] = base[idx], base[0]
```
**EN:** Assigns `base[0]`, `base[idx]` and builds a tuple.

**CN:** 将 `base[0]`, `base[idx]` and 构造一个元组.

### Block 20 — Lines 54-56 (order)
```python
54|         return base
55| 
56| 
```
**EN:** Returns `base`.

**CN:** 返回 `base`.

### Block 21 — Lines 57-59 (StridedLayoutTransformation)
```python
57| @dataclass(frozen=True)
58| class StridedLayoutTransformation(LayoutTransformation):
59| 
```
**EN:** Defines class `StridedLayoutTransformation` inheriting from `LayoutTransformation` with decorators `dataclass(frozen=True)` to organize related behavior. Key fields include `order`. Main methods are `swizzle_data`, `unswizzle_data`.

**CN:** 定义类 `StridedLayoutTransformation`，继承自 `LayoutTransformation`，带有装饰器 `dataclass(frozen=True)`，用于组织相关行为。关键字段包括 `order`。主要方法有 `swizzle_data`, `unswizzle_data`.

### Block 22 — Lines 60-61 (StridedLayoutTransformation)
```python
60|     order: list[int]
61| 
```
**EN:** Annotated assignment stores `order` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `order` and 声明该带注解的字段.

### Block 23 — Lines 62-62 (swizzle_data)
```python
62|     def swizzle_data(self, data):
```
**EN:** Defines function `swizzle_data(self, data)` for this module. The body mainly checks invariants; prepares intermediate values; branches on runtime conditions. It uses calls such as `len`, `list`, `torch.empty_strided`, `repack`, `data.numel` to implement its workflow.

**CN:** 定义函数 `swizzle_data(self, data)`，供本模块使用. 主体主要检查不变量; 准备中间值; 根据运行时条件分支. 其中会调用 `len`, `list`, `torch.empty_strided`, `repack`, `data.numel` 来实现其工作流程.

### Block 24 — Lines 63-63 (swizzle_data)
```python
63|         assert data.numel() == 0 or data.stride(-1) == 1
```
**EN:** Asserts `data.numel() == 0 or data.stride(-1) == 1` to enforce invariants.

**CN:** 断言 `data.numel() == 0 or data.stride(-1) == 1` 以确保不变量成立。

### Block 25 — Lines 64-64 (swizzle_data)
```python
64|         r = len(self.shape)
```
**EN:** Assigns `r` and calls `len`.

**CN:** 将 `r`，并调用 `len`.

### Block 26 — Lines 65-66 (swizzle_data)
```python
65|         if r == 0:
66|             return data
```
**EN:** Checks `r == 0`. The true branch mainly returns the computed result.

**CN:** 检查 `r == 0`. 真分支主要返回计算结果.

### Block 27 — Lines 67-67 (swizzle_data)
```python
67|         pd = self.order[0]  # packed/contiguous dim in output
```
**EN:** Assigns `pd` and evaluates `self.order[0]`.

**CN:** 将 `pd` and 计算 `self.order[0]`.

### Block 28 — Lines 68-68 (swizzle_data)
```python
68|         out_shape = list(self.shape)
```
**EN:** Assigns `out_shape` and calls `list`.

**CN:** 将 `out_shape`，并调用 `list`.

### Block 29 — Lines 69-71 (swizzle_data)
```python
69|         if self.is_fp4:
70|             out_shape[pd] //= 2
71|         # dense strides in minor->major `self.order`
```
**EN:** Checks `self.is_fp4`..

**CN:** 检查 `self.is_fp4`..

### Block 30 — Lines 72-72 (swizzle_data)
```python
72|         stride, s = [0] * r, 1
```
**EN:** Assigns `stride`, `s` and builds a tuple.

**CN:** 将 `stride`, `s` and 构造一个元组.

### Block 31 — Lines 73-74 (swizzle_data)
```python
73|         for d in self.order:
74|             stride[d], s = s, s * out_shape[d]
```
**EN:** Loops over `self.order` with target `d`. The loop body mainly prepares intermediate values.

**CN:** 遍历 `self.order` ，目标变量为 `d`. 循环体主要准备中间值.

### Block 32 — Lines 75-75 (swizzle_data)
```python
75|         out = torch.empty_strided(out_shape, stride, dtype=data.dtype, device=data.device)
```
**EN:** Assigns `out` and calls `torch.empty_strided`.

**CN:** 将 `out`，并调用 `torch.empty_strided`.

### Block 33 — Lines 76-76 (swizzle_data)
```python
76|         repack(data, -1, pd, self.is_fp4, out=out)
```
**EN:** Calls `repack` for side effects, registration, or validation.

**CN:** 调用 `repack` ，用于副作用、注册或校验。

### Block 34 — Lines 77-78 (swizzle_data)
```python
77|         return out
78| 
```
**EN:** Returns `out`.

**CN:** 返回 `out`.

### Block 35 — Lines 79-79 (unswizzle_data)
```python
79|     def unswizzle_data(self, data):
```
**EN:** Defines function `unswizzle_data(self, data)` for this module. The body mainly checks invariants; prepares intermediate values; branches on runtime conditions. It uses calls such as `list`, `torch.empty`, `repack`, `data.stride`, `ret.stride` to implement its workflow.

**CN:** 定义函数 `unswizzle_data(self, data)`，供本模块使用. 主体主要检查不变量; 准备中间值; 根据运行时条件分支. 其中会调用 `list`, `torch.empty`, `repack`, `data.stride`, `ret.stride` 来实现其工作流程.

### Block 36 — Lines 80-80 (unswizzle_data)
```python
80|         assert data.stride(self.order[0]) == 1
```
**EN:** Asserts `data.stride(self.order[0]) == 1` to enforce invariants.

**CN:** 断言 `data.stride(self.order[0]) == 1` 以确保不变量成立。

### Block 37 — Lines 81-81 (unswizzle_data)
```python
81|         out_shape = list(self.shape)
```
**EN:** Assigns `out_shape` and calls `list`.

**CN:** 将 `out_shape`，并调用 `list`.

### Block 38 — Lines 82-83 (unswizzle_data)
```python
82|         if self.is_fp4:
83|             out_shape[-1] //= 2
```
**EN:** Checks `self.is_fp4`..

**CN:** 检查 `self.is_fp4`..

### Block 39 — Lines 84-84 (unswizzle_data)
```python
84|         ret = torch.empty(out_shape, dtype=data.dtype, device=data.device)
```
**EN:** Assigns `ret` and calls `torch.empty`.

**CN:** 将 `ret`，并调用 `torch.empty`.

### Block 40 — Lines 85-85 (unswizzle_data)
```python
85|         repack(data, self.order[0], -1, self.is_fp4, out=ret)
```
**EN:** Calls `repack` for side effects, registration, or validation.

**CN:** 调用 `repack` ，用于副作用、注册或校验。

### Block 41 — Lines 86-86 (unswizzle_data)
```python
86|         assert ret.stride(-1) == 1
```
**EN:** Asserts `ret.stride(-1) == 1` to enforce invariants.

**CN:** 断言 `ret.stride(-1) == 1` 以确保不变量成立。

### Block 42 — Lines 87-87 (unswizzle_data)
```python
87|         return ret
```
**EN:** Returns `ret`.

**CN:** 返回 `ret`.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `StridedLayout`, `StridedLayoutTransformation`.
  **CN:** 主要符号：`StridedLayout`, `StridedLayoutTransformation`。
- **EN:** PyTorch tensors are the main data container and host-side orchestration layer.
  **CN:** PyTorch 张量是主要的数据容器和主机端编排层。
- **EN:** Custom layout objects describe how logical tensors are packed or swizzled in memory.
  **CN:** 自定义 layout 对象描述逻辑张量如何在内存中打包或重排。

## Dependencies / 依赖关系
- **EN:** External modules: `dataclasses (dataclass)`, `torch`.
  **CN:** 外部模块：`dataclasses (dataclass)`, `torch`。
- **EN:** Internal modules: `.base (Layout, LayoutTransformation)`, `.torch_utils (repack)`.
  **CN:** 内部模块：`.base (Layout, LayoutTransformation)`, `.torch_utils (repack)`。
