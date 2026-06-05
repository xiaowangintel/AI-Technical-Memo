# weight_checker.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/weight_checker.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable runtime utilities centered on `weight_checker` and the surrounding SGLang serving stack. / 提供围绕 `weight_checker` 及其所处 SGLang 服务栈的可复用运行时工具。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Module setup and shared state / 模块设置与共享状态
```python
import logging
import time
from typing import Dict, Iterable, Optional, Set, Tuple

import torch
import torch.distributed as dist
from pydantic import BaseModel, ConfigDict

from sglang.srt.layers.quantization.fp8_utils import (
    block_quant_dequant,
    inverse_transform_scale_ue8m0,
)
from sglang.srt.managers.mm_utils import tensor_hash

logger = logging.getLogger(__name__)
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `logging`, `time`, `typing`, `torch`, `torch.distributed`, `pydantic`. It also defines symbols such as `logger` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `logging`, `time`, `typing`, `torch`, `torch.distributed`, `pydantic`。 同时定义了 `logger` 等符号，供后续逻辑使用。

### Lines 18-19: Class `_StrictBaseModel` declaration / 类 `_StrictBaseModel` 声明
```python
class _StrictBaseModel(BaseModel):
    model_config = ConfigDict(extra="forbid")
```
**EN:** This class establishes `_StrictBaseModel` as the main container/coordinator for the surrounding logic. It inherits from `BaseModel`. The main stored fields include `model_config`.
**CN:** 该类将 `_StrictBaseModel` 定义为周边逻辑的主要封装体或协调者。 它继承自 `BaseModel`。 其主要存储字段包括 `model_config`。

### Lines 22-30: Class `ParallelismInfo` declaration / 类 `ParallelismInfo` 声明
```python
class ParallelismInfo(_StrictBaseModel):
    tp_rank: int
    tp_size: int
    dp_rank: int
    dp_size: int
    pp_rank: int
    pp_size: int
    rank: int
    size: int
```
**EN:** This class establishes `ParallelismInfo` as the main container/coordinator for the surrounding logic. It inherits from `_StrictBaseModel`. The main stored fields include `tp_rank`, `tp_size`, `dp_rank`, `dp_size`, `pp_rank`, `pp_size`.
**CN:** 该类将 `ParallelismInfo` 定义为周边逻辑的主要封装体或协调者。 它继承自 `_StrictBaseModel`。 其主要存储字段包括 `tp_rank`, `tp_size`, `dp_rank`, `dp_size`, `pp_rank`, `pp_size`。

### Lines 33-35: Class `ChecksumInfo` declaration / 类 `ChecksumInfo` 声明
```python
class ChecksumInfo(_StrictBaseModel):
    checksums: Dict[str, str]
    parallelism_info: ParallelismInfo
```
**EN:** This class establishes `ChecksumInfo` as the main container/coordinator for the surrounding logic. It inherits from `_StrictBaseModel`. The main stored fields include `checksums`, `parallelism_info`.
**CN:** 该类将 `ChecksumInfo` 定义为周边逻辑的主要封装体或协调者。 它继承自 `_StrictBaseModel`。 其主要存储字段包括 `checksums`, `parallelism_info`。

### Lines 38-43: Constants and shared state / 常量与共享状态
```python
_NON_PERSISTENT_BUFFER_PATTERNS = (
    "cos_sin_cache",
    "inv_freq",
    "freqs_cis",
    "_weight_fp32",
)
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `_NON_PERSISTENT_BUFFER_PATTERNS`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `_NON_PERSISTENT_BUFFER_PATTERNS`。

### Lines 46-47: Function `_is_non_persistent_buffer_name` / 函数 `_is_non_persistent_buffer_name`
```python
def _is_non_persistent_buffer_name(name: str) -> bool:
    return any(pat in name for pat in _NON_PERSISTENT_BUFFER_PATTERNS)
```
**EN:** This function implements `_is_non_persistent_buffer_name`. It primarily calls `any` to complete its work.
**CN:** 该函数实现了 `_is_non_persistent_buffer_name`。 它主要通过调用 `any` 来完成任务。

### Lines 50-50: Class `WeightChecker` declaration / 类 `WeightChecker` 声明
```python
class WeightChecker:
```
**EN:** This class establishes `WeightChecker` as the main container/coordinator for the surrounding logic. Its core interface includes methods such as `__init__`, `handle`, `_snapshot`, `_reset_tensors`, `_compare`, `_compute_checksum`.
**CN:** 该类将 `WeightChecker` 定义为周边逻辑的主要封装体或协调者。 其核心接口包括 `__init__`, `handle`, `_snapshot`, `_reset_tensors`, `_compare`, `_compute_checksum` 等方法。

