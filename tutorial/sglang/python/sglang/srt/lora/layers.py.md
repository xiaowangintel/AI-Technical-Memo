# layers.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/lora/layers.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements core LoRA runtime logic such as adapter metadata, memory management, layer wrapping, or orchestration. It is part of SGLang's infrastructure for serving models with dynamic adapters. / 该文件实现了核心 LoRA 运行时逻辑，例如适配器元数据、内存管理、层封装或整体编排。它属于 SGLang 支持动态适配器服务能力的基础设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-29: Module header, imports, and shared constants
```python
from typing import Dict, Optional, Union

import torch
import torch.nn.functional as F
from torch import nn

from sglang.srt.distributed import (
    get_tensor_model_parallel_rank,
    split_tensor_along_last_dim,
    tensor_model_parallel_all_gather,
    tensor_model_parallel_all_reduce,
)
from sglang.srt.layers.linear import (
    ColumnParallelLinear,
    MergedColumnParallelLinear,
    QKVParallelLinear,
    ReplicatedLinear,
    RowParallelLinear,
)
from sglang.srt.layers.moe.fused_moe_triton.layer import FusedMoE
from sglang.srt.layers.moe.topk import TopKOutput
from sglang.srt.layers.vocab_parallel_embedding import (
    ParallelLMHead,
    VocabParallelEmbedding,
)
from sglang.srt.lora.backend.base_backend import BaseLoRABackend
from sglang.srt.lora.utils import LoRABatchInfo, get_lm_head_lora_b_shard_size
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 30-30: BaseLayerWithLoRA class declaration
```python
class BaseLayerWithLoRA(nn.Module):
```
**EN:** This block declares the `BaseLayerWithLoRA` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `BaseLayerWithLoRA` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 31-43: BaseLayerWithLoRA initializer
```python
    def __init__(
        self,
        base_layer: nn.Module,
        lora_backend: BaseLoRABackend,
    ):
        super().__init__()
        self.base_layer: nn.Module = base_layer
        self.set_lora: bool = False
        self.lora_backend: BaseLoRABackend = lora_backend
        if hasattr(self.base_layer, "weight"):
            self.weight = self.base_layer.weight
        if hasattr(self.base_layer, "bias") and self.base_layer.bias is not None:
            self.bias = self.base_layer.bias
```
**EN:** This block initializes the `BaseLayerWithLoRA` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `BaseLayerWithLoRA` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 45-46: BaseLayerWithLoRA.forward method
```python
    def forward(self, x: torch.Tensor):
        return self.base_layer.forward(x)
```
**EN:** This block uses `BaseLayerWithLoRA.forward` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `BaseLayerWithLoRA.forward` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 48-49: BaseLayerWithLoRA.set_lora_info method
```python
    def set_lora_info(self, *args):
        pass
```
**EN:** This block uses `BaseLayerWithLoRA.set_lora_info` to update stored state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `BaseLayerWithLoRA.set_lora_info` 来更新已保存的状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 51-52: BaseLayerWithLoRA.slice_lora_a_weights method
```python
    def slice_lora_a_weights(self, A: torch.Tensor, tp_rank: int):
        pass
```
**EN:** This block uses `BaseLayerWithLoRA.slice_lora_a_weights` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `BaseLayerWithLoRA.slice_lora_a_weights` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 54-55: BaseLayerWithLoRA.slice_lora_b_weights method
```python
    def slice_lora_b_weights(self, B: torch.Tensor, tp_rank: int):
        pass
```
**EN:** This block uses `BaseLayerWithLoRA.slice_lora_b_weights` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `BaseLayerWithLoRA.slice_lora_b_weights` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 58-65: VocabParallelEmbeddingWithLoRA class declaration
```python
class VocabParallelEmbeddingWithLoRA(BaseLayerWithLoRA):
    """
    Vocab parallel embedding layer with LoRA support (simplified for TP=1, no extra tokens).

    For embedding layers: output = base_embedding(x) + lora_B @ lora_A[x]
    where lora_A[x] is direct embedding lookup from lora_A weights.
    """
```
**EN:** This block declares the `VocabParallelEmbeddingWithLoRA` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `VocabParallelEmbeddingWithLoRA` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 66-104: VocabParallelEmbeddingWithLoRA initializer
```python
    def __init__(
        self,
        base_layer: VocabParallelEmbedding,
        lora_backend: BaseLoRABackend,
    ) -> None:
        super().__init__(base_layer, lora_backend)
        self.weight = base_layer.weight
        self.embed_dim = base_layer.embedding_dim
        self.vocab_size = base_layer.org_vocab_size
        self.num_embeddings = base_layer.num_embeddings

        # Embedding LoRA with TP > 1 keeps weights fully replicated
        # (unsharded) on every rank.  This works correctly because the
        # base VocabParallelEmbedding all-reduces its output before the
        # LoRA delta is added, but it means each rank holds the full
        # LoRA A (rank, vocab_size) and LoRA B (embed_dim, rank) tensors,
        # which may cause OOM on large vocabularies or high LoRA ranks.
        #
        # input_scattered mode (DeepSeek-v2 MLA) skips the base
        # all-reduce, making the unsharded LoRA approach mathematically
        # incorrect — a sharded LoRA kernel would be needed.
        if hasattr(base_layer, "tp_size") and base_layer.tp_size > 1:
            from sglang.srt.layers.communicator import get_attn_tp_context

            assert (
                not get_attn_tp_context().allow_input_scattered
            ), "VocabParallelEmbeddingWithLoRA with TP > 1 under input_scattered mode (e.g., DeepSeek-v2 MLA with --enable-attn-tp-input-scattered) is not fully supported and may produce incorrect results. Consider disabling input_scattered or removing embed_tokens from LoRA target modules."
        offsets = [0, self.embed_dim]
        self.output_offset = torch.tensor(
            offsets,
            dtype=torch.int32,
            device=next(base_layer.parameters()).device,
        )
        self.output_offset_cpu = torch.tensor(
            offsets,
            dtype=torch.int32,
            device="cpu",
            pin_memory=True,
        )
```
**EN:** This block initializes the `VocabParallelEmbeddingWithLoRA` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `VocabParallelEmbeddingWithLoRA` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 106-116: VocabParallelEmbeddingWithLoRA.set_lora_info method
```python
    def set_lora_info(
        self,
        new_embeddings_buffer: Optional[torch.Tensor],  # For extra tokens
        embedding_A_buffer: torch.Tensor,
        embedding_B_buffer: torch.Tensor,
    ):
        """Set LoRA buffers for embedding layer."""
        self.set_lora = True
        self.new_embeddings_buffer = new_embeddings_buffer
        self.embedding_A_buffer = embedding_A_buffer  # (num_loras, rank, vocab_size)
        self.embedding_B_buffer = embedding_B_buffer  # (num_loras, embed_dim, rank)
```
**EN:** This block uses `VocabParallelEmbeddingWithLoRA.set_lora_info` to update stored state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `VocabParallelEmbeddingWithLoRA.set_lora_info` 来更新已保存的状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 118-137: VocabParallelEmbeddingWithLoRA.apply_lora method
```python
    def apply_lora(
        self, base_output: torch.Tensor, input_: torch.Tensor, batch_info
    ) -> torch.Tensor:
        """
        Apply LoRA to base embedding output.
        Formula: output = base_output + lora_B @ lora_A_embedding(input_)
        """

        # Efficient embedding lookup for LoRA A (already support extra token embedding process)
        lora_a_output = self.run_lora_a_embedding(input_, batch_info)

        # Apply LoRA B weights using backend
        lora_output = self.lora_backend.run_lora_b_sgemm(
            x=lora_a_output,
            weights=self.embedding_B_buffer,
            output_offset=self.output_offset,
            output_offset_cpu=self.output_offset_cpu,
            base_output=base_output,
        )
        return lora_output
