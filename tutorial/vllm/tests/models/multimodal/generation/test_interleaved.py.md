# test_interleaved.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/generation/test_interleaved.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers multimodal processing and model-facing behavior. The file defines 1 test(s), 0 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖多模态处理与面向模型的行为。它定义了 1 个测试、0 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L8)
```python
import pytest

from vllm.assets.image import ImageAsset
from vllm.assets.video import VideoAsset
from vllm.multimodal.image import convert_image_mode
```
**EN:** Imports third-party packages like `pytest`, project helpers such as `vllm.assets.image.ImageAsset`, `vllm.assets.video.VideoAsset`, `vllm.multimodal.image.convert_image_mode`.
**CN:** 导入第三方包（如 `pytest`）、项目内辅助模块（如 `vllm.assets.image.ImageAsset`、`vllm.assets.video.VideoAsset`、`vllm.multimodal.image.convert_image_mode`）。

### Module setup / 模块级配置: models (L10-L10)
```python
models = ["llava-hf/llava-onevision-qwen2-0.5b-ov-hf"]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `models`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `models`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: base_prompt (L13-L14)
```python
def base_prompt(modalities_str: str) -> str:
    return f"<|im_start|>user {modalities_str}\nDescribe what you see from these items.<|im_end|><|im_start|>assistant\n"  # noqa: E501
```
**EN:** This helper encapsulates reusable logic in `base_prompt`. Key inputs are `modalities_str`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `base_prompt` 中。 关键输入包括 `modalities_str`。 它把计算得到的状态或辅助对象返回给调用方。

### Module setup / 模块级配置: INTERLEAVED_PROMPT, NONINTERLEAVED_PROMPT (L17-L18)
```python
INTERLEAVED_PROMPT = base_prompt("<image><video><image>\n")
NONINTERLEAVED_PROMPT = base_prompt("<image><image><video>\n")
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `INTERLEAVED_PROMPT`, `NONINTERLEAVED_PROMPT`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `INTERLEAVED_PROMPT`、`NONINTERLEAVED_PROMPT`。后续辅助函数和测试会复用这些值，以减少重复。

### Test / 测试: test_models (L21-L81)
```python
@pytest.mark.core_model
@pytest.mark.parametrize("model", models)
@pytest.mark.parametrize("dtype", ["float16"])
@pytest.mark.parametrize("max_tokens", [128])
def test_models(vllm_runner, model, dtype: str, max_tokens: int) -> None:
    """
    This is a simple test to check if interleaved and non-interleaved prompts
    give the same result.
    """

    image_cherry = convert_image_mode(ImageAsset("cherry_blossom").pil_image, "RGB")
    image_stop = convert_image_mode(ImageAsset("stop_sign").pil_image, "RGB")
    images = [image_cherry, image_stop]
    video = VideoAsset(name="baby_reading", num_frames=16).np_ndarrays

    inputs = [
        (
            [INTERLEAVED_PROMPT],
# ... 35 lines omitted for brevity ...
    interleaved_output_str, noninterleaved_output_str = generated_strs

    # The two prompts are identical except for the order of modality tokens.
    assert interleaved_prompt_len == noninterleaved_prompt_len

    # The two generated strings should be different because of the
    # interleaved modality tokens.
    assert interleaved_output_str != noninterleaved_output_str
```
**EN:** This test validates `test_models`. It uses parameterization over `model`. Relevant pytest markers include `core_model`. Key inputs are `vllm_runner`, `model`, `dtype`, `max_tokens`. The main assertion is `interleaved_prompt_len == noninterleaved_prompt_len` and `interleaved_output_str != noninterleaved_output_str`.
**CN:** 这个测试验证 `test_models`。 它通过参数化组合 `model`。 相关的 pytest 标记包括 `core_model`。 关键输入包括 `vllm_runner`、`model`、`dtype`、`max_tokens`。 核心断言是 `interleaved_prompt_len == noninterleaved_prompt_len` and `interleaved_output_str != noninterleaved_output_str`。

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
- **Third-party / 第三方**: `pytest`
- **Project / 项目内**: `vllm.assets.image.ImageAsset`, `vllm.assets.video.VideoAsset`, `vllm.multimodal.image.convert_image_mode`
