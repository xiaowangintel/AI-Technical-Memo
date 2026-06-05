# offloader.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/offloader.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable SGLang runtime helpers for offloading and memory movement helpers. / 为 SGLang 运行时提供面向卸载与内存迁移辅助逻辑的可复用辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Module setup and shared state / 模块设置与共享状态
```python
import logging
import os
from abc import ABC
from typing import Callable, Generator, List, Optional

import torch
from torch.func import functional_call

from sglang.srt.distributed.naive_distributed import (
    NaiveDistributed,
    get_naive_distributed,
    set_naive_distributed,
)
from sglang.srt.layers.parameter import ModelWeightParameter
from sglang.srt.server_args import ServerArgs
from sglang.srt.utils import MultiprocessingSerializer, is_pin_memory_available
from sglang.srt.utils.host_shared_memory import (
    HostSharedMemoryManager,
    get_host_shared_memory_manager,
    set_host_shared_memory_manager,
)

logger = logging.getLogger(__name__)

_SubmoduleAccessor = Callable[[torch.nn.Module], torch.nn.Module]
_WhitelistParamNamesCreator = Callable[[torch.nn.Module], List[str]]
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `logging`, `os`, `abc`, `typing`, `torch`, `torch.func`. It also defines symbols such as `logger`, `_SubmoduleAccessor`, `_WhitelistParamNamesCreator` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `logging`, `os`, `abc`, `typing`, `torch`, `torch.func`。 同时定义了 `logger`, `_SubmoduleAccessor`, `_WhitelistParamNamesCreator` 等符号，供后续逻辑使用。

### Lines 29-29: Class `BaseOffloader` declaration / 类 `BaseOffloader` 声明
```python
class BaseOffloader(ABC):
```
**EN:** This class establishes `BaseOffloader` as the main container/coordinator for the surrounding logic. It inherits from `ABC`. Its core interface includes methods such as `wrap_modules`, `post_init`, `forbid_copy_engine_usage`.
**CN:** 该类将 `BaseOffloader` 定义为周边逻辑的主要封装体或协调者。 它继承自 `ABC`。 其核心接口包括 `wrap_modules`, `post_init`, `forbid_copy_engine_usage` 等方法。

### Lines 30-36: Method `BaseOffloader.wrap_modules` / 方法 `BaseOffloader.wrap_modules`
```python
    def wrap_modules(
        self,
        all_modules_generator: Generator[torch.nn.Module, None, None],
        submodule_accessor: Optional[_SubmoduleAccessor] = None,
        whitelist_param_names_creator: Optional[_WhitelistParamNamesCreator] = None,
    ):
        return list(all_modules_generator)
```
**EN:** This method implements `wrap_modules` on `BaseOffloader`. It primarily calls `list` to complete its work.
**CN:** 该方法（属于 `BaseOffloader`）实现了 `wrap_modules`。 它主要通过调用 `list` 来完成任务。

### Lines 38-39: Method `BaseOffloader.post_init` / 方法 `BaseOffloader.post_init`
```python
    def post_init(self):
        pass
```
**EN:** This method implements `post_init` on `BaseOffloader`.
**CN:** 该方法（属于 `BaseOffloader`）实现了 `post_init`。

### Lines 41-43: Method `BaseOffloader.forbid_copy_engine_usage` / 方法 `BaseOffloader.forbid_copy_engine_usage`
```python
    @property
    def forbid_copy_engine_usage(self):
        return False
```
**EN:** This method implements `forbid_copy_engine_usage` on `BaseOffloader`.
**CN:** 该方法（属于 `BaseOffloader`）实现了 `forbid_copy_engine_usage`。

### Lines 46-47: Class `NoopOffloader` declaration / 类 `NoopOffloader` 声明
```python
class NoopOffloader(BaseOffloader):
    pass
```
**EN:** This class establishes `NoopOffloader` as the main container/coordinator for the surrounding logic. It inherits from `BaseOffloader`.
**CN:** 该类将 `NoopOffloader` 定义为周边逻辑的主要封装体或协调者。 它继承自 `BaseOffloader`。

### Lines 51-51: Constants and shared state / 常量与共享状态
```python
_instance: Optional[BaseOffloader] = NoopOffloader()
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `_instance`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `_instance`。

### Lines 54-56: Function `get_offloader` / 函数 `get_offloader`
```python
def get_offloader():
    assert _instance is not None
    return _instance
```
**EN:** This function implements `get_offloader`.
**CN:** 该函数实现了 `get_offloader`。

### Lines 59-61: Function `set_offloader` / 函数 `set_offloader`
```python
def set_offloader(instance: BaseOffloader):
    global _instance
    _instance = instance
```
**EN:** This function implements `set_offloader`. State updates are written into `_instance`.
**CN:** 该函数实现了 `set_offloader`。 状态更新主要写入 `_instance`。

### Lines 64-81: Function `create_offloader_from_server_args` / 函数 `create_offloader_from_server_args`
```python
def create_offloader_from_server_args(server_args: ServerArgs, dp_rank: int):
    if server_args.cpu_offload_gb > 0:
        return OffloaderV1(
            cpu_offload_max_bytes=int(server_args.cpu_offload_gb * 1024**3)
        )
    if server_args.offload_group_size > 0:
        assert (
            server_args.cpu_offload_gb == 0
        ), "V2 offload does not support cpu_offload_gb yet"
        return OffloaderV2(
            group_size=server_args.offload_group_size,
            num_in_group=server_args.offload_num_in_group,
            prefetch_step=server_args.offload_prefetch_step,
            mode=server_args.offload_mode,
            dp_rank=dp_rank,
            dp_size=server_args.dp_size,
        )
    return NoopOffloader()
```
**EN:** This function implements `create_offloader_from_server_args`. It primarily calls `NoopOffloader`, `OffloaderV1`, `OffloaderV2`, `int` to complete its work. The implementation relies on conditional branches.
**CN:** 该函数实现了 `create_offloader_from_server_args`。 它主要通过调用 `NoopOffloader`, `OffloaderV1`, `OffloaderV2`, `int` 来完成任务。 实现中使用了条件分支。

