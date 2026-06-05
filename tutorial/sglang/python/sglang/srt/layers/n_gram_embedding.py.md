# n_gram_embedding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/n_gram_embedding.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements embedding lookup and preprocessing for the SGLang SRT runtime. It exposes symbols such as `NgramEmbedding` and connects them to backend-specific paths such as `CUDA` and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了嵌入查找与预处理。它提供了 `NgramEmbedding` 等符号，并把这些符号连接到 `CUDA` 和 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Module imports and dependency wiring
```python
import torch
from torch import nn
from torch.nn import Parameter

from sglang.jit_kernel.ngram_embedding import compute_n_gram_ids
from sglang.srt.layers.dp_attention import is_dp_attention_enabled
from sglang.srt.layers.vocab_parallel_embedding import VocabParallelEmbedding
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
```
**EN:** This section prepares the module namespace. It imports `torch`, `torch.nn`, `torch.nn.Parameter`, `sglang.jit_kernel.ngram_embedding.compute_n_gram_ids`, `sglang.srt.layers.dp_attention.is_dp_attention_enabled`, and `sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding`, so later blocks can reuse runtime, tensor, or backend helpers.
**CN:** 该部分负责准备模块命名空间。 它导入了 `torch`、`torch.nn`、`torch.nn.Parameter`、`sglang.jit_kernel.ngram_embedding.compute_n_gram_ids`、`sglang.srt.layers.dp_attention.is_dp_attention_enabled` 以及 `sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding`，让后续代码可以复用运行时、张量或后端辅助逻辑。

### Lines 11-12: Class `NgramEmbedding` declaration and shared state
```python
class NgramEmbedding(torch.nn.Module):
```
**EN:** This block introduces class `NgramEmbedding` and the state shared by its methods. It inherits from `torch.nn.Module`, which shows how it plugs into the SRT layer stack.
**CN:** 该代码块引入类 `NgramEmbedding`，并定义其方法共享的状态。 它继承自 `torch.nn.Module`，说明了它在 SRT 层栈中的接入方式。

### Lines 13-78: `NgramEmbedding` initialization and state setup
```python
    def __init__(
        self,
        num_embeddings: int,
        embedding_dim: int,
        over_embedding_m: int,
        over_embedding_k: int,
        over_embedding_n: int,
    ):
        super().__init__()
        assert (
            over_embedding_n > 1
        ), f"over_embedding_n must be > 1, got {over_embedding_n}"
        self.num_embeddings = num_embeddings
        self.embedding_dim = embedding_dim
        self.over_embedding_m = over_embedding_m
        self.over_embedding_k = over_embedding_k
        self.over_embedding_n = over_embedding_n

        self.word_embeder = VocabParallelEmbedding(
            num_embeddings,
            embedding_dim,
            enable_tp=is_dp_attention_enabled(),
        )
        self.n_grams = (over_embedding_n - 1) * over_embedding_k
        oe_hidden_dim = embedding_dim // (over_embedding_k * (over_embedding_n - 1))
        self.exclusive_oe_embedder_size_sums = torch.zeros(
            [over_embedding_k * (over_embedding_n - 1) + 1],
            dtype=torch.int32,
            device="cuda",
        )
        for i in range(over_embedding_k * (over_embedding_n - 1)):
            self.exclusive_oe_embedder_size_sums[i + 1] = (
                self.exclusive_oe_embedder_size_sums[i]
                + int(over_embedding_m + i * 2 + 1)
            )
        self.oe_embeder = VocabParallelEmbedding(
            num_embeddings=self.exclusive_oe_embedder_size_sums[-1],
            embedding_dim=oe_hidden_dim,
            enable_tp=is_dp_attention_enabled(),
        )

        self.oe_projection = nn.Parameter(
            torch.empty(
                (over_embedding_n - 1) * over_embedding_k, oe_hidden_dim, embedding_dim
            ),
            requires_grad=False,
        )

        self.oe_mods = torch.zeros(
            [self.over_embedding_n - 1, self.over_embedding_k], dtype=torch.int32
        )
        self.oe_weights = torch.zeros(
            [self.over_embedding_n - 1, self.over_embedding_k, self.over_embedding_n],
            dtype=torch.int32,
        )
        for n in range(2, self.over_embedding_n + 1):
            for k in range(self.over_embedding_k):
                mod = (
                    self.over_embedding_m
                    + 2 * ((n - 2) * self.over_embedding_k + k)
                    + 1
                )
                self.oe_mods[n - 2][k] = mod
                for delta in range(self.over_embedding_n):
                    self.oe_weights[n - 2][k][delta] = pow(num_embeddings, delta, mod)
```
**EN:** This block defines `NgramEmbedding.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `VocabParallelEmbedding`, `torch.zeros`, `range`, and `nn.Parameter`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.num_embeddings`, `self.embedding_dim`, `self.over_embedding_m`, `self.over_embedding_k`, and `self.over_embedding_n` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `NgramEmbedding.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`VocabParallelEmbedding`、`torch.zeros`、`range` 以及 `nn.Parameter`，说明该流程会编排底层辅助函数或计算内核。 像 `self.num_embeddings`、`self.embedding_dim`、`self.over_embedding_m`、`self.over_embedding_k` 以及 `self.over_embedding_n` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 79-91: Function `NgramEmbedding.init_buffers` and its core logic
```python
    def init_buffers(
        self, max_running_requests: int, chunked_prefill_size: int, device: str
    ):
        max_tokens = max(chunked_prefill_size, max_running_requests)
        self.oe_n_gram_ids = torch.zeros(
            [max_tokens, self.n_grams],
            dtype=torch.int32,
            device=device,
        )
        self.exclusive_req_len_sums = torch.zeros(
            max_running_requests + 1, dtype=torch.int32, device=device
        )
