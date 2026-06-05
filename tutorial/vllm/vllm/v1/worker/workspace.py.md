# workspace.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/workspace.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_compute_bytes`, `WorkspaceManager`, `is_workspace_manager_initialized` for the V1 `worker` subsystem. / 为 V1 的 `worker` 子系统实现 `_compute_bytes`, `WorkspaceManager`, `is_workspace_manager_initialized`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import inspect
import os
from itertools import accumulate
from math import prod

import torch

import vllm.envs as envs
from vllm.logger import init_logger
from vllm.utils.math_utils import round_up
from vllm.v1.worker.ubatching import dbo_current_ubatch_id

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `_compute_bytes` function / `_compute_bytes` 函数
```python
def _compute_bytes(shape: tuple[int, ...], dtype: torch.dtype) -> int:
    return prod(shape) * dtype.itemsize
```
**EN:** This function implements `_compute_bytes` within the module. Key calls include `prod`.
**CN:** 该函数会实现 `_compute_bytes`，其作用域位于the module。 关键调用包括 `prod`。

### Module constants / 模块常量
```python
_MB = 1024**2
_GiB = 1024**3

# Global workspace manager instance
_manager: "WorkspaceManager | None" = None
```
**EN:** Defines module-level constants or aliases such as `_MB`, `_GiB`, `_manager`, which are reused by later definitions.
**CN:** 定义 `_MB`, `_GiB`, `_manager` 等模块级常量或别名，供后续定义复用。

### `WorkspaceManager` class / `WorkspaceManager` 类
```python
class WorkspaceManager:
    """Manager for workspace allocation.

    Manages one workspace buffer per active ubatch slot.
    Can be locked to prevent further growth during execution.
    """
```
**EN:** Introduces the `WorkspaceManager` class. Core methods include `__init__`, `_workspace_size_bytes`, `lock`, `unlock`, `is_locked`, `get_simultaneous`. Docstring signal: Manager for workspace allocation.
**CN:** 这里定义 `WorkspaceManager` 类。核心方法包括 `__init__`, `_workspace_size_bytes`, `lock`, `unlock`, `is_locked`, `get_simultaneous`。

### `WorkspaceManager.__init__` method / `WorkspaceManager.__init__` 方法
```python
    def __init__(self, device: torch.device, num_ubatches: int | None = None):
        self._device = device
        # Cache num ubatches at init based on configuration (default to 1)
        self._num_ubatches = num_ubatches if num_ubatches is not None else 1
        self._current_workspaces: list[torch.Tensor | None] = [
            None
        ] * self._num_ubatches
        self._locked: bool = False
```
**EN:** This method initializes the object state within `WorkspaceManager`. It touches state such as `_device`, `_num_ubatches`, `_current_workspaces`, `_locked`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`WorkspaceManager`。 它会读写 `_device`, `_num_ubatches`, `_current_workspaces`, `_locked` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `WorkspaceManager._workspace_size_bytes` method / `WorkspaceManager._workspace_size_bytes` 方法
```python
    @staticmethod
    def _workspace_size_bytes(workspace: torch.Tensor | None) -> int:
        """Get size of workspace in bytes."""
        if workspace is None:
            return 0
        return workspace.numel() * workspace.element_size()
```
**EN:** This method implements `_workspace_size_bytes` within `WorkspaceManager`. The docstring frames it as: Get size of workspace in bytes. Key calls include `numel`, `element_size`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_workspace_size_bytes`，其作用域位于`WorkspaceManager`。 关键调用包括 `numel`, `element_size`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `WorkspaceManager.lock` method / `WorkspaceManager.lock` 方法
```python
    def lock(self) -> None:
        """Lock the workspace to prevent further growth.

        After locking, any attempt to allocate a larger workspace will raise
        an assertion error. This ensures workspace size is fixed during execution.
        """
        self._locked = True
        if envs.VLLM_DEBUG_WORKSPACE:
            logger.info(
                "[WORKSPACE DEBUG] Workspace locked. Current sizes: %s",
                [
                    self._workspace_size_bytes(ws) / _MB
                    for ws in self._current_workspaces
                    if ws is not None
                ],
            )
