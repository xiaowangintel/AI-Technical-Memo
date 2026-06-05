# test_mamba_ssm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/mamba/test_mamba_ssm.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / mamba / test_mamba_ssm, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / mamba / test_mamba_ssm 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-17)
```python
import pytest
import torch
import torch.nn.functional as F
from einops import rearrange, repeat

from tests.kernels.utils import opcheck
from vllm import _custom_ops as ops  # noqa: F401
from vllm.model_executor.layers.mamba.ops.mamba_ssm import (
    selective_scan_fn,
    selective_state_update,
)
from vllm.platforms import current_platform
from vllm.utils.torch_utils import set_random_seed
from vllm.v1.attention.backends.utils import NULL_BLOCK_ID
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch, torch.nn.functional, einops; shared test helpers from tests.kernels.utils; and vLLM components like vllm, vllm.model_executor.layers.mamba.ops.mamba_ssm, vllm.platforms, vllm.utils.torch_utils.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch、torch.nn.functional、einops；共享测试辅助模块，例如 tests.kernels.utils；vLLM 内部组件，例如 vllm、vllm.model_executor.layers.mamba.ops.mamba_ssm、vllm.platforms、vllm.utils.torch_utils。

### Function `selective_state_update_ref` (lines 20-89)
```python
def selective_state_update_ref(
    state, x, dt, A, B, C, D=None, z=None, dt_bias=None, dt_softplus=False
):
    """
    Argument:
        state: (batch, dim, dstate) or (batch, nheads, dim, dstate)
        x: (batch, dim) or (batch, nheads, dim)
        dt: (batch, dim) or (batch, nheads, dim)
        A: (dim, dstate) or (nheads, dim, dstate)
        B: (batch, dstate) or (batch, ngroups, dstate)
        C: (batch, dstate) or (batch, ngroups, dstate)
        D: (dim,) or (nheads, dim)
        z: (batch, dim) or (batch, nheads, dim)
        dt_bias: (dim,) or (nheads, dim)
    Return:
        out: (batch, dim) or (batch, nheads, dim)
    """
    has_heads = state.dim() > 3
    if state.dim() == 3:
        state = state.unsqueeze(1)
    if x.dim() == 2:
        x = x.unsqueeze(1)
    if dt.dim() == 2:
        dt = dt.unsqueeze(1)
    if A.dim() == 2:
        A = A.unsqueeze(0)
    if B.dim() == 2:
        B = B.unsqueeze(1)
    if C.dim() == 2:
        C = C.unsqueeze(1)
    if D is not None and D.dim() == 1:
        D = D.unsqueeze(0)
    if z is not None and z.dim() == 2:
        z = z.unsqueeze(1)
    if dt_bias is not None and dt_bias.dim() == 1:
        dt_bias = dt_bias.unsqueeze(0)
    batch, nheads, dim, dstate = state.shape
    assert x.shape == (batch, nheads, dim)
    assert dt.shape == x.shape
    assert A.shape == (nheads, dim, dstate)
    ngroups = B.shape[1]
    assert nheads % ngroups == 0, "nheads must be divisible by ngroups"
    assert B.shape == (batch, ngroups, dstate)
    assert C.shape == B.shape
    if D is not None:
        assert D.shape == (nheads, dim)
    if z is not None:
        assert z.shape == x.shape
    if dt_bias is not None:
        assert dt_bias.shape == (nheads, dim)
        dt = dt + dt_bias
    dt = F.softplus(dt) if dt_softplus else dt
    dA = torch.exp(
        rearrange(dt, "b h d -> b h d 1") * A
    )  # (batch, nheads, dim, dstate)
    B = repeat(B, "b g n -> b (g h) n", h=nheads // ngroups)  # (batch, nheads, dstate)
    C = repeat(C, "b g n -> b (g h) n", h=nheads // ngroups)  # (batch, nheads, dstate)
    dB = rearrange(dt, "b h d -> b h d 1") * rearrange(
        B, "b h n -> b h 1 n"
    )  # (batch, nheads, dim, dstate)
    state.copy_(
        state * dA + dB * rearrange(x, "b h d -> b h d 1")
    )  # (batch, dim, dstate
    out = torch.einsum("bhdn,bhn->bhd", state.to(C.dtype), C)
    if D is not None:
        out += (x * D).to(out.dtype)
    out = (out if z is None else out * F.silu(z)).to(x.dtype)
    if not has_heads:
        out = out.squeeze(1)
    return out
```
**EN:** This helper function implements the shared logic for selective state update ref. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该辅助函数实现了 selective state update ref 所需的共享逻辑。 结尾处的断言会固定预期行为或计算图形态。

### Function `selective_scan_ref` (lines 92-165)
```python
def selective_scan_ref(
    u,
    delta,
    A,
    B,
    C,
    D=None,
    z=None,
    delta_bias=None,
    delta_softplus=False,
    return_last_state=False,
    prev_state=None,
    final_state_out=None,
):
    """
    u: r(B D L)
    delta: r(B D L)
    A: c(D N) or r(D N)
    B: c(D N) or r(B N L) or r(B N 2L) or r(B G N L) or (B G N L)
    C: c(D N) or r(B N L) or r(B N 2L) or r(B G N L) or (B G N L)
    D: r(D)
    z: r(B D L)
    delta_bias: r(D), fp32
    prev_state: r(B D N), fp32

    out: r(B D L)
    last_state (optional): r(B D dstate) or c(B D dstate)
    """
    dtype_in = u.dtype
    u = u.float()
    delta = delta.float()
    if delta_bias is not None:
        delta = delta + delta_bias[..., None].float()
    if delta_softplus:
        delta = F.softplus(delta)
    batch, dim, dstate = u.shape[0], A.shape[0], A.shape[1]
    is_variable_B = B.dim() >= 3
    is_variable_C = C.dim() >= 3
    B = B.float()
    C = C.float()
    x = A.new_zeros((batch, dim, dstate)) if prev_state is None else prev_state
    ys = []
    deltaA = torch.exp(torch.einsum("bdl,dn->bdln", delta, A))
    if not is_variable_B:
        deltaB_u = torch.einsum("bdl,dn,bdl->bdln", delta, B, u)
    else:
        if B.dim() == 3:
            deltaB_u = torch.einsum("bdl,bnl,bdl->bdln", delta, B, u)
        else:
            B = repeat(B, "B G N L -> B (G H) N L", H=dim // B.shape[1])
            deltaB_u = torch.einsum("bdl,bdnl,bdl->bdln", delta, B, u)
    if is_variable_C and C.dim() == 4:
        C = repeat(C, "B G N L -> B (G H) N L", H=dim // C.shape[1])
    for i in range(u.shape[2]):
        x = deltaA[:, :, i] * x + deltaB_u[:, :, i]
        if not is_variable_C:
            y = torch.einsum("bdn,dn->bd", x, C)
        else:
            if C.dim() == 3:
                y = torch.einsum("bdn,bn->bd", x, C[:, :, i])
            else:
                y = torch.einsum("bdn,bdn->bd", x, C[:, :, :, i])
        if i == u.shape[2] - 1:
            if final_state_out is None:
                final_state_out = x
            else:
                final_state_out.copy_(x)
        ys.append(y)
    y = torch.stack(ys, dim=2)  # (batch dim L)
    out = y if D is None else y + u * rearrange(D, "d -> d 1")
    if z is not None:
        out = out * F.silu(z)
    out = out.to(dtype=dtype_in)
    return out if not return_last_state else (out, final_state_out)
```
**EN:** This helper function implements the shared logic for selective scan ref. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该辅助函数实现了 selective scan ref 所需的共享逻辑。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `selective_scan_opcheck_fn` (lines 168-241)
```python
def selective_scan_opcheck_fn(
    u,
    delta,
    A,
    B,
    C,
    D=None,
    z=None,
    delta_bias=None,
    delta_softplus=False,
    cu_seq_len=None,
    cache_indices=None,
    has_initial_state=None,
    ssm_states=None,
    null_block_id=NULL_BLOCK_ID,
    block_size=2048,
    block_idx_first_scheduled_token=None,
    block_idx_last_scheduled_token=None,
    initial_state_idx=None,
    cu_chunk_seqlen=None,
    last_chunk_indices=None,
):
    """if return_last_state is True, returns (out, last_state)
    last_state has shape (batch, dim, dstate).
    """
    if u.stride(-1) != 1:
        u = u.contiguous()
    if delta.stride(-1) != 1:
        delta = delta.contiguous()
    if D is not None:
        D = D.contiguous()
    if B.stride(-1) != 1:
        B = B.contiguous()
    if C.stride(-1) != 1:
        C = C.contiguous()
    if z is not None and z.stride(-1) != 1:
        z = z.contiguous()
    if B.dim() == 3 and cu_seq_len is None:
        B = B.unsqueeze(1)
    if B.dim() == 2 and cu_seq_len is not None:
        B = B.unsqueeze(0)
    if C.dim() == 3 and cu_seq_len is None:
        C = C.unsqueeze(1)
    if C.dim() == 2 and cu_seq_len is not None:
        C = C.unsqueeze(0)

    # Disable test_autograd_registration for now as it seems to trigger
    # a bogus error.
    opcheck(
        torch.ops._C.selective_scan_fwd,
        (
            u,
            delta,
            A,
            B,
            C,
            D,
            z,
            delta_bias,
            delta_softplus,
            cu_seq_len,
            cache_indices,
            has_initial_state,
            ssm_states,
            null_block_id,
            block_size,
            block_idx_first_scheduled_token,
            block_idx_last_scheduled_token,
            initial_state_idx,
            cu_chunk_seqlen,
            last_chunk_indices,
        ),
        test_utils=["test_schema", "test_faketensor"],
    )
```
**EN:** This helper function implements the shared logic for selective scan opcheck fn. it also validates that the custom operator entry point is wired correctly.
**CN:** 该辅助函数实现了 selective scan opcheck fn 所需的共享逻辑。 它还会校验自定义算子入口是否正确接线。

### Function `test_selective_scan` (lines 244-393)
```python
@pytest.mark.parametrize("wtype", [torch.float32])
@pytest.mark.parametrize("itype", [torch.float32, torch.bfloat16])
@pytest.mark.parametrize("seqlen", [128, 1024, 4096])
@pytest.mark.parametrize("has_delta_bias", [True])
@pytest.mark.parametrize("delta_softplus", [True])
@pytest.mark.parametrize("has_z", [True])
@pytest.mark.parametrize("has_D", [True])
@pytest.mark.parametrize("varBC_groups", [1, 2])
@pytest.mark.parametrize("is_variable_C", [True])
@pytest.mark.parametrize("is_variable_B", [True])
@pytest.mark.parametrize("scan_chunks", [1, 3])
def test_selective_scan(
    is_variable_B,
    is_variable_C,
    varBC_groups,
    has_D,
    has_z,
    has_delta_bias,
    delta_softplus,
    seqlen,
    itype,
    wtype,
    scan_chunks,
):
    if varBC_groups > 1 and (not is_variable_B or not is_variable_C):
        pytest.skip()  # This config is not applicable
    device = "cuda"
    rtol, atol = (6e-4, 2e-3) if itype == torch.float32 else (3e-3, 5e-3)
    if itype == torch.bfloat16:
        rtol, atol = 3e-2, 5e-2
    rtolw, atolw = (1e-3, 1e-3)
    if has_z:  # If we have z, the errors on the weights seem higher
        rtolw = max(rtolw, rtol)
        atolw = max(atolw, atol)
    # set seed
    set_random_seed(0)
    batch_size = 1
    dim = 4
    dstate = 8
    A = -0.5 * torch.rand(dim, dstate, device=device, dtype=wtype)
# ... excerpt ...
            block_idx_first_scheduled_token=None,
            block_idx_last_scheduled_token=None,
            initial_state_idx=None,
        )
        outs.append(out)
    if len(outs) > 1:
        out = torch.cat(outs, dim=-1)

    out_ref, state_ref, *rest = selective_scan_ref(
        u_ref,
        delta_ref,
        A_ref,
        B_ref,
        C_ref,
        D_ref,
        z=z_ref,
        delta_bias=delta_bias,
        delta_softplus=delta_softplus,
        return_last_state=True,
    )

    assert out is not None and out_ref is not None
    assert torch.allclose(out, out_ref, rtol=rtol, atol=atol)
    assert state is not None and state_ref is not None
    assert torch.allclose(state, state_ref.to(itype), rtol=rtol, atol=atol)

    selective_scan_opcheck_fn(
        u,
        delta,
        A,
        B,
        C,
        D,
        z,
        delta_bias=delta_bias,
        delta_softplus=delta_softplus,
        ssm_states=state,
        block_size=2048,
    )
```
**EN:** This pytest case verifies selective scan. It is parameterized across 11 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as is_variable_B, is_variable_C, varBC_groups, has_D. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 selective scan 的行为。 它通过 11 组参数化输入覆盖多种场景；它会使用诸如 is_variable_B、is_variable_C、varBC_groups、has_D 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。 由于该代码块较大，这里只展示关键片段。

### Function `test_selective_state_update` (lines 396-429)
```python
@pytest.mark.parametrize("itype", [torch.float32, torch.bfloat16])
@pytest.mark.parametrize("has_z", [False, True])
@pytest.mark.parametrize("dstate", [16, 64])
@pytest.mark.parametrize("dim", [2048, 2048 + 16, 4096])
def test_selective_state_update(dim, dstate, has_z, itype):
    device = "cuda"
    rtol, atol = (3e-4, 1e-3) if itype == torch.float32 else (5e-3, 1e-2)
    if itype == torch.bfloat16:
        rtol, atol = 1e-2, 5e-2
        if torch.version.hip:
            atol *= 2
    # set seed
    set_random_seed(0)
    batch_size = 1
    state = torch.randn(batch_size, dim, dstate, dtype=itype, device=device)
    x = torch.randn(batch_size, dim, device=device, dtype=itype)
    out = torch.empty_like(x)
    dt = torch.randn(batch_size, dim, device=device, dtype=itype)
    dt_bias = torch.rand(dim, device=device) - 4.0
    A = -torch.rand(dim, dstate, device=device) - 1.0
    B = torch.randn(batch_size, dstate, device=device)
    C = torch.randn(batch_size, dstate, device=device)
    D = torch.randn(dim, device=device)
    z = torch.randn_like(x) if has_z else None
    state_ref = state.detach().clone()
    selective_state_update(
        state, x, dt, A, B, C, D=D, z=z, dt_bias=dt_bias, dt_softplus=True, out=out
    )
    out_ref = selective_state_update_ref(
        state_ref, x, dt, A, B, C, D=D, z=z, dt_bias=dt_bias, dt_softplus=True
    )

    assert torch.allclose(state, state_ref, rtol=rtol, atol=atol)
    assert torch.allclose(out, out_ref, rtol=rtol, atol=atol)
```
**EN:** This pytest case verifies selective state update. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as dim, dstate, has_z, itype. assertions at the end lock in the intended behavior or graph shape. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 selective state update 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 dim、dstate、has_z、itype 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态；代码会控制随机性以保证场景可复现。

### Function `test_selective_state_update_stochastic_rounding` (lines 432-482)
```python
@pytest.mark.parametrize("philox_rounds", [0, 4])
@pytest.mark.parametrize("has_z", [False, True])
@pytest.mark.parametrize("dstate", [16, 64])
@pytest.mark.parametrize("dim", [2048, 4096])
@pytest.mark.skipif(
    not (
        current_platform.is_cuda() and current_platform.is_device_capability_family(100)
    ),
    reason="Stochastic rounding in triton is only supported"
    " on compute capability 10.0 CUDA devices.",
)
def test_selective_state_update_stochastic_rounding(dim, dstate, has_z, philox_rounds):
    device = "cuda"
    rtol, atol = 5e-3, 1e-1
    # set seed
    set_random_seed(0)
    batch_size = 1
    state = torch.randn(batch_size, dim, dstate, dtype=torch.float16, device=device)
    x = torch.randn(batch_size, dim, device=device, dtype=torch.bfloat16)
    out = torch.empty_like(x)
    dt = torch.randn(batch_size, dim, device=device, dtype=torch.bfloat16)
    dt_bias = torch.rand(dim, device=device) - 4.0
    A = -torch.rand(dim, dstate, device=device) - 1.0
    B = torch.randn(batch_size, dstate, device=device)
    C = torch.randn(batch_size, dstate, device=device)
    D = torch.randn(dim, device=device)
    z = torch.randn_like(x) if has_z else None
    # Reference uses fp32 state to get ground truth
    state_ref = state.float()
    selective_state_update(
        state,
        x,
        dt,
        A,
        B,
        C,
        D=D,
        z=z,
        dt_bias=dt_bias,
        dt_softplus=True,
        out=out,
        enable_stochastic_rounding=True,
        cache_philox_rounds=philox_rounds,
    )
    out_ref = selective_state_update_ref(
        state_ref, x, dt, A, B, C, D=D, z=z, dt_bias=dt_bias, dt_softplus=True
    )

    assert state.dtype == torch.float16
    assert torch.allclose(state, state_ref.to(torch.float16), rtol=rtol, atol=atol)
    assert torch.allclose(out, out_ref, rtol=rtol, atol=atol)
```
**EN:** This pytest case verifies selective state update stochastic rounding. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as dim, dstate, has_z, philox_rounds. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 selective state update stochastic rounding 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 dim、dstate、has_z、philox_rounds 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。

### Function `test_selective_state_update_varlen` (lines 485-556)
```python
@pytest.mark.parametrize("itype", [torch.float32, torch.bfloat16])
@pytest.mark.parametrize("has_z", [False, True])
@pytest.mark.parametrize("dstate", [16, 64])
@pytest.mark.parametrize("dim", [2048, 2048 + 16, 4096])
@pytest.mark.parametrize("max_seq_len", [1, 2, 4])
def test_selective_state_update_varlen(dim, dstate, has_z, itype, max_seq_len):
    device = "cuda"
    rtol, atol = (3e-4, 1e-3) if itype == torch.float32 else (5e-3, 1e-2)
    if itype == torch.bfloat16:
        rtol, atol = 5e-2, 1.5e-1
        if torch.version.hip:
            atol *= 2
    # set seed
    set_random_seed(0)
    batch_size = 4
    token_counts = torch.randint(1, max_seq_len + 1, (batch_size,), device=device)
    total_tokens = int(token_counts.sum().item())
    cu_seqlens = torch.tensor(
        [0] + torch.cumsum(token_counts, dim=0).tolist(),
        dtype=torch.int32,
        device=device,
    )
    state = torch.randn(batch_size, dim, dstate, dtype=itype, device=device)
    x = torch.randn(total_tokens, dim, device=device, dtype=itype)
    out = torch.empty_like(x)
    dt = torch.randn(total_tokens, dim, device=device, dtype=itype)
    dt_bias = torch.rand(dim, device=device) - 4.0
    A = -torch.rand(dim, dstate, device=device) - 1.0
    B = torch.randn(total_tokens, dstate, device=device)
    C = torch.randn(total_tokens, dstate, device=device)
    D = torch.randn(dim, device=device)
    z = torch.randn_like(x) if has_z else None
    state_ref = state.detach().clone()
    selective_state_update(
        state,
        x,
        dt,
        A,
        B,
        C,
        D=D,
        z=z,
        dt_bias=dt_bias,
        dt_softplus=True,
        out=out,
        cu_seqlens=cu_seqlens,
    )

    out_ref_list = []
    for seq_idx in range(batch_size):
        start_idx = cu_seqlens[seq_idx].item()
        end_idx = cu_seqlens[seq_idx + 1].item()
        num_tokens = end_idx - start_idx
        for token_idx in range(num_tokens):
            idx = start_idx + token_idx
            out_ref_list.append(
                selective_state_update_ref(
                    state_ref[seq_idx : seq_idx + 1],
                    x[idx : idx + 1],
                    dt[idx : idx + 1],
                    A,
                    B[idx : idx + 1],
                    C[idx : idx + 1],
                    D=D,
                    z=z[idx : idx + 1] if z is not None else None,
                    dt_bias=dt_bias,
                    dt_softplus=True,
                )
            )
    out_ref = torch.cat(out_ref_list, dim=0)
    assert torch.allclose(state, state_ref, rtol=rtol, atol=atol)
    assert torch.allclose(out, out_ref, rtol=rtol, atol=atol)
```
**EN:** This pytest case verifies selective state update varlen. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as dim, dstate, has_z, itype. assertions at the end lock in the intended behavior or graph shape. randomness is controlled so the scenario remains reproducible.
**CN:** 该 pytest 用例验证 selective state update varlen 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 dim、dstate、has_z、itype 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态；代码会控制随机性以保证场景可复现。

### Function `test_selective_scan_varlen` (lines 559-741)
```python
@pytest.mark.parametrize("wtype", [torch.float32])
@pytest.mark.parametrize("itype", [torch.float32])
@pytest.mark.parametrize("seqlen", [1, 256, 1024, 4096])
@pytest.mark.parametrize("return_last_state", [True])
@pytest.mark.parametrize("has_delta_bias", [True])
@pytest.mark.parametrize("delta_softplus", [True])
@pytest.mark.parametrize("has_z", [True])
@pytest.mark.parametrize("has_D", [True])
@pytest.mark.parametrize("varBC_groups", [1, 2])
@pytest.mark.parametrize("is_variable_C", [True])
@pytest.mark.parametrize("is_variable_B", [True])
# tests correctness in case subset of the sequences are padded
@pytest.mark.parametrize("with_padding", [False, True])
def test_selective_scan_varlen(
    with_padding,
    is_variable_B,
    is_variable_C,
    varBC_groups,
    has_D,
    has_z,
    has_delta_bias,
    delta_softplus,
    return_last_state,
    seqlen,
    itype,
    wtype,
):
    if varBC_groups > 1 and (not is_variable_B or not is_variable_C):
        pytest.skip()  # This config is not applicable
    device = "cuda"
    rtol, atol = (6e-4, 2e-3) if itype == torch.float32 else (3e-3, 5e-3)
    if itype == torch.bfloat16:
        rtol, atol = 3e-2, 5e-2
    rtolw, atolw = (1e-3, 1e-3)
    if has_z:  # If we have z, the errors on the weights seem higher
        rtolw = max(rtolw, rtol)
        atolw = max(atolw, atol)
    # set seed
    torch.random.manual_seed(0)
    seqlens = []
# ... excerpt ...
            A_ref,
            B_s,
            C_s,
            D_ref,
            z=z_s,
            delta_bias=delta_bias,
            delta_softplus=delta_softplus,
            return_last_state=return_last_state,
            prev_state=prev_state_ref[padded_state_indices[i]].unsqueeze(0)
            if has_initial_state[i]
            else None,
            final_state_out=prev_state_ref[padded_state_indices[i]].unsqueeze(0),
        )
        outs_ref.append(out_ref_s)
    out_ref = torch.cat(outs_ref, dim=-1)[0]

    unpadded_out = out[:, : out_ref[0].shape[-1]]
    print("Output diff max", (unpadded_out - out_ref).max())
    print("Output diff mean", (unpadded_out - out_ref).mean())
    print("Output state diff max", (prev_state - prev_state_ref).max())
    print("Output state diff mean", (prev_state - prev_state_ref).mean())
    assert torch.allclose(prev_state, prev_state_ref, rtol=rtol, atol=atol)
    assert torch.allclose(unpadded_out, out_ref, rtol=rtol, atol=atol)
    selective_scan_opcheck_fn(
        u,
        delta,
        A,
        B,
        C,
        D,
        z,
        delta_bias,
        delta_softplus,
        cumsum,
        padded_state_indices,
        has_initial_state,
        prev_state,
        block_size=2048,
    )
```
**EN:** This pytest case verifies selective scan varlen. It is parameterized across 12 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as with_padding, is_variable_B, is_variable_C, varBC_groups. unsupported hardware, backend, or configuration combinations are skipped early. assertions at the end lock in the intended behavior or graph shape. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 selective scan varlen 的行为。 它通过 12 组参数化输入覆盖多种场景；它会使用诸如 with_padding、is_variable_B、is_variable_C、varBC_groups 等 fixture 或输入；不支持的硬件、后端或配置组合会被提前跳过；结尾处的断言会固定预期行为或计算图形态。 由于该代码块较大，这里只展示关键片段。

### Function `test_selective_state_update_with_batch_indices` (lines 744-833)
```python
@pytest.mark.parametrize("itype", [torch.float32, torch.bfloat16])
@pytest.mark.parametrize("has_z", [True])
@pytest.mark.parametrize("dstate", [16, 64])
@pytest.mark.parametrize("dim", [2048, 2048 + 16, 4096])
# tests correctness in case subset of the sequences are padded
@pytest.mark.parametrize("with_padding", [True, False])
def test_selective_state_update_with_batch_indices(
    with_padding, dim, dstate, has_z, itype
):
    device = "cuda"
    rtol, atol = (3e-4, 1e-3) if itype == torch.float32 else (5e-3, 1e-2)
    if itype == torch.bfloat16:
        rtol, atol = 1e-1, 1e-1
        if torch.version.hip:
            atol *= 2
    # set seed
    torch.random.manual_seed(0)
    batch_size = 3
    padding = 5 if with_padding else 0
    padded_batch_size = batch_size + padding
    total_entries = 10 * batch_size
    state = torch.randn(total_entries, dim, dstate, dtype=itype, device=device)
    # +1 to exclude index 0 (null block)
    state_indices = (torch.randperm(total_entries - 1)[:batch_size] + 1).to(
        dtype=torch.int32, device=device
    )
    unused_states_bool = torch.ones(total_entries, dtype=torch.bool, device=device)
    unused_states_bool[state_indices] = False
    padded_state_indices = torch.concat(
        [
            state_indices,
            torch.as_tensor(
                [NULL_BLOCK_ID] * padding, dtype=torch.int32, device=device
            ),
        ],
        dim=0,
    )
    x = torch.randn(padded_batch_size, dim, device=device, dtype=itype)
    out = torch.empty_like(x)
    dt = torch.randn(padded_batch_size, dim, device=device, dtype=itype)
# ... excerpt ...
        dt,
        A,
        B,
        C,
        D=D,
        z=z,
        dt_bias=dt_bias,
        dt_softplus=True,
        state_batch_indices=padded_state_indices,
        out=out,
    )
    out_ref = selective_state_update_ref(
        state_ref,
        x[:batch_size],
        dt[:batch_size],
        A,
        B[:batch_size],
        C[:batch_size],
        D=D,
        z=z[:batch_size] if z is not None else None,
        dt_bias=dt_bias,
        dt_softplus=True,
    )

    print("Output diff max", (out[:batch_size] - out_ref).max())
    print("Output diff mean", (out[:batch_size] - out_ref).mean())
    print("Output state diff max", (state[state_indices, :] - state_ref).max())
    print("Output state diff mean", (state[state_indices, :] - state_ref).mean())
    # test padded entries stay the same
    if with_padding:
        assert torch.equal(state_before[unused_states_bool], state[unused_states_bool])
        assert torch.equal(x[batch_size + 1 :], x[batch_size + 1 :])
        assert torch.equal(dt[batch_size + 1 :], dt[batch_size + 1 :])
        assert torch.equal(B[batch_size + 1 :], B[batch_size + 1 :])
        assert torch.equal(C[batch_size + 1 :], C[batch_size + 1 :])

    # test "real" entries
    assert torch.allclose(state[state_indices, :], state_ref, rtol=rtol, atol=atol)
    assert torch.allclose(out[:batch_size], out_ref, rtol=rtol, atol=atol)
```
**EN:** This pytest case verifies selective state update with batch indices. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as with_padding, dim, dstate, has_z. assertions at the end lock in the intended behavior or graph shape. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 selective state update with batch indices 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 with_padding、dim、dstate、has_z 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。 由于该代码块较大，这里只展示关键片段。

### Function `test_selective_state_update_with_heads_with_batch_indices` (lines 836-907)
```python
@pytest.mark.parametrize("itype", [torch.float32, torch.bfloat16])
@pytest.mark.parametrize("has_z", [False, True])
@pytest.mark.parametrize("tie_hdim", [False, True])
@pytest.mark.parametrize("ngroups", [1, 4])
@pytest.mark.parametrize("dstate", [16, 64])
@pytest.mark.parametrize("dim", [2048, 4096])
def test_selective_state_update_with_heads_with_batch_indices(
    dim, dstate, ngroups, has_z, tie_hdim, itype
):
    device = "cuda"
    rtol, atol = (3e-4, 1e-3) if itype == torch.float32 else (5e-3, 3e-2)
    if itype == torch.bfloat16:
        rtol, atol = 1e-1, 1e-1
    # set seed
    torch.random.manual_seed(0)
    batch_size = 3
    headdim = 64
    nheads = dim // headdim

    total_entries = 10 * batch_size
    state = torch.randn(
        total_entries, nheads, headdim, dstate, dtype=itype, device=device
    )
    # +1 to exclude index 0 (null block)
    state_indices = (torch.randperm(total_entries - 1)[:batch_size] + 1).to(
        dtype=torch.int32, device=device
    )

    x = torch.randn(batch_size, nheads, headdim, device=device, dtype=itype)
    out = torch.empty_like(x)
    if not tie_hdim:
        dt = torch.randn(batch_size, nheads, headdim, device=device, dtype=itype)
        dt_bias = torch.rand(nheads, headdim, device=device) - 4.0
        A = -torch.rand(nheads, headdim, dstate, device=device) - 1.0
        D = torch.randn(nheads, headdim, device=device)
    else:
        dt = repeat(
            torch.randn(batch_size, nheads, device=device, dtype=itype),
            "b h -> b h p",
            p=headdim,
        )
        dt_bias = repeat(torch.rand(nheads, device=device) - 4.0, "h -> h p", p=headdim)
        A = repeat(
            -torch.rand(nheads, device=device) - 1.0, "h -> h p n", p=headdim, n=dstate
        )
        D = repeat(torch.randn(nheads, device=device), "h -> h p", p=headdim)
    B = torch.randn(batch_size, ngroups, dstate, device=device)
    C = torch.randn(batch_size, ngroups, dstate, device=device)
    z = torch.randn_like(x) if has_z else None
    state_ref = state[state_indices, :].detach().clone()
    selective_state_update(
        state,
        x,
        dt,
        A,
        B,
        C,
        D=D,
        z=z,
        dt_bias=dt_bias,
        dt_softplus=True,
        state_batch_indices=state_indices,
        out=out,
    )
    out_ref = selective_state_update_ref(
        state_ref, x, dt, A, B, C, D=D, z=z, dt_bias=dt_bias, dt_softplus=True
    )

    print(f"Output max diff: {(out - out_ref).abs().max().item()}")
    print(f"Output mean diff: {(out - out_ref).abs().mean().item()}")
    assert torch.allclose(state[state_indices, :], state_ref, rtol=rtol, atol=atol)
    assert torch.allclose(out, out_ref, rtol=rtol, atol=atol)
```
**EN:** This pytest case verifies selective state update with heads with batch indices. It is parameterized across 6 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as dim, dstate, ngroups, has_z. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 selective state update with heads with batch indices 的行为。 它通过 6 组参数化输入覆盖多种场景；它会使用诸如 dim、dstate、ngroups、has_z 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Function `test_selective_state_update_with_num_accepted_tokens` (lines 910-1033)
```python
@pytest.mark.parametrize("itype", [torch.float32, torch.bfloat16])
@pytest.mark.parametrize("has_z", [False, True])
@pytest.mark.parametrize("dstate", [16, 64])
@pytest.mark.parametrize("dim", [2048, 4096])
@pytest.mark.parametrize("max_seq_len", [2, 4])
def test_selective_state_update_with_num_accepted_tokens(
    dim, dstate, has_z, itype, max_seq_len
):
    device = "cuda"
    rtol, atol = (3e-4, 1e-3) if itype == torch.float32 else (5e-3, 1e-2)
    if itype == torch.bfloat16:
        rtol, atol = 5e-2, 1.5e-1
        if torch.version.hip:
            atol *= 2

    set_random_seed(0)
    batch_size = 4

    tokens_per_seq = torch.randint(1, max_seq_len + 1, (batch_size,), device=device)
    total_tokens = int(tokens_per_seq.sum().item())

    num_accepted_tokens = torch.randint(0, max_seq_len, (batch_size,), device=device)
    num_accepted_tokens[0] = 0  # Add edge-case of no accepted tokens
    num_accepted_tokens[1] = max_seq_len  # Add edge-case of all tokens accepted

    cu_seqlens = torch.tensor(
        [0] + torch.cumsum(tokens_per_seq, dim=0).tolist(),
        dtype=torch.int32,
        device=device,
    )

    total_state_slots = 50
    state = torch.randn(total_state_slots, dim, dstate, dtype=itype, device=device)

    state_batch_indices = torch.full(
        (batch_size, max_seq_len), NULL_BLOCK_ID, dtype=torch.int32, device=device
    )
    # Start from 1 to exclude null block at index 0
    initial_state_slots = torch.randint(
        1, 15, (batch_size,), device=device, dtype=torch.int32
# ... excerpt ...
                A,
                B[global_idx : global_idx + 1],
                C[global_idx : global_idx + 1],
                D=D,
                z=z[global_idx : global_idx + 1] if z is not None else None,
                dt_bias=dt_bias,
                dt_softplus=True,
            )
            out_ref_list.append(out_token)
            state_ref_intermediate[(seq_idx, token_idx)] = state_seq.clone()

    out_ref = torch.cat(out_ref_list, dim=0)

    selective_state_update(
        state,
        x,
        dt,
        A,
        B,
        C,
        D=D,
        z=z,
        dt_bias=dt_bias,
        dt_softplus=True,
        out=out,
        cu_seqlens=cu_seqlens,
        state_batch_indices=state_batch_indices,
        dst_state_batch_indices=dst_state_batch_indices,
        num_accepted_tokens=num_accepted_tokens,
    )

    assert torch.allclose(out, out_ref, rtol=rtol, atol=atol)

    for seq_idx in range(batch_size):
        num_tokens = tokens_per_seq[seq_idx].item()
        for token_idx in range(num_tokens):
            dst_slot = dst_slots_map[(seq_idx, token_idx)]
            state_ref = state_ref_intermediate[(seq_idx, token_idx)].squeeze(0)
            assert torch.allclose(state[dst_slot], state_ref, rtol=rtol, atol=atol)
```
**EN:** This pytest case verifies selective state update with num accepted tokens. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as dim, dstate, has_z, itype. assertions at the end lock in the intended behavior or graph shape. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 selective state update with num accepted tokens 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 dim、dstate、has_z、itype 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

### Function `test_selective_state_update_varlen_with_num_accepted` (lines 1036-1158)
```python
@pytest.mark.parametrize("itype", [torch.float32, torch.bfloat16])
@pytest.mark.parametrize("has_z", [False, True])
@pytest.mark.parametrize("dstate", [16, 64])
@pytest.mark.parametrize("dim", [2048, 4096])
@pytest.mark.parametrize("max_seq_len", [2, 4])
def test_selective_state_update_varlen_with_num_accepted(
    dim, dstate, has_z, itype, max_seq_len
):
    device = "cuda"
    rtol, atol = (3e-4, 1e-3) if itype == torch.float32 else (5e-3, 1e-2)
    if itype == torch.bfloat16:
        rtol, atol = 5e-2, 1.5e-1
        if torch.version.hip:
            atol *= 2

    set_random_seed(0)
    batch_size = 4

    tokens_per_seq = torch.randint(1, max_seq_len + 1, (batch_size,), device=device)
    total_tokens = int(tokens_per_seq.sum().item())

    num_accepted_tokens = torch.randint(0, max_seq_len, (batch_size,), device=device)
    num_accepted_tokens[0] = 0  # Add edge-case of no accepted tokens
    num_accepted_tokens[1] = max_seq_len  # Add edge-case of all tokens accepted

    cu_seqlens = torch.tensor(
        [0] + torch.cumsum(tokens_per_seq, dim=0).tolist(),
        dtype=torch.int32,
        device=device,
    )

    total_state_slots = 50
    state = torch.randn(total_state_slots, dim, dstate, dtype=itype, device=device)

    state_batch_indices = torch.full(
        (batch_size, max_seq_len), NULL_BLOCK_ID, dtype=torch.int32, device=device
    )

    # Start from 1 to exclude null block at index 0
    initial_state_slots = torch.randint(
# ... excerpt ...
                x[global_idx : global_idx + 1],
                dt[global_idx : global_idx + 1],
                A,
                B[global_idx : global_idx + 1],
                C[global_idx : global_idx + 1],
                D=D,
                z=z[global_idx : global_idx + 1] if z is not None else None,
                dt_bias=dt_bias,
                dt_softplus=True,
            )

            state_ref_intermediate[(seq_idx, token_idx)] = state_seq.clone()

    selective_state_update(
        state,
        x,
        dt,
        A,
        B,
        C,
        D=D,
        z=z,
        dt_bias=dt_bias,
        dt_softplus=True,
        out=out,
        cu_seqlens=cu_seqlens,
        state_batch_indices=state_batch_indices,
        dst_state_batch_indices=dst_state_batch_indices,
        num_accepted_tokens=num_accepted_tokens,
    )

    for seq_idx in range(batch_size):
        num_tokens = tokens_per_seq[seq_idx].item()

        for token_idx in range(num_tokens):
            dst_slot = dst_slots_map[(seq_idx, token_idx)]
            state_ref = state_ref_intermediate[(seq_idx, token_idx)].squeeze(0)

            assert torch.allclose(state[dst_slot], state_ref, rtol=rtol, atol=atol)
```
**EN:** This pytest case verifies selective state update varlen with num accepted. It is parameterized across 5 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as dim, dstate, has_z, itype. assertions at the end lock in the intended behavior or graph shape. randomness is controlled so the scenario remains reproducible. Only the key portions are shown here because the block is large.
**CN:** 该 pytest 用例验证 selective state update varlen with num accepted 的行为。 它通过 5 组参数化输入覆盖多种场景；它会使用诸如 dim、dstate、has_z、itype 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态；代码会控制随机性以保证场景可复现。 由于该代码块较大，这里只展示关键片段。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Platform guards / 平台保护:** Hardware, backend, or version checks prevent unsupported combinations from running. / 硬件、后端或版本检查会阻止不受支持的组合继续执行。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `torch.nn.functional`
- `einops -> rearrange, repeat`
- `tests.kernels.utils -> opcheck`
- `vllm -> _custom_ops`
- `vllm.model_executor.layers.mamba.ops.mamba_ssm -> selective_scan_fn, selective_state_update`
- `vllm.platforms -> current_platform`
- `vllm.utils.torch_utils -> set_random_seed`
- `vllm.v1.attention.backends.utils -> NULL_BLOCK_ID`
