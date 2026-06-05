# cpu_model_runner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/cpu_model_runner.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `CPUModelRunner`, `_torch_cuda_wrapper`, `_set_global_compilation_settings` for the V1 `worker` subsystem. / 为 V1 的 `worker` 子系统实现 `CPUModelRunner`, `_torch_cuda_wrapper`, `_set_global_compilation_settings`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
from contextlib import contextmanager
from typing import Any

import torch
import torch.nn as nn

import vllm.utils.cpu_triton_utils as cpu_tl
from vllm.config import VllmConfig
from vllm.logger import init_logger
from vllm.model_executor.model_loader import get_model
from vllm.tracing import instrument
from vllm.v1.core.sched.output import SchedulerOutput
from vllm.v1.kv_cache_interface import KVCacheConfig
from vllm.v1.utils import CpuGpuBuffer
from vllm.v1.worker.gpu_model_runner import GPUModelRunner

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `CPUModelRunner` class / `CPUModelRunner` 类
```python
class CPUModelRunner(GPUModelRunner):
```
**EN:** Introduces the `CPUModelRunner` class on top of `GPUModelRunner`. Core methods include `__init__`, `_postprocess_tensors`, `_postprocess_triton`, `load_model`, `get_model`, `warming_up_model`.
**CN:** 这里定义 `CPUModelRunner` 类，其基类包括 `GPUModelRunner`。核心方法包括 `__init__`, `_postprocess_tensors`, `_postprocess_triton`, `load_model`, `get_model`, `warming_up_model`。

### `CPUModelRunner.__init__` method / `CPUModelRunner.__init__` 方法
```python
    def __init__(self, vllm_config: VllmConfig, device: torch.device):
        # avoid calling accelerator APIs for methods inherited from super class
        _set_torch_accelerator_to_noop()

        with _torch_cuda_wrapper():
            super().__init__(vllm_config, device)

        assert device == torch.device("cpu")
        # Note: speculative decoding is now supported on CPU with C++ native impls

        self.use_cuda_graph = False
        self.cascade_attn_enabled = False

        self._postprocess_tensors()
        self._postprocess_triton()
```
**EN:** This method initializes the object state within `CPUModelRunner`. Key calls include `_set_torch_accelerator_to_noop`, `_postprocess_tensors`, `_postprocess_triton`, `_torch_cuda_wrapper`, `__init__`, `device`. It touches state such as `use_cuda_graph`, `cascade_attn_enabled`.
**CN:** 该方法会初始化对象状态，其作用域位于`CPUModelRunner`。 关键调用包括 `_set_torch_accelerator_to_noop`, `_postprocess_tensors`, `_postprocess_triton`, `_torch_cuda_wrapper`, `__init__`, `device`。 它会读写 `use_cuda_graph`, `cascade_attn_enabled` 等状态。

### `CPUModelRunner._postprocess_tensors` method / `CPUModelRunner._postprocess_tensors` 方法
```python
    def _postprocess_tensors(self) -> None:
        # Note: replace device tensors with cpu tensors
        def replace_tensor(obj: Any, cpu_attr_name: str, device_attr_name) -> None:
            cpu_tensor = getattr(obj, cpu_attr_name, None)
            device_tensor = getattr(obj, device_attr_name, None)
            if isinstance(cpu_tensor, torch.Tensor) and isinstance(
                device_tensor, torch.Tensor
            ):
                setattr(obj, device_attr_name, cpu_tensor)

        for v in vars(self).values():
            if isinstance(v, CpuGpuBuffer):
                v.gpu = v.cpu

        for k, v in vars(self.input_batch).items():
            if k.endswith("_cpu_tensor") and isinstance(v, torch.Tensor):
                replace_tensor(self.input_batch, k, k[:-11])

        for block_table in self.input_batch.block_table.block_tables:
            for v in vars(block_table).values():
                if isinstance(v, CpuGpuBuffer):
                    v.gpu = v.cpu
```
**EN:** This method implements `_postprocess_tensors` within `CPUModelRunner`. Key calls include `values`, `items`, `getattr`, `isinstance`, `setattr`, `vars`. The control flow contains 4 branch(es) and 4 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_postprocess_tensors`，其作用域位于`CPUModelRunner`。 关键调用包括 `values`, `items`, `getattr`, `isinstance`, `setattr`, `vars`。 控制流包含 4 个分支和 4 个循环，说明这里承担了较强的协调逻辑。

### `CPUModelRunner._postprocess_triton` method / `CPUModelRunner._postprocess_triton` 方法
```python
    def _postprocess_triton(self) -> None:
        import vllm.v1.worker.block_table

        vllm.v1.worker.block_table._compute_slot_mapping_kernel = (
            cpu_tl.compute_slot_mapping_kernel
        )

        # Speculative decoding fallbacks
        import vllm.v1.sample.rejection_sampler
        import vllm.v1.spec_decode.llm_base_proposer
        import vllm.v1.spec_decode.utils

        vllm.v1.spec_decode.llm_base_proposer.eagle_prepare_inputs_padded_kernel = (
            cpu_tl.eagle_prepare_inputs_padded_kernel
        )
        vllm.v1.spec_decode.llm_base_proposer.eagle_prepare_next_token_padded_kernel = (
            cpu_tl.eagle_prepare_next_token_padded_kernel
        )
        vllm.v1.spec_decode.llm_base_proposer.copy_and_expand_eagle_inputs_kernel = (
            cpu_tl.copy_and_expand_eagle_inputs_kernel
        )
        vllm.v1.spec_decode.utils.eagle_step_slot_mapping_metadata_kernel = (
            cpu_tl.eagle_step_slot_mapping_metadata_kernel
        )
        vllm.v1.sample.rejection_sampler.rejection_greedy_sample_kernel = (
            cpu_tl.rejection_greedy_sample_kernel
        )
        vllm.v1.sample.rejection_sampler.rejection_random_sample_kernel = (
            cpu_tl.rejection_random_sample_kernel
        )
        vllm.v1.sample.rejection_sampler.expand_kernel = cpu_tl.expand_kernel
        vllm.v1.sample.rejection_sampler.sample_recovered_tokens_kernel = (
            cpu_tl.sample_recovered_tokens_kernel
        )
