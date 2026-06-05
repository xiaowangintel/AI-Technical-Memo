# bert.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/bert.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Bert architecture into SGLang's serving runtime, typically defining layers, model wrappers, and weight-loading behavior for inference. / 该模块将 Bert 架构接入 SGLang 服务运行时，通常定义推理所需的层、模型封装以及权重加载逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Module header and imports / 模块头与导入
```python
# SPDX-License-Identifier: Apache-2.0
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 23-23: Assignment: BertConfig / 赋值：BertConfig
```python
BertConfig = None
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 26-77: Class: BertEmbedding / 类：BertEmbedding
```python
class BertEmbedding(nn.Module):

    def __init__(self, config: BertConfig):

        super().__init__()
        self.size = config.hidden_size
        self.word_embeddings = VocabParallelEmbedding(
            config.vocab_size, config.hidden_size
        )
        self.position_embeddings = VocabParallelEmbedding(
            config.max_position_embeddings, config.hidden_size
        )
        self.token_type_embeddings = VocabParallelEmbedding(
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

    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        input_shape = input_ids.size()

        # Input embeddings.
        inputs_embeds = self.word_embeddings(input_ids)

        # Position embeddings.
        position_embeddings = self.position_embeddings(positions)

# ... truncated for brevity ...
```
**EN:** This class defines Bert Embedding inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Bert Embedding，用于封装该模型组件的状态与方法。

### Lines 80-96: Class: BertPooler / 类：BertPooler
```python
class BertPooler(nn.Module):

    def __init__(self, config: BertConfig):
        super().__init__()
        self.dense = nn.Linear(config.hidden_size, config.hidden_size)
        self.activation = nn.Tanh()

    def forward(
        self, hidden_states: torch.Tensor, forward_batch: ForwardBatch
    ) -> torch.Tensor:
        # simply taking the hidden state corresponding
        first_token_tensor = hidden_states[0, :]

        pooled_output = self.dense(first_token_tensor)
        pooled_output = self.activation(pooled_output)

        return pooled_output
```
**EN:** This class defines Bert Pooler inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Bert Pooler，用于封装该模型组件的状态与方法。

### Lines 99-127: Class: BertEncoder / 类：BertEncoder
```python
class BertEncoder(nn.Module):

    def __init__(
        self,
        config: BertConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.quant_config = quant_config
        self.layer = nn.ModuleList(
            [
                BertLayer(
                    config=config,
                    layer_id=layer_idx,
                    quant_config=quant_config,
                    prefix=f"{prefix}.layer.{layer_idx}",
                )
                for layer_idx in range(config.num_hidden_layers)
            ]
        )

    def forward(
        self, hidden_states: torch.Tensor, forward_batch: ForwardBatch
    ) -> torch.Tensor:
        for layer in self.layer:
            hidden_states = layer(hidden_states, forward_batch)
        return hidden_states
```
**EN:** This class defines Bert Encoder inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Bert Encoder，用于封装该模型组件的状态与方法。

### Lines 130-173: Class: BertLayer / 类：BertLayer
```python
class BertLayer(nn.Module):

    def __init__(
        self,
        config: BertConfig,
        layer_id: int = 0,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()

        self.layer_id = layer_id

        self.attention = BertAttention(
            hidden_size=config.hidden_size,
            num_attention_heads=config.num_attention_heads,
            layer_id=layer_id,
            layer_norm_eps=config.layer_norm_eps,
            quant_config=quant_config,
            prefix=f"{prefix}.attention",
        )

        self.intermediate = BertIntermediate(
            hidden_size=config.hidden_size,
            intermediate_size=config.intermediate_size,
            hidden_act=config.hidden_act,
            quant_config=quant_config,
            prefix=f"{prefix}.intermediate",
        )

        self.output = BertOutput(
            hidden_size=config.hidden_size,
            intermediate_size=config.intermediate_size,
            layer_norm_eps=config.layer_norm_eps,
            quant_config=quant_config,
            prefix=f"{prefix}.output",
        )

    def forward(self, hidden_states: torch.Tensor, forward_batch: ForwardBatch):
        attn_output = self.attention(hidden_states, forward_batch)
# ... truncated for brevity ...
```
**EN:** This class defines Bert Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Bert Layer，用于封装该模型组件的状态与方法。