```
**EN:** This block uses `VocabParallelEmbeddingWithLoRA.apply_lora` to apply a transformation to tensors or state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `VocabParallelEmbeddingWithLoRA.apply_lora` 来对张量或状态应用变换。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 139-160: VocabParallelEmbeddingWithLoRA.run_lora_a_embedding method
```python
    def run_lora_a_embedding(
        self, input_: torch.Tensor, batch_info: LoRABatchInfo
    ) -> torch.Tensor:
        """
        Apply LoRA A weights using efficient embedding lookup with CUDA graph support.
        Maps tokens to their corresponding LoRA adapters internally.
        It also includes added/extra token processing.
        """
        # Efficient embedding lookup for LoRA A (already support extra token embedding process)
        lora_a_output = self.lora_backend.run_lora_a_embedding(
            input_ids=input_,
            weights=self.embedding_A_buffer,
            vocab_size=self.vocab_size,
            extra_embeddings=(
                self.new_embeddings_buffer
                if hasattr(self, "new_embeddings_buffer")
                and self.new_embeddings_buffer is not None
                else None
            ),
        )

        return lora_a_output
```
**EN:** This block uses `VocabParallelEmbeddingWithLoRA.run_lora_a_embedding` to execute the main compute path. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `VocabParallelEmbeddingWithLoRA.run_lora_a_embedding` 来执行主要计算路径。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 162-185: VocabParallelEmbeddingWithLoRA.extra_token_embedding method
```python
    def extra_token_embedding(
        self, input_: torch.Tensor, base_output: torch.Tensor
    ) -> torch.Tensor:
        """
        Need to impl:

        Process extra tokens (tokens >= vocab_size) by looking up their embeddings
        from the new_embeddings_buffer and replacing them in base_output.

        Args:
            input_: (s,) token IDs
            base_output: (s, embed_dim) base embedding output to be modified in-place

        Returns:
            base_output: (s, embed_dim) modified input base_output (tensor[0,0,0,...]) with extra token embeddings
        """
        # return base_output
        raise NotImplementedError(
            "Error in sglang/python/sglang/srt/lora/layers.py - VocabParallelEmbeddingWithLoRA \n"
            "Current SGLang codebase did not support tuned lora with extra/added tokens. \n"
            "[TODO]: \n"
            "1. Refer to this commit: https://github.com/yushengsu-thu/sglang/commit/90415211eee8a28a316de262583d4d33fa615d10#diff-191177438bcc223837963de63c005850371f8c8a860acb153b26744b66ecc623 to complete \n"
            "2. And then you need to modified the en/decoder tokenizer - tokenizer_manager.py to support extra_token_embedding in-place. \n"
        )
```
**EN:** This block uses `VocabParallelEmbeddingWithLoRA.extra_token_embedding` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `VocabParallelEmbeddingWithLoRA.extra_token_embedding` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 187-216: VocabParallelEmbeddingWithLoRA.forward method
```python
    def forward(self, input_: torch.Tensor):
        """
        Forward pass with LoRA support and CUDA graph compatibility.

        Extra tokens (tokens >= vocab_size) are now handled efficiently
        in the backend's run_lora_a_embedding method.
        """
        batch_info = self.lora_backend.batch_info

        # Get base embedding output
        # For tokens >= vocab_size, base_layer will clamp or handle them
        # We mask them to 0 to avoid out-of-bounds access
        added_tokens_mask = input_ > self.vocab_size - 1
        base_output = self.base_layer.forward(input_.masked_fill(added_tokens_mask, 0))

        # [TODO] SGLang did not support extra/added token process; thus, self.extra_token_embedding only return original input_ now
        # Extra tokens - It will replace extra token embedding with self.new_embeddings_buffer's emb (Default is 0)
        if (
            hasattr(self, "new_embeddings_buffer")
            and self.new_embeddings_buffer is not None
        ):
            base_output = self.extra_token_embedding(input_, base_output)

        # Apply LoRA if configured
        if self.set_lora:
            # The backend's run_lora_a_embedding now handles both regular
            # and extra tokens efficiently with CUDA graph support
            base_output = self.apply_lora(base_output, input_, batch_info)

        return base_output
```
**EN:** This block uses `VocabParallelEmbeddingWithLoRA.forward` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `VocabParallelEmbeddingWithLoRA.forward` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 218-222: VocabParallelEmbeddingWithLoRA.slice_lora_a_weights method
```python
    def slice_lora_a_weights(self, A: torch.Tensor, tp_rank: int):
        # LoRA A weights (rank, vocab_size) are kept unsharded.
        # Each rank does a full embedding lookup; the result is complete
        # on every rank and added to the already all-reduced base output.
        return A
```
**EN:** This block uses `VocabParallelEmbeddingWithLoRA.slice_lora_a_weights` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `VocabParallelEmbeddingWithLoRA.slice_lora_a_weights` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 224-228: VocabParallelEmbeddingWithLoRA.slice_lora_b_weights method
```python
    def slice_lora_b_weights(self, B: torch.Tensor, tp_rank: int):
        # LoRA B weights (embedding_dim, rank) are kept unsharded.
        # The base embedding output is all-reduced (full embedding_dim),
        # so LoRA B must also produce full embedding_dim.
        return B
```
**EN:** This block uses `VocabParallelEmbeddingWithLoRA.slice_lora_b_weights` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `VocabParallelEmbeddingWithLoRA.slice_lora_b_weights` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 231-242: ParallelLMHeadWithLoRA class declaration
```python
class ParallelLMHeadWithLoRA(BaseLayerWithLoRA):
    """
    Parallel LM Head layer with LoRA support.

    The LM head computes logits = hidden_states @ (W + B @ A)^T

    With TP > 1, lm_head is column-parallel: each rank holds
    weight (vocab_size/tp_size, hidden_size) and produces a shard
    of logits.  LoRA A is kept unsharded (rank, hidden_size) while
    LoRA B is sliced along the vocab dimension to (vocab_size/tp_size, rank).
    """
```
**EN:** This block declares the `ParallelLMHeadWithLoRA` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `ParallelLMHeadWithLoRA` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 243-289: ParallelLMHeadWithLoRA initializer
```python
    def __init__(
        self,
        base_layer: ParallelLMHead,
        lora_backend: BaseLoRABackend,
    ) -> None:
        super().__init__(base_layer, lora_backend)
        self.weight = base_layer.weight
        self.embed_dim = base_layer.embedding_dim
        self.vocab_size = base_layer.org_vocab_size

        offsets = [0, self.vocab_size]

        tp_size = base_layer.tp_size if hasattr(base_layer, "tp_size") else 1

        # lm_head LoRA keeps A unsharded and shards B along the vocab
        # dimension, matching the column-parallel base output.  This is
        # incompatible with input_scattered mode where the all-reduce is
        # skipped.
        if tp_size > 1:
            from sglang.srt.layers.communicator import get_attn_tp_context

            if get_attn_tp_context().allow_input_scattered:
                raise ValueError(
                    "ParallelLMHeadWithLoRA is not compatible with "
                    "input_scattered mode (e.g., DeepSeek-v2 MLA with "
                    "--enable-attn-tp-input-scattered). Please disable "
                    "input_scattered or remove lm_head from LoRA "
                    "target modules."
                )

            self.shard_vocab_size = get_lm_head_lora_b_shard_size(
                self.vocab_size,
                shard_indices=base_layer.shard_indices,
            )
            offsets = [0, self.shard_vocab_size]

        self.output_offset = torch.tensor(
            offsets,
            dtype=torch.int32,
            device=next(base_layer.parameters()).device,
        )
        self.output_offset_cpu = torch.tensor(
            offsets,
            dtype=torch.int32,
            device="cpu",
            pin_memory=True,
        )
```
**EN:** This block initializes the `ParallelLMHeadWithLoRA` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `ParallelLMHeadWithLoRA` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 291-299: ParallelLMHeadWithLoRA.set_lora_info method
```python
    def set_lora_info(
        self,
        lm_head_A_buffer: torch.Tensor,
        lm_head_B_buffer: torch.Tensor,
    ):
        """Set LoRA buffers for LM head layer."""
        self.set_lora = True
        self.lm_head_A_buffer = lm_head_A_buffer  # (num_loras, rank, hidden_dim)
        self.lm_head_B_buffer = lm_head_B_buffer  # (num_loras, vocab_size, rank)
