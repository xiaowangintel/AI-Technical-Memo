# ssu_dispatch.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/mamba/ops/ssu_dispatch.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines the main ssu dispatch classes used by the SGLang attention stack, together with helper routines that support runtime execution. / 该模块定义 SGLang 注意力栈使用的主要 ssu dispatch 类，并提供支撑运行时执行的辅助例程。
## Line-by-Line Analysis / 逐行分析
### Lines 1-7: imports
```python
from __future__ import annotations

import logging
from abc import ABC, abstractmethod
from typing import TYPE_CHECKING

import torch
```
**EN:** Imports the external and internal dependencies consumed by the code that follows.
**CN:** 导入后续代码所依赖的外部与内部模块。

### Lines 9-10: TYPE_CHECKING branch
```python
if TYPE_CHECKING:
    from sglang.srt.server_args import ServerArgs
```
**EN:** Loads type-only imports to improve static analysis without adding extra runtime dependencies.
**CN:** 加载仅供类型检查使用的导入，以改进静态分析而不增加额外运行时依赖。

### Lines 12-12: module constants
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 15-15: class MambaSSUBackend
```python
class MambaSSUBackend(ABC):
```
**EN:** Concrete attention backend that connects mamba ssubackend to SGLang runtime interfaces, metadata preparation, and kernel dispatch.
**CN:** 该具体注意力后端将 mamba ssubackend 与 SGLang 的运行时接口、元数据准备和内核分发连接起来。

### Lines 16-19: method MambaSSUBackend.name
```python
    @property
    @abstractmethod
    def name(self) -> str:
        """Human-readable name used for logging."""
```
**EN:** Implements the name routine used by this attention module.
**CN:** 实现该注意力模块使用的 name 例程。

### Lines 21-42: method MambaSSUBackend.__call__
```python
    @abstractmethod
    def __call__(
        self,
        state: torch.Tensor,
        x: torch.Tensor,
        dt: torch.Tensor,
        A: torch.Tensor,
        B: torch.Tensor,
        C: torch.Tensor,
        D: torch.Tensor | None = None,
        z: torch.Tensor | None = None,
        dt_bias: torch.Tensor | None = None,
        dt_softplus: bool = False,
        state_batch_indices: torch.Tensor | None = None,
        pad_slot_id: int = -1,
        out: torch.Tensor | None = None,
        disable_state_update: bool = False,
        intermediate_states_buffer: torch.Tensor | None = None,
        cache_steps: int | None = None,
        retrieve_parent_token: torch.Tensor | None = None,
        intermediate_state_indices: torch.Tensor | None = None,
    ) -> None: ...
```
**EN:** Runs the forward-path logic for call, transforming tensors and dispatching the required compute steps.
**CN:** 执行 call 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 45-47: class TritonSSUBackend
```python
class TritonSSUBackend(MambaSSUBackend):
    """Triton-based selective-state-update backend."""
```
**EN:** Concrete attention backend that connects triton ssubackend to SGLang runtime interfaces, metadata preparation, and kernel dispatch.
**CN:** 该具体注意力后端将 triton ssubackend 与 SGLang 的运行时接口、元数据准备和内核分发连接起来。

### Lines 48-53: method TritonSSUBackend.__init__
```python
    def __init__(self) -> None:
        from sglang.srt.layers.attention.mamba.ops.mamba_ssm import (
            selective_state_update,
        )

        self._kernel = selective_state_update
```
**EN:** Initializes the TritonSSUBackend instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 TritonSSUBackend 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 55-57: method TritonSSUBackend.name
```python
    @property
    def name(self) -> str:
        return "triton"
```
**EN:** Implements the name routine used by this attention module.
**CN:** 实现该注意力模块使用的 name 例程。

