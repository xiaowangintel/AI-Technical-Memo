# vocab_parallel_embedding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/vocab_parallel_embedding.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `UnquantizedEmbeddingMethod`, `VocabParallelEmbeddingShardIndices`, and `VocabParallelEmbedding`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `UnquantizedEmbeddingMethod`、`VocabParallelEmbeddingShardIndices` 和 `VocabParallelEmbedding` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 5-26: module setup and imports / 模块初始化与导入
```python
from collections.abc import Sequence
from dataclasses import dataclass

import torch
import torch.distributed as dist
import torch.nn.functional as F
from torch.nn.parameter import Parameter, UninitializedParameter

from sglang.multimodal_gen.runtime.distributed import (
    divide,
    get_tp_group,
    tensor_model_parallel_all_reduce,
)
from sglang.multimodal_gen.runtime.layers.quantization.configs.base_config import (
    QuantizationConfig,
    QuantizeMethodBase,
    method_has_implemented_embedding,
)
from sglang.multimodal_gen.runtime.layers.utils import get_group_rank, get_group_size
from sglang.multimodal_gen.runtime.models.parameter import BasevLLMParameter
from sglang.multimodal_gen.runtime.models.utils import set_weight_attrs
from sglang.multimodal_gen.runtime.platforms import current_platform
```
**EN:** This block establishes the module context and imports `collections.abc`, `dataclasses`, `torch`, `torch.distributed`, `torch.nn.functional`, and `torch.nn.parameter`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `collections.abc`、`dataclasses`、`torch`、`torch.distributed`、`torch.nn.functional` 和 `torch.nn.parameter`。这些依赖为后续实现提供所需符号。

### Lines 28-28: supporting statements / 辅助语句
```python
DEFAULT_VOCAB_PADDING_SIZE = 64
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `DEFAULT_VOCAB_PADDING_SIZE`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `DEFAULT_VOCAB_PADDING_SIZE` 等名称。

### Lines 31-33: `UnquantizedEmbeddingMethod` class overview / `UnquantizedEmbeddingMethod` 类概览
```python
class UnquantizedEmbeddingMethod(QuantizeMethodBase):
    """Unquantized method for embeddings."""
```
**EN:** This block defines class `UnquantizedEmbeddingMethod`. Unquantized method for embeddings. It inherits from `QuantizeMethodBase`.
**CN:** 该代码块定义了类 `UnquantizedEmbeddingMethod`。 它用于封装 unquantized embedding method 相关行为。 它继承自 `QuantizeMethodBase`。

### Lines 34-56: `create_weights` implementation / `create_weights` 实现
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: list[int],
        input_size: int,
        output_size: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        """Create weights for embedding layer."""

        weight = Parameter(
            torch.empty(
                sum(output_partition_sizes),
                input_size_per_partition,
                dtype=params_dtype,
            ),
            requires_grad=False,
        )
        set_weight_attrs(weight, {"input_dim": 1, "output_dim": 0})
        layer.register_parameter("weight", weight)
        set_weight_attrs(weight, extra_weight_attrs)
```
**EN:** This block defines method `create_weights` on `UnquantizedEmbeddingMethod`. Create weights for embedding layer. Key calls include `Parameter`, `set_weight_attrs`, `layer.register_parameter`, `torch.empty`, and `sum`. Parameters such as `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, and `output_size` drive the behavior in this section.
**CN:** 该代码块定义了 `UnquantizedEmbeddingMethod` 的方法 `create_weights`。 它用于创建weights。 关键调用包括 `Parameter`、`set_weight_attrs`、`layer.register_parameter`、`torch.empty` 和 `sum`。 本段逻辑主要由 `layer`、`input_size_per_partition`、`output_partition_sizes`、`input_size` 和 `output_size` 等参数驱动。

### Lines 58-61: `apply` implementation / `apply` 实现
```python
    def apply(
        self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None = None
    ) -> torch.Tensor:
        return F.linear(x, layer.weight, bias)
