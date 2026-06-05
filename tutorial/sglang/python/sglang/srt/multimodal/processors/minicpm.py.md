# minicpm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/processors/minicpm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This processor module converts raw multimodal inputs for minicpm models into tensors and metadata that the SGLang runtime can schedule. / 该处理器模块把 minicpm 模型的原始多模态输入转换为 SGLang 运行时可调度的张量与元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16: Imports dependencies
```python
from typing import List, Union

import torch

from sglang.srt.managers.schedule_batch import (
    Modality,
    MultimodalDataItem,
    MultimodalProcessorOutput,
)
from sglang.srt.models.minicpmo import MiniCPMO
from sglang.srt.models.minicpmv import MiniCPMV
from sglang.srt.multimodal.processors.base_processor import (
    BaseMultimodalProcessor,
    BaseMultiModalProcessorOutput,
    MultimodalSpecialTokens,
)
```
**EN:** This block groups related imports for the module, including typing.List, typing.Union, torch, sglang.srt.managers.schedule_batch.Modality, sglang.srt.managers.schedule_batch.MultimodalDataItem, and 6 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 typing.List, typing.Union, torch, sglang.srt.managers.schedule_batch.Modality, sglang.srt.managers.schedule_batch.MultimodalDataItem 等 6 项，为后续代码准备所需名称。

### Lines 17-19: Comments and module notes
```python


# Compatible with both 'O' and 'V'
```
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 20-20: Declares class MiniCPMMultimodalProcessor
```python
class MiniCPMMultimodalProcessor(BaseMultimodalProcessor):
```
**EN:** This block introduces class `MiniCPMMultimodalProcessor` as a reusable abstraction inside the module. It inherits from BaseMultimodalProcessor.
**CN:** 该代码块声明类 `MiniCPMMultimodalProcessor`，作为模块中的可复用抽象。 它继承自 BaseMultimodalProcessor。

### Lines 21-23: Declares models, support_dynamic_frame_expansion, gpu_image_decode
```python
    models = [MiniCPMV, MiniCPMO]
    support_dynamic_frame_expansion = True
    gpu_image_decode = False  # MiniCPM HF processor does not support tensor inputs
```
**EN:** This block initializes a related set of values in the MiniCPMMultimodalProcessor, including models, support_dynamic_frame_expansion, gpu_image_decode. Grouping these assignments together makes the surrounding configuration easier to follow.
**CN:** 该代码块在 MiniCPMMultimodalProcessor 中初始化一组相关值，包括 models, support_dynamic_frame_expansion, gpu_image_decode。将这些赋值集中在一起有助于理解周边配置。

### Lines 24-24: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MiniCPMMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MiniCPMMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 25-41: Defines function MiniCPMMultimodalProcessor.__init__
```python
    def __init__(self, hf_config, server_args, _processor, *args, **kwargs):
        super().__init__(hf_config, server_args, _processor, *args, **kwargs)
        # Collect special token ids
        tokenizer = self._processor.tokenizer
        self.slice_start_id = getattr(tokenizer, "slice_start_id", None)
        self.slice_end_id = getattr(tokenizer, "slice_end_id", None)
        self.audio_start_id = getattr(tokenizer, "audio_start_id", None)
        self.audio_end_id = getattr(tokenizer, "audio_end_id", None)
        self.im_start_id = getattr(tokenizer, "im_start_id", None)
        self.im_end_id = getattr(tokenizer, "im_end_id", None)
        self.im_token_id = getattr(tokenizer, "unk_id", None)
        self.mm_tokens = MultimodalSpecialTokens(
            image_token="(<image>./</image>)",
            audio_token="(<audio>./</audio>)",
            video_token="(<video>./</video>)",
            image_token_id=self.im_token_id,
        ).build(_processor)
```
**EN:** This block defines function `MiniCPMMultimodalProcessor.__init__`. Parameters: self, hf_config, server_args, _processor.
**CN:** 该代码块定义函数 `MiniCPMMultimodalProcessor.__init__`。 参数包括 self、hf_config、server_args、_processor。

### Lines 42-42: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MiniCPMMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MiniCPMMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 43-52: Defines function MiniCPMMultimodalProcessor._has_special_format
```python
    @staticmethod
    def _has_special_format(image_data, audio_data):
        """Check if any input items use processor_output or precomputed_embedding format."""
        for data in list(image_data or []) + list(audio_data or []):
            if isinstance(data, dict) and data.get("format") in (
                "processor_output",
                "precomputed_embedding",
            ):
                return True
        return False
```
**EN:** This block defines function `MiniCPMMultimodalProcessor._has_special_format`. Parameters: image_data, audio_data. Decorators: staticmethod. Check if any input items use processor_output or precomputed_embedding format.
**CN:** 该代码块定义函数 `MiniCPMMultimodalProcessor._has_special_format`。 参数包括 image_data、audio_data。 装饰器包括 staticmethod。 文档字符串摘要：Check if any input items use processor_output or precomputed_embedding format.

