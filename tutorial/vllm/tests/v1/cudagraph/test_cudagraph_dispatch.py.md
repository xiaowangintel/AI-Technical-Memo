# test_cudagraph_dispatch.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/cudagraph/test_cudagraph_dispatch.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `cudagraph dispatch` behavior and regressions in the v1 stack. / 验证 v1 栈中 `cudagraph dispatch` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-2)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 3-24)
```python
from dataclasses import replace
from unittest.mock import MagicMock, patch

import pytest
import torch
import torch.nn as nn

from tests.utils import create_new_process_for_each_test
from vllm.compilation.cuda_graph import CUDAGraphWrapper
from vllm.compilation.monitor import set_cudagraph_capturing_enabled
from vllm.config import (
    CompilationConfig,
    CompilationMode,
    CUDAGraphMode,
    ParallelConfig,
    SchedulerConfig,
    VllmConfig,
)
from vllm.config.lora import LoRAConfig
from vllm.forward_context import BatchDescriptor, set_forward_context
from vllm.platforms import current_platform
from vllm.v1.cudagraph_dispatcher import CudagraphDispatcher
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `pytest, torch, torch.nn`. vLLM modules under test include `vllm.compilation.cuda_graph, vllm.compilation.monitor, vllm.config, vllm.config.lora, vllm.forward_context, ...`. Local helpers come from `tests.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `pytest, torch, torch.nn`。 被测试的 vLLM 模块包括 `vllm.compilation.cuda_graph, vllm.compilation.monitor, vllm.config, vllm.config.lora, vllm.forward_context, ...`。 本地测试辅助逻辑来自 `tests.utils`。

### Module state / 模块级状态 (line 26)
```python
DEVICE_TYPE = current_platform.device_type
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `DEVICE_TYPE`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`DEVICE_TYPE`。

### SimpleMLP (lines 30-37)
```python
class SimpleMLP(nn.Module):
    def __init__(self):
        super().__init__()
        self.fc1 = nn.Linear(10, 10)
        self.fc2 = nn.Linear(10, 10)

    def forward(self, x):
        return self.fc2(self.fc1(x))
```
**EN:** Class `SimpleMLP` groups 0 test method(s) and 2 helper/fixture method(s). Bases: `nn.Module`.
**CN:** 类 `SimpleMLP` 组织了 0 个测试方法，以及 2 个辅助或 fixture 方法。 基类：`nn.Module`。

### _create_vllm_config (lines 40-75)
```python
def _create_vllm_config(
    compilation_config: CompilationConfig,
    max_num_seqs: int = 8,
    lora_config: bool = False,
) -> MagicMock:
    mock_config = MagicMock(spec=VllmConfig)
    mock_config.compilation_config = compilation_config
    mock_config.scheduler_config = SchedulerConfig.default_factory(
        max_num_seqs=max_num_seqs,
    )
    mock_config.parallel_config = ParallelConfig()
    mock_config.speculative_config = None  # No speculative decoding
    if not lora_config:
        mock_config.lora_config = None
    else:
        # Create a real LoRAConfig with specialize_active_lora enabled
        mock_config.lora_config = LoRAConfig(
            max_loras=4,
    # ... excerpt omitted for brevity ...
            compilation_config.cudagraph_capture_sizes[-1]
        )

        compilation_config.post_init_cudagraph_sizes()
    return mock_config
