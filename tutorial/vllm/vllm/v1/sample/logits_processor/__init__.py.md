# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/sample/logits_processor/__init__.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_load_logitsprocs_plugins`, `_load_logitsprocs_by_fqcns`, `_load_custom_logitsprocs` for the V1 `sample/logits_processor` subsystem. / 为 V1 的 `sample/logits_processor` 子系统实现 `_load_logitsprocs_plugins`, `_load_logitsprocs_by_fqcns`, `_load_custom_logitsprocs`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import importlib
import inspect
import itertools
from abc import abstractmethod
from collections.abc import Sequence
from functools import lru_cache, partial
from typing import TYPE_CHECKING

import torch

from vllm.logger import init_logger
from vllm.logits_process import LogitsProcessor as RequestLogitsProcessor
from vllm.sampling_params import SamplingParams
from vllm.utils.torch_utils import guard_cuda_initialization
from vllm.v1.sample.logits_processor.builtin import (
    LogitBiasLogitsProcessor,
    MinPLogitsProcessor,
    MinTokensLogitsProcessor,
    process_dict_updates,
)
from vllm.v1.sample.logits_processor.interface import (
    BatchUpdate,
    LogitsProcessor,
    MoveDirectionality,
)
from vllm.v1.sample.logits_processor.state import BatchUpdateBuilder, LogitsProcessors

if TYPE_CHECKING:
    from vllm.config import VllmConfig

logger = init_logger(__name__)

# Error message when the user tries to initialize vLLM with a pooling model
# and custom logitsproces
STR_POOLING_REJECTS_LOGITSPROCS = (
    "Pooling models do not support custom logits processors."
)

# Error message when the user tries to initialize vLLM with a speculative
# decoding enabled and custom logitsproces
STR_SPEC_DEC_REJECTS_LOGITSPROCS = (
    "Custom logits processors are not supported when speculative decoding is enabled."
)

LOGITSPROCS_GROUP = "vllm.logits_processors"

BUILTIN_LOGITS_PROCESSORS: list[type[LogitsProcessor]] = [
    MinTokensLogitsProcessor,
    LogitBiasLogitsProcessor,
    MinPLogitsProcessor,
]
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`, `STR_POOLING_REJECTS_LOGITSPROCS`, `STR_SPEC_DEC_REJECTS_LOGITSPROCS`, `LOGITSPROCS_GROUP`, `BUILTIN_LOGITS_PROCESSORS`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`, `STR_POOLING_REJECTS_LOGITSPROCS`, `STR_SPEC_DEC_REJECTS_LOGITSPROCS`, `LOGITSPROCS_GROUP`, `BUILTIN_LOGITS_PROCESSORS`。

