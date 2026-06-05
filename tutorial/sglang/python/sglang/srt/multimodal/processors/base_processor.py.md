# base_processor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/base_processor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Defines the shared abstractions and utility data structures used by multimodal processors in the SGLang runtime. / 定义 SGLang 运行时多模态处理器共享的抽象基类与通用数据结构。

## Line-by-Line Analysis / 逐行分析

### Lines 1-37: Imports dependencies
```python
import concurrent
import concurrent.futures
import dataclasses
import multiprocessing as mp
import os
import re
from abc import ABC, abstractmethod
from typing import Any, Dict, Iterator, List, Optional, Tuple, Union

import numpy as np
import torch
from PIL import Image
from transformers import BaseImageProcessor

from sglang.srt.managers.schedule_batch import (
    Modality,
    MultimodalDataItem,
    MultimodalInputFormat,
    MultimodalProcessorOutput,
)
from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils import (
    envs,
    is_cpu,
    is_npu,
    is_xpu,
    load_audio,
    load_image,
    load_video,
    logger,
)
from sglang.srt.utils.cuda_ipc_transport_utils import (
    MM_FEATURE_CACHE_SIZE,
    MM_ITEM_MEMORY_POOL_RECYCLE_INTERVAL,
    CudaIpcTensorTransportProxy,
    MmItemMemoryPool,
)
```
**EN:** This block groups related imports for the module, including concurrent, concurrent.futures, dataclasses, multiprocessing, os, and 31 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 concurrent, concurrent.futures, dataclasses, multiprocessing, os 等 31 项，为后续代码准备所需名称。

### Lines 38-38: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 39-41: Declares _is_cpu, _is_npu, _is_xpu
```python
_is_cpu = is_cpu()
_is_npu = is_npu()
_is_xpu = is_xpu()
```
**EN:** This block initializes a related set of values in the module, including _is_cpu, _is_npu, _is_xpu. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 模块 中初始化一组相关值，包括 _is_cpu, _is_npu, _is_xpu。将这些赋值集中在一起有助于理解周边配置。

### Lines 42-42: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 43-44: Declares SGL_USE_CUDA_IPC, _IPC_POOL_HANDLE_CACHE
```python
SGL_USE_CUDA_IPC = envs.SGLANG_USE_CUDA_IPC_TRANSPORT.get()
_IPC_POOL_HANDLE_CACHE = envs.SGLANG_USE_IPC_POOL_HANDLE_CACHE.get()
```
**EN:** This block initializes a related set of values in the module, including SGL_USE_CUDA_IPC, _IPC_POOL_HANDLE_CACHE. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 模块 中初始化一组相关值，包括 SGL_USE_CUDA_IPC, _IPC_POOL_HANDLE_CACHE。将这些赋值集中在一起有助于理解周边配置。

### Lines 45-46: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 47-49: Declares class BaseMultiModalProcessorOutput
```python
@dataclasses.dataclass
class BaseMultiModalProcessorOutput:
    # input_text with all multimodality placeholder token expanded
```
**EN:** This block introduces class `BaseMultiModalProcessorOutput` as a reusable abstraction inside the module.
**CN:** 该代码块声明类 `BaseMultiModalProcessorOutput`，作为模块中的可复用抽象。

### Lines 50-50: Declares input_text
```python
    input_text: str
```
**EN:** This statement initializes input_text in the BaseMultiModalProcessorOutput. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 BaseMultiModalProcessorOutput 中初始化 input_text。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 51-52: Comments and module notes
```python

    # frames loaded from image, in given order
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the BaseMultiModalProcessorOutput.
**CN:** 该范围包含注释或说明，用于记录 BaseMultiModalProcessorOutput 的假设、来源或实现备注。

### Lines 53-55: Declares images
```python
    images: Optional[list[Union[Image.Image, dict]]] = dataclasses.field(
        default_factory=list
    )
```
**EN:** This statement initializes images in the BaseMultiModalProcessorOutput. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 BaseMultiModalProcessorOutput 中初始化 images。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 56-57: Comments and module notes
```python

    # videos
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the BaseMultiModalProcessorOutput.
**CN:** 该范围包含注释或说明，用于记录 BaseMultiModalProcessorOutput 的假设、来源或实现备注。

### Lines 58-60: Declares videos
```python
    videos: Optional[list[Union[torch.Tensor, dict]]] = dataclasses.field(
        default_factory=list
    )
```
**EN:** This statement initializes videos in the BaseMultiModalProcessorOutput. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 BaseMultiModalProcessorOutput 中初始化 videos。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 61-62: Comments and module notes
```python

    # audios
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the BaseMultiModalProcessorOutput.
**CN:** 该范围包含注释或说明，用于记录 BaseMultiModalProcessorOutput 的假设、来源或实现备注。

### Lines 63-65: Declares audios
```python
    audios: Optional[list[Union[np.ndarray, dict]]] = dataclasses.field(
        default_factory=list
    )
```
**EN:** This statement initializes audios in the BaseMultiModalProcessorOutput. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 BaseMultiModalProcessorOutput 中初始化 audios。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 66-66: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the BaseMultiModalProcessorOutput, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 BaseMultiModalProcessorOutput 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 67-76: Defines function BaseMultiModalProcessorOutput.organize_results
```python
    def organize_results(self) -> List[Tuple[Modality, Any]]:
        """

        :return: a list of results, with their corresponding modalities
        """
        return (
            [(Modality.IMAGE, data) for data in self.images]
            + [(Modality.VIDEO, data) for data in self.videos]
            + [(Modality.AUDIO, data) for data in self.audios]
        )
```
**EN:** This block defines function `BaseMultiModalProcessorOutput.organize_results`. Parameters: self. :return: a list of results, with their corresponding modalities
**CN:** 该代码块定义函数 `BaseMultiModalProcessorOutput.organize_results`。 参数包括 self。 文档字符串摘要：:return: a list of results, with their corresponding modalities

### Lines 77-78: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 79-80: Declares class MultimodalSpecialTokens
```python
@dataclasses.dataclass
class MultimodalSpecialTokens:
```
**EN:** This block introduces class `MultimodalSpecialTokens` as a reusable abstraction inside the module.
**CN:** 该代码块声明类 `MultimodalSpecialTokens`，作为模块中的可复用抽象。

### Lines 81-83: Declares image_token, video_token, audio_token
```python
    image_token: Optional[Union[str, List[str]]] = None
    video_token: Optional[Union[str, List[str]]] = None
    audio_token: Optional[Union[str, List[str]]] = None
```
**EN:** This block initializes a related set of values in the MultimodalSpecialTokens, including image_token, video_token, audio_token. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 MultimodalSpecialTokens 中初始化一组相关值，包括 image_token, video_token, audio_token。将这些赋值集中在一起有助于理解周边配置。

### Lines 84-84: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MultimodalSpecialTokens, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MultimodalSpecialTokens 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 85-87: Declares image_token_id, video_token_id, audio_token_id
```python
    image_token_id: Optional[int] = None
    video_token_id: Optional[int] = None
    audio_token_id: Optional[int] = None
```
**EN:** This block initializes a related set of values in the MultimodalSpecialTokens, including image_token_id, video_token_id, audio_token_id. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 MultimodalSpecialTokens 中初始化一组相关值，包括 image_token_id, video_token_id, audio_token_id。将这些赋值集中在一起有助于理解周边配置。

### Lines 88-88: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MultimodalSpecialTokens, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MultimodalSpecialTokens 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 89-91: Declares image_token_regex, video_token_regex, audio_token_regex
```python
    image_token_regex: Optional[re.Pattern] = None
    video_token_regex: Optional[re.Pattern] = None
    audio_token_regex: Optional[re.Pattern] = None
```
**EN:** This block initializes a related set of values in the MultimodalSpecialTokens, including image_token_regex, video_token_regex, audio_token_regex. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 MultimodalSpecialTokens 中初始化一组相关值，包括 image_token_regex, video_token_regex, audio_token_regex。将这些赋值集中在一起有助于理解周边配置。

### Lines 92-92: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MultimodalSpecialTokens, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MultimodalSpecialTokens 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 93-93: Declares combined_regex
```python
    combined_regex: Optional[re.Pattern] = None
```
**EN:** This statement initializes combined_regex in the MultimodalSpecialTokens. Such values usually act as constants, caches, defaults, or configuration switches used by later logic.
**CN:** 该语句在 MultimodalSpecialTokens 中初始化 combined_regex。这些值通常作为后续逻辑使用的常量、缓存、默认值或配置开关。

### Lines 94-94: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MultimodalSpecialTokens, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MultimodalSpecialTokens 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 95-99: Defines function MultimodalSpecialTokens.build
```python
    def build(self, processor):
        self.convert_to_strs(processor)
        self.parse_regex()
        self.get_combined_regex()
        return self
```
**EN:** This block defines function `MultimodalSpecialTokens.build`. Parameters: self, processor.
**CN:** 该代码块定义函数 `MultimodalSpecialTokens.build`。 参数包括 self、processor。

### Lines 100-100: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MultimodalSpecialTokens, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MultimodalSpecialTokens 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 101-106: Defines function MultimodalSpecialTokens.convert_to_str
```python
    def convert_to_str(self, token: Union[str, int], processor) -> str:
        if token is None:
            return token
        if isinstance(token, str):
            return token
        return processor.tokenizer.convert_ids_to_tokens([token])[0]
