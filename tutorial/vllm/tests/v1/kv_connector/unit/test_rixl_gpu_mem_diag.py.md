# test_rixl_gpu_mem_diag.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/kv_connector/unit/test_rixl_gpu_mem_diag.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Verify that GPU memory is fully released after RixlConnector shutdown on ROCm. / 该文件的文档字符串表明其用途：`verify that gpu memory is fully released after rixlconnector shutdown on rocm`。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Module docstring / 模块说明 (lines 3-8)
```python
"""Verify that GPU memory is fully released after RixlConnector shutdown on ROCm.

Regression test for ROCm/ucx#33: UCX rocm_ipc transport permanently pinned
GPU memory via hsa_amd_ipc_memory_create during ucp_mem_map, causing
GPU memory to be unrecoverable after engine shutdown.
"""
```
**EN:** Module docstring that declares the scope of the file: Verify that GPU memory is fully released after RixlConnector shutdown on ROCm.
**CN:** 模块文档字符串直接说明了文件范围：`verify that gpu memory is fully released after rixlconnector shutdown on rocm`。

### Imports and setup / 导入与设置 (lines 10-15)
```python
import gc

import pytest
import torch

from vllm.platforms import current_platform
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch`. vLLM modules under test include `vllm.platforms, vllm, vllm.config, vllm.distributed.parallel_state`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch`。 被测试的 vLLM 模块包括 `vllm.platforms, vllm, vllm.config, vllm.distributed.parallel_state`。

### Module state / 模块级状态 (lines 17-20)
```python
pytestmark = pytest.mark.skipif(
    not current_platform.is_rocm(),
    reason="ROCm platform required",
)
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `pytestmark`. It also sets pytest marks that scope the whole file. Shared setup calls include `mark.skipif, current_platform.is_rocm`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`pytestmark`。 该块还设置了作用于整个文件的 pytest 标记。 共享初始化调用包括 `mark.skipif, current_platform.is_rocm`。

### _mb (lines 23-24)
```python
def _mb(b: int) -> float:
    return b / (1024 * 1024)
```
**EN:** Helper function `_mb` encapsulates reusable logic for `mb`. Inputs: `b`.
**CN:** 辅助函数 `_mb` 封装了与 `mb` 相关的可复用逻辑。 输入参数：`b`。

### _gpu_snapshot (lines 27-53)
```python
def _gpu_snapshot(tag: str, prev_alloc: float = 0.0) -> dict:
    """Print and return current GPU memory stats."""
    torch.accelerator.synchronize()
    alloc = torch.accelerator.memory_allocated()
    reserved = torch.accelerator.memory_reserved()
    # mem_get_info is not available on torch.accelerator
    try:
        drv_free, drv_total = torch.cuda.mem_get_info()
        drv_used = drv_total - drv_free
        drv_pct = drv_used / drv_total * 100
    except Exception:
        drv_used = drv_total = drv_pct = 0
    alloc_mb = _mb(alloc)
    drv_used_mb = _mb(drv_used)
    delta = alloc_mb - prev_alloc
    print(
        f"  {tag:<40s} | {alloc_mb:>9.1f} alloc | "
        f"{_mb(reserved):>9.1f} rsrvd | "
        f"{drv_used_mb:>9.1f} driver ({drv_pct:.1f}%) | "
        f"delta {delta:>+9.1f}"
    )
    return {
        "tag": tag,
        "alloc_mb": alloc_mb,
        "drv_used_mb": drv_used_mb,
        "drv_pct": drv_pct,
    }
```
**EN:** Helper function `_gpu_snapshot` encapsulates reusable logic for `GPU snapshot`. Inputs: `tag, prev_alloc`. Key calls include `accelerator.synchronize, accelerator.memory_allocated, accelerator.memory_reserved, _mb, print, cuda.mem_get_info`.
**CN:** 辅助函数 `_gpu_snapshot` 封装了与 `gpu snapshot` 相关的可复用逻辑。 输入参数：`tag, prev_alloc`。 关键调用包括 `accelerator.synchronize, accelerator.memory_allocated, accelerator.memory_reserved, _mb, print, cuda.mem_get_info`。

### _full_gpu_cleanup (lines 56-62)
```python
def _full_gpu_cleanup():
    """gc.collect + torch empty_cache, multiple rounds."""
    gc.unfreeze()
    for _ in range(3):
        if gc.collect() == 0:
            break
    torch.accelerator.empty_cache()
```
**EN:** Helper function `_full_gpu_cleanup` encapsulates reusable logic for `full GPU cleanup`. Key calls include `gc.unfreeze, range, accelerator.empty_cache, gc.collect`.
**CN:** 辅助函数 `_full_gpu_cleanup` 封装了与 `full gpu cleanup` 相关的可复用逻辑。 关键调用包括 `gc.unfreeze, range, accelerator.empty_cache, gc.collect`。

