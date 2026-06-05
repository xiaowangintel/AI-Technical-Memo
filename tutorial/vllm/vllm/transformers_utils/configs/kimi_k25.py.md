# kimi_k25.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/transformers_utils/configs/kimi_k25.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Kimi-K2.5 Model Configuration. / [CN] 定义 Kimi K25 的配置类或兼容辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 3-8: Module overview
```python
"""
Kimi-K2.5 Model Configuration.

This configuration supports video-chunk as an internal modality type.
A video-chunk is the smallest independently processable unit of video.
"""
```
**EN:** The module docstring frames the file's responsibility and intended usage. Its opening line is: Kimi-K2.5 Model Configuration.
**CN:** 模块文档字符串概述了该文件的职责与预期使用方式。

### Lines 10-11: Imports
```python
from transformers import DeepseekV3Config
from transformers.configuration_utils import PretrainedConfig
```
**EN:** This import block assembles the module's dependencies: standard-library helpers such as none, external APIs such as `transformers`, `transformers.configuration_utils`, and internal vLLM modules such as none. Together they define the services the rest of the file builds on.
**CN:** 该导入块汇集了模块依赖：标准库工具如 无，外部 API 如 `transformers`, `transformers.configuration_utils`，以及 vLLM 内部模块如 无。这些依赖共同构成了后续实现所依赖的基础能力。

### Lines 14-60: Class `KimiK25VisionConfig`
```python
class KimiK25VisionConfig(PretrainedConfig):
    model_type = "kimi_k25_vision"

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
```
**EN:** Defines `KimiK25VisionConfig`, a model-configuration class derived from `PretrainedConfig`. Key methods include `__init__`.
**CN:** 定义 `KimiK25VisionConfig`，这是一个模型配置类，继承自 `PretrainedConfig`。 关键方法包括 `__init__`。

### Lines 63-131: Class `KimiK25Config`
```python
class KimiK25Config(PretrainedConfig):
    """Kimi-K2.5 model configuration.

    Kimi-K2.5 extends Kimi-K2 with vision support using video-chunks.
    A video-chunk consists of multiple consecutive frames
    that are processed together with temporal pooling.

    Args:
        vision_config: Configuration for the vision tower and projector.
        text_config: Configuration for the text model (DeepseekV3).
        ignore_index: The ignore index for the loss function.
        media_placeholder_token_id: The token ID for media placeholders.
        pad_token_id: The token ID for padding.
    """

    model_type = "kimi_k25"

    def __init__(
        self,
        vision_config: dict | KimiK25VisionConfig | None = None,
        text_config: dict | DeepseekV3Config | None = None,
        ignore_index: int = -100,
        media_placeholder_token_id: int = 163605,
        pad_token_id: int = 0,
        use_unified_vision_chunk: bool = False,
        video_placeholder: str = "<|kimi_k25_video_placeholder|>",
        **kwargs,
    ):
        # Vision config
        if vision_config is None:
            self.vision_config = KimiK25VisionConfig()
        elif isinstance(vision_config, dict):
            self.vision_config = KimiK25VisionConfig(**vision_config)
        else:
            self.vision_config = vision_config

        # Text config
        if text_config is None:
            self.text_config = DeepseekV3Config()
        elif isinstance(text_config, dict):
# ... omitted for brevity ...

        super().__init__(pad_token_id=pad_token_id, **kwargs)

    @property
    def hidden_size(self) -> int:
        """Get hidden size from text config for compatibility."""
        return self.text_config.hidden_size

    @property
    def vocab_size(self) -> int:
        """Get vocab size from text config for compatibility."""
        return self.text_config.vocab_size
```
**EN:** Defines `KimiK25Config`, a model-configuration class derived from `PretrainedConfig`. The class docstring highlights that Kimi-K2.5 model configuration. Key methods include `__init__`, `hidden_size`, `vocab_size`.
**CN:** 定义 `KimiK25Config`，这是一个模型配置类，继承自 `PretrainedConfig`。 类文档字符串进一步说明了它的职责。 关键方法包括 `__init__`, `hidden_size`, `vocab_size`。

## Key Concepts / 关键概念
- **EN:** Configuration classes extend upstream Hugging Face metadata with vLLM-specific defaults, validation, or compatibility patches.
- **CN:** 配置类会在上游 Hugging Face 元数据之上加入 vLLM 专用默认值、校验逻辑或兼容补丁。
- **EN:** The module is tightly coupled to Hugging Face Transformers APIs, so version compatibility is often an explicit concern.
- **CN:** 该模块与 Hugging Face Transformers API 紧密耦合，因此版本兼容性通常是显式关注点。

## Dependencies / 依赖关系
- **EN:** Standard library modules: none.
- **CN:** 标准库模块：无。
- **EN:** External packages: `transformers`, `transformers.configuration_utils`.
- **CN:** 外部依赖包：`transformers`, `transformers.configuration_utils`。
- **EN:** Internal modules: none.
- **CN:** 内部模块：无。
