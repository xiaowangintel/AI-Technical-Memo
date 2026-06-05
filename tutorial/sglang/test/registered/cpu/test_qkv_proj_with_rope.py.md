# test_qkv_proj_with_rope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/cpu/test_qkv_proj_with_rope.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on cpu qkv proj with rope in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 cpu qkv proj with rope 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Import dependencies
```python
import unittest

import torch
from utils import (
    convert_weight,
    native_w8a8_per_token_matmul,
    per_token_quant_int8,
    precision,
)

from sglang.srt.layers.quantization.fp8_utils import input_to_float8
from sglang.srt.layers.rotary_embedding.utils import apply_rotary_emb
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 16-16: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-b-test-cpu")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 18-20: Define module constants
```python
convert_weight_packed = torch.ops.sgl_kernel.convert_weight_packed
qkv_proj_with_rope = torch.ops.sgl_kernel.qkv_proj_with_rope
qkv_proj_with_rope_fused_weight = torch.ops.sgl_kernel.qkv_proj_with_rope_fused_weight
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 21-21: Implement expr logic
```python
torch.manual_seed(1234)
```
**EN:** This block contains supporting statements that connect the surrounding setup and assertions.
**CN:** 该代码块包含连接周边初始化与断言的辅助语句。

### Lines 23-32: Define module constants
```python
kv_lora_rank = 512
qk_head_dim = 192
qk_nope_head_dim = 128
qk_rope_head_dim = 64
rotary_dim = qk_rope_head_dim
num_heads = 22
q_lora_rank = 1536
hidden_size = 7168
B = 1
eps = 1e-6
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 35-40: Define helper: layernorm
```python
def layernorm(x, weight, variance_epsilon=1e-6, residual=None):
    orig_dtype = x.dtype
    x = x.to(torch.float32)
    variance = x.pow(2).mean(dim=-1, keepdim=True)
    x = x * torch.rsqrt(variance + variance_epsilon)
    return (x * weight).to(orig_dtype)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 43-55: Define helper: rotary emb
```python
def rotary_emb(q_pe, k_pe, pos, cos_sin_cache):
    orig_dtype = q_pe.dtype
    q_pe = q_pe.float()
    k_pe = k_pe.float()
    cos_sin_cache = cos_sin_cache.float()

    query_rot = q_pe[..., :rotary_dim]
    key_rot = k_pe[..., :rotary_dim]
    cos_sin = cos_sin_cache[pos]
    cos, sin = cos_sin.chunk(2, dim=-1)
    query_rot = apply_rotary_emb(query_rot, cos, sin, False)
    key_rot = apply_rotary_emb(key_rot, cos, sin, False)
    return query_rot.to(orig_dtype), key_rot.to(orig_dtype)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 58-90: Define helper: native torch
```python
def native_torch(
    q_input,
    hidden_states,
    q_a_proj_weight,
    norm_weight1,
    q_b_proj_weight,
    w_kc,
    kv_a_proj_weight,
    norm_weight2,
    pos,
    cos_sin_cache,
):

    q = torch.matmul(hidden_states, q_a_proj_weight.t())
    q = layernorm(q, norm_weight1)
    q = torch.matmul(q, q_b_proj_weight.t()).view(-1, num_heads, qk_head_dim)

    q_nope, q_pe = q.split([qk_nope_head_dim, qk_rope_head_dim], dim=-1)
    q_nope_out = torch.bmm(q_nope.transpose(0, 1), w_kc)

    q_input[..., :kv_lora_rank] = q_nope_out.transpose(0, 1)
    latent_cache = torch.matmul(hidden_states, kv_a_proj_weight.t())
    v_input = latent_cache[..., :kv_lora_rank]
    v_input = layernorm(v_input.contiguous(), norm_weight2).unsqueeze(1)
    k_input = latent_cache.unsqueeze(1)
    k_input[..., :kv_lora_rank] = v_input
    k_pe = k_input[..., kv_lora_rank:]

    q_pe, k_pe = rotary_emb(q_pe, k_pe, pos, cos_sin_cache)
    q_input[..., kv_lora_rank:] = q_pe
    k_input[..., kv_lora_rank:] = k_pe

    return q_input, k_input, v_input
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 93-136: Define helper: native torch int8
```python
def native_torch_int8(
    q_input,
    hidden_states,
    w1_q,
    w1_s,
    norm_weight1,
    w2_q,
    w2_s,
    w_kc,
    w3_q,
    w3_s,
    norm_weight2,
    pos,
    cos_sin_cache,
):

    a_q, a_s = per_token_quant_int8(hidden_states)
    q = native_w8a8_per_token_matmul(a_q, w1_q, a_s, w1_s, None, torch.bfloat16)
    q = layernorm(q, norm_weight1)

    a_q, a_s = per_token_quant_int8(q)
    q = native_w8a8_per_token_matmul(a_q, w2_q, a_s, w2_s, None, torch.bfloat16).view(
        -1, num_heads, qk_head_dim
    )

    q_nope, q_pe = q.split([qk_nope_head_dim, qk_rope_head_dim], dim=-1)
    q_nope_out = torch.bmm(q_nope.transpose(0, 1), w_kc)

    q_input[..., :kv_lora_rank] = q_nope_out.transpose(0, 1)
    a_q, a_s = per_token_quant_int8(hidden_states)
    latent_cache = native_w8a8_per_token_matmul(
        a_q, w3_q, a_s, w3_s, None, torch.bfloat16
    )
    v_input = latent_cache[..., :kv_lora_rank]
    v_input = layernorm(v_input.contiguous(), norm_weight2).unsqueeze(1)
    k_input = latent_cache.unsqueeze(1)
    k_input[..., :kv_lora_rank] = v_input
    k_pe = k_input[..., kv_lora_rank:]

    q_pe, k_pe = rotary_emb(q_pe, k_pe, pos, cos_sin_cache)
    q_input[..., kv_lora_rank:] = q_pe
    k_input[..., kv_lora_rank:] = k_pe

    return q_input, k_input, v_input
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 139-139: Define class TestQKVProjWithROPE
```python
class TestQKVProjWithROPE(CustomTestCase):
```
**EN:** This declaration introduces the `TestQKVProjWithROPE` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestQKVProjWithROPE` 测试类，并说明它通过继承承担的职责。

