# qwen3_5_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/transformers_utils/configs/qwen3_5_moe.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Qwen3.5-MoE model configuration. / [CN] 定义 Qwen3 5 MOE 的配置类或兼容辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 17-17: Module overview
```python
"""Qwen3.5-MoE model configuration"""
```
**EN:** The module docstring frames the file's responsibility and intended usage. Its opening line is: Qwen3.5-MoE model configuration
**CN:** 模块文档字符串概述了该文件的职责与预期使用方式。

### Lines 19-19: Imports
```python
from transformers.configuration_utils import PretrainedConfig
```
**EN:** This import block assembles the module's dependencies: standard-library helpers such as none, external APIs such as `transformers.configuration_utils`, and internal vLLM modules such as none. Together they define the services the rest of the file builds on.
**CN:** 该导入块汇集了模块依赖：标准库工具如 无，外部 API 如 `transformers.configuration_utils`，以及 vLLM 内部模块如 无。这些依赖共同构成了后续实现所依赖的基础能力。

### Lines 22-135: Class `Qwen3_5MoeTextConfig`
```python
class Qwen3_5MoeTextConfig(PretrainedConfig):
    model_type = "qwen3_5_moe_text"
    keys_to_ignore_at_inference = ["past_key_values"]

    base_model_tp_plan = {
        "layers.*.self_attn.q_proj": "colwise",
        "layers.*.self_attn.k_proj": "colwise",
        "layers.*.self_attn.v_proj": "colwise",
        "layers.*.self_attn.o_proj": "rowwise",
        "layers.*.mlp.experts.gate_up_proj": "packed_colwise",
        "layers.*.mlp.experts.down_proj": "rowwise",
        "layers.*.mlp.shared_expert.gate_proj": "colwise",
        "layers.*.mlp.shared_expert.up_proj": "colwise",
        "layers.*.mlp.shared_expert.down_proj": "rowwise",
    }
    base_model_pp_plan = {
        "embed_tokens": (["input_ids"], ["inputs_embeds"]),
        "layers": (["hidden_states", "attention_mask"], ["hidden_states"]),
        "norm": (["hidden_states"], ["hidden_states"]),
    }
    base_config_key = "text_config"

    def __init__(
        self,
        vocab_size=248320,
        hidden_size=2048,
        num_hidden_layers=40,
        num_attention_heads=16,
        num_key_value_heads=2,
        hidden_act="silu",
        max_position_embeddings=32768,
        initializer_range=0.02,
        rms_norm_eps=1e-6,
        use_cache=True,
        tie_word_embeddings=False,
        rope_parameters=None,
        attention_bias=False,
        attention_dropout=0.0,
        head_dim=256,
        linear_conv_kernel_dim=4,
# ... omitted for brevity ...
        self.num_experts_per_tok = num_experts_per_tok
        self.num_experts = num_experts
        self.output_router_logits = output_router_logits
        self.router_aux_loss_coef = router_aux_loss_coef
        super().__init__(**kwargs)
        # Set these AFTER super().__init__() because transformers v4's
        # PretrainedConfig.__init__ has these as explicit params with different
        # defaults (e.g. tie_word_embeddings=True) that would overwrite our values.
        self.pad_token_id = pad_token_id
        self.bos_token_id = bos_token_id
        self.eos_token_id = eos_token_id
        self.tie_word_embeddings = tie_word_embeddings
```
**EN:** Defines `Qwen3_5MoeTextConfig`, a model-configuration class derived from `PretrainedConfig`. Key methods include `__init__`.
**CN:** 定义 `Qwen3_5MoeTextConfig`，这是一个模型配置类，继承自 `PretrainedConfig`。 关键方法包括 `__init__`。

### Lines 138-171: Class `Qwen3_5MoeVisionConfig`
```python
class Qwen3_5MoeVisionConfig(PretrainedConfig):
    model_type = "qwen3_5_moe"
    base_config_key = "vision_config"

    def __init__(
        self,
        depth=27,
        hidden_size=1152,
        hidden_act="gelu_pytorch_tanh",
        intermediate_size=4304,
        num_heads=16,
        in_channels=3,
        patch_size=16,
        spatial_merge_size=2,
        temporal_patch_size=2,
        out_hidden_size=3584,
        num_position_embeddings=2304,
        initializer_range=0.02,
        **kwargs,
    ):
        super().__init__(**kwargs)

        self.depth = depth
        self.hidden_size = hidden_size
        self.hidden_act = hidden_act
        self.intermediate_size = intermediate_size
        self.num_heads = num_heads
        self.in_channels = in_channels
        self.patch_size = patch_size
        self.spatial_merge_size = spatial_merge_size
        self.temporal_patch_size = temporal_patch_size
        self.out_hidden_size = out_hidden_size
        self.num_position_embeddings = num_position_embeddings
        self.initializer_range = initializer_range
```
**EN:** Defines `Qwen3_5MoeVisionConfig`, a model-configuration class derived from `PretrainedConfig`. Key methods include `__init__`.
**CN:** 定义 `Qwen3_5MoeVisionConfig`，这是一个模型配置类，继承自 `PretrainedConfig`。 关键方法包括 `__init__`。

### Lines 174-209: Class `Qwen3_5MoeConfig`
```python
class Qwen3_5MoeConfig(PretrainedConfig):
    model_type = "qwen3_5_moe"
    sub_configs = {
        "vision_config": Qwen3_5MoeVisionConfig,
        "text_config": Qwen3_5MoeTextConfig,
    }
    keys_to_ignore_at_inference = ["past_key_values"]

    def __init__(
        self,
        text_config=None,
        vision_config=None,
        image_token_id=248056,
        video_token_id=248057,
        vision_start_token_id=248053,
        vision_end_token_id=248054,
        tie_word_embeddings=False,
        **kwargs,
    ):
        if isinstance(vision_config, dict):
            self.vision_config = self.sub_configs["vision_config"](**vision_config)
        elif vision_config is None:
            self.vision_config = self.sub_configs["vision_config"]()

        if isinstance(text_config, dict):
            self.text_config = self.sub_configs["text_config"](**text_config)
        elif text_config is None:
            self.text_config = self.sub_configs["text_config"]()

        self.image_token_id = image_token_id
        self.video_token_id = video_token_id
        self.vision_start_token_id = vision_start_token_id
        self.vision_end_token_id = vision_end_token_id
        super().__init__(**kwargs)
        # Set after super().__init__() to avoid v4 PretrainedConfig overwrite
        self.tie_word_embeddings = tie_word_embeddings
```
**EN:** Defines `Qwen3_5MoeConfig`, a model-configuration class derived from `PretrainedConfig`. Key methods include `__init__`.
**CN:** 定义 `Qwen3_5MoeConfig`，这是一个模型配置类，继承自 `PretrainedConfig`。 关键方法包括 `__init__`。

### Lines 212-212: Module state and constants
```python
__all__ = ["Qwen3_5MoeConfig", "Qwen3_5MoeTextConfig"]
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
