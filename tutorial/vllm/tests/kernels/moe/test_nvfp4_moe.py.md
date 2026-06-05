# test_nvfp4_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_nvfp4_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_nvfp4_moe, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_nvfp4_moe 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-29)
```python
import pytest
import torch

import vllm.model_executor.layers.fused_moe.modular_kernel as mk
from tests.kernels.moe.utils import make_dummy_moe_config, make_test_weights
from tests.kernels.quantization.nvfp4_utils import (
    FLOAT4_E2M1_MAX,
    FLOAT8_E4M3_MAX,
    dequantize_nvfp4_to_dtype,
)
from tests.kernels.utils import torch_moe
from vllm import _custom_ops as ops
from vllm.config import ParallelConfig, VllmConfig, set_current_vllm_config
from vllm.model_executor.layers.fused_moe import fused_topk
from vllm.model_executor.layers.fused_moe.activation import MoEActivation
from vllm.model_executor.layers.fused_moe.all2all_utils import (
    maybe_make_prepare_finalize,
)
from vllm.model_executor.layers.fused_moe.config import nvfp4_moe_quant_config
from vllm.model_executor.layers.fused_moe.experts.cutlass_moe import (
    CutlassExpertsFp4,
)
from vllm.model_executor.layers.fused_moe.prepare_finalize import (
    make_moe_prepare_and_finalize_no_dp_ep,
)
from vllm.platforms import current_platform
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; shared test helpers from tests.kernels.moe.utils, tests.kernels.quantization.nvfp4_utils, tests.kernels.utils; and vLLM components like vllm.model_executor.layers.fused_moe.modular_kernel, vllm, vllm.config, vllm.model_executor.layers.fused_moe.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；共享测试辅助模块，例如 tests.kernels.moe.utils、tests.kernels.quantization.nvfp4_utils、tests.kernels.utils；vLLM 内部组件，例如 vllm.model_executor.layers.fused_moe.modular_kernel、vllm、vllm.config、vllm.model_executor.layers.fused_moe。

### Top-level block starting at line 31 (lines 31-34)
```python
if not current_platform.has_device_capability(100):
    pytest.skip(
        "Nvfp4 Requires compute capability of 10 or above.", allow_module_level=True
    )
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Constants and module state (lines 36-45)
```python
MNK_FACTORS = [
    (2, 1024, 1024),
    (2, 1024, 1536),
    (2, 3072, 1024),
    (64, 1024, 1024),
    (64, 3072, 1024),
    (64, 2048, 1536),
    (224, 1024, 1024),
    (224, 1024, 1536),
]
```
**EN:** This block centralizes shared constants and parameter grids, including MNK_FACTORS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 MNK_FACTORS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `test_cutlass_fp4_moe_no_graph` (lines 48-161)
```python
@pytest.mark.parametrize("m,n,k", MNK_FACTORS)
@pytest.mark.parametrize("e", [40, 64, 256])
@pytest.mark.parametrize("topk", [1, 6, 8])
@pytest.mark.parametrize("dtype", [torch.bfloat16])
@torch.inference_mode()
def test_cutlass_fp4_moe_no_graph(
    m: int, n: int, k: int, e: int, topk: int, dtype: torch.dtype, workspace_init
):
    set_random_seed(7)
    with set_current_vllm_config(
        VllmConfig(parallel_config=ParallelConfig(pipeline_parallel_size=1))
    ):
        quant_blocksize = 16

        a = torch.randn((m, k), device="cuda", dtype=dtype) / 10

        (_, w1_q, w1_blockscale, w1_gs), (_, w2_q, w2_blockscale, w2_gs) = (
            make_test_weights(
                e,
                n,
                k,
                in_dtype=dtype,
                quant_dtype="nvfp4",
                block_shape=None,  # use quant_blocksize?
                per_out_ch_quant=False,
            )
        )

        score = torch.randn((m, e), device="cuda", dtype=dtype)
        topk_weights, topk_ids, _ = fused_topk(a, score, topk, renormalize=False)

        a1_gs = torch.ones((e,), device="cuda", dtype=torch.float32)
        a2_gs = torch.ones((e,), device="cuda", dtype=torch.float32)

        assert w1_gs is not None
        assert w2_gs is not None
        assert w1_blockscale is not None
        assert w2_blockscale is not None

        quant_config = nvfp4_moe_quant_config(
# ... excerpt ...
        # Reference check:
        a_global_scale = (
            (FLOAT8_E4M3_MAX * FLOAT4_E2M1_MAX) / torch.amax(a.flatten(), dim=-1)
        ).to(torch.float32)
        a_fp4, a_scale_interleaved = ops.scaled_fp4_quant(a, a_global_scale)

        a_in_dtype = dequantize_nvfp4_to_dtype(
            a_fp4,
            a_scale_interleaved,
            a_global_scale,
            dtype=a.dtype,
            device=a.device,
            block_size=quant_blocksize,
        )

        w1_d = torch.empty((e, 2 * n, k), device="cuda", dtype=dtype)
        w2_d = torch.empty((e, k, n), device="cuda", dtype=dtype)

        for idx in range(0, e):
            w1_d[idx] = dequantize_nvfp4_to_dtype(
                w1_q[idx],
                w1_blockscale[idx],
                w1_gs[idx],
                dtype=dtype,
                device=w1_q.device,
                block_size=quant_blocksize,
            )
            w2_d[idx] = dequantize_nvfp4_to_dtype(
                w2_q[idx],
                w2_blockscale[idx],
                w2_gs[idx],
                dtype=dtype,
                device=w2_q.device,
                block_size=quant_blocksize,
            )

        torch_output = torch_moe(a_in_dtype, w1_d, w2_d, score, topk)

        torch.testing.assert_close(torch_output, cutlass_output, atol=1e-1, rtol=1e-1)
```
**EN:** This pytest case verifies cutlass fp4 MoE no graph. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as m, n, k, e. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 cutlass fp4 MoE no graph 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 m、n、k、e 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

