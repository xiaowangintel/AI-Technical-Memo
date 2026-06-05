# __init__.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/moe/token_dispatcher/__init__.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements distributed communication and tensor movement with Mixture-of-Experts routing and expert execution for the SGLang SRT runtime. It exposes symbols such as `__init__` and connects them to backend-specific paths such as `FlashInfer` and `NPU`. / 该模块为 SGLang 的 SRT 运行时实现了分布式通信与张量搬运，并结合混合专家路由与专家执行。它提供了 `__init__` 等符号，并把这些符号连接到 `FlashInfer` 和 `NPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-79: Imports, constants, and runtime setup
```python
from sglang.srt.layers.moe.token_dispatcher.base import (
    BaseDispatcher,
    BaseDispatcherConfig,
    CombineInput,
    CombineInputChecker,
    CombineInputFormat,
    DispatchOutput,
    DispatchOutputChecker,
    DispatchOutputFormat,
)
from sglang.srt.layers.moe.token_dispatcher.deepep import (
    DeepEPConfig,
    DeepEPDispatcher,
    DeepEPLLCombineInput,
    DeepEPLLDispatchOutput,
    DeepEPNormalCombineInput,
    DeepEPNormalDispatchOutput,
)
from sglang.srt.layers.moe.token_dispatcher.flashinfer import (
    FlashinferDispatcher,
    FlashinferDispatchOutput,
)
from sglang.srt.layers.moe.token_dispatcher.fuseep import NpuFuseEPDispatcher
from sglang.srt.layers.moe.token_dispatcher.mooncake import (
    MooncakeCombineInput,
    MooncakeDispatchOutput,
    MooncakeEPDispatcher,
)
from sglang.srt.layers.moe.token_dispatcher.moriep import (
    MoriEPDispatcher,
    MoriEPLLCombineInput,
    MoriEPLLDispatchOutput,
    MoriEPNormalCombineInput,
    MoriEPNormalDispatchOutput,
)
from sglang.srt.layers.moe.token_dispatcher.nixl import (
    NixlEPCombineInput,
    NixlEPDispatcher,
    NixlEPDispatchOutput,
)
from sglang.srt.layers.moe.token_dispatcher.standard import (
    StandardCombineInput,
    StandardDispatcher,
    StandardDispatchOutput,
)

__all__ = [
    "BaseDispatcher",
    "BaseDispatcherConfig",
    "CombineInput",
    "CombineInputChecker",
    "CombineInputFormat",
    "DispatchOutput",
    "DispatchOutputFormat",
    "DispatchOutputChecker",
    "FlashinferDispatchOutput",
    "FlashinferDispatcher",
    "MooncakeCombineInput",
    "MooncakeDispatchOutput",
    "MooncakeEPDispatcher",
    "MoriEPNormalDispatchOutput",
    "MoriEPNormalCombineInput",
    "MoriEPLLDispatchOutput",
    "MoriEPLLCombineInput",
    "MoriEPDispatcher",
    "NixlEPCombineInput",
    "NixlEPDispatchOutput",
    "NixlEPDispatcher",
    "StandardDispatcher",
    "StandardDispatchOutput",
    "StandardCombineInput",
    "DeepEPConfig",
    "DeepEPDispatcher",
    "DeepEPNormalDispatchOutput",
    "DeepEPLLDispatchOutput",
    "DeepEPLLCombineInput",
    "DeepEPNormalCombineInput",
    "NpuFuseEPDispatcher",
]
```
**EN:** This section prepares the module namespace. It imports `sglang.srt.layers.moe.token_dispatcher.base.BaseDispatcher`, `sglang.srt.layers.moe.token_dispatcher.base.BaseDispatcherConfig`, `sglang.srt.layers.moe.token_dispatcher.base.CombineInput`, `sglang.srt.layers.moe.token_dispatcher.base.CombineInputChecker`, `sglang.srt.layers.moe.token_dispatcher.base.CombineInputFormat`, and `sglang.srt.layers.moe.token_dispatcher.base.DispatchOutput`, so later blocks can reuse runtime, tensor, or backend helpers. Shared names such as `__all__` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 它导入了 `sglang.srt.layers.moe.token_dispatcher.base.BaseDispatcher`、`sglang.srt.layers.moe.token_dispatcher.base.BaseDispatcherConfig`、`sglang.srt.layers.moe.token_dispatcher.base.CombineInput`、`sglang.srt.layers.moe.token_dispatcher.base.CombineInputChecker`、`sglang.srt.layers.moe.token_dispatcher.base.CombineInputFormat` 以及 `sglang.srt.layers.moe.token_dispatcher.base.DispatchOutput`，让后续代码可以复用运行时、张量或后端辅助逻辑。 像 `__all__` 这样的共享名称用于保存配置、缓存句柄或特性开关。

## Key Concepts / 关键概念
- **Mixture-of-Experts routing**: Explains how tokens are assigned to experts and how expert-side compute is organized. / **混合专家路由**：解释 token 如何被分配给专家，以及专家侧计算如何组织。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。

## Dependencies / 依赖关系
- **Internal SGLang modules**: `sglang.srt.layers.moe.token_dispatcher.base.BaseDispatcher`, `sglang.srt.layers.moe.token_dispatcher.base.BaseDispatcherConfig`, `sglang.srt.layers.moe.token_dispatcher.base.CombineInput`, `sglang.srt.layers.moe.token_dispatcher.base.CombineInputChecker`, `sglang.srt.layers.moe.token_dispatcher.base.CombineInputFormat`, `sglang.srt.layers.moe.token_dispatcher.base.DispatchOutput`, `sglang.srt.layers.moe.token_dispatcher.base.DispatchOutputChecker`, `sglang.srt.layers.moe.token_dispatcher.base.DispatchOutputFormat`, `sglang.srt.layers.moe.token_dispatcher.deepep.DeepEPConfig`, `sglang.srt.layers.moe.token_dispatcher.deepep.DeepEPDispatcher`, `sglang.srt.layers.moe.token_dispatcher.deepep.DeepEPLLCombineInput`, and `sglang.srt.layers.moe.token_dispatcher.deepep.DeepEPLLDispatchOutput` / **SGLang 内部模块**：`sglang.srt.layers.moe.token_dispatcher.base.BaseDispatcher`、`sglang.srt.layers.moe.token_dispatcher.base.BaseDispatcherConfig`、`sglang.srt.layers.moe.token_dispatcher.base.CombineInput`、`sglang.srt.layers.moe.token_dispatcher.base.CombineInputChecker`、`sglang.srt.layers.moe.token_dispatcher.base.CombineInputFormat`、`sglang.srt.layers.moe.token_dispatcher.base.DispatchOutput`、`sglang.srt.layers.moe.token_dispatcher.base.DispatchOutputChecker`、`sglang.srt.layers.moe.token_dispatcher.base.DispatchOutputFormat`、`sglang.srt.layers.moe.token_dispatcher.deepep.DeepEPConfig`、`sglang.srt.layers.moe.token_dispatcher.deepep.DeepEPDispatcher`、`sglang.srt.layers.moe.token_dispatcher.deepep.DeepEPLLCombineInput` 以及 `sglang.srt.layers.moe.token_dispatcher.deepep.DeepEPLLDispatchOutput`
