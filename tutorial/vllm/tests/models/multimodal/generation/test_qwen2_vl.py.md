# test_qwen2_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/generation/test_qwen2_vl.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers multimodal processing and model-facing behavior. The file defines 3 test(s), 1 fixture(s), and 8 helper/class block(s) to validate this area. / [CN] 该文件覆盖多模态处理与面向模型的行为。它定义了 3 个测试、1 个 fixture，以及 8 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L21)
```python
from typing import Any, TypedDict

import numpy.typing as npt
import pytest
import torch
from PIL import Image

from vllm.multimodal.image import rescale_image_size
from vllm.multimodal.video import rescale_video_size, sample_frames_from_video

from ....conftest import (
    IMAGE_ASSETS,
    VIDEO_ASSETS,
    PromptImageInput,
    PromptVideoInput,
    VllmRunner,
)
from ...utils import check_logprobs_close
```
**EN:** Imports standard-library modules such as `typing.Any`, `typing.TypedDict`, third-party packages like `PIL.Image`, `numpy.typing`, `pytest`, project helpers such as `vllm.multimodal.image.rescale_image_size`, `vllm.multimodal.video.rescale_video_size`, `vllm.multimodal.video.sample_frames_from_video`.
**CN:** 导入标准库模块（如 `typing.Any`、`typing.TypedDict`）、第三方包（如 `PIL.Image`、`numpy.typing`、`pytest`）、项目内辅助模块（如 `vllm.multimodal.image.rescale_image_size`、`vllm.multimodal.video.rescale_video_size`、`vllm.multimodal.video.sample_frames_from_video`）。

### Fixture / 夹具: enable_pickle (L24-L27)
```python
@pytest.fixture(scope="function", autouse=True)
def enable_pickle(monkeypatch):
    """`LLM.apply_model` requires pickling a function."""
    monkeypatch.setenv("VLLM_ALLOW_INSECURE_SERIALIZATION", "1")
```
**EN:** This fixture prepares `enable_pickle` for dependent tests. Key inputs are `monkeypatch`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies.
**CN:** 这个 fixture 为依赖它的测试准备 `enable_pickle`。 关键输入包括 `monkeypatch`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。

### Module setup / 模块级配置: models, target_dtype, IMAGE_PLACEHOLDER (L30-L35)
```python
models = ["Qwen/Qwen2-VL-2B-Instruct"]
target_dtype = "half"

IMAGE_PLACEHOLDER = "<|vision_start|><|image_pad|><|vision_end|>"
VIDEO_PLACEHOLDER = "<|vision_start|><|video_pad|><|vision_end|>"
MODEL_HIDDEN_SIZE = 1536
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `models`, `target_dtype`, `IMAGE_PLACEHOLDER`, `VIDEO_PLACEHOLDER`, `MODEL_HIDDEN_SIZE`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `models`、`target_dtype`、`IMAGE_PLACEHOLDER`、`VIDEO_PLACEHOLDER`、`MODEL_HIDDEN_SIZE`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: qwen2_vl_chat_template (L38-L39)
```python
def qwen2_vl_chat_template(*query):
    return f"<|im_start|>system\nYou are a helpful assistant.<|im_end|>\n<|im_start|>user\n{''.join(query)}<|im_end|><|im_start|>assistant\n"  # noqa: E501
```
**EN:** This helper encapsulates reusable logic in `qwen2_vl_chat_template`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `qwen2_vl_chat_template` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Module setup / 模块级配置: IMAGE_PROMPTS, VIDEO_PROMPTS, MULTIIMAGE_PROMPT (L42-L72)
```python
IMAGE_PROMPTS = IMAGE_ASSETS.prompts(
    {
        "stop_sign": qwen2_vl_chat_template(
            IMAGE_PLACEHOLDER,
            "What is the biggest text's content in this image?",
        ),
        "cherry_blossom": qwen2_vl_chat_template(
            IMAGE_PLACEHOLDER,
            "What is the season shown in this image? ",
            "Reply with a short sentence (no more than 20 words)",
        ),
    }
)