### Constants and module state (lines 168-172)
```python
SWIGLUSTEP_MNK_FACTORS = [
    (2, 1280, 4096),
    (64, 1280, 4096),
    (224, 1280, 4096),
]
```
**EN:** This block centralizes shared constants and parameter grids, including SWIGLUSTEP_MNK_FACTORS. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 SWIGLUSTEP_MNK_FACTORS。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `test_cutlass_fp4_moe_swiglustep` (lines 175-289)
```python
@pytest.mark.parametrize("m,n,k", SWIGLUSTEP_MNK_FACTORS)
@pytest.mark.parametrize("e", [64, 288])
@pytest.mark.parametrize("topk", [1, 8])
@pytest.mark.parametrize("dtype", [torch.bfloat16])
@torch.inference_mode()
def test_cutlass_fp4_moe_swiglustep(
    m: int, n: int, k: int, e: int, topk: int, dtype: torch.dtype, workspace_init
):
    set_random_seed(7)
    with set_current_vllm_config(
        VllmConfig(parallel_config=ParallelConfig(pipeline_parallel_size=1))
    ):
        quant_blocksize = 16

        a = torch.randn((m, k), device="cuda", dtype=dtype) / 10

        (_, w1_q, w1_blockscale, w1_gs), (_, w2_q, w2_blockscale, w2_gs) = (
            make_test_weights(
                e,
                n,
                k,
                in_dtype=dtype,
                quant_dtype="nvfp4",
                block_shape=None,
                per_out_ch_quant=False,
            )
        )

        score = torch.randn((m, e), device="cuda", dtype=dtype)
        topk_weights, topk_ids, _ = fused_topk(a, score, topk, renormalize=False)

        a1_gs = torch.ones((e,), device="cuda", dtype=torch.float32)
        a2_gs = torch.ones((e,), device="cuda", dtype=torch.float32)

        assert w1_gs is not None
        assert w2_gs is not None
        assert w1_blockscale is not None
        assert w2_blockscale is not None

        quant_config = nvfp4_moe_quant_config(
# ... excerpt ...
            a_fp4,
            a_scale_interleaved,
            a_global_scale,
            dtype=a.dtype,
            device=a.device,
            block_size=quant_blocksize,
        )

        w1_d = torch.empty((e, 2 * n, k), device="cuda", dtype=dtype)
        w2_d = torch.empty((e, k, n), device="cuda", dtype=dtype)

        for idx in range(0, e):
            w1_d[idx] = dequantize_nvfp4_to_dtype(
                w1_q[idx],
                w1_blockscale[idx],
                w1_gs[idx],
                dtype=dtype,
                device=w1_q.device,
                block_size=quant_blocksize,
            )
            w2_d[idx] = dequantize_nvfp4_to_dtype(
                w2_q[idx],
                w2_blockscale[idx],
                w2_gs[idx],
                dtype=dtype,
                device=w2_q.device,
                block_size=quant_blocksize,
            )

        torch_output = torch_moe(
            a_in_dtype,
            w1_d,
            w2_d,
            score,
            topk,
            activation=MoEActivation.SWIGLUSTEP,
        )

        torch.testing.assert_close(torch_output, cutlass_output, atol=1e-1, rtol=1e-1)
```
**EN:** This pytest case verifies cutlass fp4 MoE swiglustep. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as m, n, k, e. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 cutlass fp4 MoE swiglustep 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 m、n、k、e 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

### Top-level block starting at line 292 (lines 292-293)
```python
if __name__ == "__main__":
    test_cutlass_fp4_moe_no_graph((2, 1024, 1024), 40, 1, torch.half)
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `vllm.model_executor.layers.fused_moe.modular_kernel`
- `tests.kernels.moe.utils -> make_dummy_moe_config, make_test_weights`
- `tests.kernels.quantization.nvfp4_utils -> FLOAT4_E2M1_MAX, FLOAT8_E4M3_MAX, dequantize_nvfp4_to_dtype`
- `tests.kernels.utils -> torch_moe`
- `vllm -> _custom_ops`
- `vllm.config -> ParallelConfig, VllmConfig, set_current_vllm_config`
- `vllm.model_executor.layers.fused_moe -> fused_topk`
- `vllm.model_executor.layers.fused_moe.activation -> MoEActivation`
- `vllm.model_executor.layers.fused_moe.all2all_utils -> maybe_make_prepare_finalize`
- `vllm.model_executor.layers.fused_moe.config -> nvfp4_moe_quant_config`
- `vllm.model_executor.layers.fused_moe.experts.cutlass_moe -> CutlassExpertsFp4`
- `vllm.model_executor.layers.fused_moe.prepare_finalize -> make_moe_prepare_and_finalize_no_dp_ep`
- `vllm.platforms -> current_platform`
- `vllm.utils.torch_utils -> set_random_seed`
