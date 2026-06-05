# scheduler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/config/scheduler.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements scheduler support for the `config` portion of vLLM. / 为 vLLM 的 `config` 子目录实现与 scheduler 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-14)
```python
from collections.abc import Callable

from dataclasses import InitVar

from typing import TYPE_CHECKING, Any, ClassVar, Literal, cast

from pydantic import Field, field_validator

from typing_extensions import Self

from vllm.config.utils import config

from vllm.logger import init_logger

from vllm.utils.hashing import safe_hash

from vllm.utils.import_utils import resolve_obj_by_qualname
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 19-22)
```python
logger = init_logger(__name__)

RunnerType = Literal["generate", "pooling", "draft"]

SchedulerPolicy = Literal["fcfs", "priority"]
```
**EN:** This constant/configuration block defines `logger`, `RunnerType`, `SchedulerPolicy`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `logger`, `RunnerType`, `SchedulerPolicy`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Class `SchedulerConfig` (lines 26-308)
```python
class SchedulerConfig:
    """Scheduler configuration."""

    max_model_len: InitVar[int]
    """Maximum length of a sequence (including prompt and generated text).

    Note: This is stored in the ModelConfig, and is used only here to
    provide fallbacks and validate other attributes."""

    is_encoder_decoder: InitVar[bool]
    """True if the model is an encoder-decoder model.

    Note: This is stored in the ModelConfig, and is used only here to
    disable chunked prefill and prefix caching for encoder-decoder models.
    """

    DEFAULT_MAX_NUM_BATCHED_TOKENS: ClassVar[int] = 2048
    DEFAULT_MAX_NUM_BATCHED_TOKENS_FOR_BATCHED_DP: ClassVar[int] = 256
    DEFAULT_MAX_NUM_SEQS: ClassVar[int] = 128

    runner_type: RunnerType = "generate"
    """The runner type to launch for the model."""

    # ... omitted for brevity ...

        return self
```
**EN:** Class `SchedulerConfig` is a structured building block in this module. Key methods include `default_factory`, `get_scheduler_cls`, `compute_hash`, `_skip_none_validation`, `__post_init__`, `verify_max_model_len`, which define initialization, validation, transformation, or access patterns. The class docstring says: Scheduler configuration.
**CN:** 类 `SchedulerConfig` 是该模块中的结构化构件。 关键方法包括 `default_factory`, `get_scheduler_cls`, `compute_hash`, `_skip_none_validation`, `__post_init__`, `verify_max_model_len`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Scheduler configuration.

### Method `SchedulerConfig.default_factory` (lines 158-166)
```python
    def default_factory(**kwargs):
        """
        Factory method to create `SchedulerConfig` with default values for `InitVar`s.
        """
        if "max_model_len" not in kwargs:
            kwargs["max_model_len"] = 8192
        if "is_encoder_decoder" not in kwargs:
            kwargs["is_encoder_decoder"] = False
        return SchedulerConfig(**kwargs)
```
**EN:** Method `SchedulerConfig.default_factory` provides a reusable helper around the module's main workflow. The docstring highlights: Factory method to create `SchedulerConfig` with default values for `InitVar`s. Key calls such as `SchedulerConfig` show the concrete execution path.
**CN:** Method `SchedulerConfig.default_factory` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Factory method to create `SchedulerConfig` with default values for `InitVar`s. 像 `SchedulerConfig` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SchedulerConfig.get_scheduler_cls` (lines 168-188)
```python
    def get_scheduler_cls(self) -> type["SchedulerInterface"]:
        if self.scheduler_cls is None:
            if self.async_scheduling:
                from vllm.v1.core.sched.async_scheduler import AsyncScheduler

                return AsyncScheduler
            from vllm.v1.core.sched.scheduler import Scheduler

            return Scheduler

        # This warning can be removed once the Scheduler interface is
        # finalized and we can maintain support for scheduler classes that
        # implement it
        logger.warning_once(
            "Using custom scheduler class %s. This scheduler interface is "
            "not public and compatibility may not be maintained.",
            self.scheduler_cls,  # type: ignore[arg-type]
        )
        if not isinstance(self.scheduler_cls, str):
            return cast(type["SchedulerInterface"], self.scheduler_cls)
        return resolve_obj_by_qualname(self.scheduler_cls)
```
**EN:** Method `SchedulerConfig.get_scheduler_cls` provides a reusable helper around the module's main workflow. Key calls such as `logger.warning_once`, `isinstance`, `cast`, `resolve_obj_by_qualname` show the concrete execution path.
**CN:** Method `SchedulerConfig.get_scheduler_cls` 为模块主流程提供可复用的辅助逻辑。 像 `logger.warning_once`, `isinstance`, `cast`, `resolve_obj_by_qualname` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SchedulerConfig.compute_hash` (lines 190-216)
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

        # max_num_batched_tokens need to be included in the hash due
        # to two reasons:
        # 1. LoRA creates static buffers based on max_num_batched_tokens.
        #   The tensor sizes and strides get captured in the torch.compile
        #   graph explicitly.
    # ... omitted for brevity ...
        hash_str = safe_hash(str(factors).encode(), usedforsecurity=False).hexdigest()
        return hash_str
```
**EN:** Method `SchedulerConfig.compute_hash` computes stable identifiers or fingerprints for caching/comparison. The docstring highlights: WARNING: Whenever a new field is added to this config, ensure that it is included in the factors list if it affects the computation graph. Key calls such as `factors.append`, `safe_hash(str(factors).encode(), usedforsecurity=False).hexdigest`, `safe_hash`, `str(factors).encode`, `str` show the concrete execution path.
**CN:** Method `SchedulerConfig.compute_hash` 负责计算稳定标识或指纹以支持缓存/比较。 文档字符串强调：WARNING: Whenever a new field is added to this config, ensure that it is included in the factors list if it affects the computation graph. 像 `factors.append`, `safe_hash(str(factors).encode(), usedforsecurity=False).hexdigest`, `safe_hash`, `str(factors).encode`, `str` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SchedulerConfig._skip_none_validation` (lines 220-222)
```python
    def _skip_none_validation(cls, value: Any, handler: Callable) -> Any:
        """Skip validation if the value is `None` when initialisation is delayed."""
        return None if value is None else handler(value)
