# test_rope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/cpu/test_rope.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on cpu rope in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 cpu rope 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Import dependencies
```python
import unittest

import torch
from utils import precision

from sglang.srt.layers.rotary_embedding import (
    MRotaryEmbedding,
    RotaryEmbedding,
)
from sglang.srt.layers.rotary_embedding.rope_variant import (
    DeepseekScalingRotaryEmbedding,
    apply_rotary_pos_emb_native,
)
from sglang.srt.server_args import ServerArgs, set_global_server_args_for_scheduler
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 18-18: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-b-test-cpu")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 20-20: Implement expr logic
```python
torch.manual_seed(1234)
```
**EN:** This block contains supporting statements that connect the surrounding setup and assertions.
**CN:** 该代码块包含连接周边初始化与断言的辅助语句。

### Lines 23-23: Define class TestROPE
```python
class TestROPE(CustomTestCase):
```
**EN:** This declaration introduces the `TestROPE` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestROPE` 测试类，并说明它通过继承承担的职责。

### Lines 24-73: Run test: mrope (part 1)
```python
    def test_mrope(self):
        torch.manual_seed(100)
        head_size = 128
        seq_len = 512
        num_heads = 16
        num_kv_heads = 1
        rotary_dim = 128
        max_pos = 262144
        base = 5000000
        is_neox_style = True
        dtype = torch.bfloat16
        mrope_section = [24, 20, 20]
        mrope_interleaved = True
        positions_mrope = torch.randint(0, max_pos, (3, seq_len))
        positions_text = torch.randint(0, max_pos, (seq_len,))
        set_global_server_args_for_scheduler(ServerArgs(model_path="dummy"))

        test_config = [
            # (dtype, is_neox_stype, mrope_interleaved, positions, mrope_section)
            (torch.bfloat16, False, True, positions_mrope, mrope_section),
            (torch.bfloat16, False, False, positions_mrope, mrope_section),
            (torch.bfloat16, False, False, positions_text, None),
            (torch.bfloat16, True, True, positions_mrope, mrope_section),
            (torch.bfloat16, True, False, positions_mrope, mrope_section),
            (torch.bfloat16, True, False, positions_text, None),
        ]
        for (
            dtype,
            is_neox_style,
            mrope_interleaved,
            positions,
            mrope_section,
        ) in test_config:
            rope = MRotaryEmbedding(
                head_size,
                rotary_dim,
                max_pos,
                base,
                is_neox_style,
                dtype,
                mrope_section,
                mrope_interleaved,
            )
            enable_autocast = True

            with torch.no_grad(), torch.amp.autocast("cpu", enabled=enable_autocast):
                q = torch.randn(seq_len, num_heads * head_size, dtype=dtype)
                q_clone = q.clone()
                k = torch.randn(seq_len, num_kv_heads * head_size, dtype=dtype)
                k_clone = k.clone()
```
**EN:** This test method exercises mrope and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 mrope 场景，并验证观测到的行为是否符合预期契约。

