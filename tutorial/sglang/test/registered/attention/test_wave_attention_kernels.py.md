# test_wave_attention_kernels.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/attention/test_wave_attention_kernels.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on wave attention kernels in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 wave attention kernels 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-25: Import dependencies
```python
import random
import unittest

import torch

from sglang.srt.layers.attention.triton_ops.decode_attention import (
    decode_attention_fwd_grouped as triton_decode_attention_fwd_grouped,
)
from sglang.srt.layers.attention.triton_ops.extend_attention import (
    extend_attention_fwd,
    redundant_attention,
)
from sglang.srt.layers.attention.triton_ops.prefill_attention import (
    context_attention_fwd,
)
from sglang.srt.layers.attention.wave_ops.decode_attention import (
    decode_attention_intermediate_arrays_shapes,
    decode_attention_wave,
)
from sglang.srt.layers.attention.wave_ops.extend_attention import extend_attention_wave
from sglang.srt.layers.attention.wave_ops.prefill_attention import (
    prefill_attention_wave,
)
from sglang.srt.utils import get_device
from sglang.test.ci.ci_register import register_amd_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for AMD CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 28-28: Register CI metadata
```python
register_amd_ci(est_time=60, suite="stage-a-test-1-gpu-small-amd")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 31-31: Define class TestWaveAttention
```python
class TestWaveAttention(unittest.TestCase):
```
**EN:** This declaration introduces the `TestWaveAttention` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestWaveAttention` 测试类，并说明它通过继承承担的职责。

### Lines 33-40: Define helper: set all seeds
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
**EN:** This helper function encapsulates reusable logic inside `TestWaveAttention` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestWaveAttention` 内部调用，从而让场景结构更清晰。

### Lines 42-44: Prepare per-test state
```python
    def setUp(self):
        # Set seeds before each test method
        self._set_all_seeds(42)
```
**EN:** This setup method initializes the state required before each individual test executes.
**CN:** 该初始化方法会在每个独立测试执行前准备所需状态。

