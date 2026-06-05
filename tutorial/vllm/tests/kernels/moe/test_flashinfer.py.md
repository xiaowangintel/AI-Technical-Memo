# test_flashinfer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_flashinfer.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_flashinfer, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_flashinfer 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-36)
```python
from dataclasses import dataclass

import pytest
import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from vllm.config import ParallelConfig, VllmConfig, set_current_vllm_config
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.all2all_utils import (
    maybe_make_prepare_finalize,
)
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEParallelConfig,
    FusedMoEQuantConfig,
    RoutingMethodType,
    fp8_w8a8_moe_quant_config,
)
from vllm.model_executor.layers.fused_moe.experts.flashinfer_cutlass_moe import (
    FlashInferExperts,
)
from vllm.model_executor.layers.fused_moe.experts.trtllm_fp8_moe import (
    TrtLlmFp8ExpertsMonolithic,
)
from vllm.model_executor.layers.fused_moe.fused_moe import fused_experts
from vllm.model_executor.layers.quantization.utils.flashinfer_utils import (
    rotate_weights_for_fi_trtllm_fp8_per_tensor_moe,
    swap_w13_to_w31,
)
from vllm.model_executor.layers.quantization.utils.fp8_utils import input_to_float8
from vllm.model_executor.models.llama4 import Llama4MoE
from vllm.platforms import current_platform
from vllm.utils.math_utils import next_power_of_2
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as dataclasses, pytest, torch; and vLLM components like vllm.model_executor.layers.fused_moe.modular_kernel, vllm.config, vllm.model_executor.layers.fused_moe.activation, vllm.model_executor.layers.fused_moe.all2all_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 dataclasses、pytest、torch；vLLM 内部组件，例如 vllm.model_executor.layers.fused_moe.modular_kernel、vllm.config、vllm.model_executor.layers.fused_moe.activation、vllm.model_executor.layers.fused_moe.all2all_utils。

### Top-level block starting at line 38 (lines 38-44)
```python
try:
    from vllm.utils.flashinfer import has_flashinfer_cutlass_fused_moe
except ImportError:
    if current_platform.is_rocm():
        pytest.skip(
            "flashinfer not supported for vLLM on ROCm", allow_module_level=True
        )
```
**EN:** This top-level `Try` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `Try` 代码块执行周边测试所依赖的辅助逻辑。

### Top-level block starting at line 46 (lines 46-52)
```python
if not has_flashinfer_cutlass_fused_moe() or not current_platform.has_device_capability(
    90
):
    pytest.skip(
        "Supported for sm >= 90",
        allow_module_level=True,
    )
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Constants and module state (lines 54-66)
```python
NUM_EXPERTS = [16]
TOP_KS = [1]

MNK_FACTORS = [
    (256, 8192, 5120),
    (127, 4096, 5120),
    (10, 8192, 5120),
    (10, 4096, 5120),
    (1, 8192, 5120),
    (1, 4096, 5120),
]

vllm_config = VllmConfig(parallel_config=ParallelConfig(pipeline_parallel_size=1))
```
**EN:** This block centralizes shared constants and parameter grids, including NUM_EXPERTS, TOP_KS, MNK_FACTORS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 NUM_EXPERTS、TOP_KS、MNK_FACTORS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `quant_fp8_per_tensor_batches` (lines 69-84)
```python
def quant_fp8_per_tensor_batches(a):
    num_batches = a.size(0)
    a_quant = []
    a_scales = []

    for i in range(num_batches):
        a_fp8, a_global_sf = input_to_float8(a[i])
        if a_global_sf.numel() == 1:
            a_global_sf = a_global_sf.view(1, 1)
        a_quant.append(a_fp8)
        a_scales.append(a_global_sf)

    result_a_quant = torch.stack(a_quant)
    result_a_scales = torch.stack(a_scales)

    return result_a_quant, result_a_scales
```
**EN:** This helper function implements the shared logic for quant FP8 per tensor batches. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 quant FP8 per tensor batches 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `check_accuracy` (lines 87-98)
```python
def check_accuracy(ref_output, actual_output, atol=0.1, rtol=0.85, percent=0.925):
    close = torch.isclose(ref_output, actual_output, atol=atol, rtol=rtol)
    match_ratio = close.float().mean()
    assert match_ratio >= percent, (
        f"Match ratio {match_ratio:.4f} is below the threshold {percent:.4f}"
    )

    mismatch_percent = 1.0 - match_ratio.item()
    assert mismatch_percent <= 1 - percent, (
        f"Mismatch percentage {mismatch_percent:.4f} is above the threshold "
        f"{1 - percent:.4f}"
    )
```
**EN:** This helper function implements the shared logic for check accuracy. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 check accuracy 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Class `TestData` (lines 101-111)
```python
@dataclass
class TestData:
    hidden_states: torch.Tensor
    w13_quantized: torch.Tensor
    w2_quantized: torch.Tensor
    a1_scale: torch.Tensor
    a2_scale: torch.Tensor
    w13_weight_scale: torch.Tensor
    w2_weight_scale: torch.Tensor
    layer: torch.nn.Module
```
**EN:** This dataclass packages the fields needed to describe TestData. Keeping the inputs structured makes parameter sets explicit and easy to pass between helpers.
**CN:** 这个 dataclass 把描述 TestData 所需的字段组织在一起，使参数集合更明确，也更容易在辅助函数之间传递。

