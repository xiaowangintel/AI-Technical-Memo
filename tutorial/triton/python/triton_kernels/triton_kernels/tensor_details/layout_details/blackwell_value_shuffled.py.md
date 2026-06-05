# blackwell_value_shuffled.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/triton_kernels/tensor_details/layout_details/blackwell_value_shuffled.py`
- **Purpose / 用途:** Implementation module for blackwell value shuffled; it defines the runtime logic, helper types, and GPU-facing utilities used by the Triton kernels package via symbols BlackwellMX4ValueShuffledLayout, BlackwellMX4ValueShuffledTransformation. / 用于 blackwell value shuffled 的实现模块；它定义了 Triton kernels 包使用的运行时逻辑、辅助类型以及面向 GPU 的工具；核心符号包括 BlackwellMX4ValueShuffledLayout、BlackwellMX4ValueShuffledTransformation。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-8 (module)
```python
1| import math
2| from dataclasses import dataclass
3| import torch
4| from .base import Layout, LayoutTransformation
5| from .torch_utils import repack
6| 
7| 
8| # ------------------- Blackwell MX4 Value Shuffled Layout -------------------
```
**EN:** This block imports `math`, `dataclasses (dataclass)`, `torch`, `.base (Layout, LayoutTransformation)`, `.torch_utils (repack)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `math`, `dataclasses (dataclass)`, `torch`, `.base (Layout, LayoutTransformation)`, `.torch_utils (repack)` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 9-10 (BlackwellMX4ValueShuffledLayout)
```python
 9| @dataclass(frozen=True)
10| class BlackwellMX4ValueShuffledLayout(Layout):
```
**EN:** Defines class `BlackwellMX4ValueShuffledLayout` inheriting from `Layout` with decorators `dataclass(frozen=True)` to organize related behavior. Key fields include `block_k`, `block_n`. Main methods are `name`, `make_transformation`, `swizzle_block_shape`.

**CN:** 定义类 `BlackwellMX4ValueShuffledLayout`，继承自 `Layout`，带有装饰器 `dataclass(frozen=True)`，用于组织相关行为。关键字段包括 `block_k`, `block_n`。主要方法有 `name`, `make_transformation`, `swizzle_block_shape`.

### Block 3 — Lines 11-28 (BlackwellMX4ValueShuffledLayout)
```python
11|     """
12|     Shuffled weight layout for MX4 matmul on Blackwell GPUs.
13| 
14|     Physical packed storage for mxfp4 is column-major with shape [E, K_packed, N],
15|     where K_packed = K // 2 (two FP4 values per byte).
16| 
17|     Baseline TMA loads operate on the swapped view [E, N, K_packed] with block
18|     shape [block_n, packed_block_k], then the kernel transposes to
19|     [packed_block_k, block_n].
20| 
21|     This layout pre-arranges those tiles so each tile is contiguous in memory,
22|     matching the baseline's post-transpose format [block_n, packed_block_k].
23|     We use a 5D layout:
24|     [E, num_tiles_k, num_tiles_n, tile_n, tile_k_packed]
25| 
26|     The inner dimensions [tile_n, tile_k_packed] match the baseline TMA block
27|     shape after swapping, so no transpose is needed after TMA load.
28|     """
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 4 — Lines 29-29 (BlackwellMX4ValueShuffledLayout)
```python
29|     block_k: int = 128
```
**EN:** Annotated assignment stores `block_k` and stores constant `128`.

**CN:** 带类型注解的赋值保存 `block_k` and 保存常量 `128`.

### Block 5 — Lines 30-31 (BlackwellMX4ValueShuffledLayout)
```python
30|     block_n: int = 256
31| 
```
**EN:** Annotated assignment stores `block_n` and stores constant `256`.

**CN:** 带类型注解的赋值保存 `block_n` and 保存常量 `256`.

### Block 6 — Lines 32-33 (name)
```python
32|     @property
33|     def name(self):
```
**EN:** Defines function `name(self)` with decorators `property` for this module. The body mainly returns the computed result.