```
**EN:** This block uses `ParallelLMHeadWithLoRA.set_lora_info` to update stored state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ParallelLMHeadWithLoRA.set_lora_info` 来更新已保存的状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 301-335: ParallelLMHeadWithLoRA._get_lm_head_batch_info method
```python
    def _get_lm_head_batch_info(self, num_tokens: int):
        """Resolve and validate the active lm_head batch_info.

        When the logits processor calls lm_head in multiple passes
        (chunked logprobs), _lm_head_pass_idx selects a precomputed
        per-pass batch_info.  Otherwise the full-pruned batch_info is used.

        Returns None when no lm_head pruning applies (decode, no LoRA, etc.).
        """
        pass_idx = self.lora_backend._lm_head_pass_idx
        if (
            pass_idx is not None
            and self.lora_backend.lm_head_pass_batch_infos is not None
        ):
            batch_info = self.lora_backend.lm_head_pass_batch_infos[pass_idx]
        else:
            batch_info = self.lora_backend.lm_head_batch_info

        if batch_info is not None:
            if batch_info.use_cuda_graph:
                raise RuntimeError(
                    "lm_head LoRA with pruned batch info is not supported "
                    "under CUDA graph. lm_head pruning should only occur "
                    "during extend, which does not use CUDA graph."
                )
            if num_tokens != batch_info.expected_tokens:
                raise RuntimeError(
                    f"lm_head LoRA input token count mismatch: got "
                    f"{num_tokens} tokens but lm_head_batch_info expects "
                    f"{batch_info.expected_tokens}. This likely means "
                    f"a pruning step in LogitsProcessor._get_pruned_states is "
                    f"not reflected in get_lm_head_pruned_lens()."
                )

        return batch_info
```
**EN:** This block uses `ParallelLMHeadWithLoRA._get_lm_head_batch_info` to fetch or compute a value. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ParallelLMHeadWithLoRA._get_lm_head_batch_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 337-368: ParallelLMHeadWithLoRA.apply_lora method
```python
    def apply_lora(
        self,
        base_output: torch.Tensor,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        """
        Apply LoRA to LM head layer.

        For LM head: output = hidden @ (W + B @ A)^T
                           = hidden @ W^T + hidden @ A^T @ B^T
                           = base_output + (hidden @ A^T) @ B^T
        """
        lm_head_batch_info = self._get_lm_head_batch_info(hidden_states.shape[0])

        # Apply lora_A^T: hidden_states @ A^T
        lora_a_output = self.lora_backend.run_lora_a_sgemm(
            hidden_states,
            self.lm_head_A_buffer,
            pruned_batch_info=lm_head_batch_info,
        )

        # Apply lora_B^T: lora_a_output @ B^T
        lora_output = self.lora_backend.run_lora_b_sgemm(
            x=lora_a_output,
            weights=self.lm_head_B_buffer,
            output_offset=self.output_offset,
            output_offset_cpu=self.output_offset_cpu,
            base_output=base_output,
            pruned_batch_info=lm_head_batch_info,
        )

        return lora_output
```
**EN:** This block uses `ParallelLMHeadWithLoRA.apply_lora` to apply a transformation to tensors or state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ParallelLMHeadWithLoRA.apply_lora` 来对张量或状态应用变换。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 370-380: ParallelLMHeadWithLoRA.forward method
```python
    def forward(self, hidden_states: torch.Tensor):
        # Apply base linear transformation
        base_output = F.linear(
            hidden_states, self.weight, bias=getattr(self.base_layer, "bias", None)
        )

        # Apply LoRA if set
        if self.set_lora:
            base_output = self.apply_lora(base_output, hidden_states)

        return base_output
```
**EN:** This block uses `ParallelLMHeadWithLoRA.forward` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ParallelLMHeadWithLoRA.forward` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 381-385: Class-level supporting statements
```python

    # ------------------------------------------------------------------
    # Multi-pass lm_head support (chunked logprobs)
    # ------------------------------------------------------------------
```
**EN:** This block contains supporting statements for the `ParallelLMHeadWithLoRA` class, such as constants, attributes, registrations, or small helper expressions that shape the surrounding control flow.
**CN:** 该代码块包含`ParallelLMHeadWithLoRA` 类的配套语句，例如常量、属性、注册逻辑或小型辅助表达式，它们共同塑造了周边控制流程。

### Lines 386-393: ParallelLMHeadWithLoRA.set_lm_head_pass method
```python
    def set_lm_head_pass(self, pass_idx: int):
        """Set the active lm_head pass index before a logprobs chunk.

        Called by LogitsProcessor.process_input_logprobs_by_chunk() before
        each chunk's _get_logits call.  _get_lm_head_batch_info() will
        resolve to lm_head_pass_batch_infos[pass_idx].
        """
        self.lora_backend._lm_head_pass_idx = pass_idx
```
**EN:** This block uses `ParallelLMHeadWithLoRA.set_lm_head_pass` to update stored state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ParallelLMHeadWithLoRA.set_lm_head_pass` 来更新已保存的状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 395-397: ParallelLMHeadWithLoRA.reset_lm_head_pass method
```python
    def reset_lm_head_pass(self):
        """Reset the lm_head pass index after all passes are done."""
        self.lora_backend._lm_head_pass_idx = None
```
**EN:** This block uses `ParallelLMHeadWithLoRA.reset_lm_head_pass` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ParallelLMHeadWithLoRA.reset_lm_head_pass` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 399-402: ParallelLMHeadWithLoRA.slice_lora_a_weights method
```python
    def slice_lora_a_weights(self, A: torch.Tensor, tp_rank: int):
        # LoRA A weights (rank, hidden_size) are kept unsharded.
        # Each rank receives full hidden_states, so A operates on full input.
        return A
```
**EN:** This block uses `ParallelLMHeadWithLoRA.slice_lora_a_weights` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ParallelLMHeadWithLoRA.slice_lora_a_weights` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 404-415: ParallelLMHeadWithLoRA.slice_lora_b_weights method
```python
    def slice_lora_b_weights(self, B: torch.Tensor, tp_rank: int):
        # lm_head is column-parallel: each rank produces vocab_size/tp_size (shard_vocab_size)
        # logits.  LoRA B (vocab_size, rank) must be sliced along the vocab
        # dimension to match the sharded base output.
        # Uses the base layer's shard_indices for the actual vocab range on
        # this rank, staying consistent with base model weight sharding.
        tp_size = self.base_layer.tp_size if hasattr(self.base_layer, "tp_size") else 1
        if tp_size <= 1:
            return B
        start_idx = self.base_layer.shard_indices.org_vocab_start_index
        end_idx = self.base_layer.shard_indices.org_vocab_end_index
        return B[start_idx:end_idx, :]