### Lines 84-84: Class `OffloaderV1` declaration / 类 `OffloaderV1` 声明
```python
class OffloaderV1(BaseOffloader):
```
**EN:** This class establishes `OffloaderV1` as the main container/coordinator for the surrounding logic. It inherits from `BaseOffloader`. Its core interface includes methods such as `__init__`, `wrap_modules`, `maybe_offload_to_cpu`.
**CN:** 该类将 `OffloaderV1` 定义为周边逻辑的主要封装体或协调者。 它继承自 `BaseOffloader`。 其核心接口包括 `__init__`, `wrap_modules`, `maybe_offload_to_cpu` 等方法。

### Lines 85-87: Method `OffloaderV1.__init__` / 方法 `OffloaderV1.__init__`
```python
    def __init__(self, cpu_offload_max_bytes: int):
        self._cpu_offload_bytes = 0
        self._cpu_offload_max_bytes = cpu_offload_max_bytes
```
**EN:** This method implements `__init__` on `OffloaderV1`. State updates are written into `self._cpu_offload_bytes`, `self._cpu_offload_max_bytes`.
**CN:** 该方法（属于 `OffloaderV1`）实现了 `__init__`。 状态更新主要写入 `self._cpu_offload_bytes`, `self._cpu_offload_max_bytes`。

### Lines 89-95: Method `OffloaderV1.wrap_modules` / 方法 `OffloaderV1.wrap_modules`
```python
    def wrap_modules(
        self,
        all_modules_generator: Generator[torch.nn.Module, None, None],
        submodule_accessor: Optional[_SubmoduleAccessor] = None,
        whitelist_param_names_creator: Optional[_WhitelistParamNamesCreator] = None,
    ):
        return [self.maybe_offload_to_cpu(module) for module in all_modules_generator]
```
**EN:** This method implements `wrap_modules` on `OffloaderV1`. It primarily calls `self.maybe_offload_to_cpu` to complete its work.
**CN:** 该方法（属于 `OffloaderV1`）实现了 `wrap_modules`。 它主要通过调用 `self.maybe_offload_to_cpu` 来完成任务。

### Lines 97-150: Method `OffloaderV1.maybe_offload_to_cpu` / 方法 `OffloaderV1.maybe_offload_to_cpu`
```python
    def maybe_offload_to_cpu(self, module: torch.nn.Module) -> torch.nn.Module:
        if (params := next(module.parameters(), None)) is None:
            return module

        device = params.device

        if device == torch.device("cpu"):
            return module

        if self._cpu_offload_bytes >= self._cpu_offload_max_bytes:
            return module

        pin_memory = is_pin_memory_available()
        # offload parameters to CPU
        # use pin_memory if possible, which helps cudagraph capture speed
        offloaded_parameters = False
        for p in module.parameters():
            if self._cpu_offload_bytes >= self._cpu_offload_max_bytes:
                # we use per-parameter offloading
                # one module might have some parameters offloaded and some not
                break

            # `torch.empty_like` does not support `pin_memory` argument
            cpu_data = torch.empty_strided(
                size=p.data.size(),
                stride=p.data.stride(),
                dtype=p.data.dtype,
                layout=p.data.layout,
# ... omitted for brevity ...
                module.forward = forward
                return output

            module.forward = forward

        return module
```
**EN:** This method implements `maybe_offload_to_cpu` on `OffloaderV1`. It primarily calls `is_pin_memory_available`, `module.parameters`, `torch.device`, `torch.empty_strided`, `cpu_data.copy_`, `next` to complete its work. State updates are written into `device`, `pin_memory`, `offloaded_parameters`, `cpu_data`, `p.data`, `self._cpu_offload_bytes`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `OffloaderV1`）实现了 `maybe_offload_to_cpu`。 它主要通过调用 `is_pin_memory_available`, `module.parameters`, `torch.device`, `torch.empty_strided`, `cpu_data.copy_`, `next` 来完成任务。 状态更新主要写入 `device`, `pin_memory`, `offloaded_parameters`, `cpu_data`, `p.data`, `self._cpu_offload_bytes`。 实现中使用了条件分支、迭代逻辑。

### Lines 153-153: Class `OffloaderV2` declaration / 类 `OffloaderV2` 声明
```python
class OffloaderV2(BaseOffloader):
```
**EN:** This class establishes `OffloaderV2` as the main container/coordinator for the surrounding logic. It inherits from `BaseOffloader`. Its core interface includes methods such as `__init__`, `wrap_modules`, `post_init`, `forbid_copy_engine_usage`.
**CN:** 该类将 `OffloaderV2` 定义为周边逻辑的主要封装体或协调者。 它继承自 `BaseOffloader`。 其核心接口包括 `__init__`, `wrap_modules`, `post_init`, `forbid_copy_engine_usage` 等方法。

### Lines 154-191: Method `OffloaderV2.__init__` / 方法 `OffloaderV2.__init__`
```python
    def __init__(
        self,
        group_size: int,
        num_in_group: int,
        prefetch_step: int,
        mode: str,
        dp_rank: int,
        dp_size: int,
    ):
        self.group_size = group_size
        self.num_in_group = num_in_group
        self.prefetch_step = prefetch_step
        self.mode = mode

        run_id = os.environ["SGLANG_RUN_ID"]

        # Temporarily init inside Offloader, can move if other modules also need this
        if self.mode in {"sharded_gpu", "shm_cpu"}:
            from sglang.srt.distributed import get_tensor_model_parallel_world_size

            assert (
                get_tensor_model_parallel_world_size() == 1
            ), "not yet support tp_size!=1"
            set_naive_distributed(
                NaiveDistributed(
                    rank=dp_rank,
                    world_size=dp_size,
                    rendezvous=f"/tmp/{run_id}",
# ... omitted for brevity ...
                HostSharedMemoryManager(
                    base_name=run_id,
                )
            )

        self.offloaders = []
```
**EN:** This method implements `__init__` on `OffloaderV2`. It primarily calls `set_naive_distributed`, `set_host_shared_memory_manager`, `get_tensor_model_parallel_world_size`, `NaiveDistributed`, `HostSharedMemoryManager` to complete its work. State updates are written into `self.group_size`, `self.num_in_group`, `self.prefetch_step`, `self.mode`, `run_id`, `self.offloaders`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `OffloaderV2`）实现了 `__init__`。 它主要通过调用 `set_naive_distributed`, `set_host_shared_memory_manager`, `get_tensor_model_parallel_world_size`, `NaiveDistributed`, `HostSharedMemoryManager` 来完成任务。 状态更新主要写入 `self.group_size`, `self.num_in_group`, `self.prefetch_step`, `self.mode`, `run_id`, `self.offloaders`。 实现中使用了条件分支。

