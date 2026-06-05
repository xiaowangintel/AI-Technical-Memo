# model_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/generation/vlm_utils/model_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides shared test utilities for multimodal processing and model-facing behavior. The file exposes 91 helper/class block(s) used by nearby tests. / [CN] 为多模态处理与面向模型的行为提供共享测试工具。该文件暴露了 91 个辅助函数/类块，供附近测试复用。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L8-L35)
```python
import logging
import types
import warnings
from pathlib import PosixPath

import numpy as np
import numpy.typing as npt
import PIL.Image
import pytest
import regex as re
import torch
from PIL.Image import Image
from transformers import (
    AutoConfig,
    AutoTokenizer,
    BatchFeature,
    GenerationConfig,
    GenerationMixin,
# ... 6 lines omitted for brevity ...
from vllm.utils.collection_utils import is_list_of

from .....conftest import HfRunner, ImageAsset, ImageTestAssets
from .types import RunnerOutput
```
**EN:** Imports standard-library modules such as `io`, `logging`, `pathlib.PosixPath`, third-party packages like `PIL.Image`, `PIL.Image.Image`, `mantis.models.mllava.MLlavaProcessor`, project helpers such as `vllm.logprobs.SampleLogprobs`, `vllm.model_executor.models.moondream3.reconstruct_from_crops`, `vllm.model_executor.models.tarsier.get_vision_encoder_info`.
**CN:** 导入标准库模块（如 `io`、`logging`、`pathlib.PosixPath`）、第三方包（如 `PIL.Image`、`PIL.Image.Image`、`mantis.models.mllava.MLlavaProcessor`）、项目内辅助模块（如 `vllm.logprobs.SampleLogprobs`、`vllm.model_executor.models.moondream3.reconstruct_from_crops`、`vllm.model_executor.models.tarsier.get_vision_encoder_info`）。

### Module setup / 模块级配置: logger (L37-L37)
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `logger`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `logger`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: blip2_vllm_to_hf_output (L41-L52)
```python
def blip2_vllm_to_hf_output(vllm_output: RunnerOutput, model: str) -> RunnerOutput:
    """Sanitize vllm output [blip2 models] to be comparable with hf output."""
    _, output_str, out_logprobs = vllm_output

    hf_output_str = output_str + "\n"

    tokenizer = AutoTokenizer.from_pretrained(model)
    hf_output_ids = tokenizer.encode(hf_output_str)
    assert hf_output_ids[0] == tokenizer.bos_token_id
    hf_output_ids = hf_output_ids[1:]

    return hf_output_ids, hf_output_str, out_logprobs
```
**EN:** This helper encapsulates reusable logic in `blip2_vllm_to_hf_output`. Key inputs are `vllm_output`, `model`. It returns computed state or helper objects back to the caller. The main assertion is `hf_output_ids[0] == tokenizer.bos_token_id`.
**CN:** 这个辅助函数将可复用逻辑封装在 `blip2_vllm_to_hf_output` 中。 关键输入包括 `vllm_output`、`model`。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `hf_output_ids[0] == tokenizer.bos_token_id`。

### Helper / 辅助函数: fuyu_vllm_to_hf_output (L55-L61)
```python
def fuyu_vllm_to_hf_output(vllm_output: RunnerOutput, model: str) -> RunnerOutput:
    """Sanitize vllm output [fuyu models] to be comparable with hf output."""
    output_ids, output_str, out_logprobs = vllm_output

    hf_output_str = output_str.lstrip() + "|ENDOFTEXT|"

    return output_ids, hf_output_str, out_logprobs
```
**EN:** This helper encapsulates reusable logic in `fuyu_vllm_to_hf_output`. Key inputs are `vllm_output`, `model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `fuyu_vllm_to_hf_output` 中。 关键输入包括 `vllm_output`、`model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: qwen_vllm_to_hf_output (L64-L72)
```python
def qwen_vllm_to_hf_output(
    vllm_output: RunnerOutput, model: str
) -> tuple[list[int], str, SampleLogprobs | None]:
    """Sanitize vllm output [qwen models] to be comparable with hf output."""
    output_ids, output_str, out_logprobs = vllm_output

    hf_output_str = output_str + "<|endoftext|>"

    return output_ids, hf_output_str, out_logprobs
```
**EN:** This helper encapsulates reusable logic in `qwen_vllm_to_hf_output`. Key inputs are `vllm_output`, `model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `qwen_vllm_to_hf_output` 中。 关键输入包括 `vllm_output`、`model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: qwen2_vllm_to_hf_output (L75-L83)
```python
def qwen2_vllm_to_hf_output(
    vllm_output: RunnerOutput, model: str
) -> tuple[list[int], str, SampleLogprobs | None]:
    """Sanitize vllm output [qwen2 models] to be comparable with hf output."""
    output_ids, output_str, out_logprobs = vllm_output

    hf_output_str = output_str + "<|im_end|>"

    return output_ids, hf_output_str, out_logprobs
```
**EN:** This helper encapsulates reusable logic in `qwen2_vllm_to_hf_output`. Key inputs are `vllm_output`, `model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `qwen2_vllm_to_hf_output` 中。 关键输入包括 `vllm_output`、`model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: kimiv_vl_vllm_to_hf_output (L86-L94)
```python
def kimiv_vl_vllm_to_hf_output(
    vllm_output: RunnerOutput, model: str
) -> tuple[list[int], str, SampleLogprobs | None]:
    """Sanitize vllm output [kimi_vl models] to be comparable with hf output."""
    output_ids, output_str, out_logprobs = vllm_output

    hf_output_str = output_str + "<|im_end|>[EOS]"

    return output_ids, hf_output_str, out_logprobs
```
**EN:** This helper encapsulates reusable logic in `kimiv_vl_vllm_to_hf_output`. Key inputs are `vllm_output`, `model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `kimiv_vl_vllm_to_hf_output` 中。 关键输入包括 `vllm_output`、`model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: llava_image_vllm_to_hf_output (L97-L102)
```python
def llava_image_vllm_to_hf_output(
    vllm_output: RunnerOutput, model: str
) -> RunnerOutput:
    config = AutoConfig.from_pretrained(model)
    mm_token_id = config.image_token_index
    return _llava_vllm_to_hf_output(vllm_output, model, mm_token_id)
```
**EN:** This helper encapsulates reusable logic in `llava_image_vllm_to_hf_output`. Key inputs are `vllm_output`, `model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `llava_image_vllm_to_hf_output` 中。 关键输入包括 `vllm_output`、`model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: llava_video_vllm_to_hf_output (L105-L110)
```python
def llava_video_vllm_to_hf_output(
    vllm_output: RunnerOutput, model: str
) -> tuple[list[int], str, SampleLogprobs | None]:
    config = AutoConfig.from_pretrained(model)
    mm_token_id = config.video_token_index
    return _llava_vllm_to_hf_output(vllm_output, model, mm_token_id)
```
**EN:** This helper encapsulates reusable logic in `llava_video_vllm_to_hf_output`. Key inputs are `vllm_output`, `model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `llava_video_vllm_to_hf_output` 中。 关键输入包括 `vllm_output`、`model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _llava_vllm_to_hf_output (L113-L135)
```python
def _llava_vllm_to_hf_output(
    vllm_output: RunnerOutput, model: str, mm_token_id: int
) -> RunnerOutput:
    """Sanitize vllm output [Llava models] to be comparable with hf output."""
    output_ids, output_str, out_logprobs = vllm_output

    tokenizer = AutoTokenizer.from_pretrained(model)
    eos_token_id = tokenizer.eos_token_id

    hf_output_ids = [
        token_id
        for idx, token_id in enumerate(output_ids)
        if token_id != mm_token_id or output_ids[idx - 1] != mm_token_id
    ]

    # output_str[0] is not " " in some cases, e.g., Granite Vision,
    # but for most llava based models, this is the case
    hf_output_str = output_str[1:] if output_str[0] == " " else output_str

    if hf_output_ids[-1] == eos_token_id:
        hf_output_str = hf_output_str + tokenizer.decode(eos_token_id)

    return hf_output_ids, hf_output_str, out_logprobs
```
**EN:** This helper encapsulates reusable logic in `_llava_vllm_to_hf_output`. Key inputs are `vllm_output`, `model`, `mm_token_id`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_llava_vllm_to_hf_output` 中。 关键输入包括 `vllm_output`、`model`、`mm_token_id`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: llava_onevision_hf_model_kwargs (L138-L142)
```python
def llava_onevision_hf_model_kwargs(model: str) -> dict:
    """Workaround to fix the sliding window issue in llava_onevision."""
    config = AutoConfig.from_pretrained(model)
    config.text_config.sliding_window = None
    return config.to_dict()
