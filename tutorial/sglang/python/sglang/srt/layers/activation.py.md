# activation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/activation.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements activation operators and fused elementwise paths for the SGLang SRT runtime. It exposes symbols such as `SiluAndMul`, `GeluAndMul`, `NewGELU`, and `ReLU2` and connects them to backend-specific paths such as `CUDA`, `NPU`, `XPU`, and `CPU`. / 该模块为 SGLang 的 SRT 运行时实现了激活算子与融合逐元素路径。它提供了 `SiluAndMul`、`GeluAndMul`、`NewGELU` 以及 `ReLU2` 等符号，并把这些符号连接到 `CUDA`、`NPU`、`XPU` 以及 `CPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: File header and module overview
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
```
**EN:** This opening block contains comments, licensing text, or other context that frames the rest of the file before executable code begins.
**CN:** 这一开头部分包含注释、许可证文本或其他上下文信息，用来为后续可执行代码建立背景。

### Lines 14-79: Imports, conditional backend setup, and runtime guards
```python
"""Fused operators for activation layers."""

import logging
import math
from typing import Optional

import torch
import torch.nn as nn
import torch.nn.functional as F
from transformers import PretrainedConfig

from sglang.srt.distributed import (
    divide,
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
)
from sglang.srt.environ import envs
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.layers.utils import MultiPlatformOp
from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils import (
    cpu_has_amx_support,
    is_cpu,
    is_cuda,
    is_hip,
    is_musa,
    is_npu,
    is_xpu,
    set_weight_attrs,
)
from sglang.utils import resolve_obj_by_qualname

_is_cuda = is_cuda()
_is_musa = is_musa()
_is_npu = is_npu()
_is_cpu_amx_available = cpu_has_amx_support()
_is_cpu = is_cpu()
_is_hip = is_hip()
_is_xpu = is_xpu()

if _is_cuda:
    from sglang.jit_kernel.activation import (
        gelu_and_mul,
        gelu_tanh_and_mul,
        silu_and_mul,
    )
elif _is_xpu:
    from sgl_kernel import gelu_and_mul, gelu_tanh_and_mul, silu_and_mul
elif _is_hip:
    from sgl_kernel import gelu_and_mul, gelu_quick, gelu_tanh_and_mul, silu_and_mul
elif _is_musa:
    from sglang.srt.utils.patch_torch import register_fake_if_exists

    @register_fake_if_exists("aten::_fused_swiglu_forward")
    def _(x):
        d = x.shape[-1] // 2
        output_shape = x.shape[:-1] + (d,)
        return torch.empty(output_shape, dtype=x.dtype, device=x.device)


if is_npu():
    import torch_npu

logger = logging.getLogger(__name__)
```
**EN:** This section prepares the module namespace. It imports `logging`, `math`, `typing.Optional`, `torch`, `torch.nn`, and `torch.nn.functional`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_is_cuda`, `_is_musa`, `_is_npu`, `_is_cpu_amx_available`, and `_is_cpu` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `logging`、`math`、`typing.Optional`、`torch`、`torch.nn` 以及 `torch.nn.functional`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_is_cuda`、`_is_musa`、`_is_npu`、`_is_cpu_amx_available` 以及 `_is_cpu` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 80-80: Class `SiluAndMul` declaration and shared state
```python
class SiluAndMul(MultiPlatformOp):
```
**EN:** This block introduces class `SiluAndMul` and the state shared by its methods. It inherits from `MultiPlatformOp`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `SiluAndMul`，并定义其方法共享的状态。 它继承自 `MultiPlatformOp`，说明了它在 SRT 层栈中的接入方式。

### Lines 81-85: `SiluAndMul` initialization and state setup
```python
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        if get_global_server_args().rl_on_policy_target is not None:
            self._forward_method = self.forward_native
