# cache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/config/cache.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements cache support for the `config` portion of vLLM. / 为 vLLM 的 `config` 子目录实现与 缓存 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-14)
```python
from dataclasses import field

from typing import ClassVar, Literal

from pydantic import Field, SkipValidation, field_validator, model_validator

from vllm.config.utils import config

from vllm.logger import init_logger

from vllm.utils.torch_utils import (
    is_quantized_kv_cache,
    kv_cache_uses_per_token_head_scales,
)
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 16-38)
```python
logger = init_logger(__name__)

CacheDType = Literal[
    "auto",
    "float16",
    "bfloat16",
    "fp8",
    "fp8_e4m3",
    "fp8_e5m2",
    "fp8_inc",
    "fp8_ds_mla",
    "turboquant_k8v4",
    "turboquant_4bit_nc",
    "turboquant_k3v4_nc",
    "turboquant_3bit_nc",
    "int8_per_token_head",
    "fp8_per_token_head",
    "nvfp4",
]

MambaDType = Literal["auto", "float32", "float16", "bfloat16"]

MambaCacheMode = Literal["all", "align", "none"]

PrefixCachingHashAlgo = Literal["sha256", "sha256_cbor", "xxhash", "xxhash_cbor"]

KVOffloadingBackend = Literal["native", "lmcache"]
```
**EN:** This constant/configuration block defines `logger`, `CacheDType`, `MambaDType`, `MambaCacheMode`, `PrefixCachingHashAlgo`, `KVOffloadingBackend`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `logger`, `CacheDType`, `MambaDType`, `MambaCacheMode`, `PrefixCachingHashAlgo`, `KVOffloadingBackend`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Class `CacheConfig` (lines 42-268)
```python
class CacheConfig:
    """Configuration for the KV cache."""

    DEFAULT_BLOCK_SIZE: ClassVar[int] = 16

    block_size: SkipValidation[int] = None  # type: ignore[assignment]
    """Size of a contiguous cache block in number of tokens.
    Accepts None (meaning "use default"). After construction, always int."""
    user_specified_block_size: bool = field(default=False, init=False)
    """Whether block_size was explicitly provided. Derived automatically."""
    user_specified_mamba_block_size: bool = field(default=False, init=False)
    """Whether mamba_block_size was explicitly provided. Derived automatically."""
    hash_block_size: SkipValidation[int] | None = None  # type: ignore
    """Block size (in tokens) used for computing Request's block_hashes.

    This can be set to a finer granularity than the physical KV cache block
    sizes (e.g. 8) as long as every KV cache group's `block_size` is divisible
    by it. This enables prefix-caching keys to be computed at the finest common
    granularity and then merged for larger physical block sizes.

    This config is not static default. If left unspecified, vLLM will choose a
    default based on the resolved KV cache groups (typically the smallest KV
    cache block size when there are multiple groups).
    # ... omitted for brevity ...
            )
        return cache_dtype
```
**EN:** Class `CacheConfig` is a structured building block in this module. Key methods include `compute_hash`, `metrics_info`, `_apply_block_size_default`, `_warn_deprecated_calculate_kv_scales`, `_validate_cache_dtype`, which define initialization, validation, transformation, or access patterns. The class docstring says: Configuration for the KV cache.
**CN:** 类 `CacheConfig` 是该模块中的结构化构件。 关键方法包括 `compute_hash`, `metrics_info`, `_apply_block_size_default`, `_warn_deprecated_calculate_kv_scales`, `_validate_cache_dtype`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Configuration for the KV cache.

### Method `CacheConfig.compute_hash` (lines 178-213)
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
            # Runtime/derived knobs that don't affect compiled graph shape
            "gpu_memory_utilization",
            "is_attention_free",
            "num_gpu_blocks_override",
            "enable_prefix_caching",
            "prefix_caching_hash_algo",
    # ... omitted for brevity ...
        factors = get_hash_factors(self, ignored_factors)
        return hash_factors(factors)
```
**EN:** Method `CacheConfig.compute_hash` computes stable identifiers or fingerprints for caching/comparison. The docstring highlights: WARNING: Whenever a new field is added to this config, ensure that it is included in the factors list if it affects the computation graph. Key calls such as `get_hash_factors`, `hash_factors` show the concrete execution path.
**CN:** Method `CacheConfig.compute_hash` 负责计算稳定标识或指纹以支持缓存/比较。 文档字符串强调：WARNING: Whenever a new field is added to this config, ensure that it is included in the factors list if it affects the computation graph. 像 `get_hash_factors`, `hash_factors` 这样的关键调用展示了该代码块的具体执行路径。

### Method `CacheConfig.metrics_info` (lines 215-218)
```python
    def metrics_info(self):
        # convert cache_config to dict(key: str, value: str) for prometheus
        # metrics info
        return {key: str(value) for key, value in self.__dict__.items()}
```
**EN:** Method `CacheConfig.metrics_info` provides a reusable helper around the module's main workflow. Key calls such as `str`, `self.__dict__.items` show the concrete execution path.
**CN:** Method `CacheConfig.metrics_info` 为模块主流程提供可复用的辅助逻辑。 像 `str`, `self.__dict__.items` 这样的关键调用展示了该代码块的具体执行路径。

### Method `CacheConfig._apply_block_size_default` (lines 224-236)
```python
    def _apply_block_size_default(self) -> "CacheConfig":
        # Pydantic re-runs validators when CacheConfig is nested inside
        # another pydantic model (e.g. VllmConfig). Guard against that.
        if self._block_size_resolved:
            return self
        object.__setattr__(self, "_block_size_resolved", True)
        if self.block_size is None:
            object.__setattr__(self, "block_size", self.DEFAULT_BLOCK_SIZE)
        else:
            object.__setattr__(self, "user_specified_block_size", True)
        if self.mamba_block_size is not None:
            object.__setattr__(self, "user_specified_mamba_block_size", True)
        return self
```
**EN:** Method `CacheConfig._apply_block_size_default` provides a reusable helper around the module's main workflow. Key calls such as `object.__setattr__`, `model_validator` show the concrete execution path.
**CN:** Method `CacheConfig._apply_block_size_default` 为模块主流程提供可复用的辅助逻辑。 像 `object.__setattr__`, `model_validator` 这样的关键调用展示了该代码块的具体执行路径。

### Method `CacheConfig._warn_deprecated_calculate_kv_scales` (lines 240-248)
```python
    def _warn_deprecated_calculate_kv_scales(cls, calculate_kv_scales: bool) -> bool:
        if calculate_kv_scales:
            logger.warning(
                "The `--calculate-kv-scales` option is deprecated and will "
                "be removed in v0.19. The scales will be loaded from the "
                "model checkpoint if available, otherwise they default to "
                "1.0."
            )
        return calculate_kv_scales
```
**EN:** Method `CacheConfig._warn_deprecated_calculate_kv_scales` provides a reusable helper around the module's main workflow. Key calls such as `logger.warning`, `field_validator` show the concrete execution path.
**CN:** Method `CacheConfig._warn_deprecated_calculate_kv_scales` 为模块主流程提供可复用的辅助逻辑。 像 `logger.warning`, `field_validator` 这样的关键调用展示了该代码块的具体执行路径。

### Method `CacheConfig._validate_cache_dtype` (lines 252-268)
```python
    def _validate_cache_dtype(cls, cache_dtype: CacheDType) -> CacheDType:
        if kv_cache_uses_per_token_head_scales(cache_dtype):
            logger.info(
                "Using %s data type to store kv cache. It reduces the GPU "
                "memory footprint and boosts the performance. "
                "Dynamic per-token-head scales will be computed at runtime.",
                str(cache_dtype),
            )
        elif is_quantized_kv_cache(cache_dtype):
            logger.info(
                "Using %s data type to store kv cache. It reduces the GPU "
                "memory footprint and boosts the performance. "
                "Meanwhile, it may cause accuracy drop without a proper "
                "scaling factor",
                str(cache_dtype),
            )
        return cache_dtype
```
**EN:** Method `CacheConfig._validate_cache_dtype` manages cache lookup, reuse, or lifecycle decisions. Key calls such as `kv_cache_uses_per_token_head_scales`, `logger.info`, `str`, `is_quantized_kv_cache`, `field_validator` show the concrete execution path.
**CN:** Method `CacheConfig._validate_cache_dtype` 负责管理缓存查询、复用或生命周期决策。 像 `kv_cache_uses_per_token_head_scales`, `logger.info`, `str`, `is_quantized_kv_cache`, `field_validator` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Tensor processing / 张量处理**
  - **EN:** The module moves arrays/tensors through typed helper functions or model-facing transformations.
  - **CN:** 该模块会通过带类型的辅助函数或面向模型的变换来处理数组/张量。
- **Structured types / 结构化类型**
  - **EN:** Dataclasses, typed dictionaries, or aliases are used to make runtime contracts explicit.
  - **CN:** 通过 dataclass、类型化字典或别名来显式表达运行时契约。
- **Caching / 缓存**
  - **EN:** Some definitions are designed to reuse computed state and avoid repeated work.
  - **CN:** 部分定义旨在复用已计算状态，避免重复工作。
- **Configuration / 配置**
  - **EN:** Configuration objects and validation rules centralize feature toggles and runtime settings.
  - **CN:** 配置对象与校验规则集中管理功能开关和运行时设置。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `from dataclasses import field`, `from typing import ClassVar, Literal`
- **Third-party / 第三方**: `from pydantic import Field, SkipValidation, field_validator, model_validator`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config.utils import config`, `from vllm.logger import init_logger`, `from vllm.utils.torch_utils import is_quantized_kv_cache, kv_cache_uses_per_token_head_scales`