### Lines 51-53: Method `WeightChecker.__init__` / 方法 `WeightChecker.__init__`
```python
    def __init__(self, model_runner):
        self._model_runner = model_runner
        self._snapshot_tensors = None
```
**EN:** This method implements `__init__` on `WeightChecker`. State updates are written into `self._model_runner`, `self._snapshot_tensors`.
**CN:** 该方法（属于 `WeightChecker`）实现了 `__init__`。 状态更新主要写入 `self._model_runner`, `self._snapshot_tensors`。

### Lines 55-66: Method `WeightChecker.handle` / 方法 `WeightChecker.handle`
```python
    def handle(self, action: str) -> Optional[Dict]:
        logger.info(f"[WeightChecker] handle action={action}")
        if action == "snapshot":
            return self._snapshot()
        elif action == "reset_tensors":
            return self._reset_tensors()
        elif action == "compare":
            return self._compare()
        elif action == "checksum":
            return self._compute_checksum()
        else:
            raise Exception(f"Unsupported {action=}")
```
**EN:** This method implements `handle` on `WeightChecker`. It primarily calls `logger.info`, `self._snapshot`, `self._reset_tensors`, `self._compare`, `self._compute_checksum`, `Exception` to complete its work. The implementation relies on conditional branches.
**CN:** 该方法（属于 `WeightChecker`）实现了 `handle`。 它主要通过调用 `logger.info`, `self._snapshot`, `self._reset_tensors`, `self._compare`, `self._compute_checksum`, `Exception` 来完成任务。 实现中使用了条件分支。

### Lines 68-75: Method `WeightChecker._snapshot` / 方法 `WeightChecker._snapshot`
```python
    def _snapshot(self):
        named_tensors = [
            (name, param.data.detach().cpu()) for name, param in self._model_state()
        ]
        self._snapshot_tensors = dict(named_tensors)
        assert len(self._snapshot_tensors) == len(
            named_tensors
        ), f"should not have duplicated tensor name"
```
**EN:** This method implements `_snapshot` on `WeightChecker`. It primarily calls `dict`, `len`, `param.data.detach.cpu`, `self._model_state`, `param.data.detach` to complete its work. State updates are written into `named_tensors`, `self._snapshot_tensors`.
**CN:** 该方法（属于 `WeightChecker`）实现了 `_snapshot`。 它主要通过调用 `dict`, `len`, `param.data.detach.cpu`, `self._model_state`, `param.data.detach` 来完成任务。 状态更新主要写入 `named_tensors`, `self._snapshot_tensors`。

### Lines 77-81: Method `WeightChecker._reset_tensors` / 方法 `WeightChecker._reset_tensors`
```python
    def _reset_tensors(self):
        for name, param in self._model_state():
            if _is_non_persistent_buffer_name(name):
                continue
            param.copy_(_random_like(param))
```
**EN:** This method implements `_reset_tensors` on `WeightChecker`. It primarily calls `self._model_state`, `_is_non_persistent_buffer_name`, `param.copy_`, `_random_like` to complete its work. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `WeightChecker`）实现了 `_reset_tensors`。 它主要通过调用 `self._model_state`, `_is_non_persistent_buffer_name`, `param.copy_`, `_random_like` 来完成任务。 实现中使用了条件分支、迭代逻辑。

### Lines 83-98: Method `WeightChecker._compare` / 方法 `WeightChecker._compare`
```python
    def _compare(self):
        assert self._snapshot_tensors is not None

        skip_compare_names = {
            name
            for name, param in self._model_state()
            if getattr(param, "_skip_weight_check", False)
        }
        _check_tensors(
            expect_tensors=_postprocess_tensors(
                self._snapshot_tensors, skip_compare_names
            ),
            actual_tensors=_postprocess_tensors(
                dict(self._model_state()), skip_compare_names
            ),
        )
```
**EN:** This method implements `_compare` on `WeightChecker`. It primarily calls `_check_tensors`, `self._model_state`, `getattr`, `_postprocess_tensors`, `dict` to complete its work. State updates are written into `skip_compare_names`.
**CN:** 该方法（属于 `WeightChecker`）实现了 `_compare`。 它主要通过调用 `_check_tensors`, `self._model_state`, `getattr`, `_postprocess_tensors`, `dict` 来完成任务。 状态更新主要写入 `skip_compare_names`。