### Lines 193-231: Method `OffloaderV2.wrap_modules` / 方法 `OffloaderV2.wrap_modules`
```python
    def wrap_modules(
        self,
        all_modules_generator: Generator[torch.nn.Module, None, None],
        submodule_accessor: Optional[_SubmoduleAccessor] = None,
        whitelist_param_names_creator: Optional[_WhitelistParamNamesCreator] = None,
    ):
        assert len(self.offloaders) == 0, "should only call wrap_modules once"

        alt_stream = torch.cuda.Stream()

        all_modules = []
        offload_submodules = []
        for module_index, module in enumerate(all_modules_generator):
            all_modules.append(module)
            if module_index % self.group_size >= self.group_size - self.num_in_group:
                submodule = submodule_accessor(module)
                whitelist_param_names = whitelist_param_names_creator(submodule)
                logger.info(
                    f"[offloader] offload {module_index=} submodule={type(submodule)} params={whitelist_param_names} memory_allocated={torch.cuda.memory_allocated()}"
                )
                offload_submodules.append(submodule)
                self.offloaders.append(
                    _ModuleOffloader(
                        mode=self.mode,
                        module=submodule,
                        alt_stream=alt_stream,
                        whitelist_param_names=whitelist_param_names,
                    )
# ... omitted for brevity ...
                module=module,
                offloaders=self.offloaders,
                prefetch_step=self.prefetch_step,
            )

        return all_modules
```
**EN:** This method implements `wrap_modules` on `OffloaderV2`. It primarily calls `torch.cuda.Stream`, `enumerate`, `len`, `all_modules.append`, `_hook_module_forward_for_offloader`, `submodule_accessor` to complete its work. State updates are written into `alt_stream`, `all_modules`, `offload_submodules`, `submodule`, `whitelist_param_names`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `OffloaderV2`）实现了 `wrap_modules`。 它主要通过调用 `torch.cuda.Stream`, `enumerate`, `len`, `all_modules.append`, `_hook_module_forward_for_offloader`, `submodule_accessor` 来完成任务。 状态更新主要写入 `alt_stream`, `all_modules`, `offload_submodules`, `submodule`, `whitelist_param_names`。 实现中使用了条件分支、迭代逻辑。

### Lines 233-238: Method `OffloaderV2.post_init` / 方法 `OffloaderV2.post_init`
```python
    def post_init(self):
        for offloader in self.offloaders:
            offloader.post_init()

        for i in range(self.prefetch_step):
            self.offloaders[i].start_onload()
```
**EN:** This method implements `post_init` on `OffloaderV2`. It primarily calls `range`, `offloader.post_init`, `self.offloaders.start_onload` to complete its work. The implementation relies on iteration.
**CN:** 该方法（属于 `OffloaderV2`）实现了 `post_init`。 它主要通过调用 `range`, `offloader.post_init`, `self.offloaders.start_onload` 来完成任务。 实现中使用了迭代逻辑。

### Lines 240-242: Method `OffloaderV2.forbid_copy_engine_usage` / 方法 `OffloaderV2.forbid_copy_engine_usage`
```python
    @property
    def forbid_copy_engine_usage(self):
        return self.mode == "cpu"
```
**EN:** This method implements `forbid_copy_engine_usage` on `OffloaderV2`.
**CN:** 该方法（属于 `OffloaderV2`）实现了 `forbid_copy_engine_usage`。

### Lines 245-256: Function `_hook_module_forward_for_offloader` / 函数 `_hook_module_forward_for_offloader`
```python
def _hook_module_forward_for_offloader(index, module, offloaders, prefetch_step):
    def _on_forward_end():
        offloaders[(index + prefetch_step) % len(offloaders)].start_onload()
        offloaders[index].offload()

    _hook_module_forward_raw(
        module,
        on_forward_end=_on_forward_end,
        get_parameter_and_buffer_dicts=lambda: offloaders[
            index
        ].wait_and_get_device_tensors(),
    )
```
**EN:** This function implements `_hook_module_forward_for_offloader`. It primarily calls `_hook_module_forward_raw`, `offloaders.start_onload`, `offloaders.offload`, `offloaders.wait_and_get_device_tensors`, `len` to complete its work.
**CN:** 该函数实现了 `_hook_module_forward_for_offloader`。 它主要通过调用 `_hook_module_forward_raw`, `offloaders.start_onload`, `offloaders.offload`, `offloaders.wait_and_get_device_tensors`, `len` 来完成任务。

### Lines 259-271: Function `_hook_module_forward_raw` / 函数 `_hook_module_forward_raw`
```python
def _hook_module_forward_raw(module, on_forward_end, get_parameter_and_buffer_dicts):
    original_forward = module.forward

    def forward(*args, **kwargs):
        module.forward = original_forward
        output = functional_call(
            module, get_parameter_and_buffer_dicts(), args=args, kwargs=kwargs
        )
        on_forward_end()
        module.forward = forward
        return output

    module.forward = forward
```
**EN:** This function implements `_hook_module_forward_raw`. It primarily calls `functional_call`, `on_forward_end`, `get_parameter_and_buffer_dicts` to complete its work. State updates are written into `original_forward`, `module.forward`, `output`.
**CN:** 该函数实现了 `_hook_module_forward_raw`。 它主要通过调用 `functional_call`, `on_forward_end`, `get_parameter_and_buffer_dicts` 来完成任务。 状态更新主要写入 `original_forward`, `module.forward`, `output`。

