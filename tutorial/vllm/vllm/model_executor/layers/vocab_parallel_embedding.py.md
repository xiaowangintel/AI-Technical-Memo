# vocab_parallel_embedding.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/vocab_parallel_embedding.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements tensor-parallel vocabulary embeddings and sharded logits helpers. / 实现张量并行词表嵌入层与分片 logits 辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports and module setup (lines 4-30)
```python
from collections.abc import Sequence
from dataclasses import dataclass

import torch
import torch.nn.functional as F
from torch.nn.parameter import Parameter, UninitializedParameter

import vllm.envs as envs
from vllm.distributed import (
    divide,
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
    tensor_model_parallel_all_reduce,
)
from vllm.model_executor.custom_op import PluggableLayer
from vllm.model_executor.layers.batch_invariant import (
    linear_batch_invariant,
)
from vllm.model_executor.layers.quantization.base_config import (
    QuantizationConfig,
    QuantizeMethodBase,
    method_has_implemented_embedding,
)
from vllm.model_executor.layers.utils import dispatch_unquantized_gemm
from vllm.model_executor.parameter import BasevLLMParameter
from vllm.model_executor.utils import set_weight_attrs
from vllm.platforms import current_platform
```
**EN:** This opening block pulls in external dependencies such as `collections`, `dataclasses`, `torch` and internal modules such as `vllm.envs`, `vllm.distributed`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.batch_invariant`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.layers.utils`. That import mix shows the file is part of the model-executor layers and helpers stack and whether it relies on PyTorch-only code or backend-specific helpers.
**CN:** 这个起始代码块引入了外部依赖（如 `collections`, `dataclasses`, `torch`）以及内部模块（如 `vllm.envs`, `vllm.distributed`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.batch_invariant`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.layers.utils`）。这些导入关系表明该文件属于模型执行层与辅助组件栈，并体现它更偏向纯 PyTorch 逻辑还是特定后端辅助实现。

### Module constants and metadata (lines 32-32)
```python
DEFAULT_VOCAB_PADDING_SIZE = 64
```
**EN:** This block defines module-level metadata or constants such as `DEFAULT_VOCAB_PADDING_SIZE`. Those symbols usually capture defaults, aliases, export lists, or registry data that later classes/functions reuse inside the model-executor layers and helpers pipeline.
**CN:** 这个代码块定义了模块级元数据或常量，例如 `DEFAULT_VOCAB_PADDING_SIZE`。这些符号通常用于保存默认值、别名、导出列表或注册信息，供后续类和函数在模型执行层与辅助组件流程中复用。

### Function `pad_vocab_size` (lines 81-83)
```python
def pad_vocab_size(vocab_size: int, pad_to: int = DEFAULT_VOCAB_PADDING_SIZE) -> int:
    """Pad the vocab size to the given value."""
    return ((vocab_size + pad_to - 1) // pad_to) * pad_to
```
**EN:** Defines function `pad_vocab_size` with signature `pad_vocab_size(vocab_size: int, pad_to: int=DEFAULT_VOCAB_PADDING_SIZE) -> int`. It mainly works with `vocab_size`, `pad_to`; implements one step of the module control flow. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `pad_vocab_size`，其签名为 `pad_vocab_size(vocab_size: int, pad_to: int=DEFAULT_VOCAB_PADDING_SIZE) -> int`。它主要围绕 `vocab_size`, `pad_to` 展开；实现模块控制流中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Function `vocab_range_from_per_partition_vocab_size` (lines 86-91)
```python
def vocab_range_from_per_partition_vocab_size(
    per_partition_vocab_size: int, rank: int, offset: int = 0
) -> Sequence[int]:
    index_f = rank * per_partition_vocab_size
    index_l = index_f + per_partition_vocab_size
    return index_f + offset, index_l + offset
```
**EN:** Defines function `vocab_range_from_per_partition_vocab_size` with signature `vocab_range_from_per_partition_vocab_size(per_partition_vocab_size: int, rank: int, offset: int=0) -> Sequence[int]`. It mainly works with `per_partition_vocab_size`, `rank`, `offset`; implements one step of the module control flow. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `vocab_range_from_per_partition_vocab_size`，其签名为 `vocab_range_from_per_partition_vocab_size(per_partition_vocab_size: int, rank: int, offset: int=0) -> Sequence[int]`。它主要围绕 `per_partition_vocab_size`, `rank`, `offset` 展开；实现模块控制流中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Function `vocab_range_from_global_vocab_size` (lines 94-100)
```python
def vocab_range_from_global_vocab_size(
    global_vocab_size: int, rank: int, world_size: int, offset: int = 0
) -> Sequence[int]:
    per_partition_vocab_size = divide(global_vocab_size, world_size)
    return vocab_range_from_per_partition_vocab_size(
        per_partition_vocab_size, rank, offset=offset
    )
```
**EN:** Defines function `vocab_range_from_global_vocab_size` with signature `vocab_range_from_global_vocab_size(global_vocab_size: int, rank: int, world_size: int, offset: int=0) -> Sequence[int]`. It mainly works with `global_vocab_size`, `rank`, `world_size`, `offset`; implements one step of the module control flow. The body uses mostly straightforward data movement and object wiring. Key calls include `divide`, `vocab_range_from_per_partition_vocab_size`.
**CN:** 定义函数 `vocab_range_from_global_vocab_size`，其签名为 `vocab_range_from_global_vocab_size(global_vocab_size: int, rank: int, world_size: int, offset: int=0) -> Sequence[int]`。它主要围绕 `global_vocab_size`, `rank`, `world_size`, `offset` 展开；实现模块控制流中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `divide`, `vocab_range_from_per_partition_vocab_size`。

