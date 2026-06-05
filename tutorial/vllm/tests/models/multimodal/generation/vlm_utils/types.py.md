# types.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/generation/vlm_utils/types.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides shared test utilities for multimodal processing and model-facing behavior. The file exposes 8 helper/class block(s) used by nearby tests. / [CN] 为多模态处理与面向模型的行为提供共享测试工具。该文件暴露了 8 个辅助函数/类块，供附近测试复用。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L5-L29)
```python
from collections.abc import Callable, Iterable
from enum import Enum
from pathlib import PosixPath
from typing import Any, NamedTuple

import torch
from pytest import MarkDecorator
from transformers import AutoModelForCausalLM
from transformers.models.auto.auto_factory import _BaseAutoModelClass

from vllm.config.model import RunnerOption
from vllm.logprobs import SampleLogprobs
from vllm.tokenizers import TokenizerLike

from .....conftest import (
    AUDIO_ASSETS,
    IMAGE_ASSETS,
    HfRunner,
# ... 3 lines omitted for brevity ...
    PromptImageInput,
    PromptVideoInput,
)
from ....utils import check_logprobs_close
```
**EN:** Imports standard-library modules such as `collections.abc.Callable`, `collections.abc.Iterable`, `enum.Enum`, third-party packages like `pytest.MarkDecorator`, `torch`, `transformers.AutoModelForCausalLM`, project helpers such as `vllm.config.model.RunnerOption`, `vllm.logprobs.SampleLogprobs`, `vllm.tokenizers.TokenizerLike`.
**CN:** 导入标准库模块（如 `collections.abc.Callable`、`collections.abc.Iterable`、`enum.Enum`）、第三方包（如 `pytest.MarkDecorator`、`torch`、`transformers.AutoModelForCausalLM`）、项目内辅助模块（如 `vllm.config.model.RunnerOption`、`vllm.logprobs.SampleLogprobs`、`vllm.tokenizers.TokenizerLike`）。

### Module setup / 模块级配置: TEST_IMG_PLACEHOLDER, TEST_VIDEO_PLACEHOLDER, TEST_AUDIO_PLACEHOLDER (L32-L55)
```python
TEST_IMG_PLACEHOLDER = "<vlm_image>"
TEST_VIDEO_PLACEHOLDER = "<vlm_video>"
TEST_AUDIO_PLACEHOLDER = "<lmm_audio>"

SINGLE_IMAGE_BASE_PROMPTS = IMAGE_ASSETS.prompts(
    {
        "stop_sign": f"{TEST_IMG_PLACEHOLDER}What's the content of the image?",
        "cherry_blossom": f"{TEST_IMG_PLACEHOLDER}What is the season?",
    }
)
SINGLE_AUDIO_BASE_PROMPT = AUDIO_ASSETS.prompts(
    {
        "mary_had_lamb": f"{TEST_AUDIO_PLACEHOLDER}Transcribe this audio into English.",  # noqa: E501
        "winning_call": f"{TEST_AUDIO_PLACEHOLDER}What is happening in this audio clip?",  # noqa: E501
    }
)

MULTI_IMAGE_BASE_PROMPT = f"Image-1: {TEST_IMG_PLACEHOLDER}Image-2: {TEST_IMG_PLACEHOLDER}Describe the two images in detail.\n"  # noqa: E501
VIDEO_BASE_PROMPT = f"{TEST_VIDEO_PLACEHOLDER}Why is this video funny?"


IMAGE_SIZE_FACTORS = [(1.0,), (1.0, 1.0, 1.0), (0.25, 0.5, 1.0)]
EMBEDDING_SIZE_FACTORS = [(1.0,), (1.0, 1.0, 1.0)]
RunnerOutput = tuple[list[int], str, SampleLogprobs | None]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `TEST_IMG_PLACEHOLDER`, `TEST_VIDEO_PLACEHOLDER`, `TEST_AUDIO_PLACEHOLDER`, `SINGLE_IMAGE_BASE_PROMPTS`, `SINGLE_AUDIO_BASE_PROMPT`, `MULTI_IMAGE_BASE_PROMPT`, `VIDEO_BASE_PROMPT`, `IMAGE_SIZE_FACTORS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `TEST_IMG_PLACEHOLDER`、`TEST_VIDEO_PLACEHOLDER`、`TEST_AUDIO_PLACEHOLDER`、`SINGLE_IMAGE_BASE_PROMPTS`、`SINGLE_AUDIO_BASE_PROMPT`、`MULTI_IMAGE_BASE_PROMPT`、`VIDEO_BASE_PROMPT`、`IMAGE_SIZE_FACTORS`。后续辅助函数和测试会复用这些值，以减少重复。

