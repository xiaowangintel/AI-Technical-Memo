# test_gsan_failures.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/test/gsan/test_gsan_failures.py`
- **EN:** Pytest module covering gsan failures behavior in Triton's Python tests. It contains 36 top-level definition(s) and 12 imported module reference(s).
- **CN:** 这是一个 pytest 模块，用于覆盖 Triton Python 测试中的 gsan failures 行为。 该文件包含 36 个顶层定义，以及 12 个导入模块引用。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15

```python
from __future__ import annotations

import functools
import inspect
from pathlib import Path

import pytest
import torch
import triton
import triton.language as tl

from triton._internal_testing import is_blackwell, is_cuda, run_in_process
from triton.experimental.gsan import create_mem_pool
from triton.experimental.gsan._testing_utils import atomic_poll
from triton.tools.tensor_descriptor import TensorDescriptor
```
- **EN:** Imports the modules used in this scope: `__future__`, `functools`, `inspect`, `pathlib`, `pytest`, `torch`, `triton`, `triton.language`, `triton._internal_testing`, `triton.experimental.gsan`, and 2 more. Relevant themes: Triton language operations, tensor/descriptor metadata.
- **CN:** 导入此作用域使用的模块：`__future__`、`functools`、`inspect`、`pathlib`、`pytest`、`torch`、`triton`、`triton.language`、`triton._internal_testing`、`triton.experimental.gsan` 等另外 2 项。 相关主题：Triton language 操作、张量/描述符元数据。

### Lines 16-38

```python

pytestmark = pytest.mark.skipif(not is_cuda(), reason="requires CUDA backend")

RELEASE_ACQUIRE_SYNC_CASES = (
    pytest.param("release", "acquire", id="release-acquire"),
    pytest.param("release", "acq_rel", id="release-acq-rel"),
    pytest.param("acq_rel", "acquire", id="acq-rel-acquire"),
    pytest.param("acq_rel", "acq_rel", id="acq-rel-acq-rel"),
)

CROSS_SM_SEMANTIC_MISMATCH_CASES = (
    pytest.param("relaxed", "acquire", "gpu", id="producer-relaxed-consumer-acquire-scope-gpu"),
    pytest.param("relaxed", "acquire", "sys", id="producer-relaxed-consumer-acquire-scope-sys"),
    pytest.param("release", "relaxed", "gpu", id="producer-release-consumer-relaxed-scope-gpu"),
    pytest.param("release", "relaxed", "sys", id="producer-release-consumer-relaxed-scope-sys"),
)

TRANSITIVE_RELAY_MISMATCH_CASES = (
    pytest.param("release", "relaxed", "gpu", id="relay-relaxed-scope-gpu"),
    pytest.param("release", "relaxed", "sys", id="relay-relaxed-scope-sys"),
    pytest.param("acq_rel", "release", "gpu", id="relay-release-scope-gpu"),
    pytest.param("acq_rel", "release", "sys", id="relay-release-scope-sys"),
)
```
- **EN:** Prepares or updates state through `pytestmark`, `RELEASE_ACQUIRE_SYNC_CASES`, `CROSS_SM_SEMANTIC_MISMATCH_CASES`, `TRANSITIVE_RELAY_MISMATCH_CASES`. Invokes `pytest.mark.skipif`, `is_cuda`, `pytest.param` to execute the test logic.
- **CN:** 通过 `pytestmark`、`RELEASE_ACQUIRE_SYNC_CASES`、`CROSS_SM_SEMANTIC_MISMATCH_CASES`、`TRANSITIVE_RELAY_MISMATCH_CASES` 准备或更新状态。 调用 `pytest.mark.skipif`、`is_cuda`、`pytest.param` 执行测试逻辑。

### Lines 39-42

```python


@triton.jit
def _raw_kernel(ptr, scratch_ptr, counter_ptr):
```
- **EN:** Defines the helper function `_raw_kernel`. Decorators: `triton.jit`. Parameters: `ptr`, `scratch_ptr`, `counter_ptr`. Key calls include `tl.program_id`, `tl.store`, `tl.atomic_add`, `atomic_poll`, `tl.load`. This scope touches Triton compilation or JIT kernels, Triton language operations.
- **CN:** 定义辅助函数 `_raw_kernel`。 装饰器：`triton.jit`。 参数：`ptr`、`scratch_ptr`、`counter_ptr`。 关键调用包括 `tl.program_id`、`tl.store`、`tl.atomic_add`、`atomic_poll`、`tl.load`。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作。

#### Lines 43-43

```python
    pid = tl.program_id(0)
```
- **EN:** Prepares or updates state through `pid`. Invokes `tl.program_id` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `pid` 准备或更新状态。 调用 `tl.program_id` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 44-50

```python
    if pid == 0:
        tl.store(ptr, 1)
        tl.atomic_add(counter_ptr, 1, sem="relaxed")
    else:
        atomic_poll(counter_ptr, 1)
        value = tl.load(ptr)
        tl.store(scratch_ptr, value)
```
- **EN:** Invokes `tl.store`, `tl.atomic_add`, `atomic_poll`, `tl.load` to execute the test logic. Branches on runtime or test conditions. Relevant themes: Triton language operations.
- **CN:** 调用 `tl.store`、`tl.atomic_add`、`atomic_poll`、`tl.load` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：Triton language 操作。

### Lines 51-54

```python


@triton.jit
def _war_kernel(ptr, scratch_ptr, counter_ptr):
```
- **EN:** Defines the helper function `_war_kernel`. Decorators: `triton.jit`. Parameters: `ptr`, `scratch_ptr`, `counter_ptr`. Key calls include `tl.program_id`, `tl.load`, `tl.store`, `tl.atomic_add`, `atomic_poll`. This scope touches Triton compilation or JIT kernels, Triton language operations.
- **CN:** 定义辅助函数 `_war_kernel`。 装饰器：`triton.jit`。 参数：`ptr`、`scratch_ptr`、`counter_ptr`。 关键调用包括 `tl.program_id`、`tl.load`、`tl.store`、`tl.atomic_add`、`atomic_poll`。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作。

#### Lines 55-55

```python
    pid = tl.program_id(0)
```
- **EN:** Prepares or updates state through `pid`. Invokes `tl.program_id` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `pid` 准备或更新状态。 调用 `tl.program_id` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 56-62

```python
    if pid == 0:
        value = tl.load(ptr)
        tl.store(scratch_ptr, value)
        tl.atomic_add(counter_ptr, 1, sem="relaxed")
    else:
        atomic_poll(counter_ptr, 1)
        tl.store(ptr, 1)
```
- **EN:** Invokes `tl.load`, `tl.store`, `tl.atomic_add`, `atomic_poll` to execute the test logic. Branches on runtime or test conditions. Relevant themes: Triton language operations.
- **CN:** 调用 `tl.load`、`tl.store`、`tl.atomic_add`、`atomic_poll` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：Triton language 操作。

### Lines 63-66

```python


@triton.jit
def _waw_kernel(ptr, scratch_ptr, counter_ptr):
```
- **EN:** Defines the helper function `_waw_kernel`. Decorators: `triton.jit`. Parameters: `ptr`, `scratch_ptr`, `counter_ptr`. Key calls include `tl.program_id`, `tl.store`, `tl.atomic_add`, `atomic_poll`. This scope touches Triton compilation or JIT kernels, Triton language operations.
- **CN:** 定义辅助函数 `_waw_kernel`。 装饰器：`triton.jit`。 参数：`ptr`、`scratch_ptr`、`counter_ptr`。 关键调用包括 `tl.program_id`、`tl.store`、`tl.atomic_add`、`atomic_poll`。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作。

#### Lines 67-67

```python
    pid = tl.program_id(0)
```
- **EN:** Prepares or updates state through `pid`. Invokes `tl.program_id` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `pid` 准备或更新状态。 调用 `tl.program_id` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 68-73

```python
    if pid == 0:
        tl.store(ptr, 1)
        tl.atomic_add(counter_ptr, 1, sem="relaxed")
    else:
        atomic_poll(counter_ptr, 1)
        tl.store(ptr, 2)
```
- **EN:** Invokes `tl.store`, `tl.atomic_add`, `atomic_poll` to execute the test logic. Branches on runtime or test conditions. Relevant themes: Triton language operations.
- **CN:** 调用 `tl.store`、`tl.atomic_add`、`atomic_poll` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：Triton language 操作。

### Lines 74-78

