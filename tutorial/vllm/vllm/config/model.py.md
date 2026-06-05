# model.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/config/model.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements model support for the `config` portion of vLLM. / 为 vLLM 的 `config` 子目录实现与 模型 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-58)
```python
import warnings

from collections.abc import Callable

from dataclasses import InitVar, field

from functools import cached_property

from typing import TYPE_CHECKING, Any, Literal, cast, get_args

import torch

from pydantic import ConfigDict, Field, field_validator, model_validator

import vllm.envs as envs

from vllm.config.model_arch import (
    ModelArchitectureConfig,
)

from vllm.config.multimodal import (
    MMCacheType,
    MMEncoderTPMode,
    MMTensorIPC,
    MultiModalConfig,
)

from vllm.config.pooler import PoolerConfig

from vllm.config.quantization import QuantizationConfigArgs

from vllm.config.scheduler import RunnerType

from vllm.config.utils import config, getattr_iter

from vllm.logger import init_logger
# ... omitted for brevity ...
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 81-105)
```python
logger = init_logger(__name__)

RunnerOption = Literal["auto", RunnerType]

ConvertType = Literal["none", "embed", "classify"]

ConvertOption = Literal["auto", ConvertType]

TokenizerMode = Literal[
    "auto", "hf", "slow", "mistral", "deepseek_v32", "deepseek_v4", "fastokens"
]

ModelDType = Literal["auto", "half", "float16", "bfloat16", "float", "float32"]

LogprobsMode = Literal[
    "raw_logits", "raw_logprobs", "processed_logits", "processed_logprobs"
]

HfOverrides = dict[str, Any] | Callable[[PretrainedConfig], PretrainedConfig]

ModelImpl = Literal["auto", "vllm", "transformers", "terratorch"]

LayerBlockType = Literal["attention", "linear_attention", "mamba"]