```
**EN:** This method implements `lock` within `WorkspaceManager`. The docstring frames it as: Lock the workspace to prevent further growth. Key calls include `info`, `_workspace_size_bytes`. It touches state such as `_locked`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `lock`，其作用域位于`WorkspaceManager`。 关键调用包括 `info`, `_workspace_size_bytes`。 它会读写 `_locked` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `WorkspaceManager.unlock` method / `WorkspaceManager.unlock` 方法
```python
    def unlock(self) -> None:
        """Unlock the workspace to allow growth.

        This is used during elastic EP scaling when the workspace size
        needs to grow due to changes in the number of experts.
        """
        self._locked = False
        if envs.VLLM_DEBUG_WORKSPACE:
            logger.info(
                "[WORKSPACE DEBUG] Workspace unlocked. Current sizes: %s",
                [
                    self._workspace_size_bytes(ws) / _MB
                    for ws in self._current_workspaces
                    if ws is not None
                ],
            )
```
**EN:** This method implements `unlock` within `WorkspaceManager`. The docstring frames it as: Unlock the workspace to allow growth. Key calls include `info`, `_workspace_size_bytes`. It touches state such as `_locked`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `unlock`，其作用域位于`WorkspaceManager`。 关键调用包括 `info`, `_workspace_size_bytes`。 它会读写 `_locked` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `WorkspaceManager.is_locked` method / `WorkspaceManager.is_locked` 方法
```python
    def is_locked(self) -> bool:
        """Check if workspace is locked."""
        return self._locked
```
**EN:** This method answers a boolean capability check within `WorkspaceManager`. The docstring frames it as: Check if workspace is locked.
**CN:** 该方法会回答布尔能力判断，其作用域位于`WorkspaceManager`。

### `WorkspaceManager.get_simultaneous` method / `WorkspaceManager.get_simultaneous` 方法
```python
    def get_simultaneous(
        self, *shapes_and_dtypes: tuple[tuple[int, ...], torch.dtype]
    ) -> list[torch.Tensor]:
        """Get multiple workspace tensors simultaneously from a single allocation.

        Args:
            *shapes_and_dtypes: One or more (shape, dtype) tuples.

        Returns:
            List of tensor views into the workspace buffer, one per shape/dtype pair.
        """
        actual_bytes = [_compute_bytes(s, d) for s, d in shapes_and_dtypes]
        aligned_bytes = [round_up(actual, 256) for actual in actual_bytes]
        total_bytes = sum(aligned_bytes)

        # Calculate cumulative offsets using itertools.accumulate
        offsets = list(accumulate([0] + aligned_bytes[:-1]))

        current_workspace = self._ensure_workspace_size(total_bytes)

        return [
            current_workspace[offsets[i] : offsets[i] + actual_bytes[i]]
            .view(shapes_and_dtypes[i][1])
            .reshape(shapes_and_dtypes[i][0])
            for i in range(len(shapes_and_dtypes))
        ]
