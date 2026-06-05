# target_info.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./python/triton_kernels/triton_kernels/target_info.py`
- **Purpose / 用途:** Implementation module for target info; it defines the runtime logic, helper types, and GPU-facing utilities used by the Triton kernels package via symbols get_cdna_version, get_rdna_version, has_tma_gather, has_native_mxfp, num_sms. / 用于 target info 的实现模块；它定义了 Triton kernels 包使用的运行时逻辑、辅助类型以及面向 GPU 的工具；核心符号包括 get_cdna_version、get_rdna_version、has_tma_gather、has_native_mxfp、num_sms。

## Line-by-Line Analysis / 逐行分析
### Block 1 — Lines 1-13 (module)
```python
 1| import torch
 2| import triton
 3| import triton.language as tl
 4| 
 5| from triton.language.target_info import (
 6|     cuda_capability_geq,
 7|     is_cuda,
 8|     is_hip,
 9|     is_hip_cdna3,
10|     is_hip_cdna4,
11|     is_hip_gfx1250,
12| )
13| 
```
**EN:** This block imports `torch`, `triton`, `triton.language`, `triton.language.target_info (cuda_capability_geq, is_cuda, is_hip, is_hip_cdna3)` to provide the external libraries and internal helpers used by the module.

**CN:** 该代码块导入 `torch`, `triton`, `triton.language`, `triton.language.target_info (cuda_capability_geq, is_cuda, is_hip, is_hip_cdna3)` ，为模块提供所需的外部库和内部辅助工具。

### Block 2 — Lines 14-28 (module)
```python
14| __all__ = [
15|     "cuda_capability_geq",
16|     "get_cdna_version",
17|     "get_rdna_version",
18|     "has_tma_gather",
19|     "has_native_mxfp",
20|     "is_cuda",
21|     "is_hip",
22|     "is_hip_cdna3",
23|     "is_hip_cdna4",
24|     "is_hip_gfx1250",
25|     "num_sms",
26| ]
27| 
28| 
```
**EN:** Assigns `__all__` and builds a list.

**CN:** 将 `__all__` and 构造一个列表.

### Block 3 — Lines 29-30 (get_cdna_version)
```python
29| @triton.constexpr_function
30| def get_cdna_version():
```
**EN:** Defines function `get_cdna_version()` with decorators `triton.constexpr_function` for this module. The body mainly prepares intermediate values; branches on runtime conditions; branches on runtime conditions. It uses calls such as `tl.target_info.current_target` to implement its workflow.

**CN:** 定义函数 `get_cdna_version()`，带有装饰器 `triton.constexpr_function`，供本模块使用. 主体主要准备中间值; 根据运行时条件分支; 根据运行时条件分支. 其中会调用 `tl.target_info.current_target` 来实现其工作流程.

