# activation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/activation.py`
- **Repository**: vllm-project/vllm
- **Purpose**: MoE activation function enum and utilities. / 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑；激活门控与融合逐元素计算

## Line-by-Line Analysis / 逐行分析
### Lines 5-8 — imports and setup
```python
from enum import Enum

import torch
import torch.nn.functional as F
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 11-63 — class `MoEActivation`
```python
class MoEActivation(Enum):
    """Activation functions for MoE layers."""

    # Gated activations (gate * activation(up)) expect input of shape [..., 2*d]
    # and produce output of shape [..., d]
    SILU = "silu"
    GELU = "gelu"
    GELU_TANH = "gelu_tanh"
    RELU2 = "relu2"
    SWIGLUOAI = "swigluoai"
    SWIGLUSTEP = "swiglustep"

    # Non-gated activations (no mul with gate) expect input of shape [..., d]
    # and produce output of shape [..., d].
# ... omitted for brevity ...
        valid = [m.value for m in cls]
        raise ValueError(f"Unknown MoE activation: {s!r}. Valid activations: {valid}")
```
**EN:** This class defines `MoEActivation`. It inherits from `Enum`. Activation functions for MoE layers. Important methods include `is_gated`, `custom_op_name`, `without_mul`, `from_str`. Key calls include `_WITHOUT_MUL.get`, `_STR_ALIASES.get`, `ValueError`, `self.value.endswith`. It writes or updates `SILU`, `GELU`, `GELU_TANH`, `RELU2`, `SWIGLUOAI`, `SWIGLUSTEP`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该类定义了 `MoEActivation`。 它继承自 `Enum`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `is_gated`, `custom_op_name`, `without_mul`, `from_str`。 关键调用包括 `_WITHOUT_MUL.get`, `_STR_ALIASES.get`, `ValueError`, `self.value.endswith`。 它会写入或更新 `SILU`, `GELU`, `GELU_TANH`, `RELU2`, `SWIGLUOAI`, `SWIGLUSTEP`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 31-38 — method `MoEActivation.is_gated`
```python
    @property
    def is_gated(self) -> bool:
        """Returns True if activation expects gate*activation(up) pattern.

        Gated activations expect input tensor with 2x the output size,
        where the first half is the gate and second half is the up projection.
        """
        return not self.value.endswith("_no_mul")
```
**EN:** This method defines `is_gated`. Returns True if activation expects gate*activation(up) pattern. Key calls include `self.value.endswith`.
**CN:** 该方法定义 `is_gated`。 该函数/方法的文档字符串直接说明了它的职责。 关键调用包括 `self.value.endswith`。

### Lines 40-44 — method `MoEActivation.custom_op_name`
```python
    @property
    def custom_op_name(self) -> str:
        """Maps to the CustomOp name of activations
        in vllm/model_executor/layers/activation.py."""
        return _CUSTOM_OP_NAMES[self]
```
**EN:** This method defines `custom_op_name`. Maps to the CustomOp name of activations in vllm/model_executor/layers/activation.py.
**CN:** 该方法定义 `custom_op_name`。 该函数/方法的文档字符串直接说明了它的职责。

### Lines 46-53 — method `MoEActivation.without_mul`
```python
    def without_mul(self) -> "MoEActivation":
        """Get the non-gated variant of this activation.

        For activations that have a _no_mul variant, returns that variant.
        For activations without a _no_mul variant (or already _no_mul),
        returns self.
        """
        return _WITHOUT_MUL.get(self, self)
```
**EN:** This method defines `without_mul`. Get the non-gated variant of this activation. Key calls include `_WITHOUT_MUL.get`.
**CN:** 该方法定义 `without_mul`。 该函数/方法的文档字符串直接说明了它的职责。 关键调用包括 `_WITHOUT_MUL.get`。

### Lines 55-63 — method `MoEActivation.from_str`
```python
    @classmethod
    def from_str(cls, s: str) -> "MoEActivation":
        """Parse from string for backward compatibility."""
        s = _STR_ALIASES.get(s, s)
        for member in cls:
            if member.value == s:
                return member
        valid = [m.value for m in cls]
        raise ValueError(f"Unknown MoE activation: {s!r}. Valid activations: {valid}")
