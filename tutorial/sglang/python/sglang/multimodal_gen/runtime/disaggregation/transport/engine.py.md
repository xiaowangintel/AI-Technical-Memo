# engine.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/disaggregation/transport/engine.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the disaggregation runtime layer. It centers on `BaseTransferEngine`, `MooncakeDiffusionEngine`, and `_check_mooncake`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Transfer engine abstraction for tensor transfer between role instances. / 该文件属于解耦运行时层。它围绕 `BaseTransferEngine`、`MooncakeDiffusionEngine` 和 `_check_mooncake` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 2-5: module setup and imports / 模块初始化与导入
```python
"""Transfer engine abstraction for tensor transfer between role instances."""

import logging
from abc import ABC, abstractmethod
```
**EN:** This block establishes the module context and imports `logging`, and `abc`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `logging` 和 `abc`。这些依赖为后续实现提供所需符号。

### Lines 7-9: supporting statements / 辅助语句
```python
logger = logging.getLogger(__name__)

_MOONCAKE_AVAILABLE = None
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`, and `_MOONCAKE_AVAILABLE`. The code collaborates with `logging.getLogger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 和 `_MOONCAKE_AVAILABLE` 等名称。 代码会与 `logging.getLogger` 协同工作。

### Lines 12-23: `_check_mooncake` implementation / `_check_mooncake` 实现
```python
def _check_mooncake() -> bool:
    global _MOONCAKE_AVAILABLE
    if _MOONCAKE_AVAILABLE is None:
        try:
            from sglang.srt.distributed.device_communicators.mooncake_transfer_engine import (  # noqa: F401
                MooncakeTransferEngine as _MTE,
            )

            _MOONCAKE_AVAILABLE = True
        except ImportError:
            _MOONCAKE_AVAILABLE = False
    return _MOONCAKE_AVAILABLE
```
**EN:** This block defines function `_check_mooncake`. It checks mooncake. The implementation branches on conditions, handles exceptional paths.
**CN:** 该代码块定义了函数 `_check_mooncake`。 它用于检查mooncake。 实现中包含条件分支，处理异常路径。

### Lines 26-28: `BaseTransferEngine` class overview / `BaseTransferEngine` 类概览
```python
class BaseTransferEngine(ABC):
    """Abstract transfer engine for data movement between roles."""
```
**EN:** This block defines class `BaseTransferEngine`. Abstract transfer engine for data movement between roles. It inherits from `ABC`.
**CN:** 该代码块定义了类 `BaseTransferEngine`。 它用于封装 base transfer engine 相关行为。 它继承自 `ABC`。

### Lines 29-31: `supports_gpu_direct` implementation / `supports_gpu_direct` 实现
```python
    @property
    def supports_gpu_direct(self) -> bool:
        return False
```
**EN:** This block defines method `supports_gpu_direct` on `BaseTransferEngine`. It handles supports gpu direct logic.
**CN:** 该代码块定义了 `BaseTransferEngine` 的方法 `supports_gpu_direct`。 它用于处理 supports gpu direct 相关逻辑。

### Lines 33-35: `session_id` implementation / `session_id` 实现
```python
    @property
    @abstractmethod
    def session_id(self) -> str: ...
```
**EN:** This block defines method `session_id` on `BaseTransferEngine`. It handles session id logic.
**CN:** 该代码块定义了 `BaseTransferEngine` 的方法 `session_id`。 它用于处理 session id 相关逻辑。

### Lines 37-38: `register_buffer` implementation / `register_buffer` 实现
```python
    @abstractmethod
    def register_buffer(self, ptr: int, length: int) -> None: ...
```
**EN:** This block defines method `register_buffer` on `BaseTransferEngine`. It registers buffer. Parameters such as `ptr`, and `length` drive the behavior in this section.
**CN:** 该代码块定义了 `BaseTransferEngine` 的方法 `register_buffer`。 它用于注册buffer。 本段逻辑主要由 `ptr` 和 `length` 等参数驱动。

### Lines 40-41: `deregister_buffer` implementation / `deregister_buffer` 实现
```python
    @abstractmethod
    def deregister_buffer(self, ptr: int) -> None: ...
```
**EN:** This block defines method `deregister_buffer` on `BaseTransferEngine`. It handles deregister buffer logic. Parameters such as `ptr` drive the behavior in this section.
**CN:** 该代码块定义了 `BaseTransferEngine` 的方法 `deregister_buffer`。 它用于处理 deregister buffer 相关逻辑。 本段逻辑主要由 `ptr` 等参数驱动。

### Lines 43-47: `transfer_sync` implementation / `transfer_sync` 实现
```python
    @abstractmethod
    def transfer_sync(
        self, dst_session_id: str, src_addr: int, dst_addr: int, length: int
    ) -> int:
        """Returns 0 on success, negative on failure."""