### Function `get_masked_input_and_mask` (lines 163-187)
```python
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
**EN:** Defines function `get_masked_input_and_mask` with signature `get_masked_input_and_mask(input_: torch.Tensor, org_vocab_start_index: int, org_vocab_end_index: int, num_org_vocab_padding: int, added_vocab_start_index: int, added_vocab_end_index: int) -> tuple[torch.Tensor, torch.Tensor]`. It mainly works with `input_`, `org_vocab_start_index`, `org_vocab_end_index`, `num_org_vocab_padding`, `added_vocab_start_index`, `added_vocab_end_index`; returns a derived property or capability check. The body uses tensor/kernel operations. Key calls include `torch.compile`.
**CN:** 定义函数 `get_masked_input_and_mask`，其签名为 `get_masked_input_and_mask(input_: torch.Tensor, org_vocab_start_index: int, org_vocab_end_index: int, num_org_vocab_padding: int, added_vocab_start_index: int, added_vocab_end_index: int) -> tuple[torch.Tensor, torch.Tensor]`。它主要围绕 `input_`, `org_vocab_start_index`, `org_vocab_end_index`, `num_org_vocab_padding`, `added_vocab_start_index`, `added_vocab_end_index` 展开；返回派生属性或能力判断结果。函数体包含张量或内核操作。关键调用包括 `torch.compile`。

### Class `UnquantizedEmbeddingMethod` overview (lines 35-78)
```python
class UnquantizedEmbeddingMethod(QuantizeMethodBase):
    """Unquantized method for embeddings."""

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
**EN:** Defines class `UnquantizedEmbeddingMethod` with base classes `QuantizeMethodBase` and decorators none. It acts as a backend or execution-method adapter and exposes 4 direct methods, with notable entries `create_weights`, `process_weights_after_loading`, `apply`, `embedding`. Its docstring says: Unquantized method for embeddings.
**CN:** 定义类 `UnquantizedEmbeddingMethod`，其基类为 `QuantizeMethodBase`，装饰器为 无。它在整体实现中充当后端或执行方法适配器，并直接暴露 4 个方法，较重要的包括 `create_weights`, `process_weights_after_loading`, `apply`, `embedding`。 文档字符串进一步说明了该类的定位。