### Lines 176-208: Class: BertAttention / 类：BertAttention
```python
class BertAttention(nn.Module):

    def __init__(
        self,
        hidden_size: int,
        num_attention_heads: int,
        layer_norm_eps: float,
        layer_id: int = 0,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()

        self.self_attn = BertSelfAttention(
            hidden_size=hidden_size,
            num_attention_heads=num_attention_heads,
            layer_id=layer_id,
            quant_config=quant_config,
            prefix=f"{prefix}.output",
        )

        self.output = BertSelfOutput(
            hidden_size=hidden_size,
            layer_norm_eps=layer_norm_eps,
            quant_config=quant_config,
            prefix=f"{prefix}.output",
        )

    def forward(
        self, hidden_states: torch.Tensor, forward_batch: ForwardBatch
    ) -> torch.Tensor:
        self_output = self.self_attn(hidden_states, forward_batch)
        return self.output(self_output, hidden_states)
```
**EN:** This class defines Bert Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Bert Attention，用于封装该模型组件的状态与方法。

### Lines 211-264: Class: BertSelfAttention / 类：BertSelfAttention
```python
class BertSelfAttention(nn.Module):

    def __init__(
        self,
        hidden_size: int,
        num_attention_heads: int,
        layer_id: int = 0,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = hidden_size
        tp_size = get_tensor_model_parallel_world_size()

        self.total_num_heads = num_attention_heads
        assert self.total_num_heads % tp_size == 0

        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = self.total_num_heads
        self.head_dim = self.hidden_size // self.total_num_heads
        assert self.head_dim * self.total_num_heads == self.hidden_size

        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)

        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scaling = self.head_dim**-0.5
        self.qkv_proj = QKVParallelLinear(
            hidden_size=self.hidden_size,
            head_size=self.head_dim,
            total_num_heads=self.total_num_heads,
            total_num_kv_heads=self.total_num_kv_heads,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.qkv_proj",
        )

        self.attn = RadixAttention(
            num_heads=self.num_heads,
            head_dim=self.head_dim,
# ... truncated for brevity ...
```
**EN:** This class defines Bert Self Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Bert Self Attention，用于封装该模型组件的状态与方法。

### Lines 267-291: Class: BertSelfOutput / 类：BertSelfOutput
```python
class BertSelfOutput(nn.Module):

    def __init__(
        self,
        hidden_size: int,
        layer_norm_eps: float,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.dense = RowParallelLinear(
            input_size=hidden_size,
            output_size=hidden_size,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.dense",
        )
        self.LayerNorm = nn.LayerNorm(hidden_size, eps=layer_norm_eps)

    def forward(
        self, hidden_states: torch.Tensor, input_tensor: torch.Tensor
    ) -> torch.Tensor:
        hidden_states, _ = self.dense(hidden_states)
        hidden_states = self.LayerNorm(hidden_states + input_tensor)
        return hidden_states
```
**EN:** This class defines Bert Self Output inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Bert Self Output，用于封装该模型组件的状态与方法。

### Lines 294-317: Class: BertIntermediate / 类：BertIntermediate
```python
class BertIntermediate(nn.Module):

    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.dense = ColumnParallelLinear(
            input_size=hidden_size,
            output_size=intermediate_size,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.dense",
        )
        self.intermediate_act_fn = get_act_fn(hidden_act)

    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        hidden_states, _ = self.dense(hidden_states)
        hidden_states = self.intermediate_act_fn(hidden_states)
        return hidden_states
```
**EN:** This class defines Bert Intermediate inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Bert Intermediate，用于封装该模型组件的状态与方法。

