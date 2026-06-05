# ascend_torch_native_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/npu/attention/ascend_torch_native_backend.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements NPU backend support for attention execution and masking inside the SGLang runtime. / 为 SGLang 运行时提供面向 NPU/Ascend 后端的注意力执行与掩码处理支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Module setup and shared state / 模块设置与共享状态
```python
from __future__ import annotations

import math

import torch
from torch.nn.functional import scaled_dot_product_attention
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `__future__`, `math`, `torch`, `torch.nn.functional`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `__future__`, `math`, `torch`, `torch.nn.functional`。

### Lines 9-9: Class `AscendTorchNativeAttnBackend` declaration / 类 `AscendTorchNativeAttnBackend` 声明
```python
class AscendTorchNativeAttnBackend:
```
**EN:** This class establishes `AscendTorchNativeAttnBackend` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `scaled_dot_product_attention_with_softcapping`, `run_sdpa_forward_extend`, `run_sdpa_forward_decode`, `support_triton`.
**CN:** 该类将 `AscendTorchNativeAttnBackend` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `scaled_dot_product_attention_with_softcapping`, `run_sdpa_forward_extend`, `run_sdpa_forward_decode`, `support_triton` 等方法。

### Lines 10-11: Method `AscendTorchNativeAttnBackend.__init__` / 方法 `AscendTorchNativeAttnBackend.__init__`
```python
    def __init__(self):
        pass
```
**EN:** This method implements `__init__` on `AscendTorchNativeAttnBackend`.
**CN:** 该方法（属于 `AscendTorchNativeAttnBackend`）实现了 `__init__`。

### Lines 13-54: Method `AscendTorchNativeAttnBackend.scaled_dot_product_attention_with_softcapping` / 方法 `AscendTorchNativeAttnBackend.scaled_dot_product_attention_with_softcapping`
```python
    def scaled_dot_product_attention_with_softcapping(
        self,
        query,
        key,
        value,
        attn_mask=None,
        is_causal=False,
        scale=None,
        enable_gqa=False,
        logit_cap=0.0,
        logit_capping_method="tanh",
    ) -> torch.Tensor:
        L, S = query.size(-2), key.size(-2)
        scale_factor = 1 / math.sqrt(query.size(-1)) if scale is None else scale
        attn_bias = torch.zeros(L, S, dtype=query.dtype, device=query.device)
        if is_causal:
            assert attn_mask is None
            temp_mask = torch.ones(L, S, dtype=torch.bool, device=query.device).tril(
                diagonal=0
            )
            attn_bias.masked_fill_(temp_mask.logical_not(), float("-inf"))
            attn_bias.to(query.dtype)

        if attn_mask is not None:
            if attn_mask.dtype == torch.bool:
                attn_bias.masked_fill_(attn_mask.logical_not(), float("-inf"))
            else:
                attn_bias = attn_mask + attn_bias
# ... omitted for brevity ...
            if logit_capping_method == "tanh":
                attn_weight = logit_cap * torch.tanh(attn_weight / logit_cap)

        attn_weight += attn_bias
        attn_weight = torch.softmax(attn_weight, dim=-1)
        return attn_weight @ value