```python


@triton.jit
def _cross_sm_atomic_sync_kernel(payload_ptr, flag_ptr, counter_ptr, scratch_ptr, producer_sem: tl.constexpr,
                                 consumer_sem: tl.constexpr, scope: tl.constexpr):
```
- **EN:** Defines the helper function `_cross_sm_atomic_sync_kernel`. Decorators: `triton.jit`. Parameters: `payload_ptr`, `flag_ptr`, `counter_ptr`, `scratch_ptr`, `producer_sem`, `consumer_sem`, `scope`. Key calls include `tl.program_id`, `tl.store`, `tl.atomic_xchg`, `tl.atomic_add`, `atomic_poll`, `tl.load`. This scope touches Triton compilation or JIT kernels, Triton language operations.
- **CN:** 定义辅助函数 `_cross_sm_atomic_sync_kernel`。 装饰器：`triton.jit`。 参数：`payload_ptr`、`flag_ptr`、`counter_ptr`、`scratch_ptr`、`producer_sem`、`consumer_sem`、`scope`。 关键调用包括 `tl.program_id`、`tl.store`、`tl.atomic_xchg`、`tl.atomic_add`、`atomic_poll`、`tl.load`。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作。

#### Lines 79-79

```python
    pid = tl.program_id(0)
```
- **EN:** Prepares or updates state through `pid`. Invokes `tl.program_id` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `pid` 准备或更新状态。 调用 `tl.program_id` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 80-90

```python
    if pid == 0:
        tl.store(payload_ptr, 1000)
        tl.atomic_xchg(flag_ptr, 1, sem=producer_sem, scope=scope)
        tl.atomic_add(counter_ptr, 1, sem="relaxed")
    elif pid == 1:
        atomic_poll(counter_ptr, 1)
        ready = 0
        while ready != 1:
            ready = tl.atomic_add(flag_ptr, 0, sem=consumer_sem, scope=scope)
        result = tl.load(payload_ptr)
        tl.store(scratch_ptr, result)
```
- **EN:** Invokes `tl.store`, `tl.atomic_xchg`, `tl.atomic_add`, `atomic_poll`, `tl.load` to execute the test logic. Branches on runtime or test conditions. Relevant themes: Triton language operations.
- **CN:** 调用 `tl.store`、`tl.atomic_xchg`、`tl.atomic_add`、`atomic_poll`、`tl.load` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：Triton language 操作。

### Lines 91-95

```python


@triton.jit
def _transitive_atomic_sync_kernel(payload_ptr, flag0_ptr, flag1_ptr, counter_ptr, scratch_ptr,
                                   release_sem: tl.constexpr, relay_sem: tl.constexpr, scope: tl.constexpr):
```
- **EN:** Defines the helper function `_transitive_atomic_sync_kernel`. Decorators: `triton.jit`. Parameters: `payload_ptr`, `flag0_ptr`, `flag1_ptr`, `counter_ptr`, `scratch_ptr`, `release_sem`, `relay_sem`, `scope`. Key calls include `tl.program_id`, `tl.store`, `tl.atomic_xchg`, `tl.atomic_add`, `atomic_poll`, `tl.load`. This scope touches Triton compilation or JIT kernels, Triton language operations.
- **CN:** 定义辅助函数 `_transitive_atomic_sync_kernel`。 装饰器：`triton.jit`。 参数：`payload_ptr`、`flag0_ptr`、`flag1_ptr`、`counter_ptr`、`scratch_ptr`、`release_sem`、`relay_sem`、`scope`。 关键调用包括 `tl.program_id`、`tl.store`、`tl.atomic_xchg`、`tl.atomic_add`、`atomic_poll`、`tl.load`。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作。

#### Lines 96-96

```python
    pid = tl.program_id(0)
```
- **EN:** Prepares or updates state through `pid`. Invokes `tl.program_id` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `pid` 准备或更新状态。 调用 `tl.program_id` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 97-114

```python
    if pid == 0:
        tl.store(payload_ptr, 1000)
        tl.atomic_xchg(flag0_ptr, 1, sem=release_sem, scope=scope)
        tl.atomic_add(counter_ptr, 1, sem="relaxed")
    elif pid == 1:
        atomic_poll(counter_ptr, 1)
        ready = 0
        while ready != 1:
            ready = tl.atomic_add(flag0_ptr, 0, sem=relay_sem, scope=scope)
        tl.atomic_xchg(flag1_ptr, 1, sem=release_sem, scope=scope)
        tl.atomic_add(counter_ptr, 1, sem="relaxed")
    elif pid == 2:
        atomic_poll(counter_ptr, 2)
        ready = 0
        while ready != 1:
            ready = tl.atomic_add(flag1_ptr, 0, sem="acquire", scope=scope)
        result = tl.load(payload_ptr)
        tl.store(scratch_ptr, result)
```
- **EN:** Invokes `tl.store`, `tl.atomic_xchg`, `tl.atomic_add`, `atomic_poll`, `tl.load` to execute the test logic. Branches on runtime or test conditions. Relevant themes: Triton language operations.
- **CN:** 调用 `tl.store`、`tl.atomic_xchg`、`tl.atomic_add`、`atomic_poll`、`tl.load` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：Triton language 操作。

### Lines 115-118

```python


@triton.jit
def _tma_raw_kernel(ptr, scratch_ptr, counter_ptr, m_size, n_size, row_idx, col_idx, stride_0, BLOCK: tl.constexpr):
```
- **EN:** Defines the helper function `_tma_raw_kernel`. Decorators: `triton.jit`. Parameters: `ptr`, `scratch_ptr`, `counter_ptr`, `m_size`, `n_size`, `row_idx`, `col_idx`, `stride_0`, and 1 more. Key calls include `tl.program_id`, `tl.make_tensor_descriptor`, `tl.full`, `desc.store`, `tl.atomic_add`, `atomic_poll`, and 2 more. This scope touches Triton compilation or JIT kernels, Triton language operations, tensor/descriptor metadata.
- **CN:** 定义辅助函数 `_tma_raw_kernel`。 装饰器：`triton.jit`。 参数：`ptr`、`scratch_ptr`、`counter_ptr`、`m_size`、`n_size`、`row_idx`、`col_idx`、`stride_0` 等另外 1 项。 关键调用包括 `tl.program_id`、`tl.make_tensor_descriptor`、`tl.full`、`desc.store`、`tl.atomic_add`、`atomic_poll` 等另外 2 项。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作、张量/描述符元数据。

#### Lines 119-119

```python
    pid = tl.program_id(0)
```
- **EN:** Prepares or updates state through `pid`. Invokes `tl.program_id` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `pid` 准备或更新状态。 调用 `tl.program_id` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 120-128

```python
    if pid == 0:
        desc = tl.make_tensor_descriptor(ptr, [m_size, n_size], [stride_0, 1], [BLOCK, BLOCK])
        values = tl.full((BLOCK, BLOCK), 1, dtype=tl.int32)
        desc.store([row_idx, col_idx], values)
        tl.atomic_add(counter_ptr, 1, sem="relaxed")
    else:
        atomic_poll(counter_ptr, 1)
        value = tl.load(ptr + row_idx * stride_0 + col_idx)
        tl.store(scratch_ptr, value)
```
- **EN:** Invokes `tl.make_tensor_descriptor`, `tl.full`, `desc.store`, `tl.atomic_add`, `atomic_poll`, `tl.load`, and 1 more to execute the test logic. Branches on runtime or test conditions. Relevant themes: Triton language operations, tensor/descriptor metadata.
- **CN:** 调用 `tl.make_tensor_descriptor`、`tl.full`、`desc.store`、`tl.atomic_add`、`atomic_poll`、`tl.load` 等另外 1 项 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：Triton language 操作、张量/描述符元数据。

### Lines 129-132

```python


@triton.jit
def _host_tma_war_kernel(target_ptr, target_desc, scratch_desc, counter_ptr, row_idx, col_idx, stride_0):
```
- **EN:** Defines the helper function `_host_tma_war_kernel`. Decorators: `triton.jit`. Parameters: `target_ptr`, `target_desc`, `scratch_desc`, `counter_ptr`, `row_idx`, `col_idx`, `stride_0`. Key calls include `tl.program_id`, `target_desc.load`, `scratch_desc.store`, `tl.atomic_add`, `atomic_poll`, `tl.store`. This scope touches Triton compilation or JIT kernels, Triton language operations.
- **CN:** 定义辅助函数 `_host_tma_war_kernel`。 装饰器：`triton.jit`。 参数：`target_ptr`、`target_desc`、`scratch_desc`、`counter_ptr`、`row_idx`、`col_idx`、`stride_0`。 关键调用包括 `tl.program_id`、`target_desc.load`、`scratch_desc.store`、`tl.atomic_add`、`atomic_poll`、`tl.store`。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作。

