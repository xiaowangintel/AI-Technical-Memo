# act_quant_fusion.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/compilation/passes/fusion/act_quant_fusion.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Fusion patterns that combine activation epilogues with quantization kernels. / 将激活后处理与量化内核融合的模式。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-24)
```python
import itertools
from typing import Any

import torch
from torch._higher_order_ops.auto_functionalize import auto_functionalized
from torch._ops import OpOverload

from vllm.config import VllmConfig
from vllm.logger import init_logger
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
    kFp8Dynamic64Sym,
    kFp8Dynamic128Sym,
    kFp8StaticTensorSym,
    kNvfp4Dynamic,
)
from vllm.platforms import current_platform

from ..vllm_inductor_pass import VllmFusionPatternMatcherPass, VllmPatternReplacement
from .matcher_utils import MatcherQuantFP8, MatcherSiluAndMul
from .rms_quant_fusion import QUANT_OPS, empty_bf16, empty_fp32, empty_i32
```
**EN:** This import block assembles the PyTorch/FX, Inductor, and vLLM modules that the rest of the file relies on for graph rewriting, backend integration, caching, or runtime control in vLLM's post-grad fusion pipeline.
**CN:** 这一组导入语句组织了 PyTorch/FX、Inductor 与 vLLM 模块，供文件后续代码在vLLM 的后梯度融合流程中完成图改写、后端集成、缓存或运行时控制。

### Constants / assignments (lines 26-38)
```python
logger = init_logger(__name__)
FP8_DTYPE = current_platform.fp8_dtype()
FP4_DTYPE = torch.uint8
SILU_MUL_OP = torch.ops._C.silu_and_mul.default
FUSED_OPS: dict[QuantKey, OpOverload] = {
    kFp8StaticTensorSym: torch.ops._C.silu_and_mul_quant.default,  # noqa: E501
}
silu_and_mul_nvfp4_quant_supported = current_platform.is_cuda() and hasattr(
    torch.ops._C, "silu_and_mul_nvfp4_quant"
)
```
**EN:** These assignments establish shared module state such as loggers, global flags, dtype/op handles, or counters that later logic reuses throughout vLLM's post-grad fusion pipeline.
**CN:** 这些赋值语句建立了共享模块状态，例如日志器、全局开关、dtype/算子句柄或计数器，供后续代码在vLLM 的后梯度融合流程中复用。

### Class `ActivationQuantPattern` (lines 47-74)
```python
class ActivationQuantPattern(VllmPatternReplacement):
    """
    Base class for Activation+Quant fusions.
    Should not be used directly.
    """

    def __init__(
        self,
        quant_key: QuantKey,
    ) -> None:
        self.quant_key = quant_key
        self.quant_dtype = quant_key.dtype

        assert self.quant_key in QUANT_OPS, (
            f"unsupported quantization scheme {self.quant_key}"
        )
        self.QUANT_OP = QUANT_OPS[self.quant_key]

        assert self.quant_key in FUSED_OPS, (
            f"unsupported fusion scheme {self.quant_key}"
...
```
**EN:** Class `ActivationQuantPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Base class for Activation+Quant fusions. Key methods include __init__, empty_quant.
**CN:** 类 `ActivationQuantPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 文档字符串强调：Base class for Activation+Quant fusions. 关键方法包括 __init__, empty_quant。

### Class `SiluMulFp8StaticQuantPattern` (lines 77-121)
```python
class SiluMulFp8StaticQuantPattern(ActivationQuantPattern):
    """
    Fusion for SiluMul+Fp8StaticQuant Pattern
    """

    def __init__(self) -> None:
        super().__init__(kFp8StaticTensorSym)
        self.quant_matcher = MatcherQuantFP8(kFp8StaticTensorSym)

    def get_inputs(self) -> list[torch.Tensor]:
        scale = self.quant_matcher.inputs()[1]
        return [
            *self.silu_and_mul_matcher.inputs(),  # input
            scale,
        ]

    @property
    def pattern(self):
        def _pattern(
            input: torch.Tensor,
...
```
**EN:** Class `SiluMulFp8StaticQuantPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Fusion for SiluMul+Fp8StaticQuant Pattern Key methods include __init__, get_inputs, pattern, replacement.
**CN:** 类 `SiluMulFp8StaticQuantPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 文档字符串强调：Fusion for SiluMul+Fp8StaticQuant Pattern 关键方法包括 __init__, get_inputs, pattern, replacement。