### Lines 53-53: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MiniCPMMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MiniCPMMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 54-129: Defines async function MiniCPMMultimodalProcessor._process_special_format (part 1)
```python
    async def _process_special_format(
        self, image_data, audio_data, input_text, request_obj, **kwargs
    ):
        """Handle processor_output and precomputed_embedding input formats.

        Delegates to the base class process_and_combine_mm_data which has
        built-in support for these formats.
        """
        if isinstance(input_text, list):
            user_input_ids = input_text
            prompt = ""
        else:
            user_input_ids = None
            prompt = input_text or ""

        # Normalize dicts: the HF MiniCPM processor returns "tgt_sizes" (plural)
        # but the base class ATTR_NAME_TO_MODALITY maps "tgt_size" (singular).
        # Also flatten the nested batch dimension so the structure matches
        # what the NORMAL path produces (flat list of per-patch tensors).
        normalized_images = []
        for d in image_data or []:
            if isinstance(d, dict):
                d = dict(d)
                if "tgt_sizes" in d and "tgt_size" not in d:
                    d["tgt_size"] = d.pop("tgt_sizes")
                if d.get("format") == "processor_output":
                    pixel_values = d.get("pixel_values")
                    tgt_size = d.get("tgt_size")
                    if pixel_values is not None and tgt_size is not None:
                        pv_flat, ts_flat = [], []
                        for pixel_b, tgt_b in zip(pixel_values, tgt_size):
                            if isinstance(pixel_b, (list, tuple)):
                                for pixel_n, tgt_n in zip(pixel_b, tgt_b):
                                    pv_flat.append(pixel_n)
                                    ts_flat.append(tgt_n)
                            else:
                                pv_flat.append(pixel_b)
                                ts_flat.append(tgt_b)
                        d["pixel_values"] = pv_flat
                        d["tgt_size"] = ts_flat
                normalized_images.append(d)
            else:
                normalized_images.append(d)

        normalized_audios = list(audio_data or [])

        if not prompt and (normalized_images or normalized_audios):
            images = [d for d in normalized_images if isinstance(d, dict)]
            audios = [d for d in normalized_audios if isinstance(d, dict)]

            raw_img_dropped = len(normalized_images) - len(images)
            raw_aud_dropped = len(normalized_audios) - len(audios)
            if raw_img_dropped > 0 or raw_aud_dropped > 0:
                raise ValueError(
                    f"[minicpm] Cannot process raw media with pre-tokenized "
                    f"input_ids. Provide multimodal data in 'processor_output' or "
                    f"'precomputed_embedding' format, or use a text prompt instead. "
                    f"(raw images dropped: {raw_img_dropped}, "
                    f"raw audios dropped: {raw_aud_dropped})"
                )

            base_output = BaseMultiModalProcessorOutput(
                input_text=prompt,
                images=images,
                audios=audios,
            )
        else:
            base_output = self.load_mm_data(
                prompt=prompt,
                image_data=normalized_images,
                audio_data=audio_data,
                multimodal_tokens=self.mm_tokens,
            )

        if base_output is None:
            return None
```
**EN:** This block defines async function `MiniCPMMultimodalProcessor._process_special_format`. Parameters: self, image_data, audio_data, input_text, request_obj. Handle processor_output and precomputed_embedding input formats. Delegates to the base class process_and_combine_mm_data which has built-in support for these formats. This subsection covers lines 54-129 of the same logical block.
**CN:** 该代码块定义异步函数 `MiniCPMMultimodalProcessor._process_special_format`。 参数包括 self、image_data、audio_data、input_text、request_obj。 文档字符串摘要：Handle processor_output and precomputed_embedding input formats. Delegates to the base class process_and_combine_mm_data which has built-in support for these formats. 本小节覆盖同一逻辑块中的第 54-129 行。

