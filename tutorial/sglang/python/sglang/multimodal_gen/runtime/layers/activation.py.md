# activation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/activation.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `SiluAndMul`, `GeluAndMul`, and `NewGELU`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Custom activation functions. / 该文件属于运行时算子层。它围绕 `SiluAndMul`、`GeluAndMul` 和 `NewGELU` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 5-14: module setup and imports / 模块初始化与导入
```python
"""Custom activation functions."""

import math
from typing import Any

import torch
import torch.nn as nn
import torch.nn.functional as F

from sglang.multimodal_gen.runtime.platforms import current_platform
```
**EN:** This block establishes the module context and imports `math`, `typing`, `torch`, `torch.nn`, `torch.nn.functional`, and `sglang.multimodal_gen.runtime.platforms`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `math`、`typing`、`torch`、`torch.nn`、`torch.nn.functional` 和 `sglang.multimodal_gen.runtime.platforms`。这些依赖为后续实现提供所需符号。

### Lines 16-28: supporting statements / 辅助语句
```python
_is_cuda = current_platform.is_cuda()
_is_hip = current_platform.is_hip()
_is_npu = current_platform.is_npu()
if _is_cuda:
    from sglang.jit_kernel.activation import silu_and_mul
elif _is_hip:
    from sgl_kernel import silu_and_mul


if _is_npu:
    import torch_npu
# TODO (will): remove this dependency
from sglang.multimodal_gen.runtime.layers.custom_op import CustomOp
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `_is_cuda`, `_is_hip`, and `_is_npu`. The code collaborates with `current_platform.is_cuda`, `current_platform.is_hip`, and `current_platform.is_npu`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `_is_cuda`、`_is_hip` 和 `_is_npu` 等名称。 代码会与 `current_platform.is_cuda`、`current_platform.is_hip` 和 `current_platform.is_npu` 协同工作。

### Lines 32-41: `SiluAndMul` class overview / `SiluAndMul` 类概览
```python
class SiluAndMul(CustomOp):
    """An activation function for SwiGLU.

    The function computes x -> silu(x[:d]) * x[d:] where d = x.shape[-1] // 2.

    Shapes:
        x: (num_tokens, 2 * d) or (batch_size, seq_len, 2 * d)
        return: (num_tokens, d) or (batch_size, seq_len, d)
    """
```
**EN:** This block defines class `SiluAndMul`. An activation function for SwiGLU. The function computes x -> silu(x[:d]) * x[d:] where d = x.shape[-1] // 2. It inherits from `CustomOp`.
**CN:** 该代码块定义了类 `SiluAndMul`。 它用于封装 silu and mul 相关行为。 它继承自 `CustomOp`。

### Lines 42-43: `__init__` implementation / `__init__` 实现
```python
    def __init__(self) -> None:
        super().__init__()
```
**EN:** This block defines method `__init__` on `SiluAndMul`. It initializes the instance state. Key calls include `super.__init__`, and `super`.
**CN:** 该代码块定义了 `SiluAndMul` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__` 和 `super`。

### Lines 45-50: `forward_cuda` implementation / `forward_cuda` 实现
```python
    def forward_cuda(self, x: torch.Tensor) -> torch.Tensor:
        d = x.shape[-1] // 2
        output_shape = x.shape[:-1] + (d,)
        out = torch.empty(output_shape, dtype=x.dtype, device=x.device)
        silu_and_mul(x, out)
        return out
```
**EN:** This block defines method `forward_cuda` on `SiluAndMul`. It executes cuda. Key calls include `torch.empty`, and `silu_and_mul`. Parameters such as `x` drive the behavior in this section.
**CN:** 该代码块定义了 `SiluAndMul` 的方法 `forward_cuda`。 它用于执行前向计算cuda。 关键调用包括 `torch.empty` 和 `silu_and_mul`。 本段逻辑主要由 `x` 等参数驱动。

### Lines 52-55: `forward_native` implementation / `forward_native` 实现
```python
    def forward_native(self, x: torch.Tensor) -> torch.Tensor:
        """PyTorch-native implementation equivalent to forward()."""
        d = x.shape[-1] // 2
        return F.silu(x[..., :d]) * x[..., d:]
```
**EN:** This block defines method `forward_native` on `SiluAndMul`. PyTorch-native implementation equivalent to forward(). Key calls include `F.silu`. Parameters such as `x` drive the behavior in this section.
**CN:** 该代码块定义了 `SiluAndMul` 的方法 `forward_native`。 它用于执行前向计算native。 关键调用包括 `F.silu`。 本段逻辑主要由 `x` 等参数驱动。