```
**EN:** This block defines function `MultimodalSpecialTokens.convert_to_str`. Parameters: self, token, processor.
**CN:** 该代码块定义函数 `MultimodalSpecialTokens.convert_to_str`。 参数包括 self、token、processor。

### Lines 107-107: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MultimodalSpecialTokens, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MultimodalSpecialTokens 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 108-114: Defines function MultimodalSpecialTokens.convert_to_strs
```python
    def convert_to_strs(self, processor):
        if not self.image_token:
            self.image_token = self.convert_to_str(self.image_token_id, processor)
        if not self.video_token:
            self.video_token = self.convert_to_str(self.video_token_id, processor)
        if not self.audio_token:
            self.audio_token = self.convert_to_str(self.audio_token_id, processor)
```
**EN:** This block defines function `MultimodalSpecialTokens.convert_to_strs`. Parameters: self, processor.
**CN:** 该代码块定义函数 `MultimodalSpecialTokens.convert_to_strs`。 参数包括 self、processor。

### Lines 115-115: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MultimodalSpecialTokens, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MultimodalSpecialTokens 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 116-136: Defines function MultimodalSpecialTokens.get_modality_of_token
```python
    def get_modality_of_token(self, token: str) -> Optional[Modality]:
        """
        :return: the modality associated with the given token, if the token is a special_token or matches with the multimodal token regex
        """
        modality = {
            self.image_token: Modality.IMAGE,
            self.video_token: Modality.VIDEO,
            self.audio_token: Modality.AUDIO,
        }.get(token)
        if modality:
            return modality

        for regex, modality in [
            (self.image_token_regex, Modality.IMAGE),
            (self.video_token_regex, Modality.VIDEO),
            (self.audio_token_regex, Modality.AUDIO),
        ]:
            if regex and regex.match(token):
                return modality

        return None
```
**EN:** This block defines function `MultimodalSpecialTokens.get_modality_of_token`. Parameters: self, token. :return: the modality associated with the given token, if the token is a special_token or matches with the multimodal token regex
**CN:** 该代码块定义函数 `MultimodalSpecialTokens.get_modality_of_token`。 参数包括 self、token。 文档字符串摘要：:return: the modality associated with the given token, if the token is a special_token or matches with the multimodal token regex

### Lines 137-137: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MultimodalSpecialTokens, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MultimodalSpecialTokens 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 138-143: Defines function MultimodalSpecialTokens.get_token_id_by_modality
```python
    def get_token_id_by_modality(self, modality: Modality) -> Optional[int]:
        return {
            Modality.IMAGE: self.image_token_id,
            Modality.VIDEO: self.video_token_id,
            Modality.AUDIO: self.audio_token_id,
        }.get(modality)
```
**EN:** This block defines function `MultimodalSpecialTokens.get_token_id_by_modality`. Parameters: self, modality.
**CN:** 该代码块定义函数 `MultimodalSpecialTokens.get_token_id_by_modality`。 参数包括 self、modality。

### Lines 144-144: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MultimodalSpecialTokens, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MultimodalSpecialTokens 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 145-151: Defines function MultimodalSpecialTokens.parse_regex
```python
    def parse_regex(self):
        if self.image_token_regex is None and self.image_token is not None:
            self.image_token_regex = re.compile(re.escape(self.image_token))
        if self.video_token_regex is None and self.video_token is not None:
            self.video_token_regex = re.compile(re.escape(self.video_token))
        if self.audio_token_regex is None and self.audio_token is not None:
            self.audio_token_regex = re.compile(re.escape(self.audio_token))
```
**EN:** This block defines function `MultimodalSpecialTokens.parse_regex`. Parameters: self.
**CN:** 该代码块定义函数 `MultimodalSpecialTokens.parse_regex`。 参数包括 self。

### Lines 152-152: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MultimodalSpecialTokens, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MultimodalSpecialTokens 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 153-172: Defines function MultimodalSpecialTokens.get_combined_regex
```python
    def get_combined_regex(self) -> re.Pattern:
        """
        Builds and returns a regex, used to split input str into tokens (with mm special tokens)
        """
        if self.combined_regex:
            return self.combined_regex
        tokens = [
            self.image_token_regex,
            self.video_token_regex,
            self.audio_token_regex,
        ]
        patterns = []
        flags = 0
        for t in tokens:
            if t is not None:
                patterns.append(t.pattern)
                flags |= t.flags
        combined = "(" + "|".join(f"(?:{p})" for p in patterns) + ")"
        self.combined_regex = re.compile(combined, flags)
        return self.combined_regex
```
**EN:** This block defines function `MultimodalSpecialTokens.get_combined_regex`. Parameters: self. Builds and returns a regex, used to split input str into tokens (with mm special tokens)
**CN:** 该代码块定义函数 `MultimodalSpecialTokens.get_combined_regex`。 参数包括 self。 文档字符串摘要：Builds and returns a regex, used to split input str into tokens (with mm special tokens)

### Lines 173-174: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 175-175: Declares class BaseMultimodalProcessor
```python
class BaseMultimodalProcessor(ABC):
```
**EN:** This block introduces class `BaseMultimodalProcessor` as a reusable abstraction inside the module. It inherits from ABC.
**CN:** 该代码块声明类 `BaseMultimodalProcessor`，作为模块中的可复用抽象。 它继承自 ABC。

### Lines 176-177: Declares models, gpu_image_decode
```python
    models = []
    gpu_image_decode = True  # Enable GPU decoding by default
```
**EN:** This block initializes a related set of values in the BaseMultimodalProcessor, including models, gpu_image_decode. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 BaseMultimodalProcessor 中初始化一组相关值，包括 models, gpu_image_decode。将这些赋值集中在一起有助于理解周边配置。

### Lines 178-178: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the BaseMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 BaseMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 179-254: Defines function BaseMultimodalProcessor.__init__ (part 1)
```python
    def __init__(
        self, hf_config, server_args, _processor, transport_mode, *args, **kwargs
    ):
        self.hf_config = hf_config
        self._processor = _processor
        self.server_args = server_args
        self.transport_mode = transport_mode

        mm_process_config = self.server_args.mm_process_config
        self.image_config = mm_process_config.get("image", {})
        self.video_config = mm_process_config.get("video", {})
        self.audio_config = mm_process_config.get("audio", {})

        # Resolve tokenizer: some processors (e.g. InternVL) pass a tokenizer
        # directly as _processor rather than a processor that wraps a tokenizer.
        if hasattr(self._processor, "tokenizer"):
            self._tokenizer = self._processor.tokenizer
        else:
            self._tokenizer = self._processor

        # FIXME: not accurate, model and image specific
        self.NUM_TOKEN_PER_FRAME = 330

        self.io_executor = concurrent.futures.ThreadPoolExecutor(
            max_workers=int(os.environ.get("SGLANG_IO_WORKERS", 4))
        )
        self.cpu_executor = concurrent.futures.ProcessPoolExecutor(
            mp_context=mp.get_context("fork"),
            max_workers=int(os.environ.get("SGLANG_CPU_WORKERS", os.cpu_count())),
        )

        # Mapping from attribute names to modality types
        self.ATTR_NAME_TO_MODALITY = {
            # Image-related attributes
            "pixel_values": Modality.IMAGE,
            "image_sizes": Modality.IMAGE,
            "image_grid_thw": Modality.IMAGE,
            "image_attention_mask": Modality.IMAGE,
            "image_emb_mask": Modality.IMAGE,
            "images_spatial_crop": Modality.IMAGE,
            "images_crop": Modality.IMAGE,
            "has_local_crops": Modality.IMAGE,
            "has_images": Modality.IMAGE,
            "tgt_size": Modality.IMAGE,
            "image_grid_hws": Modality.IMAGE,
            "aspect_ratio_ids": Modality.IMAGE,
            "aspect_ratio_mask": Modality.IMAGE,
            "num_patches": Modality.IMAGE,
            "patch_pixel_values": Modality.IMAGE,
            "block_sizes": Modality.IMAGE,
            "grid_thws": Modality.IMAGE,  # for kimi k2.5
            # Audio-related attributes
            "audio_features": Modality.AUDIO,
            "audio_feature_lens": Modality.AUDIO,
            "input_features": Modality.AUDIO,
            "input_features_mask": Modality.AUDIO,
            "audio_attention_mask": Modality.AUDIO,
            "feature_attention_mask": Modality.AUDIO,
            # Video-related attributes
            "pixel_values_videos": Modality.VIDEO,
            "second_per_grid_ts": Modality.VIDEO,
            "video_grid_thw": Modality.VIDEO,
            # Generic attributes that could apply to multiple modalities
            # "precomputed_embeddings" - handled specially as it can be any modality
        }

        # name of the feature filed
        # TODO: pass from processors
        self.FEATURE_NAMES = [
            "pixel_values",
            "pixel_values_videos",
            "audio_features",
            "input_features",
        ]

        skip_mm_pool = kwargs.get("skip_mm_pool", False)
