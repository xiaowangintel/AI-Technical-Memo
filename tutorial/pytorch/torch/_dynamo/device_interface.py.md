# device_interface.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_dynamo/device_interface.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Implements torch.compile front-end capture, guard management, and graph/lowering helpers in TorchDynamo. The module docstring emphasizes: Device abstraction layer for TorchDynamo and Inductor backends.
- **Purpose (CN)**: 实现 TorchDynamo 中的 torch.compile 前端捕获、守卫管理与图/降级辅助逻辑。 模块文档字符串重点说明了该实现的职责。

## Line-by-Line Analysis / 逐行分析

### Lines 1-27
```python
"""
Device abstraction layer for TorchDynamo and Inductor backends.

This module provides a unified interface for different hardware backends (CUDA, XPU,
CPU, MPS, MTIA) through a common device interface. Key components include:

- DeviceInterface: Base class defining the common API for all device types
- Device-specific implementations: CudaInterface, XpuInterface, CpuInterface, MpsInterface, MtiaInterface
- Device registration system for managing available backends
- Worker APIs for multi-processing scenarios
- Stream and event management across different devices
- Device property caching for worker processes

The abstraction layer enables device-agnostic code in TorchDynamo while allowing
specialized implementations for each hardware backend's unique features.
"""

import inspect
import time
from collections import namedtuple
from collections.abc import Callable, Iterable
from dataclasses import dataclass
from typing import Any, Literal

import torch
from torch.utils._pallas import has_torch_tpu
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 28-49
```python

get_cuda_stream: Callable[[int], int] | None
if torch.cuda._is_compiled():
    from torch._C import _cuda_getCurrentRawStream as get_cuda_stream
else:
    get_cuda_stream = None

# Recording the device properties in the main process but used in worker process.
caching_worker_device_properties: dict[str, Any] = {}
caching_worker_current_devices: dict[str, int] = {}


class DeviceInterface:
    """
    This is a simple device runtime interface for Inductor. It enables custom
    backends to be integrated with Inductor in a device-agnostic semantic.
    """

    class device:
        def __new__(cls, device: torch.types.Device) -> Any:
            raise NotImplementedError
```
- **EN**: This module-level block helps hand work to a compiler/backend pipeline. It also validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于将工作移交给编译器或后端流水线。 同时它还会在继续前校验关键假设；根据运行时条件分支处理。

### Lines 50-73
```python
    class Event:
        def __new__(cls, *args: Any, **kwargs: Any) -> Any:
            raise NotImplementedError(
                "Event should be inherited from torch.Event, otherwise, it couldn't be captured by dynamo."
            )

    class Stream:
        def __new__(cls, *args: Any, **kwargs: Any) -> Any:
            raise NotImplementedError(
                "Stream should be inherited from torch.Stream, otherwise, it couldn't be captured by dynamo."
            )

    class Worker:
        """
        Worker API to query device properties that will work in multi processing
        workers that cannot use the GPU APIs (due to processing fork() and
        initialization time issues). Properties are recorded in the main process
        before we fork the workers.
        """

        @staticmethod
        def set_device(device: int) -> None:
            raise NotImplementedError
```
- **EN**: These decorators register or transform the following definition so it can initialize learnable tensors and related state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够初始化可学习张量及相关状态。

### Lines 74-97
```python
        @staticmethod
        def current_device() -> int:
            raise NotImplementedError

        @staticmethod
        def get_device_properties(device: torch.types.Device = None) -> Any:
            raise NotImplementedError

    @staticmethod
    def current_device() -> int:
        raise NotImplementedError

    @staticmethod
    def set_device(device: torch.types.Device) -> None:
        raise NotImplementedError

    @staticmethod
    def maybe_exchange_device(device: int) -> int:
        raise NotImplementedError

    @staticmethod
    def exchange_device(device: int) -> int:
        raise NotImplementedError
