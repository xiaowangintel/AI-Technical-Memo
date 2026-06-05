# xpu_model_runner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/xpu_model_runner.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `XPUModelRunner`, `XPUModelRunnerV2`, `_torch_cuda_wrapper` for the V1 `worker` subsystem. / 为 V1 的 `worker` 子系统实现 `XPUModelRunner`, `XPUModelRunnerV2`, `_torch_cuda_wrapper`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
from contextlib import contextmanager

import torch

from vllm.config import VllmConfig
from vllm.utils.torch_utils import supports_xpu_graph
from vllm.v1.worker.gpu.model_runner import (
    GPUModelRunner as GPUModelRunnerV2,
)
from vllm.v1.worker.gpu_model_runner import GPUModelRunner
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `torch`, and internal vLLM modules such as `vllm.config`, `vllm.utils.torch_utils`, `vllm.v1.worker.gpu.model_runner`, `vllm.v1.worker.gpu_model_runner`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`torch` 等外部依赖，以及 `vllm.config`, `vllm.utils.torch_utils`, `vllm.v1.worker.gpu.model_runner`, `vllm.v1.worker.gpu_model_runner` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `XPUModelRunner` class / `XPUModelRunner` 类
```python
class XPUModelRunner(GPUModelRunner):
    """A model runner for XPU devices."""
```
**EN:** Introduces the `XPUModelRunner` class on top of `GPUModelRunner`. Core methods include `__init__`. Docstring signal: A model runner for XPU devices.
**CN:** 这里定义 `XPUModelRunner` 类，其基类包括 `GPUModelRunner`。核心方法包括 `__init__`。

### `XPUModelRunner.__init__` method / `XPUModelRunner.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        device: torch.device,
    ):
        with _torch_cuda_wrapper():
            super().__init__(vllm_config, device)
        # FIXME: To be verified.
        self.cascade_attn_enabled = False
```
**EN:** This method initializes the object state within `XPUModelRunner`. Key calls include `_torch_cuda_wrapper`, `__init__`, `super`. It touches state such as `cascade_attn_enabled`.
**CN:** 该方法会初始化对象状态，其作用域位于`XPUModelRunner`。 关键调用包括 `_torch_cuda_wrapper`, `__init__`, `super`。 它会读写 `cascade_attn_enabled` 等状态。

### `XPUModelRunnerV2` class / `XPUModelRunnerV2` 类
```python
class XPUModelRunnerV2(GPUModelRunnerV2):
    """A model runner for XPU devices."""
```
**EN:** Introduces the `XPUModelRunnerV2` class on top of `GPUModelRunnerV2`. Core methods include `__init__`. Docstring signal: A model runner for XPU devices.
**CN:** 这里定义 `XPUModelRunnerV2` 类，其基类包括 `GPUModelRunnerV2`。核心方法包括 `__init__`。

### `XPUModelRunnerV2.__init__` method / `XPUModelRunnerV2.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        device: torch.device,
    ):
        with _torch_cuda_wrapper():
            super().__init__(vllm_config, device)
```
**EN:** This method initializes the object state within `XPUModelRunnerV2`. Key calls include `_torch_cuda_wrapper`, `__init__`, `super`.
**CN:** 该方法会初始化对象状态，其作用域位于`XPUModelRunnerV2`。 关键调用包括 `_torch_cuda_wrapper`, `__init__`, `super`。

### `_torch_cuda_wrapper` function / `_torch_cuda_wrapper` 函数
```python
@contextmanager
def _torch_cuda_wrapper():
    # replace cuda APIs with xpu APIs, this should work by default
    torch.cuda.Stream = torch.xpu.Stream
    torch.cuda.default_stream = torch.xpu.current_stream
    torch.cuda.current_stream = torch.xpu.current_stream
    torch.cuda.stream = torch.xpu.stream
    torch.cuda.mem_get_info = torch.xpu.mem_get_info
    torch.cuda.Event = torch.Event
    torch.cuda.set_stream = torch.xpu.set_stream
    if supports_xpu_graph():
        torch.cuda.graph = torch.xpu.graph
        torch.cuda.CUDAGraph = torch.xpu.XPUGraph
        torch.cuda.graph_pool_handle = torch.xpu.graph_pool_handle
    yield
```
**EN:** This function implements `_torch_cuda_wrapper` within the module. Key calls include `supports_xpu_graph`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_torch_cuda_wrapper`，其作用域位于the module。 关键调用包括 `supports_xpu_graph`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `XPUModelRunner`: central class or interface in this module. / `XPUModelRunner`：本模块中的核心类或接口。
- `XPUModelRunnerV2`: central class or interface in this module. / `XPUModelRunnerV2`：本模块中的核心类或接口。
- `_torch_cuda_wrapper`: top-level helper or orchestration entry point. / `_torch_cuda_wrapper`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `contextlib`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.utils.torch_utils`, `vllm.v1.worker.gpu.model_runner`, `vllm.v1.worker.gpu_model_runner`