```
**EN:** This block defines `NgramEmbedding.init_buffers` and contains the main logic for this step. It mainly invokes `max` and `torch.zeros`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `max_tokens`, `self.oe_n_gram_ids`, and `self.exclusive_req_len_sums` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `NgramEmbedding.init_buffers`，并承载这一阶段的核心逻辑。 它主要调用 `max` 和 `torch.zeros`，说明该流程会编排底层辅助函数或计算内核。 像 `max_tokens`、`self.oe_n_gram_ids` 以及 `self.exclusive_req_len_sums` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 92-131: `NgramEmbedding.load_weight` loader for weight
```python
    def load_weight(
        self, param: Parameter, weight_name: str, loaded_weight: torch.Tensor
    ):
        if ".embed_tokens." in weight_name:
            param.weight_loader(param, loaded_weight)
        elif "model.ngram_embeddings.embedders." in weight_name:
            index = int(
                weight_name.replace("model.ngram_embeddings.embedders.", "").replace(
                    ".weight", ""
                )
            )
            oe_weight_start = self.exclusive_oe_embedder_size_sums[index]
            oe_weight_end = self.exclusive_oe_embedder_size_sums[index + 1]
            assert (
                oe_weight_end - oe_weight_start == loaded_weight.shape[0]
            ), f"{oe_weight_end - oe_weight_start=} {loaded_weight.shape[0]=}"
            tp_start = self.oe_embeder.shard_indices.org_vocab_start_index
            tp_end = self.oe_embeder.shard_indices.org_vocab_end_index
            to_load_start = max(oe_weight_start, tp_start)
            to_load_end = min(oe_weight_end, tp_end)
            if to_load_start < to_load_end:
                src_start = to_load_start - oe_weight_start
                src_end = to_load_end - oe_weight_start
                dest_start = to_load_start - tp_start
                dest_end = to_load_end - tp_start
                self.oe_embeder.weight.data[dest_start:dest_end] = loaded_weight[
                    src_start:src_end
                ]
            else:
                return
        elif "model.ngram_embeddings.post_projs." in weight_name:
            index = int(
                weight_name.replace("model.ngram_embeddings.post_projs.", "").replace(
                    ".weight", ""
                )
            )
            self.oe_projection[index].copy_(loaded_weight.data.t())
        else:
            assert False, f"Unknown ngram embedding weight name: {weight_name}"