```
**EN:** This block defines `SiluAndMul.__init__` and contains the main logic for this step. It mainly invokes `super.__init__` and `get_global_server_args`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self._forward_method` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `SiluAndMul.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__` 和 `get_global_server_args`，说明该流程会编排底层辅助函数或计算内核。 像 `self._forward_method` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 86-89: `SiluAndMul.forward_native` NATIVE execution path
```python
    def forward_native(self, x: torch.Tensor) -> torch.Tensor:
        d = x.shape[-1] // 2
        return F.silu(x[..., :d]) * x[..., d:]
```
**EN:** This block defines `SiluAndMul.forward_native` and contains the main logic for this step. It mainly invokes `F.silu`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `d` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `SiluAndMul.forward_native`，并承载这一阶段的核心逻辑。 它主要调用 `F.silu`，说明该流程会编排底层辅助函数或计算内核。 像 `d` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 90-96: `SiluAndMul.forward_cuda` CUDA execution path
```python
    def forward_cuda(self, x: torch.Tensor) -> torch.Tensor:
        d = x.shape[-1] // 2
        output_shape = x.shape[:-1] + (d,)
        out = torch.empty(output_shape, dtype=x.dtype, device=x.device)
        silu_and_mul(x, out)
        return out
```
**EN:** This block defines `SiluAndMul.forward_cuda` and contains the main logic for this step. It mainly invokes `torch.empty` and `silu_and_mul`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `d`, `output_shape`, and `out` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `SiluAndMul.forward_cuda`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty` 和 `silu_and_mul`，说明该流程会编排底层辅助函数或计算内核。 像 `d`、`output_shape` 以及 `out` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 97-103: `SiluAndMul.forward_cpu` CPU execution path
```python
    def forward_cpu(self, x: torch.Tensor) -> torch.Tensor:
        if _is_cpu_amx_available:
            out = torch.ops.sgl_kernel.silu_and_mul_cpu(x)
            return out
        else:
            return self.forward_native(x)
```
**EN:** This block defines `SiluAndMul.forward_cpu` and contains the main logic for this step. It mainly invokes `torch.ops.sgl_kernel.silu_and_mul_cpu` and `self.forward_native`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `out` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `SiluAndMul.forward_cpu`，并承载这一阶段的核心逻辑。 它主要调用 `torch.ops.sgl_kernel.silu_and_mul_cpu` 和 `self.forward_native`，说明该流程会编排底层辅助函数或计算内核。 像 `out` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 104-107: `SiluAndMul.forward_npu` NPU execution path
```python
    def forward_npu(self, x: torch.Tensor) -> torch.Tensor:
        out = torch_npu.npu_swiglu(x)
        return out
```
**EN:** This block defines `SiluAndMul.forward_npu` and contains the main logic for this step. It mainly invokes `torch_npu.npu_swiglu`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `out` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `SiluAndMul.forward_npu`，并承载这一阶段的核心逻辑。 它主要调用 `torch_npu.npu_swiglu`，说明该流程会编排底层辅助函数或计算内核。 像 `out` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 108-114: `SiluAndMul.forward_xpu` XPU execution path
```python
    def forward_xpu(self, x: torch.Tensor) -> torch.Tensor:
        d = x.shape[-1] // 2
        output_shape = x.shape[:-1] + (d,)
        out = torch.empty(output_shape, dtype=x.dtype, device=x.device)
        silu_and_mul(x, out)
        return out
```
**EN:** This block defines `SiluAndMul.forward_xpu` and contains the main logic for this step. It mainly invokes `torch.empty` and `silu_and_mul`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `d`, `output_shape`, and `out` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `SiluAndMul.forward_xpu`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty` 和 `silu_and_mul`，说明该流程会编排底层辅助函数或计算内核。 像 `d`、`output_shape` 以及 `out` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 115-121: `SiluAndMul.forward_musa` MUSA execution path
```python
    def forward_musa(self, x: torch.Tensor) -> torch.Tensor:
        if not hasattr(self, "_musa_swish_glu"):
            # XXX (MUSA): nn.SwishGLU seems to have better performance than silu_and_mul on MUSA, we can switch to it for now. We can consider implementing a silu_and_mul kernel for MUSA in the future if needed.
            self._musa_swish_glu = nn.SwishGLU()
        return self._musa_swish_glu(x)
```
**EN:** This block defines `SiluAndMul.forward_musa` and contains the main logic for this step. It mainly invokes `self._musa_swish_glu`, `hasattr`, and `nn.SwishGLU`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self._musa_swish_glu` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `SiluAndMul.forward_musa`，并承载这一阶段的核心逻辑。 它主要调用 `self._musa_swish_glu`、`hasattr` 以及 `nn.SwishGLU`，说明该流程会编排底层辅助函数或计算内核。 像 `self._musa_swish_glu` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 122-122: Class `GeluAndMul` declaration and shared state
```python
class GeluAndMul(MultiPlatformOp):
```
**EN:** This block introduces class `GeluAndMul` and the state shared by its methods. It inherits from `MultiPlatformOp`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `GeluAndMul`，并定义其方法共享的状态。 它继承自 `MultiPlatformOp`，说明了它在 SRT 层栈中的接入方式。

### Lines 123-126: `GeluAndMul` initialization and state setup
```python
    def __init__(self, approximate="tanh"):
        super().__init__()
        self.approximate = approximate