#### Lines 133-133

```python
    pid = tl.program_id(0)
```
- **EN:** Prepares or updates state through `pid`. Invokes `tl.program_id` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `pid` 准备或更新状态。 调用 `tl.program_id` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 134-140

```python
    if pid == 0:
        block = target_desc.load([row_idx, col_idx])
        scratch_desc.store([row_idx, col_idx], block)
        tl.atomic_add(counter_ptr, 1, sem="relaxed")
    else:
        atomic_poll(counter_ptr, 1)
        tl.store(target_ptr + row_idx * stride_0 + col_idx, 1)
```
- **EN:** Invokes `target_desc.load`, `scratch_desc.store`, `tl.atomic_add`, `atomic_poll`, `tl.store` to execute the test logic. Branches on runtime or test conditions. Relevant themes: Triton language operations.
- **CN:** 调用 `target_desc.load`、`scratch_desc.store`、`tl.atomic_add`、`atomic_poll`、`tl.store` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：Triton language 操作。

### Lines 141-145

```python


@triton.jit
def _host_tma_gather_war_kernel(target_ptr, target_desc, x_offsets_ptr, scratch_ptr, counter_ptr, row_idx, y_offset,
                                stride_0, scratch_stride_0, scratch_stride_1, BLOCK_X: tl.constexpr):
```
- **EN:** Defines the helper function `_host_tma_gather_war_kernel`. Decorators: `triton.jit`. Parameters: `target_ptr`, `target_desc`, `x_offsets_ptr`, `scratch_ptr`, `counter_ptr`, `row_idx`, `y_offset`, `stride_0`, and 3 more. Key calls include `tl.program_id`, `tl.load`, `target_desc.gather`, `tl.store`, `tl.atomic_add`, `atomic_poll`, and 1 more. This scope touches Triton compilation or JIT kernels, Triton language operations.
- **CN:** 定义辅助函数 `_host_tma_gather_war_kernel`。 装饰器：`triton.jit`。 参数：`target_ptr`、`target_desc`、`x_offsets_ptr`、`scratch_ptr`、`counter_ptr`、`row_idx`、`y_offset`、`stride_0` 等另外 3 项。 关键调用包括 `tl.program_id`、`tl.load`、`target_desc.gather`、`tl.store`、`tl.atomic_add`、`atomic_poll` 等另外 1 项。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作。

#### Lines 146-147

```python
    BLOCK_Y: tl.constexpr = target_desc.block_shape[1]
    pid = tl.program_id(0)
```
- **EN:** Prepares or updates state through `BLOCK_Y`, `pid`. Invokes `tl.program_id` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `BLOCK_Y`、`pid` 准备或更新状态。 调用 `tl.program_id` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 148-157

```python
    if pid == 0:
        x_offsets = tl.load(x_offsets_ptr + tl.arange(0, BLOCK_X))
        values = target_desc.gather(x_offsets, y_offset)
        indices_x = tl.arange(0, BLOCK_X)[:, None] * scratch_stride_0
        indices_y = tl.arange(0, BLOCK_Y)[None, :] * scratch_stride_1
        tl.store(scratch_ptr + indices_x + indices_y, values)
        tl.atomic_add(counter_ptr, 1, sem="relaxed")
    else:
        atomic_poll(counter_ptr, 1)
        tl.store(target_ptr + row_idx * stride_0 + y_offset, 1)
```
- **EN:** Invokes `tl.load`, `target_desc.gather`, `tl.store`, `tl.atomic_add`, `atomic_poll`, `tl.arange` to execute the test logic. Branches on runtime or test conditions. Relevant themes: Triton language operations.
- **CN:** 调用 `tl.load`、`target_desc.gather`、`tl.store`、`tl.atomic_add`、`atomic_poll`、`tl.arange` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：Triton language 操作。

### Lines 158-162

```python


@triton.jit
def _host_tma_scatter_war_kernel(target_ptr, target_desc, x_offsets_ptr, src_ptr, src_stride_0, src_stride_1,
                                 scratch_ptr, counter_ptr, row_idx, y_offset, stride_0, BLOCK_X: tl.constexpr):
```
- **EN:** Defines the helper function `_host_tma_scatter_war_kernel`. Decorators: `triton.jit`. Parameters: `target_ptr`, `target_desc`, `x_offsets_ptr`, `src_ptr`, `src_stride_0`, `src_stride_1`, `scratch_ptr`, `counter_ptr`, and 4 more. Key calls include `tl.program_id`, `tl.load`, `tl.store`, `tl.atomic_add`, `atomic_poll`, `target_desc.scatter`, and 1 more. This scope touches Triton compilation or JIT kernels, Triton language operations.
- **CN:** 定义辅助函数 `_host_tma_scatter_war_kernel`。 装饰器：`triton.jit`。 参数：`target_ptr`、`target_desc`、`x_offsets_ptr`、`src_ptr`、`src_stride_0`、`src_stride_1`、`scratch_ptr`、`counter_ptr` 等另外 4 项。 关键调用包括 `tl.program_id`、`tl.load`、`tl.store`、`tl.atomic_add`、`atomic_poll`、`target_desc.scatter` 等另外 1 项。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作。

#### Lines 163-164

```python
    BLOCK_Y: tl.constexpr = target_desc.block_shape[1]
    pid = tl.program_id(0)
```
- **EN:** Prepares or updates state through `BLOCK_Y`, `pid`. Invokes `tl.program_id` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `BLOCK_Y`、`pid` 准备或更新状态。 调用 `tl.program_id` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 165-175

```python
    if pid == 0:
        value = tl.load(target_ptr + row_idx * stride_0 + y_offset)
        tl.store(scratch_ptr, value)
        tl.atomic_add(counter_ptr, 1, sem="relaxed")
    else:
        atomic_poll(counter_ptr, 1)
        x_offsets = tl.load(x_offsets_ptr + tl.arange(0, BLOCK_X))
        indices_x = tl.arange(0, BLOCK_X)[:, None] * src_stride_0
        indices_y = tl.arange(0, BLOCK_Y)[None, :] * src_stride_1
        values = tl.load(src_ptr + indices_x + indices_y)
        target_desc.scatter(values, x_offsets, y_offset)
```
- **EN:** Invokes `tl.load`, `tl.store`, `tl.atomic_add`, `atomic_poll`, `target_desc.scatter`, `tl.arange` to execute the test logic. Branches on runtime or test conditions. Relevant themes: Triton language operations.
- **CN:** 调用 `tl.load`、`tl.store`、`tl.atomic_add`、`atomic_poll`、`target_desc.scatter`、`tl.arange` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：Triton language 操作。

### Lines 176-179

```python


@triton.jit
def _host_tma_atomic_flag_publish_kernel(payload_ptr, flag_ptr, flag_desc, counter_ptr, scratch_ptr):
```
- **EN:** Defines the helper function `_host_tma_atomic_flag_publish_kernel`. Decorators: `triton.jit`. Parameters: `payload_ptr`, `flag_ptr`, `flag_desc`, `counter_ptr`, `scratch_ptr`. Key calls include `tl.program_id`, `tl.store`, `tl.atomic_xchg`, `tl.atomic_add`, `atomic_poll`, `tl.full`, and 2 more. This scope touches Triton compilation or JIT kernels, Triton language operations.
- **CN:** 定义辅助函数 `_host_tma_atomic_flag_publish_kernel`。 装饰器：`triton.jit`。 参数：`payload_ptr`、`flag_ptr`、`flag_desc`、`counter_ptr`、`scratch_ptr`。 关键调用包括 `tl.program_id`、`tl.store`、`tl.atomic_xchg`、`tl.atomic_add`、`atomic_poll`、`tl.full` 等另外 2 项。 该作用域涉及Triton 编译或 JIT kernel、Triton language 操作。

#### Lines 180-180

```python
    pid = tl.program_id(0)
```
- **EN:** Prepares or updates state through `pid`. Invokes `tl.program_id` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 通过 `pid` 准备或更新状态。 调用 `tl.program_id` 执行测试逻辑。 相关主题：Triton language 操作。

