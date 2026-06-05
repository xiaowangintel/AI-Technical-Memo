# test_registry.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/test_registry.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers model registry metadata and model-facing behavior. The file defines 4 test(s), 0 fixture(s), and 0 helper/class block(s) to validate this area. / [CN] 该文件覆盖模型注册表元数据与面向模型的行为。它定义了 4 个测试、0 个 fixture，以及 0 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L27)
```python
import warnings

import pytest
import torch.cuda

from vllm.model_executor.models import (
    is_pooling_model,
    is_text_generation_model,
    supports_multimodal,
)
from vllm.model_executor.models.adapters import (
    as_embedding_model,
    as_seq_cls_model,
)
from vllm.model_executor.models.registry import (
    _MULTIMODAL_MODELS,
    _SPECULATIVE_DECODING_MODELS,
    _TEXT_GENERATION_MODELS,
    ModelRegistry,
)
from vllm.platforms import current_platform

from ..utils import create_new_process_for_each_test
from .registry import HF_EXAMPLE_MODELS
```
**EN:** Imports standard-library modules such as `importlib.util`, `warnings`, third-party packages like `pytest`, `torch.cuda`, project helpers such as `vllm.model_executor.models.adapters.as_embedding_model`, `vllm.model_executor.models.adapters.as_seq_cls_model`, `vllm.model_executor.models.is_pooling_model`.
**CN:** 导入标准库模块（如 `importlib.util`、`warnings`）、第三方包（如 `pytest`、`torch.cuda`）、项目内辅助模块（如 `vllm.model_executor.models.adapters.as_embedding_model`、`vllm.model_executor.models.adapters.as_seq_cls_model`、`vllm.model_executor.models.is_pooling_model`）。

### Test / 测试: test_registry_imports (L30-L65)
```python
@pytest.mark.parametrize("model_arch", ModelRegistry.get_supported_archs())
def test_registry_imports(model_arch):
    # Skip if transformers version is incompatible
    model_info = HF_EXAMPLE_MODELS.get_hf_info(model_arch)
    model_info.check_transformers_version(
        on_fail="skip",
        check_max_version=False,
        check_version_reason="vllm",
    )

    if model_arch in ("PrithviGeoSpatialMAE", "Terratorch"):
        import importlib.util

        if importlib.util.find_spec("terratorch") is None:
            pytest.skip(
                "terratorch is not installed; "
                "temporarily skipped while PyPI has `lightning` quarantined "
                "(see #41376)"
# ... 10 lines omitted for brevity ...
        assert is_text_generation_model(model_cls)

    # All vLLM models should be convertible to a pooling model
    assert is_pooling_model(as_seq_cls_model(model_cls))
    assert is_pooling_model(as_embedding_model(model_cls))

    if model_arch in _MULTIMODAL_MODELS:
        assert supports_multimodal(model_cls)
```
**EN:** This test validates `test_registry_imports`. It uses parameterization over `model_arch`. Key inputs are `model_arch`. The logic can skip unsupported environments when prerequisites are not satisfied. The main assertion is `model_cls is not None` and `is_pooling_model(as_seq_cls_model(model_cls))`.
**CN:** 这个测试验证 `test_registry_imports`。 它通过参数化组合 `model_arch`。 关键输入包括 `model_arch`。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。 核心断言是 `model_cls is not None` and `is_pooling_model(as_seq_cls_model(model_cls))`。