### test_gpu_memory_rixl_hma (lines 66-169)
```python
def test_gpu_memory_rixl_hma(model_name, sw_size):
    """Track GPU memory through NixlConnector create/infer/shutdown cycle."""
    from vllm import LLM, SamplingParams
    from vllm.config import KVTransferConfig
    from vllm.distributed.parallel_state import cleanup_dist_env_and_memory

    llm_kwargs = {
        "model": model_name,
        "enforce_eager": True,
        "gpu_memory_utilization": 0.5,
        "kv_transfer_config": KVTransferConfig(
            kv_connector="NixlConnector",
            kv_role="kv_both",
        ),
        "max_model_len": 2048,
        "disable_hybrid_kv_cache_manager": False,
        "max_num_batched_tokens": 1024,
        "enable_prefix_caching": False,
    # ... excerpt omitted for brevity ...
    max_leak_pct = 10
    assert leak_pct <= max_leak_pct, (
        f"{drv_leaked:.0f} MB ({leak_pct:.1f}%) of driver-level GPU memory "
        f"not freed after NixlConnector shutdown "
        f"(peak allocation: {drv_peak:.0f} MB, threshold: {max_leak_pct}%)"
    )
```
**EN:** Parameterized test covering `GPU memory rixl hma`. Parameter axes: `model_name, sw_size`. Inputs/fixtures: `model_name, sw_size`. It exercises `mark.parametrize, print, gc.collect, accelerator.empty_cache, accelerator.reset_peak_memory_stats, _gpu_snapshot`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `gpu memory rixl hma` 的测试用例。 参数维度：`model_name, sw_size`。 输入或 fixture：`model_name, sw_size`。 该测试会调用 `mark.parametrize, print, gc.collect, accelerator.empty_cache, accelerator.reset_peak_memory_stats, _gpu_snapshot`。 代码主体包含 1 个显式断言。

### test_gpu_memory_no_rixl_baseline (lines 173-222)
```python
def test_gpu_memory_no_rixl_baseline(model_name):
    """Same workload without NixlConnector.  Comparing driver-level memory
    between this and test_gpu_memory_rixl_hma isolates UCX/RIXL impact."""
    from vllm import LLM, SamplingParams
    from vllm.distributed.parallel_state import cleanup_dist_env_and_memory

    print("\n" + "=" * 90)
    print("CONTROL -- same model, no RIXL connector")
    print("=" * 90)
    gc.collect()
    torch.accelerator.empty_cache()
    snap0 = _gpu_snapshot("baseline", 0.0)
    llm = LLM(
        model=model_name,
        enforce_eager=True,
        gpu_memory_utilization=0.5,
        max_model_len=2048,
    # ... excerpt omitted for brevity ...
    max_leak_pct = 10
    assert leak_pct <= max_leak_pct, (
        f"{drv_leaked:.0f} MB ({leak_pct:.1f}%) of driver-level GPU memory "
        f"not freed after baseline shutdown "
        f"(peak allocation: {drv_peak:.0f} MB, threshold: {max_leak_pct}%)"
    )
```
**EN:** Parameterized test covering `GPU memory no rixl baseline`. Parameter axes: `model_name`. Inputs/fixtures: `model_name`. It exercises `mark.parametrize, print, gc.collect, accelerator.empty_cache, _gpu_snapshot, LLM`. The body contains 1 explicit assertion(s).
**CN:** 该代码块是覆盖 `gpu memory no rixl baseline` 的测试用例。 参数维度：`model_name`。 输入或 fixture：`model_name`。 该测试会调用 `mark.parametrize, print, gc.collect, accelerator.empty_cache, _gpu_snapshot, LLM`。 代码主体包含 1 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Connector contracts and transfer paths
- **CN:** 连接器契约与传输路径
- **EN:** GPU-specific execution assumptions
- **CN:** GPU 特定执行假设

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch`.
- **CN:** 外部库：`pytest, torch`。
- **EN:** vLLM modules under test: `vllm.platforms, vllm, vllm.config, vllm.distributed.parallel_state`.
- **CN:** 被测试的 vLLM 模块：`vllm.platforms, vllm, vllm.config, vllm.distributed.parallel_state`。
- **EN:** Standard-library support: `gc`.
- **CN:** 标准库支持：`gc`。
