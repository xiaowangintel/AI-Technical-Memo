# test_batched_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_batched_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_batched_moe, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_batched_moe 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-24)
```python
from dataclasses import dataclass

import pytest
import torch

from tests.kernels.moe.utils import (
    batched_moe,
    make_quantized_test_activations,
    make_test_weights,
    naive_batched_moe,
)
from tests.kernels.quant_utils import native_batched_masked_quant_matmul
from tests.kernels.utils import torch_experts
from vllm.config import VllmConfig, set_current_vllm_config
from vllm.model_executor.layers.fused_moe import fused_topk
from vllm.model_executor.layers.fused_moe.experts.fused_batched_moe import (
    invoke_moe_batched_triton_kernel,
)
from vllm.platforms import current_platform
from vllm.triton_utils import tl
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as dataclasses, pytest, torch; shared test helpers from tests.kernels.moe.utils, tests.kernels.quant_utils, tests.kernels.utils; and vLLM components like vllm.config, vllm.model_executor.layers.fused_moe, vllm.model_executor.layers.fused_moe.experts.fused_batched_moe, vllm.platforms.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 dataclasses、pytest、torch；共享测试辅助模块，例如 tests.kernels.moe.utils、tests.kernels.quant_utils、tests.kernels.utils；vLLM 内部组件，例如 vllm.config、vllm.model_executor.layers.fused_moe、vllm.model_executor.layers.fused_moe.experts.fused_batched_moe、vllm.platforms。

### Constants and module state (lines 26-43)
```python
MNK_FACTORS = [
    (1, 128, 128),
    (1, 512, 512),
    (1, 1024, 2048),
    (32, 128, 128),
    (32, 512, 512),
    (32, 1024, 2048),
    (45, 128, 2048),
    (45, 1024, 128),
    (64, 512, 512),
    (64, 1024, 2048),
    (222, 128, 2048),
    (222, 1024, 2048),
]
NUM_EXPERTS = [8, 64]
TOP_KS = [1, 2, 6]

DTYPES = [torch.bfloat16]
```
**EN:** This block centralizes shared constants and parameter grids, including MNK_FACTORS, NUM_EXPERTS, TOP_KS, DTYPES. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 MNK_FACTORS、NUM_EXPERTS、TOP_KS、DTYPES。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Top-level block starting at line 45 (lines 45-46)
```python
if not current_platform.is_fp8_fnuz():
    DTYPES.append(torch.float8_e4m3fn)
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Constants and module state (lines 48-48)
```python
vllm_config = VllmConfig()
```
**EN:** This block prepares module-level state that later helpers and tests reuse.
**CN:** 该代码块准备了后续辅助函数和测试会复用的模块级状态。

### Class `BatchedMMConfig` (lines 51-59)
```python
@dataclass
class BatchedMMConfig:
    in_dtype: torch.dtype
    quant_dtype: torch.dtype | None
    out_dtype: torch.dtype
    num_experts: int
    max_tokens_per_expert: int
    K: int
    N: int
```
**EN:** This dataclass packages the fields needed to describe BatchedMMConfig. Keeping the inputs structured makes parameter sets explicit and easy to pass between helpers.
**CN:** 这个 dataclass 把描述 BatchedMMConfig 所需的字段组织在一起，使参数集合更明确，也更容易在辅助函数之间传递。

### Class `BatchedMMTensors` (lines 62-68)
```python
@dataclass
class BatchedMMTensors:
    A: torch.Tensor  # [E, max_tokens, K]
    B: torch.Tensor  # [E, K, N] - column major
    C: torch.Tensor  # [E, max_tokens, N]
    num_expert_tokens: torch.Tensor  # [E]
```
**EN:** This dataclass packages the fields needed to describe BatchedMMTensors. Keeping the inputs structured makes parameter sets explicit and easy to pass between helpers.
**CN:** 这个 dataclass 把描述 BatchedMMTensors 所需的字段组织在一起，使参数集合更明确，也更容易在辅助函数之间传递。