#### Lines 181-194

```python
    if pid == 0:
        tl.store(payload_ptr, 1000)
        tl.atomic_xchg(flag_ptr, 1, sem="release", scope="gpu")
        tl.atomic_add(counter_ptr, 1, sem="relaxed")
    else:
        atomic_poll(counter_ptr, 1)
        BLOCK_X: tl.constexpr = flag_desc.block_shape[0]
        BLOCK_Y: tl.constexpr = flag_desc.block_shape[1]
        values = tl.full((BLOCK_X, BLOCK_Y), 1, dtype=tl.int32)
        # TMA atomics on the released flag are relaxed.gpu and must not acquire
        # the producer's prior payload store.
        flag_desc.atomic_add([0, 0], values)
        result = tl.load(payload_ptr)
        tl.store(scratch_ptr, result)
```
- **EN:** Invokes `tl.store`, `tl.atomic_xchg`, `tl.atomic_add`, `atomic_poll`, `tl.full`, `flag_desc.atomic_add`, and 1 more to execute the test logic. Branches on runtime or test conditions. Relevant themes: Triton language operations.
- **CN:** 调用 `tl.store`、`tl.atomic_xchg`、`tl.atomic_add`、`atomic_poll`、`tl.full`、`flag_desc.atomic_add` 等另外 1 项 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：Triton language 操作。

### Lines 195-197

```python


def _cuda_byte_allocator(size: int, _align: int, _stream):
```
- **EN:** Defines the helper function `_cuda_byte_allocator`. Parameters: `size`, `_align`, `_stream`. Key calls include `torch.empty`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `_cuda_byte_allocator`。 参数：`size`、`_align`、`_stream`。 关键调用包括 `torch.empty`。 该作用域涉及PyTorch 张量准备与校验。

#### Lines 198-198

```python
    return torch.empty(size, dtype=torch.int8, device="cuda")
```
- **EN:** Invokes `torch.empty` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.empty` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 199-203

```python


def run_with_gsan(fn):

    @functools.wraps(fn)
```
- **EN:** Defines the helper function `run_with_gsan`. Parameters: `fn`. Nested definitions in this scope: `wrapped`. Key calls include `functools.wraps`, `create_mem_pool`, `torch.cuda.use_mem_pool`, `fn`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `run_with_gsan`。 参数：`fn`。 该作用域中的嵌套定义：`wrapped`。 关键调用包括 `functools.wraps`、`create_mem_pool`、`torch.cuda.use_mem_pool`、`fn`。 该作用域涉及PyTorch 张量准备与校验。

#### Lines 203-204

```python
    @functools.wraps(fn)
    def wrapped(*args, **kwargs) -> None:
```
- **EN:** Defines the helper function `wrapped`. Decorators: `functools.wraps(fn)`. Parameters: `*args`, `**kwargs`. Key calls include `functools.wraps`, `create_mem_pool`, `torch.cuda.use_mem_pool`, `fn`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `wrapped`。 装饰器：`functools.wraps(fn)`。 参数：`*args`、`**kwargs`。 关键调用包括 `functools.wraps`、`create_mem_pool`、`torch.cuda.use_mem_pool`、`fn`。 该作用域涉及PyTorch 张量准备与校验。

##### Lines 205-206

```python
        triton.knobs.compilation.instrumentation_mode = "gsan"
        pool = create_mem_pool()
```
- **EN:** Prepares or updates state through `triton`, `pool`. Invokes `create_mem_pool` to execute the test logic.
- **CN:** 通过 `triton`、`pool` 准备或更新状态。 调用 `create_mem_pool` 执行测试逻辑。

##### Lines 207-208

```python
        with torch.cuda.use_mem_pool(pool):
            fn(*args, **kwargs)
```
- **EN:** Invokes `torch.cuda.use_mem_pool`, `fn` to execute the test logic. Uses context managers to control resources or expectations. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.cuda.use_mem_pool`、`fn` 执行测试逻辑。 使用上下文管理器控制资源或预期行为。 相关主题：PyTorch 张量准备与校验。

#### Lines 209-210

```python

    return wrapped
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 211-214

```python


@run_with_gsan
def _run_raw_case() -> None:
```
- **EN:** Defines the helper function `_run_raw_case`. Decorators: `run_with_gsan`. Key calls include `torch.zeros`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `_run_raw_case`。 装饰器：`run_with_gsan`。 关键调用包括 `torch.zeros`。 该作用域涉及PyTorch 张量准备与校验。

#### Lines 215-218

```python
    target = torch.zeros(1, dtype=torch.int32, device="cuda")
    scratch = torch.zeros(1, dtype=torch.int32, device="cuda")
    counter = torch.zeros(1, dtype=torch.int32, device="cuda")
    _raw_kernel[(2, )](target, scratch, counter, num_warps=1)
```
- **EN:** Prepares or updates state through `target`, `scratch`, `counter`. Invokes `torch.zeros` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `target`、`scratch`、`counter` 准备或更新状态。 调用 `torch.zeros` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 219-222

```python


@run_with_gsan
def _run_war_case() -> None:
```
- **EN:** Defines the helper function `_run_war_case`. Decorators: `run_with_gsan`. Key calls include `torch.zeros`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `_run_war_case`。 装饰器：`run_with_gsan`。 关键调用包括 `torch.zeros`。 该作用域涉及PyTorch 张量准备与校验。

#### Lines 223-226

```python
    target = torch.zeros(1, dtype=torch.int32, device="cuda")
    scratch = torch.zeros(1, dtype=torch.int32, device="cuda")
    counter = torch.zeros(1, dtype=torch.int32, device="cuda")
    _war_kernel[(2, )](target, scratch, counter, num_warps=1)
```
- **EN:** Prepares or updates state through `target`, `scratch`, `counter`. Invokes `torch.zeros` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `target`、`scratch`、`counter` 准备或更新状态。 调用 `torch.zeros` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 227-230

```python


@run_with_gsan
def _run_waw_case() -> None:
```
- **EN:** Defines the helper function `_run_waw_case`. Decorators: `run_with_gsan`. Key calls include `torch.zeros`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `_run_waw_case`。 装饰器：`run_with_gsan`。 关键调用包括 `torch.zeros`。 该作用域涉及PyTorch 张量准备与校验。

#### Lines 231-234

```python
    target = torch.zeros(1, dtype=torch.int32, device="cuda")
    scratch = torch.zeros(1, dtype=torch.int32, device="cuda")
    counter = torch.zeros(1, dtype=torch.int32, device="cuda")
    _waw_kernel[(2, )](target, scratch, counter, num_warps=1)
```
- **EN:** Prepares or updates state through `target`, `scratch`, `counter`. Invokes `torch.zeros` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `target`、`scratch`、`counter` 准备或更新状态。 调用 `torch.zeros` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 235-238

```python


@run_with_gsan
def _run_tma_raw_case() -> None:
```
- **EN:** Defines the helper function `_run_tma_raw_case`. Decorators: `run_with_gsan`. Key calls include `torch.zeros`, `triton.set_allocator`, `target.stride`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `_run_tma_raw_case`。 装饰器：`run_with_gsan`。 关键调用包括 `torch.zeros`、`triton.set_allocator`、`target.stride`。 该作用域涉及PyTorch 张量准备与校验。

#### Lines 239-251

```python
    block = 32
    m_size = 35
    n_size = 37
    padded_n = 40
    row_idx = 5
    col_idx = 8

    counter = torch.zeros(1, dtype=torch.int32, device="cuda")
    target_storage = torch.zeros((m_size, padded_n), dtype=torch.int32, device="cuda")
    target = target_storage[:, :n_size]
    scratch = torch.zeros(1, dtype=torch.int32, device="cuda")
    triton.set_allocator(_cuda_byte_allocator)
    _tma_raw_kernel[(2, )](target, scratch, counter, m_size, n_size, row_idx, col_idx, target.stride(0), BLOCK=block)
```
- **EN:** Prepares or updates state through `block`, `m_size`, `n_size`, `padded_n`, `row_idx`, `col_idx`, `counter`, `target_storage`, and 2 more. Invokes `torch.zeros`, `triton.set_allocator`, `target.stride` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `block`、`m_size`、`n_size`、`padded_n`、`row_idx`、`col_idx`、`counter`、`target_storage` 等另外 2 项 准备或更新状态。 调用 `torch.zeros`、`triton.set_allocator`、`target.stride` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 252-255

```python


