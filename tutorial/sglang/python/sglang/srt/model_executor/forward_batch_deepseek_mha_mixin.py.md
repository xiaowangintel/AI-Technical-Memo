# forward_batch_deepseek_mha_mixin.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/model_executor/forward_batch_deepseek_mha_mixin.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the model execution and graph runners part of the SRT runtime and implements logic centered on `forward_batch_deepseek_mha_mixin`. It exposes primary entry points such as `ForwardBatchDeepSeekMHAMixin`, `create_chunked_prefix_cache_kv_indices`. / 该模块属于 SRT 运行时的模型执行与图运行器部分，主要实现围绕 `forward_batch_deepseek_mha_mixin` 的逻辑。 它对外提供的主要入口包括 `ForwardBatchDeepSeekMHAMixin`, `create_chunked_prefix_cache_kv_indices`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Module imports, constants, and setup
```python
# Mixin class for metadata management of Deepseek MHA forward (chunked prefix cache)
# More details can be found in python/sglang/srt/models/deepseek_common/attention_forward_methods/forward_mha.py

from typing import List, Optional

import torch
import triton
import triton.language as tl

from sglang.srt.environ import envs
from sglang.srt.layers.attention.utils import create_flashinfer_kv_indices_triton


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; reads environment-driven configuration.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；读取环境变量驱动的配置。

### Lines 14-46: Class ForwardBatchDeepSeekMHAMixin
```python
class ForwardBatchDeepSeekMHAMixin:
    # For MLA chunked prefix cache used in chunked prefill
    # Tell attention backend whether the kv cache needs to be attended in current pass
    attn_attend_prefix_cache: Optional[bool] = None
    # Number of prefix cache chunks
    num_prefix_chunks: Optional[int] = None
    # Index of current chunk, used by attention backend
    prefix_chunk_idx: Optional[int] = None
    # Maximum number of tokens in each chunk per sequence. Computed from maximum chunk capacity
    prefix_chunk_len: Optional[int] = None
    # Start positions of prefix cache for each chunk, (num_prefix_chunks, batch_size)
    prefix_chunk_starts: Optional[torch.Tensor] = None
    # Lengths of prefix cache for each chunk, (num_prefix_chunks, batch_size)
    prefix_chunk_seq_lens: Optional[torch.Tensor] = None
    # Accumulated lengths of prefix cache for each chunk, (num_prefix_chunks, batch_size + 1)
    prefix_chunk_cu_seq_lens: Optional[torch.Tensor] = None
    # Max lengths of prefix cache for each chunk, (num_prefix_chunks,)
    prefix_chunk_max_seq_lens: Optional[List[int]] = None
    # Per-chunk flag: True if any sequence has kv_len==0 in that chunk.
    # Precomputed on CPU to avoid GPU-CPU sync in the hot path.
    prefix_chunk_has_zero_kv: Optional[List[bool]] = None
    # Number of tokens in each prefix cache chunk, (num_prefix_chunks,)
    prefix_chunk_num_tokens: Optional[List[int]] = None
    # KV Indices for each chunk
    prefix_chunk_kv_indices: Optional[List[torch.Tensor]] = None
    # For MLA chunked prefix cache used in chunked prefill
    # Tell attention backend whether lse needs to be returned
    mha_return_lse: Optional[bool] = None
    # Whether to apply MHA_ONE_SHOT forward method
    mha_one_shot: Optional[bool] = None
    # KV Indices for MHA_ONE_SHOT forward method
    mha_one_shot_kv_indices: Optional[torch.Tensor] = None

```
**EN:** This range introduces `ForwardBatchDeepSeekMHAMixin` and defines the structure or metadata that its methods rely on. In this range it sets up imports and shared symbols.
**CN:** 这一段引入 `ForwardBatchDeepSeekMHAMixin`，并定义其后续方法依赖的结构或元数据。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 47-48: Method ForwardBatchDeepSeekMHAMixin.get_max_chunk_capacity
```python
    def get_max_chunk_capacity(self):
        return envs.SGLANG_MAX_KV_CHUNK_CAPACITY.get()
```
**EN:** This callable implements `ForwardBatchDeepSeekMHAMixin.get_max_chunk_capacity` and mainly retrieves a value or derived view. In this range it reads environment-driven configuration.
**CN:** 这一可调用对象实现了 `ForwardBatchDeepSeekMHAMixin.get_max_chunk_capacity`，主要用于获取某个值或派生视图。 在这一范围内，它会读取环境变量驱动的配置。

### Lines 50-51: Method ForwardBatchDeepSeekMHAMixin.set_prefix_chunk_idx
```python
    def set_prefix_chunk_idx(self, idx: int):
        self.prefix_chunk_idx = idx