**CN:** 定义函数 `name(self)`，带有装饰器 `property`，供本模块使用. 主体主要返回计算结果.

### Block 7 — Lines 34-35 (name)
```python
34|         return "BLACKWELL_MX4_VALUE_SHUFFLED"
35| 
```
**EN:** Returns `'BLACKWELL_MX4_VALUE_SHUFFLED'`.

**CN:** 返回 `'BLACKWELL_MX4_VALUE_SHUFFLED'`.

### Block 8 — Lines 36-36 (make_transformation)
```python
36|     def make_transformation(self, shape: list[int], is_fp4: bool) -> LayoutTransformation:
```
**EN:** Defines function `make_transformation(self, shape, is_fp4)` for this module. The body mainly returns the computed result. It uses calls such as `BlackwellMX4ValueShuffledTransformation` to implement its workflow.

**CN:** 定义函数 `make_transformation(self, shape, is_fp4)`，供本模块使用. 主体主要返回计算结果. 其中会调用 `BlackwellMX4ValueShuffledTransformation` 来实现其工作流程.

### Block 9 — Lines 37-38 (make_transformation)
```python
37|         return BlackwellMX4ValueShuffledTransformation(shape, is_fp4, block_k=self.block_k, block_n=self.block_n)
38| 
```
**EN:** Returns `BlackwellMX4ValueShuffledTransformation(shape, is_fp4, block_k=self.block_k, ...`.

**CN:** 返回 `BlackwellMX4ValueShuffledTransformation(shape, is_fp4, block_k=self.block_k, ...`.

### Block 10 — Lines 39-39 (swizzle_block_shape)
```python
39|     def swizzle_block_shape(self, block_shape):
```
**EN:** Defines function `swizzle_block_shape(self, block_shape)` for this module. The body mainly branches on runtime conditions; prepares intermediate values; branches on runtime conditions. It uses calls such as `len`, `ValueError` to implement its workflow.

**CN:** 定义函数 `swizzle_block_shape(self, block_shape)`，供本模块使用. 主体主要根据运行时条件分支; 准备中间值; 根据运行时条件分支. 其中会调用 `len`, `ValueError` 来实现其工作流程.

### Block 11 — Lines 40-46 (swizzle_block_shape)
```python
40|         """
41|         Convert block shape for TMA descriptor.
42| 
43|         Logical block shape is [1, block_k, block_n]. For this layout we want
44|         TMA to load [1, 1, 1, tile_n, packed_block_k] from the shuffled buffer.
45|         This matches the inner dimensions of our 5D layout.
46|         """
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 12 — Lines 47-48 (swizzle_block_shape)
```python
47|         if len(block_shape) != 3:
48|             raise ValueError(f"Expected 3D block_shape, got {len(block_shape)}D: {block_shape}")
```
**EN:** Checks `len(block_shape) != 3`..

**CN:** 检查 `len(block_shape) != 3`..

### Block 13 — Lines 49-49 (swizzle_block_shape)
```python
49|         _, block_k, block_n = block_shape
```
**EN:** Assigns `_`, `block_k`, `block_n` and references `block_shape`.

**CN:** 将 `_`, `block_k`, `block_n` and 引用 `block_shape`.

### Block 14 — Lines 50-52 (swizzle_block_shape)
```python
50|         if block_k != self.block_k:
51|             raise ValueError(f"block_k={block_k} does not match layout block_k={self.block_k}")
52|         # Return block_k un-halved; make_dense_tma will halve it for FP4 packing
```
**EN:** Checks `block_k != self.block_k`..

**CN:** 检查 `block_k != self.block_k`..

### Block 15 — Lines 53-56 (swizzle_block_shape)
```python
53|         return [1, 1, 1, block_n, block_k]
54| 
55| 
56| # ------------------- Blackwell MX4 Value Shuffled Transformation -------------------
```
**EN:** Returns `[1, 1, 1, block_n, block_k]`.

**CN:** 返回 `[1, 1, 1, block_n, block_k]`.

### Block 16 — Lines 57-58 (BlackwellMX4ValueShuffledTransformation)
```python
57| @dataclass(frozen=True)
58| class BlackwellMX4ValueShuffledTransformation(LayoutTransformation):
```
**EN:** Defines class `BlackwellMX4ValueShuffledTransformation` inheriting from `LayoutTransformation` with decorators `dataclass(frozen=True)` to organize related behavior. Key fields include `block_k`, `block_n`. Main methods are `_compute_params`, `_canonical_to_physical`, `_physical_to_canonical`, `swizzle_data`, `unswizzle_data`.

**CN:** 定义类 `BlackwellMX4ValueShuffledTransformation`，继承自 `LayoutTransformation`，带有装饰器 `dataclass(frozen=True)`，用于组织相关行为。关键字段包括 `block_k`, `block_n`。主要方法有 `_compute_params`, `_canonical_to_physical`, `_physical_to_canonical`, `swizzle_data`, `unswizzle_data`.

### Block 17 — Lines 59-60 (BlackwellMX4ValueShuffledTransformation)
```python
59|     """Transformation for the shuffled MX4 weight layout."""
60| 
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 18 — Lines 61-61 (BlackwellMX4ValueShuffledTransformation)
```python
61|     block_k: int = 128
```
**EN:** Annotated assignment stores `block_k` and stores constant `128`.