### Method `UnquantizedEmbeddingMethod.create_weights` (lines 38-59)
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
**EN:** Defines function `UnquantizedEmbeddingMethod.create_weights` with signature `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`. It mainly works with `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs`; creates tensors, parameters, or helper objects needed later. The body uses tensor/kernel operations. Key calls include `Parameter`, `set_weight_attrs`, `layer.register_parameter`, `torch.empty`, `sum`.
**CN:** 定义函数 `UnquantizedEmbeddingMethod.create_weights`，其签名为 `create_weights(self, layer: torch.nn.Module, input_size_per_partition: int, output_partition_sizes: list[int], input_size: int, output_size: int, params_dtype: torch.dtype, **extra_weight_attrs)`。它主要围绕 `layer`, `input_size_per_partition`, `output_partition_sizes`, `input_size`, `output_size`, `params_dtype`, `**extra_weight_attrs` 展开；创建后续步骤所需的张量、参数或辅助对象。函数体包含张量或内核操作。关键调用包括 `Parameter`, `set_weight_attrs`, `layer.register_parameter`, `torch.empty`, `sum`。

### Method `UnquantizedEmbeddingMethod.process_weights_after_loading` (lines 61-65)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module) -> None:
        if current_platform.is_cpu():
            from vllm.model_executor.layers.utils import dispatch_cpu_unquantized_gemm

            dispatch_cpu_unquantized_gemm(layer, remove_weight=False)
```
**EN:** Defines function `UnquantizedEmbeddingMethod.process_weights_after_loading` with signature `process_weights_after_loading(self, layer: torch.nn.Module) -> None`. It mainly works with `layer`; post-processes intermediate or loaded state before execution. The body uses branching. Key calls include `current_platform.is_cpu`, `dispatch_cpu_unquantized_gemm`.
**CN:** 定义函数 `UnquantizedEmbeddingMethod.process_weights_after_loading`，其签名为 `process_weights_after_loading(self, layer: torch.nn.Module) -> None`。它主要围绕 `layer` 展开；在执行前对中间状态或已加载状态做后处理。函数体包含分支判断。关键调用包括 `current_platform.is_cpu`, `dispatch_cpu_unquantized_gemm`。

### Method `UnquantizedEmbeddingMethod.apply` (lines 67-75)
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        if envs.VLLM_BATCH_INVARIANT and current_platform.is_cuda_alike():
            return linear_batch_invariant(x, layer.weight, bias)
        return dispatch_unquantized_gemm()(layer, x, layer.weight, bias)
```
**EN:** Defines function `UnquantizedEmbeddingMethod.apply` with signature `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`. It mainly works with `layer`, `x`, `bias`; implements one step of the module control flow. The body uses branching, tensor/kernel operations. Key calls include `dispatch_unquantized_gemm`, `current_platform.is_cuda_alike`, `linear_batch_invariant`.
**CN:** 定义函数 `UnquantizedEmbeddingMethod.apply`，其签名为 `apply(self, layer: torch.nn.Module, x: torch.Tensor, bias: torch.Tensor | None=None) -> torch.Tensor`。它主要围绕 `layer`, `x`, `bias` 展开；实现模块控制流中的一个步骤。函数体包含分支判断、张量或内核操作。关键调用包括 `dispatch_unquantized_gemm`, `current_platform.is_cuda_alike`, `linear_batch_invariant`。

### Method `UnquantizedEmbeddingMethod.embedding` (lines 77-78)
```python
    def embedding(self, layer: torch.nn.Module, input_: torch.Tensor) -> torch.Tensor:
        return F.embedding(input_, layer.weight)
```
**EN:** Defines function `UnquantizedEmbeddingMethod.embedding` with signature `embedding(self, layer: torch.nn.Module, input_: torch.Tensor) -> torch.Tensor`. It mainly works with `layer`, `input_`; maps ids or features into embedding space. The body uses mostly straightforward data movement and object wiring. Key calls include `F.embedding`.
**CN:** 定义函数 `UnquantizedEmbeddingMethod.embedding`，其签名为 `embedding(self, layer: torch.nn.Module, input_: torch.Tensor) -> torch.Tensor`。它主要围绕 `layer`, `input_` 展开；把 id 或特征映射到嵌入空间。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 `F.embedding`。