```
**EN:** This block defines `GeluAndMul.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.approximate` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `GeluAndMul.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`，说明该流程会编排底层辅助函数或计算内核。 像 `self.approximate` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 127-138: Internal helper `GeluAndMul._forward_impl`
```python
    def _forward_impl(self, x: torch.Tensor) -> torch.Tensor:
        d = x.shape[-1] // 2
        output_shape = x.shape[:-1] + (d,)
        out = torch.empty(output_shape, dtype=x.dtype, device=x.device)
        if self.approximate == "tanh":
            gelu_tanh_and_mul(x, out)
        elif self.approximate == "none":
            gelu_and_mul(x, out)
        else:
            raise RuntimeError("GeluAndMul only support tanh or none")
        return out
```
**EN:** This block defines `GeluAndMul._forward_impl` and contains the main logic for this step. It mainly invokes `torch.empty`, `gelu_tanh_and_mul`, `gelu_and_mul`, and `RuntimeError`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `d`, `output_shape`, and `out` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `GeluAndMul._forward_impl`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty`、`gelu_tanh_and_mul`、`gelu_and_mul` 以及 `RuntimeError`，说明该流程会编排底层辅助函数或计算内核。 像 `d`、`output_shape` 以及 `out` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 139-142: `GeluAndMul.forward_native` NATIVE execution path
```python
    def forward_native(self, x: torch.Tensor) -> torch.Tensor:
        d = x.shape[-1] // 2
        return F.gelu(x[..., :d], approximate=self.approximate) * x[..., d:]
```
**EN:** This block defines `GeluAndMul.forward_native` and contains the main logic for this step. It mainly invokes `F.gelu`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `d` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `GeluAndMul.forward_native`，并承载这一阶段的核心逻辑。 它主要调用 `F.gelu`，说明该流程会编排底层辅助函数或计算内核。 像 `d` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 143-150: `GeluAndMul.forward_cpu` CPU execution path
```python
    def forward_cpu(self, x: torch.Tensor) -> torch.Tensor:
        if _is_cpu_amx_available and self.approximate == "tanh":
            return torch.ops.sgl_kernel.gelu_tanh_and_mul_cpu(x)
        elif _is_cpu_amx_available and self.approximate == "none":
            return torch.ops.sgl_kernel.gelu_and_mul_cpu(x)
        else:
            return self.forward_native(x)
```
**EN:** This block defines `GeluAndMul.forward_cpu` and contains the main logic for this step. It mainly invokes `torch.ops.sgl_kernel.gelu_tanh_and_mul_cpu`, `torch.ops.sgl_kernel.gelu_and_mul_cpu`, and `self.forward_native`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `GeluAndMul.forward_cpu`，并承载这一阶段的核心逻辑。 它主要调用 `torch.ops.sgl_kernel.gelu_tanh_and_mul_cpu`、`torch.ops.sgl_kernel.gelu_and_mul_cpu` 以及 `self.forward_native`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 151-153: `GeluAndMul.forward_cuda` CUDA execution path
```python
    def forward_cuda(self, x: torch.Tensor) -> torch.Tensor:
        return self._forward_impl(x)
```
**EN:** This block defines `GeluAndMul.forward_cuda` and contains the main logic for this step. It mainly invokes `self._forward_impl`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `GeluAndMul.forward_cuda`，并承载这一阶段的核心逻辑。 它主要调用 `self._forward_impl`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 154-156: `GeluAndMul.forward_xpu` XPU execution path
```python
    def forward_xpu(self, x: torch.Tensor) -> torch.Tensor:
        return self._forward_impl(x)
```
**EN:** This block defines `GeluAndMul.forward_xpu` and contains the main logic for this step. It mainly invokes `self._forward_impl`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `GeluAndMul.forward_xpu`，并承载这一阶段的核心逻辑。 它主要调用 `self._forward_impl`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 157-168: `GeluAndMul.forward_npu` NPU execution path
```python
    def forward_npu(self, x: torch.Tensor) -> torch.Tensor:
        if envs.SGLANG_NPU_FORWARD_NATIVE_GELUTANH.get():
            return self.forward_native(x)
        y_npu, gelu_npu = torch_npu.npu_geglu(
            x,
            dim=-1,
            approximate=1 if self.approximate == "tanh" else 0,
            activate_left=True,
        )
        return y_npu
```
**EN:** This block defines `GeluAndMul.forward_npu` and contains the main logic for this step. It mainly invokes `envs.SGLANG_NPU_FORWARD_NATIVE_GELUTANH.get`, `torch_npu.npu_geglu`, and `self.forward_native`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `y_npu` and `gelu_npu` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `GeluAndMul.forward_npu`，并承载这一阶段的核心逻辑。 它主要调用 `envs.SGLANG_NPU_FORWARD_NATIVE_GELUTANH.get`、`torch_npu.npu_geglu` 以及 `self.forward_native`，说明该流程会编排底层辅助函数或计算内核。 像 `y_npu` 和 `gelu_npu` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 169-169: Class `NewGELU` declaration and shared state
```python
class NewGELU(MultiPlatformOp):
```
**EN:** This block introduces class `NewGELU` and the state shared by its methods. It inherits from `MultiPlatformOp`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `NewGELU`，并定义其方法共享的状态。 它继承自 `MultiPlatformOp`，说明了它在 SRT 层栈中的接入方式。

