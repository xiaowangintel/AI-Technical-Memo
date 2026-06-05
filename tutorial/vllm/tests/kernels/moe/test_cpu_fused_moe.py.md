# test_cpu_fused_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_cpu_fused_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_cpu_fused_moe, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_cpu_fused_moe 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-12)
```python
import pytest
import torch

from tests.kernels.allclose_default import get_default_atol, get_default_rtol
from vllm._custom_ops import cpu_fused_moe, cpu_prepack_moe_weight
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.cpu_fused_moe import _CPU_MOE_ACT_FN
from vllm.platforms import current_platform
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; shared test helpers from tests.kernels.allclose_default; and vLLM components like vllm._custom_ops, vllm.model_executor.layers.fused_moe.activation, vllm.model_executor.layers.fused_moe.cpu_fused_moe, vllm.platforms.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；共享测试辅助模块，例如 tests.kernels.allclose_default；vLLM 内部组件，例如 vllm._custom_ops、vllm.model_executor.layers.fused_moe.activation、vllm.model_executor.layers.fused_moe.cpu_fused_moe、vllm.platforms。

### Top-level block starting at line 14 (lines 14-15)
```python
if not current_platform.is_cpu():
    pytest.skip("skipping CPU-only tests", allow_module_level=True)
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Constants and module state (lines 17-26)
```python
EXPERT_NUM = [
    8,
]
HIDDEN_DIM = [128, 2880]
INTERMEDIATE_DIM = [128, 2880]
BATCH_SIZE = [1, 64, 256]
ACT = [MoEActivation.SILU, MoEActivation.SWIGLUOAI, MoEActivation.GELU]
USE_BIAS = [True, False]
ISA = ["amx", "vec"] if torch.cpu._is_amx_tile_supported() else ["vec"]
DTYPE = [torch.bfloat16]
```
**EN:** This block centralizes shared constants and parameter grids, including EXPERT_NUM, HIDDEN_DIM, INTERMEDIATE_DIM, BATCH_SIZE, ACT. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 EXPERT_NUM、HIDDEN_DIM、INTERMEDIATE_DIM、BATCH_SIZE、ACT。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `ref_fused_moe` (lines 29-88)
```python
def ref_fused_moe(
    input: torch.Tensor,
    w13: torch.Tensor,
    w2: torch.Tensor,
    w13_bias: torch.Tensor | None,
    w2_bias: torch.Tensor | None,
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    activation: MoEActivation,
) -> torch.Tensor:
    len_experts = w13.size(0)

    cnts = topk_ids.new_zeros((topk_ids.shape[0], len_experts))
    cnts.scatter_(1, topk_ids.to(torch.int64), 1)
    tokens_per_expert = cnts.sum(dim=0)
    idxs = topk_ids.view(-1).argsort()

    sorted_tokens = input[idxs // topk_ids.shape[1]]
    tokens_per_expert = tokens_per_expert.cpu().numpy()

    outputs = []
    start_idx = 0

    for i, num_tokens in enumerate(tokens_per_expert):
        end_idx = start_idx + num_tokens
        if num_tokens == 0:
            continue
        tokens_for_this_expert = sorted_tokens[start_idx:end_idx].float()
        curr_w13 = w13[i].float()
        curr_w2 = w2[i].float()

        curr_w13_bias = None
        if w13_bias is not None:
            curr_w13_bias = w13_bias[i].float()

        curr_w2_bias = None
        if w2_bias is not None:
            curr_w2_bias = w2_bias[i].float()

        gate_up = torch.nn.functional.linear(
            tokens_for_this_expert, curr_w13, curr_w13_bias
        )
        # Note: to simulate the kernel implementation
        gate_up = _CPU_MOE_ACT_FN[activation](gate_up).to(dtype=input.dtype).float()
        expert_out = torch.nn.functional.linear(gate_up, curr_w2, curr_w2_bias)

        outputs.append(expert_out)
        start_idx = end_idx

    outs = torch.cat(outputs, dim=0) if len(outputs) else sorted_tokens.new_empty(0)
    new_x = torch.empty_like(outs)

    new_x[idxs] = outs
    final_out = (
        new_x.view(*topk_ids.shape, -1)
        .mul_(topk_weights.unsqueeze(dim=-1))
        .sum(dim=1)
        .type(input.dtype)
    )
    return final_out
```
**EN:** This helper acts as a reference implementation for fused MoE. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数为 fused MoE 提供参考实现。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_cpu_fused_moe` (lines 91-165)
```python
@pytest.mark.parametrize("batch_size", BATCH_SIZE)
@pytest.mark.parametrize("expert_num", EXPERT_NUM)
@pytest.mark.parametrize("hidden_size", HIDDEN_DIM)
@pytest.mark.parametrize("intermediate_size", INTERMEDIATE_DIM)
@pytest.mark.parametrize("use_bias", USE_BIAS)
@pytest.mark.parametrize("dtype", DTYPE)
@pytest.mark.parametrize("act", ACT)
@pytest.mark.parametrize("isa", ISA)
def test_cpu_fused_moe(
    default_vllm_config,
    batch_size: int,
    expert_num: int,
    hidden_size: int,
    intermediate_size: int,
    use_bias: bool,
    dtype: torch.dtype,
    act: MoEActivation,
    isa: str,
):
    set_random_seed(0)

    topk_num = max(expert_num // 2, 1)
    up_dim = 2 * intermediate_size

    input = torch.randn((batch_size, hidden_size), dtype=dtype) / (
        0.5 * hidden_size**0.5
    )
    w13 = torch.randn((expert_num, up_dim, hidden_size), dtype=dtype) / (
        0.5 * hidden_size**0.5
    )
    w2 = torch.randn((expert_num, hidden_size, intermediate_size), dtype=dtype) / (
        0.5 * intermediate_size**0.5
    )
    router_logits = torch.randn((batch_size, expert_num), dtype=dtype)
    w13_bias = None
    w2_bias = None
    if use_bias:
        w13_bias = torch.randn((expert_num, up_dim), dtype=dtype) / (0.5 * up_dim**0.5)
        w2_bias = torch.randn((expert_num, hidden_size), dtype=dtype) / (
            0.5 * hidden_size**0.5
        )
    score = torch.softmax(router_logits, dim=-1, dtype=torch.float32)
    topk_weight, topk_ids = torch.topk(score, topk_num)
    topk_ids = topk_ids.to(torch.int32)

    ref_output = ref_fused_moe(
        input,
        w13,
        w2,
        w13_bias,
        w2_bias,
        topk_weight,
        topk_ids,
        act,
    )

    packed_w13 = cpu_prepack_moe_weight(w13, isa)
    packed_w2 = cpu_prepack_moe_weight(w2, isa)
    output = cpu_fused_moe(
        input,
        packed_w13,
        packed_w2,
        w13_bias,
        w2_bias,
        topk_weight,
        topk_ids,
        act.value,
        isa,
    )

    atol, rtol = get_default_atol(output), get_default_rtol(output)
    (
        torch.testing.assert_close(output, ref_output, atol=atol, rtol=rtol),
        f"{torch.max(torch.abs(output - ref_output))}",
    )
```
**EN:** This pytest case verifies CPU fused MoE. It is parameterized across 8 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as default_vllm_config, batch_size, expert_num, hidden_size. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 CPU fused MoE 的行为。 它通过 8 组参数化输入覆盖多种场景；它会使用诸如 default_vllm_config、batch_size、expert_num、hidden_size 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `tests.kernels.allclose_default -> get_default_atol, get_default_rtol`
- `vllm._custom_ops -> cpu_fused_moe, cpu_prepack_moe_weight`
- `vllm.model_executor.layers.fused_moe.activation -> MoEActivation`
- `vllm.model_executor.layers.fused_moe.cpu_fused_moe -> _CPU_MOE_ACT_FN`
- `vllm.platforms -> current_platform`
- `vllm.utils.torch_utils -> set_random_seed`
