# keye_vl1_5.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/models/keye_vl1_5.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Multimodal model integration for keye_vl1_5, including encoder/decoder glue and vLLM runtime adaptation. / 面向 keye_vl1_5 的多模态模型集成模块，包含编解码胶水层与 vLLM 运行时适配逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-48)
```python
import itertools
from collections.abc import Iterator, Mapping, Sequence
from functools import partial
from typing import Annotated, Any, Literal, TypeAlias

import numpy as np
import torch
import torch.nn as nn
from einops import rearrange
from transformers import PretrainedConfig
from transformers.activations import GELUActivation
from transformers.feature_extraction_utils import BatchFeature

from vllm.config import VllmConfig
from vllm.inputs import ModalityData
from vllm.logger import init_logger
from vllm.model_executor.layers.linear import ColumnParallelLinear, RowParallelLinear
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import (
    ImageItem,
    MultiModalFeatureSpec,
    MultiModalFieldConfig,
    MultiModalKwargsItems,
    VideoItem,
)
from vllm.multimodal.parse import (
    DictEmbeddingItems,
    ModalityDataItems,
    MultiModalDataItems,
    MultiModalDataParser,
)
from vllm.multimodal.processing import (
    PromptReplacement,
    PromptUpdate,
    PromptUpdateDetails,
)
from vllm.utils.tensor_schema import TensorSchema, TensorShape

from .interfaces import SupportsLoRA, SupportsMRoPE, SupportsMultiModal, SupportsPP
from .keye import (
    BaseKeyeModule,
    BaseMultiModalProcessor,
    KeyeBaseDummyInputsBuilder,
    KeyeProcessingInfo,
)
```
**EN:** This import block gathers the library, framework, and vLLM-specific symbols required by the rest of the module.
**CN:** 这一组导入语句汇集了后续模块实现所需的标准库、框架组件以及 vLLM 专用符号。

### Function `split_thw` (lines 53-73)
```python
def split_thw(grid_thw: torch.Tensor) -> torch.Tensor:
    """
    Split grid_thw in t dimension.

    Args:
        grid_thw: [N, 3] tensor of [t, h, w]

    Returns:
        [Σt, 3] tensor where each row is [1, h, w]

    Example:
    >>> grid_thw = torch.tensor([[2, 3, 4], [1, 5, 6]])
    >>> split_thw(grid_thw)
    tensor([[1, 3, 4],
           [1, 3, 4],
           [1, 5, 6]])
    """
    t = grid_thw[:, 0]
    h_w = grid_thw[:, 1:]
    ones = torch.ones_like(h_w[:, :1])
    return torch.cat([ones, h_w], dim=1).repeat_interleave(t, dim=0)
```
**EN:** Function `split_thw` encapsulates a focused piece of reusable logic inside this module. The docstring says: Split grid_thw in t dimension.
**CN:** Function `split_thw` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Split grid_thw in t dimension。

### Function `get_num_patches` (lines 76-120)
```python
def get_num_patches(
    grid_thw: torch.Tensor, num_frames: list[int] | torch.Tensor
) -> list[int]:
    """
    Return num_patches per video.

    Args:
        grid_thw: Tensor with shape [N, 3] containing temporal, height, width
            dimensions
        num_frames: List or tensor indicating the number of frames per video

    Returns:
        List of ints representing the number of patches for each video

    Examples:
        >>> # Suppose there are 2 videos with a total of 3 grids
        >>> grid_thw = torch.tensor(
        ...     [
        ...         [2, 2, 2],  # grid 0: 2*2*2=8 patches
        ...         [2, 2, 2],  # grid 1: 2*2*2=8 patches
        ...         [1, 1, 1],
        ...     ]
        ... )  # grid 2: 1*1*1=1 patches
        >>> num_frames = [2, 1]  # The first video contains 2 grids,
                                   the second contains 1 grid.
        >>> get_num_patches(grid_thw, num_frames)
        tensor([16, 1])  # Total patches for first video: 8+8=16,
                           second video: 1.
    """

    assert len(grid_thw.shape) == 2
    if isinstance(num_frames, torch.Tensor):
        num_frames = num_frames.clone().tolist()

    num_grids_per_frame = grid_thw.prod(dim=1)
    start_idx_per_video = [0, *itertools.accumulate(num_frames)]
    num_patches = [
        num_grids_per_frame[start_idx_per_video[i] : start_idx_per_video[i + 1]].sum()
        for i in range(len(num_frames))
    ]
    return (
        torch.stack(num_patches)
        if num_patches
        else torch.zeros(0, dtype=grid_thw.dtype, device=grid_thw.device)
    )
```
**EN:** Function `get_num_patches` encapsulates a focused piece of reusable logic inside this module. The docstring says: Return num_patches per video.
**CN:** Function `get_num_patches` 封装了该模块中的一段可复用核心逻辑。 文档字符串表达的核心意思是：Return num_patches per video。

