# test_mla.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/srt/cpu/test_mla.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates mla behavior in SGLang's srt / cpu area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 SRT / CPU 领域中与 mla 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: module imports and dependencies / 模块导入与依赖
```python
import unittest

import torch
from torch.nn.functional import scaled_dot_product_attention
from utils import precision

from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `torch`, `torch.nn.functional`, `utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `torch`, `torch.nn.functional`, `utils`。

### Lines 9-9: supporting statements / 辅助语句
```python
torch.manual_seed(1234)
```
**EN:** This block performs supporting work through calls such as `manual_seed`, preparing state for nearby definitions.
**CN:** 该代码块通过 `manual_seed` 等调用完成辅助性工作，为相邻定义准备状态。

### Lines 12-12: class TestMLA declaration / 类 TestMLA 声明
```python
class TestMLA(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 13-65: method run sdpa forward decode / 方法 run sdpa forward decode
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
**EN:** This block implements `_run_sdpa_forward_decode` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_sdpa_forward_decode`，承担模块行为中的一个聚焦逻辑片段。

### Lines 67-141: method test grouped decode attention once / 方法 test grouped decode attention once
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

        cos_sim = torch.nn.functional.cosine_similarity(
            o.flatten(), o_grouped.flatten(), dim=0
        )
        atol = rtol = precision[q.dtype]
        self.assertGreater(cos_sim.item(), 0.99)
        torch.testing.assert_close(o, o_grouped, atol=atol, rtol=rtol)
        torch.testing.assert_close(k_buffer, k_buffer2, atol=atol, rtol=rtol)
        torch.testing.assert_close(v_buffer, v_buffer2, atol=atol, rtol=rtol)
```
**EN:** This block implements `_test_grouped_decode_attention_once` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_test_grouped_decode_attention_once`，承担模块行为中的一个聚焦逻辑片段。

### Lines 143-151: test case grouped decode attention / 测试用例 grouped decode attention
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
**EN:** This test exercises `test_grouped_decode_attention` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_grouped_decode_attention`。

### Lines 154-155: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestMLA`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestMLA._run_sdpa_forward_decode`: This block implements `_run_sdpa_forward_decode` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_sdpa_forward_decode`，承担模块行为中的一个聚焦逻辑片段。
- `TestMLA._test_grouped_decode_attention_once`: This block implements `_test_grouped_decode_attention_once` and captures one focused piece of the module's behavior. / 该代码块实现 `_test_grouped_decode_attention_once`，承担模块行为中的一个聚焦逻辑片段。
- `TestMLA.test_grouped_decode_attention`: This test exercises `test_grouped_decode_attention` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_grouped_decode_attention`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `torch`, `torch.nn.functional`, `utils`
- **Internal modules / 内部模块**: `sglang.test.test_utils`

- **Total lines / 总行数**: 155
