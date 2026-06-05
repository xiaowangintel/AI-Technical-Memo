# qwen3_omni.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/qwen3_omni.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for qwen3 omni so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 qwen3 omni 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-4: Imports dependencies
```python
from transformers import PretrainedConfig
from transformers.configuration_utils import layer_type_validation

from sglang.utils import logger
```
**EN:** This block groups related imports for the module, including transformers.PretrainedConfig, transformers.configuration_utils.layer_type_validation, sglang.utils.logger. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 transformers.PretrainedConfig, transformers.configuration_utils.layer_type_validation, sglang.utils.logger，为后续代码准备所需名称。

### Lines 5-6: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 7-7: Declares class Qwen3OmniMoeAudioEncoderConfig
```python
class Qwen3OmniMoeAudioEncoderConfig(PretrainedConfig):
```
**EN:** This block introduces class `Qwen3OmniMoeAudioEncoderConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `Qwen3OmniMoeAudioEncoderConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 8-8: Declares model_type
```python
    model_type = "qwen3_omni_moe_audio_encoder"
```
**EN:** This statement initializes model_type in the Qwen3OmniMoeAudioEncoderConfig. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 Qwen3OmniMoeAudioEncoderConfig 中初始化 model_type。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 9-9: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3OmniMoeAudioEncoderConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3OmniMoeAudioEncoderConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 10-52: Defines function Qwen3OmniMoeAudioEncoderConfig.__init__
```python
    def __init__(
        self,
        num_mel_bins=128,
        encoder_layers=32,
        encoder_attention_heads=20,
        encoder_ffn_dim=5120,
        d_model=1280,
        dropout=0,
        attention_dropout=0,
        activation_function="gelu",
        activation_dropout=0,
        scale_embedding=False,
        initializer_range=0.02,
        max_source_positions=1500,
        n_window=100,
        output_dim=3584,
        n_window_infer=400,
        conv_chunksize=500,
        downsample_hidden_size=480,
        **kwargs,
    ):
        super().__init__(**kwargs)

        self.num_mel_bins = num_mel_bins
        self.d_model = d_model
        self.encoder_layers = encoder_layers
        self.encoder_attention_heads = encoder_attention_heads
        self.encoder_ffn_dim = encoder_ffn_dim
        self.dropout = dropout
        self.attention_dropout = attention_dropout
        self.activation_function = activation_function
        self.activation_dropout = activation_dropout
        self.num_hidden_layers = encoder_layers
        self.initializer_range = initializer_range
        self.scale_embedding = (
            scale_embedding  # scale factor will be sqrt(d_model) if True
        )
        self.max_source_positions = max_source_positions
        self.n_window = n_window
        self.output_dim = output_dim
        self.n_window_infer = n_window_infer
        self.conv_chunksize = conv_chunksize
        self.downsample_hidden_size = downsample_hidden_size
```
**EN:** This block defines function `Qwen3OmniMoeAudioEncoderConfig.__init__`. Parameters: self, num_mel_bins, encoder_layers, encoder_attention_heads, encoder_ffn_dim, d_model, dropout, attention_dropout, activation_function, activation_dropout, scale_embedding, initializer_range, max_source_positions, n_window, output_dim, n_window_infer, conv_chunksize, downsample_hidden_size.
**CN:** 该代码块定义函数 `Qwen3OmniMoeAudioEncoderConfig.__init__`。 参数包括 self、num_mel_bins、encoder_layers、encoder_attention_heads、encoder_ffn_dim、d_model、dropout、attention_dropout、activation_function、activation_dropout、scale_embedding、initializer_range、max_source_positions、n_window、output_dim、n_window_infer、conv_chunksize、downsample_hidden_size。

### Lines 53-54: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 55-55: Declares class Qwen3OmniMoeVisionEncoderConfig
```python
class Qwen3OmniMoeVisionEncoderConfig(PretrainedConfig):
```
**EN:** This block introduces class `Qwen3OmniMoeVisionEncoderConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `Qwen3OmniMoeVisionEncoderConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 56-57: Declares model_type, base_config_key
```python
    model_type = "qwen3_omni_moe_vision_encoder"
    base_config_key = "vision_config"
```
**EN:** This block initializes a related set of values in the Qwen3OmniMoeVisionEncoderConfig, including model_type, base_config_key. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Qwen3OmniMoeVisionEncoderConfig 中初始化一组相关值，包括 model_type, base_config_key。将这些赋值集中在一起有助于理解周边配置。