```
**EN:** This block defines function `BaseMultimodalProcessor.__init__`. Parameters: self, hf_config, server_args, _processor, transport_mode. This subsection covers lines 179-254 of the same logical block.
**CN:** 该代码块定义函数 `BaseMultimodalProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor、transport_mode。 本小节覆盖同一逻辑块中的第 179-254 行。

### Lines 255-255: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 256-275: Defines function BaseMultimodalProcessor.__init__ (part 2)
```python
        if SGL_USE_CUDA_IPC and not skip_mm_pool:
            # SGLANG_MM_FEATURE_CACHE_MB is the total pool budget across all
            # tokenizer workers. Each worker gets an equal share so that adding
            # workers doesn't multiply the GPU-side footprint.
            worker_num = self.server_args.tokenizer_worker_num
            per_worker_pool_size = max(
                MM_FEATURE_CACHE_SIZE // worker_num,
                128 * 1024 * 1024,
            )
            logger.info(
                "MmItemMemoryPool size per tokenizer worker: %.0f MiB "
                "(budget %.0f MiB / %d worker(s))",
                per_worker_pool_size / (1024 * 1024),
                MM_FEATURE_CACHE_SIZE / (1024 * 1024),
                worker_num,
            )
            self.cudaipc_mmfeature_pool = MmItemMemoryPool(
                per_worker_pool_size,
                MM_ITEM_MEMORY_POOL_RECYCLE_INTERVAL,
            )
```
**EN:** This block defines function `BaseMultimodalProcessor.__init__`. Parameters: self, hf_config, server_args, _processor, transport_mode. This subsection covers lines 256-275 of the same logical block.
**CN:** 该代码块定义函数 `BaseMultimodalProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor、transport_mode。 本小节覆盖同一逻辑块中的第 256-275 行。

### Lines 276-276: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the BaseMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 BaseMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 277-283: Defines function BaseMultimodalProcessor.compute_mrope_positions
```python
    def compute_mrope_positions(self, input_ids, mm_items):
        """Compute M-RoPE positions from expanded input_ids and multimodal items.

        Returns (mrope_positions, mrope_position_delta) or (None, None) if the
        model does not use M-RoPE.
        """
        return None, None
```
**EN:** This block defines function `BaseMultimodalProcessor.compute_mrope_positions`. Parameters: self, input_ids, mm_items. Compute M-RoPE positions from expanded input_ids and multimodal items. Returns (mrope_positions, mrope_position_delta) or (None, None) if the model does not use M-RoPE.
**CN:** 该代码块定义函数 `BaseMultimodalProcessor.compute_mrope_positions`。 参数包括 self、input_ids、mm_items。 文档字符串摘要：Compute M-RoPE positions from expanded input_ids and multimodal items. Returns (mrope_positions, mrope_position_delta) or (None, None) if the model does not use M-RoPE.

### Lines 284-284: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the BaseMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 BaseMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 285-287: Defines function BaseMultimodalProcessor.spatial_merge_size
```python
    @property
    def spatial_merge_size(self):
        return self.hf_config.vision_config.spatial_merge_size
```
**EN:** This block defines function `BaseMultimodalProcessor.spatial_merge_size`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `BaseMultimodalProcessor.spatial_merge_size`。 参数包括 self。 装饰器包括 property。

### Lines 288-288: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the BaseMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 BaseMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 289-354: Defines function BaseMultimodalProcessor.build_input_ids
```python
    def build_input_ids(
        self, prompt, img_grid_thw=None, video_grid_thw=None, audio_seq_lens=None
    ):
        """
        Use prompt, img_grid_thw, video_grid_thw, and audio_seq_lens to build input_ids.
        Supports image, video, and audio tokens.
        """
        if not isinstance(prompt, list):
            prompt = self._tokenizer.encode(prompt)

        img_token_id = getattr(self, "IM_TOKEN_ID", None)
        video_token_id = getattr(self, "VIDEO_TOKEN_ID", None)
        audio_token_id = getattr(self, "audio_token_id", None)
        spatial_merge_size = getattr(self, "spatial_merge_size", 1)

        input_ids = []
        offsets = []

        cur_idx = 0

        # Use img_token_id instead of im_start_id, because a dummy im_start_id
        # may be generated by the tokenizer.
        vision_start_indices = []
        for i in range(len(prompt) - 1):
            if img_token_id is not None and prompt[i + 1] == img_token_id:
                vision_start_indices.append((i, Modality.IMAGE))
            elif video_token_id is not None and prompt[i + 1] == video_token_id:
                vision_start_indices.append((i, Modality.VIDEO))
            elif audio_token_id is not None and prompt[i + 1] == audio_token_id:
                vision_start_indices.append((i, Modality.AUDIO))
        # get modality list with order preserved
        modality_list = [modality for _, modality in vision_start_indices]

        img_idx = 0
        video_idx = 0
        audio_idx = 0
        for mm_start_idx, modality in vision_start_indices:
            if modality == Modality.IMAGE:
                mm_token_num = img_grid_thw[img_idx].prod() // (spatial_merge_size**2)
                mm_token_id = img_token_id
                img_idx += 1
            elif modality == Modality.VIDEO:
                mm_token_num = video_grid_thw[video_idx].prod() // (
                    spatial_merge_size**2
                )
                mm_token_id = video_token_id
                video_idx += 1
            elif modality == Modality.AUDIO:
                mm_token_num = int(audio_seq_lens[audio_idx].item())
                mm_token_id = audio_token_id
                audio_idx += 1
            else:
                raise ValueError(f"Invalid modality: {modality}")
            assert cur_idx <= mm_start_idx

            input_ids.extend(prompt[cur_idx : mm_start_idx + 1])
            mm_offset_start = len(input_ids)
            input_ids.extend([mm_token_id] * mm_token_num)
            cur_idx = (
                mm_start_idx + 2
            )  # jump to img_end_id, video_end_id, or audio_end_id
            offsets.append((mm_offset_start, len(input_ids) - 1))
        else:
            input_ids.extend(prompt[cur_idx:])

        return input_ids, offsets, modality_list
```
**EN:** This block defines function `BaseMultimodalProcessor.build_input_ids`. Parameters: self, prompt, img_grid_thw, video_grid_thw, audio_seq_lens. Use prompt, img_grid_thw, video_grid_thw, and audio_seq_lens to build input_ids. Supports image, video, and audio tokens.
**CN:** 该代码块定义函数 `BaseMultimodalProcessor.build_input_ids`。 参数包括 self、prompt、img_grid_thw、video_grid_thw、audio_seq_lens。 文档字符串摘要：Use prompt, img_grid_thw, video_grid_thw, and audio_seq_lens to build input_ids. Supports image, video, and audio tokens.

### Lines 355-355: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the BaseMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 BaseMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 356-394: Defines function BaseMultimodalProcessor.get_mm_data
```python
    def get_mm_data(self, prompt, embeddings, **kwargs):
        img_grid_thw = kwargs.get("img_grid_thw", None)
        video_grid_thw = kwargs.get("video_grid_thw", None)
        audio_feature_lens = kwargs.get("audio_feature_lens", None)

        input_ids, offsets, modality_list = self.build_input_ids(
            prompt,
            img_grid_thw=img_grid_thw,
            video_grid_thw=video_grid_thw,
            audio_seq_lens=audio_feature_lens,
        )
        assert all(isinstance(modality, Modality) for modality in modality_list)

        mm_items = []
        consumed_per_modality = {}

        for modality, offset in zip(modality_list, offsets):
            num_tokens = offset[1] - offset[0] + 1
            embedding_start = consumed_per_modality.get(modality, 0)
            embedding_slice = embeddings[modality][
                embedding_start : embedding_start + num_tokens
            ]
            consumed_per_modality[modality] = embedding_start + num_tokens
            mm_items.append(
                MultimodalDataItem(
                    modality=modality,
                    offsets=[offset],
                    precomputed_embeddings=embedding_slice,
                )
            )

        return MultimodalProcessorOutput(
            input_ids=input_ids,
            mm_items=mm_items,
            im_start_id=self.IM_START_TOKEN_ID,
            im_end_id=self.IM_END_TOKEN_ID,
            im_token_id=self.IM_TOKEN_ID,
            video_token_id=getattr(self, "VIDEO_TOKEN_ID", None),
        )
