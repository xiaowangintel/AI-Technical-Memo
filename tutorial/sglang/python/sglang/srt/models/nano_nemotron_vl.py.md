# nano_nemotron_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/nano_nemotron_vl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the nano nemotron vl multimodal runtime stack in SGLang, including model blocks, adapters, and inference helpers. / 该模块实现 SGLang 中 nano nemotron vl 的多模态运行时堆栈，包括模型模块、适配器与推理辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 18-44: Module imports
```python
import logging
from typing import Iterable

import torch
import torch.nn as nn

from sglang.srt.configs.nano_nemotron_vl import NemotronH_Nano_VL_V2_Config
from sglang.srt.layers.activation import ReLU2
from sglang.srt.layers.layernorm import RMSNorm
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.managers.mm_utils import (
    MultiModalityDataPaddingPatternTokenPairs,
    general_mm_embed_routine,
)
from sglang.srt.managers.schedule_batch import (
    Modality,
    MultimodalDataItem,
    MultimodalInputs,
)
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.model_loader.weight_utils import default_weight_loader
from sglang.srt.models.nemotron_h import NemotronHForCausalLM
from sglang.srt.models.parakeet import ProjectedParakeet
from sglang.srt.models.radio import RadioModel
from sglang.srt.multimodal.evs import EVS, EVSConfig
from sglang.srt.multimodal.evs.evs_module import VideoEVSDataItem
from sglang.srt.utils import add_prefix
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、SGLang 运行时组件。

### Lines 46-46: Top-level assign
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines or updates logger, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 logger，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 49-50: Class `NemotronH_Nano_VL_V2` overview
```python
class NemotronH_Nano_VL_V2(EVS):
    @staticmethod
```
**EN:** Defines `NemotronH_Nano_VL_V2` as a reusable runtime type derived from EVS. The class groups 13 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `NemotronH_Nano_VL_V2`，其继承关系为 EVS。这个类组织了 13 个方法，用于实现模型相关行为。

### Lines 50-52: Method `NemotronH_Nano_VL_V2.create_evs_config`
```python
    @staticmethod
    def create_evs_config(config: NemotronH_Nano_VL_V2_Config):
        return EVSConfig(video_pruning_rate=config.video_pruning_rate)
```
**EN:** This method implements `create_evs_config()` and implements a focused helper that supports the surrounding runtime flow inside `NemotronH_Nano_VL_V2` Decorators: staticmethod.
**CN:** 这个方法实现了 `create_evs_config()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `NemotronH_Nano_VL_V2` 内部调用 装饰器：staticmethod。