```
- **EN**: These decorators register or transform the following definition so it can normalize dtype/device related arguments and behavior.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够规范化 dtype/device 相关参数与行为。

### Lines 98-121
```python
    @staticmethod
    def device_count() -> int:
        raise NotImplementedError

    @staticmethod
    def is_available() -> bool:
        raise NotImplementedError

    @staticmethod
    def stream(stream: torch.Stream) -> Any:
        raise NotImplementedError

    @staticmethod
    def current_stream() -> torch.Stream:
        raise NotImplementedError

    @staticmethod
    def set_stream(stream: torch.Stream) -> None:
        raise NotImplementedError

    @staticmethod
    def _set_stream_by_id(stream_id: int, device_index: int, device_type: int) -> None:
        raise NotImplementedError
```
- **EN**: These decorators register or transform the following definition so it can normalize dtype/device related arguments and behavior.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够规范化 dtype/device 相关参数与行为。

### Lines 122-147
```python
    @staticmethod
    def get_raw_stream(device_idx: int) -> int:
        raise NotImplementedError

    @staticmethod
    def synchronize(device: torch.types.Device = None) -> None:
        raise NotImplementedError

    @classmethod
    def get_device_properties(cls, device: torch.types.Device = None) -> Any:
        return cls.Worker.get_device_properties(device)

    @staticmethod
    def get_compute_capability(device: torch.types.Device = None) -> Any:
        raise NotImplementedError

    @staticmethod
    def is_bf16_supported(including_emulation: bool = False) -> bool:
        raise NotImplementedError

    @classmethod
    def is_dtype_supported(
        cls, dtype: torch.dtype, including_emulation: bool = False
    ) -> bool:
        return dtype != torch.bfloat16 or cls.is_bf16_supported(including_emulation)
```
- **EN**: These decorators register or transform the following definition so it can normalize dtype/device related arguments and behavior.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够规范化 dtype/device 相关参数与行为。

### Lines 148-173
```python
    @staticmethod
    def memory_allocated(device: torch.types.Device = None) -> int:
        raise NotImplementedError

    @staticmethod
    def is_triton_capable(device: torch.types.Device = None) -> bool:
        """
        Returns True if the device has Triton support, False otherwise, even if
        the appropriate Triton backend is not available.
        """
        return False

    @classmethod
    def raise_if_triton_unavailable(cls, device: torch.types.Device = None) -> None:
        """
        Raises a `RuntimeError` with the appropriate human-readable instructions
        to resolve the issue if Triton is not available for the given device, or
        the default device if `device` is `None`.

        The caller should ensure the presence of the 'triton' package before
        calling this method.
        """
        if not cls.is_triton_capable():
            raise RuntimeError("This device is not capable of supporting Triton")
```
- **EN**: These decorators register or transform the following definition so it can hand work to a compiler/backend pipeline.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将工作移交给编译器或后端流水线。

### Lines 174-200
```python
class DeviceGuard:
    """
    This class provides a context manager for device switching. This is a stripped
    down version of torch.{device_name}.device.

    The context manager changes the current device to the given device index
    on entering the context and restores the original device on exiting.
    The device is switched using the provided device interface.
    """

    def __init__(
        self, device_interface: type[DeviceInterface], index: int | None
    ) -> None:
        self.device_interface = device_interface
        self.idx = index
        self.prev_idx = -1

    def __enter__(self) -> None:
        if self.idx is not None:
            self.prev_idx = self.device_interface.exchange_device(self.idx)

    def __exit__(self, type: Any, value: Any, traceback: Any) -> Literal[False]:
        if self.idx is not None:
            self.idx = self.device_interface.maybe_exchange_device(self.prev_idx)
        return False
```
- **EN**: This module-level block helps enforce guards that validate whether cached compiled code can be reused. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于实施守卫检查以判断缓存的编译代码能否复用。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 201-220
```python
class CudaInterface(DeviceInterface):
    device = torch.cuda.device  # type: ignore[assignment]

    # register Event and Stream class into the backend interface
    # make sure Event and Stream are implemented and inherited from the torch.Event and torch.Stream
    Event = torch.cuda.Event  # type: ignore[assignment]
    Stream = torch.cuda.Stream  # type: ignore[assignment]

    # pyrefly: ignore [bad-override]
    class Worker:
        @staticmethod
        def set_device(device: int) -> None:
            caching_worker_current_devices["cuda"] = device

        @staticmethod
        def current_device() -> int:
            if "cuda" in caching_worker_current_devices:
                return caching_worker_current_devices["cuda"]
            return torch.cuda.current_device()