@run_with_gsan
def _run_host_tma_war_case() -> None:
```
- **EN:** Defines the helper function `_run_host_tma_war_case`. Decorators: `run_with_gsan`. Key calls include `torch.zeros`, `torch.zeros_like`, `TensorDescriptor.from_tensor`, `target.stride`. This scope touches PyTorch tensor setup and checks, tensor/descriptor metadata.
- **CN:** 定义辅助函数 `_run_host_tma_war_case`。 装饰器：`run_with_gsan`。 关键调用包括 `torch.zeros`、`torch.zeros_like`、`TensorDescriptor.from_tensor`、`target.stride`。 该作用域涉及PyTorch 张量准备与校验、张量/描述符元数据。

#### Lines 256-270

```python
    block = 32
    m_size = 35
    n_size = 37
    padded_n = 40
    row_idx = 5
    col_idx = 8

    target_storage = torch.zeros((m_size, padded_n), dtype=torch.int32, device="cuda")
    scratch_storage = torch.zeros_like(target_storage)
    target = target_storage[:, :n_size]
    scratch = scratch_storage[:, :n_size]
    target_desc = TensorDescriptor.from_tensor(target, [block, block])
    scratch_desc = TensorDescriptor.from_tensor(scratch, [block, block])
    counter = torch.zeros(1, dtype=torch.int32, device="cuda")
    _host_tma_war_kernel[(2, )](target, target_desc, scratch_desc, counter, row_idx, col_idx, target.stride(0))
```
- **EN:** Prepares or updates state through `block`, `m_size`, `n_size`, `padded_n`, `row_idx`, `col_idx`, `target_storage`, `scratch_storage`, and 5 more. Invokes `torch.zeros`, `torch.zeros_like`, `TensorDescriptor.from_tensor`, `target.stride` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata.
- **CN:** 通过 `block`、`m_size`、`n_size`、`padded_n`、`row_idx`、`col_idx`、`target_storage`、`scratch_storage` 等另外 5 项 准备或更新状态。 调用 `torch.zeros`、`torch.zeros_like`、`TensorDescriptor.from_tensor`、`target.stride` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据。

### Lines 271-274

```python


@run_with_gsan
def _run_host_tma_gather_war_case() -> None:
```
- **EN:** Defines the helper function `_run_host_tma_gather_war_case`. Decorators: `run_with_gsan`. Key calls include `torch.zeros`, `torch.tensor`, `TensorDescriptor.from_tensor`, `target.stride`, `scratch.stride`. This scope touches PyTorch tensor setup and checks, tensor/descriptor metadata.
- **CN:** 定义辅助函数 `_run_host_tma_gather_war_case`。 装饰器：`run_with_gsan`。 关键调用包括 `torch.zeros`、`torch.tensor`、`TensorDescriptor.from_tensor`、`target.stride`、`scratch.stride`。 该作用域涉及PyTorch 张量准备与校验、张量/描述符元数据。

#### Lines 275-292

```python
    block_x = 8
    block_y = 8
    m_size = 11
    n_size = 13
    padded_m = 16
    padded_n = 16
    row_idx = 5
    y_offset = 8
    x_offsets_values = [5, 7, 9, 10, 1, 3, 11, 13]

    target_storage = torch.zeros((padded_m, padded_n), dtype=torch.int32, device="cuda")
    target = target_storage[:m_size, :n_size]
    x_offsets = torch.tensor(x_offsets_values, dtype=torch.int32, device="cuda")
    target_desc = TensorDescriptor.from_tensor(target, [1, block_y])
    scratch = torch.zeros((block_x, block_y), dtype=torch.int32, device="cuda")
    counter = torch.zeros(1, dtype=torch.int32, device="cuda")
    _host_tma_gather_war_kernel[(2, )](target, target_desc, x_offsets, scratch, counter, row_idx, y_offset,
                                       target.stride(0), scratch.stride(0), scratch.stride(1), BLOCK_X=block_x)
```
- **EN:** Prepares or updates state through `block_x`, `block_y`, `m_size`, `n_size`, `padded_m`, `padded_n`, `row_idx`, `y_offset`, and 7 more. Invokes `torch.zeros`, `torch.tensor`, `TensorDescriptor.from_tensor`, `target.stride`, `scratch.stride` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata.
- **CN:** 通过 `block_x`、`block_y`、`m_size`、`n_size`、`padded_m`、`padded_n`、`row_idx`、`y_offset` 等另外 7 项 准备或更新状态。 调用 `torch.zeros`、`torch.tensor`、`TensorDescriptor.from_tensor`、`target.stride`、`scratch.stride` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据。

### Lines 293-296

```python


@run_with_gsan
def _run_host_tma_scatter_war_case() -> None:
```
- **EN:** Defines the helper function `_run_host_tma_scatter_war_case`. Decorators: `run_with_gsan`. Key calls include `torch.zeros`, `torch.tensor`, `TensorDescriptor.from_tensor`, `src.stride`, `target.stride`, `torch.arange`. This scope touches PyTorch tensor setup and checks, tensor/descriptor metadata.
- **CN:** 定义辅助函数 `_run_host_tma_scatter_war_case`。 装饰器：`run_with_gsan`。 关键调用包括 `torch.zeros`、`torch.tensor`、`TensorDescriptor.from_tensor`、`src.stride`、`target.stride`、`torch.arange`。 该作用域涉及PyTorch 张量准备与校验、张量/描述符元数据。

#### Lines 297-315

```python
    block_x = 8
    block_y = 8
    m_size = 11
    n_size = 13
    padded_m = 16
    padded_n = 16
    row_idx = 5
    y_offset = 8
    x_offsets_values = [5, 7, 9, 10, 1, 3, 11, 13]

    target_storage = torch.zeros((padded_m, padded_n), dtype=torch.int32, device="cuda")
    target = target_storage[:m_size, :n_size]
    x_offsets = torch.tensor(x_offsets_values, dtype=torch.int32, device="cuda")
    target_desc = TensorDescriptor.from_tensor(target, [1, block_y])
    src = torch.arange(1, block_x * block_y + 1, dtype=torch.int32, device="cuda").reshape(block_x, block_y)
    scratch = torch.zeros(1, dtype=torch.int32, device="cuda")
    counter = torch.zeros(1, dtype=torch.int32, device="cuda")
    _host_tma_scatter_war_kernel[(2, )](target, target_desc, x_offsets, src, src.stride(0), src.stride(1), scratch,
                                        counter, row_idx, y_offset, target.stride(0), BLOCK_X=block_x)
```
- **EN:** Prepares or updates state through `block_x`, `block_y`, `m_size`, `n_size`, `padded_m`, `padded_n`, `row_idx`, `y_offset`, and 8 more. Invokes `torch.zeros`, `torch.tensor`, `TensorDescriptor.from_tensor`, `torch.arange`, `src.stride`, `target.stride` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata.
- **CN:** 通过 `block_x`、`block_y`、`m_size`、`n_size`、`padded_m`、`padded_n`、`row_idx`、`y_offset` 等另外 8 项 准备或更新状态。 调用 `torch.zeros`、`torch.tensor`、`TensorDescriptor.from_tensor`、`torch.arange`、`src.stride`、`target.stride` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据。

### Lines 316-319

```python


@run_with_gsan
def _run_host_tma_atomic_flag_publish_case() -> None:
```
- **EN:** Defines the helper function `_run_host_tma_atomic_flag_publish_case`. Decorators: `run_with_gsan`. Key calls include `torch.zeros`, `TensorDescriptor.from_tensor`, `torch.full`. This scope touches PyTorch tensor setup and checks, tensor/descriptor metadata.
- **CN:** 定义辅助函数 `_run_host_tma_atomic_flag_publish_case`。 装饰器：`run_with_gsan`。 关键调用包括 `torch.zeros`、`TensorDescriptor.from_tensor`、`torch.full`。 该作用域涉及PyTorch 张量准备与校验、张量/描述符元数据。

#### Lines 320-325

```python
    flag = torch.zeros((1, 16), dtype=torch.int32, device="cuda")
    flag_desc = TensorDescriptor.from_tensor(flag, [1, 16])
    payload = torch.zeros(1, dtype=torch.int32, device="cuda")
    counter = torch.zeros(1, dtype=torch.int32, device="cuda")
    scratch = torch.full((1, ), -1, dtype=torch.int32, device="cuda")
    _host_tma_atomic_flag_publish_kernel[(2, )](payload, flag, flag_desc, counter, scratch, num_warps=1)
