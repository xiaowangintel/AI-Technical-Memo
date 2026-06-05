# gemma3n_causal.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/gemma3n_causal.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the gemma3n Causal architecture into SGLang's serving runtime, typically defining layers, model wrappers, and weight-loading behavior for inference. / 该模块将 gemma3n Causal 架构接入 SGLang 服务运行时，通常定义推理所需的层、模型封装以及权重加载逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Module header and imports / 模块头与导入
```python
from typing import Iterable, Optional, Set, Tuple
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 34-35: Function: get_attention_sliding_window_size() / 函数：get_attention_sliding_window_size()
```python
def get_attention_sliding_window_size(config):
    return config.sliding_window - 1
```
**EN:** This function implements get attention sliding window size for the surrounding model/runtime logic. Key parameters include config.
**CN:** 该函数实现了 get attention sliding window size 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 38-59: Class: Gemma3nRMSNorm / 类：Gemma3nRMSNorm
```python
class Gemma3nRMSNorm(RMSNorm):
    def __init__(
        self,
        dim: int,
        eps: float = 1e-6,
        with_scale: bool = True,
    ) -> None:
        super().__init__(dim, eps=eps)
        if not with_scale:
            del self.weight
            self.register_buffer(
                "weight",
                torch.ones(dim, dtype=torch.get_default_dtype()),
                persistent=False,
            )

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        original_shape = x.shape
        x_2d = x.contiguous().reshape(-1, original_shape[-1])
        x_2d = super().forward(x_2d)
        x = x_2d.reshape(original_shape)
        return x
```
**EN:** This class defines Gemma3n R M S Norm inheriting from RMSNorm, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma3n R M S Norm，用于封装该模型组件的状态与方法。

### Lines 62-63: Class: Gemma3nTextScaledWordEmbedding / 类：Gemma3nTextScaledWordEmbedding
```python
class Gemma3nTextScaledWordEmbedding(Gemma3TextScaledWordEmbedding):
    pass
```
**EN:** This class defines Gemma3n Text Scaled Word Embedding inheriting from Gemma3TextScaledWordEmbedding, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma3n Text Scaled Word Embedding，用于封装该模型组件的状态与方法。

### Lines 66-130: Class: Gemma3nTextMLP / 类：Gemma3nTextMLP
```python
class Gemma3nTextMLP(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_activation: str,
        activation_sparsity: float = 0.0,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.gate_up_proj = MergedColumnParallelLinear(
            hidden_size,
            [intermediate_size] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("gate_up_proj", prefix),
        )
        self.down_proj = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("down_proj", prefix),
        )
        if hidden_activation != "gelu_pytorch_tanh":
            raise ValueError(
                "Gemma3n uses `gelu_pytorch_tanh` as the hidden activation "
                "function. Please set `hidden_activation` to "
                "`gelu_pytorch_tanh`."
            )
        # Use proper GELU with tanh approximation as specified
        self.act_fn = GeluAndMul()
        self.activation_sparsity = activation_sparsity
        self.register_buffer(
            "target_sparsity_tensor",
            torch.tensor(self.activation_sparsity, dtype=torch.float32),
            persistent=False,
        )  # moved from _gaussian_topk for cuda graph

