# runners.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/generation/vlm_utils/runners.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides shared test utilities for multimodal processing and model-facing behavior. The file exposes 6 helper/class block(s) used by nearby tests. / [CN] 为多模态处理与面向模型的行为提供共享测试工具。该文件暴露了 6 个辅助函数/类块，供附近测试复用。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L7-L17)
```python
from pathlib import PosixPath

from .....conftest import (
    AudioTestAssets,
    HfRunner,
    ImageTestAssets,
    VideoTestAssets,
    VllmRunner,
)
from . import builders, core
from .types import ExpandableVLMTestArgs, VLMTestInfo
```
**EN:** Imports standard-library modules such as `pathlib.PosixPath`, project helpers such as `.....conftest.AudioTestAssets`, `.....conftest.HfRunner`, `.....conftest.ImageTestAssets`.
**CN:** 导入标准库模块（如 `pathlib.PosixPath`）、项目内辅助模块（如 `.....conftest.AudioTestAssets`、`.....conftest.HfRunner`、`.....conftest.ImageTestAssets`）。

### Helper / 辅助函数: run_single_image_test (L21-L46)
```python
def run_single_image_test(
    *,
    tmp_path: PosixPath,
    model_test_info: VLMTestInfo,
    test_case: ExpandableVLMTestArgs,
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    image_assets: ImageTestAssets,
):
    assert test_case.size_wrapper is not None
    inputs = builders.build_single_image_inputs_from_test_info(
        model_test_info, image_assets, test_case.size_wrapper, tmp_path
    )

    core.run_test(
        hf_runner=hf_runner,
        vllm_runner=vllm_runner,
        inputs=inputs,
        model=test_case.model,
        dtype=test_case.dtype,
        max_tokens=test_case.max_tokens,
        num_logprobs=test_case.num_logprobs,
        limit_mm_per_prompt={"image": 1},
        distributed_executor_backend=test_case.distributed_executor_backend,
        **model_test_info.get_non_parametrized_runner_kwargs(),
    )
```
**EN:** This helper encapsulates reusable logic in `run_single_image_test`. The main assertion is `test_case.size_wrapper is not None`.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_single_image_test` 中。 核心断言是 `test_case.size_wrapper is not None`。

### Helper / 辅助函数: run_multi_image_test (L49-L74)
```python
def run_multi_image_test(
    *,
    tmp_path: PosixPath,
    model_test_info: VLMTestInfo,
    test_case: ExpandableVLMTestArgs,
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    image_assets: ImageTestAssets,
):
    assert test_case.size_wrapper is not None
    inputs = builders.build_multi_image_inputs_from_test_info(
        model_test_info, image_assets, test_case.size_wrapper, tmp_path
    )

    core.run_test(
        hf_runner=hf_runner,
        vllm_runner=vllm_runner,
        inputs=inputs,
        model=test_case.model,
        dtype=test_case.dtype,
        max_tokens=test_case.max_tokens,
        num_logprobs=test_case.num_logprobs,
        limit_mm_per_prompt={"image": len(image_assets)},
        distributed_executor_backend=test_case.distributed_executor_backend,
        **model_test_info.get_non_parametrized_runner_kwargs(),
    )
```
**EN:** This helper encapsulates reusable logic in `run_multi_image_test`. The main assertion is `test_case.size_wrapper is not None`.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_multi_image_test` 中。 核心断言是 `test_case.size_wrapper is not None`。