```
**EN:** This block defines method `apply` on `UnquantizedEmbeddingMethod`. It applies function. Key calls include `F.linear`. Parameters such as `layer`, `x`, and `bias` drive the behavior in this section.
**CN:** 该代码块定义了 `UnquantizedEmbeddingMethod` 的方法 `apply`。 它用于应用函数。 关键调用包括 `F.linear`。 本段逻辑主要由 `layer`、`x` 和 `bias` 等参数驱动。

### Lines 63-64: `embedding` implementation / `embedding` 实现
```python
    def embedding(self, layer: torch.nn.Module, input_: torch.Tensor) -> torch.Tensor:
        return F.embedding(input_, layer.weight)
```
**EN:** This block defines method `embedding` on `UnquantizedEmbeddingMethod`. It handles embedding logic. Key calls include `F.embedding`. Parameters such as `layer`, and `input_` drive the behavior in this section.
**CN:** 该代码块定义了 `UnquantizedEmbeddingMethod` 的方法 `embedding`。 它用于处理 embedding 相关逻辑。 关键调用包括 `F.embedding`。 本段逻辑主要由 `layer` 和 `input_` 等参数驱动。

### Lines 67-69: `pad_vocab_size` implementation / `pad_vocab_size` 实现
```python
def pad_vocab_size(vocab_size: int, pad_to: int = DEFAULT_VOCAB_PADDING_SIZE) -> int:
    """Pad the vocab size to the given value."""
    return ((vocab_size + pad_to - 1) // pad_to) * pad_to
```
**EN:** This block defines function `pad_vocab_size`. Pad the vocab size to the given value. Parameters such as `vocab_size`, and `pad_to` drive the behavior in this section.
**CN:** 该代码块定义了函数 `pad_vocab_size`。 它用于处理 pad vocab size 相关逻辑。 本段逻辑主要由 `vocab_size` 和 `pad_to` 等参数驱动。

### Lines 72-77: `vocab_range_from_per_partition_vocab_size` implementation / `vocab_range_from_per_partition_vocab_size` 实现
```python
def vocab_range_from_per_partition_vocab_size(
    per_partition_vocab_size: int, rank: int, offset: int = 0
) -> Sequence[int]:
    index_f = rank * per_partition_vocab_size
    index_l = index_f + per_partition_vocab_size
    return index_f + offset, index_l + offset
```
**EN:** This block defines function `vocab_range_from_per_partition_vocab_size`. It handles vocab range from per partition vocab size logic. Parameters such as `per_partition_vocab_size`, `rank`, and `offset` drive the behavior in this section.
**CN:** 该代码块定义了函数 `vocab_range_from_per_partition_vocab_size`。 它用于处理 vocab range from per partition vocab size 相关逻辑。 本段逻辑主要由 `per_partition_vocab_size`、`rank` 和 `offset` 等参数驱动。

### Lines 80-86: `vocab_range_from_global_vocab_size` implementation / `vocab_range_from_global_vocab_size` 实现
```python
def vocab_range_from_global_vocab_size(
    global_vocab_size: int, rank: int, world_size: int, offset: int = 0
) -> Sequence[int]:
    per_partition_vocab_size = divide(global_vocab_size, world_size)
    return vocab_range_from_per_partition_vocab_size(
        per_partition_vocab_size, rank, offset=offset
    )
```
**EN:** This block defines function `vocab_range_from_global_vocab_size`. It handles vocab range from global vocab size logic. Key calls include `divide`, and `vocab_range_from_per_partition_vocab_size`. Parameters such as `global_vocab_size`, `rank`, `world_size`, and `offset` drive the behavior in this section.
**CN:** 该代码块定义了函数 `vocab_range_from_global_vocab_size`。 它用于处理 vocab range from global vocab size 相关逻辑。 关键调用包括 `divide` 和 `vocab_range_from_per_partition_vocab_size`。 本段逻辑主要由 `global_vocab_size`、`rank`、`world_size` 和 `offset` 等参数驱动。

### Lines 90-92: `VocabParallelEmbeddingShardIndices` class overview / `VocabParallelEmbeddingShardIndices` 类概览
```python
class VocabParallelEmbeddingShardIndices:
    """Indices for a shard of a vocab parallel embedding."""
```
**EN:** This block defines class `VocabParallelEmbeddingShardIndices`. Indices for a shard of a vocab parallel embedding.
**CN:** 该代码块定义了类 `VocabParallelEmbeddingShardIndices`。 它用于封装 vocab parallel embedding shard indices 相关行为。

### Lines 93-101: supporting statements / 辅助语句
```python
    padded_org_vocab_start_index: int
    padded_org_vocab_end_index: int
    padded_added_vocab_start_index: int
    padded_added_vocab_end_index: int

    org_vocab_start_index: int
    org_vocab_end_index: int
    added_vocab_start_index: int
    added_vocab_end_index: int
```
**EN:** This block gathers supporting statements inside `VocabParallelEmbeddingShardIndices`. It updates names such as `padded_org_vocab_start_index`, `padded_org_vocab_end_index`, `padded_added_vocab_start_index`, `padded_added_vocab_end_index`, `org_vocab_start_index`, and `org_vocab_end_index`.
**CN:** 该代码块汇集了位于 `VocabParallelEmbeddingShardIndices` 内部的辅助语句。 它会更新 `padded_org_vocab_start_index`、`padded_org_vocab_end_index`、`padded_added_vocab_start_index`、`padded_added_vocab_end_index`、`org_vocab_start_index` 和 `org_vocab_end_index` 等名称。

### Lines 103-105: `num_org_elements` implementation / `num_org_elements` 实现
```python
    @property
    def num_org_elements(self) -> int:
        return self.org_vocab_end_index - self.org_vocab_start_index
```
**EN:** This block defines method `num_org_elements` on `VocabParallelEmbeddingShardIndices`. It handles num org elements logic.
**CN:** 该代码块定义了 `VocabParallelEmbeddingShardIndices` 的方法 `num_org_elements`。 它用于处理 num org elements 相关逻辑。

### Lines 107-109: `num_added_elements` implementation / `num_added_elements` 实现
```python
    @property
    def num_added_elements(self) -> int:
        return self.added_vocab_end_index - self.added_vocab_start_index
```
**EN:** This block defines method `num_added_elements` on `VocabParallelEmbeddingShardIndices`. It handles num added elements logic.
**CN:** 该代码块定义了 `VocabParallelEmbeddingShardIndices` 的方法 `num_added_elements`。 它用于处理 num added elements 相关逻辑。

### Lines 111-113: `num_org_elements_padded` implementation / `num_org_elements_padded` 实现
```python
    @property
    def num_org_elements_padded(self) -> int:
        return self.padded_org_vocab_end_index - self.padded_org_vocab_start_index
```
**EN:** This block defines method `num_org_elements_padded` on `VocabParallelEmbeddingShardIndices`. It handles num org elements padded logic.
**CN:** 该代码块定义了 `VocabParallelEmbeddingShardIndices` 的方法 `num_org_elements_padded`。 它用于处理 num org elements padded 相关逻辑。

### Lines 115-117: `num_added_elements_padded` implementation / `num_added_elements_padded` 实现
```python
    @property
    def num_added_elements_padded(self) -> int:
        return self.padded_added_vocab_end_index - self.padded_added_vocab_start_index
```
**EN:** This block defines method `num_added_elements_padded` on `VocabParallelEmbeddingShardIndices`. It handles num added elements padded logic.
**CN:** 该代码块定义了 `VocabParallelEmbeddingShardIndices` 的方法 `num_added_elements_padded`。 它用于处理 num added elements padded 相关逻辑。

### Lines 119-121: `num_org_vocab_padding` implementation / `num_org_vocab_padding` 实现
```python
    @property
    def num_org_vocab_padding(self) -> int:
        return self.num_org_elements_padded - self.num_org_elements
```
**EN:** This block defines method `num_org_vocab_padding` on `VocabParallelEmbeddingShardIndices`. It handles num org vocab padding logic.
**CN:** 该代码块定义了 `VocabParallelEmbeddingShardIndices` 的方法 `num_org_vocab_padding`。 它用于处理 num org vocab padding 相关逻辑。

### Lines 123-125: `num_added_vocab_padding` implementation / `num_added_vocab_padding` 实现
```python
    @property
    def num_added_vocab_padding(self) -> int:
        return self.num_added_elements_padded - self.num_added_elements
```
**EN:** This block defines method `num_added_vocab_padding` on `VocabParallelEmbeddingShardIndices`. It handles num added vocab padding logic.
**CN:** 该代码块定义了 `VocabParallelEmbeddingShardIndices` 的方法 `num_added_vocab_padding`。 它用于处理 num added vocab padding 相关逻辑。

### Lines 127-129: `num_elements_padded` implementation / `num_elements_padded` 实现
```python
    @property
    def num_elements_padded(self) -> int:
        return self.num_org_elements_padded + self.num_added_elements_padded
```
**EN:** This block defines method `num_elements_padded` on `VocabParallelEmbeddingShardIndices`. It handles num elements padded logic.
**CN:** 该代码块定义了 `VocabParallelEmbeddingShardIndices` 的方法 `num_elements_padded`。 它用于处理 num elements padded 相关逻辑。

### Lines 131-145: `__post_init__` implementation / `__post_init__` 实现
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
**EN:** This block defines method `__post_init__` on `VocabParallelEmbeddingShardIndices`. It post-processes init.
**CN:** 该代码块定义了 `VocabParallelEmbeddingShardIndices` 的方法 `__post_init__`。 它用于后处理init。

### Lines 148-177: `get_masked_input_and_mask` implementation / `get_masked_input_and_mask` 实现
```python
@torch.compile(
    dynamic=True,
    backend=current_platform.simple_compile_backend,
    disable=current_platform.is_npu(),
)
def get_masked_input_and_mask(
    input_: torch.Tensor,
    org_vocab_start_index: int,
    org_vocab_end_index: int,
    num_org_vocab_padding: int,
    added_vocab_start_index: int,
    added_vocab_end_index: int,
) -> tuple[torch.Tensor, torch.Tensor]:
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
**EN:** This block defines function `get_masked_input_and_mask`. It retrieves masked input and mask. Key calls include `torch.compile`, and `current_platform.is_npu`. Parameters such as `input_`, `org_vocab_start_index`, `org_vocab_end_index`, `num_org_vocab_padding`, and `added_vocab_start_index` drive the behavior in this section.
**CN:** 该代码块定义了函数 `get_masked_input_and_mask`。 它用于获取masked input and mask。 关键调用包括 `torch.compile` 和 `current_platform.is_npu`。 本段逻辑主要由 `input_`、`org_vocab_start_index`、`org_vocab_end_index`、`num_org_vocab_padding` 和 `added_vocab_start_index` 等参数驱动。

### Lines 180-218: `VocabParallelEmbedding` class overview / `VocabParallelEmbedding` 类概览
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
**EN:** This block defines class `VocabParallelEmbedding`. Embedding parallelized in the vocabulary dimension. Adapted from torch.nn.Embedding, note that we pad the vocabulary size to make sure it is divisible by the number of model parallel GPUs. It inherits from `torch.nn.Module`.
**CN:** 该代码块定义了类 `VocabParallelEmbedding`。 它用于封装 vocab parallel embedding 相关行为。 它继承自 `torch.nn.Module`。

### Lines 219-307: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        num_embeddings: int,
        embedding_dim: int,
        params_dtype: torch.dtype | None = None,
        org_num_embeddings: int | None = None,
        padding_size: int = DEFAULT_VOCAB_PADDING_SIZE,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
        tp_group: dist.ProcessGroup = None,
    ):
        super().__init__()

        # Keep the input dimensions.
        tp_group = tp_group or get_tp_group()
        tp_rank = get_group_rank(tp_group)
        self.tp_size = get_group_size(tp_group)
        self.tp_group = tp_group
        self.num_embeddings = num_embeddings
        self.padding_size = padding_size
        self.org_vocab_size = org_num_embeddings or num_embeddings
        num_added_embeddings = num_embeddings - self.org_vocab_size
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
**EN:** This block defines method `__init__` on `VocabParallelEmbedding`. It initializes the instance state. Key calls include `super.__init__`, `get_group_rank`, `get_group_size`, `pad_vocab_size`, and `self._get_indices`. The implementation branches on conditions. Parameters such as `num_embeddings`, `embedding_dim`, `params_dtype`, `org_num_embeddings`, and `padding_size` drive the behavior in this section.
**CN:** 该代码块定义了 `VocabParallelEmbedding` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `super.__init__`、`get_group_rank`、`get_group_size`、`pad_vocab_size` 和 `self._get_indices`。 实现中包含条件分支。 本段逻辑主要由 `num_embeddings`、`embedding_dim`、`params_dtype`、`org_num_embeddings` 和 `padding_size` 等参数驱动。