```
**EN:** This callable implements `ForwardBatchDeepSeekMHAMixin.set_prefix_chunk_idx`. It takes `idx` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `ForwardBatchDeepSeekMHAMixin.set_prefix_chunk_idx`。它接收 `idx`，主要用于将配置写入可变状态。

### Lines 53-54: Method ForwardBatchDeepSeekMHAMixin.set_attn_attend_prefix_cache
```python
    def set_attn_attend_prefix_cache(self, attn_attend_prefix_cache: bool):
        self.attn_attend_prefix_cache = attn_attend_prefix_cache
```
**EN:** This callable implements `ForwardBatchDeepSeekMHAMixin.set_attn_attend_prefix_cache`. It takes `attn_attend_prefix_cache` and mainly applies configuration to mutable state.
**CN:** 这一可调用对象实现了 `ForwardBatchDeepSeekMHAMixin.set_attn_attend_prefix_cache`。它接收 `attn_attend_prefix_cache`，主要用于将配置写入可变状态。

### Lines 56-77: Method ForwardBatchDeepSeekMHAMixin.prepare_chunked_kv_indices
```python
    def prepare_chunked_kv_indices(self, device: torch.device):
        self.prefix_chunk_kv_indices = []
        for idx in range(self.num_prefix_chunks):
            chunk_starts = self.prefix_chunk_starts[idx]
            chunk_seq_lens = self.prefix_chunk_seq_lens[idx]
            chunk_cu_seq_lens = self.prefix_chunk_cu_seq_lens[idx]
            num_chunk_tokens = self.prefix_chunk_num_tokens[idx]

            chunk_kv_indices = torch.empty(
                num_chunk_tokens, dtype=torch.int32, device=device
            )

            create_chunked_prefix_cache_kv_indices[(self.batch_size,)](
                self.req_to_token_pool.req_to_token,
                self.req_pool_indices,
                chunk_starts,
                chunk_seq_lens,
                chunk_cu_seq_lens,
                chunk_kv_indices,
                self.req_to_token_pool.req_to_token.shape[1],
            )
            self.prefix_chunk_kv_indices.append(chunk_kv_indices)
```
**EN:** This callable implements `ForwardBatchDeepSeekMHAMixin.prepare_chunked_kv_indices`. It takes `device` and mainly prepares runtime inputs.
**CN:** 这一可调用对象实现了 `ForwardBatchDeepSeekMHAMixin.prepare_chunked_kv_indices`。它接收 `device`，主要用于准备运行时输入。

### Lines 78-85: Class-level scaffolding for ForwardBatchDeepSeekMHAMixin
```python

    # Here we suppose the length of each chunk is equal
    # For example, if we have 4 sequences with prefix length [256, 512, 768, 1024], prefix_chunk_len = 256
    # num_prefix_chunks = cdiv(1024, 256) = 4
    # prefix_chunk_starts = [[0, 0, 0, 0], [256, 256, 256, 256], [512, 512, 512, 512], [768, 768, 768, 768]]
    # prefix_chunk_ends = [[256, 256, 256, 256], [256, 512, 512, 512], [256, 512, 768, 768], [256, 512, 768, 1024]]
    # prefix_chunk_seq_lens = [[256, 256, 256, 256], [0, 256, 256, 256], [0, 0, 256, 256], [0, 0, 0, 256]]
    # TODO: Implement a better way to allocate chunk lengths that uses memory spaces more efficiently.
```
**EN:** This callable implements `None.ForwardBatchDeepSeekMHAMixin` and mainly implements Forward Batch Deep Seek MHAMixin.
**CN:** 这一可调用对象实现了 `None.ForwardBatchDeepSeekMHAMixin`，主要用于实现 Forward Batch Deep Seek MHAMixin 相关逻辑。

### Lines 86-105: Method ForwardBatchDeepSeekMHAMixin.get_prefix_chunk_seq_lens
```python
    def get_prefix_chunk_seq_lens(
        self, prefix_lens: torch.Tensor, num_prefix_chunks: int, prefix_chunk_len: int
    ):
        device = prefix_lens.device
        prefix_chunk_starts = (
            torch.arange(num_prefix_chunks, device=device, dtype=torch.int32)
            .unsqueeze(1)
            .expand(-1, self.batch_size)
            * prefix_chunk_len
        )
        prefix_chunk_ends = torch.min(
            prefix_lens.unsqueeze(0),
            prefix_chunk_starts + prefix_chunk_len,
        ).to(torch.int32)

        prefix_chunk_seq_lens = (
            (prefix_chunk_ends - prefix_chunk_starts).clamp(min=0).to(torch.int32)
        )

        return prefix_chunk_starts, prefix_chunk_seq_lens
