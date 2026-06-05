# cpu_fused_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/cpu_fused_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fused Mixture-of-Experts routing, kernels, and runtime helpers / 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-21 — imports and setup
```python
import weakref
from collections.abc import Callable

import torch
from torch.nn import functional as F

from vllm import _custom_ops as ops
from vllm._custom_ops import (
    CPUQuantMethod,
    cpu_fused_moe,
    cpu_prepack_moe_weight,
    fused_experts_cpu,
)
from vllm.model_executor.layers.activation import SiluAndMul
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.quantization.utils.layer_utils import replace_parameter
from vllm.utils.torch_utils import direct_register_custom_op

_CPU_MOE_LAYER_CACHE = {}
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. It writes or updates `_CPU_MOE_LAYER_CACHE`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 它会写入或更新 `_CPU_MOE_LAYER_CACHE`。

### Lines 24-39 — function `_swigluoai_forward_native`
```python
def _swigluoai_forward_native(
    x: torch.Tensor,
    alpha: float = 1.702,
    limit: float = 7.0,
) -> torch.Tensor:
    """PyTorch-native implementation of SwigluOAIAndMul.forward_native.

    Standalone function to avoid instantiating SwigluOAIAndMul (a CustomOp)
    which would trigger get_current_vllm_config() before config is set.
    """
    gate, up = x[..., ::2], x[..., 1::2]
    gate = gate.clamp(min=None, max=limit)
    up = up.clamp(min=-limit, max=limit)
    glu = gate * torch.sigmoid(gate * alpha)
    gated_output = (up + 1) * glu
    return gated_output
```
**EN:** This function defines `_swigluoai_forward_native`. PyTorch-native implementation of SwigluOAIAndMul.forward_native. The main inputs are `x`, `alpha`, `limit`. Key calls include `gate.clamp`, `up.clamp`, `torch.sigmoid`. It writes or updates `gate`, `up`, `glu`, `gated_output`.
**CN:** 该函数定义 `_swigluoai_forward_native`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `x`, `alpha`, `limit`。 关键调用包括 `gate.clamp`, `up.clamp`, `torch.sigmoid`。 它会写入或更新 `gate`, `up`, `glu`, `gated_output`。

### Lines 42-46 — function `_gelu_and_mul`
```python
def _gelu_and_mul(
    x: torch.Tensor,
) -> torch.Tensor:
    d = x.shape[-1] // 2
    return F.gelu(x[..., :d], approximate="none") * x[..., d:]