```
**EN:** Method `SchedulerConfig._skip_none_validation` provides a reusable helper around the module's main workflow. The docstring highlights: Skip validation if the value is `None` when initialisation is delayed. Key calls such as `handler`, `field_validator` show the concrete execution path.
**CN:** Method `SchedulerConfig._skip_none_validation` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Skip validation if the value is `None` when initialisation is delayed. 像 `handler`, `field_validator` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SchedulerConfig.__post_init__` (lines 224-257)
```python
    def __post_init__(self, max_model_len: int, is_encoder_decoder: bool) -> None:
        if is_encoder_decoder:
            # Chunked prefill should be disabled for encoder-decoder models.
            self.disable_chunked_mm_input = True
            self.enable_chunked_prefill = False
            self.long_prefill_token_threshold = 0
            logger.info(
                "Encoder-decoder models do not support chunked prefill nor"
                " prefix caching; disabling both."
            )

        self.max_num_encoder_input_tokens = self.max_num_batched_tokens
        self.encoder_cache_size = self.max_num_batched_tokens

        if self.enable_chunked_prefill:
            logger.info_once(
                "Chunked prefill is enabled with max_num_batched_tokens=%d.",
                self.max_num_batched_tokens,
            )
    # ... omitted for brevity ...

        self.verify_max_model_len(max_model_len)
```
**EN:** Method `SchedulerConfig.__post_init__` constructs derived objects, runtime state, or helper structures. Key calls such as `logger.info`, `logger.info_once`, `int`, `self.verify_max_model_len` show the concrete execution path.
**CN:** Method `SchedulerConfig.__post_init__` 负责构造派生对象、运行时状态或辅助结构。 像 `logger.info`, `logger.info_once`, `int`, `self.verify_max_model_len` 这样的关键调用展示了该代码块的具体执行路径。

### Method `SchedulerConfig.verify_max_model_len` (lines 259-308)
```python
    def verify_max_model_len(self, max_model_len: int) -> Self:
        if (
            self.max_num_batched_tokens < max_model_len
            and not self.enable_chunked_prefill
        ):
            raise ValueError(
                f"max_num_batched_tokens ({self.max_num_batched_tokens}) is "
                f"smaller than max_model_len ({max_model_len}). "
                "This effectively limits the maximum sequence length to "
                "max_num_batched_tokens and makes vLLM reject longer "
                "sequences. Please increase max_num_batched_tokens or "
                "decrease max_model_len."
            )

        if self.max_num_batched_tokens < self.max_num_seqs:
            raise ValueError(
                f"max_num_batched_tokens ({self.max_num_batched_tokens}) must "
                "be greater than or equal to max_num_seqs "
                f"({self.max_num_seqs})."
    # ... omitted for brevity ...

        return self
```
**EN:** Method `SchedulerConfig.verify_max_model_len` provides a reusable helper around the module's main workflow. Key calls such as `ValueError`, `logger.warning` show the concrete execution path.
**CN:** Method `SchedulerConfig.verify_max_model_len` 为模块主流程提供可复用的辅助逻辑。 像 `ValueError`, `logger.warning` 这样的关键调用展示了该代码块的具体执行路径。

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
- **Standard library / 标准库**: `from collections.abc import Callable`, `from dataclasses import InitVar`, `from typing import TYPE_CHECKING, Any, ClassVar, Literal, cast`
- **Third-party / 第三方**: `from pydantic import Field, field_validator`, `from typing_extensions import Self`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config.utils import config`, `from vllm.logger import init_logger`, `from vllm.utils.hashing import safe_hash`, `from vllm.utils.import_utils import resolve_obj_by_qualname`