```
**EN:** This callable implements `ForwardBatchDeepSeekMHAMixin.get_prefix_chunk_seq_lens`. It takes `prefix_lens`, `num_prefix_chunks`, `prefix_chunk_len` and mainly retrieves a value or derived view.
**CN:** 这一可调用对象实现了 `ForwardBatchDeepSeekMHAMixin.get_prefix_chunk_seq_lens`。它接收 `prefix_lens`, `num_prefix_chunks`, `prefix_chunk_len`，主要用于获取某个值或派生视图。

### Lines 106-108: Class-level scaffolding for ForwardBatchDeepSeekMHAMixin
```python

    # Called before each attention module if using chunked kv cache for prefill
    # Some of the codes are adapted from https://github.com/vllm-project/vllm/blob/main/vllm/v1/attention/backends/mla/common.py
```
**EN:** This callable implements `None.ForwardBatchDeepSeekMHAMixin` and mainly implements Forward Batch Deep Seek MHAMixin. In this range it sets up imports and shared symbols.
**CN:** 这一可调用对象实现了 `None.ForwardBatchDeepSeekMHAMixin`，主要用于实现 Forward Batch Deep Seek MHAMixin 相关逻辑。 在这一范围内，它会建立导入关系并准备共享符号。

### Lines 109-168: Method ForwardBatchDeepSeekMHAMixin.prepare_chunked_prefix_cache_info (part 1/2)
```python
    def prepare_chunked_prefix_cache_info(self, device: torch.device):

        from sglang.srt.mem_cache.memory_pool import MLATokenToKVPool

        assert isinstance(
            self.token_to_kv_pool, MLATokenToKVPool
        ), "Currently chunked prefix cache can only be used by Deepseek models"

        if not any(self.extend_prefix_lens_cpu):
            self.num_prefix_chunks = 0
            return

        if self.prefix_chunk_len is not None:
            # Chunked kv cache info already prepared by prior modules
            return

        self.prefix_chunk_idx = -1

        # chunk_capacity is the maximum number of tokens in each chunk
        chunk_capacity = self.get_max_chunk_capacity()
        self.prefix_chunk_len = chunk_capacity // self.batch_size

        self.num_prefix_chunks = (
            max(self.extend_prefix_lens_cpu) + self.prefix_chunk_len - 1
        ) // self.prefix_chunk_len

        # Here we compute chunk lens twice to avoid stream sync, once on gpu and once on cpu.
        prefix_chunk_starts_cuda, prefix_chunk_seq_lens_cuda = (
            self.get_prefix_chunk_seq_lens(
                self.extend_prefix_lens,
                self.num_prefix_chunks,
                self.prefix_chunk_len,
            )
        )
        _, prefix_chunk_seq_lens_cpu = self.get_prefix_chunk_seq_lens(
            torch.tensor(self.extend_prefix_lens_cpu),
            self.num_prefix_chunks,
            self.prefix_chunk_len,
        )
        self.prefix_chunk_starts = prefix_chunk_starts_cuda
        self.prefix_chunk_seq_lens = prefix_chunk_seq_lens_cuda

        # Metadata for attention backend
        self.prefix_chunk_cu_seq_lens = torch.zeros(
            self.num_prefix_chunks,
            self.batch_size + 1,
            device=device,
            dtype=torch.int32,
        )
        self.prefix_chunk_cu_seq_lens[:, 1:] = prefix_chunk_seq_lens_cuda.cumsum(
            dim=1
        ).to(torch.int32)
        self.prefix_chunk_max_seq_lens = prefix_chunk_seq_lens_cpu.max(
            dim=1
        ).values.tolist()

        self.prefix_chunk_num_tokens = prefix_chunk_seq_lens_cpu.sum(dim=1).tolist()
        assert max(self.prefix_chunk_num_tokens) <= self.get_max_chunk_capacity()

        # Per-chunk flag: does any sequence have kv_len == 0?
```
**EN:** This callable implements `ForwardBatchDeepSeekMHAMixin.prepare_chunked_prefix_cache_info`. It takes `device` and mainly prepares runtime inputs. This chunk is part 1 of 2 for the same logical block. In this range it sets up imports and shared symbols; performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `ForwardBatchDeepSeekMHAMixin.prepare_chunked_prefix_cache_info`。它接收 `device`，主要用于准备运行时输入。 该片段是同一逻辑块的第 1/2 部分。 在这一范围内，它会建立导入关系并准备共享符号；对非法状态执行防御性检查。

### Lines 169-176: Method ForwardBatchDeepSeekMHAMixin.prepare_chunked_prefix_cache_info (part 2/2)
```python
        # Pure CPU check (prefix_chunk_seq_lens_cpu is on CPU), no GPU sync.
        self.prefix_chunk_has_zero_kv = [
            bool((prefix_chunk_seq_lens_cpu[i] == 0).any())
            for i in range(self.num_prefix_chunks)
        ]

        # Precompute the kv indices for each chunk
        self.prepare_chunked_kv_indices(device)