### Lines 54-107: Method `NemotronH_Nano_VL_V2.__init__`
```python
    def __init__(
        self,
        config: NemotronH_Nano_VL_V2_Config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__(config)

        self.downsample_ratio = config.downsample_ratio
        self.language_model = NemotronHForCausalLM(
            config=config.llm_config,
            quant_config=quant_config,
            prefix=add_prefix("language_model", prefix),
        )
        self.vision_model = RadioModel(config=config.create_radio_config()).to(
            self.language_model.config.dtype
        )

        vit_hidden_size = config.vit_hidden_size
        self.rmsnorm_hidden_size = (
            vit_hidden_size * int(round(1 / self.downsample_ratio)) ** 2
        )
        vision_projection_hidden_size = config.projector_hidden_size
        llm_hidden_size = config.llm_config.hidden_size
        self.llm_hidden_size = llm_hidden_size
        self.model_dtype = self.language_model.config.torch_dtype

        self.mlp1 = nn.Sequential(
            RMSNorm(
                hidden_size=self.rmsnorm_hidden_size,
                eps=1e-5,
            ),
            nn.Linear(
                self.rmsnorm_hidden_size,
                vision_projection_hidden_size,
                bias=False,
            ),
            ReLU2(),
            nn.Linear(vision_projection_hidden_size, llm_hidden_size, bias=False),
        ).to(self.model_dtype)
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 109-144: Method `NemotronH_Nano_VL_V2.pad_input_ids`
```python
    def pad_input_ids(self, input_ids: list[int], mm_inputs: MultimodalInputs):
        im_start_id: int = mm_inputs.im_start_id
        im_end_id: int = mm_inputs.im_end_id

        visual_items = [item for item in mm_inputs.mm_items if not item.is_audio()]
        audio_items = [item for item in mm_inputs.mm_items if item.is_audio()]

        all_data_offsets = []

        if visual_items:
            mm_inputs.mm_items = visual_items
            helper = MultiModalityDataPaddingPatternTokenPairs(
                [(im_start_id, im_end_id)]
            )
            input_ids = helper.pad_input_tokens(input_ids, mm_inputs)
            all_data_offsets.extend(mm_inputs.data_offsets)

        audio_start_id = getattr(mm_inputs, "audio_start_id", None)
        audio_end_id = getattr(mm_inputs, "audio_end_id", None)
        if audio_items and audio_start_id is not None and audio_end_id is not None:
            mm_inputs.mm_items = audio_items
            helper = MultiModalityDataPaddingPatternTokenPairs(
                [(audio_start_id, audio_end_id)]
            )
            input_ids = helper.pad_input_tokens(input_ids, mm_inputs)
            all_data_offsets.extend(mm_inputs.data_offsets)

        mm_inputs.mm_items = visual_items + audio_items
        mm_inputs.data_offsets = all_data_offsets

        if audio_items:
            for item in visual_items:
                if isinstance(item, VideoEVSDataItem):
                    item.pre_chunked_input_ids = input_ids

        return input_ids