### Test / 测试: test_registry_model_property (L68-L98)
```python
@create_new_process_for_each_test()
@pytest.mark.parametrize(
    "model_arch,is_mm,init_cuda,score_type",
    [
        ("LlamaForCausalLM", False, False, "bi-encoder"),
        ("LlavaForConditionalGeneration", True, True, "bi-encoder"),
        ("BertForSequenceClassification", False, False, "cross-encoder"),
        ("RobertaForSequenceClassification", False, False, "cross-encoder"),
        ("XLMRobertaForSequenceClassification", False, False, "cross-encoder"),
        ("GteNewModel", False, False, "bi-encoder"),
        ("GteNewForSequenceClassification", False, False, "cross-encoder"),
        ("HF_ColBERT", False, False, "late-interaction"),
    ],
)
def test_registry_model_property(model_arch, is_mm, init_cuda, score_type):
    model_info = ModelRegistry._try_inspect_model_cls(model_arch)
    assert model_info is not None

# ... 5 lines omitted for brevity ...

        ModelRegistry._try_load_model_cls(model_arch)
        if not torch.cuda.is_initialized():
            warnings.warn(
                "This model no longer initializes CUDA on import. "
                "Please test using a different one.",
                stacklevel=2,
            )
```
**EN:** This test validates `test_registry_model_property`. It uses parameterization over `model_arch`, `is_mm`, `init_cuda`, `score_type`. Key inputs are `model_arch`, `is_mm`, `init_cuda`, `score_type`. The main assertion is `model_info is not None` and `model_info.supports_multimodal is is_mm`.
**CN:** 这个测试验证 `test_registry_model_property`。 它通过参数化组合 `model_arch`、`is_mm`、`init_cuda`、`score_type`。 关键输入包括 `model_arch`、`is_mm`、`init_cuda`、`score_type`。 核心断言是 `model_info is not None` and `model_info.supports_multimodal is is_mm`。

### Test / 测试: test_registry_is_pp (L101-L127)
```python
@create_new_process_for_each_test()
@pytest.mark.parametrize(
    "model_arch,is_pp,init_cuda",
    [
        # TODO(woosuk): Re-enable this once the MLP Speculator is supported
        # in V1.
        # ("MLPSpeculatorPreTrainedModel", False, False),
        ("DeepseekV2ForCausalLM", True, False),
        ("Qwen2VLForConditionalGeneration", True, True),
    ],
)
def test_registry_is_pp(model_arch, is_pp, init_cuda):
    model_info = ModelRegistry._try_inspect_model_cls(model_arch)
    assert model_info is not None

    assert model_info.supports_pp is is_pp

    if init_cuda and current_platform.is_cuda_alike():
        assert not torch.cuda.is_initialized()

        ModelRegistry._try_load_model_cls(model_arch)
        if not torch.cuda.is_initialized():
            warnings.warn(
                "This model no longer initializes CUDA on import. "
                "Please test using a different one.",
                stacklevel=2,
            )
```
**EN:** This test validates `test_registry_is_pp`. It uses parameterization over `model_arch`, `is_pp`, `init_cuda`. Key inputs are `model_arch`, `is_pp`, `init_cuda`. The main assertion is `model_info is not None` and `model_info.supports_pp is is_pp`.
**CN:** 这个测试验证 `test_registry_is_pp`。 它通过参数化组合 `model_arch`、`is_pp`、`init_cuda`。 关键输入包括 `model_arch`、`is_pp`、`init_cuda`。 核心断言是 `model_info is not None` and `model_info.supports_pp is is_pp`。

### Test / 测试: test_hf_registry_coverage (L130-L138)
```python
def test_hf_registry_coverage():
    untested_archs = (
        ModelRegistry.get_supported_archs() - HF_EXAMPLE_MODELS.get_supported_archs()
    )

    assert not untested_archs, (
        "Please add the following architectures to "
        f"`tests/models/registry.py`: {untested_archs}"
    )
```
**EN:** This test validates `test_hf_registry_coverage`. The main assertion is `not untested_archs`.
**CN:** 这个测试验证 `test_hf_registry_coverage`。 核心断言是 `not untested_archs`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。
- **EN:** Shared metadata and helper utilities keep nearby tests focused on behavior.
  **CN:** 共享元数据与辅助工具让相邻测试更专注于行为本身。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `importlib.util`, `warnings`
- **Third-party / 第三方**: `pytest`, `torch.cuda`
- **Project / 项目内**: `vllm.model_executor.models.adapters.as_embedding_model`, `vllm.model_executor.models.adapters.as_seq_cls_model`, `vllm.model_executor.models.is_pooling_model`, `vllm.model_executor.models.is_text_generation_model`, `vllm.model_executor.models.registry.ModelRegistry`, `vllm.model_executor.models.registry._MULTIMODAL_MODELS`, `vllm.model_executor.models.registry._SPECULATIVE_DECODING_MODELS`, `vllm.model_executor.models.registry._TEXT_GENERATION_MODELS`, `vllm.model_executor.models.supports_multimodal`, `vllm.platforms.current_platform`
- **Local relative imports / 本地相对导入**: `..utils.create_new_process_for_each_test`, `.registry.HF_EXAMPLE_MODELS`
