# test_common.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/generation/test_common.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers multimodal processing and model-facing behavior. The file defines 12 test(s), 0 fixture(s), and 2 helper/class block(s) to validate this area. / [CN] 该文件覆盖多模态处理与面向模型的行为。它定义了 12 个测试、0 个 fixture，以及 2 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L7-L41)
```python
import math
from collections import defaultdict
from pathlib import PosixPath

import pytest
from packaging.version import Version
from transformers import (
    AutoModel,
    AutoModelForCausalLM,
    AutoModelForImageTextToText,
    AutoModelForTextToWaveform,
)
from transformers import __version__ as TRANSFORMERS_VERSION

from vllm.platforms import current_platform
from vllm.utils.func_utils import identity

from ....conftest import (
# ... 13 lines omitted for brevity ...
    ExpandableVLMTestArgs,
    VLMTestInfo,
    VLMTestType,
)
```
**EN:** Imports standard-library modules such as `collections.defaultdict`, `math`, `pathlib.PosixPath`, third-party packages like `packaging.version.Version`, `pytest`, `transformers.AutoModel`, project helpers such as `vllm.platforms.current_platform`, `vllm.utils.func_utils.identity`, `....conftest.AudioTestAssets`.
**CN:** 导入标准库模块（如 `collections.defaultdict`、`math`、`pathlib.PosixPath`）、第三方包（如 `packaging.version.Version`、`pytest`、`transformers.AutoModel`）、项目内辅助模块（如 `vllm.platforms.current_platform`、`vllm.utils.func_utils.identity`、`....conftest.AudioTestAssets`）。

### Module setup / 模块级配置: COMMON_BROADCAST_SETTINGS (L43-L54)
```python
COMMON_BROADCAST_SETTINGS = {
    "test_type": VLMTestType.IMAGE,
    "dtype": "half",
    "max_tokens": 5,
    "tensor_parallel_size": 2,
    "hf_model_kwargs": {"device_map": "auto"},
    "image_size_factors": [(0.25, 0.5, 1.0)],
    "distributed_executor_backend": (
        "ray",
        "mp",
    ),
}
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `COMMON_BROADCAST_SETTINGS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `COMMON_BROADCAST_SETTINGS`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: _granite4_vision_vllm_to_hf_output (L90-L109)
```python
def _granite4_vision_vllm_to_hf_output(vllm_output, model):
    """Post-processor for granite4_vision vLLM output.

    Self-contained to avoid calling AutoConfig/AutoTokenizer without
    trust_remote_code (needed while the model is not in upstream HF).
    """
    output_ids, output_str, out_logprobs = vllm_output
    mm_token_id = 100352
    hf_output_ids = [
        token_id
        for idx, token_id in enumerate(output_ids)
        if token_id != mm_token_id or idx == 0 or output_ids[idx - 1] != mm_token_id
    ]
    hf_output_str = (
        output_str[1:] if output_str and output_str[0] == " " else output_str
    )
    eos_token_id = 100257
    if hf_output_ids and hf_output_ids[-1] == eos_token_id:
        hf_output_str = hf_output_str + "<|end_of_text|>"
    return hf_output_ids, hf_output_str, out_logprobs
```
**EN:** This helper encapsulates reusable logic in `_granite4_vision_vllm_to_hf_output`. Key inputs are `vllm_output`, `model`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_granite4_vision_vllm_to_hf_output` 中。 关键输入包括 `vllm_output`、`model`。 它把计算得到的状态或辅助对象返回给调用方。

### Module setup / 模块级配置: VLM_TEST_SETTINGS (L112-L1126)
```python
VLM_TEST_SETTINGS = {
    #### Core tests to always run in the CI
    "llava": VLMTestInfo(
        models=["llava-hf/llava-1.5-7b-hf"],
        test_type=(VLMTestType.EMBEDDING, VLMTestType.IMAGE, VLMTestType.CUSTOM_INPUTS),
        prompt_formatter=lambda img_prompt: f"USER: {img_prompt}\nASSISTANT:",
        convert_assets_to_embeddings=model_utils.get_llava_embeddings,
        max_model_len=4096,
        auto_cls=AutoModelForImageTextToText,
        vllm_output_post_proc=model_utils.llava_image_vllm_to_hf_output,
        custom_test_opts=[
            CustomTestOptions(
                inputs=custom_inputs.multi_image_multi_aspect_ratio_inputs(
                    formatter=lambda img_prompt: f"USER: {img_prompt}\nASSISTANT:"
                ),
                limit_mm_per_prompt={"image": 4},
# ... 993 lines omitted for brevity ...
                inputs=custom_inputs.windows_attention_image_qwen2_5_vl(),
                limit_mm_per_prompt={"image": 1},
            )
        ],
    ),
}
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `VLM_TEST_SETTINGS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `VLM_TEST_SETTINGS`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: _mark_splits (L1129-L1158)
```python
def _mark_splits(
    test_settings: dict[str, VLMTestInfo],
    *,
    num_groups: int,
) -> dict[str, VLMTestInfo]:
    name_by_test_info_id = {id(v): k for k, v in test_settings.items()}
    test_infos_by_model = defaultdict[str, list[VLMTestInfo]](list)

    for info in test_settings.values():
        for model in info.models:
            test_infos_by_model[model].append(info)

    models = sorted(test_infos_by_model.keys())
    split_size = math.ceil(len(models) / num_groups)

    new_test_settings = dict[str, VLMTestInfo]()

    for i in range(num_groups):
        models_in_group = models[i * split_size : (i + 1) * split_size]

        for model in models_in_group:
            for info in test_infos_by_model[model]:
                new_marks = (info.marks or []) + [pytest.mark.split(group=i)]
                new_info = info._replace(marks=new_marks)
                new_test_settings[name_by_test_info_id[id(info)]] = new_info

    missing_keys = test_settings.keys() - new_test_settings.keys()
    assert not missing_keys, f"Missing keys: {missing_keys}"

    return new_test_settings
