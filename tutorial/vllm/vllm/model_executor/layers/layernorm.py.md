# layernorm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/layernorm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Custom normalization layers. / 归一化层与融合归一化内核

## Line-by-Line Analysis / 逐行分析
### Lines 5-17 — imports and setup
```python
import torch
import torch.nn as nn
import torch.nn.functional as F

# Import kernels
import vllm.kernels  # noqa: F401
from vllm import envs, ir
from vllm.config import get_current_vllm_config
from vllm.logger import init_logger
from vllm.model_executor.custom_op import CustomOp
from vllm.model_executor.layers.batch_invariant import rms_norm_batch_invariant

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。

### Lines 20-33 — function `poly_norm`
```python
def poly_norm(
    x: torch.Tensor, weight: torch.Tensor, bias: torch.Tensor, variance_epsilon: float
) -> torch.Tensor:
    from vllm import _custom_ops as ops

    out = torch.empty_like(x)
    ops.poly_norm(  # type: ignore[attr-defined]
        out,
        x,
        weight,
        bias,
        variance_epsilon,
    )
    return out
```
**EN:** This function defines `poly_norm`. It provides one of the file's main runtime building blocks. The main inputs are `x`, `weight`, `bias`, `variance_epsilon`. Key calls include `torch.empty_like`, `ops.poly_norm`. It writes or updates `out`.
**CN:** 该函数定义 `poly_norm`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `x`, `weight`, `bias`, `variance_epsilon`。 关键调用包括 `torch.empty_like`, `ops.poly_norm`。 它会写入或更新 `out`。

### Lines 37-128 — class `RMSNorm`
```python
@CustomOp.register("rms_norm")
class RMSNorm(CustomOp):
    """Root mean square normalization.

    Computes x -> w * x / sqrt(E[x^2] + eps) where w is the learned weight.
    Refer to https://arxiv.org/abs/1910.07467
    """

    # --8<-- [end:rms_norm]

    def __init__(
        self,
        hidden_size: int,
        eps: float = 1e-6,
# ... omitted for brevity ...
        s += f", eps={self.variance_epsilon}"
        return s
```
**EN:** This class defines `RMSNorm`. It inherits from `CustomOp`. Root mean square normalization. Important methods include `__init__`, `forward_native`, `forward_cuda`, `forward_xpu`, `extra_repr`. Key calls include `CustomOp.register`, `super.__init__`, `torch.ones`, `self.forward_native`, `self.forward_cuda`, `torch.get_default_dtype`. It writes or updates `hidden_size`, `variance_epsilon`, `variance_size_override`, `weight_dtype`, `has_weight`, `weight`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `RMSNorm`。 它继承自 `CustomOp`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `forward_native`, `forward_cuda`, `forward_xpu`, `extra_repr`。 关键调用包括 `CustomOp.register`, `super.__init__`, `torch.ones`, `self.forward_native`, `self.forward_cuda`, `torch.get_default_dtype`。 它会写入或更新 `hidden_size`, `variance_epsilon`, `variance_size_override`, `weight_dtype`, `has_weight`, `weight`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 47-80 — method `RMSNorm.__init__`
```python
    def __init__(
        self,
        hidden_size: int,
        eps: float = 1e-6,
        var_hidden_size: int | None = None,
        has_weight: bool = True,
        dtype: torch.dtype | None = None,
    ) -> None:
        super().__init__()

        self.hidden_size = hidden_size
        self.variance_epsilon = eps
        self.variance_size_override = (
            None if var_hidden_size == hidden_size else var_hidden_size
        )
        weight_dtype = dtype or torch.get_default_dtype()
        self.has_weight = has_weight
        self.weight = torch.ones(hidden_size, dtype=weight_dtype)
        if self.has_weight:
            self.weight = nn.Parameter(self.weight)

        # Do not pass identity weight to native implementation (causes issue on TPU).
        # Other implementations require weight to be passed even if all ones.
        # Cheat and predict if native will be dispatched to:
        #  1) if native is first in priority list
        #  2) if variance_size_override is given (only supported by native impl)
        # TODO(luka): address weight passing inconsistency:
        # https://github.com/vllm-project/vllm/issues/39370
        priority = get_current_vllm_config().kernel_config.ir_op_priority
        var_override = self.variance_size_override is not None
        native_rms_norm = priority.rms_norm[0] == "native" or var_override
        native_add_rms_norm = priority.fused_add_rms_norm[0] == "native" or var_override
        self.pass_weight = self.has_weight or not native_rms_norm
        self.pass_weight_add = self.has_weight or not native_add_rms_norm
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `hidden_size`, `eps`, `var_hidden_size`, `has_weight`, `dtype`. Key calls include `super.__init__`, `torch.ones`, `torch.get_default_dtype`, `nn.Parameter`, `super`, `get_current_vllm_config`. It writes or updates `hidden_size`, `variance_epsilon`, `variance_size_override`, `weight_dtype`, `has_weight`, `weight`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `hidden_size`, `eps`, `var_hidden_size`, `has_weight`, `dtype`。 关键调用包括 `super.__init__`, `torch.ones`, `torch.get_default_dtype`, `nn.Parameter`, `super`, `get_current_vllm_config`。 它会写入或更新 `hidden_size`, `variance_epsilon`, `variance_size_override`, `weight_dtype`, `has_weight`, `weight`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 82-102 — method `RMSNorm.forward_native`
```python
    def forward_native(
        self,
        x: torch.Tensor,
        residual: torch.Tensor | None = None,
    ) -> torch.Tensor | tuple[torch.Tensor, torch.Tensor]:
        """PyTorch-native implementation equivalent to forward()."""
        if residual is None:
            return ir.ops.rms_norm(
                x,
                self.weight.data if self.pass_weight else None,
                self.variance_epsilon,
                self.variance_size_override,
            )
        else:
            return ir.ops.fused_add_rms_norm.maybe_inplace(
                x,
                residual,
                self.weight.data if self.pass_weight_add else None,
                self.variance_epsilon,
                self.variance_size_override,
            )
```
**EN:** This method defines `forward_native`. PyTorch-native implementation equivalent to forward(). The main inputs are `x`, `residual`. Key calls include `ir.ops.rms_norm`, `ir.ops.fused_add_rms_norm.maybe_inplace`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `forward_native`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `x`, `residual`。 关键调用包括 `ir.ops.rms_norm`, `ir.ops.fused_add_rms_norm.maybe_inplace`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 104-116 — method `RMSNorm.forward_cuda`
```python
    def forward_cuda(
        self,
        x: torch.Tensor,
        residual: torch.Tensor | None = None,
    ) -> torch.Tensor | tuple[torch.Tensor, torch.Tensor]:
        if (
            envs.VLLM_BATCH_INVARIANT
            and residual is None
            and self.variance_size_override is None
        ):
            return rms_norm_batch_invariant(x, self.weight.data, self.variance_epsilon)

        return self.forward_native(x, residual)
```
**EN:** This method defines `forward_cuda`. It executes the main forward/runtime path for this component. The main inputs are `x`, `residual`. Key calls include `self.forward_native`, `rms_norm_batch_invariant`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `forward_cuda`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `x`, `residual`。 关键调用包括 `self.forward_native`, `rms_norm_batch_invariant`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 118-123 — method `RMSNorm.forward_xpu`
```python
    def forward_xpu(
        self,
        x: torch.Tensor,
        residual: torch.Tensor | None = None,
    ) -> torch.Tensor | tuple[torch.Tensor, torch.Tensor]:
        return self.forward_cuda(x, residual)
```
**EN:** This method defines `forward_xpu`. It executes the main forward/runtime path for this component. The main inputs are `x`, `residual`. Key calls include `self.forward_cuda`.
**CN:** 该方法定义 `forward_xpu`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `x`, `residual`。 关键调用包括 `self.forward_cuda`。

### Lines 125-128 — method `RMSNorm.extra_repr`
```python
    def extra_repr(self) -> str:
        s = f"hidden_size={self.weight.data.size(0)}"
        s += f", eps={self.variance_epsilon}"
        return s
```
**EN:** This method defines `extra_repr`. It provides one of the file's main runtime building blocks. Key calls include `self.weight.data.size`. It writes or updates `s`.
**CN:** 该方法定义 `extra_repr`。 它是该文件中的一个主要运行时构件。 关键调用包括 `self.weight.data.size`。 它会写入或更新 `s`。

### Lines 132-178 — class `GemmaRMSNorm`
```python
@CustomOp.register("gemma_rms_norm")
class GemmaRMSNorm(CustomOp):
    """RMS normalization for Gemma.

    Two differences from the above RMSNorm:
        1. x * (1 + w) instead of x * w.
        2. (x * w).to(orig_dtype) instead of x.to(orig_dtype) * w.
    """

    # --8<-- [end:gemma_rms_norm]

    def __init__(
        self,
        hidden_size: int,
# ... omitted for brevity ...
    ) -> torch.Tensor | tuple[torch.Tensor, torch.Tensor]:
        return self.forward_native(x, residual)
```
**EN:** This class defines `GemmaRMSNorm`. It inherits from `CustomOp`. RMS normalization for Gemma. Important methods include `__init__`, `forward_native`, `forward_cuda`. Key calls include `CustomOp.register`, `super.__init__`, `nn.Parameter`, `ir.ops.rms_norm`, `self.forward_native`, `torch.zeros`. It writes or updates `weight`, `variance_epsilon`, `orig_dtype`, `out`, `x`, `residual`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `GemmaRMSNorm`。 它继承自 `CustomOp`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `forward_native`, `forward_cuda`。 关键调用包括 `CustomOp.register`, `super.__init__`, `nn.Parameter`, `ir.ops.rms_norm`, `self.forward_native`, `torch.zeros`。 它会写入或更新 `weight`, `variance_epsilon`, `orig_dtype`, `out`, `x`, `residual`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 143-150 — method `GemmaRMSNorm.__init__`
```python
    def __init__(
        self,
        hidden_size: int,
        eps: float = 1e-6,
    ) -> None:
        super().__init__()
        self.weight = nn.Parameter(torch.zeros(hidden_size))
        self.variance_epsilon = eps
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `hidden_size`, `eps`. Key calls include `super.__init__`, `nn.Parameter`, `torch.zeros`, `super`. It writes or updates `weight`, `variance_epsilon`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `hidden_size`, `eps`。 关键调用包括 `super.__init__`, `nn.Parameter`, `torch.zeros`, `super`。 它会写入或更新 `weight`, `variance_epsilon`。

### Lines 152-171 — method `GemmaRMSNorm.forward_native`
```python
    def forward_native(
        self,
        x: torch.Tensor,
        residual: torch.Tensor | None = None,
    ) -> torch.Tensor | tuple[torch.Tensor, torch.Tensor]:
        """PyTorch-native implementation equivalent to forward()."""
        orig_dtype = x.dtype
        weight = self.weight.data.float() + 1.0
        if residual is not None:
            x = (
                x.float() + residual.float()
                if orig_dtype == torch.float16
                else x + residual
            )
            residual = x
        # ir.ops.rms_norm handles fp32 upcast internally
        out = ir.ops.rms_norm(x, weight, self.variance_epsilon)
        return (
            out.to(orig_dtype) if residual is None else (out.to(orig_dtype), residual)
        )
```
**EN:** This method defines `forward_native`. PyTorch-native implementation equivalent to forward(). The main inputs are `x`, `residual`. Key calls include `ir.ops.rms_norm`, `self.weight.data.float`, `out.to`, `x.float`, `residual.float`. It writes or updates `orig_dtype`, `weight`, `out`, `x`, `residual`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `forward_native`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `x`, `residual`。 关键调用包括 `ir.ops.rms_norm`, `self.weight.data.float`, `out.to`, `x.float`, `residual.float`。 它会写入或更新 `orig_dtype`, `weight`, `out`, `x`, `residual`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 173-178 — method `GemmaRMSNorm.forward_cuda`
```python
    def forward_cuda(
        self,
        x: torch.Tensor,
        residual: torch.Tensor | None = None,
    ) -> torch.Tensor | tuple[torch.Tensor, torch.Tensor]:
        return self.forward_native(x, residual)
```
**EN:** This method defines `forward_cuda`. It executes the main forward/runtime path for this component. The main inputs are `x`, `residual`. Key calls include `self.forward_native`.
**CN:** 该方法定义 `forward_cuda`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `x`, `residual`。 关键调用包括 `self.forward_native`。

### Lines 182-316 — class `RMSNormGated`
```python
@CustomOp.register("rms_norm_gated")
class RMSNormGated(CustomOp):
    """RMS Normalization with optional gating.

    This is a native PyTorch implementation that supports:
    - Standard RMS normalization
    - Group RMS normalization
    - Optional gating with SiLU activation
    """

    # --8<-- [end:rms_norm_gated]

    def __init__(
        self,
# ... omitted for brevity ...
    ) -> torch.Tensor:
        return self.forward_cuda(x, z)
```
**EN:** This class defines `RMSNormGated`. It inherits from `CustomOp`. RMS Normalization with optional gating. Important methods include `__init__`, `forward_native`, `forward_cuda`, `forward_xpu`, `reset_parameters`, `forward_static`. Key calls include `CustomOp.register`, `super.__init__`, `nn.Parameter`, `self.register_parameter`, `self.reset_parameters`, `torch.nn.init.ones_`. It writes or updates `factory_kwargs`, `eps`, `activation`, `weight`, `group_size`, `norm_before_gate`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `RMSNormGated`。 它继承自 `CustomOp`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `forward_native`, `forward_cuda`, `forward_xpu`, `reset_parameters`, `forward_static`。 关键调用包括 `CustomOp.register`, `super.__init__`, `nn.Parameter`, `self.register_parameter`, `self.reset_parameters`, `torch.nn.init.ones_`。 它会写入或更新 `factory_kwargs`, `eps`, `activation`, `weight`, `group_size`, `norm_before_gate`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 194-227 — method `RMSNormGated.__init__`
```python
    def __init__(
        self,
        hidden_size: int,
        eps: float = 1e-5,
        group_size: int | None = None,
        norm_before_gate: bool = False,
        device: torch.device | None = None,
        dtype: torch.dtype | None = None,
        activation: str = "swish",
    ):
        """Initialize RMSNormGated.

        Args:
            hidden_size: Size of the hidden dimension
            eps: Epsilon for numerical stability
            group_size: If not None, do GroupNorm with each group
                        having group_size elements.
                        group_size=None is equivalent to group_size=hidden_size
                        (i.e. there's only 1 group).
            norm_before_gate: If True and z is provided: out = norm(x) * silu(z)
                              If False and z is provided: out = norm(x * silu(z))
            device: Device to create parameters on
            dtype: Data type for parameters
            activation: Activation function name for gating
        """
        factory_kwargs = {"device": device, "dtype": dtype}
        super().__init__()
        self.eps = eps
        self.activation = activation
        self.weight = nn.Parameter(torch.empty(hidden_size, **factory_kwargs))
        self.register_parameter("bias", None)
        self.group_size = group_size
        self.norm_before_gate = norm_before_gate
        self.reset_parameters()
```
**EN:** This method defines `__init__`. Initialize RMSNormGated. The main inputs are `hidden_size`, `eps`, `group_size`, `norm_before_gate`, `device`, `dtype`. Key calls include `super.__init__`, `nn.Parameter`, `self.register_parameter`, `self.reset_parameters`, `torch.empty`, `super`. It writes or updates `factory_kwargs`, `eps`, `activation`, `weight`, `group_size`, `norm_before_gate`.
**CN:** 该方法定义 `__init__`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `hidden_size`, `eps`, `group_size`, `norm_before_gate`, `device`, `dtype`。 关键调用包括 `super.__init__`, `nn.Parameter`, `self.register_parameter`, `self.reset_parameters`, `torch.empty`, `super`。 它会写入或更新 `factory_kwargs`, `eps`, `activation`, `weight`, `group_size`, `norm_before_gate`。

## Key Concepts / 关键概念
- [EN] Normalization layers and fused normalization kernels / [CN] 归一化层与融合归一化内核
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Core symbols: `poly_norm`, `RMSNorm`, `GemmaRMSNorm`, `RMSNormGated` / [CN] 核心符号：`poly_norm`, `RMSNorm`, `GemmaRMSNorm`, `RMSNormGated`

## Dependencies / 依赖关系
- **External**: `torch`, `torch.nn`, `torch.nn.functional` / **外部依赖**: `torch`, `torch.nn`, `torch.nn.functional`
- **Internal**: `vllm.kernels`, `vllm`, `vllm.config`, `vllm.logger`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.batch_invariant` / **内部依赖**: `vllm.kernels`, `vllm`, `vllm.config`, `vllm.logger`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.batch_invariant`
