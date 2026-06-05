# test_numa_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/utils_/test_numa_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Numa Utils behavior in the Utils test area through focused pytest scenarios. It focuses on scenarios such as Make Config, Get Numactl Args With Node Binding, Get Numactl Args With CPU Binding. / 该文件在 Utils 测试域中，通过有针对性的 pytest 场景验证 Numa Utils 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-10)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import os
from types import SimpleNamespace

import pytest

from vllm.config import ParallelConfig
from vllm.utils import numa_utils
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `os`, `types`, `pytest`, `vllm.config`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: _make_config (lines 13-28)
```python
def _make_config(**parallel_kwargs):
    parallel_defaults = dict(
        numa_bind=False,
        numa_bind_nodes=None,
        numa_bind_cpus=None,
        distributed_executor_backend="mp",
        data_parallel_backend="mp",
        nnodes_within_dp=1,
        data_parallel_rank_local=0,
        data_parallel_index=0,
        pipeline_parallel_size=1,
        tensor_parallel_size=1,
    )
    parallel_defaults.update(parallel_kwargs)
    parallel_config = SimpleNamespace(**parallel_defaults)
    return SimpleNamespace(parallel_config=parallel_config)
```
**EN:** Implements a reusable helper for Make Config, reducing duplication across related tests. It coordinates operations such as `dict`, `parallel_defaults.update`, `SimpleNamespace`.
**CN:** 该辅助函数为 Make Config 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `dict`, `parallel_defaults.update`, `SimpleNamespace` 等操作。

### Test: test_get_numactl_args_with_node_binding (lines 31-36)
```python
def test_get_numactl_args_with_node_binding():
    vllm_config = _make_config(numa_bind=True, numa_bind_nodes=[0, 1])
    assert (
        numa_utils._get_numactl_args(vllm_config, local_rank=1)
        == "--cpunodebind=1 --membind=1"
    )
```
**EN:** Checks Get Numactl Args With Node Binding under a focused test scenario. The body exercises logic via `_make_config`, `numa_utils._get_numactl_args` before asserting the expected outcome.
**CN:** 该测试用例验证 Get Numactl Args With Node Binding 在特定场景下的行为。 函数体会先通过 `_make_config`, `numa_utils._get_numactl_args` 驱动目标逻辑，再断言预期结果。

### Test: test_get_numactl_args_with_cpu_binding (lines 39-48)
```python
def test_get_numactl_args_with_cpu_binding():
    vllm_config = _make_config(
        numa_bind=True,
        numa_bind_nodes=[0, 1],
        numa_bind_cpus=["0-3", "4-7"],
    )
    assert (
        numa_utils._get_numactl_args(vllm_config, local_rank=1)
        == "--physcpubind=4-7 --membind=1"
    )
```
**EN:** Checks Get Numactl Args With CPU Binding under a focused test scenario. The body exercises logic via `_make_config`, `numa_utils._get_numactl_args` before asserting the expected outcome.
**CN:** 该测试用例验证 Get Numactl Args With CPU Binding 在特定场景下的行为。 函数体会先通过 `_make_config`, `numa_utils._get_numactl_args` 驱动目标逻辑，再断言预期结果。

### Test: test_get_numactl_args_uses_dp_offset (lines 51-62)
```python
def test_get_numactl_args_uses_dp_offset():
    vllm_config = _make_config(
        numa_bind=True,
        numa_bind_nodes=[0, 0, 1, 1],
        data_parallel_rank_local=1,
        pipeline_parallel_size=1,
        tensor_parallel_size=2,
    )
    assert (
        numa_utils._get_numactl_args(vllm_config, local_rank=1)
        == "--cpunodebind=1 --membind=1"
    )
```
**EN:** Checks Get Numactl Args Uses Dp Offset under a focused test scenario. The body exercises logic via `_make_config`, `numa_utils._get_numactl_args` before asserting the expected outcome.
**CN:** 该测试用例验证 Get Numactl Args Uses Dp Offset 在特定场景下的行为。 函数体会先通过 `_make_config`, `numa_utils._get_numactl_args` 驱动目标逻辑，再断言预期结果。