**CN:** 带类型注解的赋值保存 `block_k` and 保存常量 `128`.

### Block 19 — Lines 62-63 (BlackwellMX4ValueShuffledTransformation)
```python
62|     block_n: int = 256
63| 
```
**EN:** Annotated assignment stores `block_n` and stores constant `256`.

**CN:** 带类型注解的赋值保存 `block_n` and 保存常量 `256`.

### Block 20 — Lines 64-64 (_compute_params)
```python
64|     def _compute_params(self, E, K_packed, N):
```
**EN:** Defines function `_compute_params(self, E, K_packed, N)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `math.gcd` to implement its workflow.

**CN:** 定义函数 `_compute_params(self, E, K_packed, N)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `math.gcd` 来实现其工作流程.

### Block 21 — Lines 65-65 (_compute_params)
```python
65|         """Compute tiling parameters from the physical shape."""
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 22 — Lines 66-66 (_compute_params)
```python
66|         packed_block_k = self.block_k // 2
```
**EN:** Assigns `packed_block_k` and evaluates `self.block_k // 2`.

**CN:** 将 `packed_block_k` and 计算 `self.block_k // 2`.

### Block 23 — Lines 67-67 (_compute_params)
```python
67|         tile_k_packed = packed_block_k
```
**EN:** Assigns `tile_k_packed` and references `packed_block_k`.

**CN:** 将 `tile_k_packed` and 引用 `packed_block_k`.

### Block 24 — Lines 68-70 (_compute_params)
```python
68|         tile_n = self.block_n
69| 
70|         # lcm(128, tile_k_packed): 128 is the TMA alignment requirement in bytes
```
**EN:** Assigns `tile_n` and references `self.block_n`.

**CN:** 将 `tile_n` and 引用 `self.block_n`.

### Block 25 — Lines 71-71 (_compute_params)
```python
71|         align_k = (128 * tile_k_packed) // math.gcd(128, tile_k_packed)
```
**EN:** Assigns `align_k` and evaluates `128 * tile_k_packed // math.gcd(128, tile_k_packed)`.

**CN:** 将 `align_k` and 计算 `128 * tile_k_packed // math.gcd(128, tile_k_packed)`.

### Block 26 — Lines 72-72 (_compute_params)
```python
72|         padded_K_packed = ((K_packed + align_k - 1) // align_k) * align_k
```
**EN:** Assigns `padded_K_packed` and evaluates `(K_packed + align_k - 1) // align_k * align_k`.

