# test_hicache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/jit_kernel/tests/test_hicache.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module in the jit_kernel area defines runtime logic, helpers, or kernel-facing interfaces used by SGLang. It contributes to compilation, dispatch, or validation paths for specialized GPU kernels. A visible top-level cue is: "import sys". / 这个位于 jit_kernel 相关目录的 Python 模块定义了运行时逻辑、辅助函数或面向内核的接口，供 SGLang 使用。它参与专用 GPU 内核的编译、分发或校验流程。 从文件开头可以看到一个明显线索：“import sys”。

## Line-by-Line Analysis / 逐行分析
### Lines 1-35: Imports and module setup
```python
import sys

import pytest
import torch

from sglang.srt.mem_cache.memory_pool import MHATokenToKVPool, MLATokenToKVPool
from sglang.srt.mem_cache.memory_pool_host import (
    ALLOC_MEMORY_FUNCS,
    MHATokenToKVPoolHost,
    MLATokenToKVPoolHost,
    alloc_with_pin_memory,
)
from sglang.srt.utils import is_cuda, is_hip, is_npu, is_xpu
from sglang.test.ci.ci_register import register_cuda_ci

register_cuda_ci(est_time=10, suite="base-b-kernel-unit-1-gpu-large")
register_cuda_ci(est_time=120, suite="nightly-kernel-1-gpu", nightly=True)

pytestmark = pytest.mark.skipif(
    not torch.cuda.is_available()
    or is_npu()
    or is_xpu()
    or not (is_cuda() or is_hip()),
    reason="HiCache JIT tests require CUDA/ROCm.",
)

DEVICE = "cuda"
PAGE_SIZE = 1 if is_hip() else 16
NUM_LAYERS = 2
POOL_SIZE = PAGE_SIZE * 8
MHA_ELEMENT_DIMS = [128, 256, 512, 1024]
MLA_ELEMENT_DIMS = [576]
LAYOUTS = ["layer_first", "page_first"]
```
**EN:** This block imports dependencies and prepares names used later in the module. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段导入依赖并准备后续模块要使用的名称。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 36-50: Function `_token_indices_for_pages`
```python
def _token_indices_for_pages(
    pages: torch.Tensor, page_size: int = PAGE_SIZE, device: str = DEVICE
) -> torch.Tensor:
    parts = [
        torch.arange(
            int(page) * page_size,
            (int(page) + 1) * page_size,
            device=device,
            dtype=torch.int64,
        )
        for page in pages.tolist()
    ]
    return torch.cat(parts, dim=0)
```
**EN:** This block defines `_token_indices_for_pages`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_token_indices_for_pages`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 51-66: Function `_pinned_host_pool`
```python
def _pinned_host_pool(host_pool_cls, **kwargs):
    original_alloc = ALLOC_MEMORY_FUNCS[DEVICE]
    ALLOC_MEMORY_FUNCS[DEVICE] = alloc_with_pin_memory
    try:
        return host_pool_cls(
            host_to_device_ratio=2.0,
            host_size=0,
            page_size=PAGE_SIZE,
            pin_memory=True,
            device="cpu",
            **kwargs,
        )
    finally:
        ALLOC_MEMORY_FUNCS[DEVICE] = original_alloc