VIDEO_PROMPTS = VIDEO_ASSETS.prompts(
    {
# ... 9 lines omitted for brevity ...
    IMAGE_PLACEHOLDER,
    IMAGE_PLACEHOLDER,
    "Describe these two images separately. ",
    "For each image, reply with a short sentence ",
    "(no more than 10 words).",
)
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `IMAGE_PROMPTS`, `VIDEO_PROMPTS`, `MULTIIMAGE_PROMPT`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `IMAGE_PROMPTS`、`VIDEO_PROMPTS`、`MULTIIMAGE_PROMPT`。后续辅助函数和测试会复用这些值，以减少重复。

### Class / 类: Qwen2VLPromptImageEmbeddingInput (L75-L77)
```python
class Qwen2VLPromptImageEmbeddingInput(TypedDict):
    image_embeds: torch.Tensor
    image_grid_thw: torch.Tensor
```
**EN:** This class groups related scenarios in `Qwen2VLPromptImageEmbeddingInput`. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `Qwen2VLPromptImageEmbeddingInput` 相关的场景组织在一起。 其主体主要是结构性定义，没有直接方法实现。

### Class / 类: Qwen2VLPromptVideoEmbeddingInput (L80-L82)
```python
class Qwen2VLPromptVideoEmbeddingInput(TypedDict):
    video_embeds: torch.Tensor
    video_grid_thw: torch.Tensor
```
**EN:** This class groups related scenarios in `Qwen2VLPromptVideoEmbeddingInput`. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `Qwen2VLPromptVideoEmbeddingInput` 相关的场景组织在一起。 其主体主要是结构性定义，没有直接方法实现。

### Helper / 辅助函数: batch_make_image_embeddings (L85-L168)
```python
def batch_make_image_embeddings(
    image_batches: list[Image.Image | list[Image.Image]],
    processor,
    llm: VllmRunner,
) -> list[Qwen2VLPromptImageEmbeddingInput]:
    """batched image embeddings for Qwen2-VL

    This will infer all images' embeddings in a single batch,
      and split the result according to input batches.

    image_batches:
      - Single-image batches: `list[Image.Image]`
      - Multiple-image batches: `list[list[Image.Image]]]`

    returns: `list[Qwen2VLPromptImageEmbeddingInput]`
    """

    image_batches_: list[Any] = image_batches[:]
# ... 58 lines omitted for brevity ...
        image_counter += cur_batch_image_count

    # ensure we don't lose any images or embeddings
    assert embed_counter == image_embeds.size(0)
    assert image_counter == image_grid_thw.size(0)
    assert len(image_batches) == len(result)

    return result
```
**EN:** This helper encapsulates reusable logic in `batch_make_image_embeddings`. Key inputs are `image_batches`, `processor`, `llm`. It returns computed state or helper objects back to the caller. The main assertion is `embed_counter == image_embeds.size(0)` and `image_counter == image_grid_thw.size(0)`.
**CN:** 这个辅助函数将可复用逻辑封装在 `batch_make_image_embeddings` 中。 关键输入包括 `image_batches`、`processor`、`llm`。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `embed_counter == image_embeds.size(0)` and `image_counter == image_grid_thw.size(0)`。

### Helper / 辅助函数: batch_make_video_embeddings (L171-L252)
```python
def batch_make_video_embeddings(
    video_batches: PromptVideoInput, processor, llm: VllmRunner
) -> list[Qwen2VLPromptVideoEmbeddingInput]:
    """batched video embeddings for Qwen2-VL

    A NDArray represents a single video's all frames.

    This will infer all videos' embeddings in a single batch,
      and split the result according to input batches.

    video_batches:
      - Single-video batches: `list[NDArray]`
      - Multiple-video batches: `list[list[NDArray]]`
    """

    video_batches_: list[Any] = video_batches[:]

    for idx in range(len(video_batches_)):
# ... 56 lines omitted for brevity ...
        video_counter += cur_batch_video_count

    # ensure we don't lose any videos or embeddings
    assert embed_counter == video_embeds.size(0)
    assert video_counter == video_grid_thw.size(0)
    assert len(video_batches) == len(result)

    return result
```
**EN:** This helper encapsulates reusable logic in `batch_make_video_embeddings`. Key inputs are `video_batches`, `processor`, `llm`. It returns computed state or helper objects back to the caller. The main assertion is `embed_counter == video_embeds.size(0)` and `video_counter == video_grid_thw.size(0)`.
**CN:** 这个辅助函数将可复用逻辑封装在 `batch_make_video_embeddings` 中。 关键输入包括 `video_batches`、`processor`、`llm`。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `embed_counter == video_embeds.size(0)` and `video_counter == video_grid_thw.size(0)`。

### Helper / 辅助函数: run_embedding_input_test (L255-L321)
```python
def run_embedding_input_test(
    vllm_runner: type[VllmRunner],
    inputs: list[tuple[list[str], PromptImageInput, PromptVideoInput]],
    model: str,
    *,
    dtype: str,
    max_tokens: int,
    num_logprobs: int,
    mm_limit: int,
    tensor_parallel_size: int,
    distributed_executor_backend: str | None = None,
):
    """Inference result should be the same between
    original image/video input and image/video embeddings input.
    """
    from transformers import AutoProcessor

    processor = AutoProcessor.from_pretrained(model)
# ... 41 lines omitted for brevity ...
        outputs_per_case_for_original_input, outputs_per_case_for_embeddings_input
    ):
        check_logprobs_close(
            outputs_0_lst=outputs_for_original_input,
            outputs_1_lst=outputs_for_embeddings_input,
            name_0="original_input",
            name_1="embeddings_input",
        )
```
**EN:** This helper encapsulates reusable logic in `run_embedding_input_test`. Key inputs are `vllm_runner`, `inputs`, `model`.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_embedding_input_test` 中。 关键输入包括 `vllm_runner`、`inputs`、`model`。

### Test / 测试: test_qwen2_vl_image_embeddings_input (L324-L370)
```python
@pytest.mark.core_model
@pytest.mark.parametrize("model", models)
@pytest.mark.parametrize(
    "size_factors",
    [
        # Single-scale
        [0.5],
        # Single-scale, batched
        [0.5, 0.5],
        # Multi-scale
        [0.25, 0.5, 0.5],
    ],
)
@pytest.mark.parametrize("dtype", [target_dtype])
@pytest.mark.parametrize("max_tokens", [128])
@pytest.mark.parametrize("num_logprobs", [10])
def test_qwen2_vl_image_embeddings_input(
    vllm_runner,
# ... 21 lines omitted for brevity ...
        inputs_per_case,
        model,
        dtype=dtype,
        max_tokens=max_tokens,
        num_logprobs=num_logprobs,
        mm_limit=1,
        tensor_parallel_size=1,
    )