### Lines 46-83: Define helper: test extend attention once (part 1)
```python
    def _test_extend_attention_once(self, B, N_CTX, H_Q, H_KV, D):
        dtype = torch.float16
        extend_seq_len = 1024

        b_seq_len_prefix = torch.full(
            (B,), N_CTX // B, dtype=torch.int32, device=get_device()
        )
        b_seq_len_extend = torch.full(
            (B,), extend_seq_len, dtype=torch.int32, device=get_device()
        )
        b_seq_len = b_seq_len_prefix + b_seq_len_extend
        max_len_in_batch = torch.max(b_seq_len, 0)[0].item()

        b_req_idx = torch.arange(B, dtype=torch.int32, device=get_device())
        b_start_loc = torch.zeros((B,), dtype=torch.int32, device=get_device())
        b_start_loc[1:] = torch.cumsum(b_seq_len[:-1], 0)
        b_start_loc_extend = torch.zeros((B,), dtype=torch.int32, device=get_device())
        b_start_loc_extend[1:] = torch.cumsum(b_seq_len_extend[:-1], 0)

        kv_indptr = torch.zeros((B + 1,), dtype=torch.int32, device=get_device())
        kv_indptr[1 : B + 1] = torch.cumsum(b_seq_len_prefix[:B], dim=0)
        kv_indices = torch.zeros(
            (b_seq_len_prefix.sum().item(),), dtype=torch.int32, device=get_device()
        )

        for i in range(B):
            kv_indices[kv_indptr[i] : kv_indptr[i + 1]] = torch.arange(
                b_start_loc[i], b_start_loc[i] + b_seq_len_prefix[i]
            )

        total_token_num = torch.sum(b_seq_len).item()
        extend_token_num = torch.sum(b_seq_len_extend).item()
        k_buffer = torch.empty(
            (total_token_num, H_KV, D), dtype=dtype, device=get_device()
        ).normal_(mean=0.1, std=0.2)
        v_buffer = torch.empty(
            (total_token_num, H_KV, D), dtype=dtype, device=get_device()
        ).normal_(mean=0.1, std=0.2)
```
**EN:** This helper function encapsulates reusable logic inside `TestWaveAttention` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestWaveAttention` 内部调用，从而让场景结构更清晰。

### Lines 85-139: Define helper: test extend attention once (part 2)
```python
        k_extend = torch.empty(
            (extend_token_num, H_KV, D), dtype=dtype, device=get_device()
        )
        v_extend = torch.empty(
            (extend_token_num, H_KV, D), dtype=dtype, device=get_device()
        )
        q_extend = torch.empty(
            (extend_token_num, H_Q, D), dtype=dtype, device=get_device()
        )
        for i in range(B):
            extend_start_in_buffer = b_start_loc[i] + b_seq_len_prefix[i]
            extend_end_in_buffer = b_start_loc[i] + b_seq_len[i]
            extend_start = b_start_loc_extend[i]
            extend_end = b_start_loc_extend[i] + b_seq_len_extend[i]
            k_extend[extend_start:extend_end] = k_buffer[
                extend_start_in_buffer:extend_end_in_buffer
            ]
            v_extend[extend_start:extend_end] = v_buffer[
                extend_start_in_buffer:extend_end_in_buffer
            ]
            q_extend[extend_start:extend_end] = torch.empty(
                (b_seq_len_extend[i], H_Q, D), dtype=dtype, device=get_device()
            ).normal_(mean=0.1, std=0.2)

        o_extend = torch.empty(
            (extend_token_num, H_Q, D), dtype=dtype, device=get_device()
        )
        o_extend_mask = torch.empty(
            (extend_token_num, H_Q, D), dtype=dtype, device=get_device()
        )
        o_redundant = torch.empty(
            (extend_token_num, H_Q, D), dtype=dtype, device=get_device()
        )

        b_seq_len_extend = b_seq_len - b_seq_len_prefix
        max_len_extend = torch.max(b_seq_len_extend, 0)[0].item()
        qo_indptr = torch.zeros((B + 1,), dtype=torch.int32, device=get_device())
        qo_indptr[1 : B + 1] = torch.cumsum(b_seq_len_extend[:B], dim=0)

        custom_mask = None
        mask_indptr = None

        redundant_attention(
            q_extend,
            o_redundant,
            k_buffer,
            v_buffer,
            b_req_idx,
            b_start_loc,
            b_seq_len,
            b_seq_len_prefix,
            max_len_in_batch,
        )

        is_causal = True
```
**EN:** This helper function encapsulates reusable logic inside `TestWaveAttention` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestWaveAttention` 内部调用，从而让场景结构更清晰。

### Lines 141-179: Define helper: test extend attention once (part 3)
```python
        o_extend = torch.empty(
            (extend_token_num, H_Q, D), dtype=dtype, device=get_device()
        )
        extend_attention_fwd(
            q_extend,
            k_extend,
            v_extend,
            o_extend,
            k_buffer,
            v_buffer,
            qo_indptr,
            kv_indptr,
            kv_indices,
            custom_mask,
            is_causal,
            mask_indptr,
            max_len_extend,
            1.0,
            1.0,
        )

        o_wave = torch.empty(
            (extend_token_num, H_Q, D), dtype=dtype, device=get_device()
        )
        extend_attention_wave(
            q_extend,
            k_extend,
            v_extend,
            k_buffer,
            v_buffer,
            qo_indptr,
            kv_indptr,
            kv_indices,
            custom_mask,
            mask_indptr,
            max_len_extend,
            o_wave,
            is_causal=is_causal,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestWaveAttention` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestWaveAttention` 内部调用，从而让场景结构更清晰。

### Lines 181-182: Define helper: test extend attention once (part 4)
```python
        self.assertTrue(torch.allclose(o_extend, o_redundant, rtol=1e-2))
        self.assertTrue(torch.allclose(o_wave, o_redundant, rtol=1e-2))
