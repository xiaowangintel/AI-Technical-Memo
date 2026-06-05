# test_common.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/processing/test_common.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers multimodal processing and model-facing behavior. The file defines 1 test(s), 0 fixture(s), and 12 helper/class block(s) to validate this area. / [CN] 该文件覆盖多模态处理与面向模型的行为。它定义了 1 个测试、0 个 fixture，以及 12 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L31)
```python
from collections.abc import Set as AbstractSet
from functools import partial

import numpy as np
import pytest
from PIL import Image

from vllm.config import ModelConfig
from vllm.config.multimodal import (
    AudioDummyOptions,
    BaseDummyOptions,
    ImageDummyOptions,
    VideoDummyOptions,
)
from vllm.inputs import MultiModalDataDict, MultiModalInput
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.cache import MultiModalProcessorOnlyCache
from vllm.multimodal.inputs import batched_tensors_equal
# ... 6 lines omitted for brevity ...
    _MULTIMODAL_EXAMPLE_MODELS,
    _TRANSFORMERS_BACKEND_MODELS,
    HF_EXAMPLE_MODELS,
)
```
**EN:** Imports standard-library modules such as `collections.abc.Set`, `functools.partial`, third-party packages like `PIL.Image`, `numpy`, `pytest`, project helpers such as `vllm.config.ModelConfig`, `vllm.config.multimodal.AudioDummyOptions`, `vllm.config.multimodal.BaseDummyOptions`.
**CN:** 导入标准库模块（如 `collections.abc.Set`、`functools.partial`）、第三方包（如 `PIL.Image`、`numpy`、`pytest`）、项目内辅助模块（如 `vllm.config.ModelConfig`、`vllm.config.multimodal.AudioDummyOptions`、`vllm.config.multimodal.BaseDummyOptions`）。

### Helper / 辅助函数: add_video_metadata (L34-L59)
```python
def add_video_metadata(mm_data: MultiModalDataDict) -> MultiModalDataDict:
    """
    Add metadata to video mm_data
    """

    def create_metadata(frames: np.ndarray):
        num_frames = len(frames)
        return {
            "total_num_frames": num_frames,
            "fps": 2.0,
            "duration": num_frames / 2.0,
            "video_backend": "opencv",
            "frames_indices": list(range(num_frames)),
            "do_sample_frames": True,
        }

    # Ensure video metadata is included
    if "video" in mm_data:
        video = mm_data["video"]
        if isinstance(video, list):
            # multiple videos
            mm_data["video"] = [(vid, create_metadata(vid)) for vid in video]
        else:
            # single video
            mm_data["video"] = (video, create_metadata(video))
    return mm_data
```
**EN:** This helper encapsulates reusable logic in `add_video_metadata`. Key inputs are `mm_data`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `add_video_metadata` 中。 关键输入包括 `mm_data`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: glmasr_patch_mm_data (L62-L72)
```python
def glmasr_patch_mm_data(mm_data: MultiModalDataDict) -> MultiModalDataDict:
    """
    Patch the multimodal data for GLM-ASR model.
    GLM-ASR requires text and audio to match 1:1, so we limit audio to 1.
    """
    if "audio" in mm_data:
        audio = mm_data["audio"]
        if isinstance(audio, list) and len(audio) > 1:
            # Limit to single audio to match text requirement
            mm_data["audio"] = [audio[0]]
    return mm_data
```
**EN:** This helper encapsulates reusable logic in `glmasr_patch_mm_data`. Key inputs are `mm_data`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `glmasr_patch_mm_data` 中。 关键输入包括 `mm_data`。 它把计算得到的状态或辅助对象返回给调用方。

