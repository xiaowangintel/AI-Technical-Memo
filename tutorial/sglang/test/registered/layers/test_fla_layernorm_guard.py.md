# test_fla_layernorm_guard.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/layers/test_fla_layernorm_guard.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates fla layernorm guard behavior in SGLang's layers area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 层 领域中与 fla layernorm guard 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18: module imports and dependencies / 模块导入与依赖
```python
from __future__ import annotations

import socket
import sys
from dataclasses import dataclass

import pytest
import torch
import torch.nn.functional as F

from sglang.srt.layers.attention.fla.layernorm_gated import (
    _layer_norm_fwd as layer_norm_fwd,
)
from sglang.srt.layers.attention.fla.layernorm_gated import (
    layernorm_fn,
    rms_norm_ref,
)
from sglang.test.ci.ci_register import register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `__future__`, `socket`, `sys`, `dataclasses`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `__future__`, `socket`, `sys`, `dataclasses`。

### Lines 20-37: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(
    est_time=60,
    stage="base-b",
    runner_config="2-gpu-large",
    disabled="Temporarily disabled",
)

# Optional dependency in sglang repo; skip collection cleanly if absent.
custom_all_reduce_utils = pytest.importorskip(
    "sglang.srt.distributed.device_communicators.custom_all_reduce_utils"
)
parallel_state = pytest.importorskip("sglang.srt.distributed.parallel_state")

update_environment_variables = custom_all_reduce_utils.update_environment_variables
init_distributed_environment = parallel_state.init_distributed_environment
initialize_model_parallel = parallel_state.initialize_model_parallel

NUM_GPUS = 2
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, importorskip.
**CN:** 该代码块通过 register_cuda_ci, importorskip 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 40-45: function find free port / 函数 find free port
```python
def _find_free_port() -> int:
    # Avoid hard-coded port collisions when pytest runs tests in parallel.
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        s.bind(("localhost", 0))
        s.listen(1)
        return int(s.getsockname()[1])
```
**EN:** This block implements `_find_free_port` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_find_free_port`，承担模块行为中的一个聚焦逻辑片段。

### Lines 48-52: function skip if no cuda or not enough gpus / 函数 skip if no cuda or not enough gpus
```python
def _skip_if_no_cuda_or_not_enough_gpus(required_gpus: int = NUM_GPUS) -> None:
    if not torch.cuda.is_available():
        pytest.skip("CUDA device not available")
    if torch.cuda.device_count() < required_gpus:
        pytest.skip(f"Need >= {required_gpus} GPUs, got {torch.cuda.device_count()}")
```
**EN:** This block implements `_skip_if_no_cuda_or_not_enough_gpus` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_skip_if_no_cuda_or_not_enough_gpus`，承担模块行为中的一个聚焦逻辑片段。

### Lines 55-57: function skip if dtype unsupported / 函数 skip if dtype unsupported
```python
def _skip_if_dtype_unsupported(dtype: torch.dtype) -> None:
    if dtype is torch.bfloat16 and not torch.cuda.is_bf16_supported():
        pytest.skip("bfloat16 not supported on this CUDA device")
```
**EN:** This block implements `_skip_if_dtype_unsupported` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_skip_if_dtype_unsupported`，承担模块行为中的一个聚焦逻辑片段。

