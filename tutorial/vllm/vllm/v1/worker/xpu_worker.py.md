# xpu_worker.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/xpu_worker.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `XPUWorker` for the V1 `worker` subsystem. / 为 V1 的 `worker` 子系统实现 `XPUWorker`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import gc
import os

import torch

from vllm.config import VllmConfig
from vllm.logger import init_logger
from vllm.platforms import current_platform
from vllm.profiler.wrapper import TorchProfilerWrapper
from vllm.utils.mem_utils import MemorySnapshot, format_gib
from vllm.utils.torch_utils import set_random_seed
from vllm.v1.utils import report_usage_stats
from vllm.v1.worker.gpu_worker import Worker, init_worker_distributed_environment
from vllm.v1.worker.workspace import init_workspace_manager
from vllm.v1.worker.xpu_model_runner import XPUModelRunner, XPUModelRunnerV2

from .utils import request_memory

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `XPUWorker` class / `XPUWorker` 类
```python
class XPUWorker(Worker):
    """A XPU worker class."""
```
**EN:** Introduces the `XPUWorker` class on top of `Worker`. Core methods include `__init__`, `init_device`, `profile`. Docstring signal: A XPU worker class.
**CN:** 这里定义 `XPUWorker` 类，其基类包括 `Worker`。核心方法包括 `__init__`, `init_device`, `profile`。

### `XPUWorker.__init__` method / `XPUWorker.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        local_rank: int,
        rank: int,
        distributed_init_method: str,
        is_driver_worker: bool = False,
    ):
        super().__init__(
            vllm_config, local_rank, rank, distributed_init_method, is_driver_worker
        )
        device_config = self.device_config
        assert device_config.device_type == "xpu"
        assert current_platform.is_xpu()
```
**EN:** This method initializes the object state within `XPUWorker`. Key calls include `__init__`, `is_xpu`, `super`.
**CN:** 该方法会初始化对象状态，其作用域位于`XPUWorker`。 关键调用包括 `__init__`, `is_xpu`, `super`。

### `XPUWorker.init_device` method / `XPUWorker.init_device` 方法
```python
    def init_device(self):
        # In DP mode, XPU workers see all visible devices.
        # Offset local_rank by the local DP shard.
        parallel_config = self.parallel_config
        if (
            parallel_config.distributed_executor_backend
            not in ("ray", "external_launcher")
            and parallel_config.data_parallel_backend != "ray"
            and parallel_config.nnodes_within_dp == 1
        ):
            dp_local_rank = parallel_config.data_parallel_rank_local
            if dp_local_rank is None:
                dp_local_rank = parallel_config.data_parallel_index
            tp_pp_world_size = (
                parallel_config.pipeline_parallel_size
                * parallel_config.tensor_parallel_size
            )
            self.local_rank += dp_local_rank * tp_pp_world_size

            visible_device_count = torch.accelerator.device_count()
            assert self.local_rank < visible_device_count, (
                f"DP adjusted local rank {self.local_rank} is out of bounds. "
            )
            assert parallel_config.local_world_size <= visible_device_count, (
                f"local_world_size ({parallel_config.local_world_size}) must "
                f"be less than or equal to the number of visible devices "
                f"({visible_device_count})."
            )

        device = self.device_config.device
        if (
            isinstance(device, torch.device)
            and device.type == "xpu"
            and current_platform.is_xpu()
        ):
            self.device = torch.device(f"xpu:{self.local_rank}")
            torch.accelerator.set_device_index(self.device)
            current_platform.check_if_supports_dtype(self.model_config.dtype)
            torch.accelerator.empty_cache()
            self.init_gpu_memory = torch.xpu.get_device_properties(
                self.local_rank
            ).total_memory
        else:
            raise RuntimeError(f"Not support device type: {self.device_config.device}")

        ENV_CCL_ATL_TRANSPORT = os.getenv("CCL_ATL_TRANSPORT", "ofi")
        ENV_LOCAL_WORLD_SIZE = os.getenv(
            "LOCAL_WORLD_SIZE", str(self.parallel_config.world_size)
        )
        os.environ["CCL_ATL_TRANSPORT"] = ENV_CCL_ATL_TRANSPORT
    # ... omitted for brevity ...
        # Now take memory snapshot after NCCL is initialized
        gc.collect()
        torch.accelerator.empty_cache()

        # take current memory snapshot
        self.init_snapshot = init_snapshot = MemorySnapshot(device=self.device)
        self.requested_memory = request_memory(init_snapshot, self.cache_config)
        logger.debug("worker init memory snapshot: %r", self.init_snapshot)
        logger.debug(
            "worker requested memory: %sGiB", format_gib(self.requested_memory)
        )

        # Initialize workspace manager
        num_ubatches = 2 if self.vllm_config.parallel_config.enable_dbo else 1
        init_workspace_manager(self.device, num_ubatches)

        # Construct the model runner
        model_runner = XPUModelRunnerV2 if self.use_v2_model_runner else XPUModelRunner
        self.model_runner = model_runner(  # type: ignore
            self.vllm_config, self.device
        )

        if self.rank == 0:
            # If usage stat is enabled, collect relevant info.
            report_usage_stats(self.vllm_config)