### Class `VocabParallelEmbeddingShardIndices` overview (lines 104-159)
```python
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

    @property
    def num_org_elements(self) -> int:
        return self.org_vocab_end_index - self.org_vocab_start_index

    @property
    def num_added_elements(self) -> int:
        return self.added_vocab_end_index - self.added_vocab_start_index

    @property
    def num_org_elements_padded(self) -> int:
        return self.padded_org_vocab_end_index - self.padded_org_vocab_start_index
```
**EN:** Defines class `VocabParallelEmbeddingShardIndices` with base classes no explicit base class and decorators `dataclass`. It acts as a reusable module building block and exposes 8 direct methods, with notable entries `num_org_elements`, `num_added_elements`, `num_org_elements_padded`, `num_added_elements_padded`, `num_org_vocab_padding`, `num_added_vocab_padding`. Its docstring says: Indices for a shard of a vocab parallel embedding.
**CN:** 定义类 `VocabParallelEmbeddingShardIndices`，其基类为 无显式基类，装饰器为 `dataclass`。它在整体实现中充当可复用的模块构件，并直接暴露 8 个方法，较重要的包括 `num_org_elements`, `num_added_elements`, `num_org_elements_padded`, `num_added_elements_padded`, `num_org_vocab_padding`, `num_added_vocab_padding`。 文档字符串进一步说明了该类的定位。

### Method `VocabParallelEmbeddingShardIndices.num_org_elements` (lines 118-119)
```python
    def num_org_elements(self) -> int:
        return self.org_vocab_end_index - self.org_vocab_start_index
```
**EN:** Defines function `VocabParallelEmbeddingShardIndices.num_org_elements` with signature `num_org_elements(self) -> int`. It mainly works with object context only; implements one step of the module control flow. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `VocabParallelEmbeddingShardIndices.num_org_elements`，其签名为 `num_org_elements(self) -> int`。它主要围绕 仅依赖对象上下文 展开；实现模块控制流中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `VocabParallelEmbeddingShardIndices.num_added_elements` (lines 122-123)
```python
    def num_added_elements(self) -> int:
        return self.added_vocab_end_index - self.added_vocab_start_index
```
**EN:** Defines function `VocabParallelEmbeddingShardIndices.num_added_elements` with signature `num_added_elements(self) -> int`. It mainly works with object context only; implements one step of the module control flow. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `VocabParallelEmbeddingShardIndices.num_added_elements`，其签名为 `num_added_elements(self) -> int`。它主要围绕 仅依赖对象上下文 展开；实现模块控制流中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `VocabParallelEmbeddingShardIndices.num_org_elements_padded` (lines 126-127)
```python
    def num_org_elements_padded(self) -> int:
        return self.padded_org_vocab_end_index - self.padded_org_vocab_start_index
```
**EN:** Defines function `VocabParallelEmbeddingShardIndices.num_org_elements_padded` with signature `num_org_elements_padded(self) -> int`. It mainly works with object context only; implements one step of the module control flow. The body uses mostly straightforward data movement and object wiring. Key calls include no notable helper calls.
**CN:** 定义函数 `VocabParallelEmbeddingShardIndices.num_org_elements_padded`，其签名为 `num_org_elements_padded(self) -> int`。它主要围绕 仅依赖对象上下文 展开；实现模块控制流中的一个步骤。函数体包含以直接的数据组织和对象拼装为主。关键调用包括 无显著辅助调用。

### Method `VocabParallelEmbeddingShardIndices.__post_init__` (lines 145-159)
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
**EN:** Defines function `VocabParallelEmbeddingShardIndices.__post_init__` with signature `__post_init__(self)`. It mainly works with object context only; implements one step of the module control flow. The body uses validation/error handling. Key calls include no notable helper calls.
**CN:** 定义函数 `VocabParallelEmbeddingShardIndices.__post_init__`，其签名为 `__post_init__(self)`。它主要围绕 仅依赖对象上下文 展开；实现模块控制流中的一个步骤。函数体包含校验或报错逻辑。关键调用包括 无显著辅助调用。

