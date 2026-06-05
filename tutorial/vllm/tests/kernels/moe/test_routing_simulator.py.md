# test_routing_simulator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_routing_simulator.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_routing_simulator, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_routing_simulator 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 4-10)
```python
"""
Test script for the token-to-expert routing simulator.

This script demonstrates how to use the routing simulator to test
different routing strategies and analyze their performance, including
integration tests with FusedMoE layer.
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 12-25)
```python
import tempfile

import pytest
import torch

from vllm.config import VllmConfig, set_current_vllm_config
from vllm.distributed import (
    init_distributed_environment,
    initialize_model_parallel,
)
from vllm.model_executor.layers.fused_moe.router.routing_simulator_router import (
    DistributionBasedRouting,
    RoutingSimulator,
)
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as tempfile, pytest, torch; and vLLM components like vllm.config, vllm.distributed, vllm.model_executor.layers.fused_moe.router.routing_simulator_router.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 tempfile、pytest、torch；vLLM 内部组件，例如 vllm.config、vllm.distributed、vllm.model_executor.layers.fused_moe.router.routing_simulator_router。

### Function `device` (lines 28-31)
```python
@pytest.fixture
def device():
    """Fixture to provide the appropriate device for testing."""
    return torch.device("cuda" if torch.cuda.is_available() else "cpu")
```
**EN:** This fixture prepares reusable state for device. It keeps the surrounding module logic factored into a reusable unit.
**CN:** 该 fixture 为 device 准备可复用的测试状态。 它把周边模块中的共用逻辑封装成可复用单元。