### Lines 58-58: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3OmniMoeVisionEncoderConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3OmniMoeVisionEncoderConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 59-90: Defines function Qwen3OmniMoeVisionEncoderConfig.__init__
```python
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
        deepstack_visual_indexes=[8, 16, 24],
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
        self.deepstack_visual_indexes = deepstack_visual_indexes
```
**EN:** This block defines function `Qwen3OmniMoeVisionEncoderConfig.__init__`. Parameters: self, depth, hidden_size, hidden_act, intermediate_size, num_heads, in_channels, patch_size, spatial_merge_size, temporal_patch_size, out_hidden_size, num_position_embeddings, deepstack_visual_indexes, initializer_range.
**CN:** 该代码块定义函数 `Qwen3OmniMoeVisionEncoderConfig.__init__`。 参数包括 self、depth、hidden_size、hidden_act、intermediate_size、num_heads、in_channels、patch_size、spatial_merge_size、temporal_patch_size、out_hidden_size、num_position_embeddings、deepstack_visual_indexes、initializer_range。

### Lines 91-92: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 93-93: Declares class Qwen3OmniMoeTextConfig
```python
class Qwen3OmniMoeTextConfig(PretrainedConfig):
```
**EN:** This block introduces class `Qwen3OmniMoeTextConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `Qwen3OmniMoeTextConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 94-95: Declares model_type, keys_to_ignore_at_inference
```python
    model_type = "qwen3_omni_moe_text"
    keys_to_ignore_at_inference = ["past_key_values"]
```
**EN:** This block initializes a related set of values in the Qwen3OmniMoeTextConfig, including model_type, keys_to_ignore_at_inference. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Qwen3OmniMoeTextConfig 中初始化一组相关值，包括 model_type, keys_to_ignore_at_inference。将这些赋值集中在一起有助于理解周边配置。

### Lines 96-97: Comments and module notes
```python

    # Default tensor parallel plan for base model `Qwen3OmniMoeText`
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the Qwen3OmniMoeTextConfig.
**CN:** 该范围包含注释或说明，用于记录 Qwen3OmniMoeTextConfig 的假设、来源或实现备注。

### Lines 98-114: Declares base_model_tp_plan, base_model_pp_plan
```python
    base_model_tp_plan = {
        "layers.*.self_attn.q_proj": "colwise",
        "layers.*.self_attn.k_proj": "colwise",
        "layers.*.self_attn.v_proj": "colwise",
        "layers.*.self_attn.o_proj": "rowwise",
        "layers.*.mlp.experts.*.gate_proj": "colwise",
        "layers.*.mlp.experts.*.up_proj": "colwise",
        "layers.*.mlp.experts.*.down_proj": "rowwise",
        "layers.*.mlp.gate_proj": "colwise",
        "layers.*.mlp.up_proj": "colwise",
        "layers.*.mlp.down_proj": "rowwise",
    }
    base_model_pp_plan = {
        "embed_tokens": (["input_ids"], ["inputs_embeds"]),
        "layers": (["hidden_states", "attention_mask"], ["hidden_states"]),
        "norm": (["hidden_states"], ["hidden_states"]),
    }
```
**EN:** This block initializes a related set of values in the Qwen3OmniMoeTextConfig, including base_model_tp_plan, base_model_pp_plan. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Qwen3OmniMoeTextConfig 中初始化一组相关值，包括 base_model_tp_plan, base_model_pp_plan。将这些赋值集中在一起有助于理解周边配置。

### Lines 115-115: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3OmniMoeTextConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3OmniMoeTextConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 116-179: Defines function Qwen3OmniMoeTextConfig.__init__
```python
    def __init__(
        self,
        vocab_size=3584,
        hidden_size=2048,
        intermediate_size=18944,
        num_hidden_layers=28,
        num_attention_heads=28,
        num_key_value_heads=4,
        hidden_act="silu",
        max_position_embeddings=32768,
        initializer_range=0.02,
        rms_norm_eps=1e-6,
        use_cache=True,
        tie_word_embeddings=False,
        rope_theta=1000000.0,
        rope_scaling=None,
        attention_bias=False,
        sliding_window=None,
        attention_dropout=0,
        decoder_sparse_step=1,
        moe_intermediate_size=768,
        num_experts_per_tok=8,
        num_experts=128,
        norm_topk_prob=True,
        output_router_logits=False,
        router_aux_loss_coef=0.001,
        mlp_only_layers=None,
        **kwargs,
    ):
        super().__init__(
            tie_word_embeddings=tie_word_embeddings,
            **kwargs,
        )
        self.vocab_size = vocab_size
        self.max_position_embeddings = max_position_embeddings
        self.hidden_size = hidden_size
        self.intermediate_size = intermediate_size
        self.num_hidden_layers = num_hidden_layers
        self.num_attention_heads = num_attention_heads
        self.sliding_window = sliding_window

        self.num_key_value_heads = num_key_value_heads
        self.hidden_act = hidden_act
        self.initializer_range = initializer_range
        self.rms_norm_eps = rms_norm_eps
        self.use_cache = use_cache
        self.rope_theta = rope_theta
        self.rope_scaling = rope_scaling
        self.attention_bias = attention_bias
        self.attention_dropout = attention_dropout
        # Validate the correctness of rotary position embeddings parameters
        # BC: if there is a 'type' field, move it to 'rope_type'.
        if self.rope_scaling is not None and "type" in self.rope_scaling:
            self.rope_scaling["rope_type"] = self.rope_scaling["type"]

        # MoE arguments
        self.decoder_sparse_step = decoder_sparse_step
        self.moe_intermediate_size = moe_intermediate_size
        self.num_experts_per_tok = num_experts_per_tok
        self.num_experts = num_experts
        self.norm_topk_prob = norm_topk_prob
        self.output_router_logits = output_router_logits
        self.router_aux_loss_coef = router_aux_loss_coef
        self.mlp_only_layers = [] if mlp_only_layers is None else mlp_only_layers
