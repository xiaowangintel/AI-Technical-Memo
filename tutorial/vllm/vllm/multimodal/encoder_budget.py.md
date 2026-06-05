# encoder_budget.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/multimodal/encoder_budget.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements encoder budget support for the `multimodal` portion of vLLM. / 为 vLLM 的 `multimodal` 子目录实现与 编码器budget 相关的支持逻辑。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-10)
```python
from collections.abc import Mapping

from vllm.config import ModelConfig, VllmConfig

from vllm.logger import init_logger

from vllm.multimodal.processing import BaseMultiModalProcessor

from vllm.multimodal.registry import MultiModalRegistry

from vllm.utils.torch_utils import set_default_torch_num_threads

from vllm.v1.core.encoder_cache_manager import compute_mm_encoder_budget
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 12-12)
```python
logger = init_logger(__name__)
```
**EN:** This constant/configuration block defines `logger`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `logger`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Function `get_mm_max_toks_per_item` (lines 15-41)
```python
def get_mm_max_toks_per_item(
    model_config: ModelConfig,
    mm_registry: MultiModalRegistry,
    processor: BaseMultiModalProcessor,
    mm_counts: Mapping[str, int],
) -> Mapping[str, int]:
    """
    Get the maximum number of tokens per data item from each modality based
    on underlying model configuration.
    """
    max_tokens_per_item = processor.info.get_mm_max_tokens_per_item(
        seq_len=model_config.max_model_len,
        mm_counts=mm_counts,
    )
    if max_tokens_per_item is not None:
        return max_tokens_per_item

    mm_inputs = mm_registry.get_dummy_mm_inputs(
        model_config,
        mm_counts=mm_counts,
        processor=processor,
    )

    return {
        modality: sum(item.get_num_embeds() for item in placeholders)
        for modality, placeholders in mm_inputs["mm_placeholders"].items()
    }
```
**EN:** Function `get_mm_max_toks_per_item` provides a reusable helper around the module's main workflow. The docstring highlights: Get the maximum number of tokens per data item from each modality based on underlying model configuration. Key calls such as `processor.info.get_mm_max_tokens_per_item`, `mm_registry.get_dummy_mm_inputs`, `sum`, `item.get_num_embeds`, `mm_inputs['mm_placeholders'].items` show the concrete execution path.
**CN:** Function `get_mm_max_toks_per_item` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Get the maximum number of tokens per data item from each modality based on underlying model configuration. 像 `processor.info.get_mm_max_tokens_per_item`, `mm_registry.get_dummy_mm_inputs`, `sum`, `item.get_num_embeds`, `mm_inputs['mm_placeholders'].items` 这样的关键调用展示了该代码块的具体执行路径。

### Class `MultiModalBudget` (lines 44-193)
```python
class MultiModalBudget:
    """Helper class to calculate budget information for multi-modal models."""

    def __init__(
        self,
        vllm_config: VllmConfig,
        mm_registry: MultiModalRegistry,
    ) -> None:
        super().__init__()

        self.model_config = model_config = vllm_config.model_config
        self.scheduler_config = scheduler_config = vllm_config.scheduler_config

        self.max_model_len = model_config.max_model_len
        self.max_num_reqs = scheduler_config.max_num_seqs

        with set_default_torch_num_threads():  # Avoid hang during startup
            cache = mm_registry.processor_only_cache_from_config(vllm_config)
            processor = mm_registry.create_processor(model_config, cache=cache)

            self.cache = cache
            self.processor = processor
            mm_config = model_config.get_multimodal_config()
    # ... omitted for brevity ...
        if self.cache is not None:
            self.cache.clear_cache()
```
**EN:** Class `MultiModalBudget` is a structured building block in this module. Key methods include `__init__`, `_get_max_items`, `get_modality_with_max_tokens`, `get_encoder_budget`, `reset_cache`, which define initialization, validation, transformation, or access patterns. The class docstring says: Helper class to calculate budget information for multi-modal models.
**CN:** 类 `MultiModalBudget` 是该模块中的结构化构件。 关键方法包括 `__init__`, `_get_max_items`, `get_modality_with_max_tokens`, `get_encoder_budget`, `reset_cache`，它们共同定义初始化、校验、变换或访问模式。 类文档说明：Helper class to calculate budget information for multi-modal models.

### Method `MultiModalBudget.__init__` (lines 47-138)
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        mm_registry: MultiModalRegistry,
    ) -> None:
        super().__init__()

        self.model_config = model_config = vllm_config.model_config
        self.scheduler_config = scheduler_config = vllm_config.scheduler_config

        self.max_model_len = model_config.max_model_len
        self.max_num_reqs = scheduler_config.max_num_seqs

        with set_default_torch_num_threads():  # Avoid hang during startup
            cache = mm_registry.processor_only_cache_from_config(vllm_config)
            processor = mm_registry.create_processor(model_config, cache=cache)

            self.cache = cache
            self.processor = processor
    # ... omitted for brevity ...
        self.mm_max_items_per_prompt: Mapping[str, int] = mm_max_items_per_prompt
        self.mm_max_items_per_batch: Mapping[str, int] = mm_max_items_per_batch
```
**EN:** Method `MultiModalBudget.__init__` initializes the object, wires submodules, stores configuration values, and prepares the runtime state required by later calls. Key calls such as `super().__init__`, `super`, `set_default_torch_num_threads`, `mm_registry.processor_only_cache_from_config`, `mm_registry.create_processor` show the concrete execution path.
**CN:** Method `MultiModalBudget.__init__` 负责初始化对象、组装子模块、保存配置值，并准备后续调用所需的运行时状态。 像 `super().__init__`, `super`, `set_default_torch_num_threads`, `mm_registry.processor_only_cache_from_config`, `mm_registry.create_processor` 这样的关键调用展示了该代码块的具体执行路径。