### Lines 130-130: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 131-160: Defines async function MiniCPMMultimodalProcessor._process_special_format (part 2)
```python
        mm_items, input_ids_tensor, ret = self.process_and_combine_mm_data(
            base_output, self.mm_tokens
        )

        if user_input_ids is not None:
            input_ids_tensor = torch.tensor(user_input_ids, dtype=torch.long)
            for mm_item in mm_items:
                if mm_item.modality == Modality.IMAGE:
                    image_offsets = self.get_mm_items_offset_by_pair(
                        input_ids=input_ids_tensor,
                        mm_start_id=self.im_start_id,
                        mm_end_id=self.im_end_id,
                    )
                    slice_offsets = self.get_mm_items_offset_by_pair(
                        input_ids=input_ids_tensor,
                        mm_start_id=self.slice_start_id,
                        mm_end_id=self.slice_end_id,
                    )
                    image_offsets.extend(slice_offsets)
                    mm_item.offsets = sorted(image_offsets)
                elif mm_item.modality == Modality.AUDIO:
                    if (
                        self.audio_start_id is not None
                        and self.audio_end_id is not None
                    ):
                        mm_item.offsets = self.get_mm_items_offset_by_pair(
                            input_ids=input_ids_tensor,
                            mm_start_id=self.audio_start_id,
                            mm_end_id=self.audio_end_id,
                        )
```
**EN:** This block defines async function `MiniCPMMultimodalProcessor._process_special_format`. Parameters: self, image_data, audio_data, input_text, request_obj. Handle processor_output and precomputed_embedding input formats. Delegates to the base class process_and_combine_mm_data which has built-in support for these formats. This subsection covers lines 131-160 of the same logical block.
**CN:** 该代码块定义异步函数 `MiniCPMMultimodalProcessor._process_special_format`。 参数包括 self、image_data、audio_data、input_text、request_obj。 文档字符串摘要：Handle processor_output and precomputed_embedding input formats. Delegates to the base class process_and_combine_mm_data which has built-in support for these formats. 本小节覆盖同一逻辑块中的第 131-160 行。

### Lines 161-161: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 162-172: Defines async function MiniCPMMultimodalProcessor._process_special_format (part 3)
```python
        return MultimodalProcessorOutput(
            mm_items=mm_items,
            input_ids=input_ids_tensor.flatten().tolist(),
            audio_start_id=self.audio_start_id,
            audio_end_id=self.audio_end_id,
            im_token_id=self.im_token_id,
            im_start_id=self.im_start_id,
            im_end_id=self.im_end_id,
            slice_start_id=self.slice_start_id,
            slice_end_id=self.slice_end_id,
        )
```
**EN:** This block defines async function `MiniCPMMultimodalProcessor._process_special_format`. Parameters: self, image_data, audio_data, input_text, request_obj. Handle processor_output and precomputed_embedding input formats. Delegates to the base class process_and_combine_mm_data which has built-in support for these formats. This subsection covers lines 162-172 of the same logical block.
**CN:** 该代码块定义异步函数 `MiniCPMMultimodalProcessor._process_special_format`。 参数包括 self、image_data、audio_data、input_text、request_obj。 文档字符串摘要：Handle processor_output and precomputed_embedding input formats. Delegates to the base class process_and_combine_mm_data which has built-in support for these formats. 本小节覆盖同一逻辑块中的第 162-172 行。

### Lines 173-173: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the MiniCPMMultimodalProcessor, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 MiniCPMMultimodalProcessor 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 174-242: Defines async function MiniCPMMultimodalProcessor.process_mm_data_async (part 1)
```python
    async def process_mm_data_async(
        self,
        image_data: List[Union[str, bytes]],
        audio_data: List[Union[str, bytes]],
        input_text,
        request_obj,
        **kwargs,
    ):
        if isinstance(input_text, list) or self._has_special_format(
            image_data, audio_data
        ):
            return await self._process_special_format(
                image_data=image_data,
                audio_data=audio_data,
                input_text=input_text,
                request_obj=request_obj,
                **kwargs,
            )

        base_output = self.load_mm_data(
            prompt=input_text,
            audio_data=audio_data,
            image_data=image_data,
            multimodal_tokens=self.mm_tokens,
        )
        if base_output is None:
            return None

        res = self.process_mm_data(
            input_text=base_output.input_text,
            images=base_output.images,
            audios=base_output.audios,
        )

        pixel_values = res["pixel_values"]
        tgt_sizes = res["tgt_sizes"]

        if not isinstance(pixel_values, (torch.Tensor, list)):
            raise ValueError(
                "Incorrect type of pixel values. " f"Got type: {type(pixel_values)}"
            )

        if not isinstance(tgt_sizes, (torch.Tensor, list)):
            raise ValueError(
                "Incorrect type of target sizes. " f"Got type: {type(tgt_sizes)}"
            )

        if len(pixel_values) != len(tgt_sizes):
            raise ValueError(
                "Inconsistent batch lengths, found: "
                f"{len(pixel_values)} vs. {len(tgt_sizes)}"
            )

        # Track slices per image (like vLLM's num_slices)
        slices_per_image: List[int] = []
        pixel_values_flat: List[torch.Tensor] = []
        tgt_sizes_flat: List[torch.Tensor] = []
        for pixel_b, tgt_b in zip(pixel_values, tgt_sizes):
            # per image
            if len(pixel_b) != len(tgt_b):
                raise ValueError(
                    "Inconsistent N lengths, found: " f"{len(pixel_b)} vs {len(tgt_b)}"
                )
            slices_per_image.append(len(pixel_b))
            for pixel_n, tgt_n in zip(pixel_b, tgt_b):
                pixel_values_flat += [pixel_n]
                tgt_sizes_flat += [tgt_n]

        pixel_values = pixel_values_flat
```
**EN:** This block defines async function `MiniCPMMultimodalProcessor.process_mm_data_async`. Parameters: self, image_data, audio_data, input_text, request_obj. This subsection covers lines 174-242 of the same logical block.
**CN:** 该代码块定义异步函数 `MiniCPMMultimodalProcessor.process_mm_data_async`。 参数包括 self、image_data、audio_data、input_text、request_obj。 本小节覆盖同一逻辑块中的第 174-242 行。

