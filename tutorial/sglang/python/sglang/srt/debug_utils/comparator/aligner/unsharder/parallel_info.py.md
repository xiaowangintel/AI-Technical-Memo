# parallel_info.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/aligner/unsharder/parallel_info.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on unsharding and tensor-parallel merging. It mainly implements the core logic needed by this part of the debug toolchain. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于反分片与张量并行合并。它主要用于实现该调试工具链所需的核心逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Import dependencies and shared types / 导入依赖与共享类型
```python
from typing import Optional

from sglang.srt.debug_utils.comparator.aligner.unsharder.types import AxisInfo
from sglang.srt.debug_utils.comparator.dims_spec import ParallelAxis
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 6-6: Declare module-level symbols such as `_PARALLEL_INFO_KEYS` / 声明模块级符号，例如 `_PARALLEL_INFO_KEYS`
```python
_PARALLEL_INFO_KEYS = ("sglang_parallel_info", "megatron_parallel_info")
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 9-11: Implement helper `_is_error_sentinel` / 实现辅助函数 `_is_error_sentinel`
```python
def _is_error_sentinel(value: dict) -> bool:
    """Check if a parallel_info dict is an error sentinel (e.g. {'megatron_error': True})."""
    return any(k.endswith("_error") for k in value)
```
**EN:** Function `_is_error_sentinel` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_is_error_sentinel` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 14-45: Implement function `normalize_parallel_info` / 实现函数 `normalize_parallel_info`
```python
def normalize_parallel_info(meta: dict) -> dict[ParallelAxis, AxisInfo]:
    """Extract unified parallel info from dump meta."""
    info: Optional[dict] = None
    for key in _PARALLEL_INFO_KEYS:
        value = meta.get(key)
        if isinstance(value, dict) and value and not _is_error_sentinel(value):
            if info is not None:
                raise ValueError(
                    f"Meta contains multiple parallel_info keys among {_PARALLEL_INFO_KEYS}"
                )
            info = value

    if info is None:
        info = {}

    result: dict[ParallelAxis, AxisInfo] = {}
    for axis in ParallelAxis:
        axis_rank = info.get(f"{axis.value}_rank")
        axis_size = info.get(f"{axis.value}_size")

        # Recompute pseudo-axis lives at top-level meta, not inside parallel_info
        if axis_rank is None:
            axis_rank = meta.get(f"{axis.value}_rank")
            axis_size = meta.get(f"{axis.value}_size")

        if axis_rank is not None and axis_size is not None and axis_size > 1:
            result[axis] = AxisInfo(
                axis_rank=axis_rank,
                axis_size=axis_size,
            )

    return result
```
**EN:** Function `normalize_parallel_info` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `normalize_parallel_info` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `_is_error_sentinel`, `normalize_parallel_info`
- **Module role / 模块角色**: Unsharding and tensor-parallel merging / 反分片与张量并行合并
- **Implementation focus / 实现重点**: Implements the core logic needed by this part of the debug toolchain / 实现该调试工具链所需的核心逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.aligner.unsharder.types`, `sglang.srt.debug_utils.comparator.dims_spec`
