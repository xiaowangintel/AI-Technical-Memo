# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/comparator/utils.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on shared debugging utilities. It mainly provides reusable helpers that reduce duplication across nearby modules. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于共享调试工具。它主要用于提供可复用的辅助函数，减少相邻模块中的重复代码。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Import dependencies and shared types / 导入依赖与共享类型
```python
from __future__ import annotations

import functools
import re
from pathlib import Path
from typing import TYPE_CHECKING, Callable, Generic, Optional, Tuple, TypeVar

import torch
from pydantic import BaseModel, ConfigDict
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 11-12: Declare module-level symbols such as `_T`, `_U` / 声明模块级符号，例如 `_T`, `_U`
```python
_T = TypeVar("_T")
_U = TypeVar("_U")
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 15-20: Implement helper `_check_equal_lengths` / 实现辅助函数 `_check_equal_lengths`
```python
def _check_equal_lengths(**named_lists: list) -> None:
    lengths: dict[str, int] = {name: len(lst) for name, lst in named_lists.items()}
    unique: set[int] = set(lengths.values())
    if len(unique) > 1:
        details: str = ", ".join(f"{name}={length}" for name, length in lengths.items())
        raise ValueError(f"Length mismatch: {details}")
```
**EN:** Function `_check_equal_lengths` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_check_equal_lengths` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 23-60: Implement function `auto_descend_dir` / 实现函数 `auto_descend_dir`
```python
def auto_descend_dir(directory: Path, label: str) -> Path:
    """If directory has no .pt files but exactly one subdirectory does, descend into it.

    Raises ValueError when the layout is ambiguous (>=2 subdirs with .pt)
    or when no .pt data is found at all.
    """
    if any(directory.glob("*.pt")):
        return directory

    candidates: list[Path] = [
        sub for sub in directory.iterdir() if sub.is_dir() and any(sub.glob("*.pt"))
    ]

    if len(candidates) >= 2:
        names: str = ", ".join(sorted(c.name for c in candidates))
        raise ValueError(
            f"{label}: directory {directory} has no .pt files at top level "
            f"and multiple subdirectories contain data ({names}). "
            f"Please specify the exact subdirectory."
        )

    if len(candidates) == 0:
        raise ValueError(
            f"{label}: no .pt files found in {directory} or any of its subdirectories."
        )

    resolved: Path = candidates[0]

    from sglang.srt.debug_utils.comparator.log_sink import log_sink
    from sglang.srt.debug_utils.comparator.output_types import InfoLog

    log_sink.add(
        InfoLog(
            category="auto_descend",
            message=f"auto-descend {label}: {directory} -> {resolved}",
        )
    )
    return resolved