### Lines 57-59: `forward_npu` implementation / `forward_npu` 实现
```python
    def forward_npu(self, x: torch.Tensor) -> torch.Tensor:
        out = torch_npu.npu_swiglu(x)
        return out
```
**EN:** This block defines method `forward_npu` on `SiluAndMul`. It executes npu. Key calls include `torch_npu.npu_swiglu`. Parameters such as `x` drive the behavior in this section.
**CN:** 该代码块定义了 `SiluAndMul` 的方法 `forward_npu`。 它用于执行前向计算npu。 关键调用包括 `torch_npu.npu_swiglu`。 本段逻辑主要由 `x` 等参数驱动。

### Lines 61-62: `forward_musa` implementation / `forward_musa` 实现
```python
    def forward_musa(self, x: torch.Tensor) -> torch.Tensor:
        return nn.SwishGLU()(x)
```
**EN:** This block defines method `forward_musa` on `SiluAndMul`. It executes musa. Key calls include `nn.SwishGLU`. Parameters such as `x` drive the behavior in this section.
**CN:** 该代码块定义了 `SiluAndMul` 的方法 `forward_musa`。 它用于执行前向计算musa。 关键调用包括 `nn.SwishGLU`。 本段逻辑主要由 `x` 等参数驱动。

### Lines 66-75: `GeluAndMul` class overview / `GeluAndMul` 类概览
```python
class GeluAndMul(CustomOp):
    """An activation function for GeGLU.

    The function computes x -> GELU(x[:d]) * x[d:] where d = x.shape[-1] // 2.

    Shapes:
        x: (batch_size, seq_len, 2 * d) or (num_tokens, 2 * d)
        return: (batch_size, seq_len, d) or (num_tokens, d)
    """
```
**EN:** This block defines class `GeluAndMul`. An activation function for GeGLU. The function computes x -> GELU(x[:d]) * x[d:] where d = x.shape[-1] // 2. It inherits from `CustomOp`.
**CN:** 该代码块定义了类 `GeluAndMul`。 它用于封装 gelu and mul 相关行为。 它继承自 `CustomOp`。

### Lines 76-80: `__init__` implementation / `__init__` 实现
```python
    def __init__(self, approximate: str = "none"):
        super().__init__()
        self.approximate = approximate
        if approximate not in ("none", "tanh"):
            raise ValueError(f"Unknown approximate mode: {approximate}")
```
**EN:** This block defines method `__init__` on `GeluAndMul`. It initializes the instance state. Key calls include `super.__init__`, `ValueError`, and `super`. The implementation branches on conditions. Parameters such as `approximate` drive the behavior in this section.
**CN:** 该代码块定义了 `GeluAndMul` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`ValueError` 和 `super`。 实现中包含条件分支。 本段逻辑主要由 `approximate` 等参数驱动。

### Lines 82-83: `forward_cuda` implementation / `forward_cuda` 实现
```python
    def forward_cuda(self, *args, **kwargs) -> Any:
        return self.forward_native(*args, **kwargs)
```
**EN:** This block defines method `forward_cuda` on `GeluAndMul`. It executes cuda. Key calls include `self.forward_native`.
**CN:** 该代码块定义了 `GeluAndMul` 的方法 `forward_cuda`。 它用于执行前向计算cuda。 关键调用包括 `self.forward_native`。

### Lines 85-92: `forward_npu` implementation / `forward_npu` 实现
```python
    def forward_npu(self, x: torch.Tensor) -> torch.Tensor:
        y_npu, _ = torch_npu.npu_geglu(
            x,
            dim=-1,
            approximate=1 if self.approximate == "tanh" else 0,
            activate_left=True,
        )
        return y_npu
```
**EN:** This block defines method `forward_npu` on `GeluAndMul`. It executes npu. Key calls include `torch_npu.npu_geglu`. Parameters such as `x` drive the behavior in this section.
**CN:** 该代码块定义了 `GeluAndMul` 的方法 `forward_npu`。 它用于执行前向计算npu。 关键调用包括 `torch_npu.npu_geglu`。 本段逻辑主要由 `x` 等参数驱动。

### Lines 94-97: `forward_native` implementation / `forward_native` 实现
```python
    def forward_native(self, x: torch.Tensor) -> torch.Tensor:
        """PyTorch-native implementation equivalent to forward()."""
        d = x.shape[-1] // 2
        return F.gelu(x[..., :d], approximate=self.approximate) * x[..., d:]
```
**EN:** This block defines method `forward_native` on `GeluAndMul`. PyTorch-native implementation equivalent to forward(). Key calls include `F.gelu`. Parameters such as `x` drive the behavior in this section.
**CN:** 该代码块定义了 `GeluAndMul` 的方法 `forward_native`。 它用于执行前向计算native。 关键调用包括 `F.gelu`。 本段逻辑主要由 `x` 等参数驱动。