```
**EN:** This helper encapsulates reusable logic in `llava_onevision_hf_model_kwargs`. Key inputs are `model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `llava_onevision_hf_model_kwargs` 中。 关键输入包括 `model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: llava_onevision_vllm_to_hf_output (L145-L167)
```python
def llava_onevision_vllm_to_hf_output(
    vllm_output: RunnerOutput, model: str
) -> RunnerOutput:
    """Sanitize vllm output [llava-onevision] to compare with hf output."""
    output_ids, output_str, out_logprobs = vllm_output

    config = AutoConfig.from_pretrained(model)
    video_token_id = config.video_token_index

    tokenizer = AutoTokenizer.from_pretrained(model)
    eos_token_id = tokenizer.eos_token_id

    hf_output_ids = [
        token_id
        for idx, token_id in enumerate(output_ids)
        if token_id != video_token_id or output_ids[idx - 1] != video_token_id
    ]

    hf_output_str = output_str
    if hf_output_ids[-1] == eos_token_id:
        hf_output_str = hf_output_str + tokenizer.decode(eos_token_id)

    return hf_output_ids, hf_output_str, out_logprobs
```
**EN:** This helper encapsulates reusable logic in `llava_onevision_vllm_to_hf_output`. Key inputs are `vllm_output`, `model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `llava_onevision_vllm_to_hf_output` 中。 关键输入包括 `vllm_output`、`model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: mantis_vllm_to_hf_output (L170-L176)
```python
def mantis_vllm_to_hf_output(vllm_output: RunnerOutput, model: str) -> RunnerOutput:
    """Sanitize vllm output [mantis] to compare with hf output."""
    output_ids, output_str, out_logprobs = vllm_output

    hf_output_str = output_str + "<|eot_id|>"

    return output_ids, hf_output_str, out_logprobs
```
**EN:** This helper encapsulates reusable logic in `mantis_vllm_to_hf_output`. Key inputs are `vllm_output`, `model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `mantis_vllm_to_hf_output` 中。 关键输入包括 `vllm_output`、`model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: phi3v_vllm_to_hf_output (L179-L194)
```python
def phi3v_vllm_to_hf_output(vllm_output: RunnerOutput, model: str) -> RunnerOutput:
    """Sanitize vllm output [phi3v] to be comparable with hf output."""
    _, output_str, out_logprobs = vllm_output

    output_str_without_image = re.sub(r"(<\|image_\d+\|>)+", "", output_str)
    assert output_str_without_image[0] == " "
    output_str_without_image = output_str_without_image[1:]

    hf_output_str = output_str_without_image + "<|end|><|endoftext|>"

    tokenizer = AutoTokenizer.from_pretrained(model)
    hf_output_ids = tokenizer.encode(output_str_without_image)
    assert hf_output_ids[0] == 1
    hf_output_ids = hf_output_ids[1:]

    return hf_output_ids, hf_output_str, out_logprobs
```
**EN:** This helper encapsulates reusable logic in `phi3v_vllm_to_hf_output`. Key inputs are `vllm_output`, `model`. It returns computed state or helper objects back to the caller. The main assertion is `output_str_without_image[0] == ' '` and `hf_output_ids[0] == 1`.
**CN:** 这个辅助函数将可复用逻辑封装在 `phi3v_vllm_to_hf_output` 中。 关键输入包括 `vllm_output`、`model`。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `output_str_without_image[0] == ' '` and `hf_output_ids[0] == 1`。

### Helper / 辅助函数: paligemma_vllm_to_hf_output (L197-L218)
```python
def paligemma_vllm_to_hf_output(vllm_output: RunnerOutput, model: str) -> RunnerOutput:
    """Sanitize vllm output to be comparable with hf output."""
    output_ids, output_str, out_logprobs = vllm_output

    config = AutoConfig.from_pretrained(model)
    image_token_id = config.image_token_index

    tokenizer = AutoTokenizer.from_pretrained(model)
    eos_token_id = tokenizer.eos_token_id

    hf_output_ids = [
        token_id
        for idx, token_id in enumerate(output_ids)
        if token_id != image_token_id or output_ids[idx - 1] != image_token_id
    ]

    hf_output_str = output_str

    if hf_output_ids[-1] == eos_token_id:
        hf_output_str = hf_output_str + tokenizer.decode(eos_token_id)

    return hf_output_ids, hf_output_str, out_logprobs
