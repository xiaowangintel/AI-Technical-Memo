# test_extend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/cpu/test_extend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on cpu extend in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 cpu extend 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

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

### Lines 14-14: Define class TestExtendAttention
```python
class TestExtendAttention(CustomTestCase):
```
**EN:** This declaration introduces the `TestExtendAttention` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestExtendAttention` 测试类，并说明它通过继承承担的职责。

### Lines 16-62: Define helper: run sdpa forward extend (part 1)
```python
    def _run_sdpa_forward_extend(
        self,
        query: torch.Tensor,
        output: torch.Tensor,
        k_cache: torch.Tensor,
        v_cache: torch.Tensor,
        req_to_token: torch.Tensor,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        extend_prefix_lens: torch.Tensor,
        extend_seq_lens: torch.Tensor,
        scaling=None,
        enable_gqa=False,
        causal=False,
    ):

        assert seq_lens.shape[0] == extend_prefix_lens.shape[0]
        assert seq_lens.shape[0] == extend_seq_lens.shape[0]

        # [num_tokens, num_heads, head_size] -> [num_heads, num_tokens, head_size]
        query = query.movedim(0, query.dim() - 2)

        start_q, start_kv = 0, 0
        for seq_idx in range(seq_lens.shape[0]):

            extend_seq_len_q = extend_seq_lens[seq_idx]
            prefill_seq_len_q = extend_prefix_lens[seq_idx]

            seq_len_kv = seq_lens[seq_idx]
            end_q = start_q + extend_seq_len_q
            end_kv = start_kv + seq_len_kv

            per_req_query = query[:, start_q:end_q, :]
            per_req_query_redudant = torch.empty(
                (per_req_query.shape[0], seq_len_kv, per_req_query.shape[2]),
                dtype=per_req_query.dtype,
                device=per_req_query.device,
            )

            per_req_query_redudant[:, prefill_seq_len_q:, :] = per_req_query

            # get key and value from cache. per_req_tokens contains the kv cache
            # index for each token in the sequence.
            req_pool_idx = req_pool_indices[seq_idx]
            per_req_tokens = req_to_token[req_pool_idx, :seq_len_kv]
            per_req_key = k_cache[per_req_tokens].movedim(0, query.dim() - 2)
            per_req_value = v_cache[per_req_tokens].movedim(0, query.dim() - 2)