```
**EN:** This block defines function `Qwen3OmniMoeTextConfig.__init__`. Parameters: self, vocab_size, hidden_size, intermediate_size, num_hidden_layers, num_attention_heads, num_key_value_heads, hidden_act, max_position_embeddings, initializer_range, rms_norm_eps, use_cache, tie_word_embeddings, rope_theta, rope_scaling, attention_bias, sliding_window, attention_dropout, decoder_sparse_step, moe_intermediate_size, num_experts_per_tok, num_experts, norm_topk_prob, output_router_logits, router_aux_loss_coef, mlp_only_layers.
**CN:** 该代码块定义函数 `Qwen3OmniMoeTextConfig.__init__`。 参数包括 self、vocab_size、hidden_size、intermediate_size、num_hidden_layers、num_attention_heads、num_key_value_heads、hidden_act、max_position_embeddings、initializer_range、rms_norm_eps、use_cache、tie_word_embeddings、rope_theta、rope_scaling、attention_bias、sliding_window、attention_dropout、decoder_sparse_step、moe_intermediate_size、num_experts_per_tok、num_experts、norm_topk_prob、output_router_logits、router_aux_loss_coef、mlp_only_layers。

### Lines 180-181: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 182-182: Declares class Qwen3OmniMoeThinkerConfig
```python
class Qwen3OmniMoeThinkerConfig(PretrainedConfig):
```
**EN:** This block introduces class `Qwen3OmniMoeThinkerConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `Qwen3OmniMoeThinkerConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 183-193: Declares model_type, attribute_map, sub_configs
```python
    model_type = "qwen3_omni_moe_thinker"
    attribute_map = {
        "image_token_id": "image_token_index",
        "video_token_id": "video_token_index",
        "audio_token_id": "audio_token_index",
    }
    sub_configs = {
        "audio_config": Qwen3OmniMoeAudioEncoderConfig,
        "vision_config": Qwen3OmniMoeVisionEncoderConfig,
        "text_config": Qwen3OmniMoeTextConfig,
    }
```
**EN:** This block initializes a related set of values in the Qwen3OmniMoeThinkerConfig, including model_type, attribute_map, sub_configs. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Qwen3OmniMoeThinkerConfig 中初始化一组相关值，包括 model_type, attribute_map, sub_configs。将这些赋值集中在一起有助于理解周边配置。

### Lines 194-194: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3OmniMoeThinkerConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3OmniMoeThinkerConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 195-234: Defines function Qwen3OmniMoeThinkerConfig.__init__
```python
    def __init__(
        self,
        audio_config=None,
        vision_config=None,
        text_config=None,
        audio_token_id=151646,
        image_token_id=151655,
        video_token_id=151656,
        position_id_per_seconds=25,
        audio_start_token_id=151647,
        user_token_id=872,
        initializer_range=0.02,
        **kwargs,
    ):
        super().__init__(**kwargs)
        self.user_token_id = user_token_id
        self.position_id_per_seconds = position_id_per_seconds
        self.audio_start_token_id = audio_start_token_id
        self.initializer_range = initializer_range

        if isinstance(vision_config, dict):
            vision_config = Qwen3OmniMoeVisionEncoderConfig(**vision_config)
        elif vision_config is None:
            vision_config = Qwen3OmniMoeVisionEncoderConfig()
        self.vision_config = vision_config

        if isinstance(audio_config, dict):
            audio_config = Qwen3OmniMoeAudioEncoderConfig(**audio_config)
        elif audio_config is None:
            audio_config = Qwen3OmniMoeAudioEncoderConfig()
        self.audio_config = audio_config

        if isinstance(text_config, dict):
            text_config = Qwen3OmniMoeTextConfig(**text_config)
        elif text_config is None:
            text_config = Qwen3OmniMoeTextConfig()
        self.text_config = text_config
        self.audio_token_id = audio_token_id
        self.image_token_id = image_token_id
        self.video_token_id = video_token_id
```
**EN:** This block defines function `Qwen3OmniMoeThinkerConfig.__init__`. Parameters: self, audio_config, vision_config, text_config, audio_token_id, image_token_id, video_token_id, position_id_per_seconds, audio_start_token_id, user_token_id, initializer_range.
**CN:** 该代码块定义函数 `Qwen3OmniMoeThinkerConfig.__init__`。 参数包括 self、audio_config、vision_config、text_config、audio_token_id、image_token_id、video_token_id、position_id_per_seconds、audio_start_token_id、user_token_id、initializer_range。

### Lines 235-236: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 237-238: Declares class Qwen3OmniMoeTalkerCodePredictorConfig
```python
class Qwen3OmniMoeTalkerCodePredictorConfig(PretrainedConfig):

