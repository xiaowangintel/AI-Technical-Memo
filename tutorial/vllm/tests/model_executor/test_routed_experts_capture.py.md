# test_routed_experts_capture.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/model_executor/test_routed_experts_capture.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Exercises Routed Experts Capture behavior in the Model Executor test area through focused pytest scenarios. It focuses on scenarios such as Capturer With Buffer, Dummyrouter, Make Router. / 该文件在 Model Executor 测试域中，通过有针对性的 pytest 场景验证 Routed Experts Capture 的行为。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-19)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import types
from types import SimpleNamespace
from unittest.mock import patch

import pytest
import torch

from vllm.distributed.eplb.eplb_state import EplbLayerState
from vllm.model_executor.layers.fused_moe.config import RoutingMethodType
from vllm.model_executor.layers.fused_moe.routed_experts_capturer import (
    RoutedExpertsCapturer,
)
from vllm.model_executor.layers.fused_moe.router.base_router import BaseRouter

pytestmark = pytest.mark.cpu_test

_REC_MODULE = "vllm.model_executor.layers.fused_moe.routed_experts_capturer"
```
**EN:** Sets up the test module with imports, shared constants, and reusable configuration such as `types`, `unittest.mock`, `pytest`, `torch`. These definitions prepare the fixtures and test cases that follow.
**CN:** 这一部分完成测试模块初始化，组织导入、共享常量与可复用配置，为后续夹具和测试用例做准备。

### Helper: _capturer_with_buffer (lines 22-41)
```python
def _capturer_with_buffer(
    *,
    max_tokens: int = 8,
    num_layers: int = 4,
    num_experts_per_tok: int = 2,
    dp_rank: int = 0,
    tp_size: int = 1,
) -> RoutedExpertsCapturer:
    # Bypass __init__ so the test can use a CPU buffer and skip the
    # VllmConfig dependency. The CUDA device-tensor allocation in the
    # real constructor is not what we are exercising here.
    c = RoutedExpertsCapturer.__new__(RoutedExpertsCapturer)
    c.dp_rank = dp_rank
    c.tp_size = tp_size
    c.device_buffer = torch.full(
        (max_tokens, num_layers, num_experts_per_tok),
        -1,
        dtype=torch.int32,
    )
    return c
```
**EN:** Implements a reusable helper for Capturer With Buffer, reducing duplication across related tests. It coordinates operations such as `RoutedExpertsCapturer.__new__`, `torch.full`.
**CN:** 该辅助函数为 Capturer With Buffer 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `RoutedExpertsCapturer.__new__`, `torch.full` 等操作。

### Class: DummyRouter (lines 44-58)
```python
class DummyRouter(BaseRouter):
    @property
    def routing_method_type(self) -> RoutingMethodType:
        return RoutingMethodType.FUSED_TOPK

    def _compute_routing(
        self, hidden_states, router_logits, indices_type, *, input_ids=None
    ):
        topk_ids = torch.tensor([[1, 2], [3, 4]], dtype=torch.int64)
        topk_weights = torch.ones_like(topk_ids, dtype=torch.float32)
        return topk_weights, topk_ids

    def _apply_eplb_mapping(self, topk_ids: torch.Tensor) -> torch.Tensor:
        # Make mapping observable without requiring CUDA EPLB path.
        return topk_ids + 10
```
**EN:** Groups related scenarios for Dummyrouter.
**CN:** 该类把与 Dummyrouter 相关的场景组织在一起。

### Helper: _make_router (lines 61-67)
```python
def _make_router(eplb_state: EplbLayerState | None = None) -> DummyRouter:
    return DummyRouter(
        top_k=2,
        global_num_experts=16,
        eplb_state=eplb_state,
        indices_type_getter=None,
    )
```
**EN:** Implements a reusable helper for Make Router, reducing duplication across related tests. It coordinates operations such as `DummyRouter`.
**CN:** 该辅助函数为 Make Router 提供可复用逻辑，用于减少相关测试之间的重复代码。 它会协调 `DummyRouter` 等操作。

### Test: test_base_router_capture_pre_eplb_mapping (lines 70-86)
```python
def test_base_router_capture_pre_eplb_mapping():
    router = _make_router()
    captured = []

    def capture_fn(ids):
        captured.append(ids.clone())

    router.set_capture_fn(capture_fn)
    topk_weights, topk_ids = router.select_experts(
        hidden_states=torch.empty(1),
        router_logits=torch.empty(1),
    )

    assert topk_weights.shape == topk_ids.shape
    assert len(captured) == 1
    assert torch.equal(captured[0], torch.tensor([[1, 2], [3, 4]]))
    assert torch.equal(topk_ids, torch.tensor([[11, 12], [13, 14]]))
