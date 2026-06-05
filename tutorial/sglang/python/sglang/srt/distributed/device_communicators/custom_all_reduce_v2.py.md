# custom_all_reduce_v2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/distributed/device_communicators/custom_all_reduce_v2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module belongs to the distributed execution and communication part of the SRT runtime and implements logic centered on `custom_all_reduce_v2`. It exposes primary entry points such as `ModeConfig`, `CustomAllReduceV2`, `_init_config`. / 该模块属于 SRT 运行时的分布式执行与通信部分，主要实现围绕 `custom_all_reduce_v2` 的逻辑。 它对外提供的主要入口包括 `ModeConfig`, `CustomAllReduceV2`, `_init_config`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Module imports, constants, and setup
```python
import logging
from contextlib import contextmanager
from dataclasses import dataclass, replace
from typing import Dict, List, Optional, TypeVar

import torch
import torch.distributed as dist
from torch.distributed import ProcessGroup

from sglang.jit_kernel.all_reduce import AllReduceAlgo, get_custom_all_reduce_cls
from sglang.srt.distributed import is_in_piecewise_cuda_graph
from sglang.srt.distributed.device_communicators.custom_all_reduce_utils import (
    can_use_custom_all_reduce_with_nvlink,
    is_weak_contiguous,
)
from sglang.srt.utils import is_sm100_supported, log_info_on_rank0

logger = logging.getLogger(__name__)

T = TypeVar("T")

INF = 1 << 60


```
**EN:** This range organizes module-level state and shared setup. In this range it sets up imports and shared symbols; emits logs for diagnostics; coordinates distributed communication.
**CN:** 这一段组织模块级状态与共享初始化逻辑。 在这一范围内，它会建立导入关系并准备共享符号；输出日志以便诊断；协调分布式通信。

### Lines 25-28: Class ModeConfig
```python
@dataclass(frozen=True)
class ModeConfig:
    one_shot_push_threshold: int  # below this, use one-shot push
    one_shot_pull_threshold: int  # below this, use one-shot pull
```
**EN:** This range introduces `ModeConfig` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `ModeConfig`，并定义其后续方法依赖的结构或元数据。

### Lines 31-31: Class CustomAllReduceV2
```python
class CustomAllReduceV2:
```
**EN:** This range introduces `CustomAllReduceV2` and defines the structure or metadata that its methods rely on.
**CN:** 这一段引入 `CustomAllReduceV2`，并定义其后续方法依赖的结构或元数据。

### Lines 32-76: Method CustomAllReduceV2.__init__
```python
    def __init__(
        self,
        group: ProcessGroup,
        device: torch.device,
        max_pull_size: Optional[int] = None,
        max_push_size: Optional[int] = None,
        max_pull_blocks: Optional[int] = None,
        max_push_blocks: Optional[int] = None,
    ) -> None:
        _init_config()
        self.disabled = True
        full_nvlink = can_use_custom_all_reduce_with_nvlink(
            group=group,
            device=device,
            supported_world_size=list(THRESHOLD_2_SHOT_MAP.keys()),
            cls_name="CustomAllReduceV2",
        )
        if full_nvlink != True:
            return

        self.group = group
        self.rank = dist.get_rank(group=self.group)
        self.world_size = dist.get_world_size(group=self.group)
        if max_pull_size is None:  # default to 16MB
            max_pull_size = 16 * 1024 * 1024
        if max_push_size is None:  # default to recommended size
            config = THRESHOLD_2_SHOT_MAP[self.world_size]
            max_push_size = config.one_shot_push_threshold
        self.max_pull_size = max_pull_size
        self.max_push_size = max_push_size
        self.max_size = max(max_pull_size, max_push_size)
        self.override_shot(None)  # set default config based on world size
        self.override_algo: Optional[AllReduceAlgo] = None
        self.obj = get_custom_all_reduce_cls()(
            rank=self.rank,
            world_size=self.world_size,
            pull_buffer_bytes=self.max_pull_size,
            push_buffer_bytes=self.max_push_size,
            graph_input_count=131072,
            max_pull_blocks=max_pull_blocks,
            max_push_blocks=max_push_blocks,
        )
        self._post_init_obj()
        self.disabled = False
        log_info_on_rank0(logger, "Custom allreduce v2 initialized successfully")
```
**EN:** This callable implements `CustomAllReduceV2.__init__`. It takes `group`, `device`, `max_pull_size`, `max_push_size` and mainly initializes instance state and defaults. In this range it coordinates distributed communication; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CustomAllReduceV2.__init__`。它接收 `group`, `device`, `max_pull_size`, `max_push_size`，主要用于初始化实例状态与默认值。 在这一范围内，它会协调分布式通信；管理图捕获或回放逻辑。

### Lines 78-92: Method CustomAllReduceV2.override_shot
```python
    def override_shot(self, shot: int | None):
        if shot is None:
            config = THRESHOLD_2_SHOT_MAP[self.world_size]
        else:
            assert shot in (1, 2)
            threshold = INF if shot == 1 else 0
            config = replace(self.config, one_shot_pull_threshold=threshold)
        # need to clip the config thresholds to max sizes to avoid invalid config
        push_threshold = min(config.one_shot_push_threshold, self.max_push_size)
        pull_threshold = min(config.one_shot_pull_threshold, self.max_pull_size)
        self.config: ModeConfig = replace(
            config,
            one_shot_push_threshold=push_threshold,
            one_shot_pull_threshold=pull_threshold,
        )
