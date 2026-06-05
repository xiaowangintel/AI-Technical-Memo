# marlin.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/moe_runner/marlin.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution with low-precision quantization data paths for the SGLang SRT runtime. It exposes symbols such as `MarlinRunnerInput`, `MarlinRunnerOutput`, `MarlinMoeQuantInfo`, and `fused_experts_none_to_marlin` and connects them to backend-specific paths such as `Triton` and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行，并结合低精度量化数据路径。它提供了 `MarlinRunnerInput`、`MarlinRunnerOutput`、`MarlinMoeQuantInfo` 以及 `fused_experts_none_to_marlin` 等符号，并把这些符号连接到 `Triton` 和 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-25: Imports, conditional backend setup, and runtime guards
```python
from __future__ import annotations

from dataclasses import dataclass
from typing import TYPE_CHECKING, Optional

import torch

from sglang.srt.layers.moe.moe_runner.base import (
    MoeQuantInfo,
    MoeRunnerConfig,
    RunnerInput,
    RunnerOutput,
    register_fused_func,
)
from sglang.srt.layers.moe.utils import MoeRunnerBackend

if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import (
        StandardCombineInput,
        StandardDispatchOutput,
    )

MARLIN_MOE_WORKSPACE: Optional[torch.Tensor] = None
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `dataclasses.dataclass`, `typing.TYPE_CHECKING`, `typing.Optional`, `torch`, and `sglang.srt.layers.moe.moe_runner.base.MoeQuantInfo`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `MARLIN_MOE_WORKSPACE` capture configuration, cached handles, or feature flags. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`dataclasses.dataclass`、`typing.TYPE_CHECKING`、`typing.Optional`、`torch` 以及 `sglang.srt.layers.moe.moe_runner.base.MoeQuantInfo`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `MARLIN_MOE_WORKSPACE` 这样的共享名称用于保存配置、缓存句柄或特性开关。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 26-34: Class `MarlinRunnerInput` declaration and shared state
```python
@dataclass
class MarlinRunnerInput(RunnerInput):
    """Input bundle passed to the Marlin runner core."""

    hidden_states: torch.Tensor
    topk_weights: torch.Tensor
    topk_ids: torch.Tensor
    router_logits: torch.Tensor
```
**EN:** This block introduces class `MarlinRunnerInput` and the state shared by its methods. It inherits from `RunnerInput`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Input bundle passed to the Marlin runner core.
**CN:** 该代码块引入类 `MarlinRunnerInput`，并定义其方法共享的状态。 它继承自 `RunnerInput`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 35-39: Function `MarlinRunnerInput.runner_backend` and its core logic
```python
    @property
    def runner_backend(self) -> MoeRunnerBackend:
        return MoeRunnerBackend.MARLIN