### Class `KeyeVL1_5ImagePixelInputs` (lines 123-139)
```python
class KeyeVL1_5ImagePixelInputs(TensorSchema):
    """
    Dimensions:
        - bnp: Batch size * Number of patches
        - c: Number of channels
        - ps: Patch size
        - ni: Number of images
        - g: Grid dimensions (3 for t, h, w)
    """

    type: Literal["pixel_values"]

    pixel_values: Annotated[
        torch.Tensor, TensorShape("bnp", 3, "ps", "ps", dynamic_dims={"bnp"})
    ]

    image_grid_thw: Annotated[torch.Tensor, TensorShape("ni", 3)]
```
**EN:** Class `KeyeVL1_5ImagePixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `KeyeVL1_5ImagePixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `KeyeVL1_5ImageEmbeddingInputs` (lines 142-154)
```python
class KeyeVL1_5ImageEmbeddingInputs(TensorSchema):
    """
    Dimensions:
        - nf: Number of image features
        - hs: Hidden size (must match the hidden size of language model
          backbone)
        - ni: Number of images
        - g: Grid dimensions (3 for t, h, w)
    """

    type: Literal["image_embeds"]
    image_embeds: Annotated[torch.Tensor, TensorShape("nf", "hs")]
    image_grid_thw: Annotated[torch.Tensor, TensorShape("ni", 3)]
```
**EN:** Class `KeyeVL1_5ImageEmbeddingInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `KeyeVL1_5ImageEmbeddingInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `KeyeVL1_5VideoPixelInputs` (lines 162-178)
```python
class KeyeVL1_5VideoPixelInputs(TensorSchema):
    """
    Dimensions:
        - bnp: Batch size * Number of patches
        - c: Number of channels
        - ps: Patch size
        - ni: Number of images
        - g: Grid dimensions (3 for t, h, w)
    """

    type: Literal["pixel_values_videos"]
    pixel_values_videos: Annotated[
        torch.Tensor, TensorShape("bnp", 3, "ps", "ps", dynamic_dims={"bnp"})
    ]
    video_grid_thw: Annotated[torch.Tensor, TensorShape("nv", 3)]

    num_frames: torch.Tensor
```
**EN:** Class `KeyeVL1_5VideoPixelInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `KeyeVL1_5VideoPixelInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `KeyeVL1_5VideoEmbeddingInputs` (lines 181-194)
```python
class KeyeVL1_5VideoEmbeddingInputs(TensorSchema):
    """
    Dimensions:
        - nf: Number of video features
        - hs: Hidden size (must match the hidden size of language model
          backbone)
        - nv: Number of videos
        - g: Grid dimensions (3 for t, h, w)
    """

    type: Literal["video_embeds"]
    video_embeds: Annotated[torch.Tensor, TensorShape("nf", "hs")]
    video_grid_thw: Annotated[torch.Tensor, TensorShape("nv", 3)]
    num_frames: torch.Tensor
```
**EN:** Class `KeyeVL1_5VideoEmbeddingInputs` organizes related behavior for this model family or helper component. It inherits from TensorSchema.
**CN:** 类 `KeyeVL1_5VideoEmbeddingInputs` 用于组织该模型族或辅助组件的相关行为。 它继承自 TensorSchema。