### Class `VocabParallelEmbedding` overview (lines 192-498)
```python
class VocabParallelEmbedding(PluggableLayer):
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
    corresponding token_id: |  0  |  1  | ... | 1009 |  -1  | ... |  -1  | 1010 | ... | 1025 |  -1  | ... |  -1  |
                     index: |  0  |  1  | ... | 1009 | 1010 | ... | 1023 | 1024 | ... | 1039 | 1040 | ... | 1087 |

    TP2, rank 0:
                            |< --------------------BASE--------------------- >|< -----LORA------ >|< -LORA PADDING- >|
    corresponding token_id: |  0  |  1  |  2  | ... | 497  | 498 | ...  | 511 | 1010 | ... | 1025 |  -1  | ... |  -1 |
                     index: |  0  |  1  |  2  | ... | 497  | 498 | ...  | 511 | 512  | ... | 527  |  528 | ... | 543 |
    TP2, rank 1:
```
**EN:** Defines class `VocabParallelEmbedding` with base classes `PluggableLayer` and decorators `PluggableLayer.register('vocab_parallel_embedding')`. It acts as an embedding layer with model-parallel awareness and exposes 6 direct methods, with notable entries `__init__`, `_get_indices`, `get_sharded_to_full_mapping`, `weight_loader`, `forward`, `extra_repr`. Its docstring says: Embedding parallelized in the vocabulary dimension.
**CN:** 定义类 `VocabParallelEmbedding`，其基类为 `PluggableLayer`，装饰器为 `PluggableLayer.register('vocab_parallel_embedding')`。它在整体实现中充当具备模型并行意识的嵌入层，并直接暴露 6 个方法，较重要的包括 `__init__`, `_get_indices`, `get_sharded_to_full_mapping`, `weight_loader`, `forward`, `extra_repr`。 文档字符串进一步说明了该类的定位。

### Method `VocabParallelEmbedding.__init__` (lines 233-318)
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
    ):
        super().__init__()

        # Keep the input dimensions.
        tp_rank = get_tensor_model_parallel_rank()
        self.tp_size = get_tensor_model_parallel_world_size()
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
# ... truncated for analysis ...
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
**EN:** Defines function `VocabParallelEmbedding.__init__` with signature `__init__(self, num_embeddings: int, embedding_dim: int, params_dtype: torch.dtype | None=None, org_num_embeddings: int | None=None, padding_size: int=DEFAULT_VOCAB_PADDING_SIZE, quant_config: QuantizationConfig | None=None, prefix: str='')`. It mainly works with `num_embeddings`, `embedding_dim`, `params_dtype`, `org_num_embeddings`, `padding_size`, `quant_config`, `prefix`; initializes the object state and cached resources. The body uses branching, validation/error handling, tensor/kernel operations. Key calls include `super.__init__`, `get_tensor_model_parallel_rank`, `get_tensor_model_parallel_world_size`, `pad_vocab_size`, `self._get_indices`, `method_has_implemented_embedding`.
**CN:** 定义函数 `VocabParallelEmbedding.__init__`，其签名为 `__init__(self, num_embeddings: int, embedding_dim: int, params_dtype: torch.dtype | None=None, org_num_embeddings: int | None=None, padding_size: int=DEFAULT_VOCAB_PADDING_SIZE, quant_config: QuantizationConfig | None=None, prefix: str='')`。它主要围绕 `num_embeddings`, `embedding_dim`, `params_dtype`, `org_num_embeddings`, `padding_size`, `quant_config`, `prefix` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、校验或报错逻辑、张量或内核操作。关键调用包括 `super.__init__`, `get_tensor_model_parallel_rank`, `get_tensor_model_parallel_world_size`, `pad_vocab_size`, `self._get_indices`, `method_has_implemented_embedding`。

### Method `VocabParallelEmbedding.get_sharded_to_full_mapping` (lines 358-421)
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
# ... truncated for analysis ...
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
**EN:** Defines function `VocabParallelEmbedding.get_sharded_to_full_mapping` with signature `get_sharded_to_full_mapping(self) -> list[int] | None`. It mainly works with object context only; returns a derived property or capability check. The body uses branching, iteration, validation/error handling. Key calls include `range`, `self._get_indices`, `base_embeddings.extend`, `padding.extend`, `added_embeddings.extend`, `len`.
**CN:** 定义函数 `VocabParallelEmbedding.get_sharded_to_full_mapping`，其签名为 `get_sharded_to_full_mapping(self) -> list[int] | None`。它主要围绕 仅依赖对象上下文 展开；返回派生属性或能力判断结果。函数体包含分支判断、循环处理、校验或报错逻辑。关键调用包括 `range`, `self._get_indices`, `base_embeddings.extend`, `padding.extend`, `added_embeddings.extend`, `len`。