### Lines 274-274: Class `_ModuleOffloader` declaration / 类 `_ModuleOffloader` 声明
```python
class _ModuleOffloader(ABC):
```
**EN:** This class establishes `_ModuleOffloader` as the main container/coordinator for the surrounding logic. It inherits from `ABC`. Its core interface includes methods such as `__init__`, `post_init`, `start_onload`, `offload`, `wait_and_get_device_tensors`, `_create_device_tensors`.
**CN:** 该类将 `_ModuleOffloader` 定义为周边逻辑的主要封装体或协调者。 它继承自 `ABC`。 其核心接口包括 `__init__`, `post_init`, `start_onload`, `offload`, `wait_and_get_device_tensors`, `_create_device_tensors` 等方法。

### Lines 275-302: Method `_ModuleOffloader.__init__` / 方法 `_ModuleOffloader.__init__`
```python
    def __init__(
        self,
        mode: str,
        module: torch.nn.Module,
        alt_stream: torch.cuda.Stream,
        whitelist_param_names: List[str],
    ):
        self.mode = mode
        self.module = module
        self.device = next(module.parameters()).device
        self.alt_stream = alt_stream

        assert self.device != torch.device(
            "cpu"
        ), "not handled device=cpu case yet (should skip this tensor)"

        self._device_tensors = None
        self._load_event = None

        param_dict = dict(self.module.named_parameters())
        assert all(
            name in param_dict for name in whitelist_param_names
        ), f"{whitelist_param_names=} {list(param_dict.keys())=}"

        self._param_offloaders = {
            name: _BaseParamOffloader.create(mode, module=module, param_name=name)
            for name in whitelist_param_names
        }
```
**EN:** This method implements `__init__` on `_ModuleOffloader`. It primarily calls `dict`, `all`, `next`, `torch.device`, `self.module.named_parameters`, `_BaseParamOffloader.create` to complete its work. State updates are written into `self.mode`, `self.module`, `self.device`, `self.alt_stream`, `self._device_tensors`, `self._load_event`.
**CN:** 该方法（属于 `_ModuleOffloader`）实现了 `__init__`。 它主要通过调用 `dict`, `all`, `next`, `torch.device`, `self.module.named_parameters`, `_BaseParamOffloader.create` 来完成任务。 状态更新主要写入 `self.mode`, `self.module`, `self.device`, `self.alt_stream`, `self._device_tensors`, `self._load_event`。

### Lines 304-306: Method `_ModuleOffloader.post_init` / 方法 `_ModuleOffloader.post_init`
```python
    def post_init(self):
        for name, param_offloader in self._param_offloaders.items():
            param_offloader.post_init()
```
**EN:** This method implements `post_init` on `_ModuleOffloader`. It primarily calls `self._param_offloaders.items`, `param_offloader.post_init` to complete its work. The implementation relies on iteration.
**CN:** 该方法（属于 `_ModuleOffloader`）实现了 `post_init`。 它主要通过调用 `self._param_offloaders.items`, `param_offloader.post_init` 来完成任务。 实现中使用了迭代逻辑。

### Lines 308-313: Method `_ModuleOffloader.start_onload` / 方法 `_ModuleOffloader.start_onload`
```python
    def start_onload(self):
        self.alt_stream.wait_stream(torch.cuda.current_stream())
        with torch.cuda.stream(self.alt_stream):
            self._device_tensors = self._create_device_tensors()
            self._load_event = torch.cuda.Event()
            self._load_event.record()
```
**EN:** This method implements `start_onload` on `_ModuleOffloader`. It primarily calls `self.alt_stream.wait_stream`, `torch.cuda.current_stream`, `torch.cuda.stream`, `self._create_device_tensors`, `torch.cuda.Event`, `self._load_event.record` to complete its work. State updates are written into `self._device_tensors`, `self._load_event`. The implementation relies on context-managed resources.
**CN:** 该方法（属于 `_ModuleOffloader`）实现了 `start_onload`。 它主要通过调用 `self.alt_stream.wait_stream`, `torch.cuda.current_stream`, `torch.cuda.stream`, `self._create_device_tensors`, `torch.cuda.Event`, `self._load_event.record` 来完成任务。 状态更新主要写入 `self._device_tensors`, `self._load_event`。 实现中使用了上下文管理资源。

### Lines 315-317: Method `_ModuleOffloader.offload` / 方法 `_ModuleOffloader.offload`
```python
    def offload(self):
        self._device_tensors = None
        self._load_event = None
```
**EN:** This method implements `offload` on `_ModuleOffloader`. State updates are written into `self._device_tensors`, `self._load_event`.
**CN:** 该方法（属于 `_ModuleOffloader`）实现了 `offload`。 状态更新主要写入 `self._device_tensors`, `self._load_event`。

### Lines 319-322: Method `_ModuleOffloader.wait_and_get_device_tensors` / 方法 `_ModuleOffloader.wait_and_get_device_tensors`
```python
    def wait_and_get_device_tensors(self):
        assert self._device_tensors is not None
        self._load_event.wait()
        return self._device_tensors
```
**EN:** This method implements `wait_and_get_device_tensors` on `_ModuleOffloader`. It primarily calls `self._load_event.wait` to complete its work.
**CN:** 该方法（属于 `_ModuleOffloader`）实现了 `wait_and_get_device_tensors`。 它主要通过调用 `self._load_event.wait` 来完成任务。

### Lines 324-325: Method `_ModuleOffloader._create_device_tensors` / 方法 `_ModuleOffloader._create_device_tensors`
```python
    def _create_device_tensors(self):
        return {k: v.create_device_tensor() for k, v in self._param_offloaders.items()}
```
**EN:** This method implements `_create_device_tensors` on `_ModuleOffloader`. It primarily calls `v.create_device_tensor`, `self._param_offloaders.items` to complete its work.
**CN:** 该方法（属于 `_ModuleOffloader`）实现了 `_create_device_tensors`。 它主要通过调用 `v.create_device_tensor`, `self._param_offloaders.items` 来完成任务。