```
**EN:** This block defines `MarlinRunnerInput.runner_backend` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MarlinRunnerInput.runner_backend`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 40-45: Class `MarlinRunnerOutput` declaration and shared state
```python
@dataclass
class MarlinRunnerOutput(RunnerOutput):
    """Output bundle returned from the Marlin runner core."""

    hidden_states: torch.Tensor
```
**EN:** This block introduces class `MarlinRunnerOutput` and the state shared by its methods. It inherits from `RunnerOutput`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Output bundle returned from the Marlin runner core.
**CN:** 该代码块引入类 `MarlinRunnerOutput`，并定义其方法共享的状态。 它继承自 `RunnerOutput`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 46-50: Function `MarlinRunnerOutput.runner_backend` and its core logic
```python
    @property
    def runner_backend(self) -> MoeRunnerBackend:
        return MoeRunnerBackend.MARLIN
```
**EN:** This block defines `MarlinRunnerOutput.runner_backend` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `MarlinRunnerOutput.runner_backend`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 51-75: Class `MarlinMoeQuantInfo` declaration and shared state
```python
@dataclass
class MarlinMoeQuantInfo(MoeQuantInfo):
    """Quantization payload consumed by the Marlin backend."""

    w13_qweight: torch.Tensor
    w2_qweight: torch.Tensor
    w13_scales: torch.Tensor
    w2_scales: torch.Tensor
    w13_g_idx_sort_indices: Optional[torch.Tensor]
    w2_g_idx_sort_indices: Optional[torch.Tensor]
    weight_bits: int

    # GPTQ specific (Optional)
    w13_g_idx: Optional[torch.Tensor] = None
    w2_g_idx: Optional[torch.Tensor] = None
    is_k_full: bool = True

    # AWQ specific (Optional)
    w13_qzeros: Optional[torch.Tensor] = None
    w2_qzeros: Optional[torch.Tensor] = None

    # Optional
    expert_map: Optional[torch.Tensor] = None
```
**EN:** This block introduces class `MarlinMoeQuantInfo` and the state shared by its methods. It inherits from `MoeQuantInfo`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Quantization payload consumed by the Marlin backend.
**CN:** 该代码块引入类 `MarlinMoeQuantInfo`，并定义其方法共享的状态。 它继承自 `MoeQuantInfo`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 76-144: Function `fused_experts_none_to_marlin` and its core logic
```python
@register_fused_func("none", "marlin")
def fused_experts_none_to_marlin(
    dispatch_output: StandardDispatchOutput,
    quant_info: MarlinMoeQuantInfo,
    runner_config: MoeRunnerConfig,
) -> StandardCombineInput:
    global MARLIN_MOE_WORKSPACE
    from sglang.srt.layers.moe.fused_moe_triton.fused_marlin_moe import fused_marlin_moe
    from sglang.srt.layers.moe.token_dispatcher.standard import StandardCombineInput
    from sglang.srt.layers.quantization.marlin_utils import marlin_make_workspace

    hidden_states = dispatch_output.hidden_states
    topk_output = dispatch_output.topk_output

    assert runner_config.activation == "silu", "Only SiLU activation is supported."

    if (
        MARLIN_MOE_WORKSPACE is None
        or MARLIN_MOE_WORKSPACE.device != hidden_states.device
    ):
        MARLIN_MOE_WORKSPACE = marlin_make_workspace(
            hidden_states.device, max_blocks_per_sm=4
        )

    marlin_hidden_states = hidden_states
    # Avoid aliasing the MoE input buffer until Marlin output semantics are
    # fully validated across shared-expert and overlap paths.
    marlin_inplace = False
    if (
        quant_info.weight_bits == 4
        and quant_info.w13_qzeros is None
        and quant_info.w2_qzeros is None
        and quant_info.w13_scales.dtype == torch.float8_e8m0fnu
        and quant_info.w2_scales.dtype == torch.float8_e8m0fnu
        and hidden_states.dtype == torch.float16
    ):
        # MXFP4(E8M0) Marlin kernels are only numerically valid on the bf16
        # activation path. The fp16 + E8M0 path is intentionally not generated
        # in sgl-kernel, so upcast activations here and cast the result back.
        marlin_hidden_states = hidden_states.to(torch.bfloat16)
        marlin_inplace = False

    output = fused_marlin_moe(
        hidden_states=marlin_hidden_states,
        w1=quant_info.w13_qweight,
        w2=quant_info.w2_qweight,
        w1_scale=quant_info.w13_scales,
        w2_scale=quant_info.w2_scales,
        gating_output=topk_output.router_logits,
        topk_weights=topk_output.topk_weights,
        topk_ids=topk_output.topk_ids,
        expert_map=quant_info.expert_map,
        g_idx1=quant_info.w13_g_idx,
        g_idx2=quant_info.w2_g_idx,
        sort_indices1=quant_info.w13_g_idx_sort_indices,
        sort_indices2=quant_info.w2_g_idx_sort_indices,
        w1_zeros=quant_info.w13_qzeros,
        w2_zeros=quant_info.w2_qzeros,
        workspace=MARLIN_MOE_WORKSPACE,
        num_bits=quant_info.weight_bits,
        is_k_full=quant_info.is_k_full,
        inplace=marlin_inplace,
        routed_scaling_factor=runner_config.routed_scaling_factor,
        clamp_limit=runner_config.swiglu_limit,
    ).to(hidden_states.dtype)

    return StandardCombineInput(
        hidden_states=output,
    )
```
**EN:** This block defines `fused_experts_none_to_marlin` and contains the main logic for this step. Decorators like `register_fused_func` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_fused_func`, `fused_marlin_moe.to`, `StandardCombineInput`, `marlin_make_workspace`, and `hidden_states.to`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `hidden_states`, `topk_output`, `marlin_hidden_states`, `marlin_inplace`, and `output` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `fused_experts_none_to_marlin`，并承载这一阶段的核心逻辑。 像 `register_fused_func` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_fused_func`、`fused_marlin_moe.to`、`StandardCombineInput`、`marlin_make_workspace` 以及 `hidden_states.to`，说明该流程会编排底层辅助函数或计算内核。 像 `hidden_states`、`topk_output`、`marlin_hidden_states`、`marlin_inplace` 以及 `output` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `MarlinRunnerInput`, `MarlinRunnerOutput`, `MarlinMoeQuantInfo`, and `fused_experts_none_to_marlin`. / **主要符号**：核心入口包括 `MarlinRunnerInput`、`MarlinRunnerOutput`、`MarlinMoeQuantInfo` 以及 `fused_experts_none_to_marlin`。
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `dataclasses.dataclass`, `typing.TYPE_CHECKING`, and `typing.Optional` / **标准库**：`__future__.annotations`、`dataclasses.dataclass`、`typing.TYPE_CHECKING` 以及 `typing.Optional`
- **Third-party**: `torch` / **第三方依赖**：`torch`
- **Internal SGLang modules**: `sglang.srt.layers.moe.moe_runner.base.MoeQuantInfo`, `sglang.srt.layers.moe.moe_runner.base.MoeRunnerConfig`, `sglang.srt.layers.moe.moe_runner.base.RunnerInput`, `sglang.srt.layers.moe.moe_runner.base.RunnerOutput`, `sglang.srt.layers.moe.moe_runner.base.register_fused_func`, `sglang.srt.layers.moe.utils.MoeRunnerBackend`, `sglang.srt.layers.moe.token_dispatcher.StandardCombineInput`, `sglang.srt.layers.moe.token_dispatcher.StandardDispatchOutput`, `sglang.srt.layers.moe.fused_moe_triton.fused_marlin_moe.fused_marlin_moe`, `sglang.srt.layers.moe.token_dispatcher.standard.StandardCombineInput`, and `sglang.srt.layers.quantization.marlin_utils.marlin_make_workspace` / **SGLang 内部模块**：`sglang.srt.layers.moe.moe_runner.base.MoeQuantInfo`、`sglang.srt.layers.moe.moe_runner.base.MoeRunnerConfig`、`sglang.srt.layers.moe.moe_runner.base.RunnerInput`、`sglang.srt.layers.moe.moe_runner.base.RunnerOutput`、`sglang.srt.layers.moe.moe_runner.base.register_fused_func`、`sglang.srt.layers.moe.utils.MoeRunnerBackend`、`sglang.srt.layers.moe.token_dispatcher.StandardCombineInput`、`sglang.srt.layers.moe.token_dispatcher.StandardDispatchOutput`、`sglang.srt.layers.moe.fused_moe_triton.fused_marlin_moe.fused_marlin_moe`、`sglang.srt.layers.moe.token_dispatcher.standard.StandardCombineInput` 以及 `sglang.srt.layers.quantization.marlin_utils.marlin_make_workspace`