### Lines 99-100: `extra_repr` implementation / `extra_repr` 实现
```python
    def extra_repr(self) -> str:
        return f"approximate={repr(self.approximate)}"
```
**EN:** This block defines method `extra_repr` on `GeluAndMul`. It handles extra repr logic. Key calls include `repr`.
**CN:** 该代码块定义了 `GeluAndMul` 的方法 `extra_repr`。 它用于处理 extra repr 相关逻辑。 关键调用包括 `repr`。

### Lines 104-105: `NewGELU` class overview / `NewGELU` 类概览
```python
class NewGELU(CustomOp):
```
**EN:** This block defines class `NewGELU`. It encapsulates new gelu behavior. It inherits from `CustomOp`.
**CN:** 该代码块定义了类 `NewGELU`。 它用于封装 new gelu 相关行为。 它继承自 `CustomOp`。

### Lines 106-107: `__init__` implementation / `__init__` 实现
```python
    def __init__(self):
        super().__init__()
```
**EN:** This block defines method `__init__` on `NewGELU`. It initializes the instance state. Key calls include `super.__init__`, and `super`.
**CN:** 该代码块定义了 `NewGELU` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__` 和 `super`。

### Lines 109-110: `forward_cuda` implementation / `forward_cuda` 实现
```python
    def forward_cuda(self, *args, **kwargs) -> Any:
        return self.forward_native(*args, **kwargs)
```
**EN:** This block defines method `forward_cuda` on `NewGELU`. It executes cuda. Key calls include `self.forward_native`.
**CN:** 该代码块定义了 `NewGELU` 的方法 `forward_cuda`。 它用于执行前向计算cuda。 关键调用包括 `self.forward_native`。

### Lines 112-113: `forward_xpu` implementation / `forward_xpu` 实现
```python
    def forward_xpu(self, *args, **kwargs) -> Any:
        return self.forward_native(*args, **kwargs)
```
**EN:** This block defines method `forward_xpu` on `NewGELU`. It executes xpu. Key calls include `self.forward_native`.
**CN:** 该代码块定义了 `NewGELU` 的方法 `forward_xpu`。 它用于执行前向计算xpu。 关键调用包括 `self.forward_native`。

### Lines 115-118: `forward_native` implementation / `forward_native` 实现
```python
    def forward_native(self, x: torch.Tensor) -> torch.Tensor:
        """PyTorch-native implementation equivalent to forward()."""
        c = math.sqrt(2.0 / math.pi)
        return 0.5 * x * (1.0 + torch.tanh(c * (x + 0.044715 * torch.pow(x, 3.0))))
```
**EN:** This block defines method `forward_native` on `NewGELU`. PyTorch-native implementation equivalent to forward(). Key calls include `math.sqrt`, `torch.tanh`, and `torch.pow`. Parameters such as `x` drive the behavior in this section.
**CN:** 该代码块定义了 `NewGELU` 的方法 `forward_native`。 它用于执行前向计算native。 关键调用包括 `math.sqrt`、`torch.tanh` 和 `torch.pow`。 本段逻辑主要由 `x` 等参数驱动。

### Lines 122-123: `QuickGELU` class overview / `QuickGELU` 类概览
```python
class QuickGELU(CustomOp):
    # https://github.com/huggingface/transformers/blob/main/src/transformers/activations.py#L90
```
**EN:** This block defines class `QuickGELU`. It encapsulates quick gelu behavior. It inherits from `CustomOp`.
**CN:** 该代码块定义了类 `QuickGELU`。 它用于封装 quick gelu 相关行为。 它继承自 `CustomOp`。

### Lines 124-125: `__init__` implementation / `__init__` 实现
```python
    def __init__(self):
        super().__init__()
```
**EN:** This block defines method `__init__` on `QuickGELU`. It initializes the instance state. Key calls include `super.__init__`, and `super`.
**CN:** 该代码块定义了 `QuickGELU` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__` 和 `super`。

### Lines 127-128: `forward_cuda` implementation / `forward_cuda` 实现
```python
    def forward_cuda(self, *args, **kwargs) -> Any:
        return self.forward_native(*args, **kwargs)
```
**EN:** This block defines method `forward_cuda` on `QuickGELU`. It executes cuda. Key calls include `self.forward_native`.
**CN:** 该代码块定义了 `QuickGELU` 的方法 `forward_cuda`。 它用于执行前向计算cuda。 关键调用包括 `self.forward_native`。

### Lines 130-131: `forward_xpu` implementation / `forward_xpu` 实现
```python
    def forward_xpu(self, *args, **kwargs) -> Any:
        return self.forward_native(*args, **kwargs)
```
**EN:** This block defines method `forward_xpu` on `QuickGELU`. It executes xpu. Key calls include `self.forward_native`.
**CN:** 该代码块定义了 `QuickGELU` 的方法 `forward_xpu`。 它用于执行前向计算xpu。 关键调用包括 `self.forward_native`。