```
**EN:** This block defines method `transfer_sync` on `BaseTransferEngine`. Returns 0 on success, negative on failure. Parameters such as `dst_session_id`, `src_addr`, `dst_addr`, and `length` drive the behavior in this section.
**CN:** 该代码块定义了 `BaseTransferEngine` 的方法 `transfer_sync`。 它用于处理 transfer sync 相关逻辑。 本段逻辑主要由 `dst_session_id`、`src_addr`、`dst_addr` 和 `length` 等参数驱动。

### Lines 49-56: `batch_transfer_sync` implementation / `batch_transfer_sync` 实现
```python
    @abstractmethod
    def batch_transfer_sync(
        self,
        dst_session_id: str,
        src_addrs: list[int],
        dst_addrs: list[int],
        lengths: list[int],
    ) -> int: ...
```
**EN:** This block defines method `batch_transfer_sync` on `BaseTransferEngine`. It handles batch transfer sync logic. Parameters such as `dst_session_id`, `src_addrs`, `dst_addrs`, and `lengths` drive the behavior in this section.
**CN:** 该代码块定义了 `BaseTransferEngine` 的方法 `batch_transfer_sync`。 它用于处理 batch transfer sync 相关逻辑。 本段逻辑主要由 `dst_session_id`、`src_addrs`、`dst_addrs` 和 `lengths` 等参数驱动。

### Lines 59-61: `MooncakeDiffusionEngine` class overview / `MooncakeDiffusionEngine` 类概览
```python
class MooncakeDiffusionEngine(BaseTransferEngine):
    """Production engine backed by MooncakeTransferEngine (RDMA)."""
```
**EN:** This block defines class `MooncakeDiffusionEngine`. Production engine backed by MooncakeTransferEngine (RDMA). It inherits from `BaseTransferEngine`.
**CN:** 该代码块定义了类 `MooncakeDiffusionEngine`。 它用于封装 mooncake diffusion engine 相关行为。 它继承自 `BaseTransferEngine`。

### Lines 62-64: `supports_gpu_direct` implementation / `supports_gpu_direct` 实现
```python
    @property
    def supports_gpu_direct(self) -> bool:
        return True
```
**EN:** This block defines method `supports_gpu_direct` on `MooncakeDiffusionEngine`. It handles supports gpu direct logic.
**CN:** 该代码块定义了 `MooncakeDiffusionEngine` 的方法 `supports_gpu_direct`。 它用于处理 supports gpu direct 相关逻辑。

### Lines 66-84: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        hostname: str,
        gpu_id: int = 0,
        ib_device: str | None = None,
    ):
        from sglang.srt.distributed.device_communicators.mooncake_transfer_engine import (
            MooncakeTransferEngine,
        )

        self._engine = MooncakeTransferEngine(
            hostname=hostname,
            gpu_id=gpu_id,
            ib_device=ib_device,
        )
        logger.info(
            "MooncakeDiffusionEngine initialized: session_id=%s",
            self._engine.session_id,
        )
```
**EN:** This block defines method `__init__` on `MooncakeDiffusionEngine`. It initializes the instance state. Key calls include `MooncakeTransferEngine`, and `logger.info`. Parameters such as `hostname`, `gpu_id`, and `ib_device` drive the behavior in this section.
**CN:** 该代码块定义了 `MooncakeDiffusionEngine` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `MooncakeTransferEngine` 和 `logger.info`。 本段逻辑主要由 `hostname`、`gpu_id` 和 `ib_device` 等参数驱动。

### Lines 86-88: `session_id` implementation / `session_id` 实现
```python
    @property
    def session_id(self) -> str:
        return self._engine.session_id
```
**EN:** This block defines method `session_id` on `MooncakeDiffusionEngine`. It handles session id logic.
**CN:** 该代码块定义了 `MooncakeDiffusionEngine` 的方法 `session_id`。 它用于处理 session id 相关逻辑。

### Lines 90-91: `register_buffer` implementation / `register_buffer` 实现
```python
    def register_buffer(self, ptr: int, length: int) -> None:
        self._engine.register(ptr, length)
```
**EN:** This block defines method `register_buffer` on `MooncakeDiffusionEngine`. It registers buffer. Key calls include `self._engine.register`. Parameters such as `ptr`, and `length` drive the behavior in this section.
**CN:** 该代码块定义了 `MooncakeDiffusionEngine` 的方法 `register_buffer`。 它用于注册buffer。 关键调用包括 `self._engine.register`。 本段逻辑主要由 `ptr` 和 `length` 等参数驱动。