```
**EN:** This helper function encapsulates reusable logic inside `TestWaveAttention` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestWaveAttention` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 184-191: Run test: extend attention
```python
    def test_extend_attention(self):

        # Define the varying parameter values
        attention_values = [128]

        # Loop through the values and call the method
        for value in attention_values:
            self._test_extend_attention_once(32, 16384, 6, 1, value)
```
**EN:** This test method exercises extend attention and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 extend attention 场景，并验证观测到的行为是否符合预期契约。

### Lines 193-251: Define helper: test grouped decode attention once (part 1)
```python
    def _test_grouped_decode_attention_once(self, B, S, H_Q, H_KV, D, D_V):
        dtype = torch.float16
        seq_len = S  # This represents the number of tokens already in the sequence
        total_tokens = B * seq_len
        sm_scale = 1.0 / (D**0.5)
        max_kv_splits = 8
        num_kv_splits = torch.full((B,), 4, dtype=torch.int32, device=get_device())

        # q represents the new token being generated, one per batch
        q = torch.randn(B, H_Q, D, dtype=dtype, device=get_device())

        # k_buffer and v_buffer represent all previous tokens
        k_buffer = torch.randn(total_tokens, H_KV, D, dtype=dtype, device=get_device())
        v_buffer = torch.randn(
            total_tokens, H_KV, D_V, dtype=dtype, device=get_device()
        )

        # o will have the same shape as q
        o_triton = torch.zeros(B, H_Q, D_V, dtype=dtype, device=get_device())
        o = torch.zeros(B, H_Q, D_V, dtype=dtype, device=get_device())

        req_to_token = torch.arange(
            total_tokens, device=get_device(), dtype=torch.int32
        )
        b_req_idx = torch.zeros(B + 1, device=get_device(), dtype=torch.int32)
        b_seq_len = torch.full((B,), seq_len, device=get_device(), dtype=torch.int32)
        b_req_idx[1 : B + 1] = torch.cumsum(b_seq_len, dim=0)

        attn_logits = torch.empty(
            (B, H_Q, max_kv_splits, D_V + 1),
            dtype=torch.float32,
            device=get_device(),
        )
        attn_lse = torch.empty(
            (B, H_Q, max_kv_splits),
            dtype=torch.float32,
            device=get_device(),
        )

        logit_cap = 0.0
        triton_decode_attention_fwd_grouped(
            q,
            k_buffer,
            v_buffer,
            o_triton,
            b_req_idx,
            req_to_token,
            attn_logits,
            attn_lse,
            num_kv_splits,
            max_kv_splits,
            sm_scale,
            1.0,
            logit_cap,
        )

        attn_logits_shape, attn_logits_max_shape = (
            decode_attention_intermediate_arrays_shapes(B, D_V, H_Q, max_kv_splits)
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestWaveAttention` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestWaveAttention` 内部调用，从而让场景结构更清晰。

### Lines 253-278: Define helper: test grouped decode attention once (part 2)
```python
        attn_logits = torch.empty(
            attn_logits_shape,
            dtype=torch.float32,
            device=get_device(),
        )

        attn_logits_max = torch.empty(
            attn_logits_max_shape,
            dtype=torch.float32,
            device=get_device(),
        )

        decode_attention_wave(
            q,
            k_buffer,
            v_buffer,
            o,
            b_req_idx,
            req_to_token,
            attn_logits,
            attn_logits_max,
            num_kv_splits,
            max_kv_splits,
            sm_scale,
            logit_cap,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestWaveAttention` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestWaveAttention` 内部调用，从而让场景结构更清晰。

### Lines 280-285: Define helper: test grouped decode attention once (part 3)
```python
        cos_sim = torch.nn.functional.cosine_similarity(
            o.flatten(), o_triton.flatten(), dim=0
        )
        print(cos_sim.item())
        self.assertTrue(cos_sim.item() > 0.99)
        self.assertTrue(torch.allclose(o, o_triton, atol=3e-2))
```
**EN:** This helper function encapsulates reusable logic inside `TestWaveAttention` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestWaveAttention` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 287-300: Run test: grouped decode attention
```python
    def test_grouped_decode_attention(self):
        seq_lens = [5, 100, 128, 500]
        configs = [
            (2, 16, 16, 64, 64),
            (2, 16, 1, 64, 64),
            (2, 128, 1, 80, 80),
            (32, 128, 2, 512, 512),
            (2, 128, 2, 512, 512),
            (2, 128, 1, 576, 512),
        ]

        for S in seq_lens:
            for B, H_Q, H_KV, D, D_V in configs:
                self._test_grouped_decode_attention_once(B, S, H_Q, H_KV, D, D_V)
```
**EN:** This test method exercises grouped decode attention and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 grouped decode attention 场景，并验证观测到的行为是否符合预期契约。

### Lines 302-343: Define helper: test context attention once
```python
    def _test_context_attention_once(self, head_dim, is_causal):
        # Set up a simple test case
        dtype = torch.float16
        num_heads = 4
        kv_heads = 1
        seq_lens = [128, 256]
        max_seq_len = max(seq_lens)

        # Create random input tensors
        q = torch.randn(
            sum(seq_lens), num_heads, head_dim, dtype=dtype, device=get_device()
        )
        k = torch.randn(
            sum(seq_lens), kv_heads, head_dim, dtype=dtype, device=get_device()
        )
        v = torch.randn(
            sum(seq_lens), kv_heads, head_dim, dtype=dtype, device=get_device()
        )
        o_triton = torch.zeros(
            sum(seq_lens), num_heads, head_dim, dtype=dtype, device=get_device()
        )
        o = torch.zeros(
            sum(seq_lens), num_heads, head_dim, dtype=dtype, device=get_device()
        )

        # Create b_start_loc and b_seq_len tensors
        b_start_loc = torch.tensor([0, seq_lens[0]], device=get_device())
        b_seq_len = torch.tensor(seq_lens, device=get_device())

        context_attention_fwd(
            q, k, v, o_triton, b_start_loc, b_seq_len, max_seq_len, is_causal=is_causal
        )
        prefill_attention_wave(
            q, k, v, o, b_start_loc, b_seq_len, max_seq_len, is_causal=is_causal
        )
        cos_sim = torch.nn.functional.cosine_similarity(
            o.flatten(), o_triton.flatten(), dim=0
        )

        print(cos_sim.item())
        self.assertTrue(torch.allclose(o, o_triton, atol=3e-2))
        self.assertTrue(cos_sim.item() > 1 - (1e-5))
```
**EN:** This helper function encapsulates reusable logic inside `TestWaveAttention` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestWaveAttention` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 345-350: Run test: context attention
```python
    def test_context_attention(self):
        head_dim = [128, 96]

        for dim in head_dim:
            for is_causal in [False]:
                self._test_context_attention_once(dim, is_causal)
```
**EN:** This test method exercises context attention and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 context attention 场景，并验证观测到的行为是否符合预期契约。

### Lines 353-354: Expose unittest entrypoint
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

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.layers.attention.triton_ops.decode_attention`, `sglang.srt.layers.attention.triton_ops.extend_attention`, `sglang.srt.layers.attention.triton_ops.prefill_attention`, `sglang.srt.layers.attention.wave_ops.decode_attention`, `sglang.srt.layers.attention.wave_ops.extend_attention`, `sglang.srt.layers.attention.wave_ops.prefill_attention`, `sglang.srt.utils`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `random`, `torch`, `unittest`
- Notable symbols / 关键符号: `register_amd_ci`, `unittest.main`