```
**EN:** This block introduces class `Qwen3OmniMoeTalkerCodePredictorConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `Qwen3OmniMoeTalkerCodePredictorConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 239-240: Declares model_type, keys_to_ignore_at_inference
```python
    model_type = "qwen3_omni_moe_talker_code_predictor"
    keys_to_ignore_at_inference = ["past_key_values"]
```
**EN:** This block initializes a related set of values in the Qwen3OmniMoeTalkerCodePredictorConfig, including model_type, keys_to_ignore_at_inference. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Qwen3OmniMoeTalkerCodePredictorConfig 中初始化一组相关值，包括 model_type, keys_to_ignore_at_inference。将这些赋值集中在一起有助于理解周边配置。

### Lines 241-242: Comments and module notes
```python

    # Default tensor parallel plan for base model `Qwen3OmniMoeTalkerCodePredictor`
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the Qwen3OmniMoeTalkerCodePredictorConfig.
**CN:** 该范围包含注释或说明，用于记录 Qwen3OmniMoeTalkerCodePredictorConfig 的假设、来源或实现备注。

### Lines 243-256: Declares base_model_tp_plan, base_model_pp_plan
```python
    base_model_tp_plan = {
        "layers.*.self_attn.q_proj": "colwise",
        "layers.*.self_attn.k_proj": "colwise",
        "layers.*.self_attn.v_proj": "colwise",
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
```
**EN:** This block initializes a related set of values in the Qwen3OmniMoeTalkerCodePredictorConfig, including base_model_tp_plan, base_model_pp_plan. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Qwen3OmniMoeTalkerCodePredictorConfig 中初始化一组相关值，包括 base_model_tp_plan, base_model_pp_plan。将这些赋值集中在一起有助于理解周边配置。

### Lines 257-257: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3OmniMoeTalkerCodePredictorConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3OmniMoeTalkerCodePredictorConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 258-324: Defines function Qwen3OmniMoeTalkerCodePredictorConfig.__init__
```python
    def __init__(
        self,
        vocab_size=2048,
        hidden_size=1024,
        intermediate_size=3072,
        num_hidden_layers=5,
        num_attention_heads=16,
        num_key_value_heads=8,
        head_dim=128,
        hidden_act="silu",
        max_position_embeddings=32768,
        initializer_range=0.02,
        rms_norm_eps=0.000001,
        use_cache=True,
        tie_word_embeddings=False,
        rope_theta=10000,
        rope_scaling=None,
        attention_bias=False,
        sliding_window=None,
        layer_types=None,
        attention_dropout=0,
        num_code_groups=32,
        **kwargs,
    ):
        super().__init__(
            tie_word_embeddings=tie_word_embeddings,
            **kwargs,
        )
        self.vocab_size = vocab_size
        self.max_position_embeddings = max_position_embeddings
        self.hidden_size = hidden_size
        self.intermediate_size = intermediate_size
        self.num_hidden_layers = num_hidden_layers
        self.num_attention_heads = num_attention_heads
        self.sliding_window = sliding_window

        # for backward compatibility
        if num_key_value_heads is None:
            num_key_value_heads = num_attention_heads

        self.num_key_value_heads = num_key_value_heads
        self.head_dim = head_dim
        self.hidden_act = hidden_act
        self.initializer_range = initializer_range
        self.rms_norm_eps = rms_norm_eps
        self.use_cache = use_cache
        self.rope_theta = rope_theta
        self.rope_scaling = rope_scaling
        self.attention_bias = attention_bias
        self.attention_dropout = attention_dropout
        # Validate the correctness of rotary position embeddings parameters
        # BC: if there is a 'type' field, move it to 'rope_type'.
        if self.rope_scaling is not None and "type" in self.rope_scaling:
            self.rope_scaling["rope_type"] = self.rope_scaling["type"]

        self.layer_types = layer_types
        if self.layer_types is None:
            self.layer_types = [
                (
                    "sliding_attention"
                    if self.sliding_window is not None and i >= self.max_window_layers
                    else "full_attention"
                )
                for i in range(self.num_hidden_layers)
            ]
        layer_type_validation(self.layer_types, self.num_hidden_layers)
        self.num_code_groups = num_code_groups
```
**EN:** This block defines function `Qwen3OmniMoeTalkerCodePredictorConfig.__init__`. Parameters: self, vocab_size, hidden_size, intermediate_size, num_hidden_layers, num_attention_heads, num_key_value_heads, head_dim, hidden_act, max_position_embeddings, initializer_range, rms_norm_eps, use_cache, tie_word_embeddings, rope_theta, rope_scaling, attention_bias, sliding_window, layer_types, attention_dropout, num_code_groups.
**CN:** 该代码块定义函数 `Qwen3OmniMoeTalkerCodePredictorConfig.__init__`。 参数包括 self、vocab_size、hidden_size、intermediate_size、num_hidden_layers、num_attention_heads、num_key_value_heads、head_dim、hidden_act、max_position_embeddings、initializer_range、rms_norm_eps、use_cache、tie_word_embeddings、rope_theta、rope_scaling、attention_bias、sliding_window、layer_types、attention_dropout、num_code_groups。

