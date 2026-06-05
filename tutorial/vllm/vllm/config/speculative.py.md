# speculative.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/config/speculative.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements speculative support for the `config` portion of vLLM. / 为 vLLM 的 `config` 子目录实现与 speculative 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-19)
```python
import copy

from typing import TYPE_CHECKING, Any, Literal, get_args

from pydantic import Field, SkipValidation, field_validator, model_validator

from typing_extensions import Self

from vllm.config import LoadConfig

from vllm.config.kernel import MoEBackend

from vllm.config.model import ModelConfig

from vllm.config.parallel import ParallelConfig

from vllm.config.utils import config

from vllm.logger import init_logger

from vllm.transformers_utils.config import get_hf_text_config

from vllm.utils.hashing import safe_hash

from vllm.utils.import_utils import LazyLoader, has_arctic_inference

from vllm.v1.attention.backends.registry import AttentionBackendEnum
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 32-70)
```python
logger = init_logger(__name__)

MTPModelTypes = Literal[
    "deepseek_mtp",
    "mimo_mtp",
    "mimo_v2_mtp",
    "glm4_moe_mtp",
    "glm4_moe_lite_mtp",
    "glm_ocr_mtp",
    "ernie_mtp",
    "nemotron_h_mtp",
    "exaone_moe_mtp",
    "exaone4_5_mtp",
    "qwen3_next_mtp",
    "qwen3_5_mtp",
    "longcat_flash_mtp",
    "mtp",
    "pangu_ultra_moe_mtp",
    "step3p5_mtp",
    "hy_v3_mtp",
    "gemma4_mtp",
]

NgramGPUTypes = Literal["ngram_gpu"]

DFlashModelTypes = Literal["dflash"]

