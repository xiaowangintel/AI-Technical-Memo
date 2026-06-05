# test_rope.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/srt/cpu/test_rope.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates rope behavior in SGLang's srt / cpu area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 SRT / CPU 领域中与 rope 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: module imports and dependencies / 模块导入与依赖
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
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `torch`, `utils`, `sglang.srt.layers.rotary_embedding`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `torch`, `utils`, `sglang.srt.layers.rotary_embedding`。

### Lines 17-17: supporting statements / 辅助语句
```python
torch.manual_seed(1234)
```
**EN:** This block performs supporting work through calls such as `manual_seed`, preparing state for nearby definitions.
**CN:** 该代码块通过 `manual_seed` 等调用完成辅助性工作，为相邻定义准备状态。

### Lines 20-20: class TestROPE declaration / 类 TestROPE 声明
```python
class TestROPE(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 21-91: test case mrope / 测试用例 mrope
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
**EN:** This test exercises `test_mrope` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mrope`。

### Lines 93-159: test case deepseek v2 rope / 测试用例 deepseek v2 rope
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

                # fused rope kernel
                q_pe_clone, k_pe_clone = torch.ops.sgl_kernel.rotary_embedding_cpu(
                    positions,
                    q_pe_clone,
                    k_pe_clone,
                    rope.head_size,
                    cos_sin_cache,
                    False,
                )

                atol = rtol = precision[q_pe.dtype]
                torch.testing.assert_close(q_pe, q_pe_clone, atol=atol, rtol=rtol)
                torch.testing.assert_close(k_pe, k_pe_clone, atol=atol, rtol=rtol)
                torch.testing.assert_close(k_pe, k_pe_clone)
```
**EN:** This test exercises `test_deepseek_v2_rope` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_deepseek_v2_rope`。

### Lines 161-240: test case origin rope (part 1/2) / 测试用例 origin rope（第 1/2 部分）
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

        test_config = [
            (64, 64, 32, 8000, True, torch.bfloat16, "cpu", 32, 32, 1, 1),
            (256, 128, 4096, 10000, True, torch.bfloat16, "cpu", 2, 512, 32, 8),
            (512, 128, 311, 10000, True, torch.bfloat16, "cpu", 3, 39, 4, 2),
            (128, 128, 2048, 10000, False, torch.bfloat16, "cpu", 2, 512, 32, 8),
            (128, 128, 2048, 10000, False, torch.bfloat16, "cpu", 2, 512, 16, 4),
            (512, 128, 311, 10000, False, torch.bfloat16, "cpu", 3, 39, 4, 2),
        ]

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
```
**EN:** This test exercises `test_origin_rope` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_origin_rope`。 这一段对应同一逻辑块的第 1 部分。

### Lines 241-257: test case origin rope (part 2/2) / 测试用例 origin rope（第 2/2 部分）
```python
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
**EN:** This test exercises `test_origin_rope` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_origin_rope`。 这一段对应同一逻辑块的第 2 部分。

### Lines 259-278: test case apply rotary pos emb / 测试用例 apply rotary pos emb
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
**EN:** This test exercises `test_apply_rotary_pos_emb` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_apply_rotary_pos_emb`。

### Lines 281-282: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestROPE`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestROPE.test_mrope`: This test exercises `test_mrope` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mrope`。
- `TestROPE.test_deepseek_v2_rope`: This test exercises `test_deepseek_v2_rope` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_deepseek_v2_rope`。
- `TestROPE.test_origin_rope`: This test exercises `test_origin_rope` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_origin_rope`。
- `TestROPE.test_apply_rotary_pos_emb`: This test exercises `test_apply_rotary_pos_emb` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_apply_rotary_pos_emb`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `torch`, `utils`
- **Internal modules / 内部模块**: `sglang.srt.layers.rotary_embedding`, `sglang.srt.layers.rotary_embedding.rope_variant`, `sglang.srt.server_args`, `sglang.test.test_utils`

- **Total lines / 总行数**: 282