### Method `VocabParallelEmbedding.weight_loader` (lines 423-468)
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
**EN:** Defines function `VocabParallelEmbedding.weight_loader` with signature `weight_loader(self, param: Parameter, loaded_weight: torch.Tensor)`. It mainly works with `param`, `loaded_weight`; implements one step of the module control flow. The body uses branching, validation/error handling. Key calls include `getattr`, `loaded_weight.narrow`, `param.data.copy_`, `param.data.fill_`, `loaded_weight.item`, `isinstance`.
**CN:** 定义函数 `VocabParallelEmbedding.weight_loader`，其签名为 `weight_loader(self, param: Parameter, loaded_weight: torch.Tensor)`。它主要围绕 `param`, `loaded_weight` 展开；实现模块控制流中的一个步骤。函数体包含分支判断、校验或报错逻辑。关键调用包括 `getattr`, `loaded_weight.narrow`, `param.data.copy_`, `param.data.fill_`, `loaded_weight.item`, `isinstance`。

### Method `VocabParallelEmbedding.forward` (lines 470-490)
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
        output = tensor_model_parallel_all_reduce(output_parallel)
        return output
```
**EN:** Defines function `VocabParallelEmbedding.forward` with signature `forward(self, input_)`. It mainly works with `input_`; runs the main forward-path computation. The body uses branching. Key calls include `self.quant_method.embedding`, `tensor_model_parallel_all_reduce`, `get_masked_input_and_mask`, `masked_input.long`, `output_parallel.masked_fill_`, `input_mask.unsqueeze`.
**CN:** 定义函数 `VocabParallelEmbedding.forward`，其签名为 `forward(self, input_)`。它主要围绕 `input_` 展开；执行主要的前向计算路径。函数体包含分支判断。关键调用包括 `self.quant_method.embedding`, `tensor_model_parallel_all_reduce`, `get_masked_input_and_mask`, `masked_input.long`, `output_parallel.masked_fill_`, `input_mask.unsqueeze`。

### Class `ParallelLMHead` overview (lines 503-567)
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

    # --8<-- [end:parallel_lm_head]

    def __init__(
        self,
        num_embeddings: int,
        embedding_dim: int,
        bias: bool = False,
        params_dtype: torch.dtype | None = None,
        org_num_embeddings: int | None = None,
```
**EN:** Defines class `ParallelLMHead` with base classes `VocabParallelEmbedding` and decorators `PluggableLayer.register('parallel_lm_head')`. It acts as a head that projects hidden states into task outputs and exposes 3 direct methods, with notable entries `__init__`, `tie_weights`, `forward`. Its docstring says: Parallelized LM head.
**CN:** 定义类 `ParallelLMHead`，其基类为 `VocabParallelEmbedding`，装饰器为 `PluggableLayer.register('parallel_lm_head')`。它在整体实现中充当把隐藏状态映射到任务输出的头部模块，并直接暴露 3 个方法，较重要的包括 `__init__`, `tie_weights`, `forward`。 文档字符串进一步说明了该类的定位。