### Function `test_basic_functionality` (lines 34-75)
```python
@pytest.mark.parametrize("num_tokens", [1, 16, 256])
@pytest.mark.parametrize("hidden_size", [64, 1024])
@pytest.mark.parametrize("num_experts", [16, 128])
@pytest.mark.parametrize("top_k", [1, 4])
def test_basic_functionality(
    num_tokens: int,
    hidden_size: int,
    num_experts: int,
    top_k: int,
    device,
):
    """Test basic functionality of the routing simulator."""
    # Test each routing strategy
    strategies = RoutingSimulator.get_available_strategies()

    hidden_states = torch.randn(num_tokens, hidden_size, device=device)
    router_logits = torch.randn(num_tokens, num_experts, device=device)

    for strategy in strategies:
        # Simulate routing
        topk_weights, topk_ids = RoutingSimulator.simulate_routing(
            hidden_states=hidden_states,
            router_logits=router_logits,
            strategy_name=strategy,
            top_k=top_k,
        )

        # Check output shapes
        assert topk_weights.shape == (
            num_tokens,
            top_k,
        ), f"Wrong weights shape for {strategy}"
        assert topk_ids.shape == (
            num_tokens,
            top_k,
        ), f"Wrong ids shape for {strategy}"

        # Check that expert IDs are valid
        assert topk_ids.min() >= 0, f"Invalid expert ID (negative) for {strategy}"
        assert topk_ids.max() < num_experts, (
            f"Invalid expert ID (too large) for {strategy}"
        )
```
**EN:** This pytest case verifies basic functionality. It is parameterized across 4 input dimensions so one definition covers many scenarios. it consumes fixtures or inputs such as num_tokens, hidden_size, num_experts, top_k. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 basic functionality 的行为。 它通过 4 组参数化输入覆盖多种场景；它会使用诸如 num_tokens、hidden_size、num_experts、top_k 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Function `test_routing_strategy_integration` (lines 78-156)
```python
def test_routing_strategy_integration(monkeypatch, device):
    """Test that the routing strategy environment variable works with
    FusedMoE."""
    pytest.importorskip("vllm.model_executor.layers.fused_moe.layer")

    import vllm.envs as envs
    from vllm.model_executor.layers.fused_moe.layer import FusedMoE

    # Test parameters
    num_tokens = 32
    hidden_size = 16
    num_experts = 4
    top_k = 2

    # Create test data
    hidden_states = torch.randn(num_tokens, hidden_size, device=device)
    router_logits = torch.randn(num_tokens, num_experts, device=device)

    # Test different routing strategies
    strategies = RoutingSimulator.get_available_strategies()

    vllm_config = VllmConfig()
    with set_current_vllm_config(vllm_config):
        temp_file = tempfile.mkstemp()[1]
        init_distributed_environment(
            world_size=1,
            rank=0,
            local_rank=0,
            distributed_init_method=f"file://{temp_file}",
        )
        initialize_model_parallel(
            tensor_model_parallel_size=1,
            pipeline_model_parallel_size=1,
        )

        for strategy in strategies:
            fused_moe = FusedMoE(
                num_experts=num_experts,
                top_k=top_k,
                hidden_size=hidden_size,
                intermediate_size=0,
                use_grouped_topk=False,
                renormalize=True,
                prefix=strategy,
            )

            # Set environment variable
            env_name = "VLLM_MOE_ROUTING_SIMULATION_STRATEGY"
            monkeypatch.setenv(env_name, strategy)

            # Temporarily override the envs lookup so the router factory
            # reads the monkeypatched value instead of the module-load-time
            # default. Use monkeypatch.setitem so the original lambda is
            # restored automatically at teardown.
            monkeypatch.setitem(
                envs.environment_variables,
                env_name,
                lambda s=strategy: s,
            )

            # Test the select_experts method
            topk_weights, topk_ids = fused_moe.router.select_experts(
                hidden_states=hidden_states,
                router_logits=router_logits,
            )

            # Verify output shapes
            assert topk_weights.shape == (num_tokens, top_k), (
                f"Wrong weights shape for {strategy}"
            )
            assert topk_ids.shape == (num_tokens, top_k), (
                f"Wrong ids shape for {strategy}"
            )

            # Verify expert IDs are valid
            assert topk_ids.min() >= 0, f"Invalid expert ID (negative) for {strategy}"
            assert topk_ids.max() < num_experts, (
                f"Invalid expert ID (too large) for {strategy}"
            )
```
**EN:** This pytest case verifies routing strategy integration. it consumes fixtures or inputs such as monkeypatch, device. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 routing strategy integration 的行为。 它会使用诸如 monkeypatch、device 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Function `test_distribution_based_routing_with_custom_strategy` (lines 159-191)
```python
def test_distribution_based_routing_with_custom_strategy():
    """Test registering and using DistributionBasedRouting with custom
    parameters."""
    device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

    # Register custom distribution-based strategy
    custom_strategy = DistributionBasedRouting(distribution="normal", mean=2.0, std=0.5)
    RoutingSimulator.register_strategy("custom_normal", custom_strategy)

    # Test data
    num_tokens = 60
    hidden_size = 48
    num_experts = 6
    top_k = 3

    hidden_states = torch.randn(num_tokens, hidden_size, device=device)
    router_logits = torch.randn(num_tokens, num_experts, device=device)

    # Use the custom strategy
    topk_weights, topk_ids = RoutingSimulator.simulate_routing(
        hidden_states=hidden_states,
        router_logits=router_logits,
        strategy_name="custom_normal",
        top_k=top_k,
    )

    # Check output shapes
    assert topk_weights.shape == (num_tokens, top_k)
    assert topk_ids.shape == (num_tokens, top_k)

    # Check that expert IDs are valid
    assert topk_ids.min() >= 0
    assert topk_ids.max() < num_experts
```
**EN:** This pytest case verifies distribution based routing with custom strategy. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 distribution based routing with custom strategy 的行为。 结尾处的断言会固定预期行为或计算图形态。

### Function `test_instance_compatibility` (lines 194-210)
```python
def test_instance_compatibility():
    """Test that static methods work correctly."""
    device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

    # Test static method directly
    hidden_states = torch.randn(10, 8, device=device)
    router_logits = torch.randn(10, 4, device=device)

    topk_weights, topk_ids = RoutingSimulator.simulate_routing(
        hidden_states=hidden_states,
        router_logits=router_logits,
        strategy_name="uniform_random",
        top_k=2,
    )

    assert topk_weights.shape == (10, 2)
    assert topk_ids.shape == (10, 2)
```
**EN:** This pytest case verifies instance compatibility. assertions at the end lock in the intended behavior or graph shape.
**CN:** 该 pytest 用例验证 instance compatibility 的行为。 结尾处的断言会固定预期行为或计算图形态。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。
- **Reusable fixtures / 可复用 fixture:** Fixtures package common setup so each test can focus on the scenario under study. / fixture 封装公共准备逻辑，让每个测试更专注于待验证场景。

## Dependencies / 依赖关系
- `tempfile`
- `pytest`
- `torch`
- `vllm.config -> VllmConfig, set_current_vllm_config`
- `vllm.distributed -> init_distributed_environment, initialize_model_parallel`
- `vllm.model_executor.layers.fused_moe.router.routing_simulator_router -> DistributionBasedRouting, RoutingSimulator`