_RUNNER_CONVERTS: dict[RunnerType, list[ConvertType]] = {
    "generate": [],
    "pooling": ["embed", "classify"],
    "draft": [],
# ... omitted for brevity ...
```
**EN:** This constant/configuration block defines `logger`, `RunnerOption`, `ConvertType`, `ConvertOption`, `TokenizerMode`, `ModelDType`, `LogprobsMode`, `HfOverrides`, `ModelImpl`, `LayerBlockType`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `logger`, `RunnerOption`, `ConvertType`, `ConvertOption`, `TokenizerMode`, `ModelDType`, `LogprobsMode`, `HfOverrides`, `ModelImpl`, `LayerBlockType`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Class `ModelConfig` (lines 109-1851)
```python
class ModelConfig:
    """Configuration for the model."""

    model: str = "Qwen/Qwen3-0.6B"
    """Name or path of the Hugging Face model to use. It is also used as the
    content for `model_name` tag in metrics output when `served_model_name` is
    not specified."""
    model_weights: str = ""
    """Original model weights path. Used when the model is pulled from object
    storage (e.g., RunAI) to preserve the original URI while `model` points to
    the local directory."""
    runner: RunnerOption = "auto"
    """The type of model runner to use. Each vLLM instance only supports one
    model runner, even if the same model can be used for multiple types."""
    convert: ConvertOption = "auto"
    """Convert the model using adapters defined in
    [vllm.model_executor.models.adapters][]. The most common use case is to
    adapt a text generation model to be used for pooling tasks."""
    tokenizer: str = None  # type: ignore[assignment]
    """Name or path of the Hugging Face tokenizer to use. If unspecified, model
    name or path will be used."""
    tokenizer_mode: TokenizerMode | str = "auto"
    """Tokenizer mode:
    # ... omitted for brevity ...
            and "nvfp4" in quant_config.get("format", "").lower()
        )
```
**EN:** Class `ModelConfig` is a structured building block in this module. Key methods include `compute_hash`, `_update_nested`, `_apply_dict_overrides`, `__post_init__`, `get_model_arch_config`, `_skip_none_validation`, which define initialization, validation, transformation, or access patterns. The class docstring says: Configuration for the model.
**CN:** 类 `ModelConfig` 是该模块中的结构化构件。 关键方法包括 `compute_hash`, `_update_nested`, `_apply_dict_overrides`, `__post_init__`, `get_model_arch_config`, `_skip_none_validation`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Configuration for the model.

### Method `ModelConfig.compute_hash` (lines 348-404)
```python
    def compute_hash(self) -> str:
        """
        WARNING: Whenever a new field is added to this config,
        ensure that it is included in the factors list if
        it affects the computation graph.

        Provide a hash that uniquely identifies all the configs
        that affect the structure of the computation
        graph from input ids/embeddings to the final hidden states,
        excluding anything before input ids/embeddings and after
        the final hidden states.
        """
        ignored_factors = {
            "convert",
            "tokenizer",
            "tokenizer_mode",
            "seed",
            "hf_config_path",
            "allowed_local_media_path",
    # ... omitted for brevity ...
            factors["language_model_only"] = self.multimodal_config.language_model_only
        return hash_factors(factors)
```
**EN:** Method `ModelConfig.compute_hash` computes stable identifiers or fingerprints for caching/comparison. The docstring highlights: WARNING: Whenever a new field is added to this config, ensure that it is included in the factors list if it affects the computation graph. Key calls such as `get_hash_factors`, `hash_factors` show the concrete execution path.
**CN:** Method `ModelConfig.compute_hash` 负责计算稳定标识或指纹以支持缓存/比较。 文档字符串强调：WARNING: Whenever a new field is added to this config, ensure that it is included in the factors list if it affects the computation graph. 像 `get_hash_factors`, `hash_factors` 这样的关键调用展示了该代码块的具体执行路径。

### Method `ModelConfig.__post_init__` (lines 451-712)
```python
    def __post_init__(
        self,
        # Multimodal config init vars
        language_model_only: bool,
        limit_mm_per_prompt: dict[str, int | dict[str, int]] | None,
        enable_mm_embeds: bool | None,
        media_io_kwargs: dict[str, dict[str, Any]] | None,
        mm_processor_kwargs: dict[str, Any] | None,
        mm_processor_cache_gb: float | None,
        mm_processor_cache_type: MMCacheType | None,
        mm_shm_cache_max_object_size_mb: int | None,
        mm_encoder_only: bool | None,
        mm_encoder_tp_mode: MMEncoderTPMode | None,
        mm_encoder_attn_backend: AttentionBackendEnum | str | None,
        mm_encoder_attn_dtype: str | None,
        mm_encoder_fp8_scale_path: str | None,
        mm_encoder_fp8_scale_save_path: str | None,
        mm_encoder_fp8_scale_save_margin: float | None,
        interleave_mm_strings: bool | None,
    # ... omitted for brevity ...
        self._verify_cuda_graph()
        self._verify_bnb_config()
```
**EN:** Method `ModelConfig.__post_init__` constructs derived objects, runtime state, or helper structures. Key calls such as `get_served_model_name`, `maybe_model_redirect`, `isinstance`, `callable`, `self.hf_overrides.items` show the concrete execution path.
**CN:** Method `ModelConfig.__post_init__` 负责构造派生对象、运行时状态或辅助结构。 像 `get_served_model_name`, `maybe_model_redirect`, `isinstance`, `callable`, `self.hf_overrides.items` 这样的关键调用展示了该代码块的具体执行路径。

### Method `ModelConfig.verify_with_parallel_config` (lines 1144-1208)
```python
    def verify_with_parallel_config(
        self,
        parallel_config: ParallelConfig,
    ) -> None:
        total_num_attention_heads = self.model_arch_config.total_num_attention_heads
        tensor_parallel_size = parallel_config.tensor_parallel_size
        if total_num_attention_heads % tensor_parallel_size != 0:
            raise ValueError(
                f"Total number of attention heads ({total_num_attention_heads})"
                " must be divisible by tensor parallel size "
                f"({tensor_parallel_size})."
            )

        if parallel_config.enable_expert_parallel:
            self._verify_with_expert_parallelism()

        pipeline_parallel_size = parallel_config.pipeline_parallel_size
        if pipeline_parallel_size > 1 and not self.registry.is_pp_supported_model(
            self.architectures, self
    # ... omitted for brevity ...
                "or pipeline_parallel_size > 1."
            )
```
**EN:** Method `ModelConfig.verify_with_parallel_config` parses configuration, arguments, or structured metadata. Key calls such as `ValueError`, `self._verify_with_expert_parallelism`, `self.registry.is_pp_supported_model`, `NotImplementedError`, `self.get_total_num_kv_heads` show the concrete execution path.
**CN:** Method `ModelConfig.verify_with_parallel_config` 负责解析配置、参数或结构化元数据。 像 `ValueError`, `self._verify_with_expert_parallelism`, `self.registry.is_pp_supported_model`, `NotImplementedError`, `self.get_total_num_kv_heads` 这样的关键调用展示了该代码块的具体执行路径。

### Method `ModelConfig.get_num_layers_by_block_type` (lines 1286-1354)
```python
    def get_num_layers_by_block_type(
        self,
        parallel_config: ParallelConfig,
        block_type: LayerBlockType = "attention",
    ) -> int:
        # This function relies on 'layers_block_type' in hf_config,
        # for w/o this attribute, we will need to have workarounds like so
        attn_block_type = block_type == "attention"
        is_transformer = (
            not self.is_hybrid and not self.has_noops and not self.is_attention_free
        )
        start, end = self.get_layers_start_end_indices(parallel_config)

        if is_transformer:
            # Handle the basic case first
            return end - start if attn_block_type else 0
        elif self.is_attention_free:
            # Attention free
            # Note that this code assumes there
    # ... omitted for brevity ...
                )
            raise AssertionError(f"Unsupported block type: {block_type}")
```
**EN:** Method `ModelConfig.get_num_layers_by_block_type` provides a reusable helper around the module's main workflow. Key calls such as `self.get_layers_start_end_indices`, `sum`, `getattr`, `self.get_num_layers`, `ValueError` show the concrete execution path.
**CN:** Method `ModelConfig.get_num_layers_by_block_type` 为模块主流程提供可复用的辅助逻辑。 像 `self.get_layers_start_end_indices`, `sum`, `getattr`, `self.get_num_layers`, `ValueError` 这样的关键调用展示了该代码块的具体执行路径。

### Method `ModelConfig.get_diff_sampling_param` (lines 1418-1471)
```python
    def get_diff_sampling_param(self) -> dict[str, Any]:
        """
        This method returns a dictionary containing the non-default sampling
        parameters with `override_generation_config` applied.

        The default sampling parameters are:

        - vLLM's neutral defaults if `self.generation_config="vllm"`
        - the model's defaults if `self.generation_config="auto"`
        - as defined in `generation_config.json` if
            `self.generation_config="path/to/generation_config/dir"`

        Returns:
            A dictionary containing the non-default sampling parameters.
        """
        src = self.generation_config

        config = {} if src == "vllm" else self.try_get_generation_config()

    # ... omitted for brevity ...

        return diff_sampling_param
```
**EN:** Method `ModelConfig.get_diff_sampling_param` provides a reusable helper around the module's main workflow. The docstring highlights: This method returns a dictionary containing the non-default sampling parameters with `override_generation_config` applied. Key calls such as `self.try_get_generation_config`, `config.update`, `any`, `config.get`, `diff_sampling_param.pop` show the concrete execution path.
**CN:** Method `ModelConfig.get_diff_sampling_param` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：This method returns a dictionary containing the non-default sampling parameters with `override_generation_config` applied. 像 `self.try_get_generation_config`, `config.update`, `any`, `config.get`, `diff_sampling_param.pop` 这样的关键调用展示了该代码块的具体执行路径。

### Method `ModelConfig.is_prefix_caching_supported` (lines 1774-1829)
```python
    def is_prefix_caching_supported(self) -> bool:
        attn_type = self.attn_type

        if pooler_config := self.pooler_config:
            # for pooling models
            if attn_type == "encoder_only":
                logger.debug(
                    "Pooling models with bidirectional attn "
                    "do not support prefix caching."
                )
                return False

            if attn_type == "decoder":
                if (
                    pooler_config.seq_pooling_type in ("MEAN", "CLS")
                    or pooler_config.tok_pooling_type == "STEP"
                ):
                    logger.debug(
                        "Pooling models with causal attn and %s/%s pooling "
    # ... omitted for brevity ...
                logger.debug("Generative models support prefix caching.")
                return True
```
**EN:** Method `ModelConfig.is_prefix_caching_supported` exposes a computed property so callers can access derived state without duplicating logic. Key calls such as `logger.debug` show the concrete execution path.
**CN:** Method `ModelConfig.is_prefix_caching_supported` 暴露计算属性，使调用方无需重复逻辑即可访问派生状态。 像 `logger.debug` 这样的关键调用展示了该代码块的具体执行路径。

### Function `get_served_model_name` (lines 1854-1866)
```python
def get_served_model_name(model: str, served_model_name: str | list[str] | None):
    """
    If the input is a non-empty list, the first model_name in
    `served_model_name` is taken.
    If the input is a non-empty string, it is used directly.
    For cases where the input is either an empty string or an
    empty list, the fallback is to use `self.model`.
    """
    if not served_model_name:
        return model
    if isinstance(served_model_name, list):
        return served_model_name[0]
    return served_model_name
```
**EN:** Function `get_served_model_name` coordinates benchmarking or serving-oriented control flow. The docstring highlights: If the input is a non-empty list, the first model_name in `served_model_name` is taken. Key calls such as `isinstance` show the concrete execution path.
**CN:** Function `get_served_model_name` 负责协调基准测试或服务侧控制流程。 文档字符串强调：If the input is a non-empty list, the first model_name in `served_model_name` is taken. 像 `isinstance` 这样的关键调用展示了该代码块的具体执行路径。

### Function `iter_architecture_defaults` (lines 1892-1893)
```python
def iter_architecture_defaults():
    yield from _SUFFIX_TO_DEFAULTS
```
**EN:** Function `iter_architecture_defaults` provides a reusable helper around the module's main workflow.
**CN:** Function `iter_architecture_defaults` 为模块主流程提供可复用的辅助逻辑。

### Function `try_match_architecture_defaults` (lines 1896-1913)
```python
def try_match_architecture_defaults(
    architecture: str,
    *,
    runner_type: RunnerType | None = None,
    convert_type: ConvertType | None = None,
) -> tuple[str, tuple[RunnerType, ConvertType]] | None:
    for suffix, (
        default_runner_type,
        default_convert_type,
    ) in iter_architecture_defaults():
        if (
            (runner_type is None or runner_type == default_runner_type)
            and (convert_type is None or convert_type == default_convert_type)
            and architecture.endswith(suffix)
        ):
            return suffix, (default_runner_type, default_convert_type)

    return None
```
**EN:** Function `try_match_architecture_defaults` provides a reusable helper around the module's main workflow. Key calls such as `iter_architecture_defaults`, `architecture.endswith` show the concrete execution path.
**CN:** Function `try_match_architecture_defaults` 为模块主流程提供可复用的辅助逻辑。 像 `iter_architecture_defaults`, `architecture.endswith` 这样的关键调用展示了该代码块的具体执行路径。

### Function `str_dtype_to_torch_dtype` (lines 1925-1926)
```python
def str_dtype_to_torch_dtype(type: str):
    return _STR_DTYPE_TO_TORCH_DTYPE.get(type)
```
**EN:** Function `str_dtype_to_torch_dtype` provides a reusable helper around the module's main workflow. Key calls such as `_STR_DTYPE_TO_TORCH_DTYPE.get` show the concrete execution path.
**CN:** Function `str_dtype_to_torch_dtype` 为模块主流程提供可复用的辅助逻辑。 像 `_STR_DTYPE_TO_TORCH_DTYPE.get` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_is_valid_dtype` (lines 1939-1943)
```python
def _is_valid_dtype(model_type: str, dtype: torch.dtype):
    if model_type in _FLOAT16_NOT_SUPPORTED_MODELS and dtype == torch.float16:  # noqa: E501, SIM103
        return False

    return True
```
**EN:** Function `_is_valid_dtype` provides a reusable helper around the module's main workflow.
**CN:** Function `_is_valid_dtype` 为模块主流程提供可复用的辅助逻辑。

### Function `_check_valid_dtype` (lines 1946-1953)
```python
def _check_valid_dtype(model_type: str, dtype: torch.dtype):
    if model_type in _FLOAT16_NOT_SUPPORTED_MODELS and dtype == torch.float16:
        reason = _FLOAT16_NOT_SUPPORTED_MODELS[model_type]
        raise ValueError(
            f"The model type {model_type!r} does not support float16. Reason: {reason}"
        )

    return True
```
**EN:** Function `_check_valid_dtype` validates assumptions and guards module invariants. Key calls such as `ValueError` show the concrete execution path.
**CN:** Function `_check_valid_dtype` 负责校验前置条件并保护模块不变量。 像 `ValueError` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_resolve_auto_dtype` (lines 1956-1997)
```python
def _resolve_auto_dtype(
    model_type: str,
    config_dtype: torch.dtype,
    *,
    is_pooling_model: bool,
):
    supported_dtypes = [
        dtype
        for dtype in current_platform.supported_dtypes
        if _is_valid_dtype(model_type, dtype)
    ]

    if is_pooling_model and torch.float16 in supported_dtypes:
        preferred_dtype = torch.float16
    else:
        preferred_dtype = supported_dtypes[0]

    # Downcast for float32 models
    if config_dtype == torch.float32:
        config_dtype = preferred_dtype

    if config_dtype in supported_dtypes:
        return config_dtype

    # Ensure device compatibility
    # ... omitted for brevity ...

    return preferred_dtype
```
**EN:** Function `_resolve_auto_dtype` provides a reusable helper around the module's main workflow. Key calls such as `_is_valid_dtype`, `current_platform.get_device_name`, `current_platform.get_device_capability`, `device_capability.as_version_str`, `logger.warning` show the concrete execution path.
**CN:** Function `_resolve_auto_dtype` 为模块主流程提供可复用的辅助逻辑。 像 `_is_valid_dtype`, `current_platform.get_device_name`, `current_platform.get_device_capability`, `device_capability.as_version_str`, `logger.warning` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_get_and_verify_dtype` (lines 2000-2045)
```python
def _get_and_verify_dtype(
    model_id: str,
    config: PretrainedConfig,
    dtype: str | torch.dtype,
    *,
    is_pooling_model: bool,
    revision: str | None = None,
    config_format: str | ConfigFormat = "hf",
) -> torch.dtype:
    config_dtype = ModelArchConfigConvertorBase.get_torch_dtype(
        config, model_id, revision=revision, config_format=config_format
    )
    model_type = config.model_type

    if isinstance(dtype, str):
        dtype = dtype.lower()
        if dtype == "auto":
            # Set default dtype from model config
            torch_dtype = _resolve_auto_dtype(
                model_type,
                config_dtype,
                is_pooling_model=is_pooling_model,
            )
        else:
            if dtype not in _STR_DTYPE_TO_TORCH_DTYPE:
    # ... omitted for brevity ...

    return torch_dtype
```
**EN:** Function `_get_and_verify_dtype` provides a reusable helper around the module's main workflow. Key calls such as `ModelArchConfigConvertorBase.get_torch_dtype`, `isinstance`, `dtype.lower`, `_resolve_auto_dtype`, `ValueError` show the concrete execution path.
**CN:** Function `_get_and_verify_dtype` 为模块主流程提供可复用的辅助逻辑。 像 `ModelArchConfigConvertorBase.get_torch_dtype`, `isinstance`, `dtype.lower`, `_resolve_auto_dtype`, `ValueError` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_get_head_dtype` (lines 2048-2069)
```python
def _get_head_dtype(
    config: PretrainedConfig, dtype: torch.dtype, runner_type: str
) -> torch.dtype:
    head_dtype: str | torch.dtype | None = getattr(config, "head_dtype", None)

    if head_dtype == "model":
        return dtype
    elif isinstance(head_dtype, str):
        head_dtype = head_dtype.lower()
        if head_dtype not in _STR_DTYPE_TO_TORCH_DTYPE:
            raise ValueError(f"Unknown dtype: {head_dtype!r}")
        return _STR_DTYPE_TO_TORCH_DTYPE[head_dtype]
    elif isinstance(head_dtype, torch.dtype):
        return head_dtype
    elif head_dtype is None:
        if torch.float32 not in current_platform.supported_dtypes:
            return dtype
        if runner_type == "pooling":
            return torch.float32
        return dtype
    else:
        raise ValueError(f"Unknown dtype: {head_dtype}")
```
**EN:** Function `_get_head_dtype` provides a reusable helper around the module's main workflow. Key calls such as `getattr`, `isinstance`, `head_dtype.lower`, `ValueError` show the concrete execution path.
**CN:** Function `_get_head_dtype` 为模块主流程提供可复用的辅助逻辑。 像 `getattr`, `isinstance`, `head_dtype.lower`, `ValueError` 这样的关键调用展示了该代码块的具体执行路径。

### Function `_get_and_verify_max_len` (lines 2072-2209)
```python
def _get_and_verify_max_len(
    hf_config: PretrainedConfig,
    model_arch_config: ModelArchitectureConfig,
    tokenizer_config: dict | None,
    max_model_len: int | None,
    disable_sliding_window: bool,
    sliding_window: int | None,
    spec_target_max_model_len: int | None = None,
    encoder_config: dict[str, Any] | None = None,
) -> int:
    """Get and verify the model's maximum length."""
    (derived_max_model_len, max_len_key) = (
        model_arch_config.derived_max_model_len_and_key
    )

    # If sliding window is manually disabled, max_length should be less
    # than the sliding window length in the model config.
    if (
        disable_sliding_window
        and sliding_window is not None
        and sliding_window < derived_max_model_len
    ):
        max_len_key = "sliding_window"
        derived_max_model_len = sliding_window

    # ... omitted for brevity ...
                )
    return int(max_model_len)