### Method `ParallelLMHead.__init__` (lines 521-554)
```python
    def __init__(
        self,
        num_embeddings: int,
        embedding_dim: int,
        bias: bool = False,
        params_dtype: torch.dtype | None = None,
        org_num_embeddings: int | None = None,
        padding_size: int = DEFAULT_VOCAB_PADDING_SIZE,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__(
            num_embeddings,
            embedding_dim,
            params_dtype,
            org_num_embeddings,
            padding_size,
            quant_config,
            prefix,
        )
        self.quant_config = quant_config
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
**EN:** Defines function `ParallelLMHead.__init__` with signature `__init__(self, num_embeddings: int, embedding_dim: int, bias: bool=False, params_dtype: torch.dtype | None=None, org_num_embeddings: int | None=None, padding_size: int=DEFAULT_VOCAB_PADDING_SIZE, quant_config: QuantizationConfig | None=None, prefix: str='')`. It mainly works with `num_embeddings`, `embedding_dim`, `bias`, `params_dtype`, `org_num_embeddings`, `padding_size`, `quant_config`, `prefix`; initializes the object state and cached resources. The body uses branching, tensor/kernel operations. Key calls include `super.__init__`, `Parameter`, `set_weight_attrs`, `self.register_parameter`, `super`, `torch.empty`.
**CN:** 定义函数 `ParallelLMHead.__init__`，其签名为 `__init__(self, num_embeddings: int, embedding_dim: int, bias: bool=False, params_dtype: torch.dtype | None=None, org_num_embeddings: int | None=None, padding_size: int=DEFAULT_VOCAB_PADDING_SIZE, quant_config: QuantizationConfig | None=None, prefix: str='')`。它主要围绕 `num_embeddings`, `embedding_dim`, `bias`, `params_dtype`, `org_num_embeddings`, `padding_size`, `quant_config`, `prefix` 展开；负责初始化对象状态和缓存资源。函数体包含分支判断、张量或内核操作。关键调用包括 `super.__init__`, `Parameter`, `set_weight_attrs`, `self.register_parameter`, `super`, `torch.empty`。

### Method `ParallelLMHead.tie_weights` (lines 556-563)
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
**EN:** Defines function `ParallelLMHead.tie_weights` with signature `tie_weights(self, embed_tokens: VocabParallelEmbedding)`. It mainly works with `embed_tokens`; implements one step of the module control flow. The body uses branching. Key calls include `self.quant_config.get_name`.
**CN:** 定义函数 `ParallelLMHead.tie_weights`，其签名为 `tie_weights(self, embed_tokens: VocabParallelEmbedding)`。它主要围绕 `embed_tokens` 展开；实现模块控制流中的一个步骤。函数体包含分支判断。关键调用包括 `self.quant_config.get_name`。

### Method `ParallelLMHead.forward` (lines 565-567)
```python
    def forward(self, input_):
        del input_
        raise RuntimeError("LMHead's weights should be used in the sampler.")
```
**EN:** Defines function `ParallelLMHead.forward` with signature `forward(self, input_)`. It mainly works with `input_`; runs the main forward-path computation. The body uses validation/error handling. Key calls include `RuntimeError`.
**CN:** 定义函数 `ParallelLMHead.forward`，其签名为 `forward(self, input_)`。它主要围绕 `input_` 展开；执行主要的前向计算路径。函数体包含校验或报错逻辑。关键调用包括 `RuntimeError`。

## Key Concepts / 关键概念
- **EN:** The file provides a reusable layer/helper inside the model-executor subsystem.
  **CN:** 该文件在模型执行子系统中提供可复用的层或辅助逻辑。
- **EN:** Top-level classes include `UnquantizedEmbeddingMethod`, `VocabParallelEmbeddingShardIndices`, `VocabParallelEmbedding`, `ParallelLMHead`, which organize the main reusable abstractions.
  **CN:** 顶层类包括 `UnquantizedEmbeddingMethod`, `VocabParallelEmbeddingShardIndices`, `VocabParallelEmbedding`, `ParallelLMHead`，它们组织了主要的可复用抽象。
- **EN:** Top-level helpers such as `pad_vocab_size`, `vocab_range_from_per_partition_vocab_size`, `vocab_range_from_global_vocab_size`, `get_masked_input_and_mask` expose procedural entry points around the main classes.
  **CN:** 顶层辅助函数如 `pad_vocab_size`, `vocab_range_from_per_partition_vocab_size`, `vocab_range_from_global_vocab_size`, `get_masked_input_and_mask` 为主要类提供了过程式入口。
- **EN:** PyTorch is the main execution substrate, so most logic is expressed as tensor transformations and module wrappers.
  **CN:** PyTorch 是主要执行基础，因此大部分逻辑会表现为张量变换和模块封装。

## Dependencies / 依赖关系
- **External / 外部**: `collections`, `dataclasses`, `torch`
- **Internal / 内部**: `vllm.envs`, `vllm.distributed`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.batch_invariant`, `vllm.model_executor.layers.quantization.base_config`, `vllm.model_executor.layers.utils`, `vllm.model_executor.parameter`, `vllm.model_executor.utils`, `vllm.platforms`
