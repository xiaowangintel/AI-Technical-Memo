# spec_info.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/speculative/spec_info.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file implements speculative decoding infrastructure, such as workers, metadata objects, CUDA-graph runners, or shared utilities. It coordinates draft-model execution with target-model verification to improve throughput. / 该文件实现了推测解码基础设施，例如 worker、元数据对象、CUDA Graph 运行器或共享工具函数。它通过协调草稿模型执行与目标模型校验来提升吞吐。

## Line-by-Line Analysis / 逐行分析

### Lines 1-27: Module header, imports, and shared constants
```python
from __future__ import annotations

from abc import ABC, abstractmethod
from enum import Enum, IntEnum, auto
from typing import TYPE_CHECKING, Callable, List, Optional, Tuple, Type, Union

import torch

from sglang.srt.speculative.spec_registry import (
    CustomSpecAlgo,
    ServerArgsValidator,
    WorkerFactory,
)
from sglang.srt.speculative.spec_registry import get_spec as _get_registered_spec
from sglang.srt.speculative.spec_registry import (
    register_algorithm as _register_algorithm,
)

if TYPE_CHECKING:
    from sglang.srt.managers.overlap_utils import FutureMap
    from sglang.srt.managers.schedule_batch import ScheduleBatch
    from sglang.srt.managers.tp_worker import TpModelWorker
    from sglang.srt.server_args import ServerArgs
    from sglang.srt.speculative.base_spec_worker import BaseSpecWorker
    from sglang.srt.speculative.ngram_worker import NGRAMWorker
```
**EN:** This block prepares the module by defining the file header, imports, and shared constants that later logic depends on.
**CN:** 该代码块通过定义文件头、导入项以及后续逻辑依赖的共享常量，为模块完成准备工作。