```
- **EN**: These decorators register or transform the following definition so it can hand work to a compiler/backend pipeline.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将工作移交给编译器或后端流水线。

### Lines 221-240
```python
        @staticmethod
        def get_device_properties(device: torch.types.Device = None) -> Any:
            if device is not None:
                if isinstance(device, str):
                    device = torch.device(device)
                    assert device.type == "cuda"
                if isinstance(device, torch.device):
                    device = device.index
            if device is None:
                device = CudaInterface.Worker.current_device()

            if "cuda" not in caching_worker_device_properties:
                device_prop = [
                    torch.cuda.get_device_properties(i)
                    for i in range(torch.cuda.device_count())
                ]
                caching_worker_device_properties["cuda"] = device_prop

            return caching_worker_device_properties["cuda"][device]
```
- **EN**: These decorators register or transform the following definition so it can normalize dtype/device related arguments and behavior.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够规范化 dtype/device 相关参数与行为。

### Lines 241-260
```python
    current_device = staticmethod(torch.cuda.current_device)
    set_device = staticmethod(torch.cuda.set_device)
    device_count = staticmethod(torch.cuda.device_count)
    stream = staticmethod(torch.cuda.stream)  # type: ignore[assignment]
    current_stream = staticmethod(torch.cuda.current_stream)
    set_stream = staticmethod(torch.cuda.set_stream)  # type: ignore[assignment]
    _set_stream_by_id = staticmethod(torch.cuda._set_stream_by_id)  # type: ignore[assignment]
    synchronize = staticmethod(torch.cuda.synchronize)
    get_device_properties = staticmethod(torch.cuda.get_device_properties)  # type: ignore[assignment]
    get_raw_stream = staticmethod(get_cuda_stream)  # type: ignore[assignment, arg-type]
    exchange_device = staticmethod(torch.cuda._exchange_device)  # type: ignore[arg-type, has-type]
    maybe_exchange_device = staticmethod(torch.cuda._maybe_exchange_device)  # type: ignore[arg-type, has-type]
    memory_allocated = staticmethod(torch.cuda.memory_allocated)
    is_bf16_supported = staticmethod(torch.cuda.is_bf16_supported)  # type: ignore[arg-type]

    # Can be mock patched by @patch decorator.
    @staticmethod
    def is_available() -> bool:
        return torch.cuda.is_available()
```
- **EN**: These decorators register or transform the following definition so it can normalize dtype/device related arguments and behavior.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够规范化 dtype/device 相关参数与行为。

### Lines 261-285
```python
    @staticmethod
    def get_compute_capability(device: torch.types.Device = None) -> int | str:
        if torch.version.hip is None:
            major, min = torch.cuda.get_device_capability(device)
            return major * 10 + min
        else:
            return torch.cuda.get_device_properties(device).gcnArchName.split(":", 1)[0]

    @staticmethod
    def is_triton_capable(device: torch.types.Device = None) -> bool:
        return (
            torch.version.hip is not None
            or torch.cuda.get_device_properties(device).major >= 7
        )

    @staticmethod
    def raise_if_triton_unavailable(device: torch.types.Device = None) -> None:
        from torch._inductor.exc import GPUTooOldForTriton

        if not CudaInterface.is_triton_capable(device):
            device_props = torch.cuda.get_device_properties(device)
            raise GPUTooOldForTriton(device_props, inspect.currentframe())

        import triton.backends
```
- **EN**: These decorators register or transform the following definition so it can hand work to a compiler/backend pipeline.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将工作移交给编译器或后端流水线。

### Lines 286-310
```python
        if torch.version.hip is not None:
            if "amd" not in triton.backends.backends:
                raise RuntimeError("triton not built with the 'amd' backend")
        elif "nvidia" not in triton.backends.backends:
            raise RuntimeError("triton not built with the 'nvidia' backend")


get_mtia_stream: Callable[[int], int] | None
if torch.mtia._is_compiled():
    from torch._C import _mtia_getCurrentRawStream as get_mtia_stream
