# test_block_int8.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_block_int8.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_block_int8, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_block_int8 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-15)
```python
import pytest
import torch

from tests.kernels.moe.utils import make_test_quant_config
from tests.kernels.quant_utils import (
    native_per_token_group_quant_int8,
    native_w8a8_block_matmul,
)
from vllm.config import VllmConfig, set_current_vllm_config
from vllm.model_executor.layers.activation import SiluAndMul
from vllm.model_executor.layers.fused_moe import fused_experts, fused_topk
from vllm.platforms import current_platform
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; shared test helpers from tests.kernels.moe.utils, tests.kernels.quant_utils; and vLLM components like vllm.config, vllm.model_executor.layers.activation, vllm.model_executor.layers.fused_moe, vllm.platforms.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；共享测试辅助模块，例如 tests.kernels.moe.utils、tests.kernels.quant_utils；vLLM 内部组件，例如 vllm.config、vllm.model_executor.layers.activation、vllm.model_executor.layers.fused_moe、vllm.platforms。

### Top-level block starting at line 17 (lines 17-18)
```python
if current_platform.get_device_capability() < (7, 0):
    pytest.skip("INT8 Triton requires CUDA 7.0 or higher", allow_module_level=True)
```
**EN:** This top-level `If` block performs supporting work that the surrounding tests depend on.
**CN:** 这个顶层 `If` 代码块执行周边测试所依赖的辅助逻辑。

### Constants and module state (lines 20-51)
```python
vllm_config = VllmConfig()

DTYPES = [torch.bfloat16]

MNK_FACTORS = [
    (1, 128, 128),
    (1, 128, 7168),
    (1, 1024, 7168),
    (1, 4096, 512),
    (1, 4096, 7168),
    (33, 512, 512),
    (33, 128, 7168),
    (33, 1024, 7168),
    (33, 4096, 128),
    (33, 4096, 7168),
    (128, 128, 128),
    (128, 1024, 7168),
    (128, 4096, 512),
    (128, 4096, 7168),
    (222, 512, 512),
    (222, 1024, 7168),
    (222, 4096, 7168),
    (2048, 128, 128),
    (2048, 1024, 7168),
    (2048, 4096, 4096),
]

E = [8, 24]
TOP_KS = [2, 6]
# BLOCK_SIZE = [[64, 64], [64, 128], [128, 64], [128, 128]]
BLOCK_SIZE = [[128, 128]]
SEEDS = [0]
```
**EN:** This block centralizes shared constants and parameter grids, including DTYPES, MNK_FACTORS, E, TOP_KS, BLOCK_SIZE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 DTYPES、MNK_FACTORS、E、TOP_KS、BLOCK_SIZE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `torch_w8a8_block_int8_moe` (lines 55-82)
```python
def torch_w8a8_block_int8_moe(a, w1, w2, w1_s, w2_s, score, topk, block_shape):
    """This function performs fused moe with block-wise quantization using
    native torch."""
    B, D = a.shape
    a = a.view(B, -1, D).repeat(1, topk, 1).reshape(-1, D)
    out = torch.zeros(B * topk, w2.shape[1], dtype=a.dtype, device=a.device)
    score = torch.softmax(score, dim=-1, dtype=torch.float32)
    topk_weight, topk_ids = torch.topk(score, topk)
    topk_weight = topk_weight.view(-1)
    topk_ids = topk_ids.view(-1)

    _, block_k = block_shape[0], block_shape[1]
    a_q, a_s = native_per_token_group_quant_int8(a, block_k)
    for i in range(w1.shape[0]):
        mask = topk_ids == i
        if mask.sum():
            inter_out = native_w8a8_block_matmul(
                a_q[mask], w1[i], a_s[mask], w1_s[i], block_shape, output_dtype=a.dtype
            )
            act_out = SiluAndMul().forward_native(inter_out)
            act_out_q, act_out_s = native_per_token_group_quant_int8(act_out, block_k)
            act_out = act_out.to(torch.float32)
            out[mask] = native_w8a8_block_matmul(
                act_out_q, w2[i], act_out_s, w2_s[i], block_shape, output_dtype=a.dtype
            )
    return (
        out.view(B, -1, w2.shape[1]) * topk_weight.view(B, -1, 1).to(out.dtype)
    ).sum(dim=1)
```
**EN:** This helper function implements the shared logic for torch w8a8 block int8 MoE. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 torch w8a8 block int8 MoE 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `setup_cuda` (lines 85-88)
```python
@pytest.fixture(autouse=True, scope="module")
def setup_cuda():
    """Sets the default CUDA device for all tests in this module."""
    torch.set_default_device("cuda")
```
**EN:** This fixture prepares reusable state for setup CUDA. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该 fixture 为 setup CUDA 准备可复用的测试状态。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_w8a8_block_int8_fused_moe` (lines 91-134)
```python
@pytest.mark.parametrize(("M", "N", "K"), MNK_FACTORS)
@pytest.mark.parametrize("E", E)
@pytest.mark.parametrize("topk", TOP_KS)
@pytest.mark.parametrize("block_size", BLOCK_SIZE)
@pytest.mark.parametrize("dtype", DTYPES)
@pytest.mark.parametrize("seed", SEEDS)
@torch.inference_mode()
def test_w8a8_block_int8_fused_moe(M, N, K, E, topk, block_size, dtype, seed):
    """Tests the fused_moe kernel with W8A8 INT8 block quantization against a
    native torch reference."""
    torch.manual_seed(seed)

    a = torch.randn((M, K), dtype=dtype) / 10
    score = torch.randn((M, E), dtype=dtype)
    topk_weights, topk_ids, _ = fused_topk(a, score.float(), topk, False)

    w1, w2, quant_config = make_test_quant_config(
        E,
        N,
        K,
        dtype,
        quant_dtype=torch.int8,
        per_act_token_quant=False,
        block_shape=block_size,
    )

    # Set the context to avoid lots of warning spam.
    with set_current_vllm_config(vllm_config):
        out = fused_experts(
            a, w1, w2, topk_weights, topk_ids, quant_config=quant_config
        )
        ref_out = torch_w8a8_block_int8_moe(
            a,
            w1,
            w2,
            quant_config.w1_scale,
            quant_config.w2_scale,
            score,
            topk,
            block_size,
        )

    # Check results
    torch.testing.assert_close(out, ref_out, atol=0.065, rtol=0.065)
```
**EN:** This pytest case verifies w8a8 block int8 fused MoE. It is parameterized across 6 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as M, N, K, E. numeric results are compared against a reference with explicit tolerances. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 w8a8 block int8 fused MoE 的行为。 它通过 6 组参数化输入覆盖多种场景；它会使用诸如 M、N、K、E 等 fixture 或输入；数值结果会在显式容差下与参考结果进行比较；代码会控制随机性以保证场景可复现。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。
- **Reusable fixtures / 可复用 fixture:** Fixtures package common setup so each test can focus on the scenario under study. / fixture 封装公共准备逻辑，让每个测试更专注于待验证场景。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `tests.kernels.moe.utils -> make_test_quant_config`
- `tests.kernels.quant_utils -> native_per_token_group_quant_int8, native_w8a8_block_matmul`
- `vllm.config -> VllmConfig, set_current_vllm_config`
- `vllm.model_executor.layers.activation -> SiluAndMul`
- `vllm.model_executor.layers.fused_moe -> fused_experts, fused_topk`
- `vllm.platforms -> current_platform`