### Lines 140-175: Run test: bf16 qkv proj with rope (part 1)
```python
    def test_bf16_qkv_proj_with_rope(self):
        dtype = torch.bfloat16
        hidden_states = torch.randn(B, hidden_size, dtype=dtype) / hidden_size
        q_input = torch.empty(
            B, num_heads, kv_lora_rank + qk_rope_head_dim, dtype=dtype
        )
        q_a_proj_weight = torch.randn(q_lora_rank, hidden_size, dtype=dtype) * 0.1
        norm_weight1 = torch.randn(q_lora_rank, dtype=dtype)
        q_b_proj_weight = (
            torch.randn(num_heads * qk_head_dim, q_lora_rank, dtype=dtype) * 0.1
        )
        w_kc = torch.randn(num_heads, kv_lora_rank, qk_nope_head_dim, dtype=dtype) * 0.1
        kv_a_proj_weight = (
            torch.randn(kv_lora_rank + qk_rope_head_dim, hidden_size, dtype=dtype) * 0.1
        )
        fused_weight = torch.cat([q_a_proj_weight, kv_a_proj_weight], dim=0)
        norm_weight2 = torch.randn(kv_lora_rank, dtype=dtype)
        pos = torch.randint(10, 100, (B,))
        cos_sin_cache = torch.randn(100, rotary_dim, dtype=dtype)
        q_ref, k_ref, v_ref = native_torch(
            q_input,
            hidden_states,
            q_a_proj_weight,
            norm_weight1,
            q_b_proj_weight,
            w_kc.transpose(1, 2),
            kv_a_proj_weight,
            norm_weight2,
            pos,
            cos_sin_cache,
        )
        qa_packed = convert_weight_packed(q_a_proj_weight)
        qb_packed = convert_weight_packed(q_b_proj_weight)
        kva_packed = convert_weight_packed(kv_a_proj_weight)
        wkc_packed = convert_weight_packed(w_kc)
        fused_weight_packed = convert_weight_packed(fused_weight)
```
**EN:** This test method exercises bf16 qkv proj with rope and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 bf16 qkv proj with rope 场景，并验证观测到的行为是否符合预期契约。

