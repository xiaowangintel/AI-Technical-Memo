# input_processor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/engine/input_processor.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `InputProcessor` for the V1 `engine` subsystem. / 为 V1 的 `engine` 子系统实现 `InputProcessor`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import time
from collections.abc import Mapping
from typing import Any, Literal

import vllm.envs as envs
from vllm.config import VllmConfig
from vllm.inputs import (
    EngineInput,
    PromptType,
    SingletonInput,
    split_enc_dec_input,
)
from vllm.inputs.preprocess import InputPreprocessor
from vllm.logger import init_logger
from vllm.lora.request import LoRARequest
from vllm.multimodal import MULTIMODAL_REGISTRY, MultiModalRegistry
from vllm.multimodal.encoder_budget import MultiModalBudget
from vllm.multimodal.inputs import MultiModalFeatureSpec
from vllm.multimodal.utils import argsort_mm_positions
from vllm.platforms import current_platform
from vllm.pooling_params import PoolingParams
from vllm.renderers import BaseRenderer, renderer_from_config
from vllm.sampling_params import SamplingParams
from vllm.tasks import GENERATION_TASKS, POOLING_TASKS, SupportedTask
from vllm.tokenizers import TokenizerLike
from vllm.utils import length_from_prompt_token_ids_or_embeds, random_uuid
from vllm.utils.jsontree import json_iter_leaves
from vllm.v1.engine import EngineCoreRequest

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `InputProcessor` class / `InputProcessor` 类
```python
class InputProcessor:
```
**EN:** Introduces the `InputProcessor` class. Core methods include `__init__`, `tokenizer`, `get_tokenizer`, `_validate_params`, `_validate_lora`, `_get_mm_identifier`.
**CN:** 这里定义 `InputProcessor` 类。核心方法包括 `__init__`, `tokenizer`, `get_tokenizer`, `_validate_params`, `_validate_lora`, `_get_mm_identifier`。

### `InputProcessor.__init__` method / `InputProcessor.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        renderer: BaseRenderer | None = None,
        *,
        mm_registry: MultiModalRegistry = MULTIMODAL_REGISTRY,
    ) -> None:
        self.vllm_config = vllm_config
        self.model_config = model_config = vllm_config.model_config
        self.cache_config = vllm_config.cache_config
        self.lora_config = vllm_config.lora_config
        self.scheduler_config = vllm_config.scheduler_config
        self.speculative_config = vllm_config.speculative_config
        self.structured_outputs_config = vllm_config.structured_outputs_config
        self.observability_config = vllm_config.observability_config

        self.generation_config_fields = model_config.try_get_generation_config()

        self.renderer = renderer or renderer_from_config(vllm_config)

        self.supports_mm_inputs = mm_registry.supports_multimodal_inputs(model_config)
        self.mm_encoder_cache_size = 0
        self.skip_prompt_length_check = False
        if self.supports_mm_inputs:
            mm_budget = MultiModalBudget(vllm_config, mm_registry)
            self.mm_encoder_cache_size = mm_budget.encoder_cache_size
            self.skip_prompt_length_check = (
                mm_budget.processor.info.skip_prompt_length_check
            )
            mm_budget.reset_cache()  # Not used anymore

        self.input_preprocessor = InputPreprocessor(
            vllm_config,
            renderer=renderer,
            mm_registry=mm_registry,
        )
```
**EN:** This method initializes the object state within `InputProcessor`. Key calls include `try_get_generation_config`, `supports_multimodal_inputs`, `InputPreprocessor`, `renderer_from_config`, `MultiModalBudget`, `reset_cache`. It touches state such as `vllm_config`, `model_config`, `cache_config`, `lora_config`, `scheduler_config`, `speculative_config`, `structured_outputs_config`, `observability_config`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`InputProcessor`。 关键调用包括 `try_get_generation_config`, `supports_multimodal_inputs`, `InputPreprocessor`, `renderer_from_config`, `MultiModalBudget`, `reset_cache`。 它会读写 `vllm_config`, `model_config`, `cache_config`, `lora_config`, `scheduler_config`, `speculative_config`, `structured_outputs_config`, `observability_config` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `InputProcessor._validate_params` method / `InputProcessor._validate_params` 方法
```python
    def _validate_params(
        self,
        params: SamplingParams | PoolingParams,
        supported_tasks: tuple[SupportedTask, ...],
    ) -> None:
        """Raise `ValueError` if SamplingParams or PoolingParams is not valid."""
        if isinstance(params, SamplingParams):
            supported_generation_tasks = [
                task for task in supported_tasks if task in GENERATION_TASKS
            ]
            if not supported_generation_tasks:
                raise ValueError("This model does not support generation")

            params.verify(
                self.model_config,
                self.speculative_config,
                self.structured_outputs_config,
                self.tokenizer,
            )

            if params.thinking_token_budget is not None and (
                self.vllm_config.reasoning_config is None
                or not self.vllm_config.reasoning_config.enabled
            ):
                raise ValueError(
                    "thinking_token_budget is set but reasoning_config is "
                    "not configured. Please set --reasoning-config to use "
                    "thinking_token_budget."
                )
        elif isinstance(params, PoolingParams):
            supported_pooling_tasks = [
                task for task in supported_tasks if task in POOLING_TASKS
            ]
            if not supported_pooling_tasks:
                raise ValueError("This model does not support pooling")

            if params.task is None:
                if "token_embed" in supported_pooling_tasks:
                    params.task = "token_embed"
                elif "token_classify" in supported_pooling_tasks:
                    params.task = "token_classify"
                elif "plugin" in supported_pooling_tasks:
                    params.task = "plugin"

            if params.task not in supported_pooling_tasks:
                raise ValueError(
                    f"Unsupported task: {params.task!r} "
                    f"Supported tasks: {supported_pooling_tasks}"
                )

            params.verify(self.model_config)
        else:
            raise TypeError(
                f"params must be either SamplingParams or PoolingParams, "
                f"but got {type(params).__name__}"
            )