```
**EN:** This block uses `ParallelLMHeadWithLoRA.slice_lora_b_weights` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ParallelLMHeadWithLoRA.slice_lora_b_weights` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 418-418: ColumnParallelLinearWithLoRA class declaration
```python
class ColumnParallelLinearWithLoRA(BaseLayerWithLoRA):
```
**EN:** This block declares the `ColumnParallelLinearWithLoRA` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `ColumnParallelLinearWithLoRA` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 419-437: ColumnParallelLinearWithLoRA initializer
```python
    def __init__(
        self,
        base_layer: ColumnParallelLinear,
        lora_backend: BaseLoRABackend,
    ) -> None:
        super().__init__(base_layer, lora_backend)
        shard_size = self.base_layer.output_partition_sizes[0]
        offsets = [0, shard_size]
        self.output_offset = torch.tensor(
            offsets,
            dtype=torch.int32,
            device=next(self.base_layer.parameters()).device,
        )
        self.output_offset_cpu = torch.tensor(
            offsets,
            dtype=torch.int32,
            device="cpu",
            pin_memory=True,
        )
```
**EN:** This block initializes the `ColumnParallelLinearWithLoRA` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `ColumnParallelLinearWithLoRA` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 439-446: ColumnParallelLinearWithLoRA.set_lora_info method
```python
    def set_lora_info(
        self,
        A_buffer: torch.Tensor,
        B_buffer: torch.Tensor,
    ):
        self.set_lora = True
        self.A_buffer = A_buffer
        self.B_buffer = B_buffer
```
**EN:** This block uses `ColumnParallelLinearWithLoRA.set_lora_info` to update stored state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ColumnParallelLinearWithLoRA.set_lora_info` 来更新已保存的状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 448-457: ColumnParallelLinearWithLoRA.apply_lora method
```python
    def apply_lora(self, base_output: torch.Tensor, x: torch.Tensor) -> torch.Tensor:
        lora_a_output = self.lora_backend.run_lora_a_sgemm(x, self.A_buffer)
        lora_output = self.lora_backend.run_lora_b_sgemm(
            x=lora_a_output,
            weights=self.B_buffer,
            output_offset=self.output_offset,
            output_offset_cpu=self.output_offset_cpu,
            base_output=base_output,
        )
        return lora_output
```
**EN:** This block uses `ColumnParallelLinearWithLoRA.apply_lora` to apply a transformation to tensors or state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ColumnParallelLinearWithLoRA.apply_lora` 来对张量或状态应用变换。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 459-474: ColumnParallelLinearWithLoRA.forward method
```python
    def forward(self, input_: torch.Tensor):
        # duplicate the logic in ColumnParallelLinear
        bias = self.base_layer.bias if not self.base_layer.skip_bias_add else None
        output_parallel = self.base_layer.quant_method.apply(
            self.base_layer, input_, bias
        )

        if self.set_lora:
            output_parallel = self.apply_lora(output_parallel, input_)

        if self.base_layer.gather_output:
            output = tensor_model_parallel_all_gather(output_parallel)
        else:
            output = output_parallel
        output_bias = self.base_layer.bias if self.base_layer.skip_bias_add else None
        return output, output_bias
```
**EN:** This block uses `ColumnParallelLinearWithLoRA.forward` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ColumnParallelLinearWithLoRA.forward` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 476-477: ColumnParallelLinearWithLoRA.slice_lora_a_weights method
```python
    def slice_lora_a_weights(self, A: torch.Tensor, tp_rank: int):
        return A
```
**EN:** This block uses `ColumnParallelLinearWithLoRA.slice_lora_a_weights` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ColumnParallelLinearWithLoRA.slice_lora_a_weights` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 479-484: ColumnParallelLinearWithLoRA.slice_lora_b_weights method
```python
    def slice_lora_b_weights(self, B: torch.Tensor, tp_rank: int):
        shard_size = self.base_layer.output_partition_sizes[0]
        start_idx = tp_rank * shard_size
        end_idx = (tp_rank + 1) * shard_size
        B = B[start_idx:end_idx, :]
        return B
```
**EN:** This block uses `ColumnParallelLinearWithLoRA.slice_lora_b_weights` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ColumnParallelLinearWithLoRA.slice_lora_b_weights` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 487-487: MergedColumnParallelLinearWithLoRA class declaration
```python
class MergedColumnParallelLinearWithLoRA(ColumnParallelLinearWithLoRA):
```
**EN:** This block declares the `MergedColumnParallelLinearWithLoRA` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `MergedColumnParallelLinearWithLoRA` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 488-494: MergedColumnParallelLinearWithLoRA initializer
```python
    def __init__(
        self,
        base_layer: MergedColumnParallelLinear,
        lora_backend: BaseLoRABackend,
    ) -> None:
        super().__init__(base_layer, lora_backend)
        self.n_slices = len(self.base_layer.output_partition_sizes)
```
**EN:** This block initializes the `MergedColumnParallelLinearWithLoRA` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `MergedColumnParallelLinearWithLoRA` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 496-534: MergedColumnParallelLinearWithLoRA.set_lora_info method
```python
    def set_lora_info(
        self,
        A_buffer: torch.Tensor,
        B_buffer: torch.Tensor,
    ):
        self.set_lora = True
        self.A_buffer = A_buffer
        self.B_buffer = B_buffer

        # Build cumulative output offsets from the first `lora_n_slices`
        # base partitions. `lora_n_slices` may be smaller than self.n_slices
        # when only a subset of partitions are LoRA'd (e.g. Mamba in_proj
        # has 5 partitions but stacked_multiply=2), so we can't precompute
        # these in __init__.
        lora_n_slices = self._get_lora_n_slices()
        if lora_n_slices <= 0 or lora_n_slices > self.n_slices:
            raise ValueError(
                f"Invalid LoRA slice count {lora_n_slices} for "
                f"{self.n_slices} base output partitions."
            )
        partition_sizes = list(self.base_layer.output_partition_sizes[:lora_n_slices])
        offsets = [0]
        for ps in partition_sizes:
            offsets.append(offsets[-1] + ps)
        if offsets[-1] != B_buffer.shape[-2]:
            raise ValueError(
                f"LoRA B output dim {B_buffer.shape[-2]} does not match "
                f"base partition prefix dim {offsets[-1]} for {lora_n_slices} slices."
            )
        self.output_offset = torch.tensor(
            offsets,
            dtype=torch.int32,
            device=next(self.base_layer.parameters()).device,
        )
        self.output_offset_cpu = self.output_offset.cpu().pin_memory()
        self.max_out_dim = max(partition_sizes)
        self.use_gate_up_lora = (
            lora_n_slices == 2 and partition_sizes[0] == partition_sizes[1]
        )
```
**EN:** This block uses `MergedColumnParallelLinearWithLoRA.set_lora_info` to update stored state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MergedColumnParallelLinearWithLoRA.set_lora_info` 来更新已保存的状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 536-546: MergedColumnParallelLinearWithLoRA._get_lora_n_slices method
```python
    def _get_lora_n_slices(self) -> int:
        """Actual number of LoRA slices from the buffer shapes.

        May differ from self.n_slices (base layer partitions) when only a
        subset of partitions are LoRA'd (e.g. Mamba in_proj has 5 partitions
        but stacked_multiply=2).
        """
        lora_rank = self.B_buffer.shape[-1]
        if lora_rank == 0:
            return self.n_slices
        return self.A_buffer.shape[-2] // lora_rank
```
**EN:** This block uses `MergedColumnParallelLinearWithLoRA._get_lora_n_slices` to fetch or compute a value. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MergedColumnParallelLinearWithLoRA._get_lora_n_slices` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 548-570: MergedColumnParallelLinearWithLoRA.apply_lora method
```python
    def apply_lora(self, base_output: torch.Tensor, x: torch.Tensor) -> torch.Tensor:
        lora_n_slices = self._get_lora_n_slices()
        if lora_n_slices == 2 and self.use_gate_up_lora:
            lora_output = self.lora_backend.run_gate_up_lora(
                x=x,
                gate_up_lora_a=self.A_buffer,
                gate_up_lora_b=self.B_buffer,
                output_offset=self.output_offset,
                output_offset_cpu=self.output_offset_cpu,
                base_output=base_output,
            )
        else:
            lora_output = self.lora_backend.run_qkv_lora(
                x=x,
                qkv_lora_a=self.A_buffer,
                qkv_lora_b=self.B_buffer,
                output_offset=self.output_offset,
                output_offset_cpu=self.output_offset_cpu,
                max_qkv_out_dim=self.max_out_dim,
                base_output=base_output,
                n_slices=lora_n_slices,
            )
        return lora_output