```
**EN:** This block defines function `BaseMultimodalProcessor.get_mm_data`. Parameters: self, prompt, embeddings.
**CN:** 该代码块定义函数 `BaseMultimodalProcessor.get_mm_data`。 参数包括 self、prompt、embeddings。

### Lines 395-395: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the BaseMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 BaseMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 396-468: Defines function BaseMultimodalProcessor.process_mm_data
```python
    def process_mm_data(
        self, input_text, images=None, videos=None, audios=None, **kwargs
    ) -> dict:
        """
        process multimodal data with transformers AutoProcessor
        """
        if images:
            kwargs["images"] = images
            if self.image_config:
                kwargs.setdefault("images_kwargs", {}).update(self.image_config)
        if videos:
            kwargs["videos"] = videos
            if self.video_config:
                kwargs.setdefault("videos_kwargs", {}).update(self.video_config)
        if audios:
            if self._processor.__class__.__name__ in {
                "Gemma3nProcessor",
                "Gemma4Processor",
                "GlmAsrProcessor",
                "Qwen2AudioProcessor",
                "Qwen3ASRProcessor",
                "Qwen3OmniMoeProcessor",
            }:
                # Note(Xinyuan): for gemma3n, ref: https://github.com/huggingface/transformers/blob/ccf2ca162e33f381e454cdb74bf4b41a51ab976d/src/transformers/models/gemma3n/processing_gemma3n.py#L107
                kwargs["audio"] = audios
                kwargs.setdefault("audio_kwargs", {})
                kwargs["audio_kwargs"].setdefault("truncation", False)
            else:
                kwargs["audios"] = audios
            if self.audio_config:
                kwargs.setdefault("audio_kwargs", {}).update(self.audio_config)

        processor = self._processor
        if (
            hasattr(processor, "image_processor")
            and isinstance(processor.image_processor, BaseImageProcessor)
            and not self.server_args.disable_fast_image_processor
        ):
            if _is_cpu or get_global_server_args().rl_on_policy_target is not None:
                kwargs["device"] = "cpu"
            elif _is_xpu:
                kwargs["device"] = "xpu"
            elif not _is_npu:
                kwargs["device"] = "cuda"
            elif processor.__class__.__name__ not in {
                "Glm4vProcessor",
            }:
                # Note: for qwen-vl, processor has some reshape issue because of dims restriction on Ascend.
                from sglang.srt.hardware_backend.npu.modules.qwen_vl_processor import (
                    npu_apply_qwen_image_preprocess_patch,
                )

                npu_apply_qwen_image_preprocess_patch()
                kwargs["device"] = "npu"

        result = processor.__call__(
            text=[input_text],
            padding=True,
            return_tensors="pt",
            **kwargs,
        )
        if not self.server_args.keep_mm_feature_on_device:
            # move feature tensors to cpu
            for feature_name in self.FEATURE_NAMES:
                if SGL_USE_CUDA_IPC:
                    pass
                else:
                    if feature_name in result and isinstance(
                        result[feature_name], torch.Tensor
                    ):
                        result[feature_name] = result[feature_name].to("cpu")

        return result
```
**EN:** This block defines function `BaseMultimodalProcessor.process_mm_data`. Parameters: self, input_text, images, videos, audios. process multimodal data with transformers AutoProcessor
**CN:** 该代码块定义函数 `BaseMultimodalProcessor.process_mm_data`。 参数包括 self、input_text、images、videos、audios。 文档字符串摘要：process multimodal data with transformers AutoProcessor

### Lines 469-469: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the BaseMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 BaseMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 470-479: Defines async function BaseMultimodalProcessor.process_mm_data_async
```python
    @abstractmethod
    async def process_mm_data_async(
        self,
        image_data,
        audio_data,
        input_text,
        request_obj,
        **kwargs,
    ) -> Optional[Dict[str, Any]]:
        pass
```
**EN:** This block defines async function `BaseMultimodalProcessor.process_mm_data_async`. Parameters: self, image_data, audio_data, input_text, request_obj. Decorators: abstractmethod.
**CN:** 该代码块定义异步函数 `BaseMultimodalProcessor.process_mm_data_async`。 参数包括 self、image_data、audio_data、input_text、request_obj。 装饰器包括 abstractmethod。

### Lines 480-480: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the BaseMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 BaseMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 481-501: Defines function BaseMultimodalProcessor.get_estimated_frames_list
```python
    def get_estimated_frames_list(self, image_data):
        """
        estimate the total frame count from all visual input
        """
        from sglang.srt.utils.video_decoder import VideoDecoderWrapper

        # Before processing inputs
        if not image_data or len(image_data) == 0:
            return []
        estimated_frames_list = []
        for image in image_data:
            if isinstance(image, str) and image.startswith("video:"):
                path = image[len("video:") :]
                decoder = VideoDecoderWrapper(path)
                num_frames = len(decoder)
            else:
                # For images, each contributes one frame
                num_frames = 1
            estimated_frames_list.append(num_frames)

        return estimated_frames_list
```
**EN:** This block defines function `BaseMultimodalProcessor.get_estimated_frames_list`. Parameters: self, image_data. estimate the total frame count from all visual input
**CN:** 该代码块定义函数 `BaseMultimodalProcessor.get_estimated_frames_list`。 参数包括 self、image_data。 文档字符串摘要：estimate the total frame count from all visual input

### Lines 502-502: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the BaseMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 BaseMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 503-545: Defines function BaseMultimodalProcessor._load_single_item
```python
    @classmethod
    def _load_single_item(
        cls,
        data,
        modality: Modality,
        frame_count_limit=None,
        audio_sample_rate: Optional[int] = None,
        discard_alpha_channel=True,
    ):
        """
        Load a single multimodal data.

        If data is processor_output or precomputed embedding, return directly.

        Class method that can be pickled for multiprocessing
        """
        if isinstance(data, dict):
            data_format = data.get("format")
            if data_format in (
                MultimodalInputFormat.PROCESSOR_OUTPUT.name,
                MultimodalInputFormat.PRECOMPUTED_EMBEDDING.name,
                "processor_output",
                "precomputed_embedding",
            ):
                return data
        try:
            if modality == Modality.IMAGE:
                img, _ = load_image(data, cls.gpu_image_decode)
                if (
                    discard_alpha_channel
                    and not isinstance(img, torch.Tensor)
                    and img.mode != "RGB"
                ):
                    # Needed only when `img` is a PIL image
                    img = img.convert("RGB")
                return img
            elif modality == Modality.VIDEO:
                return load_video(data, frame_count_limit)
            elif modality == Modality.AUDIO:
                return load_audio(data, audio_sample_rate)

        except Exception as e:
            raise RuntimeError(f"Error while loading data {data}: {e}")
```
**EN:** This block defines function `BaseMultimodalProcessor._load_single_item`. Parameters: cls, data, modality, frame_count_limit, audio_sample_rate, discard_alpha_channel. Decorators: classmethod. Load a single multimodal data. If data is processor_output or precomputed embedding, return directly.
**CN:** 该代码块定义函数 `BaseMultimodalProcessor._load_single_item`。 参数包括 cls、data、modality、frame_count_limit、audio_sample_rate、discard_alpha_channel。 装饰器包括 classmethod。 文档字符串摘要：Load a single multimodal data. If data is processor_output or precomputed embedding, return directly.

### Lines 546-546: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the BaseMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 BaseMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 547-586: Defines function BaseMultimodalProcessor._submit_mm_data_loading_tasks_simple
```python
    def _submit_mm_data_loading_tasks_simple(
        self,
        data_list: Optional[list],
        modality: Modality,
        audio_sample_rate: Optional[int],
        discard_alpha_channel: bool,
    ) -> List[Tuple[Modality, int, concurrent.futures.Future]]:
        """
        Simple version: For one modal data submit IO load task.
        Return:
            List[(modality, index_in_that_modality, future)]
        """
        futures: List[Tuple[Modality, int, concurrent.futures.Future]] = []

        if not data_list:
            logger.debug(
                "[_submit_mm_data_loading_tasks_simple] no data for modality=%s",
                modality.name,
            )
            return futures

        for idx, data in enumerate(data_list):
            logger.debug(
                "[_submit_mm_data_loading_tasks_simple] submit load task: "
                "modality=%s, index=%d, data_type=%s",
                modality.name,
                idx,
                type(data),
            )
            future = self.io_executor.submit(
                self.__class__._load_single_item,
                data,
                modality,
                None,  # frame_count_limit: no consider for fast path
                audio_sample_rate,
                discard_alpha_channel,
            )
            futures.append((modality, idx, future))

        return futures