### Lines 60-93: function setup sglang distributed / 函数 setup sglang distributed
```python
def _setup_sglang_distributed(
    local_rank: int,
    world_size: int,
    master_port: int,
    dtype: torch.dtype,
) -> torch.device:
    # Match sglang test style: set per-rank CUDA device + default dtype/device.
    torch.manual_seed(0)
    torch.cuda.manual_seed_all(0)

    device = torch.device(f"cuda:{local_rank}")
    torch.cuda.set_device(device)

    if hasattr(torch, "set_default_device"):
        torch.set_default_device(device)
    if hasattr(torch, "set_default_dtype"):
        torch.set_default_dtype(dtype)

    update_environment_variables(
        {
            "RANK": str(local_rank),
            "LOCAL_RANK": str(local_rank),
            "WORLD_SIZE": str(world_size),
            "MASTER_ADDR": "localhost",
            "MASTER_PORT": str(master_port),
        }
    )

    init_distributed_environment(
        world_size=world_size, rank=local_rank, local_rank=local_rank
    )
    initialize_model_parallel(tensor_model_parallel_size=world_size)

    return device
```
**EN:** This block implements `_setup_sglang_distributed` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_setup_sglang_distributed`，承担模块行为中的一个聚焦逻辑片段。

### Lines 96-151: function layer norm ref / 函数 layer norm ref
```python
def layer_norm_ref(
    x: torch.Tensor,
    weight: torch.Tensor,
    bias: torch.Tensor | None,
    z: torch.Tensor | None = None,
    eps: float = 1e-6,
    group_size: int | None = None,
    norm_before_gate: bool = True,
    is_rms_norm: bool = False,
) -> torch.Tensor:
    """Reference implementation for both LayerNorm and RMSNorm (supports optional gate + group norm)."""
    if is_rms_norm:
        return rms_norm_ref(
            x,
            weight,
            bias,
            z=z,
            eps=eps,
            group_size=group_size,
            norm_before_gate=norm_before_gate,
            upcast=True,
        )

    dtype = x.dtype
    x_f = x.float()
    w_f = weight.float()
    b_f = bias.float() if bias is not None else None
    z_f = z.float() if z is not None else None

    if z_f is not None and not norm_before_gate:
        x_f = x_f * F.silu(z_f)

    if group_size is None:
        mean = x_f.mean(dim=-1, keepdim=True)
        var = (x_f - mean).square().mean(dim=-1, keepdim=True)
        rstd = torch.rsqrt(var + eps)
        out = (x_f - mean) * rstd * w_f
        if b_f is not None:
            out = out + b_f
    else:
        hidden = x_f.shape[-1]
        assert hidden % group_size == 0
        ng = hidden // group_size
        xg = x_f.view(*x_f.shape[:-1], ng, group_size)
        mean = xg.mean(dim=-1, keepdim=True)
        var = (xg - mean).square().mean(dim=-1, keepdim=True)
        rstd = torch.rsqrt(var + eps)
        xg = (xg - mean) * rstd
        out = xg.reshape(*x_f.shape[:-1], hidden) * w_f
        if b_f is not None:
            out = out + b_f

    if z_f is not None and norm_before_gate:
        out = out * F.silu(z_f)

    return out.to(dtype)
