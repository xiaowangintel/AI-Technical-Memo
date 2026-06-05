# test_tensor_schema.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/multimodal/processing/test_tensor_schema.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers multimodal processing and model-facing behavior. The file defines 1 test(s), 0 fixture(s), and 5 helper/class block(s) to validate this area. / [CN] 该文件覆盖多模态处理与面向模型的行为。它定义了 1 个测试、0 个 fixture，以及 5 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L40)
```python
import tempfile
from collections.abc import Iterable
from contextlib import contextmanager
from functools import partial
from typing import Any, TypeAlias

import numpy as np
import pytest
import torch
import torch.nn as nn
from PIL import Image

from vllm.config import ModelConfig, VllmConfig, set_current_vllm_config
from vllm.config.cache import CacheConfig
from vllm.config.multimodal import (
    AudioDummyOptions,
    BaseDummyOptions,
    ImageDummyOptions,
# ... 16 lines omitted for brevity ...
from ....utils import create_new_process_for_each_test
from ...registry import HF_EXAMPLE_MODELS
from ...utils import dummy_hf_overrides
from .test_common import get_model_ids_to_test, get_text_token_prompts
```
**EN:** Imports standard-library modules such as `collections.abc.Iterable`, `contextlib.contextmanager`, `functools.partial`, third-party packages like `PIL.Image`, `numpy`, `pytest`, project helpers such as `vllm.config.ModelConfig`, `vllm.config.VllmConfig`, `vllm.config.cache.CacheConfig`.
**CN:** 导入标准库模块（如 `collections.abc.Iterable`、`contextlib.contextmanager`、`functools.partial`）、第三方包（如 `PIL.Image`、`numpy`、`pytest`）、项目内辅助模块（如 `vllm.config.ModelConfig`、`vllm.config.VllmConfig`、`vllm.config.cache.CacheConfig`）。

### Module setup / 模块级配置: ImageInput, VideoInput, AudioInput (L42-L46)
```python
ImageInput = list[Image.Image]
VideoInput: TypeAlias = (
    list[Image.Image] | list[np.ndarray] | list[tuple[np.ndarray, dict[str, Any]]]
)
AudioInput = list[tuple[np.ndarray, int]]
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `ImageInput`, `VideoInput`, `AudioInput`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `ImageInput`、`VideoInput`、`AudioInput`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: _resize_data (L49-L72)
```python
def _resize_data(
    _data: Image.Image | np.ndarray, size_factor: float
) -> Image.Image | np.ndarray:
    assert size_factor <= 1, "Size factor must be less than 1"
    # Image input
    if isinstance(_data, Image.Image):
        W, H = _data.width, _data.height
        W, H = map(lambda x: int(x * size_factor), (W, H))
        return _data.resize((W, H))
    # Video input with PIL Images
    elif is_list_of(_data, Image.Image):
        W, H = next(iter(_data)).width, next(iter(_data)).height
        T = len(_data)
        T, W, H = map(lambda x: max(int(x * size_factor), 2), (T, W, H))
        return [d.resize((W, H)) for d in _data[:T]]
    # Video input with numpy arrays
    elif isinstance(_data, np.ndarray) and _data.ndim >= 4:
        T, H, W, C = _data.shape[-4:]
        T, H, W = map(lambda x: max(int(x * size_factor), 2), (T, H, W))
        return _data[..., :T, :H, :W, :C]
    # Audio input
    elif isinstance(_data, np.ndarray) and _data.ndim == 1:
        return _data[: int(len(_data) * size_factor)]
    raise AssertionError("This line should be unreachable.")
```
**EN:** This helper encapsulates reusable logic in `_resize_data`. Key inputs are `_data`, `size_factor`. It returns computed state or helper objects back to the caller. The main assertion is `size_factor <= 1`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_resize_data` 中。 关键输入包括 `_data`、`size_factor`。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `size_factor <= 1`。

