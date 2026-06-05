# granitemoe.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/granitemoe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Granitemoe architecture into SGLang's serving runtime. Inference-only GraniteMoe model. / 该模块将 Granitemoe 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：Inference-only GraniteMoe model。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Module header and imports / 模块头与导入
```python
"""Inference-only GraniteMoe model."""
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 32-89: Class: GraniteMoeMoE / 类：GraniteMoeMoE
```python
class GraniteMoeMoE(nn.Module):
    """A tensor-parallel MoE implementation for GraniteMoe that shards each
    expert across all ranks.
    Each expert's weights are sharded across all ranks and a fused MoE
    kernel is used for the forward pass, and finally we reduce the outputs
    across ranks.
    """

    def __init__(
        self,
        num_experts: int,
        top_k: int,
        hidden_size: int,
        intermediate_size: int,
        layer_id: int,
        params_dtype: Optional[torch.dtype] = None,
        quant_config: Optional[QuantizationConfig] = None,
        tp_size: Optional[int] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = hidden_size

        # Gate always runs at half / full precision for now.
        self.gate = ReplicatedLinear(
            hidden_size,
            num_experts,
            bias=False,
            params_dtype=params_dtype,
            quant_config=None,
            prefix=f"{prefix}.gate",
        )

        self.topk = TopK(
            top_k=top_k,
            renormalize=True,
        )

        self.experts = FusedMoE(
            num_experts=num_experts,
# ... truncated for brevity ...
```
**EN:** This class defines Granite Moe Mo E inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: A tensor-parallel MoE implementation for GraniteMoe that shards each.
**CN:** 该类定义了 Granite Moe Mo E，用于封装该模型组件的状态与方法。 文档字符串摘要：A tensor-parallel MoE implementation for GraniteMoe that shards each。

### Lines 92-176: Class: GraniteMoeAttention / 类：GraniteMoeAttention
```python
class GraniteMoeAttention(nn.Module):

    def __init__(
        self,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        max_position: int = 4096 * 32,
        layer_id: int = 0,
        rope_theta: float = 10000,
        quant_config: Optional[QuantizationConfig] = None,
        attention_multiplier: Optional[float] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size
        tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = num_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = num_kv_heads
        if self.total_num_kv_heads >= tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % tp_size == 0
        else:
            # Number of KV heads is less than TP size, so we replicate
            # the KV heads across multiple tensor parallel GPUs.
            assert tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)
        self.head_dim = hidden_size // self.total_num_heads
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scaling = (
            attention_multiplier
            if attention_multiplier is not None
            else self.head_dim**-1
        )
        self.rope_theta = rope_theta

# ... truncated for brevity ...
```
**EN:** This class defines Granite Moe Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Granite Moe Attention，用于封装该模型组件的状态与方法。

### Lines 179-239: Class: GraniteMoeDecoderLayer / 类：GraniteMoeDecoderLayer
```python
class GraniteMoeDecoderLayer(nn.Module):

    def __init__(
        self,
        config: GraniteConfig,
        layer_id: int = 0,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        rope_theta = config.rope_parameters["rope_theta"]
        self.self_attn = GraniteMoeAttention(
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            max_position=config.max_position_embeddings,
            num_kv_heads=config.num_key_value_heads,
            rope_theta=rope_theta,
            layer_id=layer_id,
            quant_config=quant_config,
            prefix=f"{prefix}.self_attn",
            attention_multiplier=config.attention_multiplier,
        )
        self.block_sparse_moe = GraniteMoeMoE(
            num_experts=config.num_local_experts,
            top_k=config.num_experts_per_tok,
            hidden_size=config.hidden_size,
            intermediate_size=config.intermediate_size,
            layer_id=layer_id,
            quant_config=quant_config,
            prefix=f"{prefix}.block_sparse_moe",
        )

        self.input_layernorm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.post_attention_layernorm = RMSNorm(
            config.hidden_size, eps=config.rms_norm_eps
        )

        self.residual_multiplier = config.residual_multiplier

# ... truncated for brevity ...
```
**EN:** This class defines Granite Moe Decoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Granite Moe Decoder Layer，用于封装该模型组件的状态与方法。

### Lines 242-295: Class: GraniteMoeModel / 类：GraniteMoeModel
```python
class GraniteMoeModel(nn.Module):

    def __init__(
        self,
        config: GraniteConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.embed_tokens = VocabParallelEmbedding(
            config.vocab_size,
            config.hidden_size,
            org_num_embeddings=config.vocab_size,
        )
        self.embedding_multiplier = config.embedding_multiplier

        self.layers = nn.ModuleList(
            [
                GraniteMoeDecoderLayer(
                    config,
                    i,
                    quant_config=quant_config,
                    prefix=add_prefix(f"layers.{i}", prefix),
                )
                for i in range(config.num_hidden_layers)
            ]
        )
        self.norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)

    def get_input_embeddings(self, input_ids: torch.Tensor) -> torch.Tensor:
        return self.embed_tokens(input_ids)

    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        inputs_embeds: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        if inputs_embeds is not None:
# ... truncated for brevity ...
```
**EN:** This class defines Granite Moe Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Granite Moe Model，用于封装该模型组件的状态与方法。

### Lines 298-384: Class: GraniteMoeForCausalLM / 类：GraniteMoeForCausalLM
```python
class GraniteMoeForCausalLM(nn.Module):

    def __init__(
        self,
        config: GraniteConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.quant_config = quant_config

        self.model = GraniteMoeModel(
            config, quant_config=quant_config, prefix=add_prefix("model", prefix)
        )
        self.lm_head = ParallelLMHead(
            config.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=add_prefix("lm_head", prefix),
        )
        if config.tie_word_embeddings:
            self.lm_head.weight = self.model.embed_tokens.weight
        # Granite logit scaling factors are applied via division, but
        # LogitsProcessor expects a multiplicative factor.
        if hasattr(config, "logits_scaling"):
            logit_scale = 1.0 / config.logits_scaling
        else:
            logit_scale = None
        self.logits_processor = LogitsProcessor(config, logit_scale=logit_scale)
        self.pooler = Pooler(pooling_type=PoolingType.LAST, normalize=True)

    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: torch.Tensor = None,
        get_embedding: bool = False,
# ... truncated for brevity ...
```
**EN:** This class defines Granite Moe For Causal L M inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Granite Moe For Causal L M，用于封装该模型组件的状态与方法。

### Lines 387-387: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = [GraniteMoeForCausalLM]
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
- `transformers: GraniteConfig`
- `sglang.srt.distributed: get_tensor_model_parallel_world_size`
- `sglang.srt.layers.layernorm: RMSNorm`
- `sglang.srt.layers.linear: QKVParallelLinear, ReplicatedLinear, RowParallelLinear`
- `sglang.srt.layers.logits_processor: LogitsProcessor, LogitsProcessorOutput`
- `sglang.srt.layers.moe.fused_moe_triton: FusedMoE`
- `sglang.srt.layers.moe.topk: TopK`
- `sglang.srt.layers.pooler: Pooler, PoolingType`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.layers.radix_attention: RadixAttention`
- `sglang.srt.layers.rotary_embedding: get_rope`
- `sglang.srt.layers.vocab_parallel_embedding: ParallelLMHead, VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch`
- `sglang.srt.models: mixtral`
- `sglang.srt.utils: add_prefix`