```
**EN:** Reference implementation for both LayerNorm and RMSNorm (supports optional gate + group norm). This block implements `layer_norm_ref` and captures one focused piece of the module's behavior.
**CN:** Reference implementation for both LayerNorm and RMSNorm (supports optional gate + group norm). 该代码块实现 `layer_norm_ref`，承担模块行为中的一个聚焦逻辑片段。

### Lines 154-155: class FwdCase declaration / 类 FwdCase 声明
```python
@dataclass(frozen=True)
class FwdCase:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 156-160: class-level constants and configuration for `FwdCase` / 类级常量与配置
```python
    name: str
    with_gate: bool
    norm_before_gate: bool
    group_size: int | None
    is_rms_norm: bool
```
**EN:** This block defines shared names such as `name`, `with_gate`, `norm_before_gate`, `group_size`, `is_rms_norm`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `name`, `with_gate`, `norm_before_gate`, `group_size`, `is_rms_norm` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 163-213: module-level constants and configuration / 模块级常量与配置
```python
CASES: list[FwdCase] = [
    FwdCase(
        "layernorm",
        with_gate=False,
        norm_before_gate=True,
        group_size=None,
        is_rms_norm=False,
    ),
    FwdCase(
        "rmsnorm",
        with_gate=False,
        norm_before_gate=True,
        group_size=None,
        is_rms_norm=True,
    ),
    FwdCase(
        "layernorm_gate_pre",
        with_gate=True,
        norm_before_gate=True,
        group_size=None,
        is_rms_norm=False,
    ),
    FwdCase(
        "layernorm_gate_post",
        with_gate=True,
        norm_before_gate=False,
        group_size=None,
        is_rms_norm=False,
    ),
    FwdCase(
        "rmsnorm_gate_pre",
        with_gate=True,
        norm_before_gate=True,
        group_size=None,
        is_rms_norm=True,
    ),
    FwdCase(
        "group_layernorm",
        with_gate=False,
        norm_before_gate=True,
        group_size=128,
        is_rms_norm=False,
    ),
    FwdCase(
        "group_rmsnorm",
        with_gate=False,
        norm_before_gate=True,
        group_size=128,
        is_rms_norm=True,
    ),
]
```
**EN:** This block defines shared names such as `CASES`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `CASES` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 216-251: test case layernorm guard fwd spawn / 测试用例 layernorm guard fwd spawn
```python
@pytest.mark.parametrize("num_tokens", [128])
@pytest.mark.parametrize("hidden_size", [256])
@pytest.mark.parametrize("dtype", [torch.bfloat16])
@pytest.mark.parametrize("case", CASES, ids=lambda c: c.name)
def test_layernorm_guard_fwd_spawn(
    num_tokens: int,
    hidden_size: int,
    dtype: torch.dtype,
    case: FwdCase,
    device: str = "cuda",
):
    _skip_if_no_cuda_or_not_enough_gpus(NUM_GPUS)
    _skip_if_dtype_unsupported(dtype)

    if case.group_size is not None and hidden_size % case.group_size != 0:
        pytest.skip(
            f"hidden_size {hidden_size} not divisible by group_size {case.group_size}"
        )

    master_port = _find_free_port()
    world_size = NUM_GPUS

    torch.multiprocessing.spawn(
        _layernorm_guard_fwd_worker,
        args=(
            world_size,
            master_port,
            num_tokens,
            hidden_size,
            dtype,
            case,
            device,
        ),
        nprocs=world_size,
        join=True,
    )
```
**EN:** This test exercises `test_layernorm_guard_fwd_spawn` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_layernorm_guard_fwd_spawn`。

### Lines 254-319: function layernorm guard fwd worker / 函数 layernorm guard fwd worker
```python
def _layernorm_guard_fwd_worker(
    local_rank: int,
    world_size: int,
    master_port: int,
    num_tokens: int,
    hidden_size: int,
    dtype: torch.dtype,
    case: FwdCase,
    device: str,
):
    device = _setup_sglang_distributed(local_rank, world_size, master_port, dtype)

    with torch.inference_mode():
        torch.manual_seed(42 + local_rank)
        torch.cuda.manual_seed_all(42 + local_rank)

        x = torch.randn(num_tokens, hidden_size, dtype=dtype, device=device)
        z = (
            torch.randn(num_tokens, hidden_size, dtype=dtype, device=device)
            if case.with_gate
            else None
        )
        weight = torch.randn(hidden_size, dtype=dtype, device=device)
        bias = (
            None
            if case.is_rms_norm
            else torch.randn(hidden_size, dtype=dtype, device=device)
        )
        eps = 1e-6

        out, mean, rstd = layer_norm_fwd(
            x,
            weight,
            bias,
            eps,
            z=z,
            group_size=case.group_size,
            norm_before_gate=case.norm_before_gate,
            is_rms_norm=case.is_rms_norm,
        )

        ref_out = layer_norm_ref(
            x,
            weight,
            bias,
            z=z,
            eps=eps,
            group_size=case.group_size,
            norm_before_gate=case.norm_before_gate,
            is_rms_norm=case.is_rms_norm,
        )

        assert out.shape == x.shape
        assert out.dtype == x.dtype
        torch.testing.assert_close(out, ref_out, atol=1e-2, rtol=1e-2)

        # mean/rstd shape checks (same spirit as original vLLM tests)
        if case.group_size is None:
            if not case.is_rms_norm:
                assert mean.shape == (num_tokens,)
            assert rstd.shape == (num_tokens,)
        else:
            ngroups = hidden_size // case.group_size
            if not case.is_rms_norm:
                assert mean.shape == (ngroups * num_tokens,)
            assert rstd.shape == (ngroups * num_tokens,)
```
**EN:** This block implements `_layernorm_guard_fwd_worker` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_layernorm_guard_fwd_worker`，承担模块行为中的一个聚焦逻辑片段。

