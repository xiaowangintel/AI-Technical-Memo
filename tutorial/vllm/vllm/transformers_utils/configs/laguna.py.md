# laguna.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/transformers_utils/configs/laguna.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Defines model-specific configuration classes or compatibility helpers for Laguna. / [CN] 为 Laguna 定义模型专用配置类或兼容性辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 4-4: Imports
```python
from transformers.configuration_utils import PretrainedConfig
```
**EN:** This import block assembles the module's dependencies: standard-library helpers such as none, external APIs such as `transformers.configuration_utils`, and internal vLLM modules such as none. Together they define the services the rest of the file builds on.
**CN:** 该导入块汇集了模块依赖：标准库工具如 无，外部 API 如 `transformers.configuration_utils`，以及 vLLM 内部模块如 无。这些依赖共同构成了后续实现所依赖的基础能力。

### Lines 7-117: Class `LagunaConfig`
```python
class LagunaConfig(PretrainedConfig):
    model_type = "laguna"
    keys_to_ignore_at_inference = ["past_key_values"]
    base_model_tp_plan = {
        "layers.*.self_attn.q_proj": "colwise",
        "layers.*.self_attn.k_proj": "colwise",
        "layers.*.self_attn.v_proj": "colwise",
        "layers.*.self_attn.g_proj": "colwise",
        "layers.*.self_attn.o_proj": "rowwise",
        "layers.*.mlp.gate_proj": "colwise",
        "layers.*.mlp.up_proj": "colwise",
        "layers.*.mlp.down_proj": "rowwise",
    }
    base_model_pp_plan = {
        "embed_tokens": (["input_ids"], ["inputs_embeds"]),
        "layers": (["hidden_states", "attention_mask"], ["hidden_states"]),
        "norm": (["hidden_states"], ["hidden_states"]),
    }

    def __init__(
        self,
        vocab_size: int = 100352,
        hidden_size: int = 2048,
        intermediate_size: int = 8192,
        num_hidden_layers: int = 40,
        num_attention_heads: int = 48,
        num_key_value_heads: int = 8,
        head_dim: int = 128,
        qkv_bias: bool = False,
        attention_bias: bool = False,
        gating: bool | str = True,
        hidden_act: str = "silu",
        max_position_embeddings: int = 131072,
        initializer_range: float = 0.02,
        rms_norm_eps: float = 1e-6,
        use_cache: bool = True,
        tie_word_embeddings: bool = False,
        rope_theta: float = 500000.0,
        rope_scaling: dict | None = None,
        rope_parameters: dict | None = None,
# ... omitted for brevity ...
        self.num_experts_per_tok = num_experts_per_tok
        self.moe_intermediate_size = moe_intermediate_size
        self.shared_expert_intermediate_size = shared_expert_intermediate_size
        self.norm_topk_prob = norm_topk_prob
        self.decoder_sparse_step = decoder_sparse_step
        self.mlp_only_layers = mlp_only_layers
        self.router_aux_loss_coef = router_aux_loss_coef
        self.output_router_logits = output_router_logits
        self.moe_routed_scaling_factor = moe_routed_scaling_factor
        self.moe_apply_router_weight_on_input = moe_apply_router_weight_on_input

        super().__init__(tie_word_embeddings=tie_word_embeddings, **kwargs)
```
**EN:** Defines `LagunaConfig`, a model-configuration class derived from `PretrainedConfig`. Key methods include `__init__`.
**CN:** 定义 `LagunaConfig`，这是一个模型配置类，继承自 `PretrainedConfig`。 关键方法包括 `__init__`。

### Lines 120-120: Module state and constants
```python
__all__ = ["LagunaConfig"]
```
**EN:** This block declares the public symbols exported by the module via `__all__`, making the package boundary explicit.
**CN:** 该代码块通过 `__all__` 声明模块对外导出的公共符号，使包的公开边界更加明确。

## Key Concepts / 关键概念
- **EN:** Configuration classes extend upstream Hugging Face metadata with vLLM-specific defaults, validation, or compatibility patches.
- **CN:** 配置类会在上游 Hugging Face 元数据之上加入 vLLM 专用默认值、校验逻辑或兼容补丁。
- **EN:** The module is tightly coupled to Hugging Face Transformers APIs, so version compatibility is often an explicit concern.
- **CN:** 该模块与 Hugging Face Transformers API 紧密耦合，因此版本兼容性通常是显式关注点。

## Dependencies / 依赖关系
- **EN:** Standard library modules: none.
- **CN:** 标准库模块：无。
- **EN:** External packages: `transformers.configuration_utils`.
- **CN:** 外部依赖包：`transformers.configuration_utils`。
- **EN:** Internal modules: none.
- **CN:** 内部模块：无。