else:
    get_mtia_stream = None


class MtiaInterface(DeviceInterface):
    device = torch.mtia.device  # type: ignore[assignment]
    Event = torch.mtia.Event  # type: ignore[assignment]
    Stream = torch.mtia.Stream  # type: ignore[assignment]

    # pyrefly: ignore [bad-override]
    class Worker:
        @staticmethod
        def set_device(device: int) -> None:
            caching_worker_current_devices["mtia"] = device
```
- **EN**: These decorators register or transform the following definition so it can hand work to a compiler/backend pipeline.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将工作移交给编译器或后端流水线。

### Lines 311-336
```python
        @staticmethod
        def current_device() -> int:
            if "mtia" in caching_worker_current_devices:
                return caching_worker_current_devices["mtia"]
            return torch.mtia.current_device()

        @staticmethod
        def get_device_properties(device: torch.types.Device = None) -> Any:
            if device is not None:
                if isinstance(device, str):
                    device = torch.device(device)
                    assert device.type == "mtia"
                if isinstance(device, torch.device):
                    device = device.index
            if device is None:
                device = MtiaInterface.Worker.current_device()

            if "mtia" not in caching_worker_device_properties:
                device_prop = [
                    torch.mtia.get_device_properties(i)
                    for i in range(torch.mtia.device_count())
                ]
                caching_worker_device_properties["mtia"] = device_prop

            return caching_worker_device_properties["mtia"][device]
```
- **EN**: These decorators register or transform the following definition so it can normalize dtype/device related arguments and behavior.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够规范化 dtype/device 相关参数与行为。

### Lines 337-362
```python
    current_device = staticmethod(torch.mtia.current_device)
    set_device = staticmethod(torch.mtia.set_device)  # type: ignore[assignment]
    device_count = staticmethod(torch.mtia.device_count)
    stream = staticmethod(torch.mtia.stream)  # type: ignore[assignment]
    current_stream = staticmethod(torch.mtia.current_stream)
    set_stream = staticmethod(torch.mtia.set_stream)  # type: ignore[assignment]
    _set_stream_by_id = staticmethod(torch.mtia._set_stream_by_id)  # type: ignore[assignment]
    synchronize = staticmethod(torch.mtia.synchronize)
    get_device_properties = staticmethod(torch.mtia.get_device_properties)  # type: ignore[assignment]
    get_raw_stream = staticmethod(get_mtia_stream)  # type: ignore[assignment, arg-type]
    exchange_device = staticmethod(torch.mtia._exchange_device)  # type: ignore[arg-type, has-type]
    maybe_exchange_device = staticmethod(torch.mtia._maybe_exchange_device)  # type: ignore[arg-type, has-type]
    memory_allocated = staticmethod(torch.mtia.memory_allocated)  # type: ignore[assignment]
    is_bf16_supported = staticmethod(torch.mtia.is_bf16_supported)  # type: ignore[arg-type]

    # Can be mock patched by @patch decorator.
    @staticmethod
    def is_available() -> bool:
        ret = torch.mtia.is_available()
        return ret

    @staticmethod
    def get_compute_capability(device: torch.types.Device = None) -> Any:
        cc = torch.mtia.get_device_capability(device)
        return cc
```
- **EN**: These decorators register or transform the following definition so it can normalize dtype/device related arguments and behavior.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够规范化 dtype/device 相关参数与行为。

### Lines 363-386
```python
    @staticmethod
    def is_triton_capable(device: torch.types.Device = None) -> bool:
        return True

    @staticmethod
    def raise_if_triton_unavailable(device: torch.types.Device = None) -> None:
        import triton.backends

        if "mtia" not in triton.backends.backends:
            raise RuntimeError("triton not built with the 'mtia' backend")


get_xpu_stream: Callable[[int], int] | None
if torch.xpu._is_compiled():
    from torch._C import _xpu_getCurrentRawStream as get_xpu_stream
else:
    get_xpu_stream = None


class XpuInterface(DeviceInterface):
    device = torch.xpu.device  # type: ignore[assignment]
    Event = torch.xpu.Event  # type: ignore[assignment]
    Stream = torch.xpu.Stream  # type: ignore[assignment]
