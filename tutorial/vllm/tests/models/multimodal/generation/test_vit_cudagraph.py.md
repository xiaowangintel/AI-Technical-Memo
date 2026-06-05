# test_vit_cudagraph.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/generation/test_vit_cudagraph.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers multimodal processing and model-facing behavior. The file defines 2 test(s), 0 fixture(s), and 5 helper/class block(s) to validate this area. / [CN] 该文件覆盖多模态处理与面向模型的行为。它定义了 2 个测试、0 个 fixture，以及 5 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L13)
```python
from dataclasses import dataclass, field

import pytest

from vllm.multimodal.video import sample_frames_from_video
from vllm.platforms import current_platform

from ....conftest import IMAGE_ASSETS, VIDEO_ASSETS
from ....utils import create_new_process_for_each_test
from .vlm_utils.builders import sample_frames_with_video_metadata
```
**EN:** Imports standard-library modules such as `dataclasses.dataclass`, `dataclasses.field`, third-party packages like `pytest`, project helpers such as `vllm.multimodal.video.sample_frames_from_video`, `vllm.platforms.current_platform`, `....conftest.IMAGE_ASSETS`.
**CN:** 导入标准库模块（如 `dataclasses.dataclass`、`dataclasses.field`）、第三方包（如 `pytest`）、项目内辅助模块（如 `vllm.multimodal.video.sample_frames_from_video`、`vllm.platforms.current_platform`、`....conftest.IMAGE_ASSETS`）。

### Class / 类: VitCudagraphTestConfig (L16-L29)
```python
@dataclass
class VitCudagraphTestConfig:
    model: str
    modalities: list[str] = field(default_factory=lambda: ["image", "video"])
    image_prompt: str | None = None
    video_prompt: str | None = None
    dtype: str = "bfloat16"
    max_model_len: int = 4096
    max_tokens: int = 64
    max_num_seqs: int = 2
    num_video_frames: int = 16
    needs_video_metadata: bool = False
    vllm_runner_kwargs: dict = field(default_factory=dict)
    marks: list = field(default_factory=list)
```
**EN:** This class groups related scenarios in `VitCudagraphTestConfig`. Decorators such as `@dataclass` make it a compact metadata container. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `VitCudagraphTestConfig` 相关的场景组织在一起。 像 `@dataclass` 这样的装饰器使它成为紧凑的元数据容器。 其主体主要是结构性定义，没有直接方法实现。

### Helper / 辅助函数: params_with_marks (L32-L37)
```python
def params_with_marks(
    configs: dict[str, VitCudagraphTestConfig],
) -> list[pytest.param]:
    return [
        pytest.param(model_id, marks=cfg.marks) for model_id, cfg in configs.items()
    ]
```
**EN:** This helper encapsulates reusable logic in `params_with_marks`. Key inputs are `configs`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `params_with_marks` 中。 关键输入包括 `configs`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: qwen_vl_chat_template (L40-L41)
```python
def qwen_vl_chat_template(content: str) -> str:
    return f"<|im_start|>user\n{content}<|im_end|>\n<|im_start|>assistant\n"
```
**EN:** This helper encapsulates reusable logic in `qwen_vl_chat_template`. Key inputs are `content`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `qwen_vl_chat_template` 中。 关键输入包括 `content`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: step3_vl_chat_template (L44-L48)
```python
def step3_vl_chat_template(content: str) -> str:
    return (
        "<｜begin▁of▁sentence｜> You are a helpful assistant.<|BOT|>user\n "
        f"<im_patch>{content} <|EOT|><|BOT|>assistant\n"
    )
```
**EN:** This helper encapsulates reusable logic in `step3_vl_chat_template`. Key inputs are `content`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `step3_vl_chat_template` 中。 关键输入包括 `content`。 它把计算得到的状态或辅助对象返回给调用方。

