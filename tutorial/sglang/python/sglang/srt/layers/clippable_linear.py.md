# clippable_linear.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/clippable_linear.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements linear projections and GEMM backend integration for the SGLang SRT runtime. It exposes symbols such as `ClippableRowParallelLinear`, `ClippableColumnParallelLinear`, `ClippableQKVParallelLinear`, and `ClippableGLUParallelLinear` and connects them to backend-specific paths such as `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了线性投影与 GEMM 后端集成。它提供了 `ClippableRowParallelLinear`、`ClippableColumnParallelLinear`、`ClippableQKVParallelLinear` 以及 `ClippableGLUParallelLinear` 等符号，并把这些符号连接到 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: File header and module overview
```python
# Copyright 2025 SGLang Team
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

### Lines 14-42: Imports, constants, and runtime setup
```python
"""TP-sharded linear wrappers with per-tensor activation clamping.

Used by the Gemma 4 vision and audio encoders.  Each wrapper owns a parallel
linear and four scalar clip buffers (``input_min/max``, ``output_min/max``)
that default to ±inf (no-op) and are populated from the checkpoint.

For fused projections (QKV, GateUp), input bounds are shared (the checkpoint
stores identical copies per projection — last write wins during loading) and
output bounds are per-projection.
"""

from typing import Optional, Tuple

import torch
import torch.nn as nn

from sglang.srt.layers.dp_attention import get_attention_tp_size
from sglang.srt.layers.linear import (
    ColumnParallelLinear,
    MergedColumnParallelLinear,
    QKVParallelLinear,
    RowParallelLinear,
)
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.utils import add_prefix

_INF = float("inf")
```
**EN:** This section prepares the module namespace. It imports `typing.Optional`, `typing.Tuple`, `torch`, `torch.nn`, `sglang.srt.layers.dp_attention.get_attention_tp_size`, and `sglang.srt.layers.linear.ColumnParallelLinear`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `_INF` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `typing.Optional`、`typing.Tuple`、`torch`、`torch.nn`、`sglang.srt.layers.dp_attention.get_attention_tp_size` 以及 `sglang.srt.layers.linear.ColumnParallelLinear`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `_INF` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 43-49: Class `ClippableRowParallelLinear` declaration and shared state
```python
class ClippableRowParallelLinear(nn.Module):
    """``RowParallelLinear`` with input/output activation clamping.

    Checkpoint weight at ``<name>.weight`` is remapped to ``<name>.linear.weight``
    by the model's ``load_weights``.
    """
