# reduce.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/diffusion/cutedsl/common/reduce.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import math". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import math”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Imports and module setup
```python
import math

import cutlass
import cutlass.cute as cute


@cute.jit
```
**EN:** This block imports dependencies and prepares names used later in the module. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 8-15: Function `warp_reduce_sum`
```python
def warp_reduce_sum(val: cute.Numeric, reduce_size: int = 32) -> cute.Numeric:
    iters = int(math.log2(reduce_size))
    for i in range(iters):
        val = val + cute.arch.shuffle_sync_down(val, offset=1 << (iters - i - 1))
    return val


@cute.jit
```
**EN:** This block defines `warp_reduce_sum`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `warp_reduce_sum`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 16-33: Function `cta_reduce_sum`
```python
def cta_reduce_sum(
    val: cute.Numeric, num_warps: cutlass.Constexpr, tidx: cutlass.Int32
) -> cute.Numeric:
    smem = cutlass.utils.SmemAllocator()
    acc = smem.allocate_tensor(cutlass.Float32, num_warps + 1)
    warp_id = tidx >> 5
    lane_id = tidx & 31
    if lane_id == 0:
        acc[warp_id] = val
    cute.arch.sync_threads()
    if warp_id == 0:
        val = acc[lane_id] if lane_id < num_warps else cutlass.Float32(0)
        val = warp_reduce_sum(val)
        if lane_id == 0:
            acc[num_warps] = val
    cute.arch.sync_threads()
    val = acc[num_warps]
    return val
```
**EN:** This block defines `cta_reduce_sum`, including its interface and main implementation logic. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `cta_reduce_sum`，包括其接口以及主要实现逻辑。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核

## Dependencies / 依赖关系
- `math`
- `cutlass`
- `cutlass.cute as cute`