### Helper / 辅助函数: resize_mm_data (L75-L83)
```python
def resize_mm_data(
    data: ImageInput | VideoInput | AudioInput, size_factors: tuple[float, ...]
) -> ImageInput | VideoInput | AudioInput:
    size_factors = size_factors[: len(data)]
    if is_list_of(data, (Image.Image, np.ndarray, list)):
        return [_resize_data(d, s) for d, s in zip(data, size_factors)]
    elif is_list_of(data, tuple):
        return [_resize_data(d, s) for (d, _), s in zip(data, size_factors)]
    raise ValueError("Unsupported multimodal data type.")
```
**EN:** This helper encapsulates reusable logic in `resize_mm_data`. Key inputs are `data`, `size_factors`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `resize_mm_data` 中。 关键输入包括 `data`、`size_factors`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: create_batched_mm_kwargs (L86-L124)
```python
def create_batched_mm_kwargs(
    model_config: ModelConfig,
    processor: BaseMultiModalProcessor,
    size_factors: tuple[float, ...] = (1.0, 0.5, 0.25),
) -> Iterable[tuple[str, int, BatchedTensorInputs]]:
    processing_info = processor.info
    dummy_inputs = processor.dummy_inputs
    supported_mm_limits = processing_info.get_supported_mm_limits()
    mm_counts = {
        modality: 3 if limit is None else limit
        for modality, limit in supported_mm_limits.items()
    }
    processor_inputs = dummy_inputs.get_dummy_processor_inputs(
        seq_len=model_config.max_model_len,
        mm_counts=mm_counts,
        mm_options={},
    )
    mm_items = processor_inputs.mm_data_items
# ... 13 lines omitted for brevity ...

    return group_and_batch_mm_kwargs(
        [
            (modality, item)
            for modality in supported_mm_limits
            for item in mm_kwargs[modality]
        ]
    )
```
**EN:** This helper encapsulates reusable logic in `create_batched_mm_kwargs`. Key inputs are `model_config`, `processor`, `size_factors`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `create_batched_mm_kwargs` 中。 关键输入包括 `model_config`、`processor`、`size_factors`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper / 辅助函数: initialize_dummy_model (L128-L155)
```python
@contextmanager
def initialize_dummy_model(
    model_cls: type[nn.Module],
    model_config: ModelConfig,
):
    temp_file = tempfile.mkstemp()[1]
    current_device = torch.get_default_device()
    vllm_config = VllmConfig(
        model_config=model_config, cache_config=CacheConfig(block_size=16)
    )
    with set_current_vllm_config(vllm_config=vllm_config):
        init_distributed_environment(
            world_size=1,
            rank=0,
            distributed_init_method=f"file://{temp_file}",
            local_rank=0,
            backend="nccl",
        )
        initialize_model_parallel(tensor_model_parallel_size=1)

        with set_default_torch_dtype(model_config.dtype):
            torch.set_default_device(current_platform.device_type)
            model = model_cls(vllm_config=vllm_config)
            torch.set_default_device(current_device)
        yield model

    del model
    cleanup_dist_env_and_memory()
```
**EN:** This helper encapsulates reusable logic in `initialize_dummy_model`. Key inputs are `model_cls`, `model_config`.
**CN:** 这个辅助函数将可复用逻辑封装在 `initialize_dummy_model` 中。 关键输入包括 `model_cls`、`model_config`。

### Test / 测试: test_model_tensor_schema (L158-L254)
```python
@create_new_process_for_each_test()
@pytest.mark.parametrize("model_id", get_model_ids_to_test())
def test_model_tensor_schema(model_id: str):
    if model_id == "moonshotai/Kimi-K2.5":
        # FIXME(Isotr0py): Fix Kimi-K2.5's offline inference about vision chunks.
        pytest.skip(
            "Kimi-K2.5's offline inference has issues about vision chunks. Fix later."
        )

    model_info = HF_EXAMPLE_MODELS.find_hf_info(model_id)
    model_info.check_available_online(on_fail="skip")
    model_info.check_transformers_version(
        on_fail="skip",
        check_max_version=False,
        check_version_reason="vllm",
    )

    model_arch = next(
# ... 71 lines omitted for brevity ...
    with initialize_dummy_model(model_cls, model_config) as model:
        for modality, _, mm_kwargs in create_batched_mm_kwargs(model_config, processor):
            for method_name in inputs_parse_methods:
                print(
                    f"Testing `{method_name}` with modality={modality} "
                    f"and mm_kwargs{list(mm_kwargs.keys())}"
                )
                getattr(model, method_name)(modality=modality, **mm_kwargs)
```
**EN:** This test validates `test_model_tensor_schema`. It uses parameterization over `model_id`. Key inputs are `model_id`. The logic can skip unsupported environments when prerequisites are not satisfied. The main assertion is `supports_multimodal(model_cls)`.
**CN:** 这个测试验证 `test_model_tensor_schema`。 它通过参数化组合 `model_id`。 关键输入包括 `model_id`。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。 核心断言是 `supports_multimodal(model_cls)`。

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
- **Stdlib / 标准库**: `collections.abc.Iterable`, `contextlib.contextmanager`, `functools.partial`, `tempfile`, `typing.Any`, `typing.TypeAlias`
- **Third-party / 第三方**: `PIL.Image`, `numpy`, `pytest`, `torch`, `torch.nn`
- **Project / 项目内**: `vllm.config.ModelConfig`, `vllm.config.VllmConfig`, `vllm.config.cache.CacheConfig`, `vllm.config.multimodal.AudioDummyOptions`, `vllm.config.multimodal.BaseDummyOptions`, `vllm.config.multimodal.ImageDummyOptions`, `vllm.config.multimodal.VideoDummyOptions`, `vllm.config.set_current_vllm_config`, `vllm.distributed.cleanup_dist_env_and_memory`, `vllm.distributed.init_distributed_environment`, `vllm.distributed.initialize_model_parallel`, `vllm.model_executor.models.interfaces.supports_multimodal`, `vllm.multimodal.BatchedTensorInputs`, `vllm.multimodal.MULTIMODAL_REGISTRY`, `vllm.multimodal.processing.BaseMultiModalProcessor`, `vllm.multimodal.processing.InputProcessingContext`, `vllm.multimodal.utils.group_and_batch_mm_kwargs`, `vllm.platforms.current_platform`, `vllm.tokenizers.cached_tokenizer_from_config`, `vllm.utils.collection_utils.is_list_of`, `vllm.utils.torch_utils.set_default_torch_dtype`
- **Local relative imports / 本地相对导入**: `....utils.create_new_process_for_each_test`, `...registry.HF_EXAMPLE_MODELS`, `...utils.dummy_hf_overrides`, `.test_common.get_model_ids_to_test`, `.test_common.get_text_token_prompts`