```
**EN:** This helper encapsulates reusable logic in `paligemma_vllm_to_hf_output`. Key inputs are `vllm_output`, `model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `paligemma_vllm_to_hf_output` 中。 关键输入包括 `vllm_output`、`model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: deepseekvl2_trunc_hf_output (L222-L226)
```python
def deepseekvl2_trunc_hf_output(hf_output: RunnerOutput, model: str) -> RunnerOutput:
    output_ids, output_str, out_logprobs = hf_output
    if output_str.endswith("<｜end▁of▁sentence｜>"):
        output_str = output_str.split("<｜end▁of▁sentence｜>")[0]
    return output_ids, output_str, out_logprobs
```
**EN:** This helper encapsulates reusable logic in `deepseekvl2_trunc_hf_output`. Key inputs are `hf_output`, `model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `deepseekvl2_trunc_hf_output` 中。 关键输入包括 `hf_output`、`model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: idefics3_trunc_hf_output (L229-L233)
```python
def idefics3_trunc_hf_output(hf_output: RunnerOutput, model: str) -> RunnerOutput:
    output_ids, output_str, out_logprobs = hf_output
    if output_str.endswith("<end_of_utterance>"):
        output_str = output_str.split("<end_of_utterance>")[0]
    return output_ids, output_str, out_logprobs
```
**EN:** This helper encapsulates reusable logic in `idefics3_trunc_hf_output`. Key inputs are `hf_output`, `model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `idefics3_trunc_hf_output` 中。 关键输入包括 `hf_output`、`model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: smolvlm_trunc_hf_output (L236-L238)
```python
def smolvlm_trunc_hf_output(hf_output: RunnerOutput, model: str) -> RunnerOutput:
    # Based on Idefics3
    return idefics3_trunc_hf_output(hf_output, model)
```
**EN:** This helper encapsulates reusable logic in `smolvlm_trunc_hf_output`. Key inputs are `hf_output`, `model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `smolvlm_trunc_hf_output` 中。 关键输入包括 `hf_output`、`model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: minicpmv_trunc_hf_output (L241-L245)
```python
def minicpmv_trunc_hf_output(hf_output: RunnerOutput, model: str) -> RunnerOutput:
    output_ids, output_str, out_logprobs = hf_output
    if output_str.endswith("<|eot_id|>"):
        output_str = output_str.split("<|eot_id|>")[0]
    return output_ids, output_str, out_logprobs
```
**EN:** This helper encapsulates reusable logic in `minicpmv_trunc_hf_output`. Key inputs are `hf_output`, `model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `minicpmv_trunc_hf_output` 中。 关键输入包括 `hf_output`、`model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: minimax_vl_01_hf_output (L248-L252)
```python
def minimax_vl_01_hf_output(hf_output: RunnerOutput, model: str) -> RunnerOutput:
    output_ids, output_str, out_logprobs = hf_output
    if output_str.endswith("<end_of_sentence>"):
        output_str = output_str.split("<end_of_sentence>")[0]
    return output_ids, output_str, out_logprobs
```
**EN:** This helper encapsulates reusable logic in `minimax_vl_01_hf_output`. Key inputs are `hf_output`, `model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `minimax_vl_01_hf_output` 中。 关键输入包括 `hf_output`、`model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: ultravox_trunc_hf_output (L255-L263)
```python
def ultravox_trunc_hf_output(hf_output: RunnerOutput, model: str) -> RunnerOutput:
    output_ids, output_str, out_logprobs = hf_output

    tokenizer = AutoTokenizer.from_pretrained(model)
    eos_token_id = tokenizer.eos_token_id
    eos_token = tokenizer.decode(eos_token_id)
    if output_str.endswith(eos_token):
        output_str = output_str.split(eos_token)[0]
    return output_ids, output_str, out_logprobs
```
**EN:** This helper encapsulates reusable logic in `ultravox_trunc_hf_output`. Key inputs are `hf_output`, `model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `ultravox_trunc_hf_output` 中。 关键输入包括 `hf_output`、`model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: get_llava_embeddings (L267-L268)
```python
def get_llava_embeddings(image_assets: ImageTestAssets):
    return [asset.image_embeds for asset in image_assets]
```
**EN:** This helper encapsulates reusable logic in `get_llava_embeddings`. Key inputs are `image_assets`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `get_llava_embeddings` 中。 关键输入包括 `image_assets`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: qwen_prompt_path_encoder (L272-L298)
```python
def qwen_prompt_path_encoder(
    tmp_path: PosixPath, prompt: str, assets: list[ImageAsset] | ImageTestAssets
) -> str:
    """Given a temporary dir path, export one or more image assets into the
    tempdir & replace its contents with the local path to the string so that
    the HF version of Qwen-VL can resolve the path and load the image in its
    forward() call.

    Args:
        tmp_path: Tempdir for test under consideration.
        prompt: Prompt with image placeholders.
        assets: list of image assets whose len equals the num placeholders.
    """
    # Ensure that the number of placeholders matches the number of assets;
    # If this is not true, the test is probably written incorrectly.
    assert prompt.count("<img></img>") == len(assets)

    # Replace the placeholders with local paths to the exported assets
    for asset in assets:
        image_tmp_path = tmp_path / f"{asset.name}.jpg"
        asset.pil_image.save(image_tmp_path)
        prompt = prompt.replace(
            "<img></img>",
            f"<img>{image_tmp_path}</img>",
            1,
        )
    return prompt
```
**EN:** This helper encapsulates reusable logic in `qwen_prompt_path_encoder`. Key inputs are `tmp_path`, `prompt`, `assets`. It returns computed state or helper objects back to the caller. The main assertion is `prompt.count('<img></img>') == len(assets)`.
**CN:** 这个辅助函数将可复用逻辑封装在 `qwen_prompt_path_encoder` 中。 关键输入包括 `tmp_path`、`prompt`、`assets`。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `prompt.count('<img></img>') == len(assets)`。

### Helper / 辅助函数: deepseekvl2_patch_hf_runner (L302-L327)
```python
def deepseekvl2_patch_hf_runner(hf_model: HfRunner) -> HfRunner:
    """Patches and returns an instance of the HfRunner to use for GLM4."""
    hf_processor = hf_model.processor

    def processor(*args, text="", images=None, **kwargs):
        if isinstance(images, Image):
            images = [images]
        # inputs is a custom class instead of dict or BatchFeature
        inputs = hf_processor(
            *args,
            prompt=text,
            images=images,
            **kwargs,
        )
        inputs = {
            k: inputs[k]
            for k in inputs.keys()  # noqa
            if k not in ("seq_lens", "sft_format")
        }
        return BatchFeature(data=inputs, tensor_type="pt")

    hf_model.processor = processor
    hf_model.model.get_output_embeddings = (
        lambda: hf_model.model.language.model.embed_tokens
    )
    return hf_model
