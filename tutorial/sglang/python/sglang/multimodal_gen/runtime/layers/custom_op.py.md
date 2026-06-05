# custom_op.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/custom_op.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `CustomOp`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `CustomOp` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 6-13: module setup and imports / 模块初始化与导入
```python
from collections.abc import Callable
from typing import Any

import torch.nn as nn

from sglang.kernel_api_logging import debug_kernel_api
from sglang.multimodal_gen.runtime.platforms import current_platform
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
```
**EN:** This block establishes the module context and imports `collections.abc`, `typing`, `torch.nn`, `sglang.kernel_api_logging`, `sglang.multimodal_gen.runtime.platforms`, and `sglang.multimodal_gen.runtime.utils.logging_utils`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `collections.abc`、`typing`、`torch.nn`、`sglang.kernel_api_logging`、`sglang.multimodal_gen.runtime.platforms` 和 `sglang.multimodal_gen.runtime.utils.logging_utils`。这些依赖为后续实现提供所需符号。

### Lines 15-16: supporting statements / 辅助语句
```python
logger = init_logger(__name__)
_is_cuda = current_platform.is_cuda()
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`, and `_is_cuda`. The code collaborates with `init_logger`, and `current_platform.is_cuda`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 和 `_is_cuda` 等名称。 代码会与 `init_logger` 和 `current_platform.is_cuda` 协同工作。

### Lines 19-24: `CustomOp` class overview / `CustomOp` 类概览
```python
class CustomOp(nn.Module):
    """
    Base class for custom ops.
    Dispatches the forward method to the appropriate backend.
    """
```
**EN:** This block defines class `CustomOp`. Base class for custom ops. Dispatches the forward method to the appropriate backend. It inherits from `nn.Module`.
**CN:** 该代码块定义了类 `CustomOp`。 它用于封装 custom op 相关行为。 它继承自 `nn.Module`。

### Lines 25-27: `__init__` implementation / `__init__` 实现
```python
    def __init__(self) -> None:
        super().__init__()
        self._forward_method = self.dispatch_forward()
```
**EN:** This block defines method `__init__` on `CustomOp`. It initializes the instance state. Key calls include `super.__init__`, `self.dispatch_forward`, and `super`.
**CN:** 该代码块定义了 `CustomOp` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`self.dispatch_forward` 和 `super`。

### Lines 29-31: `forward` implementation / `forward` 实现
```python
    @debug_kernel_api
    def forward(self, *args, **kwargs) -> Any:
        return self._forward_method(*args, **kwargs)
```
**EN:** This block defines method `forward` on `CustomOp`. It executes function. Key calls include `self._forward_method`.
**CN:** 该代码块定义了 `CustomOp` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `self._forward_method`。

### Lines 33-39: `forward_native` implementation / `forward_native` 实现
```python
    def forward_native(self, *args, **kwargs) -> Any:
        """PyTorch-native implementation of the forward method.
        This method is optional. If implemented, it can be used with compilers
        such as torch.compile or PyTorch XLA. Also, it can be used for testing
        purposes.
        """
        raise NotImplementedError
```
**EN:** This block defines method `forward_native` on `CustomOp`. PyTorch-native implementation of the forward method. This method is optional.
**CN:** 该代码块定义了 `CustomOp` 的方法 `forward_native`。 它用于执行前向计算native。

### Lines 41-42: `forward_cuda` implementation / `forward_cuda` 实现
```python
    def forward_cuda(self, *args, **kwargs) -> Any:
        raise NotImplementedError
```
**EN:** This block defines method `forward_cuda` on `CustomOp`. It executes cuda.
**CN:** 该代码块定义了 `CustomOp` 的方法 `forward_cuda`。 它用于执行前向计算cuda。

### Lines 44-46: `forward_hip` implementation / `forward_hip` 实现
```python
    def forward_hip(self, *args, **kwargs) -> Any:
        # ROCm kernels follow the CUDA path by default.
        return self.forward_cuda(*args, **kwargs)
```
**EN:** This block defines method `forward_hip` on `CustomOp`. It executes hip. Key calls include `self.forward_cuda`.
**CN:** 该代码块定义了 `CustomOp` 的方法 `forward_hip`。 它用于执行前向计算hip。 关键调用包括 `self.forward_cuda`。

### Lines 48-50: `forward_cpu` implementation / `forward_cpu` 实现
```python
    def forward_cpu(self, *args, **kwargs) -> Any:
        # By default, we assume that CPU ops are compatible with CUDA ops.
        return self.forward_cuda(*args, **kwargs)
```
**EN:** This block defines method `forward_cpu` on `CustomOp`. It executes cpu. Key calls include `self.forward_cuda`.
**CN:** 该代码块定义了 `CustomOp` 的方法 `forward_cpu`。 它用于执行前向计算cpu。 关键调用包括 `self.forward_cuda`。

### Lines 52-56: `forward_tpu` implementation / `forward_tpu` 实现
```python
    def forward_tpu(self, *args, **kwargs) -> Any:
        # By default, we assume that TPU ops are compatible with the
        # PyTorch-native implementation.
        # NOTE(woosuk): This is a placeholder for future extensions.
        return self.forward_native(*args, **kwargs)
```
**EN:** This block defines method `forward_tpu` on `CustomOp`. It executes tpu. Key calls include `self.forward_native`.
**CN:** 该代码块定义了 `CustomOp` 的方法 `forward_tpu`。 它用于执行前向计算tpu。 关键调用包括 `self.forward_native`。