### Method `MultiModalBudget._get_max_items` (lines 140-180)
```python
    def _get_max_items(
        self,
        modality: str,
        max_tokens_per_item: int,
    ) -> tuple[int, int]:
        if max_tokens_per_item == 0:
            return 0, 0

        # Check how many items of this modality can be supported by
        # the encoder budget.
        if (encoder_budget := self.get_encoder_budget()) == 0:
            return 0, 0

        max_encoder_items_per_batch = encoder_budget // max_tokens_per_item

        # Check how many items of this modality can be supported by
        # the decoder budget.
        mm_limit = self.mm_limits[modality]

    # ... omitted for brevity ...

        return max_items_per_prompt, max_items_per_batch
```
**EN:** Method `MultiModalBudget._get_max_items` provides a reusable helper around the module's main workflow. Key calls such as `self.get_encoder_budget`, `max`, `min` show the concrete execution path.
**CN:** Method `MultiModalBudget._get_max_items` 为模块主流程提供可复用的辅助逻辑。 像 `self.get_encoder_budget`, `max`, `min` 这样的关键调用展示了该代码块的具体执行路径。

### Method `MultiModalBudget.get_modality_with_max_tokens` (lines 182-186)
```python
    def get_modality_with_max_tokens(self) -> str:
        mm_max_toks_per_item = self.mm_max_toks_per_item
        modality, _ = max(mm_max_toks_per_item.items(), key=lambda x: (x[1], x[0]))

        return modality
```
**EN:** Method `MultiModalBudget.get_modality_with_max_tokens` implements an encoding/decoding or token-transformation step. Key calls such as `max`, `mm_max_toks_per_item.items` show the concrete execution path.
**CN:** Method `MultiModalBudget.get_modality_with_max_tokens` 实现编码/解码或 Token 变换步骤。 像 `max`, `mm_max_toks_per_item.items` 这样的关键调用展示了该代码块的具体执行路径。

### Method `MultiModalBudget.get_encoder_budget` (lines 188-189)
```python
    def get_encoder_budget(self) -> int:
        return min(self.encoder_compute_budget, self.encoder_cache_size)
```
**EN:** Method `MultiModalBudget.get_encoder_budget` implements an encoding/decoding or token-transformation step. Key calls such as `min` show the concrete execution path.
**CN:** Method `MultiModalBudget.get_encoder_budget` 实现编码/解码或 Token 变换步骤。 像 `min` 这样的关键调用展示了该代码块的具体执行路径。

### Method `MultiModalBudget.reset_cache` (lines 191-193)
```python
    def reset_cache(self) -> None:
        if self.cache is not None:
            self.cache.clear_cache()
```
**EN:** Method `MultiModalBudget.reset_cache` manages cache lookup, reuse, or lifecycle decisions. Key calls such as `self.cache.clear_cache` show the concrete execution path.
**CN:** Method `MultiModalBudget.reset_cache` 负责管理缓存查询、复用或生命周期决策。 像 `self.cache.clear_cache` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Tensor processing / 张量处理**
  - **EN:** The module moves arrays/tensors through typed helper functions or model-facing transformations.
  - **CN:** 该模块会通过带类型的辅助函数或面向模型的变换来处理数组/张量。
- **Multimodality / 多模态**
  - **EN:** The code contains modality-specific paths for text, image, audio, or video data.
  - **CN:** 代码包含面向文本、图像、音频或视频数据的模态专用路径。
- **Caching / 缓存**
  - **EN:** Some definitions are designed to reuse computed state and avoid repeated work.
  - **CN:** 部分定义旨在复用已计算状态，避免重复工作。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `from collections.abc import Mapping`
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import ModelConfig, VllmConfig`, `from vllm.logger import init_logger`, `from vllm.multimodal.processing import BaseMultiModalProcessor`, `from vllm.multimodal.registry import MultiModalRegistry`, `from vllm.utils.torch_utils import set_default_torch_num_threads`, `from vllm.v1.core.encoder_cache_manager import compute_mm_encoder_budget`