### Lines 328-328: Class `_BaseParamOffloader` declaration / 类 `_BaseParamOffloader` 声明
```python
class _BaseParamOffloader(ABC):
```
**EN:** This class establishes `_BaseParamOffloader` as the main container/coordinator for the surrounding logic. It inherits from `ABC`. Its core interface includes methods such as `create`, `__init__`, `_param`, `post_init`, `create_device_tensor`.
**CN:** 该类将 `_BaseParamOffloader` 定义为周边逻辑的主要封装体或协调者。 它继承自 `ABC`。 其核心接口包括 `create`, `__init__`, `_param`, `post_init`, `create_device_tensor` 等方法。

### Lines 329-336: Method `_BaseParamOffloader.create` / 方法 `_BaseParamOffloader.create`
```python
    @staticmethod
    def create(mode: str, **kwargs) -> "_BaseParamOffloader":
        return {
            "meta": _MetaParamOffloader,
            "cpu": _CpuParamOffloader,
            "shm_cpu": _ShmCpuParamOffloader,
            "sharded_gpu": _ShardedGpuParamOffloader,
        }[mode](**kwargs)
```
**EN:** This method implements `create` on `_BaseParamOffloader`.
**CN:** 该方法（属于 `_BaseParamOffloader`）实现了 `create`。

### Lines 338-340: Method `_BaseParamOffloader.__init__` / 方法 `_BaseParamOffloader.__init__`
```python
    def __init__(self, module, param_name):
        self._module = module
        self._param_name = param_name
```
**EN:** This method implements `__init__` on `_BaseParamOffloader`. State updates are written into `self._module`, `self._param_name`.
**CN:** 该方法（属于 `_BaseParamOffloader`）实现了 `__init__`。 状态更新主要写入 `self._module`, `self._param_name`。

### Lines 342-344: Method `_BaseParamOffloader._param` / 方法 `_BaseParamOffloader._param`
```python
    @property
    def _param(self):
        return getattr(self._module, self._param_name)
```
**EN:** This method implements `_param` on `_BaseParamOffloader`. It primarily calls `getattr` to complete its work.
**CN:** 该方法（属于 `_BaseParamOffloader`）实现了 `_param`。 它主要通过调用 `getattr` 来完成任务。

### Lines 346-347: Method `_BaseParamOffloader.post_init` / 方法 `_BaseParamOffloader.post_init`
```python
    def post_init(self):
        pass
```
**EN:** This method implements `post_init` on `_BaseParamOffloader`.
**CN:** 该方法（属于 `_BaseParamOffloader`）实现了 `post_init`。

### Lines 349-350: Method `_BaseParamOffloader.create_device_tensor` / 方法 `_BaseParamOffloader.create_device_tensor`
```python
    def create_device_tensor(self):
        raise NotImplementedError
```
**EN:** This method implements `create_device_tensor` on `_BaseParamOffloader`.
**CN:** 该方法（属于 `_BaseParamOffloader`）实现了 `create_device_tensor`。

### Lines 353-354: Class `_MetaParamOffloader` declaration / 类 `_MetaParamOffloader` 声明
```python
class _MetaParamOffloader(_BaseParamOffloader):
    """Usually used for debugging."""
```
**EN:** This class establishes `_MetaParamOffloader` as the main container/coordinator for the surrounding logic. It inherits from `_BaseParamOffloader`. Its core interface includes methods such as `__init__`, `create_device_tensor`.
**CN:** 该类将 `_MetaParamOffloader` 定义为周边逻辑的主要封装体或协调者。 它继承自 `_BaseParamOffloader`。 其核心接口包括 `__init__`, `create_device_tensor` 等方法。

### Lines 356-358: Method `_MetaParamOffloader.__init__` / 方法 `_MetaParamOffloader.__init__`
```python
    def __init__(self, module, param_name):
        super().__init__(module, param_name)
        _move_param_to_meta(module, param_name)
```
**EN:** This method implements `__init__` on `_MetaParamOffloader`. It primarily calls `super.__init__`, `_move_param_to_meta`, `super` to complete its work.
**CN:** 该方法（属于 `_MetaParamOffloader`）实现了 `__init__`。 它主要通过调用 `super.__init__`, `_move_param_to_meta`, `super` 来完成任务。

### Lines 360-361: Method `_MetaParamOffloader.create_device_tensor` / 方法 `_MetaParamOffloader.create_device_tensor`
```python
    def create_device_tensor(self):
        return torch.empty_like(self._param.data, device="cuda")
```
**EN:** This method implements `create_device_tensor` on `_MetaParamOffloader`. It primarily calls `torch.empty_like` to complete its work.
**CN:** 该方法（属于 `_MetaParamOffloader`）实现了 `create_device_tensor`。 它主要通过调用 `torch.empty_like` 来完成任务。

### Lines 364-364: Class `_CpuParamOffloader` declaration / 类 `_CpuParamOffloader` 声明
```python
class _CpuParamOffloader(_BaseParamOffloader):
```
**EN:** This class establishes `_CpuParamOffloader` as the main container/coordinator for the surrounding logic. It inherits from `_BaseParamOffloader`. Its core interface includes methods such as `__init__`, `create_device_tensor`.
**CN:** 该类将 `_CpuParamOffloader` 定义为周边逻辑的主要封装体或协调者。 它继承自 `_BaseParamOffloader`。 其核心接口包括 `__init__`, `create_device_tensor` 等方法。

### Lines 365-367: Method `_CpuParamOffloader.__init__` / 方法 `_CpuParamOffloader.__init__`
```python
    def __init__(self, module, param_name):
        super().__init__(module, param_name)
        _move_param_to_cpu(self._param, pin_memory=True)
```
**EN:** This method implements `__init__` on `_CpuParamOffloader`. It primarily calls `super.__init__`, `_move_param_to_cpu`, `super` to complete its work.
**CN:** 该方法（属于 `_CpuParamOffloader`）实现了 `__init__`。 它主要通过调用 `super.__init__`, `_move_param_to_cpu`, `super` 来完成任务。

