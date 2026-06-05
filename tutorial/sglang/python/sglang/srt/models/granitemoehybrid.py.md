# granitemoehybrid.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/granitemoehybrid.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Granitemoehybrid architecture into SGLang's serving runtime, typically defining layers, model wrappers, and weight-loading behavior for inference. / 该模块将 Granitemoehybrid 架构接入 SGLang 服务运行时，通常定义推理所需的层、模型封装以及权重加载逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Module header and imports / 模块头与导入
```python
from typing import Iterable, Optional
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 40-76: Class: GraniteMoeSharedMLP / 类：GraniteMoeSharedMLP
```python
class GraniteMoeSharedMLP(nn.Module):
    def __init__(
        self,
        config: GraniteMoeSharedConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()

        self.input_size = config.hidden_size
        self.hidden_size = config.shared_intermediate_size
        self.input_linear = MergedColumnParallelLinear(
            input_size=self.input_size,
            output_sizes=[self.hidden_size] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.input_linear",
        )
        self.output_linear = RowParallelLinear(
            self.hidden_size,
            self.input_size,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.output_linear",
        )
        if config.hidden_act != "silu":
            raise ValueError(
                f"Unsupported activation: {config.hidden_act}. "
                "Only silu is supported for now."
            )
        self.act_fn = SiluAndMul()

    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        gate_up, _ = self.input_linear(hidden_states)
        x = self.act_fn(gate_up)
        x, _ = self.output_linear(x)
        return x
```
**EN:** This class defines Granite Moe Shared M L P inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Granite Moe Shared M L P，用于封装该模型组件的状态与方法。

### Lines 79-172: Class: GraniteMoeHybridMambaDecoderLayer / 类：GraniteMoeHybridMambaDecoderLayer
```python
class GraniteMoeHybridMambaDecoderLayer(nn.Module):
    def __init__(
        self,
        config: GraniteMoeHybridConfig,
        layer_idx: int,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.layer_idx = layer_idx
        self.hidden_size = config.hidden_size
        self.residual_multiplier = config.residual_multiplier

        self.mamba = MambaMixer2(
            cache_params=config.mamba2_cache_params,
            hidden_size=config.hidden_size,
            use_conv_bias=config.mamba_conv_bias,
            use_bias=config.mamba_proj_bias,
            n_groups=config.mamba_n_groups,
            rms_norm_eps=config.rms_norm_eps,
            activation=config.hidden_act,
            quant_config=quant_config,
            prefix=f"{prefix}.mixer",
        )

        self.block_sparse_moe = None
        if getattr(config, "num_local_experts", 0) > 0:
            self.block_sparse_moe = GraniteMoeMoE(
                num_experts=config.num_local_experts,
                top_k=config.num_experts_per_tok,
                hidden_size=config.hidden_size,
                intermediate_size=config.intermediate_size,
                layer_id=layer_idx,
                quant_config=quant_config,
                tp_size=get_tensor_model_parallel_world_size(),
                prefix=f"{prefix}.block_sparse_moe",
            )

        self.shared_mlp = (
# ... truncated for brevity ...
```
**EN:** This class defines Granite Moe Hybrid Mamba Decoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Granite Moe Hybrid Mamba Decoder Layer，用于封装该模型组件的状态与方法。

### Lines 175-269: Class: GraniteMoeHybridAttention / 类：GraniteMoeHybridAttention
```python
class GraniteMoeHybridAttention(nn.Module):
    def __init__(
        self,
        config: GraniteMoeHybridConfig,
        layer_id: int,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.causal = True
        self.hidden_size = config.hidden_size
        self.attention_bias = config.attention_bias
        self.attention_multiplier = config.attention_multiplier
        self.total_num_heads = config.num_attention_heads
        self.head_dim = self.hidden_size // self.total_num_heads
        self.total_num_kv_heads = config.num_key_value_heads

        # TensorParallel logic
        tp_size = get_tensor_model_parallel_world_size()
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        if self.total_num_kv_heads >= tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % tp_size == 0
        else:
            # Number of KV heads is less than TP size, so we replicate
            # the KV heads across multiple tensor parallel GPUs.
            assert tp_size % self.total_num_kv_heads == 0
        self.num_key_value_heads = max(1, self.total_num_kv_heads // tp_size)

        self.qkv_proj = QKVParallelLinear(
            self.hidden_size,
            self.head_dim,
            self.total_num_heads,
            self.total_num_kv_heads,
            bias=self.attention_bias,
            quant_config=quant_config,
            prefix=f"{prefix}.qkv_proj",
        )
# ... truncated for brevity ...
```
**EN:** This class defines Granite Moe Hybrid Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Granite Moe Hybrid Attention，用于封装该模型组件的状态与方法。

### Lines 272-351: Class: GraniteMoeHybridAttentionDecoderLayer / 类：GraniteMoeHybridAttentionDecoderLayer
```python
class GraniteMoeHybridAttentionDecoderLayer(nn.Module):
    def __init__(
        self,
        config: GraniteMoeHybridConfig,
        layer_idx: int,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        self.residual_multiplier = config.residual_multiplier

        self.self_attn = GraniteMoeHybridAttention(
            config,
            layer_id=layer_idx,
            quant_config=quant_config,
            prefix=f"{prefix}.self_attn",
        )

        self.block_sparse_moe = None
        if getattr(config, "num_local_experts", 0) > 0:
            self.block_sparse_moe = GraniteMoeMoE(
                num_experts=config.num_local_experts,
                top_k=config.num_experts_per_tok,
                hidden_size=config.hidden_size,
                intermediate_size=config.intermediate_size,
                layer_id=layer_idx,
                quant_config=quant_config,
                tp_size=get_tensor_model_parallel_world_size(),
                prefix=f"{prefix}.block_sparse_moe",
            )

        self.shared_mlp = (
            None
            if getattr(config, "shared_intermediate_size", 0) == 0
            else GraniteMoeSharedMLP(
                config, quant_config=quant_config, prefix=f"{prefix}.shared_mlp"
            )
        )

# ... truncated for brevity ...
```
**EN:** This class defines Granite Moe Hybrid Attention Decoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Granite Moe Hybrid Attention Decoder Layer，用于封装该模型组件的状态与方法。

### Lines 354-357: Assignment: ALL_DECODER_LAYER_TYPES / 赋值：ALL_DECODER_LAYER_TYPES
```python
ALL_DECODER_LAYER_TYPES = {
    "attention": GraniteMoeHybridAttentionDecoderLayer,
    "mamba": GraniteMoeHybridMambaDecoderLayer,
}
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 360-460: Class: GraniteMoeHybridModel / 类：GraniteMoeHybridModel
```python
class GraniteMoeHybridModel(nn.Module):
    def __init__(
        self,
        config: GraniteMoeHybridConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()

        self.config = config
        self.quant_config = quant_config

        self.vocab_size = config.vocab_size

        self.pp_group = get_pp_group()

        if self.pp_group.is_first_rank:
            self.embed_tokens = VocabParallelEmbedding(
                self.vocab_size,
                config.hidden_size,
                org_num_embeddings=config.vocab_size,
            )
        else:
            self.embed_tokens = PPMissingLayer()

        self.embedding_multiplier = config.embedding_multiplier

        def get_layer(idx: int, prefix: str):
            layer_idx = int(prefix.rsplit(".", 1)[1])
            layer_class = ALL_DECODER_LAYER_TYPES[config.layer_types[layer_idx]]
            return layer_class(
                config,
                layer_idx,
                quant_config=quant_config,
                prefix=prefix,
            )

        self.layers, self.start_layer, self.end_layer = make_layers(
            config.num_hidden_layers,
            get_layer,
# ... truncated for brevity ...
```
**EN:** This class defines Granite Moe Hybrid Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Granite Moe Hybrid Model，用于封装该模型组件的状态与方法。

### Lines 463-734: Class: GraniteMoeHybridForCausalLM / 类：GraniteMoeHybridForCausalLM
```python
class GraniteMoeHybridForCausalLM(
    nn.Module,
):
    packed_modules_mapping = {
        "qkv_proj": [
            "q_proj",
            "k_proj",
            "v_proj",
        ],
        "conv1d": ["conv1d"],
        "in_proj": ["in_proj"],
        "input_linear": ["input_linear"],
    }
    embedding_modules = {
        "embed_tokens": "input_embeddings",
        "lm_head": "output_embeddings",
    }

    def __init__(
        self,
        config: GraniteMoeHybridConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()

        self.capture_aux_hidden_states = False
        self.pp_group = get_pp_group()

        self.quant_config = quant_config
        self.config = config
        self.model = GraniteMoeHybridModel(
            config=config,
            quant_config=quant_config,
            prefix=maybe_prefix(prefix, "model"),
        )

        self.lm_head = ParallelLMHead(
            config.vocab_size,
            config.hidden_size,
# ... truncated for brevity ...
```
**EN:** This class defines Granite Moe Hybrid For Causal L M inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Granite Moe Hybrid For Causal L M，用于封装该模型组件的状态与方法。

### Lines 737-737: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = [GraniteMoeHybridForCausalLM]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `typing: Iterable, Optional`
- `torch`
- `torch: nn`
- `transformers.models.granitemoeshared: GraniteMoeSharedConfig`
- `sglang.srt.configs.granitemoehybrid: GraniteMoeHybridConfig`
- `sglang.srt.distributed: get_pp_group, get_tensor_model_parallel_world_size`
- `sglang.srt.layers.activation: SiluAndMul`
- `sglang.srt.layers.attention.hybrid_linear_attn_backend: HybridLinearAttnBackend, Mamba2AttnBackend`
- `sglang.srt.layers.attention.mamba.mamba: MambaMixer2`
- `sglang.srt.layers.layernorm: RMSNorm`
- `sglang.srt.layers.linear: MergedColumnParallelLinear, QKVParallelLinear, RowParallelLinear`
- `sglang.srt.layers.logits_processor: LogitsProcessor`
- `sglang.srt.layers.pooler: Pooler, PoolingType`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.layers.radix_attention: RadixAttention`
- `sglang.srt.layers.rotary_embedding: get_rope`
- `sglang.srt.layers.utils: PPMissingLayer`
- `sglang.srt.layers.vocab_parallel_embedding: ParallelLMHead, VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch, PPProxyTensors`
- `sglang.srt.model_loader.weight_utils: default_weight_loader`
