# sparse_pooler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/sparse_pooler.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements embedding lookup and preprocessing with pooling helpers for the SGLang SRT runtime. It exposes symbols such as `SparseEmbeddingOutput` and `SparsePooler` and connects them to backend-specific paths such as `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了嵌入查找与预处理，并结合池化辅助逻辑。它提供了 `SparseEmbeddingOutput` 和 `SparsePooler` 等符号，并把这些符号连接到 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Module imports and dependency wiring
```python
from dataclasses import dataclass

import torch
import torch.nn as nn
import torch.nn.functional as F
from transformers import PretrainedConfig

from sglang.srt.model_executor.model_runner import ForwardBatch
```
**EN:** This section prepares the module namespace. It imports `dataclasses.dataclass`, `torch`, `torch.nn`, `torch.nn.functional`, `transformers.PretrainedConfig`, and `sglang.srt.model_executor.model_runner.ForwardBatch`, so later blocks can reuse runtime, tensor, or backend helpers.
**CN:** 该部分负责准备模块命名空间。 它导入了 `dataclasses.dataclass`、`torch`、`torch.nn`、`torch.nn.functional`、`transformers.PretrainedConfig` 以及 `sglang.srt.model_executor.model_runner.ForwardBatch`，让后续代码可以复用运行时、张量或后端辅助逻辑。

### Lines 11-15: Class `SparseEmbeddingOutput` declaration and shared state
```python
@dataclass
class SparseEmbeddingOutput:
    embeddings: torch.Tensor  # [batch_size, vocab_size]
```
**EN:** This block introduces class `SparseEmbeddingOutput` and the state shared by its methods.
**CN:** 该代码块引入类 `SparseEmbeddingOutput`，并定义其方法共享的状态。

### Lines 16-30: Class `SparsePooler` declaration and shared state
```python
class SparsePooler(nn.Module):
    """A layer that pools hidden states into sparse vocabulary-space embeddings.

    This layer does the following:
    1. Applies a linear transformation + ReLU to get token-level weights
    2. Maps these weights to vocabulary positions using token IDs
    3. Aggregates weights for repeated tokens using max pooling
    4. Returns sparse embeddings in vocabulary space

    Attributes:
        config: Model configuration containing vocab_size and hidden_size
        sparse_linear: Linear layer for computing token weights
        vocab_size: Size of vocabulary for output embeddings
    """
```
**EN:** This block introduces class `SparsePooler` and the state shared by its methods. It inherits from `nn.Module`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: A layer that pools hidden states into sparse vocabulary-space embeddings.
**CN:** 该代码块引入类 `SparsePooler`，并定义其方法共享的状态。 它继承自 `nn.Module`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 31-47: `SparsePooler` initialization and state setup
```python
    def __init__(self, config: PretrainedConfig):
        super().__init__()

        # Validate required attributes
        if not hasattr(config, "vocab_size"):
            raise AttributeError(
                f"Config {type(config)} missing required 'vocab_size' attribute"
            )
        if not hasattr(config, "hidden_size"):
            raise AttributeError(
                f"Config {type(config)} missing required 'hidden_size' attribute"
            )

        self.vocab_size = config.vocab_size
        self.sparse_linear = nn.Linear(config.hidden_size, 1)
        self._weights_loaded = False
```
**EN:** This block defines `SparsePooler.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `nn.Linear`, `hasattr`, `AttributeError`, and `type`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.vocab_size`, `self.sparse_linear`, and `self._weights_loaded` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `SparsePooler.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`nn.Linear`、`hasattr`、`AttributeError` 以及 `type`，说明该流程会编排底层辅助函数或计算内核。 像 `self.vocab_size`、`self.sparse_linear` 以及 `self._weights_loaded` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 48-94: `SparsePooler.forward` main forward path
```python
    def forward(
        self, hidden_states: torch.Tensor, forward_batch: ForwardBatch
    ) -> SparseEmbeddingOutput:
        """
        Forward pass for sparse pooling.

        Args:
            hidden_states: Packed sequence hidden states [total_tokens, hidden_size]
            forward_batch: Batch information with sequence lengths and input_ids

        Returns:
            SparseEmbeddingOutput with embeddings of shape [batch_size, vocab_size]
        """
        if not self._weights_loaded:
            raise ValueError(
                "Sparse pooling weights not loaded. Call load_weights() first"
            )

        # Apply sparse linear + ReLU to get token weights
        token_weights = F.relu(self.sparse_linear(hidden_states)).squeeze(
            -1
        )  # [total_tokens]

        # Create batch indices for packed sequences
        batch_indices = torch.repeat_interleave(
            torch.arange(
                len(forward_batch.extend_seq_lens), device=hidden_states.device
            ),
            forward_batch.extend_seq_lens,
        )

        # Initialize sparse embedding output
        sparse_embedding = torch.zeros(
            len(forward_batch.extend_seq_lens),
            self.vocab_size,
            dtype=token_weights.dtype,
            device=token_weights.device,
        )

        # Map to vocabulary space using scatter_reduce with amax
        flat_indices = batch_indices * self.vocab_size + forward_batch.input_ids
        sparse_embedding.view(-1).scatter_reduce_(
            0, flat_indices, token_weights, reduce="amax"
        )

        return SparseEmbeddingOutput(embeddings=sparse_embedding)