```
**EN:** This block uses `MergedColumnParallelLinearWithLoRA.apply_lora` to apply a transformation to tensors or state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MergedColumnParallelLinearWithLoRA.apply_lora` 来对张量或状态应用变换。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 572-573: MergedColumnParallelLinearWithLoRA.slice_lora_a_weights method
```python
    def slice_lora_a_weights(self, A: torch.Tensor, tp_rank: int):
        return A
```
**EN:** This block uses `MergedColumnParallelLinearWithLoRA.slice_lora_a_weights` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MergedColumnParallelLinearWithLoRA.slice_lora_a_weights` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 575-585: MergedColumnParallelLinearWithLoRA.slice_lora_b_weights method
```python
    def slice_lora_b_weights(self, B: torch.Tensor, tp_rank: int):
        partition_sizes = self.base_layer.output_partition_sizes
        output_sizes = self.base_layer.output_sizes
        slices = []
        offset = 0
        for full_size, part_size in zip(output_sizes, partition_sizes):
            start_idx = tp_rank * part_size
            end_idx = start_idx + part_size
            slices.append(B[offset + start_idx : offset + end_idx, :])
            offset += full_size
        return torch.concat(slices, dim=0)
```
**EN:** This block uses `MergedColumnParallelLinearWithLoRA.slice_lora_b_weights` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MergedColumnParallelLinearWithLoRA.slice_lora_b_weights` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 588-588: QKVParallelLinearWithLoRA class declaration
```python
class QKVParallelLinearWithLoRA(ColumnParallelLinearWithLoRA):
```
**EN:** This block declares the `QKVParallelLinearWithLoRA` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `QKVParallelLinearWithLoRA` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 589-616: QKVParallelLinearWithLoRA initializer
```python
    def __init__(
        self,
        base_layer: QKVParallelLinear,
        lora_backend: BaseLoRABackend,
    ) -> None:
        super().__init__(base_layer, lora_backend)
        q_proj_shard_size = self.base_layer.q_proj_shard_size
        kv_proj_shard_size = self.base_layer.kv_proj_shard_size
        offsets = [
            0,
            q_proj_shard_size,
            q_proj_shard_size + kv_proj_shard_size,
            q_proj_shard_size + 2 * kv_proj_shard_size,
        ]
        self.output_offset = torch.tensor(
            offsets,
            dtype=torch.int32,
            device=next(self.base_layer.parameters()).device,
        )
        self.output_offset_cpu = torch.tensor(
            offsets,
            dtype=torch.int32,
            device="cpu",
            pin_memory=True,
        )

        # For computing number of launched blocks
        self.max_qkv_out_dim = max(q_proj_shard_size, kv_proj_shard_size)
```
**EN:** This block initializes the `QKVParallelLinearWithLoRA` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `QKVParallelLinearWithLoRA` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 618-625: QKVParallelLinearWithLoRA.set_lora_info method
```python
    def set_lora_info(
        self,
        A_buffer_qkv: torch.Tensor,
        B_buffer_qkv: torch.Tensor,
    ):
        self.set_lora = True
        self.A_buffer_qkv = A_buffer_qkv
        self.B_buffer_qkv = B_buffer_qkv
```
**EN:** This block uses `QKVParallelLinearWithLoRA.set_lora_info` to update stored state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `QKVParallelLinearWithLoRA.set_lora_info` 来更新已保存的状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 627-638: QKVParallelLinearWithLoRA.apply_lora method
```python
    def apply_lora(self, base_output: torch.Tensor, x: torch.Tensor) -> torch.Tensor:
        lora_output = self.lora_backend.run_qkv_lora(
            x=x,
            qkv_lora_a=self.A_buffer_qkv,
            qkv_lora_b=self.B_buffer_qkv,
            base_output=base_output,
            output_offset=self.output_offset,
            output_offset_cpu=self.output_offset_cpu,
            max_qkv_out_dim=self.max_qkv_out_dim,
        )

        return lora_output
```
**EN:** This block uses `QKVParallelLinearWithLoRA.apply_lora` to apply a transformation to tensors or state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `QKVParallelLinearWithLoRA.apply_lora` 来对张量或状态应用变换。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 640-641: QKVParallelLinearWithLoRA.slice_lora_a_weights method
```python
    def slice_lora_a_weights(self, A: torch.Tensor, tp_rank: int):
        return A
```
**EN:** This block uses `QKVParallelLinearWithLoRA.slice_lora_a_weights` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `QKVParallelLinearWithLoRA.slice_lora_a_weights` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 643-669: QKVParallelLinearWithLoRA.slice_lora_b_weights method
```python
    def slice_lora_b_weights(self, B: torch.Tensor, tp_rank: int) -> torch.Tensor:
        base_layer = self.base_layer
        q_proj_shard_size = base_layer.q_proj_shard_size
        kv_proj_shard_size = base_layer.kv_proj_shard_size
        num_kv_head_replicas = base_layer.num_kv_head_replicas

        q_start_idx = q_proj_shard_size * tp_rank
        q_end_idx = q_start_idx + q_proj_shard_size

        kv_shard_id = tp_rank // num_kv_head_replicas
        kv_start_idx = kv_proj_shard_size * kv_shard_id
        kv_end_idx = kv_start_idx + kv_proj_shard_size

        q_size = base_layer.output_sizes[0]
        k_size = base_layer.output_sizes[1] // num_kv_head_replicas
        B_q_shard = B[q_start_idx:q_end_idx, :]
        B_k_shard = B[q_size + kv_start_idx : q_size + kv_end_idx, :]
        B_v_shard = B[q_size + k_size + kv_start_idx : q_size + k_size + kv_end_idx, :]

        return torch.concat(
            (
                B_q_shard,
                B_k_shard,
                B_v_shard,
            ),
            dim=0,
        )
```
**EN:** This block uses `QKVParallelLinearWithLoRA.slice_lora_b_weights` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `QKVParallelLinearWithLoRA.slice_lora_b_weights` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 672-672: RowParallelLinearWithLoRA class declaration
```python
class RowParallelLinearWithLoRA(BaseLayerWithLoRA):
```
**EN:** This block declares the `RowParallelLinearWithLoRA` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `RowParallelLinearWithLoRA` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 673-678: RowParallelLinearWithLoRA initializer
```python
    def __init__(
        self,
        base_layer: RowParallelLinear,
        lora_backend: BaseLoRABackend,
    ) -> None:
        super().__init__(base_layer, lora_backend)
```
**EN:** This block initializes the `RowParallelLinearWithLoRA` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `RowParallelLinearWithLoRA` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 680-696: RowParallelLinearWithLoRA.set_lora_info method
```python
    def set_lora_info(self, A_buffer: torch.Tensor, B_buffer: torch.Tensor):
        self.set_lora = True
        self.A_buffer = A_buffer
        self.B_buffer = B_buffer
        output_size = self.base_layer.output_size
        offsets = [0, output_size]
        self.output_offset = torch.tensor(
            offsets,
            dtype=torch.int32,
            device=next(self.base_layer.parameters()).device,
        )
        self.output_offset_cpu = torch.tensor(
            offsets,
            dtype=torch.int32,
            device="cpu",
            pin_memory=True,
        )
```
**EN:** This block uses `RowParallelLinearWithLoRA.set_lora_info` to update stored state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `RowParallelLinearWithLoRA.set_lora_info` 来更新已保存的状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 698-707: RowParallelLinearWithLoRA.apply_lora method
```python
    def apply_lora(self, base_output: torch.Tensor, x: torch.Tensor) -> torch.Tensor:
        lora_a_output = self.lora_backend.run_lora_a_sgemm(x, self.A_buffer)
        lora_output = self.lora_backend.run_lora_b_sgemm(
            x=lora_a_output,
            weights=self.B_buffer,
            output_offset=self.output_offset,
            output_offset_cpu=self.output_offset_cpu,
            base_output=base_output,
        )
        return lora_output
```
**EN:** This block uses `RowParallelLinearWithLoRA.apply_lora` to apply a transformation to tensors or state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `RowParallelLinearWithLoRA.apply_lora` 来对张量或状态应用变换。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 709-756: RowParallelLinearWithLoRA.forward method
```python
    def forward(self, input_: torch.Tensor, skip_all_reduce=False, forward_batch=None):
        if self.base_layer.input_is_parallel:
            input_parallel = input_
        else:
            tp_rank = get_tensor_model_parallel_rank()
            splitted_input = split_tensor_along_last_dim(
                input_, num_partitions=self.base_layer.tp_size
            )
            input_parallel = splitted_input[tp_rank].contiguous()

        bias_ = (
            None
            if (self.base_layer.tp_rank > 0 or self.base_layer.skip_bias_add)
            else self.base_layer.bias
        )
        output_parallel = self.base_layer.quant_method.apply(
            self.base_layer, input_parallel, bias=bias_
        )

        should_reduce = (
            self.base_layer.reduce_results
            and self.base_layer.tp_size > 1
            and not skip_all_reduce
        )

        if self.set_lora and should_reduce:
            lora_a_output = self.lora_backend.run_lora_a_sgemm(
                input_parallel, self.A_buffer
            )
            output_ = tensor_model_parallel_all_reduce(output_parallel)
            lora_a_output = tensor_model_parallel_all_reduce(lora_a_output)
            output_ = self.lora_backend.run_lora_b_sgemm(
                x=lora_a_output,
                weights=self.B_buffer,
                output_offset=self.output_offset,
                output_offset_cpu=self.output_offset_cpu,
                base_output=output_,
            )
        else:
            if self.set_lora:
                output_parallel = self.apply_lora(output_parallel, input_parallel)
            if should_reduce:
                output_ = tensor_model_parallel_all_reduce(output_parallel)
            else:
                output_ = output_parallel

        output_bias = self.base_layer.bias if self.base_layer.skip_bias_add else None
        return output_, output_bias
```
**EN:** This block uses `RowParallelLinearWithLoRA.forward` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `RowParallelLinearWithLoRA.forward` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 758-763: RowParallelLinearWithLoRA.slice_lora_a_weights method
```python
    def slice_lora_a_weights(self, A: torch.Tensor, tp_rank: int):
        shard_size = self.base_layer.input_size_per_partition
        start_idx = tp_rank * shard_size
        end_idx = (tp_rank + 1) * shard_size
        A = A[:, start_idx:end_idx].contiguous()
        return A
```
**EN:** This block uses `RowParallelLinearWithLoRA.slice_lora_a_weights` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `RowParallelLinearWithLoRA.slice_lora_a_weights` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 765-766: RowParallelLinearWithLoRA.slice_lora_b_weights method
```python
    def slice_lora_b_weights(self, B: torch.Tensor, tp_rank: int):
        return B
```
**EN:** This block uses `RowParallelLinearWithLoRA.slice_lora_b_weights` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `RowParallelLinearWithLoRA.slice_lora_b_weights` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 769-783: ReplicatedLinearWithLoRA class declaration
```python
class ReplicatedLinearWithLoRA(BaseLayerWithLoRA):
    """LoRA wrapper for ReplicatedLinear (no TP sharding).

    Used for DeepSeek MLA's fused_qkv_a_proj_with_mqa, which fuses
    q_a_proj and kv_a_proj_with_mqa into a single replicated linear.
    The two sub-projections have unequal output dimensions, so we use
    the N-component fused kernel (run_qkv_lora) with n_slices=2 to
    handle the split inside the triton kernel rather than in Python.

    ``first_output_dim`` (set by LoRAManager after construction) marks the
    boundary between the first and second sub-projection in the output.
    """

    first_output_dim: int = 0
```
**EN:** This block declares the `ReplicatedLinearWithLoRA` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `ReplicatedLinearWithLoRA` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 784-790: ReplicatedLinearWithLoRA initializer
```python
    def __init__(
        self,
        base_layer: ReplicatedLinear,
        lora_backend: BaseLoRABackend,
    ) -> None:
        super().__init__(base_layer, lora_backend)
        self.output_size = base_layer.output_size
```
**EN:** This block initializes the `ReplicatedLinearWithLoRA` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `ReplicatedLinearWithLoRA` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 792-813: ReplicatedLinearWithLoRA.set_lora_info method
```python
    def set_lora_info(self, A_buffer: torch.Tensor, B_buffer: torch.Tensor):
        self.set_lora = True
        self.A_buffer = A_buffer
        self.B_buffer = B_buffer
        first_dim = self.first_output_dim
        if first_dim > 0:
            second_dim = B_buffer.shape[-2] - first_dim
            self._output_offset = torch.tensor(
                [0, first_dim, first_dim + second_dim],
                dtype=torch.int32,
                device=B_buffer.device,
            )
            self._output_offset_cpu = self._output_offset.cpu()
            self._max_out_dim = max(first_dim, second_dim)
        else:
            # Single-projection path: csgmv backend requires an explicit
            # slice_offsets tensor of shape [0, output_dim].
            self._output_offset = torch.tensor(
                [0, B_buffer.shape[-2]],
                dtype=torch.int32,
                device=B_buffer.device,
            )
```
**EN:** This block uses `ReplicatedLinearWithLoRA.set_lora_info` to update stored state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ReplicatedLinearWithLoRA.set_lora_info` 来更新已保存的状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 815-842: ReplicatedLinearWithLoRA.apply_lora method
```python
    def apply_lora(self, base_output: torch.Tensor, x: torch.Tensor) -> torch.Tensor:
        first_dim = self.first_output_dim

        if first_dim == 0:
            # Simple single-projection (e.g. fc1_latent_proj, fc2_latent_proj)
            lora_a_output = self.lora_backend.run_lora_a_sgemm(x, self.A_buffer)
            lora_output = self.lora_backend.run_lora_b_sgemm(
                x=lora_a_output,
                weights=self.B_buffer,
                output_offset=self._output_offset,
                base_output=base_output,
            )
            return lora_output

        # Use the fused N-component kernel with n_slices=2 to handle the
        # split inside the triton kernel, avoiding Python-level splitting
        # which breaks when adapter rank < max_lora_rank.
        lora_output = self.lora_backend.run_qkv_lora(
            x=x,
            qkv_lora_a=self.A_buffer,
            qkv_lora_b=self.B_buffer,
            output_offset=self._output_offset,
            output_offset_cpu=self._output_offset_cpu,
            max_qkv_out_dim=self._max_out_dim,
            base_output=base_output,
            n_slices=2,
        )
        return lora_output