### Class `KeyeVL1_5Projector` (lines 202-274)
```python
class KeyeVL1_5Projector(nn.Module):
    def __init__(
        self,
        text_config: PretrainedConfig,
        vision_config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.text_config = text_config
        self.vision_config = vision_config
        self.merge_kernel_size = (2, 2)

        self.hidden_size = (
            self.vision_config.hidden_size
            * self.merge_kernel_size[0]
            * self.merge_kernel_size[1]
        )

        self.pre_norm = torch.nn.LayerNorm(self.hidden_size, eps=1e-05)
        self.act = GELUActivation()

        self.linear_1 = ColumnParallelLinear(
            self.hidden_size,
            self.hidden_size,
```
**EN:** Class `KeyeVL1_5Projector` is a structural model block in the vLLM execution graph. It inherits from nn.Module. Key methods include __init__, forward.
**CN:** 类 `KeyeVL1_5Projector` 是 vLLM 执行图中的结构化模型模块。 它继承自 nn.Module。 关键方法包括 __init__, forward。

### Method `KeyeVL1_5Projector.__init__` (lines 203-237)
```python
    def __init__(
        self,
        text_config: PretrainedConfig,
        vision_config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ):
        super().__init__()
        self.text_config = text_config
        self.vision_config = vision_config
        self.merge_kernel_size = (2, 2)

        self.hidden_size = (
            self.vision_config.hidden_size
            * self.merge_kernel_size[0]
            * self.merge_kernel_size[1]
        )

        self.pre_norm = torch.nn.LayerNorm(self.hidden_size, eps=1e-05)
        self.act = GELUActivation()

        self.linear_1 = ColumnParallelLinear(
            self.hidden_size,
            self.hidden_size,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.linear_1",
        )
        self.linear_2 = RowParallelLinear(
            self.hidden_size,
            self.text_config.hidden_size,
            bias=True,
            quant_config=quant_config,
            prefix=f"{prefix}.linear_2",
        )
```
**EN:** Method `KeyeVL1_5Projector.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `KeyeVL1_5Projector.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `KeyeVL1_5Projector.forward` (lines 239-274)
```python
    def forward(
        self,
        image_features: torch.Tensor | tuple[torch.Tensor] | list[torch.Tensor],
        image_grid_thw: list[tuple[int, int, int]],
    ) -> torch.Tensor | list[torch.Tensor]:
        m1, m2 = self.merge_kernel_size
        if isinstance(image_features, (list, tuple)):
            processed_features = list()
            for image_feature, image_grid in zip(image_features, image_grid_thw):
                t, h, w = image_grid
                image_feature = rearrange(
                    image_feature,
                    "(t h p1 w p2) d -> (t h w) (p1 p2 d)",
                    t=t,
                    h=h // m1,
                    p1=m1,
                    w=w // m2,
                    p2=m2,
                )
                image_feature = self.pre_norm(image_feature)
                hidden_states, _ = self.linear_1(image_feature)
                hidden_states = self.act(hidden_states)
                hidden_states, _ = self.linear_2(hidden_states)
                processed_features.append(hidden_states)

            return processed_features

        dims = image_features.shape[:-1]
        dim = image_features.shape[-1]
        image_features = image_features.view(np.prod(dims), dim)
        hidden_states = self.pre_norm(image_features.view(-1, self.hidden_size))
        hidden_states = self.linear_1(hidden_states)
        hidden_states = self.act(hidden_states)
        hidden_states = self.linear_2(hidden_states)

        return hidden_states.view(*dims, -1)
```
**EN:** Method `KeyeVL1_5Projector.forward` is the main execution path: it transforms inputs through the module and returns the activations or outputs consumed downstream.
**CN:** Method `KeyeVL1_5Projector.forward` 是主要执行路径：它将输入送入模块内部变换，并返回下游消费的激活值或输出。