### `_load_logitsprocs_plugins` function / `_load_logitsprocs_plugins` 函数
```python
def _load_logitsprocs_plugins() -> list[type[LogitsProcessor]]:
    """Load all installed logit processor plugins"""

    from importlib.metadata import entry_points

    installed_logitsprocs_plugins = entry_points(group=LOGITSPROCS_GROUP)
    if len(installed_logitsprocs_plugins) == 0:
        logger.debug("No logitsprocs plugins installed (group %s).", LOGITSPROCS_GROUP)
        return []

    # Load logitsprocs plugins
    logger.debug("Loading installed logitsprocs plugins (group %s):", LOGITSPROCS_GROUP)
    classes: list[type[LogitsProcessor]] = []
    for entrypoint in installed_logitsprocs_plugins:
        try:
            logger.debug(
                "- Loading logitproc plugin entrypoint=%s target=%s",
                entrypoint.name,
                entrypoint.value,
            )
            with guard_cuda_initialization():
                classes.append(entrypoint.load())
        except Exception as e:
            logger.error("Failed to load LogitsProcessor plugin %s: %s", entrypoint, e)
            raise RuntimeError(
                f"Failed to load LogitsProcessor plugin {entrypoint}"
            ) from e
    return classes
```
**EN:** This function implements `_load_logitsprocs_plugins` within the module. The docstring frames it as: Load all installed logit processor plugins Key calls include `entry_points`, `debug`, `len`, `guard_cuda_initialization`, `append`, `error`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_load_logitsprocs_plugins`，其作用域位于the module。 关键调用包括 `entry_points`, `debug`, `len`, `guard_cuda_initialization`, `append`, `error`。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `_load_logitsprocs_by_fqcns` function / `_load_logitsprocs_by_fqcns` 函数
```python
def _load_logitsprocs_by_fqcns(
    logits_processors: Sequence[str | type[LogitsProcessor]] | None,
) -> list[type[LogitsProcessor]]:
    """Load logit processor types, identifying them by fully-qualified class
    names (FQCNs).

    Effectively, a mixed list of logitproc types and FQCN strings is converted
    into a list of entirely logitproc types, by loading from the FQCNs.

    FQCN syntax is <module>:<type> i.e. x.y.z:CustomLogitProc

    Already-loaded logitproc types must be subclasses of LogitsProcessor

    Args:
      logits_processors: Potentially mixed list of logitsprocs types and FQCN
                         strings for logitproc types

    Returns:
      List of logitproc types

    """
    if not logits_processors:
        return []

    logger.debug(
        "%s additional custom logits processors specified, checking whether "
        "they need to be loaded.",
        len(logits_processors),
    )

    classes: list[type[LogitsProcessor]] = []
    for ldx, logitproc in enumerate(logits_processors):
        if isinstance(logitproc, type):
            logger.debug(" - Already-loaded logit processor: %s", logitproc.__name__)
            if not issubclass(logitproc, LogitsProcessor):
                raise ValueError(
                    f"{logitproc.__name__} is not a subclass of LogitsProcessor"
                )
            classes.append(logitproc)
            continue

        logger.debug("- Loading logits processor %s", logitproc)
        module_path, qualname = logitproc.split(":")

        try:
            # Load module
            with guard_cuda_initialization():
                module = importlib.import_module(module_path)
        except Exception as e:
            logger.error(
                "Failed to load %sth LogitsProcessor plugin %s: %s",
                ldx,
                logitproc,
                e,
            )
            raise RuntimeError(
                f"Failed to load {ldx}th LogitsProcessor plugin {logitproc}"
            ) from e

        # Walk down dotted name to get logitproc class
        obj = module
        for attr in qualname.split("."):
            obj = getattr(obj, attr)
        if not isinstance(obj, type):
            raise ValueError("Loaded logit processor must be a type.")
        if not issubclass(obj, LogitsProcessor):
            raise ValueError(f"{obj.__name__} must be a subclass of LogitsProcessor")
        classes.append(obj)

    return classes
