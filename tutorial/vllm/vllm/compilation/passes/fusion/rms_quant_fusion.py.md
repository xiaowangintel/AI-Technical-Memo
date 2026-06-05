# rms_quant_fusion.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/compilation/passes/fusion/rms_quant_fusion.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Fusion patterns for RMSNorm/add-RMSNorm plus FP8/NVFP4 quantization. / 将 RMSNorm/加法 RMSNorm 与 FP8/NVFP4 量化融合的模式。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-33)
```python
from typing import Any, NamedTuple

import torch
import torch._inductor.pattern_matcher as pm
from torch import fx
from torch._higher_order_ops.auto_functionalize import auto_functionalized
from torch._inductor.pattern_matcher import PatternMatcherPass
from torch._ops import OpOverload

import vllm.ir.ops
from vllm.config import VllmConfig, get_current_vllm_config
from vllm.logger import init_logger
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    GroupShape,
    QuantKey,
    ScaleDesc,
    kFp8Dynamic64Sym,
    kFp8Dynamic128Sym,
    kFp8DynamicTensorSym,
    kFp8DynamicTokenSym,
    kFp8StaticTensorSym,
    kNvfp4Dynamic,
    kStaticTensorScale,
)
from vllm.platforms import current_platform

from ..inductor_pass import enable_fake_mode
from ..vllm_inductor_pass import VllmInductorPass, VllmPatternMatcherPass
...
```
**EN:** This import block assembles the PyTorch/FX, Inductor, and vLLM modules that the rest of the file relies on for graph rewriting, backend integration, caching, or runtime control in vLLM's post-grad fusion pipeline.
**CN:** 这一组导入语句组织了 PyTorch/FX、Inductor 与 vLLM 模块，供文件后续代码在vLLM 的后梯度融合流程中完成图改写、后端集成、缓存或运行时控制。

### Constants / assignments (lines 35-128)
```python
logger = init_logger(__name__)
FP8_DTYPE = current_platform.fp8_dtype()
FP4_DTYPE = torch.uint8
_RMS_NORM_OP = torch.ops.vllm_ir.rms_norm.default
RMS_ADD_OP = torch.ops._C.fused_add_rms_norm.default
QUANT_OPS: dict[QuantKey, OpOverload] = {
    kFp8StaticTensorSym: torch.ops._C.static_scaled_fp8_quant.default,  # noqa: E501
    kFp8DynamicTensorSym: torch.ops._C.dynamic_scaled_fp8_quant.default,  # noqa: E501
    kFp8DynamicTokenSym: torch.ops._C.dynamic_per_token_scaled_fp8_quant.default,  # noqa: E501
}
FUSED_OPS: dict[FusedRMSQuantKey, OpOverload] = {
    FusedRMSQuantKey(
        kFp8StaticTensorSym, False
    ): torch.ops._C.rms_norm_static_fp8_quant.default,  # noqa: E501
    FusedRMSQuantKey(
        kFp8StaticTensorSym, True
    ): torch.ops._C.fused_add_rms_norm_static_fp8_quant.default,  # noqa: E501
    FusedRMSQuantKey(
        kFp8DynamicTokenSym, False
    ): torch.ops._C.rms_norm_dynamic_per_token_quant.default,  # noqa: E501
    FusedRMSQuantKey(
        kFp8DynamicTokenSym, True
    ): torch.ops._C.rms_norm_dynamic_per_token_quant.default,  # noqa: E501
    FusedRMSQuantKey(
...
```
**EN:** These assignments establish shared module state such as loggers, global flags, dtype/op handles, or counters that later logic reuses throughout vLLM's post-grad fusion pipeline.
**CN:** 这些赋值语句建立了共享模块状态，例如日志器、全局开关、dtype/算子句柄或计数器，供后续代码在vLLM 的后梯度融合流程中复用。

### Function `_rms_input_weight_dtype_match` (lines 45-53)
```python
def _rms_input_weight_dtype_match(match: pm.Match) -> bool:
    """Prevent fusion when rms_norm input and weight dtypes differ."""
    for node in match.nodes:
        if node.target == _RMS_NORM_OP:
            # rms_norm(x, weight, epsilon, variance_size)
            x, weight = node.args[0], node.args[1]
            if isinstance(x, fx.Node) and isinstance(weight, fx.Node):
                return x.meta["val"].dtype == weight.meta["val"].dtype
    return True
```
**EN:** Function `_rms_input_weight_dtype_match` provides a focused helper used by the surrounding module. It fits into vLLM's post-grad fusion pipeline. Prevent fusion when rms_norm input and weight dtypes differ.
**CN:** 函数 `_rms_input_weight_dtype_match` 提供供周边模块使用的聚焦辅助逻辑，并嵌入vLLM 的后梯度融合流程。 文档字符串说明：Prevent fusion when rms_norm input and weight dtypes differ.

