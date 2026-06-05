# test_awq.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/quantization/test_awq.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers quantized model support and model-facing behavior. The file defines 2 test(s), 0 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖量化模型支持与面向模型的行为。它定义了 2 个测试、0 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L5-L11)
```python
import pytest
import torch

from vllm.multimodal.image import rescale_image_size

from ...conftest import IMAGE_ASSETS, ImageTestAssets, VllmRunner
from ..utils import check_logprobs_close
```
**EN:** Imports third-party packages like `pytest`, `torch`, project helpers such as `vllm.multimodal.image.rescale_image_size`, `...conftest.IMAGE_ASSETS`, `...conftest.ImageTestAssets`.
**CN:** 导入第三方包（如 `pytest`、`torch`）、项目内辅助模块（如 `vllm.multimodal.image.rescale_image_size`、`...conftest.IMAGE_ASSETS`、`...conftest.ImageTestAssets`）。

### Module setup / 模块级配置: HF_IMAGE_PROMPTS (L13-L18)
```python
HF_IMAGE_PROMPTS = IMAGE_ASSETS.prompts(
    {
        "stop_sign": "<|im_start|>User\n<image>\nWhat's the content in the center of the image?<|im_end|>\n<|im_start|>Assistant\n",  # noqa: E501
        "cherry_blossom": "<|im_start|>User\n<image>\nWhat is the season?<|im_end|>\n<|im_start|>Assistant\n",  # noqa: E501
    }
)
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `HF_IMAGE_PROMPTS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `HF_IMAGE_PROMPTS`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: run_awq_test (L21-L93)
```python
def run_awq_test(
    vllm_runner: type[VllmRunner],
    image_assets: ImageTestAssets,
    source_model: str,
    quant_model: str,
    *,
    size_factors: list[float],
    dtype: str,
    max_tokens: int,
    num_logprobs: int,
    tensor_parallel_size: int,
    distributed_executor_backend: str | None = None,
):
    images = [asset.pil_image for asset in image_assets]

    inputs_per_image = [
        (
            [prompt for _ in size_factors],
# ... 47 lines omitted for brevity ...
        # TODO: Check whether using original CLIPVisionModel can improve
        # consistency against HF
        check_logprobs_close(
            outputs_0_lst=source_outputs,
            outputs_1_lst=quant_outputs,
            name_0="source",
            name_1="awq",
        )
```
**EN:** This helper encapsulates reusable logic in `run_awq_test`. Key inputs are `vllm_runner`, `image_assets`, `source_model`, `quant_model`.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_awq_test` 中。 关键输入包括 `vllm_runner`、`image_assets`、`source_model`、`quant_model`。

### Test / 测试: test_awq_load (L96-L124)
```python
@pytest.mark.parametrize(
    ("model", "quantization", "dtype"),
    [
        ("mattbucci/gemma-4-26B-AWQ", "awq", "float16"),
        ("cyankiwi/gemma-4-26B-A4B-it-AWQ-4bit", "compressed-tensors", "bfloat16"),
    ],
    ids=[
        "gemma4-moe-standard-awq-dot-suffix",
        "gemma4-moe-compressed-tensors-underscore-suffix",
    ],
)
@torch.inference_mode()
def test_awq_load(
    vllm_runner: type[VllmRunner],
    example_prompts: list[str],
    model: str,
    quantization: str,
    dtype: str,
) -> None:
    """Regression test: AWQ weight loading must not KeyError."""
    with vllm_runner(
        model,
        quantization=quantization,
        dtype=dtype,
        max_model_len=128,
        enforce_eager=True,
    ) as vllm_model:
        outputs = vllm_model.generate_greedy(example_prompts[:2], max_tokens=32)
    assert len(outputs) == 2
```
**EN:** This test validates `test_awq_load`. It uses parameterization to cover `gemma4-moe-standard-awq-dot-suffix`, `gemma4-moe-compressed-tensors-underscore-suffix`. Key inputs are `vllm_runner`, `example_prompts`, `model`, `quantization`, `dtype`. The main assertion is `len(outputs) == 2`.
**CN:** 这个测试验证 `test_awq_load`。 它通过参数化覆盖 `gemma4-moe-standard-awq-dot-suffix`、`gemma4-moe-compressed-tensors-underscore-suffix` 等场景。 关键输入包括 `vllm_runner`、`example_prompts`、`model`、`quantization`、`dtype`。 核心断言是 `len(outputs) == 2`。

### Test / 测试: test_awq_models (L127-L166)
```python
@pytest.mark.parametrize(
    ("source_model", "quant_model"),
    [("OpenGVLab/InternVL2-2B", "OpenGVLab/InternVL2-2B-AWQ")],
)
@pytest.mark.parametrize(
    "size_factors",
    [
        # Single-scale
        [1.0],
        # Single-scale, batched
        [1.0, 1.0, 1.0],
        # Multi-scale
        [0.25, 0.5, 1.0],
    ],
)
@pytest.mark.parametrize("dtype", ["half"])
@pytest.mark.parametrize("max_tokens", [128])
@pytest.mark.parametrize("num_logprobs", [5])
# ... 14 lines omitted for brevity ...
        source_model,
        quant_model,
        size_factors=size_factors,
        dtype=dtype,
        max_tokens=max_tokens,
        num_logprobs=num_logprobs,
        tensor_parallel_size=1,
    )
```
**EN:** This test validates `test_awq_models`. It uses parameterization over `source_model`, `quant_model`. Key inputs are `vllm_runner`, `image_assets`, `source_model`, `quant_model`, `size_factors`, `dtype`.
**CN:** 这个测试验证 `test_awq_models`。 它通过参数化组合 `source_model`、`quant_model`。 关键输入包括 `vllm_runner`、`image_assets`、`source_model`、`quant_model`、`size_factors`、`dtype`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `pytest`, `torch`
- **Project / 项目内**: `vllm.multimodal.image.rescale_image_size`
- **Local relative imports / 本地相对导入**: `...conftest.IMAGE_ASSETS`, `...conftest.ImageTestAssets`, `...conftest.VllmRunner`, `..utils.check_logprobs_close`
