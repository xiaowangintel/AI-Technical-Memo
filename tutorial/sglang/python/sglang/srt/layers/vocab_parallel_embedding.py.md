# vocab_parallel_embedding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/vocab_parallel_embedding.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements embedding lookup and preprocessing for the SGLang SRT runtime. It exposes symbols such as `pad_vocab_size`, `vocab_range_from_per_partition_vocab_size`, `vocab_range_from_global_vocab_size`, and `VocabParallelEmbeddingShardIndices` and connects them to backend-specific paths such as `NPU` and `CPU`. / 该模块为 SGLang 的 SRT 运行时实现了嵌入查找与预处理。它提供了 `pad_vocab_size`、`vocab_range_from_per_partition_vocab_size`、`vocab_range_from_global_vocab_size` 以及 `VocabParallelEmbeddingShardIndices` 等符号，并把这些符号连接到 `NPU` 和 `CPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: File header and module overview
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from https://github.com/vllm-project/vllm/blob/v0.6.3.post1/vllm/model_executor/layers/vocab_parallel_embedding.py
```
**EN:** This opening block contains comments, licensing text, or other context that frames the rest of the file before executable code begins.
**CN:** 这一开头部分包含注释、许可证文本或其他上下文信息，用来为后续可执行代码建立背景。

### Lines 5-53: Imports, constants, and runtime setup
```python
import logging
from dataclasses import dataclass
from typing import List, Optional, Sequence, Tuple

import torch
from torch.nn.parameter import Parameter, UninitializedParameter

from sglang.srt.distributed import (
    divide,
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
    get_tp_group,
    tensor_model_parallel_all_reduce,
)
from sglang.srt.distributed.device_communicators.pynccl_allocator import (
    use_symmetric_memory,
)
from sglang.srt.layers.amx_utils import PackWeightMethod
from sglang.srt.layers.communicator import get_attn_tp_context
from sglang.srt.layers.dp_attention import (
    attn_tp_all_reduce,
    get_attention_tp_rank,
    get_attention_tp_size,
    is_allocation_symmetric,
)
from sglang.srt.layers.parameter import BasevLLMParameter
from sglang.srt.layers.quantization.base_config import (
    QuantizationConfig,
    QuantizeMethodBase,
    method_has_implemented_embedding,
)
from sglang.srt.layers.quantization.unquant import UnquantizedEmbeddingMethod
from sglang.srt.utils import (
    cpu_has_amx_support,
    get_compiler_backend,
    is_cpu,
    is_npu,
    set_weight_attrs,
)

DEFAULT_VOCAB_PADDING_SIZE = 64

_is_cpu_amx_available = cpu_has_amx_support()
_is_cpu = is_cpu()
_is_npu = is_npu()

