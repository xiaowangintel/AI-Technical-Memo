# roberta.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/roberta.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the roberta model stack used by SGLang SRT inference, including network blocks, runtime wrappers, and checkpoint-loading utilities. / 该模块实现 SGLang SRT 推理所使用的 roberta 模型栈，包括网络模块、运行时封装以及检查点加载工具。

## Line-by-Line Analysis / 逐行分析

### Lines 3-16: Module imports
```python
import os
from typing import Iterable, Optional, Tuple

import torch
from torch import nn

from sglang.srt.layers.pooler import CrossEncodingPooler, Pooler, PoolingType
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.layers.sparse_pooler import SparsePooler
from sglang.srt.layers.vocab_parallel_embedding import VocabParallelEmbedding
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.model_loader.weight_utils import default_weight_loader
from sglang.srt.models.bert import BertEncoder
from sglang.srt.utils.hf_transformers_utils import download_from_hf
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 18-18: Top-level assign
```python
RobertaConfig = None
```
**EN:** Defines or updates RobertaConfig, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 RobertaConfig，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 22-24: Class `RobertaClassificationHead` overview
```python
class RobertaClassificationHead(nn.Module):
    """Head for sentence-level classification tasks."""
```
**EN:** Defines `RobertaClassificationHead` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `RobertaClassificationHead`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 25-28: Method `RobertaClassificationHead.__init__`
```python
    def __init__(self, config: RobertaConfig):
        super().__init__()
        self.dense = nn.Linear(config.hidden_size, config.hidden_size)
        self.out_proj = nn.Linear(config.hidden_size, config.num_labels)