### Lines 309-345: `_get_indices` implementation / `_get_indices` 实现
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
**EN:** This block defines method `_get_indices` on `VocabParallelEmbedding`. Get start and end indices for vocab parallel embedding, following the layout outlined in the class docstring, based on the given tp_rank and tp_size. Key calls include `vocab_range_from_global_vocab_size`, `min`, and `VocabParallelEmbeddingShardIndices`. Parameters such as `vocab_size_padded`, `org_vocab_size_padded`, `vocab_size`, `org_vocab_size`, and `tp_rank` drive the behavior in this section.
**CN:** 该代码块定义了 `VocabParallelEmbedding` 的方法 `_get_indices`。 它用于获取indices。 关键调用包括 `vocab_range_from_global_vocab_size`、`min` 和 `VocabParallelEmbeddingShardIndices`。 本段逻辑主要由 `vocab_size_padded`、`org_vocab_size_padded`、`vocab_size`、`org_vocab_size` 和 `tp_rank` 等参数驱动。

### Lines 347-410: `get_sharded_to_full_mapping` implementation / `get_sharded_to_full_mapping` 实现
```python
    def get_sharded_to_full_mapping(self) -> list[int] | None:
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

        base_embeddings: list[int] = []
        added_embeddings: list[int] = []
        padding: list[int] = []
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
**EN:** This block defines method `get_sharded_to_full_mapping` on `VocabParallelEmbedding`. Get a mapping that can be used to reindex the gathered logits for sampling. During sampling, we gather logits from all ranks. Key calls include `range`, `self._get_indices`, `base_embeddings.extend`, `padding.extend`, and `added_embeddings.extend`. The implementation branches on conditions, iterates over collections or steps.
**CN:** 该代码块定义了 `VocabParallelEmbedding` 的方法 `get_sharded_to_full_mapping`。 它用于获取sharded to full mapping。 关键调用包括 `range`、`self._get_indices`、`base_embeddings.extend`、`padding.extend` 和 `added_embeddings.extend`。 实现中包含条件分支，会遍历集合或步骤。

### Lines 412-458: `weight_loader` implementation / `weight_loader` 实现
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
                shape[output_dim] = self.num_embeddings_per_partition
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
                else param.pack_factor
            )
            assert loaded_weight.shape[output_dim] == (
                self.org_vocab_size // param.packed_factor
            )
            start_idx = start_idx // packed_factor
            shard_size = shard_size // packed_factor
        else:
            assert loaded_weight.shape[output_dim] == self.org_vocab_size

        # Copy the data. Select chunk corresponding to current shard.
        loaded_weight = loaded_weight.narrow(output_dim, start_idx, shard_size)

        param[: loaded_weight.shape[0]].data.copy_(loaded_weight)
        param[loaded_weight.shape[0] :].data.fill_(0)
```
**EN:** This block defines method `weight_loader` on `VocabParallelEmbedding`. It handles weight loader logic. Key calls include `getattr`, `loaded_weight.narrow`, `param.data.copy_`, `param.data.fill_`, and `loaded_weight.item`. The implementation branches on conditions. Parameters such as `param`, and `loaded_weight` drive the behavior in this section.
**CN:** 该代码块定义了 `VocabParallelEmbedding` 的方法 `weight_loader`。 它用于处理 weight loader 相关逻辑。 关键调用包括 `getattr`、`loaded_weight.narrow`、`param.data.copy_`、`param.data.fill_` 和 `loaded_weight.item`。 实现中包含条件分支。 本段逻辑主要由 `param` 和 `loaded_weight` 等参数驱动。