**CN:** 将 `padded_K_packed` and 计算 `(K_packed + align_k - 1) // align_k * align_k`.

### Block 27 — Lines 73-73 (_compute_params)
```python
73|         num_tiles_k = (padded_K_packed + tile_k_packed - 1) // tile_k_packed
```
**EN:** Assigns `num_tiles_k` and evaluates `(padded_K_packed + tile_k_packed - 1) // tile_k_packed`.

**CN:** 将 `num_tiles_k` and 计算 `(padded_K_packed + tile_k_packed - 1) // tile_k_packed`.

### Block 28 — Lines 74-74 (_compute_params)
```python
74|         num_tiles_n = (N + tile_n - 1) // tile_n
```
**EN:** Assigns `num_tiles_n` and evaluates `(N + tile_n - 1) // tile_n`.

**CN:** 将 `num_tiles_n` and 计算 `(N + tile_n - 1) // tile_n`.

### Block 29 — Lines 75-76 (_compute_params)
```python
75|         padded_N = num_tiles_n * tile_n
76| 
```
**EN:** Assigns `padded_N` and evaluates `num_tiles_n * tile_n`.

**CN:** 将 `padded_N` and 计算 `num_tiles_n * tile_n`.

### Block 30 — Lines 77-78 (_compute_params)
```python
77|         return tile_k_packed, tile_n, padded_K_packed, padded_N, num_tiles_k, num_tiles_n
78| 
```
**EN:** Returns `(tile_k_packed, tile_n, padded_K_packed, padded_N, num_tiles_k, num_tiles_n)`.

**CN:** 返回 `(tile_k_packed, tile_n, padded_K_packed, padded_N, num_tiles_k, num_tiles_n)`.

### Block 31 — Lines 79-79 (_canonical_to_physical)
```python
79|     def _canonical_to_physical(self, data: torch.Tensor) -> torch.Tensor:
```
**EN:** Defines function `_canonical_to_physical(self, data)` for this module. The body mainly branches on runtime conditions; checks invariants; prepares intermediate values. It uses calls such as `list`, `torch.empty`, `repack`, `ValueError`, `data.stride` to implement its workflow.

**CN:** 定义函数 `_canonical_to_physical(self, data)`，供本模块使用. 主体主要根据运行时条件分支; 检查不变量; 准备中间值. 其中会调用 `list`, `torch.empty`, `repack`, `ValueError`, `data.stride` 来实现其工作流程.