```
**EN:** This method implements `init_device` within `XPUWorker`. Key calls include `getenv`, `str`, `init_worker_distributed_environment`, `is_xccl_available`, `set_random_seed`, `collect`. It touches state such as `init_snapshot`, `requested_memory`, `model_runner`, `local_rank`, `device`, `init_gpu_memory`. The control flow contains 8 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `init_device`，其作用域位于`XPUWorker`。 关键调用包括 `getenv`, `str`, `init_worker_distributed_environment`, `is_xccl_available`, `set_random_seed`, `collect`。 它会读写 `init_snapshot`, `requested_memory`, `model_runner`, `local_rank`, `device`, `init_gpu_memory` 等状态。 控制流包含 8 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `XPUWorker.profile` method / `XPUWorker.profile` 方法
```python
    def profile(self, is_start: bool = True, profile_prefix: str | None = None):
        if self.profiler_config is None or self.profiler_config.profiler is None:
            raise RuntimeError(
                "Profiling is not enabled. Please set --profiler-config to enable "
                "profiling. Example: "
                "'--profiler-config.profiler=torch --profiler-config.torch_profiler_dir"
                "=YOUR_DIR_PATH_TO_DUMP_TRACE'"
            )

        if is_start and self.profiler is None:
            from vllm.distributed.utils import get_worker_rank_suffix

            rank_suffix = get_worker_rank_suffix(global_rank=self.rank)
            trace_name = (
                f"{profile_prefix}_{rank_suffix}" if profile_prefix else rank_suffix
            )

            self.profiler = TorchProfilerWrapper(
                self.profiler_config,
                worker_name=trace_name,
                local_rank=self.local_rank,
                activities=["CPU", "XPU"],
            )
            logger.debug("Starting torch profiler with trace name: %s", trace_name)

        super().profile(is_start=is_start, profile_prefix=profile_prefix)
```
**EN:** This method implements `profile` within `XPUWorker`. Key calls include `profile`, `RuntimeError`, `get_worker_rank_suffix`, `TorchProfilerWrapper`, `debug`, `super`. It touches state such as `profiler`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `profile`，其作用域位于`XPUWorker`。 关键调用包括 `profile`, `RuntimeError`, `get_worker_rank_suffix`, `TorchProfilerWrapper`, `debug`, `super`。 它会读写 `profiler` 等状态。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `XPUWorker`: central class or interface in this module. / `XPUWorker`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `gc`, `os`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.logger`, `vllm.platforms`, `vllm.profiler.wrapper`, `vllm.utils.mem_utils`, `vllm.utils.torch_utils`, `vllm.v1.utils`, `vllm.v1.worker.gpu_worker`, `vllm.v1.worker.workspace`, `vllm.v1.worker.xpu_model_runner`, `.utils`, `vllm.distributed.utils`