### Class `KeyeVL1_5MultiModalDataParser` (lines 322-355)
```python
class KeyeVL1_5MultiModalDataParser(MultiModalDataParser):
    def _parse_image_data(
        self,
        data: dict[str, torch.Tensor] | ModalityData[ImageItem],
    ) -> ModalityDataItems[Any, Any] | None:
        if isinstance(data, dict):
            return DictEmbeddingItems(
                data,
                modality="image",
                required_fields={
                    "image_embeds",
                    "image_grid_thw",
                },
                fields_factory=_keye_field_config,
            )

        return super()._parse_image_data(data)

    def _parse_video_data(
        self,
        data: dict[str, torch.Tensor] | ModalityData[VideoItem],
    ) -> ModalityDataItems[Any, Any] | None:
        if isinstance(data, dict):
            return DictEmbeddingItems(
                data,
```
**EN:** Class `KeyeVL1_5MultiModalDataParser` organizes related behavior for this model family or helper component. It inherits from MultiModalDataParser. Key methods include _parse_image_data, _parse_video_data.
**CN:** 类 `KeyeVL1_5MultiModalDataParser` 用于组织该模型族或辅助组件的相关行为。 它继承自 MultiModalDataParser。 关键方法包括 _parse_image_data, _parse_video_data。

### Class `KeyeVL1_5ProcessingInfo` (lines 358-370)
```python
class KeyeVL1_5ProcessingInfo(KeyeProcessingInfo):
    def get_data_parser(self):
        return KeyeVL1_5MultiModalDataParser(
            expected_hidden_size=self._get_expected_hidden_size(),
        )

    def get_max_frame_per_video(self) -> int:
        return 2048

    def get_supported_mm_limits(
        self,
    ) -> Mapping[str, int | None]:
        return {"image": None, "video": 1}
```
**EN:** Class `KeyeVL1_5ProcessingInfo` organizes related behavior for this model family or helper component. It inherits from KeyeProcessingInfo. Key methods include get_data_parser, get_max_frame_per_video, get_supported_mm_limits.
**CN:** 类 `KeyeVL1_5ProcessingInfo` 用于组织该模型族或辅助组件的相关行为。 它继承自 KeyeProcessingInfo。 关键方法包括 get_data_parser, get_max_frame_per_video, get_supported_mm_limits。

### Method `KeyeVL1_5ProcessingInfo.get_data_parser` (lines 359-362)
```python
    def get_data_parser(self):
        return KeyeVL1_5MultiModalDataParser(
            expected_hidden_size=self._get_expected_hidden_size(),
        )
```
**EN:** Method `KeyeVL1_5ProcessingInfo.get_data_parser` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KeyeVL1_5ProcessingInfo.get_data_parser` 封装了该模块中的一段可复用核心逻辑。

### Method `KeyeVL1_5ProcessingInfo.get_max_frame_per_video` (lines 364-365)
```python
    def get_max_frame_per_video(self) -> int:
        return 2048
```
**EN:** Method `KeyeVL1_5ProcessingInfo.get_max_frame_per_video` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KeyeVL1_5ProcessingInfo.get_max_frame_per_video` 封装了该模块中的一段可复用核心逻辑。

### Class `KeyeVL1_5MultiModalProcessor` (lines 373-500)
```python
class KeyeVL1_5MultiModalProcessor(BaseMultiModalProcessor[KeyeVL1_5ProcessingInfo]):
    def _call_hf_processor(
        self,
        prompt: str,
        mm_data: Mapping[str, object],
        mm_kwargs: Mapping[str, object],
        tok_kwargs: Mapping[str, object],
    ) -> BatchFeature:
        # Override to use the text path instead of token path to use the
        # video-specific logic in processing_keye.py
        return super()._call_hf_processor(prompt, mm_data, mm_kwargs, tok_kwargs)

    def _get_prompt_updates(
        self,
        mm_items: MultiModalDataItems,
        hf_processor_mm_kwargs: Mapping[str, Any],
        out_mm_kwargs: MultiModalKwargsItems,
    ) -> Sequence[PromptUpdate]:
        hf_processor = self.info.get_hf_processor(**hf_processor_mm_kwargs)
        image_processor = self.info.get_image_processor(**hf_processor_mm_kwargs)
        tokenizer = self.info.get_tokenizer()
        vocab = tokenizer.get_vocab()
        image_token_id = vocab[hf_processor.image_token]
        video_token_id = vocab[hf_processor.video_token]
        placeholder = {"image": image_token_id, "video": video_token_id}
```
**EN:** Class `KeyeVL1_5MultiModalProcessor` organizes related behavior for this model family or helper component. It inherits from BaseMultiModalProcessor[KeyeVL1_5ProcessingInfo]. Key methods include _call_hf_processor, _get_prompt_updates, _get_mm_fields_config.
**CN:** 类 `KeyeVL1_5MultiModalProcessor` 用于组织该模型族或辅助组件的相关行为。 它继承自 BaseMultiModalProcessor[KeyeVL1_5ProcessingInfo]。 关键方法包括 _call_hf_processor, _get_prompt_updates, _get_mm_fields_config。