```
**EN:** This helper encapsulates reusable logic in `deepseekvl2_patch_hf_runner`. Key inputs are `hf_model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `deepseekvl2_patch_hf_runner` 中。 关键输入包括 `hf_model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: gemma3_patch_hf_runner (L330-L349)
```python
def gemma3_patch_hf_runner(hf_model: HfRunner) -> HfRunner:
    """Patches and returns an instance of the HfRunner to use for Gemma 3."""
    hf_processor = hf_model.processor

    def processor(*args, **kwargs):
        return hf_processor(*args, do_pan_and_scan=True, **kwargs)

    hf_model.processor = processor

    orig_generate = hf_model.model.generate

    def _generate(self, *args, **kwargs):
        # FIXME: https://github.com/huggingface/transformers/issues/38333
        kwargs["disable_compile"] = True

        return orig_generate(*args, **kwargs)

    hf_model.model.generate = types.MethodType(_generate, hf_model.model)

    return hf_model
```
**EN:** This helper encapsulates reusable logic in `gemma3_patch_hf_runner`. Key inputs are `hf_model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `gemma3_patch_hf_runner` 中。 关键输入包括 `hf_model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: gemma3_vllm_to_hf_output (L352-L372)
```python
def gemma3_vllm_to_hf_output(vllm_output: RunnerOutput, model: str) -> RunnerOutput:
    """Sanitize vllm output [gemma-3] to compare with hf output."""
    output_ids, output_str, out_logprobs = vllm_output

    config = AutoConfig.from_pretrained(model)
    image_token_id = config.image_token_id

    tokenizer = AutoTokenizer.from_pretrained(model)
    eos_token_id = tokenizer.eos_token_id

    hf_output_ids = [
        token_id
        for idx, token_id in enumerate(output_ids)
        if token_id != image_token_id
    ]

    hf_output_str = output_str
    if hf_output_ids[-1] == eos_token_id:
        hf_output_str = hf_output_str + tokenizer.decode(eos_token_id)

    return hf_output_ids, hf_output_str, out_logprobs
```
**EN:** This helper encapsulates reusable logic in `gemma3_vllm_to_hf_output`. Key inputs are `vllm_output`, `model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `gemma3_vllm_to_hf_output` 中。 关键输入包括 `vllm_output`、`model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: glm4v_patch_hf_runner (L375-L440)
```python
def glm4v_patch_hf_runner(hf_model: HfRunner) -> HfRunner:
    """Patches and returns an instance of the HfRunner to use for GLM4V."""
    if current_platform.is_rocm():
        import types

        config = hf_model.model.config
        if hasattr(config, "num_layers") and not hasattr(config, "num_hidden_layers"):
            config.num_hidden_layers = config.num_layers
        config.output_hidden_states = True

        def patched_prepare_cache(
            self, generation_config, model_kwargs, *args, **kwargs
        ):
            model_kwargs["past_key_values"] = None
            model_kwargs["use_cache"] = False
            return model_kwargs

        hf_model.model._prepare_cache_for_generation = types.MethodType(
# ... 40 lines omitted for brevity ...
            **kwargs,
        )

    hf_model.processor = processor
    hf_model.model.get_output_embeddings = (
        lambda: hf_model.model.transformer.output_layer
    )
    return hf_model
```
**EN:** This helper encapsulates reusable logic in `glm4v_patch_hf_runner`. Key inputs are `hf_model`. It returns computed state or helper objects back to the caller. The main assertion is `len(contents) == len(images)`.
**CN:** 这个辅助函数将可复用逻辑封装在 `glm4v_patch_hf_runner` 中。 关键输入包括 `hf_model`。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `len(contents) == len(images)`。

### Helper / 辅助函数: glm4_1v_patch_hf_runner (L443-L469)
```python
def glm4_1v_patch_hf_runner(hf_model: HfRunner) -> HfRunner:
    """Patches and returns an instance of the HfRunner to use for GLM4.1V."""
    hf_processor = hf_model.processor

    def processor(*args, videos=None, **kwargs):
        if videos is not None and is_list_of(videos, tuple):
            # If videos is a list of tuples, we assume each tuple contains
            # (video_array, metadata) as in the case of GLM4.1V.
            # Filter out 'do_sample_frames' as it's not a valid VideoMetadata arg
            video_metadata = [
                [
                    VideoMetadata(
                        **{k: v for k, v in video[1].items() if k != "do_sample_frames"}
                    )
                ]
                for video in videos
            ]
            videos = [[video[0]] for video in videos]
        else:
            video_metadata = None

        return hf_processor(
            *args, videos=videos, video_metadata=video_metadata, **kwargs
        )

    hf_model.processor = processor
    return hf_model
```
**EN:** This helper encapsulates reusable logic in `glm4_1v_patch_hf_runner`. Key inputs are `hf_model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `glm4_1v_patch_hf_runner` 中。 关键输入包括 `hf_model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: h2ovl_patch_hf_runner (L472-L530)
```python
def h2ovl_patch_hf_runner(hf_model: HfRunner) -> HfRunner:
    """Patches and returns an instance of the HfRunner to use for H2OVL."""

    class H2OVLProcessor:
        """A simple processor for H2OVL models."""

        def __init__(self, hf_runner: HfRunner):
            self.num_image_token = hf_runner.model.num_image_token
            self.tokenizer = hf_runner.tokenizer

            self.config = AutoConfig.from_pretrained(
                hf_runner.model_name, trust_remote_code=True
            )
            self.vision_config = self.config.vision_config
            self.use_thumbnail = self.config.use_thumbnail
            self.use_msac = self.config.use_msac
            self.min_num = self.config.min_dynamic_patch
            self.max_num = self.config.max_dynamic_patch
# ... 33 lines omitted for brevity ...
    img_context_token_id = hf_model.tokenizer.convert_tokens_to_ids("<IMG_CONTEXT>")
    hf_model.model.img_context_token_id = img_context_token_id
    hf_model.processor = H2OVLProcessor(hf_model)
    hf_model.model.get_output_embeddings = (
        lambda: hf_model.model.language_model.get_output_embeddings()
    )
    hf_model.model.generate = types.MethodType(_internvl_generate, hf_model.model)
    return hf_model