### Lines 100-131: Method `WeightChecker._compute_checksum` / 方法 `WeightChecker._compute_checksum`
```python
    def _compute_checksum(self) -> Dict:
        torch.cuda.synchronize()
        start = time.perf_counter()

        skip_compare_names = {
            name
            for name, param in self._model_state()
            if getattr(param, "_skip_weight_check", False)
        }

        # Reuse the snapshot/compare postprocess pipeline so fp8 weights are
        # dequantized to bf16 before hashing — two (qweight, scale) pairs that
        # produce the same bf16 must produce the same checksum.
        checksums = {
            name: _hash_tensor(tensor.data)
            for name, should_compare, tensor in _postprocess_tensors(
                dict(self._model_state()), skip_compare_names
            )
            if should_compare
        }

        torch.cuda.synchronize()
        elapsed = time.perf_counter() - start
        logger.info(
            f"[WeightChecker] checksum computed for {len(checksums)} tensors in {elapsed:.3f}s"
        )

        info = ChecksumInfo(
            checksums=checksums,
            parallelism_info=self._parallelism_info(),
        )
        return info.model_dump()
```
**EN:** This method implements `_compute_checksum` on `WeightChecker`. It primarily calls `torch.cuda.synchronize`, `time.perf_counter`, `logger.info`, `ChecksumInfo`, `info.model_dump`, `_hash_tensor` to complete its work. State updates are written into `start`, `skip_compare_names`, `checksums`, `elapsed`, `info`.
**CN:** 该方法（属于 `WeightChecker`）实现了 `_compute_checksum`。 它主要通过调用 `torch.cuda.synchronize`, `time.perf_counter`, `logger.info`, `ChecksumInfo`, `info.model_dump`, `_hash_tensor` 来完成任务。 状态更新主要写入 `start`, `skip_compare_names`, `checksums`, `elapsed`, `info`。

### Lines 133-144: Method `WeightChecker._parallelism_info` / 方法 `WeightChecker._parallelism_info`
```python
    def _parallelism_info(self) -> ParallelismInfo:
        mr = self._model_runner
        return ParallelismInfo(
            tp_rank=mr.tp_rank,
            tp_size=mr.tp_size,
            dp_rank=mr.dp_rank if mr.dp_rank is not None else 0,
            dp_size=mr.dp_size,
            pp_rank=mr.pp_rank,
            pp_size=mr.pp_size,
            rank=dist.get_rank() if dist.is_initialized() else 0,
            size=dist.get_world_size() if dist.is_initialized() else 1,
        )
```
**EN:** This method implements `_parallelism_info` on `WeightChecker`. It primarily calls `ParallelismInfo`, `dist.is_initialized`, `dist.get_rank`, `dist.get_world_size` to complete its work. State updates are written into `mr`.
**CN:** 该方法（属于 `WeightChecker`）实现了 `_parallelism_info`。 它主要通过调用 `ParallelismInfo`, `dist.is_initialized`, `dist.get_rank`, `dist.get_world_size` 来完成任务。 状态更新主要写入 `mr`。

### Lines 146-149: Method `WeightChecker._model_state` / 方法 `WeightChecker._model_state`
```python
    def _model_state(self):
        # TODO: support EAGLE etc (e.g. yield from both main model and draft model)
        yield from self._model_runner.model.named_parameters()
        yield from self._model_runner.model.named_buffers()
```
**EN:** This method implements `_model_state` on `WeightChecker`. It primarily calls `self._model_runner.model.named_parameters`, `self._model_runner.model.named_buffers` to complete its work.
**CN:** 该方法（属于 `WeightChecker`）实现了 `_model_state`。 它主要通过调用 `self._model_runner.model.named_parameters`, `self._model_runner.model.named_buffers` 来完成任务。

### Lines 152-153: Function `_hash_tensor` / 函数 `_hash_tensor`
```python
def _hash_tensor(t: torch.Tensor) -> str:
    return f"{tensor_hash(t):016x}"
```
**EN:** This function implements `_hash_tensor`. It primarily calls `tensor_hash` to complete its work.
**CN:** 该函数实现了 `_hash_tensor`。 它主要通过调用 `tensor_hash` 来完成任务。

### Lines 156-198: Function `_check_tensors` / 函数 `_check_tensors`
```python
def _check_tensors(
    expect_tensors: Iterable[Tuple[str, bool, torch.Tensor]],
    actual_tensors: Iterable[Tuple[str, bool, torch.Tensor]],
):
    from sglang.srt.debug_utils.dumper import get_tensor_info

    good_names = []
    error_messages = []
    info_messages = []

    for (expect_name, expect_should_compare, expect), (
        actual_name,
        actual_should_compare,
        actual,
    ) in zip(expect_tensors, actual_tensors, strict=True):
        assert expect_name == actual_name, f"{expect_name=} {actual_name=}"
        assert (
            expect_should_compare == actual_should_compare
        ), f"{expect_should_compare=} {actual_should_compare=}"
        name = expect_name
        should_compare = expect_should_compare

        expect = expect.cuda()
        actual = actual.cuda()

        if torch.all(expect == actual):
            good_names.append(name)
        else:
# ... omitted for brevity ...

    logger.info(f"[check_tensors] equal tensors: {good_names}")
    if len(info_messages) > 0:
        logger.info(f"[check_tensors] info: {info_messages}")
    if len(error_messages) > 0:
        raise Exception(f"check tensor equality failed:\n" + "\n".join(error_messages))
```
**EN:** This function implements `_check_tensors`. It primarily calls `zip`, `logger.info`, `expect.cuda`, `actual.cuda`, `torch.all`, `len` to complete its work. State updates are written into `good_names`, `error_messages`, `info_messages`, `name`, `should_compare`, `expect`. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `_check_tensors`。 它主要通过调用 `zip`, `logger.info`, `expect.cuda`, `actual.cuda`, `torch.all`, `len` 来完成任务。 状态更新主要写入 `good_names`, `error_messages`, `info_messages`, `name`, `should_compare`, `expect`。 实现中使用了条件分支、迭代逻辑。