### Class / 类: PromptWithMultiModalInput (L58-L64)
```python
class PromptWithMultiModalInput(NamedTuple):
    """Holds the multimodal input for a single test case."""

    prompts: list[str]
    image_data: PromptImageInput | None = None
    video_data: PromptVideoInput | None = None
    audio_data: PromptAudioInput | None = None
```
**EN:** This class groups related scenarios in `PromptWithMultiModalInput`. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `PromptWithMultiModalInput` 相关的场景组织在一起。 其主体主要是结构性定义，没有直接方法实现。

### Class / 类: VLMTestType (L67-L73)
```python
class VLMTestType(Enum):
    IMAGE = 1
    MULTI_IMAGE = 2
    EMBEDDING = 3
    VIDEO = 4
    AUDIO = 5
    CUSTOM_INPUTS = 6
```
**EN:** This class groups related scenarios in `VLMTestType`. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `VLMTestType` 相关的场景组织在一起。 其主体主要是结构性定义，没有直接方法实现。

### Class / 类: SizeType (L76-L78)
```python
class SizeType(Enum):
    SIZE_FACTOR = 1
    FIXED_SIZE = 2
```
**EN:** This class groups related scenarios in `SizeType`. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `SizeType` 相关的场景组织在一起。 其主体主要是结构性定义，没有直接方法实现。

### Class / 类: CustomTestOptions (L81-L83)
```python
class CustomTestOptions(NamedTuple):
    inputs: list[PromptWithMultiModalInput]
    limit_mm_per_prompt: dict[str, int]
```
**EN:** This class groups related scenarios in `CustomTestOptions`. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `CustomTestOptions` 相关的场景组织在一起。 其主体主要是结构性定义，没有直接方法实现。

### Class / 类: ImageSizeWrapper (L86-L90)
```python
class ImageSizeWrapper(NamedTuple):
    type: SizeType
    # A size factor is a wrapper of 0+ floats,
    # while a fixed size contains an iterable of integer pairs
    data: Iterable[float] | Iterable[tuple[int, int]]
```
**EN:** This class groups related scenarios in `ImageSizeWrapper`. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `ImageSizeWrapper` 相关的场景组织在一起。 其主体主要是结构性定义，没有直接方法实现。

### Class / 类: VLMTestInfo (L93-L203)
```python
class VLMTestInfo(NamedTuple):
    """Holds the configuration for 1+ tests for one model architecture."""

    models: list[str]
    test_type: VLMTestType | Iterable[VLMTestType]

    # Should be None only if this is a CUSTOM_INPUTS test
    prompt_formatter: Callable[[str], str] | None = None
    img_idx_to_prompt: Callable[[int], str] = lambda idx: "<image>\n"
    video_idx_to_prompt: Callable[[int], str] = lambda idx: "<video>\n"
    audio_idx_to_prompt: Callable[[int], str] = lambda idx: "<audio>\n"

    # Most models work on the single / multi-image prompts above, but in some
    # cases the log prob check fails, e.g., for paligemma. We allow passing
# ... 89 lines omitted for brevity ...
            "use_tokenizer_eos": self.use_tokenizer_eos,
            "comparator": self.comparator,
            "get_stop_token_ids": self.get_stop_token_ids,
            "hf_model_kwargs": self.hf_model_kwargs,
            "hf_processor": self.hf_processor,
            "stop_str": self.stop_str,
            "patch_hf_runner": self.patch_hf_runner,
        }
```
**EN:** This class groups related scenarios in `VLMTestInfo`. It contains 0 test method(s) and 1 supporting method(s). Representative methods include `get_non_parametrized_runner_kwargs`.
**CN:** 该类将与 `VLMTestInfo` 相关的场景组织在一起。 它包含 0 个测试方法和 1 个辅助方法。 代表性方法包括 `get_non_parametrized_runner_kwargs`。

