# patch_torch.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/patch_torch.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable SGLang runtime helpers for PyTorch integration helpers. / 为 SGLang 运行时提供面向PyTorch 集成辅助逻辑的可复用辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 14-38: Module setup and shared state / 模块设置与共享状态
```python
from typing import Callable, Union

import torch
from torch.multiprocessing import reductions

from sglang.srt.utils.common import is_musa, is_npu, torch_release

_is_npu = is_npu()
_is_musa = is_musa()

if _is_npu:
    from torch_npu.multiprocessing import reductions as npu_reductions

    def _rebuild_npu_tensor_modified(*args):
        args = _modify_tuple(args, _REDUCE_TENSOR_ARG_DEVICE_INDEX, npu_verl_to_sglang)
        return npu_reductions._rebuild_npu_tensor_original(*args)

    def npu_verl_to_sglang(device: int):
        assert (
            SGLANG_TP_RANK is not None
        ), "SGLANG_TP_RANK is not registered. Please call register_sgl_tp_rank() first."
        return SGLANG_TP_RANK


SGLANG_TP_RANK = None
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `typing`, `torch`, `torch.multiprocessing`, `sglang.srt.utils.common`. It also defines symbols such as `_is_npu`, `_is_musa`, `args`, `SGLANG_TP_RANK` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `typing`, `torch`, `torch.multiprocessing`, `sglang.srt.utils.common`。 同时定义了 `_is_npu`, `_is_musa`, `args`, `SGLANG_TP_RANK` 等符号，供后续逻辑使用。

### Lines 41-59: Function `monkey_patch_torch_reductions` / 函数 `monkey_patch_torch_reductions`
```python
def monkey_patch_torch_reductions():
    """Monkey patching before Torch https://github.com/pytorch/pytorch/pull/149248 is fixed"""

    if not _is_npu:
        if hasattr(reductions, "_reduce_tensor_original"):
            return
        reductions._reduce_tensor_original = reductions.reduce_tensor
        reductions._rebuild_cuda_tensor_original = reductions.rebuild_cuda_tensor

        reductions.reduce_tensor = _reduce_tensor_modified
        reductions.rebuild_cuda_tensor = _rebuild_cuda_tensor_modified
        reductions.init_reductions()
    else:
        # FIXME: This is a temp patch for npu as HDK does not support device uuid for now
        if hasattr(npu_reductions, "_rebuild_npu_tensor_original"):
            return

        npu_reductions._rebuild_npu_tensor_original = npu_reductions.rebuild_npu_tensor
        npu_reductions.rebuild_npu_tensor = _rebuild_npu_tensor_modified
```
**EN:** This function implements `monkey_patch_torch_reductions`. It primarily calls `hasattr`, `reductions.init_reductions` to complete its work. State updates are written into `reductions._reduce_tensor_original`, `reductions._rebuild_cuda_tensor_original`, `reductions.reduce_tensor`, `reductions.rebuild_cuda_tensor`, `npu_reductions._rebuild_npu_tensor_original`, `npu_reductions.rebuild_npu_tensor`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `monkey_patch_torch_reductions`。 它主要通过调用 `hasattr`, `reductions.init_reductions` 来完成任务。 状态更新主要写入 `reductions._reduce_tensor_original`, `reductions._rebuild_cuda_tensor_original`, `reductions.reduce_tensor`, `reductions.rebuild_cuda_tensor`, `npu_reductions._rebuild_npu_tensor_original`, `npu_reductions.rebuild_npu_tensor`。 实现中使用了条件分支。

### Lines 64-64: Constants and shared state / 常量与共享状态
```python
_REDUCE_TENSOR_ARG_DEVICE_INDEX = 6
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `_REDUCE_TENSOR_ARG_DEVICE_INDEX`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `_REDUCE_TENSOR_ARG_DEVICE_INDEX`。

### Lines 67-69: Function `register_sgl_tp_rank` / 函数 `register_sgl_tp_rank`
```python
def register_sgl_tp_rank(rank: int):
    global SGLANG_TP_RANK
    SGLANG_TP_RANK = rank
```
**EN:** This function implements `register_sgl_tp_rank`. State updates are written into `SGLANG_TP_RANK`.
**CN:** 该函数实现了 `register_sgl_tp_rank`。 状态更新主要写入 `SGLANG_TP_RANK`。

### Lines 72-77: Function `_reduce_tensor_modified` / 函数 `_reduce_tensor_modified`
```python
def _reduce_tensor_modified(*args, **kwargs):
    output_fn, output_args = reductions._reduce_tensor_original(*args, **kwargs)
    output_args = _modify_tuple(
        output_args, _REDUCE_TENSOR_ARG_DEVICE_INDEX, _device_to_uuid
    )
    return output_fn, output_args
```
**EN:** This function implements `_reduce_tensor_modified`. It primarily calls `reductions._reduce_tensor_original`, `_modify_tuple` to complete its work. State updates are written into `output_args`.
**CN:** 该函数实现了 `_reduce_tensor_modified`。 它主要通过调用 `reductions._reduce_tensor_original`, `_modify_tuple` 来完成任务。 状态更新主要写入 `output_args`。

### Lines 80-82: Function `_rebuild_cuda_tensor_modified` / 函数 `_rebuild_cuda_tensor_modified`
```python
def _rebuild_cuda_tensor_modified(*args):
    args = _modify_tuple(args, _REDUCE_TENSOR_ARG_DEVICE_INDEX, _device_from_maybe_uuid)
    return reductions._rebuild_cuda_tensor_original(*args)
```
**EN:** This function implements `_rebuild_cuda_tensor_modified`. It primarily calls `_modify_tuple`, `reductions._rebuild_cuda_tensor_original` to complete its work. State updates are written into `args`.
**CN:** 该函数实现了 `_rebuild_cuda_tensor_modified`。 它主要通过调用 `_modify_tuple`, `reductions._rebuild_cuda_tensor_original` 来完成任务。 状态更新主要写入 `args`。