### Lines 170-173: `NewGELU.forward_native` NATIVE execution path
```python
    def forward_native(self, x: torch.Tensor) -> torch.Tensor:
        c = math.sqrt(2.0 / math.pi)
        return 0.5 * x * (1.0 + torch.tanh(c * (x + 0.044715 * torch.pow(x, 3.0))))
```
**EN:** This block defines `NewGELU.forward_native` and contains the main logic for this step. It mainly invokes `math.sqrt`, `torch.tanh`, and `torch.pow`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `c` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `NewGELU.forward_native`，并承载这一阶段的核心逻辑。 它主要调用 `math.sqrt`、`torch.tanh` 以及 `torch.pow`，说明该流程会编排底层辅助函数或计算内核。 像 `c` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 174-178: `NewGELU.forward_cuda` CUDA execution path
```python
    def forward_cuda(self, x: torch.Tensor) -> torch.Tensor:
        # TODO: Implement the CUDA kernel for NewGELU in sgl-kernel
        return self.forward_native(x)
```
**EN:** This block defines `NewGELU.forward_cuda` and contains the main logic for this step. It mainly invokes `self.forward_native`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `NewGELU.forward_cuda`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward_native`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 179-184: Class `ReLU2` declaration and shared state
```python
class ReLU2(nn.Module):
    """
    Applies the squared Rectified Linear Unit function.
    y = max(0, x)^2
    """
```
**EN:** This block introduces class `ReLU2` and the state shared by its methods. It inherits from `nn.Module`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Applies the squared Rectified Linear Unit function.
**CN:** 该代码块引入类 `ReLU2`，并定义其方法共享的状态。 它继承自 `nn.Module`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 185-189: `ReLU2.forward` main forward path
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x = F.relu(x)
        return x * x
```
**EN:** This block defines `ReLU2.forward` and contains the main logic for this step. It mainly invokes `F.relu`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `x` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `ReLU2.forward`，并承载这一阶段的核心逻辑。 它主要调用 `F.relu`，说明该流程会编排底层辅助函数或计算内核。 像 `x` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 190-190: Class `QuickGELU` declaration and shared state
```python
class QuickGELU(MultiPlatformOp):
```
**EN:** This block introduces class `QuickGELU` and the state shared by its methods. It inherits from `MultiPlatformOp`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `QuickGELU`，并定义其方法共享的状态。 它继承自 `MultiPlatformOp`，说明了它在 SRT 层栈中的接入方式。

