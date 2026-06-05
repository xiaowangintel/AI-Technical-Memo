# registry.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/registry.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides shared test utilities for model registry metadata and model-facing behavior. The file exposes 8 helper/class block(s) used by nearby tests. / [CN] 为模型注册表元数据与面向模型的行为提供共享测试工具。该文件暴露了 8 个辅助函数/类块，供附近测试复用。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L13)
```python
from collections.abc import Mapping, Set
from dataclasses import dataclass, field
from typing import Any, Literal

import pytest
from packaging.version import Version
from transformers import PretrainedConfig
from transformers import __version__ as TRANSFORMERS_VERSION

from vllm.config.model import ModelDType, TokenizerMode
```
**EN:** Imports standard-library modules such as `collections.abc.Mapping`, `collections.abc.Set`, `dataclasses.dataclass`, third-party packages like `packaging.version.Version`, `pytest`, `transformers.PretrainedConfig`, project helpers such as `vllm.config.model.ModelDType`, `vllm.config.model.TokenizerMode`.
**CN:** 导入标准库模块（如 `collections.abc.Mapping`、`collections.abc.Set`、`dataclasses.dataclass`）、第三方包（如 `packaging.version.Version`、`pytest`、`transformers.PretrainedConfig`）、项目内辅助模块（如 `vllm.config.model.ModelDType`、`vllm.config.model.TokenizerMode`）。

### Class / 类: _HfExamplesInfo (L16-L191)
```python
@dataclass(frozen=True)
class _HfExamplesInfo:
    default: str
    """The default model to use for testing this architecture."""

    extras: Mapping[str, str] = field(default_factory=dict)
    """Extra models to use for testing this architecture."""

    tokenizer: str | None = None
    """Set the tokenizer to load for this architecture."""

    tokenizer_mode: TokenizerMode | str = "auto"
    """Set the tokenizer type for this architecture."""

# ... 154 lines omitted for brevity ...
        """
        if not self.is_available_online:
            msg = "Model is not available online"

            if on_fail == "error":
                raise RuntimeError(msg)
            else:
                pytest.skip(msg)
```
**EN:** This class groups related scenarios in `_HfExamplesInfo`. Decorators such as `@dataclass` make it a compact metadata container. It contains 0 test method(s) and 2 supporting method(s). Representative methods include `check_transformers_version`, `check_available_online`.
**CN:** 该类将与 `_HfExamplesInfo` 相关的场景组织在一起。 像 `@dataclass` 这样的装饰器使它成为紧凑的元数据容器。 它包含 0 个测试方法和 2 个辅助方法。 代表性方法包括 `check_transformers_version`、`check_available_online`。

### Helper method / 辅助方法: _HfExamplesInfo.check_transformers_version (L122-L175)
```python
    def check_transformers_version(
        self,
        *,
        on_fail: Literal["error", "skip", "return"],
        check_version_reason: Literal["vllm", "hf"] = "hf",
        check_min_version: bool = True,
        check_max_version: bool = True,
    ) -> str | None:
        """
        If the installed transformers version does not meet the requirements,
        perform the given action.
        """
        if (
            self.min_transformers_version is None
            and self.max_transformers_version is None
        ):
            return None

# ... 28 lines omitted for brevity ...
                msg += f" Reason({reason_type}): {reason}"

        if on_fail == "error":
            raise RuntimeError(msg)
        elif on_fail == "skip":
            pytest.skip(msg)

        return msg
```
**EN:** This helper encapsulates reusable logic in `_HfExamplesInfo.check_transformers_version`. The logic can skip unsupported environments when prerequisites are not satisfied. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `_HfExamplesInfo.check_transformers_version` 中。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。 它把计算得到的状态或辅助对象返回给调用方。

