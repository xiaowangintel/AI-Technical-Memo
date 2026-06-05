# phi4mm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/phi4mm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the phi4mm model stack used by SGLang SRT inference, including network blocks, runtime wrappers, and checkpoint-loading utilities. / 该模块实现 SGLang SRT 推理所使用的 phi4mm 模型栈，包括网络模块、运行时封装以及检查点加载工具。

## Line-by-Line Analysis / 逐行分析

### Lines 20-45: Module imports
```python
import logging
import math
import re
from collections.abc import Iterable
from typing import List, Optional, Tuple

import numpy as np
import torch
from torch import nn
from transformers import PretrainedConfig

from sglang.srt.layers.quantization import QuantizationConfig
from sglang.srt.managers.mm_utils import (
    MultiModalityDataPaddingPatternMultimodalTokens,
    general_mm_embed_routine,
)
from sglang.srt.managers.schedule_batch import (
    Modality,
    MultimodalDataItem,
    MultimodalInputs,
)
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.model_loader.weight_utils import default_weight_loader
from sglang.srt.models.idefics2 import Idefics2VisionTransformer
from sglang.srt.models.llama import LlamaForCausalLM
from sglang.srt.models.phi4mm_audio import AudioEmbedding
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 47-47: Top-level assign
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines or updates logger, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 logger，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 49-49: Top-level assign
```python
SIGLIP_NAME = "siglip-so400m-patch14-448"
```
**EN:** Defines or updates SIGLIP_NAME, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 SIGLIP_NAME，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 50-56: Top-level assign
```python
VISION_ENCODER_TO_PROCESSING_CONFIG = {
    "siglip-so400m-patch14-448": {
        "vit_image_size": 448,
        "vit_patch_size": 14,
        "token_compression_factor": 2,
    },
}
```
**EN:** Defines or updates VISION_ENCODER_TO_PROCESSING_CONFIG, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 VISION_ENCODER_TO_PROCESSING_CONFIG，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 59-61: Class `Phi4MMImageEncoder` overview
```python
class Phi4MMImageEncoder(nn.Module):
    """Image embedding."""
```
**EN:** Defines `Phi4MMImageEncoder` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Phi4MMImageEncoder`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 62-136: Method `Phi4MMImageEncoder.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig],
        prefix: str = "",
        model_dir: str = "",
    ) -> None:
        super().__init__()

        # n_embed or hidden_size
        hidden_size = config.n_embd if hasattr(config, "n_embd") else config.hidden_size
        self.type_feature = "patch"
        self.img_processor = Idefics2VisionTransformer(
            config=config.vision_config, require_post_norm=False
        )

        pe_weight = self.img_processor.embeddings.position_embedding.weight
        L, D = pe_weight.size()
        H = int(math.sqrt(L))
        assert H**2 == L, f"position embedding size {L} is not square"
        if H % 2 != 0:
            self.img_processor_padding = nn.ReflectionPad2d((0, 1, 0, 1))
            H += 1
        image_dim_out = D
        # ((448/14)//2)**2
        self.num_img_tokens = (H // 2) ** 2
        self.base_feat_height_target = H

        self.image_dim_out = image_dim_out
        self.img_sizes = None
        self.image_attention_mask = None

        # global_gn and sub_gn for hd transform, serves as line separator
        self.use_hd_transform = True
        self.with_learnable_separator = True
        self.hd_transform_order = "sub_glb"
        self.freeze_img_processor = False
        self.crop_size = 448

        # image token compression
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., quant_config: ..., prefix: ...=..., model_dir: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ..., prefix: ...=..., model_dir: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 138-169: Method `Phi4MMImageEncoder.get_img_features`
```python
    def get_img_features(
        self, img_embeds: torch.FloatTensor, attention_mask=None
    ) -> torch.FloatTensor:
        img_feature = self.img_processor(
            img_embeds, patch_attention_mask=attention_mask
        )

        patch_feature = img_feature

        use_token_compression = self.image_token_compression is not None
        use_padding = getattr(self, "img_processor_padding", None) is not None
        if use_token_compression or use_padding:
            # reshape to 2D tensor
            width = int(math.sqrt(patch_feature.size(1)))
            patch_feature = patch_feature.view(-1, width, width, patch_feature.size(-1))
            # convert to NCHW
            patch_feature = patch_feature.permute(0, 3, 1, 2)

            if use_padding:
                patch_feature = self.img_processor_padding(patch_feature)
            if use_token_compression:
                patch_feature = self.image_token_compression(patch_feature)

            # convert to NHWC
            patch_feature = patch_feature.permute(0, 2, 3, 1)
            patch_feature = patch_feature.view(
                -1,
                patch_feature.size(1) * patch_feature.size(2),
                patch_feature.size(-1),
            )

        return patch_feature