```
**EN:** This method implements `pad_input_ids(input_ids: ..., mm_inputs: ...)` and implements a focused helper that supports the surrounding runtime flow inside `NemotronH_Nano_VL_V2`.
**CN:** 这个方法实现了 `pad_input_ids(input_ids: ..., mm_inputs: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `NemotronH_Nano_VL_V2` 内部调用。

### Lines 146-167: Method `NemotronH_Nano_VL_V2.pixel_shuffle`
```python
    def pixel_shuffle(self, x: torch.Tensor, scale_factor: float = 0.5) -> torch.Tensor:
        n, w, h, c = x.size()
        # N, W, H, C --> N, W, H * scale, C // scale
        x = x.view(
            n,
            w,
            int(h * scale_factor),
            int(c / scale_factor),
        )
        # N, W, H * scale, C // scale --> N, H * scale, W, C // scale
        x = x.permute(0, 2, 1, 3).contiguous()
        # N, H * scale, W, C // scale -->
        # N, H * scale, W * scale, C // (scale ** 2)
        x = x.view(
            n,
            int(h * scale_factor),
            int(w * scale_factor),
            int(c / (scale_factor * scale_factor)),
        )
        if self.config.ps_version != "v1":
            x = x.permute(0, 2, 1, 3).contiguous()
        return x
```
**EN:** This method implements `pixel_shuffle(x: ..., scale_factor: ...=...)` and implements a focused helper that supports the surrounding runtime flow inside `NemotronH_Nano_VL_V2`.
**CN:** 这个方法实现了 `pixel_shuffle(x: ..., scale_factor: ...=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `NemotronH_Nano_VL_V2` 内部调用。

### Lines 169-189: Method `NemotronH_Nano_VL_V2.extract_feature_dynamic`
```python
    def extract_feature_dynamic(self, pixel_values_list: list[torch.Tensor]):
        """Extract features from variable-size images (dynamic resolution).

        Each image has different spatial dimensions. They are passed as a list
        to RADIO which handles ragged packing with cu_seqlens internally.
        """
        features, num_patches_list = self.vision_model(pixel_values_list)
        patch_size = self.config.patch_size
        results = []
        offset = 0
        for i, num_patches in enumerate(num_patches_list):
            img_feats = features[0, offset : offset + num_patches]
            h_patches = pixel_values_list[i].shape[-2] // patch_size
            w_patches = pixel_values_list[i].shape[-1] // patch_size
            img_feats = img_feats.reshape(1, h_patches, w_patches, -1)
            img_feats = self.pixel_shuffle(img_feats, self.downsample_ratio)
            img_feats = img_feats.view(-1, self.rmsnorm_hidden_size)
            img_feats = self.mlp1(img_feats)
            results.append(img_feats)
            offset += num_patches
        return torch.cat(results, dim=0)
```
**EN:** This method implements `extract_feature_dynamic(pixel_values_list: ...)` and Extract features from variable-size images (dynamic resolution).
**CN:** 这个方法实现了 `extract_feature_dynamic(pixel_values_list: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 191-203: Method `NemotronH_Nano_VL_V2.extract_video_feature_temporal`
```python
    def extract_video_feature_temporal(self, pixel_values, num_frames):
        """Extract video features with temporal compression (tubelet grouping)."""
        vit_embeds = self.vision_model(pixel_values, num_frames=num_frames)
        num_tubelets = vit_embeds.shape[0]
        patch_size = self.config.patch_size
        h_patches = pixel_values.shape[-2] // patch_size
        w_patches = pixel_values.shape[-1] // patch_size
        vit_embeds = vit_embeds.reshape(num_tubelets, h_patches, w_patches, -1)
        vit_embeds = self.pixel_shuffle(vit_embeds, self.downsample_ratio)
        vit_embeds = vit_embeds.view(-1, self.rmsnorm_hidden_size)
        vit_embeds = self.mlp1(vit_embeds)
        vit_embeds = vit_embeds.view(num_tubelets, -1, self.llm_hidden_size)
        return vit_embeds
```
**EN:** This method implements `extract_video_feature_temporal(pixel_values, num_frames)` and Extract video features with temporal compression (tubelet grouping).
**CN:** 这个方法实现了 `extract_video_feature_temporal(pixel_values, num_frames)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 205-206: Method `NemotronH_Nano_VL_V2.get_input_embeddings`
```python
    def get_input_embeddings(self):
        return self.language_model.get_input_embeddings()
```
**EN:** This method implements `get_input_embeddings()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 208-229: Method `NemotronH_Nano_VL_V2.extract_feature`
```python
    def extract_feature(self, pixel_values):
        micro_batch_size = 128
        n = pixel_values.shape[0]
        patch_size = self.config.patch_size
        h_patches = pixel_values.shape[-2] // patch_size
        w_patches = pixel_values.shape[-1] // patch_size
        vit_embeds_list = []
        for i in range(0, n, micro_batch_size):
            chunk = pixel_values[i : i + micro_batch_size]
            batch_size = chunk.shape[0]
            vit_embeds = self.vision_model(chunk)
            vit_embeds = vit_embeds.to(dtype=self.model_dtype)
            vit_embeds = vit_embeds.reshape(batch_size, h_patches, w_patches, -1)
            vit_embeds = self.pixel_shuffle(
                vit_embeds, scale_factor=self.downsample_ratio
            )
            vit_embeds = vit_embeds.view(-1, self.rmsnorm_hidden_size)
            vit_embeds = self.mlp1(vit_embeds)
            vit_embeds = vit_embeds.view(batch_size, -1, self.llm_hidden_size)
            vit_embeds_list.append(vit_embeds)
        vit_embeds = torch.cat(vit_embeds_list, dim=0)
        return vit_embeds
```
**EN:** This method implements `extract_feature(pixel_values)` and implements a focused helper that supports the surrounding runtime flow inside `NemotronH_Nano_VL_V2`.
**CN:** 这个方法实现了 `extract_feature(pixel_values)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `NemotronH_Nano_VL_V2` 内部调用。

### Lines 231-245: Method `NemotronH_Nano_VL_V2.get_image_feature`
```python
    def get_image_feature(self, items: list[MultimodalDataItem]):
        """
        Projects the last hidden state from the vision model into language model space.

        Returns:
            image_features (`torch.Tensor`): Image feature tensor of shape `(num_images, image_length, embed_dim)`).
        """
        is_dynamic = any(getattr(item, "is_dynamic", False) for item in items)
        if is_dynamic:
            pixel_values_list = [item.feature for item in items]
            return self.extract_feature_dynamic(pixel_values_list)

        pixel_values = torch.cat([item.feature for item in items])
        image_features = self.extract_feature(pixel_values)
        return image_features
```
**EN:** This method implements `get_image_feature(items: ...)` and Projects the last hidden state from the vision model into language model space.
**CN:** 这个方法实现了 `get_image_feature(items: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 247-259: Method `NemotronH_Nano_VL_V2.get_video_feature`
```python
    def get_video_feature(self, items: list[MultimodalDataItem]):
        """
        Projects the last hidden state from the video model into language model space.

        Returns:
            video_features (`torch.Tensor`): Video feature tensor of shape `(num_videos, video_length, embed_dim)`).
        """
        pixel_values = torch.cat([item.feature for item in items])
        if getattr(self.config, "video_temporal_patch_size", 1) > 1:
            num_frames = pixel_values.shape[0]
            return self.extract_video_feature_temporal(pixel_values, num_frames)
        video_features = self.extract_feature(pixel_values)
        return video_features
```
**EN:** This method implements `get_video_feature(items: ...)` and Projects the last hidden state from the video model into language model space.
**CN:** 这个方法实现了 `get_video_feature(items: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 261-307: Method `NemotronH_Nano_VL_V2.get_audio_feature`
```python
    def get_audio_feature(self, items: list[MultimodalDataItem]):
        """
        Encode audio features through the Parakeet sound encoder.

        Each item carries mel spectrogram features, an attention mask, and a
        clip count. Multiple clips per audio item are grouped and concatenated
        (trimmed to valid output lengths) to form a single embedding per item.
        """
        assert self.sound_encoder is not None

        all_features = []
        all_masks = []
        all_num_clips = []
        for item in items:
            all_features.append(item.feature)
            all_masks.append(item.feature_attention_mask)
            all_num_clips.append(item.audio_num_clips)

        input_audio_features = torch.cat(all_features, dim=0)
        feature_attention_mask = torch.cat(all_masks, dim=0)

        target_device = next(self.sound_encoder.parameters()).device
        input_audio_features = input_audio_features.to(
            dtype=self.language_model.config.torch_dtype, device=target_device
        )
        feature_attention_mask = feature_attention_mask.to(device=target_device)

        sound_embeds = self.sound_encoder(input_audio_features, feature_attention_mask)

        valid_input_lens = feature_attention_mask.sum(dim=1)
        valid_output_lens = (
            self.sound_encoder.encoder._get_subsampling_output_length(valid_input_lens)
            .long()
            .tolist()
        )

        grouped_embeds = []
        clip_offset = 0
        for num_clips in all_num_clips:
            embeds = []
# ... truncated for brevity ...
```
**EN:** This method implements `get_audio_feature(items: ...)` and Encode audio features through the Parakeet sound encoder.
**CN:** 这个方法实现了 `get_audio_feature(items: ...)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 309-332: Method `NemotronH_Nano_VL_V2.forward`
```python
    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        get_embedding: bool = False,
    ):
        data_embedding_funcs = {
            Modality.IMAGE: self.get_image_feature,
            Modality.VIDEO: self.get_video_feature,
        }
        if self.sound_encoder is not None:
            data_embedding_funcs[Modality.AUDIO] = self.get_audio_feature

        hidden_states = general_mm_embed_routine(
            input_ids=input_ids,
            forward_batch=forward_batch,
            language_model=self.language_model,
            multimodal_model=self,
            data_embedding_funcs=data_embedding_funcs,
            positions=positions,
        )
        return hidden_states
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., get_embedding: ...=...)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., get_embedding: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 334-374: Method `NemotronH_Nano_VL_V2.load_weights`
```python
    def load_weights(self, weights: Iterable[tuple[str, torch.Tensor]]):
        adapter_dict = dict(self.mlp1.named_parameters())

        def is_llm(name: str) -> bool:
            return name.startswith("language_model")

        def is_adapter_weights(weight: tuple[str, torch.Tensor]):
            return weight[0].startswith("mlp1")

        def is_vision_weights(name: str) -> bool:
            return name.startswith("vision_model.radio_model.")

        def is_sound_weights(name: str) -> bool:
            return name.startswith("sound")

        # Separate weights by component
        llm_weights = []
        vision_weights = []
        sound_weights = []

        for name, w in weights:
            if is_llm(name):
                # Strip 'language_model.' prefix for LLM weights
                llm_weights.append((".".join(name.split(".")[1:]), w))
            elif is_adapter_weights((name, w)):
                # Load vision-language adapter weights directly
                trimmed_name = ".".join(name.split(".")[1:])
                param = adapter_dict[trimmed_name]
                with torch.no_grad():
                    default_weight_loader(param, w)
            elif is_vision_weights(name):
                # Convert: vision_model.radio_model.* → radio_model.*
                hf_key = name[len("vision_model.") :]
                vision_weights.append((hf_key, w))
            elif is_sound_weights(name):
                sound_weights.append((name, w))

        self.language_model.load_weights(llm_weights)
        self.vision_model.load_weights(vision_weights)
        if self.sound_encoder is not None and len(sound_weights) > 0:
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 377-377: Class `NemotronH_Nano_Omni_Reasoning_V3` overview
```python
class NemotronH_Nano_Omni_Reasoning_V3(NemotronH_Nano_VL_V2):
```
**EN:** Defines `NemotronH_Nano_Omni_Reasoning_V3` as a reusable runtime type derived from NemotronH_Nano_VL_V2. The class groups 0 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `NemotronH_Nano_Omni_Reasoning_V3`，其继承关系为 NemotronH_Nano_VL_V2。这个类组织了 0 个方法，用于实现模型相关行为。

### Lines 378-378: Class `NemotronH_Nano_Omni_Reasoning_V3` support logic
```python
    pass
```
**EN:** Contains supporting top-level logic that prepares the runtime environment for the model definitions below.
**CN:** 包含支撑性的顶层逻辑，用于为下方模型定义准备运行时环境。

### Lines 381-381: Top-level assign
```python
EntryClass = [NemotronH_Nano_VL_V2, NemotronH_Nano_Omni_Reasoning_V3]
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
- `typing.Iterable`
- `torch`
- `torch.nn`
- `sglang.srt.configs.nano_nemotron_vl.NemotronH_Nano_VL_V2_Config`
- `sglang.srt.layers.activation.ReLU2`
- `sglang.srt.layers.layernorm.RMSNorm`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.managers.mm_utils.MultiModalityDataPaddingPatternTokenPairs`
- `sglang.srt.managers.mm_utils.general_mm_embed_routine`
- `sglang.srt.managers.schedule_batch.Modality`
- `sglang.srt.managers.schedule_batch.MultimodalDataItem`
- `sglang.srt.managers.schedule_batch.MultimodalInputs`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.models.nemotron_h.NemotronHForCausalLM`
- `sglang.srt.models.parakeet.ProjectedParakeet`
- `sglang.srt.models.radio.RadioModel`
- `sglang.srt.multimodal.evs.EVS`
- `sglang.srt.multimodal.evs.EVSConfig`
- `sglang.srt.multimodal.evs.evs_module.VideoEVSDataItem`
- `sglang.srt.utils.add_prefix`