### Lines 460-482: `forward` implementation / `forward` 实现
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
        output_parallel = self.quant_method.embedding(self, masked_input.long())
        # Mask the output embedding.
        if self.tp_size > 1:
            output_parallel.masked_fill_(input_mask.unsqueeze(-1), 0)
        # Reduce across all the model parallel GPUs.
        output = tensor_model_parallel_all_reduce(
            output_parallel, tp_group=self.tp_group
        )
        return output
```
**EN:** This block defines method `forward` on `VocabParallelEmbedding`. It executes function. Key calls include `self.quant_method.embedding`, `tensor_model_parallel_all_reduce`, `get_masked_input_and_mask`, `masked_input.long`, and `output_parallel.masked_fill_`. The implementation branches on conditions. Parameters such as `input_` drive the behavior in this section.
**CN:** 该代码块定义了 `VocabParallelEmbedding` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `self.quant_method.embedding`、`tensor_model_parallel_all_reduce`、`get_masked_input_and_mask`、`masked_input.long` 和 `output_parallel.masked_fill_`。 实现中包含条件分支。 本段逻辑主要由 `input_` 等参数驱动。

### Lines 484-490: `extra_repr` implementation / `extra_repr` 实现
```python
    def extra_repr(self) -> str:
        s = f"num_embeddings={self.num_embeddings_per_partition}"
        s += f", embedding_dim={self.embedding_dim}"
        s += f", org_vocab_size={self.org_vocab_size}"
        s += f", num_embeddings_padded={self.num_embeddings_padded}"
        s += f", tp_size={self.tp_size}"
        return s
