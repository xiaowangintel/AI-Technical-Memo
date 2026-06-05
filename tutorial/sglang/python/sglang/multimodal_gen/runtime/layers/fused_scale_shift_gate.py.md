# fused_scale_shift_gate.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/fused_scale_shift_gate.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `FusedLayerNormScaleShiftGateSelect01`, and `FusedResidualLayerNormScaleShiftGateSelect01`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `FusedLayerNormScaleShiftGateSelect01` 和 `FusedResidualLayerNormScaleShiftGateSelect01` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 3-9: module setup and imports / 模块初始化与导入
```python
from typing import Optional, Tuple

import torch
import torch.nn.functional as F

from sglang.multimodal_gen.runtime.layers.custom_op import CustomOp
from sglang.multimodal_gen.runtime.platforms import current_platform
```
**EN:** This block establishes the module context and imports `typing`, `torch`, `torch.nn.functional`, `sglang.multimodal_gen.runtime.layers.custom_op`, and `sglang.multimodal_gen.runtime.platforms`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `typing`、`torch`、`torch.nn.functional`、`sglang.multimodal_gen.runtime.layers.custom_op` 和 `sglang.multimodal_gen.runtime.platforms`。这些依赖为后续实现提供所需符号。

### Lines 11-16: supporting statements / 辅助语句
```python
_is_cuda = current_platform.is_cuda()
if _is_cuda:
    from sglang.jit_kernel.diffusion.triton.scale_shift import (
        fuse_layernorm_scale_shift_gate_select01_kernel,
        fuse_residual_layernorm_scale_shift_gate_select01_kernel,
    )
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `_is_cuda`. The code collaborates with `current_platform.is_cuda`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `_is_cuda` 等名称。 代码会与 `current_platform.is_cuda` 协同工作。

### Lines 20-25: `FusedLayerNormScaleShiftGateSelect01` class overview / `FusedLayerNormScaleShiftGateSelect01` 类概览
```python
class FusedLayerNormScaleShiftGateSelect01(CustomOp):
    """Fused layernorm + scale/shift + gate with binary index selection.

    CUDA path uses a Triton kernel; other platforms fall back to PyTorch ops.
    """