### Block 32 — Lines 80-80 (_canonical_to_physical)
```python
80|         """Repack canonical [..., K, N_packed] storage to physical [E, K_packed, N]."""
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 33 — Lines 81-82 (_canonical_to_physical)
```python
81|         if not self.is_fp4:
82|             raise ValueError("BlackwellMX4ValueShuffledLayout only supports fp4 values")
```
**EN:** Checks `not self.is_fp4`..

**CN:** 检查 `not self.is_fp4`..

### Block 34 — Lines 83-83 (_canonical_to_physical)
```python
83|         assert data.stride(-1) == 1
```
**EN:** Asserts `data.stride(-1) == 1` to enforce invariants.

**CN:** 断言 `data.stride(-1) == 1` 以确保不变量成立。

### Block 35 — Lines 84-84 (_canonical_to_physical)
```python
84|         out_shape = list(data.shape)
```
**EN:** Assigns `out_shape` and calls `list`.

**CN:** 将 `out_shape`，并调用 `list`.

### Block 36 — Lines 85-85 (_canonical_to_physical)
```python
85|         out_shape[-1] *= 2
```
**EN:** Updates `out_shape[-1]` with operator `Mult` using `2`.

**CN:** 更新 `out_shape[-1]`，使用运算符 `Mult`，并使用 `2`.

### Block 37 — Lines 86-86 (_canonical_to_physical)
```python
86|         out_shape[-2] //= 2
```
**EN:** Updates `out_shape[-2]` with operator `FloorDiv` using `2`.

**CN:** 更新 `out_shape[-2]`，使用运算符 `FloorDiv`，并使用 `2`.

### Block 38 — Lines 87-87 (_canonical_to_physical)
```python
87|         out = torch.empty(out_shape, dtype=data.dtype, device=data.device)
```
**EN:** Assigns `out` and calls `torch.empty`.

**CN:** 将 `out`，并调用 `torch.empty`.

### Block 39 — Lines 88-89 (_canonical_to_physical)
```python
88|         return repack(data, -1, -2, self.is_fp4, out=out)
89| 
```
**EN:** Returns `repack(data, -1, -2, self.is_fp4, out=out)`.

**CN:** 返回 `repack(data, -1, -2, self.is_fp4, out=out)`.

### Block 40 — Lines 90-90 (_physical_to_canonical)
```python
90|     def _physical_to_canonical(self, data: torch.Tensor) -> torch.Tensor:
```
**EN:** Defines function `_physical_to_canonical(self, data)` for this module. The body mainly branches on runtime conditions; prepares intermediate values; prepares intermediate values. It uses calls such as `list`, `torch.empty`, `repack`, `ValueError` to implement its workflow.

**CN:** 定义函数 `_physical_to_canonical(self, data)`，供本模块使用. 主体主要根据运行时条件分支; 准备中间值; 准备中间值. 其中会调用 `list`, `torch.empty`, `repack`, `ValueError` 来实现其工作流程.

### Block 41 — Lines 91-91 (_physical_to_canonical)
```python
91|         """Repack physical [E, K_packed, N] storage to canonical [..., K, N_packed]."""
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 42 — Lines 92-93 (_physical_to_canonical)
```python
92|         if not self.is_fp4:
93|             raise ValueError("BlackwellMX4ValueShuffledLayout only supports fp4 values")
```
**EN:** Checks `not self.is_fp4`..

**CN:** 检查 `not self.is_fp4`..

### Block 43 — Lines 94-94 (_physical_to_canonical)
```python
94|         out_shape = list(data.shape)
```
**EN:** Assigns `out_shape` and calls `list`.

**CN:** 将 `out_shape`，并调用 `list`.

### Block 44 — Lines 95-95 (_physical_to_canonical)
```python
95|         out_shape[-2] *= 2
```
**EN:** Updates `out_shape[-2]` with operator `Mult` using `2`.

**CN:** 更新 `out_shape[-2]`，使用运算符 `Mult`，并使用 `2`.

### Block 45 — Lines 96-96 (_physical_to_canonical)
```python
96|         out_shape[-1] //= 2
```
**EN:** Updates `out_shape[-1]` with operator `FloorDiv` using `2`.

**CN:** 更新 `out_shape[-1]`，使用运算符 `FloorDiv`，并使用 `2`.

### Block 46 — Lines 97-97 (_physical_to_canonical)
```python
97|         out = torch.empty(out_shape, dtype=data.dtype, device=data.device)
```
**EN:** Assigns `out` and calls `torch.empty`.

**CN:** 将 `out`，并调用 `torch.empty`.

### Block 47 — Lines 98-99 (_physical_to_canonical)
```python
98|         return repack(data, -2, -1, self.is_fp4, out=out)
99| 
```
**EN:** Returns `repack(data, -2, -1, self.is_fp4, out=out)`.

**CN:** 返回 `repack(data, -2, -1, self.is_fp4, out=out)`.

### Block 48 — Lines 100-100 (swizzle_data)
```python
100|     def swizzle_data(self, data: torch.Tensor) -> torch.Tensor:
```
**EN:** Defines function `swizzle_data(self, data)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `self._canonical_to_physical`, `math.prod`, `data.reshape`, `self._compute_params`, `data.transpose(1, 2).contiguous` to implement its workflow.

**CN:** 定义函数 `swizzle_data(self, data)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `self._canonical_to_physical`, `math.prod`, `data.reshape`, `self._compute_params`, `data.transpose(1, 2).contiguous` 来实现其工作流程.