### Method `BatchedMMTensors.make_tensors` (lines 69-98)
```python
    @staticmethod
    def make_tensors(config: BatchedMMConfig):
        A = (
            torch.randn(
                (config.num_experts, config.max_tokens_per_expert, config.K),
                device="cuda",
                dtype=config.in_dtype,
            )
            / 10
        )
        B = torch.randn(
            (config.num_experts, config.N, config.K),
            device="cuda",
            dtype=config.in_dtype,
        )
        C = torch.zeros(
            (config.num_experts, config.max_tokens_per_expert, config.N),
            device="cuda",
            dtype=config.out_dtype,
        )

        num_expert_tokens = torch.randint(
            low=0,
            high=config.max_tokens_per_expert,
            size=(config.num_experts,),
            device="cuda",
            dtype=torch.int32,
        )

        return BatchedMMTensors(A, B, C, num_expert_tokens)
```
**EN:** This method on `BatchedMMTensors` implements tensors. It keeps the surrounding module logic factored into a reusable unit.
**CN:** `BatchedMMTensors` 中的这个方法实现了 tensors。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_batched_mm` (lines 101-232)
```python
@pytest.mark.parametrize("num_experts", [8, 32])
@pytest.mark.parametrize("max_tokens_per_expert", [32, 224, 512])
@pytest.mark.parametrize("K", [128, 1024])
@pytest.mark.parametrize("N", [128, 1024])
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("block_shape", [None, [128, 128]])
@pytest.mark.parametrize("per_act_token_quant", [False, True])
def test_batched_mm(
    num_experts: int,
    max_tokens_per_expert: int,
    K: int,
    N: int,
    dtype: torch.dtype,
    block_shape: list[int] | None,
    per_act_token_quant: bool,
):
    """Note: float8_e4m3fn is not supported on CUDA architecture < 89,
    and those tests will be skipped on unsupported hardware."""
    set_random_seed(7)

    use_fp8_w8a8 = dtype == torch.float8_e4m3fn

    if (dtype == torch.float8_e4m3fn) and not current_platform.has_device_capability(
        89
    ):
        pytest.skip(
            "Triton limitation: fp8e4nv data type is not supported on CUDA arch < 89"
        )

    if (per_act_token_quant or block_shape is not None) and not use_fp8_w8a8:
        pytest.skip("Don't test blocking for non-quantized types.")

    if per_act_token_quant and block_shape is not None:
        pytest.skip("Skip illegal quantization test.")

    if dtype.itemsize == 1:
        act_dtype = torch.bfloat16
        quant_dtype = dtype
    else:
        act_dtype = dtype
# ... excerpt ...
        # Quantization schemes
        use_fp8_w8a8,
        False,
        False,
        config={
            "BLOCK_SIZE_M": 16,
            "BLOCK_SIZE_N": 16,
            "BLOCK_SIZE_K": 16 if dtype.itemsize > 1 else 32,
        },
        per_act_token_quant=per_act_token_quant,
        block_shape=block_shape,
    )

    ref_output = native_batched_masked_quant_matmul(
        A,
        B,
        ref_output,
        num_expert_tokens,
    )

    q_ref_output = native_batched_masked_quant_matmul(
        A_q,
        B_q,
        q_ref_output,
        num_expert_tokens,
        A_scale,
        B_scale,
        block_shape,
        per_act_token_quant,
    )

    rtol, atol = {
        torch.float16: (6e-2, 6e-2),
        torch.bfloat16: (6e-2, 6e-2),
        torch.float32: (1e-2, 1e-2),
    }[test_output.dtype]

    torch.testing.assert_close(ref_output, q_ref_output, atol=atol, rtol=rtol)
    torch.testing.assert_close(test_output, q_ref_output, atol=atol, rtol=rtol)