```
**EN:** This method implements `scaled_dot_product_attention_with_softcapping` on `AscendTorchNativeAttnBackend`. It primarily calls `torch.zeros`, `torch.softmax`, `query.size`, `key.size`, `torch.ones.tril`, `attn_bias.masked_fill_` to complete its work. State updates are written into `scale_factor`, `attn_bias`, `attn_weight`, `temp_mask`, `key`, `value`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `AscendTorchNativeAttnBackend`）实现了 `scaled_dot_product_attention_with_softcapping`。 它主要通过调用 `torch.zeros`, `torch.softmax`, `query.size`, `key.size`, `torch.ones.tril`, `attn_bias.masked_fill_` 来完成任务。 状态更新主要写入 `scale_factor`, `attn_bias`, `attn_weight`, `temp_mask`, `key`, `value`。 实现中使用了条件分支。

### Lines 56-174: Method `AscendTorchNativeAttnBackend.run_sdpa_forward_extend` / 方法 `AscendTorchNativeAttnBackend.run_sdpa_forward_extend`
```python
    def run_sdpa_forward_extend(
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
        encoder_lens: torch.Tensor = None,
        is_cross_attention: bool = False,
        scaling=None,
        enable_gqa=False,
        causal=False,
        logit_cap: float = 0.0,
        logit_capping_method: str = "tanh",
    ):
        """Run the extend forward by using torch native sdpa op.

        Args:
            query: [num_tokens, num_heads, head_size]
            output: [num_tokens, num_heads, head_size]
            k_cache: [max_total_num_tokens, num_heads, head_size]
            v_cache: [max_total_num_tokens, num_heads, head_size]
            req_to_token: [max_num_reqs, max_context_len]
            req_pool_indices: [num_seqs]
# ... omitted for brevity ...
                    .squeeze(0)
                    .movedim(query.dim() - 2, 0)
                )
            output[start_q:end_q, :, :] = per_req_out_redudant[prefill_seq_len_q:, :, :]
            start_q, start_kv = end_q, end_kv
        return output
```
**EN:** This method implements `run_sdpa_forward_extend` on `AscendTorchNativeAttnBackend`. It primarily calls `query.movedim`, `range`, `torch.empty`, `k_cache.movedim`, `v_cache.movedim`, `query.dim` to complete its work. State updates are written into `query`, `extend_seq_len_q`, `prefill_seq_len_q`, `seq_len_kv`, `end_q`, `end_kv`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `AscendTorchNativeAttnBackend`）实现了 `run_sdpa_forward_extend`。 它主要通过调用 `query.movedim`, `range`, `torch.empty`, `k_cache.movedim`, `v_cache.movedim`, `query.dim` 来完成任务。 状态更新主要写入 `query`, `extend_seq_len_q`, `prefill_seq_len_q`, `seq_len_kv`, `end_q`, `end_kv`。 实现中使用了条件分支、迭代逻辑。

### Lines 176-279: Method `AscendTorchNativeAttnBackend.run_sdpa_forward_decode` / 方法 `AscendTorchNativeAttnBackend.run_sdpa_forward_decode`
```python
    def run_sdpa_forward_decode(
        self,
        query: torch.Tensor,
        output: torch.Tensor,
        k_cache: torch.Tensor,
        v_cache: torch.Tensor,
        req_to_token: torch.Tensor,
        req_pool_indices: torch.Tensor,
        seq_lens: torch.Tensor,
        encoder_lens: torch.Tensor = None,
        is_cross_attention: bool = False,
        scaling=None,
        enable_gqa=False,
        causal=False,
        logit_cap: float = 0.0,
        logit_capping_method: str = "tanh",
    ):
        """Run the decode forward by using torch native sdpa op.

        Args:
            query: [num_tokens, num_heads, head_size]
            output: [num_tokens, num_heads, head_size]
            k_cache: [max_total_num_tokens, num_heads, head_size]
            v_cache: [max_total_num_tokens, num_heads, head_size]
            req_to_token: [max_num_reqs, max_context_len]
            req_pool_indices: [num_seqs]
            seq_lens: [num_seqs]
            encoder_lens: [num_seqs]
# ... omitted for brevity ...
                    .movedim(query.dim() - 2, 0)
                )
            output[start_q:end_q, :, :] = per_req_out
            start_q, start_kv = end_q, end_kv

        return output
```
**EN:** This method implements `run_sdpa_forward_decode` on `AscendTorchNativeAttnBackend`. It primarily calls `query.movedim`, `range`, `k_cache.movedim`, `v_cache.movedim`, `query.dim`, `per_req_key.to` to complete its work. State updates are written into `query`, `seq_len_q`, `seq_len_kv`, `end_q`, `end_kv`, `atten_start_kv`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `AscendTorchNativeAttnBackend`）实现了 `run_sdpa_forward_decode`。 它主要通过调用 `query.movedim`, `range`, `k_cache.movedim`, `v_cache.movedim`, `query.dim`, `per_req_key.to` 来完成任务。 状态更新主要写入 `query`, `seq_len_q`, `seq_len_kv`, `end_q`, `end_kv`, `atten_start_kv`。 实现中使用了条件分支、迭代逻辑。

### Lines 281-282: Method `AscendTorchNativeAttnBackend.support_triton` / 方法 `AscendTorchNativeAttnBackend.support_triton`
```python
    def support_triton(self):
        return False
```
**EN:** This method implements `support_triton` on `AscendTorchNativeAttnBackend`.
**CN:** 该方法（属于 `AscendTorchNativeAttnBackend`）实现了 `support_triton`。

## Key Concepts / 关键概念
- **Classes / 类**: `AscendTorchNativeAttnBackend`
- **Functions / 函数**: `__init__`, `scaled_dot_product_attention_with_softcapping`, `run_sdpa_forward_extend`, `run_sdpa_forward_decode`, `support_triton`
- **Themes / 主题**: `attention`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: None / 无
- **External / 外部依赖**: `torch`, `torch.nn.functional`
- **Standard library / 标准库**: `__future__`, `math`
