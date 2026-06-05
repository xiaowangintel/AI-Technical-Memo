# layernorm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/layernorm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `RMSNorm`, `LayerNorm`, and `FP32LayerNorm`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Custom normalization layers. / 该文件属于运行时算子层。它围绕 `RMSNorm`、`LayerNorm` 和 `FP32LayerNorm` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 5-28: module setup and imports / 模块初始化与导入
```python
"""Custom normalization layers."""

import os
from typing import Optional, Tuple, Union

import torch
import torch.nn as nn
import torch.nn.functional as F

from sglang.jit_kernel.diffusion.qknorm_rope import (
    can_use_fused_inplace_qknorm_rope,
    fused_inplace_qknorm_rope,
)
from sglang.jit_kernel.diffusion.triton.rmsnorm_onepass import triton_one_pass_rms_norm
from sglang.jit_kernel.diffusion.triton.scale_shift import fuse_scale_shift_kernel
from sglang.jit_kernel.norm import can_use_fused_inplace_qknorm, fused_inplace_qknorm
from sglang.multimodal_gen.runtime.distributed.parallel_state import (
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
    get_tp_group,
)
from sglang.multimodal_gen.runtime.layers.custom_op import CustomOp
from sglang.multimodal_gen.runtime.platforms import current_platform
from sglang.multimodal_gen.runtime.utils.common import get_bool_env_var
```
**EN:** This block establishes the module context and imports `os`, `typing`, `torch`, `torch.nn`, `torch.nn.functional`, and `sglang.jit_kernel.diffusion.qknorm_rope`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `os`、`typing`、`torch`、`torch.nn`、`torch.nn.functional` 和 `sglang.jit_kernel.diffusion.qknorm_rope`。这些依赖为后续实现提供所需符号。

### Lines 30-52: supporting statements / 辅助语句
```python
_is_cuda = current_platform.is_cuda()
_is_hip = current_platform.is_hip()
_is_npu = current_platform.is_npu()
_is_musa = current_platform.is_musa()
_is_cpu = current_platform.is_cpu()
_is_xpu = current_platform.is_xpu()
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip

if _is_cuda or _is_xpu:
    from sgl_kernel import fused_add_rmsnorm, rmsnorm

if _is_npu:
    import torch_npu

if _is_musa:
    from sgl_kernel import fused_add_rmsnorm

if _use_aiter:
    from aiter import rmsnorm2d_fwd as rms_norm
    from aiter import rmsnorm2d_fwd_with_add as fused_add_rms_norm

if not _is_cpu:
    from sglang.jit_kernel.diffusion.triton.norm import norm_infer, rms_norm_fn
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `_is_cuda`, `_is_hip`, `_is_npu`, `_is_musa`, `_is_cpu`, and `_is_xpu`. The code collaborates with `current_platform.is_cuda`, `current_platform.is_hip`, `current_platform.is_npu`, `current_platform.is_musa`, `current_platform.is_cpu`, and `current_platform.is_xpu`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `_is_cuda`、`_is_hip`、`_is_npu`、`_is_musa`、`_is_cpu` 和 `_is_xpu` 等名称。 代码会与 `current_platform.is_cuda`、`current_platform.is_hip`、`current_platform.is_npu`、`current_platform.is_musa`、`current_platform.is_cpu` 和 `current_platform.is_xpu` 协同工作。

### Lines 57-63: `RMSNorm` class overview / `RMSNorm` 类概览
```python
class RMSNorm(CustomOp):
    """Root mean square normalization.

    Computes x -> w * x / sqrt(E[x^2] + eps) where w is the learned weight.
    Refer to https://arxiv.org/abs/1910.07467
    """
```
**EN:** This block defines class `RMSNorm`. Root mean square normalization. Computes x -> w * x / sqrt(E[x^2] + eps) where w is the learned weight. It inherits from `CustomOp`.
**CN:** 该代码块定义了类 `RMSNorm`。 它用于封装 rmsnorm 相关行为。 它继承自 `CustomOp`。

### Lines 64-81: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        hidden_size: int,
        eps: float = 1e-6,
        dtype: torch.dtype = torch.float32,
        var_hidden_size: Optional[int] = None,
    ) -> None:
        super().__init__()
        self.weight = nn.Parameter(torch.ones(hidden_size))
        self.variance_epsilon = eps
        self.hidden_size = hidden_size
        self.variance_size_override = (
            None if var_hidden_size == hidden_size else var_hidden_size
        )
        if get_bool_env_var("SGLANG_ENABLE_DETERMINISTIC_INFERENCE"):
            self._forward_method = self.forward_native
        elif _use_aiter:
            self._forward_method = self.forward_aiter
```
**EN:** This block defines method `__init__` on `RMSNorm`. It initializes the instance state. Key calls include `super.__init__`, `nn.Parameter`, `get_bool_env_var`, `torch.ones`, and `super`. The implementation branches on conditions. Parameters such as `hidden_size`, `eps`, `dtype`, and `var_hidden_size` drive the behavior in this section.
**CN:** 该代码块定义了 `RMSNorm` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`nn.Parameter`、`get_bool_env_var`、`torch.ones` 和 `super`。 实现中包含条件分支。 本段逻辑主要由 `hidden_size`、`eps`、`dtype` 和 `var_hidden_size` 等参数驱动。

### Lines 83-86: `forward_triton` implementation / `forward_triton` 实现
```python
    def forward_triton(self, x: torch.Tensor, residual: Optional[torch.Tensor] = None):
        return rms_norm_fn(
            x, self.weight, bias=None, residual=residual, eps=self.variance_epsilon
        )
```
**EN:** This block defines method `forward_triton` on `RMSNorm`. It executes triton. Key calls include `rms_norm_fn`. Parameters such as `x`, and `residual` drive the behavior in this section.
**CN:** 该代码块定义了 `RMSNorm` 的方法 `forward_triton`。 它用于执行前向计算triton。 关键调用包括 `rms_norm_fn`。 本段逻辑主要由 `x` 和 `residual` 等参数驱动。

### Lines 88-121: `forward_cuda` implementation / `forward_cuda` 实现
```python
    def forward_cuda(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        shape = x.shape
        x = x.reshape(-1, shape[-1])
        if residual is not None:
            residual_shape = residual.shape
            residual = residual.view(-1, shape[-1])

        if x.dtype == torch.float:
            if residual is None and self.variance_size_override is None:
                return self.forward_native(x).view(shape)
            out = self.forward_triton(x, residual)
            if residual is not None:
                return out[0].view(shape), out[1].view(residual_shape)
            out = out.view(shape)
            return out
        elif self.variance_size_override is not None:
            return self.forward_native(x, residual)
        elif residual is not None:
            fused_add_rmsnorm(x, residual, self.weight.data, self.variance_epsilon)
            return x.view(shape), residual.view(residual_shape)
        else:
            if x.shape[-1] <= 128:
                out = triton_one_pass_rms_norm(
                    x, self.weight.data, self.variance_epsilon
                )
            else:
                out = rmsnorm(x, self.weight.data, self.variance_epsilon)
        out = out.view(shape)

        return out
```
**EN:** This block defines method `forward_cuda` on `RMSNorm`. It executes cuda. Key calls include `x.reshape`, `out.view`, `residual.view`, `self.forward_triton`, and `self.forward_native.view`. The implementation branches on conditions. Parameters such as `x`, and `residual` drive the behavior in this section.
**CN:** 该代码块定义了 `RMSNorm` 的方法 `forward_cuda`。 它用于执行前向计算cuda。 关键调用包括 `x.reshape`、`out.view`、`residual.view`、`self.forward_triton` 和 `self.forward_native.view`。 实现中包含条件分支。 本段逻辑主要由 `x` 和 `residual` 等参数驱动。

### Lines 123-160: `forward_native` implementation / `forward_native` 实现
```python
    def forward_native(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        if not x.is_contiguous():
            x = x.contiguous()
        orig_dtype = x.dtype
        x = x.to(torch.float32)
        if residual is not None:
            x = x + residual.to(torch.float32)
            residual = x.to(orig_dtype)

        hidden_size = x.shape[-1]
        if hidden_size != self.hidden_size:
            raise ValueError(
                "Expected hidden_size to be "
                f"{self.hidden_size}, but found: {hidden_size}"
            )

        if self.variance_size_override is None:
            x_var = x
        else:
            if hidden_size < self.variance_size_override:
                raise ValueError(
                    "Expected hidden_size to be at least "
                    f"{self.variance_size_override}, but found: {hidden_size}"
                )

            x_var = x[..., : self.variance_size_override]

        variance = x_var.pow(2).mean(dim=-1, keepdim=True)
        x = x * torch.rsqrt(variance + self.variance_epsilon)
        x = (x * self.weight).to(orig_dtype)
        if residual is None:
            return x
        else:
            return x, residual
```
**EN:** This block defines method `forward_native` on `RMSNorm`. It executes native. Key calls include `x.to`, `x_var.pow.mean`, `to`, `x.is_contiguous`, and `x.contiguous`. The implementation branches on conditions. Parameters such as `x`, and `residual` drive the behavior in this section.
**CN:** 该代码块定义了 `RMSNorm` 的方法 `forward_native`。 它用于执行前向计算native。 关键调用包括 `x.to`、`x_var.pow.mean`、`to`、`x.is_contiguous` 和 `x.contiguous`。 实现中包含条件分支。 本段逻辑主要由 `x` 和 `residual` 等参数驱动。