### Test: test_get_numactl_args_requires_detectable_nodes (lines 65-69)
```python
def test_get_numactl_args_requires_detectable_nodes(monkeypatch):
    vllm_config = _make_config(numa_bind=True)
    monkeypatch.setattr(numa_utils, "get_auto_numa_nodes", lambda: None)
    with pytest.raises(RuntimeError):
        numa_utils._get_numactl_args(vllm_config, local_rank=0)
```
**EN:** Checks Get Numactl Args Requires Detectable Nodes under a focused test scenario. The body exercises logic via `_make_config`, `monkeypatch.setattr`, `pytest.raises` before asserting the expected outcome.
**CN:** 该测试用例验证 Get Numactl Args Requires Detectable Nodes 在特定场景下的行为。 函数体会先通过 `_make_config`, `monkeypatch.setattr`, `pytest.raises` 驱动目标逻辑，再断言预期结果。

### Test: test_log_numactl_show (lines 72-90)
```python
def test_log_numactl_show(monkeypatch):
    log_lines = []

    def fake_debug(msg, *args):
        log_lines.append(msg % args)

    monkeypatch.setattr(numa_utils.logger, "debug", fake_debug)
    monkeypatch.setattr(
        numa_utils.subprocess,
        "run",
        lambda *args, **kwargs: SimpleNamespace(
            stdout="policy: bind\nphyscpubind: 0 1 2 3\n", returncode=0
        ),
    )

    assert numa_utils._log_numactl_show("Worker_0") is True
    assert log_lines == [
        "Worker_0 affinity: policy: bind, physcpubind: 0 1 2 3",
    ]
```
**EN:** Checks Log Numactl Show under a focused test scenario. The body exercises logic via `monkeypatch.setattr`, `log_lines.append`, `numa_utils._log_numactl_show` before asserting the expected outcome.
**CN:** 该测试用例验证 Log Numactl Show 在特定场景下的行为。 函数体会先通过 `monkeypatch.setattr`, `log_lines.append`, `numa_utils._log_numactl_show` 驱动目标逻辑，再断言预期结果。

### Test: test_get_numactl_executable_points_to_fixed_wrapper (lines 93-97)
```python
def test_get_numactl_executable_points_to_fixed_wrapper(monkeypatch):
    monkeypatch.setattr("shutil.which", lambda name: "/usr/bin/numactl")
    executable, debug_str = numa_utils._get_numactl_executable()
    assert executable.endswith("/vllm/utils/numa_wrapper.sh")
    assert "_VLLM_INTERNAL_NUMACTL_ARGS" in debug_str
```
**EN:** Checks Get Numactl Executable Points To Fixed Wrapper under a focused test scenario. The body exercises logic via `monkeypatch.setattr`, `numa_utils._get_numactl_executable`, `executable.endswith` before asserting the expected outcome.
**CN:** 该测试用例验证 Get Numactl Executable Points To Fixed Wrapper 在特定场景下的行为。 函数体会先通过 `monkeypatch.setattr`, `numa_utils._get_numactl_executable`, `executable.endswith` 驱动目标逻辑，再断言预期结果。

### Additional scenarios (summary)
```python
test_set_numa_wrapper_env_restores_previous_values
test_set_numa_wrapper_env_clears_values_when_unset
test_parallel_config_validates_numa_bind_nodes
test_parallel_config_rejects_invalid_numa_bind_cpus
```
**EN:** The file contains more helper or test blocks than can be shown compactly here. This summary lists additional scenarios that continue the same validation strategy.
**CN:** 该文件包含的辅助逻辑或测试场景较多，无法在此完整展开；这里列出其余场景名称，便于快速把握覆盖范围。

## Key Concepts / 关键概念
- **Parameterized testing / 参数化测试**
  - **EN:** The file expands coverage across many argument combinations instead of a single hard-coded case.
  - **CN:** 该文件通过多组参数组合扩展覆盖面，而不是只验证一个固定样例。
- **Mocking and patching / 模拟与补丁**
  - **EN:** Several scenarios replace dependencies or environment state so the tests can isolate one behavior at a time.
  - **CN:** 多个场景会替换依赖或环境状态，从而把验证范围限定在单一行为上。
- **Distributed coordination / 分布式协同**
  - **EN:** The file validates multi-process or multi-worker coordination primitives such as collectives and rank-aware setup.
  - **CN:** 该文件验证多进程或多工作节点之间的协同原语，例如集合通信和基于 rank 的初始化。
- **Configuration / 配置**
  - **EN:** The file checks how options, environment variables, and defaults are interpreted.
  - **CN:** 该文件检查选项、环境变量与默认值如何被解析和生效。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `types`
- **Third-party / 第三方依赖**: `pytest`
- **vLLM internal / vLLM 内部依赖**: `vllm.config`, `vllm.utils`