### Lines 325-326: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 327-328: Declares class Qwen3OmniMoeTalkerTextConfig
```python
class Qwen3OmniMoeTalkerTextConfig(PretrainedConfig):

```
**EN:** This block introduces class `Qwen3OmniMoeTalkerTextConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `Qwen3OmniMoeTalkerTextConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 329-330: Declares model_type, keys_to_ignore_at_inference
```python
    model_type = "qwen3_omni_moe_talker_text"
    keys_to_ignore_at_inference = ["past_key_values"]
```
**EN:** This block initializes a related set of values in the Qwen3OmniMoeTalkerTextConfig, including model_type, keys_to_ignore_at_inference. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Qwen3OmniMoeTalkerTextConfig 中初始化一组相关值，包括 model_type, keys_to_ignore_at_inference。将这些赋值集中在一起有助于理解周边配置。

### Lines 331-332: Comments and module notes
```python

    # Default tensor parallel plan for base model `Qwen3OmniMoeTalkerText`
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the Qwen3OmniMoeTalkerTextConfig.
**CN:** 该范围包含注释或说明，用于记录 Qwen3OmniMoeTalkerTextConfig 的假设、来源或实现备注。

### Lines 333-349: Declares base_model_tp_plan, base_model_pp_plan
```python
    base_model_tp_plan = {
        "layers.*.self_attn.q_proj": "colwise",
        "layers.*.self_attn.k_proj": "colwise",
        "layers.*.self_attn.v_proj": "colwise",
        "layers.*.self_attn.o_proj": "rowwise",
        "layers.*.mlp.experts.*.gate_proj": "colwise",
        "layers.*.mlp.experts.*.up_proj": "colwise",
        "layers.*.mlp.experts.*.down_proj": "rowwise",
        "layers.*.mlp.gate_proj": "colwise",
        "layers.*.mlp.up_proj": "colwise",
        "layers.*.mlp.down_proj": "rowwise",
    }
    base_model_pp_plan = {
        "embed_tokens": (["input_ids"], ["inputs_embeds"]),
        "layers": (["hidden_states", "attention_mask"], ["hidden_states"]),
        "norm": (["hidden_states"], ["hidden_states"]),
    }
```
**EN:** This block initializes a related set of values in the Qwen3OmniMoeTalkerTextConfig, including base_model_tp_plan, base_model_pp_plan. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Qwen3OmniMoeTalkerTextConfig 中初始化一组相关值，包括 base_model_tp_plan, base_model_pp_plan。将这些赋值集中在一起有助于理解周边配置。

### Lines 350-350: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3OmniMoeTalkerTextConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3OmniMoeTalkerTextConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 351-414: Defines function Qwen3OmniMoeTalkerTextConfig.__init__
```python
    def __init__(
        self,
        vocab_size=3072,
        hidden_size=1024,
        intermediate_size=2048,
        num_hidden_layers=20,
        num_attention_heads=16,
        num_key_value_heads=2,
        hidden_act="silu",
        max_position_embeddings=32768,
        initializer_range=0.02,
        rms_norm_eps=0.000001,
        use_cache=True,
        tie_word_embeddings=False,
        rope_theta=10000,
        rope_scaling=None,
        attention_bias=False,
        sliding_window=None,
        attention_dropout=0,
        decoder_sparse_step=1,
        moe_intermediate_size=384,
        num_experts_per_tok=8,
        num_experts=128,
        norm_topk_prob=False,
        output_router_logits=False,
        router_aux_loss_coef=0.001,
        mlp_only_layers=None,
        **kwargs,
    ):
        super().__init__(
            tie_word_embeddings=tie_word_embeddings,
            **kwargs,
        )
        self.vocab_size = vocab_size
        self.max_position_embeddings = max_position_embeddings
        self.hidden_size = hidden_size
        self.intermediate_size = intermediate_size
        self.num_hidden_layers = num_hidden_layers
        self.num_attention_heads = num_attention_heads
        self.sliding_window = sliding_window

        self.num_key_value_heads = num_key_value_heads
        self.hidden_act = hidden_act
        self.initializer_range = initializer_range
        self.rms_norm_eps = rms_norm_eps
        self.use_cache = use_cache
        self.rope_theta = rope_theta
        self.rope_scaling = rope_scaling
        self.attention_bias = attention_bias
        self.attention_dropout = attention_dropout
        # Validate the correctness of rotary position embeddings parameters
        # BC: if there is a 'type' field, move it to 'rope_type'.
        if self.rope_scaling is not None and "type" in self.rope_scaling:
            self.rope_scaling["rope_type"] = self.rope_scaling["type"]

        # MoE arguments
        self.decoder_sparse_step = decoder_sparse_step
        self.moe_intermediate_size = moe_intermediate_size
        self.num_experts_per_tok = num_experts_per_tok
        self.num_experts = num_experts
        self.norm_topk_prob = norm_topk_prob
        self.output_router_logits = output_router_logits
        self.router_aux_loss_coef = router_aux_loss_coef
        self.mlp_only_layers = [] if mlp_only_layers is None else mlp_only_layers