```
**EN:** This test validates `test_qwen2_vl_image_embeddings_input`. It uses parameterization over `model`. Relevant pytest markers include `core_model`. Key inputs are `vllm_runner`, `image_assets`, `model`, `size_factors`, `dtype`, `max_tokens`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies.
**CN:** 这个测试验证 `test_qwen2_vl_image_embeddings_input`。 它通过参数化组合 `model`。 相关的 pytest 标记包括 `core_model`。 关键输入包括 `vllm_runner`、`image_assets`、`model`、`size_factors`、`dtype`、`max_tokens`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。

### Test / 测试: test_qwen2_vl_multiple_image_embeddings_input (L373-L420)
```python
@pytest.mark.core_model
@pytest.mark.parametrize("model", models)
@pytest.mark.parametrize(
    "size_factors",
    [
        # Single-scale
        [0.5],
        # Single-scale, batched
        [0.5, 0.5],
        # Multi-scale
        [0.25, 0.5, 0.5],
    ],
)
@pytest.mark.parametrize("dtype", [target_dtype])
@pytest.mark.parametrize("max_tokens", [128])
@pytest.mark.parametrize("num_logprobs", [10])
def test_qwen2_vl_multiple_image_embeddings_input(
    vllm_runner,
# ... 22 lines omitted for brevity ...
        inputs_per_case,
        model,
        dtype=dtype,
        max_tokens=max_tokens,
        num_logprobs=num_logprobs,
        mm_limit=2,
        tensor_parallel_size=1,
    )
```
**EN:** This test validates `test_qwen2_vl_multiple_image_embeddings_input`. It uses parameterization over `model`. Relevant pytest markers include `core_model`. Key inputs are `vllm_runner`, `image_assets`, `model`, `size_factors`, `dtype`, `max_tokens`.
**CN:** 这个测试验证 `test_qwen2_vl_multiple_image_embeddings_input`。 它通过参数化组合 `model`。 相关的 pytest 标记包括 `core_model`。 关键输入包括 `vllm_runner`、`image_assets`、`model`、`size_factors`、`dtype`、`max_tokens`。

### Test / 测试: test_qwen2_vl_video_embeddings_input (L423-L472)
```python
@pytest.mark.core_model
@pytest.mark.parametrize("model", models)
@pytest.mark.parametrize(
    "size_factors",
    [
        # Single-scale
        [0.5],
        # Single-scale, batched
        [0.5, 0.5],
        # Multi-scale
        [0.25, 0.25, 0.5],
    ],
)
@pytest.mark.parametrize("dtype", [target_dtype])
@pytest.mark.parametrize("max_tokens", [128])
@pytest.mark.parametrize("num_logprobs", [10])
def test_qwen2_vl_video_embeddings_input(
    vllm_runner,
# ... 24 lines omitted for brevity ...
        inputs_per_case,
        model,
        dtype=dtype,
        max_tokens=max_tokens,
        num_logprobs=num_logprobs,
        mm_limit=1,
        tensor_parallel_size=1,
    )
```
**EN:** This test validates `test_qwen2_vl_video_embeddings_input`. It uses parameterization over `model`. Relevant pytest markers include `core_model`. Key inputs are `vllm_runner`, `video_assets`, `model`, `size_factors`, `dtype`, `max_tokens`.
**CN:** 这个测试验证 `test_qwen2_vl_video_embeddings_input`。 它通过参数化组合 `model`。 相关的 pytest 标记包括 `core_model`。 关键输入包括 `vllm_runner`、`video_assets`、`model`、`size_factors`、`dtype`、`max_tokens`。

## Key Concepts / 关键概念
- **EN:** Pytest fixtures provide reusable setup and teardown boundaries.
  **CN:** Pytest fixture 提供可复用的初始化与清理边界。
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `typing.Any`, `typing.TypedDict`
- **Third-party / 第三方**: `PIL.Image`, `numpy.typing`, `pytest`, `torch`, `transformers.AutoProcessor`
- **Project / 项目内**: `vllm.multimodal.image.rescale_image_size`, `vllm.multimodal.video.rescale_video_size`, `vllm.multimodal.video.sample_frames_from_video`
- **Local relative imports / 本地相对导入**: `....conftest.IMAGE_ASSETS`, `....conftest.PromptImageInput`, `....conftest.PromptVideoInput`, `....conftest.VIDEO_ASSETS`, `....conftest.VllmRunner`, `...utils.check_logprobs_close`
