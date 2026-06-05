# test_qkv_proj_with_rope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/srt/cpu/test_qkv_proj_with_rope.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates qkv proj with rope behavior in SGLang's srt / cpu area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 SRT / CPU 领域中与 qkv proj with rope 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: module imports and dependencies / 模块导入与依赖
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
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `torch`, `utils`, `sglang.srt.layers.quantization.fp8_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `torch`, `utils`, `sglang.srt.layers.quantization.fp8_utils`。

### Lines 15-29: module-level constants and configuration / 模块级常量与配置
```python
convert_weight_packed = torch.ops.sgl_kernel.convert_weight_packed
qkv_proj_with_rope = torch.ops.sgl_kernel.qkv_proj_with_rope
qkv_proj_with_rope_fused_weight = torch.ops.sgl_kernel.qkv_proj_with_rope_fused_weight
torch.manual_seed(1234)
# constants
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
**EN:** This block defines shared names such as `convert_weight_packed`, `qkv_proj_with_rope`, `qkv_proj_with_rope_fused_weight`, `kv_lora_rank`, `qk_head_dim`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `convert_weight_packed`, `qkv_proj_with_rope`, `qkv_proj_with_rope_fused_weight`, `kv_lora_rank`, `qk_head_dim` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 32-37: function layernorm / 函数 layernorm
```python
def layernorm(x, weight, variance_epsilon=1e-6, residual=None):
    orig_dtype = x.dtype
    x = x.to(torch.float32)
    variance = x.pow(2).mean(dim=-1, keepdim=True)
    x = x * torch.rsqrt(variance + variance_epsilon)
    return (x * weight).to(orig_dtype)
```
**EN:** This block implements `layernorm` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `layernorm`，承担模块行为中的一个聚焦逻辑片段。

### Lines 40-52: function rotary emb / 函数 rotary emb
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
**EN:** This block implements `rotary_emb` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `rotary_emb`，承担模块行为中的一个聚焦逻辑片段。

### Lines 55-87: function native torch / 函数 native torch
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
**EN:** This block implements `native_torch` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `native_torch`，承担模块行为中的一个聚焦逻辑片段。

### Lines 90-133: function native torch int8 / 函数 native torch int8
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
**EN:** This block implements `native_torch_int8` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `native_torch_int8`，承担模块行为中的一个聚焦逻辑片段。

### Lines 136-136: class TestQKVProjWithROPE declaration / 类 TestQKVProjWithROPE 声明
```python
class TestQKVProjWithROPE(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 137-216: test case bf16 qkv proj with rope (part 1/2) / 测试用例 bf16 qkv proj with rope（第 1/2 部分）
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
```
**EN:** This test exercises `test_bf16_qkv_proj_with_rope` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bf16_qkv_proj_with_rope`。 这一段对应同一逻辑块的第 1 部分。

### Lines 217-221: test case bf16 qkv proj with rope (part 2/2) / 测试用例 bf16 qkv proj with rope（第 2/2 部分）
```python
        torch.testing.assert_close(k_ref, k_out, atol=atol, rtol=rtol)
        torch.testing.assert_close(v_ref, v_out, atol=atol, rtol=rtol)
        torch.testing.assert_close(fused_q_out, q_out)
        torch.testing.assert_close(fused_k_out, k_out)
        torch.testing.assert_close(fused_v_out, v_out)
```
**EN:** This test exercises `test_bf16_qkv_proj_with_rope` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bf16_qkv_proj_with_rope`。 这一段对应同一逻辑块的第 2 部分。

### Lines 223-302: test case int8 qkv proj with rope (part 1/2) / 测试用例 int8 qkv proj with rope（第 1/2 部分）
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
            True,
```
**EN:** This test exercises `test_int8_qkv_proj_with_rope` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_int8_qkv_proj_with_rope`。 这一段对应同一逻辑块的第 1 部分。

### Lines 303-314: test case int8 qkv proj with rope (part 2/2) / 测试用例 int8 qkv proj with rope（第 2/2 部分）
```python
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
**EN:** This test exercises `test_int8_qkv_proj_with_rope` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_int8_qkv_proj_with_rope`。 这一段对应同一逻辑块的第 2 部分。

### Lines 316-395: test case fp8 qkv proj with rope (part 1/2) / 测试用例 fp8 qkv proj with rope（第 1/2 部分）
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
**EN:** This test exercises `test_fp8_qkv_proj_with_rope` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fp8_qkv_proj_with_rope`。 这一段对应同一逻辑块的第 1 部分。

### Lines 396-436: test case fp8 qkv proj with rope (part 2/2) / 测试用例 fp8 qkv proj with rope（第 2/2 部分）
```python
            [scale_block_size_N, scale_block_size_K],
        )

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
**EN:** This test exercises `test_fp8_qkv_proj_with_rope` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fp8_qkv_proj_with_rope`。 这一段对应同一逻辑块的第 2 部分。

### Lines 439-440: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `layernorm`: This block implements `layernorm` and captures one focused piece of the module's behavior. / 该代码块实现 `layernorm`，承担模块行为中的一个聚焦逻辑片段。
- `rotary_emb`: This block implements `rotary_emb` and captures one focused piece of the module's behavior. / 该代码块实现 `rotary_emb`，承担模块行为中的一个聚焦逻辑片段。
- `native_torch`: This block implements `native_torch` and captures one focused piece of the module's behavior. / 该代码块实现 `native_torch`，承担模块行为中的一个聚焦逻辑片段。
- `native_torch_int8`: This block implements `native_torch_int8` and captures one focused piece of the module's behavior. / 该代码块实现 `native_torch_int8`，承担模块行为中的一个聚焦逻辑片段。
- `TestQKVProjWithROPE`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestQKVProjWithROPE.test_bf16_qkv_proj_with_rope`: This test exercises `test_bf16_qkv_proj_with_rope` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_bf16_qkv_proj_with_rope`。
- `TestQKVProjWithROPE.test_int8_qkv_proj_with_rope`: This test exercises `test_int8_qkv_proj_with_rope` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_int8_qkv_proj_with_rope`。
- `TestQKVProjWithROPE.test_fp8_qkv_proj_with_rope`: This test exercises `test_fp8_qkv_proj_with_rope` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fp8_qkv_proj_with_rope`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `torch`, `utils`
- **Internal modules / 内部模块**: `sglang.srt.layers.quantization.fp8_utils`, `sglang.srt.layers.rotary_embedding.utils`, `sglang.test.test_utils`

- **Total lines / 总行数**: 440
