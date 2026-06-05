# qianfan_ocr.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/qianfan_ocr.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements the Qianfan OCR multimodal model adapter used for inference in vLLM. / 实现 Qianfan OCR 在 vLLM 中用于推理的多模态模型适配器。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-25)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

# QianfanOCR is built on InternVL with a Qwen3 language backbone.
# The model architecture and weights are fully compatible with InternVLChatModel,
# only the config model_type / architectures strings differ.

from transformers import PretrainedConfig

from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.quantization.fp8 import Fp8Config
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.transformers_utils.processors.internvl import (
    InternVLImageProcessor,
    InternVLProcessor,
)

from .internvl import (
    BaseInternVLDummyInputsBuilder,
    BaseInternVLMultiModalProcessor,
    BaseInternVLProcessingInfo,
    InternVLChatModel,
)
```
**EN:** The opening block establishes the module context: license header, docstring, and imports. External dependencies such as transformers supply framework primitives, while internal modules like vllm.model_executor.layers.quantization, vllm.model_executor.layers.quantization.fp8, vllm.multimodal, vllm.transformers_utils.processors.internvl connect the file to vLLM runtimes, layers, and utilities.
**CN:** 开头代码块建立了模块上下文：许可证头、文档字符串和导入。像 transformers 这样的外部依赖提供基础框架能力，而 vllm.model_executor.layers.quantization, vllm.model_executor.layers.quantization.fp8, vllm.multimodal, vllm.transformers_utils.processors.internvl 等内部模块把该文件连接到 vLLM 的运行时、层实现和工具集合。

### Class `QianfanOCRProcessingInfo` (lines 26-52)
```python
class QianfanOCRProcessingInfo(BaseInternVLProcessingInfo):
    """Image-only ProcessingInfo for QianfanOCR (no video support)."""

    def get_hf_processor(self, **kwargs: object) -> InternVLProcessor:
        config = self.get_hf_config()
        vision_config = config.vision_config

        kwargs = self.ctx.get_merged_mm_kwargs(kwargs)
        kwargs.setdefault("image_size", vision_config.image_size)
        kwargs.setdefault("min_dynamic_patch", config.min_dynamic_patch)
        kwargs.setdefault("max_dynamic_patch", config.max_dynamic_patch)
        kwargs.setdefault("dynamic_image_size", config.dynamic_image_size)
        kwargs.setdefault("use_thumbnail", config.use_thumbnail)

        image_processor = InternVLImageProcessor(**kwargs)
        image_size = image_processor.image_size
        patch_size = vision_config.patch_size
        downsample_ratio = config.downsample_ratio
        image_seq_length = int((image_size // patch_size) ** 2 * (downsample_ratio**2))

        return InternVLProcessor(
            tokenizer=self.get_tokenizer(),
            image_processor=image_processor,
            video_processor=None,
            image_seq_length=image_seq_length,
            ctx_video_token=None,
        )
```
**EN:** Defines `QianfanOCRProcessingInfo`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from BaseInternVLProcessingInfo. Key methods such as `get_hf_processor` show where construction, forward execution, or weight adaptation happens. Docstring hint: "Image-only ProcessingInfo for QianfanOCR (no video support)."
**CN:** 定义 `QianfanOCRProcessingInfo`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 BaseInternVLProcessingInfo。 `get_hf_processor` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“Image-only ProcessingInfo for QianfanOCR (no video support)。”

### Class `QianfanOCRForConditionalGeneration` (lines 55-92)
```python
@MULTIMODAL_REGISTRY.register_processor(
    BaseInternVLMultiModalProcessor,
    info=QianfanOCRProcessingInfo,
    dummy_inputs=BaseInternVLDummyInputsBuilder,
)
class QianfanOCRForConditionalGeneration(InternVLChatModel):
    """QianfanOCR multimodal model.

    Identical in structure to InternVLChatModel (InternViT vision encoder +
    pixel-shuffle MLP connector + Qwen3 language model).  This class exists
    solely to register the ``QianfanOCRForConditionalGeneration`` architecture
    name that appears in the model's config.json.
    """

    def _patch_quant_config(
        self, config: PretrainedConfig, quant_config: QuantizationConfig
    ) -> None:
        super()._patch_quant_config(config, quant_config)
        # ignore vit layers to preserve model performance
        if isinstance(quant_config, Fp8Config):
            _FP8_IGNORED_LAYERS = [
                *(
                    layer
                    for i in range(config.vision_config.num_hidden_layers)
                    for layer in [
                        f"vision_model.encoder.layers.{i}.attn.qkv",
                        f"vision_model.encoder.layers.{i}.attn.proj",
                        f"vision_model.encoder.layers.{i}.mlp.fc1",
                        f"vision_model.encoder.layers.{i}.mlp.fc2",
                    ]
                ),
                "language_model.lm_head",
                "mlp1.1",
                "mlp1.3",
            ]
            for layer in _FP8_IGNORED_LAYERS:
                if layer not in quant_config.ignored_layers:
                    quant_config.ignored_layers.append(layer)
```
**EN:** Defines `QianfanOCRForConditionalGeneration`, a modality encoder or projector that turns non-text inputs into model-ready embeddings. It inherits from InternVLChatModel. Key methods such as `_patch_quant_config` show where construction, forward execution, or weight adaptation happens. Docstring hint: "QianfanOCR multimodal model."
**CN:** 定义 `QianfanOCRForConditionalGeneration`，它是一个把非文本输入转换为模型可用嵌入的模态编码器或投影器。 它继承自 InternVLChatModel。 `_patch_quant_config` 等关键方法展示了构造、前向执行或权重适配发生的位置。 文档提示：“QianfanOCR multimodal model。”

## Key Concepts / 关键概念
- **Multimodal bridging**
  - **EN:** The file combines text decoding with image, video, audio, or OCR features before tokens are generated.
  - **CN:** 该文件在生成 token 之前把文本解码与图像、视频、音频或 OCR 特征连接起来。
- **Attention and KV cache**
  - **EN:** Core logic manages query/key/value projection and integrates with vLLM attention backends and cache layout.
  - **CN:** 核心逻辑管理 Q/K/V 投影，并与 vLLM 的注意力后端及 KV Cache 布局集成。
- **Quantized weights**
  - **EN:** Quantization hooks allow the model to load compressed checkpoints or specialized linear layers.
  - **CN:** 量化钩子使模型能够加载压缩检查点或专用线性层。

## Dependencies / 依赖关系
- **External libraries**: transformers
  - **EN:** These packages provide tensor operations, model configs, or utility primitives.
  - **CN:** 这些外部库提供张量运算、模型配置或通用基础能力。
- **vLLM internal modules**: vllm.model_executor.layers.quantization, vllm.model_executor.layers.quantization.fp8, vllm.multimodal, vllm.transformers_utils.processors.internvl
  - **EN:** Internal imports connect the file to vLLM execution, parallelism, layers, and sequence abstractions.
  - **CN:** 内部导入将该文件连接到 vLLM 的执行、并行、层实现和序列抽象。
- **Local relative modules**: .internvl
  - **EN:** Relative imports reuse sibling adapters, interfaces, and helpers in the same package.
  - **CN:** 相对导入复用了同一包中的相邻适配器、接口与辅助逻辑。
