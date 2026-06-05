# test_vit_backend_functionality.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/generation/test_vit_backend_functionality.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers multimodal processing and model-facing behavior. The file defines 1 test(s), 0 fixture(s), and 7 helper/class block(s) to validate this area. / [CN] 该文件覆盖多模态处理与面向模型的行为。它定义了 1 个测试、0 个 fixture，以及 7 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L10-L22)
```python
from typing import Any

import pytest
from transformers import AutoProcessor

from vllm import LLM, SamplingParams
from vllm.multimodal.utils import encode_image_url
from vllm.multimodal.video import sample_frames_from_video
from vllm.platforms import current_platform
from vllm.v1.attention.backends.registry import AttentionBackendEnum

from ....utils import create_new_process_for_each_test
from ...utils import dummy_hf_overrides
```
**EN:** Imports standard-library modules such as `typing.Any`, third-party packages like `pytest`, `transformers.AutoProcessor`, project helpers such as `vllm.LLM`, `vllm.SamplingParams`, `vllm.multimodal.utils.encode_image_url`.
**CN:** 导入标准库模块（如 `typing.Any`）、第三方包（如 `pytest`、`transformers.AutoProcessor`）、项目内辅助模块（如 `vllm.LLM`、`vllm.SamplingParams`、`vllm.multimodal.utils.encode_image_url`）。

