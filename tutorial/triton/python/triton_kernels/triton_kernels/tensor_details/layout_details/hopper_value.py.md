# hopper_value.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/triton_kernels/tensor_details/layout_details/hopper_value.py`
- **Purpose / 用途:** Implementation module for hopper value; it defines the runtime logic, helper types, and GPU-facing utilities used by the Triton kernels package via symbols HopperMXValueLayout, HopperMXValueLayoutTransformation, right_shift_unsigned, _compress_fp4, _compress_fourth. / 用于 hopper value 的实现模块；它定义了 Triton kernels 包使用的运行时逻辑、辅助类型以及面向 GPU 的工具；核心符号包括 HopperMXValueLayout、HopperMXValueLayoutTransformation、right_shift_unsigned、_compress_fp4、_compress_fourth。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-11 (module)
```python
 1| import torch
 2| import triton
 3| import triton.language as tl
 4| from dataclasses import dataclass
 5| from .base import Layout, LayoutTransformation
 6| from triton_kernels.numerics_details.mxfp_details._downcast_to_mxfp import MXFP_BLOCK_SIZE
 7| from triton_kernels.target_info import cuda_capability_geq
 8| from .torch_utils import repack
 9| 
10| 
11| # ------------------- Hopper MX Value Layout -------------------
```
**EN:** This block imports `torch`, `triton`, `triton.language`, `dataclasses (dataclass)`, `.base (Layout, LayoutTransformation)`, `triton_kernels.numerics_details.mxfp_details._downcast_to_mxfp (MXFP_BLOCK_SIZE)`, `triton_kernels.target_info (cuda_capability_geq)`, `.torch_utils (repack)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `torch`, `triton`, `triton.language`, `dataclasses (dataclass)`, `.base (Layout, LayoutTransformation)`, `triton_kernels.numerics_details.mxfp_details._downcast_to_mxfp (MXFP_BLOCK_SIZE)`, `triton_kernels.target_info (cuda_capability_geq)`, `.torch_utils (repack)` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 12-13 (HopperMXValueLayout)
```python
12| @dataclass(frozen=True)
13| class HopperMXValueLayout(Layout):
```
**EN:** Defines class `HopperMXValueLayout` inheriting from `Layout` with decorators `dataclass(frozen=True)` to organize related behavior. Key fields include `mx_axis`, `mma_version`. Main methods are `__post_init__`, `name`, `swizzle_block_shape`, `make_transformation`.

**CN:** 定义类 `HopperMXValueLayout`，继承自 `Layout`，带有装饰器 `dataclass(frozen=True)`，用于组织相关行为。关键字段包括 `mx_axis`, `mma_version`。主要方法有 `__post_init__`, `name`, `swizzle_block_shape`, `make_transformation`.

### Block 3 — Lines 14-14 (HopperMXValueLayout)
```python
14|     mx_axis: int
```
**EN:** Annotated assignment stores `mx_axis` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `mx_axis` and 声明该带注解的字段.

### Block 4 — Lines 15-16 (HopperMXValueLayout)
```python
15|     mma_version: int
16| 
```
**EN:** Annotated assignment stores `mma_version` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `mma_version` and 声明该带注解的字段.

### Block 5 — Lines 17-17 (__post_init__)
```python
17|     def __post_init__(self):
```
**EN:** Defines function `__post_init__(self)` for this module. The body mainly checks invariants; checks invariants.

**CN:** 定义函数 `__post_init__(self)`，供本模块使用. 主体主要检查不变量; 检查不变量.

### Block 6 — Lines 18-18 (__post_init__)
```python
18|         assert self.mx_axis in [-1, -2]
```
**EN:** Asserts `self.mx_axis in [-1, -2]` to enforce invariants.

**CN:** 断言 `self.mx_axis in [-1, -2]` 以确保不变量成立。

### Block 7 — Lines 19-20 (__post_init__)
```python
19|         assert self.mma_version in [2, 3]
20| 
```
**EN:** Asserts `self.mma_version in [2, 3]` to enforce invariants.

**CN:** 断言 `self.mma_version in [2, 3]` 以确保不变量成立。

### Block 8 — Lines 21-22 (name)
```python
21|     @property
22|     def name(self):
```
**EN:** Defines function `name(self)` with decorators `property` for this module. The body mainly returns the computed result.

**CN:** 定义函数 `name(self)`，带有装饰器 `property`，供本模块使用. 主体主要返回计算结果.

### Block 9 — Lines 23-24 (name)
```python
23|         return "HOPPER_VALUE"
24| 
```
**EN:** Returns `'HOPPER_VALUE'`.

**CN:** 返回 `'HOPPER_VALUE'`.

### Block 10 — Lines 25-25 (swizzle_block_shape)
```python
25|     def swizzle_block_shape(self, block_shape):
```
**EN:** Defines function `swizzle_block_shape(self, block_shape)` for this module. The body mainly branches on runtime conditions.

**CN:** 定义函数 `swizzle_block_shape(self, block_shape)`，供本模块使用. 主体主要根据运行时条件分支.

### Block 11 — Lines 26-35 (swizzle_block_shape)
```python
26|         if self.mx_axis == -1:
27|             *head, N, K = block_shape
28|             assert N % 4 == 0
29|             return [*head, N // 4, K * 4]
30|         else:
31|             assert self.mx_axis == -2
32|             *head, K, N = block_shape
33|             assert N % 4 == 0
34|             return [*head, K * 4, N // 4]
35| 
```
**EN:** Checks `self.mx_axis == -1`. The true branch mainly prepares intermediate values; checks invariants, while the else branch checks invariants; prepares intermediate values.

**CN:** 检查 `self.mx_axis == -1`. 真分支主要准备中间值; 检查不变量；而 else 分支检查不变量; 准备中间值.

### Block 12 — Lines 36-36 (make_transformation)
```python
36|     def make_transformation(self, shape: list[int], is_fp4) -> LayoutTransformation:
```
**EN:** Defines function `make_transformation(self, shape, is_fp4)` for this module. The body mainly returns the computed result. It uses calls such as `HopperMXValueLayoutTransformation` to implement its workflow.

**CN:** 定义函数 `make_transformation(self, shape, is_fp4)`，供本模块使用. 主体主要返回计算结果. 其中会调用 `HopperMXValueLayoutTransformation` 来实现其工作流程.

### Block 13 — Lines 37-42 (make_transformation)
```python
37|         return HopperMXValueLayoutTransformation(shape, is_fp4, self.mx_axis, self.mma_version)
38| 
39| 
40| # ------------------- Hopper MX Value Layout Transformation -------------------
41| 
42| 
```
**EN:** Returns `HopperMXValueLayoutTransformation(shape, is_fp4, self.mx_axis, self.mma_version)`.

**CN:** 返回 `HopperMXValueLayoutTransformation(shape, is_fp4, self.mx_axis, self.mma_version)`.

### Block 14 — Lines 43-44 (HopperMXValueLayoutTransformation)
```python
43| @dataclass(frozen=True)
44| class HopperMXValueLayoutTransformation(LayoutTransformation):
```
**EN:** Defines class `HopperMXValueLayoutTransformation` inheriting from `LayoutTransformation` with decorators `dataclass(frozen=True)` to organize related behavior. Key fields include `mx_axis`, `mma_version`. Main methods are `__post_init__`, `_maybe_mT`, `swizzle_data`, `unswizzle_data`.

**CN:** 定义类 `HopperMXValueLayoutTransformation`，继承自 `LayoutTransformation`，带有装饰器 `dataclass(frozen=True)`，用于组织相关行为。关键字段包括 `mx_axis`, `mma_version`。主要方法有 `__post_init__`, `_maybe_mT`, `swizzle_data`, `unswizzle_data`.

### Block 15 — Lines 45-45 (HopperMXValueLayoutTransformation)
```python
45|     mx_axis: int
```
**EN:** Annotated assignment stores `mx_axis` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `mx_axis` and 声明该带注解的字段.

### Block 16 — Lines 46-47 (HopperMXValueLayoutTransformation)
```python
46|     mma_version: int
47| 
```
**EN:** Annotated assignment stores `mma_version` and declares the annotated field.

**CN:** 带类型注解的赋值保存 `mma_version` and 声明该带注解的字段.

### Block 17 — Lines 48-48 (__post_init__)
```python
48|     def __post_init__(self):
```
**EN:** Defines function `__post_init__(self)` for this module. The body mainly prepares intermediate values; branches on runtime conditions; invokes `object.__setattr__`. It uses calls such as `object.__setattr__`, `len` to implement its workflow.

**CN:** 定义函数 `__post_init__(self)`，供本模块使用. 主体主要准备中间值; 根据运行时条件分支; invokes `object.__setattr__`. 其中会调用 `object.__setattr__`, `len` 来实现其工作流程.

### Block 18 — Lines 49-49 (__post_init__)
```python
49|         *leading_shape, K, N, = self.shape
```
**EN:** Assigns `leading_shape`, `K`, `N` and references `self.shape`.

**CN:** 将 `leading_shape`, `K`, `N` and 引用 `self.shape`.

### Block 19 — Lines 50-51 (__post_init__)
```python
50|         if self.mx_axis < 0:
51|             object.__setattr__(self, "mx_axis", self.mx_axis + len(self.shape))
```
**EN:** Checks `self.mx_axis < 0`. The true branch mainly invokes `object.__setattr__`.

**CN:** 检查 `self.mx_axis < 0`. 真分支主要invokes `object.__setattr__`.

### Block 20 — Lines 52-52 (__post_init__)
```python
52|         object.__setattr__(self, "leading_shape", leading_shape)
```
**EN:** Calls `object.__setattr__` for side effects, registration, or validation.

**CN:** 调用 `object.__setattr__` ，用于副作用、注册或校验。

### Block 21 — Lines 53-53 (__post_init__)
```python
53|         object.__setattr__(self, "K", K)
```
**EN:** Calls `object.__setattr__` for side effects, registration, or validation.

**CN:** 调用 `object.__setattr__` ，用于副作用、注册或校验。

### Block 22 — Lines 54-55 (__post_init__)
```python
54|         object.__setattr__(self, "N", N)
55| 
```
**EN:** Calls `object.__setattr__` for side effects, registration, or validation.

**CN:** 调用 `object.__setattr__` ，用于副作用、注册或校验。

### Block 23 — Lines 56-56 (_maybe_mT)
```python
56|     def _maybe_mT(self, data):
```
**EN:** Defines function `_maybe_mT(self, data)` for this module. The body mainly branches on runtime conditions; returns the computed result. It uses calls such as `len` to implement its workflow.

**CN:** 定义函数 `_maybe_mT(self, data)`，供本模块使用. 主体主要根据运行时条件分支; 返回计算结果. 其中会调用 `len` 来实现其工作流程.

### Block 24 — Lines 57-58 (_maybe_mT)
```python
57|         if self.mx_axis == len(self.leading_shape):
58|             return data.mT
```
**EN:** Checks `self.mx_axis == len(self.leading_shape)`. The true branch mainly returns the computed result.

**CN:** 检查 `self.mx_axis == len(self.leading_shape)`. 真分支主要返回计算结果.

### Block 25 — Lines 59-60 (_maybe_mT)
```python
59|         return data
60| 
```
**EN:** Returns `data`.

**CN:** 返回 `data`.

### Block 26 — Lines 61-61 (swizzle_data)
```python
61|     def swizzle_data(self, data):
```
**EN:** Defines function `swizzle_data(self, data)` for this module. The body mainly prepares intermediate values; prepares intermediate values; checks invariants. It uses calls such as `repack`, `torch.nn.functional.pad`, `self._maybe_mT`, `list`, `enumerate` to implement its workflow.

**CN:** 定义函数 `swizzle_data(self, data)`，供本模块使用. 主体主要准备中间值; 准备中间值; 检查不变量. 其中会调用 `repack`, `torch.nn.functional.pad`, `self._maybe_mT`, `list`, `enumerate` 来实现其工作流程.

### Block 27 — Lines 62-81 (swizzle_data)
```python
62|         """
63|         Given a uint8 tensor of shape (*, M, K), returns a tensor of shape
64|         (*, M // 4, K * 4) such that:
65| 
66|         1) Groups contiguously all the elements owned by the same thread of 4
67|         mma tiles along the K axis. The following animation shows a similar
68|         grouping for 2 tiles along M and 2 tiles along K rather than 4 along K
69|         as done here:
70|         https://neuralmagic.com/wp-content/uploads/2024/10/animation_4.gif
71| 
72|         2) Moves the elements belonging to thread 4-7 to be contiguous with those
73|         from thread 0-3. This is done to get a full cache line when loading them
74|         from HBM.
75| 
76|         mx_axis selects the lhs or rhs of the matmul.
77| 
78|         WARNING: Assumes that the matmul will be done in bf16 or fp16!
79|         Implementing it for fp8 is as easy as making the tile size (8, 8)
80|         """
81|         # re-pack as column-major
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 28 — Lines 82-82 (swizzle_data)
```python
82|         data = repack(data, -1, self.mx_axis, self.is_fp4)
```
**EN:** Assigns `data` and calls `repack`.

**CN:** 将 `data`，并调用 `repack`.

### Block 29 — Lines 83-83 (swizzle_data)
```python
83|         batch = data.ndim - 2
```
**EN:** Assigns `batch` and evaluates `data.ndim - 2`.

**CN:** 将 `batch` and 计算 `data.ndim - 2`.

### Block 30 — Lines 84-84 (swizzle_data)
```python
84|         assert batch >= 0
```
**EN:** Asserts `batch >= 0` to enforce invariants.

**CN:** 断言 `batch >= 0` 以确保不变量成立。

### Block 31 — Lines 85-86 (swizzle_data)
```python
85|         assert self.mma_version in (2, 3)
86|         # Pre-pad both matrix dims to multiples of 64
```
**EN:** Asserts `self.mma_version in (2, 3)` to enforce invariants.

**CN:** 断言 `self.mma_version in (2, 3)` 以确保不变量成立。

### Block 32 — Lines 87-87 (swizzle_data)
```python
87|         *_, M_in, K_in = data.shape
```
**EN:** Assigns `_`, `M_in`, `K_in` and references `data.shape`.

**CN:** 将 `_`, `M_in`, `K_in` and 引用 `data.shape`.

### Block 33 — Lines 88-88 (swizzle_data)
```python
88|         SWIZZLE_ALIGN_M = 64
```
**EN:** Assigns `SWIZZLE_ALIGN_M` and stores constant `64`.

**CN:** 将 `SWIZZLE_ALIGN_M` and 保存常量 `64`.

### Block 34 — Lines 89-89 (swizzle_data)
```python
89|         SWIZZLE_ALIGN_K = 64
```
**EN:** Assigns `SWIZZLE_ALIGN_K` and stores constant `64`.

**CN:** 将 `SWIZZLE_ALIGN_K` and 保存常量 `64`.

### Block 35 — Lines 90-90 (swizzle_data)
```python
90|         pad_m = (SWIZZLE_ALIGN_M - (M_in % SWIZZLE_ALIGN_M)) % SWIZZLE_ALIGN_M
```
**EN:** Assigns `pad_m` and evaluates `(SWIZZLE_ALIGN_M - M_in % SWIZZLE_ALIGN_M) % SWIZZLE_ALIGN_M`.

**CN:** 将 `pad_m` and 计算 `(SWIZZLE_ALIGN_M - M_in % SWIZZLE_ALIGN_M) % SWIZZLE_ALIGN_M`.

### Block 36 — Lines 91-91 (swizzle_data)
```python
91|         pad_k = (SWIZZLE_ALIGN_K - (K_in % SWIZZLE_ALIGN_K)) % SWIZZLE_ALIGN_K
```
**EN:** Assigns `pad_k` and evaluates `(SWIZZLE_ALIGN_K - K_in % SWIZZLE_ALIGN_K) % SWIZZLE_ALIGN_K`.

**CN:** 将 `pad_k` and 计算 `(SWIZZLE_ALIGN_K - K_in % SWIZZLE_ALIGN_K) % SWIZZLE_ALIGN_K`.

### Block 37 — Lines 92-93 (swizzle_data)
```python
92|         data = torch.nn.functional.pad(data, (0, pad_k, 0, pad_m))
93| 
```
**EN:** Assigns `data` and calls `torch.nn.functional.pad`.

**CN:** 将 `data`，并调用 `torch.nn.functional.pad`.

### Block 38 — Lines 94-94 (swizzle_data)
```python
94|         data = self._maybe_mT(data)
```
**EN:** Assigns `data` and calls `self._maybe_mT`.

**CN:** 将 `data`，并调用 `self._maybe_mT`.

### Block 39 — Lines 95-98 (swizzle_data)
```python
95|         init_shape = data.shape
96| 
97|         # We are loading 8 bf16 elements per thread to use ld.global.v4
98|         # Every u8 represents 2 mxfp4 elements
```
**EN:** Assigns `init_shape` and references `data.shape`.

**CN:** 将 `init_shape` and 引用 `data.shape`.

### Block 40 — Lines 99-101 (swizzle_data)
```python
 99|         u8_kwidth = 8 // 2 if self.mma_version == 2 else 1
100| 
101|         # Pack the 4 // u8_kwidth subtiles of an mma into a u4x8
```
**EN:** Assigns `u8_kwidth` and uses conditional expression `8 // 2 if self.mma_version == 2 else 1`.

**CN:** 将 `u8_kwidth` and 使用条件表达式 `8 // 2 if self.mma_version == 2 else 1`.

### Block 41 — Lines 102-102 (swizzle_data)
```python
102|         contig = (1, u8_kwidth)
```
**EN:** Assigns `contig` and builds a tuple.

**CN:** 将 `contig` and 构造一个元组.

### Block 42 — Lines 103-103 (swizzle_data)
```python
103|         scott_trick = (2, 1)
```
**EN:** Assigns `scott_trick` and builds a tuple.

**CN:** 将 `scott_trick` and 构造一个元组.

### Block 43 — Lines 104-104 (swizzle_data)
```python
104|         threads = (4, 4)
```
**EN:** Assigns `threads` and builds a tuple.

**CN:** 将 `threads` and 构造一个元组.

### Block 44 — Lines 105-105 (swizzle_data)
```python
105|         warp_tile = (2, 2)
```
**EN:** Assigns `warp_tile` and builds a tuple.

**CN:** 将 `warp_tile` and 构造一个元组.

### Block 45 — Lines 106-107 (swizzle_data)
```python
106|         k_tile = (1, 4 // u8_kwidth)
107| 
```
**EN:** Assigns `k_tile` and builds a tuple.

**CN:** 将 `k_tile` and 构造一个元组.

### Block 46 — Lines 108-108 (swizzle_data)
```python
108|         sizes = list(data.shape[:-2])
```
**EN:** Assigns `sizes` and calls `list`.

**CN:** 将 `sizes`，并调用 `list`.

### Block 47 — Lines 109-110 (swizzle_data)
```python
109|         pads = []
110|         # [rest, K, tile, threads] per dimension
```
**EN:** Assigns `pads` and builds a list.

**CN:** 将 `pads` and 构造一个列表.

### Block 48 — Lines 111-118 (swizzle_data)
```python
111|         for i, (a, b, c, s, d) in enumerate(zip(k_tile, warp_tile, threads, scott_trick, contig)):
112|             packed = a * b * c * s * d
113|             size = data.shape[batch + i]
114|             pad = (packed - size % packed) % packed
115|             pads += [(0, pad)]
116|             sizes.append((size + pad) // packed)
117|             sizes += [a, b, c, s, d]
118| 
```
**EN:** Loops over `enumerate(zip(k_tile, warp_tile, threads, scott_trick, co...` with target `(i, (a, b, c, s, d))`. The loop body mainly prepares intermediate values; prepares intermediate values.

**CN:** 遍历 `enumerate(zip(k_tile, warp_tile, threads, scott_trick, co...` ，目标变量为 `(i, (a, b, c, s, d))`. 循环体主要准备中间值; 准备中间值.

### Block 49 — Lines 119-119 (swizzle_data)
```python
119|         pads = tuple(x for t in pads[::-1] for x in t)
```
**EN:** Assigns `pads` and calls `tuple`.

**CN:** 将 `pads`，并调用 `tuple`.

### Block 50 — Lines 120-120 (swizzle_data)
```python
120|         data = torch.nn.functional.pad(data, pads)
```
**EN:** Assigns `data` and calls `torch.nn.functional.pad`.

**CN:** 将 `data`，并调用 `torch.nn.functional.pad`.

### Block 51 — Lines 121-133 (swizzle_data)
```python
121|         init_shape = data.shape
122|         # 0: rest[0]
123|         # 1: k_tile[0]
124|         # 2: warp_tile[0]
125|         # 3: threads[0]
126|         # 4: scott_trick[0]
127|         # 5: contig[0]
128|         # 6: rest[1]
129|         # 7: k_tile[1]
130|         # 8: warp_tile[1]
131|         # 9: threads[1]
132|         # 10: scott_trick[1]
133|         # 11: contig[1]
```
**EN:** Assigns `init_shape` and references `data.shape`.

**CN:** 将 `init_shape` and 引用 `data.shape`.

### Block 52 — Lines 134-135 (swizzle_data)
```python
134|         data = data.view(*sizes)
135|         # Want [rest[0], threads[0], rest[1], scott_trick[0], scott_trick[0], threads[1], contig[1], contig[0], k_tile[1], k_tile[0], warp_tile[1], warp_tile[0]]
```
**EN:** Assigns `data` and calls `data.view`.

**CN:** 将 `data`，并调用 `data.view`.

### Block 53 — Lines 136-136 (swizzle_data)
```python
136|         perm = [0, 3, 6, 10, 4, 9, 7, 1, 8, 2, 5, 11]
```
**EN:** Assigns `perm` and builds a list.

**CN:** 将 `perm` and 构造一个列表.

### Block 54 — Lines 137-137 (swizzle_data)
```python
137|         perm = list(range(batch)) + [batch + p for p in perm]
```
**EN:** Assigns `perm` and evaluates `list(range(batch)) + [batch + p for p in perm]`.

**CN:** 将 `perm` and 计算 `list(range(batch)) + [batch + p for p in perm]`.

### Block 55 — Lines 138-139 (swizzle_data)
```python
138|         data = data.permute(*perm).contiguous()
139|         # These are views
```
**EN:** Assigns `data` and calls `data.permute(*perm).contiguous`.

**CN:** 将 `data`，并调用 `data.permute(*perm).contiguous`.

### Block 56 — Lines 140-140 (swizzle_data)
```python
140|         data = data.flatten(-10, -1)
```
**EN:** Assigns `data` and calls `data.flatten`.

**CN:** 将 `data`，并调用 `data.flatten`.

### Block 57 — Lines 141-141 (swizzle_data)
```python
141|         data = data.flatten(-3, -2)
```
**EN:** Assigns `data` and calls `data.flatten`.

**CN:** 将 `data`，并调用 `data.flatten`.

### Block 58 — Lines 142-142 (swizzle_data)
```python
142|         assert data.is_contiguous()
```
**EN:** Asserts `data.is_contiguous()` to enforce invariants.

**CN:** 断言 `data.is_contiguous()` 以确保不变量成立。

### Block 59 — Lines 143-143 (swizzle_data)
```python
143|         assert data.shape[-2] == init_shape[-2] // 4
```
**EN:** Asserts `data.shape[-2] == init_shape[-2] // 4` to enforce invariants.

**CN:** 断言 `data.shape[-2] == init_shape[-2] // 4` 以确保不变量成立。

### Block 60 — Lines 144-145 (swizzle_data)
```python
144|         assert data.shape[-1] == init_shape[-1] * 4
145|         # twiddle the bits
```
**EN:** Asserts `data.shape[-1] == init_shape[-1] * 4` to enforce invariants.

**CN:** 断言 `data.shape[-1] == init_shape[-1] * 4` 以确保不变量成立。

### Block 61 — Lines 146-146 (swizzle_data)
```python
146|         data = _pack_bits(data, self.mx_axis)
```
**EN:** Assigns `data` and calls `_pack_bits`.

**CN:** 将 `data`，并调用 `_pack_bits`.

### Block 62 — Lines 147-147 (swizzle_data)
```python
147|         data = self._maybe_mT(data)
```
**EN:** Assigns `data` and calls `self._maybe_mT`.

**CN:** 将 `data`，并调用 `self._maybe_mT`.

### Block 63 — Lines 148-149 (swizzle_data)
```python
148|         return data
149| 
```
**EN:** Returns `data`.

**CN:** 返回 `data`.

### Block 64 — Lines 150-150 (unswizzle_data)
```python
150|     def unswizzle_data(self, data):
```
**EN:** Defines function `unswizzle_data(self, data)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `self._maybe_mT`, `_unpack_bits`, `data.reshape`, `len`, `data.permute` to implement its workflow.

**CN:** 定义函数 `unswizzle_data(self, data)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `self._maybe_mT`, `_unpack_bits`, `data.reshape`, `len`, `data.permute` 来实现其工作流程.

### Block 65 — Lines 151-151 (unswizzle_data)
```python
151|         data = self._maybe_mT(data)
```
**EN:** Assigns `data` and calls `self._maybe_mT`.

**CN:** 将 `data`，并调用 `self._maybe_mT`.

### Block 66 — Lines 152-152 (unswizzle_data)
```python
152|         data = _unpack_bits(data, self.mx_axis)
```
**EN:** Assigns `data` and calls `_unpack_bits`.

**CN:** 将 `data`，并调用 `_unpack_bits`.

### Block 67 — Lines 153-154 (unswizzle_data)
```python
153|         *batch, M, K = data.shape
154|         # We have two times the elements if we already upcasted to bfloat16
```
**EN:** Assigns `batch`, `M`, `K` and references `data.shape`.

**CN:** 将 `batch`, `M`, `K` and 引用 `data.shape`.

### Block 68 — Lines 155-155 (unswizzle_data)
```python
155|         mult = 2 if data.dtype == torch.bfloat16 else 1
```
**EN:** Assigns `mult` and uses conditional expression `2 if data.dtype == torch.bfloat16 else 1`.

**CN:** 将 `mult` and 使用条件表达式 `2 if data.dtype == torch.bfloat16 else 1`.

### Block 69 — Lines 156-156 (unswizzle_data)
```python
156|         assert M % 4 == 0, "M must be divisible by 4"
```
**EN:** Asserts `M % 4 == 0` to enforce invariants.

**CN:** 断言 `M % 4 == 0` 以确保不变量成立。

### Block 70 — Lines 157-159 (unswizzle_data)
```python
157|         assert K % (4 * 8 * 2 * 2 * mult) == 0, f"K must be divisible by {4 * 8 * 2 * 2 * mult}"
158|         # We are loading 8 bf16 elements per thread to use ld.global.v4
159|         # Every u8 represents 2 mxfp4 elements
```
**EN:** Asserts `K % (4 * 8 * 2 * 2 * mult) == 0` to enforce invariants.

**CN:** 断言 `K % (4 * 8 * 2 * 2 * mult) == 0` 以确保不变量成立。

### Block 71 — Lines 160-160 (unswizzle_data)
```python
160|         u8_kwidth = 8 // 2 if self.mma_version == 2 else 1
```
**EN:** Assigns `u8_kwidth` and uses conditional expression `8 // 2 if self.mma_version == 2 else 1`.

**CN:** 将 `u8_kwidth` and 使用条件表达式 `8 // 2 if self.mma_version == 2 else 1`.

### Block 72 — Lines 161-161 (unswizzle_data)
```python
161|         data = data.reshape(*batch, M // 4, 4, K // (4 * 8 * 2 * 2 * mult), 2, 4, 8 // u8_kwidth, 2, u8_kwidth * mult)
```
**EN:** Assigns `data` and calls `data.reshape`.

**CN:** 将 `data`，并调用 `data.reshape`.

### Block 73 — Lines 162-162 (unswizzle_data)
```python
162|         b = len(batch)
```
**EN:** Assigns `b` and calls `len`.

**CN:** 将 `b`，并调用 `len`.

### Block 74 — Lines 163-163 (unswizzle_data)
```python
163|         perm = [0, 6, 1, 3, 2, 5, 4, 7]
```
**EN:** Assigns `perm` and builds a list.

**CN:** 将 `perm` and 构造一个列表.

### Block 75 — Lines 164-164 (unswizzle_data)
```python
164|         perm = list(range(b)) + [b + p for p in perm]
```
**EN:** Assigns `perm` and evaluates `list(range(b)) + [b + p for p in perm]`.

**CN:** 将 `perm` and 计算 `list(range(b)) + [b + p for p in perm]`.

### Block 76 — Lines 165-165 (unswizzle_data)
```python
165|         data = data.permute(*perm)
```
**EN:** Assigns `data` and calls `data.permute`.

**CN:** 将 `data`，并调用 `data.permute`.

### Block 77 — Lines 166-166 (unswizzle_data)
```python
166|         data = data.reshape(*batch, M * 4, K // 4)
```
**EN:** Assigns `data` and calls `data.reshape`.

**CN:** 将 `data`，并调用 `data.reshape`.

### Block 78 — Lines 167-167 (unswizzle_data)
```python
167|         data = self._maybe_mT(data)
```
**EN:** Assigns `data` and calls `self._maybe_mT`.

**CN:** 将 `data`，并调用 `self._maybe_mT`.

### Block 79 — Lines 168-168 (unswizzle_data)
```python
168|         data = repack(data, -2, -1, self.is_fp4)
```
**EN:** Assigns `data` and calls `repack`.

**CN:** 将 `data`，并调用 `repack`.

### Block 80 — Lines 169-169 (unswizzle_data)
```python
169|         data = data[..., :self.K, :self.N // 2]
```
**EN:** Assigns `data` and evaluates `data[..., :self.K, :self.N // 2]`.

**CN:** 将 `data` and 计算 `data[..., :self.K, :self.N // 2]`.

### Block 81 — Lines 170-170 (unswizzle_data)
```python
170|         data = data.contiguous()
```
**EN:** Assigns `data` and calls `data.contiguous`.

**CN:** 将 `data`，并调用 `data.contiguous`.

### Block 82 — Lines 171-173 (unswizzle_data)
```python
171|         return data
172| 
173| 
```
**EN:** Returns `data`.

**CN:** 返回 `data`.

### Block 83 — Lines 174-174 (right_shift_unsigned)
```python
174| def right_shift_unsigned(x, shift):
```
**EN:** Defines function `right_shift_unsigned(x, shift)` for this module. The body mainly returns the computed result.

**CN:** 定义函数 `right_shift_unsigned(x, shift)`，供本模块使用. 主体主要返回计算结果.

### Block 84 — Lines 175-187 (right_shift_unsigned)
```python
175|     return (x >> shift) & ((1 << (32 - shift)) - 1)
176| 
177| 
178| # -----------------------------------------------------------------------
179| # Interleave the bits of four consecutive fp4 values (i.e. 16-bits) as:
180| #     1000000111000000         (first fp4)
181| #        1000000111000000      (second fp4)
182| #           1000000111000000   (third fp4)
183| #     0110110000000000         (fourth fp4)
184| # This is done so that dequantization can be done in 14 SASS instructions
185| # -----------------------------------------------------------------------
186| 
187| 
```
**EN:** Returns `x >> shift & (1 << 32 - shift) - 1`.

**CN:** 返回 `x >> shift & (1 << 32 - shift) - 1`.

### Block 85 — Lines 188-188 (_compress_fp4)
```python
188| def _compress_fp4(x):
```
**EN:** Defines function `_compress_fp4(x)` for this module. The body mainly prepares intermediate values; returns the computed result. It uses calls such as `x.to` to implement its workflow.

**CN:** 定义函数 `_compress_fp4(x)`，供本模块使用. 主体主要准备中间值; 返回计算结果. 其中会调用 `x.to` 来实现其工作流程.

### Block 86 — Lines 189-189 (_compress_fp4)
```python
189|     x = x.to(torch.int32)
```
**EN:** Assigns `x` and calls `x.to`.

**CN:** 将 `x`，并调用 `x.to`.

### Block 87 — Lines 190-192 (_compress_fp4)
```python
190|     return ((x & 0x8) << 12) | ((x & 0x7) << 6)
191| 
192| 
```
**EN:** Returns `(x & 8) << 12 | (x & 7) << 6`.

**CN:** 返回 `(x & 8) << 12 | (x & 7) << 6`.

### Block 88 — Lines 193-193 (_compress_fourth)
```python
193| def _compress_fourth(x):
```
**EN:** Defines function `_compress_fourth(x)` for this module. The body mainly prepares intermediate values; returns the computed result. It uses calls such as `x.to` to implement its workflow.

**CN:** 定义函数 `_compress_fourth(x)`，供本模块使用. 主体主要准备中间值; 返回计算结果. 其中会调用 `x.to` 来实现其工作流程.

### Block 89 — Lines 194-194 (_compress_fourth)
```python
194|     x = x.to(torch.int32)
```
**EN:** Assigns `x` and calls `x.to`.

**CN:** 将 `x`，并调用 `x.to`.

### Block 90 — Lines 195-197 (_compress_fourth)
```python
195|     return ((x & 0x8) << 11) | ((x & 0x6) << 9) | ((x & 0x1) << 13)
196| 
197| 
```
**EN:** Returns `(x & 8) << 11 | (x & 6) << 9 | (x & 1) << 13`.

**CN:** 返回 `(x & 8) << 11 | (x & 6) << 9 | (x & 1) << 13`.

### Block 91 — Lines 198-198 (_pack_bits)
```python
198| def _pack_bits(x: torch.Tensor, mx_axis: int):
```
**EN:** Defines function `_pack_bits(x, mx_axis)` for this module. The body mainly prepares intermediate values; checks invariants; prepares intermediate values. It uses calls such as `x.contiguous`, `x.reshape`, `right_shift_unsigned`, `ret.is_contiguous`, `ret.view` to implement its workflow.

**CN:** 定义函数 `_pack_bits(x, mx_axis)`，供本模块使用. 主体主要准备中间值; 检查不变量; 准备中间值. 其中会调用 `x.contiguous`, `x.reshape`, `right_shift_unsigned`, `ret.is_contiguous`, `ret.view` 来实现其工作流程.

### Block 92 — Lines 199-199 (_pack_bits)
```python
199|     x = x.contiguous()
```
**EN:** Assigns `x` and calls `x.contiguous`.

**CN:** 将 `x`，并调用 `x.contiguous`.

### Block 93 — Lines 200-200 (_pack_bits)
```python
200|     assert x.shape[-1] % 4 == 0, "Input tensor must have a last dimension divisible by 4"
```
**EN:** Asserts `x.shape[-1] % 4 == 0` to enforce invariants.

**CN:** 断言 `x.shape[-1] % 4 == 0` 以确保不变量成立。

### Block 94 — Lines 201-201 (_pack_bits)
```python
201|     x = x.reshape(x.shape[:-1] + (x.shape[-1] // 4, 4))
```
**EN:** Assigns `x` and calls `x.reshape`.

**CN:** 将 `x`，并调用 `x.reshape`.

### Block 95 — Lines 202-202 (_pack_bits)
```python
202|     ret = _compress_fp4(x[..., 0]) | (_compress_fp4(x[..., 0] >> 4) << 16)
```
**EN:** Assigns `ret` and evaluates `_compress_fp4(x[..., 0]) | _compress_fp4(x[..., 0] >> 4) << 16`.

**CN:** 将 `ret` and 计算 `_compress_fp4(x[..., 0]) | _compress_fp4(x[..., 0] >> 4) << 16`.

### Block 96 — Lines 203-203 (_pack_bits)
```python
203|     ret |= right_shift_unsigned(_compress_fp4(x[..., 1]) | (_compress_fp4(x[..., 1] >> 4) << 16), 3)
```
**EN:** Updates `ret` with operator `BitOr` using `right_shift_unsigned(_compress_fp4(x[..., 1]) | _compress...`.

**CN:** 更新 `ret`，使用运算符 `BitOr`，并使用 `right_shift_unsigned(_compress_fp4(x[..., 1]) | _compress...`.

### Block 97 — Lines 204-204 (_pack_bits)
```python
204|     ret |= right_shift_unsigned(_compress_fp4(x[..., 2]) | (_compress_fp4(x[..., 2] >> 4) << 16), 6)
```
**EN:** Updates `ret` with operator `BitOr` using `right_shift_unsigned(_compress_fp4(x[..., 2]) | _compress...`.

**CN:** 更新 `ret`，使用运算符 `BitOr`，并使用 `right_shift_unsigned(_compress_fp4(x[..., 2]) | _compress...`.

### Block 98 — Lines 205-205 (_pack_bits)
```python
205|     ret |= _compress_fourth(x[..., 3]) | (_compress_fourth(x[..., 3] >> 4) << 16)
```
**EN:** Updates `ret` with operator `BitOr` using `_compress_fourth(x[..., 3]) | _compress_fourth(x[..., 3] ...`.

**CN:** 更新 `ret`，使用运算符 `BitOr`，并使用 `_compress_fourth(x[..., 3]) | _compress_fourth(x[..., 3] ...`.

### Block 99 — Lines 206-206 (_pack_bits)
```python
206|     assert ret.is_contiguous()
```
**EN:** Asserts `ret.is_contiguous()` to enforce invariants.

**CN:** 断言 `ret.is_contiguous()` 以确保不变量成立。

### Block 100 — Lines 207-207 (_pack_bits)
```python
207|     ret = ret.view(torch.uint8)
```
**EN:** Assigns `ret` and calls `ret.view`.

**CN:** 将 `ret`，并调用 `ret.view`.

### Block 101 — Lines 208-215 (_pack_bits)
```python
208|     return ret
209| 
210| 
211| # -----------------------------------------------------------------------
212| # inverse operation of _pack_bits
213| # -----------------------------------------------------------------------
214| 
215| 
```
**EN:** Returns `ret`.

**CN:** 返回 `ret`.

### Block 102 — Lines 216-217 (_bf16_to_fp4e2m1)
```python
216| def _bf16_to_fp4e2m1(x):
217|     # 0bAxxxxxxBCDxxxxxx (int16) -> 0b0000ABCD (uint8)
```
**EN:** Defines function `_bf16_to_fp4e2m1(x)` for this module. The body mainly checks invariants; prepares intermediate values; prepares intermediate values. It uses calls such as `(s | em).to`, `right_shift_unsigned` to implement its workflow.

**CN:** 定义函数 `_bf16_to_fp4e2m1(x)`，供本模块使用. 主体主要检查不变量; 准备中间值; 准备中间值. 其中会调用 `(s | em).to`, `right_shift_unsigned` 来实现其工作流程.

### Block 103 — Lines 218-218 (_bf16_to_fp4e2m1)
```python
218|     assert x.dtype == torch.int16
```
**EN:** Asserts `x.dtype == torch.int16` to enforce invariants.

**CN:** 断言 `x.dtype == torch.int16` 以确保不变量成立。

### Block 104 — Lines 219-219 (_bf16_to_fp4e2m1)
```python
219|     s = (right_shift_unsigned(x, 15) & 0x1) << 3
```
**EN:** Assigns `s` and evaluates `(right_shift_unsigned(x, 15) & 1) << 3`.

**CN:** 将 `s` and 计算 `(right_shift_unsigned(x, 15) & 1) << 3`.

### Block 105 — Lines 220-220 (_bf16_to_fp4e2m1)
```python
220|     em = right_shift_unsigned(x, 6) & 0x7
```
**EN:** Assigns `em` and evaluates `right_shift_unsigned(x, 6) & 7`.

**CN:** 将 `em` and 计算 `right_shift_unsigned(x, 6) & 7`.

### Block 106 — Lines 221-223 (_bf16_to_fp4e2m1)
```python
221|     return (s | em).to(torch.uint8)
222| 
223| 
```
**EN:** Returns `(s | em).to(torch.uint8)`.

**CN:** 返回 `(s | em).to(torch.uint8)`.

### Block 107 — Lines 224-225 (_bf16x2_to_fp4e2m1x2)
```python
224| def _bf16x2_to_fp4e2m1x2(x):
225|     # 0bAxxxxxxBCDxxxxxx_0bExxxxxxFGHxxxxxx  (int32) -> 0bABCD_EFGH (uint8)
```
**EN:** Defines function `_bf16x2_to_fp4e2m1x2(x)` for this module. The body mainly checks invariants; prepares intermediate values; prepares intermediate values. It uses calls such as `(x & 65535).to`, `(right_shift_unsigned(x, 16) & 65535).to`, `_bf16_to_fp4e2m1`, `right_shift_unsigned` to implement its workflow.

**CN:** 定义函数 `_bf16x2_to_fp4e2m1x2(x)`，供本模块使用. 主体主要检查不变量; 准备中间值; 准备中间值. 其中会调用 `(x & 65535).to`, `(right_shift_unsigned(x, 16) & 65535).to`, `_bf16_to_fp4e2m1`, `right_shift_unsigned` 来实现其工作流程.

### Block 108 — Lines 226-226 (_bf16x2_to_fp4e2m1x2)
```python
226|     assert x.dtype == torch.int32
```
**EN:** Asserts `x.dtype == torch.int32` to enforce invariants.

**CN:** 断言 `x.dtype == torch.int32` 以确保不变量成立。

### Block 109 — Lines 227-227 (_bf16x2_to_fp4e2m1x2)
```python
227|     lo = (x & 0xFFFF).to(torch.int16)
```
**EN:** Assigns `lo` and calls `(x & 65535).to`.

**CN:** 将 `lo`，并调用 `(x & 65535).to`.

### Block 110 — Lines 228-228 (_bf16x2_to_fp4e2m1x2)
```python
228|     hi = (right_shift_unsigned(x, 16) & 0xFFFF).to(torch.int16)
```
**EN:** Assigns `hi` and calls `(right_shift_unsigned(x, 16) & 65535).to`.

**CN:** 将 `hi`，并调用 `(right_shift_unsigned(x, 16) & 65535).to`.

### Block 111 — Lines 229-229 (_bf16x2_to_fp4e2m1x2)
```python
229|     ret_lo = _bf16_to_fp4e2m1(lo)
```
**EN:** Assigns `ret_lo` and calls `_bf16_to_fp4e2m1`.

**CN:** 将 `ret_lo`，并调用 `_bf16_to_fp4e2m1`.

### Block 112 — Lines 230-230 (_bf16x2_to_fp4e2m1x2)
```python
230|     ret_hi = _bf16_to_fp4e2m1(hi)
```
**EN:** Assigns `ret_hi` and calls `_bf16_to_fp4e2m1`.

**CN:** 将 `ret_hi`，并调用 `_bf16_to_fp4e2m1`.

### Block 113 — Lines 231-233 (_bf16x2_to_fp4e2m1x2)
```python
231|     return ret_lo | (ret_hi << 4)
232| 
233| 
```
**EN:** Returns `ret_lo | ret_hi << 4`.

**CN:** 返回 `ret_lo | ret_hi << 4`.

### Block 114 — Lines 234-234 (_unpack_bits)
```python
234| def _unpack_bits(x, mx_axis: int):
```
**EN:** Defines function `_unpack_bits(x, mx_axis)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `x.view`, `torch.stack`, `x.flatten`, `_bf16x2_to_fp4e2m1x2`, `right_shift_unsigned` to implement its workflow.

**CN:** 定义函数 `_unpack_bits(x, mx_axis)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `x.view`, `torch.stack`, `x.flatten`, `_bf16x2_to_fp4e2m1x2`, `right_shift_unsigned` 来实现其工作流程.

### Block 115 — Lines 235-235 (_unpack_bits)
```python
235|     x = x.view(torch.int32)
```
**EN:** Assigns `x` and calls `x.view`.

**CN:** 将 `x`，并调用 `x.view`.

### Block 116 — Lines 236-236 (_unpack_bits)
```python
236|     m = 0b10000001110000001000000111000000
```
**EN:** Assigns `m` and stores constant `2176876992`.

**CN:** 将 `m` and 保存常量 `2176876992`.

### Block 117 — Lines 237-237 (_unpack_bits)
```python
237|     a = (x << 1) & 0b10000000000000001000000000000000
```
**EN:** Assigns `a` and evaluates `x << 1 & 2147516416`.

**CN:** 将 `a` and 计算 `x << 1 & 2147516416`.

### Block 118 — Lines 238-238 (_unpack_bits)
```python
238|     b = right_shift_unsigned(x, 3) & 0b00000001100000000000000110000000
```
**EN:** Assigns `b` and evaluates `right_shift_unsigned(x, 3) & 25166208`.

**CN:** 将 `b` and 计算 `right_shift_unsigned(x, 3) & 25166208`.

### Block 119 — Lines 239-239 (_unpack_bits)
```python
239|     c = right_shift_unsigned(x, 7) & 0b00000000010000000000000001000000
```
**EN:** Assigns `c` and evaluates `right_shift_unsigned(x, 7) & 4194368`.

**CN:** 将 `c` and 计算 `right_shift_unsigned(x, 7) & 4194368`.

### Block 120 — Lines 240-240 (_unpack_bits)
```python
240|     unpacked = [x & m, (x << 3) & m, (x << 6) & m, (a | b) | c]
```
**EN:** Assigns `unpacked` and builds a list.

**CN:** 将 `unpacked` and 构造一个列表.

### Block 121 — Lines 241-241 (_unpack_bits)
```python
241|     x = torch.stack(unpacked, dim=-1)
```
**EN:** Assigns `x` and calls `torch.stack`.

**CN:** 将 `x`，并调用 `torch.stack`.

### Block 122 — Lines 242-242 (_unpack_bits)
```python
242|     x = x.flatten(-2, -1)
```
**EN:** Assigns `x` and calls `x.flatten`.

**CN:** 将 `x`，并调用 `x.flatten`.

### Block 123 — Lines 243-243 (_unpack_bits)
```python
243|     x = _bf16x2_to_fp4e2m1x2(x)
```
**EN:** Assigns `x` and calls `_bf16x2_to_fp4e2m1x2`.

**CN:** 将 `x`，并调用 `_bf16x2_to_fp4e2m1x2`.

### Block 124 — Lines 244-249 (_unpack_bits)
```python
244|     return x
245| 
246| 
247| # -----------------------------------------------------------------------
248| 
249| 
```
**EN:** Returns `x`.

**CN:** 返回 `x`.

### Block 125 — Lines 250-251 (_unshuffle_triton)
```python
250| @triton.jit
251| def _unshuffle_triton(x, mma_version: tl.constexpr):
```
**EN:** Defines function `_unshuffle_triton(x, mma_version)` with decorators `triton.jit` for this module. The body mainly invokes `tl.static_assert`; prepares intermediate values; prepares intermediate values. It uses calls such as `tl.static_assert`, `x.reshape`, `x.trans` to implement its workflow.

**CN:** 定义函数 `_unshuffle_triton(x, mma_version)`，带有装饰器 `triton.jit`，供本模块使用. 主体主要invokes `tl.static_assert`; 准备中间值; 准备中间值. 其中会调用 `tl.static_assert`, `x.reshape`, `x.trans` 来实现其工作流程.

### Block 126 — Lines 252-254 (_unshuffle_triton)
```python
252|     """
253|     Triton inverse of swizzle_mxfp4_value_hopper
254|     """
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 127 — Lines 255-259 (_unshuffle_triton)
```python
255|     tl.static_assert(mma_version == 2 or mma_version == 3, "mma_version must be 2 or 3")
256|     # if mx_axis == 0:
257|     #     x = x.trans()
258| 
259|     # We have two times the elements if we already upcasted to bfloat16
```
**EN:** Calls `tl.static_assert` for side effects, registration, or validation.

**CN:** 调用 `tl.static_assert` ，用于副作用、注册或校验。

### Block 128 — Lines 260-260 (_unshuffle_triton)
```python
260|     mult: tl.constexpr = 2 if x.dtype == tl.bfloat16 else 1
```
**EN:** Annotated assignment stores `mult` and uses conditional expression `2 if x.dtype == tl.bfloat16 else 1`.

**CN:** 带类型注解的赋值保存 `mult` and 使用条件表达式 `2 if x.dtype == tl.bfloat16 else 1`.

### Block 129 — Lines 261-261 (_unshuffle_triton)
```python
261|     M: tl.constexpr = x.shape[0]
```
**EN:** Annotated assignment stores `M` and evaluates `x.shape[0]`.

**CN:** 带类型注解的赋值保存 `M` and 计算 `x.shape[0]`.

### Block 130 — Lines 262-262 (_unshuffle_triton)
```python
262|     K: tl.constexpr = x.shape[1]
```
**EN:** Annotated assignment stores `K` and evaluates `x.shape[1]`.

**CN:** 带类型注解的赋值保存 `K` and 计算 `x.shape[1]`.

### Block 131 — Lines 263-263 (_unshuffle_triton)
```python
263|     tl.static_assert(M % 4 == 0, "M must be divisible by 4")
```
**EN:** Calls `tl.static_assert` for side effects, registration, or validation.

**CN:** 调用 `tl.static_assert` ，用于副作用、注册或校验。

### Block 132 — Lines 264-267 (_unshuffle_triton)
```python
264|     tl.static_assert(K % (4 * 8 * 2 * 2 * mult) == 0, f"K must be divisible by {4 * 8 * 2 * 2 * mult}")
265| 
266|     # We are loading 8 bf16 elements per thread to use ld.global.v4
267|     # Every u8 represents 2 mxfp4 elements
```
**EN:** Calls `tl.static_assert` for side effects, registration, or validation.

**CN:** 调用 `tl.static_assert` ，用于副作用、注册或校验。

### Block 133 — Lines 268-268 (_unshuffle_triton)
```python
268|     u8_kwidth: tl.constexpr = 8 // 2 if mma_version == 2 else 1
```
**EN:** Annotated assignment stores `u8_kwidth` and uses conditional expression `8 // 2 if mma_version == 2 else 1`.

**CN:** 带类型注解的赋值保存 `u8_kwidth` and 使用条件表达式 `8 // 2 if mma_version == 2 else 1`.

### Block 134 — Lines 269-269 (_unshuffle_triton)
```python
269|     x = x.reshape(M // 4, 4, K // (4 * 8 * 2 * 2 * mult), 2, 4, 8 // u8_kwidth, 2, u8_kwidth * mult)
```
**EN:** Assigns `x` and calls `x.reshape`.

**CN:** 将 `x`，并调用 `x.reshape`.

### Block 135 — Lines 270-270 (_unshuffle_triton)
```python
270|     x = x.trans(0, 6, 1, 3, 2, 5, 4, 7)
```
**EN:** Assigns `x` and calls `x.trans`.

**CN:** 将 `x`，并调用 `x.trans`.

### Block 136 — Lines 271-273 (_unshuffle_triton)
```python
271|     x = x.reshape(M * 4, K // 4)
272|     # if mx_axis == 0:
273|     #     x = x.trans()
```
**EN:** Assigns `x` and calls `x.reshape`.

**CN:** 将 `x`，并调用 `x.reshape`.

### Block 137 — Lines 274-276 (_unshuffle_triton)
```python
274|     return x
275| 
276| 
```
**EN:** Returns `x`.

**CN:** 返回 `x`.

### Block 138 — Lines 277-279 (_unpack_fp4_to_bf16_triton)
```python
277| @triton.jit
278| def _unpack_fp4_to_bf16_triton(x):
279|     # Use fma on a100 as there is no mul.bf16x2.
```
**EN:** Defines function `_unpack_fp4_to_bf16_triton(x)` with decorators `triton.jit` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `cuda_capability_geq`, `tl.inline_asm_elementwise`, `tl.join`, `x.reshape`, `x.trans` to implement its workflow.

**CN:** 定义函数 `_unpack_fp4_to_bf16_triton(x)`，带有装饰器 `triton.jit`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `cuda_capability_geq`, `tl.inline_asm_elementwise`, `tl.join`, `x.reshape`, `x.trans` 来实现其工作流程.

### Block 139 — Lines 280-280 (_unpack_fp4_to_bf16_triton)
```python
280|     use_mul: tl.constexpr = cuda_capability_geq(9)
```
**EN:** Annotated assignment stores `use_mul` and calls `cuda_capability_geq`.

**CN:** 带类型注解的赋值保存 `use_mul`，并调用 `cuda_capability_geq`.

### Block 140 — Lines 281-281 (_unpack_fp4_to_bf16_triton)
```python
281|     op_instr: tl.constexpr = "mul.bf16x2" if use_mul else "fma.rn.bf16x2"
```
**EN:** Annotated assignment stores `op_instr` and uses conditional expression `'mul.bf16x2' if use_mul else 'fma.rn.bf16x2'`.

**CN:** 带类型注解的赋值保存 `op_instr` and 使用条件表达式 `'mul.bf16x2' if use_mul else 'fma.rn.bf16x2'`.

### Block 141 — Lines 282-282 (_unpack_fp4_to_bf16_triton)
```python
282|     op_suffix: tl.constexpr = "" if use_mul else ", z"
```
**EN:** Annotated assignment stores `op_suffix` and uses conditional expression `'' if use_mul else ', z'`.

**CN:** 带类型注解的赋值保存 `op_suffix` and 使用条件表达式 `'' if use_mul else ', z'`.

### Block 142 — Lines 283-315 (_unpack_fp4_to_bf16_triton)
```python
283|     r0, r1 = tl.inline_asm_elementwise(
284|         asm=f"""{{
285|             .reg .b32 b, c, z, d<7>, scale;
286|             .reg .b32 bias;
287|             mov.b32 z, 0;
288|             mov.b32 bias, 0x7e807e80; // 2 ** 126 == 2 ** (bias_bf16 - bias_fp2)
289|             // We add the missing bias to the scale directly
290|             and.b32 $0, $4, 0b10000001110000001000000111000000;
291|             {op_instr} $0, $0, bias{op_suffix};
292|             shl.b32 b, $4, 3;
293|             and.b32 $1, b,  0b10000001110000001000000111000000;
294|             {op_instr} $1, $1, bias{op_suffix};
295|             shl.b32 c, $4, 6;
296|             and.b32 $2, c,  0b10000001110000001000000111000000;
297|             {op_instr} $2, $2, bias{op_suffix};
298|             // Unpack last two elements
299|             shl.b32 d0, $4, 1;
300|             and.b32 d1, d0, 0b10000000000000001000000000000000;
301|             shr.b32 d2, $4, 3;
302|             and.b32 d3, d2, 0b00000001100000000000000110000000;
303|             or.b32 d4, d1, d3;
304|             shr.b32 d5, $4, 7;
305|             and.b32 d6, d5, 0b00000000010000000000000001000000;
306|             or.b32 $3, d4, d6;
307|             {op_instr} $3, $3, bias{op_suffix};
308|         }}""",
309|         constraints="=r,=r,=r,=r,r",
310|         args=[x],
311|         dtype=(tl.bfloat16, tl.bfloat16),
312|         is_pure=True,
313|         pack=4,
314|     )
315|     # Concat each pack of 4
```
**EN:** Assigns `r0`, `r1` and calls `tl.inline_asm_elementwise`.

**CN:** 将 `r0`, `r1`，并调用 `tl.inline_asm_elementwise`.

### Block 143 — Lines 316-316 (_unpack_fp4_to_bf16_triton)
```python
316|     x = tl.join(r0, r1)
```
**EN:** Assigns `x` and calls `tl.join`.

**CN:** 将 `x`，并调用 `tl.join`.

### Block 144 — Lines 317-317 (_unpack_fp4_to_bf16_triton)
```python
317|     x = x.reshape(x.shape[0], x.shape[1] // 4, 4, x.shape[2])
```
**EN:** Assigns `x` and calls `x.reshape`.

**CN:** 将 `x`，并调用 `x.reshape`.

### Block 145 — Lines 318-318 (_unpack_fp4_to_bf16_triton)
```python
318|     x = x.trans(0, 1, 3, 2)
```
**EN:** Assigns `x` and calls `x.trans`.

**CN:** 将 `x`，并调用 `x.trans`.

### Block 146 — Lines 319-319 (_unpack_fp4_to_bf16_triton)
```python
319|     x = x.reshape(x.shape[0], x.shape[1] * x.shape[2] * x.shape[3])
```
**EN:** Assigns `x` and calls `x.reshape`.

**CN:** 将 `x`，并调用 `x.reshape`.

### Block 147 — Lines 320-322 (_unpack_fp4_to_bf16_triton)
```python
320|     return x
321| 
322| 
```
**EN:** Returns `x`.

**CN:** 返回 `x`.

### Block 148 — Lines 323-324 (mul_bf16x2)
```python
323| @triton.jit
324| def mul_bf16x2(a, b):
```
**EN:** Defines function `mul_bf16x2(a, b)` with decorators `triton.jit` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `cuda_capability_geq`, `tl.inline_asm_elementwise` to implement its workflow.

**CN:** 定义函数 `mul_bf16x2(a, b)`，带有装饰器 `triton.jit`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `cuda_capability_geq`, `tl.inline_asm_elementwise` 来实现其工作流程.

### Block 149 — Lines 325-325 (mul_bf16x2)
```python
325|     use_mul: tl.constexpr = cuda_capability_geq(9)
```
**EN:** Annotated assignment stores `use_mul` and calls `cuda_capability_geq`.

**CN:** 带类型注解的赋值保存 `use_mul`，并调用 `cuda_capability_geq`.

### Block 150 — Lines 326-326 (mul_bf16x2)
```python
326|     op_instr: tl.constexpr = "mul.bf16x2" if use_mul else "fma.rn.bf16x2"
```
**EN:** Annotated assignment stores `op_instr` and uses conditional expression `'mul.bf16x2' if use_mul else 'fma.rn.bf16x2'`.

**CN:** 带类型注解的赋值保存 `op_instr` and 使用条件表达式 `'mul.bf16x2' if use_mul else 'fma.rn.bf16x2'`.

### Block 151 — Lines 327-328 (mul_bf16x2)
```python
327|     op_suffix: tl.constexpr = "" if use_mul else ", z"
328| 
```
**EN:** Annotated assignment stores `op_suffix` and uses conditional expression `'' if use_mul else ', z'`.

**CN:** 带类型注解的赋值保存 `op_suffix` and 使用条件表达式 `'' if use_mul else ', z'`.

### Block 152 — Lines 329-338 (mul_bf16x2)
```python
329|     return tl.inline_asm_elementwise(
330|         asm=f"{op_instr} $0, $1, $2{op_suffix};",
331|         constraints="=r,r,r",
332|         args=[a, b],
333|         dtype=tl.bfloat16,
334|         is_pure=True,
335|         pack=2,
336|     )
337| 
338| 
```
**EN:** Returns `tl.inline_asm_elementwise(asm=f'{op_instr} $0, $1, $2{op_suffix};', constrain...`.

**CN:** 返回 `tl.inline_asm_elementwise(asm=f'{op_instr} $0, $1, $2{op_suffix};', constrain...`.

### Block 153 — Lines 339-340 (mxfp4_to_bf16_triton)
```python
339| @triton.jit
340| def mxfp4_to_bf16_triton(x, scale, mx_axis: tl.constexpr):
```
**EN:** Defines function `mxfp4_to_bf16_triton(x, scale, mx_axis)` with decorators `triton.jit` for this module. The body mainly invokes `tl.static_assert`; invokes `tl.static_assert`; invokes `tl.static_assert`. It uses calls such as `tl.static_assert`, `_unpack_fp4_to_bf16_triton`, `_unshuffle_triton`, `tl.inline_asm_elementwise`, `tl.static_range` to implement its workflow.

**CN:** 定义函数 `mxfp4_to_bf16_triton(x, scale, mx_axis)`，带有装饰器 `triton.jit`，供本模块使用. 主体主要invokes `tl.static_assert`; invokes `tl.static_assert`; invokes `tl.static_assert`. 其中会调用 `tl.static_assert`, `_unpack_fp4_to_bf16_triton`, `_unshuffle_triton`, `tl.inline_asm_elementwise`, `tl.static_range` 来实现其工作流程.

### Block 154 — Lines 341-348 (mxfp4_to_bf16_triton)
```python
341|     """
342|     Implements the bit-untwiddling of a 32-bit integer (8 mxfp4 elements):
343|     (x << 0) & 0b1000000111000000
344|     (x << 3) & 0b1000000111000000
345|     (x << 6) & 0b1000000111000000
346|     ((x << 1) & 0b1000000000000000) | ((x >> 3) & 0b0000000110000000) | ((x >> 7) & 0b0000000001000000)
347|     """
348|     # upcast values to bfloat16
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 155 — Lines 349-349 (mxfp4_to_bf16_triton)
```python
349|     tl.static_assert(len(x.shape) == 2)
```
**EN:** Calls `tl.static_assert` for side effects, registration, or validation.

**CN:** 调用 `tl.static_assert` ，用于副作用、注册或校验。

### Block 156 — Lines 350-350 (mxfp4_to_bf16_triton)
```python
350|     tl.static_assert(mx_axis == 0 or mx_axis == 1, "mx_axis must be 0 or 1")
```
**EN:** Calls `tl.static_assert` for side effects, registration, or validation.

**CN:** 调用 `tl.static_assert` ，用于副作用、注册或校验。

### Block 157 — Lines 351-351 (mxfp4_to_bf16_triton)
```python
351|     tl.static_assert(x.shape[1] % 4 == 0)
```
**EN:** Calls `tl.static_assert` for side effects, registration, or validation.

**CN:** 调用 `tl.static_assert` ，用于副作用、注册或校验。

### Block 158 — Lines 352-352 (mxfp4_to_bf16_triton)
```python
352|     tl.static_assert(x.dtype == tl.uint8)
```
**EN:** Calls `tl.static_assert` for side effects, registration, or validation.

**CN:** 调用 `tl.static_assert` ，用于副作用、注册或校验。

### Block 159 — Lines 353-354 (mxfp4_to_bf16_triton)
```python
353|     if mx_axis == 0:
354|         x = x.trans()
```
**EN:** Checks `mx_axis == 0`. The true branch mainly prepares intermediate values.

**CN:** 检查 `mx_axis == 0`. 真分支主要准备中间值.

### Block 160 — Lines 355-355 (mxfp4_to_bf16_triton)
```python
355|     x = _unpack_fp4_to_bf16_triton(x)
```
**EN:** Assigns `x` and calls `_unpack_fp4_to_bf16_triton`.

**CN:** 将 `x`，并调用 `_unpack_fp4_to_bf16_triton`.

### Block 161 — Lines 356-356 (mxfp4_to_bf16_triton)
```python
356|     x = _unshuffle_triton(x, mma_version=3)
```
**EN:** Assigns `x` and calls `_unshuffle_triton`.

**CN:** 将 `x`，并调用 `_unshuffle_triton`.

### Block 162 — Lines 357-365 (mxfp4_to_bf16_triton)
```python
357|     if mx_axis == 0:
358|         x = x.trans()
359| 
360|     # upcast scale to bfloat16
361|     # Add bias missing from the bf16 upcasting sequence
362|     # triton / LLVM generates terrible code for this sequence
363|     # scale = scale.to(tl.uint16)
364|     # scale = scale << 7
365|     # scale = scale.to(tl.bfloat16, bitcast=True)
```
**EN:** Checks `mx_axis == 0`. The true branch mainly prepares intermediate values.

**CN:** 检查 `mx_axis == 0`. 真分支主要准备中间值.

### Block 163 — Lines 366-381 (mxfp4_to_bf16_triton)
```python
366|     scale = tl.inline_asm_elementwise(
367|         r"""
368|         {
369|             prmt.b32 $0, $2, 0, 0x5140;
370|             shl.b32 $0, $0, 7;
371|             prmt.b32 $1, $2, 0, 0x7362;
372|             shl.b32 $1, $1, 7;
373|         }
374|         """,
375|         constraints="=r,=r,r",
376|         args=[scale],
377|         dtype=tl.bfloat16,
378|         is_pure=True,
379|         pack=4,
380|     )
381|     # Sanity check shape
```
**EN:** Assigns `scale` and calls `tl.inline_asm_elementwise`.

**CN:** 将 `scale`，并调用 `tl.inline_asm_elementwise`.

### Block 164 — Lines 382-387 (mxfp4_to_bf16_triton)
```python
382|     for axis in tl.static_range(len(x.shape)):
383|         if axis == mx_axis:
384|             tl.static_assert(x.shape[axis] == MXFP_BLOCK_SIZE * scale.shape[axis])
385|         else:
386|             tl.static_assert(x.shape[axis] == scale.shape[axis])
387|     # Broadcast scale
```
**EN:** Loops over `tl.static_range(len(x.shape))` with target `axis`. The loop body mainly branches on runtime conditions.

**CN:** 遍历 `tl.static_range(len(x.shape))` ，目标变量为 `axis`. 循环体主要根据运行时条件分支.

### Block 165 — Lines 388-388 (mxfp4_to_bf16_triton)
```python
388|     scale = scale.expand_dims(mx_axis + 1)
```
**EN:** Assigns `scale` and calls `scale.expand_dims`.

**CN:** 将 `scale`，并调用 `scale.expand_dims`.

### Block 166 — Lines 389-389 (mxfp4_to_bf16_triton)
```python
389|     scale = scale.broadcast_to(scale.shape[:mx_axis + 1] + [MXFP_BLOCK_SIZE] + scale.shape[mx_axis + 2:])
```
**EN:** Assigns `scale` and calls `scale.broadcast_to`.

**CN:** 将 `scale`，并调用 `scale.broadcast_to`.

### Block 167 — Lines 390-392 (mxfp4_to_bf16_triton)
```python
390|     scale = scale.reshape(x.shape)
391| 
392|     # Combine scale and x
```
**EN:** Assigns `scale` and calls `scale.reshape`.

**CN:** 将 `scale`，并调用 `scale.reshape`.

### Block 168 — Lines 393-393 (mxfp4_to_bf16_triton)
```python
393|     x = mul_bf16x2(x, scale)
```
**EN:** Assigns `x` and calls `mul_bf16x2`.

**CN:** 将 `x`，并调用 `mul_bf16x2`.

### Block 169 — Lines 394-394 (mxfp4_to_bf16_triton)
```python
394|     return x
```
**EN:** Returns `x`.

**CN:** 返回 `x`.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `HopperMXValueLayout`, `HopperMXValueLayoutTransformation`, `right_shift_unsigned`, `_compress_fp4`, `_compress_fourth`, `_pack_bits`, `_bf16_to_fp4e2m1`, `_bf16x2_to_fp4e2m1x2`.
  **CN:** 主要符号：`HopperMXValueLayout`, `HopperMXValueLayoutTransformation`, `right_shift_unsigned`, `_compress_fp4`, `_compress_fourth`, `_pack_bits`, `_bf16_to_fp4e2m1`, `_bf16x2_to_fp4e2m1x2`。
- **EN:** PyTorch tensors are the main data container and host-side orchestration layer.
  **CN:** PyTorch 张量是主要的数据容器和主机端编排层。
- **EN:** Triton is used to JIT-compile or launch GPU kernels and to express tile-level execution details.
  **CN:** Triton 用于 JIT 编译或启动 GPU 内核，并表达分块级执行细节。
- **EN:** Mixed-precision MXFP/NVFP formats and scale tensors are part of the numeric pipeline.
  **CN:** 混合精度 MXFP/NVFP 格式及其缩放张量是数值流程的一部分。
- **EN:** Custom layout objects describe how logical tensors are packed or swizzled in memory.
  **CN:** 自定义 layout 对象描述逻辑张量如何在内存中打包或重排。

## Dependencies / 依赖关系
- **EN:** External modules: `torch`, `triton`, `triton.language`, `dataclasses (dataclass)`.
  **CN:** 外部模块：`torch`, `triton`, `triton.language`, `dataclasses (dataclass)`。
- **EN:** Internal modules: `.base (Layout, LayoutTransformation)`, `triton_kernels.numerics_details.mxfp_details._downcast_to_mxfp (MXFP_BLOCK_SIZE)`, `triton_kernels.target_info (cuda_capability_geq)`, `.torch_utils (repack)`.
  **CN:** 内部模块：`.base (Layout, LayoutTransformation)`, `triton_kernels.numerics_details.mxfp_details._downcast_to_mxfp (MXFP_BLOCK_SIZE)`, `triton_kernels.target_info (cuda_capability_geq)`, `.torch_utils (repack)`。