```
**EN:** Function `auto_descend_dir` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `auto_descend_dir` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 63-63: Define class `_StrictBase` and class context / 定义类 `_StrictBase`及类上下文
```python
class _StrictBase(BaseModel):
```
**EN:** This section introduces `_StrictBase`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_StrictBase`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 64-64: Declare fields for `_StrictBase` such as `model_config` / 为 `_StrictBase` 声明字段，例如 `model_config`
```python
    model_config = ConfigDict(extra="forbid")
```
**EN:** These lines declare the state carried by `_StrictBase`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `_StrictBase` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 67-67: Define class `_FrozenBase` and class context / 定义类 `_FrozenBase`及类上下文
```python
class _FrozenBase(BaseModel):
```
**EN:** This section introduces `_FrozenBase`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `_FrozenBase`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 68-68: Declare fields for `_FrozenBase` such as `model_config` / 为 `_FrozenBase` 声明字段，例如 `model_config`
```python
    model_config = ConfigDict(frozen=True, extra="forbid")
```
**EN:** These lines declare the state carried by `_FrozenBase`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `_FrozenBase` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 71-71: Define class `Pair` and class context / 定义类 `Pair`及类上下文
```python
class Pair(_FrozenBase, Generic[_T]):
```
**EN:** This section introduces `Pair`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `Pair`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 72-73: Declare fields for `Pair` such as `x`, `y` / 为 `Pair` 声明字段，例如 `x`, `y`
```python
    x: _T
    y: _T
```
**EN:** These lines declare the state carried by `Pair`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `Pair` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 75-76: Implement method `map` for `Pair` / 为 `Pair` 实现方法 `map`
```python
    def map(self, fn: Callable[[_T], _U]) -> Pair[_U]:
        return Pair(x=fn(self.x), y=fn(self.y))
```
**EN:** Method `map` implements behavior on `Pair`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `map` 为 `Pair` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 79-81: Implement function `argmax_coord` / 实现函数 `argmax_coord`
```python
def argmax_coord(x: torch.Tensor) -> Tuple[int, ...]:
    flat_idx = x.argmax()
    return tuple(idx.item() for idx in torch.unravel_index(flat_idx, x.shape))
```
**EN:** Function `argmax_coord` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `argmax_coord` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 84-91: Implement function `compute_smaller_dtype` / 实现函数 `compute_smaller_dtype`
```python
def compute_smaller_dtype(
    dtypes: Pair[torch.dtype],
) -> Optional[torch.dtype]:
    info_dict = {
        (torch.float32, torch.bfloat16): torch.bfloat16,
        # ... add more ...
    }
    return info_dict.get((dtypes.x, dtypes.y)) or info_dict.get((dtypes.y, dtypes.x))
```
**EN:** Function `compute_smaller_dtype` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `compute_smaller_dtype` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 94-102: Implement function `try_unify_shape` / 实现函数 `try_unify_shape`
```python
def try_unify_shape(x: torch.Tensor, target_shape: torch.Size) -> torch.Tensor:
    x_shape = x.shape
    num_dim_to_remove = len(x_shape) - len(target_shape)
    if (x_shape[num_dim_to_remove:] == target_shape) and all(
        val == 1 for val in x_shape[:num_dim_to_remove]
    ):
        return functools.reduce(lambda a, _: a.squeeze(0), range(num_dim_to_remove), x)

    return x
```
**EN:** Function `try_unify_shape` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `try_unify_shape` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 106-110: Implement function `calc_rel_diff` / 实现函数 `calc_rel_diff`
```python
def calc_rel_diff(x: torch.Tensor, y: torch.Tensor) -> torch.Tensor:
    x, y = x.double(), y.double()
    denominator = (x * x + y * y).sum()
    sim = 2 * (x * y).sum() / denominator
    return 1 - sim
```
**EN:** Function `calc_rel_diff` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `calc_rel_diff` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 113-130: Implement function `calc_per_token_rel_diff` / 实现函数 `calc_per_token_rel_diff`
```python
def calc_per_token_rel_diff(
    x: torch.Tensor, y: torch.Tensor, *, seq_dim: int
) -> torch.Tensor:
    """Cosine-distance-like metric per token position.

    Sums over all dims except seq_dim.
    """
    x, y = x.double(), y.double()
    other_dims: list[int] = [d for d in range(x.dim()) if d != seq_dim]

    if other_dims:
        denominator: torch.Tensor = (x * x + y * y).sum(dim=other_dims)
        sim: torch.Tensor = 2 * (x * y).sum(dim=other_dims) / (denominator + 1e-10)
    else:
        denominator = x * x + y * y
        sim = 2 * (x * y) / (denominator + 1e-10)

    return (1 - sim).float()
```
**EN:** Function `calc_per_token_rel_diff` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `calc_per_token_rel_diff` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 133-134: Handle conditional module logic / 处理条件模块逻辑
```python
if TYPE_CHECKING:
    from sglang.srt.debug_utils.comparator.output_types import SummaryRecord
```
**EN:** This conditional branch selects behavior based on runtime state, optional inputs, or developer-facing entry conditions.
**CN:** 该条件分支会根据运行时状态、可选输入或面向开发者的入口条件来选择不同的行为。

### Lines 137-158: Implement function `compute_exit_code` / 实现函数 `compute_exit_code`
```python
def compute_exit_code(
    summary: SummaryRecord,
    *,
    allow_skipped_pattern: str,
    skipped_names: list[str],
    allow_failed_pattern: Optional[str],
    failed_names: list[str],
    errored_names: Optional[list[str]] = None,
) -> int:
    if summary.passed == 0:
        return 1

    if errored_names:
        return 1

    if not _is_all_match_pattern(pattern=allow_failed_pattern, strings=failed_names):
        return 1

    if not _is_all_match_pattern(pattern=allow_skipped_pattern, strings=skipped_names):
        return 1

    return 0
```
**EN:** Function `compute_exit_code` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `compute_exit_code` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 161-165: Implement helper `_is_all_match_pattern` / 实现辅助函数 `_is_all_match_pattern`
```python
def _is_all_match_pattern(*, pattern: Optional[str], strings: list[str]) -> bool:
    if pattern is None:
        return len(strings) == 0
    compiled: re.Pattern[str] = re.compile(pattern)
    return all(compiled.fullmatch(s) for s in strings)
```
**EN:** Function `_is_all_match_pattern` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_is_all_match_pattern` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `_StrictBase`, `_FrozenBase`, `Pair`, `_check_equal_lengths`, `auto_descend_dir`, `argmax_coord`, `compute_smaller_dtype`, `try_unify_shape`, `calc_rel_diff`, `calc_per_token_rel_diff`, `compute_exit_code`, `_is_all_match_pattern`
- **Module role / 模块角色**: Shared debugging utilities / 共享调试工具
- **Implementation focus / 实现重点**: Provides reusable helpers that reduce duplication across nearby modules / 提供可复用的辅助函数，减少相邻模块中的重复代码

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `functools`, `re`, `pathlib`, `typing`
- **Third-party / 第三方**: `torch`, `pydantic`
- **Internal / 内部**: `sglang.srt.debug_utils.comparator.log_sink`, `sglang.srt.debug_utils.comparator.output_types`
