# test_extend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/srt/cpu/test_extend.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates extend behavior in SGLang's srt / cpu area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 SRT / CPU 领域中与 extend 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: module imports and dependencies / 模块导入与依赖
```python
import unittest

import torch
from torch.nn.functional import scaled_dot_product_attention

from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `torch`, `torch.nn.functional`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `torch`, `torch.nn.functional`, `sglang.test.test_utils`。

### Lines 8-8: supporting statements / 辅助语句
```python
torch.manual_seed(1234)
```
**EN:** This block performs supporting work through calls such as `manual_seed`, preparing state for nearby definitions.
**CN:** 该代码块通过 `manual_seed` 等调用完成辅助性工作，为相邻定义准备状态。

### Lines 11-12: class TestExtendAttention declaration / 类 TestExtendAttention 声明
```python
class TestExtendAttention(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 13-75: method run sdpa forward extend / 方法 run sdpa forward extend
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
**EN:** This block implements `_run_sdpa_forward_extend` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_sdpa_forward_extend`，承担模块行为中的一个聚焦逻辑片段。

### Lines 77-156: method test extend attention once (part 1/2) / 方法 test extend attention once（第 1/2 部分）
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

```
**EN:** This block implements `_test_extend_attention_once` and captures one focused piece of the module's behavior. This chunk continues part 1 of the same logical block.
**CN:** 该代码块实现 `_test_extend_attention_once`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 1 部分。

### Lines 157-199: method test extend attention once (part 2/2) / 方法 test extend attention once（第 2/2 部分）
```python
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

        torch.testing.assert_close(o_ref, o_extend, atol=1e-2, rtol=1e-2)
```
**EN:** This block implements `_test_extend_attention_once` and captures one focused piece of the module's behavior. This chunk continues part 2 of the same logical block.
**CN:** 该代码块实现 `_test_extend_attention_once`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 2 部分。

### Lines 201-206: test case extend attention / 测试用例 extend attention
```python
    def test_extend_attention(self):
        for is_mla in [True, False]:
            self._test_extend_attention_once(1, 123, 1, 1, 128, 96, is_mla)
            self._test_extend_attention_once(1, 123, 16, 1, 128, 96, is_mla)
            self._test_extend_attention_once(4, 1230, 16, 4, 128, 96, is_mla)
            self._test_extend_attention_once(1, 9000, 16, 1, 32, 32, is_mla)
```
**EN:** This test exercises `test_extend_attention` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_extend_attention`。

### Lines 208-218: test case extend attention large seq causal mask / 测试用例 extend attention large seq causal mask
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
**EN:** This test exercises `test_extend_attention_large_seq_causal_mask` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_extend_attention_large_seq_causal_mask`。

### Lines 220-230: test case extend attention gqa partial extend with prefix / 测试用例 extend attention gqa partial extend with prefix
```python
    def test_extend_attention_gqa_partial_extend_with_prefix(self):
        self._test_extend_attention_once(
            B=1,
            N_CTX=256,
            H_Q=16,
            H_KV=4,
            D=128,
            DV=96,
            b_seq_len_prefix=[97],
            b_seq_len_extend=[37],
        )
```
**EN:** This test exercises `test_extend_attention_gqa_partial_extend_with_prefix` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_extend_attention_gqa_partial_extend_with_prefix`。

### Lines 233-234: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestExtendAttention`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestExtendAttention._run_sdpa_forward_extend`: This block implements `_run_sdpa_forward_extend` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_sdpa_forward_extend`，承担模块行为中的一个聚焦逻辑片段。
- `TestExtendAttention._test_extend_attention_once`: This block implements `_test_extend_attention_once` and captures one focused piece of the module's behavior. / 该代码块实现 `_test_extend_attention_once`，承担模块行为中的一个聚焦逻辑片段。
- `TestExtendAttention.test_extend_attention`: This test exercises `test_extend_attention` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_extend_attention`。
- `TestExtendAttention.test_extend_attention_large_seq_causal_mask`: This test exercises `test_extend_attention_large_seq_causal_mask` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_extend_attention_large_seq_causal_mask`。
- `TestExtendAttention.test_extend_attention_gqa_partial_extend_with_prefix`: This test exercises `test_extend_attention_gqa_partial_extend_with_prefix` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_extend_attention_gqa_partial_extend_with_prefix`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `torch`, `torch.nn.functional`
- **Internal modules / 内部模块**: `sglang.test.test_utils`

- **Total lines / 总行数**: 234
