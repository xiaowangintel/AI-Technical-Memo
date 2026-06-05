# gpu_ubatch_wrapper.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu_ubatch_wrapper.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_cat_ubatch_outputs`, `UbatchMetadata`, `CUDAGraphMetaData` for the V1 `worker` subsystem. / 为 V1 的 `worker` 子系统实现 `_cat_ubatch_outputs`, `UbatchMetadata`, `CUDAGraphMetaData`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import threading
from collections.abc import Callable
from dataclasses import dataclass
from typing import Any

import torch

import vllm.envs as envs
from vllm.compilation.cuda_graph import CUDAGraphWrapper
from vllm.config import CUDAGraphMode, VllmConfig
from vllm.distributed import get_ep_group
from vllm.distributed.device_communicators.pynccl_allocator import set_graph_pool_id
from vllm.forward_context import (
    DPMetadata,
    create_forward_context,
    get_forward_context,
    override_forward_context,
)
from vllm.logger import init_logger
from vllm.model_executor.offloader.base import get_offloader
from vllm.platforms import current_platform
from vllm.sequence import IntermediateTensors
from vllm.utils.deep_gemm import set_num_sms as deep_gemm_set_num_sms
from vllm.utils.import_utils import has_deep_gemm
from vllm.utils.platform_utils import num_compute_units
from vllm.v1.worker.ubatching import UBatchContext, make_ubatch_contexts

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `_cat_ubatch_outputs` function / `_cat_ubatch_outputs` 函数
```python
def _cat_ubatch_outputs(
    sorted_results: list,
) -> "torch.Tensor | tuple[torch.Tensor, ...]":
    """Concatenate per-ubatch model outputs along the batch dim.

    Most models return a single hidden-states tensor per ubatch. Target
    models running with auxiliary output (e.g. EAGLE3 speculative decoding,
    which collects aux hidden states for the drafter) return a tuple of
    tensors instead. Fan out over tuple components so `torch.cat` sees
    matching shapes and the caller receives the same structure the model
    produced for a single ubatch (#40769).
    """
    if sorted_results and isinstance(sorted_results[0], tuple):
        return tuple(torch.cat(parts, dim=0) for parts in zip(*sorted_results))
    return torch.cat(sorted_results, dim=0)
```
**EN:** This function implements `_cat_ubatch_outputs` within the module. The docstring frames it as: Concatenate per-ubatch model outputs along the batch dim. Key calls include `cat`, `isinstance`, `tuple`, `zip`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_cat_ubatch_outputs`，其作用域位于the module。 关键调用包括 `cat`, `isinstance`, `tuple`, `zip`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `UbatchMetadata` class / `UbatchMetadata` 类
```python
@dataclass
class UbatchMetadata:
    context: UBatchContext
    input_ids: torch.Tensor
    positions: torch.Tensor
    inputs_embeds: torch.Tensor | None
    intermediate_tensors: IntermediateTensors | None
    num_tokens: int
```
**EN:** Uses `@dataclass` to package related state for `UbatchMetadata`. Typical fields include `context`, `input_ids`, `positions`, `inputs_embeds`, `intermediate_tensors`, `num_tokens`.
**CN:** `UbatchMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `context`, `input_ids`, `positions`, `inputs_embeds`, `intermediate_tensors`, `num_tokens`。

### `CUDAGraphMetaData` class / `CUDAGraphMetaData` 类
```python
@dataclass
class CUDAGraphMetaData:
    cudagraph: torch.cuda.CUDAGraph
    ubatch_metadata: UbatchMetadata
    outputs: Any | None = None
```
**EN:** Uses `@dataclass` to package related state for `CUDAGraphMetaData`. Typical fields include `cudagraph`, `ubatch_metadata`, `outputs`.
**CN:** `CUDAGraphMetaData` 使用 `@dataclass` 打包相关状态。典型字段包括 `cudagraph`, `ubatch_metadata`, `outputs`。

### `SMControlContextManager` class / `SMControlContextManager` 类
```python
class SMControlContextManager:
```
**EN:** Introduces the `SMControlContextManager` class. Core methods include `__init__`, `__enter__`, `__exit__`.
**CN:** 这里定义 `SMControlContextManager` 类。核心方法包括 `__init__`, `__enter__`, `__exit__`。