```
- **EN**: These decorators register or transform the following definition so it can hand work to a compiler/backend pipeline.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将工作移交给编译器或后端流水线。

### Lines 387-409
```python
    # pyrefly: ignore [bad-override]
    class Worker:
        @staticmethod
        def set_device(device: int) -> None:
            caching_worker_current_devices["xpu"] = device

        @staticmethod
        def current_device() -> int:
            if "xpu" in caching_worker_current_devices:
                return caching_worker_current_devices["xpu"]
            return torch.xpu.current_device()

        @staticmethod
        def get_device_properties(device: torch.types.Device = None) -> Any:
            if device is not None:
                if isinstance(device, str):
                    device = torch.device(device)
                    assert device.type == "xpu"
                if isinstance(device, torch.device):
                    device = device.index
            if device is None:
                device = XpuInterface.Worker.current_device()
```
- **EN**: These decorators register or transform the following definition so it can normalize dtype/device related arguments and behavior.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够规范化 dtype/device 相关参数与行为。

### Lines 410-432
```python
            if "xpu" not in caching_worker_device_properties:
                device_prop = [
                    torch.xpu.get_device_properties(i)
                    for i in range(torch.xpu.device_count())
                ]
                caching_worker_device_properties["xpu"] = device_prop

            return caching_worker_device_properties["xpu"][device]

    current_device = staticmethod(torch.xpu.current_device)
    set_device = staticmethod(torch.xpu.set_device)
    device_count = staticmethod(torch.xpu.device_count)  # type: ignore[has-type]
    stream = staticmethod(torch.xpu.stream)  # type: ignore[assignment]
    current_stream = staticmethod(torch.xpu.current_stream)
    set_stream = staticmethod(torch.xpu.set_stream)  # type: ignore[assignment]
    _set_stream_by_id = staticmethod(torch.xpu._set_stream_by_id)  # type: ignore[assignment]
    synchronize = staticmethod(torch.xpu.synchronize)
    get_device_properties = staticmethod(torch.xpu.get_device_properties)  # type: ignore[assignment]
    get_raw_stream = staticmethod(get_xpu_stream)  # type: ignore[assignment, arg-type]
    exchange_device = staticmethod(torch.xpu._exchange_device)  # type: ignore[arg-type, has-type]
    maybe_exchange_device = staticmethod(torch.xpu._maybe_exchange_device)  # type: ignore[arg-type, has-type]
    memory_allocated = staticmethod(torch.xpu.memory_allocated)
```
- **EN**: This block continues `XpuInterface` and works to normalize dtype/device related arguments and behavior. It also computes and returns an intermediate/result value; iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `XpuInterface`，用于规范化 dtype/device 相关参数与行为。 同时它还会计算并返回中间值或结果；遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 433-458
```python
    # Can be mock patched by @patch decorator.
    @staticmethod
    def is_available() -> bool:
        return torch.xpu.is_available()

    @staticmethod
    def get_compute_capability(device: torch.types.Device = None) -> Any:
        cc = torch.xpu.get_device_capability(device)
        return cc

    @staticmethod
    def is_bf16_supported(including_emulation: bool = False) -> bool:
        return torch.xpu.is_bf16_supported()

    @staticmethod
    def is_triton_capable(device: torch.types.Device = None) -> bool:
        return True

    @staticmethod
    def raise_if_triton_unavailable(device: torch.types.Device = None) -> None:
        import triton.backends

        if "intel" not in triton.backends.backends:
            raise RuntimeError("triton not built with the 'intel' backend")
```
- **EN**: These decorators register or transform the following definition so it can hand work to a compiler/backend pipeline.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将工作移交给编译器或后端流水线。

### Lines 459-483
```python
@dataclass
class CpuDeviceProperties:
    multi_processor_count: int


class CpuInterface(DeviceInterface):
    # pyrefly: ignore [bad-override]
    class Event(torch.Event):
        def __init__(self, enable_timing: bool = True) -> None:
            self.time = 0.0

        def elapsed_time(self, other: Any) -> float:
            return (other.time - self.time) * 1000

        def record(self, stream: Any = None) -> None:
            self.time = time.perf_counter()

    # pyrefly: ignore [bad-override]
    class Worker:
        @staticmethod
        def get_device_properties(
            device: torch.types.Device = None,
        ) -> CpuDeviceProperties:
            import multiprocessing