```
- **EN:** Prepares or updates state through `flag`, `flag_desc`, `payload`, `counter`, `scratch`. Invokes `torch.zeros`, `TensorDescriptor.from_tensor`, `torch.full` to execute the test logic. Relevant themes: PyTorch tensor setup and checks, tensor/descriptor metadata.
- **CN:** 通过 `flag`、`flag_desc`、`payload`、`counter`、`scratch` 准备或更新状态。 调用 `torch.zeros`、`TensorDescriptor.from_tensor`、`torch.full` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验、张量/描述符元数据。

### Lines 326-329

```python


@run_with_gsan
def _run_cross_sm_atomic_sync_case(producer_sem: str, consumer_sem: str, scope: str) -> None:
```
- **EN:** Defines the helper function `_run_cross_sm_atomic_sync_case`. Decorators: `run_with_gsan`. Parameters: `producer_sem`, `consumer_sem`, `scope`. Key calls include `torch.zeros`, `torch.full`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `_run_cross_sm_atomic_sync_case`。 装饰器：`run_with_gsan`。 参数：`producer_sem`、`consumer_sem`、`scope`。 关键调用包括 `torch.zeros`、`torch.full`。 该作用域涉及PyTorch 张量准备与校验。

#### Lines 330-343

```python
    payload = torch.zeros(1, dtype=torch.int32, device="cuda")
    flags = torch.zeros(1, dtype=torch.int32, device="cuda")
    counter = torch.zeros(1, dtype=torch.int32, device="cuda")
    scratch = torch.full((1, ), -1, dtype=torch.int32, device="cuda")
    _cross_sm_atomic_sync_kernel[(2, )](
        payload,
        flags,
        counter,
        scratch,
        producer_sem=producer_sem,
        consumer_sem=consumer_sem,
        scope=scope,
        num_warps=1,
    )
```
- **EN:** Prepares or updates state through `payload`, `flags`, `counter`, `scratch`. Invokes `torch.zeros`, `torch.full` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `payload`、`flags`、`counter`、`scratch` 准备或更新状态。 调用 `torch.zeros`、`torch.full` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 344-347

```python


@run_with_gsan
def _run_transitive_atomic_sync_case(release_sem: str, relay_sem: str, scope: str) -> None:
```
- **EN:** Defines the helper function `_run_transitive_atomic_sync_case`. Decorators: `run_with_gsan`. Parameters: `release_sem`, `relay_sem`, `scope`. Key calls include `torch.zeros`, `torch.full`. This scope touches PyTorch tensor setup and checks.
- **CN:** 定义辅助函数 `_run_transitive_atomic_sync_case`。 装饰器：`run_with_gsan`。 参数：`release_sem`、`relay_sem`、`scope`。 关键调用包括 `torch.zeros`、`torch.full`。 该作用域涉及PyTorch 张量准备与校验。

#### Lines 348-363

```python
    payload = torch.zeros(1, dtype=torch.int32, device="cuda")
    flag0 = torch.zeros(1, dtype=torch.int32, device="cuda")
    flag1 = torch.zeros(1, dtype=torch.int32, device="cuda")
    counter = torch.zeros(1, dtype=torch.int32, device="cuda")
    scratch = torch.full((1, ), -1, dtype=torch.int32, device="cuda")
    _transitive_atomic_sync_kernel[(3, )](
        payload,
        flag0,
        flag1,
        counter,
        scratch,
        release_sem=release_sem,
        relay_sem=relay_sem,
        scope=scope,
        num_warps=1,
    )
```
- **EN:** Prepares or updates state through `payload`, `flag0`, `flag1`, `counter`, `scratch`. Invokes `torch.zeros`, `torch.full` to execute the test logic. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 通过 `payload`、`flag0`、`flag1`、`counter`、`scratch` 准备或更新状态。 调用 `torch.zeros`、`torch.full` 执行测试逻辑。 相关主题：PyTorch 张量准备与校验。

### Lines 364-366

```python


def _expected_file_line(source_function, marker: str) -> str:
```
- **EN:** Defines the helper function `_expected_file_line`. Parameters: `source_function`, `marker`. Key calls include `inspect.getsourcelines`, `enumerate`, `AssertionError`, `Path`.
- **CN:** 定义辅助函数 `_expected_file_line`。 参数：`source_function`、`marker`。 关键调用包括 `inspect.getsourcelines`、`enumerate`、`AssertionError`、`Path`。

#### Lines 367-367

```python
    source_lines, starting_line = inspect.getsourcelines(source_function)
```
- **EN:** Prepares or updates state through `source_lines`, `starting_line`. Invokes `inspect.getsourcelines` to execute the test logic.
- **CN:** 通过 `source_lines`、`starting_line` 准备或更新状态。 调用 `inspect.getsourcelines` 执行测试逻辑。

#### Lines 368-370

```python
    for line_offset, line in enumerate(source_lines):
        if marker in line:
            return f"{Path(__file__).name}:{starting_line + line_offset}"
```
- **EN:** Invokes `enumerate`, `Path` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `enumerate`、`Path` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

#### Lines 371-371

```python
    raise AssertionError(f"Could not find marker {marker!r} for function {source_function!r}")
```
- **EN:** Invokes `AssertionError` to execute the test logic.
- **CN:** 调用 `AssertionError` 执行测试逻辑。

### Lines 372-375

```python


def _run_failure_case(case: str, *, runner, source_function, marker: str, error: str, runner_args=(),
                      runner_kwargs=None) -> None:
```
- **EN:** Defines the helper function `_run_failure_case`. Parameters: `case`, `runner`, `source_function`, `marker`, `error`, `runner_args`, `runner_kwargs`. Key calls include `run_in_process`, `isinstance`, `torch.cuda.device_count`, `pytest.skip`, `_expected_file_line`, `Path`. This scope touches PyTorch tensor setup and checks, debugging and inspection paths, runtime driver interaction.
- **CN:** 定义辅助函数 `_run_failure_case`。 参数：`case`、`runner`、`source_function`、`marker`、`error`、`runner_args`、`runner_kwargs`。 关键调用包括 `run_in_process`、`isinstance`、`torch.cuda.device_count`、`pytest.skip`、`_expected_file_line`、`Path`。 该作用域涉及PyTorch 张量准备与校验、调试与检查路径、运行时驱动交互。

#### Lines 376-377

```python
    if torch.cuda.device_count() < 1:
        pytest.skip("requires at least 1 CUDA device")
```
- **EN:** Invokes `torch.cuda.device_count`, `pytest.skip` to execute the test logic. Branches on runtime or test conditions. Relevant themes: PyTorch tensor setup and checks.
- **CN:** 调用 `torch.cuda.device_count`、`pytest.skip` 执行测试逻辑。 根据运行时或测试条件进行分支。 相关主题：PyTorch 张量准备与校验。

#### Lines 378-380

```python

    if runner_kwargs is None:
        runner_kwargs = {}
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 381-383

```python

    result = run_in_process(runner, runner_args, runner_kwargs)
    print(result.driver_stderr_output)
```
- **EN:** Prepares or updates state through `result`. Invokes `run_in_process` to execute the test logic. Relevant themes: debugging and inspection paths, runtime driver interaction.
- **CN:** 通过 `result` 准备或更新状态。 调用 `run_in_process` 执行测试逻辑。 相关主题：调试与检查路径、运行时驱动交互。

#### Lines 384-390

```python
    assert isinstance(result.exc, RuntimeError), (f"case={case} completed without the expected GSan failure\n"
                                                  f"exc={result.exc!r}\n"
                                                  f"driver stderr:\n{result.driver_stderr_output}")
    assert "GSanLibrary.cu" not in result.driver_stderr_output
    assert Path(__file__).name in result.driver_stderr_output
    assert _expected_file_line(source_function, marker) in result.driver_stderr_output
    assert error in result.driver_stderr_output
```
- **EN:** Invokes `isinstance`, `Path`, `_expected_file_line` to execute the test logic. Validates behavior with 5 assertion(s). Relevant themes: runtime driver interaction.
- **CN:** 调用 `isinstance`、`Path`、`_expected_file_line` 执行测试逻辑。 通过 5 个断言验证行为。 相关主题：运行时驱动交互。

### Lines 391-393

