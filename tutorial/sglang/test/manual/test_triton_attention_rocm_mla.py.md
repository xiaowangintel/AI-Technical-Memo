# test_triton_attention_rocm_mla.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/test_triton_attention_rocm_mla.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `triton attention rocm mla` scenario in `test/manual`. It uses SGLang's shared test infrastructure to configure models or services and verify expected performance behavior. / 该手动测试模块覆盖 `test/manual` 中的 `triton attention rocm mla` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的性能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Imports and shared helpers / 导入与共享辅助项
```python
import random
import unittest

import torch

from sglang.srt.layers.attention.triton_ops.decode_attention import (
    decode_attention_fwd_grouped,
)
from sglang.srt.layers.attention.triton_ops.rocm_mla_decode_rope import (
    decode_attention_fwd_grouped_rope,
)
from sglang.srt.layers.rotary_embedding import DeepseekScalingRotaryEmbedding
from sglang.test.test_utils import CustomTestCase
```
**EN:** This range imports `random`, `unittest`, `torch` and `sglang.srt.layers.attention.triton_ops.decode_attention`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。

### Lines 16-17: Class definition for TestTritonAttentionMLA / 类定义
```python
class TestTritonAttentionMLA(CustomTestCase):
```
**EN:** This range declares `TestTritonAttentionMLA`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 18-25: Helper routines around _set_all_seeds / 辅助例程
```python
    def _set_all_seeds(self, seed):
        """Set all random seeds for reproducibility."""
        random.seed(seed)
        torch.manual_seed(seed)
        torch.cuda.manual_seed(seed)
        torch.cuda.manual_seed_all(seed)
        torch.backends.cudnn.deterministic = True
        torch.backends.cudnn.benchmark = False
```
**EN:** This range implements helper routine(s) `_set_all_seeds` so setup, transformation, or validation logic can be reused cleanly. The code also ties this block to evaluation or benchmark execution. Representative call sites include `seed`, `manual_seed` and `manual_seed_all`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 这段代码还把当前逻辑与评测或基准执行联系起来。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 27-29: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUp(self):
        # Set seeds before each test method
        self._set_all_seeds(42)
```
**EN:** This range implements lifecycle helper(s) `setUp` to prepare or clean up shared resources across the test flow. Representative call sites include `_set_all_seeds`.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 31-38: Helper routines around preprocess_kv_cache / 辅助例程
```python
    def preprocess_kv_cache(self, kv_cache, kv_lora_rank):
        latent_cache = kv_cache
        v_input = latent_cache[..., :kv_lora_rank]
        v_input = v_input.contiguous().unsqueeze(1)
        k_input = latent_cache.unsqueeze(1)
        k_input[..., :kv_lora_rank] = v_input

        return k_input, v_input
```
**EN:** This range implements helper routine(s) `preprocess_kv_cache` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `contiguous` and `unsqueeze`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 40-40: Helper routines around input_helper / 辅助例程
```python
    def input_helper(
```
**EN:** This range implements helper routine(s) `input_helper` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 41-64: Scenario logic / 场景逻辑
```python
        self,
        B,
        H,
        S,
        kv_lora_rank,
        rotary_dim,
        qk_rope_head_dim,
        num_kv_splits,
        dtype,
        device,
        rope_base=10,
        rope_max_seq_len=16384,
        rope_scaling=1.0,
        is_neox_style=False,
    ):
        q = torch.randn(
            B, H, kv_lora_rank + qk_rope_head_dim, device=device, dtype=dtype
        )
        kv_cache = torch.randn(
            B * S, kv_lora_rank + qk_rope_head_dim, dtype=dtype, device=device
        )
        kv_indptr = torch.arange(B + 1, device=device) * S
        kv_indices = torch.arange(B * S, device=device)
        attn_logits = torch.empty(
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `randn`, `arange` and `empty`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 65-65: Scenario logic / 场景逻辑
```python
            B, H, num_kv_splits, kv_lora_rank + 1, dtype=dtype, device=device
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 66-79: Scenario logic / 场景逻辑
```python
        )
        rotary_emb = DeepseekScalingRotaryEmbedding(
            qk_rope_head_dim,
            rotary_dim,
            rope_max_seq_len,
            rope_base,
            is_neox_style,
            rope_scaling,
            q.dtype,
            device="cpu",
        ).cuda()
        positions = torch.tensor([S], device=device).unsqueeze(0).repeat(B, 1)

        return kv_indptr, kv_indices, q, kv_cache, attn_logits, rotary_emb, positions
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `DeepseekScalingRotaryEmbedding`, `cuda`, `tensor` and `unsqueeze`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 81-90: Helper routines around ref_compute_full_fwd / 辅助例程
```python
    def ref_compute_full_fwd(
        self,
        q,
        k_input,
        v_input,
        kv_lora_rank,
        kv_indptr,
        kv_indices,
        num_kv_splits,
        sm_scale,
```
**EN:** This range implements helper routine(s) `ref_compute_full_fwd` so setup, transformation, or validation logic can be reused cleanly.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。