```
**EN:** This method returns or derives a value within `WorkspaceManager`. The docstring frames it as: Get multiple workspace tensors simultaneously from a single allocation. Key calls include `sum`, `list`, `_ensure_workspace_size`, `_compute_bytes`, `round_up`, `accumulate`.
**CN:** 该方法会返回或推导一个值，其作用域位于`WorkspaceManager`。 关键调用包括 `sum`, `list`, `_ensure_workspace_size`, `_compute_bytes`, `round_up`, `accumulate`。

### `WorkspaceManager._ensure_workspace_size` method / `WorkspaceManager._ensure_workspace_size` 方法
```python
    def _ensure_workspace_size(self, required_bytes: int) -> torch.Tensor:
        """Ensure workspace is allocated and large enough, return current workspace.

        Args:
            required_bytes: The number of bytes required.

        Returns:
            The current workspace tensor.
        """
        ubatch_id = dbo_current_ubatch_id()
        current_workspace = self._current_workspaces[ubatch_id]
        current_size = self._workspace_size_bytes(current_workspace)

        if current_size < required_bytes:

            def get_caller_info() -> str:
                """Find first frame outside WorkspaceManager."""
                curr_frame = inspect.currentframe()
                if curr_frame is None:
                    return "unknown"
                # Walk up the stack skipping WorkspaceManager frames
                curr_frame = curr_frame.f_back
                while curr_frame is not None:
                    # TODO: This only catches instance methods (self), missing
                    # classmethods and staticmethods. Once Python 3.11+ is the
                    # minimum supported version, use co_qualname instead:
                    #   qualname = curr_frame.f_code.co_qualname
                    #   if qualname.startswith("WorkspaceManager."):
                    if isinstance(curr_frame.f_locals.get("self"), WorkspaceManager):
                        curr_frame = curr_frame.f_back
                        continue
                    filename = os.path.basename(curr_frame.f_code.co_filename)
                    return (
                        f"{filename}:{curr_frame.f_lineno}:{curr_frame.f_code.co_name}"
                    )
                return "unknown"

            if self._locked:
                raise AssertionError(
                    f"Workspace is locked but allocation from '{get_caller_info()}' "
                    f"requires {required_bytes / _MB:.2f} MB, current size is "
                    f"{current_size / _MB:.2f} MB. "
                    "Workspace growth is not allowed after locking."
                )

            # Only resize the requesting ubatch's workspace.  Other
            # ubatches resize lazily on their next get_simultaneous call.
            # Resizing all ubatches here would orphan the other ubatch's
            # old tensor when it still holds views into it (DBO leak).
            self._current_workspaces[ubatch_id] = None
            del current_workspace
            # Release the freed segment back to CUDA so the caching
            # allocator can reuse the GPU memory for the larger
            # allocation below. Without this, each resize may leave a
            # dead segment in reserved memory which can cause higher peak
            # memory usage.
            torch.accelerator.empty_cache()
            self._current_workspaces[ubatch_id] = torch.empty(
                (required_bytes,), dtype=torch.uint8, device=self._device
            )
            current_workspace = self._current_workspaces[ubatch_id]

            if envs.VLLM_DEBUG_WORKSPACE:
                logger.info(
                    "[WORKSPACE DEBUG] Resized workspace from '%s': %.2f MB -> "
                    "%.2f MB (ubatch %d)",
                    get_caller_info(),
                    current_size / _MB,
                    required_bytes / _MB,
                    ubatch_id,
                )

        return current_workspace
```
**EN:** This method implements `_ensure_workspace_size` within `WorkspaceManager`. The docstring frames it as: Ensure workspace is allocated and large enough, return current workspace. Key calls include `dbo_current_ubatch_id`, `_workspace_size_bytes`, `empty_cache`, `empty`, `currentframe`, `AssertionError`. It touches state such as `_current_workspaces`. The control flow contains 5 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_ensure_workspace_size`，其作用域位于`WorkspaceManager`。 关键调用包括 `dbo_current_ubatch_id`, `_workspace_size_bytes`, `empty_cache`, `empty`, `currentframe`, `AssertionError`。 它会读写 `_current_workspaces` 等状态。 控制流包含 5 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `is_workspace_manager_initialized` function / `is_workspace_manager_initialized` 函数
```python
def is_workspace_manager_initialized() -> bool:
    """Check if workspace manager has been initialized.

    Returns:
        True if workspace manager is initialized, False otherwise.
    """
    return _manager is not None
```
**EN:** This function answers a boolean capability check within the module. The docstring frames it as: Check if workspace manager has been initialized.
**CN:** 该函数会回答布尔能力判断，其作用域位于the module。

### `current_workspace_manager` function / `current_workspace_manager` 函数
```python
def current_workspace_manager() -> "WorkspaceManager":
    """Get the current workspace manager instance.

    Raises:
        AssertionError: If workspace manager has not been initialized.
    """
    assert _manager is not None, (
        "WorkspaceManager not initialized. Call init_workspace_manager() "
        "with a device before using workspace functions."
    )
    return _manager
```
**EN:** This function implements `current_workspace_manager` within the module. The docstring frames it as: Get the current workspace manager instance.
**CN:** 该函数会实现 `current_workspace_manager`，其作用域位于the module。