```
**EN:** This block defines function `BaseMultimodalProcessor._submit_mm_data_loading_tasks_simple`. Parameters: self, data_list, modality, audio_sample_rate, discard_alpha_channel. Simple version: For one modal data submit IO load task. Return: List[(modality, index_in_that_modality, future)]
**CN:** 该代码块定义函数 `BaseMultimodalProcessor._submit_mm_data_loading_tasks_simple`。 参数包括 self、data_list、modality、audio_sample_rate、discard_alpha_channel。 文档字符串摘要：Simple version: For one modal data submit IO load task. Return: List[(modality, index_in_that_modality, future)]

### Lines 587-587: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the BaseMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 BaseMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 588-658: Defines function BaseMultimodalProcessor.submit_data_loading_tasks
```python
    def submit_data_loading_tasks(
        self,
        text_parts: List[str],
        multimodal_tokens: MultimodalSpecialTokens,
        data_iterators: dict[Modality, Iterator[Any]],
        discard_alpha_channel: bool = True,
        image_estimated_frames_iter: Optional[iter] = None,
        image_scaling_factor: float = 1.0,
        max_image_frames: int = 30,
        audio_sample_rate: Optional[int] = None,
    ) -> Tuple[List, List]:
        """
        load multimodal data parallelly using iterators.
        """
        futures = []
        task_info = []

        for text_part in text_parts:
            modality = multimodal_tokens.get_modality_of_token(text_part)
            if modality is not None:
                data_iterator = data_iterators.get(modality)
                if data_iterator is None:
                    raise ValueError(f"No data iterator found for token: {text_part}")

                try:
                    data = next(data_iterator)
                except StopIteration:
                    logger.warning(
                        f"Mismatch: More '{modality.name}' tokens found than corresponding data provided."
                    )
                    return futures, task_info

                frame_count_limit = None
                if modality == Modality.IMAGE and image_estimated_frames_iter:
                    try:
                        estimated_frames = next(image_estimated_frames_iter)
                        # Use the pre-calculated scaling factor and max frames
                        frame_count_limit = max(
                            1, int(estimated_frames * image_scaling_factor)
                        )
                        # Ensure we don't exceed the absolute max (redundant if scaling_factor handles it)
                        # frame_count_limit = min(frame_count_limit, max_image_frames)
                    except StopIteration:
                        raise ValueError(
                            "Mismatch between image tokens and estimated frame counts."
                        )

                futures.append(
                    self.io_executor.submit(
                        self.__class__._load_single_item,
                        data,
                        modality,
                        frame_count_limit,
                        audio_sample_rate,
                        discard_alpha_channel,
                    )
                )
                task_info.append((modality, data, frame_count_limit))

        for modality, iterator in data_iterators.items():
            try:
                next(iterator)
                logger.warning(
                    f"Warning: More {modality.name.lower()} data items provided than corresponding tokens found in the prompt."
                )
            except StopIteration:
                pass
            except Exception:
                pass

        return futures, task_info
```
**EN:** This block defines function `BaseMultimodalProcessor.submit_data_loading_tasks`. Parameters: self, text_parts, multimodal_tokens, data_iterators, discard_alpha_channel, image_estimated_frames_iter, image_scaling_factor, max_image_frames, audio_sample_rate. load multimodal data parallelly using iterators.
**CN:** 该代码块定义函数 `BaseMultimodalProcessor.submit_data_loading_tasks`。 参数包括 self、text_parts、multimodal_tokens、data_iterators、discard_alpha_channel、image_estimated_frames_iter、image_scaling_factor、max_image_frames、audio_sample_rate。 文档字符串摘要：load multimodal data parallelly using iterators.

### Lines 659-659: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the BaseMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 BaseMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 660-682: Defines function BaseMultimodalProcessor._validate_one_modality
```python
    @staticmethod
    def _validate_one_modality(modality: Modality, data_list: Optional[list]):
        if data_list is None:
            return
        if not isinstance(data_list, list):
            raise TypeError(
                f"{modality.name} must be a list or None, got {type(data_list)}"
            )

        formatted_indices = []
        for idx, item in enumerate(data_list):
            if isinstance(item, dict):
                fmt = item.get("format")
                if fmt in {"processor_output", "precomputed_embedding"}:
                    formatted_indices.append(idx)

        if formatted_indices:
            if len(data_list) != 1:
                raise ValueError(
                    f"For {modality}, when providing a 'processor_output' or "
                    f"'precomputed_embedding', you must pass exactly one item; "
                    f"received {len(data_list)} items (formatted at indices {formatted_indices})."
                )
```
**EN:** This block defines function `BaseMultimodalProcessor._validate_one_modality`. Parameters: modality, data_list. Decorators: staticmethod.
**CN:** 该代码块定义函数 `BaseMultimodalProcessor._validate_one_modality`。 参数包括 modality、data_list。 装饰器包括 staticmethod。

### Lines 683-683: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the BaseMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 BaseMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 684-704: Defines function BaseMultimodalProcessor.validate_mm_data
```python
    @staticmethod
    def validate_mm_data(
        image_data: Optional[list] = None,
        video_data: Optional[list] = None,
        audio_data: Optional[list] = None,
    ):
        """
        Validate multimodal input lists per modality.

        Rule per modality (image/video/audio):
        - Either the list has exactly one item and that single item is a dict with
          format in {"processor_output", "precomputed_embedding"};
        - Or, the list contains only "normal" items (i.e., does not include any
          item whose format is one of the two above).

        Empty or None lists are considered valid.
        """

        BaseMultimodalProcessor._validate_one_modality(Modality.IMAGE, image_data)
        BaseMultimodalProcessor._validate_one_modality(Modality.VIDEO, video_data)
        BaseMultimodalProcessor._validate_one_modality(Modality.AUDIO, audio_data)
```
**EN:** This block defines function `BaseMultimodalProcessor.validate_mm_data`. Parameters: image_data, video_data, audio_data. Decorators: staticmethod. Validate multimodal input lists per modality. Rule per modality (image/video/audio): - Either the list has exactly one item and that single item is a dict with format in {"processor_output", "precomputed_embedding"}; - Or, the list contains only "normal" items
**CN:** 该代码块定义函数 `BaseMultimodalProcessor.validate_mm_data`。 参数包括 image_data、video_data、audio_data。 装饰器包括 staticmethod。 文档字符串摘要：Validate multimodal input lists per modality. Rule per modality (image/video/audio): - Either the list has exactly one item and that single item is a dict with format in {"processor_output", "precomputed_embedding"}; - Or, the list contains only "normal" items

### Lines 705-705: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the BaseMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 BaseMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 706-729: Defines function BaseMultimodalProcessor._process_loaded_mm_data
```python
    def _process_loaded_mm_data(self, modality, raw_data, result):
        images, videos, audios = [], [], []

        is_precomputed = isinstance(raw_data, dict) and raw_data.get("format") in [
            MultimodalInputFormat.PROCESSOR_OUTPUT.name,
            MultimodalInputFormat.PRECOMPUTED_EMBEDDING.name,
            "processor_output",
            "precomputed_embedding",
        ]

        if modality == Modality.IMAGE:
            if is_precomputed:
                images.append(result)
            else:
                if isinstance(result, list):
                    images.extend(result)
                else:
                    images.append(result)
        elif modality == Modality.VIDEO:
            videos.append(result)
        elif modality == Modality.AUDIO:
            audios.append(result)

        return is_precomputed, images, videos, audios
```
**EN:** This block defines function `BaseMultimodalProcessor._process_loaded_mm_data`. Parameters: self, modality, raw_data, result.
**CN:** 该代码块定义函数 `BaseMultimodalProcessor._process_loaded_mm_data`。 参数包括 self、modality、raw_data、result。

### Lines 730-730: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the BaseMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 BaseMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 731-795: Defines function BaseMultimodalProcessor.load_mm_data
```python
    def load_mm_data(
        self,
        prompt: str,
        multimodal_tokens: MultimodalSpecialTokens,
        image_data: Optional[list] = None,
        video_data: Optional[list] = None,
        audio_data: Optional[list] = None,
        return_text: Optional[bool] = True,
        discard_alpha_channel: bool = True,
        audio_sample_rate: Optional[int] = None,
    ) -> BaseMultiModalProcessorOutput:

        BaseMultimodalProcessor.validate_mm_data(image_data, video_data, audio_data)

        multimodal_tokens_pattern = multimodal_tokens.get_combined_regex()
        if isinstance(prompt, list) and return_text:
            assert len(prompt) and isinstance(prompt[0], int)
            prompt = self._tokenizer.decode(prompt)
        else:
            prompt = prompt

        assert isinstance(prompt, str)
        # split text into list of normal text and special tokens
        text_parts = re.split(multimodal_tokens_pattern, prompt)

        cnt = {Modality.IMAGE: 0, Modality.VIDEO: 0, Modality.AUDIO: 0}
        for text_part in text_parts:
            modality = multimodal_tokens.get_modality_of_token(text_part)
            if modality is not None:
                cnt[modality] += 1

        n_image = len(image_data) if image_data else 0
        n_video = len(video_data) if video_data else 0
        n_audio = len(audio_data) if audio_data else 0

        # For MiniCPMO and MiniCPMV or multimodal_tokens not totally align, legacy show path
        if (
            self.server_args.skip_tokenizer_init
            or cnt[Modality.IMAGE] != n_image
            or cnt[Modality.VIDEO] != n_video
            or cnt[Modality.AUDIO] != n_audio
            or getattr(self, "support_dynamic_frame_expansion", False)
        ):
            return self.legacy_load_mm_data(
                prompt=prompt,
                multimodal_tokens=multimodal_tokens,
                image_data=image_data,
                video_data=video_data,
                audio_data=audio_data,
                return_text=return_text,
                discard_alpha_channel=discard_alpha_channel,
                audio_sample_rate=audio_sample_rate,
            )
        # For models other than MiniCPMO and MiniCPMV,
        # totally align multimodal_tokens, fast path
        return self.fast_load_mm_data(
            prompt=prompt,
            multimodal_tokens=multimodal_tokens,
            image_data=image_data,
            video_data=video_data,
            audio_data=audio_data,
            return_text=return_text,
            discard_alpha_channel=discard_alpha_channel,
            audio_sample_rate=audio_sample_rate,
        )