### Lines 58-60: `forward_musa` implementation / `forward_musa` 实现
```python
    def forward_musa(self, *args, **kwargs) -> Any:
        # MUSA kernels follow the CUDA path by default.
        return self.forward_cuda(*args, **kwargs)
```
**EN:** This block defines method `forward_musa` on `CustomOp`. It executes musa. Key calls include `self.forward_cuda`.
**CN:** 该代码块定义了 `CustomOp` 的方法 `forward_musa`。 它用于执行前向计算musa。 关键调用包括 `self.forward_cuda`。

### Lines 62-65: `forward_oot` implementation / `forward_oot` 实现
```python
    def forward_oot(self, *args, **kwargs) -> Any:
        # By default, we assume that OOT ops are compatible with the
        # PyTorch-native implementation.
        return self.forward_native(*args, **kwargs)
```
**EN:** This block defines method `forward_oot` on `CustomOp`. It executes oot. Key calls include `self.forward_native`.
**CN:** 该代码块定义了 `CustomOp` 的方法 `forward_oot`。 它用于执行前向计算oot。 关键调用包括 `self.forward_native`。

### Lines 67-70: `forward_npu` implementation / `forward_npu` 实现
```python
    def forward_npu(self, *args, **kwargs) -> Any:
        # By default, we assume that NPU ops are compatible with the
        # PyTorch-native implementation.
        return self.forward_native(*args, **kwargs)
```
**EN:** This block defines method `forward_npu` on `CustomOp`. It executes npu. Key calls include `self.forward_native`.
**CN:** 该代码块定义了 `CustomOp` 的方法 `forward_npu`。 它用于执行前向计算npu。 关键调用包括 `self.forward_native`。

### Lines 72-84: `dispatch_forward` implementation / `dispatch_forward` 实现
```python
    def dispatch_forward(self) -> Callable:
        if _is_cuda:
            return self.forward_cuda
        elif current_platform.is_hip():
            return self.forward_hip
        elif current_platform.is_npu():
            return self.forward_npu
        elif current_platform.is_xpu():
            return self.forward_xpu
        elif current_platform.is_musa():
            return self.forward_musa
        else:
            return self.forward_native
```
**EN:** This block defines method `dispatch_forward` on `CustomOp`. It handles dispatch forward logic. Key calls include `current_platform.is_hip`, `current_platform.is_npu`, `current_platform.is_xpu`, and `current_platform.is_musa`. The implementation branches on conditions.
**CN:** 该代码块定义了 `CustomOp` 的方法 `dispatch_forward`。 它用于处理 dispatch forward 相关逻辑。 关键调用包括 `current_platform.is_hip`、`current_platform.is_npu`、`current_platform.is_xpu` 和 `current_platform.is_musa`。 实现中包含条件分支。

### Lines 86-89: `enabled` implementation / `enabled` 实现
```python
    @classmethod
    def enabled(cls) -> bool:
        # since we are not using Inductor, we always return True
        return True
```
**EN:** This block defines method `enabled` on `CustomOp`. It handles enabled logic.
**CN:** 该代码块定义了 `CustomOp` 的方法 `enabled`。 它用于处理 enabled 相关逻辑。

### Lines 91-97: `default_on` implementation / `default_on` 实现
```python
    @staticmethod
    def default_on() -> bool:
        """
        On by default if level < CompilationLevel.PIECEWISE
        Specifying 'all' or 'none' in custom_op takes precedence.
        """
        raise NotImplementedError
```
**EN:** This block defines method `default_on` on `CustomOp`. On by default if level < CompilationLevel.PIECEWISE Specifying 'all' or 'none' in custom_op takes precedence.
**CN:** 该代码块定义了 `CustomOp` 的方法 `default_on`。 它用于处理 default on 相关逻辑。

### Lines 104-104: supporting statements / 辅助语句
```python
    op_registry: dict[str, type["CustomOp"]] = {}
```
**EN:** This block gathers supporting statements inside `CustomOp`. It updates names such as `op_registry`.
**CN:** 该代码块汇集了位于 `CustomOp` 内部的辅助语句。 它会更新 `op_registry` 等名称。

### Lines 107-116: `register` implementation / `register` 实现
```python
    @classmethod
    def register(cls, name: str) -> Callable:

        def decorator(op_cls):
            assert name not in cls.op_registry, f"Duplicate op name: {name}"
            op_cls.name = name
            cls.op_registry[name] = op_cls
            return op_cls

        return decorator
```
**EN:** This block defines method `register` on `CustomOp`. It registers function. Parameters such as `name` drive the behavior in this section.
**CN:** 该代码块定义了 `CustomOp` 的方法 `register`。 它用于注册函数。 本段逻辑主要由 `name` 等参数驱动。

## Key Concepts / 关键概念
- `CustomOp`: Base class for custom ops. / 核心类，用于封装 custom op 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `collections.abc`, `typing`
- **Third-party / 第三方依赖**: `torch.nn`
- **Internal modules / 内部模块**: `sglang.kernel_api_logging`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 116