EagleModelTypes = Literal[
# ... omitted for brevity ...
```
**EN:** This constant/configuration block defines `logger`, `MTPModelTypes`, `NgramGPUTypes`, `DFlashModelTypes`, `EagleModelTypes`, `SpeculativeMethod`, `RejectionSampleMethod`, `DraftSampleMethod`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `logger`, `MTPModelTypes`, `NgramGPUTypes`, `DFlashModelTypes`, `EagleModelTypes`, `SpeculativeMethod`, `RejectionSampleMethod`, `DraftSampleMethod`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Class `SpeculativeConfig` (lines 74-1088)
```python
class SpeculativeConfig:
    """Configuration for speculative decoding."""

    enforce_eager: bool | None = None
    """Override the default enforce_eager from model_config"""
    # General speculative decoding control
    num_speculative_tokens: int = Field(default=None, gt=0)  # type: ignore[assignment]
    """The number of speculative tokens, if provided. It will default to the
    number in the draft model config if present, otherwise, it is required."""
    model: str | None = None
    """The name of the draft model, eagle head, or additional weights, if
    provided."""
    method: SpeculativeMethod | None = None
    """The name of the speculative method to use. If users provide and set the
    `model` param, the speculative method type will be detected automatically
    if possible, if `model` param is not provided, the method name must be
    provided.

    If using `ngram` method, the related configuration `prompt_lookup_max` and
    `prompt_lookup_min` should be considered."""
    draft_tensor_parallel_size: int | None = Field(default=None, ge=1)
    """The degree of the tensor parallelism for the draft model. Can only be 1
    or the same as the target model's tensor parallel size."""
    # ... omitted for brevity ...
        num_spec_tokens = self.num_speculative_tokens
        return f"SpeculativeConfig({method=}, {model=}, {num_spec_tokens=})"
```
**EN:** Class `SpeculativeConfig` is a structured building block in this module. Key methods include `_acceptance_length_to_rates`, `_resolve_synthetic_acceptance_rates`, `compute_hash`, `hf_config_override`, `__post_init__`, `_validate_suffix_decoding`, which define initialization, validation, transformation, or access patterns. The class docstring says: Configuration for speculative decoding.
**CN:** 类 `SpeculativeConfig` 是该模块中的结构化构件。 关键方法包括 `_acceptance_length_to_rates`, `_resolve_synthetic_acceptance_rates`, `compute_hash`, `hf_config_override`, `__post_init__`, `_validate_suffix_decoding`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Configuration for speculative decoding.

### Method `SpeculativeConfig.compute_hash` (lines 264-298)
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
        factors: list[Any] = []
        # Eagle3 and extract_hidden_states affect the computation graph because
        # they return intermediate hidden states in addition to the final hidden state.
        uses_aux_hidden_states = self.method in (
            "eagle3",
            "extract_hidden_states",
            "dflash",
    # ... omitted for brevity ...
        hash_str = safe_hash(str(factors).encode(), usedforsecurity=False).hexdigest()
        return hash_str
```
**EN:** Method `SpeculativeConfig.compute_hash` computes stable identifiers or fingerprints for caching/comparison. The docstring highlights: WARNING: Whenever a new field is added to this config, ensure that it is included in the factors list if it affects the computation graph. Key calls such as `factors.append`, `getattr`, `tuple`, `safe_hash(str(factors).encode(), usedforsecurity=False).hexdigest`, `safe_hash` show the concrete execution path.
**CN:** Method `SpeculativeConfig.compute_hash` 负责计算稳定标识或指纹以支持缓存/比较。 文档字符串强调：WARNING: Whenever a new field is added to this config, ensure that it is included in the factors list if it affects the computation graph. 像 `factors.append`, `getattr`, `tuple`, `safe_hash(str(factors).encode(), usedforsecurity=False).hexdigest`, `safe_hash` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SpeculativeConfig.hf_config_override` (lines 301-514)
```python
    def hf_config_override(hf_config: PretrainedConfig) -> PretrainedConfig:
        initial_architecture = hf_config.architectures[0]
        if hf_config.model_type in (
            "deepseek_v3",
            "deepseek_v32",
            "glm_moe_dsa",
        ):
            hf_config.model_type = "deepseek_mtp"
        if hf_config.model_type == "deepseek_mtp":
            n_predict = getattr(hf_config, "num_nextn_predict_layers", None)
            hf_config.update(
                {"n_predict": n_predict, "architectures": ["DeepSeekMTPModel"]}
            )
        if hf_config.model_type == "deepseek_v4":
            hf_config.model_type = "deepseek_mtp"
            n_predict = getattr(hf_config, "num_nextn_predict_layers", None)
            hf_config.update(
                {"n_predict": n_predict, "architectures": ["DeepSeekV4MTPModel"]}
            )
    # ... omitted for brevity ...

        return hf_config
```
**EN:** Method `SpeculativeConfig.hf_config_override` parses configuration, arguments, or structured metadata. Key calls such as `getattr`, `hf_config.update`, `hasattr` show the concrete execution path.
**CN:** Method `SpeculativeConfig.hf_config_override` 负责解析配置、参数或结构化元数据。 像 `getattr`, `hf_config.update`, `hasattr` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SpeculativeConfig.__post_init__` (lines 516-806)
```python
    def __post_init__(self):
        # Note: "method" is a new parameter that helps to extend the
        # configuration of non-model-based proposers, and the "model" parameter
        # will be used to set the draft model, eagle head, or additional weight
        # when needed. If users do not specify "method", the speculative method
        # will be detected automatically if possible. If the speculative method
        # can not be detected, it will be considered as the "draft_model" by
        # default.

        # infer method from user args
        # Check if the model field contains a custom module path (e.g., 'pkg.Mod')
        if (
            self.model is not None
            and "." in self.model
            and not self.model.startswith(("http://", "https://", "file://"))
            and "/" not in self.model  # not a HuggingFace repo (org/model)
        ):
            # Treat as a custom class path
            self.method = "custom_class"
    # ... omitted for brevity ...
                )
        return self
```
**EN:** Method `SpeculativeConfig.__post_init__` constructs derived objects, runtime state, or helper structures. Key calls such as `self.model.startswith`, `get_args`, `logger.warning`, `ValueError`, `self._validate_suffix_decoding` show the concrete execution path.
**CN:** Method `SpeculativeConfig.__post_init__` 负责构造派生对象、运行时状态或辅助结构。 像 `self.model.startswith`, `get_args`, `logger.warning`, `ValueError`, `self._validate_suffix_decoding` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SpeculativeConfig.update_arch_` (lines 924-940)
```python
    def update_arch_(self):
        """
        EagleConfig and ExtractHiddenStatesConfig update architectures, so update all
        architectures-related fields in self.draft_model_config
        """
        self.draft_model_config.hf_text_config = get_hf_text_config(
            self.draft_model_config.hf_config
        )
        self.draft_model_config.model_arch_config = (
            self.draft_model_config.get_model_arch_config()
        )
        model_info, arch = self.draft_model_config.registry.inspect_model_cls(
            self.draft_model_config.architectures,
            self.draft_model_config,
        )
        self.draft_model_config._model_info = model_info
        self.draft_model_config._architecture = arch
```
**EN:** Method `SpeculativeConfig.update_arch_` provides a reusable helper around the module's main workflow. The docstring highlights: EagleConfig and ExtractHiddenStatesConfig update architectures, so update all architectures-related fields in self.draft_model_config Key calls such as `get_hf_text_config`, `self.draft_model_config.get_model_arch_config`, `self.draft_model_config.registry.inspect_model_cls` show the concrete execution path.
**CN:** Method `SpeculativeConfig.update_arch_` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：EagleConfig and ExtractHiddenStatesConfig update architectures, so update all architectures-related fields in self.draft_model_config 像 `get_hf_text_config`, `self.draft_model_config.get_model_arch_config`, `self.draft_model_config.registry.inspect_model_cls` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SpeculativeConfig.create_draft_parallel_config` (lines 943-961)
```python
    def create_draft_parallel_config(
        target_parallel_config: ParallelConfig,
        speculative_draft_tensor_parallel_size: int,
    ) -> ParallelConfig:
        """Create a parallel config for use by the draft worker.

        This is mostly a copy of the target parallel config, except the tp_size.
        """
        draft_parallel_config = ParallelConfig(
            pipeline_parallel_size=target_parallel_config.pipeline_parallel_size,
            tensor_parallel_size=speculative_draft_tensor_parallel_size,
            distributed_executor_backend=target_parallel_config.distributed_executor_backend,
            max_parallel_loading_workers=target_parallel_config.max_parallel_loading_workers,
            disable_custom_all_reduce=target_parallel_config.disable_custom_all_reduce,
            ray_workers_use_nsight=target_parallel_config.ray_workers_use_nsight,
            placement_group=target_parallel_config.placement_group,
        )

        return draft_parallel_config
```
**EN:** Method `SpeculativeConfig.create_draft_parallel_config` parses configuration, arguments, or structured metadata. The docstring highlights: Create a parallel config for use by the draft worker. Key calls such as `ParallelConfig` show the concrete execution path.
**CN:** Method `SpeculativeConfig.create_draft_parallel_config` 负责解析配置、参数或结构化元数据。 文档字符串强调：Create a parallel config for use by the draft worker. 像 `ParallelConfig` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SpeculativeConfig.verify_equal_vocab_size_if_draft_model` (lines 1018-1033)
```python
    def verify_equal_vocab_size_if_draft_model(self):
        if (
            self.method == "draft_model"
            and self.target_model_config is not None
            and self.draft_model_config is not None
        ):
            target_vocab_size = self.target_model_config.get_vocab_size()
            draft_vocab_size = self.draft_model_config.get_vocab_size()
            if target_vocab_size != draft_vocab_size:
                raise ValueError(
                    f"Target and draft model should have the same vocabulary size. "
                    f"Target model vocab_size={target_vocab_size}. "
                    f"Draft model vocab_size={draft_vocab_size}. "
                    f"Using models with different tokenizers can cause out-of-bounds "
                    f"errors during speculative decoding."
                )
```
**EN:** Method `SpeculativeConfig.verify_equal_vocab_size_if_draft_model` provides a reusable helper around the module's main workflow. Key calls such as `self.target_model_config.get_vocab_size`, `self.draft_model_config.get_vocab_size`, `ValueError` show the concrete execution path.
**CN:** Method `SpeculativeConfig.verify_equal_vocab_size_if_draft_model` 为模块主流程提供可复用的辅助逻辑。 像 `self.target_model_config.get_vocab_size`, `self.draft_model_config.get_vocab_size`, `ValueError` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Tensor processing / 张量处理**
  - **EN:** The module moves arrays/tensors through typed helper functions or model-facing transformations.
  - **CN:** 该模块会通过带类型的辅助函数或面向模型的变换来处理数组/张量。
- **Caching / 缓存**
  - **EN:** Some definitions are designed to reuse computed state and avoid repeated work.
  - **CN:** 部分定义旨在复用已计算状态，避免重复工作。
- **Configuration / 配置**
  - **EN:** Configuration objects and validation rules centralize feature toggles and runtime settings.
  - **CN:** 配置对象与校验规则集中管理功能开关和运行时设置。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `import copy`, `from typing import TYPE_CHECKING, Any, Literal, get_args`
- **Third-party / 第三方**: `from pydantic import Field, SkipValidation, field_validator, model_validator`, `from typing_extensions import Self`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import LoadConfig`, `from vllm.config.kernel import MoEBackend`, `from vllm.config.model import ModelConfig`, `from vllm.config.parallel import ParallelConfig`, `from vllm.config.utils import config`, `from vllm.logger import init_logger`, `from vllm.transformers_utils.config import get_hf_text_config`, `from vllm.utils.hashing import safe_hash`, `from vllm.utils.import_utils import LazyLoader, has_arctic_inference`, `from vllm.v1.attention.backends.registry import AttentionBackendEnum`