```
**EN:** This block defines function `BaseMultimodalProcessor.load_mm_data`. Parameters: self, prompt, multimodal_tokens, image_data, video_data, audio_data, return_text, discard_alpha_channel, audio_sample_rate.
**CN:** 该代码块定义函数 `BaseMultimodalProcessor.load_mm_data`。 参数包括 self、prompt、multimodal_tokens、image_data、video_data、audio_data、return_text、discard_alpha_channel、audio_sample_rate。

### Lines 796-796: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the BaseMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 BaseMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 797-872: Defines function BaseMultimodalProcessor.fast_load_mm_data (part 1)
```python
    def fast_load_mm_data(
        self,
        prompt: str,
        multimodal_tokens: MultimodalSpecialTokens,
        image_data: Optional[list] = None,
        video_data: Optional[list] = None,
        audio_data: Optional[list] = None,
        return_text: Optional[bool] = True,
        discard_alpha_channel: bool = True,
        audio_sample_rate: Optional[int] = None,
    ) -> BaseMultiModalProcessorOutput:
        """
        A fast version of `load_mm_data` that loads multimodal data directly.
        This version does not scan the prompt to recognize tokens. It assumes
        that the caller has already aligned the tokens and data in a 1:1 manner.
        The behavior is as follows:
          1. It runs `_load_single_item` for all input data concurrently.
          2. It returns the loaded images, videos, and audios in their original order.
          3. It returns the input prompt as a string.
        """

        # Convert prompt into str
        if isinstance(prompt, list) and return_text:
            assert len(prompt) and isinstance(prompt[0], int)
            prompt_str = self._tokenizer.decode(prompt)
        else:
            assert isinstance(prompt, str)
            prompt_str = prompt

        futures: List[Tuple[Modality, int, concurrent.futures.Future]] = []

        modalities_data = [
            (image_data, Modality.IMAGE),
            (video_data, Modality.VIDEO),
            (audio_data, Modality.AUDIO),
        ]

        for data_list, modality in modalities_data:
            futures.extend(
                self._submit_mm_data_loading_tasks_simple(
                    data_list, modality, audio_sample_rate, discard_alpha_channel
                )
            )

        logger.debug("[load_mm_data(simple)] total futures submitted: %d", len(futures))

        images: List[Any] = [None] * len(image_data) if image_data else []
        videos: List[Any] = [None] * len(video_data) if video_data else []
        audios: List[Any] = [None] * len(audio_data) if audio_data else []

        for modality, idx, future in futures:
            try:
                result = future.result()
            except Exception as e:
                logger.exception(
                    "[load_mm_data(simple)] error loading %s data at index=%d",
                    modality.name,
                    idx,
                )
                raise RuntimeError(
                    f"An exception occurred while loading {modality.name} data at index {idx}: {e}"
                )

            if modality == Modality.IMAGE:
                images[idx] = result
            elif modality == Modality.VIDEO:
                videos[idx] = result
            elif modality == Modality.AUDIO:
                audios[idx] = result

        logger.debug(
            "[load_mm_data(simple)] loaded counts: images=%d, videos=%d, audios=%d",
            len(images),
            len(videos),
            len(audios),
        )
```
**EN:** This block defines function `BaseMultimodalProcessor.fast_load_mm_data`. Parameters: self, prompt, multimodal_tokens, image_data, video_data, audio_data, return_text, discard_alpha_channel, audio_sample_rate. A fast version of `load_mm_data` that loads multimodal data directly. This version does not scan the prompt to recognize tokens. This subsection covers lines 797-872 of the same logical block.
**CN:** 该代码块定义函数 `BaseMultimodalProcessor.fast_load_mm_data`。 参数包括 self、prompt、multimodal_tokens、image_data、video_data、audio_data、return_text、discard_alpha_channel、audio_sample_rate。 文档字符串摘要：A fast version of `load_mm_data` that loads multimodal data directly. This version does not scan the prompt to recognize tokens. 本小节覆盖同一逻辑块中的第 797-872 行。

### Lines 873-873: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 874-879: Defines function BaseMultimodalProcessor.fast_load_mm_data (part 2)
```python
        return BaseMultiModalProcessorOutput(
            images=images,
            audios=audios,
            videos=videos,
            input_text=prompt_str,
        )
```
**EN:** This block defines function `BaseMultimodalProcessor.fast_load_mm_data`. Parameters: self, prompt, multimodal_tokens, image_data, video_data, audio_data, return_text, discard_alpha_channel, audio_sample_rate. A fast version of `load_mm_data` that loads multimodal data directly. This version does not scan the prompt to recognize tokens. This subsection covers lines 874-879 of the same logical block.
**CN:** 该代码块定义函数 `BaseMultimodalProcessor.fast_load_mm_data`。 参数包括 self、prompt、multimodal_tokens、image_data、video_data、audio_data、return_text、discard_alpha_channel、audio_sample_rate。 文档字符串摘要：A fast version of `load_mm_data` that loads multimodal data directly. This version does not scan the prompt to recognize tokens. 本小节覆盖同一逻辑块中的第 874-879 行。

### Lines 880-880: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the BaseMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 BaseMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 881-950: Defines function BaseMultimodalProcessor.legacy_load_mm_data (part 1)
```python
    def legacy_load_mm_data(
        self,
        prompt: str,
        multimodal_tokens: MultimodalSpecialTokens,
        image_data: Optional[list] = None,
        video_data: Optional[list] = None,
        audio_data: Optional[list] = None,
        return_text: Optional[bool] = True,
        discard_alpha_channel: bool = True,
        audio_sample_rate: Optional[int] = None,
    ) -> BaseMultiModalProcessorOutput:
        """
        Each frame of video/image will be replaced by a single image token

        Args:
            multimodal_tokens (list[str]): list of special token which denoting a single multimodal data
                e.g. image token or audio token
            discard_alpha_channel: if True, discards the alpha channel in the returned images

        """

        multimodal_tokens_pattern = multimodal_tokens.get_combined_regex()
        if isinstance(prompt, list) and return_text:
            assert len(prompt) and isinstance(prompt[0], int)
            prompt = self._tokenizer.decode(prompt)
        else:
            prompt = prompt

        assert isinstance(prompt, str)
        # split text into list of normal text and special tokens
        text_parts = re.split(multimodal_tokens_pattern, prompt)
        # collect all data
        data_iterators = {}
        if multimodal_tokens.image_token and image_data:
            data_iterators[Modality.IMAGE] = iter(image_data)
        if multimodal_tokens.video_token and video_data:
            data_iterators[Modality.VIDEO] = iter(video_data)
        if multimodal_tokens.audio_token and audio_data:
            data_iterators[Modality.AUDIO] = iter(audio_data)

        # futures: the futures of loaded data
        # task_info: modality, raw_data, and other metadata of each data
        futures, task_info = self.submit_data_loading_tasks(
            text_parts=text_parts,
            multimodal_tokens=multimodal_tokens,
            data_iterators=data_iterators,
            discard_alpha_channel=discard_alpha_channel,
            audio_sample_rate=audio_sample_rate,
        )
        task_info_iter = iter(task_info)
        futures_iter = iter(futures)

        # Process results
        images, videos, audios = [], [], []
        new_text_parts = []
        has_precomputed_input = False
        for text_part in text_parts:
            try:
                if multimodal_tokens_pattern.match(text_part):
                    modality, raw_data, frame_limit = next(task_info_iter)
                    result = next(futures_iter).result()

                    is_precomputed, new_imgs, new_vids, new_auds = (
                        self._process_loaded_mm_data(modality, raw_data, result)
                    )

                    has_precomputed_input |= is_precomputed
                    images.extend(new_imgs)
                    videos.extend(new_vids)
                    audios.extend(new_auds)
```
**EN:** This block defines function `BaseMultimodalProcessor.legacy_load_mm_data`. Parameters: self, prompt, multimodal_tokens, image_data, video_data, audio_data, return_text, discard_alpha_channel, audio_sample_rate. Each frame of video/image will be replaced by a single image token Args: multimodal_tokens (list[str]): list of special token which denoting a single multimodal data e.g. image token or audio token discard_alpha_channel: if True, discards the alpha channel in This subsection covers lines 881-950 of the same logical block.
**CN:** 该代码块定义函数 `BaseMultimodalProcessor.legacy_load_mm_data`。 参数包括 self、prompt、multimodal_tokens、image_data、video_data、audio_data、return_text、discard_alpha_channel、audio_sample_rate。 文档字符串摘要：Each frame of video/image will be replaced by a single image token Args: multimodal_tokens (list[str]): list of special token which denoting a single multimodal data e.g. image token or audio token discard_alpha_channel: if True, discards the alpha channel in 本小节覆盖同一逻辑块中的第 881-950 行。

### Lines 951-951: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 952-979: Defines function BaseMultimodalProcessor.legacy_load_mm_data (part 2)
```python
                    if modality == Modality.IMAGE:
                        if is_precomputed:
                            new_text_parts += [text_part]
                        else:
                            count = len(new_imgs)
                            if count > 0:
                                new_text_parts += [
                                    multimodal_tokens.image_token
                                ] * count
                    elif modality == Modality.VIDEO:
                        # load as video
                        mm_tokens = (
                            text_part
                            if is_precomputed
                            else multimodal_tokens.video_token
                        )
                        new_text_parts += mm_tokens
                    elif modality == Modality.AUDIO:
                        # audio
                        mm_tokens = (
                            text_part
                            if is_precomputed
                            else multimodal_tokens.audio_token
                        )
                        new_text_parts += mm_tokens
                else:
                    # normal text
                    new_text_parts += [text_part]
