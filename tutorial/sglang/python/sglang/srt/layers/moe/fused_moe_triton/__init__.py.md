# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/fused_moe_triton/__init__.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `__init__` and connects them to backend-specific paths such as `Triton`. / 该模块为 SGLang 的 SRT 运行时实现了混合专家路由与专家执行。它提供了 `__init__` 等符号，并把这些符号连接到 `Triton` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Imports, constants, and runtime setup
```python
from sglang.srt.layers.moe.fused_moe_triton.layer import (
    FusedMoE,
    FusedMoeWeightScaleSupported,
)
from sglang.srt.layers.moe.moe_runner.triton_utils import (
    fused_experts,
    get_config,
    get_config_file_name,
    moe_align_block_size,
    override_config,
    try_get_optimal_moe_config,
)

__all__ = [
    "FusedMoE",
    "FusedMoeWeightScaleSupported",
    "override_config",
    "get_config",
    "fused_experts",
    "get_config_file_name",
    "moe_align_block_size",
    "try_get_optimal_moe_config",
]
```
**EN:** This section prepares the module namespace. It imports `sglang.srt.layers.moe.fused_moe_triton.layer.FusedMoE`, `sglang.srt.layers.moe.fused_moe_triton.layer.FusedMoeWeightScaleSupported`, `sglang.srt.layers.moe.moe_runner.triton_utils.fused_experts`, `sglang.srt.layers.moe.moe_runner.triton_utils.get_config`, `sglang.srt.layers.moe.moe_runner.triton_utils.get_config_file_name`, and `sglang.srt.layers.moe.moe_runner.triton_utils.moe_align_block_size`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `__all__` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `sglang.srt.layers.moe.fused_moe_triton.layer.FusedMoE`、`sglang.srt.layers.moe.fused_moe_triton.layer.FusedMoeWeightScaleSupported`、`sglang.srt.layers.moe.moe_runner.triton_utils.fused_experts`、`sglang.srt.layers.moe.moe_runner.triton_utils.get_config`、`sglang.srt.layers.moe.moe_runner.triton_utils.get_config_file_name` 以及 `sglang.srt.layers.moe.moe_runner.triton_utils.moe_align_block_size`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `__all__` 这样的共享名称用于保存配置、缓存句柄或特性开关。

## Key Concepts / 关键概念
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Internal SGLang modules**: `sglang.srt.layers.moe.fused_moe_triton.layer.FusedMoE`, `sglang.srt.layers.moe.fused_moe_triton.layer.FusedMoeWeightScaleSupported`, `sglang.srt.layers.moe.moe_runner.triton_utils.fused_experts`, `sglang.srt.layers.moe.moe_runner.triton_utils.get_config`, `sglang.srt.layers.moe.moe_runner.triton_utils.get_config_file_name`, `sglang.srt.layers.moe.moe_runner.triton_utils.moe_align_block_size`, `sglang.srt.layers.moe.moe_runner.triton_utils.override_config`, and `sglang.srt.layers.moe.moe_runner.triton_utils.try_get_optimal_moe_config` / **SGLang 内部模块**：`sglang.srt.layers.moe.fused_moe_triton.layer.FusedMoE`、`sglang.srt.layers.moe.fused_moe_triton.layer.FusedMoeWeightScaleSupported`、`sglang.srt.layers.moe.moe_runner.triton_utils.fused_experts`、`sglang.srt.layers.moe.moe_runner.triton_utils.get_config`、`sglang.srt.layers.moe.moe_runner.triton_utils.get_config_file_name`、`sglang.srt.layers.moe.moe_runner.triton_utils.moe_align_block_size`、`sglang.srt.layers.moe.moe_runner.triton_utils.override_config` 以及 `sglang.srt.layers.moe.moe_runner.triton_utils.try_get_optimal_moe_config`