### Lines 177-224: Run test: bf16 qkv proj with rope (part 2)
```python
        q_out, k_out, v_out = qkv_proj_with_rope(
            hidden_states,
            qa_packed,
            qb_packed,
            kva_packed,
            wkc_packed,
            norm_weight1,
            norm_weight2,
            pos,
            cos_sin_cache,
            eps,
            False,
            False,
            None,
            None,
            None,
            None,
            True,
            None,
        )
        fused_q_out, fused_k_out, fused_v_out = qkv_proj_with_rope_fused_weight(
            hidden_states,
            fused_weight_packed,
            qb_packed,
            wkc_packed,
            norm_weight1,
            norm_weight2,
            pos,
            cos_sin_cache,
            eps,
            False,
            False,
            None,
            None,
            None,
            True,
            None,
            q_lora_rank,
            kv_lora_rank,
            qk_rope_head_dim,
        )
        atol = rtol = precision[q_ref.dtype]
        torch.testing.assert_close(q_ref, q_out, atol=atol, rtol=rtol)
        torch.testing.assert_close(k_ref, k_out, atol=atol, rtol=rtol)
        torch.testing.assert_close(v_ref, v_out, atol=atol, rtol=rtol)
        torch.testing.assert_close(fused_q_out, q_out)
        torch.testing.assert_close(fused_k_out, k_out)
        torch.testing.assert_close(fused_v_out, v_out)
```
**EN:** This test method exercises bf16 qkv proj with rope and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 bf16 qkv proj with rope 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 226-243: Run test: int8 qkv proj with rope (part 1)
```python
    def test_int8_qkv_proj_with_rope(self):
        dtype = torch.bfloat16
        hidden_states = torch.randn(B, hidden_size, dtype=dtype) / hidden_size
        q_input = torch.empty(
            B, num_heads, kv_lora_rank + qk_rope_head_dim, dtype=dtype
        )
        q_a_proj_weight = torch.randn(q_lora_rank, hidden_size, dtype=dtype) * 0.1
        norm_weight1 = torch.randn(q_lora_rank, dtype=dtype)
        q_b_proj_weight = (
            torch.randn(num_heads * qk_head_dim, q_lora_rank, dtype=dtype) * 0.1
        )
        w_kc = torch.randn(num_heads, kv_lora_rank, qk_nope_head_dim, dtype=dtype) * 0.1
        kv_a_proj_weight = (
            torch.randn(kv_lora_rank + qk_rope_head_dim, hidden_size, dtype=dtype) * 0.1
        )
        norm_weight2 = torch.randn(kv_lora_rank, dtype=dtype)
        pos = torch.randint(10, 100, (B,))
        cos_sin_cache = torch.randn(100, rotary_dim, dtype=dtype)
```
**EN:** This test method exercises int8 qkv proj with rope and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 int8 qkv proj with rope 场景，并验证观测到的行为是否符合预期契约。

### Lines 245-304: Run test: int8 qkv proj with rope (part 2)
```python
        w1_q, w1_s = per_token_quant_int8(q_a_proj_weight)
        w2_q, w2_s = per_token_quant_int8(q_b_proj_weight)
        w3_q, w3_s = per_token_quant_int8(kv_a_proj_weight)
        q_ref, k_ref, v_ref = native_torch_int8(
            q_input,
            hidden_states,
            w1_q,
            w1_s,
            norm_weight1,
            w2_q,
            w2_s,
            w_kc.transpose(1, 2),
            w3_q,
            w3_s,
            norm_weight2,
            pos,
            cos_sin_cache,
        )
        w1_q_packed = convert_weight_packed(w1_q)
        w2_q_packed = convert_weight_packed(w2_q)
        w3_q_packed = convert_weight_packed(w3_q)
        wkc_packed = convert_weight_packed(w_kc)
        q_out, k_out, v_out = qkv_proj_with_rope(
            hidden_states,
            w1_q_packed,
            w2_q_packed,
            w3_q_packed,
            wkc_packed,
            norm_weight1,
            norm_weight2,
            pos,
            cos_sin_cache,
            eps,
            True,
            False,
            w1_s,
            w2_s,
            w3_s,
            None,
            True,
            None,
        )
        fused_weight = torch.cat([w1_q, w3_q], dim=0)
        fused_weight_s = torch.cat([w1_s, w3_s], dim=0)
        w_fused_q_packed = convert_weight_packed(fused_weight)
        fused_q_out, fused_k_out, fused_v_out = qkv_proj_with_rope_fused_weight(
            hidden_states,
            w_fused_q_packed,
            w2_q_packed,
            wkc_packed,
            norm_weight1,
            norm_weight2,
            pos,
            cos_sin_cache,
            eps,
            True,
            False,
            fused_weight_s,
            w2_s,
            None,
```
**EN:** This test method exercises int8 qkv proj with rope and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 int8 qkv proj with rope 场景，并验证观测到的行为是否符合预期契约。