### Method `TestData.make_moe_tensors_8bit` (lines 112-190)
```python
    @staticmethod
    def make_moe_tensors_8bit(
        m: int,
        k: int,
        n: int,
        e: int,
        is_trtllm: bool,
        activation: MoEActivation = MoEActivation.SILU,
        topk: int = 1,
    ) -> "TestData":
        is_gated = activation.is_gated

        hidden_states = torch.randn((m, k), device="cuda", dtype=torch.bfloat16) / 10
        w13 = (
            torch.randn(
                (e, (2 * n) if is_gated else n, k), device="cuda", dtype=torch.bfloat16
            )
            / 10
        )
        w2 = torch.randn((e, k, n), device="cuda", dtype=torch.bfloat16) / 10

        # Scale to fp8
        _, a1_scale = input_to_float8(hidden_states)
        a2_scale = torch.scalar_tensor(1.0).to(device="cuda").to(dtype=torch.float32)
        w13_quantized, w13_weight_scale = quant_fp8_per_tensor_batches(w13)
        w2_quantized, w2_weight_scale = quant_fp8_per_tensor_batches(w2)

        layer = torch.nn.Module()
        layer.orig_dtype = torch.bfloat16
        layer.w13_weight = w13_quantized.clone()
        layer.w2_weight = w2_quantized.clone()
        layer.w13_input_scale = a1_scale
        layer.w2_input_scale = a2_scale
        layer.w13_weight_scale = w13_weight_scale
        layer.w2_weight_scale = w2_weight_scale
        layer.activation = activation
        # Setup dummy config.
        layer.moe_parallel_config = mk.FusedMoEParallelConfig.make_no_parallel()

        # flashinfer expects swapped rows for w13
        if is_gated:
            layer.w13_weight.data = swap_w13_to_w31(layer.w13_weight.data)
        if is_trtllm:
            rotate_weights_for_fi_trtllm_fp8_per_tensor_moe(
                layer.w13_weight, layer.w2_weight, is_gated
            )
        layer.custom_routing_function = Llama4MoE.custom_routing_function
        layer.routing_method_type = RoutingMethodType.Llama4
        layer.renormalize = False
        layer.intermediate_size_per_partition = n
        layer.ep_rank = 0
        layer.local_num_experts = e

        layer.moe = FusedMoEConfig(
            num_experts=e,
            experts_per_token=topk,
            hidden_dim=k,
            intermediate_size_per_partition=n,
            num_local_experts=e,
            num_logical_experts=e,
            moe_parallel_config=layer.moe_parallel_config,
            in_dtype=hidden_states.dtype,
            is_act_and_mul=is_gated,
            routing_method=layer.routing_method_type,
            activation=activation,
            device=w13_quantized.device,
            max_num_tokens=next_power_of_2(m),
        )

        return TestData(
            hidden_states=hidden_states,
            w13_quantized=w13_quantized,
            w2_quantized=w2_quantized,
            a1_scale=a1_scale,
            a2_scale=a2_scale,
            w13_weight_scale=w13_weight_scale,
            w2_weight_scale=w2_weight_scale,
            layer=layer,
        )
```
**EN:** This method on `TestData` implements MoE tensors 8bit. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `TestData` 中的这个方法实现了 MoE tensors 8bit。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_flashinfer_per_tensor_moe_fp8_no_graph` (lines 193-275)
```python
@pytest.mark.parametrize("m,n,k", MNK_FACTORS)
@pytest.mark.parametrize("e", NUM_EXPERTS)
@pytest.mark.parametrize("topk", TOP_KS)
@pytest.mark.parametrize("activation", [MoEActivation.SILU, MoEActivation.RELU2_NO_MUL])
def test_flashinfer_per_tensor_moe_fp8_no_graph(
    m: int,
    n: int,
    k: int,
    e: int,
    topk: int,
    activation: MoEActivation,
    monkeypatch,
):
    if not current_platform.has_device_capability(100):
        pytest.skip("Test is only supported for sm >= 100")
    set_random_seed(7)
    with set_current_vllm_config(vllm_config):
        td = TestData.make_moe_tensors_8bit(
            m, k, n, e, is_trtllm=True, activation=activation
        )

        score = torch.randn((m, e), device="cuda", dtype=torch.bfloat16)
        topk_weights, topk_ids = Llama4MoE.custom_routing_function(
            hidden_states=td.hidden_states,
            gating_output=score,
            topk=topk,
            renormalize=False,
        )

        quant_config = fp8_w8a8_moe_quant_config(
            w1_scale=td.w13_weight_scale,
            w2_scale=td.w2_weight_scale,
            a1_scale=td.a1_scale,
            a2_scale=td.a2_scale,
            per_act_token_quant=False,
        )

        output = fused_experts(
            td.hidden_states,
            td.w13_quantized,
# ... excerpt ...
            activation=activation,
            global_num_experts=e,
            expert_map=None,
            apply_router_weight_on_input=True,
            quant_config=quant_config,
        )

        kernel = mk.FusedMoEKernel(
            maybe_make_prepare_finalize(
                moe=td.layer.moe,
                quant_config=quant_config,
                allow_new_interface=True,
                use_monolithic=True,
            ),
            TrtLlmFp8ExpertsMonolithic(
                moe_config=td.layer.moe,
                quant_config=quant_config,
            ),
        )

        flashinfer_output = kernel.apply_monolithic(
            hidden_states=td.hidden_states,
            w1=td.layer.w13_weight,
            w2=td.layer.w2_weight,
            router_logits=score,
            activation=activation,
            global_num_experts=e,
            expert_map=None,
            apply_router_weight_on_input=True,
            routed_scaling_factor=1.0,
        )

        check_accuracy(
            ref_output=output,
            actual_output=flashinfer_output,
            atol=0.1,
            rtol=0.85,
            percent=0.925,
        )