### `SMControlContextManager.__init__` method / `SMControlContextManager.__init__` 方法
```python
    def __init__(
        self,
        comm_sms: int,
        set_comm_sms: Callable[[int], None],
        set_compute_sms: Callable[[int], None],
    ):
        """
        Context manager for controlling SM (Streaming Multiprocessor)
        allocation. Upon entering the context, it sets the number of SMs
        allocated for communication and computation to comm_sms and
        total_sms - comm_sms respectively. Upon exiting, it restores the
        allocation to use all available SMs (i.e. total_sms).

        Args:
            comm_sms (int): The number of SMs to allocate for communication.
                (The remainder will be used for computation.)
            set_comm_sms (Callable[[int], None]):
                A function that sets the number of SMs for communication.
            set_compute_sms (Callable[[int], None]):
                A function that sets the number of SMs for computation.
        """

        assert current_platform.is_cuda() or current_platform.is_rocm(), (
            "SM/CU control is supported on CUDA and ROCm platforms"
        )
        device = torch.accelerator.current_device_index()
        total_sms = num_compute_units(device)

        assert comm_sms < total_sms
        self.total_sms = total_sms
        self.compute_sms = total_sms - comm_sms
        self.comm_sms = comm_sms
        self.set_comm_sms = set_comm_sms
        self.set_compute_sms = set_compute_sms
```
**EN:** This method initializes the object state within `SMControlContextManager`. The docstring frames it as: Context manager for controlling SM (Streaming Multiprocessor) allocation. Key calls include `current_device_index`, `num_compute_units`, `is_cuda`, `is_rocm`. It touches state such as `total_sms`, `compute_sms`, `comm_sms`, `set_comm_sms`, `set_compute_sms`.
**CN:** 该方法会初始化对象状态，其作用域位于`SMControlContextManager`。 关键调用包括 `current_device_index`, `num_compute_units`, `is_cuda`, `is_rocm`。 它会读写 `total_sms`, `compute_sms`, `comm_sms`, `set_comm_sms`, `set_compute_sms` 等状态。

### `SMControlContextManager.__enter__` method / `SMControlContextManager.__enter__` 方法
```python
    def __enter__(self):
        self.set_comm_sms(self.comm_sms)
        self.set_compute_sms(self.compute_sms)
```
**EN:** This method implements `__enter__` within `SMControlContextManager`. Key calls include `set_comm_sms`, `set_compute_sms`.
**CN:** 该方法会实现 `__enter__`，其作用域位于`SMControlContextManager`。 关键调用包括 `set_comm_sms`, `set_compute_sms`。

### `SMControlContextManager.__exit__` method / `SMControlContextManager.__exit__` 方法
```python
    def __exit__(self, exc_type, exc_value, traceback):
        self.set_comm_sms(self.total_sms)
        self.set_compute_sms(self.total_sms)
```
**EN:** This method implements `__exit__` within `SMControlContextManager`. Key calls include `set_comm_sms`, `set_compute_sms`.
**CN:** 该方法会实现 `__exit__`，其作用域位于`SMControlContextManager`。 关键调用包括 `set_comm_sms`, `set_compute_sms`。

### `UBatchWrapper` class / `UBatchWrapper` 类
```python
class UBatchWrapper:
```
**EN:** Introduces the `UBatchWrapper` class. Core methods include `__init__`, `graph_pool`, `clear_graphs`, `_create_sm_control_context`, `__getattr__`, `unwrap`.
**CN:** 这里定义 `UBatchWrapper` 类。核心方法包括 `__init__`, `graph_pool`, `clear_graphs`, `_create_sm_control_context`, `__getattr__`, `unwrap`。

