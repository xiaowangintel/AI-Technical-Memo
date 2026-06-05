# recurrent_gated_delta_rule.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/mamba/ops/cpu/recurrent_gated_delta_rule.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Provides helpers such as `l2norm`, `recurrent_gated_delta_rule`, `gdn_gating` for Mamba/state-space layers and kernels. / 提供诸如 `l2norm`, `recurrent_gated_delta_rule`, `gdn_gating` 之类的辅助函数，用于Mamba/状态空间模型层与内核。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 5-6)
```python
import torch
import torch.nn.functional as F
```
**EN:** This opening block pulls in external dependencies such as `torch` and internal modules such as no internal imports. That import mix shows the file is part of the Mamba/state-space layers and kernels stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `torch`）以及内部模块（如 no internal imports）。这些导入关系表明该文件属于Mamba/状态空间模型层与内核栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Function `l2norm` (lines 9-15)
```python
def l2norm(
    x: torch.Tensor,
    dim: int = -1,
    eps: float = 1e-6,
) -> torch.Tensor:
    inv_norm = torch.rsqrt((x * x).sum(dim=dim, keepdim=True) + eps)
    return x * inv_norm
```
**EN:** Defines function `l2norm` with signature `l2norm(x: torch.Tensor, dim: int=-1, eps: float=1e-06) -> torch.Tensor`. It mainly works with `x`, `dim`, `eps`; implements one step of the Mamba/SSM execution path. The body uses tensor/kernel operations. Key calls include `torch.rsqrt`, `sum`.
**CN:** 定义函数 `l2norm`，其签名为 `l2norm(x: torch.Tensor, dim: int=-1, eps: float=1e-06) -> torch.Tensor`。它主要围绕 `x`, `dim`, `eps` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含张量或内核操作。关键调用包括 `torch.rsqrt`, `sum`。

### Function `recurrent_gated_delta_rule` (lines 18-76)
```python
def recurrent_gated_delta_rule(
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    g: torch.Tensor,
    beta: torch.Tensor,
    initial_state: torch.Tensor,
    scale: float | None = None,
    use_qk_l2norm_in_kernel: bool = False,
) -> tuple[torch.Tensor, torch.Tensor]:
    initial_dtype = query.dtype
    if use_qk_l2norm_in_kernel:
        query = l2norm(query, dim=-1, eps=1e-6)
        key = l2norm(key, dim=-1, eps=1e-6)

    if query.shape[2] != value.shape[2]:
        repeat_factor = value.shape[2] // query.shape[2]
        query = query.repeat_interleave(repeat_factor, dim=2)
        key = key.repeat_interleave(repeat_factor, dim=2)

    query, key, value, beta, g = [
        x.transpose(1, 2).contiguous().to(torch.float32)
        for x in (query, key, value, beta, g)
    ]

    batch_size, num_heads, sequence_length, _ = key.shape
    v_head_dim = value.shape[-1]
    if scale is None:
        scale = 1 / (query.shape[-1] ** 0.5)
    query = query * scale

    core_attn_out = torch.empty(
        batch_size,
        num_heads,
        sequence_length,
        v_head_dim,
        dtype=value.dtype,
    )
    last_recurrent_state = initial_state.to(value)

    for token_idx in range(sequence_length):
        q_t = query[:, :, token_idx]
        k_t = key[:, :, token_idx]
        v_t = value[:, :, token_idx]
        g_t = g[:, :, token_idx].exp().unsqueeze(-1).unsqueeze(-1)
        beta_t = beta[:, :, token_idx].unsqueeze(-1)

        last_recurrent_state = last_recurrent_state * g_t
        kv_mem = (last_recurrent_state * k_t.unsqueeze(-2)).sum(dim=-1)
        delta = (v_t - kv_mem) * beta_t
        last_recurrent_state = last_recurrent_state + delta.unsqueeze(
            -1
        ) * k_t.unsqueeze(-2)
        core_attn_out[:, :, token_idx] = (last_recurrent_state * q_t.unsqueeze(-2)).sum(
            dim=-1
        )

    core_attn_out = core_attn_out.transpose(1, 2).contiguous().to(initial_dtype)
    return core_attn_out, last_recurrent_state
```
**EN:** Defines function `recurrent_gated_delta_rule` with signature `recurrent_gated_delta_rule(query: torch.Tensor, key: torch.Tensor, value: torch.Tensor, g: torch.Tensor, beta: torch.Tensor, initial_state: torch.Tensor, scale: float | None=None, use_qk_l2norm_in_kernel: bool=False) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `query`, `key`, `value`, `g`, `beta`, `initial_state`, `scale`, `use_qk_l2norm_in_kernel`; implements one step of the Mamba/SSM execution path. The body uses branching, iteration, comprehensions, tensor/kernel operations. Key calls include `torch.empty`, `initial_state.to`, `range`, `core_attn_out.transpose.contiguous.to`, `l2norm`, `query.repeat_interleave`.
**CN:** 定义函数 `recurrent_gated_delta_rule`，其签名为 `recurrent_gated_delta_rule(query: torch.Tensor, key: torch.Tensor, value: torch.Tensor, g: torch.Tensor, beta: torch.Tensor, initial_state: torch.Tensor, scale: float | None=None, use_qk_l2norm_in_kernel: bool=False) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `query`, `key`, `value`, `g`, `beta`, `initial_state`, `scale`, `use_qk_l2norm_in_kernel` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断、循环处理、推导式、张量或内核操作。关键调用包括 `torch.empty`, `initial_state.to`, `range`, `core_attn_out.transpose.contiguous.to`, `l2norm`, `query.repeat_interleave`。