### Helper method / 辅助方法: _HfExamplesInfo.check_available_online (L177-L191)
```python
    def check_available_online(
        self,
        *,
        on_fail: Literal["error", "skip"],
    ) -> None:
        """
        If the model is not available online, perform the given action.
        """
        if not self.is_available_online:
            msg = "Model is not available online"

            if on_fail == "error":
                raise RuntimeError(msg)
            else:
                pytest.skip(msg)
```
**EN:** This helper encapsulates reusable logic in `_HfExamplesInfo.check_available_online`. The logic can skip unsupported environments when prerequisites are not satisfied.
**CN:** 这个辅助函数将可复用逻辑封装在 `_HfExamplesInfo.check_available_online` 中。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。

### Module setup / 模块级配置: _TEXT_GENERATION_EXAMPLE_MODELS, _EMBEDDING_EXAMPLE_MODELS, _LATE_INTERACTION_EXAMPLE_MODELS (L194-L1675)
```python
_TEXT_GENERATION_EXAMPLE_MODELS = {
    # [Decoder-only]
    "AfmoeForCausalLM": _HfExamplesInfo("arcee-ai/Trinity-Nano-Preview"),
    "ApertusForCausalLM": _HfExamplesInfo("swiss-ai/Apertus-8B-Instruct-2509"),
    "AquilaModel": _HfExamplesInfo("BAAI/AquilaChat-7B", trust_remote_code=True),
    "AquilaForCausalLM": _HfExamplesInfo("BAAI/AquilaChat2-7B", trust_remote_code=True),
    "ArceeForCausalLM": _HfExamplesInfo("arcee-ai/AFM-4.5B-Base"),
    "ArcticForCausalLM": _HfExamplesInfo(
        "Snowflake/snowflake-arctic-instruct", trust_remote_code=True
    ),
    "AXK1ForCausalLM": _HfExamplesInfo("skt/A.X-K1", trust_remote_code=True),
    "BaiChuanForCausalLM": _HfExamplesInfo(
        "baichuan-inc/Baichuan-7B", trust_remote_code=True
    ),
    "BaichuanForCausalLM": _HfExamplesInfo(
        "baichuan-inc/Baichuan2-7B-chat", trust_remote_code=True
# ... 1460 lines omitted for brevity ...
    **_TOKEN_CLASSIFICATION_EXAMPLE_MODELS,
    **_SEQUENCE_CLASSIFICATION_EXAMPLE_MODELS,
    **_MULTIMODAL_EXAMPLE_MODELS,
    **_SPECULATIVE_DECODING_EXAMPLE_MODELS,
    **_TRANSFORMERS_BACKEND_MODELS,
}
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `_TEXT_GENERATION_EXAMPLE_MODELS`, `_EMBEDDING_EXAMPLE_MODELS`, `_LATE_INTERACTION_EXAMPLE_MODELS`, `_REWARD_EXAMPLE_MODELS`, `_TOKEN_CLASSIFICATION_EXAMPLE_MODELS`, `_SEQUENCE_CLASSIFICATION_EXAMPLE_MODELS`, `_AUTOMATIC_CONVERTED_MODELS`, `_MULTIMODAL_EXAMPLE_MODELS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `_TEXT_GENERATION_EXAMPLE_MODELS`、`_EMBEDDING_EXAMPLE_MODELS`、`_LATE_INTERACTION_EXAMPLE_MODELS`、`_REWARD_EXAMPLE_MODELS`、`_TOKEN_CLASSIFICATION_EXAMPLE_MODELS`、`_SEQUENCE_CLASSIFICATION_EXAMPLE_MODELS`、`_AUTOMATIC_CONVERTED_MODELS`、`_MULTIMODAL_EXAMPLE_MODELS`。后续辅助函数和测试会复用这些值，以减少重复。

### Class / 类: HfExampleModels (L1678-L1707)
```python
class HfExampleModels:
    def __init__(self, hf_models: Mapping[str, _HfExamplesInfo]) -> None:
        super().__init__()

        self.hf_models = hf_models

    def get_supported_archs(self) -> Set[str]:
        return self.hf_models.keys()

    def get_hf_info(self, model_arch: str) -> _HfExamplesInfo:
        try:
            return self.hf_models[model_arch]
        except KeyError:
            raise ValueError(
# ... 8 lines omitted for brevity ...
        # Fallback to extras
        for info in self.hf_models.values():
            if any(extra == model_id for extra in info.extras.values()):
                return info

        raise ValueError(
            f"No example model defined for {model_id}; please update this file."
        )
```
**EN:** This class groups related scenarios in `HfExampleModels`. It contains 0 test method(s) and 4 supporting method(s). Representative methods include `__init__`, `get_supported_archs`.
**CN:** 该类将与 `HfExampleModels` 相关的场景组织在一起。 它包含 0 个测试方法和 4 个辅助方法。 代表性方法包括 `__init__`、`get_supported_archs`。