### `UBatchWrapper.__init__` method / `UBatchWrapper.__init__` 方法
```python
    def __init__(
        self,
        runnable: Callable,
        vllm_config: VllmConfig,
        runtime_mode: CUDAGraphMode,
        device: torch.cuda.device,
    ):
        self.runnable = runnable
        self.vllm_config = vllm_config
        self.compilation_config = vllm_config.compilation_config
        self.comm_stream = torch.cuda.Stream(device=device)
        # Ubatch threads plus the main thread
        self.ready_barrier = threading.Barrier(
            self.vllm_config.parallel_config.num_ubatches + 1
        )

        self.cudagraphs: dict[int, CUDAGraphMetaData] = {}

        self.cudagraph_wrapper = None
        if runtime_mode is not CUDAGraphMode.NONE:
            self.cudagraph_wrapper = CUDAGraphWrapper(
                runnable, vllm_config, runtime_mode=runtime_mode
            )

        self.sm_control = self._create_sm_control_context(vllm_config)
        self.device = device
        self.is_debugging_mode = envs.VLLM_LOGGING_LEVEL == "DEBUG"
        self._runnable_str = str(runnable) if self.is_debugging_mode else None
```
**EN:** This method initializes the object state within `UBatchWrapper`. Key calls include `Stream`, `Barrier`, `_create_sm_control_context`, `CUDAGraphWrapper`, `str`. It touches state such as `runnable`, `vllm_config`, `compilation_config`, `comm_stream`, `ready_barrier`, `cudagraphs`, `cudagraph_wrapper`, `sm_control`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`UBatchWrapper`。 关键调用包括 `Stream`, `Barrier`, `_create_sm_control_context`, `CUDAGraphWrapper`, `str`。 它会读写 `runnable`, `vllm_config`, `compilation_config`, `comm_stream`, `ready_barrier`, `cudagraphs`, `cudagraph_wrapper`, `sm_control` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `UBatchWrapper._create_sm_control_context` method / `UBatchWrapper._create_sm_control_context` 方法
```python
    @staticmethod
    def _create_sm_control_context(vllm_config: VllmConfig):
        comm_sms: int = envs.VLLM_DBO_COMM_SMS

        set_comm_sms = lambda sms: None
        if vllm_config.parallel_config.enable_expert_parallel:
            # Currently only DeepEP highthroughput supports SM control so this
            # only affects that case.
            ep_group = get_ep_group()
            device_communicator = ep_group.device_communicator
            all2all_manager = None
            if device_communicator is not None:
                all2all_manager = device_communicator.all2all_manager

            if all2all_manager is not None:
                max_sms_used = all2all_manager.max_sms_used()
                if max_sms_used is not None:
                    comm_sms = min(comm_sms, max_sms_used)

            if comm_sms > 0 and all2all_manager is not None:
                set_comm_sms = lambda sms: all2all_manager.set_num_sms(sms)

        # TODO(lucas): support other kernels besides DeepGEMM
        set_compute_sms = lambda sms: None
        if has_deep_gemm() and comm_sms > 0:
            set_compute_sms = lambda sms: deep_gemm_set_num_sms(sms)

        return SMControlContextManager(
            comm_sms=comm_sms,
            set_comm_sms=set_comm_sms,
            set_compute_sms=set_compute_sms,
        )
```
**EN:** This method implements `_create_sm_control_context` within `UBatchWrapper`. Key calls include `SMControlContextManager`, `get_ep_group`, `has_deep_gemm`, `max_sms_used`, `deep_gemm_set_num_sms`, `min`. The control flow contains 6 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_create_sm_control_context`，其作用域位于`UBatchWrapper`。 关键调用包括 `SMControlContextManager`, `get_ep_group`, `has_deep_gemm`, `max_sms_used`, `deep_gemm_set_num_sms`, `min`。 控制流包含 6 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `UBatchWrapper._run_ubatches` method / `UBatchWrapper._run_ubatches` 方法
```python
    def _run_ubatches(self, ubatch_metadata, model) -> torch.Tensor:
        @torch.inference_mode()
        def _ubatch_thread(results, model, ubatch_metadata):
            with ubatch_metadata.context:
                model_output = model(
                    input_ids=ubatch_metadata.input_ids,
                    positions=ubatch_metadata.positions,
                    intermediate_tensors=ubatch_metadata.intermediate_tensors,
                    inputs_embeds=ubatch_metadata.inputs_embeds,
                )
            results.append((ubatch_metadata.context.id, model_output))

        results: list[tuple[int, torch.Tensor]] = []

        # Ubatch threads will manually manage the forward context, so we
        # override it to None here so we can have it restored correctly
        # after both threads have finished
        with override_forward_context(None):
            ubatch_threads = []
            for metadata in ubatch_metadata:
                thread = threading.Thread(
                    target=_ubatch_thread,
                    args=(
                        results,
                        model,
                        metadata,
                    ),
                )
                ubatch_threads.append(thread)
                thread.start()
            self.ready_barrier.wait()  # Wait for both threads to be ready
            ubatch_metadata[0].context.cpu_wait_event.set()
            for thread in ubatch_threads:
                thread.join()
        sorted_results = [value for position, value in sorted(results)]
        result = _cat_ubatch_outputs(sorted_results)
        return result
```
**EN:** This method implements `_run_ubatches` within `UBatchWrapper`. Key calls include `inference_mode`, `_cat_ubatch_outputs`, `append`, `override_forward_context`, `wait`, `set`. The control flow contains 0 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_run_ubatches`，其作用域位于`UBatchWrapper`。 关键调用包括 `inference_mode`, `_cat_ubatch_outputs`, `append`, `override_forward_context`, `wait`, `set`。 控制流包含 0 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `_cat_ubatch_outputs`: top-level helper or orchestration entry point. / `_cat_ubatch_outputs`：顶层辅助函数或编排入口。
- `UbatchMetadata`: central class or interface in this module. / `UbatchMetadata`：本模块中的核心类或接口。
- `CUDAGraphMetaData`: central class or interface in this module. / `CUDAGraphMetaData`：本模块中的核心类或接口。
- `SMControlContextManager`: central class or interface in this module. / `SMControlContextManager`：本模块中的核心类或接口。
- `UBatchWrapper`: central class or interface in this module. / `UBatchWrapper`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `threading`, `collections`, `dataclasses`, `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.envs`, `vllm.compilation.cuda_graph`, `vllm.config`, `vllm.distributed`, `vllm.distributed.device_communicators.pynccl_allocator`, `vllm.forward_context`, `vllm.logger`, `vllm.model_executor.offloader.base`, `vllm.platforms`, `vllm.sequence`, `vllm.utils.deep_gemm`, `vllm.utils.import_utils`
