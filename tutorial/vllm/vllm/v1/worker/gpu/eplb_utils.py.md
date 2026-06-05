# eplb_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/eplb_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_unwrap_moe`, `step_eplb_after`, `EPLBController` for the V1 `worker/gpu` subsystem. / 为 V1 的 `worker/gpu` 子系统实现 `_unwrap_moe`, `step_eplb_after`, `EPLBController`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
from collections.abc import Callable
from functools import wraps
from typing import Any

import torch
import torch.nn as nn

from vllm.distributed.eplb.eplb_state import EplbState
from vllm.logger import init_logger
from vllm.model_executor.models.interfaces import (
    SupportsMultiModal,
    is_mixture_of_experts,
)

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `_unwrap_moe` function / `_unwrap_moe` 函数
```python
def _unwrap_moe(model: nn.Module) -> nn.Module:
    # VLM wrappers (e.g. KimiK25ForConditionalGeneration) hold the MoE
    # language model under `.language_model` but don't implement
    # MixtureOfExperts themselves. Mirror the V1 path
    # (see vllm/v1/worker/gpu_model_runner.py, PR #39805).
    if not is_mixture_of_experts(model) and isinstance(model, SupportsMultiModal):
        return model.get_language_model()
    return model
```
**EN:** This function implements `_unwrap_moe` within the module. Key calls include `isinstance`, `get_language_model`, `is_mixture_of_experts`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_unwrap_moe`，其作用域位于the module。 关键调用包括 `isinstance`, `get_language_model`, `is_mixture_of_experts`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `step_eplb_after` function / `step_eplb_after` 函数
```python
def step_eplb_after(*, is_dummy: bool = False) -> Callable:
    """Step EPLB after a model runner method completes successfully."""

    def decorator(fn: Callable) -> Callable:
        @wraps(fn)
        def wrapper(self: Any, *args, **kwargs) -> Any:
            result = fn(self, *args, **kwargs)
            if kwargs.get("skip_eplb", False):
                return result

            is_profile = kwargs.get("is_profile", False) if is_dummy else False
            self.eplb.step(is_dummy=is_dummy, is_profile=is_profile)
            return result

        return wrapper

    return decorator
```
**EN:** This function advances one execution step within the module. The docstring frames it as: Step EPLB after a model runner method completes successfully. Key calls include `wraps`, `fn`, `get`, `step`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会推进一次执行步骤，其作用域位于the module。 关键调用包括 `wraps`, `fn`, `get`, `step`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `EPLBController` class / `EPLBController` 类
```python
class EPLBController:
```
**EN:** Introduces the `EPLBController` class. Core methods include `__init__`, `prepare_load`, `maybe_register_speculator`, `maybe_register_model`, `maybe_start_async_loop`, `step`.
**CN:** 这里定义 `EPLBController` 类。核心方法包括 `__init__`, `prepare_load`, `maybe_register_speculator`, `maybe_register_model`, `maybe_start_async_loop`, `step`。

### `EPLBController.__init__` method / `EPLBController.__init__` 方法
```python
    def __init__(self, parallel_config: Any, device: torch.device):
        self.parallel_config = parallel_config
        self.device = device
        self.state: EplbState | None = None
        self.suppressed = False
        self._has_registered_models = False
```
**EN:** This method initializes the object state within `EPLBController`. It touches state such as `parallel_config`, `device`, `state`, `suppressed`, `_has_registered_models`.
**CN:** 该方法会初始化对象状态，其作用域位于`EPLBController`。 它会读写 `parallel_config`, `device`, `state`, `suppressed`, `_has_registered_models` 等状态。

### `EPLBController.prepare_load` method / `EPLBController.prepare_load` 方法
```python
    def prepare_load(self) -> None:
        self.state = None
        self._has_registered_models = False
        if self.parallel_config.enable_eplb:
            self.state = EplbState(self.parallel_config, self.device)
```
**EN:** This method prepares inputs and state within `EPLBController`. Key calls include `EplbState`. It touches state such as `state`, `_has_registered_models`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会准备输入与状态，其作用域位于`EPLBController`。 关键调用包括 `EplbState`。 它会读写 `state`, `_has_registered_models` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `EPLBController.maybe_register_speculator` method / `EPLBController.maybe_register_speculator` 方法
```python
    def maybe_register_speculator(
        self,
        speculator: Any | None,
        speculative_config: Any | None,
        load_dummy_weights: bool,
    ) -> bool:
        # if speculator is a moe model, add it to eplb
        if (
            speculator is None
            or not hasattr(speculator, "model")
            or not self.parallel_config.enable_eplb
            or load_dummy_weights
        ):
            return False

        draft_model = speculator.model
        if not is_mixture_of_experts(draft_model):
            return False

        assert not self.parallel_config.enable_elastic_ep, (
            "Elastic EP is not supported with draft model."
        )
        assert speculative_config is not None
        assert speculative_config.draft_model_config is not None
        assert self.state is not None
        self.state.add_model(
            draft_model,
            speculative_config.draft_model_config,
        )
        self._has_registered_models = True
        return True