### Lines 305-317: Run test: int8 qkv proj with rope (part 3)
```python
            True,
            None,
            q_lora_rank,
            kv_lora_rank,
            qk_rope_head_dim,
        )
        atol = rtol = precision[q_ref.dtype]
        torch.testing.assert_close(q_ref, q_out, atol=atol, rtol=rtol)
        torch.testing.assert_close(k_ref, k_out, atol=atol, rtol=rtol)
        torch.testing.assert_close(v_ref, v_out, atol=atol, rtol=rtol)
        torch.testing.assert_close(fused_q_out, q_out)
        torch.testing.assert_close(fused_k_out, k_out)
        torch.testing.assert_close(fused_v_out, v_out)
```
**EN:** This test method exercises int8 qkv proj with rope and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 int8 qkv proj with rope 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 319-337: Run test: fp8 qkv proj with rope (part 1)
```python
    def test_fp8_qkv_proj_with_rope(self):
        dtype = torch.bfloat16
        hidden_states = torch.randn(B, hidden_size, dtype=dtype) / hidden_size
        q_input = torch.empty(
            B, num_heads, kv_lora_rank + qk_rope_head_dim, dtype=dtype
        )
        q_a_proj_weight = torch.randn(q_lora_rank, hidden_size, dtype=dtype) * 0.1
        norm_weight1 = torch.randn(q_lora_rank, dtype=dtype)
        q_b_proj_weight = (
            torch.randn(num_heads * qk_head_dim, q_lora_rank, dtype=dtype) * 0.1
        )
        w_kc = torch.randn(num_heads, kv_lora_rank, qk_nope_head_dim, dtype=dtype) * 0.1
        w_kc_q, w_kc_s = input_to_float8(w_kc)
        kv_a_proj_weight = (
            torch.randn(kv_lora_rank + qk_rope_head_dim, hidden_size, dtype=dtype) * 0.1
        )
        norm_weight2 = torch.randn(kv_lora_rank, dtype=dtype)
        pos = torch.randint(10, 100, (B,))
        cos_sin_cache = torch.randn(100, rotary_dim, dtype=dtype)
```
**EN:** This test method exercises fp8 qkv proj with rope and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 fp8 qkv proj with rope 场景，并验证观测到的行为是否符合预期契约。