```
**EN:** This function implements `_load_logitsprocs_by_fqcns` within the module. The docstring frames it as: Load logit processor types, identifying them by fully-qualified class names (FQCNs). Key calls include `debug`, `enumerate`, `len`, `isinstance`, `split`, `append`. The control flow contains 6 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_load_logitsprocs_by_fqcns`，其作用域位于the module。 关键调用包括 `debug`, `enumerate`, `len`, `isinstance`, `split`, `append`。 控制流包含 6 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `_load_custom_logitsprocs` function / `_load_custom_logitsprocs` 函数
```python
def _load_custom_logitsprocs(
    logits_processors: Sequence[str | type[LogitsProcessor]] | None,
) -> list[type[LogitsProcessor]]:
    """Load all custom logits processors.

    * First load all installed logitproc plugins
    * Second load custom logitsprocs pass by the user at initialization time

    Args:
      logits_processors: potentially mixed list of logitproc types and
                         logitproc type fully-qualified names (FQCNs)
                         which need to be loaded

    Returns:
      A list of all loaded logitproc types
    """
    from vllm.platforms import current_platform

    if current_platform.is_tpu():
        # No logitsprocs specified by caller
        # TODO(andy) - vLLM V1 on TPU does not support custom logitsprocs
        return []

    return _load_logitsprocs_plugins() + _load_logitsprocs_by_fqcns(logits_processors)
```
**EN:** This function implements `_load_custom_logitsprocs` within the module. The docstring frames it as: Load all custom logits processors. Key calls include `is_tpu`, `_load_logitsprocs_plugins`, `_load_logitsprocs_by_fqcns`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_load_custom_logitsprocs`，其作用域位于the module。 关键调用包括 `is_tpu`, `_load_logitsprocs_plugins`, `_load_logitsprocs_by_fqcns`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `build_logitsprocs` function / `build_logitsprocs` 函数
```python
def build_logitsprocs(
    vllm_config: "VllmConfig",
    device: torch.device,
    is_pin_memory: bool,
    is_pooling_model: bool,
    custom_logitsprocs: Sequence[str | type[LogitsProcessor]] = (),
) -> LogitsProcessors:
    if is_pooling_model:
        if custom_logitsprocs:
            raise ValueError(STR_POOLING_REJECTS_LOGITSPROCS)
        logger.debug(
            "Skipping logits processor loading because pooling models"
            " do not support logits processors."
        )
        return LogitsProcessors()

    # Check if speculative decoding is enabled.
    if vllm_config.speculative_config:
        if custom_logitsprocs:
            raise ValueError(STR_SPEC_DEC_REJECTS_LOGITSPROCS)
        logger.warning(
            "min_p and logit_bias parameters won't work with speculative decoding."
        )
        return LogitsProcessors(
            [MinTokensLogitsProcessor(vllm_config, device, is_pin_memory)]
        )

    custom_logitsprocs_classes = _load_custom_logitsprocs(custom_logitsprocs)
    return LogitsProcessors(
        ctor(vllm_config, device, is_pin_memory)
        for ctor in itertools.chain(
            BUILTIN_LOGITS_PROCESSORS, custom_logitsprocs_classes
        )
    )
```
**EN:** This function builds derived structures within the module. Key calls include `_load_custom_logitsprocs`, `LogitsProcessors`, `debug`, `warning`, `ValueError`, `ctor`. The control flow contains 4 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会构建派生结构，其作用域位于the module。 关键调用包括 `_load_custom_logitsprocs`, `LogitsProcessors`, `debug`, `warning`, `ValueError`, `ctor`。 控制流包含 4 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### Module constants / 模块常量
```python
cached_load_custom_logitsprocs = lru_cache(_load_custom_logitsprocs)
```
**EN:** Defines module-level constants or aliases such as `cached_load_custom_logitsprocs`, which are reused by later definitions.
**CN:** 定义 `cached_load_custom_logitsprocs` 等模块级常量或别名，供后续定义复用。

### `validate_logits_processors_parameters` function / `validate_logits_processors_parameters` 函数
```python
def validate_logits_processors_parameters(
    logits_processors: Sequence[str | type[LogitsProcessor]] | None,
    sampling_params: SamplingParams,
):
    logits_processors = (
        tuple(logits_processors) if logits_processors is not None else None
    )
    for logits_procs in cached_load_custom_logitsprocs(logits_processors):
        logits_procs.validate_params(sampling_params)
```
**EN:** This function validates assumptions or constraints within the module. Key calls include `cached_load_custom_logitsprocs`, `tuple`, `validate_params`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会校验前提与约束，其作用域位于the module。 关键调用包括 `cached_load_custom_logitsprocs`, `tuple`, `validate_params`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `AdapterLogitsProcessor` class / `AdapterLogitsProcessor` 类
```python
class AdapterLogitsProcessor(LogitsProcessor):
    """Wrapper for per-request logits processors

    To wrap a specific per-request logits processor,
    * Subclass `AdapterLogitsProcessor`
    * Implement `self.is_argmax_invariant()` base-class method
    * Implement `self.new_req_logits_processor(params)`

    `self.__init__(vllm_config, device, is_pin_memory)` does not need to be
    overridden in general. However, to implement custom constructor behavior -
    especially any logic which operates on or stores `vllm_config`, `device`,
    or `is_pin_memory` - `self.__init__(vllm_config, device, is_pin_memory)`
    must be overridden and the override must call
    `super().__init__(vllm_config, device, is_pin_memory)`
    """