### Lines 133-135: `forward_native` implementation / `forward_native` 实现
```python
    def forward_native(self, x: torch.Tensor) -> torch.Tensor:
        """PyTorch-native implementation equivalent to forward()."""
        return x * torch.sigmoid(1.702 * x)
```
**EN:** This block defines method `forward_native` on `QuickGELU`. PyTorch-native implementation equivalent to forward(). Key calls include `torch.sigmoid`. Parameters such as `x` drive the behavior in this section.
**CN:** 该代码块定义了 `QuickGELU` 的方法 `forward_native`。 它用于执行前向计算native。 关键调用包括 `torch.sigmoid`。 本段逻辑主要由 `x` 等参数驱动。

### Lines 138-145: supporting statements / 辅助语句
```python
_ACTIVATION_REGISTRY = {
    "gelu": nn.GELU,
    "gelu_new": NewGELU,
    "gelu_pytorch_tanh": lambda: nn.GELU(approximate="tanh"),
    "relu": nn.ReLU,
    "silu": nn.SiLU,
    "quick_gelu": QuickGELU,
}
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `_ACTIVATION_REGISTRY`. The code collaborates with `nn.GELU`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `_ACTIVATION_REGISTRY` 等名称。 代码会与 `nn.GELU` 协同工作。

### Lines 148-154: `get_act_fn` implementation / `get_act_fn` 实现
```python
def get_act_fn(act_fn_name: str) -> nn.Module:
    """Get an activation function by name."""
    act_fn_name = act_fn_name.lower()
    if act_fn_name not in _ACTIVATION_REGISTRY:
        raise ValueError(f"Activation function {act_fn_name!r} is not supported.")

    return _ACTIVATION_REGISTRY[act_fn_name]()
```
**EN:** This block defines function `get_act_fn`. Get an activation function by name. Key calls include `act_fn_name.lower`, `_ACTIVATION_REGISTRY`, and `ValueError`. The implementation branches on conditions. Parameters such as `act_fn_name` drive the behavior in this section.
**CN:** 该代码块定义了函数 `get_act_fn`。 它用于获取act fn。 关键调用包括 `act_fn_name.lower`、`_ACTIVATION_REGISTRY` 和 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `act_fn_name` 等参数驱动。

### Lines 157-160: supporting statements / 辅助语句
```python
_ACTIVATION_AND_MUL_REGISTRY = {
    "gelu": GeluAndMul,
    "silu": SiluAndMul,
}
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `_ACTIVATION_AND_MUL_REGISTRY`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `_ACTIVATION_AND_MUL_REGISTRY` 等名称。

### Lines 163-169: `get_act_and_mul_fn` implementation / `get_act_and_mul_fn` 实现
```python
def get_act_and_mul_fn(act_fn_name: str) -> nn.Module:
    """Get an activation-and-mul (i.e. SiluAndMul) function by name."""
    act_fn_name = act_fn_name.lower()
    if act_fn_name not in _ACTIVATION_AND_MUL_REGISTRY:
        raise ValueError(f"Activation function {act_fn_name!r} is not supported.")

    return _ACTIVATION_AND_MUL_REGISTRY[act_fn_name]()
```
**EN:** This block defines function `get_act_and_mul_fn`. Get an activation-and-mul (i.e. SiluAndMul) function by name. Key calls include `act_fn_name.lower`, `_ACTIVATION_AND_MUL_REGISTRY`, and `ValueError`. The implementation branches on conditions. Parameters such as `act_fn_name` drive the behavior in this section.
**CN:** 该代码块定义了函数 `get_act_and_mul_fn`。 它用于获取act and mul fn。 关键调用包括 `act_fn_name.lower`、`_ACTIVATION_AND_MUL_REGISTRY` 和 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `act_fn_name` 等参数驱动。

## Key Concepts / 关键概念
- `SiluAndMul`: An activation function for SwiGLU. / 核心类，用于封装 silu and mul 相关行为。
- `GeluAndMul`: An activation function for GeGLU. / 核心类，用于封装 gelu and mul 相关行为。
- `NewGELU`: Primary class that encapsulates new gelu behavior. / 核心类，用于封装 new gelu 相关行为。
- `QuickGELU`: Primary class that encapsulates quick gelu behavior. / 核心类，用于封装 quick gelu 相关行为。
- `get_act_fn`: Get an activation function by name. / 顶层函数，用于获取act fn。
- `get_act_and_mul_fn`: Get an activation-and-mul (i.e. / 顶层函数，用于获取act and mul fn。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `math`, `typing`
- **Third-party / 第三方依赖**: `torch`, `torch.nn`, `torch.nn.functional`, `torch_npu`, `sgl_kernel`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.layers.custom_op`, `sglang.jit_kernel.activation`

- **Total lines / 总行数**: 169