logger = logging.getLogger(__name__)
```
**EN:** This section prepares the module namespace. It imports `logging`, `dataclasses.dataclass`, `typing.List`, `typing.Optional`, `typing.Sequence`, and `typing.Tuple`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `DEFAULT_VOCAB_PADDING_SIZE`, `_is_cpu_amx_available`, `_is_cpu`, `_is_npu`, and `logger` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `logging`、`dataclasses.dataclass`、`typing.List`、`typing.Optional`、`typing.Sequence` 以及 `typing.Tuple`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `DEFAULT_VOCAB_PADDING_SIZE`、`_is_cpu_amx_available`、`_is_cpu`、`_is_npu` 以及 `logger` 这样的共享名称用于保存配置、缓存句柄或特性开关。

### Lines 54-58: Function `pad_vocab_size` and its core logic
```python
def pad_vocab_size(vocab_size: int, pad_to: int = DEFAULT_VOCAB_PADDING_SIZE) -> int:
    """Pad the vocab size to the given value."""
    return ((vocab_size + pad_to - 1) // pad_to) * pad_to
```
**EN:** This block defines `pad_vocab_size` and contains the main logic for this step. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `pad_vocab_size`，并承载这一阶段的核心逻辑。 该函数最终会把计算结果返回给调用方。

### Lines 59-66: Function `vocab_range_from_per_partition_vocab_size` and its core logic
```python
def vocab_range_from_per_partition_vocab_size(
    per_partition_vocab_size: int, rank: int, offset: int = 0
) -> Sequence[int]:
    index_f = rank * per_partition_vocab_size
    index_l = index_f + per_partition_vocab_size
    return index_f + offset, index_l + offset
```
**EN:** This block defines `vocab_range_from_per_partition_vocab_size` and contains the main logic for this step. Intermediate names such as `index_f` and `index_l` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `vocab_range_from_per_partition_vocab_size`，并承载这一阶段的核心逻辑。 像 `index_f` 和 `index_l` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 67-75: Function `vocab_range_from_global_vocab_size` and its core logic
```python
def vocab_range_from_global_vocab_size(
    global_vocab_size: int, rank: int, world_size: int, offset: int = 0
) -> Sequence[int]:
    per_partition_vocab_size = divide(global_vocab_size, world_size)
    return vocab_range_from_per_partition_vocab_size(
        per_partition_vocab_size, rank, offset=offset
    )
```
**EN:** This block defines `vocab_range_from_global_vocab_size` and contains the main logic for this step. It mainly invokes `divide` and `vocab_range_from_per_partition_vocab_size`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `per_partition_vocab_size` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `vocab_range_from_global_vocab_size`，并承载这一阶段的核心逻辑。 它主要调用 `divide` 和 `vocab_range_from_per_partition_vocab_size`，说明该流程会编排底层辅助函数或计算内核。 像 `per_partition_vocab_size` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 76-89: Class `VocabParallelEmbeddingShardIndices` declaration and shared state
```python
@dataclass
class VocabParallelEmbeddingShardIndices:
    """Indices for a shard of a vocab parallel embedding."""

    padded_org_vocab_start_index: int
    padded_org_vocab_end_index: int
    padded_added_vocab_start_index: int
    padded_added_vocab_end_index: int

    org_vocab_start_index: int
    org_vocab_end_index: int
    added_vocab_start_index: int
    added_vocab_end_index: int
```
**EN:** This block introduces class `VocabParallelEmbeddingShardIndices` and the state shared by its methods. The class docstring summarizes its role: Indices for a shard of a vocab parallel embedding.
**CN:** 该代码块引入类 `VocabParallelEmbeddingShardIndices`，并定义其方法共享的状态。 类级文档进一步概括了它的职责。

### Lines 90-93: Function `VocabParallelEmbeddingShardIndices.num_org_elements` and its core logic
```python
    @property
    def num_org_elements(self) -> int:
        return self.org_vocab_end_index - self.org_vocab_start_index
```
**EN:** This block defines `VocabParallelEmbeddingShardIndices.num_org_elements` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `VocabParallelEmbeddingShardIndices.num_org_elements`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 94-97: Function `VocabParallelEmbeddingShardIndices.num_added_elements` and its core logic
```python
    @property
    def num_added_elements(self) -> int:
        return self.added_vocab_end_index - self.added_vocab_start_index
```
**EN:** This block defines `VocabParallelEmbeddingShardIndices.num_added_elements` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `VocabParallelEmbeddingShardIndices.num_added_elements`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 98-101: Function `VocabParallelEmbeddingShardIndices.num_org_elements_padded` and its core logic
```python
    @property
    def num_org_elements_padded(self) -> int:
        return self.padded_org_vocab_end_index - self.padded_org_vocab_start_index
```
**EN:** This block defines `VocabParallelEmbeddingShardIndices.num_org_elements_padded` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `VocabParallelEmbeddingShardIndices.num_org_elements_padded`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 102-105: Function `VocabParallelEmbeddingShardIndices.num_added_elements_padded` and its core logic
```python
    @property
    def num_added_elements_padded(self) -> int:
        return self.padded_added_vocab_end_index - self.padded_added_vocab_start_index
```
**EN:** This block defines `VocabParallelEmbeddingShardIndices.num_added_elements_padded` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `VocabParallelEmbeddingShardIndices.num_added_elements_padded`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 106-109: Function `VocabParallelEmbeddingShardIndices.num_org_vocab_padding` and its core logic
```python
    @property
    def num_org_vocab_padding(self) -> int:
        return self.num_org_elements_padded - self.num_org_elements
```
**EN:** This block defines `VocabParallelEmbeddingShardIndices.num_org_vocab_padding` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `VocabParallelEmbeddingShardIndices.num_org_vocab_padding`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 110-113: Function `VocabParallelEmbeddingShardIndices.num_added_vocab_padding` and its core logic
```python
    @property
    def num_added_vocab_padding(self) -> int:
        return self.num_added_elements_padded - self.num_added_elements
```
**EN:** This block defines `VocabParallelEmbeddingShardIndices.num_added_vocab_padding` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `VocabParallelEmbeddingShardIndices.num_added_vocab_padding`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 114-117: Function `VocabParallelEmbeddingShardIndices.num_elements_padded` and its core logic
```python
    @property
    def num_elements_padded(self) -> int:
        return self.num_org_elements_padded + self.num_added_elements_padded
```
**EN:** This block defines `VocabParallelEmbeddingShardIndices.num_elements_padded` and contains the main logic for this step. Decorators like `property` connect it to registration, caching, tracing, or dispatch hooks. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `VocabParallelEmbeddingShardIndices.num_elements_padded`，并承载这一阶段的核心逻辑。 像 `property` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 该函数最终会把计算结果返回给调用方。

### Lines 118-134: Internal helper `VocabParallelEmbeddingShardIndices.__post_init__`
```python
    def __post_init__(self):
        # sanity checks
        assert self.padded_org_vocab_start_index <= self.padded_org_vocab_end_index
        assert self.padded_added_vocab_start_index <= self.padded_added_vocab_end_index

        assert self.org_vocab_start_index <= self.org_vocab_end_index
        assert self.added_vocab_start_index <= self.added_vocab_end_index

        assert self.org_vocab_start_index <= self.padded_org_vocab_start_index
        assert self.added_vocab_start_index <= self.padded_added_vocab_start_index
        assert self.org_vocab_end_index <= self.padded_org_vocab_end_index
        assert self.added_vocab_end_index <= self.padded_added_vocab_end_index

        assert self.num_org_elements <= self.num_org_elements_padded
        assert self.num_added_elements <= self.num_added_elements_padded
```
**EN:** This block defines `VocabParallelEmbeddingShardIndices.__post_init__` and contains the main logic for this step.
**CN:** 该代码块定义了 `VocabParallelEmbeddingShardIndices.__post_init__`，并承载这一阶段的核心逻辑。

### Lines 135-162: `get_masked_input_and_mask` getter for masked input and mask
```python
@torch.compile(dynamic=True, backend=get_compiler_backend(), disable=_is_npu)
def get_masked_input_and_mask(
    input_: torch.Tensor,
    org_vocab_start_index: int,
    org_vocab_end_index: int,
    num_org_vocab_padding: int,
    added_vocab_start_index: int,
    added_vocab_end_index: int,
) -> Tuple[torch.Tensor, torch.Tensor]:
    # torch.compile will fuse all of the pointwise ops below
    # into a single kernel, making it very fast
    org_vocab_mask = (input_ >= org_vocab_start_index) & (input_ < org_vocab_end_index)
    added_vocab_mask = (input_ >= added_vocab_start_index) & (
        input_ < added_vocab_end_index
    )
    added_offset = (
        added_vocab_start_index
        - (org_vocab_end_index - org_vocab_start_index)
        - num_org_vocab_padding
    )
    valid_offset = (org_vocab_start_index * org_vocab_mask) + (
        added_offset * added_vocab_mask
    )
    vocab_mask = org_vocab_mask | added_vocab_mask
    input_ = vocab_mask * (input_ - valid_offset)
    return input_, ~vocab_mask
```
**EN:** This block defines `get_masked_input_and_mask` and contains the main logic for this step. Decorators like `torch.compile` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `torch.compile` and `get_compiler_backend`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `org_vocab_mask`, `added_vocab_mask`, `added_offset`, `valid_offset`, and `vocab_mask` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `get_masked_input_and_mask`，并承载这一阶段的核心逻辑。 像 `torch.compile` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `torch.compile` 和 `get_compiler_backend`，说明该流程会编排底层辅助函数或计算内核。 像 `org_vocab_mask`、`added_vocab_mask`、`added_offset`、`valid_offset` 以及 `vocab_mask` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 163-201: Class `VocabParallelEmbedding` declaration and shared state
```python
class VocabParallelEmbedding(torch.nn.Module):
    """Embedding parallelized in the vocabulary dimension.

    Adapted from torch.nn.Embedding, note that we pad the vocabulary size to
    make sure it is divisible by the number of model parallel GPUs.

    In order to support various loading methods, we ensure that LoRA-added
    embeddings are always at the end of TP-sharded tensors. In other words,
    we shard base embeddings and LoRA embeddings separately (both padded),
    and place them in the same tensor.
    In this example, we will have the original vocab size = 1010,
    added vocab size = 16 and padding to 64. Therefore, the total
    vocab size with padding will be 1088 (because we first pad 1010 to
    1024, add 16, and then pad to 1088).
    Therefore, the tensor format looks like the following:
    TP1, rank 0 (no sharding):
                            |< --------BASE-------- >|< -BASE PADDING-- >|< -----LORA------ >|< -LORA PADDING-- >|
    corresponding token_id: |  0  |  1  | ... | 1009 |  -1  | ... |  -1  | 1010 | ... | 1015 |  -1  | ... |  -1  |
                     index: |  0  |  1  | ... | 1009 | 1010 | ... | 1023 | 1024 | ... | 1039 | 1040 | ... | 1087 |

    TP2, rank 0:
                            |< --------------------BASE--------------------- >|< -----LORA------ >|< -LORA PADDING- >|
    corresponding token_id: |  0  |  1  |  2  | ... | 497  | 498 | ...  | 511 | 1000 | ... | 1015 |  -1  | ... |  -1 |
                     index: |  0  |  1  |  2  | ... | 497  | 498 | ...  | 511 | 512  | ... | 527  |  520 | ... | 543 |
    TP2, rank 1:
                            |< -----------BASE----------- >|< -BASE PADDING- >|< -----------LORA PADDING----------- >|
    corresponding token_id: | 512 | 513 | 514 | ... | 1009 | -1  | ...  | -1  |  -1  | ... |  -1  | -1  | ... |   -1 |
                     index: |  0  |  1  |  2  | ... | 497  | 498 | ...  | 511 | 512  | ... | 519  | 520 | ... |  543 |

    Args:
        num_embeddings: vocabulary size.
        embedding_dim: size of hidden state.
        params_dtype: type of the parameters.
        org_num_embeddings: original vocabulary size (without LoRA).
        padding_size: padding size for the vocabulary.
        quant_config: quant config for the layer
        prefix: full name of the layer in the state dict
    """  # noqa: E501
```
**EN:** This block introduces class `VocabParallelEmbedding` and the state shared by its methods. It inherits from `torch.nn.Module`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Embedding parallelized in the vocabulary dimension.
**CN:** 该代码块引入类 `VocabParallelEmbedding`，并定义其方法共享的状态。 它继承自 `torch.nn.Module`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 202-318: `VocabParallelEmbedding` initialization and state setup
```python
    def __init__(
        self,
        num_embeddings: int,
        embedding_dim: int,
        *,
        params_dtype: Optional[torch.dtype] = None,
        org_num_embeddings: Optional[int] = None,
        padding_size: int = DEFAULT_VOCAB_PADDING_SIZE,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        enable_tp: bool = True,
        use_attn_tp_group: bool = False,
        use_presharded_weights: bool = False,
    ):
        super().__init__()
        self.quant_config = quant_config

        self.enable_tp = enable_tp
        self.use_attn_tp_group = use_attn_tp_group
        if self.enable_tp:
            if use_attn_tp_group:
                tp_rank = get_attention_tp_rank()
                self.tp_size = get_attention_tp_size()
            else:
                tp_rank = get_tensor_model_parallel_rank()
                self.tp_size = get_tensor_model_parallel_world_size()
        else:
            assert use_attn_tp_group is False
            tp_rank = 0
            self.tp_size = 1

        self.num_embeddings = num_embeddings
        self.org_vocab_size = org_num_embeddings or num_embeddings

        # Support the case where the vocab size is not divisible by the TP size.
        if (
            _is_cpu
            and pad_vocab_size(self.org_vocab_size, padding_size) % self.tp_size != 0
        ):
            padding_size *= self.tp_size
        self.padding_size = padding_size

        num_added_embeddings = num_embeddings - self.org_vocab_size
        self.use_presharded_weights = use_presharded_weights
        if use_presharded_weights:
            assert (
                num_added_embeddings == 0
            ), "Lora is not supported with presharded weights."

        self.org_vocab_size_padded = pad_vocab_size(
            self.org_vocab_size, self.padding_size
        )
        self.num_embeddings_padded = pad_vocab_size(
            self.org_vocab_size_padded + num_added_embeddings, self.padding_size
        )
        assert self.org_vocab_size_padded <= self.num_embeddings_padded

        self.shard_indices = self._get_indices(
            self.num_embeddings_padded,
            self.org_vocab_size_padded,
            self.num_embeddings,
            self.org_vocab_size,
            tp_rank,
            self.tp_size,
        )
        self.embedding_dim = embedding_dim

        quant_method = None
        if quant_config is not None:
            quant_method = quant_config.get_quant_method(self, prefix=prefix)
        if quant_method is None:
            quant_method = UnquantizedEmbeddingMethod()

        # If we are making an embedding layer, then our quantization linear
        # method must implement the embedding operation. If we are another
        # layer type like ParallelLMHead, this is not important.
        is_embedding_layer = type(self.__class__) is VocabParallelEmbedding
        quant_method_implements_embedding = method_has_implemented_embedding(
            type(quant_method)
        )
        if is_embedding_layer and not quant_method_implements_embedding:
            raise NotImplementedError(
                f"The class {type(quant_method).__name__} must implement "
                "the 'embedding' method, see UnquantizedEmbeddingMethod."
            )

        self.quant_method: QuantizeMethodBase = quant_method

        if params_dtype is None:
            params_dtype = torch.get_default_dtype()
        # Divide the weight matrix along the vocaburaly dimension.
        self.num_added_embeddings = self.num_embeddings - self.org_vocab_size
        self.num_embeddings_per_partition = divide(
            self.num_embeddings_padded, self.tp_size
        )
        assert (
            self.shard_indices.num_elements_padded == self.num_embeddings_per_partition
        )
        self.num_org_embeddings_per_partition = (
            self.shard_indices.org_vocab_end_index
            - self.shard_indices.org_vocab_start_index
        )
        self.num_added_embeddings_per_partition = (
            self.shard_indices.added_vocab_end_index
            - self.shard_indices.added_vocab_start_index
        )

        self.quant_method.create_weights(
            self,
            self.embedding_dim,
            [self.num_embeddings_per_partition],
            self.embedding_dim,
            self.num_embeddings_padded,
            params_dtype=params_dtype,
            weight_loader=self.weight_loader,
        )
```
**EN:** This block defines `VocabParallelEmbedding.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `pad_vocab_size`, `self._get_indices`, `method_has_implemented_embedding`, and `divide`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.quant_config`, `self.enable_tp`, `self.use_attn_tp_group`, `self.num_embeddings`, and `self.org_vocab_size` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `VocabParallelEmbedding.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`pad_vocab_size`、`self._get_indices`、`method_has_implemented_embedding` 以及 `divide`，说明该流程会编排底层辅助函数或计算内核。 像 `self.quant_config`、`self.enable_tp`、`self.use_attn_tp_group`、`self.num_embeddings` 以及 `self.org_vocab_size` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 319-356: Internal helper `VocabParallelEmbedding._get_indices`
```python
    @classmethod
    def _get_indices(
        cls,
        vocab_size_padded: int,
        org_vocab_size_padded: int,
        vocab_size: int,
        org_vocab_size: int,
        tp_rank: int,
        tp_size: int,
    ) -> VocabParallelEmbeddingShardIndices:
        """Get start and end indices for vocab parallel embedding, following the
        layout outlined in the class docstring, based on the given tp_rank and
        tp_size."""
        num_added_embeddings_padded = vocab_size_padded - org_vocab_size_padded
        padded_org_vocab_start_index, padded_org_vocab_end_index = (
            vocab_range_from_global_vocab_size(org_vocab_size_padded, tp_rank, tp_size)
        )
        padded_added_vocab_start_index, padded_added_vocab_end_index = (
            vocab_range_from_global_vocab_size(
                num_added_embeddings_padded, tp_rank, tp_size, offset=org_vocab_size
            )
        )
        # remove padding
        org_vocab_start_index = min(padded_org_vocab_start_index, org_vocab_size)
        org_vocab_end_index = min(padded_org_vocab_end_index, org_vocab_size)
        added_vocab_start_index = min(padded_added_vocab_start_index, vocab_size)
        added_vocab_end_index = min(padded_added_vocab_end_index, vocab_size)
        return VocabParallelEmbeddingShardIndices(
            padded_org_vocab_start_index,
            padded_org_vocab_end_index,
            padded_added_vocab_start_index,
            padded_added_vocab_end_index,
            org_vocab_start_index,
            org_vocab_end_index,
            added_vocab_start_index,
            added_vocab_end_index,
        )
```
**EN:** This block defines `VocabParallelEmbedding._get_indices` and contains the main logic for this step. Decorators like `classmethod` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `vocab_range_from_global_vocab_size`, `min`, and `VocabParallelEmbeddingShardIndices`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `num_added_embeddings_padded`, `padded_org_vocab_start_index`, `padded_org_vocab_end_index`, `padded_added_vocab_start_index`, and `padded_added_vocab_end_index` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `VocabParallelEmbedding._get_indices`，并承载这一阶段的核心逻辑。 像 `classmethod` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `vocab_range_from_global_vocab_size`、`min` 以及 `VocabParallelEmbeddingShardIndices`，说明该流程会编排底层辅助函数或计算内核。 像 `num_added_embeddings_padded`、`padded_org_vocab_start_index`、`padded_org_vocab_end_index`、`padded_added_vocab_start_index` 以及 `padded_added_vocab_end_index` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 357-421: `VocabParallelEmbedding.get_sharded_to_full_mapping` getter for sharded to full mapping
```python
    def get_sharded_to_full_mapping(self) -> Optional[List[int]]:
        """Get a mapping that can be used to reindex the gathered
        logits for sampling.

        During sampling, we gather logits from all ranks. The relationship
        of index->token_id will follow the same format as outlined in the class
        docstring. However, after the gather, we want to reindex the final
        logits tensor to map index->token_id one-to-one (the index is always
        equal the token_id it corresponds to). The indices returned by this
        method allow us to do that.
        """
        if self.tp_size < 2:
            return None

        base_embeddings: List[int] = []
        added_embeddings: List[int] = []
        padding: List[int] = []
        for tp_rank in range(self.tp_size):
            shard_indices = self._get_indices(
                self.num_embeddings_padded,
                self.org_vocab_size_padded,
                self.num_embeddings,
                self.org_vocab_size,
                tp_rank,
                self.tp_size,
            )
            range_start = self.num_embeddings_per_partition * tp_rank
            range_end = self.num_embeddings_per_partition * (tp_rank + 1)
            base_embeddings.extend(
                range(range_start, range_start + shard_indices.num_org_elements)
            )
            padding.extend(
                range(
                    range_start + shard_indices.num_org_elements,
                    range_start + shard_indices.num_org_elements_padded,
                )
            )
            added_embeddings.extend(
                range(
                    range_start + shard_indices.num_org_elements_padded,
                    range_start
                    + shard_indices.num_org_elements_padded
                    + shard_indices.num_added_elements,
                )
            )
            padding.extend(
                range(
                    range_start
                    + shard_indices.num_org_elements_padded
                    + shard_indices.num_added_elements,
                    range_start
                    + shard_indices.num_org_elements_padded
                    + shard_indices.num_added_elements_padded,
                )
            )
            assert (
                range_start
                + shard_indices.num_org_elements_padded
                + shard_indices.num_added_elements_padded
                == range_end
            )
        ret = base_embeddings + added_embeddings + padding
        assert len(ret) == self.num_embeddings_padded
        return ret
```
**EN:** This block defines `VocabParallelEmbedding.get_sharded_to_full_mapping` and contains the main logic for this step. It mainly invokes `range`, `self._get_indices`, `base_embeddings.extend`, `padding.extend`, and `added_embeddings.extend`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `base_embeddings`, `added_embeddings`, `padding`, `ret`, and `shard_indices` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `VocabParallelEmbedding.get_sharded_to_full_mapping`，并承载这一阶段的核心逻辑。 它主要调用 `range`、`self._get_indices`、`base_embeddings.extend`、`padding.extend` 以及 `added_embeddings.extend`，说明该流程会编排底层辅助函数或计算内核。 像 `base_embeddings`、`added_embeddings`、`padding`、`ret` 以及 `shard_indices` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 422-472: Function `VocabParallelEmbedding.weight_loader` and its core logic
```python
    def weight_loader(self, param: Parameter, loaded_weight: torch.Tensor):
        output_dim = getattr(param, "output_dim", None)
        packed_dim = getattr(param, "packed_dim", None)

        # If the parameter is a gguf weight, then load it directly.
        if getattr(param, "is_gguf_weight_type", None):
            param.data.copy_(loaded_weight)
            param.weight_type = loaded_weight.item()
            return
        elif isinstance(param, UninitializedParameter):
            shape = list(loaded_weight.shape)
            if output_dim is not None:
                shape[output_dim] = shape[output_dim] // self.tp_size
            param.materialize(tuple(shape), dtype=loaded_weight.dtype)

        # If parameter does not have output dim, then it should
        # be copied onto all gpus (e.g. g_idx for act_order gptq).
        if output_dim is None:
            assert param.data.shape == loaded_weight.shape
            param.data.copy_(loaded_weight)
            return

        # Shard indexes for loading the weight
        start_idx = self.shard_indices.org_vocab_start_index
        shard_size = self.shard_indices.org_vocab_end_index - start_idx

        # If param packed on the same dim we are sharding on, then
        # need to adjust offsets of loaded weight by pack_factor.
        if packed_dim is not None and packed_dim == output_dim:
            packed_factor = (
                param.packed_factor
                if isinstance(param, BasevLLMParameter)
                else param.packed_factor
            )
            assert loaded_weight.shape[output_dim] == (
                self.org_vocab_size // param.packed_factor
            )
            start_idx = start_idx // packed_factor
            shard_size = shard_size // packed_factor
        else:
            assert loaded_weight.shape[output_dim] == (
                self.org_vocab_size
                // (self.tp_size if self.use_presharded_weights else 1)
            ), f"{self.org_vocab_size=} {self.use_presharded_weights=} {loaded_weight.shape[output_dim]=}"

        # Copy the data.
        if not self.use_presharded_weights:
            loaded_weight = loaded_weight.narrow(output_dim, start_idx, shard_size)
        param[: loaded_weight.shape[0]].data.copy_(loaded_weight)
        param[loaded_weight.shape[0] :].data.fill_(0)
```
**EN:** This block defines `VocabParallelEmbedding.weight_loader` and contains the main logic for this step. It mainly invokes `getattr`, `param.data.copy_`, `param.data.fill_`, `loaded_weight.item`, and `isinstance`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `output_dim`, `packed_dim`, `start_idx`, `shard_size`, and `packed_factor` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `VocabParallelEmbedding.weight_loader`，并承载这一阶段的核心逻辑。 它主要调用 `getattr`、`param.data.copy_`、`param.data.fill_`、`loaded_weight.item` 以及 `isinstance`，说明该流程会编排底层辅助函数或计算内核。 像 `output_dim`、`packed_dim`、`start_idx`、`shard_size` 以及 `packed_factor` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 473-503: `VocabParallelEmbedding.forward` main forward path
```python
    def forward(self, input_):
        if self.tp_size > 1:
            # Build the mask.
            masked_input, input_mask = get_masked_input_and_mask(
                input_,
                self.shard_indices.org_vocab_start_index,
                self.shard_indices.org_vocab_end_index,
                self.shard_indices.num_org_vocab_padding,
                self.shard_indices.added_vocab_start_index,
                self.shard_indices.added_vocab_end_index,
            )
        else:
            masked_input = input_

        # Get the embeddings.
        with use_symmetric_memory(
            get_tp_group(), disabled=not is_allocation_symmetric()
        ):
            output_parallel = self.quant_method.embedding(self, masked_input.long())

        if self.tp_size > 1:
            # Mask the output embedding.
            output_parallel.masked_fill_(input_mask.unsqueeze(-1), 0)
            if not get_attn_tp_context().input_scattered:
                if self.use_attn_tp_group:
                    output_parallel = attn_tp_all_reduce(output_parallel)
                else:
                    # Reduce across all the model parallel GPUs.
                    output_parallel = tensor_model_parallel_all_reduce(output_parallel)
        return output_parallel
```
**EN:** This block defines `VocabParallelEmbedding.forward` and contains the main logic for this step. It mainly invokes `get_masked_input_and_mask`, `use_symmetric_memory`, `self.quant_method.embedding`, `output_parallel.masked_fill_`, and `get_tp_group`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `masked_input`, `input_mask`, and `output_parallel` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `VocabParallelEmbedding.forward`，并承载这一阶段的核心逻辑。 它主要调用 `get_masked_input_and_mask`、`use_symmetric_memory`、`self.quant_method.embedding`、`output_parallel.masked_fill_` 以及 `get_tp_group`，说明该流程会编排底层辅助函数或计算内核。 像 `masked_input`、`input_mask` 以及 `output_parallel` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 504-513: Function `VocabParallelEmbedding.extra_repr` and its core logic
```python
    def extra_repr(self) -> str:
        s = f"num_embeddings={self.num_embeddings_per_partition}"
        s += f", embedding_dim={self.embedding_dim}"
        s += f", org_vocab_size={self.org_vocab_size}"
        s += f", num_embeddings_padded={self.num_embeddings_padded}"
        if self.enable_tp:
            s += f", tp_size={self.tp_size}"
        return s
```
**EN:** This block defines `VocabParallelEmbedding.extra_repr` and contains the main logic for this step. Intermediate names such as `s` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `VocabParallelEmbedding.extra_repr`，并承载这一阶段的核心逻辑。 像 `s` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 514-529: Class `ParallelLMHead` declaration and shared state
```python
class ParallelLMHead(VocabParallelEmbedding):
    """Parallelized LM head.

    Output logits weight matrices used in the Sampler. The weight and bias
    tensors are padded to make sure they are divisible by the number of
    model parallel GPUs.

    Args:
        num_embeddings: vocabulary size.
        embedding_dim: size of hidden state.
        bias: whether to use bias.
        params_dtype: type of the parameters.
        org_num_embeddings: original vocabulary size (without LoRA).
        padding_size: padding size for the vocabulary.
    """
```
**EN:** This block introduces class `ParallelLMHead` and the state shared by its methods. It inherits from `VocabParallelEmbedding`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Parallelized LM head.
**CN:** 该代码块引入类 `ParallelLMHead`，并定义其方法共享的状态。 它继承自 `VocabParallelEmbedding`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 530-578: `ParallelLMHead` initialization and state setup
```python
    def __init__(
        self,
        num_embeddings: int,
        embedding_dim: int,
        *,
        bias: bool = False,
        params_dtype: Optional[torch.dtype] = None,
        org_num_embeddings: Optional[int] = None,
        padding_size: int = DEFAULT_VOCAB_PADDING_SIZE,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        use_attn_tp_group: bool = False,
        use_presharded_weights: bool = False,
    ):
        super().__init__(
            num_embeddings,
            embedding_dim,
            params_dtype=params_dtype,
            org_num_embeddings=org_num_embeddings,
            padding_size=padding_size,
            quant_config=quant_config,
            prefix=prefix,
            use_attn_tp_group=use_attn_tp_group,
            use_presharded_weights=use_presharded_weights,
        )
        self.quant_config = quant_config

        # We only support pack LMHead if it's not quantized.
        if _is_cpu and _is_cpu_amx_available:
            if hasattr(self, "weight") and self.weight.dtype in [
                torch.bfloat16,
                torch.float16,
            ]:
                self.quant_method = PackWeightMethod(weight_names=["weight"])

        if bias:
            self.bias = Parameter(
                torch.empty(self.num_embeddings_per_partition, dtype=params_dtype)
            )
            set_weight_attrs(
                self.bias,
                {
                    "output_dim": 0,
                    "weight_loader": self.weight_loader,
                },
            )
        else:
            self.register_parameter("bias", None)
```
**EN:** This block defines `ParallelLMHead.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `Parameter`, `set_weight_attrs`, `self.register_parameter`, and `hasattr`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.quant_config`, `self.bias`, and `self.quant_method` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `ParallelLMHead.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`Parameter`、`set_weight_attrs`、`self.register_parameter` 以及 `hasattr`，说明该流程会编排底层辅助函数或计算内核。 像 `self.quant_config`、`self.bias` 以及 `self.quant_method` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 579-587: Function `ParallelLMHead.tie_weights` and its core logic
```python
    def tie_weights(self, embed_tokens: VocabParallelEmbedding):
        """Tie the weights with word embeddings."""
        # GGUF quantized embed_tokens.
        if self.quant_config and self.quant_config.get_name() == "gguf":
            return embed_tokens
        else:
            self.weight = embed_tokens.weight
            return self
```
**EN:** This block defines `ParallelLMHead.tie_weights` and contains the main logic for this step. It mainly invokes `self.quant_config.get_name`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.weight` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `ParallelLMHead.tie_weights`，并承载这一阶段的核心逻辑。 它主要调用 `self.quant_config.get_name`，说明该流程会编排底层辅助函数或计算内核。 像 `self.weight` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 588-590: `ParallelLMHead.forward` main forward path
```python
    def forward(self, input_):
        del input_
        raise RuntimeError("LMHead's weights should be used in the sampler.")
```
**EN:** This block defines `ParallelLMHead.forward` and contains the main logic for this step. It mainly invokes `RuntimeError`, showing how the routine orchestrates lower-level helpers or kernels.
**CN:** 该代码块定义了 `ParallelLMHead.forward`，并承载这一阶段的核心逻辑。 它主要调用 `RuntimeError`，说明该流程会编排底层辅助函数或计算内核。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `pad_vocab_size`, `vocab_range_from_per_partition_vocab_size`, `vocab_range_from_global_vocab_size`, `VocabParallelEmbeddingShardIndices`, and `get_masked_input_and_mask`. / **主要符号**：核心入口包括 `pad_vocab_size`、`vocab_range_from_per_partition_vocab_size`、`vocab_range_from_global_vocab_size`、`VocabParallelEmbeddingShardIndices` 以及 `get_masked_input_and_mask`。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Position encoding**: Describes how rotary embeddings or related position transforms are applied. / **位置编码**：说明如何应用旋转位置编码或相关位置变换。

## Dependencies / 依赖关系
- **Standard library**: `logging`, `dataclasses.dataclass`, `typing.List`, `typing.Optional`, `typing.Sequence`, and `typing.Tuple` / **标准库**：`logging`、`dataclasses.dataclass`、`typing.List`、`typing.Optional`、`typing.Sequence` 以及 `typing.Tuple`
- **Third-party**: `torch`, `torch.nn.parameter.Parameter`, and `torch.nn.parameter.UninitializedParameter` / **第三方依赖**：`torch`、`torch.nn.parameter.Parameter` 以及 `torch.nn.parameter.UninitializedParameter`
- **Internal SGLang modules**: `sglang.srt.distributed.divide`, `sglang.srt.distributed.get_tensor_model_parallel_rank`, `sglang.srt.distributed.get_tensor_model_parallel_world_size`, `sglang.srt.distributed.get_tp_group`, `sglang.srt.distributed.tensor_model_parallel_all_reduce`, `sglang.srt.distributed.device_communicators.pynccl_allocator.use_symmetric_memory`, `sglang.srt.layers.amx_utils.PackWeightMethod`, `sglang.srt.layers.communicator.get_attn_tp_context`, `sglang.srt.layers.dp_attention.attn_tp_all_reduce`, `sglang.srt.layers.dp_attention.get_attention_tp_rank`, `sglang.srt.layers.dp_attention.get_attention_tp_size`, and `sglang.srt.layers.dp_attention.is_allocation_symmetric` / **SGLang 内部模块**：`sglang.srt.distributed.divide`、`sglang.srt.distributed.get_tensor_model_parallel_rank`、`sglang.srt.distributed.get_tensor_model_parallel_world_size`、`sglang.srt.distributed.get_tp_group`、`sglang.srt.distributed.tensor_model_parallel_all_reduce`、`sglang.srt.distributed.device_communicators.pynccl_allocator.use_symmetric_memory`、`sglang.srt.layers.amx_utils.PackWeightMethod`、`sglang.srt.layers.communicator.get_attn_tp_context`、`sglang.srt.layers.dp_attention.attn_tp_all_reduce`、`sglang.srt.layers.dp_attention.get_attention_tp_rank`、`sglang.srt.layers.dp_attention.get_attention_tp_size` 以及 `sglang.srt.layers.dp_attention.is_allocation_symmetric`