### Block 49 — Lines 101-106 (swizzle_data)
```python
101|         """
102|         Convert data from canonical [..., K, N_packed] to 5D shuffled layout.
103| 
104|         Target layout: [E, num_tiles_k, num_tiles_n, tile_n, tile_k_packed]
105|         This matches the baseline TMA block shape [block_n, packed_block_k] after swapping.
106|         """
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 50 — Lines 107-107 (swizzle_data)
```python
107|         data = self._canonical_to_physical(data)
```
**EN:** Assigns `data` and calls `self._canonical_to_physical`.

**CN:** 将 `data`，并调用 `self._canonical_to_physical`.

### Block 51 — Lines 108-108 (swizzle_data)
```python
108|         leading_shape = data.shape[:-2]
```
**EN:** Assigns `leading_shape` and evaluates `data.shape[:-2]`.

**CN:** 将 `leading_shape` and 计算 `data.shape[:-2]`.

### Block 52 — Lines 109-109 (swizzle_data)
```python
109|         E = math.prod(leading_shape)
```
**EN:** Assigns `E` and calls `math.prod`.

**CN:** 将 `E`，并调用 `math.prod`.

### Block 53 — Lines 110-110 (swizzle_data)
```python
110|         K_packed, N = data.shape[-2:]
```
**EN:** Assigns `K_packed`, `N` and evaluates `data.shape[-2:]`.

**CN:** 将 `K_packed`, `N` and 计算 `data.shape[-2:]`.

### Block 54 — Lines 111-111 (swizzle_data)
```python
111|         data = data.reshape(E, K_packed, N)
```
**EN:** Assigns `data` and calls `data.reshape`.

**CN:** 将 `data`，并调用 `data.reshape`.

### Block 55 — Lines 112-115 (swizzle_data)
```python
112|         tile_k_packed, tile_n, padded_K_packed, padded_N, num_tiles_k, num_tiles_n = \
113|             self._compute_params(E, K_packed, N)
114| 
115|         # Pad to tile boundaries if needed (in original [E, K_packed, N] space)
```
**EN:** Assigns `tile_k_packed`, `tile_n`, `padded_K_packed`, `padded_N`, `num_tiles_k`, `num_tiles_n` and calls `self._compute_params`.

**CN:** 将 `tile_k_packed`, `tile_n`, `padded_K_packed`, `padded_N`, `num_tiles_k`, `num_tiles_n`，并调用 `self._compute_params`.

### Block 56 — Lines 116-121 (swizzle_data)
```python
116|         if K_packed != padded_K_packed or N != padded_N:
117|             padded = torch.zeros((E, padded_K_packed, padded_N), dtype=data.dtype, device=data.device)
118|             padded[:, :K_packed, :N] = data
119|             data = padded
120| 
121|         # Transpose to swapped view: [E, K_packed, N] -> [E, N, K_packed]
```
**EN:** Checks `K_packed != padded_K_packed or N != padded_N`. The true branch mainly prepares intermediate values; prepares intermediate values.

**CN:** 检查 `K_packed != padded_K_packed or N != padded_N`. 真分支主要准备中间值; 准备中间值.

### Block 57 — Lines 122-124 (swizzle_data)
```python
122|         data = data.transpose(1, 2).contiguous()
123| 
124|         # [E, N, K_packed] -> [E, num_tiles_n, tile_n, num_tiles_k, tile_k_packed]
```
**EN:** Assigns `data` and calls `data.transpose(1, 2).contiguous`.

**CN:** 将 `data`，并调用 `data.transpose(1, 2).contiguous`.

### Block 58 — Lines 125-129 (swizzle_data)
```python
125|         data = data.view(E, num_tiles_n, tile_n, num_tiles_k, tile_k_packed)
126| 
127|         # Permute to [E, num_tiles_k, num_tiles_n, tile_n, tile_k_packed]
128|         # This puts K tiles first (for inner loop locality) and arranges
129|         # inner dims as [tile_n, tile_k_packed] to match baseline TMA block.
```
**EN:** Assigns `data` and calls `data.view`.

**CN:** 将 `data`，并调用 `data.view`.

### Block 59 — Lines 130-130 (swizzle_data)
```python
130|         data = data.permute(0, 3, 1, 2, 4).contiguous()
```
**EN:** Assigns `data` and calls `data.permute(0, 3, 1, 2, 4).contiguous`.

**CN:** 将 `data`，并调用 `data.permute(0, 3, 1, 2, 4).contiguous`.

### Block 60 — Lines 131-132 (swizzle_data)
```python
131|         return data
132| 
```
**EN:** Returns `data`.

**CN:** 返回 `data`.

### Block 61 — Lines 133-133 (unswizzle_data)
```python
133|     def unswizzle_data(self, data: torch.Tensor) -> torch.Tensor:
```
**EN:** Defines function `unswizzle_data(self, data)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `self._compute_params`, `data.permute(0, 2, 3, 1, 4).contiguous`, `data.view`, `data.transpose(1, 2).contiguous`, `data[:, :orig_K_packed, :orig_N].cont...` to implement its workflow.