### Lines 201-215: Function `_random_like` / 函数 `_random_like`
```python
def _random_like(t: torch.Tensor):
    device = t.device
    shape = t.shape
    dtype = t.dtype

    if dtype.is_floating_point:
        return torch.rand(shape, device=device, dtype=torch.float32).to(dtype)

    if dtype == torch.bool:
        return torch.rand(shape, device=device) > 0.5

    info = torch.iinfo(dtype)
    return torch.randint(
        low=int(info.min), high=int(info.max), size=shape, device=device, dtype=dtype
    )
```
**EN:** This function implements `_random_like`. It primarily calls `torch.iinfo`, `torch.randint`, `torch.rand.to`, `torch.rand`, `int` to complete its work. State updates are written into `device`, `shape`, `dtype`, `info`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_random_like`。 它主要通过调用 `torch.iinfo`, `torch.randint`, `torch.rand.to`, `torch.rand`, `int` 来完成任务。 状态更新主要写入 `device`, `shape`, `dtype`, `info`。 实现中使用了条件分支。

### Lines 218-272: Function `_postprocess_tensors` / 函数 `_postprocess_tensors`
```python
def _postprocess_tensors(
    raw: Dict[str, torch.Tensor],
    skip_compare_names: Set[str],
) -> Iterable[Tuple[str, bool, torch.Tensor]]:
    from sglang.srt.debug_utils.dumper import get_tensor_info

    skip_compare_names = set(skip_compare_names)

    # Skip non-persistent buffers (registered with persistent=False; recomputed
    # after weight load and not part of the synced payload).
    for name in raw:
        if _is_non_persistent_buffer_name(name):
            skip_compare_names.add(name)
            logger.info(f"[check_tensors] Skipping non-persistent buffer: {name}")

    # dequant fp8
    quant_names = [
        name
        for name in raw
        # Match: `something.weight`, `something.experts.w2_weight`
        if name.endswith("weight") and name.replace("weight", "weight_scale_inv") in raw
    ]
    quant_scale_names = [
        name.replace("weight", "weight_scale_inv") for name in quant_names
    ]
    skip_compare_names.update(quant_names)
    skip_compare_names.update(quant_scale_names)
    for name in quant_names:
# ... omitted for brevity ...
            )
            raise

    for name in raw:
        should_compare = name not in skip_compare_names
        yield name, should_compare, raw[name]
```
**EN:** This function implements `_postprocess_tensors`. It primarily calls `set`, `skip_compare_names.update`, `_is_non_persistent_buffer_name`, `name.replace`, `skip_compare_names.add`, `logger.info` to complete its work. State updates are written into `skip_compare_names`, `quant_names`, `quant_scale_names`, `w_q`, `w_s`, `should_compare`. The implementation relies on conditional branches, iteration, error handling.
**CN:** 该函数实现了 `_postprocess_tensors`。 它主要通过调用 `set`, `skip_compare_names.update`, `_is_non_persistent_buffer_name`, `name.replace`, `skip_compare_names.add`, `logger.info` 来完成任务。 状态更新主要写入 `skip_compare_names`, `quant_names`, `quant_scale_names`, `w_q`, `w_s`, `should_compare`。 实现中使用了条件分支、迭代逻辑、错误处理。

## Key Concepts / 关键概念
- **Classes / 类**: `_StrictBaseModel`, `ParallelismInfo`, `ChecksumInfo`, `WeightChecker`
- **Functions / 函数**: `_is_non_persistent_buffer_name`, `_hash_tensor`, `_check_tensors`, `_random_like`, `_postprocess_tensors`, `__init__`, `handle`, `_snapshot`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.layers.quantization.fp8_utils`, `sglang.srt.managers.mm_utils`, `sglang.srt.debug_utils.dumper`
- **External / 外部依赖**: `torch`, `torch.distributed`, `pydantic`
- **Standard library / 标准库**: `logging`, `time`, `typing`