### Module setup / 模块级配置: _IGNORE_MM_KEYS, MM_DATA_PATCHES (L75-L84)
```python
_IGNORE_MM_KEYS = {
    # In Ultravox, the audio_features can be different depending on padding
    # The slight difference should not be a problem though, since
    # attention_mask lets us ignore the difference.
    "ultravox": {"audio_features"},
}

MM_DATA_PATCHES = {
    "glmasr": glmasr_patch_mm_data,
}
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `_IGNORE_MM_KEYS`, `MM_DATA_PATCHES`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `_IGNORE_MM_KEYS`、`MM_DATA_PATCHES`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: _iter_model_ids_to_test (L87-L96)
```python
def _iter_model_ids_to_test(model_arch_list: AbstractSet[str]):
    for model_arch in model_arch_list:
        model_info = HF_EXAMPLE_MODELS.get_hf_info(model_arch)
        yield model_info.default

        for extra_type, extra_model_id in model_info.extras.items():
            if "fp" in extra_type:
                continue  # Redundant to test quantized models

            yield extra_model_id
```
**EN:** This helper encapsulates reusable logic in `_iter_model_ids_to_test`. Key inputs are `model_arch_list`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_iter_model_ids_to_test` 中。 关键输入包括 `model_arch_list`。

### Helper / 辅助函数: _get_model_ids_to_test (L99-L100)
```python
def _get_model_ids_to_test(model_arch_list: AbstractSet[str]):
    return list(_iter_model_ids_to_test(model_arch_list))
```
**EN:** This helper encapsulates reusable logic in `_get_model_ids_to_test`. Key inputs are `model_arch_list`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_get_model_ids_to_test` 中。 关键输入包括 `model_arch_list`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: get_model_ids_to_test (L103-L118)
```python
def get_model_ids_to_test():
    transformers_arch_ids = {
        model_id
        for info in _TRANSFORMERS_BACKEND_MODELS.values()
        for model_id in (info.default, *info.extras.values())
    }
    vllm_only_archs = {
        arch
        for arch, info in _MULTIMODAL_EXAMPLE_MODELS.items()
        if not any(
            model_id in transformers_arch_ids
            for model_id in (info.default, *info.extras.values())
        )
    }

    return _get_model_ids_to_test(vllm_only_archs)
```
**EN:** This helper encapsulates reusable logic in `get_model_ids_to_test`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `get_model_ids_to_test` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: get_text_token_prompts (L121-L168)
```python
def get_text_token_prompts(
    processor: BaseMultiModalProcessor,
    mm_data: MultiModalDataDict,
):
    dummy_inputs = processor.dummy_inputs
    tokenizer: TokenizerLike = processor.info.get_tokenizer()
    model_config = processor.info.ctx.model_config

    if processor.info.data_parser.video_needs_metadata:
        mm_data = add_video_metadata(mm_data)

    model_type = model_config.hf_config.model_type
    if model_type in MM_DATA_PATCHES:
        mm_data = MM_DATA_PATCHES[model_type](mm_data)

    parsed_data = processor.info.parse_mm_data(mm_data)
    mm_counts = {k: len(vs) for k, vs in parsed_data.items()}

# ... 22 lines omitted for brevity ...
        token_prompt = tokenizer.encode(
            text_prompt,
            **processor.info.get_default_tok_params().get_encode_kwargs(),
        )
    else:
        raise TypeError(type(inputs.prompt))

    return text_prompt, token_prompt
```
**EN:** This helper encapsulates reusable logic in `get_text_token_prompts`. Key inputs are `processor`, `mm_data`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `get_text_token_prompts` 中。 关键输入包括 `processor`、`mm_data`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: random_vision_chunk (L171-L190)
```python
def random_vision_chunk(
    rng: np.random.RandomState,
    min_wh: int,
    max_wh: int,
    min_frames: int,
    max_frames: int,
) -> dict:
    num_frames = rng.randint(min_frames, max_frames + 1)
    if num_frames == 1:
        # Single image chunk
        wh = rng.randint(min_wh, max_wh + 1)
        image = random_image(rng, wh, wh + 1)
        return {"type": "image", "image": image}
    frames = []
    for _ in range(num_frames):
        wh = rng.randint(min_wh, max_wh + 1)
        frame = rng.randint(0, 256, size=(wh, wh, 3), dtype=np.uint8)
        frames.append(frame)
    video_array = np.stack(frames, axis=0)
    return {"type": "video_chunk", "video_chunk": video_array}