```
**EN:** This method implements `__init__(config: ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 30-35: Method `RobertaClassificationHead.forward`
```python
    def forward(self, features, **kwargs):
        x = features[0, :]  # take <s> token (equiv. to [CLS])
        x = self.dense(x)
        x = torch.tanh(x)
        x = self.out_proj(x)
        return x
```
**EN:** This method implements `forward(features, **kwargs)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(features, **kwargs)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 38-39: Class `RobertaEmbedding` overview
```python
class RobertaEmbedding(nn.Module):
```
**EN:** Defines `RobertaEmbedding` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `RobertaEmbedding`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 40-66: Method `RobertaEmbedding.__init__`
```python
    def __init__(self, config: RobertaConfig):
        super().__init__()
        self.size = config.hidden_size
        self.word_embeddings = VocabParallelEmbedding(
            config.vocab_size, config.hidden_size
        )
        self.padding_idx = config.pad_token_id
        self.position_embeddings = nn.Embedding(
            config.max_position_embeddings,
            config.hidden_size,
            padding_idx=self.padding_idx,
        )

        self.token_type_embeddings = nn.Embedding(
            config.type_vocab_size, config.hidden_size
        )
        self.LayerNorm = nn.LayerNorm(config.hidden_size, eps=config.layer_norm_eps)

        self.position_ids = nn.Parameter(
            torch.empty((1, config.max_position_embeddings)),
        )

        self.position_embedding_type = config.position_embedding_type
        if self.position_embedding_type != "absolute":
            raise ValueError(
                "Only 'absolute' position_embedding_type" + " is supported"
            )
```
**EN:** This method implements `__init__(config: ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 68-113: Method `RobertaEmbedding.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        seq_lens: torch.Tensor,
        position_ids: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        input_shape = input_ids.size()
        inputs_embeds = self.word_embeddings(input_ids)

        # Adapted from vllm: https://github.com/vllm-project/vllm/commit/4a18fd14ba4a349291c798a16bf62fa8a9af0b6b/vllm/model_executor/models/roberta.py

        pos_list = []
        token_list = []
        offset = 0
        for seq_len in seq_lens:
            pos_list.append(position_ids[offset : offset + seq_len])
            token_list.append(input_ids[offset : offset + seq_len])
            offset += seq_len

        new_pos_list = []
        for positions, tokens in zip(pos_list, token_list):
            # Verify assumption that incoming position are
            # always a sequence from 0 to N.
            expected_pos = torch.arange(
                positions.size()[0], dtype=torch.long, device=inputs_embeds.device
            )
            assert torch.equal(positions, expected_pos)
            new_pos_list.append(
                create_position_ids_from_input_ids(tokens, self.padding_idx)
            )
        position_ids = torch.cat(new_pos_list)

        # Position embeddings.
        position_embeddings = self.position_embeddings(position_ids)

        token_type_ids = forward_batch.token_type_ids
        if token_type_ids is None:
            token_type_ids = torch.zeros(
                input_shape, dtype=torch.long, device=inputs_embeds.device
# ... truncated for brevity ...
```
**EN:** This method implements `forward(input_ids: ..., seq_lens: ..., position_ids: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., seq_lens: ..., position_ids: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 116-116: Class `XLMRobertaBaseModel` overview
```python
class XLMRobertaBaseModel(nn.Module):
```
**EN:** Defines `XLMRobertaBaseModel` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `XLMRobertaBaseModel`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 117-134: Method `XLMRobertaBaseModel.__init__`
```python
    def __init__(
        self,
        *,
        config: RobertaConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        add_pooling_layer: bool = False,
    ):
        super().__init__()

        self.config = config
        self.embeddings = RobertaEmbedding(config)
        self.encoder = BertEncoder(config=config, quant_config=quant_config, prefix="")
        self.pooler = (
            Pooler(pooling_type=PoolingType.CLS, normalize=True)
            if add_pooling_layer
            else None
        )
```
**EN:** This method implements `__init__(*, config: ..., quant_config: ...=..., prefix: ...=..., add_pooling_layer: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(*, config: ..., quant_config: ...=..., prefix: ...=..., add_pooling_layer: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 136-157: Method `XLMRobertaBaseModel.forward`
```python
    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: torch.Tensor = None,
        get_embedding: bool = False,
    ) -> torch.Tensor:
        assert get_embedding == True
        # Your tokenized IDs

        hidden_states = self.embeddings(
            input_ids=input_ids,
            position_ids=positions,
            seq_lens=forward_batch.seq_lens,
            forward_batch=forward_batch,
        )

        hidden_states = self.encoder(hidden_states, forward_batch=forward_batch)

        return hidden_states
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., get_embedding: ...=...)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., get_embedding: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 159-190: Method `XLMRobertaBaseModel.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("qkv_proj", "query", "q"),
            ("qkv_proj", "key", "k"),
            ("qkv_proj", "value", "v"),
        ]

        params_dict = dict(self.named_parameters())
        for name, loaded_weight in weights:
            name = name.replace("self", "self_attn")
            if self.pooler is None and "pooler" in name:
                continue
            for param_name, weight_name, shard_id in stacked_params_mapping:

                if weight_name not in name:
                    continue
                name = name.replace(weight_name, param_name)
                # Skip loading extra bias for GPTQ models.
                if name.endswith(".bias") and name not in params_dict:
                    continue
                param = params_dict[name]
                weight_loader = param.weight_loader
                weight_loader(param, loaded_weight, shard_id)
                break
            else:
                # Skip loading extra bias for GPTQ models.
                if name.endswith(".bias") and name not in params_dict:
                    continue
                param = params_dict[name]
                weight_loader = getattr(param, "weight_loader", default_weight_loader)
                weight_loader(param, loaded_weight)
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 194-201: Function `create_position_ids_from_input_ids`
```python
def create_position_ids_from_input_ids(
    input_ids, padding_idx, past_key_values_length=0
):
    mask = input_ids.ne(padding_idx).int()
    incremental_indices = (
        torch.cumsum(mask, dim=0).type_as(mask) + past_key_values_length
    ) * mask
    return incremental_indices.long() + padding_idx
```
**EN:** This function implements `create_position_ids_from_input_ids(input_ids, padding_idx, past_key_values_length=...)` and implements a focused helper that supports the surrounding runtime flow.
**CN:** 这个函数实现了 `create_position_ids_from_input_ids(input_ids, padding_idx, past_key_values_length=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑。

### Lines 204-204: Class `XLMRobertaModel` overview
```python
class XLMRobertaModel(nn.Module):
```
**EN:** Defines `XLMRobertaModel` as a reusable runtime type derived from nn.Module. The class groups 4 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `XLMRobertaModel`，其继承关系为 nn.Module。这个类组织了 4 个方法，用于实现模型相关行为。

### Lines 205-233: Method `XLMRobertaModel.__init__`
```python
    def __init__(
        self,
        *,
        config: RobertaConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        sparse_head: Optional[str] = None,
        model_path: Optional[str] = None,
    ):
        super().__init__()
        self.roberta = XLMRobertaBaseModel(
            config=config, quant_config=quant_config, prefix=prefix
        )
        if sparse_head is not None:
            self._is_sparse = True
            self._model_path = model_path
            self._sparse_head = sparse_head
            self.pooler = SparsePooler(config=config)
            # Zero out special tokens
            self._special_tokens = [
                config.bos_token_id,
                config.eos_token_id,
                config.pad_token_id,
                # self.config.unk_token_id # not available in the XLMRobertaConfig
            ]
            self._special_tokens = [t for t in self._special_tokens if t is not None]
        else:
            self._is_sparse = False
            self.pooler = Pooler(pooling_type=PoolingType.CLS, normalize=True)
```
**EN:** This method implements `__init__(*, config: ..., quant_config: ...=..., prefix: ...=..., sparse_head: ...=..., model_path: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(*, config: ..., quant_config: ...=..., prefix: ...=..., sparse_head: ...=..., model_path: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 235-253: Method `XLMRobertaModel.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: torch.Tensor = None,
        get_embedding: bool = False,
    ) -> torch.Tensor:
        hidden_states = self.roberta(
            input_ids, positions, forward_batch, input_embeds, get_embedding
        )
        embeddings = self.pooler(hidden_states, forward_batch)

        if self._is_sparse:
            for token_id in self._special_tokens:
                embeddings.embeddings[:, token_id] = 0.0
            embeddings.embeddings = embeddings.embeddings.to_sparse()

        return embeddings
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., get_embedding: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., get_embedding: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 255-262: Method `XLMRobertaModel.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        self.roberta.load_weights(weights)

        if self._is_sparse:
            sparse_dict = XLMRobertaModel._load_sparse_linear(
                self._model_path, self._sparse_head
            )
            self.pooler.load_weights(sparse_dict)
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 264-282: Method `XLMRobertaModel._load_sparse_linear`
```python
    @staticmethod
    def _load_sparse_linear(model_path_or_dir: str, sparse_head: str) -> dict:
        """
        Load sparse_head from local dir or HF Hub.
        Returns a state_dict suitable for nn.Linear.load_state_dict().
        """
        if os.path.isdir(model_path_or_dir):
            path = os.path.join(model_path_or_dir, sparse_head)
            if not os.path.exists(path):
                raise FileNotFoundError(
                    f"'{sparse_head}' not found in {model_path_or_dir}"
                )
        else:
            # remote → use SGLang HF utility
            local_dir = download_from_hf(model_path_or_dir, allow_patterns=sparse_head)
            path = os.path.join(local_dir, sparse_head)

        state_dict = torch.load(path)
        return state_dict
```
**EN:** This method implements `_load_sparse_linear(sparse_head: ...)` and Load sparse_head from local dir or HF Hub. Decorators: staticmethod.
**CN:** 这个方法实现了 `_load_sparse_linear(sparse_head: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责 装饰器：staticmethod。

### Lines 285-285: Class `XLMRobertaForSequenceClassification` overview
```python
class XLMRobertaForSequenceClassification(nn.Module):
```
**EN:** Defines `XLMRobertaForSequenceClassification` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `XLMRobertaForSequenceClassification`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 286-298: Method `XLMRobertaForSequenceClassification.__init__`
```python
    def __init__(
        self,
        *,
        config: RobertaConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.roberta = XLMRobertaBaseModel(
            config=config, quant_config=quant_config, prefix=prefix
        )
        self.classifier = RobertaClassificationHead(config)
        self.pooler = CrossEncodingPooler(config, self.classifier, self.roberta.pooler)
```
**EN:** This method implements `__init__(*, config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(*, config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 300-315: Method `XLMRobertaForSequenceClassification.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: torch.Tensor = None,
        get_embedding: bool = True,
    ) -> torch.Tensor:
        assert (
            get_embedding
        ), "XLMRobertaForSequenceClassification is only used for rerank"

        hidden_states = self.roberta(
            input_ids, positions, forward_batch, input_embeds, get_embedding
        )
        return self.pooler(hidden_states, forward_batch)
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., get_embedding: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., get_embedding: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 317-335: Method `XLMRobertaForSequenceClassification.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        self_weights = []

        def weight_filter():
            for name, weight in weights:
                if name.startswith("roberta."):
                    yield (name[len("roberta.") :], weight)
                else:
                    self_weights.append((name, weight))

        self.roberta.load_weights(weight_filter())

        params_dict = dict(self.named_parameters())

        for name, loaded_weight in self_weights:
            if name.startswith("classifier"):
                param = params_dict[name]
                weight_loader = getattr(param, "weight_loader", default_weight_loader)
                weight_loader(param, loaded_weight)
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 338-338: Top-level assign
```python
EntryClass = [XLMRobertaModel, XLMRobertaForSequenceClassification]
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影
- **EN:** Checkpoint remapping and weight loading / **CN:** 检查点重映射与权重加载

## Dependencies / 依赖关系
- `os`
- `typing.Iterable`
- `typing.Optional`
- `typing.Tuple`
- `torch`
- `torch.nn`
- `sglang.srt.layers.pooler.CrossEncodingPooler`
- `sglang.srt.layers.pooler.Pooler`
- `sglang.srt.layers.pooler.PoolingType`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.layers.sparse_pooler.SparsePooler`
- `sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.models.bert.BertEncoder`
- `sglang.srt.utils.hf_transformers_utils.download_from_hf`
