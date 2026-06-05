# registry.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/multimodal/registry.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Defines multimodal parsing, data structures, or helpers used by the input pipeline. / 定义输入流水线使用的多模态解析逻辑、数据结构或辅助函数。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-39)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import threading
from collections import defaultdict
from collections.abc import Mapping
from dataclasses import dataclass
from multiprocessing.synchronize import Lock as LockType
from typing import TYPE_CHECKING, Generic, Literal, Protocol, TypeVar, cast

from vllm.inputs import MultiModalInput
from vllm.logger import init_logger
from vllm.tokenizers import TokenizerLike, cached_tokenizer_from_config

from .cache import (
    BaseMultiModalProcessorCache,
    BaseMultiModalReceiverCache,
    MultiModalProcessorOnlyCache,
    MultiModalProcessorSenderCache,
    MultiModalReceiverCache,
    ShmObjectStoreReceiverCache,
    ShmObjectStoreSenderCache,
)
from .processing import (
    BaseDummyInputsBuilder,
    # ...

logger = init_logger(__name__)

N = TypeVar("N", bound=type["SupportsMultiModal"])
_I = TypeVar("_I", bound=BaseProcessingInfo)
_I_co = TypeVar("_I_co", bound=BaseProcessingInfo, covariant=True)
```
**EN:** Sets up the module with standard-library support such as `threading`, `collections`, `collections.abc`, vLLM modules such as `vllm.inputs`, `vllm.logger`, `vllm.tokenizers`. It prepares the symbols later used by `ProcessingInfoFactory`, `DummyInputsBuilderFactory`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.inputs`, `vllm.logger`, `vllm.tokenizers` 等 vLLM 内部依赖。 这些准备工作为后续的 `ProcessingInfoFactory`, `DummyInputsBuilderFactory` 提供上下文。

### ProcessingInfoFactory (lines 42-52)
```python
class ProcessingInfoFactory(Protocol[_I_co]):
    """
    Constructs a
    [`BaseMultiModalProcessor`][vllm.multimodal.processing.BaseMultiModalProcessor]
    instance from the context.
    """

    def __call__(
        self,
        ctx: InputProcessingContext,
    ) -> _I_co: ...
```
**EN:** `ProcessingInfoFactory`: Constructs a [`BaseMultiModalProcessor`][vllm.multimodal.processing.BaseMultiModalProcessor] instance from the context. It extends `Protocol`.
**CN:** `ProcessingInfoFactory` 是该文件中的核心类，用于封装与 `ProcessingInfoFactory` 相关的状态和行为。 它继承自 `Protocol`。

### DummyInputsBuilderFactory (lines 55-62)
```python
class DummyInputsBuilderFactory(Protocol[_I]):  # type: ignore[misc]
    """
    Constructs a
    [`BaseDummyInputsBuilder`][vllm.multimodal.processing.BaseDummyInputsBuilder]
    instance from the context.
    """

    def __call__(self, info: _I) -> BaseDummyInputsBuilder[_I]: ...
```
**EN:** `DummyInputsBuilderFactory`: Constructs a [`BaseDummyInputsBuilder`][vllm.multimodal.processing.BaseDummyInputsBuilder] instance from the context. It extends `Protocol`.
**CN:** `DummyInputsBuilderFactory` 是该文件中的核心类，用于封装与 `DummyInputsBuilderFactory` 相关的状态和行为。 它继承自 `Protocol`。

### MultiModalProcessorFactory (lines 65-78)
```python
class MultiModalProcessorFactory(Protocol[_I]):  # type: ignore[misc]
    """
    Constructs a
    [`BaseMultiModalProcessor`][vllm.multimodal.processing.BaseMultiModalProcessor]
    instance from the context.
    """

    def __call__(
        self,
        info: _I,
        dummy_inputs: BaseDummyInputsBuilder[_I],
        *,
        cache: BaseMultiModalProcessorCache | None = None,
    ) -> BaseMultiModalProcessor[_I]: ...
```
**EN:** `MultiModalProcessorFactory`: Constructs a [`BaseMultiModalProcessor`][vllm.multimodal.processing.BaseMultiModalProcessor] instance from the context. It extends `Protocol`.
**CN:** `MultiModalProcessorFactory` 是该文件中的核心类，用于封装与 `MultiModalProcessorFactory` 相关的状态和行为。 它继承自 `Protocol`。

### _ProcessorFactories (lines 82-95)
```python
class _ProcessorFactories(Generic[_I]):
    info: ProcessingInfoFactory[_I]
    processor: MultiModalProcessorFactory[_I]
    dummy_inputs: DummyInputsBuilderFactory[_I]

    def build_processor(
        self,
        ctx: InputProcessingContext,
        *,
        cache: BaseMultiModalProcessorCache | None = None,
    ):
        info = self.info(ctx)
        dummy_inputs_builder = self.dummy_inputs(info)
        return self.processor(info, dummy_inputs_builder, cache=cache)
```
**EN:** Defines the `_ProcessorFactories` class used by this module. It extends `Generic`. Key methods include `build_processor`.
**CN:** `_ProcessorFactories` 是该文件中的核心类，用于封装与 `_ProcessorFactories` 相关的状态和行为。 它继承自 `Generic`。 关键方法包括 `build_processor`。

### MultiModalRegistry overview (lines 98-347)
```python
class MultiModalRegistry:
    """
    A registry that dispatches data processing according to the model.
    """

    def supports_multimodal_inputs(self, model_config: "ModelConfig") -> bool:
        """
        Checks if the model supports multimodal inputs.
        Returns True if the model is multimodal with any non-zero supported
        modalities, otherwise returns False, effectively running in
        text-only mode.
        """
        if not model_config.is_multimodal_model:
            return False

        mm_config = model_config.get_multimodal_config()
        try:
            info = self._create_processing_info(model_config, tokenizer=None)
        except ValueError:
            logger.warning_once(
                "Model %s is treated as multimodal but has no registered "
                "multimodal processor; running in text-only mode.",
                model_config.model,
            )
            return False

        # Check if all supported modalities have limit == 0
    # ...
```
**EN:** `MultiModalRegistry`: A registry that dispatches data processing according to the model. Key methods include `supports_multimodal_inputs`, `register_processor`, `get_processing_info`, `create_processor`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `MultiModalRegistry` 是该文件中的核心类，用于封装与 `MultiModalRegistry` 相关的状态和行为。 关键方法包括 `supports_multimodal_inputs`, `register_processor`, `get_processing_info`, `create_processor`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### MultiModalRegistry.get_dummy_mm_inputs (lines 232-266)
```python
    def get_dummy_mm_inputs(
        self,
        model_config: "ModelConfig",
        mm_counts: Mapping[str, int],
        *,
        cache: BaseMultiModalProcessorCache | None = None,
        processor: BaseMultiModalProcessor | None = None,
    ) -> MultiModalInput:
        """
        Create dummy data for profiling the memory usage of a model.

        The model is identified by `model_config`.
        """
        seq_len = model_config.max_model_len

        if processor is None:
            processor = self.create_processor(model_config, cache=cache)

        mm_config = model_config.get_multimodal_config()
        processor_inputs = processor.dummy_inputs.get_dummy_processor_inputs(
    # ...
        prompt_token_ids = mm_inputs["prompt_token_ids"]
        total_len = len(prompt_token_ids)
        if total_len < seq_len:
            prompt_token_ids.extend([0] * (seq_len - total_len))

        return mm_inputs
```
**EN:** `get_dummy_mm_inputs`: Create dummy data for profiling the memory usage of a model. It mainly works with `model_config`, `mm_counts`, `cache`, `processor`. Inside the body, it relies on `model_config.get_multimodal_config`, `processor.dummy_inputs.get_dummy_processor_inputs`, `processor.apply` to complete the main steps.
**CN:** `get_dummy_mm_inputs` 负责获取流水线所需的数据或状态。 它主要处理 `model_config`, `mm_counts`, `cache`, `processor` 等参数。 实现过程中会调用 `model_config.get_multimodal_config`, `processor.dummy_inputs.get_dummy_processor_inputs`, `processor.apply` 等函数完成关键步骤。

### MultiModalRegistry.register_processor (lines 142-174)
```python
    def register_processor(
        self,
        processor: MultiModalProcessorFactory[_I],
        *,
        info: ProcessingInfoFactory[_I],
        dummy_inputs: DummyInputsBuilderFactory[_I],
    ):
        """
        Register a multi-modal processor to a model class. The processor
        is constructed lazily, hence a factory method should be passed.

        When the model receives multi-modal data, the provided function is
        invoked to transform the data into a dictionary of model inputs.
        """

        def wrapper(model_cls: N) -> N:
            if "_processor_factory" in model_cls.__dict__:
                logger.warning(
                    "Model class %s already has a multi-modal processor "
                    "registered to %s. It is overwritten by the new one.",
    # ...
                processor=processor,
            )

            return model_cls

        return wrapper
```
**EN:** `register_processor`: Register a multi-modal processor to a model class. It mainly works with `processor`, `info`, `dummy_inputs`. Inside the body, it relies on `_ProcessorFactories`, `logger.warning` to complete the main steps.
**CN:** `register_processor` 负责在分发表中注册实现。 它主要处理 `processor`, `info`, `dummy_inputs` 等参数。 实现过程中会调用 `_ProcessorFactories`, `logger.warning` 等函数完成关键步骤。

### MultiModalRegistry.create_processor (lines 211-230)
```python
    def create_processor(
        self,
        model_config: "ModelConfig",
        *,
        tokenizer: TokenizerLike | None = None,
        cache: BaseMultiModalProcessorCache | None = None,
    ) -> BaseMultiModalProcessor[BaseProcessingInfo]:
        """
        Create a multi-modal processor for a specific model and tokenizer.
        """
        if not model_config.is_multimodal_model:
            model_name = model_config.served_model_name or model_config.model
            raise ValueError(f"{model_name} is not a multimodal model")

        model_cls = self._get_model_cls(model_config)
        factories = model_cls._processor_factory

        ctx = self._create_processing_ctx(model_config, tokenizer)

        return factories.build_processor(ctx, cache=cache)
```
**EN:** `create_processor`: Create a multi-modal processor for a specific model and tokenizer. It mainly works with `model_config`, `tokenizer`, `cache`. Inside the body, it relies on `self._get_model_cls`, `self._create_processing_ctx`, `factories.build_processor` to complete the main steps.
**CN:** `create_processor` 负责构造新的对象或配置集合。 它主要处理 `model_config`, `tokenizer`, `cache` 等参数。 实现过程中会调用 `self._get_model_cls`, `self._create_processing_ctx`, `factories.build_processor` 等函数完成关键步骤。

### MultiModalRegistry.supports_multimodal_inputs (lines 103-140)
```python
    def supports_multimodal_inputs(self, model_config: "ModelConfig") -> bool:
        """
        Checks if the model supports multimodal inputs.
        Returns True if the model is multimodal with any non-zero supported
        modalities, otherwise returns False, effectively running in
        text-only mode.
        """
        if not model_config.is_multimodal_model:
            return False

        mm_config = model_config.get_multimodal_config()
        try:
            info = self._create_processing_info(model_config, tokenizer=None)
        except ValueError:
            logger.warning_once(
                "Model %s is treated as multimodal but has no registered "
                "multimodal processor; running in text-only mode.",
                model_config.model,
            )
            return False
    # ...
                "All limits of multimodal modalities supported by the model "
                "are set to 0, running in text-only mode."
            )
            return False

        return True
```
**EN:** `supports_multimodal_inputs`: Checks if the model supports multimodal inputs. It mainly works with `model_config`. Inside the body, it relies on `model_config.get_multimodal_config`, `all`, `self._create_processing_info` to complete the main steps.
**CN:** `supports_multimodal_inputs` 负责实现本模块使用的辅助逻辑。 它主要处理 `model_config` 等参数。 实现过程中会调用 `model_config.get_multimodal_config`, `all`, `self._create_processing_info` 等函数完成关键步骤。

### MultiModalTimingRegistry (lines 350-378)
```python
class MultiModalTimingRegistry:
    def __init__(self, observability_config: "ObservabilityConfig | None") -> None:
        super().__init__()

        if observability_config and observability_config.enable_mm_processor_stats:
            self._lock = threading.Lock()
            self._ctx_by_request_id = defaultdict[str, TimingContext](TimingContext)
            self._enabled = True
        else:
            self._enabled = False

    def get(self, request_id: str) -> TimingContext:
        if not self._enabled:
            return TimingContext(enabled=False)

        with self._lock:
            return self._ctx_by_request_id[request_id]

    def stat(self) -> dict[str, dict[str, float]]:
        if not self._enabled:
            return {}

        with self._lock:
            stats = {
                req_id: ctx.get_stats_dict()
                for req_id, ctx in self._ctx_by_request_id.items()
            }
            self._ctx_by_request_id.clear()
            return stats
```
**EN:** Defines the `MultiModalTimingRegistry` class used by this module. Key methods include `__init__`, `get`, `stat`.
**CN:** `MultiModalTimingRegistry` 是该文件中的核心类，用于封装与 `MultiModalTimingRegistry` 相关的状态和行为。 关键方法包括 `__init__`, `get`, `stat`。

## Key Concepts / 关键概念
- **`ProcessingInfoFactory`**: Core class that organizes module behavior. / **`ProcessingInfoFactory`**：组织模块行为的核心类。
- **`DummyInputsBuilderFactory`**: Core class that organizes module behavior. / **`DummyInputsBuilderFactory`**：组织模块行为的核心类。
- **`MultiModalProcessorFactory`**: Core class that organizes module behavior. / **`MultiModalProcessorFactory`**：组织模块行为的核心类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: threading, collections, collections.abc, dataclasses, multiprocessing.synchronize, typing
- **Third-party / 第三方**: None / 无
- **Internal vLLM / vLLM 内部依赖**: vllm.inputs, vllm.logger, vllm.tokenizers, .cache, .processing, vllm.config, vllm.model_executor.models.interfaces, vllm.model_executor.model_loader