```
**EN:** This block defines function `Qwen3OmniMoeTalkerTextConfig.__init__`. Parameters: self, vocab_size, hidden_size, intermediate_size, num_hidden_layers, num_attention_heads, num_key_value_heads, hidden_act, max_position_embeddings, initializer_range, rms_norm_eps, use_cache, tie_word_embeddings, rope_theta, rope_scaling, attention_bias, sliding_window, attention_dropout, decoder_sparse_step, moe_intermediate_size, num_experts_per_tok, num_experts, norm_topk_prob, output_router_logits, router_aux_loss_coef, mlp_only_layers.
**CN:** 该代码块定义函数 `Qwen3OmniMoeTalkerTextConfig.__init__`。 参数包括 self、vocab_size、hidden_size、intermediate_size、num_hidden_layers、num_attention_heads、num_key_value_heads、hidden_act、max_position_embeddings、initializer_range、rms_norm_eps、use_cache、tie_word_embeddings、rope_theta、rope_scaling、attention_bias、sliding_window、attention_dropout、decoder_sparse_step、moe_intermediate_size、num_experts_per_tok、num_experts、norm_topk_prob、output_router_logits、router_aux_loss_coef、mlp_only_layers。

### Lines 415-416: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 417-418: Declares class Qwen3OmniMoeTalkerConfig
```python
class Qwen3OmniMoeTalkerConfig(PretrainedConfig):

```
**EN:** This block introduces class `Qwen3OmniMoeTalkerConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `Qwen3OmniMoeTalkerConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 419-422: Declares sub_configs
```python
    sub_configs = {
        "code_predictor_config": Qwen3OmniMoeTalkerCodePredictorConfig,
        "text_config": Qwen3OmniMoeTalkerTextConfig,
    }
```
**EN:** This statement initializes sub_configs in the Qwen3OmniMoeTalkerConfig. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 Qwen3OmniMoeTalkerConfig 中初始化 sub_configs。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 423-423: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3OmniMoeTalkerConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3OmniMoeTalkerConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 424-485: Defines function Qwen3OmniMoeTalkerConfig.__init__
```python
    def __init__(
        self,
        code_predictor_config=None,
        text_config=None,
        num_code_groups=32,
        thinker_hidden_size=2048,
        codec_eos_token_id=4198,
        accept_hidden_layer=18,
        codec_nothink_id=4203,
        codec_think_bos_id=4204,
        codec_think_eos_id=4205,
        codec_pad_id=4196,
        codec_bos_id=4197,
        audio_token_id=151646,
        image_token_id=151655,
        video_token_id=151656,
        vision_start_token_id=151652,
        position_id_per_seconds=25,
        audio_start_token_id=151669,
        speaker_id=None,
        **kwargs,
    ):
        super().__init__(**kwargs)
        if code_predictor_config is None:
            code_predictor_config = {}
            self.code_predictor_config = Qwen3OmniMoeTalkerCodePredictorConfig()
            logger.info(
                "code_predictor_config is None. Initializing code_predictor_config model with default values"
            )
        elif isinstance(code_predictor_config, Qwen3OmniMoeTalkerCodePredictorConfig):
            self.code_predictor_config = code_predictor_config
        else:
            self.code_predictor_config = Qwen3OmniMoeTalkerCodePredictorConfig(
                **code_predictor_config
            )

        if text_config is None:
            text_config = {}
            self.text_config = Qwen3OmniMoeTalkerTextConfig()
            logger.info(
                "talker text_config is None. Initializing talker text model with default values"
            )
        elif isinstance(text_config, Qwen3OmniMoeTalkerTextConfig):
            self.text_config = text_config
        else:
            self.text_config = Qwen3OmniMoeTalkerTextConfig(**text_config)
        self.num_code_groups = num_code_groups
        self.thinker_hidden_size = thinker_hidden_size
        self.codec_eos_token_id = codec_eos_token_id
        self.accept_hidden_layer = accept_hidden_layer
        self.codec_nothink_id = codec_nothink_id
        self.codec_think_bos_id = codec_think_bos_id
        self.codec_think_eos_id = codec_think_eos_id
        self.codec_pad_id = codec_pad_id
        self.codec_bos_id = codec_bos_id
        self.audio_token_id = audio_token_id
        self.image_token_id = image_token_id
        self.video_token_id = video_token_id
        self.position_id_per_seconds = position_id_per_seconds
        self.audio_start_token_id = audio_start_token_id
        self.vision_start_token_id = vision_start_token_id
        self.speaker_id = speaker_id
