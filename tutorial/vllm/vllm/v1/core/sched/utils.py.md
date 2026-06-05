# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/core/sched/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_has_repeating_pattern`, `check_sequence_repetition`, `remove_all` for the V1 `core/sched` subsystem. / 为 V1 的 `core/sched` 子系统实现 `_has_repeating_pattern`, `check_sequence_repetition`, `remove_all`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import contextlib
from collections.abc import Sequence

from vllm.sampling_params import RepetitionDetectionParams
from vllm.v1.request import Request, RequestStatus
```
**EN:** Sets up the module namespace with standard-library helpers, and internal vLLM modules such as `vllm.sampling_params`, `vllm.v1.request`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具，以及 `vllm.sampling_params`, `vllm.v1.request` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `_has_repeating_pattern` function / `_has_repeating_pattern` 函数
```python
def _has_repeating_pattern(
    token_ids: Sequence[int],
    pattern_len: int,
    repetition_min_count: int,
) -> bool:
    """Check if the tail of token_ids contains a repeating pattern.

    Compares the last pattern_len tokens against the preceding
    (repetition_min_count - 1) repetitions of the same length.
    """
    for n in range(1, pattern_len + 1):
        target_token = token_ids[-n]
        for m in range(1, repetition_min_count):
            if token_ids[-(pattern_len * m + n)] != target_token:
                return False
    return True
```
**EN:** This function implements `_has_repeating_pattern` within the module. The docstring frames it as: Check if the tail of token_ids contains a repeating pattern. Key calls include `range`. The control flow contains 1 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_has_repeating_pattern`，其作用域位于the module。 关键调用包括 `range`。 控制流包含 1 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `check_sequence_repetition` function / `check_sequence_repetition` 函数
```python
def check_sequence_repetition(
    token_ids: Sequence[int],
    params: RepetitionDetectionParams,
) -> bool:
    """Check if a sequence of token IDs has a repetition pattern.
    Args:
        token_ids: List of token IDs
        params: Repetition detection parameters.
    Returns:
        True if a repetition pattern is found, False otherwise.
    """
    max_pattern_size = params.max_pattern_size
    min_pattern_size = params.min_pattern_size
    min_count = params.min_count

    if min_pattern_size <= 0:
        min_pattern_size = 1

    if max_pattern_size <= 0 or min_count < 2 or min_pattern_size > max_pattern_size:
        return False

    for pattern_len in range(
        min_pattern_size,
        max_pattern_size + 1,
    ):
        if pattern_len * min_count > len(token_ids):
            return False

        if _has_repeating_pattern(token_ids, pattern_len, min_count):
            return True

    return False
```
**EN:** This function validates assumptions or constraints within the module. The docstring frames it as: Check if a sequence of token IDs has a repetition pattern. Key calls include `range`, `_has_repeating_pattern`, `len`. The control flow contains 4 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会校验前提与约束，其作用域位于the module。 关键调用包括 `range`, `_has_repeating_pattern`, `len`。 控制流包含 4 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `remove_all` function / `remove_all` 函数
```python
def remove_all(lst: list, items_to_remove: set) -> list:
    """Remove all items from a list that are in the items_to_remove set.

    This method optimizes for the common case of removing a single item,
    falling back to list comprehension for multiple items.

    Args:
        lst: The list to remove items from
        items_to_remove: Set of items to remove

    Returns:
        Either the modified original list (for single item removal) or
        a new list (for multiple item removal). Callers should use the
        returned value.

    Note:
        For single item removal, this modifies the original list in-place
        and returns it. For multiple items, it creates and returns a new list.
    """
    if not items_to_remove:
        return lst

    if len(items_to_remove) == 1:
        # Fast path for single item removal (most common case)
        item = next(iter(items_to_remove))
        with contextlib.suppress(ValueError):
            lst.remove(item)
        return lst
    # For multiple items, use list comprehension
    return [item for item in lst if item not in items_to_remove]
```
**EN:** This function implements `remove_all` within the module. The docstring frames it as: Remove all items from a list that are in the items_to_remove set. Key calls include `len`, `next`, `iter`, `suppress`, `remove`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `remove_all`，其作用域位于the module。 关键调用包括 `len`, `next`, `iter`, `suppress`, `remove`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `check_stop` function / `check_stop` 函数
```python
def check_stop(request: Request, max_model_len: int) -> bool:
    assert not request.pooling_params

    sampling_params = request.sampling_params
    assert sampling_params is not None

    if request.num_output_tokens < sampling_params.min_tokens:
        return False

    last_token_id = request.output_token_ids[-1]
    if last_token_id == sampling_params.eos_token_id:
        request.status = RequestStatus.FINISHED_STOPPED
        return True

    if last_token_id in (sampling_params.stop_token_ids or ()):
        request.status = RequestStatus.FINISHED_STOPPED
        request.stop_reason = last_token_id
        return True
    if (
        request.num_tokens >= max_model_len
        or request.num_output_tokens >= request.max_tokens
    ):
        request.status = RequestStatus.FINISHED_LENGTH_CAPPED
        return True

    repetition_detection = sampling_params.repetition_detection
    if repetition_detection is not None and (
        check_sequence_repetition(
            request.output_token_ids,
            repetition_detection,
        )
    ):
        request.status = RequestStatus.FINISHED_REPETITION
        request.stop_reason = "repetition_detected"
        return True

    return False
```
**EN:** This function validates assumptions or constraints within the module. Key calls include `check_sequence_repetition`. The control flow contains 5 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会校验前提与约束，其作用域位于the module。 关键调用包括 `check_sequence_repetition`。 控制流包含 5 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `_has_repeating_pattern`: top-level helper or orchestration entry point. / `_has_repeating_pattern`：顶层辅助函数或编排入口。
- `check_sequence_repetition`: top-level helper or orchestration entry point. / `check_sequence_repetition`：顶层辅助函数或编排入口。
- `remove_all`: top-level helper or orchestration entry point. / `remove_all`：顶层辅助函数或编排入口。
- `check_stop`: top-level helper or orchestration entry point. / `check_stop`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `contextlib`, `collections`
- Internal vLLM / 内部依赖: `vllm.sampling_params`, `vllm.v1.request`