```
**EN:** This block defines method `extra_repr` on `VocabParallelEmbedding`. It handles extra repr logic.
**CN:** 该代码块定义了 `VocabParallelEmbedding` 的方法 `extra_repr`。 它用于处理 extra repr 相关逻辑。

## Key Concepts / 关键概念
- `UnquantizedEmbeddingMethod`: Unquantized method for embeddings. / 核心类，用于封装 unquantized embedding method 相关行为。
- `pad_vocab_size`: Pad the vocab size to the given value. / 顶层函数，用于处理 pad vocab size 相关逻辑。
- `vocab_range_from_per_partition_vocab_size`: Top-level function that handles vocab range from per partition vocab size logic. / 顶层函数，用于处理 vocab range from per partition vocab size 相关逻辑。
- `vocab_range_from_global_vocab_size`: Top-level function that handles vocab range from global vocab size logic. / 顶层函数，用于处理 vocab range from global vocab size 相关逻辑。
- `VocabParallelEmbeddingShardIndices`: Indices for a shard of a vocab parallel embedding. / 核心类，用于封装 vocab parallel embedding shard indices 相关行为。
- `get_masked_input_and_mask`: Top-level function that retrieves masked input and mask. / 顶层函数，用于获取masked input and mask。
- `VocabParallelEmbedding`: Embedding parallelized in the vocabulary dimension. / 核心类，用于封装 vocab parallel embedding 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `collections.abc`, `dataclasses`
- **Third-party / 第三方依赖**: `torch`, `torch.distributed`, `torch.nn.functional`, `torch.nn.parameter`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.layers.quantization.configs.base_config`, `sglang.multimodal_gen.runtime.layers.utils`, `sglang.multimodal_gen.runtime.models.parameter`, `sglang.multimodal_gen.runtime.models.utils`, `sglang.multimodal_gen.runtime.platforms`

- **Total lines / 总行数**: 490