```
**EN:** This callable implements `CustomAllReduceV2.override_shot`. It takes `shot` and mainly implements override shot. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `CustomAllReduceV2.override_shot`。它接收 `shot`，主要用于实现 override shot 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 94-115: Method CustomAllReduceV2.capture
```python
    @contextmanager
    def capture(self):
        if self.disabled:
            yield
            return
        try:
            self.obj.set_cuda_graph_capture(True)
            yield
        finally:
            self.obj.set_cuda_graph_capture(False)
        # cannot call when graph is capturing
        assert (
            torch.cuda.is_current_stream_capturing() == False
        ), "Cannot register graph inputs while capturing CUDA graph"
        pairs = self.obj.share_graph_inputs()
        handles = [handle for _, handle in pairs]
        offsets = [offset for offset, _ in pairs]
        handles_all = self._share_list(handles)
        offsets_all = self._share_list(offsets)
        result = [list(zip(o, h)) for o, h in zip(offsets_all, handles_all)]
        self.obj.register_inputs(result)
        log_info_on_rank0(logger, f"Registering {len(pairs)} cuda graph addresses")
```
**EN:** This callable implements `CustomAllReduceV2.capture` and mainly implements capture. In this range it performs defensive checks on invalid state; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CustomAllReduceV2.capture`，主要用于实现 capture 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查；管理图捕获或回放逻辑。

### Lines 117-127: Method CustomAllReduceV2.should_custom_ar
```python
    def should_custom_ar(self, inp: torch.Tensor) -> bool:
        """Check if the input tensor is suitable for custom all-reduce."""
        if self.disabled:
            return False
        inp_size = inp.numel() * inp.element_size()
        # custom allreduce requires input byte size to be multiples of 16
        if inp_size % 16 != 0:
            return False
        if not is_weak_contiguous(inp):
            return False
        return inp_size <= self.max_size
```
**EN:** This callable implements `CustomAllReduceV2.should_custom_ar`. It takes `inp` and mainly converts data into another representation. The docstring states: "Check if the input tensor is suitable for custom all-reduce."
**CN:** 这一可调用对象实现了 `CustomAllReduceV2.should_custom_ar`。它接收 `inp`，主要用于将数据转换为另一种表示。

### Lines 129-136: Method CustomAllReduceV2.custom_all_reduce
```python
    def custom_all_reduce(self, input: torch.Tensor) -> torch.Tensor:
        if is_in_piecewise_cuda_graph():  # disable inplace optimization
            try:
                self.obj.set_cuda_graph_capture(False)
                return self._all_reduce(input)
            finally:
                self.obj.set_cuda_graph_capture(True)
        return self._all_reduce(input)
```
**EN:** This callable implements `CustomAllReduceV2.custom_all_reduce`. It takes `input` and mainly converts data into another representation. In this range it coordinates distributed communication; manages graph capture or replay logic.
**CN:** 这一可调用对象实现了 `CustomAllReduceV2.custom_all_reduce`。它接收 `input`，主要用于将数据转换为另一种表示。 在这一范围内，它会协调分布式通信；管理图捕获或回放逻辑。

### Lines 138-140: Method CustomAllReduceV2.close
```python
    def close(self):
        if not self.disabled and hasattr(self, "obj"):
            self.obj.free(self.group)
```
**EN:** This callable implements `CustomAllReduceV2.close` and mainly implements close.
**CN:** 这一可调用对象实现了 `CustomAllReduceV2.close`，主要用于实现 close 相关逻辑。

### Lines 142-145: Method CustomAllReduceV2._all_reduce
```python
    def _all_reduce(self, input: torch.Tensor) -> torch.Tensor:
        """Perform the actual all-reduce via JIT kernel."""
        algo = self._determine_algo(input)
        return torch.from_dlpack(self.obj.all_reduce(input, algo))
```
**EN:** This callable implements `CustomAllReduceV2._all_reduce`. It takes `input` and mainly reduces or aggregates values. The docstring states: "Perform the actual all-reduce via JIT kernel." In this range it coordinates distributed communication.
**CN:** 这一可调用对象实现了 `CustomAllReduceV2._all_reduce`。它接收 `input`，主要用于归约或聚合数值。 在这一范围内，它会协调分布式通信。