### Lines 28-42: SpeculativeAlgorithm class declaration
```python
class SpeculativeAlgorithm(Enum):
    """Builtin speculative decoding algorithms. Plugin-registered ones are
    ``CustomSpecAlgo`` instances; ``from_string`` returns either type, and
    both expose the same ``is_*()`` / ``create_worker`` interface so callers
    dispatch uniformly without isinstance checks.
    """

    DFLASH = auto()
    EAGLE = auto()
    EAGLE3 = auto()
    FROZEN_KV_MTP = auto()
    STANDALONE = auto()
    NGRAM = auto()
    NONE = auto()
```
**EN:** This block declares the `SpeculativeAlgorithm` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `SpeculativeAlgorithm` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 43-57: SpeculativeAlgorithm.from_string method
```python
    @classmethod
    def from_string(
        cls, name: Optional[str]
    ) -> Union[SpeculativeAlgorithm, CustomSpecAlgo]:
        if name is None:
            return cls.NONE
        upper = name.upper()
        try:
            return cls[upper]
        except KeyError:
            pass
        spec = _get_registered_spec(upper)
        if spec is not None:
            return spec
        raise ValueError(f"Unknown speculative algorithm name: {name}")
```
**EN:** This block uses `SpeculativeAlgorithm.from_string` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `SpeculativeAlgorithm.from_string` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 59-83: SpeculativeAlgorithm.register method
```python
    @classmethod
    def register(
        cls,
        name: str,
        *,
        supports_overlap: bool = False,
        validate_server_args: Optional[ServerArgsValidator] = None,
        spec_class: Type[CustomSpecAlgo] = CustomSpecAlgo,
    ) -> Callable[[WorkerFactory], WorkerFactory]:
        """Decorator to register a plugin speculative algorithm. The factory
        takes ``server_args`` and returns the worker class. Pass a
        ``CustomSpecAlgo`` subclass via ``spec_class`` to override any
        ``is_*()`` / ``create_worker`` method.

        Example:
            @SpeculativeAlgorithm.register("MY_SPEC", supports_overlap=False)
            def _factory(server_args):
                return MySpecWorker
        """
        return _register_algorithm(
            name,
            supports_overlap=supports_overlap,
            validate_server_args=validate_server_args,
            spec_class=spec_class,
        )
```
**EN:** This block uses `SpeculativeAlgorithm.register` to register components for later lookup. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `SpeculativeAlgorithm.register` 来注册组件以供后续查找。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 85-86: SpeculativeAlgorithm.is_none method
```python
    def is_none(self) -> bool:
        return self == SpeculativeAlgorithm.NONE
```
**EN:** This block uses `SpeculativeAlgorithm.is_none` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `SpeculativeAlgorithm.is_none` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 88-89: SpeculativeAlgorithm.is_speculative method
```python
    def is_speculative(self) -> bool:
        return self != SpeculativeAlgorithm.NONE
```
**EN:** This block uses `SpeculativeAlgorithm.is_speculative` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `SpeculativeAlgorithm.is_speculative` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 91-98: SpeculativeAlgorithm.is_eagle method
```python
    def is_eagle(self) -> bool:
        # FIXME(kpham_sgl): Remove FROZEN_KV_MTP here once we
        # have established support for it in the scheduler.
        return self in (
            SpeculativeAlgorithm.EAGLE,
            SpeculativeAlgorithm.EAGLE3,
            SpeculativeAlgorithm.FROZEN_KV_MTP,
        )
```
**EN:** This block uses `SpeculativeAlgorithm.is_eagle` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `SpeculativeAlgorithm.is_eagle` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 100-101: SpeculativeAlgorithm.is_eagle3 method
```python
    def is_eagle3(self) -> bool:
        return self == SpeculativeAlgorithm.EAGLE3
```
**EN:** This block uses `SpeculativeAlgorithm.is_eagle3` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `SpeculativeAlgorithm.is_eagle3` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 103-104: SpeculativeAlgorithm.is_frozen_kv_mtp method
```python
    def is_frozen_kv_mtp(self) -> bool:
        return self == SpeculativeAlgorithm.FROZEN_KV_MTP
```
**EN:** This block uses `SpeculativeAlgorithm.is_frozen_kv_mtp` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `SpeculativeAlgorithm.is_frozen_kv_mtp` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 106-107: SpeculativeAlgorithm.is_dflash method
```python
    def is_dflash(self) -> bool:
        return self == SpeculativeAlgorithm.DFLASH
```
**EN:** This block uses `SpeculativeAlgorithm.is_dflash` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `SpeculativeAlgorithm.is_dflash` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 109-110: SpeculativeAlgorithm.is_standalone method
```python
    def is_standalone(self) -> bool:
        return self == SpeculativeAlgorithm.STANDALONE
```
**EN:** This block uses `SpeculativeAlgorithm.is_standalone` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `SpeculativeAlgorithm.is_standalone` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 112-113: SpeculativeAlgorithm.is_ngram method
```python
    def is_ngram(self) -> bool:
        return self == SpeculativeAlgorithm.NGRAM
```
**EN:** This block uses `SpeculativeAlgorithm.is_ngram` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `SpeculativeAlgorithm.is_ngram` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 115-116: SpeculativeAlgorithm.supports_target_verify_for_draft method
```python
    def supports_target_verify_for_draft(self) -> bool:
        return self.is_dflash()
```
**EN:** This block uses `SpeculativeAlgorithm.supports_target_verify_for_draft` to verify draft results against the target model. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `SpeculativeAlgorithm.supports_target_verify_for_draft` 来根据目标模型校验草稿结果。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 118-133: SpeculativeAlgorithm.create_future_map method
```python
    def create_future_map(
        self,
        max_running_requests: int,
        chunked_prefill_size: int,
        context_len: int,
        device: torch.device,
    ) -> FutureMap:
        from sglang.srt.managers.overlap_utils import FutureMap

        return FutureMap(
            max_running_requests,
            chunked_prefill_size,
            context_len,
            device,
            self,
        )
```
**EN:** This block uses `SpeculativeAlgorithm.create_future_map` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `SpeculativeAlgorithm.create_future_map` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 135-136: SpeculativeAlgorithm.supports_spec_v2 method
```python
    def supports_spec_v2(self) -> bool:
        return (self.is_eagle() and not self.is_frozen_kv_mtp()) or self.is_standalone()
```
**EN:** This block uses `SpeculativeAlgorithm.supports_spec_v2` to implement the core logic for this symbol. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `SpeculativeAlgorithm.supports_spec_v2` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 138-197: SpeculativeAlgorithm.create_worker method (part 1/2)
```python
    def create_worker(
        self, server_args: ServerArgs
    ) -> Optional[Union[Type[BaseSpecWorker], Type[TpModelWorker], Type[NGRAMWorker]]]:
        assert (
            not self.is_none()
        ), "Cannot create worker for NONE speculative algorithm."

        enable_overlap = not server_args.disable_overlap_schedule

        if self.is_dflash():
            if enable_overlap:
                raise ValueError(
                    "DFLASH does not support overlap scheduling (spec v2)."
                )
            from sglang.srt.speculative.dflash_worker import DFlashWorker

            return DFlashWorker

        if self.is_frozen_kv_mtp():
            if enable_overlap:
                raise ValueError(
                    "FROZEN_KV_MTP does not support spec v2. Disable overlap "
                    "scheduling to use FrozenKVMTPWorker."
                )

            from sglang.srt.speculative.frozen_kv_mtp_worker import (
                FrozenKVMTPWorker,
            )

            return FrozenKVMTPWorker

        if self.is_eagle() and server_args.enable_multi_layer_eagle:
            # FIXME: migrate to EagleWorker
            if enable_overlap:
                from sglang.srt.speculative.multi_layer_eagle_worker_v2 import (
                    MultiLayerEagleWorkerV2,
                )

                return MultiLayerEagleWorkerV2

            from sglang.srt.speculative.multi_layer_eagle_worker import (
                MultiLayerEagleWorker,
            )

            return MultiLayerEagleWorker

        elif self.is_eagle():
            if enable_overlap:
                from sglang.srt.speculative.eagle_worker_v2 import EAGLEWorkerV2

                return EAGLEWorkerV2

            from sglang.srt.speculative.eagle_worker import EAGLEWorker

            return EAGLEWorker
        elif self.is_standalone():
            if enable_overlap:
                from sglang.srt.speculative.standalone_worker_v2 import (
                    StandaloneWorkerV2,
                )
```
**EN:** This block uses `SpeculativeAlgorithm.create_worker` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the beginning of the routine and its initial control flow.
**CN:** 该代码块通过 `SpeculativeAlgorithm.create_worker` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的开头及其初始控制流。