```
**EN:** Function `_get_and_verify_max_len` provides a reusable helper around the module's main workflow. The docstring highlights: Get and verify the model's maximum length. Key calls such as `tokenizer_config.get`, `min`, `float`, `logger.warning`, `getattr` show the concrete execution path.
**CN:** Function `_get_and_verify_max_len` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Get and verify the model's maximum length. 像 `tokenizer_config.get`, `min`, `float`, `logger.warning`, `getattr` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Tensor processing / 张量处理**
  - **EN:** The module moves arrays/tensors through typed helper functions or model-facing transformations.
  - **CN:** 该模块会通过带类型的辅助函数或面向模型的变换来处理数组/张量。
- **Multimodality / 多模态**
  - **EN:** The code contains modality-specific paths for text, image, audio, or video data.
  - **CN:** 代码包含面向文本、图像、音频或视频数据的模态专用路径。
- **Structured types / 结构化类型**
  - **EN:** Dataclasses, typed dictionaries, or aliases are used to make runtime contracts explicit.
  - **CN:** 通过 dataclass、类型化字典或别名来显式表达运行时契约。
- **Caching / 缓存**
  - **EN:** Some definitions are designed to reuse computed state and avoid repeated work.
  - **CN:** 部分定义旨在复用已计算状态，避免重复工作。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import warnings`, `from collections.abc import Callable`, `from dataclasses import InitVar, field`, `from functools import cached_property`, `from typing import TYPE_CHECKING, Any, Literal, cast, get_args`