### Lines 243-243: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 244-273: Defines async function MiniCPMMultimodalProcessor.process_mm_data_async (part 2)
```python
        items = []
        input_ids = res["input_ids"].flatten()
        image_offsets = self.get_mm_items_offset_by_pair(
            input_ids=input_ids, mm_start_id=self.im_start_id, mm_end_id=self.im_end_id
        )
        slice_offsets = self.get_mm_items_offset_by_pair(
            input_ids=input_ids,
            mm_start_id=self.slice_start_id,
            mm_end_id=self.slice_end_id,
        )
        image_offsets.extend(slice_offsets)
        image_offsets = sorted(image_offsets)

        # Create one item per image, each with its own slices and offsets
        if len(pixel_values) != 0:
            pv_idx = 0
            offset_idx = 0
            for num_slices in slices_per_image:
                items.append(
                    MultimodalDataItem(
                        feature=pixel_values[pv_idx : pv_idx + num_slices],
                        offsets=image_offsets[offset_idx : offset_idx + num_slices],
                        model_specific_data={
                            "tgt_size": tgt_sizes_flat[pv_idx : pv_idx + num_slices]
                        },
                        modality=Modality.IMAGE,
                    )
                )
                pv_idx += num_slices
                offset_idx += num_slices
```
**EN:** This block defines async function `MiniCPMMultimodalProcessor.process_mm_data_async`. Parameters: self, image_data, audio_data, input_text, request_obj. This subsection covers lines 244-273 of the same logical block.
**CN:** 该代码块定义异步函数 `MiniCPMMultimodalProcessor.process_mm_data_async`。 参数包括 self、image_data、audio_data、input_text、request_obj。 本小节覆盖同一逻辑块中的第 244-273 行。

### Lines 274-274: Spacing and separators
```python

```
**EN:** This blank line separates adjacent pieces of logic inside the same block.
**CN:** 该空行用于分隔同一代码块中的相邻逻辑。

### Lines 275-305: Defines async function MiniCPMMultimodalProcessor.process_mm_data_async (part 3)
```python
        if (
            "audio_features" in res
            and res["audio_features"] is not None
            and len(res["audio_features"]) != 0
        ):
            if self.audio_start_id is not None and self.audio_end_id is not None:
                audio_offsets = self.get_mm_items_offset_by_pair(
                    input_ids=input_ids,
                    mm_start_id=self.audio_start_id,
                    mm_end_id=self.audio_end_id,
                )
            else:
                audio_offsets = None
            item = MultimodalDataItem(
                feature=[res["audio_features"]],
                model_specific_data={"audio_feature_lens": res["audio_feature_lens"]},
                offsets=audio_offsets,
                modality=Modality.AUDIO,
            )
            items += [item]
        return MultimodalProcessorOutput(
            mm_items=items,
            input_ids=input_ids.tolist(),
            audio_start_id=self.audio_start_id,
            audio_end_id=self.audio_end_id,
            im_token_id=self.im_token_id,
            im_start_id=self.im_start_id,
            im_end_id=self.im_end_id,
            slice_start_id=self.slice_start_id,
            slice_end_id=self.slice_end_id,
        )
```
**EN:** This block defines async function `MiniCPMMultimodalProcessor.process_mm_data_async`. Parameters: self, image_data, audio_data, input_text, request_obj. This subsection covers lines 275-305 of the same logical block.
**CN:** 该代码块定义异步函数 `MiniCPMMultimodalProcessor.process_mm_data_async`。 参数包括 self、image_data、audio_data、input_text、request_obj。 本小节覆盖同一逻辑块中的第 275-305 行。

## Key Concepts / 关键概念
- **Classes / 类**: `MiniCPMMultimodalProcessor`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `typing`
- **Third-Party / 第三方**: `torch`
- **Local Modules / 本地模块**: `sglang.srt.managers.schedule_batch`, `sglang.srt.models.minicpmo`, `sglang.srt.models.minicpmv`, `sglang.srt.multimodal.processors.base_processor`