```
**EN:** This method implements `_validate_params` within `InputProcessor`. The docstring frames it as: Raise `ValueError` if SamplingParams or PoolingParams is not valid. Key calls include `isinstance`, `verify`, `ValueError`, `TypeError`, `type`. The control flow contains 10 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_validate_params`，其作用域位于`InputProcessor`。 关键调用包括 `isinstance`, `verify`, `ValueError`, `TypeError`, `type`。 控制流包含 10 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `InputProcessor.process_inputs` method / `InputProcessor.process_inputs` 方法
```python
    def process_inputs(
        self,
        request_id: str,
        prompt: PromptType | EngineInput,
        params: SamplingParams | PoolingParams,
        supported_tasks: tuple[SupportedTask, ...],
        arrival_time: float | None = None,
        lora_request: LoRARequest | None = None,
        tokenization_kwargs: dict[str, Any] | None = None,
        trace_headers: Mapping[str, str] | None = None,
        priority: int = 0,
        data_parallel_rank: int | None = None,
        resumable: bool = False,
    ) -> EngineCoreRequest:
        self._validate_params(params, supported_tasks)
        self._validate_lora(lora_request)

        parallel_config = self.vllm_config.parallel_config
        dp_size = parallel_config.data_parallel_size
        dp_local_size = parallel_config.data_parallel_size_local
        num_ranks = dp_local_size if parallel_config.local_engines_only else dp_size
        if data_parallel_rank is not None and not (0 <= data_parallel_rank < num_ranks):
            raise ValueError(
                f"data_parallel_rank {data_parallel_rank} "
                f"is out of range [0, {num_ranks})."
            )

        if isinstance(prompt, dict) and "type" in prompt:
            if tokenization_kwargs:
                logger.warning_once(
                    "Passing tokenization_kwargs to InputProcessor is deprecated "
                    "and will be removed in v0.18. You should instead pass "
                    "them to Renderer.render_cmpl() or Renderer.render_chat()."
                )

            if arrival_time is None:
                arrival_time = prompt.get("arrival_time", time.time())  # type: ignore[assignment]

            processed_inputs: EngineInput = prompt  # type: ignore[assignment]
        else:
            logger.warning_once(
                "Passing raw prompts to InputProcessor is deprecated "
                "and will be removed in v0.18. You should instead pass "
                "the outputs of Renderer.render_cmpl() or Renderer.render_chat()."
            )

            if arrival_time is None:
                arrival_time = time.time()

            processed_inputs = self.input_preprocessor.preprocess(
    # ... omitted for brevity ...
                        identifier=self._get_mm_identifier(
                            base_mm_hash,
                            lora_request,
                        ),
                        mm_position=decoder_mm_positions[modality][idx],
                        mm_hash=base_mm_hash,
                    )
                )

        return EngineCoreRequest(
            request_id=request_id,
            prompt_token_ids=prompt_token_ids,
            prompt_embeds=prompt_embeds,
            prompt_is_token_ids=prompt_is_token_ids,
            mm_features=mm_features,
            sampling_params=sampling_params,
            pooling_params=pooling_params,
            arrival_time=arrival_time,
            lora_request=lora_request,
            cache_salt=decoder_inputs.get("cache_salt"),
            priority=priority,
            data_parallel_rank=data_parallel_rank,
            trace_headers=trace_headers,
            resumable=resumable,
        )
```
**EN:** This method implements `process_inputs` within `InputProcessor`. Key calls include `_validate_params`, `_validate_lora`, `validate_request`, `split_enc_dec_input`, `_validate_model_inputs`, `isinstance`. The control flow contains 12 branch(es) and 1 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `process_inputs`，其作用域位于`InputProcessor`。 关键调用包括 `_validate_params`, `_validate_lora`, `validate_request`, `split_enc_dec_input`, `_validate_model_inputs`, `isinstance`。 控制流包含 12 个分支和 1 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

## Key Concepts / 关键概念
- `InputProcessor`: central class or interface in this module. / `InputProcessor`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `time`, `collections`, `typing`
- Internal vLLM / 内部依赖: `vllm.envs`, `vllm.config`, `vllm.inputs`, `vllm.inputs.preprocess`, `vllm.logger`, `vllm.lora.request`, `vllm.multimodal`, `vllm.multimodal.encoder_budget`, `vllm.multimodal.inputs`, `vllm.multimodal.utils`, `vllm.platforms`, `vllm.pooling_params`