```
**EN:** This helper encapsulates reusable logic in `_mark_splits`. Key inputs are `test_settings`. It returns computed state or helper objects back to the caller. The main assertion is `not missing_keys`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_mark_splits` 中。 关键输入包括 `test_settings`。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `not missing_keys`。

### Module setup / 模块级配置: VLM_TEST_SETTINGS (L1161-L1161)
```python
VLM_TEST_SETTINGS = _mark_splits(VLM_TEST_SETTINGS, num_groups=2)
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `VLM_TEST_SETTINGS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `VLM_TEST_SETTINGS`。后续辅助函数和测试会复用这些值，以减少重复。

### Test / 测试: test_single_image_models (L1172-L1196)
```python
@pytest.mark.parametrize(
    "model_type,test_case",
    get_parametrized_options(
        VLM_TEST_SETTINGS,
        test_type=VLMTestType.IMAGE,
        create_new_process_for_each_test=False,
    ),
)
def test_single_image_models(
    tmp_path: PosixPath,
    model_type: str,
    test_case: ExpandableVLMTestArgs,
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    image_assets: ImageTestAssets,
):
    model_test_info = VLM_TEST_SETTINGS[model_type]
    runners.run_single_image_test(
        tmp_path=tmp_path,
        model_test_info=model_test_info,
        test_case=test_case,
        hf_runner=hf_runner,
        vllm_runner=vllm_runner,
        image_assets=image_assets,
    )
```
**EN:** This test validates `test_single_image_models`. It uses parameterization over `model_type`, `test_case`. Key inputs are `tmp_path`, `model_type`, `test_case`, `hf_runner`, `vllm_runner`, `image_assets`.
**CN:** 这个测试验证 `test_single_image_models`。 它通过参数化组合 `model_type`、`test_case`。 关键输入包括 `tmp_path`、`model_type`、`test_case`、`hf_runner`、`vllm_runner`、`image_assets`。

### Test / 测试: test_multi_image_models (L1199-L1223)
```python
@pytest.mark.parametrize(
    "model_type,test_case",
    get_parametrized_options(
        VLM_TEST_SETTINGS,
        test_type=VLMTestType.MULTI_IMAGE,
        create_new_process_for_each_test=False,
    ),
)
def test_multi_image_models(
    tmp_path: PosixPath,
    model_type: str,
    test_case: ExpandableVLMTestArgs,
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    image_assets: ImageTestAssets,
):
    model_test_info = VLM_TEST_SETTINGS[model_type]
    runners.run_multi_image_test(
        tmp_path=tmp_path,
        model_test_info=model_test_info,
        test_case=test_case,
        hf_runner=hf_runner,
        vllm_runner=vllm_runner,
        image_assets=image_assets,
    )
```
**EN:** This test validates `test_multi_image_models`. It uses parameterization over `model_type`, `test_case`. Key inputs are `tmp_path`, `model_type`, `test_case`, `hf_runner`, `vllm_runner`, `image_assets`.
**CN:** 这个测试验证 `test_multi_image_models`。 它通过参数化组合 `model_type`、`test_case`。 关键输入包括 `tmp_path`、`model_type`、`test_case`、`hf_runner`、`vllm_runner`、`image_assets`。

