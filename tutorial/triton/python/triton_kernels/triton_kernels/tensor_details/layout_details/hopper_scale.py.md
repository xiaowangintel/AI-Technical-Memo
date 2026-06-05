# hopper_scale.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/triton_kernels/tensor_details/layout_details/hopper_scale.py`
- **Purpose / 用途:** Implementation module for hopper scale; it defines the runtime logic, helper types, and GPU-facing utilities used by the Triton kernels package via symbols HopperMXScaleLayout, HopperMXScaleLayoutTransformation, unswizzle_mxfp4_scale_hopper. / 用于 hopper scale 的实现模块；它定义了 Triton kernels 包使用的运行时逻辑、辅助类型以及面向 GPU 的工具；核心符号包括 HopperMXScaleLayout、HopperMXScaleLayoutTransformation、unswizzle_mxfp4_scale_hopper。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-9 (module)
```python
1| from dataclasses import dataclass
2| import torch
3| import triton
4| import triton.language as tl
5| from .base import Layout, LayoutTransformation
6| 
7| # ------------------- Hopper MX Scale Layout -------------------
8| 
9| 
```
**EN:** This block imports `dataclasses (dataclass)`, `torch`, `triton`, `triton.language`, `.base (Layout, LayoutTransformation)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `dataclasses (dataclass)`, `torch`, `triton`, `triton.language`, `.base (Layout, LayoutTransformation)` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 10-11 (HopperMXScaleLayout)
```python
10| @dataclass(frozen=True)
11| class HopperMXScaleLayout(Layout):
```
**EN:** Defines class `HopperMXScaleLayout` inheriting from `Layout` with decorators `dataclass(frozen=True)` to organize related behavior. Key fields include `mx_axis`, `num_warps`. Main methods are `__post_init__`, `name`, `make_transformation`, `swizzle_block_shape`.

**CN:** 定义类 `HopperMXScaleLayout`，继承自 `Layout`，带有装饰器 `dataclass(frozen=True)`，用于组织相关行为。关键字段包括 `mx_axis`, `num_warps`。主要方法有 `__post_init__`, `name`, `make_transformation`, `swizzle_block_shape`.

### Block 3 — Lines 12-12 (HopperMXScaleLayout)
```python
12|     mx_axis: int
```
**EN:** Annotated assignment stores `mx_axis` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `mx_axis` and 声明该带注解的字段.

### Block 4 — Lines 13-14 (HopperMXScaleLayout)
```python
13|     num_warps: int
14| 
```
**EN:** Annotated assignment stores `num_warps` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `num_warps` and 声明该带注解的字段.

### Block 5 — Lines 15-15 (__post_init__)
```python
15|     def __post_init__(self):
```
**EN:** Defines function `__post_init__(self)` for this module. The body mainly checks invariants.

**CN:** 定义函数 `__post_init__(self)`，供本模块使用. 主体主要检查不变量.

### Block 6 — Lines 16-17 (__post_init__)
```python
16|         assert self.num_warps & (self.num_warps - 1) == 0, "warps_n must be a power of 2"
17| 
```
**EN:** Asserts `self.num_warps & self.num_warps - 1 == 0` to enforce invariants.

**CN:** 断言 `self.num_warps & self.num_warps - 1 == 0` 以确保不变量成立。

### Block 7 — Lines 18-19 (name)
```python
18|     @property
19|     def name(self):
```
**EN:** Defines function `name(self)` with decorators `property` for this module. The body mainly returns the computed result.

**CN:** 定义函数 `name(self)`，带有装饰器 `property`，供本模块使用. 主体主要返回计算结果.

### Block 8 — Lines 20-21 (name)
```python
20|         return "HOPPER_SCALE"
21| 
```
**EN:** Returns `'HOPPER_SCALE'`.

**CN:** 返回 `'HOPPER_SCALE'`.

### Block 9 — Lines 22-22 (make_transformation)
```python
22|     def make_transformation(self, shape: list[int], is_fp4) -> LayoutTransformation:
```
**EN:** Defines function `make_transformation(self, shape, is_fp4)` for this module. The body mainly returns the computed result. It uses calls such as `HopperMXScaleLayoutTransformation` to implement its workflow.

**CN:** 定义函数 `make_transformation(self, shape, is_fp4)`，供本模块使用. 主体主要返回计算结果. 其中会调用 `HopperMXScaleLayoutTransformation` 来实现其工作流程.

### Block 10 — Lines 23-24 (make_transformation)
```python
23|         return HopperMXScaleLayoutTransformation(shape, is_fp4, self.mx_axis, self.num_warps)
24| 
```
**EN:** Returns `HopperMXScaleLayoutTransformation(shape, is_fp4, self.mx_axis, self.num_warps)`.

**CN:** 返回 `HopperMXScaleLayoutTransformation(shape, is_fp4, self.mx_axis, self.num_warps)`.

### Block 11 — Lines 25-26 (swizzle_block_shape)
```python
25|     def swizzle_block_shape(self, block_shape):
26|         # wrong ? this seems like a transposition
```
**EN:** Defines function `swizzle_block_shape(self, block_shape)` for this module. The body mainly branches on runtime conditions.

**CN:** 定义函数 `swizzle_block_shape(self, block_shape)`，供本模块使用. 主体主要根据运行时条件分支.

### Block 12 — Lines 27-40 (swizzle_block_shape)
```python
27|         if self.mx_axis == -1:
28|             *head, N, K = block_shape
29|             assert N % 32 == 0, N
30|             return [*head, N // 32, K * 32]
31|         else:
32|             assert self.mx_axis == -2
33|             *head, K, N = block_shape
34|             assert N % 32 == 0, N
35|             return [*head, K * 32, N // 32]
36| 
37| 
38| # ------------------- Hopper MX Scale Layout Transformation -------------------
39| 
40| 
```
**EN:** Checks `self.mx_axis == -1`. The true branch mainly prepares intermediate values; checks invariants, while the else branch checks invariants; prepares intermediate values.

**CN:** 检查 `self.mx_axis == -1`. 真分支主要准备中间值; 检查不变量；而 else 分支检查不变量; 准备中间值.

### Block 13 — Lines 41-42 (HopperMXScaleLayoutTransformation)
```python
41| @dataclass(frozen=True)
42| class HopperMXScaleLayoutTransformation(LayoutTransformation):
```
**EN:** Defines class `HopperMXScaleLayoutTransformation` inheriting from `LayoutTransformation` with decorators `dataclass(frozen=True)` to organize related behavior. Key fields include `mx_axis`, `num_warps`. Main methods are `__post_init__`, `_maybe_mT`, `swizzle_data`, `unswizzle_data`.

**CN:** 定义类 `HopperMXScaleLayoutTransformation`，继承自 `LayoutTransformation`，带有装饰器 `dataclass(frozen=True)`，用于组织相关行为。关键字段包括 `mx_axis`, `num_warps`。主要方法有 `__post_init__`, `_maybe_mT`, `swizzle_data`, `unswizzle_data`.

### Block 14 — Lines 43-43 (HopperMXScaleLayoutTransformation)
```python
43|     mx_axis: int
```
**EN:** Annotated assignment stores `mx_axis` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `mx_axis` and 声明该带注解的字段.

### Block 15 — Lines 44-45 (HopperMXScaleLayoutTransformation)
```python
44|     num_warps: int
45| 
```
**EN:** Annotated assignment stores `num_warps` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `num_warps` and 声明该带注解的字段.

### Block 16 — Lines 46-46 (__post_init__)
```python
46|     def __post_init__(self):
```
**EN:** Defines function `__post_init__(self)` for this module. The body mainly prepares intermediate values; branches on runtime conditions; invokes `object.__setattr__`. It uses calls such as `object.__setattr__`, `len` to implement its workflow.

**CN:** 定义函数 `__post_init__(self)`，供本模块使用. 主体主要准备中间值; 根据运行时条件分支; invokes `object.__setattr__`. 其中会调用 `object.__setattr__`, `len` 来实现其工作流程.

### Block 17 — Lines 47-47 (__post_init__)
```python
47|         *leading_shape, M, K = self.shape
```
**EN:** Assigns `leading_shape`, `M`, `K` and references `self.shape`.

**CN:** 将 `leading_shape`, `M`, `K` and 引用 `self.shape`.

### Block 18 — Lines 48-49 (__post_init__)
```python
48|         if self.mx_axis < 0:
49|             object.__setattr__(self, "mx_axis", self.mx_axis + len(self.shape))
```
**EN:** Checks `self.mx_axis < 0`. The true branch mainly invokes `object.__setattr__`.

**CN:** 检查 `self.mx_axis < 0`. 真分支主要invokes `object.__setattr__`.

### Block 19 — Lines 50-50 (__post_init__)
```python
50|         object.__setattr__(self, "leading_shape", leading_shape)
```
**EN:** Calls `object.__setattr__` for side effects, registration, or validation.

**CN:** 调用 `object.__setattr__` ，用于副作用、注册或校验。

### Block 20 — Lines 51-51 (__post_init__)
```python
51|         object.__setattr__(self, "M", M)
```
**EN:** Calls `object.__setattr__` for side effects, registration, or validation.

**CN:** 调用 `object.__setattr__` ，用于副作用、注册或校验。

### Block 21 — Lines 52-53 (__post_init__)
```python
52|         object.__setattr__(self, "K", K)
53| 
```
**EN:** Calls `object.__setattr__` for side effects, registration, or validation.

**CN:** 调用 `object.__setattr__` ，用于副作用、注册或校验。

### Block 22 — Lines 54-54 (_maybe_mT)
```python
54|     def _maybe_mT(self, data):
```
**EN:** Defines function `_maybe_mT(self, data)` for this module. The body mainly branches on runtime conditions; returns the computed result. It uses calls such as `len`, `data.contiguous` to implement its workflow.

**CN:** 定义函数 `_maybe_mT(self, data)`，供本模块使用. 主体主要根据运行时条件分支; 返回计算结果. 其中会调用 `len`, `data.contiguous` 来实现其工作流程.

### Block 23 — Lines 55-56 (_maybe_mT)
```python
55|         if self.mx_axis == len(self.leading_shape):
56|             return data.contiguous().mT
```
**EN:** Checks `self.mx_axis == len(self.leading_shape)`. The true branch mainly returns the computed result.

**CN:** 检查 `self.mx_axis == len(self.leading_shape)`. 真分支主要返回计算结果.

### Block 24 — Lines 57-58 (_maybe_mT)
```python
57|         return data
58| 
```
**EN:** Returns `data`.

**CN:** 返回 `data`.

### Block 25 — Lines 59-59 (swizzle_data)
```python
59|     def swizzle_data(self, data):
```
**EN:** Defines function `swizzle_data(self, data)` for this module. The body mainly checks invariants; prepares intermediate values; prepares intermediate values. It uses calls such as `self._maybe_mT(data).contiguous`, `data.numel`, `data.is_contiguous`, `len`, `data.reshape` to implement its workflow.

**CN:** 定义函数 `swizzle_data(self, data)`，供本模块使用. 主体主要检查不变量; 准备中间值; 准备中间值. 其中会调用 `self._maybe_mT(data).contiguous`, `data.numel`, `data.is_contiguous`, `len`, `data.reshape` 来实现其工作流程.

### Block 26 — Lines 60-60 (swizzle_data)
```python
60|         assert data.shape == (*self.leading_shape, self.M, self.K)
```
**EN:** Asserts `data.shape == (*self.leading_shape, self.M, self.K)` to enforce invariants.

**CN:** 断言 `data.shape == (*self.leading_shape, self.M, self.K)` 以确保不变量成立。

### Block 27 — Lines 61-61 (swizzle_data)
```python
61|         data = self._maybe_mT(data).contiguous()
```
**EN:** Assigns `data` and calls `self._maybe_mT(data).contiguous`.

**CN:** 将 `data`，并调用 `self._maybe_mT(data).contiguous`.

### Block 28 — Lines 62-62 (swizzle_data)
```python
62|         *batch, M, K = data.shape
```
**EN:** Assigns `batch`, `M`, `K` and references `data.shape`.

**CN:** 将 `batch`, `M`, `K` and 引用 `data.shape`.

### Block 29 — Lines 63-63 (swizzle_data)
```python
63|         SWIZZLE_ALIGN_M = 2 * self.num_warps * 2 * 8
```
**EN:** Assigns `SWIZZLE_ALIGN_M` and evaluates `2 * self.num_warps * 2 * 8`.

**CN:** 将 `SWIZZLE_ALIGN_M` and 计算 `2 * self.num_warps * 2 * 8`.

### Block 30 — Lines 64-64 (swizzle_data)
```python
64|         SWIZZLE_ALIGN_K = 2
```
**EN:** Assigns `SWIZZLE_ALIGN_K` and stores constant `2`.

**CN:** 将 `SWIZZLE_ALIGN_K` and 保存常量 `2`.

### Block 31 — Lines 65-65 (swizzle_data)
```python
65|         pad_m = (SWIZZLE_ALIGN_M - (M % SWIZZLE_ALIGN_M)) % SWIZZLE_ALIGN_M
```
**EN:** Assigns `pad_m` and evaluates `(SWIZZLE_ALIGN_M - M % SWIZZLE_ALIGN_M) % SWIZZLE_ALIGN_M`.

**CN:** 将 `pad_m` and 计算 `(SWIZZLE_ALIGN_M - M % SWIZZLE_ALIGN_M) % SWIZZLE_ALIGN_M`.

### Block 32 — Lines 66-66 (swizzle_data)
```python
66|         pad_k = (SWIZZLE_ALIGN_K - (K % SWIZZLE_ALIGN_K)) % SWIZZLE_ALIGN_K
```
**EN:** Assigns `pad_k` and evaluates `(SWIZZLE_ALIGN_K - K % SWIZZLE_ALIGN_K) % SWIZZLE_ALIGN_K`.

**CN:** 将 `pad_k` and 计算 `(SWIZZLE_ALIGN_K - K % SWIZZLE_ALIGN_K) % SWIZZLE_ALIGN_K`.

### Block 33 — Lines 67-68 (swizzle_data)
```python
67|         if data.numel():
68|             data = torch.nn.functional.pad(data, (0, pad_k, 0, pad_m))
```
**EN:** Checks `data.numel()`. The true branch mainly prepares intermediate values.

**CN:** 检查 `data.numel()`. 真分支主要准备中间值.

### Block 34 — Lines 69-69 (swizzle_data)
```python
69|         M += pad_m
```
**EN:** Updates `M` with operator `Add` using `pad_m`.

**CN:** 更新 `M`，使用运算符 `Add`，并使用 `pad_m`.

### Block 35 — Lines 70-70 (swizzle_data)
```python
70|         K += pad_k
```
**EN:** Updates `K` with operator `Add` using `pad_k`.

**CN:** 更新 `K`，使用运算符 `Add`，并使用 `pad_k`.

### Block 36 — Lines 71-71 (swizzle_data)
```python
71|         assert data.is_contiguous()
```
**EN:** Asserts `data.is_contiguous()` to enforce invariants.

**CN:** 断言 `data.is_contiguous()` 以确保不变量成立。

### Block 37 — Lines 72-74 (swizzle_data)
```python
72|         assert M % (
73|             2 * self.num_warps * 2 *
74|             8) == 0 and K % 2 == 0, f"Input tensor must have a subtile of shape (..., {2 * self.num_warps * 2 * 8}, 2)"
```
**EN:** Asserts `M % (2 * self.num_warps * 2 * 8) == 0 and K % 2 == 0` to enforce invariants.

**CN:** 断言 `M % (2 * self.num_warps * 2 * 8) == 0 and K % 2 == 0` 以确保不变量成立。

### Block 38 — Lines 75-75 (swizzle_data)
```python
75|         b = len(batch)
```
**EN:** Assigns `b` and calls `len`.

**CN:** 将 `b`，并调用 `len`.

### Block 39 — Lines 76-76 (swizzle_data)
```python
76|         data = data.reshape(*batch, M // (2 * self.num_warps * 2 * 8), 2, self.num_warps, 2, 8, K // 2, 2)
```
**EN:** Assigns `data` and calls `data.reshape`.

**CN:** 将 `data`，并调用 `data.reshape`.

### Block 40 — Lines 77-77 (swizzle_data)
```python
77|         perm = [0, 2, 5, 1, 4, 6, 3]
```
**EN:** Assigns `perm` and builds a list.

**CN:** 将 `perm` and 构造一个列表.

### Block 41 — Lines 78-78 (swizzle_data)
```python
78|         perm = list(range(b)) + [b + p for p in perm]
```
**EN:** Assigns `perm` and evaluates `list(range(b)) + [b + p for p in perm]`.

**CN:** 将 `perm` and 计算 `list(range(b)) + [b + p for p in perm]`.

### Block 42 — Lines 79-79 (swizzle_data)
```python
79|         data = data.permute(*perm)
```
**EN:** Assigns `data` and calls `data.permute`.

**CN:** 将 `data`，并调用 `data.permute`.

### Block 43 — Lines 80-80 (swizzle_data)
```python
80|         data = data.flatten(-5, -1)
```
**EN:** Assigns `data` and calls `data.flatten`.

**CN:** 将 `data`，并调用 `data.flatten`.

### Block 44 — Lines 81-81 (swizzle_data)
```python
81|         data = data.flatten(-3, -2)
```
**EN:** Assigns `data` and calls `data.flatten`.

**CN:** 将 `data`，并调用 `data.flatten`.

### Block 45 — Lines 82-82 (swizzle_data)
```python
82|         assert data.shape[-2] == M // 32
```
**EN:** Asserts `data.shape[-2] == M // 32` to enforce invariants.

**CN:** 断言 `data.shape[-2] == M // 32` 以确保不变量成立。

### Block 46 — Lines 83-83 (swizzle_data)
```python
83|         assert data.shape[-1] == K * 32
```
**EN:** Asserts `data.shape[-1] == K * 32` to enforce invariants.

**CN:** 断言 `data.shape[-1] == K * 32` 以确保不变量成立。

### Block 47 — Lines 84-84 (swizzle_data)
```python
84|         data = self._maybe_mT(data)
```
**EN:** Assigns `data` and calls `self._maybe_mT`.

**CN:** 将 `data`，并调用 `self._maybe_mT`.

### Block 48 — Lines 85-86 (swizzle_data)
```python
85|         return data
86| 
```
**EN:** Returns `data`.

**CN:** 返回 `data`.

### Block 49 — Lines 87-87 (unswizzle_data)
```python
87|     def unswizzle_data(self, data):
```
**EN:** Defines function `unswizzle_data(self, data)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `self._maybe_mT`, `len`, `data.reshape`, `data.permute`, `data.contiguous` to implement its workflow.

**CN:** 定义函数 `unswizzle_data(self, data)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `self._maybe_mT`, `len`, `data.reshape`, `data.permute`, `data.contiguous` 来实现其工作流程.

### Block 50 — Lines 88-88 (unswizzle_data)
```python
88|         data = self._maybe_mT(data)
```
**EN:** Assigns `data` and calls `self._maybe_mT`.

**CN:** 将 `data`，并调用 `self._maybe_mT`.

### Block 51 — Lines 89-89 (unswizzle_data)
```python
89|         *batch, M, K = data.shape
```
**EN:** Assigns `batch`, `M`, `K` and references `data.shape`.

**CN:** 将 `batch`, `M`, `K` and 引用 `data.shape`.

### Block 52 — Lines 90-90 (unswizzle_data)
```python
90|         b = len(batch)
```
**EN:** Assigns `b` and calls `len`.

**CN:** 将 `b`，并调用 `len`.

### Block 53 — Lines 91-91 (unswizzle_data)
```python
91|         data = data.reshape(*batch, M // self.num_warps, self.num_warps, K // 64, 2, 8, 2, 2)
```
**EN:** Assigns `data` and calls `data.reshape`.

**CN:** 将 `data`，并调用 `data.reshape`.

### Block 54 — Lines 92-92 (unswizzle_data)
```python
92|         perm = [0, 3, 1, 6, 4, 2, 5]
```
**EN:** Assigns `perm` and builds a list.

**CN:** 将 `perm` and 构造一个列表.

### Block 55 — Lines 93-93 (unswizzle_data)
```python
93|         perm = list(range(b)) + [b + p for p in perm]
```
**EN:** Assigns `perm` and evaluates `list(range(b)) + [b + p for p in perm]`.

**CN:** 将 `perm` and 计算 `list(range(b)) + [b + p for p in perm]`.

### Block 56 — Lines 94-94 (unswizzle_data)
```python
94|         data = data.permute(*perm)
```
**EN:** Assigns `data` and calls `data.permute`.

**CN:** 将 `data`，并调用 `data.permute`.

### Block 57 — Lines 95-95 (unswizzle_data)
```python
95|         data = data.reshape(*batch, M * 32, K // 32)
```
**EN:** Assigns `data` and calls `data.reshape`.

**CN:** 将 `data`，并调用 `data.reshape`.

### Block 58 — Lines 96-96 (unswizzle_data)
```python
96|         data = self._maybe_mT(data)
```
**EN:** Assigns `data` and calls `self._maybe_mT`.

**CN:** 将 `data`，并调用 `self._maybe_mT`.

### Block 59 — Lines 97-97 (unswizzle_data)
```python
97|         data = data[..., :self.M, :self.K]
```
**EN:** Assigns `data` and evaluates `data[..., :self.M, :self.K]`.

**CN:** 将 `data` and 计算 `data[..., :self.M, :self.K]`.

### Block 60 — Lines 98-98 (unswizzle_data)
```python
98|         data = data.contiguous()
```
**EN:** Assigns `data` and calls `data.contiguous`.

**CN:** 将 `data`，并调用 `data.contiguous`.

### Block 61 — Lines 99-101 (unswizzle_data)
```python
 99|         return data
100| 
101| 
```
**EN:** Returns `data`.

**CN:** 返回 `data`.

### Block 62 — Lines 102-103 (unswizzle_mxfp4_scale_hopper)
```python
102| @triton.jit
103| def unswizzle_mxfp4_scale_hopper(x, mx_axis: tl.constexpr, num_warps: tl.constexpr):
```
**EN:** Defines function `unswizzle_mxfp4_scale_hopper(x, mx_axis, num_warps)` with decorators `triton.jit` for this module. The body mainly branches on runtime conditions; invokes `tl.static_assert`; prepares intermediate values. It uses calls such as `tl.static_assert`, `x.reshape`, `x.trans`, `len` to implement its workflow.

**CN:** 定义函数 `unswizzle_mxfp4_scale_hopper(x, mx_axis, num_warps)`，带有装饰器 `triton.jit`，供本模块使用. 主体主要根据运行时条件分支; invokes `tl.static_assert`; 准备中间值. 其中会调用 `tl.static_assert`, `x.reshape`, `x.trans`, `len` 来实现其工作流程.

### Block 63 — Lines 104-106 (unswizzle_mxfp4_scale_hopper)
```python
104|     """
105|     Triton inverse of swizzle_mxfp4_scale_hopper
106|     """
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 64 — Lines 107-108 (unswizzle_mxfp4_scale_hopper)
```python
107|     if mx_axis is not None and mx_axis < 0:
108|         mx_axis += len(x.shape)
```
**EN:** Checks `mx_axis is not None and mx_axis < 0`..

**CN:** 检查 `mx_axis is not None and mx_axis < 0`..

### Block 65 — Lines 109-110 (unswizzle_mxfp4_scale_hopper)
```python
109|     tl.static_assert(len(x.shape) == 2, "NYI")
110|     # implementation assumes mxfp data is packed along the last dimension
```
**EN:** Calls `tl.static_assert` for side effects, registration, or validation.

**CN:** 调用 `tl.static_assert` ，用于副作用、注册或校验。

### Block 66 — Lines 111-111 (unswizzle_mxfp4_scale_hopper)
```python
111|     x = x.trans() if mx_axis == 0 else x
```
**EN:** Assigns `x` and uses conditional expression `x.trans() if mx_axis == 0 else x`.

**CN:** 将 `x` and 使用条件表达式 `x.trans() if mx_axis == 0 else x`.

### Block 67 — Lines 112-112 (unswizzle_mxfp4_scale_hopper)
```python
112|     M: tl.constexpr = x.shape[0]
```
**EN:** Annotated assignment stores `M` and evaluates `x.shape[0]`.

**CN:** 带类型注解的赋值保存 `M` and 计算 `x.shape[0]`.

### Block 68 — Lines 113-113 (unswizzle_mxfp4_scale_hopper)
```python
113|     K: tl.constexpr = x.shape[1]
```
**EN:** Annotated assignment stores `K` and evaluates `x.shape[1]`.

**CN:** 带类型注解的赋值保存 `K` and 计算 `x.shape[1]`.

### Block 69 — Lines 114-114 (unswizzle_mxfp4_scale_hopper)
```python
114|     tl.static_assert(M % num_warps == 0, f"M must be divisible by {num_warps}. Got {M}")
```
**EN:** Calls `tl.static_assert` for side effects, registration, or validation.

**CN:** 调用 `tl.static_assert` ，用于副作用、注册或校验。

### Block 70 — Lines 115-115 (unswizzle_mxfp4_scale_hopper)
```python
115|     tl.static_assert(K % 64 == 0, f"K must be divisible by 64. Got {K}")
```
**EN:** Calls `tl.static_assert` for side effects, registration, or validation.

**CN:** 调用 `tl.static_assert` ，用于副作用、注册或校验。

### Block 71 — Lines 116-116 (unswizzle_mxfp4_scale_hopper)
```python
116|     x = x.reshape(M // num_warps, num_warps, K // 64, 2, 8, 2, 2)
```
**EN:** Assigns `x` and calls `x.reshape`.

**CN:** 将 `x`，并调用 `x.reshape`.

### Block 72 — Lines 117-117 (unswizzle_mxfp4_scale_hopper)
```python
117|     x = x.trans(0, 3, 1, 6, 4, 2, 5)
```
**EN:** Assigns `x` and calls `x.trans`.

**CN:** 将 `x`，并调用 `x.trans`.

### Block 73 — Lines 118-119 (unswizzle_mxfp4_scale_hopper)
```python
118|     x = x.reshape(M * 32, K // 32)
119|     # implementation assumed mxfp data is packed along the last dimension
```
**EN:** Assigns `x` and calls `x.reshape`.

**CN:** 将 `x`，并调用 `x.reshape`.

### Block 74 — Lines 120-120 (unswizzle_mxfp4_scale_hopper)
```python
120|     x = x.trans() if mx_axis == 0 else x
```
**EN:** Assigns `x` and uses conditional expression `x.trans() if mx_axis == 0 else x`.

**CN:** 将 `x` and 使用条件表达式 `x.trans() if mx_axis == 0 else x`.

### Block 75 — Lines 121-121 (unswizzle_mxfp4_scale_hopper)
```python
121|     return x
```
**EN:** Returns `x`.

**CN:** 返回 `x`.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `HopperMXScaleLayout`, `HopperMXScaleLayoutTransformation`, `unswizzle_mxfp4_scale_hopper`.
  **CN:** 主要符号：`HopperMXScaleLayout`, `HopperMXScaleLayoutTransformation`, `unswizzle_mxfp4_scale_hopper`。
- **EN:** PyTorch tensors are the main data container and host-side orchestration layer.
  **CN:** PyTorch 张量是主要的数据容器和主机端编排层。
- **EN:** Triton is used to JIT-compile or launch GPU kernels and to express tile-level execution details.
  **CN:** Triton 用于 JIT 编译或启动 GPU 内核，并表达分块级执行细节。
- **EN:** Mixed-precision MXFP/NVFP formats and scale tensors are part of the numeric pipeline.
  **CN:** 混合精度 MXFP/NVFP 格式及其缩放张量是数值流程的一部分。
- **EN:** Custom layout objects describe how logical tensors are packed or swizzled in memory.
  **CN:** 自定义 layout 对象描述逻辑张量如何在内存中打包或重排。

## Dependencies / 依赖关系
- **EN:** External modules: `dataclasses (dataclass)`, `torch`, `triton`, `triton.language`.
  **CN:** 外部模块：`dataclasses (dataclass)`, `torch`, `triton`, `triton.language`。
- **EN:** Internal modules: `.base (Layout, LayoutTransformation)`.
  **CN:** 内部模块：`.base (Layout, LayoutTransformation)`。