### Module setup / 模块级配置: MODEL_CONFIGS (L51-L105)
```python
MODEL_CONFIGS: dict[str, VitCudagraphTestConfig] = {
    "qwen2_5_vl": VitCudagraphTestConfig(
        model="Qwen/Qwen2.5-VL-3B-Instruct",
        image_prompt=qwen_vl_chat_template(
            "<|vision_start|><|image_pad|><|vision_end|>What is in this image?"
        ),
        video_prompt=qwen_vl_chat_template(
            "<|vision_start|><|video_pad|><|vision_end|>"
            "Describe this video in one sentence."
        ),
        needs_video_metadata=False,
        marks=[pytest.mark.core_model],
    ),
    "qwen3_vl": VitCudagraphTestConfig(
        model="Qwen/Qwen3-VL-2B-Instruct",
        image_prompt=qwen_vl_chat_template(
# ... 33 lines omitted for brevity ...
    "step3_vl": VitCudagraphTestConfig(
        model="stepfun-ai/Step3-VL-10B",
        image_prompt=step3_vl_chat_template("What is in this image?"),
        video_prompt=None,
    ),
}
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_CONFIGS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_CONFIGS`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: get_compilation_config (L108-L113)
```python
def get_compilation_config():
    return {
        "cudagraph_mm_encoder": True,
        "encoder_cudagraph_max_vision_items_per_batch": 1,
        "encoder_cudagraph_max_frames_per_batch": 16,
    }
```
**EN:** This helper encapsulates reusable logic in `get_compilation_config`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `get_compilation_config` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_vit_cudagraph_image (L121-L160)
```python
@pytest.mark.parametrize("model_id", params_with_marks(MODEL_CONFIGS))
@pytest.mark.skipif(not current_platform.is_cuda(), reason="Requires CUDA")
@create_new_process_for_each_test()
def test_vit_cudagraph_image(model_id, vllm_runner, image_assets):
    config = MODEL_CONFIGS[model_id]

    if "image" not in config.modalities:
        pytest.skip(f"{model_id} does not support the image modality.")

    image_prompts = IMAGE_ASSETS.prompts(
        {
            "stop_sign": config.image_prompt,  # type: ignore[typeddict-item]
            "cherry_blossom": config.image_prompt,  # type: ignore[typeddict-item]
        }
    )
    images = [[asset.pil_image] for asset in image_assets]

    with vllm_runner(
# ... 14 lines omitted for brevity ...
        output_ids, output_text = outputs[0]

        # Ensure we got some output
        assert len(output_ids) > 0
        assert len(output_text) > 0

        # Ensure the output is a string
        assert isinstance(output_text, str)
```
**EN:** This test validates `test_vit_cudagraph_image`. It uses parameterization over `model_id`. Relevant pytest markers include `skipif`. Key inputs are `model_id`, `vllm_runner`, `image_assets`. The logic can skip unsupported environments when prerequisites are not satisfied. The main assertion is `len(outputs) == 2` and `len(output_ids) > 0`.
**CN:** 这个测试验证 `test_vit_cudagraph_image`。 它通过参数化组合 `model_id`。 相关的 pytest 标记包括 `skipif`。 关键输入包括 `model_id`、`vllm_runner`、`image_assets`。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。 核心断言是 `len(outputs) == 2` and `len(output_ids) > 0`。

### Test / 测试: test_vit_cudagraph_video (L163-L213)
```python
@pytest.mark.parametrize("model_id", params_with_marks(MODEL_CONFIGS))
@pytest.mark.skipif(not current_platform.is_cuda(), reason="Requires CUDA")
@create_new_process_for_each_test()
def test_vit_cudagraph_video(model_id, vllm_runner, video_assets):
    config = MODEL_CONFIGS[model_id]

    if "video" not in config.modalities:
        pytest.skip(f"{model_id} does not support the video modality")

    video_prompts = VIDEO_ASSETS.prompts(
        {
            "baby_reading": config.video_prompt,  # type: ignore[typeddict-item]
        }
    )
    if config.needs_video_metadata:
        sampled_vids = [
            sample_frames_with_video_metadata(
                (asset.np_ndarrays, asset.metadata), config.num_video_frames
# ... 25 lines omitted for brevity ...
        output_ids, output_text = outputs[0]

        # Ensure we got some output
        assert len(output_ids) > 0
        assert len(output_text) > 0

        # Ensure the output is a string
        assert isinstance(output_text, str)
```
**EN:** This test validates `test_vit_cudagraph_video`. It uses parameterization over `model_id`. Relevant pytest markers include `skipif`. Key inputs are `model_id`, `vllm_runner`, `video_assets`. The logic can skip unsupported environments when prerequisites are not satisfied. The main assertion is `len(outputs) == 1` and `len(output_ids) > 0`.
**CN:** 这个测试验证 `test_vit_cudagraph_video`。 它通过参数化组合 `model_id`。 相关的 pytest 标记包括 `skipif`。 关键输入包括 `model_id`、`vllm_runner`、`video_assets`。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。 核心断言是 `len(outputs) == 1` and `len(output_ids) > 0`。

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
- **Stdlib / 标准库**: `dataclasses.dataclass`, `dataclasses.field`
- **Third-party / 第三方**: `pytest`
- **Project / 项目内**: `vllm.multimodal.video.sample_frames_from_video`, `vllm.platforms.current_platform`
- **Local relative imports / 本地相对导入**: `....conftest.IMAGE_ASSETS`, `....conftest.VIDEO_ASSETS`, `....utils.create_new_process_for_each_test`, `.vlm_utils.builders.sample_frames_with_video_metadata`