### Function `empty_bf16` (lines 56-57)
```python
def empty_bf16(*args: Any, **kwargs: Any) -> torch.Tensor:
    return torch.empty(*args, **kwargs, dtype=torch.bfloat16, device="cuda")
```
**EN:** Function `empty_bf16` provides a focused helper used by the surrounding module. It fits into vLLM's post-grad fusion pipeline.
**CN:** 函数 `empty_bf16` 提供供周边模块使用的聚焦辅助逻辑，并嵌入vLLM 的后梯度融合流程。

### Function `empty_fp32` (lines 60-61)
```python
def empty_fp32(*args: Any, **kwargs: Any) -> torch.Tensor:
    return torch.empty(*args, **kwargs, dtype=torch.float32, device="cuda")
```
**EN:** Function `empty_fp32` provides a focused helper used by the surrounding module. It fits into vLLM's post-grad fusion pipeline.
**CN:** 函数 `empty_fp32` 提供供周边模块使用的聚焦辅助逻辑，并嵌入vLLM 的后梯度融合流程。

### Function `empty_i32` (lines 64-65)
```python
def empty_i32(*args: Any, **kwargs: Any) -> torch.Tensor:
    return torch.empty(*args, **kwargs, dtype=torch.int32, device="cuda")
```
**EN:** Function `empty_i32` provides a focused helper used by the surrounding module. It fits into vLLM's post-grad fusion pipeline.
**CN:** 函数 `empty_i32` 提供供周边模块使用的聚焦辅助逻辑，并嵌入vLLM 的后梯度融合流程。

### Function `empty_i64` (lines 68-69)
```python
def empty_i64(*args: Any, **kwargs: Any) -> torch.Tensor:
    return torch.empty(*args, **kwargs, dtype=torch.int64, device="cuda")
```
**EN:** Function `empty_i64` provides a focused helper used by the surrounding module. It fits into vLLM's post-grad fusion pipeline.
**CN:** 函数 `empty_i64` 提供供周边模块使用的聚焦辅助逻辑，并嵌入vLLM 的后梯度融合流程。

### Class `FusedRMSQuantKey` (lines 86-100)
```python
class FusedRMSQuantKey(NamedTuple):
    """
    Named tuple for identifying the type of RMSNorm + quant fusion.
    quant: type of quantization
    fused_add: does the op also perform the residual add
    """

    quant: QuantKey
    fused_add: bool

    def __str__(self) -> str:
        return (
            f"FusedQuantKey({self.quant}, with"
            f"{'' if self.fused_add else 'out'} residual)"
        )
```
**EN:** Class `FusedRMSQuantKey` holds a main piece of file-specific logic. It operates inside vLLM's post-grad fusion pipeline. Named tuple for identifying the type of RMSNorm + quant fusion. Key methods include __str__.
**CN:** 类 `FusedRMSQuantKey` 承载了该文件中的一块核心逻辑，并服务于vLLM 的后梯度融合流程。 文档字符串强调：Named tuple for identifying the type of RMSNorm + quant fusion. 关键方法包括 __str__。

### Class `RMSNormQuantPattern` (lines 131-153)
```python
class RMSNormQuantPattern:
    def __init__(
        self,
        epsilon: float,
        key: FusedRMSQuantKey,
        has_col_major_scales: bool = False,
        is_e8m0: bool = False,
        is_tma_aligned: bool = False,
    ) -> None:
        self.epsilon = epsilon
        self.quant_dtype = key.quant.dtype
        config = get_current_vllm_config()
        self.model_dtype = config.model_config.dtype if config.model_config else None

        assert key in FUSED_OPS, f"unsupported fused rmsnorm+quant op for {key}"
        self.FUSED_OP = FUSED_OPS[key]

        self.quant_matcher = MatcherQuantFP8(
            key.quant,
            has_col_major_scales=has_col_major_scales,
...
```
**EN:** Class `RMSNormQuantPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__.
**CN:** 类 `RMSNormQuantPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__。