```
**EN:** This block uses `ReplicatedLinearWithLoRA.apply_lora` to apply a transformation to tensors or state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ReplicatedLinearWithLoRA.apply_lora` 来对张量或状态应用变换。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 844-850: ReplicatedLinearWithLoRA.forward method
```python
    def forward(self, x: torch.Tensor):
        bias = self.base_layer.bias if not self.base_layer.skip_bias_add else None
        output = self.base_layer.quant_method.apply(self.base_layer, x, bias)
        if self.set_lora:
            output = self.apply_lora(output, x)
        output_bias = self.base_layer.bias if self.base_layer.skip_bias_add else None
        return output, output_bias
```
**EN:** This block uses `ReplicatedLinearWithLoRA.forward` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ReplicatedLinearWithLoRA.forward` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 852-853: ReplicatedLinearWithLoRA.slice_lora_a_weights method
```python
    def slice_lora_a_weights(self, A: torch.Tensor, tp_rank: int):
        return A
```
**EN:** This block uses `ReplicatedLinearWithLoRA.slice_lora_a_weights` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ReplicatedLinearWithLoRA.slice_lora_a_weights` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 855-856: ReplicatedLinearWithLoRA.slice_lora_b_weights method
```python
    def slice_lora_b_weights(self, B: torch.Tensor, tp_rank: int):
        return B
```
**EN:** This block uses `ReplicatedLinearWithLoRA.slice_lora_b_weights` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `ReplicatedLinearWithLoRA.slice_lora_b_weights` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 859-870: FusedMoEWithLoRA class declaration
```python
class FusedMoEWithLoRA(BaseLayerWithLoRA):
    """
    Wrapper around FusedMoE that integrates LoRA into the MoE computation.

    Design: LoRA deltas are added at specific points in the MoE forward pass:
    1. After gate_up projection, BEFORE activation (halfway through)
    2. After down projection, BEFORE final reduction

    This follows the vLLM/HF approach where LoRA is fused into the computation
    rather than computed independently and added at the end.
    """
```
**EN:** This block declares the `FusedMoEWithLoRA` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `FusedMoEWithLoRA` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 871-930: FusedMoEWithLoRA initializer (part 1/2)
```python
    def __init__(
        self,
        base_layer: FusedMoE,
        lora_backend: BaseLoRABackend,
    ):
        # initializes FusedMoE with its own moe_runner for base path
        super().__init__(base_layer, lora_backend)

        self.experts_shared_outer_loras: bool = False
        self.lora_use_virtual_experts: bool = False
        self.quant_method = base_layer.quant_method

        self.tp_size = getattr(base_layer, "moe_tp_size", 1)
        self.tp_rank = getattr(base_layer, "moe_tp_rank", 0)
        self.intermediate_size_per_partition = getattr(
            base_layer, "intermediate_size_per_partition", None
        )
        self._uses_interleaved_gate_up = (
            getattr(base_layer.moe_runner_config, "gemm1_alpha", None) is not None
        )

        # Initialize triton_lora moe runner for batches with lora enabled
        from sglang.srt.layers.moe import MoeRunnerBackend
        from sglang.srt.layers.moe.moe_runner.runner import MoeRunner
        from sglang.srt.layers.moe.utils import get_moe_runner_backend

        # Determine runner backend: prefer server arg, fall back to quant method's runner
        global_backend = get_moe_runner_backend()
        if not global_backend.is_auto():
            runner_backend = global_backend
        elif (
            hasattr(base_layer.quant_method, "runner")
            and base_layer.quant_method.runner is not None
        ):
            runner_backend = base_layer.quant_method.runner.runner_backend
        else:
            runner_backend = MoeRunnerBackend.TRITON

        self._lora_runner = MoeRunner(
            runner_backend,
            base_layer.moe_runner_config,
            lora_enabled=True,
        )

        if runner_backend.is_marlin():
            from sglang.srt.layers.quantization.compressed_tensors.compressed_tensors import (
                CompressedTensorsFusedMoEMethod,
            )

            assert isinstance(
                base_layer.quant_method, CompressedTensorsFusedMoEMethod
            ), (
                f"Marlin MoE backend requires CompressedTensorsFusedMoEMethod, "
                f"got {type(base_layer.quant_method).__name__}"
            )
            self._quant_info = base_layer.quant_method.get_marlin_quant_info(base_layer)
        elif runner_backend.is_triton():
            assert base_layer.quant_method is not None, "Quant method must be set"
            self._quant_info = base_layer.quant_method.get_triton_quant_info(base_layer)
        else:
```
**EN:** This block initializes the `FusedMoEWithLoRA` object, setting up the state, buffers, and references that later methods rely on. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块初始化 `FusedMoEWithLoRA` 对象，建立后续方法依赖的状态、缓冲区与引用。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 931-933: FusedMoEWithLoRA initializer (part 2/2)
```python
            raise NotImplementedError(
                f"LoRA MoE not supported for backend {runner_backend}"
            )
```
**EN:** This block initializes the `FusedMoEWithLoRA` object, setting up the state, buffers, and references that later methods rely on. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块初始化 `FusedMoEWithLoRA` 对象，建立后续方法依赖的状态、缓冲区与引用。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 935-947: FusedMoEWithLoRA.set_lora_info method
```python
    def set_lora_info(
        self,
        gate_up_lora_a_weights: torch.Tensor,
        gate_up_lora_b_weights: torch.Tensor,
        down_lora_a_weights: torch.Tensor = None,
        down_lora_b_weights: torch.Tensor = None,
    ):
        """Set LoRA weight tensors from memory pool."""
        self.set_lora = True
        self.gate_up_lora_a_weights = gate_up_lora_a_weights
        self.gate_up_lora_b_weights = gate_up_lora_b_weights
        self.down_lora_a_weights = down_lora_a_weights
        self.down_lora_b_weights = down_lora_b_weights
```
**EN:** This block uses `FusedMoEWithLoRA.set_lora_info` to update stored state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FusedMoEWithLoRA.set_lora_info` 来更新已保存的状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 949-1000: FusedMoEWithLoRA._get_lora_info method
```python
    def _get_lora_info(self):
        """Build LoRAInfo for the current batch."""
        from sglang.srt.lora.lora_moe_runners import LoRAInfo

        batch_info = self.lora_backend.batch_info
        lora_ranks = batch_info.lora_ranks

        max_lora_rank = self.down_lora_a_weights.shape[2]

        cg_buffers = getattr(self.lora_backend, "moe_cg_buffers", None)
        wi = (
            batch_info.req_weight_indices
            if batch_info.req_weight_indices is not None
            else batch_info.weight_indices
        )
        if cg_buffers is not None and batch_info.use_cuda_graph:
            adapter_enabled = cg_buffers["adapter_enabled"]
            adapter_enabled.zero_()
            idx_buf = cg_buffers["weight_indices_long"]
            idx_buf[: batch_info.bs] = wi[: batch_info.bs]
            adapter_enabled.index_fill_(0, idx_buf[: batch_info.bs], 1)
        else:
            adapter_enabled = torch.zeros(
                len(lora_ranks), dtype=torch.int32, device=lora_ranks.device
            )
            adapter_enabled.index_fill_(0, wi.long(), 1)

        seg_indptr = (
            batch_info.req_seg_indptr
            if batch_info.req_seg_indptr is not None
            else batch_info.seg_indptr
        )
        req_to_lora = wi

        return LoRAInfo(
            gate_up_lora_a_weights=self.gate_up_lora_a_weights,
            gate_up_lora_b_weights=self.gate_up_lora_b_weights,
            down_lora_a_weights=self.down_lora_a_weights,
            down_lora_b_weights=self.down_lora_b_weights,
            seg_indptr=seg_indptr,
            req_to_lora=req_to_lora,
            lora_ranks=lora_ranks,
            adapter_enabled=adapter_enabled,
            max_lora_rank=max_lora_rank,
            num_experts=self.base_layer.num_experts,
            experts_shared_outer_loras=self.experts_shared_outer_loras,
            cg_buffers=cg_buffers,
            tp_size=self.tp_size,
            tp_rank=self.tp_rank,
            hidden_size=getattr(self.base_layer, "hidden_size", 0),
            lora_use_virtual_experts=self.lora_use_virtual_experts,
        )
