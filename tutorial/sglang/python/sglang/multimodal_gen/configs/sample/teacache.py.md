# teacache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/sample/teacache.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `TeaCacheParams`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `TeaCacheParams` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-9: module setup and imports / 模块初始化与导入
```python
from __future__ import annotations

from dataclasses import dataclass, field
from typing import Callable

from sglang.multimodal_gen.configs.sample.sampling_params import CacheParams
```
**EN:** This block establishes the module context and imports `__future__`, `dataclasses`, `typing`, and `sglang.multimodal_gen.configs.sample.sampling_params`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `__future__`、`dataclasses`、`typing` 和 `sglang.multimodal_gen.configs.sample.sampling_params`。这些依赖为后续实现提供所需符号。

### Lines 13-49: `TeaCacheParams` class overview / `TeaCacheParams` 类概览
```python
class TeaCacheParams(CacheParams):
    """
    Parameters for [TeaCache](https://arxiv.org/abs/2411.14324).

    Attributes:
        cache_type: (`str`, defaults to `teacache`):
            A string labeling these parameters as belonging to teacache.
        teacache_thresh (`float`, defaults to `0.0`):
            Threshold for accumulated relative L1 distance. When below this threshold, the
            forward pass is skipped. Recommended values: 0.25 for ~1.5x speedup, 0.4 for ~1.8x,
            0.6 for ~2.0x.
        start_skipping (`int` or `float`, defaults to `5`):
            The number of timesteps after which we may skip a forward pass. These early
            steps define the global structure and are too critical to not skip.
            int: The number of timesteps after which we can skip. If negative,
                 this is an offset from the end of the schedule.
            float (0.0 - 1.0): A percentage of the total steps (e.g., 0.1
                               computes the first 10%).
        end_skipping (`int` or `float`, defaults to `-1`):
            The number of timesteps after which we are no longer able to skip
            forward passes. The last steps refine fine textures and details.
            int: The number of timesteps after which skipping ends. If negative,
                 this is an offset from the total number of steps.
            float (0.0 - 1.0): A percentage of the total steps (e.g., 0.1
                               computes the first 10%).
        coefficients (`List[float]`, defaults to `[]`):
            Polynomial coefficients for rescaling the raw relative L1 distance,
            evaluated as `c[0]*x**4 + c[1]*x**3 + c[2]*x**2 + c[3]*x + c[4]`.
        coefficients_callback (`Callable[[TeaCacheParams], List[float]]`, *optional*):
            A function that receives this `TeaCacheParams` instance and returns
            the polynomial coefficients to use. When set, it takes precedence over
            the `coefficients` field, allowing dynamic coefficient selection based
            on any property of the params (e.g., `use_ret_steps` for Wan models).
        use_ret_steps: (`bool`, `None`, defaults to `None`):
            Used exclusively for wanvideo models to select different modulated inputs.
    """
```
**EN:** This block defines class `TeaCacheParams`. Parameters for [TeaCache](https://arxiv.org/abs/2411.14324). Attributes: cache_type: (`str`, defaults to `teacache`): A string labeling these parameters as belonging to teacache. It inherits from `CacheParams`.
**CN:** 该代码块定义了类 `TeaCacheParams`。 它用于封装 tea cache params 相关行为。 它继承自 `CacheParams`。

### Lines 50-58: supporting statements / 辅助语句
```python
    cache_type: str = "teacache"
    teacache_thresh: float = 0.0
    start_skipping: int | float = 5
    end_skipping: int | float = -1
    coefficients: list[float] = field(default_factory=list)
    coefficients_callback: Callable[[TeaCacheParams], list[float]] | None = field(
        default=None, repr=False
    )
    use_ret_steps: bool | None = None
```
**EN:** This block gathers supporting statements inside `TeaCacheParams`. It updates names such as `cache_type`, `teacache_thresh`, `start_skipping`, `end_skipping`, `coefficients`, and `coefficients_callback`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `TeaCacheParams` 内部的辅助语句。 它会更新 `cache_type`、`teacache_thresh`、`start_skipping`、`end_skipping`、`coefficients` 和 `coefficients_callback` 等名称。 代码会与 `field` 协同工作。

### Lines 60-63: `get_coefficients` implementation / `get_coefficients` 实现
```python
    def get_coefficients(self) -> list[float]:
        if self.coefficients_callback is not None:
            return self.coefficients_callback(self)
        return self.coefficients
```
**EN:** This block defines method `get_coefficients` on `TeaCacheParams`. It retrieves coefficients. Key calls include `self.coefficients_callback`. The implementation branches on conditions.
**CN:** 该代码块定义了 `TeaCacheParams` 的方法 `get_coefficients`。 它用于获取coefficients。 关键调用包括 `self.coefficients_callback`。 实现中包含条件分支。

### Lines 65-82: `get_skip_boundaries` implementation / `get_skip_boundaries` 实现
```python
    def get_skip_boundaries(
        self, num_inference_steps: int, do_cfg: bool
    ) -> tuple[int, int]:
        def _resolve_boundary(value: int | float) -> int:
            if isinstance(value, float):
                return int(num_inference_steps * value)
            if value < 0:
                return num_inference_steps + value
            return value

        start_skipping = _resolve_boundary(self.start_skipping)
        end_skipping = _resolve_boundary(self.end_skipping)

        if do_cfg:
            start_skipping *= 2
            end_skipping *= 2

        return start_skipping, end_skipping
```
**EN:** This block defines method `get_skip_boundaries` on `TeaCacheParams`. It retrieves skip boundaries. Key calls include `_resolve_boundary`, `isinstance`, and `int`. The implementation branches on conditions. Parameters such as `num_inference_steps`, and `do_cfg` drive the behavior in this section.
**CN:** 该代码块定义了 `TeaCacheParams` 的方法 `get_skip_boundaries`。 它用于获取skip boundaries。 关键调用包括 `_resolve_boundary`、`isinstance` 和 `int`。 实现中包含条件分支。 本段逻辑主要由 `num_inference_steps` 和 `do_cfg` 等参数驱动。

## Key Concepts / 关键概念
- `TeaCacheParams`: Parameters for [TeaCache](https://arxiv.org/abs/2411.14324). / 核心类，用于封装 tea cache params 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `dataclasses`, `typing`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.sample.sampling_params`

- **Total lines / 总行数**: 82