```
**EN:** This method implements `get_img_features(img_embeds: ..., attention_mask=...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_img_features(img_embeds: ..., attention_mask=...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 171-376: Method `Phi4MMImageEncoder.forward`
```python
    def forward(
        self,
        pixel_values: torch.FloatTensor,
        image_sizes: torch.Tensor,
        image_attention_mask: torch.Tensor,
    ) -> list[torch.FloatTensor]:
        """
        process image and return vision embeddings.

        pixel_values: (num_images, num_crops, c, h, w)
        image_sizes: [[h1, w1], [h2, w2]]
        image_attention_mask: num_images x num_crops x 32 x 32
        output: (num_images, num_img_tokens, hidden_size)
        """

        # eg
        # pixel_values: torch.Size([1, 7, 3, 448, 448])
        # image_sizes: tensor([[ 896, 1344]], device='cuda:0')
        # output: torch.Size([1, 1841, 3072])

        img_projection_params = next(self.img_projection.parameters())
        target_device = img_projection_params.device
        target_dtype = img_projection_params.dtype

        img_sizes = image_sizes
        num_images, num_crops, c, h, w = pixel_values.shape
        bs = num_images
        pixel_values = pixel_values.flatten(0, 1)

        img_features = self.get_img_features(
            pixel_values,
            image_attention_mask.type(torch.BoolTensor).flatten(0, 1).to(target_device),
        )

        base_feat_height_target = self.base_feat_height_target
        base_resolution = self.crop_size
        base_feat_height_reduction = self.base_feat_height_reduction

        base_feat_height = base_feat_width = int(np.sqrt(img_features.shape[1]))
        assert (
# ... truncated for brevity ...
```
**EN:** This method implements `forward(pixel_values: ..., image_sizes: ..., image_attention_mask: ...)` and process image and return vision embeddings.
**CN:** 这个方法实现了 `forward(pixel_values: ..., image_sizes: ..., image_attention_mask: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 379-379: Class `Phi4MMForCausalLM` overview
```python
class Phi4MMForCausalLM(nn.Module):
```
**EN:** Defines `Phi4MMForCausalLM` as a reusable runtime type derived from nn.Module. The class groups 7 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Phi4MMForCausalLM`，其继承关系为 nn.Module。这个类组织了 7 个方法，用于实现模型相关行为。

### Lines 380-387: Class `Phi4MMForCausalLM` attributes
```python
    packed_modules_mapping = {
        "qkv_proj": ["q_proj", "k_proj", "v_proj"],
        "gate_up_proj": ["gate_proj", "up_proj"],
    }

    lora_pattern = re.compile(
        r"^language_model\.model\.layers\.(\d+)\.(?:self_attn|mlp)\.(?:qkv_proj|o_proj|down_proj|gate_up_proj)"
    )
```
**EN:** Defines class-level attributes and metadata that shape how `Phi4MMForCausalLM` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `Phi4MMForCausalLM` 在运行时的行为。

### Lines 389-416: Method `Phi4MMForCausalLM.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()

        self.language_model = LlamaForCausalLM(
            config=config, quant_config=quant_config, prefix=prefix
        )

        self.vision_encoder = Phi4MMImageEncoder(
            config,
            quant_config,
            prefix="model.vision_embed_tokens",
            model_dir=config._name_or_path,
        )

        if isinstance(config.embd_layer["audio_embd_layer"], dict):
            embedding_config = {
                "embedding_cls": config.embd_layer["audio_embd_layer"]["embedding_cls"],
                **config.embd_layer["audio_embd_layer"],
            }
        else:
            embedding_config = {"embedding_cls": config.embd_layer["embedding_cls"]}

        self.embed_tokens_extend = AudioEmbedding(config, **embedding_config)
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 418-433: Method `Phi4MMForCausalLM.get_image_feature`
```python
    def get_image_feature(self, items: List[MultimodalDataItem]) -> torch.Tensor:
        dtype = next(self.vision_encoder.parameters()).dtype
        pixel_values = torch.cat([item.feature for item in items], dim=0).type(dtype)
        image_attention_mask = torch.cat(
            [
                item.image_attention_mask
                for item in items
                if hasattr(item, "image_attention_mask")
            ],
            dim=0,
        )
        image_sizes = torch.cat([item.image_sizes for item in items], dim=0)
        image_embeds = self.vision_encoder(
            pixel_values, image_sizes, image_attention_mask
        )
        return torch.cat(image_embeds).type(dtype)
```
**EN:** This method implements `get_image_feature(items: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_image_feature(items: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 435-454: Method `Phi4MMForCausalLM.get_audio_feature`
```python
    def get_audio_feature(self, items: List[MultimodalDataItem]) -> torch.Tensor:
        # (e.g. multiple examples) and the second dim is the multi-audio dim
        # (e.g. multiple audios in the same example)
        embed_tokens_extend_param = next(self.embed_tokens_extend.parameters())
        device = embed_tokens_extend_param.device
        dtype = embed_tokens_extend_param.dtype
        audio_embeds = [
            self.embed_tokens_extend(
                # item.feature: (num_audios_in_a_sequence, T, D)
                # item.audio_attention_mask: (num_audios_in_a_sequence, T, D) BoolTensor or None
                audio_features=item.feature.type(dtype),
                audio_attention_mask=(
                    item.audio_attention_mask.to(device)
                    if hasattr(item, "audio_attention_mask")
                    else None
                ),
            )
            for item in items
        ]
        return torch.cat(audio_embeds).type(dtype)
```
**EN:** This method implements `get_audio_feature(items: ...)` and returns a cached or derived object used elsewhere in the pipeline.
**CN:** 这个方法实现了 `get_audio_feature(items: ...)`，其作用是返回流水线其他部分会使用的缓存对象或派生结果。

### Lines 456-474: Method `Phi4MMForCausalLM.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        **kwargs: object,
    ) -> torch.Tensor:
        hidden_states = general_mm_embed_routine(
            input_ids=input_ids,
            forward_batch=forward_batch,
            language_model=self.language_model,
            data_embedding_funcs={
                Modality.IMAGE: self.get_image_feature,
                Modality.AUDIO: self.get_audio_feature,
            },
            positions=positions,
        )

        return hidden_states
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., **kwargs)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., **kwargs)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 476-478: Method `Phi4MMForCausalLM.pad_input_ids`
```python
    def pad_input_ids(self, input_ids: List[int], mm_inputs: MultimodalInputs):
        pattern = MultiModalityDataPaddingPatternMultimodalTokens()
        return pattern.pad_input_tokens(input_ids, mm_inputs)
```
**EN:** This method implements `pad_input_ids(input_ids: ..., mm_inputs: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Phi4MMForCausalLM`.
**CN:** 这个方法实现了 `pad_input_ids(input_ids: ..., mm_inputs: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Phi4MMForCausalLM` 内部调用。