### Lines 59-99: method TritonSSUBackend.__call__
```python
    def __call__(
        self,
        state: torch.Tensor,
        x: torch.Tensor,
        dt: torch.Tensor,
        A: torch.Tensor,
        B: torch.Tensor,
        C: torch.Tensor,
        D: torch.Tensor | None = None,
        z: torch.Tensor | None = None,
        dt_bias: torch.Tensor | None = None,
        dt_softplus: bool = False,
        state_batch_indices: torch.Tensor | None = None,
        pad_slot_id: int = -1,
        out: torch.Tensor | None = None,
        disable_state_update: bool = False,
        intermediate_states_buffer: torch.Tensor | None = None,
        cache_steps: int | None = None,
        retrieve_parent_token: torch.Tensor | None = None,
        intermediate_state_indices: torch.Tensor | None = None,
    ) -> None:
        self._kernel(
            state,
            x,
            dt,
            A,
            B,
            C,
            D=D,
            z=z,
            dt_bias=dt_bias,
            dt_softplus=dt_softplus,
            state_batch_indices=state_batch_indices,
            pad_slot_id=pad_slot_id,
            out=out,
            disable_state_update=disable_state_update,
            intermediate_states_buffer=intermediate_states_buffer,
            cache_steps=cache_steps,
            retrieve_parent_token=retrieve_parent_token,
            intermediate_state_indices=intermediate_state_indices,
        )
```
**EN:** Runs the forward-path logic for call, transforming tensors and dispatching the required compute steps.
**CN:** 执行 call 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 102-104: class FlashInferSSUBackend
```python
class FlashInferSSUBackend(MambaSSUBackend):
    """FlashInfer-based selective-state-update backend."""
```
**EN:** Concrete attention backend that connects flash infer ssubackend to SGLang runtime interfaces, metadata preparation, and kernel dispatch.
**CN:** 该具体注意力后端将 flash infer ssubackend 与 SGLang 的运行时接口、元数据准备和内核分发连接起来。

### Lines 105-108: method FlashInferSSUBackend.__init__
```python
    def __init__(self) -> None:
        from flashinfer.mamba import selective_state_update

        self._kernel = selective_state_update
```
**EN:** Initializes the FlashInferSSUBackend instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 FlashInferSSUBackend 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 110-112: method FlashInferSSUBackend.name
```python
    @property
    def name(self) -> str:
        return "flashinfer"
```
**EN:** Implements the name routine used by this attention module.
**CN:** 实现该注意力模块使用的 name 例程。

### Lines 114-159: method FlashInferSSUBackend.__call__
```python
    def __call__(
        self,
        state: torch.Tensor,
        x: torch.Tensor,
        dt: torch.Tensor,
        A: torch.Tensor,
        B: torch.Tensor,
        C: torch.Tensor,
        D: torch.Tensor | None = None,
        z: torch.Tensor | None = None,
        dt_bias: torch.Tensor | None = None,
        dt_softplus: bool = False,
        state_batch_indices: torch.Tensor | None = None,
        pad_slot_id: int = -1,
        out: torch.Tensor | None = None,
        disable_state_update: bool = False,
        intermediate_states_buffer: torch.Tensor | None = None,
        cache_steps: int | None = None,
        retrieve_parent_token: torch.Tensor | None = None,
        intermediate_state_indices: torch.Tensor | None = None,
    ) -> None:
        if retrieve_parent_token is not None:
            raise ValueError(
                "FlashInfer backend does not support retrieve_parent_token. "
                "Use --mamba-backend triton for EAGLE tree attention."
            )
        # FlashInfer expects cache_steps as an int (0 when unused).
        self._kernel(
            state,
            x,
            dt,
            A,
            B,
            C,
            D=D,
            z=z,
            dt_bias=dt_bias,
            dt_softplus=dt_softplus,
            state_batch_indices=state_batch_indices,
            pad_slot_id=pad_slot_id,
            out=out,
            disable_state_update=disable_state_update,
            intermediate_states_buffer=intermediate_states_buffer,
            cache_steps=0 if cache_steps is None else cache_steps,
            intermediate_state_indices=intermediate_state_indices,
        )
```
**EN:** Runs the forward-path logic for call, transforming tensors and dispatching the required compute steps.
**CN:** 执行 call 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 162-167: module constants
```python
_BACKEND_REGISTRY: dict[str, type[MambaSSUBackend]] = {
    "triton": TritonSSUBackend,
    "flashinfer": FlashInferSSUBackend,
}

_mamba_ssu_backend: MambaSSUBackend | None = None
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 170-204: function initialize_mamba_selective_state_update_backend
```python
def initialize_mamba_selective_state_update_backend(server_args: ServerArgs) -> None:
    """Instantiate the selective-state-update backend from server config.

    This should be called once during scheduler initialization.

    Args:
        server_args: Server arguments containing ``mamba_backend`` setting.

    Raises:
        ValueError: If the requested backend is unavailable or cannot be imported.
    """
    global _mamba_ssu_backend

    requested = server_args.mamba_backend or "triton"

    backend_cls = _BACKEND_REGISTRY.get(requested)
    if backend_cls is None:
        raise ValueError(
            f"Unknown mamba backend '{requested}'. "
            f"Available backends: {list(_BACKEND_REGISTRY.keys())}"
        )

    try:
        _mamba_ssu_backend = backend_cls()
    except ImportError:
        raise ValueError(
            f"Mamba backend '{requested}' requested but its dependencies are not "
            f"available. Install the required package or use a different "
            f"--mamba-backend value."
        )

    logger.debug(
        "Mamba selective_state_update backend initialized: %s",
        _mamba_ssu_backend.name,
    )
