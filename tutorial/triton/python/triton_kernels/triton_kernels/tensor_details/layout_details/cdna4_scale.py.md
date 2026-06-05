# cdna4_scale.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/triton_kernels/tensor_details/layout_details/cdna4_scale.py`
- **Purpose / 用途:** Implementation module for cdna4 scale; it defines the runtime logic, helper types, and GPU-facing utilities used by the Triton kernels package via symbols CDNA4MXScaleLayout, CDNA4MXScaleLayoutTransformation, unswizzle_mx_scale_cdna4. / 用于 cdna4 scale 的实现模块；它定义了 Triton kernels 包使用的运行时逻辑、辅助类型以及面向 GPU 的工具；核心符号包括 CDNA4MXScaleLayout、CDNA4MXScaleLayoutTransformation、unswizzle_mx_scale_cdna4。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-10 (module)
```python
 1| import math
 2| import torch
 3| from dataclasses import dataclass
 4| import triton
 5| import triton.language as tl
 6| from .base import Layout, LayoutTransformation
 7| from .torch_utils import repack
 8| 
 9| 
10| # ------------------- CDNA4 MX Scale Layout -------------------
```
**EN:** This block imports `math`, `torch`, `dataclasses (dataclass)`, `triton`, `triton.language`, `.base (Layout, LayoutTransformation)`, `.torch_utils (repack)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `math`, `torch`, `dataclasses (dataclass)`, `triton`, `triton.language`, `.base (Layout, LayoutTransformation)`, `.torch_utils (repack)` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 11-13 (CDNA4MXScaleLayout)
```python
11| @dataclass(frozen=True)
12| class CDNA4MXScaleLayout(Layout):
13| 
```
**EN:** Defines class `CDNA4MXScaleLayout` inheriting from `Layout` with decorators `dataclass(frozen=True)` to organize related behavior. Main methods are `name`, `make_transformation`, `swizzle_block_shape`.

**CN:** 定义类 `CDNA4MXScaleLayout`，继承自 `Layout`，带有装饰器 `dataclass(frozen=True)`，用于组织相关行为。主要方法有 `name`, `make_transformation`, `swizzle_block_shape`.

### Block 3 — Lines 14-15 (name)
```python
14|     @property
15|     def name(self):
```
**EN:** Defines function `name(self)` with decorators `property` for this module. The body mainly returns the computed result.

**CN:** 定义函数 `name(self)`，带有装饰器 `property`，供本模块使用. 主体主要返回计算结果.

### Block 4 — Lines 16-17 (name)
```python
16|         return "CDNA4_SCALE"
17| 
```
**EN:** Returns `'CDNA4_SCALE'`.

**CN:** 返回 `'CDNA4_SCALE'`.

### Block 5 — Lines 18-18 (make_transformation)
```python
18|     def make_transformation(self, shape: list[int], is_fp4: bool) -> LayoutTransformation:
```
**EN:** Defines function `make_transformation(self, shape, is_fp4)` for this module. The body mainly returns the computed result. It uses calls such as `CDNA4MXScaleLayoutTransformation` to implement its workflow.

**CN:** 定义函数 `make_transformation(self, shape, is_fp4)`，供本模块使用. 主体主要返回计算结果. 其中会调用 `CDNA4MXScaleLayoutTransformation` 来实现其工作流程.

### Block 6 — Lines 19-20 (make_transformation)
```python
19|         return CDNA4MXScaleLayoutTransformation(shape, is_fp4)
20| 
```
**EN:** Returns `CDNA4MXScaleLayoutTransformation(shape, is_fp4)`.

**CN:** 返回 `CDNA4MXScaleLayoutTransformation(shape, is_fp4)`.

### Block 7 — Lines 21-21 (swizzle_block_shape)
```python
21|     def swizzle_block_shape(self, block_shape):
```
**EN:** Defines function `swizzle_block_shape(self, block_shape)` for this module. The body mainly prepares intermediate values; prepares intermediate values; returns the computed result.

**CN:** 定义函数 `swizzle_block_shape(self, block_shape)`，供本模块使用. 主体主要准备中间值; 准备中间值; 返回计算结果.

### Block 8 — Lines 22-22 (swizzle_block_shape)
```python
22|         SCALE_K = block_shape[-2]
```
**EN:** Assigns `SCALE_K` and evaluates `block_shape[-2]`.

**CN:** 将 `SCALE_K` and 计算 `block_shape[-2]`.

### Block 9 — Lines 23-23 (swizzle_block_shape)
```python
23|         N = block_shape[-1]
```
**EN:** Assigns `N` and evaluates `block_shape[-1]`.

**CN:** 将 `N` and 计算 `block_shape[-1]`.

### Block 10 — Lines 24-28 (swizzle_block_shape)
```python
24|         return block_shape[:-2] + [N // 32, SCALE_K * 32]
25| 
26| 
27| # ------------------- CDNA4 MX Scale Layout Transformation -------------------
28| 
```
**EN:** Returns `block_shape[:-2] + [N // 32, SCALE_K * 32]`.

**CN:** 返回 `block_shape[:-2] + [N // 32, SCALE_K * 32]`.

### Block 11 — Lines 29-31 (module)
```python
29| NON_K_PRESHUFFLE_BLOCK_SIZE = 32
30| 
31| 
```
**EN:** Assigns `NON_K_PRESHUFFLE_BLOCK_SIZE` and stores constant `32`.

**CN:** 将 `NON_K_PRESHUFFLE_BLOCK_SIZE` and 保存常量 `32`.

### Block 12 — Lines 32-34 (CDNA4MXScaleLayoutTransformation)
```python
32| @dataclass(frozen=True)
33| class CDNA4MXScaleLayoutTransformation(LayoutTransformation):
34| 
```
**EN:** Defines class `CDNA4MXScaleLayoutTransformation` inheriting from `LayoutTransformation` with decorators `dataclass(frozen=True)` to organize related behavior. Main methods are `__post_init__`, `swizzle_data`, `unswizzle_data`.

**CN:** 定义类 `CDNA4MXScaleLayoutTransformation`，继承自 `LayoutTransformation`，带有装饰器 `dataclass(frozen=True)`，用于组织相关行为。主要方法有 `__post_init__`, `swizzle_data`, `unswizzle_data`.

### Block 13 — Lines 35-35 (__post_init__)
```python
35|     def __post_init__(self) -> None:
```
**EN:** Defines function `__post_init__(self)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `math.prod`, `object.__setattr__`, `math.ceil` to implement its workflow.

**CN:** 定义函数 `__post_init__(self)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `math.prod`, `object.__setattr__`, `math.ceil` 来实现其工作流程.

### Block 14 — Lines 36-36 (__post_init__)
```python
36|         *leading_shape, K_SCALE, N = self.shape
```
**EN:** Assigns `leading_shape`, `K_SCALE`, `N` and references `self.shape`.

**CN:** 将 `leading_shape`, `K_SCALE`, `N` and 引用 `self.shape`.

### Block 15 — Lines 37-37 (__post_init__)
```python
37|         B = math.prod(leading_shape)
```
**EN:** Assigns `B` and calls `math.prod`.

**CN:** 将 `B`，并调用 `math.prod`.

### Block 16 — Lines 38-38 (__post_init__)
```python
38|         ALIGN_K_SCALE = 8
```
**EN:** Assigns `ALIGN_K_SCALE` and stores constant `8`.

**CN:** 将 `ALIGN_K_SCALE` and 保存常量 `8`.

### Block 17 — Lines 39-39 (__post_init__)
```python
39|         ALIGN_N = 32
```
**EN:** Assigns `ALIGN_N` and stores constant `32`.

**CN:** 将 `ALIGN_N` and 保存常量 `32`.

### Block 18 — Lines 40-40 (__post_init__)
```python
40|         K_SCALE_pad = math.ceil(K_SCALE / ALIGN_K_SCALE) * ALIGN_K_SCALE
```
**EN:** Assigns `K_SCALE_pad` and evaluates `math.ceil(K_SCALE / ALIGN_K_SCALE) * ALIGN_K_SCALE`.

**CN:** 将 `K_SCALE_pad` and 计算 `math.ceil(K_SCALE / ALIGN_K_SCALE) * ALIGN_K_SCALE`.

### Block 19 — Lines 41-41 (__post_init__)
```python
41|         N_pad = math.ceil(N / ALIGN_N) * ALIGN_N
```
**EN:** Assigns `N_pad` and evaluates `math.ceil(N / ALIGN_N) * ALIGN_N`.

**CN:** 将 `N_pad` and 计算 `math.ceil(N / ALIGN_N) * ALIGN_N`.

### Block 20 — Lines 42-42 (__post_init__)
```python
42|         object.__setattr__(self, "leading_shape", leading_shape)
```
**EN:** Calls `object.__setattr__` for side effects, registration, or validation.

**CN:** 调用 `object.__setattr__` ，用于副作用、注册或校验。

### Block 21 — Lines 43-43 (__post_init__)
```python
43|         object.__setattr__(self, "B", B)
```
**EN:** Calls `object.__setattr__` for side effects, registration, or validation.

**CN:** 调用 `object.__setattr__` ，用于副作用、注册或校验。

### Block 22 — Lines 44-44 (__post_init__)
```python
44|         object.__setattr__(self, "ALIGN_K_SCALE", ALIGN_K_SCALE)
```
**EN:** Calls `object.__setattr__` for side effects, registration, or validation.

**CN:** 调用 `object.__setattr__` ，用于副作用、注册或校验。

### Block 23 — Lines 45-45 (__post_init__)
```python
45|         object.__setattr__(self, "ALIGN_N", ALIGN_N)
```
**EN:** Calls `object.__setattr__` for side effects, registration, or validation.

**CN:** 调用 `object.__setattr__` ，用于副作用、注册或校验。

### Block 24 — Lines 46-46 (__post_init__)
```python
46|         object.__setattr__(self, "K_SCALE_pad", K_SCALE_pad)
```
**EN:** Calls `object.__setattr__` for side effects, registration, or validation.

**CN:** 调用 `object.__setattr__` ，用于副作用、注册或校验。

### Block 25 — Lines 47-47 (__post_init__)
```python
47|         object.__setattr__(self, "N_pad", N_pad)
```
**EN:** Calls `object.__setattr__` for side effects, registration, or validation.

**CN:** 调用 `object.__setattr__` ，用于副作用、注册或校验。

### Block 26 — Lines 48-48 (__post_init__)
```python
48|         object.__setattr__(self, "K_SCALE", K_SCALE)
```
**EN:** Calls `object.__setattr__` for side effects, registration, or validation.

**CN:** 调用 `object.__setattr__` ，用于副作用、注册或校验。

### Block 27 — Lines 49-50 (__post_init__)
```python
49|         object.__setattr__(self, "N", N)
50| 
```
**EN:** Calls `object.__setattr__` for side effects, registration, or validation.

**CN:** 调用 `object.__setattr__` ，用于副作用、注册或校验。

### Block 28 — Lines 51-51 (swizzle_data)
```python
51|     def swizzle_data(self, data):
```
**EN:** Defines function `swizzle_data(self, data)` for this module. The body mainly checks invariants; prepares intermediate values; prepares intermediate values. It uses calls such as `repack`, `torch.nn.functional.pad`, `data.transpose`, `data.view`, `data.permute(0, 1, 4, 6, 3, 5, 2, 7)....` to implement its workflow.

**CN:** 定义函数 `swizzle_data(self, data)`，供本模块使用. 主体主要检查不变量; 准备中间值; 准备中间值. 其中会调用 `repack`, `torch.nn.functional.pad`, `data.transpose`, `data.view`, `data.permute(0, 1, 4, 6, 3, 5, 2, 7)....` 来实现其工作流程.

### Block 29 — Lines 52-53 (swizzle_data)
```python
52|         assert data.stride(-1) == 1
53|         # re-pack as column-major
```
**EN:** Asserts `data.stride(-1) == 1` to enforce invariants.

**CN:** 断言 `data.stride(-1) == 1` 以确保不变量成立。

### Block 30 — Lines 54-54 (swizzle_data)
```python
54|         data = repack(data, -1, -2, self.is_fp4)
```
**EN:** Assigns `data` and calls `repack`.

**CN:** 将 `data`，并调用 `repack`.

### Block 31 — Lines 55-55 (swizzle_data)
```python
55|         data = data.mT.contiguous().mT
```
**EN:** Assigns `data` and references `data.mT.contiguous().mT`.

**CN:** 将 `data` and 引用 `data.mT.contiguous().mT`.

### Block 32 — Lines 56-56 (swizzle_data)
```python
56|         data = torch.nn.functional.pad(data, (0, self.N_pad - self.N, 0, self.K_SCALE_pad - self.K_SCALE))
```
**EN:** Assigns `data` and calls `torch.nn.functional.pad`.

**CN:** 将 `data`，并调用 `torch.nn.functional.pad`.

### Block 33 — Lines 57-57 (swizzle_data)
```python
57|         data = data.transpose(-1, -2)
```
**EN:** Assigns `data` and calls `data.transpose`.

**CN:** 将 `data`，并调用 `data.transpose`.

### Block 34 — Lines 58-58 (swizzle_data)
```python
58|         data = data.view(-1, self.N_pad // NON_K_PRESHUFFLE_BLOCK_SIZE, 2, 16, self.K_SCALE_pad // 8, 2, 4, 1)
```
**EN:** Assigns `data` and calls `data.view`.

**CN:** 将 `data`，并调用 `data.view`.

### Block 35 — Lines 59-59 (swizzle_data)
```python
59|         data = data.permute(0, 1, 4, 6, 3, 5, 2, 7).contiguous()
```
**EN:** Assigns `data` and calls `data.permute(0, 1, 4, 6, 3, 5, 2, 7).contiguous`.

**CN:** 将 `data`，并调用 `data.permute(0, 1, 4, 6, 3, 5, 2, 7).contiguous`.

### Block 36 — Lines 60-60 (swizzle_data)
```python
60|         data = data.reshape(self.B, self.N_pad // 32, self.K_SCALE_pad * 32)
```
**EN:** Assigns `data` and calls `data.reshape`.

**CN:** 将 `data`，并调用 `data.reshape`.

### Block 37 — Lines 61-61 (swizzle_data)
```python
61|         data = data.transpose(-1, -2)
```
**EN:** Assigns `data` and calls `data.transpose`.

**CN:** 将 `data`，并调用 `data.transpose`.

### Block 38 — Lines 62-62 (swizzle_data)
```python
62|         assert data.stride(-2) == 1
```
**EN:** Asserts `data.stride(-2) == 1` to enforce invariants.

**CN:** 断言 `data.stride(-2) == 1` 以确保不变量成立。

### Block 39 — Lines 63-64 (swizzle_data)
```python
63|         return data
64| 
```
**EN:** Returns `data`.

**CN:** 返回 `data`.

### Block 40 — Lines 65-65 (unswizzle_data)
```python
65|     def unswizzle_data(self, data):
```
**EN:** Defines function `unswizzle_data(self, data)` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `data.transpose`, `data.view`, `data.permute`, `data.reshape`, `repack` to implement its workflow.

**CN:** 定义函数 `unswizzle_data(self, data)`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `data.transpose`, `data.view`, `data.permute`, `data.reshape`, `repack` 来实现其工作流程.

### Block 41 — Lines 66-66 (unswizzle_data)
```python
66|         data = data.transpose(-1, -2)
```
**EN:** Assigns `data` and calls `data.transpose`.

**CN:** 将 `data`，并调用 `data.transpose`.

### Block 42 — Lines 67-67 (unswizzle_data)
```python
67|         data = data.view(-1, self.N_pad // NON_K_PRESHUFFLE_BLOCK_SIZE, self.K_SCALE_pad // 8, 4, 16, 2, 2, 1)
```
**EN:** Assigns `data` and calls `data.view`.

**CN:** 将 `data`，并调用 `data.view`.

### Block 43 — Lines 68-68 (unswizzle_data)
```python
68|         data = data.permute(0, 1, 6, 4, 2, 5, 3, 7)
```
**EN:** Assigns `data` and calls `data.permute`.

**CN:** 将 `data`，并调用 `data.permute`.

### Block 44 — Lines 69-69 (unswizzle_data)
```python
69|         data = data.reshape(*self.leading_shape, self.N_pad, self.K_SCALE_pad)
```
**EN:** Assigns `data` and calls `data.reshape`.

**CN:** 将 `data`，并调用 `data.reshape`.

### Block 45 — Lines 70-70 (unswizzle_data)
```python
70|         data = data.transpose(-1, -2)[..., :self.K_SCALE, :self.N]
```
**EN:** Assigns `data` and evaluates `data.transpose(-1, -2)[..., :self.K_SCALE, :self.N]`.

**CN:** 将 `data` and 计算 `data.transpose(-1, -2)[..., :self.K_SCALE, :self.N]`.

### Block 46 — Lines 71-71 (unswizzle_data)
```python
71|         data = repack(data, -2, -1, self.is_fp4)
```
**EN:** Assigns `data` and calls `repack`.

**CN:** 将 `data`，并调用 `repack`.

### Block 47 — Lines 72-72 (unswizzle_data)
```python
72|         data = data.contiguous()
```
**EN:** Assigns `data` and calls `data.contiguous`.

**CN:** 将 `data`，并调用 `data.contiguous`.

### Block 48 — Lines 73-73 (unswizzle_data)
```python
73|         assert data.stride(-1) == 1
```
**EN:** Asserts `data.stride(-1) == 1` to enforce invariants.

**CN:** 断言 `data.stride(-1) == 1` 以确保不变量成立。

### Block 49 — Lines 74-76 (unswizzle_data)
```python
74|         return data
75| 
76| 
```
**EN:** Returns `data`.

**CN:** 返回 `data`.

### Block 50 — Lines 77-79 (unswizzle_mx_scale_cdna4)
```python
77| @triton.jit
78| def unswizzle_mx_scale_cdna4(x, BLOCK_N: tl.constexpr, MX_SCALE_BLOCK_K: tl.constexpr,
79|                              N_PRESHUFFLE_FACTOR: tl.constexpr = NON_K_PRESHUFFLE_BLOCK_SIZE):
```
**EN:** Defines function `unswizzle_mx_scale_cdna4(x, BLOCK_N, MX_SCALE_BLOCK_K, N_PRESHUFFLE_FACTOR)` with decorators `triton.jit` for this module. The body mainly prepares intermediate values; prepares intermediate values; prepares intermediate values. It uses calls such as `x.reshape`, `x.permute` to implement its workflow.

**CN:** 定义函数 `unswizzle_mx_scale_cdna4(x, BLOCK_N, MX_SCALE_BLOCK_K, N_PRESHUFFLE_FACTOR)`，带有装饰器 `triton.jit`，供本模块使用. 主体主要准备中间值; 准备中间值; 准备中间值. 其中会调用 `x.reshape`, `x.permute` 来实现其工作流程.

### Block 51 — Lines 80-80 (unswizzle_mx_scale_cdna4)
```python
80|     x = x.reshape(BLOCK_N // N_PRESHUFFLE_FACTOR, MX_SCALE_BLOCK_K // 8, 4, 16, 2, 2, 1)
```
**EN:** Assigns `x` and calls `x.reshape`.

**CN:** 将 `x`，并调用 `x.reshape`.

### Block 52 — Lines 81-81 (unswizzle_mx_scale_cdna4)
```python
81|     x = x.permute(0, 5, 3, 1, 4, 2, 6)
```
**EN:** Assigns `x` and calls `x.permute`.

**CN:** 将 `x`，并调用 `x.permute`.

### Block 53 — Lines 82-82 (unswizzle_mx_scale_cdna4)
```python
82|     x = x.reshape(BLOCK_N, MX_SCALE_BLOCK_K)
```
**EN:** Assigns `x` and calls `x.reshape`.

**CN:** 将 `x`，并调用 `x.reshape`.

### Block 54 — Lines 83-83 (unswizzle_mx_scale_cdna4)
```python
83|     return x
```
**EN:** Returns `x`.

**CN:** 返回 `x`.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `CDNA4MXScaleLayout`, `CDNA4MXScaleLayoutTransformation`, `unswizzle_mx_scale_cdna4`.
  **CN:** 主要符号：`CDNA4MXScaleLayout`, `CDNA4MXScaleLayoutTransformation`, `unswizzle_mx_scale_cdna4`。
- **EN:** PyTorch tensors are the main data container and host-side orchestration layer.
  **CN:** PyTorch 张量是主要的数据容器和主机端编排层。
- **EN:** Triton is used to JIT-compile or launch GPU kernels and to express tile-level execution details.
  **CN:** Triton 用于 JIT 编译或启动 GPU 内核，并表达分块级执行细节。
- **EN:** Custom layout objects describe how logical tensors are packed or swizzled in memory.
  **CN:** 自定义 layout 对象描述逻辑张量如何在内存中打包或重排。

## Dependencies / 依赖关系
- **EN:** External modules: `math`, `torch`, `dataclasses (dataclass)`, `triton`, `triton.language`.
  **CN:** 外部模块：`math`, `torch`, `dataclasses (dataclass)`, `triton`, `triton.language`。
- **EN:** Internal modules: `.base (Layout, LayoutTransformation)`, `.torch_utils (repack)`.
  **CN:** 内部模块：`.base (Layout, LayoutTransformation)`, `.torch_utils (repack)`。