### Lines 191-193: `QuickGELU.forward_native` NATIVE execution path
```python
    def forward_native(self, x: torch.Tensor) -> torch.Tensor:
        return x * torch.sigmoid(1.702 * x)
```
**EN:** This block defines `QuickGELU.forward_native` and contains the main logic for this step. It mainly invokes `torch.sigmoid`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `QuickGELU.forward_native`，并承载这一阶段的核心逻辑。 它主要调用 `torch.sigmoid`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 194-196: `QuickGELU.forward_cuda` CUDA execution path
```python
    def forward_cuda(self, x: torch.Tensor) -> torch.Tensor:
        return self.forward_native(x)
```
**EN:** This block defines `QuickGELU.forward_cuda` and contains the main logic for this step. It mainly invokes `self.forward_native`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `QuickGELU.forward_cuda`，并承载这一阶段的核心逻辑。 它主要调用 `self.forward_native`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 197-201: `QuickGELU.forward_hip` HIP execution path
```python
    def forward_hip(self, x: torch.Tensor) -> torch.Tensor:
        out = torch.empty(x.shape, dtype=x.dtype, device=x.device)
        gelu_quick(x, out)
        return out
```
**EN:** This block defines `QuickGELU.forward_hip` and contains the main logic for this step. It mainly invokes `torch.empty` and `gelu_quick`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `out` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `QuickGELU.forward_hip`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty` 和 `gelu_quick`，说明该流程会编排底层辅助函数或计算内核。 像 `out` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 202-205: `QuickGELU.forward_npu` NPU execution path
```python
    def forward_npu(self, x: torch.Tensor) -> torch.Tensor:
        return torch_npu.npu_fast_gelu(x)
```
**EN:** This block defines `QuickGELU.forward_npu` and contains the main logic for this step. It mainly invokes `torch_npu.npu_fast_gelu`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `QuickGELU.forward_npu`，并承载这一阶段的核心逻辑。 它主要调用 `torch_npu.npu_fast_gelu`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 206-212: Class `XIELU` declaration and shared state
```python
class XIELU(MultiPlatformOp):
    """
    Applies the xIELU activation function introduced in https://arxiv.org/abs/2411.13010
    If the user has installed the nickjbrowning/XIELU, we import xIELU CUDA
    Otherwise, we emit a single warning and use xIELU Python
    """
```
**EN:** This block introduces class `XIELU` and the state shared by its methods. It inherits from `MultiPlatformOp`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Applies the xIELU activation function introduced in https://arxiv.org/abs/2411.13010 If the user has installed the nickjbrowning/XIELU, we import xIELU CUDA Otherwise, we emit a single warning and use xIELU Python
**CN:** 该代码块引入类 `XIELU`，并定义其方法共享的状态。 它继承自 `MultiPlatformOp`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 213-266: `XIELU` initialization and state setup
```python
    def __init__(
        self,
        alpha_p_init: float = 0.8,
        alpha_n_init: float = 0.8,
        beta: float = 0.5,
        eps: float = -1e-6,
        dtype: torch.dtype = torch.bfloat16,
        with_vector_loads: bool = False,
    ):
        super().__init__()
        self.alpha_p = nn.Parameter(
            torch.log(torch.exp(torch.tensor(alpha_p_init, dtype=dtype)) - 1).unsqueeze(
                0
            )
        )
        self.alpha_n = nn.Parameter(
            torch.log(
                torch.exp(torch.tensor(alpha_n_init - beta, dtype=dtype)) - 1
            ).unsqueeze(0)
        )
        self.register_buffer("beta", torch.tensor(beta, dtype=dtype))
        self.register_buffer("eps", torch.tensor(eps, dtype=dtype))
        self.with_vector_loads = with_vector_loads
        # Temporary until xIELU CUDA fully implemented
        self._beta_scalar = float(self.beta.detach().cpu().float().item())
        self._eps_scalar = float(self.eps.detach().cpu().float().item())

        self._xielu_cuda_obj = None
        try:
            import xielu.ops  # noqa: F401

            self._xielu_cuda_obj = torch.classes.xielu.XIELU()
            msg = "Using experimental xIELU CUDA."
            try:
                from torch._dynamo import allow_in_graph

                self._xielu_cuda_fn = allow_in_graph(self._xielu_cuda)
                msg += " Enabled torch._dynamo for xIELU CUDA."
            except Exception as err:
                msg += (
                    f" Could not enable torch._dynamo for xIELU ({err}) - "
                    "this may result in slower performance."
                )
                self._xielu_cuda_fn = self._xielu_cuda
            logger.warning_once(msg)
        except Exception as err:
            pass
            # logger.warning_once(
            #     "CUDA-fused xIELU not available (%s) –"
            #     " falling back to a Python version.\n"
            #     "For CUDA xIELU (experimental), `pip install git+https://github.com/nickjbrowning/XIELU`",
            #     str(err),
            # )