### Lines 198-214: SpeculativeAlgorithm.create_worker method (part 2/2)
```python

                return StandaloneWorkerV2

            from sglang.srt.speculative.standalone_worker import StandaloneWorker

            return StandaloneWorker
        elif self.is_ngram():
            if enable_overlap:
                raise ValueError(
                    f"Speculative algorithm {self.name} does not support overlap worker creation."
                )

            from sglang.srt.speculative.ngram_worker import NGRAMWorker

            return NGRAMWorker

        raise ValueError("Unreachable code path in create_worker.")
```
**EN:** This block uses `SpeculativeAlgorithm.create_worker` to create runtime objects or tensors. It is part of the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or returned values.
**CN:** 该代码块通过 `SpeculativeAlgorithm.create_worker` 来创建运行时对象或张量。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程的后段，包括最终状态更新或返回值。

### Lines 217-228: SpecInputType class declaration
```python
class SpecInputType(IntEnum):
    # NOTE: introduce this to distinguish the SpecInput types of multiple algorithms when asserting in attention backends.
    # If all algorithms can share the same datastrucutre of draft_input and verify_input, consider simplify it
    EAGLE_DRAFT = auto()
    EAGLE_DRAFT_EXTEND = auto()
    EAGLE_VERIFY = auto()
    FROZEN_KV_MTP_DRAFT = auto()
    FROZEN_KV_MTP_DRAFT_EXTEND = auto()
    FROZEN_KV_MTP_VERIFY = auto()
    DFLASH_DRAFT = auto()
    DFLASH_VERIFY = auto()
    NGRAM_VERIFY = auto()
```
**EN:** This block declares the `SpecInputType` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `SpecInputType` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 231-231: SpecInput class declaration
```python
class SpecInput(ABC):
```
**EN:** This block declares the `SpecInput` class, which exists to implement the core logic for this symbol. It defines the structural contract and member layout used by the rest of the file.
**CN:** 该代码块声明了 `SpecInput` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分所依赖的结构约定与成员布局。