### Lines 75-94: Run test: mrope (part 2)
```python
                # ref kernel
                q_ref, k_ref = rope.forward_native(
                    query=q,
                    key=k,
                    positions=positions,
                )
                # fused rope kernel
                q_sgl, k_sgl = torch.ops.sgl_kernel.multimodal_rotary_embedding_cpu(
                    positions,
                    q_clone,
                    k_clone,
                    rope.head_size,
                    rope.cos_sin_cache,
                    rope.mrope_section,
                    rope.mrope_interleaved,
                    is_neox_style,
                )
                atol = rtol = precision[q_ref.dtype]
                torch.testing.assert_close(q_ref, q_sgl, atol=atol, rtol=rtol)
                torch.testing.assert_close(k_ref, k_sgl, atol=atol, rtol=rtol)
```
**EN:** This test method exercises mrope and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 mrope 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 96-147: Run test: deepseek v2 rope (part 1)
```python
    def test_deepseek_v2_rope(self):
        num_head = 16
        seq_len = 1024
        q_head_dim = 192
        qk_nope_head_dim = 128
        qk_rope_head_dim = 64
        max_pos = 256
        k_dim = 576
        rotary_dim = 64
        is_neox_style = False
        set_global_server_args_for_scheduler(ServerArgs(model_path="dummy"))

        # Create cos_sin_cache
        freqs = torch.rand(max_pos, qk_rope_head_dim // 2)
        cos = freqs.cos() * 0.7
        sin = freqs.sin() * 0.7
        cos_sin_cache = torch.cat((cos, sin), dim=-1).to(torch.bfloat16)
        positions = torch.randint(0, max_pos, (seq_len,))

        rope = DeepseekScalingRotaryEmbedding(
            qk_rope_head_dim,
            rotary_dim,
            max_pos,
            16,  # not used since cos_sin_cache is provided
            is_neox_style,
            1.0,
            torch.bfloat16,
            device="cpu",
        )
        rope.register_buffer("cos_sin_cache", cos_sin_cache)

        for dtype in [torch.bfloat16]:
            enable_autocast = True

            with torch.no_grad(), torch.amp.autocast("cpu", enabled=enable_autocast):
                q = torch.randn(seq_len, num_head, q_head_dim, dtype=dtype)
                q_clone = q.clone()
                k = torch.randn(seq_len, 1, k_dim, dtype=dtype)
                k_clone = k.clone()
                _, q_pe = q.split([qk_nope_head_dim, qk_rope_head_dim], dim=-1)
                _, q_pe_clone = q_clone.split(
                    [qk_nope_head_dim, qk_rope_head_dim], dim=-1
                )
                k_pe = k[:, :, k_dim - qk_rope_head_dim :]
                k_pe_clone = k_clone[:, :, k_dim - qk_rope_head_dim :]

                # ref kernel
                q_pe, k_pe = rope.forward_native(
                    query=q_pe,
                    key=k_pe,
                    positions=positions,
                )
```
**EN:** This test method exercises deepseek v2 rope and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 deepseek v2 rope 场景，并验证观测到的行为是否符合预期契约。

### Lines 149-157: Run test: deepseek v2 rope (part 2)
```python
                # fused rope kernel
                q_pe_clone, k_pe_clone = torch.ops.sgl_kernel.rotary_embedding_cpu(
                    positions,
                    q_pe_clone,
                    k_pe_clone,
                    rope.head_size,
                    cos_sin_cache,
                    False,
                )
```
**EN:** This test method exercises deepseek v2 rope and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 deepseek v2 rope 场景，并验证观测到的行为是否符合预期契约。