### Lines 147-156: Method CustomAllReduceV2._determine_algo
```python
    def _determine_algo(self, input: torch.Tensor) -> AllReduceAlgo:
        if self.override_algo is not None:
            return self.override_algo
        input_bytes = input.numel() * input.element_size()
        if input_bytes <= self.config.one_shot_push_threshold:
            return AllReduceAlgo.ONE_SHOT_PUSH
        if input_bytes <= self.config.one_shot_pull_threshold:
            return AllReduceAlgo.ONE_SHOT_PULL
        else:
            return AllReduceAlgo.TWO_SHOT_PULL
```
**EN:** This callable implements `CustomAllReduceV2._determine_algo`. It takes `input` and mainly implements determine algo.
**CN:** 这一可调用对象实现了 `CustomAllReduceV2._determine_algo`。它接收 `input`，主要用于实现 determine algo 相关逻辑。

### Lines 158-163: Method CustomAllReduceV2._post_init_obj
```python
    def _post_init_obj(self):
        handles = [self.obj.share_storage()]
        result = self._share_list(handles)
        assert all(len(r) == 1 for r in result)
        result = [h[0] for h in result]
        self.obj.post_init(result)
```
**EN:** This callable implements `CustomAllReduceV2._post_init_obj` and mainly implements post init obj. In this range it performs defensive checks on invalid state.
**CN:** 这一可调用对象实现了 `CustomAllReduceV2._post_init_obj`，主要用于实现 post init obj 相关逻辑。 在这一范围内，它会对非法状态执行防御性检查。

### Lines 165-169: Method CustomAllReduceV2._share_list
```python
    def _share_list(self, input: List[T]) -> List[List[T]]:
        input_tensor = torch.tensor(input, dtype=torch.int64, device="cpu")
        gather_list = [torch.empty_like(input_tensor) for _ in range(self.world_size)]
        dist.all_gather(gather_list, input_tensor, group=self.group)
        return [g.tolist() for g in gather_list]
```
**EN:** This callable implements `CustomAllReduceV2._share_list`. It takes `input` and mainly implements share list.
**CN:** 这一可调用对象实现了 `CustomAllReduceV2._share_list`。它接收 `input`，主要用于实现 share list 相关逻辑。

### Lines 171-172: Method CustomAllReduceV2.__del__
```python
    def __del__(self):
        self.close()
```
**EN:** This callable implements `CustomAllReduceV2.__del__` and mainly implements del.
**CN:** 这一可调用对象实现了 `CustomAllReduceV2.__del__`，主要用于实现 del 相关逻辑。

### Lines 175-200: Function _init_config
```python
def _init_config():
    global THRESHOLD_2_SHOT_MAP
    KB, MB = 1024, 1024 * 1024

    if is_sm100_supported():
        # NOTE: This result is based on benchmarks on B200 GPUs
        THRESHOLD_2_SHOT_MAP = {
            2: ModeConfig(4 * MB, INF),
            3: ModeConfig(4 * MB, 4 * MB),
            4: ModeConfig(2 * MB, 2 * MB),
            5: ModeConfig(2 * MB, 2 * MB),
            6: ModeConfig(1 * MB, 1 * MB),
            7: ModeConfig(896 * KB, 896 * KB),
            8: ModeConfig(720 * KB, 720 * KB),
        }
    else:
        # NOTE: This result is based on benchmarks on H200 GPUs
        THRESHOLD_2_SHOT_MAP = {
            2: ModeConfig(2 * MB, INF),
            3: ModeConfig(512 * KB, 512 * KB),
            4: ModeConfig(384 * KB, 256 * KB),
            5: ModeConfig(256 * KB, 256 * KB),
            6: ModeConfig(192 * KB, 192 * KB),
            7: ModeConfig(192 * KB, 192 * KB),
            8: ModeConfig(160 * KB, 160 * KB),
        }
```
**EN:** This callable implements `_init_config` and mainly implements init config.
**CN:** 这一可调用对象实现了 `_init_config`，主要用于实现 init config 相关逻辑。

### Lines 201-204: Module-level constants and helpers
```python
    # TODO: tune on more GPUs, e.g A100


THRESHOLD_2_SHOT_MAP: Dict[int, ModeConfig] = {}
```
**EN:** This range organizes module-level state and shared setup.
**CN:** 这一段组织模块级状态与共享初始化逻辑。

## Key Concepts / 关键概念
- `T`: module constant or capability flag / 模块常量或能力标记
- `INF`: module constant or capability flag / 模块常量或能力标记
- `ModeConfig`: core class or state container / 核心类或状态容器
- `CustomAllReduceV2`: core class or state container / 核心类或状态容器
- `_init_config`: implements init config / 实现 init config 相关逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `contextlib`, `dataclasses`, `typing`
- **Third-party / 第三方**: `torch`, `torch.distributed`
- **Internal modules / 内部模块**: `sglang.jit_kernel.all_reduce`, `sglang.srt.distributed`, `sglang.srt.distributed.device_communicators.custom_all_reduce_utils`, `sglang.srt.utils`