### Lines 322-335: test case layernorm guard misc spawn / 测试用例 layernorm guard misc spawn
```python
@pytest.mark.parametrize("dtype", [torch.bfloat16])
def test_layernorm_guard_misc_spawn(dtype: torch.dtype, device: str = "cuda"):
    _skip_if_no_cuda_or_not_enough_gpus(NUM_GPUS)
    _skip_if_dtype_unsupported(dtype)

    master_port = _find_free_port()
    world_size = NUM_GPUS

    torch.multiprocessing.spawn(
        _layernorm_guard_misc_worker,
        args=(world_size, master_port, dtype, device),
        nprocs=world_size,
        join=True,
    )
```
**EN:** This test exercises `test_layernorm_guard_misc_spawn` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_layernorm_guard_misc_spawn`。

### Lines 338-392: function layernorm guard misc worker / 函数 layernorm guard misc worker
```python
def _layernorm_guard_misc_worker(
    local_rank: int,
    world_size: int,
    master_port: int,
    dtype: torch.dtype,
    device: str,
):
    device = _setup_sglang_distributed(local_rank, world_size, master_port, dtype)

    with torch.inference_mode():
        torch.manual_seed(123 + local_rank)
        torch.cuda.manual_seed_all(123 + local_rank)

        # 1) rows_per_block-like sizes
        hidden_size = 1024
        weight = torch.randn(hidden_size, dtype=dtype, device=device)
        bias = torch.randn(hidden_size, dtype=dtype, device=device)
        eps = 1e-6
        for num_tokens in [513]:
            x = torch.randn(num_tokens, hidden_size, dtype=dtype, device=device)
            out, _, _ = layer_norm_fwd(x, weight, bias, eps, z=None, is_rms_norm=False)
            ref = layer_norm_ref(x, weight, bias, z=None, eps=eps, is_rms_norm=False)
            torch.testing.assert_close(out, ref, atol=1e-2, rtol=1e-2)

        # 2) strided input (slice then contiguous)
        num_tokens = 128
        x_large = torch.randn(num_tokens, hidden_size * 2, dtype=dtype, device=device)
        x = x_large[:, :hidden_size]
        x_contig = x.contiguous()
        out, _, _ = layer_norm_fwd(
            x_contig, weight, bias, eps, z=None, is_rms_norm=False
        )
        ref = layer_norm_ref(x_contig, weight, bias, z=None, eps=eps, is_rms_norm=False)
        torch.testing.assert_close(out, ref, atol=1e-2, rtol=1e-2)

        # 3) provided output buffer
        num_tokens = 256
        x = torch.randn(num_tokens, hidden_size, dtype=dtype, device=device)
        out_buf = torch.empty_like(x)
        out, _, _ = layer_norm_fwd(
            x, weight, bias, eps, z=None, out=out_buf, is_rms_norm=False
        )
        assert out.data_ptr() == out_buf.data_ptr()
        ref = layer_norm_ref(x, weight, bias, z=None, eps=eps, is_rms_norm=False)
        torch.testing.assert_close(out, ref, atol=1e-2, rtol=1e-2)

        # 4) multidimensional input via autograd fn
        for shape in [(4, 16, 1024)]:
            hs = shape[-1]
            x = torch.randn(*shape, dtype=dtype, device=device)
            w = torch.randn(hs, dtype=dtype, device=device)
            b = torch.randn(hs, dtype=dtype, device=device)
            out = layernorm_fn(x, w, b, z=None, eps=eps)
            ref = layer_norm_ref(x, w, b, z=None, eps=eps, is_rms_norm=False)
            torch.testing.assert_close(out, ref, atol=1e-2, rtol=1e-2)
```
**EN:** This block implements `_layernorm_guard_misc_worker` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_layernorm_guard_misc_worker`，承担模块行为中的一个聚焦逻辑片段。

### Lines 395-396: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_find_free_port`: This block implements `_find_free_port` and captures one focused piece of the module's behavior. / 该代码块实现 `_find_free_port`，承担模块行为中的一个聚焦逻辑片段。
- `_skip_if_no_cuda_or_not_enough_gpus`: This block implements `_skip_if_no_cuda_or_not_enough_gpus` and captures one focused piece of the module's behavior. / 该代码块实现 `_skip_if_no_cuda_or_not_enough_gpus`，承担模块行为中的一个聚焦逻辑片段。
- `_skip_if_dtype_unsupported`: This block implements `_skip_if_dtype_unsupported` and captures one focused piece of the module's behavior. / 该代码块实现 `_skip_if_dtype_unsupported`，承担模块行为中的一个聚焦逻辑片段。
- `_setup_sglang_distributed`: This block implements `_setup_sglang_distributed` and captures one focused piece of the module's behavior. / 该代码块实现 `_setup_sglang_distributed`，承担模块行为中的一个聚焦逻辑片段。
- `layer_norm_ref`: Reference implementation for both LayerNorm and RMSNorm (supports optional gate + group norm). / 该代码块实现 `layer_norm_ref`，承担模块行为中的一个聚焦逻辑片段。
- `FwdCase`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `test_layernorm_guard_fwd_spawn`: This test exercises `test_layernorm_guard_fwd_spawn` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_layernorm_guard_fwd_spawn`。
- `_layernorm_guard_fwd_worker`: This block implements `_layernorm_guard_fwd_worker` and captures one focused piece of the module's behavior. / 该代码块实现 `_layernorm_guard_fwd_worker`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `socket`, `sys`, `dataclasses`
- **Third-party modules / 第三方模块**: `pytest`, `torch`, `torch.nn.functional`
- **Internal modules / 内部模块**: `sglang.srt.layers.attention.fla.layernorm_gated`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 396