```
**EN:** This method defines `from_str`. Parse from string for backward compatibility. The main inputs are `s`. Key calls include `_STR_ALIASES.get`, `ValueError`. It writes or updates `s`, `valid`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该方法定义 `from_str`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `s`。 关键调用包括 `_STR_ALIASES.get`, `ValueError`。 它会写入或更新 `s`, `valid`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 67-69 — constant `_STR_ALIASES`
```python
_STR_ALIASES: dict[str, str] = {
    "gelu_pytorch_tanh": "gelu_tanh",
}
```
**EN:** This constant defines `_STR_ALIASES`. This block defines module-level constants or shared state used later in execution.
**CN:** 该模块级常量/状态定义 `_STR_ALIASES`。 该代码块定义了后续执行会复用的模块级常量或共享状态。

### Lines 71-82 — constant `_CUSTOM_OP_NAMES`
```python
_CUSTOM_OP_NAMES: dict[MoEActivation, str] = {
    MoEActivation.SILU: "silu_and_mul",
    MoEActivation.GELU: "gelu_and_mul",
    MoEActivation.GELU_TANH: "gelu_tanh_and_mul",
    MoEActivation.SWIGLUOAI: "swigluoai_and_mul",
    MoEActivation.SWIGLUSTEP: "swiglustep_and_mul",
    MoEActivation.RELU2: "relu2",
    MoEActivation.SILU_NO_MUL: "silu_and_mul",
    MoEActivation.GELU_NO_MUL: "gelu_and_mul",
    MoEActivation.GELU_TANH_NO_MUL: "gelu_tanh_and_mul",
    MoEActivation.RELU2_NO_MUL: "relu2",
}
```
**EN:** This constant defines `_CUSTOM_OP_NAMES`. This block defines module-level constants or shared state used later in execution.
**CN:** 该模块级常量/状态定义 `_CUSTOM_OP_NAMES`。 该代码块定义了后续执行会复用的模块级常量或共享状态。

### Lines 84-89 — constant `_WITHOUT_MUL`
```python
_WITHOUT_MUL: dict[MoEActivation, MoEActivation] = {
    MoEActivation.SILU: MoEActivation.SILU_NO_MUL,
    MoEActivation.GELU: MoEActivation.GELU_NO_MUL,
    MoEActivation.GELU_TANH: MoEActivation.GELU_TANH_NO_MUL,
    MoEActivation.RELU2: MoEActivation.RELU2_NO_MUL,
}
```
**EN:** This constant defines `_WITHOUT_MUL`. This block defines module-level constants or shared state used later in execution.
**CN:** 该模块级常量/状态定义 `_WITHOUT_MUL`。 该代码块定义了后续执行会复用的模块级常量或共享状态。

### Lines 92-101 — function `activation_without_mul`
```python
def activation_without_mul(activation: str) -> str:
    """Get the non-gated variant of an activation function.

    Args:
        activation: The activation function name (e.g., "silu", "gelu")

    Returns:
        The non-gated activation name (e.g., "silu_no_mul", "gelu_no_mul")
    """
    return MoEActivation.from_str(activation).without_mul().value
```
**EN:** This function defines `activation_without_mul`. Get the non-gated variant of an activation function. The main inputs are `activation`. Key calls include `MoEActivation.from_str.without_mul`, `MoEActivation.from_str`.
**CN:** 该函数定义 `activation_without_mul`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `activation`。 关键调用包括 `MoEActivation.from_str.without_mul`, `MoEActivation.from_str`。

### Lines 104-150 — function `apply_moe_activation`
```python
def apply_moe_activation(
    activation: MoEActivation,
    output: torch.Tensor,
    input: torch.Tensor,
) -> torch.Tensor:
    """Apply MoE activation function."""
    assert input.dim() == 2, "Input must be 2D"
    assert output.dim() == 2, "Output must be 2D"
    if activation.is_gated:
        assert output.size(-1) * 2 == input.size(-1), (
            f"{activation.value} expects 2x ratio: "
            f"{output.size(-1) * 2} vs {input.size(-1)}"
        )
    else:
        assert output.size(-1) == input.size(-1), (
            f"{activation.value} expects equal sizes: "
            f"{output.size(-1)} vs {input.size(-1)}"
        )

    # Activations with gated multiplication (gate × activation(up))
    if activation == MoEActivation.SILU:
        torch.ops._C.silu_and_mul(output, input)
    elif activation == MoEActivation.GELU:
        torch.ops._C.gelu_and_mul(output, input)
    elif activation == MoEActivation.GELU_TANH:
        torch.ops._C.gelu_tanh_and_mul(output, input)
    elif activation == MoEActivation.SWIGLUOAI:
        torch.ops._C.swigluoai_and_mul(output, input)
    elif activation == MoEActivation.SWIGLUSTEP:
        from vllm.model_executor.layers.activation import swiglustep_and_mul_triton

        swiglustep_and_mul_triton(output, input)

    # Activations without gated multiplication
    elif activation == MoEActivation.SILU_NO_MUL:
        output.copy_(F.silu(input))
    elif activation == MoEActivation.GELU_NO_MUL:
        output.copy_(F.gelu(input))
    elif activation == MoEActivation.GELU_TANH_NO_MUL:
        output.copy_(F.gelu(input, approximate="tanh"))
    elif activation == MoEActivation.RELU2_NO_MUL:
        F.relu(input, inplace=True)
        torch.square(input, out=output)
    else:
        raise ValueError(f"Unsupported FusedMoe activation: {activation}")

    return output
```
**EN:** This function defines `apply_moe_activation`. Apply MoE activation function. The main inputs are `activation`, `output`, `input`. Key calls include `input.dim`, `output.dim`, `torch.ops._C.silu_and_mul`, `input.size`, `output.size`, `torch.ops._C.gelu_and_mul`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `apply_moe_activation`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `activation`, `output`, `input`。 关键调用包括 `input.dim`, `output.dim`, `torch.ops._C.silu_and_mul`, `input.size`, `output.size`, `torch.ops._C.gelu_and_mul`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] Activation gating and fused elementwise math / [CN] 激活门控与融合逐元素计算
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Core symbols: `MoEActivation`, `activation_without_mul`, `apply_moe_activation` / [CN] 核心符号：`MoEActivation`, `activation_without_mul`, `apply_moe_activation`

## Dependencies / 依赖关系
- **External**: `enum`, `torch`, `torch.nn.functional` / **外部依赖**: `enum`, `torch`, `torch.nn.functional`
- **Internal**: none / **内部依赖**: 无
- **Runtime traits**: custom C++/CUDA ops, Triton kernels / **运行时特征**: custom C++/CUDA ops, Triton kernels