```
**EN:** This block defines `NgramEmbedding.load_weight` and contains the main logic for this step. It mainly invokes `param.weight_loader`, `int`, `max`, `min`, and `weight_name.replace.replace`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `index`, `oe_weight_start`, `oe_weight_end`, `tp_start`, and `tp_end` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `NgramEmbedding.load_weight`，并承载这一阶段的核心逻辑。 它主要调用 `param.weight_loader`、`int`、`max`、`min` 以及 `weight_name.replace.replace`，说明该流程会编排底层辅助函数或计算内核。 像 `index`、`oe_weight_start`、`oe_weight_end`、`tp_start` 以及 `tp_end` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 132-172: `NgramEmbedding.forward` main forward path
```python
    def forward(self, input_ids: torch.Tensor, forward_batch: ForwardBatch):
        if (
            forward_batch.forward_mode.is_extend()
            or forward_batch.forward_mode.is_decode()
        ):
            ngram_embedding_info = forward_batch.ngram_embedding_info
            torch.cumsum(
                ngram_embedding_info.req_lens,
                dim=0,
                dtype=torch.int32,
                out=self.exclusive_req_len_sums[1 : 1 + forward_batch.batch_size],
            )
            compute_n_gram_ids(
                ne_n=self.over_embedding_n,
                ne_k=self.over_embedding_k,
                ne_weights=self.oe_weights,
                ne_mods=self.oe_mods,
                tokens=input_ids.to(torch.int32),
                exclusive_ne_embedder_size_sums=self.exclusive_oe_embedder_size_sums,
                exclusive_req_len_sums=self.exclusive_req_len_sums[
                    : forward_batch.batch_size + 1
                ],
                ne_token_table=ngram_embedding_info.token_table,
                row_indices=forward_batch.req_pool_indices,
                column_starts=ngram_embedding_info.column_starts,
                n_gram_ids=self.oe_n_gram_ids[: len(input_ids)],
            )

        # [13, seq_len, hidden_dim]
        all_hidden_states = torch.empty(
            [self.n_grams + 1, len(input_ids), self.embedding_dim],
            dtype=self.oe_projection.dtype,
            device=input_ids.device,
        )
        all_hidden_states[0] = self.word_embeder(input_ids)
        # oe_hidden_states: [12, seq_len, hidden_dim / 12]
        oe_hidden_states = self.oe_embeder(
            self.oe_n_gram_ids[: len(input_ids)].permute(1, 0).contiguous()
        )
        torch.bmm(oe_hidden_states, self.oe_projection, out=all_hidden_states[1:])
        return all_hidden_states.mean(dim=0)
```
**EN:** This block defines `NgramEmbedding.forward` and contains the main logic for this step. It mainly invokes `torch.empty`, `self.word_embeder`, `self.oe_embeder`, `torch.bmm`, and `all_hidden_states.mean`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `all_hidden_states`, `oe_hidden_states`, and `ngram_embedding_info` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `NgramEmbedding.forward`，并承载这一阶段的核心逻辑。 它主要调用 `torch.empty`、`self.word_embeder`、`self.oe_embeder`、`torch.bmm` 以及 `all_hidden_states.mean`，说明该流程会编排底层辅助函数或计算内核。 像 `all_hidden_states`、`oe_hidden_states` 以及 `ngram_embedding_info` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `NgramEmbedding`. / **主要符号**：核心入口包括 `NgramEmbedding`。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Embedding pipeline**: Describes how IDs or features are mapped into model-space tensors. / **嵌入流水线**：说明 ID 或特征如何映射到模型空间张量。

## Dependencies / 依赖关系
- **Third-party**: `torch`, `torch.nn`, and `torch.nn.Parameter` / **第三方依赖**：`torch`、`torch.nn` 以及 `torch.nn.Parameter`
- **Internal SGLang modules**: `sglang.jit_kernel.ngram_embedding.compute_n_gram_ids`, `sglang.srt.layers.dp_attention.is_dp_attention_enabled`, `sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding`, and `sglang.srt.model_executor.forward_batch_info.ForwardBatch` / **SGLang 内部模块**：`sglang.jit_kernel.ngram_embedding.compute_n_gram_ids`、`sglang.srt.layers.dp_attention.is_dp_attention_enabled`、`sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding` 以及 `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