### Class `RMSNormStaticQuantPattern` (lines 156-208)
```python
class RMSNormStaticQuantPattern(RMSNormQuantPattern):
    def __init__(
        self, epsilon: float, quant_dtype: torch.dtype, symmetric: bool = True
    ) -> None:
        fused_key = FusedRMSQuantKey(
            fused_add=False,
            quant=QuantKey(
                dtype=quant_dtype, scale=kStaticTensorScale, symmetric=symmetric
            ),
        )
        super().__init__(epsilon, fused_key)

    def register(self, pm_pass: PatternMatcherPass) -> None:
        # Cannot use methods, as the self argument affects tracing
        def pattern(
            input: torch.Tensor, weight: torch.Tensor, scale: torch.Tensor
        ) -> torch.Tensor:
            result_rms = vllm.ir.ops.rms_norm(input, weight, self.epsilon)
            return self.quant_matcher(result_rms, scale)[0]

...
```
**EN:** Class `RMSNormStaticQuantPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__, register.
**CN:** 类 `RMSNormStaticQuantPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__, register。

### Class `FusedAddRMSNormStaticQuantPattern` (lines 211-275)
```python
class FusedAddRMSNormStaticQuantPattern(RMSNormQuantPattern):
    def __init__(
        self, epsilon: float, quant_dtype: torch.dtype, symmetric: bool = True
    ) -> None:
        key = FusedRMSQuantKey(
            fused_add=True,
            quant=QuantKey(
                dtype=quant_dtype, scale=kStaticTensorScale, symmetric=symmetric
            ),
        )
        super().__init__(epsilon, key)

    def register(self, pm_pass: PatternMatcherPass) -> None:
        def pattern(
            input: torch.Tensor,
            weight: torch.Tensor,
            residual: torch.Tensor,
            scale: torch.Tensor,
        ) -> tuple[torch.Tensor, torch.Tensor]:
            result_rms, residual = vllm.ir.ops.fused_add_rms_norm(
...
```
**EN:** Class `FusedAddRMSNormStaticQuantPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__, register.
**CN:** 类 `FusedAddRMSNormStaticQuantPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__, register。

### Class `FusedAddRMSNormGroupQuantPattern` (lines 278-384)
```python
class FusedAddRMSNormGroupQuantPattern(RMSNormQuantPattern):
    def __init__(
        self,
        epsilon: float,
        quant_dtype: torch.dtype,
        group_shape: GroupShape,
        symmetric: bool = True,
        is_e8m0: bool = False,
        has_col_major_scales: bool = True,
        is_tma_aligned: bool = True,
    ) -> None:
        scale = ScaleDesc(torch.float32, False, group_shape)
        key = FusedRMSQuantKey(
            fused_add=True,
            quant=QuantKey(dtype=quant_dtype, scale=scale, symmetric=symmetric),
        )
        self.group_shape = group_shape
        self.is_e8m0 = is_e8m0
        self.has_col_major_scales = has_col_major_scales
        self.is_tma_aligned = is_tma_aligned
...
```
**EN:** Class `FusedAddRMSNormGroupQuantPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__, register.
**CN:** 类 `FusedAddRMSNormGroupQuantPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__, register。

### Class `RMSNormGroupQuantPattern` (lines 387-480)
```python
class RMSNormGroupQuantPattern(RMSNormQuantPattern):
    def __init__(
        self,
        epsilon: float,
        quant_dtype: torch.dtype,
        group_shape: GroupShape,
        symmetric: bool = True,
        is_e8m0: bool = False,
        has_col_major_scales: bool = True,
        is_tma_aligned: bool = True,
    ) -> None:
        scale = ScaleDesc(torch.float32, False, group_shape)
        key = FusedRMSQuantKey(
            fused_add=False,
            quant=QuantKey(dtype=quant_dtype, scale=scale, symmetric=symmetric),
        )
        self.group_shape = group_shape
        self.has_col_major_scales = has_col_major_scales
        self.is_tma_aligned = is_tma_aligned
        super().__init__(
...
```
**EN:** Class `RMSNormGroupQuantPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__, register.
**CN:** 类 `RMSNormGroupQuantPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__, register。

### Class `RMSNormDynamicQuantPattern` (lines 483-539)
```python
class RMSNormDynamicQuantPattern(RMSNormQuantPattern):
    def __init__(
        self,
        epsilon: float,
        quant_dtype: torch.dtype,
        group_shape: GroupShape = GroupShape.PER_TOKEN,
        symmetric: bool = True,
    ) -> None:
        scale = ScaleDesc(torch.float32, False, group_shape)
        key = FusedRMSQuantKey(
            fused_add=False,
            quant=QuantKey(dtype=quant_dtype, scale=scale, symmetric=symmetric),
        )
        super().__init__(epsilon, key)

    def register(self, pm_pass: PatternMatcherPass) -> None:
        def pattern(
            input: torch.Tensor, weight: torch.Tensor
        ) -> tuple[torch.Tensor, torch.Tensor]:
            result_rms = vllm.ir.ops.rms_norm(input, weight, self.epsilon)
...
```
**EN:** Class `RMSNormDynamicQuantPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__, register.
**CN:** 类 `RMSNormDynamicQuantPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__, register。