```
**EN:** This helper encapsulates reusable logic in `h2ovl_patch_hf_runner`. Key inputs are `hf_model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `h2ovl_patch_hf_runner` 中。 关键输入包括 `hf_model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: isaac_patch_hf_runner (L533-L737)
```python
def isaac_patch_hf_runner(hf_model: HfRunner) -> HfRunner:
    """Patch HF runner for Isaac:
    1) Move processor outputs to model device
    2) Ensure IsaacModel.forward returns hidden_states
    for compatibility with hidden_states_to_seq_logprobs()
    """

    from perceptron.tensorstream import TextType
    from perceptron.tensorstream.ops import compute_mrope_pos_tensor, modality_mask
    from transformers.modeling_outputs import BaseModelOutputWithPast

    def compute_position_ids_input_ids(input_ids: torch.Tensor) -> torch.Tensor:
        """
        Create 3D positional indices for token input.
        """
        batch_size, seq_length = input_ids.shape
        position_ids = torch.arange(seq_length, device=input_ids.device)
        position_ids = position_ids.view(1, -1).expand(batch_size, -1)
# ... 179 lines omitted for brevity ...
            last_hidden_state=hidden_states,
            past_key_values=past_key_values,
            hidden_states=all_hidden_states,
        )

    isaac_model.forward = types.MethodType(patched_forward, isaac_model)

    return hf_model
```
**EN:** This helper encapsulates reusable logic in `isaac_patch_hf_runner`. Key inputs are `hf_model`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `isaac_patch_hf_runner` 中。 关键输入包括 `hf_model`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: skyworkr1v_patch_hf_runner (L740-L796)
```python
def skyworkr1v_patch_hf_runner(hf_model: HfRunner) -> HfRunner:
    """Patches and returns an instance of the HfRunner to use for SkyworkR1V."""

    class SkyworkR1VProcessor:
        """A simple processor for SkyworkR1V."""

        def __init__(self, hf_runner: HfRunner):
            self.num_image_token = hf_runner.model.num_image_token
            self.tokenizer = hf_runner.tokenizer

            self.config = AutoConfig.from_pretrained(
                hf_runner.model_name, trust_remote_code=True
            )
            self.vision_config = self.config.vision_config
            self.use_thumbnail = self.config.use_thumbnail
            self.min_num = self.config.min_dynamic_patch
            self.max_num = self.config.max_dynamic_patch
            self.image_size = self.vision_config.image_size
# ... 31 lines omitted for brevity ...
    img_context_token_id = hf_model.tokenizer.convert_tokens_to_ids("<IMG_CONTEXT>")
    hf_model.model.img_context_token_id = img_context_token_id
    hf_model.processor = SkyworkR1VProcessor(hf_model)
    hf_model.model.get_output_embeddings = (
        lambda: hf_model.model.language_model.get_output_embeddings()
    )
    hf_model.model.generate = types.MethodType(_internvl_generate, hf_model.model)
    return hf_model
```
**EN:** This helper encapsulates reusable logic in `skyworkr1v_patch_hf_runner`. Key inputs are `hf_model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `skyworkr1v_patch_hf_runner` 中。 关键输入包括 `hf_model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: internvl_patch_hf_runner (L799-L905)
```python
def internvl_patch_hf_runner(hf_model: HfRunner) -> HfRunner:
    """Patches and returns an instance of the HfRunner to use for InternVL."""

    class InternVLProcessor:
        """A simple processor for InternVL2 which misses a processor."""

        def __init__(self, hf_runner: HfRunner):
            self.num_image_token = hf_runner.model.num_image_token
            self.tokenizer = hf_runner.tokenizer

            self.config = AutoConfig.from_pretrained(
                hf_runner.model_name, trust_remote_code=True
            )
            self.vision_config = self.config.vision_config
            self.use_thumbnail = self.config.use_thumbnail
            self.min_num = self.config.min_dynamic_patch
            self.max_num = self.config.max_dynamic_patch
            self.image_size = self.vision_config.image_size
# ... 81 lines omitted for brevity ...
    img_context_token_id = hf_model.tokenizer.convert_tokens_to_ids("<IMG_CONTEXT>")
    hf_model.model.img_context_token_id = img_context_token_id
    hf_model.processor = InternVLProcessor(hf_model)
    hf_model.model.get_output_embeddings = (
        lambda: hf_model.model.language_model.get_output_embeddings()
    )
    hf_model.model.generate = types.MethodType(_internvl_generate, hf_model.model)
    return hf_model
```
**EN:** This helper encapsulates reusable logic in `internvl_patch_hf_runner`. Key inputs are `hf_model`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `internvl_patch_hf_runner` 中。 关键输入包括 `hf_model`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _internvl_generate (L908-L947)
```python
def _internvl_generate(
    self,
    pixel_values: torch.FloatTensor,
    input_ids: torch.FloatTensor,
    attention_mask: torch.LongTensor | None = None,
    **generate_kwargs,
) -> torch.LongTensor:
    """Generate method for InternVL2 model without fixed use_cache."""
    assert self.img_context_token_id is not None
    target_dtype = next(self.parameters()).dtype
    vit_embeds = self.extract_feature(pixel_values.to(target_dtype))
    input_embeds = self.language_model.get_input_embeddings()(input_ids)
    B, N, C = input_embeds.shape
    input_embeds = input_embeds.reshape(B * N, C)

    input_ids = input_ids.reshape(B * N)
    selected = input_ids == self.img_context_token_id
    assert selected.sum() != 0
# ... 14 lines omitted for brevity ...
        pytest.skip("HF impl is not compatible with current transformers")

    outputs = self.language_model.generate(
        **forward_kwargs,
        **generate_kwargs,
    )

    return outputs
```
**EN:** This helper encapsulates reusable logic in `_internvl_generate`. Key inputs are `pixel_values`, `input_ids`, `attention_mask`. The logic can skip unsupported environments when prerequisites are not satisfied. It returns computed state or helper objects back to the caller. The main assertion is `self.img_context_token_id is not None` and `selected.sum() != 0`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_internvl_generate` 中。 关键输入包括 `pixel_values`、`input_ids`、`attention_mask`。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `self.img_context_token_id is not None` and `selected.sum() != 0`。