```
**EN:** This pytest case verifies flashinfer per tensor MoE FP8 no graph. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as m, n, k, e. unsupported hardware, backend, or configuration combinations are skipped early. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 flashinfer per tensor MoE FP8 no graph 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 m、n、k、e 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

### Function `test_flashinfer_cutlass_moe_fp8_no_graph` (lines 278-388)
```python
@pytest.mark.parametrize("m,n,k", MNK_FACTORS)
@pytest.mark.parametrize("e", NUM_EXPERTS)
@pytest.mark.parametrize("topk", TOP_KS)
@pytest.mark.parametrize("activation", [MoEActivation.SILU, MoEActivation.RELU2_NO_MUL])
def test_flashinfer_cutlass_moe_fp8_no_graph(
    m: int,
    n: int,
    k: int,
    e: int,
    topk: int,
    activation: MoEActivation,
    monkeypatch,
    workspace_init,
):
    set_random_seed(7)
    with set_current_vllm_config(vllm_config):
        td = TestData.make_moe_tensors_8bit(
            m, k, n, e, is_trtllm=False, activation=activation
        )

        score = torch.randn((m, e), device="cuda", dtype=torch.bfloat16)
        topk_weights, topk_ids = Llama4MoE.custom_routing_function(
            hidden_states=td.hidden_states,
            gating_output=score,
            topk=topk,
            renormalize=False,
        )

        quant_config = fp8_w8a8_moe_quant_config(
            w1_scale=td.w13_weight_scale,
            g1_alphas=(td.w13_weight_scale * td.a1_scale).squeeze(),
            w2_scale=td.w2_weight_scale,
            g2_alphas=(td.w2_weight_scale * td.a2_scale).squeeze(),
            a1_scale=td.a1_scale,
            a1_gscale=td.a1_scale,
            a2_scale=td.a2_scale,
            a2_gscale=1.0 / td.a2_scale,
            per_act_token_quant=False,
        )