```
**EN:** Helper function `_create_vllm_config` encapsulates reusable logic for `vllm config`. Inputs: `compilation_config, max_num_seqs, lora_config`. Key calls include `MagicMock, SchedulerConfig.default_factory, ParallelConfig, LoRAConfig, compilation_config.set_splitting_ops_for_v1, compilation_config.post_init_cudagraph_sizes`.
**CN:** 辅助函数 `_create_vllm_config` 封装了与 `vllm config` 相关的可复用逻辑。 输入参数：`compilation_config, max_num_seqs, lora_config`。 关键调用包括 `MagicMock, SchedulerConfig.default_factory, ParallelConfig, LoRAConfig, compilation_config.set_splitting_ops_for_v1, compilation_config.post_init_cudagraph_sizes`。

### TestCudagraphDispatcher (lines 78-267)
```python
class TestCudagraphDispatcher:
    @pytest.mark.parametrize(
        "cudagraph_mode_str,compilation_mode,lora_config",
        [
            # Test case 0: Full CG for mixed batches, no separate routine
            ("FULL", CompilationMode.NONE, False),
            # Test case 1: Full CG for uniform batches, piecewise for mixed
            ("FULL_AND_PIECEWISE", CompilationMode.NONE, False),
            # Test case 2: Full CG for uniform batches, no CG for mixed
            ("FULL_DECODE_ONLY", CompilationMode.NONE, False),
            # Test case 3: PIECEWISE for all
            ("PIECEWISE", CompilationMode.VLLM_COMPILE, False),
            # Test case 4: PIECEWISE for all, specialize LoRA cases
            ("PIECEWISE", CompilationMode.VLLM_COMPILE, True),
        ],
    )
    def test_dispatcher(self, cudagraph_mode_str, compilation_mode, lora_config):
        # Setup dispatcher
    # ... excerpt omitted for brevity ...
            with pytest.raises(AssertionError):
            assert len(dispatcher.cudagraph_keys[CUDAGraphMode.PIECEWISE]) == (
            assert len(dispatcher.cudagraph_keys[CUDAGraphMode.PIECEWISE]) == 0
            assert len(dispatcher.cudagraph_keys[CUDAGraphMode.FULL]) == (
            assert len(dispatcher.cudagraph_keys[CUDAGraphMode.FULL]) == 0
            assert rt_mode == CUDAGraphMode.FULL
        )
        config = _create_vllm_config(comp_config, max_num_seqs=8)
        dispatcher = CudagraphDispatcher(config)
        # Don't initialize keys

        assert dispatcher.get_capture_descs() == []
```
**EN:** Class `TestCudagraphDispatcher` groups 3 test method(s). Representative scenarios: `test_dispatcher, test_get_capture_descs, test_get_capture_descs_empty_when_not_initialized`.
**CN:** 类 `TestCudagraphDispatcher` 组织了 3 个测试方法。 代表性场景：`test_dispatcher, test_get_capture_descs, test_get_capture_descs_empty_when_not_initialized`。

### TestCUDAGraphWrapper (lines 271-371)
```python
class TestCUDAGraphWrapper:
    def setup_method(self):
        self.vllm_config = _create_vllm_config(CompilationConfig())
        self.model = SimpleMLP().to(DEVICE_TYPE)
        self.persistent_input_buffer = torch.zeros(1, 10, device=DEVICE_TYPE)
        self.input_tensor = torch.randn(1, 10, device=DEVICE_TYPE)

    def test_capture_and_replay(self):
        wrapper = CUDAGraphWrapper(
            self.model, self.vllm_config, runtime_mode=CUDAGraphMode.FULL
        )
        batch_descriptor = BatchDescriptor(num_tokens=10)
        # 0. global warmup
        with set_forward_context(
            attn_metadata=None,
            vllm_config=self.vllm_config,
            cudagraph_runtime_mode=CUDAGraphMode.NONE,
    # ... excerpt omitted for brevity ...
            assert torch.allclose(output1, torch.zeros_like(output1))
        assert batch_descriptor in wrapper.concrete_cudagraph_entries
        assert entry.cudagraph is not None
        assert not wrapper.concrete_cudagraph_entries
            ),
            patch("torch.cuda.graph", wraps=torch.cuda.graph) as mock_cuda_graph,
        ):
            wrapper(self.input_tensor)
            mock_cuda_graph.assert_not_called()
```
**EN:** Class `TestCUDAGraphWrapper` groups 3 test method(s) and 1 helper/fixture method(s). Representative scenarios: `test_capture_and_replay, test_bypass_on_mode_mismatch, test_bypass_on_mode_none`.
**CN:** 类 `TestCUDAGraphWrapper` 组织了 3 个测试方法，以及 1 个辅助或 fixture 方法。 代表性场景：`test_capture_and_replay, test_bypass_on_mode_mismatch, test_bypass_on_mode_none`。

### _run_and_monitor_call (lines 374-414)
```python
def _run_and_monitor_call(
    wrapper, input_tensor, runtime_mode, batch_descriptor, vllm_config
):
    """Helper to run a single call and monitor the action."""

    with (
        patch("torch.cuda.graph", wraps=torch.cuda.graph) as mock_graph_context,
        patch.object(wrapper, "runnable", wraps=wrapper.runnable) as mock_runnable,
    ):
        entry = wrapper.concrete_cudagraph_entries.get(batch_descriptor, None)
        context = set_forward_context(
            attn_metadata=None,
            vllm_config=vllm_config,
            cudagraph_runtime_mode=runtime_mode,
            batch_descriptor=batch_descriptor,
        )
        mock_replay = MagicMock()
    # ... excerpt omitted for brevity ...
            return "capture_global"
            # only for outer wrapper
            return "replay"
        if mock_runnable.call_count > 0:
            return "bypass"
        return "unknown"
