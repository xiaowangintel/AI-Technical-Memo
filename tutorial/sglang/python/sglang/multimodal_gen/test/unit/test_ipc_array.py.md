# test_ipc_array.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/unit/test_ipc_array.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates ipc array with focused assertions and fixtures. Key symbols include `test_spill_large_arrays_round_trips_and_removes_file`, `test_small_arrays_are_kept_inline`, `test_large_arrays_are_kept_inline_without_shm`. / 该测试模块通过有针对性的断言与夹具，验证 ipc array 的实现。 关键符号包括 `test_spill_large_arrays_round_trips_and_removes_file`, `test_small_arrays_are_kept_inline`, `test_large_arrays_are_kept_inline_without_shm`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0

import tempfile
from pathlib import Path

import numpy as np
import pytest

from sglang.multimodal_gen.runtime import ipc_array
from sglang.multimodal_gen.runtime.ipc_array import (
    NumpyArrayFileRef,
    is_local_endpoint,
    materialize_file_refs,
    spill_large_arrays_to_file_refs,
)
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 18-31: Function `test_spill_large_arrays_round_trips_and_removes_file` / 函数 `test_spill_large_arrays_round_trips_and_removes_file`
```python
def test_spill_large_arrays_round_trips_and_removes_file(monkeypatch, tmp_path):
    monkeypatch.setattr(ipc_array, "_array_ipc_dir", lambda: str(tmp_path))
    array = np.arange(ipc_array._MIN_FILE_REF_BYTES, dtype=np.uint8)

    spilled = spill_large_arrays_to_file_refs([array])

    assert isinstance(spilled[0], NumpyArrayFileRef)
    spilled_path = Path(spilled[0].path)
    assert spilled_path.exists()

    materialized = materialize_file_refs(spilled)

    assert np.array_equal(materialized[0], array)
    assert not spilled_path.exists()
```
**EN:** This function drives `test_spill_large_arrays_round_trips_and_removes_file` with inputs such as `monkeypatch`, `tmp_path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `test_spill_large_arrays_round_trips_and_removes_file`，主要处理 `monkeypatch`, `tmp_path` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 34-39: Function `test_small_arrays_are_kept_inline` / 函数 `test_small_arrays_are_kept_inline`
```python
def test_small_arrays_are_kept_inline():
    array = np.arange(16, dtype=np.uint8)

    spilled = spill_large_arrays_to_file_refs((array,))

    assert spilled[0] is array
```
**EN:** This function drives `test_small_arrays_are_kept_inline`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `test_small_arrays_are_kept_inline`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 42-48: Function `test_large_arrays_are_kept_inline_without_shm` / 函数 `test_large_arrays_are_kept_inline_without_shm`
```python
def test_large_arrays_are_kept_inline_without_shm(monkeypatch):
    monkeypatch.setattr(ipc_array, "_array_ipc_dir", lambda: None)
    array = np.arange(ipc_array._MIN_FILE_REF_BYTES, dtype=np.uint8)

    spilled = spill_large_arrays_to_file_refs(array)

    assert spilled is array
```
**EN:** This function drives `test_large_arrays_are_kept_inline_without_shm` with inputs such as `monkeypatch`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `test_large_arrays_are_kept_inline_without_shm`，主要处理 `monkeypatch` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 51-73: Function `test_spill_removes_temp_file_when_save_fails` / 函数 `test_spill_removes_temp_file_when_save_fails`
```python
def test_spill_removes_temp_file_when_save_fails(monkeypatch, tmp_path):
    monkeypatch.setattr(ipc_array, "_array_ipc_dir", lambda: str(tmp_path))
    array = np.arange(ipc_array._MIN_FILE_REF_BYTES, dtype=np.uint8)
    created_paths = []

    def fail_save(*args, **kwargs):
        raise OSError("simulated write failure")

    original_mkstemp = tempfile.mkstemp

    def tracked_mkstemp(*args, **kwargs):
        fd, path = original_mkstemp(*args, **kwargs)
        created_paths.append(Path(path))
        return fd, path
# ...
        spill_large_arrays_to_file_refs(array)

    assert created_paths
    assert not created_paths[0].exists()
```
**EN:** This function drives `test_spill_removes_temp_file_when_save_fails` with inputs such as `monkeypatch`, `tmp_path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `test_spill_removes_temp_file_when_save_fails`，主要处理 `monkeypatch`, `tmp_path` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 76-81: Function `test_local_endpoint_detection` / 函数 `test_local_endpoint_detection`
```python
def test_local_endpoint_detection():
    assert is_local_endpoint("tcp://127.0.0.1:30000")
    assert is_local_endpoint("tcp://localhost:30000")
    assert is_local_endpoint("ipc:///tmp/sgl.sock")
    assert is_local_endpoint("inproc://scheduler")
    assert not is_local_endpoint("tcp://10.0.0.2:30000")
```
**EN:** This function drives `test_local_endpoint_detection`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `test_local_endpoint_detection`。 这一段实现了该操作的主要控制流、校验和数据传递。

## Key Concepts / 关键概念
- Scheduling and batching / 调度与批处理
- Automated verification / 自动化验证
- Pytest-based assertions / 基于 Pytest 的断言
- Symbol `test_spill_large_arrays_round_trips_and_removes_file` anchors the module API / 符号 `test_spill_large_arrays_round_trips_and_removes_file` 构成该模块的核心 API
- Symbol `test_small_arrays_are_kept_inline` anchors the module API / 符号 `test_small_arrays_are_kept_inline` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime`, `sglang.multimodal_gen.runtime.ipc_array`
- **External / 外部**: `numpy`, `pytest`
- **Stdlib / 标准库**: `tempfile`, `pathlib`