### Test / 测试: test_image_embedding_models (L1226-L1248)
```python
@pytest.mark.parametrize(
    "model_type,test_case",
    get_parametrized_options(
        VLM_TEST_SETTINGS,
        test_type=VLMTestType.EMBEDDING,
        create_new_process_for_each_test=False,
    ),
)
def test_image_embedding_models(
    model_type: str,
    test_case: ExpandableVLMTestArgs,
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    image_assets: ImageTestAssets,
):
    model_test_info = VLM_TEST_SETTINGS[model_type]
    runners.run_embedding_test(
        model_test_info=model_test_info,
        test_case=test_case,
        hf_runner=hf_runner,
        vllm_runner=vllm_runner,
        image_assets=image_assets,
    )
```
**EN:** This test validates `test_image_embedding_models`. It uses parameterization over `model_type`, `test_case`. Key inputs are `model_type`, `test_case`, `hf_runner`, `vllm_runner`, `image_assets`.
**CN:** 这个测试验证 `test_image_embedding_models`。 它通过参数化组合 `model_type`、`test_case`。 关键输入包括 `model_type`、`test_case`、`hf_runner`、`vllm_runner`、`image_assets`。

### Test / 测试: test_video_models (L1251-L1273)
```python
@pytest.mark.parametrize(
    "model_type,test_case",
    get_parametrized_options(
        VLM_TEST_SETTINGS,
        test_type=VLMTestType.VIDEO,
        create_new_process_for_each_test=False,
    ),
)
def test_video_models(
    model_type: str,
    test_case: ExpandableVLMTestArgs,
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    video_assets: VideoTestAssets,
):
    model_test_info = VLM_TEST_SETTINGS[model_type]
    runners.run_video_test(
        model_test_info=model_test_info,
        test_case=test_case,
        hf_runner=hf_runner,
        vllm_runner=vllm_runner,
        video_assets=video_assets,
    )
```
**EN:** This test validates `test_video_models`. It uses parameterization over `model_type`, `test_case`. Key inputs are `model_type`, `test_case`, `hf_runner`, `vllm_runner`, `video_assets`.
**CN:** 这个测试验证 `test_video_models`。 它通过参数化组合 `model_type`、`test_case`。 关键输入包括 `model_type`、`test_case`、`hf_runner`、`vllm_runner`、`video_assets`。

### Test / 测试: test_audio_models (L1276-L1298)
```python
@pytest.mark.parametrize(
    "model_type,test_case",
    get_parametrized_options(
        VLM_TEST_SETTINGS,
        test_type=VLMTestType.AUDIO,
        create_new_process_for_each_test=False,
    ),
)
def test_audio_models(
    model_type: str,
    test_case: ExpandableVLMTestArgs,
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    audio_assets: AudioTestAssets,
):
    model_test_info = VLM_TEST_SETTINGS[model_type]
    runners.run_audio_test(
        model_test_info=model_test_info,
        test_case=test_case,
        hf_runner=hf_runner,
        vllm_runner=vllm_runner,
        audio_assets=audio_assets,
    )
```
**EN:** This test validates `test_audio_models`. It uses parameterization over `model_type`, `test_case`. Key inputs are `model_type`, `test_case`, `hf_runner`, `vllm_runner`, `audio_assets`.
**CN:** 这个测试验证 `test_audio_models`。 它通过参数化组合 `model_type`、`test_case`。 关键输入包括 `model_type`、`test_case`、`hf_runner`、`vllm_runner`、`audio_assets`。

### Test / 测试: test_custom_inputs_models (L1301-L1321)
```python
@pytest.mark.parametrize(
    "model_type,test_case",
    get_parametrized_options(
        VLM_TEST_SETTINGS,
        test_type=VLMTestType.CUSTOM_INPUTS,
        create_new_process_for_each_test=False,
    ),
)
def test_custom_inputs_models(
    model_type: str,
    test_case: ExpandableVLMTestArgs,
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
):
    model_test_info = VLM_TEST_SETTINGS[model_type]
    runners.run_custom_inputs_test(
        model_test_info=model_test_info,
        test_case=test_case,
        hf_runner=hf_runner,
        vllm_runner=vllm_runner,
    )
```
**EN:** This test validates `test_custom_inputs_models`. It uses parameterization over `model_type`, `test_case`. Key inputs are `model_type`, `test_case`, `hf_runner`, `vllm_runner`.
**CN:** 这个测试验证 `test_custom_inputs_models`。 它通过参数化组合 `model_type`、`test_case`。 关键输入包括 `model_type`、`test_case`、`hf_runner`、`vllm_runner`。