### Lines 369-370: Method `_CpuParamOffloader.create_device_tensor` / 方法 `_CpuParamOffloader.create_device_tensor`
```python
    def create_device_tensor(self):
        return self._param.to("cuda", non_blocking=True)
```
**EN:** This method implements `create_device_tensor` on `_CpuParamOffloader`. It primarily calls `self._param.to` to complete its work.
**CN:** 该方法（属于 `_CpuParamOffloader`）实现了 `create_device_tensor`。 它主要通过调用 `self._param.to` 来完成任务。

### Lines 373-373: Class `_ShmCpuParamOffloader` declaration / 类 `_ShmCpuParamOffloader` 声明
```python
class _ShmCpuParamOffloader(_BaseParamOffloader):
```
**EN:** This class establishes `_ShmCpuParamOffloader` as the main container/coordinator for the surrounding logic. It inherits from `_BaseParamOffloader`. Its core interface includes methods such as `__init__`, `post_init`, `create_device_tensor`.
**CN:** 该类将 `_ShmCpuParamOffloader` 定义为周边逻辑的主要封装体或协调者。 它继承自 `_BaseParamOffloader`。 其核心接口包括 `__init__`, `post_init`, `create_device_tensor` 等方法。

### Lines 374-395: Method `_ShmCpuParamOffloader.__init__` / 方法 `_ShmCpuParamOffloader.__init__`
```python
    def __init__(self, module, param_name):
        super().__init__(module, param_name)
        self._rank = get_naive_distributed().get_rank()
        self._world_size = get_naive_distributed().get_world_size()

        from sglang.srt.distributed import get_tensor_model_parallel_world_size

        assert get_tensor_model_parallel_world_size() == 1, "not yet support tp_size!=1"
        assert (
            self._param.data.is_contiguous()
        ), f"not yet support non-contiguous tensor {self._param.shape=} {self._param.stride()=}"

        self.shm_cpu_data = get_host_shared_memory_manager().malloc(
            shape=self._param.shape, dtype=self._param.dtype
        )

        if self._rank == 0:
            self.shm_cpu_data.copy_(self._param.data.to("cpu"))
            self._param.data = self.shm_cpu_data
        else:
            _move_param_to_meta(self._module, self._param_name)
        get_naive_distributed().barrier()
```
**EN:** This method implements `__init__` on `_ShmCpuParamOffloader`. It primarily calls `super.__init__`, `get_naive_distributed.get_rank`, `get_naive_distributed.get_world_size`, `self._param.data.is_contiguous`, `get_host_shared_memory_manager.malloc`, `get_naive_distributed.barrier` to complete its work. State updates are written into `self._rank`, `self._world_size`, `self.shm_cpu_data`, `self._param.data`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `_ShmCpuParamOffloader`）实现了 `__init__`。 它主要通过调用 `super.__init__`, `get_naive_distributed.get_rank`, `get_naive_distributed.get_world_size`, `self._param.data.is_contiguous`, `get_host_shared_memory_manager.malloc`, `get_naive_distributed.barrier` 来完成任务。 状态更新主要写入 `self._rank`, `self._world_size`, `self.shm_cpu_data`, `self._param.data`。 实现中使用了条件分支。

### Lines 397-403: Method `_ShmCpuParamOffloader.post_init` / 方法 `_ShmCpuParamOffloader.post_init`
```python
    def post_init(self):
        if self._rank == 0:
            assert (
                self.shm_cpu_data.data_ptr() == self._param.data.data_ptr()
            ), f"{self.shm_cpu_data.data_ptr()=} {self._param.data.data_ptr()=} {self.shm_cpu_data=} {self._param.data=}"

        _move_param_to_meta(self._module, self._param_name)
```
**EN:** This method implements `post_init` on `_ShmCpuParamOffloader`. It primarily calls `_move_param_to_meta`, `self.shm_cpu_data.data_ptr`, `self._param.data.data_ptr` to complete its work. The implementation relies on conditional branches.
**CN:** 该方法（属于 `_ShmCpuParamOffloader`）实现了 `post_init`。 它主要通过调用 `_move_param_to_meta`, `self.shm_cpu_data.data_ptr`, `self._param.data.data_ptr` 来完成任务。 实现中使用了条件分支。

### Lines 405-406: Method `_ShmCpuParamOffloader.create_device_tensor` / 方法 `_ShmCpuParamOffloader.create_device_tensor`
```python
    def create_device_tensor(self):
        return self.shm_cpu_data.to("cuda", non_blocking=True)
```
**EN:** This method implements `create_device_tensor` on `_ShmCpuParamOffloader`. It primarily calls `self.shm_cpu_data.to` to complete its work.
**CN:** 该方法（属于 `_ShmCpuParamOffloader`）实现了 `create_device_tensor`。 它主要通过调用 `self.shm_cpu_data.to` 来完成任务。

### Lines 409-418: Function `update_param` / 函数 `update_param`
```python
def update_param(param, new_tensor):
    """Update parameter while keeping properties needed by Offloader (e.g. pinned host memory)."""

    if param.device == new_tensor.device:
        param.data = new_tensor
    else:
        assert param.device == torch.device(
            "cpu"
        ), f"{param.device=} {new_tensor.device=}"
        param.data = _create_cpu_data(new_tensor, pin_memory=True)
```
**EN:** This function implements `update_param`. It primarily calls `_create_cpu_data`, `torch.device` to complete its work. State updates are written into `param.data`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `update_param`。 它主要通过调用 `_create_cpu_data`, `torch.device` 来完成任务。 状态更新主要写入 `param.data`。 实现中使用了条件分支。

### Lines 421-422: Function `_move_param_to_cpu` / 函数 `_move_param_to_cpu`
```python
def _move_param_to_cpu(param, pin_memory: bool):
    param.data = _create_cpu_data(param.data, pin_memory=pin_memory)
```
**EN:** This function implements `_move_param_to_cpu`. It primarily calls `_create_cpu_data` to complete its work. State updates are written into `param.data`.
**CN:** 该函数实现了 `_move_param_to_cpu`。 它主要通过调用 `_create_cpu_data` 来完成任务。 状态更新主要写入 `param.data`。