### Helper / 辅助函数: mantis_patch_hf_runner (L950-L970)
```python
def mantis_patch_hf_runner(hf_model: HfRunner) -> HfRunner:
    from mantis.models.mllava import MLlavaProcessor

    hf_model.processor = MLlavaProcessor.from_pretrained(hf_model.model_name)

    orig_generate = hf_model.model.generate
    tokenizer = hf_model.processor.tokenizer

    def _generate(self, *args, **kwargs):
        return orig_generate(
            *args,
            **kwargs,
            eos_token_id=[
                tokenizer.eos_token_id,
                tokenizer.convert_tokens_to_ids("<|eot_id|>"),
            ],
        )

    hf_model.model.generate = types.MethodType(_generate, hf_model.model)

    return hf_model
```
**EN:** This helper encapsulates reusable logic in `mantis_patch_hf_runner`. Key inputs are `hf_model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `mantis_patch_hf_runner` 中。 关键输入包括 `hf_model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: minicpmv_25_patch_hf_runner (L973-L1001)
```python
def minicpmv_25_patch_hf_runner(hf_model: HfRunner) -> HfRunner:
    orig_generate = hf_model.model.generate

    def _generate(
        self,
        *args,
        input_ids=None,
        pixel_values=None,
        image_sizes=None,
        image_bound=None,
        tgt_sizes=None,
        **kwargs,
    ):
        model_inputs = {
            "input_ids": input_ids,
            "pixel_values": pixel_values,
            "image_sizes": image_sizes,
            "image_bound": image_bound,
            "tgt_sizes": tgt_sizes,
        }
        for k in list(model_inputs.keys()):
            if model_inputs[k] is None:
                model_inputs.pop(k)

        return orig_generate(model_inputs, *args, decode_text=False, **kwargs)

    hf_model.model.generate = types.MethodType(_generate, hf_model.model)

    return hf_model
```
**EN:** This helper encapsulates reusable logic in `minicpmv_25_patch_hf_runner`. Key inputs are `hf_model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `minicpmv_25_patch_hf_runner` 中。 关键输入包括 `hf_model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: minicpmo_26_patch_hf_runner (L1004-L1012)
```python
def minicpmo_26_patch_hf_runner(hf_model: HfRunner) -> HfRunner:
    orig_generate = hf_model.model.generate

    def _generate(self, *args, image_sizes=None, **kwargs):
        return orig_generate(*args, decode_text=False, **kwargs)

    hf_model.model.generate = types.MethodType(_generate, hf_model.model)

    return hf_model
```
**EN:** This helper encapsulates reusable logic in `minicpmo_26_patch_hf_runner`. Key inputs are `hf_model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `minicpmo_26_patch_hf_runner` 中。 关键输入包括 `hf_model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: minicpmv_26_patch_hf_runner (L1015-L1023)
```python
def minicpmv_26_patch_hf_runner(hf_model: HfRunner) -> HfRunner:
    orig_generate = hf_model.model.generate

    def _generate(self, *args, image_sizes=None, **kwargs):
        return orig_generate(*args, decode_text=False, **kwargs)

    hf_model.model.generate = types.MethodType(_generate, hf_model.model)

    return hf_model
```
**EN:** This helper encapsulates reusable logic in `minicpmv_26_patch_hf_runner`. Key inputs are `hf_model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `minicpmv_26_patch_hf_runner` 中。 关键输入包括 `hf_model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: minimax_vl_01_patch_hf_runner (L1026-L1034)
```python
def minimax_vl_01_patch_hf_runner(hf_model: HfRunner) -> HfRunner:
    orig_generate = hf_model.model.generate

    def _generate(self, *args, image_sizes=None, **kwargs):
        return orig_generate(*args, decode_text=False, **kwargs)

    hf_model.model.generate = types.MethodType(_generate, hf_model.model)

    return hf_model
```
**EN:** This helper encapsulates reusable logic in `minimax_vl_01_patch_hf_runner`. Key inputs are `hf_model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `minimax_vl_01_patch_hf_runner` 中。 关键输入包括 `hf_model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: molmo_patch_hf_runner (L1037-L1066)
```python
def molmo_patch_hf_runner(hf_model: HfRunner) -> HfRunner:
    """Patches and returns an instance of the HfRunner to use for Molmo."""
    hf_processor = hf_model.processor

    def _processor(*args, **kwargs):
        return hf_processor.process(*args, **kwargs)

    hf_model.processor = _processor

    def _generate(self, max_new_tokens=None, do_sample=None, **kwargs):
        batch = {
            k: kwargs.pop(k).unsqueeze(0)
            for k in ("input_ids", "images", "image_input_idx", "image_masks")
            if k in kwargs
        }
        batch = BatchFeature(batch).to(dtype=self.dtype)

        return self.generate_from_batch(
            batch,
            generation_config=GenerationConfig(
                max_new_tokens=max_new_tokens,
                stop_strings="<|endoftext|>",
                do_sample=do_sample,
            ),
            **kwargs,
        )

    hf_model.model.generate = types.MethodType(_generate, hf_model.model)

    return hf_model
```
**EN:** This helper encapsulates reusable logic in `molmo_patch_hf_runner`. Key inputs are `hf_model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `molmo_patch_hf_runner` 中。 关键输入包括 `hf_model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: ovis_patch_hf_runner (L1069-L1102)
```python
def ovis_patch_hf_runner(hf_model: HfRunner) -> HfRunner:
    """Patches and returns an instance of the HfRunner to use for Ovis2."""
    hf_model.model.get_output_embeddings = (
        lambda: hf_model.model.llm.get_output_embeddings()
    )

    def processor(*args, text="", images=None, **kwargs):
        text_tokenizer = hf_model.model.get_text_tokenizer()
        images = [images] if isinstance(images, Image) else images

        prompt_start_and_end = {
            "qwen2": ("<|im_start|>user\n", "<|im_end|>\n"),
            "llama": ("<|start_header_id|>user<|end_header_id|>\n\n", "<|eot_id|>"),
            "gemma2": ("<start_of_turn>user\n", "<end_of_turn>\n"),
        }
        for start, end in prompt_start_and_end.values():
            if start in text and end in text:
                text = text.split(start)[1].split(end)[0]
# ... 8 lines omitted for brevity ...
            "inputs": input_ids.unsqueeze(0),
            "pixel_values": pixel_values.unsqueeze(0),
            "attention_mask": attention_mask.unsqueeze(0),
        }
        return BatchFeature(data=inputs, tensor_type="pt")

    hf_model.processor = processor
    return hf_model
```
**EN:** This helper encapsulates reusable logic in `ovis_patch_hf_runner`. Key inputs are `hf_model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `ovis_patch_hf_runner` 中。 关键输入包括 `hf_model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: ovis2_5_patch_hf_runner (L1105-L1157)
```python
def ovis2_5_patch_hf_runner(hf_model: HfRunner) -> HfRunner:
    """Patches and returns an instance of the HfRunner to use for Ovis2."""
    hf_model.model.get_output_embeddings = (
        lambda: hf_model.model.llm.get_output_embeddings()
    )

    def processor(*args, text="", images=None, videos=None, **kwargs):
        if images is None:
            images = []
        else:
            images = [images] if isinstance(images, Image) else images
        if videos is None:
            videos = []
        else:
            videos = [videos] if isinstance(videos, np.ndarray) else videos
            videos = [[PIL.Image.fromarray(frame) for frame in vid] for vid in videos]

        prompt_start_and_end = {
# ... 27 lines omitted for brevity ...
            "inputs": input_ids,
            "pixel_values": pixel_values,
            "grid_thws": grid_thws,
        }
        return BatchFeature(data=inputs, tensor_type="pt")

    hf_model.processor = processor
    return hf_model
```
**EN:** This helper encapsulates reusable logic in `ovis2_5_patch_hf_runner`. Key inputs are `hf_model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `ovis2_5_patch_hf_runner` 中。 关键输入包括 `hf_model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: paddleocr_vl_patch_hf_runner (L1160-L1182)
```python
def paddleocr_vl_patch_hf_runner(hf_model: HfRunner) -> HfRunner:
    """Patches the HfRunner to fix create_causal_mask API mismatch.

    The PaddleOCR-VL HF model passes `inputs_embeds` to create_causal_mask,
    but transformers renamed this parameter to `input_embeds`.
    """
    import sys

    model_module = sys.modules.get(type(hf_model.model.model).__module__)
    if model_module is None:
        return hf_model

    original_create_causal_mask = getattr(model_module, "create_causal_mask", None)
    if original_create_causal_mask is None:
        return hf_model

    def patched_create_causal_mask(*args, **kwargs):
        if "inputs_embeds" in kwargs:
            kwargs["input_embeds"] = kwargs.pop("inputs_embeds")
        return original_create_causal_mask(*args, **kwargs)

    model_module.create_causal_mask = patched_create_causal_mask  # type: ignore[attr-defined]
    return hf_model
```
**EN:** This helper encapsulates reusable logic in `paddleocr_vl_patch_hf_runner`. Key inputs are `hf_model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `paddleocr_vl_patch_hf_runner` 中。 关键输入包括 `hf_model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: qwen2_5_omni_patch_hf_runner (L1185-L1190)
```python
def qwen2_5_omni_patch_hf_runner(hf_model: HfRunner) -> HfRunner:
    """Patches and returns an instance of the HfRunner for Qwen2.5-Omni."""
    thinker = hf_model.model.thinker
    thinker.get_output_embeddings = lambda: thinker.lm_head
    hf_model.model = thinker
    return hf_model
```
**EN:** This helper encapsulates reusable logic in `qwen2_5_omni_patch_hf_runner`. Key inputs are `hf_model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `qwen2_5_omni_patch_hf_runner` 中。 关键输入包括 `hf_model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: qwen3_vl_patch_hf_runner (L1193-L1238)
```python
def qwen3_vl_patch_hf_runner(hf_model: HfRunner) -> HfRunner:
    """Patches and returns an instance of the HfRunner to use for GLM4.1V."""
    hf_processor = hf_model.processor

    def processor(*args, videos=None, **kwargs):
        if videos is not None and is_list_of(videos, tuple):
            # batched multi videos
            do_sample_frames = {video[1]["do_sample_frames"] for video in videos}
            assert len(do_sample_frames) == 1
            if kwargs.get("do_sample_frames") is None:
                kwargs["do_sample_frames"] = do_sample_frames
            video_metadata = [
                [
                    VideoMetadata(
                        **{k: v for k, v in video[1].items() if k != "do_sample_frames"}
                    )
                ]
                for video in videos
# ... 20 lines omitted for brevity ...
            video_metadata = None

        return hf_processor(
            *args, videos=videos, video_metadata=video_metadata, **kwargs
        )

    hf_model.processor = processor
    return hf_model
```
**EN:** This helper encapsulates reusable logic in `qwen3_vl_patch_hf_runner`. Key inputs are `hf_model`. It returns computed state or helper objects back to the caller. The main assertion is `len(do_sample_frames) == 1`.
**CN:** 这个辅助函数将可复用逻辑封装在 `qwen3_vl_patch_hf_runner` 中。 关键输入包括 `hf_model`。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `len(do_sample_frames) == 1`。

