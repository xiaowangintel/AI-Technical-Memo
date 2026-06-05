# kimi_k25.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/configs/kimi_k25.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines model-specific configuration adapters for kimi k25 so SGLang can interpret architecture metadata correctly at runtime. / 该模块为 kimi k25 定义模型专用配置适配逻辑，使 SGLang 能在运行时正确解析该架构的元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-3: Documents the scope
```python
"""
Kimi K25 Model Configuration.
"""
```
**EN:** This string literal serves as documentation for the module, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 模块 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 4-4: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 5-6: Imports dependencies
```python
from transformers import DeepseekV3Config
from transformers.configuration_utils import PretrainedConfig
```
**EN:** This block groups related imports for the module, including transformers.DeepseekV3Config, transformers.configuration_utils.PretrainedConfig. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 transformers.DeepseekV3Config, transformers.configuration_utils.PretrainedConfig，为后续代码准备所需名称。

### Lines 7-8: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 9-9: Declares class KimiK25VisionConfig
```python
class KimiK25VisionConfig(PretrainedConfig):
```
**EN:** This block introduces class `KimiK25VisionConfig` as a reusable abstraction inside the module. It inherits from PretrainedConfig. Vision configuration for K2-VL (vision tower + mm projector). Args: Vision Tower Parameters: patch_size: Patch size for vision tower.
**CN:** 该代码块声明类 `KimiK25VisionConfig`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。 文档字符串摘要：Vision configuration for K2-VL (vision tower + mm projector). Args: Vision Tower Parameters: patch_size: Patch size for vision tower.

### Lines 10-32: Documents the scope
```python
    """Vision configuration for K2-VL (vision tower + mm projector).

    Args:
        Vision Tower Parameters:
            patch_size: Patch size for vision tower.
            init_pos_emb_height: Initial position embedding height.
            init_pos_emb_width: Initial position embedding width.
            init_pos_emb_time: Initial position embedding time dimension.
            pos_emb_type: Type of position embedding.
            num_attention_heads: Number of attention heads in vision tower.
            num_hidden_layers: Number of hidden layers in vision tower.
            hidden_size: Hidden size of vision tower.
            intermediate_size: Intermediate size in vision tower FFN.
            merge_kernel_size: Kernel size for spatial patch merging.
            video_attn_type: Type of video attention.
            merge_type: Type of merge operation.

        MM Projector Parameters:
            mm_projector_type: Type of multimodal projector.
            mm_hidden_size: Hidden size for projector (defaults to hidden_size).
            projector_hidden_act: Activation function for projector.
            projector_ln_eps: Layer norm epsilon for projector.
    """
```
**EN:** This string literal serves as documentation for the KimiK25VisionConfig, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 KimiK25VisionConfig 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 33-33: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiK25VisionConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiK25VisionConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 34-34: Declares model_type
```python
    model_type = "kimi_k25"
```
**EN:** This statement initializes model_type in the KimiK25VisionConfig. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 KimiK25VisionConfig 中初始化 model_type。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 35-35: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiK25VisionConfig, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiK25VisionConfig 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 36-81: Defines function KimiK25VisionConfig.__init__
```python
    def __init__(
        self,
        # Vision Tower
        patch_size: int = 14,
        init_pos_emb_height: int = 64,
        init_pos_emb_width: int = 64,
        init_pos_emb_time: int = 4,
        pos_emb_type: str = "divided_fixed",
        num_attention_heads: int = 16,
        num_hidden_layers: int = 27,
        hidden_size: int = 1152,
        intermediate_size: int = 4304,
        merge_kernel_size: tuple[int, int] = (2, 2),
        video_attn_type: str = "spatial_temporal",
        merge_type: str = "sd2_tpool",
        # MM Projector
        mm_projector_type: str = "patchmerger",
        mm_hidden_size: int | None = None,
        projector_hidden_act: str = "gelu",
        projector_ln_eps: float = 1e-5,
        text_hidden_size: int = 7168,
        **kwargs,
    ):
        super().__init__(**kwargs)
        # Vision Tower
        self.patch_size = patch_size
        self.init_pos_emb_height = init_pos_emb_height
        self.init_pos_emb_width = init_pos_emb_width
        self.init_pos_emb_time = init_pos_emb_time
        self.pos_emb_type = pos_emb_type
        self.num_attention_heads = num_attention_heads
        self.num_hidden_layers = num_hidden_layers
        self.hidden_size = hidden_size
        self.intermediate_size = intermediate_size
        self.merge_kernel_size = merge_kernel_size
        self.video_attn_type = video_attn_type
        self.merge_type = merge_type
        # MM Projector
        self.mm_projector_type = mm_projector_type
        if mm_hidden_size is not None:
            self.mm_hidden_size = mm_hidden_size
        else:
            self.mm_hidden_size = hidden_size
        self.projector_hidden_act = projector_hidden_act
        self.projector_ln_eps = projector_ln_eps
        self.text_hidden_size = text_hidden_size
```
**EN:** This block defines function `KimiK25VisionConfig.__init__`. Parameters: self, patch_size, init_pos_emb_height, init_pos_emb_width, init_pos_emb_time, pos_emb_type, num_attention_heads, num_hidden_layers, hidden_size, intermediate_size, merge_kernel_size, video_attn_type, merge_type, mm_projector_type, mm_hidden_size, projector_hidden_act, projector_ln_eps, text_hidden_size.
**CN:** 该代码块定义函数 `KimiK25VisionConfig.__init__`。 参数包括 self、patch_size、init_pos_emb_height、init_pos_emb_width、init_pos_emb_time、pos_emb_type、num_attention_heads、num_hidden_layers、hidden_size、intermediate_size、merge_kernel_size、video_attn_type、merge_type、mm_projector_type、mm_hidden_size、projector_hidden_act、projector_ln_eps、text_hidden_size。