```
**EN:** This block defines class `FusedLayerNormScaleShiftGateSelect01`. Fused layernorm + scale/shift + gate with binary index selection. CUDA path uses a Triton kernel; other platforms fall back to PyTorch ops. It inherits from `CustomOp`.
**CN:** 该代码块定义了类 `FusedLayerNormScaleShiftGateSelect01`。 它用于封装 fused layer norm scale shift gate select01 相关行为。 它继承自 `CustomOp`。

### Lines 26-56: `forward_cuda` implementation / `forward_cuda` 实现
```python
    def forward_cuda(
        self,
        x: torch.Tensor,
        weight: Optional[torch.Tensor],
        bias: Optional[torch.Tensor],
        scale0: torch.Tensor,
        shift0: torch.Tensor,
        gate0: torch.Tensor,
        scale1: torch.Tensor,
        shift1: torch.Tensor,
        gate1: torch.Tensor,
        index: torch.Tensor,
        eps: float,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        if not x.is_contiguous():
            x = x.contiguous()
        if not index.is_contiguous():
            index = index.contiguous()
        return fuse_layernorm_scale_shift_gate_select01_kernel(
            x,
            weight=weight,
            bias=bias,
            scale0=scale0.contiguous(),
            shift0=shift0.contiguous(),
            gate0=gate0.contiguous(),
            scale1=scale1.contiguous(),
            shift1=shift1.contiguous(),
            gate1=gate1.contiguous(),
            index=index,
            eps=eps,
        )
```
**EN:** This block defines method `forward_cuda` on `FusedLayerNormScaleShiftGateSelect01`. It executes cuda. Key calls include `fuse_layernorm_scale_shift_gate_select01_kernel`, `x.is_contiguous`, `x.contiguous`, `index.is_contiguous`, and `index.contiguous`. The implementation branches on conditions. Parameters such as `x`, `weight`, `bias`, `scale0`, and `shift0` drive the behavior in this section.
**CN:** 该代码块定义了 `FusedLayerNormScaleShiftGateSelect01` 的方法 `forward_cuda`。 它用于执行前向计算cuda。 关键调用包括 `fuse_layernorm_scale_shift_gate_select01_kernel`、`x.is_contiguous`、`x.contiguous`、`index.is_contiguous` 和 `index.contiguous`。 实现中包含条件分支。 本段逻辑主要由 `x`、`weight`、`bias`、`scale0` 和 `shift0` 等参数驱动。

### Lines 58-59: `forward_hip` implementation / `forward_hip` 实现
```python
    def forward_hip(self, *args, **kwargs):
        return self.forward_native(*args, **kwargs)
```
**EN:** This block defines method `forward_hip` on `FusedLayerNormScaleShiftGateSelect01`. It executes hip. Key calls include `self.forward_native`.
**CN:** 该代码块定义了 `FusedLayerNormScaleShiftGateSelect01` 的方法 `forward_hip`。 它用于执行前向计算hip。 关键调用包括 `self.forward_native`。

### Lines 61-81: `forward_native` implementation / `forward_native` 实现
```python
    def forward_native(
        self,
        x: torch.Tensor,
        weight: Optional[torch.Tensor],
        bias: Optional[torch.Tensor],
        scale0: torch.Tensor,
        shift0: torch.Tensor,
        gate0: torch.Tensor,
        scale1: torch.Tensor,
        shift1: torch.Tensor,
        gate1: torch.Tensor,
        index: torch.Tensor,
        eps: float,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        idx = index.to(dtype=torch.bool).unsqueeze(-1)
        shift = torch.where(idx, shift1.unsqueeze(1), shift0.unsqueeze(1))
        scale = torch.where(idx, scale1.unsqueeze(1), scale0.unsqueeze(1))
        gate = torch.where(idx, gate1.unsqueeze(1), gate0.unsqueeze(1))
        x = F.layer_norm(x, (x.shape[-1],), weight=weight, bias=bias, eps=eps)
        x = x * (1 + scale) + shift
        return x, gate
```
**EN:** This block defines method `forward_native` on `FusedLayerNormScaleShiftGateSelect01`. It executes native. Key calls include `index.to.unsqueeze`, `torch.where`, `F.layer_norm`, `shift1.unsqueeze`, and `shift0.unsqueeze`. Parameters such as `x`, `weight`, `bias`, `scale0`, and `shift0` drive the behavior in this section.
**CN:** 该代码块定义了 `FusedLayerNormScaleShiftGateSelect01` 的方法 `forward_native`。 它用于执行前向计算native。 关键调用包括 `index.to.unsqueeze`、`torch.where`、`F.layer_norm`、`shift1.unsqueeze` 和 `shift0.unsqueeze`。 本段逻辑主要由 `x`、`weight`、`bias`、`scale0` 和 `shift0` 等参数驱动。

### Lines 85-90: `FusedResidualLayerNormScaleShiftGateSelect01` class overview / `FusedResidualLayerNormScaleShiftGateSelect01` 类概览
```python
class FusedResidualLayerNormScaleShiftGateSelect01(CustomOp):
    """Fused residual + layernorm + scale/shift + gate with binary index selection.

    CUDA path uses a Triton kernel; other platforms fall back to PyTorch ops.
    """
```
**EN:** This block defines class `FusedResidualLayerNormScaleShiftGateSelect01`. Fused residual + layernorm + scale/shift + gate with binary index selection. CUDA path uses a Triton kernel; other platforms fall back to PyTorch ops. It inherits from `CustomOp`.
**CN:** 该代码块定义了类 `FusedResidualLayerNormScaleShiftGateSelect01`。 它用于封装 fused residual layer norm scale shift gate select01 相关行为。 它继承自 `CustomOp`。

### Lines 91-129: `forward_cuda` implementation / `forward_cuda` 实现
```python
    def forward_cuda(
        self,
        x: torch.Tensor,
        residual: torch.Tensor,
        residual_gate: torch.Tensor,
        weight: Optional[torch.Tensor],
        bias: Optional[torch.Tensor],
        scale0: torch.Tensor,
        shift0: torch.Tensor,
        gate0: torch.Tensor,
        scale1: torch.Tensor,
        shift1: torch.Tensor,
        gate1: torch.Tensor,
        index: torch.Tensor,
        eps: float,
    ) -> Tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
        if not x.is_contiguous():
            x = x.contiguous()
        if not index.is_contiguous():
            index = index.contiguous()
        if not residual.is_contiguous():
            residual = residual.contiguous()
        if not residual_gate.is_contiguous():
            residual_gate = residual_gate.contiguous()
        return fuse_residual_layernorm_scale_shift_gate_select01_kernel(
            x,
            residual=residual,
            residual_gate=residual_gate,
            weight=weight,
            bias=bias,
            scale0=scale0.contiguous(),
            shift0=shift0.contiguous(),
            gate0=gate0.contiguous(),
            scale1=scale1.contiguous(),
            shift1=shift1.contiguous(),
            gate1=gate1.contiguous(),
            index=index,
            eps=eps,
        )
```
**EN:** This block defines method `forward_cuda` on `FusedResidualLayerNormScaleShiftGateSelect01`. It executes cuda. Key calls include `fuse_residual_layernorm_scale_shift_gate_select01_kernel`, `x.is_contiguous`, `x.contiguous`, `index.is_contiguous`, and `index.contiguous`. The implementation branches on conditions. Parameters such as `x`, `residual`, `residual_gate`, `weight`, and `bias` drive the behavior in this section.
**CN:** 该代码块定义了 `FusedResidualLayerNormScaleShiftGateSelect01` 的方法 `forward_cuda`。 它用于执行前向计算cuda。 关键调用包括 `fuse_residual_layernorm_scale_shift_gate_select01_kernel`、`x.is_contiguous`、`x.contiguous`、`index.is_contiguous` 和 `index.contiguous`。 实现中包含条件分支。 本段逻辑主要由 `x`、`residual`、`residual_gate`、`weight` 和 `bias` 等参数驱动。

### Lines 131-132: `forward_hip` implementation / `forward_hip` 实现
```python
    def forward_hip(self, *args, **kwargs):
        return self.forward_native(*args, **kwargs)
```
**EN:** This block defines method `forward_hip` on `FusedResidualLayerNormScaleShiftGateSelect01`. It executes hip. Key calls include `self.forward_native`.
**CN:** 该代码块定义了 `FusedResidualLayerNormScaleShiftGateSelect01` 的方法 `forward_hip`。 它用于执行前向计算hip。 关键调用包括 `self.forward_native`。

### Lines 134-159: `forward_native` implementation / `forward_native` 实现
```python
    def forward_native(
        self,
        x: torch.Tensor,
        residual: torch.Tensor,
        residual_gate: torch.Tensor,
        weight: Optional[torch.Tensor],
        bias: Optional[torch.Tensor],
        scale0: torch.Tensor,
        shift0: torch.Tensor,
        gate0: torch.Tensor,
        scale1: torch.Tensor,
        shift1: torch.Tensor,
        gate1: torch.Tensor,
        index: torch.Tensor,
        eps: float,
    ) -> Tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
        idx = index.to(dtype=torch.bool).unsqueeze(-1)
        shift = torch.where(idx, shift1.unsqueeze(1), shift0.unsqueeze(1))
        scale = torch.where(idx, scale1.unsqueeze(1), scale0.unsqueeze(1))
        gate = torch.where(idx, gate1.unsqueeze(1), gate0.unsqueeze(1))
        residual_out = residual_gate * x + residual
        x = F.layer_norm(
            residual_out, (residual_out.shape[-1],), weight=weight, bias=bias, eps=eps
        )
        x = x * (1 + scale) + shift
        return x, residual_out, gate
```
**EN:** This block defines method `forward_native` on `FusedResidualLayerNormScaleShiftGateSelect01`. It executes native. Key calls include `index.to.unsqueeze`, `torch.where`, `F.layer_norm`, `shift1.unsqueeze`, and `shift0.unsqueeze`. Parameters such as `x`, `residual`, `residual_gate`, `weight`, and `bias` drive the behavior in this section.
**CN:** 该代码块定义了 `FusedResidualLayerNormScaleShiftGateSelect01` 的方法 `forward_native`。 它用于执行前向计算native。 关键调用包括 `index.to.unsqueeze`、`torch.where`、`F.layer_norm`、`shift1.unsqueeze` 和 `shift0.unsqueeze`。 本段逻辑主要由 `x`、`residual`、`residual_gate`、`weight` 和 `bias` 等参数驱动。

## Key Concepts / 关键概念
- `FusedLayerNormScaleShiftGateSelect01`: Fused layernorm + scale/shift + gate with binary index selection. / 核心类，用于封装 fused layer norm scale shift gate select01 相关行为。
- `FusedResidualLayerNormScaleShiftGateSelect01`: Fused residual + layernorm + scale/shift + gate with binary index selection. / 核心类，用于封装 fused residual layer norm scale shift gate select01 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
- **Third-party / 第三方依赖**: `torch`, `torch.nn.functional`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.layers.custom_op`, `sglang.multimodal_gen.runtime.platforms`, `sglang.jit_kernel.diffusion.triton.scale_shift`

- **Total lines / 总行数**: 159