```
**EN:** This block defines `XIELU.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `nn.Parameter`, `self.register_buffer`, `float`, and `torch.log.unsqueeze`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.alpha_p`, `self.alpha_n`, `self.with_vector_loads`, `self._beta_scalar`, and `self._eps_scalar` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `XIELU.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`nn.Parameter`、`self.register_buffer`、`float` 以及 `torch.log.unsqueeze`，说明该流程会编排底层辅助函数或计算内核。 像 `self.alpha_p`、`self.alpha_n`、`self.with_vector_loads`、`self._beta_scalar` 以及 `self._eps_scalar` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 267-275: Internal helper `XIELU._xielu_python`
```python
    def _xielu_python(self, x: torch.Tensor) -> torch.Tensor:
        alpha_p = nn.functional.softplus(self.alpha_p)
        alpha_n = self.beta + nn.functional.softplus(self.alpha_n)
        return torch.where(
            x > 0,
            alpha_p * x * x + self.beta * x,
            (torch.expm1(torch.min(x, self.eps)) - x) * alpha_n + self.beta * x,
        )
```
**EN:** This block defines `XIELU._xielu_python` and contains the main logic for this step. It mainly invokes `nn.functional.softplus`, `torch.where`, `torch.expm1`, and `torch.min`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `alpha_p` and `alpha_n` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `XIELU._xielu_python`，并承载这一阶段的核心逻辑。 它主要调用 `nn.functional.softplus`、`torch.where`、`torch.expm1` 以及 `torch.min`，说明该流程会编排底层辅助函数或计算内核。 像 `alpha_p` 和 `alpha_n` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 276-304: Internal helper `XIELU._xielu_cuda`
```python
    def _xielu_cuda(self, x: torch.Tensor) -> torch.Tensor:
        """Firewall function to prevent torch.compile from seeing .item()"""
        assert self._xielu_cuda_obj is not None, "XIELU CUDA object must not be None"
        original_shape = x.shape
        # CUDA kernel expects 3D tensors, reshape if needed
        while x.dim() < 3:
            x = x.unsqueeze(0)
        if x.dim() > 3:
            x = x.view(-1, 1, x.size(-1))
        if original_shape != x.shape:
            logger.warning_once(
                "Warning: xIELU input tensor expects 3 dimensions"
                " but got (shape: %s). Reshaping to (shape: %s).\n"
                "Note: For SGLang this may be expected if sending"
                "[B*S,D] instead of [B,S,D].",
                original_shape,
                x.shape,
            )
        result = self._xielu_cuda_obj.forward(
            x,
            self.alpha_p,
            self.alpha_n,
            # Temporary until xIELU CUDA fully implemented -> self.{beta,eps}.item()
            self._beta_scalar,
            self._eps_scalar,
            self.with_vector_loads,
        )
        return result.view(original_shape)
```
**EN:** This block defines `XIELU._xielu_cuda` and contains the main logic for this step. It mainly invokes `self._xielu_cuda_obj.forward`, `result.view`, `x.dim`, `x.unsqueeze`, and `x.view`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `original_shape`, `result`, and `x` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `XIELU._xielu_cuda`，并承载这一阶段的核心逻辑。 它主要调用 `self._xielu_cuda_obj.forward`、`result.view`、`x.dim`、`x.unsqueeze` 以及 `x.view`，说明该流程会编排底层辅助函数或计算内核。 像 `original_shape`、`result` 以及 `x` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 305-315: `XIELU.forward` main forward path
```python
    def forward(self, input: torch.Tensor) -> torch.Tensor:
        if self._xielu_cuda_obj is not None and input.is_cuda:
            if not torch._dynamo.is_compiling():
                return self._xielu_cuda_fn(input)
            else:
                logger.warning_once(
                    "torch._dynamo is compiling, using Python version of xIELU."
                )
        return self._xielu_python(input)
```
**EN:** This block defines `XIELU.forward` and contains the main logic for this step. It mainly invokes `self._xielu_python`, `torch._dynamo.is_compiling`, `self._xielu_cuda_fn`, and `logger.warning_once`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `XIELU.forward`，并承载这一阶段的核心逻辑。 它主要调用 `self._xielu_python`、`torch._dynamo.is_compiling`、`self._xielu_cuda_fn` 以及 `logger.warning_once`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 316-321: Class `ScaledActivation` declaration and shared state
```python
class ScaledActivation(nn.Module):
    """An activation function with post-scale parameters.

    This is used for some quantization methods like AWQ.
    """
```
**EN:** This block introduces class `ScaledActivation` and the state shared by its methods. It inherits from `nn.Module`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: An activation function with post-scale parameters.
**CN:** 该代码块引入类 `ScaledActivation`，并定义其方法共享的状态。 它继承自 `nn.Module`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 322-343: `ScaledActivation` initialization and state setup
```python
    def __init__(
        self,
        act_module: nn.Module,
        intermediate_size: int,
        input_is_parallel: bool = True,
        params_dtype: Optional[torch.dtype] = None,
    ):
        super().__init__()
        self.act = act_module
        self.input_is_parallel = input_is_parallel
        if input_is_parallel:
            tp_size = get_tensor_model_parallel_world_size()
            intermediate_size_per_partition = divide(intermediate_size, tp_size)
        else:
            intermediate_size_per_partition = intermediate_size
        if params_dtype is None:
            params_dtype = torch.get_default_dtype()
        self.scales = nn.Parameter(
            torch.empty(intermediate_size_per_partition, dtype=params_dtype)
        )
        set_weight_attrs(self.scales, {"weight_loader": self.weight_loader})
```
**EN:** This block defines `ScaledActivation.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `nn.Parameter`, `set_weight_attrs`, `get_tensor_model_parallel_world_size`, and `divide`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.act`, `self.input_is_parallel`, `self.scales`, `tp_size`, and `intermediate_size_per_partition` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `ScaledActivation.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`nn.Parameter`、`set_weight_attrs`、`get_tensor_model_parallel_world_size` 以及 `divide`，说明该流程会编排底层辅助函数或计算内核。 像 `self.act`、`self.input_is_parallel`、`self.scales`、`tp_size` 以及 `intermediate_size_per_partition` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 344-346: `ScaledActivation.forward` main forward path
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return self.act(x) / self.scales
```
**EN:** This block defines `ScaledActivation.forward` and contains the main logic for this step. It mainly invokes `self.act`, showing how the routine orchestrates lower-level helpers or kernels. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `ScaledActivation.forward`，并承载这一阶段的核心逻辑。 它主要调用 `self.act`，说明该流程会编排底层辅助函数或计算内核。 该函数最终会把计算结果返回给调用方。