### Class `KeyeVL1_5DummyInputsBuilder` (lines 503-505)
```python
class KeyeVL1_5DummyInputsBuilder(
    KeyeBaseDummyInputsBuilder[KeyeVL1_5ProcessingInfo]
): ...
```
**EN:** Class `KeyeVL1_5DummyInputsBuilder` organizes related behavior for this model family or helper component. It inherits from KeyeBaseDummyInputsBuilder[KeyeVL1_5ProcessingInfo].
**CN:** 类 `KeyeVL1_5DummyInputsBuilder` 用于组织该模型族或辅助组件的相关行为。 它继承自 KeyeBaseDummyInputsBuilder[KeyeVL1_5ProcessingInfo]。

### Class `KeyeVL1_5ForConditionalGeneration` (lines 513-715)
```python
@MULTIMODAL_REGISTRY.register_processor(
    KeyeVL1_5MultiModalProcessor,
    info=KeyeVL1_5ProcessingInfo,
    dummy_inputs=KeyeVL1_5DummyInputsBuilder,
)
class KeyeVL1_5ForConditionalGeneration(
    BaseKeyeModule, SupportsMultiModal, SupportsLoRA, SupportsPP, SupportsMRoPE
):
    def _build_projector(
        self,
        text_config: PretrainedConfig,
        vision_config: PretrainedConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> nn.Module:
        return KeyeVL1_5Projector(text_config, vision_config, quant_config, prefix)

    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        config: PretrainedConfig = vllm_config.model_config.hf_config
        self.merge_size = config.vision_config.spatial_merge_size
        super().__init__(vllm_config=vllm_config, prefix=prefix)

    def _parse_and_validate_image_input(
        self, **kwargs: object
    ) -> KeyeVL1_5ImageInputs | None:
```
**EN:** Class `KeyeVL1_5ForConditionalGeneration` is a structural model block in the vLLM execution graph. It inherits from BaseKeyeModule, SupportsMultiModal, SupportsLoRA, SupportsPP, SupportsMRoPE. Key methods include _build_projector, __init__, _parse_and_validate_image_input, _parse_and_validate_video_input, _process_video_input, iter_mm_grid_thw.
**CN:** 类 `KeyeVL1_5ForConditionalGeneration` 是 vLLM 执行图中的结构化模型模块。 它继承自 BaseKeyeModule、SupportsMultiModal、SupportsLoRA、SupportsPP、SupportsMRoPE。 关键方法包括 _build_projector, __init__, _parse_and_validate_image_input, _parse_and_validate_video_input, _process_video_input, iter_mm_grid_thw。

### Method `KeyeVL1_5ForConditionalGeneration.__init__` (lines 525-528)
```python
    def __init__(self, *, vllm_config: VllmConfig, prefix: str = ""):
        config: PretrainedConfig = vllm_config.model_config.hf_config
        self.merge_size = config.vision_config.spatial_merge_size
        super().__init__(vllm_config=vllm_config, prefix=prefix)
```
**EN:** Method `KeyeVL1_5ForConditionalGeneration.__init__` initializes the object, creates submodules, and captures the configuration required for later execution.
**CN:** Method `KeyeVL1_5ForConditionalGeneration.__init__` 负责初始化对象、创建子模块，并保存后续执行所需的配置。