```
**EN:** This method implements `_postprocess_triton` within `CPUModelRunner`.
**CN:** 该方法会实现 `_postprocess_triton`，其作用域位于`CPUModelRunner`。

### `CPUModelRunner.load_model` method / `CPUModelRunner.load_model` 方法
```python
    @instrument(span_name="Loading (CPU)")
    def load_model(self, load_dummy_weights: bool = False) -> None:
        if load_dummy_weights:
            raise ValueError(
                "Loading dummy weights (needed for elastic EP scale-up) "
                "Is not supported by the CPU Model Runner."
            )
        logger.info("Starting to load model %s...", self.model_config.model)
        self.model = get_model(vllm_config=self.vllm_config)

        if self.lora_config:
            self.model = self.load_lora_model(self.model, self.vllm_config, self.device)

        if hasattr(self, "drafter"):
            logger.info_once("Loading drafter model...")
            self.drafter.load_model(self.model)
```
**EN:** This method loads external or cached state within `CPUModelRunner`. Key calls include `instrument`, `info`, `get_model`, `hasattr`, `ValueError`, `load_lora_model`. It touches state such as `model`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会加载外部或缓存状态，其作用域位于`CPUModelRunner`。 关键调用包括 `instrument`, `info`, `get_model`, `hasattr`, `ValueError`, `load_lora_model`。 它会读写 `model` 等状态。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `CPUModelRunner.warming_up_model` method / `CPUModelRunner.warming_up_model` 方法
```python
    @instrument(span_name="Warmup (CPU)")
    def warming_up_model(self) -> None:
        logger.info("Warming up model for the compilation...")
        # Only generate graph for the generic shape
        with _set_global_compilation_settings(self.vllm_config):
            self.profile_run()
        logger.info("Warming up done.")
```
**EN:** This method implements `warming_up_model` within `CPUModelRunner`. Key calls include `instrument`, `info`, `_set_global_compilation_settings`, `profile_run`.
**CN:** 该方法会实现 `warming_up_model`，其作用域位于`CPUModelRunner`。 关键调用包括 `instrument`, `info`, `_set_global_compilation_settings`, `profile_run`。

### `CPUModelRunner.initialize_kv_cache` method / `CPUModelRunner.initialize_kv_cache` 方法
```python
    def initialize_kv_cache(
        self,
        kv_cache_config: KVCacheConfig,
        is_profiling: bool = False,
    ) -> None:
        super().initialize_kv_cache(kv_cache_config, is_profiling)

        if self.speculative_config:
            if self.speculative_config.use_eagle():
                logger.info("EAGLE drafter KV cache initialized for CPU backend")
            elif self.speculative_config.uses_draft_model():
                logger.info("Draft model KV cache initialized for CPU backend")
```
**EN:** This method implements `initialize_kv_cache` within `CPUModelRunner`. Key calls include `initialize_kv_cache`, `use_eagle`, `super`, `info`, `uses_draft_model`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `initialize_kv_cache`，其作用域位于`CPUModelRunner`。 关键调用包括 `initialize_kv_cache`, `use_eagle`, `super`, `info`, `uses_draft_model`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `CPUModelRunner._copy_valid_sampled_token_count` method / `CPUModelRunner._copy_valid_sampled_token_count` 方法
```python
    def _copy_valid_sampled_token_count(
        self, next_token_ids: torch.Tensor, valid_sampled_tokens_count: torch.Tensor
    ) -> None:
        """CPU-safe version: direct copy without CUDA streams."""
        if self.valid_sampled_token_count_cpu is None:
            return

        counts = valid_sampled_tokens_count
        counts_cpu = self.valid_sampled_token_count_cpu
        counts_cpu[: counts.shape[0]].copy_(counts)
        self.input_batch.prev_sampled_token_ids = next_token_ids.unsqueeze(1)
