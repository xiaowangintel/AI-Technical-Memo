# norm_gate_linear.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/router/norm_gate_linear.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Fused RMSNorm + GateLinear for DeepSeek V4 MoE routing. / MoE 路由打分与 top-k 专家选择；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 5-16 — imports and setup
```python
import torch
from torch import nn

import vllm._custom_ops as ops
from vllm.model_executor.custom_op import PluggableLayer
from vllm.model_executor.layers.fused_moe.router.gate_linear import GateLinear
from vllm.model_executor.layers.layernorm import RMSNorm
from vllm.utils.torch_utils import direct_register_custom_op

DSV4_PRO_NUM_EXPERTS = 384
DSV4_PRO_HIDDEN_SIZE = 7168
DSV4_PRO_MAX_NUM_TOKENS = 16
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. It writes or updates `DSV4_PRO_NUM_EXPERTS`, `DSV4_PRO_HIDDEN_SIZE`, `DSV4_PRO_MAX_NUM_TOKENS`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 它会写入或更新 `DSV4_PRO_NUM_EXPERTS`, `DSV4_PRO_HIDDEN_SIZE`, `DSV4_PRO_MAX_NUM_TOKENS`。

### Lines 19-36 — function `_dsv4_pro_norm_gate`
```python
def _dsv4_pro_norm_gate(
    x: torch.Tensor,
    norm_weight: torch.Tensor,
    gate_weight: torch.Tensor,
    rms_eps: float,
) -> tuple[torch.Tensor, torch.Tensor]:
    """Runtime dispatcher: fused ``dsv4_norm_router_gemm`` (M<=16) vs the
    unfused ``rms_norm + dsv3_router_gemm`` fallback (M>16).

    """
    if x.shape[0] <= DSV4_PRO_MAX_NUM_TOKENS:
        return ops.dsv4_norm_router_gemm(x, norm_weight, gate_weight, rms_eps)

    normed = torch.empty_like(x)
    # Call `_C::rms_norm` here to avoid select the path of native rms
    torch.ops._C.rms_norm(normed, x, norm_weight, rms_eps)
    logits = torch.mm(normed, gate_weight.t(), out_dtype=torch.float32)
    return normed, logits
```
**EN:** This function defines `_dsv4_pro_norm_gate`. Runtime dispatcher: fused ``dsv4_norm_router_gemm`` (M<=16) vs the unfused ``rms_norm + dsv3_router_gemm`` fallback (M>16). The main inputs are `x`, `norm_weight`, `gate_weight`, `rms_eps`. Key calls include `torch.empty_like`, `torch.ops._C.rms_norm`, `torch.mm`, `ops.dsv4_norm_router_gemm`, `gate_weight.t`. It writes or updates `normed`, `logits`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_dsv4_pro_norm_gate`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `x`, `norm_weight`, `gate_weight`, `rms_eps`。 关键调用包括 `torch.empty_like`, `torch.ops._C.rms_norm`, `torch.mm`, `ops.dsv4_norm_router_gemm`, `gate_weight.t`。 它会写入或更新 `normed`, `logits`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 39-50 — function `_dsv4_pro_norm_gate_fake`
```python
def _dsv4_pro_norm_gate_fake(
    x: torch.Tensor,
    norm_weight: torch.Tensor,
    gate_weight: torch.Tensor,
    rms_eps: float,
) -> tuple[torch.Tensor, torch.Tensor]:
    num_tokens = x.shape[0]
    num_experts = gate_weight.shape[0]
    return (
        torch.empty_like(x),
        torch.empty(num_tokens, num_experts, dtype=torch.float32, device=x.device),
    )
```
**EN:** This function defines `_dsv4_pro_norm_gate_fake`. It computes routing scores or expert-selection behavior for MoE execution. The main inputs are `x`, `norm_weight`, `gate_weight`, `rms_eps`. Key calls include `torch.empty_like`, `torch.empty`. It writes or updates `num_tokens`, `num_experts`.
**CN:** 该函数定义 `_dsv4_pro_norm_gate_fake`。 它为 MoE 执行计算路由得分或专家选择行为。 其主要输入参数包括 `x`, `norm_weight`, `gate_weight`, `rms_eps`。 关键调用包括 `torch.empty_like`, `torch.empty`。 它会写入或更新 `num_tokens`, `num_experts`。