```
**EN:** This block defines function `BaseMultimodalProcessor.legacy_load_mm_data`. Parameters: self, prompt, multimodal_tokens, image_data, video_data, audio_data, return_text, discard_alpha_channel, audio_sample_rate. Each frame of video/image will be replaced by a single image token Args: multimodal_tokens (list[str]): list of special token which denoting a single multimodal data e.g. image token or audio token discard_alpha_channel: if True, discards the alpha channel in This subsection covers lines 952-979 of the same logical block.
**CN:** 该代码块定义函数 `BaseMultimodalProcessor.legacy_load_mm_data`。 参数包括 self、prompt、multimodal_tokens、image_data、video_data、audio_data、return_text、discard_alpha_channel、audio_sample_rate。 文档字符串摘要：Each frame of video/image will be replaced by a single image token Args: multimodal_tokens (list[str]): list of special token which denoting a single multimodal data e.g. image token or audio token discard_alpha_channel: if True, discards the alpha channel in 本小节覆盖同一逻辑块中的第 952-979 行。

### Lines 980-980: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 981-998: Defines function BaseMultimodalProcessor.legacy_load_mm_data (part 3)
```python
            except StopIteration as e:
                # when precomputed_input is presented with multi-images, StopIteration is expected
                if has_precomputed_input:
                    new_text_parts += [text_part]
                    continue
                raise RuntimeError(
                    f"An exception occurred while loading multimodal data: {e}"
                )
            except Exception as e:
                raise RuntimeError(
                    f"An exception occurred while loading multimodal data: {e}"
                )
        return BaseMultiModalProcessorOutput(
            images=images,
            audios=audios,
            videos=videos,
            input_text="".join(new_text_parts),
        )
```
**EN:** This block defines function `BaseMultimodalProcessor.legacy_load_mm_data`. Parameters: self, prompt, multimodal_tokens, image_data, video_data, audio_data, return_text, discard_alpha_channel, audio_sample_rate. Each frame of video/image will be replaced by a single image token Args: multimodal_tokens (list[str]): list of special token which denoting a single multimodal data e.g. image token or audio token discard_alpha_channel: if True, discards the alpha channel in This subsection covers lines 981-998 of the same logical block.
**CN:** 该代码块定义函数 `BaseMultimodalProcessor.legacy_load_mm_data`。 参数包括 self、prompt、multimodal_tokens、image_data、video_data、audio_data、return_text、discard_alpha_channel、audio_sample_rate。 文档字符串摘要：Each frame of video/image will be replaced by a single image token Args: multimodal_tokens (list[str]): list of special token which denoting a single multimodal data e.g. image token or audio token discard_alpha_channel: if True, discards the alpha channel in 本小节覆盖同一逻辑块中的第 981-998 行。

### Lines 999-999: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the BaseMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 BaseMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 1000-1014: Defines function BaseMultimodalProcessor.get_mm_items_offset
```python
    @staticmethod
    def get_mm_items_offset(
        input_ids: torch.Tensor, mm_token_id: int
    ) -> List[Tuple[int, int]]:
        """
        Get a set of range for mm_items from input_ids
        Example:
            input_ids = [1, 2, 3, 3, 3, 4, 3, 3]
            mm_token_id = 3
            return result = [(2,4),(6,7)]
        """
        mask = input_ids == mm_token_id
        start_positions = (mask & ~torch.roll(mask, 1)).nonzero(as_tuple=True)[0]
        end_positions = (mask & ~torch.roll(mask, -1)).nonzero(as_tuple=True)[0]
        return list(zip(start_positions.tolist(), end_positions.tolist()))
```
**EN:** This block defines function `BaseMultimodalProcessor.get_mm_items_offset`. Parameters: input_ids, mm_token_id. Decorators: staticmethod. Get a set of range for mm_items from input_ids Example: input_ids = [1, 2, 3, 3, 3, 4, 3, 3] mm_token_id = 3 return result = [(2,4),(6,7)]
**CN:** 该代码块定义函数 `BaseMultimodalProcessor.get_mm_items_offset`。 参数包括 input_ids、mm_token_id。 装饰器包括 staticmethod。 文档字符串摘要：Get a set of range for mm_items from input_ids Example: input_ids = [1, 2, 3, 3, 3, 4, 3, 3] mm_token_id = 3 return result = [(2,4),(6,7)]

### Lines 1015-1015: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the BaseMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 BaseMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 1016-1023: Defines function BaseMultimodalProcessor.get_mm_items_offset_by_pair
```python
    @staticmethod
    def get_mm_items_offset_by_pair(
        input_ids: torch.Tensor, mm_start_id: int, mm_end_id: int
    ) -> List[Tuple[int, int]]:
        indices_start = (input_ids == mm_start_id).nonzero(as_tuple=True)[0] + 1
        indices_end = (input_ids == mm_end_id).nonzero(as_tuple=True)[0] - 1

        return list(zip(indices_start.tolist(), indices_end.tolist()))
```
**EN:** This block defines function `BaseMultimodalProcessor.get_mm_items_offset_by_pair`. Parameters: input_ids, mm_start_id, mm_end_id. Decorators: staticmethod.
**CN:** 该代码块定义函数 `BaseMultimodalProcessor.get_mm_items_offset_by_pair`。 参数包括 input_ids、mm_start_id、mm_end_id。 装饰器包括 staticmethod。

### Lines 1024-1024: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the BaseMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 BaseMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 1025-1064: Defines function BaseMultimodalProcessor.collect_mm_items_from_processor_output
```python
    def collect_mm_items_from_processor_output(
        self, data_dict: dict, modality: Modality = None
    ) -> List[MultimodalDataItem]:
        """
        Create mm_items from processor output. Initially creates one item per modality;
        these are later split into per-image/video items by get_new_expanded_mm_items.

        Note that the data_dict can be passed via offline engine api
        """

        items: dict[Modality, MultimodalDataItem] = {}
        for attr_name, value in data_dict.items():
            if attr_name == "input_ids":
                continue

            # Get modality for this attribute
            current_modality = modality or self.ATTR_NAME_TO_MODALITY.get(attr_name)

            if attr_name == "precomputed_embeddings":
                modality_str = data_dict.get("modality")
                current_modality = Modality.IMAGE
                if modality_str:
                    try:
                        current_modality = Modality.from_str(modality_str)
                    except ValueError:
                        pass

            if current_modality:
                # Create item if needed
                if current_modality not in items:
                    items[current_modality] = MultimodalDataItem(
                        modality=current_modality
                    )

                if attr_name in self.FEATURE_NAMES:
                    attr_name = "feature"

                items[current_modality].set(attr_name, value)

        return list(items.values())
```
**EN:** This block defines function `BaseMultimodalProcessor.collect_mm_items_from_processor_output`. Parameters: self, data_dict, modality. Create mm_items from processor output. Initially creates one item per modality; these are later split into per-image/video items by get_new_expanded_mm_items.
**CN:** 该代码块定义函数 `BaseMultimodalProcessor.collect_mm_items_from_processor_output`。 参数包括 self、data_dict、modality。 文档字符串摘要：Create mm_items from processor output. Initially creates one item per modality; these are later split into per-image/video items by get_new_expanded_mm_items.

### Lines 1065-1065: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the BaseMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 BaseMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 1066-1082: Defines function BaseMultimodalProcessor._process_and_collect_mm_items
```python
    def _process_and_collect_mm_items(
        self, input_text: str, images=None, audios=None, videos=None, **kwargs
    ) -> Tuple[List[MultimodalDataItem], torch.Tensor, dict]:
        """
        Helper method to process multimodal data and create mm_items in one step.

        Returns:
            Tuple of (created mm_items, input_ids)
        """
        ret = self.process_mm_data(
            input_text=input_text, images=images, audios=audios, videos=videos, **kwargs
        )

        input_ids = ret["input_ids"].flatten()
        collected_items = self.collect_mm_items_from_processor_output(ret)

        return collected_items, input_ids, ret