**CN:** 定义函数 `unswizzle_data(self, data)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `self._compute_params`, `data.permute(0, 2, 3, 1, 4).contiguous`, `data.view`, `data.transpose(1, 2).contiguous`, `data[:, :orig_K_packed, :orig_N].cont...` 来实现其工作流程.

### Block 62 — Lines 134-138 (unswizzle_data)
```python
134|         """
135|         Convert data from shuffled back to canonical [..., K, N_packed].
136| 
137|         Input layout: [E, num_tiles_k, num_tiles_n, tile_n, tile_k_packed]
138|         """
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 63 — Lines 139-139 (unswizzle_data)
```python
139|         E = data.shape[0]
```
**EN:** Assigns `E` and evaluates `data.shape[0]`.

**CN:** 将 `E` and 计算 `data.shape[0]`.

### Block 64 — Lines 140-141 (unswizzle_data)
```python
140|         leading_shape = self.shape[:-2]
141|         # Recover original shape from self.shape (the logical shape passed to convert_layout)
```
**EN:** Assigns `leading_shape` and evaluates `self.shape[:-2]`.

**CN:** 将 `leading_shape` and 计算 `self.shape[:-2]`.

### Block 65 — Lines 142-142 (unswizzle_data)
```python
142|         orig_K_packed = self.shape[-2] // 2 if self.is_fp4 else self.shape[-2]
```
**EN:** Assigns `orig_K_packed` and uses conditional expression `self.shape[-2] // 2 if self.is_fp4 else self.shape[-2]`.

**CN:** 将 `orig_K_packed` and 使用条件表达式 `self.shape[-2] // 2 if self.is_fp4 else self.shape[-2]`.

### Block 66 — Lines 143-143 (unswizzle_data)
```python
143|         orig_N = self.shape[-1]
```
**EN:** Assigns `orig_N` and evaluates `self.shape[-1]`.

**CN:** 将 `orig_N` and 计算 `self.shape[-1]`.

### Block 67 — Lines 144-149 (unswizzle_data)
```python
144|         tile_k_packed, tile_n, padded_K_packed, padded_N, num_tiles_k, num_tiles_n = \
145|             self._compute_params(E, orig_K_packed, orig_N)
146| 
147|         # Inverse of permute(0, 3, 1, 2, 4) is permute(0, 2, 3, 1, 4)
148|         # [E, num_tiles_k, num_tiles_n, tile_n, tile_k_packed] ->
149|         # [E, num_tiles_n, tile_n, num_tiles_k, tile_k_packed]
```
**EN:** Assigns `tile_k_packed`, `tile_n`, `padded_K_packed`, `padded_N`, `num_tiles_k`, `num_tiles_n` and calls `self._compute_params`.

