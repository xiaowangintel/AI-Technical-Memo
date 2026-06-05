# test_decode.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/srt/cpu/test_decode.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates decode behavior in SGLang's srt / cpu area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 SRT / CPU 领域中与 decode 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

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

### Lines 11-11: class TestDecodeAttention declaration / 类 TestDecodeAttention 声明
```python
class TestDecodeAttention(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 12-59: method run sdpa forward decode / 方法 run sdpa forward decode
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
**EN:** This block implements `_run_sdpa_forward_decode` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_sdpa_forward_decode`，承担模块行为中的一个聚焦逻辑片段。

### Lines 61-140: method test grouped decode attention once (part 1/2) / 方法 test grouped decode attention once（第 1/2 部分）
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

        cos_sim = torch.nn.functional.cosine_similarity(
            o.flatten(), o_grouped.flatten(), dim=0
        )
        self.assertGreater(cos_sim.item(), 0.99)
```
**EN:** This block implements `_test_grouped_decode_attention_once` and captures one focused piece of the module's behavior. This chunk continues part 1 of the same logical block.
**CN:** 该代码块实现 `_test_grouped_decode_attention_once`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 1 部分。

### Lines 141-141: method test grouped decode attention once (part 2/2) / 方法 test grouped decode attention once（第 2/2 部分）
```python
        torch.testing.assert_close(o, o_grouped, atol=3e-2, rtol=1e-6)
```
**EN:** This block implements `_test_grouped_decode_attention_once` and captures one focused piece of the module's behavior. This chunk continues part 2 of the same logical block.
**CN:** 该代码块实现 `_test_grouped_decode_attention_once`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 2 部分。

### Lines 143-162: method test grouped decode attention / 方法 test grouped decode attention
```python
    def _test_grouped_decode_attention(self, device="cuda"):
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
**EN:** This block implements `_test_grouped_decode_attention` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_test_grouped_decode_attention`，承担模块行为中的一个聚焦逻辑片段。

### Lines 164-165: test case grouped decode attention / 测试用例 grouped decode attention
```python
    def test_grouped_decode_attention(self):
        self._test_grouped_decode_attention("cpu")
```
**EN:** This test exercises `test_grouped_decode_attention` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_grouped_decode_attention`。

### Lines 168-169: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestDecodeAttention`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDecodeAttention._run_sdpa_forward_decode`: This block implements `_run_sdpa_forward_decode` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_sdpa_forward_decode`，承担模块行为中的一个聚焦逻辑片段。
- `TestDecodeAttention._test_grouped_decode_attention_once`: This block implements `_test_grouped_decode_attention_once` and captures one focused piece of the module's behavior. / 该代码块实现 `_test_grouped_decode_attention_once`，承担模块行为中的一个聚焦逻辑片段。
- `TestDecodeAttention._test_grouped_decode_attention`: This block implements `_test_grouped_decode_attention` and captures one focused piece of the module's behavior. / 该代码块实现 `_test_grouped_decode_attention`，承担模块行为中的一个聚焦逻辑片段。
- `TestDecodeAttention.test_grouped_decode_attention`: This test exercises `test_grouped_decode_attention` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_grouped_decode_attention`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `torch`, `torch.nn.functional`
- **Internal modules / 内部模块**: `sglang.test.test_utils`

- **Total lines / 总行数**: 169