### Class `SiluMulNvfp4QuantPattern` (lines 124-177)
```python
class SiluMulNvfp4QuantPattern(ActivationQuantPattern):
    """
    Fusion for SiluMul+Nvfp4Quant Pattern
    """

    def __init__(self) -> None:
        super().__init__(kNvfp4Dynamic)

    def get_inputs(self) -> list[torch.Tensor]:
        result = self.empty_quant(5, 32)
        output_scale = empty_i32(128, 4)
        input_ = empty_bf16(5, 64)
        scale = empty_fp32(1, 1)
        return [result, output_scale, input_, scale]

    @property
    def pattern(self):
        def _pattern(
            result: torch.Tensor,
            output_scale: torch.Tensor,
...
```
**EN:** Class `SiluMulNvfp4QuantPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Fusion for SiluMul+Nvfp4Quant Pattern Key methods include __init__, get_inputs, pattern, replacement.
**CN:** 类 `SiluMulNvfp4QuantPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 文档字符串强调：Fusion for SiluMul+Nvfp4Quant Pattern 关键方法包括 __init__, get_inputs, pattern, replacement。

### Class `SiluMulBlockQuantPattern` (lines 180-276)
```python
class SiluMulBlockQuantPattern(ActivationQuantPattern):
    """
    Fusion for SiluMul+BlockQuant (FP8 dynamic per-group) Pattern.
    Supports group_size 128 and 64 via QuantKey.
    Parameterized on is_scale_transposed for different scale layouts.
    """

    def __init__(
        self,
        quant_key: QuantKey,
        is_scale_transposed: bool = False,
        is_e8m0: bool = False,
        is_tma_aligned: bool = False,
        match_aiter: bool = False,
    ) -> None:
        super().__init__(quant_key)
        self.quant_matcher = MatcherQuantFP8(
            quant_key,
            has_col_major_scales=is_scale_transposed,
            is_e8m0=is_e8m0,
...
```
**EN:** Class `SiluMulBlockQuantPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Fusion for SiluMul+BlockQuant (FP8 dynamic per-group) Pattern. Key methods include __init__, get_inputs, pattern, replacement.
**CN:** 类 `SiluMulBlockQuantPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 文档字符串强调：Fusion for SiluMul+BlockQuant (FP8 dynamic per-group) Pattern. 关键方法包括 __init__, get_inputs, pattern, replacement。

### Class `ActivationQuantFusionPass` (lines 279-318)
```python
class ActivationQuantFusionPass(VllmFusionPatternMatcherPass):
    """
    This pass fuses a pre-defined set of custom ops into fused ops.
    It uses the torch pattern matcher to find the patterns and replace them.

    Because patterns can only be registered once, the pass is a singleton.
    This will be addressed in a future version of PyTorch:
    https://github.com/pytorch/pytorch/pull/139321#issuecomment-2452354980
    """

    def __init__(self, config: VllmConfig) -> None:
        super().__init__(config, "activation_quant_fusion_pass")

        self.register(SiluMulFp8StaticQuantPattern())

        if silu_and_mul_nvfp4_quant_supported:
            self.register(SiluMulNvfp4QuantPattern())

        if current_platform.is_cuda():
            for (
...
```
**EN:** Class `ActivationQuantFusionPass` defines a transformation pass that walks an FX/IR graph and rewrites matched structures. It operates inside vLLM's post-grad fusion pipeline. This pass fuses a pre-defined set of custom ops into fused ops. Key methods include __init__.
**CN:** 类 `ActivationQuantFusionPass` 定义了一个变换 pass，用于遍历 FX/IR 图并改写匹配到的结构，并服务于vLLM 的后梯度融合流程。 文档字符串强调：This pass fuses a pre-defined set of custom ops into fused ops. 关键方法包括 __init__。

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
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.quantization.utils.quant_utils import QuantKey, kFp8Dynamic64Sym, kFp8Dynamic128Sym, kFp8StaticTensorSym, kNvfp4Dynamic`, `from vllm.platforms import current_platform`, `from ..vllm_inductor_pass import VllmFusionPatternMatcherPass, VllmPatternReplacement`, `from .matcher_utils import MatcherQuantFP8, MatcherSiluAndMul`, `from .rms_quant_fusion import QUANT_OPS, empty_bf16, empty_fp32, empty_i32`
- **PyTorch / Torch 栈**: `import torch`, `from torch._higher_order_ops.auto_functionalize import auto_functionalized`, `from torch._ops import OpOverload`
- **Stdlib / 标准库**: `import itertools`, `from typing import Any`