```
**EN:** Checks Base Router Capture Pre Eplb Mapping under a focused test scenario. The body exercises logic via `_make_router`, `router.set_capture_fn`, `router.select_experts` before asserting the expected outcome.
**CN:** 该测试用例验证 Base Router Capture Pre Eplb Mapping 在特定场景下的行为。 函数体会先通过 `_make_router`, `router.set_capture_fn`, `router.select_experts` 驱动目标逻辑，再断言预期结果。

### Test: test_base_router_capture_with_eplb_enabled (lines 89-112)
```python
def test_base_router_capture_with_eplb_enabled():
    eplb_state = EplbLayerState()
    eplb_state.expert_load_view = torch.zeros(32, dtype=torch.int64)
    eplb_state.logical_to_physical_map = torch.arange(32).view(32, 1)
    eplb_state.logical_replica_count = torch.ones(32, dtype=torch.int64)
    eplb_state.should_record_tensor = torch.ones((), dtype=torch.bool)
    router = _make_router(eplb_state=eplb_state)

    captured = []

    def capture_fn(ids):
        captured.append(ids.clone())

    router.set_capture_fn(capture_fn)
    _, topk_ids = router.select_experts(
        hidden_states=torch.empty(1),
        router_logits=torch.empty(1),
    )

    assert len(captured) == 1
    # Capture should see logical ids pre-EPLB mapping.
    assert torch.equal(captured[0], torch.tensor([[1, 2], [3, 4]]))
    # Our DummyRouter mapping adds +10.
    assert torch.equal(topk_ids, torch.tensor([[11, 12], [13, 14]]))
```
**EN:** Checks Base Router Capture With Eplb Enabled under a focused test scenario. The body exercises logic via `EplbLayerState`, `torch.zeros`, `torch.arange(32).view` before asserting the expected outcome.
**CN:** 该测试用例验证 Base Router Capture With Eplb Enabled 在特定场景下的行为。 函数体会先通过 `EplbLayerState`, `torch.zeros`, `torch.arange(32).view` 驱动目标逻辑，再断言预期结果。

### Test: test_gpu_model_runner_binds_router_capture (lines 115-152)
```python
def test_gpu_model_runner_binds_router_capture(monkeypatch):
    from vllm.v1.worker import gpu_model_runner as gmr

    class DummyFusedMoE:
        def __init__(self):
            self.layer_id = 7
            self.router = _make_router()

    class DummyCapturer:
        def __init__(self):
            self.calls = []

        def capture(self, layer_id, topk_ids):
            self.calls.append((layer_id, topk_ids))

    dummy_module = DummyFusedMoE()

    # Patch the runtime import inside _bind_routed_experts_capturer.
    import vllm.model_executor.layers.fused_moe.layer as fused_moe_layer
# ... omitted for brevity ...
            static_forward_context={"dummy": dummy_module}
        )
    )

    capturer = DummyCapturer()
    gmr.GPUModelRunner._bind_routed_experts_capturer(dummy_self, capturer)

    assert dummy_module.router.capture_fn is not None
    dummy_module.router.capture_fn(torch.tensor([[5, 6]]))

    assert len(capturer.calls) == 1
    layer_id, topk_ids = capturer.calls[0]
    assert layer_id == 7
    assert torch.equal(topk_ids, torch.tensor([[5, 6]]))
```
**EN:** Checks GPU Model Runner Binds Router Capture under a focused test scenario. The body exercises logic via `DummyFusedMoE`, `monkeypatch.setattr`, `types.SimpleNamespace` before asserting the expected outcome.
**CN:** 该测试用例验证 GPU Model Runner Binds Router Capture 在特定场景下的行为。 函数体会先通过 `DummyFusedMoE`, `monkeypatch.setattr`, `types.SimpleNamespace` 驱动目标逻辑，再断言预期结果。

### Test: test_gpu_model_runner_binding_stage (lines 155-191)
```python
def test_gpu_model_runner_binding_stage(monkeypatch):
    from vllm.v1.worker import gpu_model_runner as gmr

    class DummyFusedMoE:
        def __init__(self):
            self.layer_id = 11
            self.router = _make_router()

    class DummyCapturer:
        def __init__(self):
            self.calls = []

        def capture(self, layer_id, topk_ids):
            self.calls.append((layer_id, topk_ids))

    dummy_module = DummyFusedMoE()

    import vllm.model_executor.layers.fused_moe.layer as fused_moe_layer