### Class `FusedAddRMSNormDynamicQuantPattern` (lines 542-604)
```python
class FusedAddRMSNormDynamicQuantPattern(RMSNormQuantPattern):
    def __init__(
        self,
        epsilon: float,
        quant_dtype: torch.dtype,
        group_shape: GroupShape = GroupShape.PER_TOKEN,
        symmetric: bool = True,
    ) -> None:
        scale = ScaleDesc(torch.float32, False, group_shape)
        key = FusedRMSQuantKey(
            fused_add=True,
            quant=QuantKey(dtype=quant_dtype, scale=scale, symmetric=symmetric),
        )
        super().__init__(epsilon, key)

    def register(self, pm_pass: PatternMatcherPass) -> None:
        def pattern(
            input: torch.Tensor, weight: torch.Tensor, residual: torch.Tensor
        ) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
            result_rms, residual = vllm.ir.ops.fused_add_rms_norm(
...
```
**EN:** Class `FusedAddRMSNormDynamicQuantPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__, register.
**CN:** 类 `FusedAddRMSNormDynamicQuantPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__, register。

### Class `RMSNormQuantFusionPass` (lines 607-683)
```python
class RMSNormQuantFusionPass(VllmPatternMatcherPass):
    """
    This pass fuses rms_norm & quant custom ops into a fused rms_norm_quant op.
    It also supports fused_add_rms_norm.
    """

    @enable_fake_mode
    def __init__(self, config: VllmConfig) -> None:
        super().__init__(config)

        self.patterns: PatternMatcherPass = PatternMatcherPass(
            pass_name="rmsnorm_quant_fusion_pass"
        )

        # Make sure fused add patterns are before simple rms norm,
        # as the latter is a subset of the former in torch ops
        for epsilon in [1e-5, 1e-6]:
            # Fuse fused_add_rms_norm + static fp8 quant
            FusedAddRMSNormStaticQuantPattern(epsilon, FP8_DTYPE).register(
                self.patterns
...
```
**EN:** Class `RMSNormQuantFusionPass` defines a transformation pass that walks an FX/IR graph and rewrites matched structures. It operates inside vLLM's post-grad fusion pipeline. This pass fuses rms_norm & quant custom ops into a fused rms_norm_quant op. Key methods include __init__, __call__, uuid.
**CN:** 类 `RMSNormQuantFusionPass` 定义了一个变换 pass，用于遍历 FX/IR 图并改写匹配到的结构，并服务于vLLM 的后梯度融合流程。 文档字符串强调：This pass fuses rms_norm & quant custom ops into a fused rms_norm_quant op. 关键方法包括 __init__, __call__, uuid。

## Key Concepts / 关键概念
- **Compilation pipeline / 编译流水线**
  - **EN:** The file participates in turning FX graphs into optimized executable code or runtime wrappers.
  - **CN:** 该文件参与将 FX 图转换为优化后的可执行代码或运行时封装。
- **Pattern-based graph fusion / 基于模式的图融合**
  - **EN:** The implementation matches recognizable FX subgraphs and replaces them with fewer, more specialized ops.
  - **CN:** 该实现会匹配可识别的 FX 子图，并将其替换为数量更少、专用性更强的算子。
- **Quantization-aware rewrites / 量化感知改写**
  - **EN:** The logic keeps scale/format constraints in mind while folding operations into fused quantized kernels.
  - **CN:** 这些逻辑在进行融合量化内核折叠时会同时考虑 scale/格式约束。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `import vllm.ir.ops`, `from vllm.config import VllmConfig, get_current_vllm_config`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.quantization.utils.quant_utils import GroupShape, QuantKey, ScaleDesc, kFp8Dynamic64Sym, kFp8Dynamic128Sym, kFp8DynamicTensorSym, kFp8DynamicTokenSym, kFp8StaticTensorSym, kNvfp4Dynamic, kStaticTensorScale`, `from vllm.platforms import current_platform`, `from ..inductor_pass import enable_fake_mode`, `from ..vllm_inductor_pass import VllmInductorPass, VllmPatternMatcherPass`, `from .matcher_utils import MatcherQuantFP8`
- **PyTorch / Torch 栈**: `import torch`, `import torch._inductor.pattern_matcher as pm`, `from torch import fx`, `from torch._higher_order_ops.auto_functionalize import auto_functionalized`, `from torch._inductor.pattern_matcher import PatternMatcherPass`, `from torch._ops import OpOverload`
- **Stdlib / 标准库**: `from typing import Any, NamedTuple`