```
**EN:** Introduces the `AdapterLogitsProcessor` class on top of `LogitsProcessor`. Core methods include `__init__`, `new_req_logits_processor`, `_new_state`, `update_state`, `apply`. Docstring signal: Wrapper for per-request logits processors To wrap a specific per-request logits processor, * Subclass `AdapterLogitsProcessor` * Implement `self.is_argmax_invariant()` base-class method * Implement `self.new_req_logits_processor(params)` `self.__init__(vllm_config, device, is_pin_memory)` does not need to be overridden in general.
**CN:** 这里定义 `AdapterLogitsProcessor` 类，其基类包括 `LogitsProcessor`。核心方法包括 `__init__`, `new_req_logits_processor`, `_new_state`, `update_state`, `apply`。

### `AdapterLogitsProcessor.__init__` method / `AdapterLogitsProcessor.__init__` 方法
```python
    def __init__(
        self, vllm_config: "VllmConfig", device: torch.device, is_pin_memory: bool
    ):
        """Subclass must invoke
        `super().__init__(vllm_config, device, is_pin_memory)`.

        Subclass constructor may find it useful to utilize the `vllm_config`,
        `device` and `is_pin_memory` argument. However regardless of whether
        these arguments are used, the vLLM logits processor interface requires
        all three arguments to be present.
        """

        # Map req index -> logits processor state
        #
        # State representation is a partial[Tensor] comprising a request-level
        # logits processor with the output token ids argument and (if required)
        # the prompt token ids argument pre-populated
        #
        # Note that the partial carries a *reference* to output token ids, and
        # will thus always operate on the list as it is currently, not as it
        # was when the partial was created.
        self.req_info: dict[int, partial[torch.Tensor]] = {}
```
**EN:** This method initializes the object state within `AdapterLogitsProcessor`. The docstring frames it as: Subclass must invoke `super().__init__(vllm_config, device, is_pin_memory)`. It touches state such as `req_info`.
**CN:** 该方法会初始化对象状态，其作用域位于`AdapterLogitsProcessor`。 它会读写 `req_info` 等状态。

### `AdapterLogitsProcessor.new_req_logits_processor` method / `AdapterLogitsProcessor.new_req_logits_processor` 方法
```python
    @abstractmethod
    def new_req_logits_processor(
        self,
        params: SamplingParams,
    ) -> RequestLogitsProcessor | None:
        """Consume request info; return a per-request logits processor.

        Return None if logits processor does not need to be applied to request

        Args:
          params: request sampling params

        Returns:
          None if logits processor should not be applied to request; otherwise
          returns a `RequestLogitsProcessor` instance

        """
        raise NotImplementedError
```
**EN:** This method implements `new_req_logits_processor` within `AdapterLogitsProcessor`. The docstring frames it as: Consume request info; return a per-request logits processor.
**CN:** 该方法会实现 `new_req_logits_processor`，其作用域位于`AdapterLogitsProcessor`。

### `AdapterLogitsProcessor._new_state` method / `AdapterLogitsProcessor._new_state` 方法
```python
    def _new_state(
        self,
        params: SamplingParams,
        prompt_ids: list[int] | None,
        output_ids: list[int],
    ) -> partial[torch.Tensor] | None:
        """Return state representation for new request

        Returns None if logits processor is not applicable to request

        Args:
          params: request sampling params
          prompt_ids: request prompt token ids
          output_ids: decoded tokens so far for this request

        Returns:
          logits processor partial[Tensor] or None

        """
        if req_lp := self.new_req_logits_processor(params):
            if len(inspect.signature(req_lp).parameters) == 3:
                if prompt_ids is None:
                    raise ValueError(
                        "Prompt token ids are required for this "
                        "logits processor but were not provided."
                    )
                args = [prompt_ids, output_ids]
            else:
                args = [output_ids]
            return partial(req_lp, *args)
        return None