### Lines 480-481: Method `Phi4MMForCausalLM.should_apply_lora`
```python
    def should_apply_lora(self, module_name: str) -> bool:
        return bool(self.lora_pattern.match(module_name))
```
**EN:** This method implements `should_apply_lora(module_name: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Phi4MMForCausalLM`.
**CN:** 这个方法实现了 `should_apply_lora(module_name: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Phi4MMForCausalLM` 内部调用。

### Lines 483-537: Method `Phi4MMForCausalLM.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            (".self_attn.qkv_proj", ".self_attn.q_proj", "q"),
            (".self_attn.qkv_proj", ".self_attn.k_proj", "k"),
            (".self_attn.qkv_proj", ".self_attn.v_proj", "v"),
        ]
        prefix_mapping = {
            "model.embed_tokens_extend.audio_embed.audio_projection.vision.": "embed_tokens_extend.audio_projection_for_vision.",
            "model.embed_tokens_extend.audio_embed.audio_projection.speech.": "embed_tokens_extend.audio_projection.",
            "model.embed_tokens_extend.audio_embed.": "embed_tokens_extend.",
            "model.embed_tokens_extend.image_embed.": "vision_encoder.",
            "model.": "language_model.model.",
        }

        skip_list = [
            "img_processor.encoder.layers.26",
            "img_processor.head",
            "img_processor.post_layernorm",
        ]

        def _should_skip(name: str) -> bool:
            return any(substr in name for substr in skip_list)

        params_dict = dict(self.named_parameters())
        for name, loaded_weight in weights:
            # Skip the last layer
            if _should_skip(name):
                continue

            for old_name, new_name in prefix_mapping.items():
                if name.startswith(old_name):
                    name = name.replace(old_name, new_name)
                    break

            # Adapt to VisionAttention
            name = name.replace(r"self_attn.out_proj", r"self_attn.proj")
            name = name.replace(r"base_layer.", r"")

            for param_name, weight_name, shard_id in stacked_params_mapping:
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 540-540: Top-level assign
```python
EntryClass = [Phi4MMForCausalLM]
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影
- **EN:** Multimodal feature processing / **CN:** 多模态特征处理

## Dependencies / 依赖关系
- `logging`
- `math`
- `re`
- `collections.abc.Iterable`
- `typing.List`
- `typing.Optional`
- `typing.Tuple`
- `numpy`
- `torch`
- `torch.nn`
- `transformers.PretrainedConfig`
- `sglang.srt.layers.quantization.QuantizationConfig`
- `sglang.srt.managers.mm_utils.MultiModalityDataPaddingPatternMultimodalTokens`
- `sglang.srt.managers.mm_utils.general_mm_embed_routine`
- `sglang.srt.managers.schedule_batch.Modality`
- `sglang.srt.managers.schedule_batch.MultimodalDataItem`
- `sglang.srt.managers.schedule_batch.MultimodalInputs`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.models.idefics2.Idefics2VisionTransformer`
- `sglang.srt.models.llama.LlamaForCausalLM`
- `sglang.srt.models.phi4mm_audio.AudioEmbedding`
