# test_decode.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/cpu/test_decode.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on cpu decode in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 cpu decode 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Import dependencies
```python
import unittest

import torch
from torch.nn.functional import scaled_dot_product_attention

from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 9-9: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-b-test-cpu")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 11-11: Implement expr logic
```python
torch.manual_seed(1234)
```
**EN:** This block contains supporting statements that connect the surrounding setup and assertions.
**CN:** 该代码块包含连接周边初始化与断言的辅助语句。

### Lines 14-14: Define class TestDecodeAttention
```python
class TestDecodeAttention(CustomTestCase):
```
**EN:** This declaration introduces the `TestDecodeAttention` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDecodeAttention` 测试类，并说明它通过继承承担的职责。

### Lines 15-62: Define helper: run sdpa forward decode
```python
    def _run_sdpa_forward_decode(
        self,
        query: torch.Tensor,
        output: torch.Tensor,
        k_cache: torch.Tensor,
        v_cache: torch.Tensor,
        req_to_token: torch.Tensor,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        scaling=None,
        enable_gqa=False,
        causal=False,
    ):
        # [num_tokens, num_heads, head_size] -> [num_heads, num_tokens, head_size]
        query = query.movedim(0, query.dim() - 2)

        start_q, start_kv = 0, 0
        for seq_idx in range(seq_lens.shape[0]):
            seq_len_q = 1
            seq_len_kv = seq_lens[seq_idx]
            end_q = start_q + seq_len_q
            end_kv = start_kv + seq_len_kv

            per_req_query = query[:, start_q:end_q, :]

            # get key and value from cache. per_req_tokens contains the kv cache
            # index for each token in the sequence.
            req_pool_idx = req_pool_indices[seq_idx]
            per_req_tokens = req_to_token[req_pool_idx, :seq_len_kv]
            per_req_key = k_cache[per_req_tokens].movedim(0, query.dim() - 2)
            per_req_value = v_cache[per_req_tokens].movedim(0, query.dim() - 2)

            per_req_out = (
                scaled_dot_product_attention(
                    per_req_query.unsqueeze(0),
                    per_req_key.unsqueeze(0),
                    per_req_value.unsqueeze(0),
                    enable_gqa=enable_gqa,
                    scale=scaling,
                    is_causal=causal,
                )
                .squeeze(0)
                .movedim(query.dim() - 2, 0)
            )
            output[start_q:end_q, :, :] = per_req_out
            start_q, start_kv = end_q, end_kv

        return output
```
**EN:** This helper function encapsulates reusable logic inside `TestDecodeAttention` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDecodeAttention` 内部调用，从而让场景结构更清晰。

### Lines 64-104: Define helper: test grouped decode attention once (part 1)
```python
    def _test_grouped_decode_attention_once(self, B, H_Q, H_KV, D, D_V, dtype, device):
        # This represents the number of tokens already in the sequence
        seq_len = 1024
        total_tokens = B * seq_len
        sm_scale = 1.0 / (D**0.5)
        logit_cap = 0.0
        num_kv_splits = 8
        enable_gqa = H_Q != H_KV

        # q represents the new token being generated, one per batch
        q = torch.randn(B, H_Q, D, dtype=dtype, device=device)

        # k_buffer and v_buffer represent all previous tokens
        k_buffer = torch.randn(total_tokens, H_KV, D, dtype=dtype, device=device)
        v_buffer = torch.randn(total_tokens, H_KV, D_V, dtype=dtype, device=device)

        key = torch.randn(B, H_KV, D, dtype=dtype)
        value = torch.randn(B, H_KV, D_V, dtype=dtype)
        loc = torch.randint(0, 10, (B,)).to(torch.int64)

        # set kv cache
        k_buffer[loc] = key
        v_buffer[loc] = value

        # o will have the same shape as q
        o = torch.zeros(B, H_Q, D_V, dtype=dtype, device=device)
        o_grouped = torch.zeros(B, H_Q, D_V, dtype=dtype, device=device)

        req_to_token = (
            torch.arange(total_tokens, device=device)
            .reshape(B, seq_len)
            .to(torch.int32)
        )
        b_req_idx = torch.arange(B, device=device).to(torch.int64)
        b_seq_len = torch.full((B,), seq_len, device=device).to(torch.int64)

        attn_logits = torch.empty(
            (B, H_Q, num_kv_splits, D_V + 1),
            dtype=torch.float32,
            device=device,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDecodeAttention` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDecodeAttention` 内部调用，从而让场景结构更清晰。

### Lines 106-138: Define helper: test grouped decode attention once (part 2)
```python
        # k_buffer, v_buffer, query, key and value supports non-contiguous tensors
        k_buffer = k_buffer.transpose(0, 1).contiguous().transpose(0, 1)
        v_buffer = v_buffer.transpose(0, 1).contiguous().transpose(0, 1)
        q = q.transpose(0, 1).contiguous().transpose(0, 1)
        key = key.transpose(0, 1).contiguous().transpose(0, 1)
        value = value.transpose(0, 1).contiguous().transpose(0, 1)
        torch.ops.sgl_kernel.decode_attention_cpu(
            q,
            k_buffer,
            v_buffer,
            o,
            key,
            value,
            loc,
            attn_logits,
            req_to_token,
            b_req_idx,
            b_seq_len,
            sm_scale,
            logit_cap,
        )

        self._run_sdpa_forward_decode(
            q,
            o_grouped,
            k_buffer,
            v_buffer,
            req_to_token,
            b_req_idx,
            b_seq_len,
            scaling=sm_scale,
            enable_gqa=enable_gqa,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDecodeAttention` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDecodeAttention` 内部调用，从而让场景结构更清晰。

### Lines 140-144: Define helper: test grouped decode attention once (part 3)
```python
        cos_sim = torch.nn.functional.cosine_similarity(
            o.flatten(), o_grouped.flatten(), dim=0
        )
        self.assertGreater(cos_sim.item(), 0.99)
        torch.testing.assert_close(o, o_grouped, atol=3e-2, rtol=1e-6)
```
**EN:** This helper function encapsulates reusable logic inside `TestDecodeAttention` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDecodeAttention` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 146-165: Define helper: test grouped decode attention
```python
    def _test_grouped_decode_attention(self, device="cpu"):
        configs = [
            (2, 16, 16, 64, 64),
            (2, 16, 1, 16, 16),
            (2, 32, 8, 33, 55),
            (2, 16, 1, 64, 64),
            (2, 64, 1, 13, 13),
            (2, 128, 1, 80, 80),
            (2, 128, 2, 512, 512),
            (1, 16, 1, 576, 512),
            (1, 16, 16, 576, 512),
            (1, 22, 1, 576, 512),
            (1, 40, 8, 128, 128),
        ]

        for B, H_Q, H_KV, D, D_V in configs:
            for dtype in [torch.bfloat16, torch.float16]:
                self._test_grouped_decode_attention_once(
                    B, H_Q, H_KV, D, D_V, dtype=dtype, device=device
                )
```
**EN:** This helper function encapsulates reusable logic inside `TestDecodeAttention` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDecodeAttention` 内部调用，从而让场景结构更清晰。

### Lines 167-168: Run test: grouped decode attention
```python
    def test_grouped_decode_attention(self):
        self._test_grouped_decode_attention("cpu")
```
**EN:** This test method exercises grouped decode attention and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 grouped decode attention 场景，并验证观测到的行为是否符合预期契约。

### Lines 171-172: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `torch`, `torch.nn.functional`, `unittest`
- Notable symbols / 关键符号: `CustomTestCase`, `unittest.main`