# ... omitted for brevity ...
            static_forward_context={"dummy": dummy_module}
        )
    )

    # Before binding, no capture hook.
    assert dummy_module.router.capture_fn is None

    capturer = DummyCapturer()
    gmr.GPUModelRunner._bind_routed_experts_capturer(dummy_self, capturer)

    # After binding, hook should exist and be callable.
    assert callable(dummy_module.router.capture_fn)
    dummy_module.router.capture_fn(torch.tensor([[9, 10]]))
    assert len(capturer.calls) == 1
```
**EN:** Checks GPU Model Runner Binding Stage under a focused test scenario. The body exercises logic via `DummyFusedMoE`, `monkeypatch.setattr`, `types.SimpleNamespace` before asserting the expected outcome.
**CN:** 该测试用例验证 GPU Model Runner Binding Stage 在特定场景下的行为。 函数体会先通过 `DummyFusedMoE`, `monkeypatch.setattr`, `types.SimpleNamespace` 驱动目标逻辑，再断言预期结果。

### Test: test_routed_experts_capturer_single_dp_no_metadata (lines 194-202)
```python
def test_routed_experts_capturer_single_dp_no_metadata():
    """dp_metadata is None: capture writes the full topk_ids rows."""
    capturer = _capturer_with_buffer(dp_rank=0)
    topk = torch.tensor([[1, 2], [3, 4], [5, 6]], dtype=torch.int32)
    ctx = SimpleNamespace(dp_metadata=None)
    with patch(f"{_REC_MODULE}.get_forward_context", return_value=ctx):
        capturer.capture(layer_id=0, topk_ids=topk)
    assert torch.equal(capturer.device_buffer[:3, 0, :], topk)
    assert capturer.device_buffer[3, 0, 0].item() == -1
```
**EN:** dp_metadata is None: capture writes the full topk_ids rows. The body exercises logic via `_capturer_with_buffer`, `torch.tensor`, `SimpleNamespace` before asserting the expected outcome.
**CN:** 该测试用例验证 Routed Experts Capturer Single Dp No Metadata 在特定场景下的行为。 函数体会先通过 `_capturer_with_buffer`, `torch.tensor`, `SimpleNamespace` 驱动目标逻辑，再断言预期结果。

### Additional scenarios (summary)
```python
test_routed_experts_capturer_dp_naive_concatenated_all_ranks
test_routed_experts_capturer_dp_modular_local_tokens
test_routed_experts_capturer_dp_unexpected_batch_raises
```
**EN:** The file contains more helper or test blocks than can be shown compactly here. This summary lists additional scenarios that continue the same validation strategy.
**CN:** 该文件包含的辅助逻辑或测试场景较多，无法在此完整展开；这里列出其余场景名称，便于快速把握覆盖范围。

## Key Concepts / 关键概念
- **Mocking and patching / 模拟与补丁**
  - **EN:** Several scenarios replace dependencies or environment state so the tests can isolate one behavior at a time.
  - **CN:** 多个场景会替换依赖或环境状态，从而把验证范围限定在单一行为上。
- **GPU execution / GPU 执行**
  - **EN:** The tests exercise CUDA-like execution paths, device placement, or tensor kernels.
  - **CN:** 这些测试覆盖 CUDA 类执行路径、设备放置或张量内核行为。
- **Distributed coordination / 分布式协同**
  - **EN:** The file validates multi-process or multi-worker coordination primitives such as collectives and rank-aware setup.
  - **CN:** 该文件验证多进程或多工作节点之间的协同原语，例如集合通信和基于 rank 的初始化。
- **Configuration / 配置**
  - **EN:** The file checks how options, environment variables, and defaults are interpreted.
  - **CN:** 该文件检查选项、环境变量与默认值如何被解析和生效。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `types`, `unittest.mock`
- **Third-party / 第三方依赖**: `pytest`, `torch`
- **vLLM internal / vLLM 内部依赖**: `vllm.distributed.eplb.eplb_state`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.routed_experts_capturer`, `vllm.model_executor.layers.fused_moe.router.base_router`, `vllm.v1.worker`, `vllm.model_executor.layers.fused_moe.layer`