**CN:** 将 `tile_k_packed`, `tile_n`, `padded_K_packed`, `padded_N`, `num_tiles_k`, `num_tiles_n`，并调用 `self._compute_params`.

### Block 68 — Lines 150-152 (unswizzle_data)
```python
150|         data = data.permute(0, 2, 3, 1, 4).contiguous()
151| 
152|         # Back to swapped view [E, padded_N, padded_K_packed]
```
**EN:** Assigns `data` and calls `data.permute(0, 2, 3, 1, 4).contiguous`.

**CN:** 将 `data`，并调用 `data.permute(0, 2, 3, 1, 4).contiguous`.

### Block 69 — Lines 153-155 (unswizzle_data)
```python
153|         data = data.view(E, padded_N, padded_K_packed)
154| 
155|         # Transpose back to physical [E, padded_K_packed, padded_N]
```
**EN:** Assigns `data` and calls `data.view`.

**CN:** 将 `data`，并调用 `data.view`.

### Block 70 — Lines 156-158 (unswizzle_data)
```python
156|         data = data.transpose(1, 2).contiguous()
157| 
158|         # Trim padding back to original shape
```
**EN:** Assigns `data` and calls `data.transpose(1, 2).contiguous`.

**CN:** 将 `data`，并调用 `data.transpose(1, 2).contiguous`.

### Block 71 — Lines 159-159 (unswizzle_data)
```python
159|         data = data[:, :orig_K_packed, :orig_N].contiguous()
```
**EN:** Assigns `data` and calls `data[:, :orig_K_packed, :orig_N].contiguous`.

**CN:** 将 `data`，并调用 `data[:, :orig_K_packed, :orig_N].contiguous`.

### Block 72 — Lines 160-160 (unswizzle_data)
```python
160|         data = self._physical_to_canonical(data)
```
**EN:** Assigns `data` and calls `self._physical_to_canonical`.

**CN:** 将 `data`，并调用 `self._physical_to_canonical`.

### Block 73 — Lines 161-162 (unswizzle_data)
```python
161|         if not leading_shape:
162|             return data.squeeze(0)
```
**EN:** Checks `not leading_shape`. The true branch mainly returns the computed result.

**CN:** 检查 `not leading_shape`. 真分支主要返回计算结果.

### Block 74 — Lines 163-163 (unswizzle_data)
```python
163|         return data.reshape(*leading_shape, data.shape[-2], data.shape[-1])
```
**EN:** Returns `data.reshape(*leading_shape, data.shape[-2], data.shape[-1])`.

**CN:** 返回 `data.reshape(*leading_shape, data.shape[-2], data.shape[-1])`.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `BlackwellMX4ValueShuffledLayout`, `BlackwellMX4ValueShuffledTransformation`.
  **CN:** 主要符号：`BlackwellMX4ValueShuffledLayout`, `BlackwellMX4ValueShuffledTransformation`。
- **EN:** PyTorch tensors are the main data container and host-side orchestration layer.
  **CN:** PyTorch 张量是主要的数据容器和主机端编排层。
- **EN:** Mixed-precision MXFP/NVFP formats and scale tensors are part of the numeric pipeline.
  **CN:** 混合精度 MXFP/NVFP 格式及其缩放张量是数值流程的一部分。
- **EN:** Custom layout objects describe how logical tensors are packed or swizzled in memory.
  **CN:** 自定义 layout 对象描述逻辑张量如何在内存中打包或重排。

## Dependencies / 依赖关系
- **EN:** External modules: `math`, `dataclasses (dataclass)`, `torch`.
  **CN:** 外部模块：`math`, `dataclasses (dataclass)`, `torch`。
- **EN:** Internal modules: `.base (Layout, LayoutTransformation)`, `.torch_utils (repack)`.
  **CN:** 内部模块：`.base (Layout, LayoutTransformation)`, `.torch_utils (repack)`。