### Lines 347-357: Function `ScaledActivation.weight_loader` and its core logic
```python
    def weight_loader(self, param: nn.Parameter, loaded_weight: torch.Tensor):
        param_data = param.data
        if self.input_is_parallel:
            tp_rank = get_tensor_model_parallel_rank()
            shard_size = param_data.shape[0]
            start_idx = tp_rank * shard_size
            loaded_weight = loaded_weight.narrow(0, start_idx, shard_size)
        assert param_data.shape == loaded_weight.shape
        param_data.copy_(loaded_weight)
```
**EN:** This block defines `ScaledActivation.weight_loader` and contains the main logic for this step. It mainly invokes `param_data.copy_`, `get_tensor_model_parallel_rank`, and `loaded_weight.narrow`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `param_data`, `tp_rank`, `shard_size`, `start_idx`, and `loaded_weight` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `ScaledActivation.weight_loader`，并承载这一阶段的核心逻辑。 它主要调用 `param_data.copy_`、`get_tensor_model_parallel_rank` 以及 `loaded_weight.narrow`，说明该流程会编排底层辅助函数或计算内核。 像 `param_data`、`tp_rank`、`shard_size`、`start_idx` 以及 `loaded_weight` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 358-366: Module constants and shared configuration
```python
_ACTIVATION_REGISTRY = {
    "gelu": nn.GELU(),
    "gelu_pytorch_tanh": nn.GELU(approximate="tanh"),
    "gelu_new": NewGELU(),
    "relu2": ReLU2(),
    "xielu": XIELU(),
}
```
**EN:** This section prepares the module namespace. Shared names such as `_ACTIVATION_REGISTRY` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 像 `_ACTIVATION_REGISTRY` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 367-391: `get_act_fn` getter for act fn
```python
def get_act_fn(
    act_fn_name: str,
    quant_config: Optional[QuantizationConfig] = None,
    intermediate_size: Optional[int] = None,
    input_is_parallel: bool = True,
    params_dtype: Optional[torch.dtype] = None,
) -> nn.Module:
    """Get an activation function by name."""
    act_fn_name = act_fn_name.lower()
    if act_fn_name not in _ACTIVATION_REGISTRY:
        raise ValueError(f"Activation function {act_fn_name!r} is not supported.")

    act_fn = _ACTIVATION_REGISTRY[act_fn_name]
    if quant_config is not None and act_fn_name in quant_config.get_scaled_act_names():
        if intermediate_size is None:
            raise ValueError(
                "intermediate_size must be specified for scaled "
                "activation functions."
            )
        return ScaledActivation(
            act_fn, intermediate_size, input_is_parallel, params_dtype
        )
    return act_fn
```
**EN:** This block defines `get_act_fn` and contains the main logic for this step. It mainly invokes `act_fn_name.lower`, `ValueError`, `ScaledActivation`, and `quant_config.get_scaled_act_names`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `act_fn_name` and `act_fn` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_act_fn`，并承载这一阶段的核心逻辑。 它主要调用 `act_fn_name.lower`、`ValueError`、`ScaledActivation` 以及 `quant_config.get_scaled_act_names`，说明该流程会编排底层辅助函数或计算内核。 像 `act_fn_name` 和 `act_fn` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 392-406: `get_cross_encoder_activation_function` getter for cross encoder activation function
```python
def get_cross_encoder_activation_function(config: PretrainedConfig):
    if (
        hasattr(config, "sbert_ce_default_activation_function")
        and config.sbert_ce_default_activation_function is not None
    ):

        function_name = config.sbert_ce_default_activation_function
        assert function_name.startswith("torch.nn.modules."), (
            "Loading of activation functions is restricted to "
            "torch.nn.modules for security reasons"
        )
        return resolve_obj_by_qualname(function_name)()
    else:
        # adapt bge-reranker
        return nn.Identity()