```
**EN:** This pytest case verifies batched mm. It is parameterized across 7 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_experts, max_tokens_per_expert, K, N. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 batched mm 的行为。 它通过 7 组参数化输入覆盖多种场景；它会使用诸如 num_experts、max_tokens_per_expert、K、N 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

### Function `test_fused_moe_batched_experts` (lines 235-353)
```python
@pytest.mark.parametrize(("m", "n", "k"), MNK_FACTORS)
@pytest.mark.parametrize("e", NUM_EXPERTS)
@pytest.mark.parametrize("topk", TOP_KS)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("per_act_token_quant", [False, True])
@pytest.mark.parametrize("block_shape", [None, [128, 128]])
@pytest.mark.parametrize("input_scales", [False])
def test_fused_moe_batched_experts(
    m: int,
    n: int,
    k: int,
    e: int,
    topk: int,
    dtype: torch.dtype,
    per_act_token_quant: bool,
    block_shape: list[int] | None,
    input_scales: bool,
    workspace_init,
):
    """Note: float8_e4m3fn is not supported on CUDA architecture < 89,
    and those tests will be skipped on unsupported hardware."""
    set_random_seed(7)

    use_fp8_w8a8 = dtype == torch.float8_e4m3fn

    if (dtype == torch.float8_e4m3fn) and not current_platform.has_device_capability(
        89
    ):
        pytest.skip(
            "Triton limitation: fp8e4nv data type is not supported on CUDA arch < 89"
        )

    if topk > e:
        pytest.skip("topk > e")

    if not use_fp8_w8a8 and (per_act_token_quant or block_shape is not None):
        pytest.skip("Skip quantization test for non-quantized type")

    if per_act_token_quant and block_shape is not None:
        pytest.skip("Skip illegal quantization test.")
# ... excerpt ...
            a2_scale=a2_scale,
            quant_dtype=quant_dtype,
            per_act_token_quant=per_act_token_quant,
            block_shape=block_shape,
        )

        batched_output = naive_batched_moe(
            a,
            w1,
            w2,
            topk_weight,
            topk_ids,
            w1_scale=w1_s,
            w2_scale=w2_s,
            a1_scale=a1_scale,
            a2_scale=a2_scale,
            quant_dtype=quant_dtype,
            per_act_token_quant=per_act_token_quant,
            block_shape=block_shape,
        )

        triton_output = batched_moe(
            a,
            w1,
            w2,
            topk_weight,
            topk_ids,
            w1_scale=w1_s,
            w2_scale=w2_s,
            a1_scale=a1_scale,
            a2_scale=a2_scale,
            quant_dtype=quant_dtype,
            per_act_token_quant=per_act_token_quant,
            block_shape=block_shape,
        )

    torch.testing.assert_close(batched_output, baseline_output, atol=3e-2, rtol=2e-2)

    torch.testing.assert_close(triton_output, batched_output, atol=2e-2, rtol=2e-2)
```
**EN:** This pytest case verifies fused MoE batched experts. It is parameterized across 7 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as m, n, k, e. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 fused MoE batched experts 的行为。 它通过 7 组参数化输入覆盖多种场景；它会使用诸如 m、n、k、e 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `dataclasses -> dataclass`
- `pytest`
- `torch`
- `tests.kernels.moe.utils -> batched_moe, make_quantized_test_activations, make_test_weights, naive_batched_moe`
- `tests.kernels.quant_utils -> native_batched_masked_quant_matmul`
- `tests.kernels.utils -> torch_experts`
- `vllm.config -> VllmConfig, set_current_vllm_config`
- `vllm.model_executor.layers.fused_moe -> fused_topk`
- `vllm.model_executor.layers.fused_moe.experts.fused_batched_moe -> invoke_moe_batched_triton_kernel`
- `vllm.platforms -> current_platform`
- `vllm.triton_utils -> tl`
- `vllm.utils.torch_utils -> set_random_seed`