### Helper method / 辅助方法: HfExampleModels.__init__ (L1679-L1682)
```python
    def __init__(self, hf_models: Mapping[str, _HfExamplesInfo]) -> None:
        super().__init__()

        self.hf_models = hf_models
```
**EN:** This helper encapsulates reusable logic in `HfExampleModels.__init__`. Key inputs are `hf_models`.
**CN:** 这个辅助函数将可复用逻辑封装在 `HfExampleModels.__init__` 中。 关键输入包括 `hf_models`。

### Helper method / 辅助方法: HfExampleModels.get_supported_archs (L1684-L1685)
```python
    def get_supported_archs(self) -> Set[str]:
        return self.hf_models.keys()
```
**EN:** This helper encapsulates reusable logic in `HfExampleModels.get_supported_archs`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `HfExampleModels.get_supported_archs` 中。 它把计算得到的状态或辅助对象返回给调用方。

### Helper method / 辅助方法: HfExampleModels.get_hf_info (L1687-L1693)
```python
    def get_hf_info(self, model_arch: str) -> _HfExamplesInfo:
        try:
            return self.hf_models[model_arch]
        except KeyError:
            raise ValueError(
                f"No example model defined for {model_arch}; please update this file."
            ) from None
```
**EN:** This helper encapsulates reusable logic in `HfExampleModels.get_hf_info`. Key inputs are `model_arch`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `HfExampleModels.get_hf_info` 中。 关键输入包括 `model_arch`。 它把计算得到的状态或辅助对象返回给调用方。

### Helper method / 辅助方法: HfExampleModels.find_hf_info (L1695-L1707)
```python
    def find_hf_info(self, model_id: str) -> _HfExamplesInfo:
        for info in self.hf_models.values():
            if info.default == model_id:
                return info

        # Fallback to extras
        for info in self.hf_models.values():
            if any(extra == model_id for extra in info.extras.values()):
                return info

        raise ValueError(
            f"No example model defined for {model_id}; please update this file."
        )
```
**EN:** This helper encapsulates reusable logic in `HfExampleModels.find_hf_info`. Key inputs are `model_id`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `HfExampleModels.find_hf_info` 中。 关键输入包括 `model_id`。 它把计算得到的状态或辅助对象返回给调用方。

### Module setup / 模块级配置: HF_EXAMPLE_MODELS, AUTO_EXAMPLE_MODELS (L1710-L1711)
```python
HF_EXAMPLE_MODELS = HfExampleModels(_EXAMPLE_MODELS)
AUTO_EXAMPLE_MODELS = HfExampleModels(_AUTOMATIC_CONVERTED_MODELS)
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `HF_EXAMPLE_MODELS`, `AUTO_EXAMPLE_MODELS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `HF_EXAMPLE_MODELS`、`AUTO_EXAMPLE_MODELS`。后续辅助函数和测试会复用这些值，以减少重复。

## Key Concepts / 关键概念
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Shared metadata and helper utilities keep nearby tests focused on behavior.
  **CN:** 共享元数据与辅助工具让相邻测试更专注于行为本身。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `collections.abc.Mapping`, `collections.abc.Set`, `dataclasses.dataclass`, `dataclasses.field`, `typing.Any`, `typing.Literal`
- **Third-party / 第三方**: `packaging.version.Version`, `pytest`, `transformers.PretrainedConfig`, `transformers.__version__`
- **Project / 项目内**: `vllm.config.model.ModelDType`, `vllm.config.model.TokenizerMode`