### Lines 339-398: Run test: fp8 qkv proj with rope (part 2)
```python
        scale_block_size_N = 128
        scale_block_size_K = 128
        fp8_q_a_proj_weight, q_a_proj_weight_scale_inv, q_a_proj_weight_dq = (
            convert_weight(
                q_a_proj_weight,
                [scale_block_size_N, scale_block_size_K],
                torch.bfloat16,
            )
        )
        fp8_q_b_proj_weight, q_b_proj_weight_scale_inv, q_b_proj_weight_dq = (
            convert_weight(
                q_b_proj_weight,
                [scale_block_size_N, scale_block_size_K],
                torch.bfloat16,
            )
        )
        (
            fp8_kv_a_proj_with_mqa_weight,
            kv_a_proj_with_mqa_weight_scale_inv,
            kv_a_proj_with_mqa_weight_dq,
        ) = convert_weight(
            kv_a_proj_weight, [scale_block_size_N, scale_block_size_K], torch.bfloat16
        )
        w_kc_dq = w_kc_q.to(torch.bfloat16) * w_kc_s
        q_ref, k_ref, v_ref = native_torch(
            q_input,
            hidden_states,
            q_a_proj_weight_dq,
            norm_weight1,
            q_b_proj_weight_dq,
            w_kc_dq.transpose(1, 2),
            kv_a_proj_with_mqa_weight_dq,
            norm_weight2,
            pos,
            cos_sin_cache,
        )
        fp8_q_a_proj_weight_packed = convert_weight_packed(fp8_q_a_proj_weight)
        fp8_q_b_proj_weight_packed = convert_weight_packed(fp8_q_b_proj_weight)
        fp8_kv_a_proj_with_mqa_weight_packed = convert_weight_packed(
            fp8_kv_a_proj_with_mqa_weight
        )
        w_kc_q = convert_weight_packed(w_kc_q)
        q_out, k_out, v_out = qkv_proj_with_rope(
            hidden_states,
            fp8_q_a_proj_weight_packed,
            fp8_q_b_proj_weight_packed,
            fp8_kv_a_proj_with_mqa_weight_packed,
            w_kc_q,
            norm_weight1,
            norm_weight2,
            pos,
            cos_sin_cache,
            eps,
            False,
            True,
            q_a_proj_weight_scale_inv.float(),
            q_b_proj_weight_scale_inv.float(),
            kv_a_proj_with_mqa_weight_scale_inv.float(),
            w_kc_s,
            True,
```
**EN:** This test method exercises fp8 qkv proj with rope and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 fp8 qkv proj with rope 场景，并验证观测到的行为是否符合预期契约。

### Lines 399-400: Run test: fp8 qkv proj with rope (part 3)
```python
            [scale_block_size_N, scale_block_size_K],
        )
```
**EN:** This test method exercises fp8 qkv proj with rope and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 fp8 qkv proj with rope 场景，并验证观测到的行为是否符合预期契约。

### Lines 402-439: Run test: fp8 qkv proj with rope (part 4)
```python
        fused_weight = torch.cat(
            [fp8_q_a_proj_weight, fp8_kv_a_proj_with_mqa_weight], dim=0
        )
        fused_weight_s = torch.cat(
            [q_a_proj_weight_scale_inv, kv_a_proj_with_mqa_weight_scale_inv], dim=0
        )
        fused_weight_packed = convert_weight_packed(fused_weight)
        fused_q_out, fused_k_out, fused_v_out = qkv_proj_with_rope_fused_weight(
            hidden_states,
            fused_weight_packed,
            fp8_q_b_proj_weight_packed,
            w_kc_q,
            norm_weight1,
            norm_weight2,
            pos,
            cos_sin_cache,
            eps,
            False,
            True,
            fused_weight_s.float(),
            q_b_proj_weight_scale_inv.float(),
            w_kc_s,
            True,
            [scale_block_size_N, scale_block_size_K],
            q_lora_rank,
            kv_lora_rank,
            qk_rope_head_dim,
        )
        atol = rtol = precision[q_ref.dtype]
        # Due to the change in multiplication order, the error is amplified.
        # In the model, with fewer layers, this doesn't cause issues, but in
        # tests with more layers, we need to enlarge the tolerance to pass the tests.
        torch.testing.assert_close(q_ref, q_out, atol=1e-1, rtol=1e-1)
        torch.testing.assert_close(k_ref, k_out, atol=atol, rtol=rtol)
        torch.testing.assert_close(v_ref, v_out, atol=atol, rtol=rtol)
        torch.testing.assert_close(fused_q_out, q_out)
        torch.testing.assert_close(fused_k_out, k_out)
        torch.testing.assert_close(fused_v_out, v_out)
```
**EN:** This test method exercises fp8 qkv proj with rope and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 fp8 qkv proj with rope 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 442-443: Expose unittest entrypoint
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This conditional keeps the module executable as a standalone unittest target. It also keeps a standalone unittest entrypoint available.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。 其中还会保留可独立运行的 unittest 入口。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。
- Shared base classes centralize common setup, assertions, and diagnostics. / 共享基类集中封装了通用初始化、断言与诊断逻辑。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.layers.quantization.fp8_utils`, `sglang.srt.layers.rotary_embedding.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `torch`, `unittest`, `utils`
- Notable symbols / 关键符号: `CustomTestCase`, `unittest.main`