### Block 4 — Lines 31-35 (get_cdna_version)
```python
31|     """
32|     Gets the AMD architecture version, i.e. CDNA3 or CDNA4, currently
33|     only supports 3 (gfx942) or 4 (gfx950). Returns -1 if it is not AMD
34|     hardware or unsupported architecture
35|     """
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 5 — Lines 36-36 (get_cdna_version)
```python
36|     target = tl.target_info.current_target()
```
**EN:** Assigns `target` and calls `tl.target_info.current_target`.

**CN:** 将 `target`，并调用 `tl.target_info.current_target`.

### Block 6 — Lines 37-38 (get_cdna_version)
```python
37|     if target.backend != 'hip':
38|         return -1
```
**EN:** Checks `target.backend != 'hip'`. The true branch mainly returns the computed result.

**CN:** 检查 `target.backend != 'hip'`. 真分支主要返回计算结果.

### Block 7 — Lines 39-40 (get_cdna_version)
```python
39|     if target.arch == 'gfx942':
40|         return 3
```
**EN:** Checks `target.arch == 'gfx942'`. The true branch mainly returns the computed result.

**CN:** 检查 `target.arch == 'gfx942'`. 真分支主要返回计算结果.

### Block 8 — Lines 41-42 (get_cdna_version)
```python
41|     if target.arch == 'gfx950':
42|         return 4
```
**EN:** Checks `target.arch == 'gfx950'`. The true branch mainly returns the computed result.

**CN:** 检查 `target.arch == 'gfx950'`. 真分支主要返回计算结果.

### Block 9 — Lines 43-45 (get_cdna_version)
```python
43|     return -1
44| 
45| 
```
**EN:** Returns `-1`.

**CN:** 返回 `-1`.

### Block 10 — Lines 46-47 (get_rdna_version)
```python
46| @triton.constexpr_function
47| def get_rdna_version():
```
**EN:** Defines function `get_rdna_version()` with decorators `triton.constexpr_function` for this module. The body mainly prepares intermediate values; branches on runtime conditions; branches on runtime conditions. It uses calls such as `tl.target_info.current_target`, `target.arch.startswith` to implement its workflow.

**CN:** 定义函数 `get_rdna_version()`，带有装饰器 `triton.constexpr_function`，供本模块使用. 主体主要准备中间值; 根据运行时条件分支; 根据运行时条件分支. 其中会调用 `tl.target_info.current_target`, `target.arch.startswith` 来实现其工作流程.

### Block 11 — Lines 48-52 (get_rdna_version)
```python
48|     """
49|     Gets the AMD architecture version, i.e. RDNA3 or RDNA4, by matching
50|     gfx11* (RDNA3) or gfx12* (RDNA4). Returns -1 if it is not AMD
51|     hardware or unsupported architecture.
52|     """
```
**EN:** Stores a documentation string that explains the surrounding API or block.

**CN:** 保存一个用于解释周围 API 或代码块的文档字符串。

### Block 12 — Lines 53-53 (get_rdna_version)
```python
53|     target = tl.target_info.current_target()
```
**EN:** Assigns `target` and calls `tl.target_info.current_target`.

**CN:** 将 `target`，并调用 `tl.target_info.current_target`.

### Block 13 — Lines 54-55 (get_rdna_version)
```python
54|     if target.backend != 'hip':
55|         return -1
```
**EN:** Checks `target.backend != 'hip'`. The true branch mainly returns the computed result.

**CN:** 检查 `target.backend != 'hip'`. 真分支主要返回计算结果.

### Block 14 — Lines 56-57 (get_rdna_version)
```python
56|     if target.arch.startswith('gfx11'):
57|         return 3
```
**EN:** Checks `target.arch.startswith('gfx11')`. The true branch mainly returns the computed result.

**CN:** 检查 `target.arch.startswith('gfx11')`. 真分支主要返回计算结果.

### Block 15 — Lines 58-59 (get_rdna_version)
```python
58|     if target.arch.startswith('gfx12') and not target.arch.startswith('gfx125'):
59|         return 4
```
**EN:** Checks `target.arch.startswith('gfx12') and (not target.arch.startswith('gfx125'))`. The true branch mainly returns the computed result.

**CN:** 检查 `target.arch.startswith('gfx12') and (not target.arch.startswith('gfx125'))`. 真分支主要返回计算结果.

### Block 16 — Lines 60-62 (get_rdna_version)
```python
60|     return -1
61| 
62| 
```
**EN:** Returns `-1`.

**CN:** 返回 `-1`.

### Block 17 — Lines 63-64 (has_tma_gather)
```python
63| @triton.constexpr_function
64| def has_tma_gather():
```
**EN:** Defines function `has_tma_gather()` with decorators `triton.constexpr_function` for this module. The body mainly returns the computed result. It uses calls such as `cuda_capability_geq` to implement its workflow.

**CN:** 定义函数 `has_tma_gather()`，带有装饰器 `triton.constexpr_function`，供本模块使用. 主体主要返回计算结果. 其中会调用 `cuda_capability_geq` 来实现其工作流程.

### Block 18 — Lines 65-67 (has_tma_gather)
```python
65|     return cuda_capability_geq(10, 0)
66| 
67| 
```
**EN:** Returns `cuda_capability_geq(10, 0)`.

**CN:** 返回 `cuda_capability_geq(10, 0)`.

### Block 19 — Lines 68-69 (has_native_mxfp)
```python
68| @triton.constexpr_function
69| def has_native_mxfp():
```
**EN:** Defines function `has_native_mxfp()` with decorators `triton.constexpr_function` for this module. The body mainly returns the computed result. It uses calls such as `cuda_capability_geq` to implement its workflow.

**CN:** 定义函数 `has_native_mxfp()`，带有装饰器 `triton.constexpr_function`，供本模块使用. 主体主要返回计算结果. 其中会调用 `cuda_capability_geq` 来实现其工作流程.

### Block 20 — Lines 70-72 (has_native_mxfp)
```python
70|     return cuda_capability_geq(10, 0)
71| 
72| 
```
**EN:** Returns `cuda_capability_geq(10, 0)`.

**CN:** 返回 `cuda_capability_geq(10, 0)`.

### Block 21 — Lines 73-73 (num_sms)
```python
73| def num_sms():
```
**EN:** Defines function `num_sms()` for this module. The body mainly returns the computed result. It uses calls such as `torch.cuda.get_device_properties` to implement its workflow.

**CN:** 定义函数 `num_sms()`，供本模块使用. 主体主要返回计算结果. 其中会调用 `torch.cuda.get_device_properties` 来实现其工作流程.

### Block 22 — Lines 74-74 (num_sms)
```python
74|     return torch.cuda.get_device_properties(0).multi_processor_count
```
**EN:** Returns `torch.cuda.get_device_properties(0).multi_processor_count`.

**CN:** 返回 `torch.cuda.get_device_properties(0).multi_processor_count`.

## Key Concepts / 关键概念
- **EN:** Primary symbols: `get_cdna_version`, `get_rdna_version`, `has_tma_gather`, `has_native_mxfp`, `num_sms`.
  **CN:** 主要符号：`get_cdna_version`, `get_rdna_version`, `has_tma_gather`, `has_native_mxfp`, `num_sms`。
- **EN:** PyTorch tensors are the main data container and host-side orchestration layer.
  **CN:** PyTorch 张量是主要的数据容器和主机端编排层。
- **EN:** Triton is used to JIT-compile or launch GPU kernels and to express tile-level execution details.
  **CN:** Triton 用于 JIT 编译或启动 GPU 内核，并表达分块级执行细节。
- **EN:** Mixed-precision MXFP/NVFP formats and scale tensors are part of the numeric pipeline.
  **CN:** 混合精度 MXFP/NVFP 格式及其缩放张量是数值流程的一部分。

## Dependencies / 依赖关系
- **EN:** External modules: `torch`, `triton`, `triton.language`, `triton.language.target_info (cuda_capability_geq, is_cuda, is_hip, is_hip_cdna3, is_hip_cdna4, is_hip_gfx1250)`.
  **CN:** 外部模块：`torch`, `triton`, `triton.language`, `triton.language.target_info (cuda_capability_geq, is_cuda, is_hip, is_hip_cdna3, is_hip_cdna4, is_hip_gfx1250)`。
- **EN:** Internal modules: none at the top level.
  **CN:** 顶层内部模块：无。