```
**EN:** Implements the initialize mamba selective state update backend routine used by this attention module.
**CN:** 实现该注意力模块使用的 initialize mamba selective state update backend 例程。

### Lines 207-277: function selective_state_update
```python
def selective_state_update(
    state: torch.Tensor,
    x: torch.Tensor,
    dt: torch.Tensor,
    A: torch.Tensor,
    B: torch.Tensor,
    C: torch.Tensor,
    D: torch.Tensor | None = None,
    z: torch.Tensor | None = None,
    dt_bias: torch.Tensor | None = None,
    dt_softplus: bool = False,
    state_batch_indices: torch.Tensor | None = None,
    pad_slot_id: int = -1,
    out: torch.Tensor | None = None,
    disable_state_update: bool = False,
    intermediate_states_buffer: torch.Tensor | None = None,
    cache_steps: int | None = None,
    retrieve_parent_token: torch.Tensor | None = None,
    intermediate_state_indices: torch.Tensor | None = None,
) -> None:
    """Dispatch selective-state-update to the configured backend.

    This function provides a unified interface regardless of the underlying
    backend. Backend-specific argument adaptation is handled inside each
    :class:`MambaSSUBackend` subclass.

    Args:
        state: SSM state tensor (batch, nheads, dim, dstate)
        x: Input tensor
        dt: Delta time tensor
        A: A matrix
        B: B matrix
# ... omitted 27 lines ...
        z=z,
        dt_bias=dt_bias,
        dt_softplus=dt_softplus,
        state_batch_indices=state_batch_indices,
        pad_slot_id=pad_slot_id,
        out=out,
        disable_state_update=disable_state_update,
        intermediate_states_buffer=intermediate_states_buffer,
        cache_steps=cache_steps,
        retrieve_parent_token=retrieve_parent_token,
        intermediate_state_indices=intermediate_state_indices,
    )
```
**EN:** Implements the selective state update routine used by this attention module.
**CN:** 实现该注意力模块使用的 selective state update 例程。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** FlashInfer execution path / **CN:** FlashInfer 执行路径
- **EN:** Speculative decoding support / **CN:** 投机解码支持
- **EN:** Position encoding handling / **CN:** 位置编码处理

## Dependencies / 依赖关系
- `__future__.annotations`
- `logging`
- `abc.ABC`
- `abc.abstractmethod`
- `typing.TYPE_CHECKING`
- `torch`
- `sglang.srt.server_args.ServerArgs`
- `sglang.srt.layers.attention.mamba.ops.mamba_ssm.selective_state_update`
- `flashinfer.mamba.selective_state_update`