### Lines 82-83: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 84-84: Declares class KimiK25Config
```python
class KimiK25Config(PretrainedConfig):
```
**EN:** This block introduces class `KimiK25Config` as a reusable abstraction inside the module. It inherits from PretrainedConfig. K2-VL model configuration. K2-VL extends Kimi-VL with video support using video-chunks.
**CN:** 该代码块声明类 `KimiK25Config`，作为模块中的可复用抽象。 它继承自 PretrainedConfig。 文档字符串摘要：K2-VL model configuration. K2-VL extends Kimi-VL with video support using video-chunks.

### Lines 85-124: Documents the scope
```python
    """K2-VL model configuration.

    K2-VL extends Kimi-VL with video support using video-chunks.
    A video-chunk consists of multiple consecutive frames (default: 4)
    that are processed together with temporal pooling.

    Args:
        text_config: Configuration for the text model (DeepseekV3).

        Vision Tower Parameters:
            patch_size: Patch size for vision tower.
            init_pos_emb_height: Initial position embedding height.
            init_pos_emb_width: Initial position embedding width.
            init_pos_emb_time: Initial position embedding time dimension.
            pos_emb_type: Type of position embedding.
            vt_num_attention_heads: Number of attention heads in vision tower.
            vt_num_hidden_layers: Number of hidden layers in vision tower.
            vt_hidden_size: Hidden size of vision tower.
            vt_intermediate_size: Intermediate size in vision tower FFN.
            merge_kernel_size: Kernel size for spatial patch merging.
            video_attn_type: Type of video attention.
            merge_type: Type of merge operation.

        Video-Chunk Parameters:
            temporal_merge_kernel_size: Number of frames per video chunk.
                Default is 4, meaning 4 frames are merged into 1 chunk.
            sample_fps: Video sampling frame rate.
            timestamp_mode: Format for chunk timestamps.

        MM Projector Parameters:
            mm_projector_type: Type of multimodal projector.
            mm_hidden_size: Hidden size from vision tower.
            projector_hidden_act: Activation function for projector.
            projector_ln_eps: Layer norm epsilon for projector.

        Other Parameters:
            ignore_index: The ignore index for the loss function.
            media_placeholder_token_id: The token ID for media placeholders.
            pad_token_id: The token ID for padding.
    """
```
**EN:** This string literal serves as documentation for the KimiK25Config, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 KimiK25Config 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 125-125: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiK25Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiK25Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 126-126: Declares model_type
```python
    model_type = "kimi_k25"
```
**EN:** This statement initializes model_type in the KimiK25Config. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 KimiK25Config 中初始化 model_type。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 127-127: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiK25Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiK25Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 128-161: Defines function KimiK25Config.__init__
```python
    def __init__(
        self,
        text_config: dict | DeepseekV3Config | None = None,
        vision_config: dict | KimiK25VisionConfig | None = None,
        # Other parameters
        ignore_index: int = -100,
        media_placeholder_token_id: int = 163605,
        pad_token_id: int = 0,
        use_unified_vision_chunk: bool = False,
        video_placeholder: str = "<|kimi_k25_video_placeholder|>",
        **kwargs,
    ):
        if text_config is None:
            text_config = DeepseekV3Config()
        elif isinstance(text_config, dict):
            text_config = DeepseekV3Config(**text_config)

        if vision_config is None:
            vision_config = KimiK25VisionConfig()
        elif isinstance(vision_config, dict):
            vision_config = KimiK25VisionConfig(**vision_config)
        self.vision_config = vision_config
        self.text_config = text_config
        # Other config
        self.ignore_index = ignore_index
        self.media_placeholder_token_id = media_placeholder_token_id
        self.use_unified_vision_chunk = use_unified_vision_chunk
        self.video_placeholder = video_placeholder

        # Propagate quantization config from text model
        if getattr(self.text_config, "quantization_config", None) is not None:
            self.quantization_config = self.text_config.quantization_config

        super().__init__(pad_token_id=pad_token_id, **kwargs)
```
**EN:** This block defines function `KimiK25Config.__init__`. Parameters: self, text_config, vision_config, ignore_index, media_placeholder_token_id, pad_token_id, use_unified_vision_chunk, video_placeholder.
**CN:** 该代码块定义函数 `KimiK25Config.__init__`。 参数包括 self、text_config、vision_config、ignore_index、media_placeholder_token_id、pad_token_id、use_unified_vision_chunk、video_placeholder。