```python


def test_read_after_write():
```
- **EN:** Defines the test function `test_read_after_write`. Key calls include `_run_failure_case`. This scope touches Triton language operations.
- **CN:** 定义测试函数 `test_read_after_write`。 关键调用包括 `_run_failure_case`。 该作用域涉及Triton language 操作。

#### Lines 394-395

```python
    _run_failure_case("raw", runner=_run_raw_case, source_function=_raw_kernel.fn, marker="value = tl.load(ptr)",
                      error="Read after write race detected")
```
- **EN:** Invokes `_run_failure_case` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 调用 `_run_failure_case` 执行测试逻辑。 相关主题：Triton language 操作。

### Lines 396-398

```python


def test_write_after_read():
```
- **EN:** Defines the test function `test_write_after_read`. Key calls include `_run_failure_case`. This scope touches Triton language operations.
- **CN:** 定义测试函数 `test_write_after_read`。 关键调用包括 `_run_failure_case`。 该作用域涉及Triton language 操作。

#### Lines 399-400

```python
    _run_failure_case("war", runner=_run_war_case, source_function=_war_kernel.fn, marker="tl.store(ptr, 1)",
                      error="Write after read race detected")
```
- **EN:** Invokes `_run_failure_case` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 调用 `_run_failure_case` 执行测试逻辑。 相关主题：Triton language 操作。

### Lines 401-403

```python


def test_write_after_write():
```
- **EN:** Defines the test function `test_write_after_write`. Key calls include `_run_failure_case`. This scope touches Triton language operations.
- **CN:** 定义测试函数 `test_write_after_write`。 关键调用包括 `_run_failure_case`。 该作用域涉及Triton language 操作。

#### Lines 404-405

```python
    _run_failure_case("waw", runner=_run_waw_case, source_function=_waw_kernel.fn, marker="tl.store(ptr, 2)",
                      error="Write after write race detected")
```
- **EN:** Invokes `_run_failure_case` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 调用 `_run_failure_case` 执行测试逻辑。 相关主题：Triton language 操作。

### Lines 406-408

```python


def test_tma_read_after_write():
```
- **EN:** Defines the test function `test_tma_read_after_write`. Key calls include `_run_failure_case`. This scope touches Triton language operations.
- **CN:** 定义测试函数 `test_tma_read_after_write`。 关键调用包括 `_run_failure_case`。 该作用域涉及Triton language 操作。

#### Lines 409-411

```python
    _run_failure_case("tma_raw", runner=_run_tma_raw_case, source_function=_tma_raw_kernel.fn,
                      marker="value = tl.load(ptr + row_idx * stride_0 + col_idx)",
                      error="Read after write race detected")
```
- **EN:** Invokes `_run_failure_case` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 调用 `_run_failure_case` 执行测试逻辑。 相关主题：Triton language 操作。

### Lines 412-414

```python


def test_host_tma_write_after_read():
```
- **EN:** Defines the test function `test_host_tma_write_after_read`. Key calls include `_run_failure_case`. This scope touches Triton language operations.
- **CN:** 定义测试函数 `test_host_tma_write_after_read`。 关键调用包括 `_run_failure_case`。 该作用域涉及Triton language 操作。

#### Lines 415-417

```python
    _run_failure_case("host_tma_war", runner=_run_host_tma_war_case, source_function=_host_tma_war_kernel.fn,
                      marker="tl.store(target_ptr + row_idx * stride_0 + col_idx, 1)",
                      error="Write after read race detected")
```
- **EN:** Invokes `_run_failure_case` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 调用 `_run_failure_case` 执行测试逻辑。 相关主题：Triton language 操作。

### Lines 418-421

```python


@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_host_tma_gather_write_after_read():
```
- **EN:** Defines the test function `test_host_tma_gather_write_after_read`. Decorators: `pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`. Key calls include `pytest.mark.skipif`, `_run_failure_case`, `is_blackwell`. This scope touches Triton language operations.
- **CN:** 定义测试函数 `test_host_tma_gather_write_after_read`。 装饰器：`pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`。 关键调用包括 `pytest.mark.skipif`、`_run_failure_case`、`is_blackwell`。 该作用域涉及Triton language 操作。

#### Lines 422-425

```python
    _run_failure_case("host_tma_gather_war", runner=_run_host_tma_gather_war_case,
                      source_function=_host_tma_gather_war_kernel.fn,
                      marker="tl.store(target_ptr + row_idx * stride_0 + y_offset, 1)",
                      error="Write after read race detected")
```
- **EN:** Invokes `_run_failure_case` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 调用 `_run_failure_case` 执行测试逻辑。 相关主题：Triton language 操作。

### Lines 426-429

```python


@pytest.mark.skipif(not is_blackwell(), reason="Requires Blackwell")
def test_host_tma_scatter_write_after_read():
```
- **EN:** Defines the test function `test_host_tma_scatter_write_after_read`. Decorators: `pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`. Key calls include `pytest.mark.skipif`, `_run_failure_case`, `is_blackwell`.
- **CN:** 定义测试函数 `test_host_tma_scatter_write_after_read`。 装饰器：`pytest.mark.skipif(not is_blackwell(), reason='Requires Blackwell')`。 关键调用包括 `pytest.mark.skipif`、`_run_failure_case`、`is_blackwell`。

#### Lines 430-432

```python
    _run_failure_case("host_tma_scatter_war", runner=_run_host_tma_scatter_war_case,
                      source_function=_host_tma_scatter_war_kernel.fn,
                      marker="target_desc.scatter(values, x_offsets, y_offset)", error="Write after read race detected")
```
- **EN:** Invokes `_run_failure_case` to execute the test logic.
- **CN:** 调用 `_run_failure_case` 执行测试逻辑。

### Lines 433-436

```python


@pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason="Requires Hopper or newer")
def test_host_tma_atomic_on_release_flag_does_not_publish_data():
```
- **EN:** Defines the test function `test_host_tma_atomic_on_release_flag_does_not_publish_data`. Decorators: `pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires Hopper or newer')`. Key calls include `pytest.mark.skipif`, `_run_failure_case`, `is_cuda`, `torch.cuda.get_device_capability`. This scope touches Triton language operations, PyTorch tensor setup and checks.
- **CN:** 定义测试函数 `test_host_tma_atomic_on_release_flag_does_not_publish_data`。 装饰器：`pytest.mark.skipif(not is_cuda() or torch.cuda.get_device_capability()[0] < 9, reason='Requires Hopper or newer')`。 关键调用包括 `pytest.mark.skipif`、`_run_failure_case`、`is_cuda`、`torch.cuda.get_device_capability`。 该作用域涉及Triton language 操作、PyTorch 张量准备与校验。

#### Lines 437-439

```python
    _run_failure_case("host_tma_atomic_flag_publish", runner=_run_host_tma_atomic_flag_publish_case,
                      source_function=_host_tma_atomic_flag_publish_kernel.fn, marker="result = tl.load(payload_ptr)",
                      error="Read after write race detected")
```
- **EN:** Invokes `_run_failure_case` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 调用 `_run_failure_case` 执行测试逻辑。 相关主题：Triton language 操作。

### Lines 440-443

```python


@pytest.mark.parametrize("producer_sem, consumer_sem, scope", CROSS_SM_SEMANTIC_MISMATCH_CASES)
def test_cross_sm_semantic_mismatch_read_after_write(producer_sem, consumer_sem, scope):
```
- **EN:** Defines the test function `test_cross_sm_semantic_mismatch_read_after_write`. Decorators: `pytest.mark.parametrize('producer_sem, consumer_sem, scope', CROSS_SM_SEMANTIC_MISMATCH_CASES)`. Parameters: `producer_sem`, `consumer_sem`, `scope`. Key calls include `pytest.mark.parametrize`, `_run_failure_case`. This scope touches pytest parametrization, Triton language operations.
- **CN:** 定义测试函数 `test_cross_sm_semantic_mismatch_read_after_write`。 装饰器：`pytest.mark.parametrize('producer_sem, consumer_sem, scope', CROSS_SM_SEMANTIC_MISMATCH_CASES)`。 参数：`producer_sem`、`consumer_sem`、`scope`。 关键调用包括 `pytest.mark.parametrize`、`_run_failure_case`。 该作用域涉及pytest 参数化、Triton language 操作。

#### Lines 444-447