```
**EN:** This block uses `FusedMoEWithLoRA._get_lora_info` to fetch or compute a value. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FusedMoEWithLoRA._get_lora_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1002-1015: FusedMoEWithLoRA.forward method
```python
    def forward(self, hidden_states: torch.Tensor, topk_output: TopKOutput, **kwargs):
        """
        Forward pass with integrated LoRA computation.

        LoRA deltas are added at the correct points inside the MoE computation:
        1. After gate_up projection, before activation
        2. After down projection, before final reduction
        """

        # Build LoRA info for this batch
        lora_info = self._get_lora_info()

        # run lora moe_runner
        return self._forward_with_lora(hidden_states, topk_output, lora_info, **kwargs)
```
**EN:** This block uses `FusedMoEWithLoRA.forward` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FusedMoEWithLoRA.forward` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1017-1045: FusedMoEWithLoRA._forward_with_lora method
```python
    def _forward_with_lora(
        self,
        hidden_states: torch.Tensor,
        topk_output: TopKOutput,
        lora_info,
        **kwargs,
    ):
        """
        Run MoE forward with LoRA integration at the correct points.
        """
        # Get the base layer's dispatch and combine logic
        base_layer = self.base_layer

        # Dispatch tokens (doesn't do much in the LoRA case)
        dispatch_output = base_layer.dispatcher.dispatch(
            hidden_states=hidden_states, topk_output=topk_output
        )

        # Use pre-computed quant info (doesn't change so not sure why we need to pass it in every time)
        quant_info = self._quant_info

        # Run the only lora moe runner (Triton)
        combine_input = self._lora_runner.run(
            dispatch_output, quant_info, lora_info=lora_info
        )

        final_hidden_states = base_layer.dispatcher.combine(combine_input=combine_input)

        return final_hidden_states
```
**EN:** This block uses `FusedMoEWithLoRA._forward_with_lora` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FusedMoEWithLoRA._forward_with_lora` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1047-1048: FusedMoEWithLoRA.slice_lora_a_weights method
```python
    def slice_lora_a_weights(self, A: torch.Tensor, tp_rank: int):
        return A