### Lines 162-162: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiK25Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiK25Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 163-166: Defines function KimiK25Config.hidden_size
```python
    @property
    def hidden_size(self) -> int:
        """Get hidden size from text config for compatibility."""
        return self.text_config.hidden_size
```
**EN:** This block defines function `KimiK25Config.hidden_size`. Parameters: self. Decorators: property. Get hidden size from text config for compatibility.
**CN:** 该代码块定义函数 `KimiK25Config.hidden_size`。 参数包括 self。 装饰器包括 property。 文档字符串摘要：Get hidden size from text config for compatibility.

### Lines 167-167: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the KimiK25Config, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 KimiK25Config 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 168-171: Defines function KimiK25Config.vocab_size
```python
    @property
    def vocab_size(self) -> int:
        """Get vocab size from text config for compatibility."""
        return self.text_config.vocab_size
```
**EN:** This block defines function `KimiK25Config.vocab_size`. Parameters: self. Decorators: property. Get vocab size from text config for compatibility.
**CN:** 该代码块定义函数 `KimiK25Config.vocab_size`。 参数包括 self。 装饰器包括 property。 文档字符串摘要：Get vocab size from text config for compatibility.

## Key Concepts / 关键概念
- **Classes / 类**: `KimiK25VisionConfig`, `KimiK25Config`

## Dependencies / 依赖关系
- **Third-Party / 第三方**: `transformers`, `transformers.configuration_utils`