```
**EN:** This method implements `_new_state` within `AdapterLogitsProcessor`. The docstring frames it as: Return state representation for new request Returns None if logits processor is not applicable to request Args: params: request sampling params prompt_ids: request prompt token ids output_ids: decoded tokens so far for this request Returns: logits processor partial[Tensor] or None Key calls include `new_req_logits_processor`, `partial`, `len`, `ValueError`, `signature`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_new_state`，其作用域位于`AdapterLogitsProcessor`。 关键调用包括 `new_req_logits_processor`, `partial`, `len`, `ValueError`, `signature`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `AdapterLogitsProcessor.update_state` method / `AdapterLogitsProcessor.update_state` 方法
```python
    def update_state(self, batch_update: BatchUpdate | None):
        process_dict_updates(
            self.req_info,
            batch_update,
            self._new_state,
        )
```
**EN:** This method updates existing state within `AdapterLogitsProcessor`. Key calls include `process_dict_updates`.
**CN:** 该方法会更新现有状态，其作用域位于`AdapterLogitsProcessor`。 关键调用包括 `process_dict_updates`。

### `AdapterLogitsProcessor.apply` method / `AdapterLogitsProcessor.apply` 方法
```python
    def apply(self, logits: torch.Tensor) -> torch.Tensor:
        if self.req_info:
            # Apply per-request logits processors to corresponding rows of
            # logits tensor
            for req_idx, req_lp in self.req_info.items():
                req_logits = logits[req_idx]
                new_logits = req_lp(req_logits)
                if new_logits is not req_logits:
                    # Modify logits tensor row in-place if necessary
                    logits[req_idx] = new_logits
        return logits
```
**EN:** This method implements `apply` within `AdapterLogitsProcessor`. Key calls include `items`, `req_lp`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `apply`，其作用域位于`AdapterLogitsProcessor`。 关键调用包括 `items`, `req_lp`。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### Module constants / 模块常量
```python
__all__ = [
    "LogitsProcessor",
    "LogitBiasLogitsProcessor",
    "MinPLogitsProcessor",
    "MinTokensLogitsProcessor",
    "BatchUpdate",
    "BatchUpdateBuilder",
    "MoveDirectionality",
    "LogitsProcessors",
    "build_logitsprocs",
    "STR_POOLING_REJECTS_LOGITSPROCS",
    "LOGITSPROCS_GROUP",
    "AdapterLogitsProcessor",
]
```
**EN:** Defines module-level constants or aliases such as `__all__`, which are reused by later definitions.
**CN:** 定义 `__all__` 等模块级常量或别名，供后续定义复用。

## Key Concepts / 关键概念
- `_load_logitsprocs_plugins`: top-level helper or orchestration entry point. / `_load_logitsprocs_plugins`：顶层辅助函数或编排入口。
- `_load_logitsprocs_by_fqcns`: top-level helper or orchestration entry point. / `_load_logitsprocs_by_fqcns`：顶层辅助函数或编排入口。
- `_load_custom_logitsprocs`: top-level helper or orchestration entry point. / `_load_custom_logitsprocs`：顶层辅助函数或编排入口。
- `build_logitsprocs`: top-level helper or orchestration entry point. / `build_logitsprocs`：顶层辅助函数或编排入口。
- `validate_logits_processors_parameters`: top-level helper or orchestration entry point. / `validate_logits_processors_parameters`：顶层辅助函数或编排入口。
- `AdapterLogitsProcessor`: central class or interface in this module. / `AdapterLogitsProcessor`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `importlib`, `inspect`, `itertools`, `abc`, `collections`, `functools`, `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.logger`, `vllm.logits_process`, `vllm.sampling_params`, `vllm.utils.torch_utils`, `vllm.v1.sample.logits_processor.builtin`, `vllm.v1.sample.logits_processor.interface`, `vllm.v1.sample.logits_processor.state`, `vllm.config`, `vllm.platforms`