```
**EN:** This block uses `FusedMoEWithLoRA.slice_lora_a_weights` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FusedMoEWithLoRA.slice_lora_a_weights` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1050-1051: FusedMoEWithLoRA.slice_lora_b_weights method
```python
    def slice_lora_b_weights(self, B: torch.Tensor, tp_rank: int):
        return B
```
**EN:** This block uses `FusedMoEWithLoRA.slice_lora_b_weights` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FusedMoEWithLoRA.slice_lora_b_weights` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1053-1079: FusedMoEWithLoRA.slice_moe_lora_a_weights method
```python
    def slice_moe_lora_a_weights(
        self,
        A: Union[torch.Tensor, Dict[int, torch.Tensor]],
        tp_rank: int,
        target_module: str,
    ):
        """Slice LoRA A weights for MoE with TP.

        Accepts:
          - 2D tensor [rank, hidden] (single expert)
          - 3D tensor [num_experts_or_1, rank, hidden]
          - dict {expert_id: 2D tensor}

        Per-expert weight shapes:
          gate_up_proj_moe A: [rank, hidden_size]  — input is full hidden_states, no slice
          down_proj_moe A:    [rank, intermediate_size] — input is sharded intermediate
        """
        if self.tp_size <= 1:
            return A
        if target_module != "down_proj_moe":
            return A
        if isinstance(A, dict):
            return {
                eid: self._slice_moe_a(w, tp_rank, target_module)
                for eid, w in A.items()
            }
        return self._slice_moe_a(A, tp_rank, target_module)
```
**EN:** This block uses `FusedMoEWithLoRA.slice_moe_lora_a_weights` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FusedMoEWithLoRA.slice_moe_lora_a_weights` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1081-1087: FusedMoEWithLoRA._slice_moe_a method
```python
    def _slice_moe_a(
        self, A: torch.Tensor, tp_rank: int, target_module: str
    ) -> torch.Tensor:
        shard_size = self.intermediate_size_per_partition
        start = tp_rank * shard_size
        end = start + shard_size
        return A[..., start:end].contiguous()
```
**EN:** This block uses `FusedMoEWithLoRA._slice_moe_a` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FusedMoEWithLoRA._slice_moe_a` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1089-1125: FusedMoEWithLoRA.slice_moe_lora_b_weights method
```python
    def slice_moe_lora_b_weights(
        self,
        B: Union[torch.Tensor, Dict[int, torch.Tensor]],
        tp_rank: int,
        target_module: str,
    ):
        """Slice LoRA B weights for MoE with TP.

        Accepts:
          - 2D tensor [output_dim, rank] (single expert)
          - 3D tensor [num_experts_or_1, output_dim, rank]
          - dict {expert_id: 2D tensor}

        Per-expert weight shapes:
          gate_up_proj_moe B: [intermediate_size*2, rank] — output matches sharded base w13
          down_proj_moe B:    [hidden_size, rank] — output is all-reduced, no slice
        """
        needs_processing = (self.tp_size > 1) or (
            target_module == "gate_up_proj_moe" and self._uses_interleaved_gate_up
        )
        if not needs_processing:
            return B
        if target_module != "gate_up_proj_moe":
            return B
        if isinstance(B, dict):
            return {
                eid: self._slice_moe_b_2d(w, tp_rank, target_module)
                for eid, w in B.items()
            }
        if isinstance(B, torch.Tensor) and B.dim() == 3:
            return torch.stack(
                [
                    self._slice_moe_b_2d(B[i], tp_rank, target_module)
                    for i in range(B.shape[0])
                ]
            )
        return self._slice_moe_b_2d(B, tp_rank, target_module)
```
**EN:** This block uses `FusedMoEWithLoRA.slice_moe_lora_b_weights` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FusedMoEWithLoRA.slice_moe_lora_b_weights` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1127-1151: FusedMoEWithLoRA._slice_moe_b_2d method
```python
    def _slice_moe_b_2d(
        self, B: torch.Tensor, tp_rank: int, target_module: str
    ) -> torch.Tensor:
        if target_module == "gate_up_proj_moe":
            # Non-gated MoE (e.g. Nemotron-H): only w1, no w3.
            # B has shape [intermediate_size, rank] — TP-shard directly.
            is_gated = self.base_layer.moe_runner_config.is_gated
            if not is_gated:
                if self.tp_size > 1:
                    shard_size = self.intermediate_size_per_partition
                    start = tp_rank * shard_size
                    end = start + shard_size
                    return B[start:end, :]
                return B

            shard_size = self.intermediate_size_per_partition
            start = tp_rank * shard_size
            end = start + shard_size
            full_inter = B.shape[0] // 2
            gate_b = B[start:end, :]
            up_b = B[full_inter + start : full_inter + end, :]
            if self._uses_interleaved_gate_up:
                return torch.stack([gate_b, up_b], dim=1).reshape(-1, B.shape[-1])
            return torch.cat([gate_b, up_b], dim=0).contiguous()
        return B
```
**EN:** This block uses `FusedMoEWithLoRA._slice_moe_b_2d` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `FusedMoEWithLoRA._slice_moe_b_2d` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 1154-1172: get lora layer function
```python
def get_lora_layer(
    layer: nn.Module, lora_backend: BaseLoRABackend
) -> BaseLayerWithLoRA:
    supported_layer_types = {
        # the order matters
        FusedMoE: FusedMoEWithLoRA,
        ParallelLMHead: ParallelLMHeadWithLoRA,
        VocabParallelEmbedding: VocabParallelEmbeddingWithLoRA,
        ReplicatedLinear: ReplicatedLinearWithLoRA,
        QKVParallelLinear: QKVParallelLinearWithLoRA,
        MergedColumnParallelLinear: MergedColumnParallelLinearWithLoRA,
        ColumnParallelLinear: ColumnParallelLinearWithLoRA,
        RowParallelLinear: RowParallelLinearWithLoRA,
    }
    for src_layer_type, lora_layer_type in supported_layer_types.items():
        if isinstance(layer, src_layer_type):  # pylint: disable=unidiomatic-typecheck
            ret = lora_layer_type(layer, lora_backend)
            return ret
    raise Exception(f"No corresponding LoRA layer supported for {type(layer)}.")
```
**EN:** This block uses `get_lora_layer` to fetch or compute a value. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `get_lora_layer` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- LoRA adapter execution / LoRA 适配器执行
- Tensor computation with PyTorch / 基于 PyTorch 的张量计算

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.distributed`
- `sglang.srt.layers.communicator`
- `sglang.srt.layers.linear`
- `sglang.srt.layers.moe`
- `sglang.srt.layers.moe.fused_moe_triton.layer`
- `sglang.srt.layers.moe.moe_runner.runner`
- `sglang.srt.layers.moe.topk`
- `sglang.srt.layers.moe.utils`
- `sglang.srt.layers.quantization.compressed_tensors.compressed_tensors`
- `sglang.srt.layers.vocab_parallel_embedding`
- `sglang.srt.lora.backend.base_backend`
- `sglang.srt.lora.lora_moe_runners`
- `sglang.srt.lora.utils`
### External / 外部
- `torch`
- `typing` (stdlib)