### `init_workspace_manager` function / `init_workspace_manager` 函数
```python
def init_workspace_manager(
    device: torch.device, num_ubatches: int | None = None
) -> None:
    """Initialize the workspace manager with a device.

    Must be called before using any workspace functions. Typically called
    from GPUModelRunner.__init__.

    Args:
        device: The device to allocate workspace on.
        num_ubatches: Number of workspace ubatch slots. Defaults to 1.
    """
    global _manager
    if _manager is not None:
        logger.warning(
            "WorkspaceManager already initialized on device %s, "
            "reinitializing on device %s",
            _manager._device,
            device,
        )
    _manager = WorkspaceManager(device, num_ubatches)
```
**EN:** This function implements `init_workspace_manager` within the module. The docstring frames it as: Initialize the workspace manager with a device. Key calls include `WorkspaceManager`, `warning`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `init_workspace_manager`，其作用域位于the module。 关键调用包括 `WorkspaceManager`, `warning`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `lock_workspace` function / `lock_workspace` 函数
```python
def lock_workspace() -> None:
    """Lock the workspace to prevent further growth.

    After calling this function, any attempt to allocate a workspace larger
    than the current size will raise an AssertionError. This ensures that
    workspace size is fixed during execution and prevents unexpected memory
    allocations in the hot path.

    Example:
        # During initialization
        init_workspace_manager(device)
        reserve_workspace(shape1, dtype1)
        reserve_workspace(shape2, dtype2)

        # Lock after warmup/profiling
        lock_workspace()

        # Now all get_workspace calls must fit in pre-allocated size
    """
    current_workspace_manager().lock()
```
**EN:** This function implements `lock_workspace` within the module. The docstring frames it as: Lock the workspace to prevent further growth. Key calls include `lock`, `current_workspace_manager`.
**CN:** 该函数会实现 `lock_workspace`，其作用域位于the module。 关键调用包括 `lock`, `current_workspace_manager`。

### `unlock_workspace` function / `unlock_workspace` 函数
```python
def unlock_workspace() -> None:
    """Unlock the workspace to allow growth.

    This is used during elastic EP scaling when the workspace size
    needs to grow due to changes in the number of experts.
    After scaling operations complete, lock_workspace() should be
    called again to prevent unexpected allocations.
    """
    current_workspace_manager().unlock()
```
**EN:** This function implements `unlock_workspace` within the module. The docstring frames it as: Unlock the workspace to allow growth. Key calls include `unlock`, `current_workspace_manager`.
**CN:** 该函数会实现 `unlock_workspace`，其作用域位于the module。 关键调用包括 `unlock`, `current_workspace_manager`。

### `reset_workspace_manager` function / `reset_workspace_manager` 函数
```python
def reset_workspace_manager() -> None:
    """Reset the workspace manager to uninitialized state.

    This is primarily intended for testing purposes to allow tests
    to reinitialize the workspace manager cleanly.
    """
    global _manager
    _manager = None
```
**EN:** This function implements `reset_workspace_manager` within the module. The docstring frames it as: Reset the workspace manager to uninitialized state.
**CN:** 该函数会实现 `reset_workspace_manager`，其作用域位于the module。

## Key Concepts / 关键概念
- `_compute_bytes`: top-level helper or orchestration entry point. / `_compute_bytes`：顶层辅助函数或编排入口。
- `WorkspaceManager`: central class or interface in this module. / `WorkspaceManager`：本模块中的核心类或接口。
- `is_workspace_manager_initialized`: top-level helper or orchestration entry point. / `is_workspace_manager_initialized`：顶层辅助函数或编排入口。
- `current_workspace_manager`: top-level helper or orchestration entry point. / `current_workspace_manager`：顶层辅助函数或编排入口。
- `init_workspace_manager`: top-level helper or orchestration entry point. / `init_workspace_manager`：顶层辅助函数或编排入口。
- `lock_workspace`: top-level helper or orchestration entry point. / `lock_workspace`：顶层辅助函数或编排入口。
- `unlock_workspace`: top-level helper or orchestration entry point. / `unlock_workspace`：顶层辅助函数或编排入口。
- `reset_workspace_manager`: top-level helper or orchestration entry point. / `reset_workspace_manager`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `inspect`, `os`, `itertools`, `math`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.envs`, `vllm.logger`, `vllm.utils.math_utils`, `vllm.v1.worker.ubatching`
