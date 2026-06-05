# test_mamba.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/srt/cpu/test_mamba.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates mamba behavior in SGLang's srt / cpu area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 SRT / CPU 领域中与 mamba 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: module imports and dependencies / 模块导入与依赖
```python
import unittest

import torch
import torch.nn.functional as F
from torch.nn.functional import softplus
from utils import precision

from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `torch`, `torch.nn.functional`, `utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `torch`, `torch.nn.functional`, `utils`。

### Lines 10-10: supporting statements / 辅助语句
```python
torch.manual_seed(1234)
```
**EN:** This block performs supporting work through calls such as `manual_seed`, preparing state for nearby definitions.
**CN:** 该代码块通过 `manual_seed` 等调用完成辅助性工作，为相邻定义准备状态。

### Lines 13-16: function l2norm / 函数 l2norm
```python
def l2norm(x: torch.FloatTensor, dim: int = -1, eps: float = 1e-6):
    """This function is intended to align with the l2norm implementation in the FLA library."""
    inv_norm = torch.rsqrt((x * x).sum(dim=dim, keepdim=True) + eps)
    return x * inv_norm
```
**EN:** This function is intended to align with the l2norm implementation in the FLA library. This block implements `l2norm` and captures one focused piece of the module's behavior.
**CN:** This function is intended to align with the l2norm implementation in the FLA library. 该代码块实现 `l2norm`，承担模块行为中的一个聚焦逻辑片段。

### Lines 19-98: function torch chunk gated delta rule (part 1/2) / 函数 torch chunk gated delta rule（第 1/2 部分）
```python
def torch_chunk_gated_delta_rule(
    query,
    key,
    value,
    g,
    beta,
    chunk_size=64,
    initial_state=None,
    output_final_state=False,
    use_qk_l2norm_in_kernel=False,
):
    initial_dtype = query.dtype
    if use_qk_l2norm_in_kernel:
        query = l2norm(query, dim=-1, eps=1e-6)
        key = l2norm(key, dim=-1, eps=1e-6)
    query, key, value, beta, g = [
        x.transpose(1, 2).contiguous().to(torch.float32)
        for x in (query, key, value, beta, g)
    ]

    batch_size, sequence_length, num_heads, k_head_dim = key.shape
    v_head_dim = value.shape[-1]
    pad_size = (chunk_size - num_heads % chunk_size) % chunk_size
    query = F.pad(query, (0, 0, 0, pad_size))
    key = F.pad(key, (0, 0, 0, pad_size))
    value = F.pad(value, (0, 0, 0, pad_size))
    beta = F.pad(beta, (0, pad_size))
    g = F.pad(g, (0, pad_size))
    tot_heads = num_heads + pad_size
    scale = 1 / (query.shape[-1] ** 0.5)
    query = query * scale

    v_beta = value * beta.unsqueeze(-1)
    k_beta = key * beta.unsqueeze(-1)
    # reshape to chunks
    query, key, value, k_beta, v_beta = [
        x.reshape(x.shape[0], x.shape[1], -1, chunk_size, x.shape[-1])
        for x in (query, key, value, k_beta, v_beta)
    ]
    g = g.reshape(g.shape[0], g.shape[1], -1, chunk_size)
    mask = torch.triu(
        torch.ones(chunk_size, chunk_size, dtype=torch.bool, device=query.device),
        diagonal=0,
    )

    # chunk decay
    g = g.cumsum(dim=-1)
    decay_mask = ((g.unsqueeze(-1) - g.unsqueeze(-2)).tril().exp().float()).tril()
    attn = -((k_beta @ key.transpose(-1, -2)) * decay_mask).masked_fill(mask, 0)
    for i in range(1, chunk_size):
        row = attn[..., i, :i].clone()
        sub = attn[..., :i, :i].clone()
        attn[..., i, :i] = row + (row.unsqueeze(-1) * sub).sum(-2)
    attn = attn + torch.eye(chunk_size, dtype=attn.dtype, device=attn.device)
    value = attn @ v_beta
    k_cumdecay = attn @ (k_beta * g.exp().unsqueeze(-1))
    last_recurrent_state = (
        torch.zeros(batch_size, sequence_length, k_head_dim, v_head_dim).to(value)
        if initial_state is None
        else initial_state.to(value)
    )
    core_attn_out = torch.zeros_like(value)
    mask = torch.triu(
        torch.ones(chunk_size, chunk_size, dtype=torch.bool, device=query.device),
        diagonal=1,
    )

    # for each chunk
    for i in range(0, tot_heads // chunk_size):
        q_i, k_i, v_i = query[:, :, i], key[:, :, i], value[:, :, i]
        attn = (q_i @ k_i.transpose(-1, -2) * decay_mask[:, :, i]).masked_fill_(mask, 0)
        v_prime = (k_cumdecay[:, :, i]) @ last_recurrent_state
        v_new = v_i - v_prime
        attn_inter = (q_i * g[:, :, i, :, None].exp()) @ last_recurrent_state
        core_attn_out[:, :, i] = attn_inter + attn @ v_new
        last_recurrent_state = (
            last_recurrent_state * g[:, :, i, -1, None, None].exp()
            + (k_i * (g[:, :, i, -1, None] - g[:, :, i]).exp()[..., None]).transpose(
                -1, -2
            )
```
**EN:** This block implements `torch_chunk_gated_delta_rule` and captures one focused piece of the module's behavior. This chunk continues part 1 of the same logical block.
**CN:** 该代码块实现 `torch_chunk_gated_delta_rule`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 1 部分。