```
**EN:** Helper function `_run_and_monitor_call` encapsulates reusable logic for `run and monitor call`. Inputs: `wrapper, input_tensor, runtime_mode, batch_descriptor, vllm_config`. Key calls include `patch, patch.object, concrete_cudagraph_entries.get, set_forward_context, MagicMock, wrapper`.
**CN:** 辅助函数 `_run_and_monitor_call` 封装了与 `run and monitor call` 相关的可复用逻辑。 输入参数：`wrapper, input_tensor, runtime_mode, batch_descriptor, vllm_config`。 关键调用包括 `patch, patch.object, concrete_cudagraph_entries.get, set_forward_context, MagicMock, wrapper`。

### test_capture_replay_bypass_logic (lines 419-483)
```python
def test_capture_replay_bypass_logic():
    comp_config = CompilationConfig(
        mode=CompilationMode.VLLM_COMPILE,
        cudagraph_mode="FULL",
        cudagraph_capture_sizes=[1, 2],
    )
    vllm_config = _create_vllm_config(comp_config)
    dispatcher = CudagraphDispatcher(vllm_config)
    dispatcher.initialize_cudagraph_keys(
        comp_config.cudagraph_mode, uniform_decode_query_len=1
    model = SimpleMLP().to(DEVICE_TYPE)
    full_wrapper = CUDAGraphWrapper(model, vllm_config, CUDAGraphMode.FULL)
    max_bs = 16
    persistent_input_buffer = torch.zeros(max_bs, 10, device=DEVICE_TYPE)
    input_1 = persistent_input_buffer[:1]
    input_2 = persistent_input_buffer[:2]
    input_3 = persistent_input_buffer[:3]
    # ... excerpt omitted for brevity ...
    assert action == "capture_global"
    assert action == "replay"
    assert rt_mode == CUDAGraphMode.NONE
    assert action == "bypass"
    set_cudagraph_capturing_enabled(False)
    with pytest.raises(RuntimeError):
        _run_and_monitor_call(
            full_wrapper, input_3, CUDAGraphMode.FULL, desc_3_unseen, vllm_config
        )
    set_cudagraph_capturing_enabled(True)
```
**EN:** Test case covering `capture replay bypass logic`. It exercises `create_new_process_for_each_test, mark.skipif, CompilationConfig, _create_vllm_config, CudagraphDispatcher, dispatcher.initialize_cudagraph_keys`. The body contains 6 explicit assertion(s).
**CN:** 该代码块是覆盖 `capture replay bypass logic` 的测试用例。 该测试会调用 `create_new_process_for_each_test, mark.skipif, CompilationConfig, _create_vllm_config, CudagraphDispatcher, dispatcher.initialize_cudagraph_keys`。 代码主体包含 6 个显式断言。

### test_nested_wrappers (lines 488-570)
```python
def test_nested_wrappers():
    """Tests a scenario with a PIECEWISE wrapper inside a FULL one."""
    comp_config = CompilationConfig(
        mode=CompilationMode.VLLM_COMPILE,
        cudagraph_mode="FULL",
        cudagraph_capture_sizes=[1],
    )
    vllm_config = _create_vllm_config(comp_config)
    dispatcher = CudagraphDispatcher(vllm_config)
    dispatcher.initialize_cudagraph_keys(
        comp_config.cudagraph_mode, uniform_decode_query_len=1
    model = SimpleMLP().to(DEVICE_TYPE)
    full_wrapper = CUDAGraphWrapper(model, vllm_config, CUDAGraphMode.FULL)
    input_1 = torch.randn(1, 10, device=DEVICE_TYPE)

    # Setup: Inner model is wrapped with PIECEWISE, outer with FULL
    inner_model = SimpleMLP().to(DEVICE_TYPE)
    # ... excerpt omitted for brevity ...
    inner_model.forward = MagicMock(wraps=inner_model.forward)
    outer_model.forward = MagicMock(
    assert action == "capture_global"
    assert outer_model.forward.call_count == 1
    assert inner_model.forward.call_count == 1
    assert action == "replay"
    action = _run_and_monitor_call(
        full_wrapper, input_1, CUDAGraphMode.PIECEWISE, desc_1, vllm_config
    assert action == "bypass"
    assert outer_model.forward.call_count == 2
```
**EN:** Test case covering `nested wrappers`. It exercises `create_new_process_for_each_test, mark.skipif, CompilationConfig, _create_vllm_config, CudagraphDispatcher, dispatcher.initialize_cudagraph_keys`. The body contains 12 explicit assertion(s).
**CN:** 该代码块是覆盖 `nested wrappers` 的测试用例。 该测试会调用 `create_new_process_for_each_test, mark.skipif, CompilationConfig, _create_vllm_config, CudagraphDispatcher, dispatcher.initialize_cudagraph_keys`。 代码主体包含 12 个显式断言。

## Key Concepts / 关键概念
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为

## Dependencies / 依赖关系
- **EN:** External libraries: `pytest, torch, torch.nn`.
- **CN:** 外部库：`pytest, torch, torch.nn`。
- **EN:** vLLM modules under test: `vllm.compilation.cuda_graph, vllm.compilation.monitor, vllm.config, vllm.config.lora, vllm.forward_context, vllm.platforms, vllm.v1.cudagraph_dispatcher`.
- **CN:** 被测试的 vLLM 模块：`vllm.compilation.cuda_graph, vllm.compilation.monitor, vllm.config, vllm.config.lora, vllm.forward_context, vllm.platforms, vllm.v1.cudagraph_dispatcher`。
- **EN:** Local test helpers: `tests.utils`.
- **CN:** 本地测试辅助模块：`tests.utils`。
- **EN:** Standard-library support: `dataclasses, unittest.mock`.
- **CN:** 标准库支持：`dataclasses, unittest.mock`。