```
**EN:** This block defines `get_cross_encoder_activation_function` and contains the main logic for this step. It mainly invokes `hasattr`, `function_name.startswith`, `resolve_obj_by_qualname`, and `nn.Identity`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `function_name` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_cross_encoder_activation_function`，并承载这一阶段的核心逻辑。 它主要调用 `hasattr`、`function_name.startswith`、`resolve_obj_by_qualname` 以及 `nn.Identity`，说明该流程会编排底层辅助函数或计算内核。 像 `function_name` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `SiluAndMul`, `GeluAndMul`, `NewGELU`, `ReLU2`, and `QuickGELU`. / **主要符号**：核心入口包括 `SiluAndMul`、`GeluAndMul`、`NewGELU`、`ReLU2` 以及 `QuickGELU`。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Activation math**: Captures fused nonlinear transforms and their device-specific fast paths. / **激活数学**：描述融合非线性变换及其设备专用快速路径。

## Dependencies / 依赖关系
- **Standard library**: `logging`, `math`, and `typing.Optional` / **标准库**：`logging`、`math` 以及 `typing.Optional`
- **Third-party**: `torch`, `torch.nn`, `torch.nn.functional`, `transformers.PretrainedConfig`, `torch_npu`, `sgl_kernel.gelu_and_mul`, `sgl_kernel.gelu_tanh_and_mul`, `sgl_kernel.silu_and_mul`, `sgl_kernel.gelu_quick`, and `xielu.ops` / **第三方依赖**：`torch`、`torch.nn`、`torch.nn.functional`、`transformers.PretrainedConfig`、`torch_npu`、`sgl_kernel.gelu_and_mul`、`sgl_kernel.gelu_tanh_and_mul`、`sgl_kernel.silu_and_mul`、`sgl_kernel.gelu_quick` 以及 `xielu.ops`
- **Internal SGLang modules**: `sglang.srt.distributed.divide`, `sglang.srt.distributed.get_tensor_model_parallel_rank`, `sglang.srt.distributed.get_tensor_model_parallel_world_size`, `sglang.srt.environ.envs`, `sglang.srt.layers.quantization.base_config.QuantizationConfig`, `sglang.srt.layers.utils.MultiPlatformOp`, `sglang.srt.server_args.get_global_server_args`, `sglang.srt.utils.cpu_has_amx_support`, `sglang.srt.utils.is_cpu`, `sglang.srt.utils.is_cuda`, `sglang.srt.utils.is_hip`, and `sglang.srt.utils.is_musa` / **SGLang 内部模块**：`sglang.srt.distributed.divide`、`sglang.srt.distributed.get_tensor_model_parallel_rank`、`sglang.srt.distributed.get_tensor_model_parallel_world_size`、`sglang.srt.environ.envs`、`sglang.srt.layers.quantization.base_config.QuantizationConfig`、`sglang.srt.layers.utils.MultiPlatformOp`、`sglang.srt.server_args.get_global_server_args`、`sglang.srt.utils.cpu_has_amx_support`、`sglang.srt.utils.is_cpu`、`sglang.srt.utils.is_cuda`、`sglang.srt.utils.is_hip` 以及 `sglang.srt.utils.is_musa`