### Lines 162-167: `forward_cpu` implementation / `forward_cpu` 实现
```python
    def forward_cpu(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        return self.forward_native(x, residual)
```
**EN:** This block defines method `forward_cpu` on `RMSNorm`. It executes cpu. Key calls include `self.forward_native`. Parameters such as `x`, and `residual` drive the behavior in this section.
**CN:** 该代码块定义了 `RMSNorm` 的方法 `forward_cpu`。 它用于执行前向计算cpu。 关键调用包括 `self.forward_native`。 本段逻辑主要由 `x` 和 `residual` 等参数驱动。

### Lines 169-179: `forward_npu` implementation / `forward_npu` 实现
```python
    def forward_npu(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        if residual is not None:
            out, _, residual_out = torch_npu.npu_add_rms_norm(
                residual, x, self.weight.data, self.variance_epsilon
            )
            return out, residual_out
        return torch_npu.npu_rms_norm(x, self.weight.data, self.variance_epsilon)[0]
```
**EN:** This block defines method `forward_npu` on `RMSNorm`. It executes npu. Key calls include `torch_npu.npu_add_rms_norm`, and `torch_npu.npu_rms_norm`. The implementation branches on conditions. Parameters such as `x`, and `residual` drive the behavior in this section.
**CN:** 该代码块定义了 `RMSNorm` 的方法 `forward_npu`。 它用于执行前向计算npu。 关键调用包括 `torch_npu.npu_add_rms_norm` 和 `torch_npu.npu_rms_norm`。 实现中包含条件分支。 本段逻辑主要由 `x` 和 `residual` 等参数驱动。

### Lines 181-187: `forward_hip` implementation / `forward_hip` 实现
```python
    def forward_hip(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        # ROCm builds of sgl-kernel do not expose rmsnorm custom ops yet.
        return self.forward_native(x, residual)
```
**EN:** This block defines method `forward_hip` on `RMSNorm`. It executes hip. Key calls include `self.forward_native`. Parameters such as `x`, and `residual` drive the behavior in this section.
**CN:** 该代码块定义了 `RMSNorm` 的方法 `forward_hip`。 它用于执行前向计算hip。 关键调用包括 `self.forward_native`。 本段逻辑主要由 `x` 和 `residual` 等参数驱动。

### Lines 189-228: `forward_aiter` implementation / `forward_aiter` 实现
```python
    def forward_aiter(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        # Fall back to the native fp32 path for cases aiter cannot serve:
        #   - fp32 input  (CK kernel is templated on fp16/bf16 only;
        #                  out.dtype check rejects fp32 with "not support output type: float")
        if (
            x.dtype not in (torch.float16, torch.bfloat16)
            or self.variance_size_override is not None
        ):
            return self.forward_native(x, residual)

        weight = self._get_weight(x.dtype)

        shape = x.shape
        x_2d = x.reshape(
            -1, shape[-1]
        )  # (bs, seq_len, hidden_size) -> (bs*seq_len, hidden_size)
        if not x_2d.is_contiguous():
            x_2d = x_2d.contiguous()

        if residual is not None:
            residual_shape = residual.shape
            residual_2d = residual.reshape(-1, shape[-1])
            if not residual_2d.is_contiguous():
                residual_2d = residual_2d.contiguous()
            output = torch.empty_like(x_2d)
            residual_out = torch.empty_like(x_2d)
            fused_add_rms_norm(
                output,
                x_2d,
                residual_2d,
                residual_out,
                weight,
                self.variance_epsilon,
            )
            return output.view(shape), residual_out.view(residual_shape)
        return rms_norm(x_2d, weight, self.variance_epsilon).view(shape)
```
**EN:** This block defines method `forward_aiter` on `RMSNorm`. It executes aiter. Key calls include `self._get_weight`, `x.reshape`, `rms_norm.view`, `self.forward_native`, and `x_2d.is_contiguous`. The implementation branches on conditions. Parameters such as `x`, and `residual` drive the behavior in this section.
**CN:** 该代码块定义了 `RMSNorm` 的方法 `forward_aiter`。 它用于执行前向计算aiter。 关键调用包括 `self._get_weight`、`x.reshape`、`rms_norm.view`、`self.forward_native` 和 `x_2d.is_contiguous`。 实现中包含条件分支。 本段逻辑主要由 `x` 和 `residual` 等参数驱动。