```
**EN:** This function defines `_gelu_and_mul`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `x`. Key calls include `F.gelu`. It writes or updates `d`.
**CN:** 该函数定义 `_gelu_and_mul`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `x`。 关键调用包括 `F.gelu`。 它会写入或更新 `d`。

### Lines 52-56 — constant `_CPU_MOE_ACT_FN`
```python
_CPU_MOE_ACT_FN: dict[MoEActivation, Callable[[torch.Tensor], torch.Tensor]] = {
    MoEActivation.SILU: lambda x: SiluAndMul(compile_native=False).forward_native(x),
    MoEActivation.SWIGLUOAI: _swigluoai_forward_native,
    MoEActivation.GELU: _gelu_and_mul,
}
```
**EN:** This constant defines `_CPU_MOE_ACT_FN`. This block defines module-level constants or shared state used later in execution.
**CN:** 该模块级常量/状态定义 `_CPU_MOE_ACT_FN`。 该代码块定义了后续执行会复用的模块级常量或共享状态。

### Lines 59-114 — function `grouped_topk`
```python
def grouped_topk(
    hidden_states: torch.Tensor,
    gating_output: torch.Tensor,
    topk: int,
    renormalize: bool,
    num_expert_group: int = 0,
    topk_group: int = 0,
    scoring_func: str = "softmax",
    routed_scaling_factor: float = 1.0,
    e_score_correction_bias: torch.Tensor | None = None,
) -> tuple[torch.Tensor, torch.Tensor]:
    assert hidden_states.shape[0] == gating_output.shape[0], "Number of tokens mismatch"

    gating_output = gating_output.float()
    if scoring_func == "softmax":
        scores = torch.softmax(gating_output, dim=-1)
    elif scoring_func == "sigmoid":
        scores = gating_output.sigmoid()
    else:
        raise ValueError(f"Unsupported scoring function: {scoring_func}")

    num_token = scores.shape[0]
    if e_score_correction_bias is not None:
        original_scores = scores
        scores = scores + e_score_correction_bias.unsqueeze(0)
        group_scores = (
            scores.view(num_token, num_expert_group, -1).topk(2, dim=-1)[0].sum(dim=-1)
        )
    else:
        group_scores = (
            scores.view(num_token, num_expert_group, -1).max(dim=-1).values
        )  # [n, n_group]
    group_idx = torch.topk(group_scores, k=topk_group, dim=-1, sorted=False)[
        1
    ]  # [n, top_k_group]
    group_mask = torch.zeros_like(group_scores)  # [n, n_group]
    group_mask.scatter_(1, group_idx, 1)  # [n, n_group]
    score_mask = (
        group_mask.unsqueeze(-1)
        .expand(num_token, num_expert_group, scores.shape[-1] // num_expert_group)
        .reshape(num_token, -1)
    )  # [n, e]
    tmp_scores = scores.masked_fill(~score_mask.bool(), float("-inf"))  # [n, e]

# ... omitted for brevity ...
        topk_weights = topk_weights * routed_scaling_factor
    return topk_weights, topk_ids.to(torch.int32)
```
**EN:** This function defines `grouped_topk`. It computes routing scores or expert-selection behavior for MoE execution. The main inputs are `hidden_states`, `gating_output`, `topk`, `renormalize`, `num_expert_group`, `topk_group`. Key calls include `gating_output.float`, `torch.zeros_like`, `group_mask.scatter_`, `group_mask.unsqueeze.expand.reshape`, `scores.masked_fill`, `torch.softmax`. It writes or updates `gating_output`, `num_token`, `group_idx`, `group_mask`, `score_mask`, `tmp_scores`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `grouped_topk`。 它为 MoE 执行计算路由得分或专家选择行为。 其主要输入参数包括 `hidden_states`, `gating_output`, `topk`, `renormalize`, `num_expert_group`, `topk_group`。 关键调用包括 `gating_output.float`, `torch.zeros_like`, `group_mask.scatter_`, `group_mask.unsqueeze.expand.reshape`, `scores.masked_fill`, `torch.softmax`。 它会写入或更新 `gating_output`, `num_token`, `group_idx`, `group_mask`, `score_mask`, `tmp_scores`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 117-161 — function `select_experts`
```python
def select_experts(
    hidden_states: torch.Tensor,
    router_logits: torch.Tensor,
    top_k: int,
    use_grouped_topk: bool,
    renormalize: bool,
    topk_group: int | None = None,
    num_expert_group: int | None = None,
    custom_routing_function: Callable | None = None,
    scoring_func: str = "softmax",
    routed_scaling_factor: float = 1.0,
    e_score_correction_bias: torch.Tensor | None = None,
) -> tuple[torch.Tensor, torch.Tensor]:
    if use_grouped_topk:
        assert topk_group is not None
        assert num_expert_group is not None
        return grouped_topk(
            hidden_states=hidden_states,
            gating_output=router_logits,
            topk=top_k,
            renormalize=renormalize,
            num_expert_group=num_expert_group,
            topk_group=topk_group,
            scoring_func=scoring_func,
            routed_scaling_factor=routed_scaling_factor,
            e_score_correction_bias=e_score_correction_bias,
        )
    elif custom_routing_function is None:
        assert scoring_func == "softmax"
        topk_logit_vals, topk_idx = torch.topk(
            router_logits, k=top_k, dim=-1, sorted=False
        )
        if renormalize:
            topk_vals = torch.softmax(topk_logit_vals, dim=-1)
        else:
            logZ = torch.logsumexp(router_logits, dim=-1, keepdim=True)
            topk_vals = (topk_logit_vals - logZ).exp()
        return topk_vals.to(torch.float32), topk_idx.to(torch.int32)
    else:
        return custom_routing_function(
            hidden_states=hidden_states,
            gating_output=router_logits,
            topk=top_k,
            renormalize=renormalize,
        )
```
**EN:** This function defines `select_experts`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `hidden_states`, `router_logits`, `top_k`, `use_grouped_topk`, `renormalize`, `topk_group`. Key calls include `grouped_topk`, `torch.topk`, `custom_routing_function`, `torch.softmax`, `torch.logsumexp`, `exp`. It writes or updates `topk_logit_vals`, `topk_idx`, `topk_vals`, `logZ`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `select_experts`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `hidden_states`, `router_logits`, `top_k`, `use_grouped_topk`, `renormalize`, `topk_group`。 关键调用包括 `grouped_topk`, `torch.topk`, `custom_routing_function`, `torch.softmax`, `torch.logsumexp`, `exp`。 它会写入或更新 `topk_logit_vals`, `topk_idx`, `topk_vals`, `logZ`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 164-221 — class `SGLFusedMOE`
```python
class SGLFusedMOE:
    def __init__(self, layer: torch.nn.Module) -> None:
        pass

    def __call__(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        use_grouped_topk: bool,
        top_k: int,
        router_logits: torch.Tensor,
        renormalize: bool,
        topk_group: int | None = None,
        num_expert_group: int | None = None,
# ... omitted for brevity ...
            True,  # is_vnni
        )
```
**EN:** This class defines `SGLFusedMOE`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. Important methods include `__init__`, `__call__`. Key calls include `select_experts`, `fused_experts_cpu`. It writes or updates `topk_weights`, `topk_ids`.
**CN:** 该类定义了 `SGLFusedMOE`。 它管理专家分发、融合内核或 MoE 特有的张量流。 重要方法包括 `__init__`, `__call__`。 关键调用包括 `select_experts`, `fused_experts_cpu`。 它会写入或更新 `topk_weights`, `topk_ids`。

### Lines 165-166 — method `SGLFusedMOE.__init__`
```python
    def __init__(self, layer: torch.nn.Module) -> None:
        pass
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `layer`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `layer`。

### Lines 168-221 — method `SGLFusedMOE.__call__`
```python
    def __call__(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        use_grouped_topk: bool,
        top_k: int,
        router_logits: torch.Tensor,
        renormalize: bool,
        topk_group: int | None = None,
        num_expert_group: int | None = None,
        global_num_experts: int = -1,
        expert_map: torch.Tensor | None = None,
        custom_routing_function: Callable | None = None,
        scoring_func: str = "softmax",
        routed_scaling_factor: float = 1.0,
        e_score_correction_bias: torch.Tensor | None = None,
        apply_router_weight_on_input: bool = False,
        activation: MoEActivation = MoEActivation.SILU,
    ) -> torch.Tensor:
        assert activation == MoEActivation.SILU, f"{activation} is not supported."
        assert not apply_router_weight_on_input
        topk_weights, topk_ids = select_experts(
            hidden_states=x,
            router_logits=router_logits,
            use_grouped_topk=use_grouped_topk,
            top_k=top_k,
            renormalize=renormalize,
            topk_group=topk_group,
            num_expert_group=num_expert_group,
            custom_routing_function=custom_routing_function,
            scoring_func=scoring_func,
            routed_scaling_factor=routed_scaling_factor,
            e_score_correction_bias=e_score_correction_bias,
        )

        return fused_experts_cpu(
            x,
            layer.w13_weight,
            layer.w2_weight,
            topk_weights,
            topk_ids,
            False,  # inplace
            CPUQuantMethod.UNQUANT,  # moe_comp_method
            None,  # w1_scale
# ... omitted for brevity ...
            True,  # is_vnni
        )
```
**EN:** This method defines `__call__`. It executes the main forward/runtime path for this component. The main inputs are `layer`, `x`, `use_grouped_topk`, `top_k`, `router_logits`, `renormalize`. Key calls include `select_experts`, `fused_experts_cpu`. It writes or updates `topk_weights`, `topk_ids`.
**CN:** 该方法定义 `__call__`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `layer`, `x`, `use_grouped_topk`, `top_k`, `router_logits`, `renormalize`。 关键调用包括 `select_experts`, `fused_experts_cpu`。 它会写入或更新 `topk_weights`, `topk_ids`。

### Lines 224-425 — class `CPUFusedMOE`
```python
class CPUFusedMOE:
    """CPU-based fused MoE implementation."""

    def __init__(self, layer: torch.nn.Module) -> None:
        use_grouped_gemm, isa = self.check_grouped_gemm(layer)
        self.isa = isa
        if use_grouped_gemm:
            self.forward_method = self.forward_grouped_gemm
            self.init_moe_grouped_gemm(layer=layer)
        else:
            self.forward_method = self.forward_torch
            self.init_moe_torch(layer=layer)

    def __call__(
# ... omitted for brevity ...

        return output
```
**EN:** This class defines `CPUFusedMOE`. CPU-based fused MoE implementation. Important methods include `__init__`, `__call__`, `check_grouped_gemm`, `init_moe_grouped_gemm`, `init_moe_torch`, `forward_grouped_gemm`. Key calls include `self.check_grouped_gemm`, `select_experts`, `self.forward_method`, `layer.w13_weight.size`, `layer.w2_weight.size`, `torch.cpu._is_amx_tile_supported`. It writes or updates `use_grouped_gemm`, `isa`, `topk_weights`, `topk_ids`, `dtype`, `w13_input_size`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该类定义了 `CPUFusedMOE`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `__call__`, `check_grouped_gemm`, `init_moe_grouped_gemm`, `init_moe_torch`, `forward_grouped_gemm`。 关键调用包括 `self.check_grouped_gemm`, `select_experts`, `self.forward_method`, `layer.w13_weight.size`, `layer.w2_weight.size`, `torch.cpu._is_amx_tile_supported`。 它会写入或更新 `use_grouped_gemm`, `isa`, `topk_weights`, `topk_ids`, `dtype`, `w13_input_size`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 227-235 — method `CPUFusedMOE.__init__`
```python
    def __init__(self, layer: torch.nn.Module) -> None:
        use_grouped_gemm, isa = self.check_grouped_gemm(layer)
        self.isa = isa
        if use_grouped_gemm:
            self.forward_method = self.forward_grouped_gemm
            self.init_moe_grouped_gemm(layer=layer)
        else:
            self.forward_method = self.forward_torch
            self.init_moe_torch(layer=layer)
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `layer`. Key calls include `self.check_grouped_gemm`, `self.init_moe_grouped_gemm`, `self.init_moe_torch`. It writes or updates `use_grouped_gemm`, `isa`, `forward_method`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `layer`。 关键调用包括 `self.check_grouped_gemm`, `self.init_moe_grouped_gemm`, `self.init_moe_torch`。 它会写入或更新 `use_grouped_gemm`, `isa`, `forward_method`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 237-280 — method `CPUFusedMOE.__call__`
```python
    def __call__(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        use_grouped_topk: bool,
        top_k: int,
        router_logits: torch.Tensor,
        renormalize: bool,
        topk_group: int | None = None,
        num_expert_group: int | None = None,
        global_num_experts: int = -1,
        expert_map: torch.Tensor | None = None,
        custom_routing_function: Callable | None = None,
        scoring_func: str = "softmax",
        routed_scaling_factor: float = 1.0,
        e_score_correction_bias: torch.Tensor | None = None,
        apply_router_weight_on_input: bool = False,
        activation: MoEActivation = MoEActivation.SILU,
    ) -> torch.Tensor:
        assert activation in _CPU_MOE_ACT_FN, f"{activation} is not supported."

        topk_weights, topk_ids = select_experts(
            hidden_states=x,
            router_logits=router_logits,
            use_grouped_topk=use_grouped_topk,
            top_k=top_k,
            renormalize=renormalize,
            topk_group=topk_group,
            num_expert_group=num_expert_group,
            custom_routing_function=custom_routing_function,
            scoring_func=scoring_func,
            routed_scaling_factor=routed_scaling_factor,
            e_score_correction_bias=e_score_correction_bias,
        )

        return self.forward_method(
            layer,
            x,
            topk_weights,
            topk_ids,
            activation,
            global_num_experts,
            apply_router_weight_on_input,
        )
```
**EN:** This method defines `__call__`. It executes the main forward/runtime path for this component. The main inputs are `layer`, `x`, `use_grouped_topk`, `top_k`, `router_logits`, `renormalize`. Key calls include `select_experts`, `self.forward_method`. It writes or updates `topk_weights`, `topk_ids`.
**CN:** 该方法定义 `__call__`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `layer`, `x`, `use_grouped_topk`, `top_k`, `router_logits`, `renormalize`。 关键调用包括 `select_experts`, `self.forward_method`。 它会写入或更新 `topk_weights`, `topk_ids`。

### Lines 282-311 — method `CPUFusedMOE.check_grouped_gemm`
```python
    def check_grouped_gemm(
        self,
        layer: torch.nn.Module,
    ) -> tuple[bool, str]:
        if not hasattr(torch.ops._C, "prepack_moe_weight"):
            return False, "none"

        dtype = layer.w13_weight.dtype
        w13_input_size = layer.w13_weight.size(2)
        w13_output_size = layer.w13_weight.size(1)
        w2_input_size = layer.w2_weight.size(2)
        w2_output_size = layer.w2_weight.size(1)

        if not (w13_output_size % 32 == 0 and w2_output_size % 32 == 0):
            return False, "none"

        supports_amx = torch.cpu._is_amx_tile_supported()

        if (
            supports_amx
            and dtype == torch.bfloat16
            and w13_input_size % 32 == 0
            and w2_input_size % 32 == 0
        ):
            return True, "amx"

        if supports_amx:
            return False, "none"

        return True, "vec"
```
**EN:** This method defines `check_grouped_gemm`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `layer`. Key calls include `layer.w13_weight.size`, `layer.w2_weight.size`, `torch.cpu._is_amx_tile_supported`, `hasattr`. It writes or updates `dtype`, `w13_input_size`, `w13_output_size`, `w2_input_size`, `w2_output_size`, `supports_amx`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `check_grouped_gemm`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `layer`。 关键调用包括 `layer.w13_weight.size`, `layer.w2_weight.size`, `torch.cpu._is_amx_tile_supported`, `hasattr`。 它会写入或更新 `dtype`, `w13_input_size`, `w13_output_size`, `w2_input_size`, `w2_output_size`, `supports_amx`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 313-320 — method `CPUFusedMOE.init_moe_grouped_gemm`
```python
    def init_moe_grouped_gemm(
        self,
        layer: torch.nn.Module,
    ) -> None:
        new_w13 = cpu_prepack_moe_weight(layer.w13_weight, self.isa)
        replace_parameter(layer, "w13_weight", new_w13)
        new_w2 = cpu_prepack_moe_weight(layer.w2_weight, self.isa)
        replace_parameter(layer, "w2_weight", new_w2)
```
**EN:** This method defines `init_moe_grouped_gemm`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `layer`. Key calls include `cpu_prepack_moe_weight`, `replace_parameter`. It writes or updates `new_w13`, `new_w2`.
**CN:** 该方法定义 `init_moe_grouped_gemm`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `layer`。 关键调用包括 `cpu_prepack_moe_weight`, `replace_parameter`。 它会写入或更新 `new_w13`, `new_w2`。

## Key Concepts / 关键概念
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `_swigluoai_forward_native`, `_gelu_and_mul`, `grouped_topk`, `select_experts` / [CN] 核心符号：`_swigluoai_forward_native`, `_gelu_and_mul`, `grouped_topk`, `select_experts`

## Dependencies / 依赖关系
- **External**: `weakref`, `collections.abc`, `torch`, `torch.nn` / **外部依赖**: `weakref`, `collections.abc`, `torch`, `torch.nn`
- **Internal**: `vllm`, `vllm._custom_ops`, `vllm.model_executor.layers.activation`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.quantization.utils.layer_utils`, `vllm.utils.torch_utils` / **内部依赖**: `vllm`, `vllm._custom_ops`, `vllm.model_executor.layers.activation`, `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.quantization.utils.layer_utils`, `vllm.utils.torch_utils`
- **Runtime traits**: custom C++/CUDA ops / **运行时特征**: custom C++/CUDA ops
