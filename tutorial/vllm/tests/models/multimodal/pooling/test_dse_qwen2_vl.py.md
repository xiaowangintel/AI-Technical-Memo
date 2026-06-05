# test_dse_qwen2_vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/pooling/test_dse_qwen2_vl.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and multimodal processing. The file defines 2 test(s), 0 fixture(s), and 3 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与多模态处理。它定义了 2 个测试、0 个 fixture，以及 3 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L14)
```python
from collections.abc import Callable

import pytest
import torch
import torch.nn.functional as F
from PIL import Image
from transformers import Qwen2VLForConditionalGeneration

from ....conftest import IMAGE_ASSETS, HfRunner, PromptImageInput, VllmRunner
from ....utils import large_gpu_test
from ...utils import check_embeddings_close
```
**EN:** Imports standard-library modules such as `collections.abc.Callable`, third-party packages like `PIL.Image`, `pytest`, `torch`, project helpers such as `....conftest.HfRunner`, `....conftest.IMAGE_ASSETS`, `....conftest.PromptImageInput`.
**CN:** 导入标准库模块（如 `collections.abc.Callable`）、第三方包（如 `PIL.Image`、`pytest`、`torch`）、项目内辅助模块（如 `....conftest.HfRunner`、`....conftest.IMAGE_ASSETS`、`....conftest.PromptImageInput`）。

### Module setup / 模块级配置: HF_TEXT_PROMPTS, HF_IMAGE_PROMPTS, MODELS (L16-L36)
```python
HF_TEXT_PROMPTS = [
    # T -> X
    (
        "Query: Find me an everyday image that matches the given caption: The label of the object is stop sign",  # noqa: E501,
        Image.new("RGB", (56, 56)),
    ),
    # T -> X
    (
        "Query: Retrieve an image of this caption: cherry blossom",
        Image.new("RGB", (56, 56)),
    ),
]

HF_IMAGE_PROMPTS = IMAGE_ASSETS.prompts(
    {
        "stop_sign": "What is shown in this image?",
        "cherry_blossom": "What is shown in this image?",
    }
)

MODELS = ["MrLight/dse-qwen2-2b-mrl-v1"]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `HF_TEXT_PROMPTS`, `HF_IMAGE_PROMPTS`, `MODELS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `HF_TEXT_PROMPTS`、`HF_IMAGE_PROMPTS`、`MODELS`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: get_messages (L39-L66)
```python
def get_messages(image: Image.Image, text: str, embed_text: bool):
    # assert False, 'remember to use outer [] as required'
    if embed_text:
        messages = [
            {
                "role": "user",
                "content": [
                    {
                        "type": "image",
                        "image": Image.new("RGB", (56, 56)),
                        "resized_height": 1,
                        "resized_width": 1,
                    },  # need a dummy image here for an easier process.
                    {"type": "text", "text": text},
                ],
            }
        ]
    else:
        messages = [
            {
                "role": "user",
                "content": [
                    {"type": "image", "image": image},
                    {"type": "text", "text": text},
                ],
            }
        ]
    return messages
```
**EN:** This helper encapsulates reusable logic in `get_messages`. Key inputs are `image`, `text`, `embed_text`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `get_messages` 中。 关键输入包括 `image`、`text`、`embed_text`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: apply_chat_template_and_add_eos (L69-L77)
```python
def apply_chat_template_and_add_eos(
    messages: list[dict],
    apply_chat_template_fn: Callable,
):
    prompt = (
        apply_chat_template_fn(messages, tokenize=False, add_generation_prompt=True)
        + "<|endoftext|>"
    )
    return prompt
```
**EN:** This helper encapsulates reusable logic in `apply_chat_template_and_add_eos`. Key inputs are `messages`, `apply_chat_template_fn`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `apply_chat_template_and_add_eos` 中。 关键输入包括 `messages`、`apply_chat_template_fn`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: _run_test (L80-L160)
```python
def _run_test(
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    input_texts: list[str],
    input_images: PromptImageInput,
    embed_texts: list[bool],
    model: str,
    *,
    dtype: str,
) -> None:
    """SET PYTHONPATH"""
    # NOTE: take care of the order. run vLLM first, and then run HF.
    # vLLM needs a fresh new process without cuda initialization.
    # if we run HF first, the cuda initialization will be done and it
    # will hurt multiprocessing backend with fork method (the default method).
    with vllm_runner(
        model, runner="pooling", dtype=dtype, enforce_eager=True, max_model_len=8192
    ) as vllm_model:
# ... 55 lines omitted for brevity ...
            hf_outputs = all_outputs

    check_embeddings_close(
        embeddings_0_lst=hf_outputs,
        embeddings_1_lst=vllm_outputs,
        name_0="hf",
        name_1="vllm",
    )
```
**EN:** This helper encapsulates reusable logic in `_run_test`. Key inputs are `hf_runner`, `vllm_runner`, `input_texts`, `input_images`, `embed_texts`, `model`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_run_test` 中。 关键输入包括 `hf_runner`、`vllm_runner`、`input_texts`、`input_images`、`embed_texts`、`model`。

### Test / 测试: test_models_text (L163-L187)
```python
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", ["bfloat16"])
def test_models_text(
    hf_runner,
    vllm_runner,
    image_assets,
    model: str,
    dtype: str,
) -> None:
    input_texts_images = [
        (text, image_placeholder) for text, image_placeholder in HF_TEXT_PROMPTS
    ]
    input_texts = [text for text, _ in input_texts_images]
    input_images = [image for _, image in input_texts_images]
    embed_texts = [True] * len(input_texts)

    _run_test(
        hf_runner,
        vllm_runner,
        input_texts,
        input_images,  # type: ignore
        embed_texts,
        model,
        dtype=dtype,
    )
```
**EN:** This test validates `test_models_text`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `image_assets`, `model`, `dtype`.
**CN:** 这个测试验证 `test_models_text`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`image_assets`、`model`、`dtype`。

### Test / 测试: test_models_image (L190-L215)
```python
@large_gpu_test(min_gb=48)
@pytest.mark.parametrize("model", MODELS)
@pytest.mark.parametrize("dtype", ["bfloat16"])
def test_models_image(
    hf_runner,
    vllm_runner,
    image_assets,
    model: str,
    dtype: str,
) -> None:
    input_texts_images = [
        (text, asset.pil_image) for text, asset in zip(HF_IMAGE_PROMPTS, image_assets)
    ]
    input_texts = [text for text, _ in input_texts_images]
    input_images = [image for _, image in input_texts_images]
    embed_texts = [False] * len(input_texts)

    _run_test(
        hf_runner,
        vllm_runner,
        input_texts,
        input_images,
        embed_texts,
        model,
        dtype=dtype,
    )
```
**EN:** This test validates `test_models_image`. It uses parameterization over `model`. Key inputs are `hf_runner`, `vllm_runner`, `image_assets`, `model`, `dtype`.
**CN:** 这个测试验证 `test_models_image`。 它通过参数化组合 `model`。 关键输入包括 `hf_runner`、`vllm_runner`、`image_assets`、`model`、`dtype`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc.Callable`
- **Third-party / 第三方**: `PIL.Image`, `pytest`, `torch`, `torch.nn.functional`, `transformers.Qwen2VLForConditionalGeneration`
- **Local relative imports / 本地相对导入**: `....conftest.HfRunner`, `....conftest.IMAGE_ASSETS`, `....conftest.PromptImageInput`, `....conftest.VllmRunner`, `....utils.large_gpu_test`, `...utils.check_embeddings_close`