```
**EN:** This block defines `SparsePooler.forward` and contains the main logic for this step. It mainly invokes `F.relu.squeeze`, `torch.repeat_interleave`, `torch.zeros`, `sparse_embedding.view.scatter_reduce_`, and `SparseEmbeddingOutput`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `token_weights`, `batch_indices`, `sparse_embedding`, and `flat_indices` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `SparsePooler.forward`，并承载这一阶段的核心逻辑。 它主要调用 `F.relu.squeeze`、`torch.repeat_interleave`、`torch.zeros`、`sparse_embedding.view.scatter_reduce_` 以及 `SparseEmbeddingOutput`，说明该流程会编排底层辅助函数或计算内核。 像 `token_weights`、`batch_indices`、`sparse_embedding` 以及 `flat_indices` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 95-98: `SparsePooler.load_weights` loader for weights
```python
    def load_weights(self, state_dict: dict):
        """Load weights from state dict (called by the model)."""
        self.sparse_linear.load_state_dict(state_dict)
        self._weights_loaded = True
```
**EN:** This block defines `SparsePooler.load_weights` and contains the main logic for this step. It mainly invokes `self.sparse_linear.load_state_dict`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self._weights_loaded` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `SparsePooler.load_weights`，并承载这一阶段的核心逻辑。 它主要调用 `self.sparse_linear.load_state_dict`，说明该流程会编排底层辅助函数或计算内核。 像 `self._weights_loaded` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `SparseEmbeddingOutput` and `SparsePooler`. / **主要符号**：核心入口包括 `SparseEmbeddingOutput` 和 `SparsePooler`。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Activation math**: Captures fused nonlinear transforms and their device-specific fast paths. / **激活数学**：描述融合非线性变换及其设备专用快速路径。
- **Projection layers**: Focuses on matrix multiplication, weight layout, and projection-oriented wrappers. / **投影层**：关注矩阵乘法、权重布局与面向投影的包装层。
- **Embedding pipeline**: Describes how IDs or features are mapped into model-space tensors. / **嵌入流水线**：说明 ID 或特征如何映射到模型空间张量。

## Dependencies / 依赖关系
- **Standard library**: `dataclasses.dataclass` / **标准库**：`dataclasses.dataclass`
- **Third-party**: `torch`, `torch.nn`, `torch.nn.functional`, and `transformers.PretrainedConfig` / **第三方依赖**：`torch`、`torch.nn`、`torch.nn.functional` 以及 `transformers.PretrainedConfig`
- **Internal SGLang modules**: `sglang.srt.model_executor.model_runner.ForwardBatch` / **SGLang 内部模块**：`sglang.srt.model_executor.model_runner.ForwardBatch`
