# routing_simulator_router.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/router/routing_simulator_router.py`
- **Repository**: vllm-project/vllm
- **Purpose**: MoE router scoring and top-k expert selection; fused Mixture-of-Experts routing, kernels, and runtime helpers / MoE 路由打分与 top-k 专家选择；融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 3-15 — imports and setup
```python
from abc import ABC, abstractmethod
from collections.abc import Callable
from typing import Any

import torch

import vllm.envs as envs
from vllm.distributed.eplb.eplb_state import EplbLayerState
from vllm.logger import init_logger
from vllm.model_executor.layers.fused_moe.config import RoutingMethodType
from vllm.model_executor.layers.fused_moe.router.base_router import BaseRouter

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。

### Lines 18-41 — class `RoutingStrategy`
```python
class RoutingStrategy(ABC):
    """Base class for token-to-expert routing strategies."""

    @abstractmethod
    def route_tokens(
        self,
        hidden_states: torch.Tensor,
        router_logits: torch.Tensor,
        top_k: int,
        indices_type: torch.dtype | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        """
        Route tokens to experts.

# ... omitted for brevity ...
        """
        pass
```
**EN:** This class defines `RoutingStrategy`. It inherits from `ABC`. Base class for token-to-expert routing strategies. Important methods include `route_tokens`.
**CN:** 该类定义了 `RoutingStrategy`。 它继承自 `ABC`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `route_tokens`。

### Lines 21-41 — method `RoutingStrategy.route_tokens`
```python
    @abstractmethod
    def route_tokens(
        self,
        hidden_states: torch.Tensor,
        router_logits: torch.Tensor,
        top_k: int,
        indices_type: torch.dtype | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        """
        Route tokens to experts.

        Args:
            hidden_states: Input hidden states [num_tokens, hidden_size]
            router_logits: Router logits [num_tokens, num_experts]
            top_k: Number of experts to select per token
            indices_type: Data type for expert indices

        Returns:
            tuple of (topk_weights, topk_ids)
        """
        pass
```
**EN:** This method defines `route_tokens`. Route tokens to experts. The main inputs are `hidden_states`, `router_logits`, `top_k`, `indices_type`.
**CN:** 该方法定义 `route_tokens`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `hidden_states`, `router_logits`, `top_k`, `indices_type`。

### Lines 44-222 — class `DistributionBasedRouting`
```python
class DistributionBasedRouting(RoutingStrategy):
    """
    Distribution-based random routing strategy with configurable distributions.

    This routing strategy randomly selects experts for each token based on
    different probability distributions. Currently supports uniform and normal
    distributions for testing different routing patterns.
    """

    def __init__(self, distribution: str = "uniform", **distribution_params: Any):
        """
        Initialize distribution-based routing.

        Args:
# ... omitted for brevity ...
            "parameters": self.distribution_params.copy(),
        }
```
**EN:** This class defines `DistributionBasedRouting`. It inherits from `RoutingStrategy`. Distribution-based random routing strategy with configurable distributions. Important methods include `__init__`, `route_tokens`, `get_distribution_info`. Key calls include `distribution.lower`, `self._validate_distribution_params`, `self._sample_expert_ids`, `self._generate_weights`, `ValueError`, `self.distribution_params.setdefault`. It writes or updates `distribution`, `distribution_params`, `valid_distributions`, `num_tokens`, `num_experts`, `topk_ids`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `DistributionBasedRouting`。 它继承自 `RoutingStrategy`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `route_tokens`, `get_distribution_info`。 关键调用包括 `distribution.lower`, `self._validate_distribution_params`, `self._sample_expert_ids`, `self._generate_weights`, `ValueError`, `self.distribution_params.setdefault`。 它会写入或更新 `distribution`, `distribution_params`, `valid_distributions`, `num_tokens`, `num_experts`, `topk_ids`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 53-70 — method `DistributionBasedRouting.__init__`
```python
    def __init__(self, distribution: str = "uniform", **distribution_params: Any):
        """
        Initialize distribution-based routing.

        Args:
            distribution: Type of distribution to use for sampling
                - "uniform": Uniform distribution (default)
                - "normal": Normal/Gaussian distribution
            **distribution_params: Parameters specific to the
                chosen distribution
                For "uniform": No additional parameters needed
                For "normal": mean (default: 0.0), std (default: 1.0)
        """
        self.distribution = distribution.lower()
        self.distribution_params = distribution_params

        # Validate distribution and parameters
        self._validate_distribution_params()
```
**EN:** This method defines `__init__`. Initialize distribution-based routing. The main inputs are `distribution`, `**distribution_params`. Key calls include `distribution.lower`, `self._validate_distribution_params`. It writes or updates `distribution`, `distribution_params`.
**CN:** 该方法定义 `__init__`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `distribution`, `**distribution_params`。 关键调用包括 `distribution.lower`, `self._validate_distribution_params`。 它会写入或更新 `distribution`, `distribution_params`。

### Lines 87-122 — method `DistributionBasedRouting.route_tokens`
```python
    def route_tokens(
        self,
        hidden_states: torch.Tensor,
        router_logits: torch.Tensor,
        top_k: int,
        indices_type: torch.dtype | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        """
        Randomly select experts for each token using the specified distribution.

        Args:
            hidden_states: Input hidden states [num_tokens, hidden_size]
            router_logits: Router logits [num_tokens, num_experts]
            top_k: Number of experts to select per token
            indices_type: Data type for expert indices

        Returns:
            tuple of (topk_weights, topk_ids) where:
            - topk_weights: Weights based on distribution sampling
            - topk_ids: Expert indices sampled from the distribution
        """
        num_tokens = hidden_states.shape[0]
        num_experts = router_logits.shape[-1]

        if indices_type is None:
            indices_type = torch.long

        # Generate expert IDs based on the specified distribution
        topk_ids = self._sample_expert_ids(
            num_tokens, num_experts, top_k, hidden_states.device, indices_type
        )

        # Generate weights based on the distribution
        topk_weights = self._generate_weights(num_tokens, top_k, hidden_states.device)

        return topk_weights, topk_ids
```
**EN:** This method defines `route_tokens`. Randomly select experts for each token using the specified distribution. The main inputs are `hidden_states`, `router_logits`, `top_k`, `indices_type`. Key calls include `self._sample_expert_ids`, `self._generate_weights`. It writes or updates `num_tokens`, `num_experts`, `topk_ids`, `topk_weights`, `indices_type`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `route_tokens`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `hidden_states`, `router_logits`, `top_k`, `indices_type`。 关键调用包括 `self._sample_expert_ids`, `self._generate_weights`。 它会写入或更新 `num_tokens`, `num_experts`, `topk_ids`, `topk_weights`, `indices_type`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 217-222 — method `DistributionBasedRouting.get_distribution_info`
```python
    def get_distribution_info(self) -> dict:
        """Get information about the current distribution configuration."""
        return {
            "distribution": self.distribution,
            "parameters": self.distribution_params.copy(),
        }
```
**EN:** This method defines `get_distribution_info`. Get information about the current distribution configuration. Key calls include `self.distribution_params.copy`.
**CN:** 该方法定义 `get_distribution_info`。 该函数/方法的文档字符串直接说明了它的职责。 关键调用包括 `self.distribution_params.copy`。

### Lines 225-306 — class `RoutingSimulator`
```python
class RoutingSimulator:
    """
    Token-to-Expert Routing Simulator.

    This class provides a framework for testing and comparing different
    routing strategies for MoE models. It can simulate routing behavior
    and collect statistics for analysis.
    """

    # Class-level registry of routing strategies
    _routing_strategies: dict[str, RoutingStrategy] = {
        # Basic routing strategies
        "uniform_random": DistributionBasedRouting(
            distribution="uniform", mean=0.0, std=1.0
# ... omitted for brevity ...
            indices_type=indices_type,
        )
```
**EN:** This class defines `RoutingSimulator`. Token-to-Expert Routing Simulator. Important methods include `register_strategy`, `get_available_strategies`, `simulate_routing`. Key calls include `DistributionBasedRouting`, `list`, `logger.warning_once`, `strategy.route_tokens`, `cls._routing_strategies.keys`, `ValueError`. It writes or updates `_routing_strategies`, `strategy`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `RoutingSimulator`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `register_strategy`, `get_available_strategies`, `simulate_routing`。 关键调用包括 `DistributionBasedRouting`, `list`, `logger.warning_once`, `strategy.route_tokens`, `cls._routing_strategies.keys`, `ValueError`。 它会写入或更新 `_routing_strategies`, `strategy`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 245-254 — method `RoutingSimulator.register_strategy`
```python
    @classmethod
    def register_strategy(cls, name: str, strategy: RoutingStrategy):
        """
        Register a custom routing strategy.

        Args:
            name: Name of the strategy
            strategy: RoutingStrategy instance
        """
        cls._routing_strategies[name] = strategy
```
**EN:** This method defines `register_strategy`. Register a custom routing strategy. The main inputs are `name`, `strategy`.
**CN:** 该方法定义 `register_strategy`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `name`, `strategy`。

### Lines 256-264 — method `RoutingSimulator.get_available_strategies`
```python
    @classmethod
    def get_available_strategies(cls) -> list[str]:
        """
        Get list of available routing strategy names.

        Returns:
            List of available strategy names
        """
        return list(cls._routing_strategies.keys())
```
**EN:** This method defines `get_available_strategies`. Get list of available routing strategy names. Key calls include `list`, `cls._routing_strategies.keys`.
**CN:** 该方法定义 `get_available_strategies`。 该函数/方法的文档字符串直接说明了它的职责。 关键调用包括 `list`, `cls._routing_strategies.keys`。

### Lines 266-306 — method `RoutingSimulator.simulate_routing`
```python
    @staticmethod
    def simulate_routing(
        hidden_states: torch.Tensor,
        router_logits: torch.Tensor,
        strategy_name: str,
        top_k: int,
        indices_type: torch.dtype | None = None,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        """
        Simulate token-to-expert routing using the specified strategy.

        Args:
            hidden_states: Input hidden states [num_tokens, hidden_size]
            router_logits: Router logits [num_tokens, num_experts]
            strategy_name: Name of the routing strategy to use
            top_k: Number of experts to select per token
            indices_type: Data type for expert indices

        Returns:
            tuple of (topk_weights, topk_ids)
        """
        if strategy_name not in RoutingSimulator._routing_strategies:
            raise ValueError(
                f"Unknown routing strategy: {strategy_name}. "
                f"Available strategies: "
                f"{list(RoutingSimulator._routing_strategies.keys())}"
            )
        logger.warning_once(
            "Simulating MoE routing using a %s strategy. "
            "This should only be used for performance testing. "
            "Model outputs will not be valid.",
            strategy_name,
        )

        strategy = RoutingSimulator._routing_strategies[strategy_name]
        return strategy.route_tokens(
            hidden_states=hidden_states,
            router_logits=router_logits,
            top_k=top_k,
            indices_type=indices_type,
        )
```
**EN:** This method defines `simulate_routing`. Simulate token-to-expert routing using the specified strategy. The main inputs are `hidden_states`, `router_logits`, `strategy_name`, `top_k`, `indices_type`. Key calls include `logger.warning_once`, `strategy.route_tokens`, `ValueError`, `list`, `RoutingSimulator._routing_strategies.keys`. It writes or updates `strategy`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `simulate_routing`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `hidden_states`, `router_logits`, `strategy_name`, `top_k`, `indices_type`。 关键调用包括 `logger.warning_once`, `strategy.route_tokens`, `ValueError`, `list`, `RoutingSimulator._routing_strategies.keys`。 它会写入或更新 `strategy`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 309-347 — class `RoutingSimulatorRouter`
```python
class RoutingSimulatorRouter(BaseRouter):
    """Router that uses routing simulation strategies for testing/debugging."""

    def __init__(
        self,
        top_k: int,
        global_num_experts: int,
        eplb_state: EplbLayerState | None = None,
        indices_type_getter: Callable[[], torch.dtype | None] | None = None,
    ):
        super().__init__(
            top_k=top_k,
            global_num_experts=global_num_experts,
            eplb_state=eplb_state,
# ... omitted for brevity ...
        )
        return topk_weights, topk_ids
```
**EN:** This class defines `RoutingSimulatorRouter`. It inherits from `BaseRouter`. Router that uses routing simulation strategies for testing/debugging. Important methods include `__init__`, `routing_method_type`. Key calls include `super.__init__`, `RoutingSimulator.simulate_routing`, `super`. It writes or updates `routing_strategy`, `topk_weights`, `topk_ids`.
**CN:** 该类定义了 `RoutingSimulatorRouter`。 它继承自 `BaseRouter`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `routing_method_type`。 关键调用包括 `super.__init__`, `RoutingSimulator.simulate_routing`, `super`。 它会写入或更新 `routing_strategy`, `topk_weights`, `topk_ids`。

### Lines 312-324 — method `RoutingSimulatorRouter.__init__`
```python
    def __init__(
        self,
        top_k: int,
        global_num_experts: int,
        eplb_state: EplbLayerState | None = None,
        indices_type_getter: Callable[[], torch.dtype | None] | None = None,
    ):
        super().__init__(
            top_k=top_k,
            global_num_experts=global_num_experts,
            eplb_state=eplb_state,
            indices_type_getter=indices_type_getter,
        )
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `top_k`, `global_num_experts`, `eplb_state`, `indices_type_getter`. Key calls include `super.__init__`, `super`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `top_k`, `global_num_experts`, `eplb_state`, `indices_type_getter`。 关键调用包括 `super.__init__`, `super`。

### Lines 326-328 — method `RoutingSimulatorRouter.routing_method_type`
```python
    @property
    def routing_method_type(self) -> RoutingMethodType:
        return RoutingMethodType.Simulated
```
**EN:** This method defines `routing_method_type`. It computes routing scores or expert-selection behavior for MoE execution.
**CN:** 该方法定义 `routing_method_type`。 它为 MoE 执行计算路由得分或专家选择行为。

## Key Concepts / 关键概念
- [EN] Moe router scoring and top-k expert selection / [CN] MoE 路由打分与 top-k 专家选择
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Distributed or tensor-parallel coordination / [CN] 分布式或张量并行协同
- [EN] Core symbols: `RoutingStrategy`, `DistributionBasedRouting`, `RoutingSimulator`, `RoutingSimulatorRouter` / [CN] 核心符号：`RoutingStrategy`, `DistributionBasedRouting`, `RoutingSimulator`, `RoutingSimulatorRouter`

## Dependencies / 依赖关系
- **External**: `abc`, `collections.abc`, `typing`, `torch` / **外部依赖**: `abc`, `collections.abc`, `typing`, `torch`
- **Internal**: `vllm.envs`, `vllm.distributed.eplb.eplb_state`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.router.base_router` / **内部依赖**: `vllm.envs`, `vllm.distributed.eplb.eplb_state`, `vllm.logger`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.router.base_router`
- **Runtime traits**: distributed collectives / **运行时特征**: distributed collectives