### Method `KeyeVL1_5ForConditionalGeneration.iter_mm_grid_thw` (lines 611-655)
```python
    def iter_mm_grid_thw(
        self, mm_features: list[MultiModalFeatureSpec]
    ) -> Iterator[tuple[int, int, int, int]]:
        spatial_merge_size = self.config.vision_config.spatial_merge_size

        for mm_feature in sorted(mm_features, key=lambda f: f.mm_position.offset):
            if mm_feature.data is None:
                raise ValueError("M-RoPE calculation requires multimodal feature data")

            embed_ranges = mm_feature.mm_position.extract_embeds_range()
            if mm_feature.modality == "image":
                assert len(embed_ranges) == 1
                grid_thw = mm_feature.data["image_grid_thw"].data
                if isinstance(grid_thw, torch.Tensor):
                    if grid_thw.ndim == 2:
                        assert grid_thw.shape[0] == 1
                        t, h, w = grid_thw[0].tolist()
                    else:
                        t, h, w = grid_thw.tolist()
                else:
                    if isinstance(grid_thw[0], list):
                        assert len(grid_thw) == 1
                        t, h, w = grid_thw[0]
                    else:
                        t, h, w = grid_thw

                yield (
                    embed_ranges[0][0],
                    t,
                    h // spatial_merge_size,
                    w // spatial_merge_size,
                )
            elif mm_feature.modality == "video":
                split_video_grids = split_thw(mm_feature.data["video_grid_thw"].data)
                assert len(embed_ranges) == split_video_grids.shape[0]
                for (start_idx, end_idx), (t, h, w) in zip(
                    embed_ranges, split_video_grids.tolist()
                ):
                    llm_grid_h = h // spatial_merge_size
                    llm_grid_w = w // spatial_merge_size
                    num_mm_tokens = t * llm_grid_h * llm_grid_w
                    assert end_idx - start_idx + 1 == num_mm_tokens
                    yield (start_idx, t, llm_grid_h, llm_grid_w)
            else:
                raise ValueError(f"Unsupported modality: {mm_feature.modality}")
```
**EN:** Method `KeyeVL1_5ForConditionalGeneration.iter_mm_grid_thw` encapsulates a focused piece of reusable logic inside this module.
**CN:** Method `KeyeVL1_5ForConditionalGeneration.iter_mm_grid_thw` 封装了该模块中的一段可复用核心逻辑。

## Key Concepts / 关键概念
- **LoRA support / LoRA 支持**
  - **EN:** The file is aware of low-rank adaptation interfaces or compatible parameter layouts.
  - **CN:** 该文件考虑了 LoRA 接口或兼容的低秩参数布局。
- **Multimodality / 多模态**
  - **EN:** The implementation fuses or coordinates text features with image/audio/other modality signals.
  - **CN:** 该实现负责融合或协调文本特征与图像/音频/其他模态信号。
- **Quantization / 量化**
  - **EN:** Some execution paths are adapted to quantized weights, kernels, or configuration branches.
  - **CN:** 部分执行路径会适配量化权重、算子或配置分支。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import itertools`, `from collections.abc import Iterator, Mapping, Sequence`, `from functools import partial`, `from typing import Annotated, Any, Literal, TypeAlias`, `from einops import rearrange`
- **Third-party / 第三方**: `import numpy as np`, `import torch`, `import torch.nn as nn`, `from transformers import PretrainedConfig`, `from transformers.activations import GELUActivation`, `from transformers.feature_extraction_utils import BatchFeature`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.inputs import ModalityData`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.linear import ColumnParallelLinear, RowParallelLinear`, `from vllm.model_executor.layers.quantization import QuantizationConfig`, `from vllm.multimodal import MULTIMODAL_REGISTRY`, `from vllm.multimodal.inputs import (`, `from vllm.multimodal.parse import (`, `from vllm.multimodal.processing import (`, `from vllm.utils.tensor_schema import TensorSchema, TensorShape`, `from .interfaces import SupportsLoRA, SupportsMRoPE, SupportsMultiModal, SupportsPP`, `from .keye import (`