### Helper / 辅助函数: tarsier_patch_hf_runner (L1241-L1250)
```python
def tarsier_patch_hf_runner(hf_model: HfRunner) -> HfRunner:
    from vllm.model_executor.models.tarsier import get_vision_encoder_info

    vision_encoder_info = get_vision_encoder_info(hf_model.config)

    hf_processor = hf_model.processor
    if hf_processor.patch_size is None:
        hf_processor.patch_size = vision_encoder_info.get_patch_size()

    return hf_model
```
**EN:** This helper encapsulates reusable logic in `tarsier_patch_hf_runner`. Key inputs are `hf_model`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `tarsier_patch_hf_runner` 中。 关键输入包括 `hf_model`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: voxtral_patch_hf_runner (L1253-L1338)
```python
def voxtral_patch_hf_runner(hf_model: "HfRunner") -> "HfRunner":
    """Patch HfRunner for Voxtral's conversation-based processor.

    Two issues in HfRunner require patching:

    1. VoxtralProcessor requires ``apply_chat_template()`` with conversation
       dicts (accepting ``url``, ``path``, or ``base64`` audio) rather than
       the standard ``processor(text=, audio=, sampling_rate=)`` interface.
    2. HfRunner.get_inputs cannot handle multi-audio per prompt because it
       incorrectly unpacks ``[(arr1, sr1), (arr2, sr2)]`` via a ``len == 2`` check.

    We override ``get_inputs`` to build conversation dicts and call
    ``apply_chat_template`` directly, bypassing both issues. We also wrap
    ``model.generate`` to strip prompt tokens before decoding, since
    HfRunner.generate calls batch_decode on the full sequence (prompt +
    generated).
    """

# ... 60 lines omitted for brevity ...
                # scores/logits so generate_greedy_logprobs_limit can
                # extract per-token logprobs.
                output.sequences = output.sequences[:, prompt_len:]
        return output

    hf_model.get_inputs = patched_get_inputs  # type: ignore[method-assign, assignment]
    hf_model.model.generate = patched_generate  # type: ignore[method-assign]
    return hf_model
```
**EN:** This helper encapsulates reusable logic in `voxtral_patch_hf_runner`. Key inputs are `hf_model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `voxtral_patch_hf_runner` 中。 关键输入包括 `hf_model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: moondream3_processor (L1341-L1344)
```python
def moondream3_processor(model: str):
    from vllm.transformers_utils.processors.moondream3 import Moondream3Processor

    return Moondream3Processor.from_pretrained(model, trust_remote_code=True)
```
**EN:** This helper encapsulates reusable logic in `moondream3_processor`. Key inputs are `model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `moondream3_processor` 中。 关键输入包括 `model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: moondream3_patch_hf_runner (L1347-L1556)
```python
def moondream3_patch_hf_runner(hf_model: HfRunner) -> HfRunner:
    """Patch HfRunner for Moondream3."""
    moondream_processor = hf_model.processor

    def processor(*args, text="", images=None, **kwargs):
        if images is None:
            return moondream_processor(text=text, **kwargs)

        images_list = [images] if isinstance(images, Image) else images
        return moondream_processor(images=images_list, text=text, **kwargs)

    hf_model.processor = processor

    # Expose the LM head for logprob extraction.
    hf_model.model.get_output_embeddings = lambda: hf_model.model.model.text.lm_head

    native_model = hf_model.model.model  # MoondreamModel instance

# ... 184 lines omitted for brevity ...
                return types.SimpleNamespace(
                    sequences=sequences,
                    hidden_states=tuple(all_hidden_states) if output_hs else None,
                )
            return sequences

    hf_model.model.generate = types.MethodType(_generate, hf_model.model)
    return hf_model
```
**EN:** This helper encapsulates reusable logic in `moondream3_patch_hf_runner`. Key inputs are `hf_model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `moondream3_patch_hf_runner` 中。 关键输入包括 `hf_model`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: qianfan_ocr_hf_model_kwargs (L1559-L1569)
```python
def qianfan_ocr_hf_model_kwargs(model_name: str) -> dict:
    """Return hf_model_kwargs with a patched config for QianfanOCR."""
    from vllm.transformers_utils.configs.qianfan_ocr import QianfanOCRConfig

    config = QianfanOCRConfig.from_pretrained(model_name)
    vc = config.vision_config
    if isinstance(vc.image_size, int):
        vc.image_size = (vc.image_size, vc.image_size)
    if isinstance(vc.patch_size, int):
        vc.patch_size = (vc.patch_size, vc.patch_size)
    return {"config": config}