```
**EN:** This helper function encapsulates reusable logic inside `TestExtendAttention` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestExtendAttention` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 64-78: Define helper: run sdpa forward extend (part 2)
```python
            per_req_out_redudant = (
                scaled_dot_product_attention(
                    per_req_query_redudant.unsqueeze(0),
                    per_req_key.unsqueeze(0),
                    per_req_value.unsqueeze(0),
                    enable_gqa=enable_gqa,
                    scale=scaling,
                    is_causal=causal,
                )
                .squeeze(0)
                .movedim(query.dim() - 2, 0)
            )
            output[start_q:end_q, :, :] = per_req_out_redudant[prefill_seq_len_q:, :, :]
            start_q, start_kv = end_q, end_kv
        return output
```
**EN:** This helper function encapsulates reusable logic inside `TestExtendAttention` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestExtendAttention` 内部调用，从而让场景结构更清晰。

### Lines 80-131: Define helper: test extend attention once (part 1)
```python
    def _test_extend_attention_once(
        self,
        B,
        N_CTX,
        H_Q,
        H_KV,
        D,
        DV,
        mla=False,
        *,
        b_seq_len_prefix=None,
        b_seq_len_extend=None,
    ):
        dtype = torch.bfloat16

        if b_seq_len_prefix is None:
            b_seq_len_prefix = torch.randint(1, N_CTX // 2, (B,), dtype=torch.int32)
            if mla:
                b_seq_len_prefix.zero_()
        else:
            b_seq_len_prefix = torch.as_tensor(b_seq_len_prefix, dtype=torch.int32)

        if b_seq_len_extend is None:
            b_seq_len_extend = torch.randint(1, N_CTX // 2, (B,), dtype=torch.int32)
        else:
            b_seq_len_extend = torch.as_tensor(b_seq_len_extend, dtype=torch.int32)

        b_seq_len = b_seq_len_prefix + b_seq_len_extend
        max_len_in_batch = torch.max(b_seq_len, 0)[0].item()

        b_req_idx = torch.arange(B, dtype=torch.int32)
        req_to_tokens = torch.empty((B, max_len_in_batch), dtype=torch.int32)
        b_start_loc = torch.zeros((B,), dtype=torch.int32)
        b_start_loc[1:] = torch.cumsum(b_seq_len[:-1], 0)
        b_start_loc_extend = torch.zeros((B,), dtype=torch.int32)
        b_start_loc_extend[1:] = torch.cumsum(b_seq_len_extend[:-1], 0)

        for i in range(B):
            req_to_tokens[i, : b_seq_len[i]] = torch.arange(
                b_start_loc[i], b_start_loc[i] + b_seq_len[i]
            )

        total_token_num = torch.sum(b_seq_len).item()
        extend_token_num = torch.sum(b_seq_len_extend).item()

        H_BUF = 1 if mla else H_KV
        k_buffer = torch.randn((total_token_num, H_BUF, D), dtype=dtype)
        v_buffer = torch.randn((total_token_num, H_BUF, DV), dtype=dtype)

        k_extend = torch.empty((extend_token_num, H_KV, D), dtype=dtype)
        v_extend = torch.empty((extend_token_num, H_KV, DV), dtype=dtype)
        q_extend = torch.empty((extend_token_num, H_Q, D), dtype=dtype)
```
**EN:** This helper function encapsulates reusable logic inside `TestExtendAttention` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestExtendAttention` 内部调用，从而让场景结构更清晰。

### Lines 133-182: Define helper: test extend attention once (part 2)
```python
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
            q_extend[extend_start:extend_end] = (
                torch.randn((b_seq_len_extend[i], H_Q, D), dtype=dtype) * 20
            )

        # q_extend, k_extend, v_extend, k_buffer and v_buffer supports non-contiguous tensors
        q_extend = q_extend.transpose(0, 1).contiguous().transpose(0, 1)
        k_extend = k_extend.transpose(0, 1).contiguous().transpose(0, 1)
        v_extend = v_extend.transpose(0, 1).contiguous().transpose(0, 1)
        k_buffer = k_buffer.transpose(0, 1).contiguous().transpose(0, 1)
        v_buffer = v_buffer.transpose(0, 1).contiguous().transpose(0, 1)

        b_seq_len_extend = b_seq_len - b_seq_len_prefix
        b_start_loc_extend = torch.zeros_like(b_seq_len)
        b_start_loc_extend[1:] = torch.cumsum(b_seq_len_extend[:-1], 0)
        max_len_extend = torch.max(b_seq_len_extend, 0)[0].item()

        sm_scale = 1.0 / (D**0.5)
        logit_cap = 0.0

        # handle index type
        b_req_idx = b_req_idx.to(torch.int64)
        b_seq_len = b_seq_len.to(torch.int64)

        enable_gqa = H_Q != H_KV
        o_ref = torch.empty((extend_token_num, H_Q, DV), dtype=dtype)
        self._run_sdpa_forward_extend(
            q_extend,
            o_ref,
            k_buffer,
            v_buffer,
            req_to_tokens,
            b_req_idx,
            b_seq_len,
            b_seq_len_prefix,
            b_seq_len_extend,
            scaling=sm_scale,
            enable_gqa=enable_gqa,
            causal=True,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestExtendAttention` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestExtendAttention` 内部调用，从而让场景结构更清晰。

### Lines 184-200: Define helper: test extend attention once (part 3)
```python
        o_extend = torch.empty((extend_token_num, H_Q, DV), dtype=dtype)
        torch.ops.sgl_kernel.extend_attention_cpu(
            q_extend,
            k_extend,
            v_extend,
            o_extend,
            k_buffer,
            v_buffer,
            req_to_tokens,
            b_req_idx,
            b_seq_len,
            b_seq_len_extend,
            b_start_loc_extend,
            max_len_extend,
            sm_scale,
            logit_cap,
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestExtendAttention` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestExtendAttention` 内部调用，从而让场景结构更清晰。

### Lines 202-202: Define helper: test extend attention once (part 4)
```python
        torch.testing.assert_close(o_ref, o_extend, atol=1e-2, rtol=1e-2)
```
**EN:** This helper function encapsulates reusable logic inside `TestExtendAttention` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestExtendAttention` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 204-209: Run test: extend attention
```python
    def test_extend_attention(self):
        for is_mla in [True, False]:
            self._test_extend_attention_once(1, 123, 1, 1, 128, 96, is_mla)
            self._test_extend_attention_once(1, 123, 16, 1, 128, 96, is_mla)
            self._test_extend_attention_once(4, 1230, 16, 4, 128, 96, is_mla)
            self._test_extend_attention_once(1, 9000, 16, 1, 32, 32, is_mla)
```
**EN:** This test method exercises extend attention and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 extend attention 场景，并验证观测到的行为是否符合预期契约。

### Lines 211-221: Run test: extend attention large seq causal mask
```python
    def test_extend_attention_large_seq_causal_mask(self):
        self._test_extend_attention_once(
            B=1,
            N_CTX=5001,
            H_Q=8,
            H_KV=2,
            D=64,
            DV=64,
            b_seq_len_prefix=[0],
            b_seq_len_extend=[5000],
        )
```
**EN:** This test method exercises extend attention large seq causal mask and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 extend attention large seq causal mask 场景，并验证观测到的行为是否符合预期契约。

### Lines 224-225: Expose unittest entrypoint
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