### Lines 99-109: function torch chunk gated delta rule (part 2/2) / 函数 torch chunk gated delta rule（第 2/2 部分）
```python
            @ v_new
        )

    if not output_final_state:
        last_recurrent_state = None
    core_attn_out = core_attn_out.reshape(
        core_attn_out.shape[0], core_attn_out.shape[1], -1, core_attn_out.shape[-1]
    )
    core_attn_out = core_attn_out[:, :, :num_heads]
    core_attn_out = core_attn_out.transpose(1, 2).contiguous().to(initial_dtype)
    return core_attn_out, last_recurrent_state
```
**EN:** This block implements `torch_chunk_gated_delta_rule` and captures one focused piece of the module's behavior. This chunk continues part 2 of the same logical block.
**CN:** 该代码块实现 `torch_chunk_gated_delta_rule`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 2 部分。

### Lines 112-146: function chunk gated delta rule update / 函数 chunk gated delta rule update
```python
def chunk_gated_delta_rule_update(
    query,  # [B, T, HK, K]
    key,  # [B, T, HK, K]
    value,  # [B, T, HV, V]
    g,  # [B, T, HV]
    beta,  # [B, T, HV]
    cu_seqlens,  # [N+1]
    initial_state,  # [N, HV, K, V]
    use_qk_l2norm_in_kernel,  # True
):
    num_heads = query.shape[2]
    num_value_heads = value.shape[2]
    batch_size = initial_state.shape[0]
    if num_value_heads // num_heads > 1:
        query = query.repeat_interleave(num_value_heads // num_heads, dim=2)
        key = key.repeat_interleave(num_value_heads // num_heads, dim=2)
    output = torch.empty_like(value)
    final_state = torch.empty_like(initial_state)
    start_q = 0
    for i in range(batch_size):
        end_q = cu_seqlens[i + 1]
        core_attn_outi, last_recurrent_state = torch_chunk_gated_delta_rule(
            query=query[:, start_q:end_q, :, :],
            key=key[:, start_q:end_q, :, :],
            value=value[:, start_q:end_q, :, :],
            g=g[:, start_q:end_q, :],
            beta=beta[:, start_q:end_q, :],
            initial_state=initial_state[i],
            output_final_state=True,
            use_qk_l2norm_in_kernel=use_qk_l2norm_in_kernel,
        )
        output[:, start_q:end_q, :, :] = core_attn_outi
        final_state[i] = last_recurrent_state
        start_q = end_q
    return output, final_state
```
**EN:** This block implements `chunk_gated_delta_rule_update` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `chunk_gated_delta_rule_update`，承担模块行为中的一个聚焦逻辑片段。