```
**EN:** This helper encapsulates reusable logic in `qianfan_ocr_hf_model_kwargs`. Key inputs are `model_name`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `qianfan_ocr_hf_model_kwargs` 中。 关键输入包括 `model_name`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: qianfan_ocr_patch_hf_runner (L1572-L1647)
```python
def qianfan_ocr_patch_hf_runner(hf_model: HfRunner) -> HfRunner:
    """Patches an HfRunner instance to run QianfanOCR model inference.

    QianfanOCR shares the same architecture as InternVLChatModel, so the
    patching logic mirrors ``internvl_patch_hf_runner``.  The only difference
    is that we load the config via vllm's registered ``QianfanOCRConfig``
    instead of relying on ``trust_remote_code``.
    """

    class QianfanOCRProcessor:
        def __init__(self, hf_runner: HfRunner):
            self.tokenizer = hf_runner.tokenizer

            from vllm.transformers_utils.configs.qianfan_ocr import QianfanOCRConfig

            self.config = QianfanOCRConfig.from_pretrained(hf_runner.model_name)
            self.vision_config = self.config.vision_config
            self.use_thumbnail = self.config.use_thumbnail
# ... 50 lines omitted for brevity ...
    img_context_token_id = hf_model.tokenizer.convert_tokens_to_ids("<IMG_CONTEXT>")
    hf_model.model.img_context_token_id = img_context_token_id
    hf_model.processor = QianfanOCRProcessor(hf_model)
    hf_model.model.get_output_embeddings = (
        lambda: hf_model.model.language_model.get_output_embeddings()
    )
    hf_model.model.generate = types.MethodType(_internvl_generate, hf_model.model)
    return hf_model
```
**EN:** This helper encapsulates reusable logic in `qianfan_ocr_patch_hf_runner`. Key inputs are `hf_model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `qianfan_ocr_patch_hf_runner` 中。 关键输入包括 `hf_model`。 它把计算得到的状态或辅助对象返回给调用方。

## Key Concepts / 关键概念
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。
- **EN:** Shared metadata and helper utilities keep nearby tests focused on behavior.
  **CN:** 共享元数据与辅助工具让相邻测试更专注于行为本身。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `io`, `logging`, `pathlib.PosixPath`, `sys`, `types`, `warnings`
- **Third-party / 第三方**: `PIL.Image`, `PIL.Image.Image`, `mantis.models.mllava.MLlavaProcessor`, `numpy`, `numpy.typing`, `perceptron.tensorstream.TextType`, `perceptron.tensorstream.ops.compute_mrope_pos_tensor`, `perceptron.tensorstream.ops.modality_mask`, `pybase64`, `pytest`, `regex`, `soundfile`, `torch`, `torch.nn.functional`, `transformers.AutoConfig`, `transformers.AutoTokenizer`, `transformers.BatchFeature`, `transformers.GenerationConfig`, `transformers.GenerationMixin`, `transformers.masking_utils.create_causal_mask`, `transformers.modeling_outputs.BaseModelOutputWithPast`, `transformers.video_utils.VideoMetadata`
- **Project / 项目内**: `vllm.logprobs.SampleLogprobs`, `vllm.model_executor.models.moondream3.reconstruct_from_crops`, `vllm.model_executor.models.tarsier.get_vision_encoder_info`, `vllm.platforms.current_platform`, `vllm.transformers_utils.configs.qianfan_ocr.QianfanOCRConfig`, `vllm.transformers_utils.processors.h2ovl.image_to_pixel_values_h2ovl`, `vllm.transformers_utils.processors.internvl.image_to_pixel_values_internvl`, `vllm.transformers_utils.processors.internvl.video_to_pixel_values_internvl`, `vllm.transformers_utils.processors.moondream3.Moondream3Processor`, `vllm.utils.collection_utils.is_list_of`
- **Local relative imports / 本地相对导入**: `.....conftest.HfRunner`, `.....conftest.ImageAsset`, `.....conftest.ImageTestAssets`, `...conftest.patch_hf_vision_attn_for_rocm`, `.types.RunnerOutput`