### Function `gdn_gating` (lines 79-90)
```python
def gdn_gating(
    A_log: torch.Tensor,
    a: torch.Tensor,
    b: torch.Tensor,
    dt_bias: torch.Tensor,
    beta: float = 1.0,
    threshold: float = 20.0,
) -> tuple[torch.Tensor, torch.Tensor]:
    softplus_x = F.softplus(a.float() + dt_bias.float(), beta=beta, threshold=threshold)
    g = -torch.exp(A_log.float()) * softplus_x
    beta_output = torch.sigmoid(b.float()).to(dtype=b.dtype)
    return g, beta_output
```
**EN:** Defines function `gdn_gating` with signature `gdn_gating(A_log: torch.Tensor, a: torch.Tensor, b: torch.Tensor, dt_bias: torch.Tensor, beta: float=1.0, threshold: float=20.0) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `A_log`, `a`, `b`, `dt_bias`, `beta`, `threshold`; implements one step of the Mamba/SSM execution path. The body uses tensor/kernel operations. Key calls include `F.softplus`, `torch.sigmoid.to`, `a.float`, `dt_bias.float`, `torch.exp`, `torch.sigmoid`.
**CN:** 定义函数 `gdn_gating`，其签名为 `gdn_gating(A_log: torch.Tensor, a: torch.Tensor, b: torch.Tensor, dt_bias: torch.Tensor, beta: float=1.0, threshold: float=20.0) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `A_log`, `a`, `b`, `dt_bias`, `beta`, `threshold` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含张量或内核操作。关键调用包括 `F.softplus`, `torch.sigmoid.to`, `a.float`, `dt_bias.float`, `torch.exp`, `torch.sigmoid`。