```
**EN:** This helper encapsulates reusable logic in `random_vision_chunk`. Key inputs are `rng`, `min_wh`, `max_wh`, `min_frames`, `max_frames`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `random_vision_chunk` 中。 关键输入包括 `rng`、`min_wh`、`max_wh`、`min_frames`、`max_frames`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _test_processing_correctness (L193-L317)
```python
def _test_processing_correctness(
    model_id_or_arch: str,
    hit_rate: float,
    num_batches: int,
    simplify_rate: float,
):
    if model_id_or_arch in HF_EXAMPLE_MODELS.get_supported_archs():
        # Use model architecture to get the default model id
        model_info = HF_EXAMPLE_MODELS.get_hf_info(model_id_or_arch)
        model_id = model_info.default
    else:
        model_info = HF_EXAMPLE_MODELS.find_hf_info(model_id_or_arch)
        model_id = model_id_or_arch
    model_info.check_available_online(on_fail="skip")
    model_info.check_transformers_version(
        on_fail="skip",
        check_max_version=False,
        check_version_reason="vllm",
# ... 99 lines omitted for brevity ...
            mm_data,
            baseline_processor,
            cached_processor,
            batch_idx,
            hit_rate,
            num_batches,
            simplify_rate,
        )
```
**EN:** This helper encapsulates reusable logic in `_test_processing_correctness`. Key inputs are `model_id_or_arch`, `hit_rate`, `num_batches`, `simplify_rate`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_test_processing_correctness` 中。 关键输入包括 `model_id_or_arch`、`hit_rate`、`num_batches`、`simplify_rate`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _test_processing_correctness_one (L320-L402)
```python
def _test_processing_correctness_one(
    model_config: ModelConfig,
    mm_data: MultiModalDataDict,
    baseline_processor: BaseMultiModalProcessor,
    cached_processor: BaseMultiModalProcessor,
    batch_idx: int,
    hit_rate: float,
    num_batches: int,
    simplify_rate: float,
):
    model_type = model_config.hf_config.model_type

    text_prompt, token_prompt = get_text_token_prompts(baseline_processor, mm_data)
    mm_items = baseline_processor.info.parse_mm_data(mm_data)
    ignore_mm_keys = _IGNORE_MM_KEYS.get(model_type, set[str]())

    baseline_tokenized_result = baseline_processor(
        token_prompt,
# ... 57 lines omitted for brevity ...
            cached_tokenized_result,
            ignore_mm_keys=ignore_mm_keys,
            msg=(
                f"Failed ({batch_idx=}, {hit_rate=}, "
                f"{num_batches=}, {simplify_rate=}, "
                f"{text_prompt=}, {token_prompt=}, {mm_data=})"
            ),
        )
```
**EN:** This helper encapsulates reusable logic in `_test_processing_correctness_one`. Key inputs are `model_config`, `mm_data`, `baseline_processor`, `cached_processor`, `batch_idx`, `hit_rate`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_test_processing_correctness_one` 中。 关键输入包括 `model_config`、`mm_data`、`baseline_processor`、`cached_processor`、`batch_idx`、`hit_rate`。

### Test / 测试: test_processing_correctness (L405-L441)
```python
@pytest.mark.parametrize("model_id", get_model_ids_to_test())
@pytest.mark.parametrize("hit_rate", [0.3, 0.5, 1.0])
@pytest.mark.parametrize("num_batches", [32])
@pytest.mark.parametrize("simplify_rate", [1.0])
def test_processing_correctness(
    model_id: str,
    hit_rate: float,
    num_batches: int,
    simplify_rate: float,
):
    if model_id == "google/gemma-3n-E2B-it":
        pytest.skip("Fix later")
    if model_id == "OpenGVLab/InternVL2-2B":
        pytest.skip("Fix later")
    if model_id == "jinaai/jina-reranker-m0":
        pytest.skip("Fix later")
    if model_id in {"Qwen/Qwen-VL", "Qwen/Qwen-VL-Chat"}:
        pytest.skip(
# ... 11 lines omitted for brevity ...
        pytest.skip("Fix later")

    _test_processing_correctness(
        model_id,
        hit_rate=hit_rate,
        num_batches=num_batches,
        simplify_rate=simplify_rate,
    )
```
**EN:** This test validates `test_processing_correctness`. It uses parameterization over `model_id`. Key inputs are `model_id`, `hit_rate`, `num_batches`, `simplify_rate`. The logic can skip unsupported environments when prerequisites are not satisfied.
**CN:** 这个测试验证 `test_processing_correctness`。 它通过参数化组合 `model_id`。 关键输入包括 `model_id`、`hit_rate`、`num_batches`、`simplify_rate`。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。

### Helper / 辅助函数: _assert_inputs_equal (L444-L467)
```python
def _assert_inputs_equal(
    a: MultiModalInput,
    b: MultiModalInput,
    *,
    ignore_mm_keys: set[str] | None = None,
    msg: str = "",
):
    if ignore_mm_keys is None:
        ignore_mm_keys = set()

    ignore_prompt_keys = ("prompt", "mm_kwargs")
    a_rest = {k: v for k, v in a.items() if k not in ignore_prompt_keys}
    b_rest = {k: v for k, v in b.items() if k not in ignore_prompt_keys}

    assert a_rest == b_rest, msg

    a_data = a["mm_kwargs"].get_data()
    b_data = b["mm_kwargs"].get_data()

    for key in ignore_mm_keys:
        a_data.pop(key, None)
        b_data.pop(key, None)

    assert batched_tensors_equal(a_data, b_data), msg
```
**EN:** This helper encapsulates reusable logic in `_assert_inputs_equal`. Key inputs are `a`, `b`. The main assertion is `a_rest == b_rest` and `batched_tensors_equal(a_data, b_data)`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_assert_inputs_equal` 中。 关键输入包括 `a`、`b`。 核心断言是 `a_rest == b_rest` and `batched_tensors_equal(a_data, b_data)`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc.Set`, `functools.partial`
- **Third-party / 第三方**: `PIL.Image`, `numpy`, `pytest`
- **Project / 项目内**: `vllm.config.ModelConfig`, `vllm.config.multimodal.AudioDummyOptions`, `vllm.config.multimodal.BaseDummyOptions`, `vllm.config.multimodal.ImageDummyOptions`, `vllm.config.multimodal.VideoDummyOptions`, `vllm.inputs.MultiModalDataDict`, `vllm.inputs.MultiModalInput`, `vllm.multimodal.MULTIMODAL_REGISTRY`, `vllm.multimodal.cache.MultiModalProcessorOnlyCache`, `vllm.multimodal.inputs.batched_tensors_equal`, `vllm.multimodal.processing.BaseMultiModalProcessor`, `vllm.multimodal.processing.InputProcessingContext`, `vllm.tokenizers.TokenizerLike`, `vllm.tokenizers.cached_tokenizer_from_config`, `vllm.utils.mistral.is_mistral_tokenizer`
- **Local relative imports / 本地相对导入**: `....multimodal.utils.random_audio`, `....multimodal.utils.random_image`, `....multimodal.utils.random_video`, `...registry.HF_EXAMPLE_MODELS`, `...registry._MULTIMODAL_EXAMPLE_MODELS`, `...registry._TRANSFORMERS_BACKEND_MODELS`