### Lines 93-94: `deregister_buffer` implementation / `deregister_buffer` 实现
```python
    def deregister_buffer(self, ptr: int) -> None:
        self._engine.deregister(ptr)
```
**EN:** This block defines method `deregister_buffer` on `MooncakeDiffusionEngine`. It handles deregister buffer logic. Key calls include `self._engine.deregister`. Parameters such as `ptr` drive the behavior in this section.
**CN:** 该代码块定义了 `MooncakeDiffusionEngine` 的方法 `deregister_buffer`。 它用于处理 deregister buffer 相关逻辑。 关键调用包括 `self._engine.deregister`。 本段逻辑主要由 `ptr` 等参数驱动。

### Lines 96-99: `transfer_sync` implementation / `transfer_sync` 实现
```python
    def transfer_sync(
        self, dst_session_id: str, src_addr: int, dst_addr: int, length: int
    ) -> int:
        return self._engine.transfer_sync(dst_session_id, src_addr, dst_addr, length)
```
**EN:** This block defines method `transfer_sync` on `MooncakeDiffusionEngine`. It handles transfer sync logic. Key calls include `self._engine.transfer_sync`. Parameters such as `dst_session_id`, `src_addr`, `dst_addr`, and `length` drive the behavior in this section.
**CN:** 该代码块定义了 `MooncakeDiffusionEngine` 的方法 `transfer_sync`。 它用于处理 transfer sync 相关逻辑。 关键调用包括 `self._engine.transfer_sync`。 本段逻辑主要由 `dst_session_id`、`src_addr`、`dst_addr` 和 `length` 等参数驱动。

### Lines 101-110: `batch_transfer_sync` implementation / `batch_transfer_sync` 实现
```python
    def batch_transfer_sync(
        self,
        dst_session_id: str,
        src_addrs: list[int],
        dst_addrs: list[int],
        lengths: list[int],
    ) -> int:
        return self._engine.batch_transfer_sync(
            dst_session_id, src_addrs, dst_addrs, lengths
        )
```
**EN:** This block defines method `batch_transfer_sync` on `MooncakeDiffusionEngine`. It handles batch transfer sync logic. Key calls include `self._engine.batch_transfer_sync`. Parameters such as `dst_session_id`, `src_addrs`, `dst_addrs`, and `lengths` drive the behavior in this section.
**CN:** 该代码块定义了 `MooncakeDiffusionEngine` 的方法 `batch_transfer_sync`。 它用于处理 batch transfer sync 相关逻辑。 关键调用包括 `self._engine.batch_transfer_sync`。 本段逻辑主要由 `dst_session_id`、`src_addrs`、`dst_addrs` 和 `lengths` 等参数驱动。

### Lines 113-126: `create_transfer_engine` implementation / `create_transfer_engine` 实现
```python
def create_transfer_engine(
    hostname: str = "127.0.0.1",
    gpu_id: int = 0,
    ib_device: str | None = None,
) -> BaseTransferEngine:
    """Factory: returns MooncakeDiffusionEngine if mooncake is available."""
    if not _check_mooncake():
        raise RuntimeError(
            "Mooncake transfer engine is required for disaggregated diffusion "
            "but is not installed. Please install mooncake first."
        )
    return MooncakeDiffusionEngine(
        hostname=hostname, gpu_id=gpu_id, ib_device=ib_device
    )
```
**EN:** This block defines function `create_transfer_engine`. Factory: returns MooncakeDiffusionEngine if mooncake is available. Key calls include `MooncakeDiffusionEngine`, `_check_mooncake`, and `RuntimeError`. The implementation branches on conditions. Parameters such as `hostname`, `gpu_id`, and `ib_device` drive the behavior in this section.
**CN:** 该代码块定义了函数 `create_transfer_engine`。 它用于创建transfer engine。 关键调用包括 `MooncakeDiffusionEngine`、`_check_mooncake` 和 `RuntimeError`。 实现中包含条件分支。 本段逻辑主要由 `hostname`、`gpu_id` 和 `ib_device` 等参数驱动。

## Key Concepts / 关键概念
- `_check_mooncake`: Top-level function that checks mooncake. / 顶层函数，用于检查mooncake。
- `BaseTransferEngine`: Abstract transfer engine for data movement between roles. / 核心类，用于封装 base transfer engine 相关行为。
- `MooncakeDiffusionEngine`: Production engine backed by MooncakeTransferEngine (RDMA). / 核心类，用于封装 mooncake diffusion engine 相关行为。
- `create_transfer_engine`: Factory: returns MooncakeDiffusionEngine if mooncake is available. / 顶层函数，用于创建transfer engine。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `abc`
- **Internal modules / 内部模块**: `sglang.srt.distributed.device_communicators.mooncake_transfer_engine`

- **Total lines / 总行数**: 126