### Helper / 辅助函数: run_embedding_test (L77-L102)
```python
def run_embedding_test(
    *,
    model_test_info: VLMTestInfo,
    test_case: ExpandableVLMTestArgs,
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    image_assets: ImageTestAssets,
):
    assert test_case.size_wrapper is not None
    inputs, vllm_embeddings = builders.build_embedding_inputs_from_test_info(
        model_test_info, image_assets, test_case.size_wrapper
    )

    core.run_test(
        hf_runner=hf_runner,
        vllm_runner=vllm_runner,
        inputs=inputs,
        model=test_case.model,
        dtype=test_case.dtype,
        max_tokens=test_case.max_tokens,
        num_logprobs=test_case.num_logprobs,
        limit_mm_per_prompt={"image": 1},
        vllm_embeddings=vllm_embeddings,
        distributed_executor_backend=test_case.distributed_executor_backend,
        **model_test_info.get_non_parametrized_runner_kwargs(),
    )
```
**EN:** This helper encapsulates reusable logic in `run_embedding_test`. The main assertion is `test_case.size_wrapper is not None`.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_embedding_test` 中。 核心断言是 `test_case.size_wrapper is not None`。

### Helper / 辅助函数: run_video_test (L105-L134)
```python
def run_video_test(
    *,
    model_test_info: VLMTestInfo,
    test_case: ExpandableVLMTestArgs,
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    video_assets: VideoTestAssets,
):
    assert test_case.size_wrapper is not None
    assert test_case.num_video_frames is not None
    inputs = builders.build_video_inputs_from_test_info(
        model_test_info,
        video_assets,
        test_case.size_wrapper,
        test_case.num_video_frames,
        test_case.needs_video_metadata,
    )

    core.run_test(
        hf_runner=hf_runner,
        vllm_runner=vllm_runner,
        inputs=inputs,
        model=test_case.model,
        dtype=test_case.dtype,
        max_tokens=test_case.max_tokens,
        num_logprobs=test_case.num_logprobs,
        limit_mm_per_prompt={"video": len(video_assets)},
        distributed_executor_backend=test_case.distributed_executor_backend,
        **model_test_info.get_non_parametrized_runner_kwargs(),
    )
```
**EN:** This helper encapsulates reusable logic in `run_video_test`. The main assertion is `test_case.size_wrapper is not None` and `test_case.num_video_frames is not None`.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_video_test` 中。 核心断言是 `test_case.size_wrapper is not None` and `test_case.num_video_frames is not None`。

### Helper / 辅助函数: run_audio_test (L137-L158)
```python
def run_audio_test(
    *,
    model_test_info: VLMTestInfo,
    test_case: ExpandableVLMTestArgs,
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    audio_assets: AudioTestAssets,
):
    inputs = builders.build_audio_inputs_from_test_info(model_test_info, audio_assets)

    core.run_test(
        hf_runner=hf_runner,
        vllm_runner=vllm_runner,
        inputs=inputs,
        model=test_case.model,
        dtype=test_case.dtype,
        max_tokens=test_case.max_tokens,
        num_logprobs=test_case.num_logprobs,
        limit_mm_per_prompt={"audio": 1},
        distributed_executor_backend=test_case.distributed_executor_backend,
        **model_test_info.get_non_parametrized_runner_kwargs(),
    )
```
**EN:** This helper encapsulates reusable logic in `run_audio_test`.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_audio_test` 中。

### Helper / 辅助函数: run_custom_inputs_test (L161-L190)
```python
def run_custom_inputs_test(
    *,
    model_test_info: VLMTestInfo,
    test_case: ExpandableVLMTestArgs,
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
):
    # Custom test cases can provide inputs directly, but they need to
    # explicitly provided a CustomTestConfig, which wraps the inputs and
    # the limit_mm_per_prompt
    assert test_case.custom_test_opts is not None

    inputs = test_case.custom_test_opts.inputs
    limit_mm_per_prompt = test_case.custom_test_opts.limit_mm_per_prompt
    # Inputs and limit_mm_per_prompt should all be set
    assert inputs is not None
    assert limit_mm_per_prompt is not None

    core.run_test(
        hf_runner=hf_runner,
        vllm_runner=vllm_runner,
        inputs=inputs,
        model=test_case.model,
        dtype=test_case.dtype,
        max_tokens=test_case.max_tokens,
        num_logprobs=test_case.num_logprobs,
        limit_mm_per_prompt=limit_mm_per_prompt,
        distributed_executor_backend=test_case.distributed_executor_backend,
        **model_test_info.get_non_parametrized_runner_kwargs(),
    )
```
**EN:** This helper encapsulates reusable logic in `run_custom_inputs_test`. The main assertion is `test_case.custom_test_opts is not None` and `inputs is not None`.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_custom_inputs_test` 中。 核心断言是 `test_case.custom_test_opts is not None` and `inputs is not None`。

## Key Concepts / 关键概念
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `pathlib.PosixPath`
- **Local relative imports / 本地相对导入**: `.....conftest.AudioTestAssets`, `.....conftest.HfRunner`, `.....conftest.ImageTestAssets`, `.....conftest.VideoTestAssets`, `.....conftest.VllmRunner`, `..builders`, `..core`, `.types.ExpandableVLMTestArgs`, `.types.VLMTestInfo`