### Lines 91-105: Scenario logic / 场景逻辑
```python
        logit_cap,
        rotary_emb,
        positions,
        use_rope,
        device="cuda",
    ):

        B, H = q.shape[0], q.shape[1]
        S = kv_indptr[1].item()
        qk_rope_head_dim = k_input.shape[-1] - kv_lora_rank

        q_input = torch.empty(B, H, kv_lora_rank + qk_rope_head_dim, dtype=q.dtype).to(
            device
        )
        q_nope_out, q_pe = q.split([kv_lora_rank, qk_rope_head_dim], dim=-1)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `item`, `empty`, `to` and `split`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 106-115: Scenario logic / 场景逻辑
```python
        k_pe_t = k_input.view(B, 1, S, -1)[:, :, -1:, kv_lora_rank:]

        if use_rope:
            q_pe, k_pe_t = rotary_emb(positions, q_pe.unsqueeze(2), k_pe_t)
            q_pe = q_pe.squeeze()

        k_input.view(B, 1, S, -1)[:, :, -1:, kv_lora_rank:] = k_pe_t

        q_input[..., :kv_lora_rank] = q_nope_out
        q_input[..., kv_lora_rank:] = q_pe
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `view`, `rotary_emb`, `unsqueeze` and `squeeze`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 116-130: Scenario logic / 场景逻辑
```python

        B, H = q_input.shape[0], q_input.shape[1]
        kv_lora_rank = v_input.shape[-1]
        device = q_input.device

        attn_logits = torch.empty(
            B, H, num_kv_splits, kv_lora_rank + 1, dtype=q_input.dtype, device=device
        )
        o = torch.empty(B, H, kv_lora_rank, dtype=q_input.dtype, device=device)

        decode_attention_fwd_grouped(
            q_input,
            k_input,
            v_input,
            o,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `empty` and `decode_attention_fwd_grouped`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 131-139: Scenario logic / 场景逻辑
```python
            kv_indptr,
            kv_indices,
            attn_logits,
            num_kv_splits,
            sm_scale,
            logit_cap,
        )

        return attn_logits, o, k_pe_t.squeeze()
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `squeeze`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 141-165: Helper routines around _test_rocm_fused_mla_kernel / 辅助例程
```python
    def _test_rocm_fused_mla_kernel(
        self,
        B,
        H,
        S,
        kv_lora_rank,
        qk_rope_head_dim,
        rotary_dim,
        dtype,
        use_rope,
        is_neox_style,
        num_kv_splits=2,
        sm_scale=1.0,
        logit_cap=0.0,
        device="cuda",
    ):
        kv_indptr, kv_indices, q, kv_cache, attn_logits, rotary_emb, positions = (
            self.input_helper(
                B,
                H,
                S,
                kv_lora_rank,
                rotary_dim,
                qk_rope_head_dim,
                num_kv_splits,
```
**EN:** This range implements helper routine(s) `_test_rocm_fused_mla_kernel` so setup, transformation, or validation logic can be reused cleanly. Representative call sites include `input_helper`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 166-190: Scenario logic / 场景逻辑
```python
                dtype,
                device=device,
                is_neox_style=is_neox_style,
            )
        )

        k_input, v_input = self.preprocess_kv_cache(kv_cache, kv_lora_rank)
        k_pe_tokens = torch.empty(
            B, qk_rope_head_dim, dtype=kv_cache.dtype, device=device
        )
        tri_o = torch.empty(B, H, kv_lora_rank, dtype=kv_cache.dtype, device=device)

        decode_attention_fwd_grouped_rope(
            q,
            k_input,
            v_input,
            tri_o,
            kv_indptr,
            kv_indices,
            k_pe_tokens if use_rope else None,
            kv_lora_rank,
            rotary_dim if use_rope else None,
            rotary_emb.cos_sin_cache if use_rope else None,
            positions if use_rope else None,
            attn_logits,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `preprocess_kv_cache`, `empty` and `decode_attention_fwd_grouped_rope`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 191-215: Scenario logic / 场景逻辑
```python
            num_kv_splits,
            sm_scale,
            logit_cap,
            use_rope,
            is_neox_style,
        )

        tri_logits = attn_logits

        # reference
        ref_logits, ref_o, ref_k_pe_tokens = self.ref_compute_full_fwd(
            q,
            k_input,
            v_input,
            kv_lora_rank,
            kv_indptr,
            kv_indices,
            num_kv_splits,
            sm_scale,
            logit_cap,
            rotary_emb,
            positions,
            use_rope,
            device="cuda",
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `ref_compute_full_fwd`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 216-222: Assertions and result checks / 断言与结果检查
```python

        if use_rope:
            torch.testing.assert_close(
                ref_k_pe_tokens, k_pe_tokens.squeeze(), atol=1e-2, rtol=1e-2
            )
        torch.testing.assert_close(ref_logits, tri_logits, atol=1e-2, rtol=1e-2)
        torch.testing.assert_close(ref_o, tri_o, atol=1e-2, rtol=1e-2)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assert_close` and `squeeze`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 224-240: Test routines around test_grouped_rocm_fused_mla / 测试例程
```python
    def test_grouped_rocm_fused_mla(self):
        configs = [
            (1, 128, 2048, 512, 64, 64),
            (1, 128, 2048, 512, 128, 64),
            (1, 128, 2048, 512, 127, 64),
            (1, 128, 2050, 512, 127, 64),
            (1, 128, 2050, 512, 128, 64),
            (8, 128, 2048, 512, 64, 64),
            (8, 128, 2048, 512, 128, 64),
            (8, 128, 2048, 512, 127, 64),
            (8, 128, 2050, 512, 127, 64),
            (8, 128, 2050, 512, 128, 64),
        ]
        dtypes = [torch.bfloat16, torch.float32]
        use_rope_list = [True, False]
        is_neox_style_list = [True, False]
```
**EN:** This range defines concrete test routine(s) `test_grouped_rocm_fused_mla`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 241-248: Scenario logic / 场景逻辑
```python
        for B, H, S, kv_lora_rank, qk_rope_head_dim, rotary_dim in configs:
            for dtype in dtypes:
                for use_rope in use_rope_list:
                    for is_neox_style in is_neox_style_list:
                        self._test_rocm_fused_mla_kernel(
                            B,
                            H,
                            S,
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `_test_rocm_fused_mla_kernel`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 249-255: Scenario logic / 场景逻辑
```python
                            kv_lora_rank,
                            qk_rope_head_dim,
                            rotary_dim,
                            dtype,
                            use_rope,
                            is_neox_style,
                        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 256-259: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Performance benchmarking / 性能基准测试
- Multi-GPU orchestration / 多 GPU 编排
- Token-level inspection / Token 级分析
- LoRA adaptation / LoRA 适配
- Disaggregated serving / 解耦式服务

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `random`, `unittest`
- **Third-party / 第三方库**: `torch`
- **Project Modules / 项目模块**: `sglang.srt.layers.attention.triton_ops.decode_attention`, `sglang.srt.layers.attention.triton_ops.rocm_mla_decode_rope`, `sglang.srt.layers.rotary_embedding`, `sglang.test.test_utils`
