# layout.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/triton_kernels/tensor_details/layout.py`
- **Purpose / 用途:** Implementation module for layout; it defines the runtime logic, helper types, and GPU-facing utilities used by the Triton kernels package via symbols make_default_matmul_mxfp4_w_layout, make_default_matmul_mxfp4_w_scale_layout, make_default_matmul_mx_act_scale_layout, make_default_matmul_mxfp8_act_scale_layout. / 用于 layout 的实现模块；它定义了 Triton kernels 包使用的运行时逻辑、辅助类型以及面向 GPU 的工具；核心符号包括 make_default_matmul_mxfp4_w_layout、make_default_matmul_mxfp4_w_scale_layout、make_default_matmul_mx_act_scale_layout、make_default_matmul_mxfp8_act_scale_layout。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-12 (module)
```python
 1| from .layout_details.base import Layout
 2| from .layout_details.blackwell_scale import BlackwellMXScaleLayout
 3| from .layout_details.blackwell_scale import BlackwellActMXScaleLayout
 4| from .layout_details.blackwell_value import BlackwellMXValueLayout
 5| from .layout_details.blackwell_value_shuffled import BlackwellMX4ValueShuffledLayout
 6| from .layout_details.hopper_scale import HopperMXScaleLayout
 7| from .layout_details.hopper_value import HopperMXValueLayout
 8| from .layout_details.cdna4_scale import CDNA4MXScaleLayout
 9| from .layout_details.gfx1250_scale import GFX1250MXScaleLayout
10| from .layout_details.strided import StridedLayout
11| from ..target_info import cuda_capability_geq, is_hip_cdna4, is_hip_gfx1250
12| 
```
**EN:** This block imports `.layout_details.base (Layout)`, `.layout_details.blackwell_scale (BlackwellMXScaleLayout)`, `.layout_details.blackwell_scale (BlackwellActMXScaleLayout)`, `.layout_details.blackwell_value (BlackwellMXValueLayout)`, `.layout_details.blackwell_value_shuffled (BlackwellMX4ValueShuffledLayout)`, `.layout_details.hopper_scale (HopperMXScaleLayout)`, `.layout_details.hopper_value (HopperMXValueLayout)`, `.layout_details.cdna4_scale (CDNA4MXScaleLayout)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `.layout_details.base (Layout)`, `.layout_details.blackwell_scale (BlackwellMXScaleLayout)`, `.layout_details.blackwell_scale (BlackwellActMXScaleLayout)`, `.layout_details.blackwell_value (BlackwellMXValueLayout)`, `.layout_details.blackwell_value_shuffled (BlackwellMX4ValueShuffledLayout)`, `.layout_details.hopper_scale (HopperMXScaleLayout)`, `.layout_details.hopper_value (HopperMXValueLayout)`, `.layout_details.cdna4_scale (CDNA4MXScaleLayout)` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 13-26 (module)
```python
13| __all__ = [
14|     "Layout",
15|     "BlackwellMXValueLayout",
16|     "BlackwellMX4ValueShuffledLayout",
17|     "BlackwellMXScaleLayout",
18|     "HopperMXScaleLayout",
19|     "HopperMXValueLayout",
20|     "CDNA4MXScaleLayout",
21|     "GFX1250MXScaleLayout",
22|     "StridedLayout",
23|     "BlackwellActMXScaleLayout",
24| ]
25| 
26| 
```
**EN:** Assigns `__all__` and builds a list.

**CN:** 将 `__all__` and 构造一个列表.

### Block 3 — Lines 27-32 (make_default_matmul_mxfp4_w_layout)
```python
27| def make_default_matmul_mxfp4_w_layout(
28|     mx_axis: int,
29|     allow_blackwell_value_shuffle: bool = False,
30|     block_k: int = 128,
31|     block_n: int = 256,
32| ):
```
**EN:** Defines function `make_default_matmul_mxfp4_w_layout(mx_axis, allow_blackwell_value_shuffle, block_k, block_n)` for this module. The body mainly branches on runtime conditions. It uses calls such as `cuda_capability_geq`, `BlackwellMXValueLayout`, `BlackwellMX4ValueShuffledLayout`, `HopperMXValueLayout`, `StridedLayout` to implement its workflow.

**CN:** 定义函数 `make_default_matmul_mxfp4_w_layout(mx_axis, allow_blackwell_value_shuffle, block_k, block_n)`，供本模块使用. 主体主要根据运行时条件分支. 其中会调用 `cuda_capability_geq`, `BlackwellMXValueLayout`, `BlackwellMX4ValueShuffledLayout`, `HopperMXValueLayout`, `StridedLayout` 来实现其工作流程.

### Block 4 — Lines 33-42 (make_default_matmul_mxfp4_w_layout)
```python
33|     if cuda_capability_geq(10):
34|         if allow_blackwell_value_shuffle:
35|             return BlackwellMX4ValueShuffledLayout(block_k=block_k, block_n=block_n)
36|         return BlackwellMXValueLayout()
37|     elif cuda_capability_geq(9):
38|         return HopperMXValueLayout(mx_axis=mx_axis, mma_version=3)
39|     else:
40|         return StridedLayout(-2)
41| 
42| 
```
**EN:** Checks `cuda_capability_geq(10)`. The true branch mainly branches on runtime conditions; returns the computed result, while the else branch branches on runtime conditions.

**CN:** 检查 `cuda_capability_geq(10)`. 真分支主要根据运行时条件分支; 返回计算结果；而 else 分支根据运行时条件分支.

### Block 5 — Lines 43-43 (make_default_matmul_mxfp4_w_scale_layout)
```python
43| def make_default_matmul_mxfp4_w_scale_layout(mx_axis: int, num_warps: int = 8):
```
**EN:** Defines function `make_default_matmul_mxfp4_w_scale_layout(mx_axis, num_warps)` for this module. The body mainly branches on runtime conditions; returns the computed result. It uses calls such as `is_hip_cdna4`, `StridedLayout`, `CDNA4MXScaleLayout`, `is_hip_gfx1250`, `GFX1250MXScaleLayout` to implement its workflow.

**CN:** 定义函数 `make_default_matmul_mxfp4_w_scale_layout(mx_axis, num_warps)`，供本模块使用. 主体主要根据运行时条件分支; 返回计算结果. 其中会调用 `is_hip_cdna4`, `StridedLayout`, `CDNA4MXScaleLayout`, `is_hip_gfx1250`, `GFX1250MXScaleLayout` 来实现其工作流程.

### Block 6 — Lines 44-53 (make_default_matmul_mxfp4_w_scale_layout)
```python
44|     if is_hip_cdna4():
45|         return CDNA4MXScaleLayout()
46|     elif is_hip_gfx1250():
47|         return GFX1250MXScaleLayout()
48|     else:
49|         if cuda_capability_geq(10):
50|             return BlackwellMXScaleLayout()
51|         elif cuda_capability_geq(9):
52|             return HopperMXScaleLayout(mx_axis=mx_axis, num_warps=num_warps)
53| 
```
**EN:** Checks `is_hip_cdna4()`. The true branch mainly returns the computed result, while the else branch branches on runtime conditions.

**CN:** 检查 `is_hip_cdna4()`. 真分支主要返回计算结果；而 else 分支根据运行时条件分支.

### Block 7 — Lines 54-56 (make_default_matmul_mxfp4_w_scale_layout)
```python
54|     return StridedLayout(-2)
55| 
56| 
```
**EN:** Returns `StridedLayout(-2)`.

**CN:** 返回 `StridedLayout(-2)`.

### Block 8 — Lines 57-57 (make_default_matmul_mx_act_scale_layout)
```python
57| def make_default_matmul_mx_act_scale_layout(ragged_metadata):
```
**EN:** Defines function `make_default_matmul_mx_act_scale_layout(ragged_metadata)` for this module. The body mainly branches on runtime conditions; returns the computed result. It uses calls such as `cuda_capability_geq`, `StridedLayout`, `BlackwellActMXScaleLayout` to implement its workflow.

**CN:** 定义函数 `make_default_matmul_mx_act_scale_layout(ragged_metadata)`，供本模块使用. 主体主要根据运行时条件分支; 返回计算结果. 其中会调用 `cuda_capability_geq`, `StridedLayout`, `BlackwellActMXScaleLayout` 来实现其工作流程.

### Block 9 — Lines 58-59 (make_default_matmul_mx_act_scale_layout)
```python
58|     if cuda_capability_geq(10):
59|         return BlackwellActMXScaleLayout(ragged_metadata)
```
**EN:** Checks `cuda_capability_geq(10)`. The true branch mainly returns the computed result.

**CN:** 检查 `cuda_capability_geq(10)`. 真分支主要返回计算结果.

### Block 10 — Lines 60-62 (make_default_matmul_mx_act_scale_layout)
```python
60|     return StridedLayout(-2)
61| 
62| 
```
**EN:** Returns `StridedLayout(-2)`.

**CN:** 返回 `StridedLayout(-2)`.

### Block 11 — Lines 63-63 (make_default_matmul_mxfp8_act_scale_layout)
```python
63| def make_default_matmul_mxfp8_act_scale_layout(ragged_metadata):
```
**EN:** Defines function `make_default_matmul_mxfp8_act_scale_layout(ragged_metadata)` for this module. The body mainly returns the computed result. It uses calls such as `make_default_matmul_mx_act_scale_layout` to implement its workflow.

**CN:** 定义函数 `make_default_matmul_mxfp8_act_scale_layout(ragged_metadata)`，供本模块使用. 主体主要返回计算结果. 其中会调用 `make_default_matmul_mx_act_scale_layout` 来实现其工作流程.

### Block 12 — Lines 64-64 (make_default_matmul_mxfp8_act_scale_layout)
```python
64|     return make_default_matmul_mx_act_scale_layout(ragged_metadata)
```
**EN:** Returns `make_default_matmul_mx_act_scale_layout(ragged_metadata)`.

**CN:** 返回 `make_default_matmul_mx_act_scale_layout(ragged_metadata)`.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `make_default_matmul_mxfp4_w_layout`, `make_default_matmul_mxfp4_w_scale_layout`, `make_default_matmul_mx_act_scale_layout`, `make_default_matmul_mxfp8_act_scale_layout`.
  **CN:** 主要符号：`make_default_matmul_mxfp4_w_layout`, `make_default_matmul_mxfp4_w_scale_layout`, `make_default_matmul_mx_act_scale_layout`, `make_default_matmul_mxfp8_act_scale_layout`。
- **EN:** Mixed-precision MXFP/NVFP formats and scale tensors are part of the numeric pipeline.
  **CN:** 混合精度 MXFP/NVFP 格式及其缩放张量是数值流程的一部分。
- **EN:** Custom layout objects describe how logical tensors are packed or swizzled in memory.
  **CN:** 自定义 layout 对象描述逻辑张量如何在内存中打包或重排。
- **EN:** Ragged tensor metadata is used when rows or slices have non-uniform sizes.
  **CN:** 当行或切片大小不一致时，会使用 ragged tensor 元数据。

## Dependencies / 依赖关系
- **EN:** External modules: none at the top level.
  **CN:** 顶层外部模块：无。
- **EN:** Internal modules: `.layout_details.base (Layout)`, `.layout_details.blackwell_scale (BlackwellMXScaleLayout)`, `.layout_details.blackwell_scale (BlackwellActMXScaleLayout)`, `.layout_details.blackwell_value (BlackwellMXValueLayout)`, `.layout_details.blackwell_value_shuffled (BlackwellMX4ValueShuffledLayout)`, `.layout_details.hopper_scale (HopperMXScaleLayout)`, `.layout_details.hopper_value (HopperMXValueLayout)`, `.layout_details.cdna4_scale (CDNA4MXScaleLayout)`, `.layout_details.gfx1250_scale (GFX1250MXScaleLayout)`, `.layout_details.strided (StridedLayout)`, `..target_info (cuda_capability_geq, is_hip_cdna4, is_hip_gfx1250)`.
  **CN:** 内部模块：`.layout_details.base (Layout)`, `.layout_details.blackwell_scale (BlackwellMXScaleLayout)`, `.layout_details.blackwell_scale (BlackwellActMXScaleLayout)`, `.layout_details.blackwell_value (BlackwellMXValueLayout)`, `.layout_details.blackwell_value_shuffled (BlackwellMX4ValueShuffledLayout)`, `.layout_details.hopper_scale (HopperMXScaleLayout)`, `.layout_details.hopper_value (HopperMXValueLayout)`, `.layout_details.cdna4_scale (CDNA4MXScaleLayout)`, `.layout_details.gfx1250_scale (GFX1250MXScaleLayout)`, `.layout_details.strided (StridedLayout)`, `..target_info (cuda_capability_geq, is_hip_cdna4, is_hip_gfx1250)`。
