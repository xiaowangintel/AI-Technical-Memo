# test_radio.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/pooling/test_radio.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers pooling or embedding behavior and multimodal processing. The file defines 1 test(s), 0 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖池化或嵌入行为与多模态处理。它定义了 1 个测试、0 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L15)
```python
import pytest
import torch
import torch.nn as nn
from huggingface_hub import snapshot_download
from transformers import AutoConfig, AutoModel, CLIPImageProcessor

from vllm.distributed import cleanup_dist_env_and_memory
from vllm.model_executor.models.radio import RadioModel
from vllm.platforms import current_platform
from vllm.transformers_utils.configs.radio import RadioConfig
from vllm.utils.torch_utils import STR_DTYPE_TO_TORCH_DTYPE

from ....conftest import ImageTestAssets
```
**EN:** Imports third-party packages like `huggingface_hub.snapshot_download`, `pytest`, `torch`, project helpers such as `vllm.distributed.cleanup_dist_env_and_memory`, `vllm.model_executor.models.radio.RadioModel`, `vllm.platforms.current_platform`.
**CN:** 导入第三方包（如 `huggingface_hub.snapshot_download`、`pytest`、`torch`）、项目内辅助模块（如 `vllm.distributed.cleanup_dist_env_and_memory`、`vllm.model_executor.models.radio.RadioModel`、`vllm.platforms.current_platform`）。

### Module setup / 模块级配置: DOWNLOAD_PATTERN, DEVICE_TYPE (L19-L21)
```python
DOWNLOAD_PATTERN = ["*.json", "*.py", "*.safetensors", "*.txt", "*.model"]

DEVICE_TYPE = current_platform.device_type
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `DOWNLOAD_PATTERN`, `DEVICE_TYPE`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `DOWNLOAD_PATTERN`、`DEVICE_TYPE`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: run_radio_test (L24-L89)
```python
@torch.inference_mode()
def run_radio_test(
    image_assets: ImageTestAssets,
    model_id: str,
    *,
    dtype: str,
):
    model = snapshot_download(model_id, allow_patterns=DOWNLOAD_PATTERN)
    torch_dtype = STR_DTYPE_TO_TORCH_DTYPE[dtype]

    img_processor = CLIPImageProcessor.from_pretrained(model)
    images = [asset.pil_image for asset in image_assets]
    # Input resolution must be a multiple of `self.min_resolution_step`.
    # Using `self.get_nearest_supported_resolution`, for assets 432x642 the
    # nearest supported resolution is 432x640.
    pixel_values = [
        img_processor(image, return_tensors="pt").pixel_values.to(torch_dtype)[
            :, :, :, :640
# ... 40 lines omitted for brevity ...
    ]
    del vllm_model, hf_model
    cleanup_dist_env_and_memory()

    cos_similar = nn.CosineSimilarity(dim=-1)
    for vllm_output, hf_output in zip(vllm_outputs_per_image, hf_outputs_per_image):
        assert cos_similar(vllm_output[0], hf_output[0]).mean() > 0.99
        assert cos_similar(vllm_output[1], hf_output[1]).mean() > 0.99
```
**EN:** This helper encapsulates reusable logic in `run_radio_test`. Key inputs are `image_assets`, `model_id`. The main assertion is `cos_similar(vllm_output[0], hf_output[0]).mean() > 0.99` and `cos_similar(vllm_output[1], hf_output[1]).mean() > 0.99`.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_radio_test` 中。 关键输入包括 `image_assets`、`model_id`。 核心断言是 `cos_similar(vllm_output[0], hf_output[0]).mean() > 0.99` and `cos_similar(vllm_output[1], hf_output[1]).mean() > 0.99`。

### Test / 测试: test_radio (L92-L106)
```python
@pytest.mark.parametrize(
    "model_id",
    [
        "nvidia/C-RADIOv2-H",
    ],
)
@pytest.mark.parametrize("dtype", ["half", "bfloat16"])
def test_radio(
    default_vllm_config, dist_init, image_assets, model_id, dtype: str
) -> None:
    run_radio_test(
        image_assets,
        model_id,
        dtype=dtype,
    )
```
**EN:** This test validates `test_radio`. It uses parameterization over `model_id`. Key inputs are `default_vllm_config`, `dist_init`, `image_assets`, `model_id`, `dtype`.
**CN:** 这个测试验证 `test_radio`。 它通过参数化组合 `model_id`。 关键输入包括 `default_vllm_config`、`dist_init`、`image_assets`、`model_id`、`dtype`。

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
- **Third-party / 第三方**: `huggingface_hub.snapshot_download`, `pytest`, `torch`, `torch.nn`, `transformers.AutoConfig`, `transformers.AutoModel`, `transformers.CLIPImageProcessor`
- **Project / 项目内**: `vllm.distributed.cleanup_dist_env_and_memory`, `vllm.model_executor.models.radio.RadioModel`, `vllm.platforms.current_platform`, `vllm.transformers_utils.configs.radio.RadioConfig`, `vllm.utils.torch_utils.STR_DTYPE_TO_TORCH_DTYPE`
- **Local relative imports / 本地相对导入**: `....conftest.ImageTestAssets`