```
- **EN**: These decorators register or transform the following definition so it can initialize learnable tensors and related state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够初始化可学习张量及相关状态。

### Lines 484-510
```python
            cpu_count = multiprocessing.cpu_count()
            return CpuDeviceProperties(cpu_count)

    @staticmethod
    def is_available() -> bool:
        return True

    @staticmethod
    def is_bf16_supported(including_emulation: bool = False) -> bool:
        return True

    @staticmethod
    def get_compute_capability(device: torch.types.Device = None) -> str:
        return ""

    @staticmethod
    def get_raw_stream(device_idx: Any) -> int:
        return 0

    @staticmethod
    def current_device() -> int:
        return 0

    @staticmethod
    def synchronize(device: torch.types.Device = None) -> None:
        pass
```
- **EN**: These decorators register or transform the following definition so it can normalize dtype/device related arguments and behavior.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够规范化 dtype/device 相关参数与行为。

### Lines 511-535
```python
    @staticmethod
    def is_triton_capable(device: torch.types.Device = None) -> bool:
        return True

    @staticmethod
    def raise_if_triton_unavailable(device: torch.types.Device = None) -> None:
        import triton.backends

        if "cpu" not in triton.backends.backends:
            raise RuntimeError("triton not built with the 'cpu' backend")


class MpsInterface(DeviceInterface):
    @staticmethod
    def is_bf16_supported(including_emulation: bool = False) -> bool:
        return True

    @classmethod
    def is_dtype_supported(
        cls, dtype: torch.dtype, including_emulation: bool = False
    ) -> bool:
        if dtype in [torch.float64, torch.complex128]:
            return False
        return True
```
- **EN**: These decorators register or transform the following definition so it can hand work to a compiler/backend pipeline.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将工作移交给编译器或后端流水线。

### Lines 536-559
```python
    @staticmethod
    def is_available() -> bool:
        return torch.backends.mps.is_available()

    @staticmethod
    def current_device() -> int:
        return 0

    @staticmethod
    def get_compute_capability(device: torch.types.Device = None) -> str:
        return ""

    @staticmethod
    def synchronize(device: torch.types.Device = None) -> None:
        torch.mps.synchronize()

    # pyrefly: ignore [bad-override]
    class Worker:
        @staticmethod
        def get_device_properties(device: torch.types.Device = None) -> Any:
            return namedtuple("MPSProperties", ["multi_processor_count"])(
                torch.backends.mps.get_core_count()  # type: ignore[arg-type]
            )
```
- **EN**: These decorators register or transform the following definition so it can hand work to a compiler/backend pipeline.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将工作移交给编译器或后端流水线。

### Lines 560-585
```python
        @staticmethod
        def current_device() -> int:
            return 0


class TpuInterface(DeviceInterface):
    @staticmethod
    def is_bf16_supported(including_emulation: bool = False) -> bool:
        return True

    @classmethod
    def is_dtype_supported(
        cls, dtype: torch.dtype, including_emulation: bool = False
    ) -> bool:
        return dtype not in (
            torch.float64,
            torch.complex32,
            torch.complex64,
            torch.complex128,
            torch.half,
        )

    @staticmethod
    def is_available() -> bool:
        return has_torch_tpu()
```
- **EN**: These decorators register or transform the following definition so it can normalize dtype/device related arguments and behavior.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够规范化 dtype/device 相关参数与行为。

### Lines 586-610
```python
    @staticmethod
    def current_device() -> int:
        return 0

    @staticmethod
    def get_compute_capability(device: torch.types.Device = None) -> str:
        return ""

    # pyrefly: ignore [bad-override]
    class Worker:
        @staticmethod
        def get_device_properties(device: torch.types.Device = None) -> Any:
            return namedtuple("TPUProperties", ["multi_processor_count"])(
                1  # type: ignore[arg-type]
            )

        @staticmethod
        def current_device() -> int:
            return 0