```
**EN:** This block defines `_pinned_host_pool`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_pinned_host_pool`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 67-73: Function `_copy_tensor_with_offset`
```python
def _copy_tensor_with_offset(tensor: torch.Tensor, offset: int) -> None:
    data = torch.arange(
        tensor.numel(), device=tensor.device, dtype=tensor.dtype
    ).view_as(tensor)
    tensor.copy_(data + offset)
```
**EN:** This block defines `_copy_tensor_with_offset`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_copy_tensor_with_offset`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 74-162: Function `_run_transfer_roundtrip_mha`
```python
def _run_transfer_roundtrip_mha(layout: str, element_dim: int) -> None:
    device_pool = MHATokenToKVPool(
        size=POOL_SIZE,
        page_size=PAGE_SIZE,
        head_num=element_dim // 128,
        head_dim=128,
        dtype=torch.bfloat16,
        layer_num=NUM_LAYERS,
        device=DEVICE,
        enable_memory_saver=False,
    )
    host_pool = _pinned_host_pool(
        MHATokenToKVPoolHost,
        device_pool=device_pool,
        layout=layout,
    )
    assert (
        host_pool.can_use_jit
    ), f"Expected JIT HiCache kernel for MHA dim={element_dim}"

    for layer_id in range(NUM_LAYERS):
        _copy_tensor_with_offset(device_pool.k_buffer[layer_id], layer_id)
        _copy_tensor_with_offset(device_pool.v_buffer[layer_id], layer_id + 100)

    device_pages = torch.tensor([1, 2, 3], device=DEVICE, dtype=torch.int64)
    host_pages = torch.tensor([0, 1, 2], device=DEVICE, dtype=torch.int64)
    device_indices = _token_indices_for_pages(device_pages)
    host_indices = _token_indices_for_pages(host_pages)

    host_pool.backup_from_device_all_layer(
        device_pool, host_indices, device_indices, "kernel"
    )
    torch.cuda.synchronize()

    for layer_id in range(NUM_LAYERS):
        for host_page, device_page in zip(host_pages.tolist(), device_pages.tolist()):
            host_start = host_page * PAGE_SIZE
            device_start = device_page * PAGE_SIZE
            assert torch.equal(
                host_pool.k_data_refs[layer_id][
                    host_start : host_start + PAGE_SIZE
                ].cpu(),
                device_pool.k_buffer[layer_id][
                    device_start : device_start + PAGE_SIZE
                ].cpu(),
            )
            assert torch.equal(
                host_pool.v_data_refs[layer_id][
                    host_start : host_start + PAGE_SIZE
                ].cpu(),
                device_pool.v_buffer[layer_id][
                    device_start : device_start + PAGE_SIZE
                ].cpu(),
            )

    for layer_id in range(NUM_LAYERS):
        device_pool.k_buffer[layer_id].zero_()
        device_pool.v_buffer[layer_id].zero_()

    load_pages = torch.tensor([4, 5, 6], device=DEVICE, dtype=torch.int64)
    load_indices = _token_indices_for_pages(load_pages)
    for layer_id in range(NUM_LAYERS):
        host_pool.load_to_device_per_layer(
            device_pool, host_indices, load_indices, layer_id, "kernel"
        )
    torch.cuda.synchronize()

    for layer_id in range(NUM_LAYERS):
        for host_page, device_page in zip(host_pages.tolist(), load_pages.tolist()):
            host_start = host_page * PAGE_SIZE
            device_start = device_page * PAGE_SIZE
            assert torch.equal(
                device_pool.k_buffer[layer_id][
                    device_start : device_start + PAGE_SIZE
                ].cpu(),
                host_pool.k_data_refs[layer_id][
                    host_start : host_start + PAGE_SIZE
                ].cpu(),
            )
            assert torch.equal(
# ...
```
**EN:** This block defines `_run_transfer_roundtrip_mha`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_run_transfer_roundtrip_mha`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 163-235: Function `_run_transfer_roundtrip_mla`
```python
def _run_transfer_roundtrip_mla(layout: str, element_dim: int) -> None:
    device_pool = MLATokenToKVPool(
        size=POOL_SIZE,
        page_size=PAGE_SIZE,
        kv_lora_rank=element_dim - 64,
        qk_rope_head_dim=64,
        dtype=torch.bfloat16,
        layer_num=NUM_LAYERS,
        device=DEVICE,
        enable_memory_saver=False,
    )
    host_pool = _pinned_host_pool(
        MLATokenToKVPoolHost,
        device_pool=device_pool,
        layout=layout,
    )
    assert (
        host_pool.can_use_jit
    ), f"Expected JIT HiCache kernel for MLA dim={element_dim}"

    for layer_id in range(NUM_LAYERS):
        _copy_tensor_with_offset(device_pool.kv_buffer[layer_id], layer_id)

    device_pages = torch.tensor([1, 2, 3], device=DEVICE, dtype=torch.int64)
    host_pages = torch.tensor([0, 1, 2], device=DEVICE, dtype=torch.int64)
    device_indices = _token_indices_for_pages(device_pages)
    host_indices = _token_indices_for_pages(host_pages)

    host_pool.backup_from_device_all_layer(
        device_pool, host_indices, device_indices, "kernel"
    )
    torch.cuda.synchronize()

    for layer_id in range(NUM_LAYERS):
        for host_page, device_page in zip(host_pages.tolist(), device_pages.tolist()):
            host_start = host_page * PAGE_SIZE
            device_start = device_page * PAGE_SIZE
            assert torch.equal(
                host_pool.data_refs[layer_id][
                    host_start : host_start + PAGE_SIZE
                ].cpu(),
                device_pool.kv_buffer[layer_id][
                    device_start : device_start + PAGE_SIZE
                ].cpu(),
            )

    for layer_id in range(NUM_LAYERS):
        device_pool.kv_buffer[layer_id].zero_()

    load_pages = torch.tensor([4, 5, 6], device=DEVICE, dtype=torch.int64)
    load_indices = _token_indices_for_pages(load_pages)
    for layer_id in range(NUM_LAYERS):
        host_pool.load_to_device_per_layer(
            device_pool, host_indices, load_indices, layer_id, "kernel"
        )
    torch.cuda.synchronize()

    for layer_id in range(NUM_LAYERS):
        for host_page, device_page in zip(host_pages.tolist(), load_pages.tolist()):
            host_start = host_page * PAGE_SIZE
            device_start = device_page * PAGE_SIZE
            assert torch.equal(
                device_pool.kv_buffer[layer_id][
                    device_start : device_start + PAGE_SIZE
                ].cpu(),
                host_pool.data_refs[layer_id][
                    host_start : host_start + PAGE_SIZE
                ].cpu(),
            )


@pytest.mark.parametrize("layout", LAYOUTS)
@pytest.mark.parametrize("element_dim", MHA_ELEMENT_DIMS)
```
**EN:** This block defines `_run_transfer_roundtrip_mla`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `_run_transfer_roundtrip_mla`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 236-241: Function `test_hicache_transfer_mha`
```python
def test_hicache_transfer_mha(layout: str, element_dim: int) -> None:
    _run_transfer_roundtrip_mha(layout, element_dim)


@pytest.mark.parametrize("layout", LAYOUTS)
@pytest.mark.parametrize("element_dim", MLA_ELEMENT_DIMS)
```
**EN:** This block defines `test_hicache_transfer_mha`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_hicache_transfer_mha`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

### Lines 242-247: Function `test_hicache_transfer_mla`
```python
def test_hicache_transfer_mla(layout: str, element_dim: int) -> None:
    _run_transfer_roundtrip_mla(layout, element_dim)


if __name__ == "__main__":
    sys.exit(pytest.main([__file__, "-v", "-s"]))
```
**EN:** This block defines `test_hicache_transfer_mla`, including its interface and main implementation logic. Because this file sits in a test path, the code is likely validating expected behavior. In context, it is related to specialized kernel generation, launch, or verification.
**CN:** 这一段定义了 `test_hicache_transfer_mla`，包括其接口以及主要实现逻辑。 由于该文件位于测试路径中，这里的代码很可能用于校验预期行为。 在上下文中，它与专用内核的生成、启动或校验有关。

## Key Concepts / 关键概念
- **Module structure**: How imports, declarations, and helper code are organized / 导入、声明与辅助代码如何组织
- **Kernel orchestration**: How Python prepares or dispatches specialized kernels / Python 如何准备或分发专用内核
- **Behavior validation**: How expected outputs or invariants are checked / 如何检查预期输出或不变量

## Dependencies / 依赖关系
- `sys`
- `pytest`
- `torch`
- `sglang.srt.mem_cache.memory_pool -> MHATokenToKVPool`
- `sglang.srt.mem_cache.memory_pool_host -> (`
- `sglang.srt.utils -> is_cuda`
- `sglang.test.ci.ci_register -> register_cuda_ci`