```
**EN:** This block defines function `Qwen3OmniMoeTalkerConfig.__init__`. Parameters: self, code_predictor_config, text_config, num_code_groups, thinker_hidden_size, codec_eos_token_id, accept_hidden_layer, codec_nothink_id, codec_think_bos_id, codec_think_eos_id, codec_pad_id, codec_bos_id, audio_token_id, image_token_id, video_token_id, vision_start_token_id, position_id_per_seconds, audio_start_token_id, speaker_id.
**CN:** 该代码块定义函数 `Qwen3OmniMoeTalkerConfig.__init__`。 参数包括 self、code_predictor_config、text_config、num_code_groups、thinker_hidden_size、codec_eos_token_id、accept_hidden_layer、codec_nothink_id、codec_think_bos_id、codec_think_eos_id、codec_pad_id、codec_bos_id、audio_token_id、image_token_id、video_token_id、vision_start_token_id、position_id_per_seconds、audio_start_token_id、speaker_id。

### Lines 486-487: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 488-489: Declares class Qwen3OmniMoeCode2WavConfig
```python
class Qwen3OmniMoeCode2WavConfig(PretrainedConfig):

```
**EN:** This block introduces class `Qwen3OmniMoeCode2WavConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `Qwen3OmniMoeCode2WavConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 490-530: Defines function Qwen3OmniMoeCode2WavConfig.__init__
```python
    def __init__(
        self,
        codebook_size=2048,
        hidden_size=1024,
        max_position_embeddings=8000,
        rope_theta=10000,
        num_attention_heads=16,
        num_key_value_heads=16,
        attention_bias=False,
        sliding_window=72,
        intermediate_size=3072,
        hidden_act="silu",
        layer_scale_initial_scale=0.01,
        rms_norm_eps=1e-5,
        num_hidden_layers=8,
        num_quantizers=16,
        upsample_rates=(8, 5, 4, 3),
        upsampling_ratios=(2, 2),
        decoder_dim=1536,
        attention_dropout=0.0,
        **kwargs,
    ):
        super().__init__(**kwargs)
        self.codebook_size = codebook_size
        self.hidden_size = hidden_size
        self.max_position_embeddings = max_position_embeddings
        self.rope_theta = rope_theta
        self.num_attention_heads = num_attention_heads
        self.num_key_value_heads = num_key_value_heads
        self.attention_bias = attention_bias
        self.sliding_window = sliding_window
        self.intermediate_size = intermediate_size
        self.hidden_act = hidden_act
        self.layer_scale_initial_scale = layer_scale_initial_scale
        self.rms_norm_eps = rms_norm_eps
        self.num_hidden_layers = num_hidden_layers
        self.num_quantizers = num_quantizers
        self.upsample_rates = upsample_rates
        self.upsampling_ratios = upsampling_ratios
        self.decoder_dim = decoder_dim
        self.attention_dropout = attention_dropout
```
**EN:** This block defines function `Qwen3OmniMoeCode2WavConfig.__init__`. Parameters: self, codebook_size, hidden_size, max_position_embeddings, rope_theta, num_attention_heads, num_key_value_heads, attention_bias, sliding_window, intermediate_size, hidden_act, layer_scale_initial_scale, rms_norm_eps, num_hidden_layers, num_quantizers, upsample_rates, upsampling_ratios, decoder_dim, attention_dropout.
**CN:** 该代码块定义函数 `Qwen3OmniMoeCode2WavConfig.__init__`。 参数包括 self、codebook_size、hidden_size、max_position_embeddings、rope_theta、num_attention_heads、num_key_value_heads、attention_bias、sliding_window、intermediate_size、hidden_act、layer_scale_initial_scale、rms_norm_eps、num_hidden_layers、num_quantizers、upsample_rates、upsampling_ratios、decoder_dim、attention_dropout。

### Lines 531-531: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3OmniMoeCode2WavConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3OmniMoeCode2WavConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 532-537: Defines function Qwen3OmniMoeCode2WavConfig.layer_types
```python
    @property
    def layer_types(self):
        """
        All layer in code2wav should be sliding attention
        """
        return ["sliding_attention"] * self.num_hidden_layers
```
**EN:** This block defines function `Qwen3OmniMoeCode2WavConfig.layer_types`. Parameters: self. Decorators: property. All layer in code2wav should be sliding attention
**CN:** 该代码块定义函数 `Qwen3OmniMoeCode2WavConfig.layer_types`。 参数包括 self。 装饰器包括 property。 文档字符串摘要：All layer in code2wav should be sliding attention

### Lines 538-539: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 540-541: Declares class Qwen3OmniMoeConfig
```python
class Qwen3OmniMoeConfig(PretrainedConfig):