```
**EN:** This method implements `_copy_valid_sampled_token_count` within `CPUModelRunner`. The docstring frames it as: CPU-safe version: direct copy without CUDA streams. Key calls include `copy_`, `unsqueeze`. It touches state such as `input_batch`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_copy_valid_sampled_token_count`，其作用域位于`CPUModelRunner`。 关键调用包括 `copy_`, `unsqueeze`。 它会读写 `input_batch` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `CPUModelRunner._get_valid_sampled_token_count` method / `CPUModelRunner._get_valid_sampled_token_count` 方法
```python
    def _get_valid_sampled_token_count(self) -> list[int]:
        """CPU-safe version: no event synchronization needed."""
        prev_sampled_token_ids = self.input_batch.prev_sampled_token_ids
        if prev_sampled_token_ids is None:
            return []

        counts_cpu = self.valid_sampled_token_count_cpu
        if counts_cpu is None:
            return []
        return counts_cpu[: prev_sampled_token_ids.shape[0]].tolist()
```
**EN:** This method implements `_get_valid_sampled_token_count` within `CPUModelRunner`. The docstring frames it as: CPU-safe version: no event synchronization needed. Key calls include `tolist`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_get_valid_sampled_token_count`，其作用域位于`CPUModelRunner`。 关键调用包括 `tolist`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_torch_cuda_wrapper` function / `_torch_cuda_wrapper` 函数
```python
@contextmanager
def _torch_cuda_wrapper():
    class _EventPlaceholder:
        def __init__(self, *args, **kwargs) -> None:
            self.record = lambda: None
            self.synchronize = lambda: None

    class _StreamPlaceholder:
        def __init__(self, *args, **kwargs) -> None:
            pass

    cuda_event = torch.Event
    cuda_stream = torch.cuda.Stream
    try:
        torch.Event = _EventPlaceholder
        torch.cuda.Stream = _StreamPlaceholder
        yield
    finally:
        torch.Event = cuda_event
        torch.cuda.Stream = cuda_stream
```
**EN:** This function implements `_torch_cuda_wrapper` within the module. It touches state such as `record`, `synchronize`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_torch_cuda_wrapper`，其作用域位于the module。 它会读写 `record`, `synchronize` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_set_global_compilation_settings` function / `_set_global_compilation_settings` 函数
```python
@contextmanager
def _set_global_compilation_settings(config: VllmConfig):
    import torch._inductor.config as torch_inductor_config

    inductor_config = config.compilation_config.inductor_compile_config
    # Note: The MKLDNN and CPPGEMM backend requires freezing parameters.
    freezing_value = torch_inductor_config.freezing
    try:
        if inductor_config.get("max_autotune", False):
            torch_inductor_config.freezing = True
        yield
    finally:
        torch_inductor_config.freezing = freezing_value
```
**EN:** This function implements `_set_global_compilation_settings` within the module. Key calls include `get`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_set_global_compilation_settings`，其作用域位于the module。 关键调用包括 `get`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_set_torch_accelerator_to_noop` function / `_set_torch_accelerator_to_noop` 函数
```python
def _set_torch_accelerator_to_noop() -> None:
    def noop(*args: Any, **kwargs: Any) -> None:
        pass

    torch.accelerator.synchronize = noop
    torch.accelerator.empty_cache = noop
```
**EN:** This function implements `_set_torch_accelerator_to_noop` within the module.
**CN:** 该函数会实现 `_set_torch_accelerator_to_noop`，其作用域位于the module。

## Key Concepts / 关键概念
- `CPUModelRunner`: central class or interface in this module. / `CPUModelRunner`：本模块中的核心类或接口。
- `_torch_cuda_wrapper`: top-level helper or orchestration entry point. / `_torch_cuda_wrapper`：顶层辅助函数或编排入口。
- `_set_global_compilation_settings`: top-level helper or orchestration entry point. / `_set_global_compilation_settings`：顶层辅助函数或编排入口。
- `_set_torch_accelerator_to_noop`: top-level helper or orchestration entry point. / `_set_torch_accelerator_to_noop`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `contextlib`, `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.utils.cpu_triton_utils`, `vllm.config`, `vllm.logger`, `vllm.model_executor.model_loader`, `vllm.tracing`, `vllm.v1.core.sched.output`, `vllm.v1.kv_cache_interface`, `vllm.v1.utils`, `vllm.v1.worker.gpu_model_runner`, `vllm.v1.worker.block_table`, `vllm.v1.sample.rejection_sampler`, `vllm.v1.spec_decode.llm_base_proposer`