```
**EN:** This block introduces class `ClippableRowParallelLinear` and the state shared by its methods. It inherits from `nn.Module`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: ``RowParallelLinear`` with input/output activation clamping.
**CN:** 该代码块引入类 `ClippableRowParallelLinear`，并定义其方法共享的状态。 它继承自 `nn.Module`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 50-71: `ClippableRowParallelLinear` initialization and state setup
```python
    def __init__(
        self,
        input_size: int,
        output_size: int,
        *,
        bias: bool = True,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.linear = RowParallelLinear(
            input_size=input_size,
            output_size=output_size,
            bias=bias,
            quant_config=quant_config,
            prefix=add_prefix("linear", prefix),
        )
        self.input_min = nn.parameter.Buffer(torch.tensor(-_INF), persistent=False)
        self.input_max = nn.parameter.Buffer(torch.tensor(_INF), persistent=False)
        self.output_min = nn.parameter.Buffer(torch.tensor(-_INF), persistent=False)
        self.output_max = nn.parameter.Buffer(torch.tensor(_INF), persistent=False)
```
**EN:** This block defines `ClippableRowParallelLinear.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `RowParallelLinear`, `nn.parameter.Buffer`, `torch.tensor`, and `add_prefix`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.linear`, `self.input_min`, `self.input_max`, `self.output_min`, and `self.output_max` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `ClippableRowParallelLinear.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`RowParallelLinear`、`nn.parameter.Buffer`、`torch.tensor` 以及 `add_prefix`，说明该流程会编排底层辅助函数或计算内核。 像 `self.linear`、`self.input_min`、`self.input_max`、`self.output_min` 以及 `self.output_max` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 72-78: `ClippableRowParallelLinear.forward` main forward path
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x = torch.clamp(x, self.input_min, self.input_max)
        x, _ = self.linear(x)
        x = torch.clamp(x, self.output_min, self.output_max)
        return x
```
**EN:** This block defines `ClippableRowParallelLinear.forward` and contains the main logic for this step. It mainly invokes `torch.clamp` and `self.linear`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `x` and `_` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `ClippableRowParallelLinear.forward`，并承载这一阶段的核心逻辑。 它主要调用 `torch.clamp` 和 `self.linear`，说明该流程会编排底层辅助函数或计算内核。 像 `x` 和 `_` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 79-81: Class `ClippableColumnParallelLinear` declaration and shared state
```python
class ClippableColumnParallelLinear(nn.Module):
    """``ColumnParallelLinear`` with input/output activation clamping."""
```
**EN:** This block introduces class `ClippableColumnParallelLinear` and the state shared by its methods. It inherits from `nn.Module`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: ``ColumnParallelLinear`` with input/output activation clamping.
**CN:** 该代码块引入类 `ClippableColumnParallelLinear`，并定义其方法共享的状态。 它继承自 `nn.Module`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 82-103: `ClippableColumnParallelLinear` initialization and state setup
```python
    def __init__(
        self,
        input_size: int,
        output_size: int,
        *,
        bias: bool = False,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.linear = ColumnParallelLinear(
            input_size=input_size,
            output_size=output_size,
            bias=bias,
            quant_config=quant_config,
            prefix=add_prefix("linear", prefix),
        )
        self.input_min = nn.parameter.Buffer(torch.tensor(-_INF), persistent=False)
        self.input_max = nn.parameter.Buffer(torch.tensor(_INF), persistent=False)
        self.output_min = nn.parameter.Buffer(torch.tensor(-_INF), persistent=False)
        self.output_max = nn.parameter.Buffer(torch.tensor(_INF), persistent=False)
```
**EN:** This block defines `ClippableColumnParallelLinear.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `ColumnParallelLinear`, `nn.parameter.Buffer`, `torch.tensor`, and `add_prefix`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.linear`, `self.input_min`, `self.input_max`, `self.output_min`, and `self.output_max` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `ClippableColumnParallelLinear.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`ColumnParallelLinear`、`nn.parameter.Buffer`、`torch.tensor` 以及 `add_prefix`，说明该流程会编排底层辅助函数或计算内核。 像 `self.linear`、`self.input_min`、`self.input_max`、`self.output_min` 以及 `self.output_max` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 104-110: `ClippableColumnParallelLinear.forward` main forward path
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x = torch.clamp(x, self.input_min, self.input_max)
        x, _ = self.linear(x)
        x = torch.clamp(x, self.output_min, self.output_max)
        return x
```
**EN:** This block defines `ClippableColumnParallelLinear.forward` and contains the main logic for this step. It mainly invokes `torch.clamp` and `self.linear`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `x` and `_` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `ClippableColumnParallelLinear.forward`，并承载这一阶段的核心逻辑。 它主要调用 `torch.clamp` 和 `self.linear`，说明该流程会编排底层辅助函数或计算内核。 像 `x` 和 `_` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 111-118: Class `ClippableQKVParallelLinear` declaration and shared state
```python
class ClippableQKVParallelLinear(nn.Module):
    """Fused QKV projection with per-projection activation clamping.

    Owns a single ``QKVParallelLinear`` for the fused matmul.  Clip bounds
    are stored as flat buffers: shared ``input_min/max`` (applied before the
    matmul) and per-projection ``q/k/v_output_min/max`` (applied after split).
    """
```
**EN:** This block introduces class `ClippableQKVParallelLinear` and the state shared by its methods. It inherits from `nn.Module`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Fused QKV projection with per-projection activation clamping.
**CN:** 该代码块引入类 `ClippableQKVParallelLinear`，并定义其方法共享的状态。 它继承自 `nn.Module`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 119-152: `ClippableQKVParallelLinear` initialization and state setup
```python
    def __init__(
        self,
        hidden_size: int,
        head_size: int,
        total_num_heads: int,
        total_num_kv_heads: int,
        *,
        bias: bool = False,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        tp_size = get_attention_tp_size()
        self.q_size = (total_num_heads // tp_size) * head_size
        self.kv_size = (total_num_kv_heads // tp_size) * head_size

        self.qkv_proj = QKVParallelLinear(
            hidden_size=hidden_size,
            head_size=head_size,
            total_num_heads=total_num_heads,
            total_num_kv_heads=total_num_kv_heads,
            bias=bias,
            quant_config=quant_config,
            prefix=add_prefix("qkv_proj", prefix),
        )
        self.input_min = nn.parameter.Buffer(torch.tensor(-_INF), persistent=False)
        self.input_max = nn.parameter.Buffer(torch.tensor(_INF), persistent=False)
        self.q_output_min = nn.parameter.Buffer(torch.tensor(-_INF), persistent=False)
        self.q_output_max = nn.parameter.Buffer(torch.tensor(_INF), persistent=False)
        self.k_output_min = nn.parameter.Buffer(torch.tensor(-_INF), persistent=False)
        self.k_output_max = nn.parameter.Buffer(torch.tensor(_INF), persistent=False)
        self.v_output_min = nn.parameter.Buffer(torch.tensor(-_INF), persistent=False)
        self.v_output_max = nn.parameter.Buffer(torch.tensor(_INF), persistent=False)
```
**EN:** This block defines `ClippableQKVParallelLinear.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `get_attention_tp_size`, `QKVParallelLinear`, `nn.parameter.Buffer`, and `torch.tensor`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `tp_size`, `self.q_size`, `self.kv_size`, `self.qkv_proj`, and `self.input_min` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `ClippableQKVParallelLinear.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`get_attention_tp_size`、`QKVParallelLinear`、`nn.parameter.Buffer` 以及 `torch.tensor`，说明该流程会编排底层辅助函数或计算内核。 像 `tp_size`、`self.q_size`、`self.kv_size`、`self.qkv_proj` 以及 `self.input_min` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 153-164: `ClippableQKVParallelLinear.forward` main forward path
```python
    def forward(
        self, hidden_states: torch.Tensor
    ) -> Tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
        x = torch.clamp(hidden_states, self.input_min, self.input_max)
        qkv, _ = self.qkv_proj(x)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        q = torch.clamp(q, self.q_output_min, self.q_output_max)
        k = torch.clamp(k, self.k_output_min, self.k_output_max)
        v = torch.clamp(v, self.v_output_min, self.v_output_max)
        return q, k, v
```
**EN:** This block defines `ClippableQKVParallelLinear.forward` and contains the main logic for this step. It mainly invokes `torch.clamp`, `self.qkv_proj`, and `qkv.split`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `x`, `qkv`, `_`, `q`, and `k` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `ClippableQKVParallelLinear.forward`，并承载这一阶段的核心逻辑。 它主要调用 `torch.clamp`、`self.qkv_proj` 以及 `qkv.split`，说明该流程会编排底层辅助函数或计算内核。 像 `x`、`qkv`、`_`、`q` 以及 `k` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 165-184: Class `ClippableGLUParallelLinear` declaration and shared state
```python
class ClippableGLUParallelLinear(nn.Module):
    """Fused linear + GLU gating with correct TP sharding.

    Used by the audio encoder's ``LightConv1d``, where a single linear
    projects to ``[hidden * 2]`` and GLU splits into value/gate halves.
    A plain ``ColumnParallelLinear`` is *incorrect* here under TP because it
    shards the output contiguously, mixing value and gate across ranks.
    This wrapper uses ``MergedColumnParallelLinear`` to shard each half
    independently, then applies GLU (``value * sigmoid(gate)``) on each
    rank's correctly-paired shard.

    Output clamping is applied once *after* the GLU gate, using a single
    ``output_min/max`` pair (matching the checkpoint layout).

    The checkpoint stores a single fused ``[hidden * 2, input]`` weight.
    A custom ``weight_loader`` on the inner param automatically splits it
    into value (first half) and gate (second half) shards, so no special
    handling is needed in the model's ``load_weights``.
    """
```
**EN:** This block introduces class `ClippableGLUParallelLinear` and the state shared by its methods. It inherits from `nn.Module`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Fused linear + GLU gating with correct TP sharding.
**CN:** 该代码块引入类 `ClippableGLUParallelLinear`，并定义其方法共享的状态。 它继承自 `nn.Module`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 185-225: `ClippableGLUParallelLinear` initialization and state setup
```python
    def __init__(
        self,
        input_size: int,
        hidden_size: int,
        *,
        bias: bool = False,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        tp_size = get_attention_tp_size()
        self.proj_size = hidden_size // tp_size

        self.linear = MergedColumnParallelLinear(
            input_size=input_size,
            output_sizes=[hidden_size, hidden_size],
            bias=bias,
            quant_config=quant_config,
            prefix=add_prefix("linear", prefix),
        )

        # The checkpoint has a single fused weight; MergedColumnParallelLinear
        # expects per-shard loading.  Wrap the original weight_loader so that
        # a call *without* shard_id (the generic load_weights path) splits
        # automatically.
        orig_loader = self.linear.weight.weight_loader

        def _fused_weight_loader(param, loaded_weight, loaded_shard_id=None):
            if loaded_shard_id is not None:
                return orig_loader(param, loaded_weight, loaded_shard_id)
            half = loaded_weight.shape[0] // 2
            orig_loader(param, loaded_weight[:half], 0)
            orig_loader(param, loaded_weight[half:], 1)

        self.linear.weight.weight_loader = _fused_weight_loader

        self.input_min = nn.parameter.Buffer(torch.tensor(-_INF), persistent=False)
        self.input_max = nn.parameter.Buffer(torch.tensor(_INF), persistent=False)
        self.output_min = nn.parameter.Buffer(torch.tensor(-_INF), persistent=False)
        self.output_max = nn.parameter.Buffer(torch.tensor(_INF), persistent=False)
```
**EN:** This block defines `ClippableGLUParallelLinear.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `get_attention_tp_size`, `MergedColumnParallelLinear`, `nn.parameter.Buffer`, and `orig_loader`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `tp_size`, `self.proj_size`, `self.linear`, `orig_loader`, and `self.input_min` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `ClippableGLUParallelLinear.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`get_attention_tp_size`、`MergedColumnParallelLinear`、`nn.parameter.Buffer` 以及 `orig_loader`，说明该流程会编排底层辅助函数或计算内核。 像 `tp_size`、`self.proj_size`、`self.linear`、`orig_loader` 以及 `self.input_min` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 226-234: `ClippableGLUParallelLinear.forward` main forward path
```python
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x = torch.clamp(x, self.input_min, self.input_max)
        merged, _ = self.linear(x)
        value, gate = merged.split([self.proj_size, self.proj_size], dim=-1)
        x = value * torch.sigmoid(gate)
        x = torch.clamp(x, self.output_min, self.output_max)
        return x
```
**EN:** This block defines `ClippableGLUParallelLinear.forward` and contains the main logic for this step. It mainly invokes `torch.clamp`, `self.linear`, `merged.split`, and `torch.sigmoid`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `x`, `merged`, `_`, `value`, and `gate` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `ClippableGLUParallelLinear.forward`，并承载这一阶段的核心逻辑。 它主要调用 `torch.clamp`、`self.linear`、`merged.split` 以及 `torch.sigmoid`，说明该流程会编排底层辅助函数或计算内核。 像 `x`、`merged`、`_`、`value` 以及 `gate` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 235-247: Class `ClippableGateUpParallelLinear` declaration and shared state
```python
class ClippableGateUpParallelLinear(nn.Module):
    """Fused gate/up projection with per-projection activation clamping.

    Used by the MLP layers in the vision/audio encoders.  Owns a single
    ``MergedColumnParallelLinear`` for the fused matmul and returns the
    two projections separately so the caller can apply its own activation
    (e.g. ``SiLU(gate) * up``).

    Output clamping is applied *per-projection before* the caller's
    activation, using separate ``gate_output_min/max`` and
    ``up_output_min/max`` bounds.
    """
```
**EN:** This block introduces class `ClippableGateUpParallelLinear` and the state shared by its methods. It inherits from `nn.Module`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Fused gate/up projection with per-projection activation clamping.
**CN:** 该代码块引入类 `ClippableGateUpParallelLinear`，并定义其方法共享的状态。 它继承自 `nn.Module`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 248-276: `ClippableGateUpParallelLinear` initialization and state setup
```python
    def __init__(
        self,
        input_size: int,
        intermediate_size: int,
        *,
        bias: bool = False,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        tp_size = get_attention_tp_size()
        self.proj_size = intermediate_size // tp_size

        self.gate_up_proj = MergedColumnParallelLinear(
            input_size=input_size,
            output_sizes=[intermediate_size, intermediate_size],
            bias=bias,
            quant_config=quant_config,
            prefix=add_prefix("gate_up_proj", prefix),
        )
        self.input_min = nn.parameter.Buffer(torch.tensor(-_INF), persistent=False)
        self.input_max = nn.parameter.Buffer(torch.tensor(_INF), persistent=False)
        self.gate_output_min = nn.parameter.Buffer(
            torch.tensor(-_INF), persistent=False
        )
        self.gate_output_max = nn.parameter.Buffer(torch.tensor(_INF), persistent=False)
        self.up_output_min = nn.parameter.Buffer(torch.tensor(-_INF), persistent=False)
        self.up_output_max = nn.parameter.Buffer(torch.tensor(_INF), persistent=False)
```
**EN:** This block defines `ClippableGateUpParallelLinear.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `get_attention_tp_size`, `MergedColumnParallelLinear`, `nn.parameter.Buffer`, and `torch.tensor`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `tp_size`, `self.proj_size`, `self.gate_up_proj`, `self.input_min`, and `self.input_max` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `ClippableGateUpParallelLinear.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`get_attention_tp_size`、`MergedColumnParallelLinear`、`nn.parameter.Buffer` 以及 `torch.tensor`，说明该流程会编排底层辅助函数或计算内核。 像 `tp_size`、`self.proj_size`、`self.gate_up_proj`、`self.input_min` 以及 `self.input_max` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 277-283: `ClippableGateUpParallelLinear.forward` main forward path
```python
    def forward(self, x: torch.Tensor) -> Tuple[torch.Tensor, torch.Tensor]:
        x = torch.clamp(x, self.input_min, self.input_max)
        gate_up, _ = self.gate_up_proj(x)
        gate, up = gate_up.split([self.proj_size, self.proj_size], dim=-1)
        gate = torch.clamp(gate, self.gate_output_min, self.gate_output_max)
        up = torch.clamp(up, self.up_output_min, self.up_output_max)
        return gate, up
```
**EN:** This block defines `ClippableGateUpParallelLinear.forward` and contains the main logic for this step. It mainly invokes `torch.clamp`, `self.gate_up_proj`, and `gate_up.split`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `x`, `gate_up`, `_`, `gate`, and `up` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `ClippableGateUpParallelLinear.forward`，并承载这一阶段的核心逻辑。 它主要调用 `torch.clamp`、`self.gate_up_proj` 以及 `gate_up.split`，说明该流程会编排底层辅助函数或计算内核。 像 `x`、`gate_up`、`_`、`gate` 以及 `up` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `ClippableRowParallelLinear`, `ClippableColumnParallelLinear`, `ClippableQKVParallelLinear`, `ClippableGLUParallelLinear`, and `ClippableGateUpParallelLinear`. / **主要符号**：核心入口包括 `ClippableRowParallelLinear`、`ClippableColumnParallelLinear`、`ClippableQKVParallelLinear`、`ClippableGLUParallelLinear` 以及 `ClippableGateUpParallelLinear`。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Activation math**: Captures fused nonlinear transforms and their device-specific fast paths. / **激活数学**：描述融合非线性变换及其设备专用快速路径。

## Dependencies / 依赖关系
- **Standard library**: `typing.Optional` and `typing.Tuple` / **标准库**：`typing.Optional` 和 `typing.Tuple`
- **Third-party**: `torch` and `torch.nn` / **第三方依赖**：`torch` 和 `torch.nn`
- **Internal SGLang modules**: `sglang.srt.layers.dp_attention.get_attention_tp_size`, `sglang.srt.layers.linear.ColumnParallelLinear`, `sglang.srt.layers.linear.MergedColumnParallelLinear`, `sglang.srt.layers.linear.QKVParallelLinear`, `sglang.srt.layers.linear.RowParallelLinear`, `sglang.srt.layers.quantization.base_config.QuantizationConfig`, and `sglang.srt.utils.add_prefix` / **SGLang 内部模块**：`sglang.srt.layers.dp_attention.get_attention_tp_size`、`sglang.srt.layers.linear.ColumnParallelLinear`、`sglang.srt.layers.linear.MergedColumnParallelLinear`、`sglang.srt.layers.linear.QKVParallelLinear`、`sglang.srt.layers.linear.RowParallelLinear`、`sglang.srt.layers.quantization.base_config.QuantizationConfig` 以及 `sglang.srt.utils.add_prefix`