```
**EN:** This block introduces class `Qwen3OmniMoeConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig.
**CN:** 该代码块声明类 `Qwen3OmniMoeConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。

### Lines 542-547: Declares model_type, sub_configs
```python
    model_type = "qwen3_omni_moe"
    sub_configs = {
        "thinker_config": Qwen3OmniMoeThinkerConfig,
        "talker_config": Qwen3OmniMoeTalkerConfig,
        "code2wav_config": Qwen3OmniMoeCode2WavConfig,
    }
```
**EN:** This block initializes a related set of values in the Qwen3OmniMoeConfig, including model_type, sub_configs. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 Qwen3OmniMoeConfig 中初始化一组相关值，包括 model_type, sub_configs。将这些赋值集中在一起有助于理解周边配置。

### Lines 548-548: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3OmniMoeConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3OmniMoeConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 549-595: Defines function Qwen3OmniMoeConfig.__init__
```python
    def __init__(
        self,
        thinker_config=None,
        talker_config=None,
        code2wav_config=None,
        enable_audio_output=True,
        im_start_token_id=151644,
        im_end_token_id=151645,
        tts_pad_token_id=151671,
        tts_bos_token_id=151672,
        tts_eos_token_id=151673,
        system_token_id=8948,
        user_token_id=872,
        assistant_token_id=77091,
        **kwargs,
    ):
        super().__init__(**kwargs)
        if thinker_config is None:
            thinker_config = {}
            logger.info(
                "thinker_config is None. Initializing thinker model with default values"
            )

        if talker_config is None:
            talker_config = {}
            logger.info(
                "talker_config is None. Initializing talker model with default values"
            )

        if code2wav_config is None:
            code2wav_config = {}
            logger.info(
                "code2wav_config is None. Initializing code2wav model with default values"
            )

        self.thinker_config = Qwen3OmniMoeThinkerConfig(**thinker_config)
        self.talker_config = Qwen3OmniMoeTalkerConfig(**talker_config)
        self.code2wav_config = Qwen3OmniMoeCode2WavConfig(**code2wav_config)
        self.enable_audio_output = enable_audio_output
        self.im_start_token_id = im_start_token_id
        self.im_end_token_id = im_end_token_id
        self.tts_pad_token_id = tts_pad_token_id
        self.tts_bos_token_id = tts_bos_token_id
        self.tts_eos_token_id = tts_eos_token_id
        self.system_token_id = system_token_id
        self.user_token_id = user_token_id
        self.assistant_token_id = assistant_token_id
```
**EN:** This block defines function `Qwen3OmniMoeConfig.__init__`. Parameters: self, thinker_config, talker_config, code2wav_config, enable_audio_output, im_start_token_id, im_end_token_id, tts_pad_token_id, tts_bos_token_id, tts_eos_token_id, system_token_id, user_token_id, assistant_token_id.
**CN:** 该代码块定义函数 `Qwen3OmniMoeConfig.__init__`。 参数包括 self、thinker_config、talker_config、code2wav_config、enable_audio_output、im_start_token_id、im_end_token_id、tts_pad_token_id、tts_bos_token_id、tts_eos_token_id、system_token_id、user_token_id、assistant_token_id。

### Lines 596-596: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the Qwen3OmniMoeConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 Qwen3OmniMoeConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 597-609: Defines function Qwen3OmniMoeConfig.get_text_config
```python
    def get_text_config(self, decoder=False) -> "PretrainedConfig":
        """
        Returns the config that is meant to be used with text IO. On most models, it is the original config instance
        itself. On specific composite models, it is under a set of valid names.

        Args:
            decoder (`Optional[bool]`, *optional*, defaults to `False`):
                If set to `True`, then only search for decoder config names.
        """
        # Overridden for deeply nested config like Qwen2-Omni. We don't have any omni model
        # except for Qwen yet. This has to be generalized if more deeply nested configs are
        # added. NOTE: currently method used only by vLLM
        return self.thinker_config.get_text_config()
```
**EN:** This block defines function `Qwen3OmniMoeConfig.get_text_config`. Parameters: self, decoder. Returns the config that is meant to be used with text IO. On most models, it is the original config instance itself.
**CN:** 该代码块定义函数 `Qwen3OmniMoeConfig.get_text_config`。 参数包括 self、decoder。 文档字符串摘要：Returns the config that is meant to be used with text IO. On most models, it is the original config instance itself.

## Key Concepts / 关键概念
- **Classes / 类**: `Qwen3OmniMoeAudioEncoderConfig`, `Qwen3OmniMoeVisionEncoderConfig`, `Qwen3OmniMoeTextConfig`, `Qwen3OmniMoeThinkerConfig`, `Qwen3OmniMoeTalkerCodePredictorConfig`, `Qwen3OmniMoeTalkerTextConfig`, `Qwen3OmniMoeTalkerConfig`, `Qwen3OmniMoeCode2WavConfig`, `Qwen3OmniMoeConfig`

## Dependencies / 依赖关系
- **Third-Party / 第三方**: `transformers`, `transformers.configuration_utils`
- **Local Modules / 本地模块**: `sglang.utils`