### Function `chunk_gated_delta_rule` (lines 93-223)
```python
def chunk_gated_delta_rule(
    q: torch.Tensor,
    k: torch.Tensor,
    v: torch.Tensor,
    g: torch.Tensor,
    beta: torch.Tensor,
    *,
    initial_state: torch.Tensor,
    scale: float | None = None,
    cu_seqlens: torch.Tensor,
    use_qk_l2norm_in_kernel: bool = False,
) -> tuple[torch.Tensor, torch.Tensor]:
    output = torch.empty_like(v)
    state_dtype = initial_state.dtype
    chunk_size = 128
    sequence_bounds = [
        (
            seq_idx,
            int(cu_seqlens[seq_idx].item()),
            int(cu_seqlens[seq_idx + 1].item()),
        )
        for seq_idx in range(len(cu_seqlens) - 1)
    ]
    chunk_eye = torch.eye(chunk_size, dtype=torch.float32)
    num_sequences = len(sequence_bounds)
    num_value_heads = v.shape[2]
    value_head_dim = v.shape[3]
    key_head_dim = k.shape[3]
    final_state = torch.empty(
        (num_sequences, num_value_heads, value_head_dim, key_head_dim),
        dtype=state_dtype,
    )

    for seq_idx, begin, end in sequence_bounds:
        q_seq = q[:, begin:end]
        k_seq = k[:, begin:end]
        v_seq = v[:, begin:end]
        g_seq = g[:, begin:end]
# ... truncated for analysis ...

            # Carry the recurrent state forward to the next chunk boundary.
            end_decay = (cum_g[:, :, -1:] - cum_g).exp().unsqueeze(-1)
            decayed_keys = k_chunk * end_decay
            seq_state = seq_state * exp_cum_g[:, :, -1, None, None] + torch.einsum(
                "bhcv,bhck->bhvk", transformed_values, decayed_keys
            )

        output[0, begin:end].copy_(
            seq_output.transpose(1, 2).contiguous().to(initial_dtype).squeeze(0)
        )
        final_state[seq_idx].copy_(seq_state.squeeze(0).to(state_dtype).contiguous())

    return output, final_state
```
**EN:** Defines function `chunk_gated_delta_rule` with signature `chunk_gated_delta_rule(q: torch.Tensor, k: torch.Tensor, v: torch.Tensor, g: torch.Tensor, beta: torch.Tensor, *, initial_state: torch.Tensor, scale: float | None=None, cu_seqlens: torch.Tensor, use_qk_l2norm_in_kernel: bool=False) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `q`, `k`, `v`, `g`, `beta`, `initial_state`, `scale`, `cu_seqlens`; implements one step of the Mamba/SSM execution path. The body uses branching, iteration, comprehensions, tensor/kernel operations. Key calls include `torch.empty_like`, `torch.eye`, `len`, `torch.empty`, `initial_state.to`, `range`.
**CN:** 定义函数 `chunk_gated_delta_rule`，其签名为 `chunk_gated_delta_rule(q: torch.Tensor, k: torch.Tensor, v: torch.Tensor, g: torch.Tensor, beta: torch.Tensor, *, initial_state: torch.Tensor, scale: float | None=None, cu_seqlens: torch.Tensor, use_qk_l2norm_in_kernel: bool=False) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `q`, `k`, `v`, `g`, `beta`, `initial_state`, `scale`, `cu_seqlens` 展开；实现 Mamba/状态空间执行路径中的一个步骤。函数体包含分支判断、循环处理、推导式、张量或内核操作。关键调用包括 `torch.empty_like`, `torch.eye`, `len`, `torch.empty`, `initial_state.to`, `range`。

## Key Concepts / 关键概念
- **EN:** The file belongs to the Mamba/SSM path, so recurrent state updates, scan-style kernels, or short convolutions are recurring themes.
  **CN:** 该文件位于 Mamba/状态空间模型路径中，因此循环状态更新、scan 风格内核或短卷积是反复出现的主题。
- **EN:** Top-level helpers such as `l2norm`, `recurrent_gated_delta_rule`, `gdn_gating`, `chunk_gated_delta_rule` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `l2norm`, `recurrent_gated_delta_rule`, `gdn_gating`, `chunk_gated_delta_rule` 为主要类提供了过程式入口。
- **EN:** Backend-specific kernels are important here, as the module imports Triton/custom-op infrastructure or sits under an ops directory.
  **CN:** 这里明显依赖后端特定内核，因为模块导入了 Triton/自定义算子基础设施，或本身位于 ops 目录下。

## Dependencies / 依赖关系
- **External / 外部**: `torch`
- **Internal / 内部**: None / 无