### Lines 149-200: function torch recurrent gated delta rule / 函数 torch recurrent gated delta rule
```python
def torch_recurrent_gated_delta_rule(
    query,
    key,
    value,
    g,
    beta,
    initial_state,
    output_final_state,
    use_qk_l2norm_in_kernel=False,
):
    initial_dtype = query.dtype
    if use_qk_l2norm_in_kernel:
        query = l2norm(query, dim=-1, eps=1e-6)
        key = l2norm(key, dim=-1, eps=1e-6)
    query, key, value, beta, g = [
        x.transpose(1, 2).contiguous().to(torch.float32)
        for x in (query, key, value, beta, g)
    ]

    batch_size, num_heads, sequence_length, k_head_dim = key.shape
    v_head_dim = value.shape[-1]
    scale = 1 / (query.shape[-1] ** 0.5)
    query = query * scale

    core_attn_out = torch.zeros(batch_size, num_heads, sequence_length, v_head_dim).to(
        value
    )
    last_recurrent_state = (
        torch.zeros(batch_size, num_heads, k_head_dim, v_head_dim).to(value)
        if initial_state is None
        else initial_state.to(value)
    )

    for i in range(sequence_length):
        q_t = query[:, :, i]
        k_t = key[:, :, i]
        v_t = value[:, :, i]
        g_t = g[:, :, i].exp().unsqueeze(-1).unsqueeze(-1)
        beta_t = beta[:, :, i].unsqueeze(-1)

        last_recurrent_state = last_recurrent_state * g_t
        kv_mem = (last_recurrent_state * k_t.unsqueeze(-1)).sum(dim=-2)
        delta = (v_t - kv_mem) * beta_t
        last_recurrent_state = last_recurrent_state + k_t.unsqueeze(
            -1
        ) * delta.unsqueeze(-2)
        core_attn_out[:, :, i] = (last_recurrent_state * q_t.unsqueeze(-1)).sum(dim=-2)

    if not output_final_state:
        last_recurrent_state = None
    core_attn_out = core_attn_out.transpose(1, 2).contiguous().to(initial_dtype)
    return core_attn_out, last_recurrent_state
```
**EN:** This block implements `torch_recurrent_gated_delta_rule` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `torch_recurrent_gated_delta_rule`，承担模块行为中的一个聚焦逻辑片段。

### Lines 203-226: function sigmoid gating delta rule update / 函数 sigmoid gating delta rule update
```python
def sigmoid_gating_delta_rule_update(
    query,
    key,
    value,
    A_log,
    a,
    dt_bias,
    b,
    initial_state,
    output_final_state,
    use_qk_l2norm_in_kernel=False,
):
    beta = b.sigmoid()
    g = -A_log.float().exp() * softplus(a.float() + dt_bias)
    return torch_recurrent_gated_delta_rule(
        query,
        key,
        value,
        g.unsqueeze(0),
        beta.unsqueeze(0),
        initial_state,
        output_final_state,
        use_qk_l2norm_in_kernel=use_qk_l2norm_in_kernel,
    )
```
**EN:** This block implements `sigmoid_gating_delta_rule_update` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `sigmoid_gating_delta_rule_update`，承担模块行为中的一个聚焦逻辑片段。

### Lines 229-232: function torch gdn gating / 函数 torch gdn gating
```python
def torch_gdn_gating(A_log, a, b, dt_bias):
    return -A_log.float().exp() * softplus(a.float() + dt_bias).unsqueeze(
        0
    ), b.sigmoid().unsqueeze(0)
```
**EN:** This block implements `torch_gdn_gating` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `torch_gdn_gating`，承担模块行为中的一个聚焦逻辑片段。

### Lines 235-235: class TestMambaAttention declaration / 类 TestMambaAttention 声明
```python
class TestMambaAttention(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 236-289: test case chunk gated delta rule / 测试用例 chunk gated delta rule
```python
    def test_chunk_gated_delta_rule(self):
        B, L, HK, HV, EK, EV, N = 1, 100, 3, 6, 64, 64, 4
        seqlens = torch.randint(1, L, (N + 1,))
        seqlens[0] = 0
        cu_seqlens_ = torch.cumsum(seqlens, dim=0).to(torch.int32)
        T = cu_seqlens_[-1].item()
        query_ = torch.rand((B, T, HK, EK), dtype=torch.bfloat16) * 0.05
        key_ = torch.rand((B, T, HK, EK), dtype=torch.bfloat16) * 0.05
        value_ = torch.rand((B, T, HV, EV), dtype=torch.bfloat16) * 0.05
        g_ = torch.rand((B, T, HV), dtype=torch.float32) * 0.05
        beta_ = torch.rand((B, T, HV), dtype=torch.bfloat16) * 0.05
        initial_state_ = torch.rand((N, HV, EK, EV), dtype=torch.float32) * 0.05

        for use_qk_l2norm_in_kernel in [True, False]:
            core_attn_out_ref, last_recurrent_state_ref = chunk_gated_delta_rule_update(
                query=query_,
                key=key_,
                value=value_,
                g=g_,
                beta=beta_,
                cu_seqlens=cu_seqlens_,
                initial_state=initial_state_,
                use_qk_l2norm_in_kernel=use_qk_l2norm_in_kernel,
            )

            query = query_.clone()
            key = key_.clone()
            value = value_.clone()
            g = g_.clone()
            beta = beta_.clone()
            cu_seqlens = cu_seqlens_.clone()
            initial_state = initial_state_.clone()

            core_attn_out, last_recurrent_state = (
                torch.ops.sgl_kernel.chunk_gated_delta_rule_cpu(
                    query=query,
                    key=key,
                    value=value,
                    g=g,
                    beta=beta,
                    initial_state=initial_state,
                    output_final_state=True,
                    cu_seqlens=cu_seqlens,
                    head_first=False,
                    use_qk_l2norm_in_kernel=use_qk_l2norm_in_kernel,
                )
            )
            atol = rtol = precision[core_attn_out.dtype]
            torch.testing.assert_close(
                core_attn_out, core_attn_out_ref, atol=atol, rtol=rtol
            )
            torch.testing.assert_close(
                last_recurrent_state, last_recurrent_state_ref, atol=atol, rtol=rtol
            )