### Lines 230-239: `_get_weight` implementation / `_get_weight` 实现
```python
    def _get_weight(self, dtype: torch.dtype) -> torch.Tensor:
        """Return weight matched to *dtype*.

        MUSA kernels require input and weight to share the same dtype,
        unlike CUDA kernels which may handle mixed dtypes internally.
        """
        weight = self.weight.data
        if weight.dtype != dtype:
            weight = weight.to(dtype=dtype)
        return weight
```
**EN:** This block defines method `_get_weight` on `RMSNorm`. Return weight matched to *dtype*. MUSA kernels require input and weight to share the same dtype, unlike CUDA kernels which may handle mixed dtypes internally. Key calls include `weight.to`. The implementation branches on conditions. Parameters such as `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `RMSNorm` 的方法 `_get_weight`。 它用于获取weight。 关键调用包括 `weight.to`。 实现中包含条件分支。 本段逻辑主要由 `dtype` 等参数驱动。

### Lines 241-267: `forward_musa` implementation / `forward_musa` 实现
```python
    def forward_musa(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        shape = x.shape
        x = x.reshape(-1, shape[-1])
        if residual is not None:
            residual_shape = residual.shape
            residual = residual.view(-1, shape[-1])

        if self.variance_size_override is not None:
            return self.forward_native(x, residual)
        elif residual is not None:
            # fused_add_rmsnorm requires contiguous inputs.
            if not x.is_contiguous():
                x = x.contiguous()
            if not residual.is_contiguous():
                residual = residual.contiguous()
            weight = self._get_weight(x.dtype)
            fused_add_rmsnorm(x, residual, weight, self.variance_epsilon)
            return x.view(shape), residual.view(residual_shape)
        else:
            weight = self._get_weight(x.dtype)
            out = F.rms_norm(x, (self.hidden_size,), weight, self.variance_epsilon)
        out = out.view(shape)
        return out
```
**EN:** This block defines method `forward_musa` on `RMSNorm`. It executes musa. Key calls include `x.reshape`, `out.view`, `residual.view`, `self.forward_native`, and `self._get_weight`. The implementation branches on conditions. Parameters such as `x`, and `residual` drive the behavior in this section.
**CN:** 该代码块定义了 `RMSNorm` 的方法 `forward_musa`。 它用于执行前向计算musa。 关键调用包括 `x.reshape`、`out.view`、`residual.view`、`self.forward_native` 和 `self._get_weight`。 实现中包含条件分支。 本段逻辑主要由 `x` 和 `residual` 等参数驱动。

### Lines 269-288: `forward_xpu` implementation / `forward_xpu` 实现
```python
    def forward_xpu(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        shape = x.shape
        x = x.reshape(-1, shape[-1])
        if residual is not None:
            residual_shape = residual.shape
            residual = residual.view(-1, shape[-1])

        if self.variance_size_override is not None:
            return self.forward_native(x, residual)
        elif residual is not None:
            fused_add_rmsnorm(x, residual, self.weight.data, self.variance_epsilon)
            return x.view(shape), residual.view(residual_shape)
        else:
            out = rmsnorm(x, self.weight.data, self.variance_epsilon)
        out = out.view(shape)
        return out
```
**EN:** This block defines method `forward_xpu` on `RMSNorm`. It executes xpu. Key calls include `x.reshape`, `out.view`, `residual.view`, `self.forward_native`, and `fused_add_rmsnorm`. The implementation branches on conditions. Parameters such as `x`, and `residual` drive the behavior in this section.
**CN:** 该代码块定义了 `RMSNorm` 的方法 `forward_xpu`。 它用于执行前向计算xpu。 关键调用包括 `x.reshape`、`out.view`、`residual.view`、`self.forward_native` 和 `fused_add_rmsnorm`。 实现中包含条件分支。 本段逻辑主要由 `x` 和 `residual` 等参数驱动。

### Lines 290-291: `extra_repr` implementation / `extra_repr` 实现
```python
    def extra_repr(self) -> str:
        return f"hidden_size={self.hidden_size}, eps={self.variance_epsilon}"
```
**EN:** This block defines method `extra_repr` on `RMSNorm`. It handles extra repr logic.
**CN:** 该代码块定义了 `RMSNorm` 的方法 `extra_repr`。 它用于处理 extra repr 相关逻辑。

### Lines 296-296: `LayerNorm` class overview / `LayerNorm` 类概览
```python
class LayerNorm(CustomOp):
```
**EN:** This block defines class `LayerNorm`. It encapsulates layer norm behavior. It inherits from `CustomOp`.
**CN:** 该代码块定义了类 `LayerNorm`。 它用于封装 layer norm 相关行为。 它继承自 `CustomOp`。

### Lines 297-321: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        hidden_size: int,
        eps=1e-5,
        bias: bool = True,
        elementwise_affine=True,
        device=None,
        dtype=None,
    ) -> None:
        super().__init__()
        self.eps = eps
        factory_kwargs = {"device": device, "dtype": dtype}
        self.hidden_size = hidden_size
        if elementwise_affine:
            self.weight = torch.nn.Parameter(torch.empty(hidden_size, **factory_kwargs))
            self.bias = (
                torch.nn.Parameter(torch.empty(hidden_size, **factory_kwargs))
                if bias
                else None
            )
        else:
            self.register_parameter("weight", None)
            self.register_parameter("bias", None)
            # Lazy cache for ones vector (not a registered buffer to avoid FSDP/meta issues)
            self._weight_fallback_cache = None
```
**EN:** This block defines method `__init__` on `LayerNorm`. It initializes the instance state. Key calls include `super.__init__`, `torch.nn.Parameter`, `self.register_parameter`, `super`, and `torch.empty`. The implementation branches on conditions. Parameters such as `hidden_size`, `eps`, `bias`, `elementwise_affine`, and `device` drive the behavior in this section.
**CN:** 该代码块定义了 `LayerNorm` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`torch.nn.Parameter`、`self.register_parameter`、`super` 和 `torch.empty`。 实现中包含条件分支。 本段逻辑主要由 `hidden_size`、`eps`、`bias`、`elementwise_affine` 和 `device` 等参数驱动。

### Lines 323-333: `_get_weight_fallback` implementation / `_get_weight_fallback` 实现
```python
    def _get_weight_fallback(self, x: torch.Tensor) -> torch.Tensor:
        wf = getattr(self, "_weight_fallback_cache", None)
        if (
            wf is None
            or wf.device != x.device
            or wf.dtype != x.dtype
            or wf.numel() != self.hidden_size
        ):
            wf = torch.ones(self.hidden_size, device=x.device, dtype=x.dtype)
            self._weight_fallback_cache = wf
        return wf
```
**EN:** This block defines method `_get_weight_fallback` on `LayerNorm`. It retrieves weight fallback. Key calls include `getattr`, `torch.ones`, and `wf.numel`. The implementation branches on conditions. Parameters such as `x` drive the behavior in this section.
**CN:** 该代码块定义了 `LayerNorm` 的方法 `_get_weight_fallback`。 它用于获取weight fallback。 关键调用包括 `getattr`、`torch.ones` 和 `wf.numel`。 实现中包含条件分支。 本段逻辑主要由 `x` 等参数驱动。

### Lines 335-343: `forward_triton` implementation / `forward_triton` 实现
```python
    def forward_triton(self, x: torch.Tensor):
        # Fast inference kernel without residual/dropout branches
        return norm_infer(
            x.view(-1, self.hidden_size),
            self.weight,
            self.bias,
            eps=self.eps,
            is_rms_norm=False,
        ).view(x.shape)
```
**EN:** This block defines method `forward_triton` on `LayerNorm`. It executes triton. Key calls include `norm_infer.view`, `norm_infer`, and `x.view`. Parameters such as `x` drive the behavior in this section.
**CN:** 该代码块定义了 `LayerNorm` 的方法 `forward_triton`。 它用于执行前向计算triton。 关键调用包括 `norm_infer.view`、`norm_infer` 和 `x.view`。 本段逻辑主要由 `x` 等参数驱动。

### Lines 345-351: `forward_cuda` implementation / `forward_cuda` 实现
```python
    def forward_cuda(
        self,
        x: torch.Tensor,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        shape = x.shape
        x = x.view(-1, self.hidden_size)
        return self.forward_triton(x).view(shape)
```
**EN:** This block defines method `forward_cuda` on `LayerNorm`. It executes cuda. Key calls include `x.view`, `self.forward_triton.view`, and `self.forward_triton`. Parameters such as `x` drive the behavior in this section.
**CN:** 该代码块定义了 `LayerNorm` 的方法 `forward_cuda`。 它用于执行前向计算cuda。 关键调用包括 `x.view`、`self.forward_triton.view` 和 `self.forward_triton`。 本段逻辑主要由 `x` 等参数驱动。

### Lines 353-368: `forward_native` implementation / `forward_native` 实现
```python
    @torch.compile(backend="inductor", disable=current_platform.is_npu())
    def forward_native(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        input_dtype = x.dtype
        mean = x.mean(-1, keepdim=True)
        variance = (x - mean).pow(2).mean(-1, keepdim=True)
        x = (x - mean) * torch.rsqrt(variance + self.eps)
        if self.weight is not None:
            x = self.weight * x
        # if no affine, this is a no-op
        if self.bias is not None:
            x = x + self.bias
        return x.to(input_dtype)
```
**EN:** This block defines method `forward_native` on `LayerNorm`. It executes native. Key calls include `torch.compile`, `x.mean`, `pow.mean`, `x.to`, and `torch.rsqrt`. The implementation branches on conditions. Parameters such as `x`, and `residual` drive the behavior in this section.
**CN:** 该代码块定义了 `LayerNorm` 的方法 `forward_native`。 它用于执行前向计算native。 关键调用包括 `torch.compile`、`x.mean`、`pow.mean`、`x.to` 和 `torch.rsqrt`。 实现中包含条件分支。 本段逻辑主要由 `x` 和 `residual` 等参数驱动。

### Lines 370-375: `forward_cpu` implementation / `forward_cpu` 实现
```python
    def forward_cpu(
        self,
        x: torch.Tensor,
        residual: Optional[torch.Tensor] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, torch.Tensor]]:
        return self.forward_native(x, residual)
```
**EN:** This block defines method `forward_cpu` on `LayerNorm`. It executes cpu. Key calls include `self.forward_native`. Parameters such as `x`, and `residual` drive the behavior in this section.
**CN:** 该代码块定义了 `LayerNorm` 的方法 `forward_cpu`。 它用于执行前向计算cpu。 关键调用包括 `self.forward_native`。 本段逻辑主要由 `x` 和 `residual` 等参数驱动。

### Lines 377-378: `forward_musa` implementation / `forward_musa` 实现
```python
    def forward_musa(self, x: torch.Tensor):
        return F.layer_norm(x, (self.hidden_size,), self.weight, self.bias, self.eps)
```
**EN:** This block defines method `forward_musa` on `LayerNorm`. It executes musa. Key calls include `F.layer_norm`. Parameters such as `x` drive the behavior in this section.
**CN:** 该代码块定义了 `LayerNorm` 的方法 `forward_musa`。 它用于执行前向计算musa。 关键调用包括 `F.layer_norm`。 本段逻辑主要由 `x` 等参数驱动。

### Lines 380-383: `extra_repr` implementation / `extra_repr` 实现
```python
    def extra_repr(self) -> str:
        s = f"hidden_size={self.weight.data.size(0)}"
        s += f", eps={self.variance_epsilon}"
        return s
```
**EN:** This block defines method `extra_repr` on `LayerNorm`. It handles extra repr logic. Key calls include `self.weight.data.size`.
**CN:** 该代码块定义了 `LayerNorm` 的方法 `extra_repr`。 它用于处理 extra repr 相关逻辑。 关键调用包括 `self.weight.data.size`。

### Lines 389-389: `FP32LayerNorm` class overview / `FP32LayerNorm` 类概览
```python
class FP32LayerNorm(nn.LayerNorm):
```
**EN:** This block defines class `FP32LayerNorm`. It encapsulates fp32 layer norm behavior. It inherits from `nn.LayerNorm`.
**CN:** 该代码块定义了类 `FP32LayerNorm`。 它用于封装 fp32 layer norm 相关行为。 它继承自 `nn.LayerNorm`。

### Lines 390-399: `forward` implementation / `forward` 实现
```python
    def forward(self, inputs: torch.Tensor) -> torch.Tensor:
        origin_dtype = inputs.dtype
        device = inputs.device
        return F.layer_norm(
            inputs.float(),
            self.normalized_shape,
            self.weight.float().to(device=device) if self.weight is not None else None,
            self.bias.float().to(device=device) if self.bias is not None else None,
            self.eps,
        ).to(origin_dtype)
```
**EN:** This block defines method `forward` on `FP32LayerNorm`. It executes function. Key calls include `F.layer_norm.to`, `F.layer_norm`, `inputs.float`, `self.weight.float.to`, and `self.bias.float.to`. Parameters such as `inputs` drive the behavior in this section.
**CN:** 该代码块定义了 `FP32LayerNorm` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `F.layer_norm.to`、`F.layer_norm`、`inputs.float`、`self.weight.float.to` 和 `self.bias.float.to`。 本段逻辑主要由 `inputs` 等参数驱动。

### Lines 405-406: `_ensure_contiguous` implementation / `_ensure_contiguous` 实现
```python
def _ensure_contiguous(tensor: Optional[torch.Tensor]) -> Optional[torch.Tensor]:
    return tensor.contiguous() if tensor is not None else None
```
**EN:** This block defines function `_ensure_contiguous`. It handles ensure contiguous logic. Key calls include `tensor.contiguous`. Parameters such as `tensor` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_ensure_contiguous`。 它用于处理 ensure contiguous 相关逻辑。 关键调用包括 `tensor.contiguous`。 本段逻辑主要由 `tensor` 等参数驱动。

### Lines 409-417: `_ScaleResidualNormScaleShift` class overview / `_ScaleResidualNormScaleShift` 类概览
```python
class _ScaleResidualNormScaleShift(CustomOp):
    """
    Fused kernel that combines:
    1. residual_out = residual + gate * x
    2. normed = layernorm(residual_out) or rmsnorm(residual_out)
    3. out = normed * (1 + scale) + shift
    compute_dtype is always fp32 for higher precision.
    """
```
**EN:** This block defines class `_ScaleResidualNormScaleShift`. Fused kernel that combines: 1. residual_out = residual + gate * x 2. It inherits from `CustomOp`.
**CN:** 该代码块定义了类 `_ScaleResidualNormScaleShift`。 它用于封装 scale residual norm scale shift 相关行为。 它继承自 `CustomOp`。

### Lines 418-418: supporting statements / 辅助语句
```python
    norm_type: str
```
**EN:** This block gathers supporting statements inside `_ScaleResidualNormScaleShift`. It updates names such as `norm_type`.
**CN:** 该代码块汇集了位于 `_ScaleResidualNormScaleShift` 内部的辅助语句。 它会更新 `norm_type` 等名称。

### Lines 420-438: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        hidden_size: int,
        eps: float = 1e-6,
        elementwise_affine: bool = False,
        dtype: torch.dtype = torch.float32,
        prefix: str = "",
    ):
        super().__init__()
        self.eps = eps
        self.dtype = dtype
        if self.norm_type == "rms":
            self.norm = RMSNorm(hidden_size, eps=eps, dtype=dtype)
        elif self.norm_type == "layer":
            self.norm = FP32LayerNorm(
                hidden_size, elementwise_affine=elementwise_affine, eps=eps, dtype=dtype
            )
        else:
            raise NotImplementedError(f"Norm type {self.norm_type} not implemented")
```
**EN:** This block defines method `__init__` on `_ScaleResidualNormScaleShift`. It initializes the instance state. Key calls include `super.__init__`, `RMSNorm`, `super`, `FP32LayerNorm`, and `NotImplementedError`. The implementation branches on conditions. Parameters such as `hidden_size`, `eps`, `elementwise_affine`, `dtype`, and `prefix` drive the behavior in this section.
**CN:** 该代码块定义了 `_ScaleResidualNormScaleShift` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`RMSNorm`、`super`、`FP32LayerNorm` 和 `NotImplementedError`。 实现中包含条件分支。 本段逻辑主要由 `hidden_size`、`eps`、`elementwise_affine`、`dtype` 和 `prefix` 等参数驱动。

### Lines 440-476: `forward_cuda` implementation / `forward_cuda` 实现
```python
    def forward_cuda(
        self,
        residual: torch.Tensor,
        x: torch.Tensor,
        gate: torch.Tensor | int,
        shift: torch.Tensor,
        scale: torch.Tensor,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        if x.shape[-1] % 256 != 0 and x.shape[-1] <= 8192:
            import warnings

            warnings.warn(
                "FusedScaleResidualNormScaleShift cuda not available, using native fallback",
                stacklevel=2,
            )
            return self.forward_native(residual, x, gate, shift, scale)

        from sglang.jit_kernel.diffusion.cutedsl.scale_residual_norm_scale_shift import (
            fused_scale_residual_norm_scale_shift,
        )

        if isinstance(gate, int) and gate != 1:
            raise ValueError(
                f"Only gate value of 1 is supported for int type, but got {gate}"
            )

        return fused_scale_residual_norm_scale_shift(
            residual.contiguous(),
            x.contiguous(),
            gate.contiguous() if isinstance(gate, torch.Tensor) else None,
            _ensure_contiguous(getattr(self.norm, "weight", None)),
            _ensure_contiguous(getattr(self.norm, "bias", None)),
            scale.contiguous(),
            shift.contiguous(),
            self.norm_type,
            self.eps,
        )
```
**EN:** This block defines method `forward_cuda` on `_ScaleResidualNormScaleShift`. It executes cuda. Key calls include `fused_scale_residual_norm_scale_shift`, `warnings.warn`, `self.forward_native`, `isinstance`, and `ValueError`. The implementation branches on conditions. Parameters such as `residual`, `x`, `gate`, `shift`, and `scale` drive the behavior in this section.
**CN:** 该代码块定义了 `_ScaleResidualNormScaleShift` 的方法 `forward_cuda`。 它用于执行前向计算cuda。 关键调用包括 `fused_scale_residual_norm_scale_shift`、`warnings.warn`、`self.forward_native`、`isinstance` 和 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `residual`、`x`、`gate`、`shift` 和 `scale` 等参数驱动。

### Lines 478-481: `forward_hip` implementation / `forward_hip` 实现
```python
    def forward_hip(self, *args, **kwargs):
        # ROCm does not support CUDA/CUTLASS-based fused kernels yet,
        # so we fall back to the native PyTorch implementation.
        return self.forward_native(*args, **kwargs)
```
**EN:** This block defines method `forward_hip` on `_ScaleResidualNormScaleShift`. It executes hip. Key calls include `self.forward_native`.
**CN:** 该代码块定义了 `_ScaleResidualNormScaleShift` 的方法 `forward_hip`。 它用于执行前向计算hip。 关键调用包括 `self.forward_native`。

### Lines 483-486: `forward_musa` implementation / `forward_musa` 实现
```python
    def forward_musa(self, *args, **kwargs):
        # MUSA does not support CUDA/CUTLASS-based fused kernels yet,
        # so we fall back to the native PyTorch implementation.
        return self.forward_native(*args, **kwargs)
```
**EN:** This block defines method `forward_musa` on `_ScaleResidualNormScaleShift`. It executes musa. Key calls include `self.forward_native`.
**CN:** 该代码块定义了 `_ScaleResidualNormScaleShift` 的方法 `forward_musa`。 它用于执行前向计算musa。 关键调用包括 `self.forward_native`。

### Lines 488-491: `forward_xpu` implementation / `forward_xpu` 实现
```python
    def forward_xpu(self, *args, **kwargs):
        # XPU does not support CUDA/CUTLASS-based fused kernels yet,
        # so we fall back to the native PyTorch implementation.
        return self.forward_native(*args, **kwargs)
```
**EN:** This block defines method `forward_xpu` on `_ScaleResidualNormScaleShift`. It executes xpu. Key calls include `self.forward_native`.
**CN:** 该代码块定义了 `_ScaleResidualNormScaleShift` 的方法 `forward_xpu`。 它用于执行前向计算xpu。 关键调用包括 `self.forward_native`。

### Lines 493-522: `forward_native` implementation / `forward_native` 实现
```python
    @torch.compile(disable=current_platform.is_npu())
    def forward_native(
        self,
        residual: torch.Tensor,
        x: torch.Tensor,
        gate: torch.Tensor | int,
        shift: torch.Tensor,
        scale: torch.Tensor,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        # x.shape: [batch_size, seq_len, inner_dim]
        if isinstance(gate, int):
            # used by cross-attention, should be 1
            assert gate == 1
            residual_output = residual + x
        elif isinstance(gate, torch.Tensor):
            if gate.dim() == 4:
                # gate.shape: [batch_size, num_frames, 1, inner_dim]
                num_frames = gate.shape[1]
                frame_seqlen = x.shape[1] // num_frames
                residual_output = residual + (
                    x.unflatten(dim=1, sizes=(num_frames, frame_seqlen)) * gate
                ).flatten(1, 2)
            else:
                # gate.shape: [batch_size, 1, inner_dim]
                residual_output = residual + x * gate
        else:
            raise ValueError(f"Gate type {type(gate)} not supported")
        normalized = self.norm(residual_output)
        modulated = fuse_scale_shift_kernel(normalized, scale, shift)
        return modulated, residual_output
```
**EN:** This block defines method `forward_native` on `_ScaleResidualNormScaleShift`. It executes native. Key calls include `torch.compile`, `isinstance`, `self.norm`, `fuse_scale_shift_kernel`, and `current_platform.is_npu`. The implementation branches on conditions. Parameters such as `residual`, `x`, `gate`, `shift`, and `scale` drive the behavior in this section.
**CN:** 该代码块定义了 `_ScaleResidualNormScaleShift` 的方法 `forward_native`。 它用于执行前向计算native。 关键调用包括 `torch.compile`、`isinstance`、`self.norm`、`fuse_scale_shift_kernel` 和 `current_platform.is_npu`。 实现中包含条件分支。 本段逻辑主要由 `residual`、`x`、`gate`、`shift` 和 `scale` 等参数驱动。

### Lines 524-554: `forward_npu` implementation / `forward_npu` 实现
```python
    def forward_npu(
        self,
        residual: torch.Tensor,
        x: torch.Tensor,
        gate: torch.Tensor | int,
        shift: torch.Tensor,
        scale: torch.Tensor,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        from sgl_kernel_npu.norm.scale_shift import fused_scale_shift

        # x.shape: [batch_size, seq_len, inner_dim]
        if isinstance(gate, int):
            # used by cross-attention, should be 1
            assert gate == 1
            residual_output = residual + x
        elif isinstance(gate, torch.Tensor):
            if gate.dim() == 4:
                # gate.shape: [batch_size, num_frames, 1, inner_dim]
                num_frames = gate.shape[1]
                frame_seqlen = x.shape[1] // num_frames
                residual_output = residual + (
                    x.unflatten(dim=1, sizes=(num_frames, frame_seqlen)) * gate
                ).flatten(1, 2)
            else:
                # gate.shape: [batch_size, 1, inner_dim]
                residual_output = residual + x * gate
        else:
            raise ValueError(f"Gate type {type(gate)} not supported")
        normalized = self.norm(residual_output)
        modulated = fused_scale_shift(normalized, scale, shift)
        return modulated, residual_output
```
**EN:** This block defines method `forward_npu` on `_ScaleResidualNormScaleShift`. It executes npu. Key calls include `isinstance`, `self.norm`, `fused_scale_shift`, `ValueError`, and `gate.dim`. The implementation branches on conditions. Parameters such as `residual`, `x`, `gate`, `shift`, and `scale` drive the behavior in this section.
**CN:** 该代码块定义了 `_ScaleResidualNormScaleShift` 的方法 `forward_npu`。 它用于执行前向计算npu。 关键调用包括 `isinstance`、`self.norm`、`fused_scale_shift`、`ValueError` 和 `gate.dim`。 实现中包含条件分支。 本段逻辑主要由 `residual`、`x`、`gate`、`shift` 和 `scale` 等参数驱动。

### Lines 557-557: `ScaleResidualLayerNormScaleShift` class overview / `ScaleResidualLayerNormScaleShift` 类概览
```python
class ScaleResidualLayerNormScaleShift(_ScaleResidualNormScaleShift):
```
**EN:** This block defines class `ScaleResidualLayerNormScaleShift`. It encapsulates scale residual layer norm scale shift behavior. It inherits from `_ScaleResidualNormScaleShift`.
**CN:** 该代码块定义了类 `ScaleResidualLayerNormScaleShift`。 它用于封装 scale residual layer norm scale shift 相关行为。 它继承自 `_ScaleResidualNormScaleShift`。

### Lines 558-558: supporting statements / 辅助语句
```python
    norm_type = "layer"
```
**EN:** This block gathers supporting statements inside `ScaleResidualLayerNormScaleShift`. It updates names such as `norm_type`.
**CN:** 该代码块汇集了位于 `ScaleResidualLayerNormScaleShift` 内部的辅助语句。 它会更新 `norm_type` 等名称。

### Lines 561-561: `ScaleResidualRMSNormScaleShift` class overview / `ScaleResidualRMSNormScaleShift` 类概览
```python
class ScaleResidualRMSNormScaleShift(_ScaleResidualNormScaleShift):
```
**EN:** This block defines class `ScaleResidualRMSNormScaleShift`. It encapsulates scale residual rmsnorm scale shift behavior. It inherits from `_ScaleResidualNormScaleShift`.
**CN:** 该代码块定义了类 `ScaleResidualRMSNormScaleShift`。 它用于封装 scale residual rmsnorm scale shift 相关行为。 它继承自 `_ScaleResidualNormScaleShift`。

### Lines 562-562: supporting statements / 辅助语句
```python
    norm_type = "rms"
```
**EN:** This block gathers supporting statements inside `ScaleResidualRMSNormScaleShift`. It updates names such as `norm_type`.
**CN:** 该代码块汇集了位于 `ScaleResidualRMSNormScaleShift` 内部的辅助语句。 它会更新 `norm_type` 等名称。

### Lines 565-572: `_NormScaleShift` class overview / `_NormScaleShift` 类概览
```python
class _NormScaleShift(CustomOp):
    """
    Fused kernel that combines:
    1. normed = layernorm(x) or rmsnorm(x)
    2. out = normed * (1 + scale) + shift
    compute_dtype is always fp32 for higher precision.
    """
```
**EN:** This block defines class `_NormScaleShift`. Fused kernel that combines: 1. normed = layernorm(x) or rmsnorm(x) 2. It inherits from `CustomOp`.
**CN:** 该代码块定义了类 `_NormScaleShift`。 它用于封装 norm scale shift 相关行为。 它继承自 `CustomOp`。

### Lines 573-573: supporting statements / 辅助语句
```python
    norm_type: str
```
**EN:** This block gathers supporting statements inside `_NormScaleShift`. It updates names such as `norm_type`.
**CN:** 该代码块汇集了位于 `_NormScaleShift` 内部的辅助语句。 它会更新 `norm_type` 等名称。

### Lines 575-592: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        hidden_size: int,
        eps: float = 1e-6,
        elementwise_affine: bool = False,
        dtype: torch.dtype = torch.float32,
        prefix: str = "",
    ):
        super().__init__()
        self.eps = eps
        if self.norm_type == "rms":
            self.norm = RMSNorm(hidden_size, eps=eps, dtype=dtype)
        elif self.norm_type == "layer":
            self.norm = FP32LayerNorm(
                hidden_size, elementwise_affine=elementwise_affine, eps=eps, dtype=dtype
            )
        else:
            raise NotImplementedError(f"Norm type {self.norm_type} not implemented")
```
**EN:** This block defines method `__init__` on `_NormScaleShift`. It initializes the instance state. Key calls include `super.__init__`, `RMSNorm`, `super`, `FP32LayerNorm`, and `NotImplementedError`. The implementation branches on conditions. Parameters such as `hidden_size`, `eps`, `elementwise_affine`, `dtype`, and `prefix` drive the behavior in this section.
**CN:** 该代码块定义了 `_NormScaleShift` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`RMSNorm`、`super`、`FP32LayerNorm` 和 `NotImplementedError`。 实现中包含条件分支。 本段逻辑主要由 `hidden_size`、`eps`、`elementwise_affine`、`dtype` 和 `prefix` 等参数驱动。

### Lines 594-618: `forward_cuda` implementation / `forward_cuda` 实现
```python
    def forward_cuda(
        self, x: torch.Tensor, shift: torch.Tensor, scale: torch.Tensor
    ) -> torch.Tensor:
        if x.shape[-1] % 256 != 0 and x.shape[-1] <= 8192:
            import warnings

            warnings.warn(
                "FusedNormScaleShift cuda not available, using native fallback",
                stacklevel=2,
            )
            return self.forward_native(x, shift, scale)

        from sglang.jit_kernel.diffusion.cutedsl.scale_residual_norm_scale_shift import (
            fused_norm_scale_shift,
        )

        return fused_norm_scale_shift(
            x.contiguous(),
            _ensure_contiguous(getattr(self.norm, "weight", None)),
            _ensure_contiguous(getattr(self.norm, "bias", None)),
            scale.contiguous(),
            shift.contiguous(),
            self.norm_type,
            self.eps,
        )
```
**EN:** This block defines method `forward_cuda` on `_NormScaleShift`. It executes cuda. Key calls include `fused_norm_scale_shift`, `warnings.warn`, `self.forward_native`, `x.contiguous`, and `_ensure_contiguous`. The implementation branches on conditions. Parameters such as `x`, `shift`, and `scale` drive the behavior in this section.
**CN:** 该代码块定义了 `_NormScaleShift` 的方法 `forward_cuda`。 它用于执行前向计算cuda。 关键调用包括 `fused_norm_scale_shift`、`warnings.warn`、`self.forward_native`、`x.contiguous` 和 `_ensure_contiguous`。 实现中包含条件分支。 本段逻辑主要由 `x`、`shift` 和 `scale` 等参数驱动。

### Lines 620-623: `forward_hip` implementation / `forward_hip` 实现
```python
    def forward_hip(self, *args, **kwargs):
        # ROCm does not support CUDA/CUTLASS-based fused kernels yet,
        # so we fall back to the native PyTorch implementation.
        return self.forward_native(*args, **kwargs)
```
**EN:** This block defines method `forward_hip` on `_NormScaleShift`. It executes hip. Key calls include `self.forward_native`.
**CN:** 该代码块定义了 `_NormScaleShift` 的方法 `forward_hip`。 它用于执行前向计算hip。 关键调用包括 `self.forward_native`。

### Lines 625-628: `forward_musa` implementation / `forward_musa` 实现
```python
    def forward_musa(self, *args, **kwargs):
        # MUSA does not support CUDA/CUTLASS-based fused kernels yet,
        # so we fall back to the native PyTorch implementation.
        return self.forward_native(*args, **kwargs)
```
**EN:** This block defines method `forward_musa` on `_NormScaleShift`. It executes musa. Key calls include `self.forward_native`.
**CN:** 该代码块定义了 `_NormScaleShift` 的方法 `forward_musa`。 它用于执行前向计算musa。 关键调用包括 `self.forward_native`。

### Lines 630-633: `forward_xpu` implementation / `forward_xpu` 实现
```python
    def forward_xpu(self, *args, **kwargs):
        # XPU does not support CUDA/CUTLASS-based fused kernels yet,
        # so we fall back to the native PyTorch implementation.
        return self.forward_native(*args, **kwargs)
```
**EN:** This block defines method `forward_xpu` on `_NormScaleShift`. It executes xpu. Key calls include `self.forward_native`.
**CN:** 该代码块定义了 `_NormScaleShift` 的方法 `forward_xpu`。 它用于执行前向计算xpu。 关键调用包括 `self.forward_native`。

### Lines 635-641: `forward_native` implementation / `forward_native` 实现
```python
    @torch.compile(disable=current_platform.is_npu())
    def forward_native(
        self, x: torch.Tensor, shift: torch.Tensor, scale: torch.Tensor
    ) -> torch.Tensor:
        normalized = self.norm(x)
        modulated = fuse_scale_shift_kernel(normalized, scale, shift)
        return modulated.to(x.dtype)
```
**EN:** This block defines method `forward_native` on `_NormScaleShift`. It executes native. Key calls include `torch.compile`, `self.norm`, `fuse_scale_shift_kernel`, `modulated.to`, and `current_platform.is_npu`. Parameters such as `x`, `shift`, and `scale` drive the behavior in this section.
**CN:** 该代码块定义了 `_NormScaleShift` 的方法 `forward_native`。 它用于执行前向计算native。 关键调用包括 `torch.compile`、`self.norm`、`fuse_scale_shift_kernel`、`modulated.to` 和 `current_platform.is_npu`。 本段逻辑主要由 `x`、`shift` 和 `scale` 等参数驱动。

### Lines 643-650: `forward_npu` implementation / `forward_npu` 实现
```python
    def forward_npu(
        self, x: torch.Tensor, shift: torch.Tensor, scale: torch.Tensor
    ) -> torch.Tensor:
        from sgl_kernel_npu.norm.scale_shift import fused_scale_shift

        normalized = self.norm(x)
        modulated = fused_scale_shift(normalized, scale, shift)
        return modulated.to(x.dtype)
```
**EN:** This block defines method `forward_npu` on `_NormScaleShift`. It executes npu. Key calls include `self.norm`, `fused_scale_shift`, and `modulated.to`. Parameters such as `x`, `shift`, and `scale` drive the behavior in this section.
**CN:** 该代码块定义了 `_NormScaleShift` 的方法 `forward_npu`。 它用于执行前向计算npu。 关键调用包括 `self.norm`、`fused_scale_shift` 和 `modulated.to`。 本段逻辑主要由 `x`、`shift` 和 `scale` 等参数驱动。

### Lines 653-653: `LayerNormScaleShift` class overview / `LayerNormScaleShift` 类概览
```python
class LayerNormScaleShift(_NormScaleShift):
```
**EN:** This block defines class `LayerNormScaleShift`. It encapsulates layer norm scale shift behavior. It inherits from `_NormScaleShift`.
**CN:** 该代码块定义了类 `LayerNormScaleShift`。 它用于封装 layer norm scale shift 相关行为。 它继承自 `_NormScaleShift`。

### Lines 654-654: supporting statements / 辅助语句
```python
    norm_type = "layer"
```
**EN:** This block gathers supporting statements inside `LayerNormScaleShift`. It updates names such as `norm_type`.
**CN:** 该代码块汇集了位于 `LayerNormScaleShift` 内部的辅助语句。 它会更新 `norm_type` 等名称。

### Lines 657-657: `RMSNormScaleShift` class overview / `RMSNormScaleShift` 类概览
```python
class RMSNormScaleShift(_NormScaleShift):
```
**EN:** This block defines class `RMSNormScaleShift`. It encapsulates rmsnorm scale shift behavior. It inherits from `_NormScaleShift`.
**CN:** 该代码块定义了类 `RMSNormScaleShift`。 它用于封装 rmsnorm scale shift 相关行为。 它继承自 `_NormScaleShift`。

### Lines 658-658: supporting statements / 辅助语句
```python
    norm_type = "rms"
```
**EN:** This block gathers supporting statements inside `RMSNormScaleShift`. It updates names such as `norm_type`.
**CN:** 该代码块汇集了位于 `RMSNormScaleShift` 内部的辅助语句。 它会更新 `norm_type` 等名称。

### Lines 666-666: `_NormTanhMulAdd` class overview / `_NormTanhMulAdd` 类概览
```python
class _NormTanhMulAdd(CustomOp):
```
**EN:** This block defines class `_NormTanhMulAdd`. It encapsulates norm tanh mul add behavior. It inherits from `CustomOp`.
**CN:** 该代码块定义了类 `_NormTanhMulAdd`。 它用于封装 norm tanh mul add 相关行为。 它继承自 `CustomOp`。

### Lines 667-667: supporting statements / 辅助语句
```python
    norm_type: str
```
**EN:** This block gathers supporting statements inside `_NormTanhMulAdd`. It updates names such as `norm_type`.
**CN:** 该代码块汇集了位于 `_NormTanhMulAdd` 内部的辅助语句。 它会更新 `norm_type` 等名称。

### Lines 669-685: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        hidden_size: int,
        eps: float = 1e-6,
        affine: bool = False,
        dtype: torch.dtype = torch.float32,
    ):
        super().__init__()
        self.eps = eps
        if self.norm_type == "rms":
            self.norm = RMSNorm(hidden_size, eps=eps, dtype=dtype)
        elif self.norm_type == "layer":
            self.norm = FP32LayerNorm(
                hidden_size, elementwise_affine=affine, eps=eps, dtype=dtype
            )
        else:
            raise NotImplementedError(f"Norm type {self.norm_type} not implemented")
```
**EN:** This block defines method `__init__` on `_NormTanhMulAdd`. It initializes the instance state. Key calls include `super.__init__`, `RMSNorm`, `super`, `FP32LayerNorm`, and `NotImplementedError`. The implementation branches on conditions. Parameters such as `hidden_size`, `eps`, `affine`, and `dtype` drive the behavior in this section.
**CN:** 该代码块定义了 `_NormTanhMulAdd` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`RMSNorm`、`super`、`FP32LayerNorm` 和 `NotImplementedError`。 实现中包含条件分支。 本段逻辑主要由 `hidden_size`、`eps`、`affine` 和 `dtype` 等参数驱动。

### Lines 687-714: `forward_cuda` implementation / `forward_cuda` 实现
```python
    def forward_cuda(
        self, x: torch.Tensor, scale: torch.Tensor, shift: torch.Tensor
    ) -> torch.Tensor:
        if x.shape[-1] % 256 != 0 and x.shape[-1] <= 8192:
            import warnings

            warnings.warn(
                "FusedNormScaleShift cuda not available, using native fallback",
                stacklevel=2,
            )
            return self.forward_native(x, scale, shift)

        from sglang.jit_kernel.diffusion.cutedsl.norm_tanh_mul_add_norm_scale import (
            fused_norm_tanh_mul_add,
        )

        x, scale, shift = x.contiguous(), scale.contiguous(), shift.contiguous()
        weight = _ensure_contiguous(getattr(self.norm, "weight", None))
        bias = _ensure_contiguous(getattr(self.norm, "bias", None))
        return fused_norm_tanh_mul_add(
            x,
            weight,
            bias,
            scale,
            shift,
            self.norm_type,
            self.eps,
        )
```
**EN:** This block defines method `forward_cuda` on `_NormTanhMulAdd`. It executes cuda. Key calls include `_ensure_contiguous`, `fused_norm_tanh_mul_add`, `warnings.warn`, `self.forward_native`, and `x.contiguous`. The implementation branches on conditions. Parameters such as `x`, `scale`, and `shift` drive the behavior in this section.
**CN:** 该代码块定义了 `_NormTanhMulAdd` 的方法 `forward_cuda`。 它用于执行前向计算cuda。 关键调用包括 `_ensure_contiguous`、`fused_norm_tanh_mul_add`、`warnings.warn`、`self.forward_native` 和 `x.contiguous`。 实现中包含条件分支。 本段逻辑主要由 `x`、`scale` 和 `shift` 等参数驱动。

### Lines 716-718: `forward_hip` implementation / `forward_hip` 实现
```python
    def forward_hip(self, *args, **kwargs):
        # Fallback to native because ROCm does not support CuTeDSL.
        return self.forward_native(*args, **kwargs)
```
**EN:** This block defines method `forward_hip` on `_NormTanhMulAdd`. It executes hip. Key calls include `self.forward_native`.
**CN:** 该代码块定义了 `_NormTanhMulAdd` 的方法 `forward_hip`。 它用于执行前向计算hip。 关键调用包括 `self.forward_native`。

### Lines 720-725: `forward_native` implementation / `forward_native` 实现
```python
    @torch.compile(disable=current_platform.is_npu())
    def forward_native(
        self, x: torch.Tensor, scale: torch.Tensor, shift: torch.Tensor
    ) -> torch.Tensor:
        y = self.norm(x) * torch.tanh(scale) + shift
        return y.to(x.dtype)
```
**EN:** This block defines method `forward_native` on `_NormTanhMulAdd`. It executes native. Key calls include `torch.compile`, `y.to`, `current_platform.is_npu`, `self.norm`, and `torch.tanh`. Parameters such as `x`, `scale`, and `shift` drive the behavior in this section.
**CN:** 该代码块定义了 `_NormTanhMulAdd` 的方法 `forward_native`。 它用于执行前向计算native。 关键调用包括 `torch.compile`、`y.to`、`current_platform.is_npu`、`self.norm` 和 `torch.tanh`。 本段逻辑主要由 `x`、`scale` 和 `shift` 等参数驱动。

### Lines 728-728: `LayerNormTanhMulAdd` class overview / `LayerNormTanhMulAdd` 类概览
```python
class LayerNormTanhMulAdd(_NormTanhMulAdd):
```
**EN:** This block defines class `LayerNormTanhMulAdd`. It encapsulates layer norm tanh mul add behavior. It inherits from `_NormTanhMulAdd`.
**CN:** 该代码块定义了类 `LayerNormTanhMulAdd`。 它用于封装 layer norm tanh mul add 相关行为。 它继承自 `_NormTanhMulAdd`。

### Lines 729-729: supporting statements / 辅助语句
```python
    norm_type = "layer"
```
**EN:** This block gathers supporting statements inside `LayerNormTanhMulAdd`. It updates names such as `norm_type`.
**CN:** 该代码块汇集了位于 `LayerNormTanhMulAdd` 内部的辅助语句。 它会更新 `norm_type` 等名称。

### Lines 732-732: `RMSNormTanhMulAdd` class overview / `RMSNormTanhMulAdd` 类概览
```python
class RMSNormTanhMulAdd(_NormTanhMulAdd):
```
**EN:** This block defines class `RMSNormTanhMulAdd`. It encapsulates rmsnorm tanh mul add behavior. It inherits from `_NormTanhMulAdd`.
**CN:** 该代码块定义了类 `RMSNormTanhMulAdd`。 它用于封装 rmsnorm tanh mul add 相关行为。 它继承自 `_NormTanhMulAdd`。

### Lines 733-733: supporting statements / 辅助语句
```python
    norm_type = "rms"
```
**EN:** This block gathers supporting statements inside `RMSNormTanhMulAdd`. It updates names such as `norm_type`.
**CN:** 该代码块汇集了位于 `RMSNormTanhMulAdd` 内部的辅助语句。 它会更新 `norm_type` 等名称。

### Lines 736-776: `apply_qk_norm` implementation / `apply_qk_norm` 实现
```python
def apply_qk_norm(
    q: torch.Tensor,
    k: torch.Tensor,
    q_norm: "RMSNorm",
    k_norm: "RMSNorm",
    head_dim: int,
    allow_inplace: bool = True,
) -> Tuple[torch.Tensor, torch.Tensor]:
    """Apply QK normalization for query and key tensors.

    Uses JIT fused inplace kernel when available, falls back to standard RMSNorm.
    """

    batch_size = q.size(0)
    q_eps = q_norm.variance_epsilon
    k_eps = k_norm.variance_epsilon
    # Only try fused path on CUDA and when it won't introduce implicit copies.
    if (
        _is_cuda
        and allow_inplace
        and (q_eps == k_eps)
        and q.dtype in (torch.float16, torch.bfloat16)
        and q_norm.weight.dtype == q.dtype
        and k_norm.weight.dtype == k.dtype
        and can_use_fused_inplace_qknorm(head_dim, q.dtype)
    ):
        fused_inplace_qknorm(
            q=q.view(batch_size, -1, head_dim),
            k=k.view(batch_size, -1, head_dim),
            q_weight=q_norm.weight,
            k_weight=k_norm.weight,
            head_dim=head_dim,
            eps=q_eps,
        )
        return q, k

    q_shape = q.shape
    k_shape = k.shape
    q_out = q_norm(q.view(-1, head_dim)).view(q_shape)
    k_out = k_norm(k.view(-1, head_dim)).view(k_shape)
    return q_out, k_out
```
**EN:** This block defines function `apply_qk_norm`. Apply QK normalization for query and key tensors. Uses JIT fused inplace kernel when available, falls back to standard RMSNorm. Key calls include `q.size`, `q_norm.view`, `k_norm.view`, `can_use_fused_inplace_qknorm`, and `fused_inplace_qknorm`. The implementation branches on conditions. Parameters such as `q`, `k`, `q_norm`, `k_norm`, and `head_dim` drive the behavior in this section.
**CN:** 该代码块定义了函数 `apply_qk_norm`。 它用于应用qk norm。 关键调用包括 `q.size`、`q_norm.view`、`k_norm.view`、`can_use_fused_inplace_qknorm` 和 `fused_inplace_qknorm`。 实现中包含条件分支。 本段逻辑主要由 `q`、`k`、`q_norm`、`k_norm` 和 `head_dim` 等参数驱动。

### Lines 779-815: `apply_qk_norm_with_optional_rope` implementation / `apply_qk_norm_with_optional_rope` 实现
```python
def apply_qk_norm_with_optional_rope(
    q: torch.Tensor,
    k: torch.Tensor,
    q_norm: "RMSNorm",
    k_norm: "RMSNorm",
    head_dim: int,
    cos_sin_cache: Optional[torch.Tensor] = None,
    *,
    is_neox: bool = False,
    positions: Optional[torch.Tensor] = None,
    position_offset: int = 0,
    allow_inplace: bool = True,
) -> Tuple[torch.Tensor, torch.Tensor]:
    """Apply QK RMSNorm and optionally RoPE when a cos/sin cache is provided."""

    if cos_sin_cache is None:
        return apply_qk_norm(
            q=q,
            k=k,
            q_norm=q_norm,
            k_norm=k_norm,
            head_dim=head_dim,
            allow_inplace=allow_inplace,
        )

    return apply_qk_norm_rope(
        q=q,
        k=k,
        q_norm=q_norm,
        k_norm=k_norm,
        head_dim=head_dim,
        cos_sin_cache=cos_sin_cache,
        is_neox=is_neox,
        positions=positions,
        position_offset=position_offset,
        allow_inplace=allow_inplace,
    )
```
**EN:** This block defines function `apply_qk_norm_with_optional_rope`. Apply QK RMSNorm and optionally RoPE when a cos/sin cache is provided. Key calls include `apply_qk_norm_rope`, and `apply_qk_norm`. The implementation branches on conditions. Parameters such as `q`, `k`, `q_norm`, `k_norm`, and `head_dim` drive the behavior in this section.
**CN:** 该代码块定义了函数 `apply_qk_norm_with_optional_rope`。 它用于应用qk norm with optional rope。 关键调用包括 `apply_qk_norm_rope` 和 `apply_qk_norm`。 实现中包含条件分支。 本段逻辑主要由 `q`、`k`、`q_norm`、`k_norm` 和 `head_dim` 等参数驱动。

### Lines 818-912: `apply_qk_norm_rope` implementation / `apply_qk_norm_rope` 实现
```python
def apply_qk_norm_rope(
    q: torch.Tensor,
    k: torch.Tensor,
    q_norm: "RMSNorm",
    k_norm: "RMSNorm",
    head_dim: int,
    cos_sin_cache: torch.Tensor,
    *,
    is_neox: bool = False,
    positions: Optional[torch.Tensor] = None,
    position_offset: int = 0,
    allow_inplace: bool = True,
) -> Tuple[torch.Tensor, torch.Tensor]:
    """Apply QK RMSNorm followed by RoPE, fusing both on supported CUDA shapes."""

    from sglang.multimodal_gen.runtime.layers.rotary_embedding import (
        apply_flashinfer_rope_qk_inplace,
    )

    if q.dim() != 4 or k.dim() != 4:
        raise ValueError(
            f"apply_qk_norm_rope expects 4D q/k tensors, got q:{tuple(q.shape)} k:{tuple(k.shape)}"
        )
    if q.shape != k.shape:
        raise ValueError(
            f"apply_qk_norm_rope expects q/k to have the same shape, got {q.shape} vs {k.shape}"
        )

    batch_size, seq_len, _, _ = q.shape
    q_eps = q_norm.variance_epsilon
    k_eps = k_norm.variance_epsilon
    rope_dim = cos_sin_cache.size(-1)
    fused_enabled = os.getenv("SGLANG_ENABLE_FUSED_QKNORM_ROPE", "1").lower() not in {
        "0",
        "false",
        "off",
        "no",
    }

    if positions is None:
        pos_1d = torch.arange(
            position_offset,
            position_offset + seq_len,
            device=q.device,
            dtype=torch.int64,
        )
        positions = pos_1d if batch_size == 1 else pos_1d.repeat(batch_size)
    else:
        if positions.dim() != 1 or positions.numel() != batch_size * seq_len:
            raise ValueError(
                f"positions must be 1D of length {batch_size * seq_len}, got shape={tuple(positions.shape)}"
            )

    if (
        fused_enabled
        and _is_cuda
        and allow_inplace
        and (q_eps == k_eps)
        and q.dtype in (torch.float16, torch.bfloat16)
        and q_norm.weight.dtype == q.dtype
        and k_norm.weight.dtype == k.dtype
        and q.is_contiguous()
        and k.is_contiguous()
        and can_use_fused_inplace_qknorm_rope(head_dim, rope_dim, is_neox, q.dtype)
    ):
        fused_inplace_qknorm_rope(
            q=q.reshape(-1, q.shape[-2], head_dim),
            k=k.reshape(-1, k.shape[-2], head_dim),
            q_weight=q_norm.weight,
            k_weight=k_norm.weight,
            cos_sin_cache=cos_sin_cache,
            positions=positions,
            is_neox=is_neox,
            eps=q_eps,
            head_dim=head_dim,
            rope_dim=rope_dim,
        )
        return q, k

    q, k = apply_qk_norm(
        q=q,
        k=k,
        q_norm=q_norm,
        k_norm=k_norm,
        head_dim=head_dim,
        allow_inplace=allow_inplace,
    )
    return apply_flashinfer_rope_qk_inplace(
        q=q,
        k=k,
        cos_sin_cache=cos_sin_cache,
        head_size=head_dim,
        is_neox=is_neox,
        positions=positions,
    )
```
**EN:** This block defines function `apply_qk_norm_rope`. Apply QK RMSNorm followed by RoPE, fusing both on supported CUDA shapes. Key calls include `cos_sin_cache.size`, `apply_qk_norm`, `apply_flashinfer_rope_qk_inplace`, `ValueError`, and `os.getenv.lower`. The implementation branches on conditions. Parameters such as `q`, `k`, `q_norm`, `k_norm`, and `head_dim` drive the behavior in this section.
**CN:** 该代码块定义了函数 `apply_qk_norm_rope`。 它用于应用qk norm rope。 关键调用包括 `cos_sin_cache.size`、`apply_qk_norm`、`apply_flashinfer_rope_qk_inplace`、`ValueError` 和 `os.getenv.lower`。 实现中包含条件分支。 本段逻辑主要由 `q`、`k`、`q_norm`、`k_norm` 和 `head_dim` 等参数驱动。

### Lines 915-940: `apply_rmsnorm_tanh_mul_add` implementation / `apply_rmsnorm_tanh_mul_add` 实现
```python
def apply_rmsnorm_tanh_mul_add(
    x: torch.Tensor,
    gate: torch.Tensor,
    residual: torch.Tensor,
    norm: "RMSNorm",
) -> torch.Tensor:
    """Compute residual + tanh(gate) * rmsnorm(x), with a fused CUDA fast path."""
    if get_bool_env_var("SGLANG_ENABLE_DETERMINISTIC_INFERENCE"):
        return residual + torch.tanh(gate) * norm(x)

    if _is_cuda and x.is_cuda and x.shape[-1] % 256 == 0 and x.shape[-1] <= 8192:
        from sglang.jit_kernel.diffusion.cutedsl.norm_tanh_mul_add_norm_scale import (
            fused_norm_tanh_mul_add,
        )

        return fused_norm_tanh_mul_add(
            x.contiguous(),
            norm.weight.data.contiguous(),
            None,
            gate.contiguous(),
            residual.contiguous(),
            "rms",
            norm.variance_epsilon,
        )

    return residual + torch.tanh(gate) * norm(x)
```
**EN:** This block defines function `apply_rmsnorm_tanh_mul_add`. Compute residual + tanh(gate) * rmsnorm(x), with a fused CUDA fast path. Key calls include `get_bool_env_var`, `fused_norm_tanh_mul_add`, `x.contiguous`, `norm.weight.data.contiguous`, and `gate.contiguous`. The implementation branches on conditions. Parameters such as `x`, `gate`, `residual`, and `norm` drive the behavior in this section.
**CN:** 该代码块定义了函数 `apply_rmsnorm_tanh_mul_add`。 它用于应用rmsnorm tanh mul add。 关键调用包括 `get_bool_env_var`、`fused_norm_tanh_mul_add`、`x.contiguous`、`norm.weight.data.contiguous` 和 `gate.contiguous`。 实现中包含条件分支。 本段逻辑主要由 `x`、`gate`、`residual` 和 `norm` 等参数驱动。

### Lines 943-964: `tensor_parallel_rms_norm` implementation / `tensor_parallel_rms_norm` 实现
```python
def tensor_parallel_rms_norm(x: torch.Tensor, norm: "RMSNorm") -> torch.Tensor:
    tp_rank = get_tensor_model_parallel_rank()
    tp_size = get_tensor_model_parallel_world_size()
    src_dtype = x.dtype
    weight = norm.weight.tensor_split(tp_size)[tp_rank].float()
    x_fp32 = x.float()
    if _is_npu:
        from sgl_kernel_npu.norm.rmsnorm_split import fused_rsqrt_mul, fused_variance

        variance = fused_variance(x_fp32)
    else:
        variance = x_fp32.pow(2).mean(dim=-1, keepdim=True)

    variance = get_tp_group().all_reduce(
        variance, op=torch._C._distributed_c10d.ReduceOp.AVG
    )

    if _is_npu:
        output = fused_rsqrt_mul(x_fp32, variance, weight, norm.variance_epsilon)
    else:
        output = x_fp32 * torch.rsqrt(variance + norm.variance_epsilon) * weight
    return output.to(dtype=src_dtype)
```
**EN:** This block defines function `tensor_parallel_rms_norm`. It handles tensor parallel rms norm logic. Key calls include `get_tensor_model_parallel_rank`, `get_tensor_model_parallel_world_size`, `norm.weight.tensor_split.float`, `x.float`, and `get_tp_group.all_reduce`. The implementation branches on conditions. Parameters such as `x`, and `norm` drive the behavior in this section.
**CN:** 该代码块定义了函数 `tensor_parallel_rms_norm`。 它用于处理 tensor parallel rms norm 相关逻辑。 关键调用包括 `get_tensor_model_parallel_rank`、`get_tensor_model_parallel_world_size`、`norm.weight.tensor_split.float`、`x.float` 和 `get_tp_group.all_reduce`。 实现中包含条件分支。 本段逻辑主要由 `x` 和 `norm` 等参数驱动。

## Key Concepts / 关键概念
- `RMSNorm`: Root mean square normalization. / 核心类，用于封装 rmsnorm 相关行为。
- `LayerNorm`: Primary class that encapsulates layer norm behavior. / 核心类，用于封装 layer norm 相关行为。
- `FP32LayerNorm`: Primary class that encapsulates fp32 layer norm behavior. / 核心类，用于封装 fp32 layer norm 相关行为。
- `_ensure_contiguous`: Top-level function that handles ensure contiguous logic. / 顶层函数，用于处理 ensure contiguous 相关逻辑。
- `_ScaleResidualNormScaleShift`: Fused kernel that combines: 1. / 核心类，用于封装 scale residual norm scale shift 相关行为。
- `ScaleResidualLayerNormScaleShift`: Primary class that encapsulates scale residual layer norm scale shift behavior. / 核心类，用于封装 scale residual layer norm scale shift 相关行为。
- `ScaleResidualRMSNormScaleShift`: Primary class that encapsulates scale residual rmsnorm scale shift behavior. / 核心类，用于封装 scale residual rmsnorm scale shift 相关行为。
- `_NormScaleShift`: Fused kernel that combines: 1. / 核心类，用于封装 norm scale shift 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `typing`, `warnings`
- **Third-party / 第三方依赖**: `torch`, `torch.nn`, `torch.nn.functional`, `sgl_kernel`, `torch_npu`, `aiter`, `sgl_kernel_npu.norm.scale_shift`, `sgl_kernel_npu.norm.rmsnorm_split`
- **Internal modules / 内部模块**: `sglang.jit_kernel.diffusion.qknorm_rope`, `sglang.jit_kernel.diffusion.triton.rmsnorm_onepass`, `sglang.jit_kernel.diffusion.triton.scale_shift`, `sglang.jit_kernel.norm`, `sglang.multimodal_gen.runtime.distributed.parallel_state`, `sglang.multimodal_gen.runtime.layers.custom_op`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.utils.common`, `sglang.jit_kernel.diffusion.triton.norm`, `sglang.multimodal_gen.runtime.layers.rotary_embedding`, `sglang.jit_kernel.diffusion.cutedsl.scale_residual_norm_scale_shift`, `sglang.jit_kernel.diffusion.cutedsl.norm_tanh_mul_add_norm_scale`

- **Total lines / 总行数**: 964