device_interfaces: dict[str, type[DeviceInterface]] = {}
_device_initialized = False
```
- **EN**: These decorators register or transform the following definition so it can initialize learnable tensors and related state.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够初始化可学习张量及相关状态。

### Lines 611-634
```python
def register_interface_for_device(
    device: str | torch.device, device_interface: type[DeviceInterface]
) -> None:
    if isinstance(device, torch.device):
        device = device.type
    device_interfaces[device] = device_interface


def get_interface_for_device(device: str | torch.device) -> type[DeviceInterface]:
    if isinstance(device, torch.device):
        device = device.type
    if not _device_initialized:
        init_device_reg()
    if device in device_interfaces:
        return device_interfaces[device]
    raise NotImplementedError(f"No interface for device {device}")


def get_registered_device_interfaces() -> Iterable[tuple[str, type[DeviceInterface]]]:
    if not _device_initialized:
        init_device_reg()
    return device_interfaces.items()
```
- **EN**: This module-level block helps initialize learnable tensors and related state. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于初始化可学习张量及相关状态。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 635-652
```python
def init_device_reg() -> None:
    global _device_initialized
    register_interface_for_device("cuda", CudaInterface)
    for i in range(torch.cuda.device_count()):
        register_interface_for_device(f"cuda:{i}", CudaInterface)

    register_interface_for_device("xpu", XpuInterface)
    for i in range(torch.xpu.device_count()):
        register_interface_for_device(f"xpu:{i}", XpuInterface)

    register_interface_for_device("mtia", MtiaInterface)
    for i in range(torch.mtia.device_count()):
        register_interface_for_device(f"mtia:{i}", MtiaInterface)

    register_interface_for_device("cpu", CpuInterface)
    register_interface_for_device("mps", MpsInterface)
    register_interface_for_device("tpu", TpuInterface)
```
- **EN**: Defines the `init_device_reg` function; this block introduces logic that initialize learnable tensors and related state.
- **CN**: 定义`init_device_reg` 函数；该代码块引入了用于初始化可学习张量及相关状态的逻辑。

### Lines 653-653
```python
    _device_initialized = True
```
- **EN**: This block continues `init_device_reg` and works to initialize learnable tensors and related state.
- **CN**: 该代码块继续实现 `init_device_reg`，用于初始化可学习张量及相关状态。

## Key Concepts / 关键概念

- **Graph capture / 图捕获**
  - EN: The file participates in converting eager Python execution into compilable graph form.
  - CN: 该文件参与把即时执行的 Python 程序转换为可编译的图形式。
- **Guards / 守卫**
  - EN: Runtime predicates preserve correctness when cached compiled graphs are reused.
  - CN: 运行时谓词用于在复用缓存编译图时保持正确性。
- **Tracing / 跟踪**
  - EN: The file captures Python-level operations and turns them into analyzable graph structures.
  - CN: 该文件会捕获 Python 层操作，并将其转换为可分析的图结构。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Compiler handoff / 编译器交接**
  - EN: The implementation prepares captured work for backend compilation or execution.
  - CN: 该实现会把捕获到的工作整理后交给后端编译或执行。
- **Python execution capture / Python 执行捕获**
  - EN: The file hooks or rewrites Python execution machinery to observe user programs.
  - CN: 该文件会挂接或改写 Python 执行机制，以观察用户程序。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch.utils._pallas`, `torch._C`, `torch._inductor.exc`
- **Standard library / 标准库**: `inspect`, `time`, `collections`, `collections.abc`, `dataclasses`, `typing`, `multiprocessing`
- **Third-party packages / 第三方包**: `triton.backends`
- **Primary symbols / 核心符号**: `DeviceInterface`, `DeviceGuard`, `CudaInterface`, `MtiaInterface`, `XpuInterface`, `CpuDeviceProperties`, `CpuInterface`, `MpsInterface`, `TpuInterface`, `register_interface_for_device`, `get_interface_for_device`, `get_registered_device_interfaces`
- **Relationship summary / 关系总结**: Dependencies center on graph capture, guard checking, bytecode analysis, and backend handoff. / 依赖关系主要围绕图捕获、守卫检查、字节码分析和后端交接展开。
