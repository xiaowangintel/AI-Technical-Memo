# granite4_vision.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/transformers_utils/processors/granite4_vision.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Implements model-specific processor integration for Granite4 Vision inputs. / [CN] 为 Granite4 Vision 输入实现模型专用的处理器集成。

## Line-by-Line Analysis / 逐行分析

### Lines 4-7: Imports
```python
from fractions import Fraction

from transformers import LlavaNextProcessor
from transformers.image_processing_utils import select_best_resolution
```
**EN:** This import block assembles the module's dependencies: standard-library helpers such as `fractions`, external APIs such as `transformers`, `transformers.image_processing_utils`, and internal vLLM modules such as none. Together they define the services the rest of the file builds on.
**CN:** 该导入块汇集了模块依赖：标准库工具如 `fractions`，外部 API 如 `transformers`, `transformers.image_processing_utils`，以及 vLLM 内部模块如 无。这些依赖共同构成了后续实现所依赖的基础能力。

### Lines 10-78: Class `Granite4VisionProcessor`
```python
class Granite4VisionProcessor(LlavaNextProcessor):
    """Processor for Granite 4 Vision.

    Extends LlavaNextProcessor to account for the Window Q-Former
    downsampling when computing the number of image features.

    This processor is needed because the granite4_vision processor type
    is not yet in the transformers version pinned by vLLM.
    """

    model_type = "granite4_vision"

    def __init__(
        self,
        image_processor=None,
        tokenizer=None,
        patch_size=None,
        vision_feature_select_strategy=None,
        chat_template=None,
        image_token="<image>",
        num_additional_image_tokens=0,
        downsample_rate=None,
        **kwargs,
    ):
        super().__init__(
            image_processor=image_processor,
            tokenizer=tokenizer,
            patch_size=patch_size,
            vision_feature_select_strategy=vision_feature_select_strategy,
            chat_template=chat_template,
            image_token=image_token,
            num_additional_image_tokens=num_additional_image_tokens,
        )
        self.downsample_rate = downsample_rate

    def _get_number_of_features(
        self,
        orig_height: int,
        orig_width: int,
        height: int,
# ... omitted for brevity ...
        unpadded_features, newline_features = self._get_unpadded_features(
            orig_height,
            orig_width,
            patches_height,
            patches_width,
            scale_height,
            scale_width,
        )
        base_features = (
            patches_height * patches_width + self.num_additional_image_tokens
        )
        return unpadded_features + newline_features + base_features
```
**EN:** Defines `Granite4VisionProcessor`, a processor-related class derived from `LlavaNextProcessor`. The class docstring highlights that Processor for Granite 4 Vision. Key methods include `__init__`, `_get_number_of_features`.
**CN:** 定义 `Granite4VisionProcessor`，这是一个处理器相关类，继承自 `LlavaNextProcessor`。 类文档字符串进一步说明了它的职责。 关键方法包括 `__init__`, `_get_number_of_features`。

## Key Concepts / 关键概念
- **EN:** Processor wrappers unify tokenizers and modality-specific preprocessors behind a single serving-oriented interface.
- **CN:** 处理器封装把 tokenizer 与模态专用预处理器统一到面向服务的单一接口之下。
- **EN:** The module is tightly coupled to Hugging Face Transformers APIs, so version compatibility is often an explicit concern.
- **CN:** 该模块与 Hugging Face Transformers API 紧密耦合，因此版本兼容性通常是显式关注点。

## Dependencies / 依赖关系
- **EN:** Standard library modules: `fractions`.
- **CN:** 标准库模块：`fractions`。
- **EN:** External packages: `transformers`, `transformers.image_processing_utils`.
- **CN:** 外部依赖包：`transformers`, `transformers.image_processing_utils`。
- **EN:** Internal modules: none.
- **CN:** 内部模块：无。