### Module setup / 模块级配置: DOTS_OCR_PROMPT, VIDEO_PLACEHOLDER, MODEL_CONFIGS (L27-L185)
```python
DOTS_OCR_PROMPT = """Please output the layout information from the PDF image, including each layout element's bbox, its category, and the corresponding text content within the bbox.

1. Bbox format: [x1, y1, x2, y2]

2. Layout Categories: The possible categories are ['Caption', 'Footnote', 'Formula', 'List-item', 'Page-footer', 'Page-header', 'Picture', 'Section-header', 'Table', 'Text', 'Title'].

3. Text Extraction & Formatting Rules:
    - Picture: For the 'Picture' category, the text field should be omitted.
    - Formula: Format its text as LaTeX.
    - Table: Format its text as HTML.
    - All Others (Text, Title, etc.): Format their text as Markdown.

4. Constraints:
    - The output text must be the original text from the image, with no translation.
    - All layout elements must be sorted according to human reading order.

# ... 137 lines omitted for brevity ...
            "max_tokens": 16384,
        },
        "use_processor": True,
        "question": "What is the content of each image?",
    },
}
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `DOTS_OCR_PROMPT`, `VIDEO_PLACEHOLDER`, `MODEL_CONFIGS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `DOTS_OCR_PROMPT`、`VIDEO_PLACEHOLDER`、`MODEL_CONFIGS`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: build_dots_ocr_prompt (L189-L209)
```python
def build_dots_ocr_prompt(images, config):
    """Build Dots.OCR specific prompt with OCR instructions."""
    # Use only stop_sign image for Dots.OCR
    image = images[0]  # Already filtered to stop_sign
    image_url = encode_image_url(image)

    placeholders = [{"type": "image_url", "image_url": {"url": image_url}}]
    messages = [
        {
            "role": "user",
            "content": [
                *placeholders,
                {
                    "type": "text",
                    "text": f"<|img|><|imgpad|><|endofimg|>{DOTS_OCR_PROMPT}",
                },
            ],
        },
    ]

    return messages
```
**EN:** This helper encapsulates reusable logic in `build_dots_ocr_prompt`. Key inputs are `images`, `config`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `build_dots_ocr_prompt` 中。 关键输入包括 `images`、`config`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: build_processor_prompt (L212-L232)
```python
def build_processor_prompt(images, config):
    """Build prompt using AutoProcessor.apply_chat_template()."""
    processor = AutoProcessor.from_pretrained(
        config["model_name"], trust_remote_code=True
    )

    image_urls = [encode_image_url(img) for img in images]
    placeholders = [{"type": "image", "image": url} for url in image_urls]
    messages = [
        {
            "role": "user",
            "content": [
                *placeholders,
                {"type": "text", "text": config["question"]},
            ],
        },
    ]

    return processor.apply_chat_template(
        messages, tokenize=False, add_generation_prompt=True
    )
```
**EN:** This helper encapsulates reusable logic in `build_processor_prompt`. Key inputs are `images`, `config`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `build_processor_prompt` 中。 关键输入包括 `images`、`config`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: build_ovis_prompt (L235-L246)
```python
def build_ovis_prompt(images, config):
    """Build Ovis2.5 specific prompt with custom format."""
    image_urls = [encode_image_url(img) for img in images]

    placeholders = "\n".join(
        f"Image-{i}: <image>\n" for i, _ in enumerate(image_urls, start=1)
    )

    return (
        f"<|im_start|>user\n\n{placeholders}\n{config['question']}<|im_end|>\n"
        "<|im_start|>assistant\n"
    )
```
**EN:** This helper encapsulates reusable logic in `build_ovis_prompt`. Key inputs are `images`, `config`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `build_ovis_prompt` 中。 关键输入包括 `images`、`config`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: build_qwen2_5_video_prompt (L249-L256)
```python
def build_qwen2_5_video_prompt():
    """Build Qwen2.5-VL video prompt with EVS placeholder."""
    return (
        f"<|im_start|>system\nYou are a helpful assistant.<|im_end|>\n"
        f"<|im_start|>user\n{VIDEO_PLACEHOLDER}"
        "Describe this video with a short sentence (no more than 20 words)"
        "<|im_end|><|im_start|>assistant\n"
    )
```
**EN:** This helper encapsulates reusable logic in `build_qwen2_5_video_prompt`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `build_qwen2_5_video_prompt` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: run_llm_generate_test (L260-L304)
```python
def run_llm_generate_test(config, mm_encoder_attn_backend, image_assets):
    """Standard LLM.generate() interface handler."""
    images = [asset.pil_image for asset in image_assets]

    # Build prompt
    if config.get("use_processor"):
        prompt = build_processor_prompt(images, config)
    else:
        prompt_builder_name = config.get("prompt_builder", "build_ovis_prompt")
        prompt_builder = globals()[prompt_builder_name]
        prompt = prompt_builder(images, config)

    # Determine limit_mm_per_prompt
    limit_mm_per_prompt = config.get("limit_mm_per_prompt", {"image": len(images)})

    # Create engine
    llm = LLM(
        model=config["model_name"],
# ... 19 lines omitted for brevity ...

    # Validate
    for o in outputs:
        generated_text = o.outputs[0].text
        validator = config.get("output_validator", lambda x: len(x) > 10)
        assert validator(generated_text), (
            f"Validation failed for {config['model_name']}: {generated_text}"
        )
```
**EN:** This helper encapsulates reusable logic in `run_llm_generate_test`. Key inputs are `config`, `mm_encoder_attn_backend`, `image_assets`. It touches the core vLLM initialization or engine path directly. The main assertion is `validator(generated_text)`.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_llm_generate_test` 中。 关键输入包括 `config`、`mm_encoder_attn_backend`、`image_assets`。 它直接覆盖 vLLM 的核心初始化或引擎执行路径。 核心断言是 `validator(generated_text)`。

### Helper / 辅助函数: run_llm_chat_test (L307-L340)
```python
def run_llm_chat_test(config, mm_encoder_attn_backend, image_assets):
    """LLM.chat() interface handler for Dots.OCR."""
    # Filter to stop_sign image only
    stop_sign_image = [
        asset.pil_image for asset in image_assets if asset.name == "stop_sign"
    ][0]

    # Build messages
    messages = build_dots_ocr_prompt([stop_sign_image], config)

    # Create engine
    llm = LLM(
        model=config["model_name"],
        trust_remote_code=True,
        max_model_len=config["max_model_len"],
        max_num_seqs=config["max_num_seqs"],
        limit_mm_per_prompt=config["limit_mm_per_prompt"],
        mm_encoder_attn_backend=mm_encoder_attn_backend,
# ... 8 lines omitted for brevity ...

    # Validate
    for o in outputs:
        generated_text = o.outputs[0].text
        validator = config.get("output_validator", lambda x: len(x) > 10)
        assert validator(generated_text), (
            f"Validation failed for {config['model_name']}: {generated_text}"
        )
```
**EN:** This helper encapsulates reusable logic in `run_llm_chat_test`. Key inputs are `config`, `mm_encoder_attn_backend`, `image_assets`. It touches the core vLLM initialization or engine path directly. The main assertion is `validator(generated_text)`.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_llm_chat_test` 中。 关键输入包括 `config`、`mm_encoder_attn_backend`、`image_assets`。 它直接覆盖 vLLM 的核心初始化或引擎执行路径。 核心断言是 `validator(generated_text)`。

### Helper / 辅助函数: run_video_test (L343-L385)
```python
def run_video_test(config, mm_encoder_attn_backend, video_assets, vllm_runner):
    """Video test with EVS (Efficient Video Sampling) handler."""
    for pruning_rate in config["video_params"]["pruning_rates"]:
        num_frames = config["video_params"]["num_frames"]

        # Sample frames from video
        sampled_vids = [
            sample_frames_from_video(asset.np_ndarrays, num_frames)
            for asset in video_assets
        ]

        # Build prompt and prepare video
        prompt = build_qwen2_5_video_prompt()
        prompts = [prompt]
        videos = [sampled_vids[0]]

        # Run with vllm_runner context manager
        with vllm_runner(
# ... 17 lines omitted for brevity ...
            # Validate output
            assert len(outputs) == 1, f"Expected 1 output, got {len(outputs)}"
            output_ids, output_text = outputs[0]
            assert len(output_ids) > 0, "Generated no output IDs"
            assert len(output_text) > 0, "Generated empty text"
            assert isinstance(output_text, str), (
                f"Output is not string: {type(output_text)}"
            )
```
**EN:** This helper encapsulates reusable logic in `run_video_test`. Key inputs are `config`, `mm_encoder_attn_backend`, `video_assets`, `vllm_runner`. The main assertion is `len(outputs) == 1` and `len(output_ids) > 0`.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_video_test` 中。 关键输入包括 `config`、`mm_encoder_attn_backend`、`video_assets`、`vllm_runner`。 核心断言是 `len(outputs) == 1` and `len(output_ids) > 0`。

### Test / 测试: test_vit_backend_functionality (L389-L438)
```python
@pytest.mark.parametrize("model_key", list(MODEL_CONFIGS.keys()))
@pytest.mark.parametrize(
    "mm_encoder_attn_backend",
    [None] + current_platform.get_supported_vit_attn_backends(),
)
@pytest.mark.skip(reason="Broken test due to memory segmentation fault")
@create_new_process_for_each_test()
def test_vit_backend_functionality(
    model_key: str,
    mm_encoder_attn_backend: AttentionBackendEnum | None,
    image_assets,
    video_assets,
    vllm_runner,
    request,
):
    """Test ViT attention backend functionality for multimodal models.

    This test validates that each model can successfully generate outputs
# ... 24 lines omitted for brevity ...
    if config.get("media_type") == "video":
        run_video_test(config, mm_encoder_attn_backend, video_assets, vllm_runner)
    elif config["interface"] == "llm_chat":
        run_llm_chat_test(config, mm_encoder_attn_backend, image_assets)
    elif config["interface"] == "llm_generate":
        run_llm_generate_test(config, mm_encoder_attn_backend, image_assets)
    else:
        raise ValueError(f"Unknown interface: {config['interface']}")
```
**EN:** This test validates `test_vit_backend_functionality`. It uses parameterization over `model_key`. Relevant pytest markers include `skip`. Key inputs are `model_key`, `mm_encoder_attn_backend`, `image_assets`, `video_assets`, `vllm_runner`, `request`. The logic can skip unsupported environments when prerequisites are not satisfied.
**CN:** 这个测试验证 `test_vit_backend_functionality`。 它通过参数化组合 `model_key`。 相关的 pytest 标记包括 `skip`。 关键输入包括 `model_key`、`mm_encoder_attn_backend`、`image_assets`、`video_assets`、`vllm_runner`、`request`。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。

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
- **Stdlib / 标准库**: `typing.Any`
- **Third-party / 第三方**: `pytest`, `transformers.AutoProcessor`
- **Project / 项目内**: `vllm.LLM`, `vllm.SamplingParams`, `vllm.multimodal.utils.encode_image_url`, `vllm.multimodal.video.sample_frames_from_video`, `vllm.platforms.current_platform`, `vllm.v1.attention.backends.registry.AttentionBackendEnum`
- **Local relative imports / 本地相对导入**: `....utils.create_new_process_for_each_test`, `...utils.dummy_hf_overrides`