```
**EN:** This block defines function `BaseMultimodalProcessor._process_and_collect_mm_items`. Parameters: self, input_text, images, audios, videos. Helper method to process multimodal data and create mm_items in one step. Returns: Tuple of (created mm_items, input_ids)
**CN:** 该代码块定义函数 `BaseMultimodalProcessor._process_and_collect_mm_items`。 参数包括 self、input_text、images、audios、videos。 文档字符串摘要：Helper method to process multimodal data and create mm_items in one step. Returns: Tuple of (created mm_items, input_ids)

### Lines 1083-1083: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the BaseMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 BaseMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 1084-1161: Defines function BaseMultimodalProcessor.process_and_combine_mm_data (part 1)
```python
    def process_and_combine_mm_data(
        self,
        base_output: BaseMultiModalProcessorOutput,
        mm_tokens: MultimodalSpecialTokens,
        **kwargs,
    ) -> Tuple[List[MultimodalDataItem], torch.Tensor, dict]:
        """
        Process multimodal data and return the combined multimodal items and input_ids.
        Supports mixed modalities (images and audio in the same request).

        Returns:
            Tuple of (list of mm_items, input_ids)
        """
        # Collect all items and categorize them
        all_loaded_data = base_output.organize_results()
        # Handle text-only case
        if not all_loaded_data:
            input_ids = self._tokenizer(
                base_output.input_text,
                return_tensors="pt",
                add_special_tokens=True,
            ).input_ids.flatten()
            return [], input_ids, {}

        dict_items, raw_images, raw_audios, raw_videos = [], [], [], []
        for modality, item in all_loaded_data:
            if isinstance(item, dict):
                dict_items.append((modality, item))
            elif modality == Modality.IMAGE:
                raw_images.append(item)
            elif modality == Modality.AUDIO:
                raw_audios.append(item)
            elif modality == Modality.VIDEO:
                raw_videos.append(item)
            else:
                raise ValueError(f"Unknown multimodal item type: {type(item)}")
        # Process items and get input_ids
        all_collected_items: list[MultimodalDataItem] = []
        input_ids = None
        # Handle raw items (need processing)
        if raw_images or raw_audios or raw_videos:
            collected_items, input_ids, ret = self._process_and_collect_mm_items(
                input_text=base_output.input_text,
                images=raw_images,
                audios=raw_audios,
                videos=raw_videos,
                **kwargs,
            )
            all_collected_items = collected_items
        else:
            ret = None

        # Handle dict items (processed or precomputed)
        for modality, dict_item in dict_items:
            input_format = dict_item.get("format", None)
            if input_format == "processor_output":
                items = self.collect_mm_items_from_processor_output(dict_item)
                for item in items:
                    item.format = MultimodalInputFormat.PROCESSOR_OUTPUT
                all_collected_items.extend(items)
            elif input_format == "precomputed_embedding":
                feature = dict_item["feature"]
                del dict_item["feature"]
                all_collected_items.append(
                    MultimodalDataItem(
                        modality=modality,
                        feature=feature,
                        format=MultimodalInputFormat.PRECOMPUTED_EMBEDDING,
                        model_specific_data=dict_item,
                    )
                )
        # Fallback tokenization if no raw items were processed
        if input_ids is None:
            input_ids = self._tokenizer(
                base_output.input_text,
                return_tensors="pt",
                add_special_tokens=True,
            ).input_ids.flatten()
```
**EN:** This block defines function `BaseMultimodalProcessor.process_and_combine_mm_data`. Parameters: self, base_output, mm_tokens. Process multimodal data and return the combined multimodal items and input_ids. Supports mixed modalities (images and audio in the same request). This subsection covers lines 1084-1161 of the same logical block.
**CN:** 该代码块定义函数 `BaseMultimodalProcessor.process_and_combine_mm_data`。 参数包括 self、base_output、mm_tokens。 文档字符串摘要：Process multimodal data and return the combined multimodal items and input_ids. Supports mixed modalities (images and audio in the same request). 本小节覆盖同一逻辑块中的第 1084-1161 行。

### Lines 1162-1162: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 1163-1241: Defines function BaseMultimodalProcessor.process_and_combine_mm_data (part 2)
```python
        # Add offsets to all items
        for mm_item in all_collected_items:
            mm_token_id = mm_tokens.get_token_id_by_modality(mm_item.modality)
            if mm_token_id is None:
                raise ValueError(f"No token id found for modality: {mm_item.modality}")
            mm_item.offsets = self.get_mm_items_offset(
                input_ids=input_ids,
                mm_token_id=mm_token_id,
            )

        # Split bundled items into per-image/video items for better cache granularity
        from sglang.srt.managers.mm_utils import get_new_expanded_mm_items

        all_collected_items = get_new_expanded_mm_items(all_collected_items)

        """
        solution for cuda-ipc memory-leak:
        1. memory-pool:  each time get a slice from memory-pool and use it as transport-data (with async lock guard)
        2. if can not get a slice , transport normal tensor
        3. copy tensor in scheduler and release it (use position mark)
        4. copy
        """

        if SGL_USE_CUDA_IPC:
            # post-process
            for item in all_collected_items:
                if isinstance(item.feature, torch.Tensor) and item.feature.is_cuda:
                    sync_flag, available_slice, byte_offset = (
                        self.cudaipc_mmfeature_pool.return_a_slice_tensor_with_flag(
                            item.feature
                        )
                    )
                    if isinstance(available_slice, torch.Tensor):
                        available_slice.copy_(
                            item.feature.view(torch.int8).view(-1), non_blocking=True
                        )
                        item.feature = CudaIpcTensorTransportProxy(
                            data=available_slice,
                            info_data=item.feature,
                            sync_buffer_meta=sync_flag,
                            pool_ipc_handle=(
                                self.cudaipc_mmfeature_pool._pool_ipc_handle
                                if _IPC_POOL_HANDLE_CACHE
                                else None
                            ),
                            pool_byte_offset=byte_offset,
                            pool_device_index=self.cudaipc_mmfeature_pool._pool_device_index,
                        )
                    elif not self.server_args.keep_mm_feature_on_device:
                        item.feature = item.feature.cpu()
                elif (
                    isinstance(item.precomputed_embeddings, torch.Tensor)
                    and item.precomputed_embeddings.is_cuda
                ):

                    sync_flag, available_slice, byte_offset = (
                        self.cudaipc_mmfeature_pool.return_a_slice_tensor_with_flag(
                            item.precomputed_embeddings
                        )
                    )
                    if isinstance(available_slice, torch.Tensor):
                        available_slice.copy_(
                            item.precomputed_embeddings.view(torch.int8).view(-1),
                            non_blocking=True,
                        )
                        item.precomputed_embeddings = CudaIpcTensorTransportProxy(
                            data=available_slice,
                            info_data=item.precomputed_embeddings,
                            sync_buffer_meta=sync_flag,
                            pool_ipc_handle=(
                                self.cudaipc_mmfeature_pool._pool_ipc_handle
                                if _IPC_POOL_HANDLE_CACHE
                                else None
                            ),
                            pool_byte_offset=byte_offset,
                            pool_device_index=self.cudaipc_mmfeature_pool._pool_device_index,
                        )
                    elif not self.server_args.keep_mm_feature_on_device:
                        item.precomputed_embeddings = item.precomputed_embeddings.cpu()
```
**EN:** This block defines function `BaseMultimodalProcessor.process_and_combine_mm_data`. Parameters: self, base_output, mm_tokens. Process multimodal data and return the combined multimodal items and input_ids. Supports mixed modalities (images and audio in the same request). This subsection covers lines 1163-1241 of the same logical block.
**CN:** 该代码块定义函数 `BaseMultimodalProcessor.process_and_combine_mm_data`。 参数包括 self、base_output、mm_tokens。 文档字符串摘要：Process multimodal data and return the combined multimodal items and input_ids. Supports mixed modalities (images and audio in the same request). 本小节覆盖同一逻辑块中的第 1163-1241 行。

### Lines 1242-1242: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 1243-1243: Defines function BaseMultimodalProcessor.process_and_combine_mm_data (part 3)
```python
        return all_collected_items, input_ids, ret
```
**EN:** This block defines function `BaseMultimodalProcessor.process_and_combine_mm_data`. Parameters: self, base_output, mm_tokens. Process multimodal data and return the combined multimodal items and input_ids. Supports mixed modalities (images and audio in the same request). This subsection covers lines 1243-1243 of the same logical block.
**CN:** 该代码块定义函数 `BaseMultimodalProcessor.process_and_combine_mm_data`。 参数包括 self、base_output、mm_tokens。 文档字符串摘要：Process multimodal data and return the combined multimodal items and input_ids. Supports mixed modalities (images and audio in the same request). 本小节覆盖同一逻辑块中的第 1243-1243 行。

## Key Concepts / 关键概念
- **Classes / 类**: `BaseMultiModalProcessorOutput`, `MultimodalSpecialTokens`, `BaseMultimodalProcessor`
- **Constants / 常量**: `SGL_USE_CUDA_IPC`, `_IPC_POOL_HANDLE_CACHE`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `abc`, `concurrent`, `concurrent.futures`, `dataclasses`, `multiprocessing`, `os`, `re`, `typing`
- **Third-Party / 第三方**: `PIL`, `numpy`, `torch`, `transformers`
- **Local Modules / 本地模块**: `sglang.srt.hardware_backend.npu.modules.qwen_vl_processor`, `sglang.srt.managers.mm_utils`, `sglang.srt.managers.schedule_batch`, `sglang.srt.server_args`, `sglang.srt.utils`, `sglang.srt.utils.cuda_ipc_transport_utils`, `sglang.srt.utils.video_decoder`