### Lines 232-233: SpecInput initializer
```python
    def __init__(self, spec_input_type: SpecInputType):
        self.spec_input_type = spec_input_type
```
**EN:** This block initializes the `SpecInput` object, setting up the state, buffers, and references that later methods rely on.
**CN:** 该代码块初始化 `SpecInput` 对象，建立后续方法依赖的状态、缓冲区与引用。

### Lines 235-244: SpecInput.is_draft_input method
```python
    def is_draft_input(self) -> bool:
        # FIXME: remove this function which is only used for assertion
        # or use another variable name like `draft_input` to substitute `spec_info`
        return self.spec_input_type in {
            SpecInputType.EAGLE_DRAFT,
            SpecInputType.EAGLE_DRAFT_EXTEND,
            SpecInputType.FROZEN_KV_MTP_DRAFT,
            SpecInputType.FROZEN_KV_MTP_DRAFT_EXTEND,
            SpecInputType.DFLASH_DRAFT,
        }
```
**EN:** This block uses `SpecInput.is_draft_input` to produce draft tokens or draft-side state. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `SpecInput.is_draft_input` 来生成草稿 token 或草稿侧状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 246-252: SpecInput.is_verify_input method
```python
    def is_verify_input(self) -> bool:
        return self.spec_input_type in {
            SpecInputType.EAGLE_VERIFY,
            SpecInputType.FROZEN_KV_MTP_VERIFY,
            SpecInputType.DFLASH_VERIFY,
            SpecInputType.NGRAM_VERIFY,
        }
```
**EN:** This block uses `SpecInput.is_verify_input` to verify draft results against the target model. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `SpecInput.is_verify_input` 来根据目标模型校验草稿结果。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 254-256: SpecInput.get_spec_adjust_token_coefficient method
```python
    @abstractmethod
    def get_spec_adjust_token_coefficient(self) -> Tuple[int, int]:
        pass
```
**EN:** This block uses `SpecInput.get_spec_adjust_token_coefficient` to fetch or compute a value. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `SpecInput.get_spec_adjust_token_coefficient` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 258-266: SpecInput.get_spec_adjusted_global_num_tokens method
```python
    def get_spec_adjusted_global_num_tokens(
        self, batch: ScheduleBatch
    ) -> Tuple[List[int], List[int]]:
        c1, c2 = self.get_spec_adjust_token_coefficient()
        global_num_tokens = [x * c1 for x in batch.global_num_tokens]
        global_num_tokens_for_logprob = [
            x * c2 for x in batch.global_num_tokens_for_logprob
        ]
        return global_num_tokens, global_num_tokens_for_logprob
```
**EN:** This block uses `SpecInput.get_spec_adjusted_global_num_tokens` to fetch or compute a value. It is part of the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `SpecInput.get_spec_adjusted_global_num_tokens` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

## Key Concepts / 关键概念
- Speculative decoding / 推测解码
- Tensor computation with PyTorch / 基于 PyTorch 的张量计算
- Integration with model workers / 与模型 worker 集成

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.managers.overlap_utils`
- `sglang.srt.managers.schedule_batch`
- `sglang.srt.managers.tp_worker`
- `sglang.srt.server_args`
- `sglang.srt.speculative.base_spec_worker`
- `sglang.srt.speculative.dflash_worker`
- `sglang.srt.speculative.eagle_worker`
- `sglang.srt.speculative.eagle_worker_v2`
- `sglang.srt.speculative.frozen_kv_mtp_worker`
- `sglang.srt.speculative.multi_layer_eagle_worker`
- `sglang.srt.speculative.multi_layer_eagle_worker_v2`
- `sglang.srt.speculative.ngram_worker`
- `sglang.srt.speculative.spec_registry`
- `sglang.srt.speculative.standalone_worker`
- `sglang.srt.speculative.standalone_worker_v2`
### External / 外部
- `__future__`
- `torch`
- `abc` (stdlib)
- `enum` (stdlib)
- `typing` (stdlib)
