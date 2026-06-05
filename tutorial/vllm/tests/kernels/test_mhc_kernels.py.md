# test_mhc_kernels.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/test_mhc_kernels.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / test_mhc_kernels, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / test_mhc_kernels 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 3-8)
```python
import pytest
import torch

import vllm.model_executor.kernels.mhc  # noqa: F401
from vllm.platforms import current_platform
from vllm.utils.torch_utils import set_random_seed
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; and vLLM components like vllm.model_executor.kernels.mhc  # noqa: F401, vllm.platforms, vllm.utils.torch_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；vLLM 内部组件，例如 vllm.model_executor.kernels.mhc  # noqa: F401、vllm.platforms、vllm.utils.torch_utils。

### Constants and module state (lines 10-10)
```python
DEVICE = current_platform.device_type
```
**EN:** This block centralizes shared constants and parameter grids, including DEVICE. Those values keep later pytest scenarios consistent and make the test matrix easier to audit.
**CN:** 该代码块集中定义共享常量与参数网格，例如 DEVICE。这些值让后续 pytest 场景保持一致，也便于检查测试矩阵是否完整。

### Function `sinkhorn_normalize_ref` (lines 13-19)
```python
def sinkhorn_normalize_ref(x: torch.Tensor, repeat: int, eps: float) -> torch.Tensor:
    x = x.softmax(-1) + eps
    x = x / (x.sum(-2, keepdim=True) + eps)
    for _ in range(repeat - 1):
        x = x / (x.sum(-1, keepdim=True) + eps)
        x = x / (x.sum(-2, keepdim=True) + eps)
    return x