- **Third-party / 第三方**: `import torch`, `from pydantic import ConfigDict, Field, field_validator, model_validator`
- **vLLM internal / vLLM 内部依赖**: `import vllm.envs as envs`, `from vllm.config.model_arch import ModelArchitectureConfig`, `from vllm.config.multimodal import MMCacheType, MMEncoderTPMode, MMTensorIPC, MultiModalConfig`, `from vllm.config.pooler import PoolerConfig`, `from vllm.config.quantization import QuantizationConfigArgs`, `from vllm.config.scheduler import RunnerType`, `from vllm.config.utils import config, getattr_iter`, `from vllm.logger import init_logger`, `from vllm.platforms import current_platform`, `from vllm.tasks import PoolingTask, ScoreType, SupportedTask`, `from vllm.transformers_utils.config import ConfigFormat, get_config, get_hf_image_processor_config, get_hf_text_config, get_pooling_config, get_sentence_transformer_tokenizer_config, is_encoder_decoder, is_rope_parameters_nested, try_get_dense_modules, try_get_generation_config, try_get_tokenizer_config, uses_mrope, uses_xdrope_dim`, `from vllm.transformers_utils.gguf_utils import is_gguf, is_remote_gguf, maybe_patch_hf_config_from_gguf, split_remote_gguf`