### Lines 159-162: Run test: deepseek v2 rope (part 3)
```python
                atol = rtol = precision[q_pe.dtype]
                torch.testing.assert_close(q_pe, q_pe_clone, atol=atol, rtol=rtol)
                torch.testing.assert_close(k_pe, k_pe_clone, atol=atol, rtol=rtol)
                torch.testing.assert_close(k_pe, k_pe_clone)
```
**EN:** This test method exercises deepseek v2 rope and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 deepseek v2 rope 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 164-222: Run test: origin rope (part 1)
```python
    def test_origin_rope(self):
        def single_test(
            head_size: int,
            rotary_dim: int,
            max_position_embeddings: int,
            base: int,
            dims: int,
            is_neox_style: bool,
            dtype: torch.dtype,
            device: str,
            batch_size: int,
            seq_len: int,
            num_q_heads: int,
            num_kv_heads: int,
        ):
            set_global_server_args_for_scheduler(ServerArgs(model_path="dummy"))
            torch.manual_seed(100)
            rope_ref = RotaryEmbedding(
                head_size,
                rotary_dim,
                max_position_embeddings,
                base,
                is_neox_style,
                dtype,
            ).to(device)
            pos_ids = torch.arange(seq_len, device=device).repeat(batch_size)
            query = torch.randn(
                batch_size * seq_len,
                num_q_heads * head_size,
                dtype=dtype,
                device=device,
            )
            key = torch.randn(
                batch_size * seq_len,
                num_kv_heads * head_size,
                dtype=dtype,
                device=device,
            )
            if dims == 4:
                query = query.view(batch_size, seq_len, num_q_heads, head_size)
                key = key.view(batch_size, seq_len, num_kv_heads, head_size)
            query_ref, key_ref = query.clone(), key.clone()
            query_cpu, key_cpu = query.clone(), key.clone()

            query_ref_out, key_ref_out = rope_ref.forward_native(
                pos_ids, query_ref, key_ref
            )
            query_cpu_out, key_cpu_out = torch.ops.sgl_kernel.rotary_embedding_cpu(
                pos_ids,
                query_cpu,
                key_cpu,
                rope_ref.head_size,
                rope_ref.cos_sin_cache.to(query.dtype),
                rope_ref.is_neox_style,
            )
            torch.testing.assert_close(
                query_ref_out, query_cpu_out, atol=1e-2, rtol=1e-2
            )
            torch.testing.assert_close(key_ref_out, key_cpu_out, atol=1e-2, rtol=1e-2)
```
**EN:** This test method exercises origin rope and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 origin rope 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 224-231: Run test: origin rope (part 2)
```python
        test_config = [
            (64, 64, 32, 8000, True, torch.bfloat16, "cpu", 32, 32, 1, 1),
            (256, 128, 4096, 10000, True, torch.bfloat16, "cpu", 2, 512, 32, 8),
            (512, 128, 311, 10000, True, torch.bfloat16, "cpu", 3, 39, 4, 2),
            (128, 128, 2048, 10000, False, torch.bfloat16, "cpu", 2, 512, 32, 8),
            (128, 128, 2048, 10000, False, torch.bfloat16, "cpu", 2, 512, 16, 4),
            (512, 128, 311, 10000, False, torch.bfloat16, "cpu", 3, 39, 4, 2),
        ]
```
**EN:** This test method exercises origin rope and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 origin rope 场景，并验证观测到的行为是否符合预期契约。

### Lines 233-260: Run test: origin rope (part 3)
```python
        for (
            head_size,
            rotary_dim,
            max_position_embeddings,
            base,
            is_neox_style,
            dtype,
            device,
            batch_size,
            seq_len,
            num_q_heads,
            num_kv_heads,
        ) in test_config:
            for dim in [2, 4]:
                single_test(
                    head_size,
                    rotary_dim,
                    max_position_embeddings,
                    base,
                    dim,
                    is_neox_style,
                    dtype,
                    device,
                    batch_size,
                    seq_len,
                    num_q_heads,
                    num_kv_heads,
                )
```
**EN:** This test method exercises origin rope and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 origin rope 场景，并验证观测到的行为是否符合预期契约。

### Lines 262-281: Run test: apply rotary pos emb
```python
    def test_apply_rotary_pos_emb(self):
        num_tokens = 1024
        num_heads = 8
        head_size = 72
        qkv = torch.randn(num_tokens, num_heads * head_size * 3).to(torch.bfloat16)
        query, key, _ = qkv.split(
            [num_heads * head_size, num_heads * head_size, num_heads * head_size],
            dim=-1,
        )
        query = query.view(num_tokens, num_heads, head_size)
        key = key.view(num_tokens, num_heads, head_size)
        for sincos_dtype in [torch.float32, torch.bfloat16]:
            cos = torch.rand(num_tokens, head_size).to(sincos_dtype)
            sin = torch.rand(num_tokens, head_size).to(sincos_dtype)
            q_out_ref, k_out_ref = apply_rotary_pos_emb_native(query, key, cos, sin)
            q_out_sgl, k_out_sgl = torch.ops.sgl_kernel.apply_rotary_pos_emb_cpu(
                query, key, cos, sin
            )
            torch.testing.assert_close(q_out_ref, q_out_sgl, atol=1e-2, rtol=1e-2)
            torch.testing.assert_close(k_out_ref, k_out_sgl, atol=1e-2, rtol=1e-2)
```
**EN:** This test method exercises apply rotary pos emb and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 apply rotary pos emb 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 284-285: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.layers.rotary_embedding`, `sglang.srt.layers.rotary_embedding.rope_variant`, `sglang.srt.server_args`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `torch`, `unittest`, `utils`
- Notable symbols / 关键符号: `CustomTestCase`, `unittest.main`