### Lines 85-86: Function `_device_to_uuid` / 函数 `_device_to_uuid`
```python
def _device_to_uuid(device: int) -> str:
    return str(torch.cuda.get_device_properties(device).uuid)
```
**EN:** This function implements `_device_to_uuid`. It primarily calls `str`, `torch.cuda.get_device_properties` to complete its work.
**CN:** 该函数实现了 `_device_to_uuid`。 它主要通过调用 `str`, `torch.cuda.get_device_properties` 来完成任务。

### Lines 89-99: Function `_device_from_maybe_uuid` / 函数 `_device_from_maybe_uuid`
```python
def _device_from_maybe_uuid(device_maybe_uuid: Union[int, str]) -> int:
    if isinstance(device_maybe_uuid, int):
        return device_maybe_uuid

    if isinstance(device_maybe_uuid, str):
        for device in range(torch.cuda.device_count()):
            if str(torch.cuda.get_device_properties(device).uuid) == device_maybe_uuid:
                return device
        raise Exception("Invalid device_uuid=" + device_maybe_uuid)

    raise Exception(f"Unknown type: {device_maybe_uuid=}")
```
**EN:** This function implements `_device_from_maybe_uuid`. It primarily calls `isinstance`, `Exception`, `range`, `torch.cuda.device_count`, `str`, `torch.cuda.get_device_properties` to complete its work. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `_device_from_maybe_uuid`。 它主要通过调用 `isinstance`, `Exception`, `range`, `torch.cuda.device_count`, `str`, `torch.cuda.get_device_properties` 来完成任务。 实现中使用了条件分支、迭代逻辑。

### Lines 102-103: Function `_modify_tuple` / 函数 `_modify_tuple`
```python
def _modify_tuple(t, index: int, modifier: Callable):
    return *t[:index], modifier(t[index]), *t[index + 1 :]
```
**EN:** This function implements `_modify_tuple`. It primarily calls `modifier` to complete its work.
**CN:** 该函数实现了 `_modify_tuple`。 它主要通过调用 `modifier` 来完成任务。

### Lines 106-113: Function `monkey_patch_torch_compile` / 函数 `monkey_patch_torch_compile`
```python
def monkey_patch_torch_compile():
    if torch_release < (2, 8):
        # These things are cacheable by torch.compile. torch.compile just doesn't know it.
        # This was fixed in PyTorch 2.8, but until then, we monkey patch.
        import torch._higher_order_ops.auto_functionalize as af

        af.auto_functionalized_v2._cacheable = True
        af.auto_functionalized._cacheable = True
```
**EN:** This function implements `monkey_patch_torch_compile`. State updates are written into `af.auto_functionalized_v2._cacheable`, `af.auto_functionalized._cacheable`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `monkey_patch_torch_compile`。 状态更新主要写入 `af.auto_functionalized_v2._cacheable`, `af.auto_functionalized._cacheable`。 实现中使用了条件分支。

### Lines 116-139: Function `register_fake_if_exists` / 函数 `register_fake_if_exists`
```python
def register_fake_if_exists(op_name):
    """
    Decorator factory to conditionally register a fake for a custom op if it exists.
    Parses op_name (e.g., 'sgl_kernel::gptq_gemm'), checks if the op exists via hasattr
    on the namespace attribute of torch.ops. Registers the fake if present; otherwise,
    returns the function unchanged.
    Args:
        op_name (str): Full operator name (e.g., 'sgl_kernel::gptq_gemm').
    Returns:
        callable: Decorator for the fake function.
    Example:
        @register_fake_if_exists('sgl_kernel::gptq_gemm')
        def fake_gptq_gemm(a, b_q_weight, b_gptq_qzeros, b_gptq_scales, b_g_idx, use_shuffle, bit):
            return a.new_empty((a.shape[0], b_q_weight.shape[-1]), dtype=a.dtype)
    """

    def decorator(func):
        namespace, bare_op = op_name.split("::")
        ops_namespace = getattr(torch.ops, namespace, None)
        if ops_namespace and hasattr(ops_namespace, bare_op):
            torch.library.register_fake(op_name, func)
        return func

    return decorator
```
**EN:** This function implements `register_fake_if_exists`. It primarily calls `op_name.split`, `getattr`, `hasattr`, `torch.library.register_fake` to complete its work. State updates are written into `ops_namespace`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `register_fake_if_exists`。 它主要通过调用 `op_name.split`, `getattr`, `hasattr`, `torch.library.register_fake` 来完成任务。 状态更新主要写入 `ops_namespace`。 实现中使用了条件分支。

## Key Concepts / 关键概念
- **Functions / 函数**: `monkey_patch_torch_reductions`, `register_sgl_tp_rank`, `_reduce_tensor_modified`, `_rebuild_cuda_tensor_modified`, `_device_to_uuid`, `_device_from_maybe_uuid`, `_modify_tuple`, `monkey_patch_torch_compile`
- **Themes / 主题**: `patch`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.utils.common`
- **External / 外部依赖**: `torch`, `torch.multiprocessing`, `torch_npu.multiprocessing`, `torch._higher_order_ops.auto_functionalize`
- **Standard library / 标准库**: `typing`