```
**EN:** This test exercises `test_chunk_gated_delta_rule` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chunk_gated_delta_rule`。

### Lines 291-307: test case fused gdn gating / 测试用例 fused gdn gating
```python
    def test_fused_gdn_gating(self):
        dims = [6, 32]
        for dim in dims:
            for A_log_dtype in [torch.float32, torch.bfloat16]:
                A_log = torch.rand(dim, dtype=A_log_dtype)
                a = torch.rand(1024, dim, dtype=torch.bfloat16)
                b = torch.rand(1024, dim, dtype=torch.bfloat16)
                dt_bias = torch.rand(dim, dtype=torch.bfloat16)

                g, beta = torch_gdn_gating(A_log, a, b, dt_bias)
                g_sgl, beta_sgl = torch.ops.sgl_kernel.fused_gdn_gating_cpu(
                    A_log, a, b, dt_bias
                )
                atol = rtol = precision[g.dtype]
                atol2 = rtol2 = precision[beta.dtype]
                torch.testing.assert_close(g, g_sgl, atol=atol, rtol=rtol)
                torch.testing.assert_close(beta, beta_sgl, atol=atol2, rtol=rtol2)
```
**EN:** This test exercises `test_fused_gdn_gating` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_gdn_gating`。

### Lines 309-388: test case fused sigmoid gating delta rule update (part 1/2) / 测试用例 fused sigmoid gating delta rule update（第 1/2 部分）
```python
    def test_fused_sigmoid_gating_delta_rule_update(self):
        batch_size = 1
        num_value_heads = 32
        head_k_dim = 128
        head_v_dim = 128
        num_heads = 16
        seq_len = 1
        attn_tp_size = 1
        key_dim = head_k_dim * num_heads
        value_dim = head_v_dim * num_value_heads
        mixed_qkv_dim = (key_dim * 2 + value_dim) // attn_tp_size
        mixed_qkv = torch.rand(
            seq_len * batch_size, mixed_qkv_dim, dtype=torch.bfloat16
        )
        query, key, value = torch.split(
            mixed_qkv,
            [
                key_dim // attn_tp_size,
                key_dim // attn_tp_size,
                value_dim // attn_tp_size,
            ],
            dim=-1,
        )
        query = query.view(1, seq_len, num_heads, head_k_dim)
        key = key.view(1, seq_len, num_heads, head_k_dim)
        value = value.view(1, seq_len, num_value_heads, head_v_dim)
        A_log = torch.rand(num_value_heads, dtype=torch.float32)
        a = torch.rand(batch_size, num_value_heads, dtype=torch.bfloat16)
        b = torch.rand(batch_size, num_value_heads, dtype=torch.bfloat16)
        dt_bias = torch.rand(num_value_heads, dtype=torch.bfloat16)
        ssm_states = torch.rand(
            513, num_value_heads, head_k_dim, head_v_dim, dtype=torch.float32
        )
        cache_indices = torch.randint(0, 513, (batch_size,), dtype=torch.int32)
        query_start_loc = torch.tensor([0, 1], dtype=torch.int32)
        use_qk_l2norm_in_kernel = True
        query_ref = query.clone()
        key_ref = key.clone()
        if num_value_heads // num_heads > 1:
            query_ref = query_ref.repeat_interleave(num_value_heads // num_heads, dim=2)
            key_ref = key_ref.repeat_interleave(num_value_heads // num_heads, dim=2)
        for A_log_dtype in [torch.float32, torch.bfloat16]:
            A_log = A_log.to(A_log_dtype)
            core_attn_out_ref, last_recurrent_state_ref = (
                sigmoid_gating_delta_rule_update(
                    query_ref.transpose(0, 1),
                    key_ref.transpose(0, 1),
                    value.transpose(0, 1),
                    A_log,
                    a,
                    dt_bias,
                    b,
                    initial_state=ssm_states[cache_indices],
                    output_final_state=True,
                    use_qk_l2norm_in_kernel=use_qk_l2norm_in_kernel,
                )
            )
            core_attn_out = (
                torch.ops.sgl_kernel.fused_sigmoid_gating_delta_rule_update_cpu(
                    A_log=A_log,
                    dt_bias=dt_bias,
                    q=query,
                    k=key,
                    v=value,
                    a=a,
                    b=b,
                    initial_state_source=ssm_states,
                    initial_state_indices=cache_indices,
                    cu_seqlens=query_start_loc,
                    use_qk_l2norm_in_kernel=use_qk_l2norm_in_kernel,
                    softplus_beta=1.0,
                    softplus_threshold=20.0,
                )
            )
            last_recurrent_state = ssm_states[cache_indices]
            atol = rtol = precision[core_attn_out.dtype]
            torch.testing.assert_close(
                core_attn_out, core_attn_out_ref, atol=atol, rtol=rtol
            )
            torch.testing.assert_close(
```
**EN:** This test exercises `test_fused_sigmoid_gating_delta_rule_update` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_sigmoid_gating_delta_rule_update`。 这一段对应同一逻辑块的第 1 部分。

### Lines 389-390: test case fused sigmoid gating delta rule update (part 2/2) / 测试用例 fused sigmoid gating delta rule update（第 2/2 部分）
```python
                last_recurrent_state, last_recurrent_state_ref, atol=atol, rtol=rtol
            )
```
**EN:** This test exercises `test_fused_sigmoid_gating_delta_rule_update` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_sigmoid_gating_delta_rule_update`。 这一段对应同一逻辑块的第 2 部分。

### Lines 393-394: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `l2norm`: This function is intended to align with the l2norm implementation in the FLA library. / 该代码块实现 `l2norm`，承担模块行为中的一个聚焦逻辑片段。
- `torch_chunk_gated_delta_rule`: This block implements `torch_chunk_gated_delta_rule` and captures one focused piece of the module's behavior. / 该代码块实现 `torch_chunk_gated_delta_rule`，承担模块行为中的一个聚焦逻辑片段。
- `chunk_gated_delta_rule_update`: This block implements `chunk_gated_delta_rule_update` and captures one focused piece of the module's behavior. / 该代码块实现 `chunk_gated_delta_rule_update`，承担模块行为中的一个聚焦逻辑片段。
- `torch_recurrent_gated_delta_rule`: This block implements `torch_recurrent_gated_delta_rule` and captures one focused piece of the module's behavior. / 该代码块实现 `torch_recurrent_gated_delta_rule`，承担模块行为中的一个聚焦逻辑片段。
- `sigmoid_gating_delta_rule_update`: This block implements `sigmoid_gating_delta_rule_update` and captures one focused piece of the module's behavior. / 该代码块实现 `sigmoid_gating_delta_rule_update`，承担模块行为中的一个聚焦逻辑片段。
- `torch_gdn_gating`: This block implements `torch_gdn_gating` and captures one focused piece of the module's behavior. / 该代码块实现 `torch_gdn_gating`，承担模块行为中的一个聚焦逻辑片段。
- `TestMambaAttention`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestMambaAttention.test_chunk_gated_delta_rule`: This test exercises `test_chunk_gated_delta_rule` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chunk_gated_delta_rule`。
- `TestMambaAttention.test_fused_gdn_gating`: This test exercises `test_fused_gdn_gating` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_gdn_gating`。
- `TestMambaAttention.test_fused_sigmoid_gating_delta_rule_update`: This test exercises `test_fused_sigmoid_gating_delta_rule_update` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_sigmoid_gating_delta_rule_update`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `torch`, `torch.nn.functional`, `utils`
- **Internal modules / 内部模块**: `sglang.test.test_utils`

- **Total lines / 总行数**: 394