### Lines 425-432: Function `_create_cpu_data` / 函数 `_create_cpu_data`
```python
def _create_cpu_data(data, pin_memory: bool):
    cpu_data = _empty_strided_like(
        data,
        device="cpu",
        pin_memory=pin_memory,
    )
    cpu_data.copy_(data)
    return cpu_data
```
**EN:** This function implements `_create_cpu_data`. It primarily calls `_empty_strided_like`, `cpu_data.copy_` to complete its work. State updates are written into `cpu_data`.
**CN:** 该函数实现了 `_create_cpu_data`。 它主要通过调用 `_empty_strided_like`, `cpu_data.copy_` 来完成任务。 状态更新主要写入 `cpu_data`。

### Lines 435-462: Function `_move_param_to_meta` / 函数 `_move_param_to_meta`
```python
def _move_param_to_meta(module, param_name):
    old_param = getattr(module, param_name)
    old_param_type = type(old_param)

    new_data = old_param.data.to("meta")

    if old_param_type == ModelWeightParameter:
        # manually checked how `w13_weight` and `w2_weight` are constructed
        new_param = ModelWeightParameter(
            data=new_data,
            **{
                k: getattr(old_param, k)
                for k in ["input_dim", "output_dim", "weight_loader"]
            },
        )
    elif old_param_type == torch.nn.Parameter:
        new_param = torch.nn.Parameter(
            data=new_data,
            requires_grad=False,
        )
        if hasattr(old_param, "weihgt_loader"):
            new_param.weight_loader = old_param.weight_loader
        else:
            new_param.weight_loader = lambda *args, **kwargs: None
    else:
        raise ValueError(f"Unknown {old_param_type=} {old_param=}")

    setattr(module, param_name, new_param)
```
**EN:** This function implements `_move_param_to_meta`. It primarily calls `getattr`, `type`, `old_param.data.to`, `setattr`, `ModelWeightParameter`, `torch.nn.Parameter` to complete its work. State updates are written into `old_param`, `old_param_type`, `new_data`, `new_param`, `new_param.weight_loader`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_move_param_to_meta`。 它主要通过调用 `getattr`, `type`, `old_param.data.to`, `setattr`, `ModelWeightParameter`, `torch.nn.Parameter` 来完成任务。 状态更新主要写入 `old_param`, `old_param_type`, `new_data`, `new_param`, `new_param.weight_loader`。 实现中使用了条件分支。

### Lines 465-473: Function `_empty_strided_like` / 函数 `_empty_strided_like`
```python
def _empty_strided_like(x: torch.Tensor, device, pin_memory=False):
    return torch.empty_strided(
        size=x.size(),
        stride=x.stride(),
        dtype=x.dtype,
        layout=x.layout,
        device=device,
        pin_memory=pin_memory,
    )
```
**EN:** This function implements `_empty_strided_like`. It primarily calls `torch.empty_strided`, `x.size`, `x.stride` to complete its work.
**CN:** 该函数实现了 `_empty_strided_like`。 它主要通过调用 `torch.empty_strided`, `x.size`, `x.stride` 来完成任务。

### Lines 480-480: Class `_ShardedGpuParamOffloader` declaration / 类 `_ShardedGpuParamOffloader` 声明
```python
class _ShardedGpuParamOffloader(_BaseParamOffloader):
```
**EN:** This class establishes `_ShardedGpuParamOffloader` as the main container/coordinator for the surrounding logic. It inherits from `_BaseParamOffloader`. Its core interface includes methods such as `__init__`, `post_init`, `create_device_tensor`.
**CN:** 该类将 `_ShardedGpuParamOffloader` 定义为周边逻辑的主要封装体或协调者。 它继承自 `_BaseParamOffloader`。 其核心接口包括 `__init__`, `post_init`, `create_device_tensor` 等方法。

### Lines 481-498: Method `_ShardedGpuParamOffloader.__init__` / 方法 `_ShardedGpuParamOffloader.__init__`
```python
    def __init__(self, module, param_name):
        super().__init__(module, param_name)
        self._rank = get_naive_distributed().get_rank()
        self._world_size = get_naive_distributed().get_world_size()

        from sglang.srt.distributed import get_tensor_model_parallel_world_size

        assert get_tensor_model_parallel_world_size() == 1, "not yet support tp_size!=1"
        assert (
            self._param.data.is_contiguous()
        ), f"not yet support non-contiguous tensor {self._param.shape=} {self._param.stride()=}"

        if self._rank == 0:
            _move_param_to_cpu(self._param, pin_memory=True)
        else:
            _move_param_to_meta(self._module, self._param_name)

        self.sharded_param_handles = None
```
**EN:** This method implements `__init__` on `_ShardedGpuParamOffloader`. It primarily calls `super.__init__`, `get_naive_distributed.get_rank`, `get_naive_distributed.get_world_size`, `self._param.data.is_contiguous`, `get_tensor_model_parallel_world_size`, `_move_param_to_cpu` to complete its work. State updates are written into `self._rank`, `self._world_size`, `self.sharded_param_handles`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `_ShardedGpuParamOffloader`）实现了 `__init__`。 它主要通过调用 `super.__init__`, `get_naive_distributed.get_rank`, `get_naive_distributed.get_world_size`, `self._param.data.is_contiguous`, `get_tensor_model_parallel_world_size`, `_move_param_to_cpu` 来完成任务。 状态更新主要写入 `self._rank`, `self._world_size`, `self.sharded_param_handles`。 实现中使用了条件分支。

### Lines 500-527: Method `_ShardedGpuParamOffloader.post_init` / 方法 `_ShardedGpuParamOffloader.post_init`
```python
    def post_init(self):
        # check again since it may be changed
        assert (
            self._param.data.is_contiguous()
        ), f"not yet support non-contiguous tensor {self._param.shape=} {self._param.stride()=}"

        scatter_src = self._param.data

        logger.info(
            f"[offloader] post_init {scatter_src.nbytes=} {scatter_src.dtype=} {scatter_src.shape=} {torch.cuda.memory_allocated()=}"
        )

        if self._rank == 0:
            scatter_src = scatter_src.to("cuda")
        scatter_list = _even_chunk(scatter_src, self._world_size)

        sharded_param = torch.empty(
            scatter_list[0].shape, dtype=scatter_list[0].dtype, device="cuda"
        )
        self.sharded_param_handles = _create_shared_buffer_tensors(
            local_tensor=sharded_param
        )

        get_naive_distributed().scatter(
            sharded_param, scatter_list if self._rank == 0 else None
        )

        _move_param_to_meta(self._module, self._param_name)
