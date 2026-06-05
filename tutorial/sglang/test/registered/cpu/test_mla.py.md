# test_mla.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/cpu/test_mla.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on cpu mla in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 cpu mla 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Import dependencies
```python
import unittest

import torch
from torch.nn.functional import scaled_dot_product_attention
from utils import precision

from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 10-10: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-b-test-cpu")
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 12-12: Implement expr logic
```python
torch.manual_seed(1234)
```
**EN:** This block contains supporting statements that connect the surrounding setup and assertions.
**CN:** 该代码块包含连接周边初始化与断言的辅助语句。

### Lines 15-15: Define class TestMLA
```python
class TestMLA(CustomTestCase):
```
**EN:** This declaration introduces the `TestMLA` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestMLA` 测试类，并说明它通过继承承担的职责。

### Lines 16-68: Define helper: run sdpa forward decode
```python
    def _run_sdpa_forward_decode(
        self,
        query: torch.Tensor,
        output: torch.Tensor,
        k_cache: torch.Tensor,
        v_cache: torch.Tensor,
        key: torch.Tensor,
        loc: torch.Tensor,
        req_to_token: torch.Tensor,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        scaling=None,
        enable_gqa=False,
        causal=False,
    ):
        # set kv cache
        k_cache[loc] = key

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
**EN:** This helper function encapsulates reusable logic inside `TestMLA` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestMLA` 内部调用，从而让场景结构更清晰。

### Lines 70-121: Define helper: test grouped decode attention once (part 1)
```python
    def _test_grouped_decode_attention_once(self, B, H_Q, H_KV, D, D_V, seq_len):
        dtype = torch.bfloat16

        total_tokens = B * seq_len
        sm_scale = 1.0 / (D**0.5)
        logit_cap = 0.0
        num_kv_splits = 8
        enable_gqa = H_Q != H_KV

        # q represents the new token being generated, one per batch
        q = torch.randn(B, H_Q, D, dtype=dtype)

        # k_buffer and v_buffer represent all previous tokens
        k_buffer = torch.randn(total_tokens, H_KV, D, dtype=dtype)
        v_buffer = k_buffer.narrow(2, 0, D_V)

        key = torch.randn(B, H_KV, D, dtype=dtype)
        value = key.narrow(2, 0, D_V)
        # make sure no duplicates in loc
        loc = torch.randperm(total_tokens)[:B].to(torch.int64)

        k_buffer2 = k_buffer.clone()
        v_buffer2 = k_buffer2.narrow(2, 0, D_V)

        # o will have the same shape as q
        o = torch.zeros(B, H_Q, D_V, dtype=dtype)
        o_grouped = torch.zeros(B, H_Q, D_V, dtype=dtype)

        req_to_token = torch.arange(total_tokens).reshape(B, seq_len).to(torch.int32)
        b_req_idx = torch.arange(B).to(torch.int64)
        b_seq_len = torch.full((B,), seq_len).to(torch.int64)

        attn_logits = torch.empty(
            (B, H_Q, num_kv_splits, D_V + 1),
            dtype=torch.float32,
        )

        torch.ops.sgl_kernel.decode_attention_cpu(
            q,
            k_buffer2,
            v_buffer2,
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
```
**EN:** This helper function encapsulates reusable logic inside `TestMLA` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestMLA` 内部调用，从而让场景结构更清晰。

### Lines 123-135: Define helper: test grouped decode attention once (part 2)
```python
        self._run_sdpa_forward_decode(
            q,
            o_grouped,
            k_buffer,
            v_buffer,
            key,
            loc,
            req_to_token,
            b_req_idx,
            b_seq_len,
            scaling=sm_scale,
            enable_gqa=enable_gqa,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestMLA` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestMLA` 内部调用，从而让场景结构更清晰。

### Lines 137-144: Define helper: test grouped decode attention once (part 3)
```python
        cos_sim = torch.nn.functional.cosine_similarity(
            o.flatten(), o_grouped.flatten(), dim=0
        )
        atol = rtol = precision[q.dtype]
        self.assertGreater(cos_sim.item(), 0.99)
        torch.testing.assert_close(o, o_grouped, atol=atol, rtol=rtol)
        torch.testing.assert_close(k_buffer, k_buffer2, atol=atol, rtol=rtol)
        torch.testing.assert_close(v_buffer, v_buffer2, atol=atol, rtol=rtol)
```
**EN:** This helper function encapsulates reusable logic inside `TestMLA` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestMLA` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 146-154: Run test: grouped decode attention
```python
    def test_grouped_decode_attention(self):
        configs = [
            (1, 22, 1, 576, 512, 8 * 111),
            (4, 22, 1, 576, 512, 8 * 128),
            (40, 22, 1, 576, 512, 8 * 133),
        ]

        for B, H_Q, H_KV, D, D_V, seqlen in configs:
            self._test_grouped_decode_attention_once(B, H_Q, H_KV, D, D_V, seqlen)
```
**EN:** This test method exercises grouped decode attention and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 grouped decode attention 场景，并验证观测到的行为是否符合预期契约。

### Lines 157-158: Expose unittest entrypoint
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
- External and stdlib modules / 外部与标准库模块: `torch`, `torch.nn.functional`, `unittest`, `utils`
- Notable symbols / 关键符号: `CustomTestCase`, `unittest.main`