### Test / 测试: test_single_image_models_heavy (L1325-L1350)
```python
@pytest.mark.parametrize(
    "model_type,test_case",
    get_parametrized_options(
        VLM_TEST_SETTINGS,
        test_type=VLMTestType.IMAGE,
        create_new_process_for_each_test=True,
    ),
)
@create_new_process_for_each_test()
def test_single_image_models_heavy(
    tmp_path: PosixPath,
    model_type: str,
    test_case: ExpandableVLMTestArgs,
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    image_assets: ImageTestAssets,
):
    model_test_info = VLM_TEST_SETTINGS[model_type]
    runners.run_single_image_test(
        tmp_path=tmp_path,
        model_test_info=model_test_info,
        test_case=test_case,
        hf_runner=hf_runner,
        vllm_runner=vllm_runner,
        image_assets=image_assets,
    )
```
**EN:** This test validates `test_single_image_models_heavy`. It uses parameterization over `model_type`, `test_case`. Key inputs are `tmp_path`, `model_type`, `test_case`, `hf_runner`, `vllm_runner`, `image_assets`.
**CN:** 这个测试验证 `test_single_image_models_heavy`。 它通过参数化组合 `model_type`、`test_case`。 关键输入包括 `tmp_path`、`model_type`、`test_case`、`hf_runner`、`vllm_runner`、`image_assets`。

### Test / 测试: test_multi_image_models_heavy (L1353-L1378)
```python
@pytest.mark.parametrize(
    "model_type,test_case",
    get_parametrized_options(
        VLM_TEST_SETTINGS,
        test_type=VLMTestType.MULTI_IMAGE,
        create_new_process_for_each_test=True,
    ),
)
@create_new_process_for_each_test()
def test_multi_image_models_heavy(
    tmp_path: PosixPath,
    model_type: str,
    test_case: ExpandableVLMTestArgs,
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    image_assets: ImageTestAssets,
):
    model_test_info = VLM_TEST_SETTINGS[model_type]
    runners.run_multi_image_test(
        tmp_path=tmp_path,
        model_test_info=model_test_info,
        test_case=test_case,
        hf_runner=hf_runner,
        vllm_runner=vllm_runner,
        image_assets=image_assets,
    )
```
**EN:** This test validates `test_multi_image_models_heavy`. It uses parameterization over `model_type`, `test_case`. Key inputs are `tmp_path`, `model_type`, `test_case`, `hf_runner`, `vllm_runner`, `image_assets`.
**CN:** 这个测试验证 `test_multi_image_models_heavy`。 它通过参数化组合 `model_type`、`test_case`。 关键输入包括 `tmp_path`、`model_type`、`test_case`、`hf_runner`、`vllm_runner`、`image_assets`。

### Test / 测试: test_image_embedding_models_heavy (L1381-L1404)
```python
@pytest.mark.parametrize(
    "model_type,test_case",
    get_parametrized_options(
        VLM_TEST_SETTINGS,
        test_type=VLMTestType.EMBEDDING,
        create_new_process_for_each_test=True,
    ),
)
@create_new_process_for_each_test()
def test_image_embedding_models_heavy(
    model_type: str,
    test_case: ExpandableVLMTestArgs,
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    image_assets: ImageTestAssets,
):
    model_test_info = VLM_TEST_SETTINGS[model_type]
    runners.run_embedding_test(
        model_test_info=model_test_info,
        test_case=test_case,
        hf_runner=hf_runner,
        vllm_runner=vllm_runner,
        image_assets=image_assets,
    )
```
**EN:** This test validates `test_image_embedding_models_heavy`. It uses parameterization over `model_type`, `test_case`. Key inputs are `model_type`, `test_case`, `hf_runner`, `vllm_runner`, `image_assets`.
**CN:** 这个测试验证 `test_image_embedding_models_heavy`。 它通过参数化组合 `model_type`、`test_case`。 关键输入包括 `model_type`、`test_case`、`hf_runner`、`vllm_runner`、`image_assets`。