```
**EN:** This callable implements `ForwardBatchDeepSeekMHAMixin.prepare_chunked_prefix_cache_info`. It takes `device` and mainly prepares runtime inputs. This chunk is part 2 of 2 for the same logical block.
**CN:** 这一可调用对象实现了 `ForwardBatchDeepSeekMHAMixin.prepare_chunked_prefix_cache_info`。它接收 `device`，主要用于准备运行时输入。 该片段是同一逻辑块的第 2/2 部分。

### Lines 178-204: Method ForwardBatchDeepSeekMHAMixin.fetch_mha_one_shot_kv_indices
```python
    def fetch_mha_one_shot_kv_indices(self):
        if self.mha_one_shot_kv_indices is not None:
            return self.mha_one_shot_kv_indices
        batch_size = self.batch_size
        paged_kernel_lens_sum = sum(self.seq_lens_cpu)
        kv_indices = torch.empty(
            paged_kernel_lens_sum,
            dtype=torch.int32,
            device=self.req_pool_indices.device,
        )
        kv_indptr = torch.zeros(
            batch_size + 1,
            dtype=torch.int32,
            device=self.req_pool_indices.device,
        )
        kv_indptr[1:] = torch.cumsum(self.seq_lens, dim=0)
        create_flashinfer_kv_indices_triton[(self.batch_size,)](
            self.req_to_token_pool.req_to_token,
            self.req_pool_indices,
            self.seq_lens,
            kv_indptr,
            None,
            kv_indices,
            self.req_to_token_pool.req_to_token.shape[1],
        )
        self.mha_one_shot_kv_indices = kv_indices
        return kv_indices
```
**EN:** This callable implements `ForwardBatchDeepSeekMHAMixin.fetch_mha_one_shot_kv_indices` and mainly implements fetch mha one shot kv indices.
**CN:** 这一可调用对象实现了 `ForwardBatchDeepSeekMHAMixin.fetch_mha_one_shot_kv_indices`，主要用于实现 fetch mha one shot kv indices 相关逻辑。

### Lines 207-241: Function create_chunked_prefix_cache_kv_indices
```python
@triton.jit
def create_chunked_prefix_cache_kv_indices(
    req_to_token_ptr,  # (max_batch, max_context_len,)
    req_pool_indices_ptr,  # (batch_size,)
    chunk_start_idx_ptr,  # (batch_size,)
    chunk_seq_lens_ptr,  # (batch_size,)
    chunk_cu_seq_lens_ptr,  # (batch_size + 1,)
    chunk_kv_indices_ptr,  # (num_chunk_tokens,)
    req_to_token_ptr_stride: tl.constexpr,
):
    BLOCK_SIZE: tl.constexpr = 512
    pid = tl.program_id(axis=0)

    # find the req pool idx, this is for batch to token
    req_pool_index = tl.load(req_pool_indices_ptr + pid)
    chunk_kv_indices_offset = tl.load(chunk_cu_seq_lens_ptr + pid)

    # get the token positions of current chunk
    chunk_start_pos = tl.load(chunk_start_idx_ptr + pid).to(tl.int32)
    chunk_seq_len = tl.load(chunk_seq_lens_ptr + pid).to(tl.int32)

    num_loop = tl.cdiv(chunk_seq_len, BLOCK_SIZE)
    for i in range(num_loop):
        offset = tl.arange(0, BLOCK_SIZE) + i * BLOCK_SIZE
        mask = offset < chunk_seq_len
        data = tl.load(
            req_to_token_ptr
            + req_pool_index * req_to_token_ptr_stride
            + chunk_start_pos
            + offset,
            mask=mask,
        )
        tl.store(
            chunk_kv_indices_ptr + chunk_kv_indices_offset + offset, data, mask=mask
        )
```
**EN:** This callable implements `create_chunked_prefix_cache_kv_indices`. It takes `req_to_token_ptr`, `req_pool_indices_ptr`, `chunk_start_idx_ptr`, `chunk_seq_lens_ptr` and mainly constructs new objects or contexts.
**CN:** 这一可调用对象实现了 `create_chunked_prefix_cache_kv_indices`。它接收 `req_to_token_ptr`, `req_pool_indices_ptr`, `chunk_start_idx_ptr`, `chunk_seq_lens_ptr`，主要用于构造新的对象或上下文。

## Key Concepts / 关键概念
- `ForwardBatchDeepSeekMHAMixin`: core class or state container / 核心类或状态容器
- `create_chunked_prefix_cache_kv_indices`: constructs new objects or contexts / 构造新的对象或上下文

## Dependencies / 依赖关系
- **Standard library / 标准库**: `typing`
- **Third-party / 第三方**: `torch`, `triton`, `triton.language`
- **Internal modules / 内部模块**: `sglang.srt.environ`, `sglang.srt.layers.attention.utils`, `sglang.srt.mem_cache.memory_pool`