### Lines 320-347: Class: BertOutput / 类：BertOutput
```python
class BertOutput(nn.Module):

    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        layer_norm_eps: float,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()

        self.dense = RowParallelLinear(
            input_size=intermediate_size,
            output_size=hidden_size,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.dense",
        )

        self.LayerNorm = nn.LayerNorm(hidden_size, eps=layer_norm_eps)

    def forward(
        self, hidden_states: torch.Tensor, input_tensor: torch.Tensor
    ) -> torch.Tensor:
        hidden_states, _ = self.dense(hidden_states)
        hidden_states = self.LayerNorm(hidden_states + input_tensor)
        return hidden_states
```
**EN:** This class defines Bert Output inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Bert Output，用于封装该模型组件的状态与方法。

### Lines 350-436: Class: BertModel / 类：BertModel
```python
class BertModel(nn.Module):

    def __init__(
        self,
        *,
        config: BertConfig,
        quant_config: Optional[QuantizationConfig] = None,
        use_bert_pooler: bool = False,
        prefix: str = "",
    ):
        super().__init__()
        self.use_bert_pooler = use_bert_pooler
        self.config = config
        self.embeddings = BertEmbedding(config)
        self.encoder = BertEncoder(
            config=config,
            quant_config=quant_config,
            prefix=add_prefix("encoder", prefix),
        )
        pooling_type = (
            PoolingType.CLS
            if get_global_server_args().is_embedding
            else PoolingType.LAST
        )
        self.pooler = (
            BertPooler(config)
            if self.use_bert_pooler
            else Pooler(pooling_type=pooling_type, normalize=True)
        )

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
# ... truncated for brevity ...
```
**EN:** This class defines Bert Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Bert Model，用于封装该模型组件的状态与方法。

### Lines 439-440: Class: Contriever / 类：Contriever
```python
class Contriever(BertModel):
    pass
```
**EN:** This class defines Contriever inheriting from BertModel, grouping state and methods for this model component.
**CN:** 该类定义了 Contriever，用于封装该模型组件的状态与方法。

### Lines 443-501: Class: BertForSequenceClassification / 类：BertForSequenceClassification
```python
class BertForSequenceClassification(nn.Module):

    def __init__(
        self,
        *,
        config: BertConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()

        self.num_labels = config.num_labels
        self.bert = BertModel(
            config=config,
            quant_config=quant_config,
            use_bert_pooler=True,
            prefix=add_prefix("bert", prefix),
        )
        self.classifier = nn.Linear(config.hidden_size, config.num_labels)
        self.pooler = CrossEncodingPooler(config, self.classifier, self.bert.pooler)

    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        self_weights = []

        def weight_filter():
            for name, weight in weights:
                if name.startswith("bert."):
                    yield (name[len("bert.") :], weight)
                else:
                    self_weights.append((name, weight))

        self.bert.load_weights(weight_filter())

        params_dict = dict(self.named_parameters())

        for name, loaded_weight in self_weights:
            if name.startswith("classifier"):
                param = params_dict[name]
                weight_loader = getattr(param, "weight_loader", default_weight_loader)
                weight_loader(param, loaded_weight)
# ... truncated for brevity ...
```
**EN:** This class defines Bert For Sequence Classification inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Bert For Sequence Classification，用于封装该模型组件的状态与方法。

### Lines 504-504: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = [BertModel, Contriever, BertForSequenceClassification]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `typing: Iterable, Optional, Set, Tuple`
- `torch`
- `torch: nn`
- `sglang.srt.distributed: get_tensor_model_parallel_world_size`
- `sglang.srt.layers.activation: get_act_fn`
- `sglang.srt.layers.linear: ColumnParallelLinear, QKVParallelLinear, RowParallelLinear`
- `sglang.srt.layers.pooler: CrossEncodingPooler, Pooler, PoolingType`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.layers.radix_attention: AttentionType, RadixAttention`
- `sglang.srt.layers.vocab_parallel_embedding: VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch`
- `sglang.srt.model_loader.weight_utils: default_weight_loader`
- `sglang.srt.server_args: get_global_server_args`
- `sglang.srt.utils: add_prefix`