```
**EN:** This method implements `post_init` on `_ShardedGpuParamOffloader`. It primarily calls `self._param.data.is_contiguous`, `logger.info`, `_even_chunk`, `torch.empty`, `_create_shared_buffer_tensors`, `get_naive_distributed.scatter` to complete its work. State updates are written into `scatter_src`, `scatter_list`, `sharded_param`, `self.sharded_param_handles`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `_ShardedGpuParamOffloader`）实现了 `post_init`。 它主要通过调用 `self._param.data.is_contiguous`, `logger.info`, `_even_chunk`, `torch.empty`, `_create_shared_buffer_tensors`, `get_naive_distributed.scatter` 来完成任务。 状态更新主要写入 `scatter_src`, `scatter_list`, `sharded_param`, `self.sharded_param_handles`。 实现中使用了条件分支。

### Lines 529-538: Method `_ShardedGpuParamOffloader.create_device_tensor` / 方法 `_ShardedGpuParamOffloader.create_device_tensor`
```python
    def create_device_tensor(self):
        output = _empty_strided_like(self._param, device="cuda")
        output_chunks = output.chunk(self._world_size)

        for index in range(self._world_size):
            src_rank = (self._rank + index) % self._world_size
            src_buf = self.sharded_param_handles[src_rank]
            output_chunks[src_rank].copy_(src_buf)

        return output
```
**EN:** This method implements `create_device_tensor` on `_ShardedGpuParamOffloader`. It primarily calls `_empty_strided_like`, `output.chunk`, `range`, `output_chunks.copy_` to complete its work. State updates are written into `output`, `output_chunks`, `src_rank`, `src_buf`. The implementation relies on iteration.
**CN:** 该方法（属于 `_ShardedGpuParamOffloader`）实现了 `create_device_tensor`。 它主要通过调用 `_empty_strided_like`, `output.chunk`, `range`, `output_chunks.copy_` 来完成任务。 状态更新主要写入 `output`, `output_chunks`, `src_rank`, `src_buf`。 实现中使用了迭代逻辑。

### Lines 541-543: Function `_even_chunk` / 函数 `_even_chunk`
```python
def _even_chunk(x: torch.Tensor, chunks: int):
    assert x.shape[0] % chunks == 0, f"{x.shape=} {chunks=}"
    return list(x.chunk(chunks))
```
**EN:** This function implements `_even_chunk`. It primarily calls `list`, `x.chunk` to complete its work.
**CN:** 该函数实现了 `_even_chunk`。 它主要通过调用 `list`, `x.chunk` 来完成任务。

### Lines 546-576: Function `_create_shared_buffer_tensors` / 函数 `_create_shared_buffer_tensors`
```python
def _create_shared_buffer_tensors(local_tensor: torch.Tensor) -> List[torch.Tensor]:
    self_rank = get_naive_distributed().get_rank()
    world_size = get_naive_distributed().get_world_size()

    object_list = get_naive_distributed().all_gather_object(
        dict(
            dup_serialized_local_tensor=[
                (
                    None
                    if interesting_rank == self_rank
                    else MultiprocessingSerializer.serialize(local_tensor)
                )
                for interesting_rank in range(world_size)
            ]
        )
    )

    output_tensors = []
    for output_rank in range(world_size):
        remote_serialized_tensor = object_list[output_rank][
            "dup_serialized_local_tensor"
        ][self_rank]
        if output_rank == self_rank:
            assert remote_serialized_tensor is None
            output_tensors.append(local_tensor)
        else:
            output_tensors.append(
                MultiprocessingSerializer.deserialize(remote_serialized_tensor)
            )

    return output_tensors
```
**EN:** This function implements `_create_shared_buffer_tensors`. It primarily calls `get_naive_distributed.get_rank`, `get_naive_distributed.get_world_size`, `get_naive_distributed.all_gather_object`, `range`, `dict`, `get_naive_distributed` to complete its work. State updates are written into `self_rank`, `world_size`, `object_list`, `output_tensors`, `remote_serialized_tensor`. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `_create_shared_buffer_tensors`。 它主要通过调用 `get_naive_distributed.get_rank`, `get_naive_distributed.get_world_size`, `get_naive_distributed.all_gather_object`, `range`, `dict`, `get_naive_distributed` 来完成任务。 状态更新主要写入 `self_rank`, `world_size`, `object_list`, `output_tensors`, `remote_serialized_tensor`。 实现中使用了条件分支、迭代逻辑。

## Key Concepts / 关键概念
- **Classes / 类**: `BaseOffloader`, `NoopOffloader`, `OffloaderV1`, `OffloaderV2`, `_ModuleOffloader`, `_BaseParamOffloader`, `_MetaParamOffloader`, `_CpuParamOffloader`
- **Functions / 函数**: `get_offloader`, `set_offloader`, `create_offloader_from_server_args`, `_hook_module_forward_for_offloader`, `_hook_module_forward_raw`, `update_param`, `_move_param_to_cpu`, `_create_cpu_data`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.distributed.naive_distributed`, `sglang.srt.layers.parameter`, `sglang.srt.server_args`, `sglang.srt.utils`, `sglang.srt.utils.host_shared_memory`, `sglang.srt.distributed`
- **External / 外部依赖**: `torch`, `torch.func`
- **Standard library / 标准库**: `logging`, `os`, `abc`, `typing`
