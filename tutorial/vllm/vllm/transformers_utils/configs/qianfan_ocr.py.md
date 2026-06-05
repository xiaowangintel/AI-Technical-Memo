# qianfan_ocr.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/transformers_utils/configs/qianfan_ocr.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Defines model-specific configuration classes or compatibility helpers for Qianfan Ocr. / [CN] 为 Qianfan Ocr 定义模型专用配置类或兼容性辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 4-7: Imports
```python
from typing import Any

from transformers import PretrainedConfig
from transformers.models.auto import CONFIG_MAPPING
```
**EN:** This import block assembles the module's dependencies: standard-library helpers such as `typing`, external APIs such as `transformers`, `transformers.models.auto`, and internal vLLM modules such as none. Together they define the services the rest of the file builds on.
**CN:** 该导入块汇集了模块依赖：标准库工具如 `typing`，外部 API 如 `transformers`, `transformers.models.auto`，以及 vLLM 内部模块如 无。这些依赖共同构成了后续实现所依赖的基础能力。

### Lines 10-53: Class `QianfanOCRVisionConfig`
```python
class QianfanOCRVisionConfig(PretrainedConfig):
    model_type = "qianfan_ocr_vision"

    def __init__(
        self,
        hidden_size: int = 1024,
        intermediate_size: int = 4096,
        num_hidden_layers: int = 24,
        num_attention_heads: int = 16,
        num_channels: int = 3,
        image_size: int = 448,
        patch_size: int = 14,
        hidden_act: str = "gelu",
        layer_norm_eps: float = 1e-6,
        attention_dropout: float = 0.0,
        drop_path_rate: float = 0.1,
        qkv_bias: bool = True,
        qk_normalization: bool = False,
        norm_type: str = "layer_norm",
        initializer_range: float = 0.02,
        initializer_factor: float = 0.1,
        use_mask_token: bool = False,
        use_mean_pooling: bool = True,
        **kwargs: Any,
    ):
        super().__init__(**kwargs)
        self.hidden_size = hidden_size
        self.intermediate_size = intermediate_size
        self.num_hidden_layers = num_hidden_layers
        self.num_attention_heads = num_attention_heads
        self.num_channels = num_channels
        self.image_size = image_size
        self.patch_size = patch_size
        self.hidden_act = hidden_act
        self.layer_norm_eps = layer_norm_eps
        self.attention_dropout = attention_dropout
        self.drop_path_rate = drop_path_rate
        self.qkv_bias = qkv_bias
        self.qk_normalization = qk_normalization
        self.norm_type = norm_type
        self.initializer_range = initializer_range
        self.initializer_factor = initializer_factor
        self.use_mask_token = use_mask_token
        self.use_mean_pooling = use_mean_pooling
```
**EN:** Defines `QianfanOCRVisionConfig`, a model-configuration class derived from `PretrainedConfig`. Key methods include `__init__`.
**CN:** 定义 `QianfanOCRVisionConfig`，这是一个模型配置类，继承自 `PretrainedConfig`。 关键方法包括 `__init__`。

### Lines 56-105: Class `QianfanOCRConfig`
```python
class QianfanOCRConfig(PretrainedConfig):
    model_type = "qianfan_ocr"

    def __init__(
        self,
        vision_config: dict | None = None,
        text_config: dict | None = None,
        downsample_ratio: float = 0.5,
        dynamic_image_size: bool = True,
        force_image_size: int = 448,
        image_token_id: int = 151671,
        max_dynamic_patch: int = 12,
        min_dynamic_patch: int = 1,
        pad2square: bool = False,
        ps_version: str = "v2",
        select_layer: int = -1,
        template: str = "internvl2_5",
        use_thumbnail: bool = True,
        tie_word_embeddings: bool = False,
        **kwargs: Any,
    ):
        super().__init__(**kwargs)

        if isinstance(vision_config, dict):
            self.vision_config = QianfanOCRVisionConfig(**vision_config)
        elif vision_config is None:
            self.vision_config = QianfanOCRVisionConfig()
        else:
            self.vision_config = vision_config

        if isinstance(text_config, dict):
            model_type = text_config.get("model_type", "qwen3")
            self.text_config = CONFIG_MAPPING[model_type](**text_config)
        elif text_config is None:
            self.text_config = CONFIG_MAPPING["qwen3"]()
        else:
            self.text_config = text_config

        self.downsample_ratio = downsample_ratio
        self.dynamic_image_size = dynamic_image_size
        self.force_image_size = force_image_size
        self.image_token_id = image_token_id
        self.max_dynamic_patch = max_dynamic_patch
        self.min_dynamic_patch = min_dynamic_patch
        self.pad2square = pad2square
        self.ps_version = ps_version
        self.select_layer = select_layer
        self.template = template
        self.use_thumbnail = use_thumbnail
        self.tie_word_embeddings = tie_word_embeddings
```
**EN:** Defines `QianfanOCRConfig`, a model-configuration class derived from `PretrainedConfig`. Key methods include `__init__`.
**CN:** 定义 `QianfanOCRConfig`，这是一个模型配置类，继承自 `PretrainedConfig`。 关键方法包括 `__init__`。

## Key Concepts / 关键概念
- **EN:** Configuration classes extend upstream Hugging Face metadata with vLLM-specific defaults, validation, or compatibility patches.
- **CN:** 配置类会在上游 Hugging Face 元数据之上加入 vLLM 专用默认值、校验逻辑或兼容补丁。
- **EN:** The module is tightly coupled to Hugging Face Transformers APIs, so version compatibility is often an explicit concern.
- **CN:** 该模块与 Hugging Face Transformers API 紧密耦合，因此版本兼容性通常是显式关注点。

## Dependencies / 依赖关系
- **EN:** Standard library modules: `typing`.
- **CN:** 标准库模块：`typing`。
- **EN:** External packages: `transformers`, `transformers.models.auto`.
- **CN:** 外部依赖包：`transformers`, `transformers.models.auto`。
- **EN:** Internal modules: none.
- **CN:** 内部模块：无。