# ... excerpt ...
            in_dtype=torch.bfloat16,
            is_act_and_mul=activation.is_gated,
            routing_method=RoutingMethodType.TopK,
            max_num_tokens=next_power_of_2(m),
        )

        kernel = mk.FusedMoEKernel(
            maybe_make_prepare_finalize(
                moe=moe_config,
                quant_config=quant_config,
                allow_new_interface=True,
                use_monolithic=False,
            ),
            FlashInferExperts(
                moe_config=moe_config,
                quant_config=quant_config,
            ),
            inplace=False,
        )

        flashinfer_cutlass_output = kernel.apply(
            td.hidden_states,
            td.layer.w13_weight,
            td.layer.w2_weight,
            topk_weights,
            topk_ids,
            activation=activation,
            global_num_experts=e,
            expert_map=None,
            apply_router_weight_on_input=True,
        )

        check_accuracy(
            ref_output=output,
            actual_output=flashinfer_cutlass_output,
            atol=0.1,
            rtol=0.85,
            percent=0.925,
        )
```
**EN:** This pytest case verifies flashinfer cutlass MoE FP8 no graph. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as m, n, k, e. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 flashinfer cutlass MoE FP8 no graph 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 m、n、k、e 等 fixture 或输入；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

### Function `test_convert_moe_weights_to_flashinfer_trtllm_block_layout` (lines 391-429)
```python
@pytest.mark.parametrize(
    "num_experts,intermediate,hidden",
    [
        (8, 2048, 1536),
        (64, 4096, 4096),
    ],
)
def test_convert_moe_weights_to_flashinfer_trtllm_block_layout(
    num_experts, intermediate, hidden
):
    from vllm.model_executor.layers.quantization.utils.flashinfer_utils import (
        convert_moe_weights_to_flashinfer_trtllm_block_layout,
    )

    w13 = torch.randn(
        (num_experts, 2 * intermediate, hidden), dtype=torch.bfloat16, device="cuda"
    )
    w2 = torch.randn(
        (num_experts, hidden, intermediate), dtype=torch.bfloat16, device="cuda"
    )

    cache: dict[torch.Size, torch.Tensor] = {}
    w13_converted, w2_converted = convert_moe_weights_to_flashinfer_trtllm_block_layout(
        cache, w13, w2
    )

    assert w13_converted.ndim == 4, (
        f"Expected 4D tensor, got shape {w13_converted.shape}"
    )
    assert w2_converted.ndim == 4, f"Expected 4D tensor, got shape {w2_converted.shape}"

    assert w13_converted.numel() == w13.numel(), "W13 element count should be preserved"
    assert w2_converted.numel() == w2.numel(), "W2 element count should be preserved"

    assert w13_converted.dtype == torch.bfloat16
    assert w2_converted.dtype == torch.bfloat16

    assert w13_converted.shape[0] == num_experts
    assert w2_converted.shape[0] == num_experts
```
**EN:** This pytest case verifies convert MoE weights to flashinfer trtllm block layout. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as num_experts, intermediate, hidden. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 convert MoE weights to flashinfer trtllm block layout 的行为。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 num_experts、intermediate、hidden 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `dataclasses -> dataclass`
- `pytest`
- `torch`
- `vllm.model_executor.layers.fused_moe.modular_kernel`
- `vllm.config -> ParallelConfig, VllmConfig, set_current_vllm_config`
- `vllm.model_executor.layers.fused_moe.activation -> MoEActivation`
- `vllm.model_executor.layers.fused_moe.all2all_utils -> maybe_make_prepare_finalize`
- `vllm.model_executor.layers.fused_moe.config -> FusedMoEConfig, FusedMoEParallelConfig, FusedMoEQuantConfig, RoutingMethodType, fp8_w8a8_moe_quant_config`
- `vllm.model_executor.layers.fused_moe.experts.flashinfer_cutlass_moe -> FlashInferExperts`
- `vllm.model_executor.layers.fused_moe.experts.trtllm_fp8_moe -> TrtLlmFp8ExpertsMonolithic`
- `vllm.model_executor.layers.fused_moe.fused_moe -> fused_experts`
- `vllm.model_executor.layers.quantization.utils.flashinfer_utils -> rotate_weights_for_fi_trtllm_fp8_per_tensor_moe, swap_w13_to_w31`
- `vllm.model_executor.layers.quantization.utils.fp8_utils -> input_to_float8`
- `vllm.model_executor.models.llama4 -> Llama4MoE`
- `vllm.platforms -> current_platform`
- `vllm.utils.math_utils -> next_power_of_2`