```
**EN:** This method applies logic conditionally within `EPLBController`. Key calls include `add_model`, `is_mixture_of_experts`, `hasattr`. It touches state such as `_has_registered_models`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会按条件执行逻辑，其作用域位于`EPLBController`。 关键调用包括 `add_model`, `is_mixture_of_experts`, `hasattr`。 它会读写 `_has_registered_models` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `EPLBController.maybe_register_model` method / `EPLBController.maybe_register_model` 方法
```python
    def maybe_register_model(
        self,
        model: nn.Module,
        model_config: Any,
        load_dummy_weights: bool,
    ) -> bool:
        if not self.parallel_config.enable_eplb or load_dummy_weights:
            return False

        model = _unwrap_moe(model)
        if not is_mixture_of_experts(model):
            return False

        logger.info_once("EPLB is enabled for model %s.", model_config.model)
        assert self.state is not None
        self.state.add_model(model, model_config)
        self._has_registered_models = True
        return True
```
**EN:** This method applies logic conditionally within `EPLBController`. Key calls include `_unwrap_moe`, `info_once`, `add_model`, `is_mixture_of_experts`. It touches state such as `_has_registered_models`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会按条件执行逻辑，其作用域位于`EPLBController`。 关键调用包括 `_unwrap_moe`, `info_once`, `add_model`, `is_mixture_of_experts`。 它会读写 `_has_registered_models` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `EPLBController.maybe_start_async_loop` method / `EPLBController.maybe_start_async_loop` 方法
```python
    def maybe_start_async_loop(self, eplb_models_added: bool) -> None:
        if eplb_models_added and self.state is not None and self.state.is_async:
            self.state.start_async_loop()
```
**EN:** This method applies logic conditionally within `EPLBController`. Key calls include `start_async_loop`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会按条件执行逻辑，其作用域位于`EPLBController`。 关键调用包括 `start_async_loop`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `EPLBController.step` method / `EPLBController.step` 方法
```python
    def step(
        self,
        is_dummy: bool = False,
        is_profile: bool = False,
    ) -> None:
        if (
            not self.parallel_config.enable_eplb
            or self.suppressed
            or self.state is None
            or not self._has_registered_models
        ):
            return

        self.state.step(
            is_dummy,
            is_profile,
            log_stats=self.parallel_config.eplb_config.log_balancedness,
        )
```
**EN:** This method advances one execution step within `EPLBController`. Key calls include `step`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会推进一次执行步骤，其作用域位于`EPLBController`。 关键调用包括 `step`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `EPLBController.setup_from_mapping` method / `EPLBController.setup_from_mapping` 方法
```python
    def setup_from_mapping(
        self,
        model: nn.Module,
        model_config: Any,
        expanded_physical_to_logical: torch.Tensor,
        old_num_physical_experts: int,
    ) -> None:
        model = _unwrap_moe(model)
        assert is_mixture_of_experts(model)

        self.state = EplbState.from_mapping(
            model=model,
            model_config=model_config,
            device=self.device,
            parallel_config=self.parallel_config,
            expanded_physical_to_logical=expanded_physical_to_logical,
            num_valid_physical_experts=old_num_physical_experts,
        )
        self._has_registered_models = True
```
**EN:** This method implements `setup_from_mapping` within `EPLBController`. Key calls include `_unwrap_moe`, `is_mixture_of_experts`, `from_mapping`. It touches state such as `state`, `_has_registered_models`.
**CN:** 该方法会实现 `setup_from_mapping`，其作用域位于`EPLBController`。 关键调用包括 `_unwrap_moe`, `is_mixture_of_experts`, `from_mapping`。 它会读写 `state`, `_has_registered_models` 等状态。

## Key Concepts / 关键概念
- `_unwrap_moe`: top-level helper or orchestration entry point. / `_unwrap_moe`：顶层辅助函数或编排入口。
- `step_eplb_after`: top-level helper or orchestration entry point. / `step_eplb_after`：顶层辅助函数或编排入口。
- `EPLBController`: central class or interface in this module. / `EPLBController`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `collections`, `functools`, `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.distributed.eplb.eplb_state`, `vllm.logger`, `vllm.model_executor.models.interfaces`