### Lines 61-114 — class `NormGateLinear`
```python
@PluggableLayer.register("norm_gated_linear")
class NormGateLinear(nn.Module):
    """RMSNorm + GateLinear, fused on DSV4-Pro only."""

    def __init__(
        self,
        hidden_size: int,
        num_experts: int,
        rms_eps: float = 1e-6,
        params_dtype: torch.dtype | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size
# ... omitted for brevity ...
        logits, _ = self.gate(normed_x)
        return normed_x, logits
```
**EN:** This class defines `NormGateLinear`. It inherits from `nn.Module`. RMSNorm + GateLinear, fused on DSV4-Pro only. Important methods include `__init__`, `forward`. Key calls include `PluggableLayer.register`, `super.__init__`, `RMSNorm`, `GateLinear`, `self.norm`, `self.gate`. It writes or updates `hidden_size`, `num_experts`, `rms_eps`, `norm`, `gate`, `e_score_correction_bias`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `NormGateLinear`。 它继承自 `nn.Module`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `forward`。 关键调用包括 `PluggableLayer.register`, `super.__init__`, `RMSNorm`, `GateLinear`, `self.norm`, `self.gate`。 它会写入或更新 `hidden_size`, `num_experts`, `rms_eps`, `norm`, `gate`, `e_score_correction_bias`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 65-95 — method `NormGateLinear.__init__`
```python
    def __init__(
        self,
        hidden_size: int,
        num_experts: int,
        rms_eps: float = 1e-6,
        params_dtype: torch.dtype | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size
        self.num_experts = num_experts
        self.rms_eps = rms_eps

        self.norm = RMSNorm(hidden_size, eps=rms_eps, dtype=params_dtype)
        self.gate = GateLinear(
            hidden_size,
            num_experts,
            bias=False,
            out_dtype=torch.float32,  # DSV4 router output is fp32
            params_dtype=params_dtype,
            prefix=f"{prefix}.gate" if prefix else "gate",
        )

        self.e_score_correction_bias = None
        self.tid2eid = None

        self._fused_kernel_supported = (
            hidden_size == DSV4_PRO_HIDDEN_SIZE
            and num_experts == DSV4_PRO_NUM_EXPERTS
            and self.gate.allow_dsv3_router_gemm  # cuda platform
        )
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `hidden_size`, `num_experts`, `rms_eps`, `params_dtype`, `prefix`. Key calls include `super.__init__`, `RMSNorm`, `GateLinear`, `super`. It writes or updates `hidden_size`, `num_experts`, `rms_eps`, `norm`, `gate`, `e_score_correction_bias`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `hidden_size`, `num_experts`, `rms_eps`, `params_dtype`, `prefix`。 关键调用包括 `super.__init__`, `RMSNorm`, `GateLinear`, `super`。 它会写入或更新 `hidden_size`, `num_experts`, `rms_eps`, `norm`, `gate`, `e_score_correction_bias`。

### Lines 97-114 — method `NormGateLinear.forward`
```python
    def forward(self, x: torch.Tensor) -> tuple[torch.Tensor, torch.Tensor]:
        if self._fused_kernel_supported:
            assert x.shape[1] == DSV4_PRO_HIDDEN_SIZE
            assert self.gate.weight.shape == (
                DSV4_PRO_NUM_EXPERTS,
                DSV4_PRO_HIDDEN_SIZE,
            )
            # This must be wrapped in a custom op because our torch.compile integration
            # does not support runtime dispatching on num_tokens.
            return torch.ops.vllm.dsv4_pro_norm_gate(
                x, self.norm.weight, self.gate.weight, self.rms_eps
            )

        # Non-Pro fallback (e.g. DSV4-Flash with hidden_size=4096):

        normed_x = self.norm(x)
        logits, _ = self.gate(normed_x)
        return normed_x, logits
```
**EN:** This method defines `forward`. It executes the main forward/runtime path for this component. The main inputs are `x`. Key calls include `self.norm`, `self.gate`, `torch.ops.vllm.dsv4_pro_norm_gate`. It writes or updates `normed_x`, `logits`, `_`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `forward`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `x`。 关键调用包括 `self.norm`, `self.gate`, `torch.ops.vllm.dsv4_pro_norm_gate`。 它会写入或更新 `normed_x`, `logits`, `_`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Moe router scoring and top-k expert selection / [CN] MoE 路由打分与 top-k 专家选择
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Core symbols: `_dsv4_pro_norm_gate`, `_dsv4_pro_norm_gate_fake`, `NormGateLinear` / [CN] 核心符号：`_dsv4_pro_norm_gate`, `_dsv4_pro_norm_gate_fake`, `NormGateLinear`

## Dependencies / 依赖关系
- **External**: `torch` / **外部依赖**: `torch`
- **Internal**: `vllm._custom_ops`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.fused_moe.router.gate_linear`, `vllm.model_executor.layers.layernorm`, `vllm.utils.torch_utils` / **内部依赖**: `vllm._custom_ops`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.fused_moe.router.gate_linear`, `vllm.model_executor.layers.layernorm`, `vllm.utils.torch_utils`
- **Runtime traits**: custom C++/CUDA ops / **运行时特征**: custom C++/CUDA ops
