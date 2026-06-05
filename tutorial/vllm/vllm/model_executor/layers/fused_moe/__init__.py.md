# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/__init__.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Package export surface for fused Mixture-of-Experts routing, kernels, and runtime helpers / 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑的包导出入口

## Line-by-Line Analysis / 逐行分析
### Lines 4-41 — imports and setup
```python
from contextlib import contextmanager
from typing import Any, TypeAlias

from vllm.model_executor.layers.fused_moe.activation import (
    MoEActivation,
    activation_without_mul,
    apply_moe_activation,
)
from vllm.model_executor.layers.fused_moe.config import (
    FusedMoEConfig,
    FusedMoEParallelConfig,
    FusedMoEQuantConfig,
    RoutingMethodType,
)
from vllm.model_executor.layers.fused_moe.fused_moe_method_base import (
    FusedMoEMethodBase,
)
from vllm.model_executor.layers.fused_moe.layer import (
    FusedMoE,
    FusedMoeWeightScaleSupported,
    fused_moe_make_expert_params_mapping,
)
from vllm.model_executor.layers.fused_moe.modular_kernel import (
    FusedMoEActivationFormat,
    FusedMoEExpertsModular,
    FusedMoEPrepareAndFinalizeModular,
)
from vllm.model_executor.layers.fused_moe.router.fused_moe_router import (
    FusedMoERouter,
)
from vllm.model_executor.layers.fused_moe.router.gate_linear import GateLinear
from vllm.model_executor.layers.fused_moe.runner.shared_experts import (
    SharedExperts,
)
from vllm.model_executor.layers.fused_moe.unquantized_fused_moe_method import (
    UnquantizedFusedMoEMethod,
)
from vllm.triton_utils import HAS_TRITON
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 50-56 — function `override_config`
```python
@contextmanager
def override_config(config):
    global _config
    old_config = _config
    _config = config
    yield
    _config = old_config
```
**EN:** This function defines `override_config`. It manages expert dispatch, fused kernels, or MoE-specific tensor flow. The main inputs are `config`. It writes or updates `old_config`, `_config`.
**CN:** 该函数定义 `override_config`。 它管理专家分发、融合内核或 MoE 特有的张量流。 其主要输入参数包括 `config`。 它会写入或更新 `old_config`, `_config`。

### Lines 59-60 — function `get_config`
```python
def get_config() -> dict[str, Any] | None:
    return _config
```
**EN:** This function defines `get_config`. It derives and returns computed metadata or outputs needed by later stages.
**CN:** 该函数定义 `get_config`。 它推导并返回后续阶段所需的元数据或输出。

### Lines 63-85 — constant `__all__`
```python
__all__ = [
    "FusedMoE",
    "FusedMoERouter",
    "FusedMoEConfig",
    "FusedMoEQuantConfig",
    "FusedMoEParallelConfig",
    "FusedMoEMethodBase",
    "MoEActivation",
    "UnquantizedFusedMoEMethod",
    "FusedMoeWeightScaleSupported",
    "FusedMoEExpertsModular",
    "FusedMoEActivationFormat",
    "FusedMoEPrepareAndFinalizeModular",
    "GateLinear",
    "RoutingMethodType",
    "RoutedExperts",
    "SharedExperts",
    "activation_without_mul",
    "apply_moe_activation",
    "fused_moe_make_expert_params_mapping",
    "override_config",
    "get_config",
]
```
**EN:** This constant defines `__all__`. This block defines module-level constants or shared state used later in execution.
**CN:** 该模块级常量/状态定义 `__all__`。 该代码块定义了后续执行会复用的模块级常量或共享状态。

## Key Concepts / 关键概念
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `override_config`, `get_config` / [CN] 核心符号：`override_config`, `get_config`

## Dependencies / 依赖关系
- **External**: `contextlib`, `typing` / **外部依赖**: `contextlib`, `typing`
- **Internal**: `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.fused_moe_method_base`, `vllm.model_executor.layers.fused_moe.layer`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.model_executor.layers.fused_moe.router.fused_moe_router`, `vllm.model_executor.layers.fused_moe.router.gate_linear`, `vllm.model_executor.layers.fused_moe.runner.shared_experts`, `vllm.model_executor.layers.fused_moe.unquantized_fused_moe_method`, `vllm.triton_utils` / **内部依赖**: `vllm.model_executor.layers.fused_moe.activation`, `vllm.model_executor.layers.fused_moe.config`, `vllm.model_executor.layers.fused_moe.fused_moe_method_base`, `vllm.model_executor.layers.fused_moe.layer`, `vllm.model_executor.layers.fused_moe.modular_kernel`, `vllm.model_executor.layers.fused_moe.router.fused_moe_router`, `vllm.model_executor.layers.fused_moe.router.gate_linear`, `vllm.model_executor.layers.fused_moe.runner.shared_experts`, `vllm.model_executor.layers.fused_moe.unquantized_fused_moe_method`, `vllm.triton_utils`
- **Runtime traits**: Triton kernels / **运行时特征**: Triton kernels
