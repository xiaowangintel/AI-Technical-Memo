# test_glm4_1v.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/processing/test_glm4_1v.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers multimodal processing and model-facing behavior. The file defines 2 test(s), 0 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖多模态处理与面向模型的行为。它定义了 2 个测试、0 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L11)
```python
import pytest

from vllm.assets.video import VideoAsset
from vllm.multimodal import MULTIMODAL_REGISTRY
from vllm.multimodal.inputs import batched_tensors_equal
from vllm.multimodal.video import DynamicVideoBackend, VideoBackend

from ...utils import build_model_context
```
**EN:** Imports third-party packages like `pytest`, project helpers such as `vllm.assets.video.VideoAsset`, `vllm.multimodal.MULTIMODAL_REGISTRY`, `vllm.multimodal.inputs.batched_tensors_equal`.
**CN:** 导入第三方包（如 `pytest`）、项目内辅助模块（如 `vllm.assets.video.VideoAsset`、`vllm.multimodal.MULTIMODAL_REGISTRY`、`vllm.multimodal.inputs.batched_tensors_equal`）。

### Test / 测试: test_processor_override (L14-L68)
```python
@pytest.mark.parametrize("model_id", ["zai-org/GLM-4.1V-9B-Thinking"])
@pytest.mark.parametrize("expected_toks_per_frame", [299])
@pytest.mark.parametrize(
    "num_frames, fps, expected_grid_t",
    [
        # pre-sampled fixed frames (unexpected behavior,
        # but we still expect it to work without errors)
        (32, 1, 16),
        (32, 2, 16),
        (128, 1, 64),
        (128, 2, 64),
        # post-sampled frames (expected behavior)
        (-1, 1, 5),
        (-1, 2, 10),
    ],
)
def test_processor_override(
    model_id: str,
# ... 29 lines omitted for brevity ...
    # Ensure we have the right number of placeholders per num_crops size
    hf_processor = processor.info.get_hf_processor(**hf_processor_mm_kwargs)
    video_token_id = tokenizer.convert_tokens_to_ids(hf_processor.video_token)
    video_tok_count = processed_inputs["prompt_token_ids"].count(video_token_id)
    grid_t, _, _ = processed_inputs["mm_kwargs"].get_data()["video_grid_thw"][0]

    assert grid_t == expected_grid_t
    assert video_tok_count == expected_toks_per_frame * grid_t
```
**EN:** This test validates `test_processor_override`. It uses parameterization over `model_id`. Key inputs are `model_id`, `expected_toks_per_frame`, `expected_grid_t`, `fps`, `num_frames`. It drives client-facing request creation through the API surface under test. The main assertion is `grid_t == expected_grid_t` and `video_tok_count == expected_toks_per_frame * grid_t`.
**CN:** 这个测试验证 `test_processor_override`。 它通过参数化组合 `model_id`。 关键输入包括 `model_id`、`expected_toks_per_frame`、`expected_grid_t`、`fps`、`num_frames`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `grid_t == expected_grid_t` and `video_tok_count == expected_toks_per_frame * grid_t`。

### Test / 测试: test_video_loader_consistency (L71-L126)
```python
@pytest.mark.parametrize("model_id", ["zai-org/GLM-4.1V-9B-Thinking"])
@pytest.mark.parametrize("fps", [2])
@pytest.mark.parametrize("backend", ["opencv", "pyav"])
def test_video_loader_consistency(
    model_id: str,
    fps: int,
    backend: str,
):
    """
    Ensure dynamic video loader (pre-sampled by loader) and normal video
    loader (post-sampled by processor) produce same video processing outputs.
    """
    ctx = build_model_context(
        model_id,
        mm_processor_kwargs=None,
        limit_mm_per_prompt={"video": 1},
    )
    processor = MULTIMODAL_REGISTRY.create_processor(ctx.model_config)
# ... 30 lines omitted for brevity ...
        hf_processor_mm_kwargs=hf_processor_mm_kwargs,
    )

    assert static_outputs["prompt_token_ids"] == dynamic_outputs["prompt_token_ids"]
    assert batched_tensors_equal(
        static_outputs["mm_kwargs"].get_data(),
        dynamic_outputs["mm_kwargs"].get_data(),
    )
```
**EN:** This test validates `test_video_loader_consistency`. It uses parameterization over `model_id`. Key inputs are `model_id`, `fps`, `backend`. It drives client-facing request creation through the API surface under test. The main assertion is `len(dynamic_video) < len(static_video)` and `static_outputs['prompt_token_ids'] == dynamic_outputs['prompt_token_ids']`.
**CN:** 这个测试验证 `test_video_loader_consistency`。 它通过参数化组合 `model_id`。 关键输入包括 `model_id`、`fps`、`backend`。 它通过被测 API 表面触发面向客户端的请求创建流程。 核心断言是 `len(dynamic_video) < len(static_video)` and `static_outputs['prompt_token_ids'] == dynamic_outputs['prompt_token_ids']`。

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
- **Project / 项目内**: `vllm.assets.video.VideoAsset`, `vllm.multimodal.MULTIMODAL_REGISTRY`, `vllm.multimodal.inputs.batched_tensors_equal`, `vllm.multimodal.video.DynamicVideoBackend`, `vllm.multimodal.video.VideoBackend`
- **Local relative imports / 本地相对导入**: `...utils.build_model_context`