```
**EN:** This helper function implements the shared logic for sinkhorn normalize ref. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 sinkhorn normalize ref 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `mhc_pre_ref` (lines 22-63)
```python
def mhc_pre_ref(
    residual: torch.Tensor,
    fn: torch.Tensor,
    hc_scale: torch.Tensor,
    hc_base: torch.Tensor,
    rms_eps: float,
    hc_pre_eps: float,
    hc_sinkhorn_eps: float,
    hc_post_mult_value: float,
    sinkhorn_repeat: int,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    """mHC pre reference kernel from tilelang repo: https://github.com/tile-ai/tilelang/blob/d135bd1cd2d2eee74fbb41dd0a0831a427194c86/examples/deepseek_mhc/example_mhc_pre.py#L303"""
    hc_mult = residual.shape[-2]

    residual_flat = residual.flatten(-2, -1).float()
    sqrsum = residual_flat.square().sum(-1)
    mixes = (
        residual_flat @ fn.T * (sqrsum.unsqueeze(-1) / fn.shape[-1] + rms_eps).rsqrt()
    )

    hc_scale = torch.cat(
        [
            hc_scale[0].expand(hc_mult),
            hc_scale[1].expand(hc_mult),
            hc_scale[2].expand(hc_mult * hc_mult),
        ],
    )
    mixes = mixes * hc_scale + hc_base

    pre_mix = mixes[:, :hc_mult].sigmoid().unsqueeze(-1) + hc_pre_eps
    post_mix = (
        mixes[:, hc_mult : 2 * hc_mult].sigmoid() * hc_post_mult_value
    ).unsqueeze(-1)
    res_mix = mixes[:, 2 * hc_mult :].view(-1, hc_mult, hc_mult)

    res_mix = sinkhorn_normalize_ref(
        res_mix, repeat=sinkhorn_repeat, eps=hc_sinkhorn_eps
    )

    layer_input = (residual * pre_mix).sum(-2).bfloat16()

    return post_mix, res_mix, layer_input
```
**EN:** This helper function implements the shared logic for mhc pre ref. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 mhc pre ref 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `mhc_post_ref` (lines 66-74)
```python
def mhc_post_ref(
    x: torch.Tensor,
    residual: torch.Tensor,
    post_layer_mix: torch.Tensor,
    comb_res_mix: torch.Tensor,
) -> torch.Tensor:
    """mHC post reference kernel from tilelang repo: https://github.com/tile-ai/tilelang/blob/d135bd1cd2d2eee74fbb41dd0a0831a427194c86/examples/deepseek_mhc/example_mhc_post.py#L68"""
    term2 = torch.bmm(comb_res_mix.mT, residual.float())
    return (x.float().unsqueeze(-2) * post_layer_mix + term2).bfloat16()
```
**EN:** This helper function implements the shared logic for mhc post ref. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 mhc post ref 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `hc_head_ref` (lines 77-91)
```python
def hc_head_ref(
    residual: torch.Tensor,
    fn: torch.Tensor,
    hc_scale: torch.Tensor,
    hc_base: torch.Tensor,
    rms_eps: float,
    hc_eps: float,
) -> torch.Tensor:
    residual_flat = residual.flatten(-2).float()
    residual_norm = residual_flat * torch.rsqrt(
        residual_flat.square().mean(dim=-1, keepdim=True) + rms_eps
    )
    pre_mix = torch.nn.functional.linear(residual_norm, fn)
    pre_mix = torch.sigmoid(pre_mix * hc_scale + hc_base) + hc_eps
    return torch.sum(pre_mix.unsqueeze(-1) * residual.float(), dim=-2).bfloat16()
```
**EN:** This helper function implements the shared logic for hc head ref. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 hc head ref 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_mhc_fused_post_pre` (lines 94-159)
```python
@pytest.mark.skipif(
    not current_platform.is_cuda(),
    reason="CUDA required",
)
@pytest.mark.parametrize("num_tokens", [1, 4, 8, 128])
@pytest.mark.parametrize("hidden_size", [4096, 7168])
@pytest.mark.parametrize("hc_mult", [4])
def test_mhc_fused_post_pre(num_tokens, hidden_size, hc_mult):
    torch.set_default_device(DEVICE)
    set_random_seed(0)

    x = torch.randn((num_tokens, hidden_size), dtype=torch.bfloat16)
    residual = torch.randn((num_tokens, hc_mult, hidden_size), dtype=torch.bfloat16)
    post_layer_mix = torch.randn((num_tokens, hc_mult, 1), dtype=torch.float32)
    comb_res_mix = torch.randn((num_tokens, hc_mult, hc_mult), dtype=torch.float32)

    hc_mult2 = hc_mult * hc_mult
    hc_mult3 = hc_mult * 2 + hc_mult2
    fn = (
        torch.randn((hc_mult3, hc_mult, hidden_size), dtype=torch.float)
        * 1e-4
        * (1 + torch.arange(hc_mult).mul(0.01).view(1, -1, 1))
    ).flatten(1, 2)
    hc_scale = torch.randn((3,), dtype=torch.float) * 0.1
    hc_base = torch.randn((hc_mult3,), dtype=torch.float) * 0.1

    hc_sinkhorn_eps = hc_pre_eps = rms_eps = 1e-6
    sinkhorn_repeat = 20
    hc_post_alpha = 1.0

    def run_ref():
        residual_ref = mhc_post_ref(x, residual, post_layer_mix, comb_res_mix)
        post_mix_ref, res_mix_ref, layer_input_ref = mhc_pre_ref(
            residual_ref,
            fn,
            hc_scale,
            hc_base,
            rms_eps,
            hc_pre_eps,
            hc_sinkhorn_eps,
            hc_post_alpha,
            sinkhorn_repeat,
        )
        return residual_ref, post_mix_ref, res_mix_ref, layer_input_ref

    residual_ref, post_mix_ref, res_mix_ref, layer_input_ref = run_ref()

    residual, post_mix, res_mix, x = torch.ops.vllm.mhc_fused_post_pre_tilelang(
        x,
        residual,
        post_layer_mix,
        comb_res_mix,
        fn,
        hc_scale,
        hc_base,
        rms_eps,
        hc_pre_eps,
        hc_sinkhorn_eps,
        hc_post_alpha,
        sinkhorn_repeat,
    )

    torch.testing.assert_close(residual, residual_ref, atol=1e-2, rtol=1e-2)
    torch.testing.assert_close(post_mix, post_mix_ref, atol=1e-2, rtol=1e-2)
    torch.testing.assert_close(res_mix, res_mix_ref, atol=1e-2, rtol=1e-2)
    torch.testing.assert_close(x, layer_input_ref, atol=1e-2, rtol=1e-2)
```
**EN:** This pytest case verifies mhc fused post pre. It is parameterized across 3 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens, hidden_size, hc_mult. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 mhc fused post pre 的行为。 它通过 3 组参数化输入覆盖多种场景；它会使用诸如 num_tokens、hidden_size、hc_mult 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

### Function `test_hc_head_triton` (lines 162-198)
```python
@pytest.mark.skipif(
    not current_platform.is_rocm(),
    reason="ROCm required",
)
@pytest.mark.parametrize("num_tokens", [1, 4, 8, 128])
@pytest.mark.parametrize("hidden_size", [4096, 7168])
@pytest.mark.parametrize("hc_mult", [4])
def test_hc_head_triton(num_tokens, hidden_size, hc_mult):
    torch.set_default_device(DEVICE)
    set_random_seed(0)

    residual = torch.randn((num_tokens, hc_mult, hidden_size), dtype=torch.bfloat16)
    fn = torch.randn((hc_mult, hc_mult * hidden_size), dtype=torch.float32) * 1e-4
    hc_scale = torch.randn((1,), dtype=torch.float32) * 0.1
    hc_base = torch.randn((hc_mult,), dtype=torch.float32) * 0.1
    rms_eps = hc_eps = 1e-6

    out = torch.empty((num_tokens, hidden_size), dtype=torch.bfloat16)
    out.fill_(float("nan"))

    result = torch.ops.vllm.hc_head_triton(
        residual,
        fn,
        hc_scale,
        hc_base,
        out,
        hidden_size,
        rms_eps,
        hc_eps,
        hc_mult,
    )

    assert result is None
    assert not torch.isnan(out).any()

    out_ref = hc_head_ref(residual, fn, hc_scale, hc_base, rms_eps, hc_eps)
    torch.testing.assert_close(out, out_ref, atol=5e-2, rtol=1e-2)
```
**EN:** This pytest case verifies hc head triton. It is parameterized across 3 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens, hidden_size, hc_mult. unsupported hardware, backend, or configuration combinations are skipped early. numeric results are compared against a reference with explicit tolerances.
**CN:** 该 pytest 用例验证 hc head triton 的行为。 它通过 3 组参数化输入覆盖多种场景；它会使用诸如 num_tokens、hidden_size、hc_mult 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；数值结果会在显式容差下与参考结果进行比较。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `vllm.model_executor.kernels.mhc`
- `vllm.platforms -> current_platform`
- `vllm.utils.torch_utils -> set_random_seed`