```python
    _run_failure_case(f"cross_sm_semantic_mismatch_{producer_sem}_{consumer_sem}_{scope}",
                      runner=_run_cross_sm_atomic_sync_case, runner_args=(producer_sem, consumer_sem, scope),
                      source_function=_cross_sm_atomic_sync_kernel.fn, marker="result = tl.load(payload_ptr)",
                      error="Read after write race detected")
```
- **EN:** Invokes `_run_failure_case` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 调用 `_run_failure_case` 执行测试逻辑。 相关主题：Triton language 操作。

### Lines 448-451

```python


@pytest.mark.parametrize("producer_sem, consumer_sem", RELEASE_ACQUIRE_SYNC_CASES)
def test_cross_sm_cta_scope_read_after_write(producer_sem, consumer_sem):
```
- **EN:** Defines the test function `test_cross_sm_cta_scope_read_after_write`. Decorators: `pytest.mark.parametrize('producer_sem, consumer_sem', RELEASE_ACQUIRE_SYNC_CASES)`. Parameters: `producer_sem`, `consumer_sem`. Key calls include `pytest.mark.parametrize`, `_run_failure_case`. This scope touches pytest parametrization, Triton language operations.
- **CN:** 定义测试函数 `test_cross_sm_cta_scope_read_after_write`。 装饰器：`pytest.mark.parametrize('producer_sem, consumer_sem', RELEASE_ACQUIRE_SYNC_CASES)`。 参数：`producer_sem`、`consumer_sem`。 关键调用包括 `pytest.mark.parametrize`、`_run_failure_case`。 该作用域涉及pytest 参数化、Triton language 操作。

#### Lines 452-455

```python
    _run_failure_case(f"cross_sm_cta_scope_{producer_sem}_{consumer_sem}", runner=_run_cross_sm_atomic_sync_case,
                      runner_args=(producer_sem, consumer_sem, "cta"), source_function=_cross_sm_atomic_sync_kernel.fn,
                      marker="ready = tl.atomic_add(flag_ptr, 0, sem=consumer_sem, scope=scope)",
                      error="Read after write race detected")
```
- **EN:** Invokes `_run_failure_case` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 调用 `_run_failure_case` 执行测试逻辑。 相关主题：Triton language 操作。

### Lines 456-459

```python


@pytest.mark.parametrize("release_sem, relay_sem, scope", TRANSITIVE_RELAY_MISMATCH_CASES)
def test_transitive_release_acquire_requires_middle_acquire(release_sem, relay_sem, scope):
```
- **EN:** Defines the test function `test_transitive_release_acquire_requires_middle_acquire`. Decorators: `pytest.mark.parametrize('release_sem, relay_sem, scope', TRANSITIVE_RELAY_MISMATCH_CASES)`. Parameters: `release_sem`, `relay_sem`, `scope`. Key calls include `pytest.mark.parametrize`, `_run_failure_case`. This scope touches pytest parametrization, Triton language operations.
- **CN:** 定义测试函数 `test_transitive_release_acquire_requires_middle_acquire`。 装饰器：`pytest.mark.parametrize('release_sem, relay_sem, scope', TRANSITIVE_RELAY_MISMATCH_CASES)`。 参数：`release_sem`、`relay_sem`、`scope`。 关键调用包括 `pytest.mark.parametrize`、`_run_failure_case`。 该作用域涉及pytest 参数化、Triton language 操作。

#### Lines 460-462

```python
    _run_failure_case(f"transitive_sync_{release_sem}_{relay_sem}_{scope}", runner=_run_transitive_atomic_sync_case,
                      runner_args=(release_sem, relay_sem, scope), source_function=_transitive_atomic_sync_kernel.fn,
                      marker="result = tl.load(payload_ptr)", error="Read after write race detected")
```
- **EN:** Invokes `_run_failure_case` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 调用 `_run_failure_case` 执行测试逻辑。 相关主题：Triton language 操作。

### Lines 463-466

```python


@pytest.mark.parametrize("release_sem, relay_sem", RELEASE_ACQUIRE_SYNC_CASES)
def test_transitive_cta_scope_read_after_write(release_sem, relay_sem):
```
- **EN:** Defines the test function `test_transitive_cta_scope_read_after_write`. Decorators: `pytest.mark.parametrize('release_sem, relay_sem', RELEASE_ACQUIRE_SYNC_CASES)`. Parameters: `release_sem`, `relay_sem`. Key calls include `pytest.mark.parametrize`, `_run_failure_case`. This scope touches pytest parametrization, Triton language operations.
- **CN:** 定义测试函数 `test_transitive_cta_scope_read_after_write`。 装饰器：`pytest.mark.parametrize('release_sem, relay_sem', RELEASE_ACQUIRE_SYNC_CASES)`。 参数：`release_sem`、`relay_sem`。 关键调用包括 `pytest.mark.parametrize`、`_run_failure_case`。 该作用域涉及pytest 参数化、Triton language 操作。

#### Lines 467-470

```python
    _run_failure_case(f"transitive_cta_scope_{release_sem}_{relay_sem}", runner=_run_transitive_atomic_sync_case,
                      runner_args=(release_sem, relay_sem, "cta"), source_function=_transitive_atomic_sync_kernel.fn,
                      marker="ready = tl.atomic_add(flag0_ptr, 0, sem=relay_sem, scope=scope)",
                      error="Read after write race detected")
```
- **EN:** Invokes `_run_failure_case` to execute the test logic. Relevant themes: Triton language operations.
- **CN:** 调用 `_run_failure_case` 执行测试逻辑。 相关主题：Triton language 操作。

## Key Concepts / 关键概念

- **EN:** Top-level scopes such as `_raw_kernel`, `_war_kernel`, `_waw_kernel`, `_cross_sm_atomic_sync_kernel`, `_transitive_atomic_sync_kernel`, `_tma_raw_kernel`, `_host_tma_war_kernel`, `_host_tma_gather_war_kernel`
  **CN:** 顶层作用域，例如 `_raw_kernel`、`_war_kernel`、`_waw_kernel`、`_cross_sm_atomic_sync_kernel`、`_transitive_atomic_sync_kernel`、`_tma_raw_kernel`、`_host_tma_war_kernel`、`_host_tma_gather_war_kernel`
- **EN:** pytest parametrization
  **CN:** pytest 参数化
- **EN:** Triton compilation or JIT kernels
  **CN:** Triton 编译或 JIT kernel
- **EN:** Triton language operations
  **CN:** Triton language 操作
- **EN:** PyTorch tensor setup and checks
  **CN:** PyTorch 张量准备与校验
- **EN:** tensor/descriptor metadata
  **CN:** 张量/描述符元数据
- **EN:** debugging and inspection paths
  **CN:** 调试与检查路径
- **EN:** runtime driver interaction
  **CN:** 运行时驱动交互

## Dependencies / 依赖关系

- **EN:** External or absolute imports include `__future__`, `functools`, `inspect`, `pathlib`, `pytest`, `torch`, `triton`, `triton.language`, `triton._internal_testing`, `triton.experimental.gsan`, `triton.experimental.gsan._testing_utils`, `triton.tools.tensor_descriptor`.
  **CN:** 外部或绝对导入包括 `__future__`、`functools`、`inspect`、`pathlib`、`pytest`、`torch`、`triton`、`triton.language`、`triton._internal_testing`、`triton.experimental.gsan`、`triton.experimental.gsan._testing_utils`、`triton.tools.tensor_descriptor`。
- **EN:** Execution centers on top-level definitions such as `_raw_kernel`, `_war_kernel`, `_waw_kernel`, `_cross_sm_atomic_sync_kernel`, `_transitive_atomic_sync_kernel`, `_tma_raw_kernel`, `_host_tma_war_kernel`, `_host_tma_gather_war_kernel`, `_host_tma_scatter_war_kernel`, `_host_tma_atomic_flag_publish_kernel`.
  **CN:** 执行逻辑主要围绕顶层定义展开，例如 `_raw_kernel`、`_war_kernel`、`_waw_kernel`、`_cross_sm_atomic_sync_kernel`、`_transitive_atomic_sync_kernel`、`_tma_raw_kernel`、`_host_tma_war_kernel`、`_host_tma_gather_war_kernel`、`_host_tma_scatter_war_kernel`、`_host_tma_atomic_flag_publish_kernel`。
- **EN:** Runtime behavior also depends on pytest collection, Python execution semantics, and the imported Triton/PyTorch utilities visible above.
  **CN:** 运行时行为还依赖 pytest 的收集机制、Python 执行语义，以及上文可见的 Triton/PyTorch 工具。