### Test / 测试: test_video_models_heavy (L1407-L1429)
```python
@pytest.mark.parametrize(
    "model_type,test_case",
    get_parametrized_options(
        VLM_TEST_SETTINGS,
        test_type=VLMTestType.VIDEO,
        create_new_process_for_each_test=True,
    ),
)
def test_video_models_heavy(
    model_type: str,
    test_case: ExpandableVLMTestArgs,
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    video_assets: VideoTestAssets,
):
    model_test_info = VLM_TEST_SETTINGS[model_type]
    runners.run_video_test(
        model_test_info=model_test_info,
        test_case=test_case,
        hf_runner=hf_runner,
        vllm_runner=vllm_runner,
        video_assets=video_assets,
    )
```
**EN:** This test validates `test_video_models_heavy`. It uses parameterization over `model_type`, `test_case`. Key inputs are `model_type`, `test_case`, `hf_runner`, `vllm_runner`, `video_assets`.
**CN:** 这个测试验证 `test_video_models_heavy`。 它通过参数化组合 `model_type`、`test_case`。 关键输入包括 `model_type`、`test_case`、`hf_runner`、`vllm_runner`、`video_assets`。

### Test / 测试: test_audio_models_heavy (L1432-L1454)
```python
@pytest.mark.parametrize(
    "model_type,test_case",
    get_parametrized_options(
        VLM_TEST_SETTINGS,
        test_type=VLMTestType.AUDIO,
        create_new_process_for_each_test=True,
    ),
)
def test_audio_models_heavy(
    model_type: str,
    test_case: ExpandableVLMTestArgs,
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
    audio_assets: AudioTestAssets,
):
    model_test_info = VLM_TEST_SETTINGS[model_type]
    runners.run_audio_test(
        model_test_info=model_test_info,
        test_case=test_case,
        hf_runner=hf_runner,
        vllm_runner=vllm_runner,
        audio_assets=audio_assets,
    )
```
**EN:** This test validates `test_audio_models_heavy`. It uses parameterization over `model_type`, `test_case`. Key inputs are `model_type`, `test_case`, `hf_runner`, `vllm_runner`, `audio_assets`.
**CN:** 这个测试验证 `test_audio_models_heavy`。 它通过参数化组合 `model_type`、`test_case`。 关键输入包括 `model_type`、`test_case`、`hf_runner`、`vllm_runner`、`audio_assets`。

### Test / 测试: test_custom_inputs_models_heavy (L1457-L1478)
```python
@pytest.mark.parametrize(
    "model_type,test_case",
    get_parametrized_options(
        VLM_TEST_SETTINGS,
        test_type=VLMTestType.CUSTOM_INPUTS,
        create_new_process_for_each_test=True,
    ),
)
@create_new_process_for_each_test()
def test_custom_inputs_models_heavy(
    model_type: str,
    test_case: ExpandableVLMTestArgs,
    hf_runner: type[HfRunner],
    vllm_runner: type[VllmRunner],
):
    model_test_info = VLM_TEST_SETTINGS[model_type]
    runners.run_custom_inputs_test(
        model_test_info=model_test_info,
        test_case=test_case,
        hf_runner=hf_runner,
        vllm_runner=vllm_runner,
    )
```
**EN:** This test validates `test_custom_inputs_models_heavy`. It uses parameterization over `model_type`, `test_case`. Key inputs are `model_type`, `test_case`, `hf_runner`, `vllm_runner`.
**CN:** 这个测试验证 `test_custom_inputs_models_heavy`。 它通过参数化组合 `model_type`、`test_case`。 关键输入包括 `model_type`、`test_case`、`hf_runner`、`vllm_runner`。

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
- **Stdlib / 标准库**: `collections.defaultdict`, `math`, `pathlib.PosixPath`
- **Third-party / 第三方**: `packaging.version.Version`, `pytest`, `transformers.AutoModel`, `transformers.AutoModelForCausalLM`, `transformers.AutoModelForImageTextToText`, `transformers.AutoModelForTextToWaveform`, `transformers.__version__`
- **Project / 项目内**: `vllm.platforms.current_platform`, `vllm.utils.func_utils.identity`
- **Local relative imports / 本地相对导入**: `....conftest.AudioTestAssets`, `....conftest.HfRunner`, `....conftest.IMAGE_ASSETS`, `....conftest.ImageTestAssets`, `....conftest.VideoTestAssets`, `....conftest.VllmRunner`, `....utils.create_new_process_for_each_test`, `....utils.large_gpu_mark`, `....utils.multi_gpu_marks`, `...utils.check_outputs_equal`, `.vlm_utils.case_filtering.get_parametrized_options`, `.vlm_utils.custom_inputs`, `.vlm_utils.model_utils`, `.vlm_utils.runners`, `.vlm_utils.types.CustomTestOptions`, `.vlm_utils.types.ExpandableVLMTestArgs`, `.vlm_utils.types.VLMTestInfo`, `.vlm_utils.types.VLMTestType`
