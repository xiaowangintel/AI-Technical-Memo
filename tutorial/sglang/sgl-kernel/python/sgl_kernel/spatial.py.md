# spatial.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/python/sgl_kernel/spatial.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module wraps native operators related to `spatial`, performs Python-side validation, and exposes a convenient interface. / 该模块封装与 `spatial` 相关的原生算子，执行 Python 侧校验，并暴露便捷接口。

## Line-by-Line Analysis / 逐行分析
### Lines 1-2: Imports and module setup
````python
import torch
from torch.cuda.streams import ExternalStream
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 4-9: Optional dependency handling
````python
try:
    from . import spatial_ops  # triggers TORCH extension registration
except Exception as _e:
    _spatial_import_error = _e
else:
    _spatial_import_error = None
````
**EN:** This block attempts an operation that may fail, then provides fallback behavior so the module can keep working in reduced mode.
**CN:** 该部分尝试执行可能失败的操作，并提供回退行为，使模块能够以降级模式继续工作。

### Lines 11-13: Constants and configuration
````python
_IMPORT_ERROR = ImportError(
    "Failed to load sgl_kernel.spatial_ops extension. Ensure CUDA Driver >= 12.4"
)
````
**EN:** This block defines shared constants or configuration values such as `_IMPORT_ERROR`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `_IMPORT_ERROR`），供后续函数或控制流程复用。

### Lines 16-42: `create_greenctx_stream_by_value` definition
````python
def create_greenctx_stream_by_value(
    SM_a: int, SM_b: int, device_id: int = None
) -> tuple[ExternalStream, ExternalStream]:
    """
    Create two streams for greenctx.
    Args:
        sm_A (int): The SM of stream A.
        sm_B (int): The weight of stream B.
        device_id (int): The device id.
    Returns:
        tuple[ExternalStream, ExternalStream]: The two streams.
    """
    if _spatial_import_error is not None:
        raise _IMPORT_ERROR from _spatial_import_error
    if device_id is None:
        device_id = torch.cuda.current_device()

    res = torch.ops.sgl_kernel.create_greenctx_stream_by_value(SM_a, SM_b, device_id)

    stream_a = ExternalStream(
        stream_ptr=res[0], device=torch.device(f"cuda:{device_id}")
    )
    stream_b = ExternalStream(
        stream_ptr=res[1], device=torch.device(f"cuda:{device_id}")
    )

    return stream_a, stream_b
````
**EN:** This section defines `create_greenctx_stream_by_value` and implements the core logic associated with create GreenCtx stream by value. Docstring summary: Create two streams for greenctx. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `create_greenctx_stream_by_value`，并实现与 create GreenCtx stream by value 相关的核心逻辑。 文档字符串摘要：Create two streams for greenctx. 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 45-63: `get_sm_available` definition
````python
def get_sm_available(device_id: int = None) -> int:
    """
    Get the SMs available on the device.
    Args:
        device_id (int): The device id.
    Returns:
        int: The SMs available.
    """
    if _spatial_import_error is not None:
        raise _IMPORT_ERROR from _spatial_import_error
    if device_id is None:
        device_id = torch.cuda.current_device()

    device_props = torch.cuda.get_device_properties(device_id)

    # Get the number of Streaming Multiprocessors (SMs)
    sm_count = device_props.multi_processor_count

    return sm_count
````
**EN:** This section defines `get_sm_available`. It retrieves or computes the `sm available` path used by the module. Docstring summary: Get the SMs available on the device.
**CN:** 该部分定义 `get_sm_available`。它负责获取或计算模块中与 `sm available` 相关的处理路径。 文档字符串摘要：Get the SMs available on the device.

## Key Concepts / 关键概念
- **Role / 角色**: Python wrapper over compiled ops / 已编译算子的 Python 封装
- **Primary symbols / 主要符号**: `create_greenctx_stream_by_value`, `get_sm_available`
- **Core technologies / 核心技术**: compiled custom ops / 已编译自定义算子, PyTorch tensors / PyTorch 张量

## Dependencies / 依赖关系
- **Internal / 内部**: `.`
- **External / 外部**: `torch`, `torch.cuda.streams`
- **Runtime hooks / 运行时钩子**: `torch.ops.sgl_kernel` custom operator namespace