### Helper method / 辅助方法: VLMTestInfo.get_non_parametrized_runner_kwargs (L181-L203)
```python
    def get_non_parametrized_runner_kwargs(self):
        """Returns a dictionary of expandable kwargs for items that are used
        in all test types, which are NOT used when creating the parametrized
        test cases.
        """
        return {
            "enforce_eager": self.enforce_eager,
            "max_model_len": self.max_model_len,
            "max_num_seqs": self.max_num_seqs,
            "runner": self.runner,
            "tensor_parallel_size": self.tensor_parallel_size,
            "vllm_runner_kwargs": self.vllm_runner_kwargs,
            "hf_output_post_proc": self.hf_output_post_proc,
            "vllm_output_post_proc": self.vllm_output_post_proc,
            "auto_cls": self.auto_cls,
            "use_tokenizer_eos": self.use_tokenizer_eos,
            "comparator": self.comparator,
            "get_stop_token_ids": self.get_stop_token_ids,
            "hf_model_kwargs": self.hf_model_kwargs,
            "hf_processor": self.hf_processor,
            "stop_str": self.stop_str,
            "patch_hf_runner": self.patch_hf_runner,
        }
```
**EN:** This helper encapsulates reusable logic in `VLMTestInfo.get_non_parametrized_runner_kwargs`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `VLMTestInfo.get_non_parametrized_runner_kwargs` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Class / 类: ExpandableVLMTestArgs (L206-L220)
```python
class ExpandableVLMTestArgs(NamedTuple):
    """The expanded kwargs which correspond to a single test case."""

    model: str
    max_tokens: int
    num_logprobs: int
    dtype: str
    distributed_executor_backend: str | None
    # Sizes are used for everything except for custom input tests
    size_wrapper: ImageSizeWrapper | None = None
    # Video only
    num_video_frames: int | None = None
    needs_video_metadata: bool = False
    # Custom inputs only
    custom_test_opts: CustomTestOptions | None = None
```
**EN:** This class groups related scenarios in `ExpandableVLMTestArgs`. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `ExpandableVLMTestArgs` 相关的场景组织在一起。 其主体主要是结构性定义，没有直接方法实现。

## Key Concepts / 关键概念
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc.Callable`, `collections.abc.Iterable`, `enum.Enum`, `pathlib.PosixPath`, `typing.Any`, `typing.NamedTuple`
- **Third-party / 第三方**: `pytest.MarkDecorator`, `torch`, `transformers.AutoModelForCausalLM`, `transformers.models.auto.auto_factory._BaseAutoModelClass`
- **Project / 项目内**: `vllm.config.model.RunnerOption`, `vllm.logprobs.SampleLogprobs`, `vllm.tokenizers.TokenizerLike`
- **Local relative imports / 本地相对导入**: `.....conftest.AUDIO_ASSETS`, `.....conftest.HfRunner`, `.....conftest.IMAGE_ASSETS`, `.....conftest.ImageAsset`, `.....conftest.ImageTestAssets`, `.....conftest.PromptAudioInput`, `.....conftest.PromptImageInput`, `.....conftest.PromptVideoInput`, `....utils.check_logprobs_close`