# ... truncated for brevity ...
```
**EN:** This class defines Gemma3n Text M L P inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma3n Text M L P，用于封装该模型组件的状态与方法。

### Lines 133-169: Class: Gemma3nLaurelBlock / 类：Gemma3nLaurelBlock
```python
class Gemma3nLaurelBlock(nn.Module):
    """Learned Augmented Residual Layer"""

    def __init__(
        self,
        config: Gemma3nTextConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config

        self.linear_left = ColumnParallelLinear(
            config.hidden_size,
            config.laurel_rank,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("linear_left", prefix),
        )
        self.linear_right = RowParallelLinear(
            config.laurel_rank,
            config.hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("linear_right", prefix),
        )
        self.post_laurel_norm = Gemma3nRMSNorm(
            dim=config.hidden_size,
            eps=config.rms_norm_eps,
        )

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        # [num_tokens, hidden_size]
        laurel_x, _ = self.linear_left(x)
        laurel_x, _ = self.linear_right(laurel_x)
        normed_laurel_x = self.post_laurel_norm(laurel_x)
        return x + normed_laurel_x
```
**EN:** This class defines Gemma3n Laurel Block inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: Learned Augmented Residual Layer.
**CN:** 该类定义了 Gemma3n Laurel Block，用于封装该模型组件的状态与方法。 文档字符串摘要：Learned Augmented Residual Layer。

### Lines 172-311: Class: Gemma3nAltUp / 类：Gemma3nAltUp
```python
class Gemma3nAltUp(nn.Module):
    """Alternating Updates (AltUp)"""

    def __init__(
        self,
        config: Gemma3nTextConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config

        self.correct_output_scale = nn.Parameter(
            torch.zeros(config.hidden_size, dtype=torch.float32)
        )
        self.correction_coefs = ReplicatedLinear(
            config.altup_num_inputs,
            config.altup_num_inputs,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("correction_coefs", prefix),
        )
        self.prediction_coefs = ReplicatedLinear(
            config.altup_num_inputs,
            config.altup_num_inputs**2,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("prediction_coefs", prefix),
        )
        self.modality_router = ReplicatedLinear(
            config.hidden_size,
            config.altup_num_inputs,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("modality_router", prefix),
        )

        self.router_norm = Gemma3nRMSNorm(
            dim=config.hidden_size,
            eps=config.rms_norm_eps,
# ... truncated for brevity ...
```
**EN:** This class defines Gemma3n Alt Up inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: Alternating Updates (AltUp).
**CN:** 该类定义了 Gemma3n Alt Up，用于封装该模型组件的状态与方法。 文档字符串摘要：Alternating Updates (AltUp)。

### Lines 314-493: Class: Gemma3nAttention / 类：Gemma3nAttention
```python
class Gemma3nAttention(nn.Module):
    """Multi-headed attention from 'Attention Is All You Need' paper"""

    def __init__(
        self,
        layer_id: int,
        config: Gemma3nTextConfig,
        max_position_embeddings: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.layer_id = layer_id
        self.config = config
        tp_size = get_tensor_model_parallel_world_size()

        self.total_num_heads = config.num_attention_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = config.num_key_value_heads

        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)

        if self.total_num_kv_heads >= tp_size:
            assert self.total_num_kv_heads % tp_size == 0
        else:
            assert tp_size % self.total_num_kv_heads == 0

        hidden_size = config.hidden_size
        head_dim = getattr(
            config, "head_dim", hidden_size // config.num_attention_heads
        )
        self.head_dim = head_dim

        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        # self.scaling = config.query_rescale_scalar / config.query_pre_attn_scalar
        self.scaling = 1.0

        self.qkv_proj = QKVParallelLinear(
# ... truncated for brevity ...
```
**EN:** This class defines Gemma3n Attention inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: Multi-headed attention from 'Attention Is All You Need' paper.
**CN:** 该类定义了 Gemma3n Attention，用于封装该模型组件的状态与方法。 文档字符串摘要：Multi-headed attention from 'Attention Is All You Need' paper。

### Lines 496-626: Class: Gemma3nDecoderLayer / 类：Gemma3nDecoderLayer
```python
class Gemma3nDecoderLayer(nn.Module):
    def __init__(
        self,
        layer_id: int,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        self.layer_id = layer_id
        self.attention_type = config.layer_types[layer_id]
        self.config = config

        self.self_attn = Gemma3nAttention(
            layer_id=layer_id,
            config=config,
            max_position_embeddings=config.max_position_embeddings,
            quant_config=quant_config,
            prefix=add_prefix("self_attn", prefix),
        )

        intermediate_size = config.intermediate_size[layer_id]
        activation_sparsity = config.activation_sparsity_pattern[layer_id]
        self.mlp = Gemma3nTextMLP(
            hidden_size=self.hidden_size,
            intermediate_size=intermediate_size,
            hidden_activation=config.hidden_activation,
            activation_sparsity=activation_sparsity,
            quant_config=quant_config,
            prefix=add_prefix("mlp", prefix),
        )

        self.input_layernorm = Gemma3nRMSNorm(self.hidden_size, eps=config.rms_norm_eps)
        self.post_attention_layernorm = Gemma3nRMSNorm(
            self.hidden_size, eps=config.rms_norm_eps
        )
        self.pre_feedforward_layernorm = Gemma3nRMSNorm(
            self.hidden_size, eps=config.rms_norm_eps
        )
# ... truncated for brevity ...
```
**EN:** This class defines Gemma3n Decoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma3n Decoder Layer，用于封装该模型组件的状态与方法。

### Lines 629-847: Class: Gemma3nTextModel / 类：Gemma3nTextModel
```python
class Gemma3nTextModel(PreTrainedModel):
    def __init__(
        self,
        config: Gemma3nTextConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__(config=config)
        self.config = config
        self.quant_config = quant_config
        self.vocab_size = config.vocab_size
        self.padding_idx = config.pad_token_id

        # Gemma3n downcasts the below to float16, causing sqrt(3072)=55.4256 to become 55.5
        self.embed_tokens = Gemma3nTextScaledWordEmbedding(
            config.vocab_size,
            config.hidden_size,
            self.padding_idx,
            embed_scale=self.config.hidden_size**0.5,
        )

        self.norm = Gemma3nRMSNorm(
            config.hidden_size,
            eps=config.rms_norm_eps,
        )

        self.layers = make_layers(
            config.num_hidden_layers,
            lambda idx, prefix: Gemma3nDecoderLayer(
                layer_id=idx,
                config=config,
                quant_config=quant_config,
                prefix=prefix,
            ),
            prefix=add_prefix("layers", prefix),
        )

        # Per-layer input embeddings
        self.hidden_size = config.hidden_size
        self.hidden_size_per_layer_input = config.hidden_size_per_layer_input
# ... truncated for brevity ...
```
**EN:** This class defines Gemma3n Text Model inheriting from PreTrainedModel, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma3n Text Model，用于封装该模型组件的状态与方法。

### Lines 850-1007: Class: Gemma3nForCausalLM / 类：Gemma3nForCausalLM
```python
class Gemma3nForCausalLM(PreTrainedModel):
    config_class = Gemma3nTextConfig

    _tied_weights_keys = {"lm_head.weight": "model.embed_tokens.weight"}
    _tp_plan = {"lm_head": "colwise_rep"}
    _pp_plan = {"lm_head": (["hidden_states"], ["logits"])}
    config_class = Gemma3nTextConfig
    base_model_prefix = "language_model"

    # BitandBytes specific attributes
    default_bitsandbytes_target_modules = [
        ".gate_proj.",
        ".down_proj.",
        ".up_proj.",
        ".q_proj.",
        ".k_proj.",
        ".v_proj.",
        ".o_proj.",
    ]
    bitsandbytes_stacked_params_mapping = {
        ".q_proj": (".qkv_proj", 0),
        ".k_proj": (".qkv_proj", 1),
        ".v_proj": (".qkv_proj", 2),
        ".gate_proj": (".gate_up_proj", 0),
        ".up_proj": (".gate_up_proj", 1),
    }

    packed_modules_mapping = {
        ".qkv_proj": [
            ".q_proj",
            ".k_proj",
            ".v_proj",
        ],
        ".gate_up_proj": [
            ".gate_proj",
            ".up_proj",
        ],
    }

    # LoRA specific attributes
# ... truncated for brevity ...
```
**EN:** This class defines Gemma3n For Causal L M inheriting from PreTrainedModel, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma3n For Causal L M，用于封装该模型组件的状态与方法。

### Lines 1010-1010: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = Gemma3nForCausalLM
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 1011-1011: Code block / 代码块
```python
AutoModel.register(Gemma3nTextConfig, Gemma3nForCausalLM, exist_ok=True)
```
**EN:** This block provides supporting module logic.
**CN:** 该代码块提供辅助性的模块逻辑。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `typing: Iterable, Optional, Set, Tuple`
- `torch`
- `torch.nn.functional`
- `torch: nn`
- `transformers: AutoModel, Gemma3nTextConfig, PretrainedConfig, PreTrainedModel`
- `sglang.srt.distributed: get_tensor_model_parallel_world_size`
- `sglang.srt.layers.activation: GeluAndMul`
- `sglang.srt.layers.layernorm: RMSNorm`
- `sglang.srt.layers.linear: ColumnParallelLinear, MergedColumnParallelLinear, QKVParallelLinear, ReplicatedLinear, RowParallelLinear`
- `sglang.srt.layers.logits_processor: LogitsProcessor`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.layers.radix_attention: RadixAttention`
- `sglang.srt.layers.rotary_embedding: get_rope`
- `sglang.srt.layers.vocab_parallel_embedding: ParallelLMHead`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch`
- `sglang.srt.model_loader.weight_utils: default_weight_loader, maybe_remap_kv_scale_name`
- `sglang.srt.models.gemma3_causal: Gemma3TextScaledWordEmbedding`
- `sglang.srt.utils: add_prefix, make_layers`
