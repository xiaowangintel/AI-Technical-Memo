# allreduce_rms_fusion.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/compilation/passes/fusion/allreduce_rms_fusion.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Fusion patterns for all-reduce plus RMSNorm/add-RMSNorm and quantized variants. / 将 all-reduce 与 RMSNorm/加法 RMSNorm 及其量化变体融合的模式。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-43)
```python
import contextlib
from importlib.util import find_spec
from types import ModuleType
from typing import Any

import torch
import torch._inductor.pattern_matcher as pm
import torch.fx as fx
from torch._higher_order_ops.auto_functionalize import auto_functionalized
from torch._inductor.pattern_matcher import PatternMatcherPass

import vllm.ir.ops
from vllm._aiter_ops import rocm_aiter_ops
from vllm.compilation.passes.fusion.rms_quant_fusion import (
    _rms_input_weight_dtype_match,
)
from vllm.config import VllmConfig
from vllm.config.utils import Range
from vllm.distributed import get_tp_group, tensor_model_parallel_all_reduce
from vllm.distributed.device_communicators.custom_all_reduce import CustomAllreduce
from vllm.distributed.parallel_state import (
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
)
from vllm.logger import init_logger
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    kFp8StaticTensorSym,
)
...
```
**EN:** This import block assembles the PyTorch/FX, Inductor, and vLLM modules that the rest of the file relies on for graph rewriting, backend integration, caching, or runtime control in vLLM's post-grad fusion pipeline.
**CN:** 这一组导入语句组织了 PyTorch/FX、Inductor 与 vLLM 模块，供文件后续代码在vLLM 的后梯度融合流程中完成图改写、后端集成、缓存或运行时控制。

### Constants / assignments (lines 45-103)
```python
FP8_DTYPE = current_platform.fp8_dtype()
logger = init_logger(__name__)
flashinfer_comm: ModuleType | None = None
FI_ALLREDUCE_FUSION_MAX_SIZE_MB: dict[int, dict[int, float]] = {
    90: {
        2: 64,  # 64MB
        4: 2,  # 2MB
        8: 0.5,  # 0.5MB
    },
    100: {
        2: 64,  # 64MB
        4: 32,  # 32MB
        8: 1,  # 1MB
    },
    103: {
        2: 64,  # 64MB
        4: 64,  # 64MB
        8: 2,  # 2MB
    },
}
_FI_ALLREDUCE_ONE_SHOT_MAX_SIZES_MB: dict[int, dict[int, float]] = {
    90: {
        2: 32,  # 32MB
        4: 2,  # 2MB
...
```
**EN:** These assignments establish shared module state such as loggers, global flags, dtype/op handles, or counters that later logic reuses throughout vLLM's post-grad fusion pipeline.
**CN:** 这些赋值语句建立了共享模块状态，例如日志器、全局开关、dtype/算子句柄或计数器，供后续代码在vLLM 的后梯度融合流程中复用。

### Class `FlashInferFusedAllReduceParams` (lines 243-262)
```python
class FlashInferFusedAllReduceParams:
    """Parameters for FlashInfer fused allreduce operations."""

    def __init__(
        self,
        world_size: int,
        max_token_num: int = 1024,
    ) -> None:
        self.world_size = world_size
        self.launch_with_pdl = True
        self.fp32_acc = True
        self.max_token_num = max_token_num

    def get_trtllm_fused_allreduce_kwargs(self) -> dict[str, bool | int]:
        return {
            "world_size": self.world_size,
            "launch_with_pdl": self.launch_with_pdl,
            "fp32_acc": self.fp32_acc,
            "max_token_num": self.max_token_num,
        }
```
**EN:** Class `FlashInferFusedAllReduceParams` holds a main piece of file-specific logic. It operates inside vLLM's post-grad fusion pipeline. Parameters for FlashInfer fused allreduce operations. Key methods include __init__, get_trtllm_fused_allreduce_kwargs.
**CN:** 类 `FlashInferFusedAllReduceParams` 承载了该文件中的一块核心逻辑，并服务于vLLM 的后梯度融合流程。 文档字符串强调：Parameters for FlashInfer fused allreduce operations. 关键方法包括 __init__, get_trtllm_fused_allreduce_kwargs。

### Class `BasePattern` (lines 266-277)
```python
class BasePattern:
    def __init__(self, dtype: torch.dtype, device: str | None) -> None:
        self.dtype = dtype
        self.device = device
        self.tp = get_tp_group()
        self.tp_size = get_tensor_model_parallel_world_size()

    def empty(self, *args: Any, **kwargs: Any) -> torch.Tensor:
        return torch.empty(*args, dtype=self.dtype, device=self.device, **kwargs)

    def empty_f32(self, *args: Any, **kwargs: Any) -> torch.Tensor:
        return torch.empty(*args, dtype=torch.float32, device=self.device, **kwargs)
```
**EN:** Class `BasePattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__, empty, empty_f32.
**CN:** 类 `BasePattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__, empty, empty_f32。

### Class `AllReduceRMSNormPattern` (lines 280-339)
```python
class AllReduceRMSNormPattern(BasePattern):
    """
    This pattern replaces the allreduce + rms norm (without residual)
    with fused flashinfer implementation.
    Applies to allreduce + rmsnorm before attn in the first Transformer block.
    """

    def __init__(
        self,
        epsilon: float,
        dtype: torch.dtype,
        device: str | None,
        allreduce_params: FlashInferFusedAllReduceParams,
    ) -> None:
        super().__init__(dtype, device)
        self.epsilon = epsilon
        self.allreduce_params = allreduce_params

    def get_inputs(self) -> list[torch.Tensor]:
        # input, weight
...
```
**EN:** Class `AllReduceRMSNormPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. This pattern replaces the allreduce + rms norm (without residual) with fused flashinfer implementation. Key methods include __init__, get_inputs, register.
**CN:** 类 `AllReduceRMSNormPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 文档字符串强调：This pattern replaces the allreduce + rms norm (without residual) with fused flashinfer implementation. 关键方法包括 __init__, get_inputs, register。

### Class `AllReduceFusedAddRMSNormPattern` (lines 342-411)
```python
class AllReduceFusedAddRMSNormPattern(BasePattern):
    """
    This pattern replaces the allreduce + rms norm (with residual)
    with fused flashinfer implementation.
    Applies to o_proj + rmsnorm after attn and mlp + rmsnorm before attn.
    """

    def __init__(
        self,
        epsilon: float,
        dtype: torch.dtype,
        device: str | None,
        allreduce_params: FlashInferFusedAllReduceParams,
    ) -> None:
        super().__init__(dtype, device)
        self.epsilon = epsilon
        self.allreduce_params = allreduce_params

    def get_inputs(self) -> list[torch.Tensor]:
        input = self.empty(5, 16)
...
```
**EN:** Class `AllReduceFusedAddRMSNormPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. This pattern replaces the allreduce + rms norm (with residual) with fused flashinfer implementation. Key methods include __init__, get_inputs, register.
**CN:** 类 `AllReduceFusedAddRMSNormPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 文档字符串强调：This pattern replaces the allreduce + rms norm (with residual) with fused flashinfer implementation. 关键方法包括 __init__, get_inputs, register。

### Class `AllReduceFusedRMSNormStaticQuantFP8Pattern` (lines 414-486)
```python
class AllReduceFusedRMSNormStaticQuantFP8Pattern(BasePattern):
    """
    This pattern replaces the allreduce + rms norm (without residual)
    + static fp8 quant with fused flashinfer implementation.
    Applies to allreduce + rmsnorm + quant before attn
    in the first Transformer block.
    """

    def __init__(
        self,
        epsilon: float,
        dtype: torch.dtype,
        device: str | None,
        allreduce_params: FlashInferFusedAllReduceParams,
    ) -> None:
        super().__init__(dtype, device)
        self.epsilon = epsilon
        self.allreduce_params = allreduce_params
        self.quant_dtype = torch.float8_e4m3fn
        self.quant_matcher = MatcherQuantFP8(kFp8StaticTensorSym)
...
```
**EN:** Class `AllReduceFusedRMSNormStaticQuantFP8Pattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. This pattern replaces the allreduce + rms norm (without residual) + static fp8 quant with fused flashinfer implementation. Key methods include __init__, get_inputs, register.
**CN:** 类 `AllReduceFusedRMSNormStaticQuantFP8Pattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 文档字符串强调：This pattern replaces the allreduce + rms norm (without residual) + static fp8 quant with fused flashinfer implementation. 关键方法包括 __init__, get_inputs, register。

### Class `AllReduceFusedAddRMSNormStaticQuantFP8Pattern` (lines 489-564)
```python
class AllReduceFusedAddRMSNormStaticQuantFP8Pattern(BasePattern):
    """
    This pattern replaces the allreduce + rms norm (with residual)
    + static fp8 quant with fused flashinfer implementation.
    Applies to o_proj + rmsnorm after attn + quant and
    mlp + rmsnorm + quant before attn.
    """

    def __init__(
        self,
        epsilon: float,
        dtype: torch.dtype,
        device: str | None,
        allreduce_params: FlashInferFusedAllReduceParams,
    ) -> None:
        super().__init__(dtype, device)
        self.epsilon = epsilon
        self.allreduce_params = allreduce_params
        self.quant_dtype = torch.float8_e4m3fn

...
```
**EN:** Class `AllReduceFusedAddRMSNormStaticQuantFP8Pattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. This pattern replaces the allreduce + rms norm (with residual) + static fp8 quant with fused flashinfer implementation. Key methods include __init__, get_inputs, register.
**CN:** 类 `AllReduceFusedAddRMSNormStaticQuantFP8Pattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 文档字符串强调：This pattern replaces the allreduce + rms norm (with residual) + static fp8 quant with fused flashinfer implementation. 关键方法包括 __init__, get_inputs, register。

### Class `AllReduceFusedRMSNormStaticQuantNVFP4Pattern` (lines 567-656)
```python
class AllReduceFusedRMSNormStaticQuantNVFP4Pattern(BasePattern):
    """
    This pattern replaces the allreduce + rms norm (without residual)
    + static nvfp4 quant with fused flashinfer implementation.
    Applies to allreduce + rmsnorm + quant before attn
    in the first Transformer block.
    """

    def __init__(
        self,
        epsilon: float,
        dtype: torch.dtype,
        device: str | None,
        allreduce_params: FlashInferFusedAllReduceParams,
    ) -> None:
        super().__init__(dtype, device)
        self.epsilon = epsilon
        self.allreduce_params = allreduce_params

    def get_inputs(self) -> list[torch.Tensor]:
...
```
**EN:** Class `AllReduceFusedRMSNormStaticQuantNVFP4Pattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. This pattern replaces the allreduce + rms norm (without residual) + static nvfp4 quant with fused flashinfer implementation. Key methods include __init__, get_inputs, register.
**CN:** 类 `AllReduceFusedRMSNormStaticQuantNVFP4Pattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 文档字符串强调：This pattern replaces the allreduce + rms norm (without residual) + static nvfp4 quant with fused flashinfer implementation. 关键方法包括 __init__, get_inputs, register。

### Class `AllReduceFusedAddRMSNormStaticQuantNVFP4Pattern` (lines 659-753)
```python
class AllReduceFusedAddRMSNormStaticQuantNVFP4Pattern(BasePattern):
    """
    This pattern replaces the allreduce + rms norm (with residual)
    + static nvfp4 quant with fused flashinfer implementation.
    Applies to o_proj + rmsnorm after attn + quant and
    mlp + rmsnorm + quant before attn.
    """

    def __init__(
        self,
        epsilon: float,
        dtype: torch.dtype,
        device: str | None,
        allreduce_params: FlashInferFusedAllReduceParams,
    ) -> None:
        super().__init__(dtype, device)
        self.epsilon = epsilon
        self.allreduce_params = allreduce_params

    def get_inputs(self) -> list[torch.Tensor]:
...
```
**EN:** Class `AllReduceFusedAddRMSNormStaticQuantNVFP4Pattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. This pattern replaces the allreduce + rms norm (with residual) + static nvfp4 quant with fused flashinfer implementation. Key methods include __init__, get_inputs, register.
**CN:** 类 `AllReduceFusedAddRMSNormStaticQuantNVFP4Pattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 文档字符串强调：This pattern replaces the allreduce + rms norm (with residual) + static nvfp4 quant with fused flashinfer implementation. 关键方法包括 __init__, get_inputs, register。

### Class `AllReduceFusionPass` (lines 756-905)
```python
class AllReduceFusionPass(VllmPatternMatcherPass):
    def __init__(self, config: VllmConfig) -> None:
        super().__init__(config)
        self.disabled = True
        self.tp_size = get_tensor_model_parallel_world_size()
        if self.tp_size <= 1:
            logger.warning_once("AllReduce fusion pass is disabled for tp_size <= 1.")
            return
        self.patterns: PatternMatcherPass = PatternMatcherPass(
            pass_name="all_reduce_fusion_pass"
        )
        if config.model_config is None:
            logger.warning_once(
                "AllReduce fusion pass is disabled for missing model_config."
            )
            return
        self.hidden_dim = config.model_config.get_hidden_size()
        self.group = get_tp_group().device_group
        rank = get_tensor_model_parallel_rank()
        if flashinfer_comm is None:
...
```
**EN:** Class `AllReduceFusionPass` defines a transformation pass that walks an FX/IR graph and rewrites matched structures. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__, register_patterns, is_applicable_for_range, __call__, __del__.
**CN:** 类 `AllReduceFusionPass` 定义了一个变换 pass，用于遍历 FX/IR 图并改写匹配到的结构，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__, register_patterns, is_applicable_for_range, __call__, __del__。

### Class `AiterAllreduceFusedRMSNormPattern` (lines 909-951)
```python
class AiterAllreduceFusedRMSNormPattern(BasePattern, VllmPatternReplacement):
    def __init__(
        self,
        epsilon: float,
        dtype: torch.dtype,
        device: str | None,
        use_aiter_rmsnorm: bool = True,
    ) -> None:
        super().__init__(dtype, device)
        self.dtype = dtype
        self.epsilon = epsilon
        self.FUSED_AR_RMSNORM_OP = rocm_aiter_ops.get_fused_allreduce_rmsnorm_op()

    def get_inputs(self) -> list[torch.Tensor]:
        return [self.empty(5, 16), self.empty(16)]

    @property
    def pattern(self):
        def _pattern(
            input: torch.Tensor, weight: torch.Tensor
...
```
**EN:** Class `AiterAllreduceFusedRMSNormPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__, get_inputs, pattern, replacement.
**CN:** 类 `AiterAllreduceFusedRMSNormPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__, get_inputs, pattern, replacement。

### Class `AiterAllreduceFusedAddRMSNormPattern` (lines 954-997)
```python
class AiterAllreduceFusedAddRMSNormPattern(BasePattern, VllmPatternReplacement):
    def __init__(
        self,
        epsilon: float,
        dtype: torch.dtype,
        device: str | None,
        use_aiter_rmsnorm: bool = True,
    ) -> None:
        super().__init__(dtype, device)
        self.epsilon = epsilon
        self.dtype = dtype
        self.FUSED_AR_RMSNORM_OP = rocm_aiter_ops.get_fused_allreduce_rmsnorm_op()

    def get_inputs(self) -> list[torch.Tensor]:
        # input, residual, weight
        return [self.empty(5, 16), self.empty(5, 16), self.empty(16)]

    @property
    def pattern(self):
        def _pattern(
...
```
**EN:** Class `AiterAllreduceFusedAddRMSNormPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__, get_inputs, pattern, replacement.
**CN:** 类 `AiterAllreduceFusedAddRMSNormPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__, get_inputs, pattern, replacement。

### Class `RocmAiterAllReduceFusionPass` (lines 1000-1103)
```python
class RocmAiterAllReduceFusionPass(VllmFusionPatternMatcherPass):
    def __init__(self, config: VllmConfig) -> None:
        super().__init__(config, "rocm_aiter_allreduce_fusion_pass")
        self.disabled = True
        self.tp_size = get_tensor_model_parallel_world_size()
        if self.tp_size <= 1:
            logger.warning_once("AllReduce fusion pass is disabled for tp_size <= 1.")
            return

        if config.model_config is None:
            logger.warning_once(
                "AllReduce fusion pass is disabled for missing model_config."
            )
            return

        device_comm = get_tp_group().device_communicator
        if device_comm is None:
            logger.warning_once("Device communicator is required.")
            return

...
```
**EN:** Class `RocmAiterAllReduceFusionPass` defines a transformation pass that walks an FX/IR graph and rewrites matched structures. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__, is_applicable_for_range, __del__.
**CN:** 类 `RocmAiterAllReduceFusionPass` 定义了一个变换 pass，用于遍历 FX/IR 图并改写匹配到的结构，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__, is_applicable_for_range, __del__。

## Key Concepts / 关键概念
- **Compilation pipeline / 编译流水线**
  - **EN:** The file participates in turning FX graphs into optimized executable code or runtime wrappers.
  - **CN:** 该文件参与将 FX 图转换为优化后的可执行代码或运行时封装。
- **Pattern-based graph fusion / 基于模式的图融合**
  - **EN:** The implementation matches recognizable FX subgraphs and replaces them with fewer, more specialized ops.
  - **CN:** 该实现会匹配可识别的 FX 子图，并将其替换为数量更少、专用性更强的算子。
- **Distributed collectives / 分布式通信集体算子**
  - **EN:** The file reasons about all-reduce, all-gather, reduce-scatter, or sequence-parallel communication.
  - **CN:** 该文件会处理 all-reduce、all-gather、reduce-scatter 或序列并行通信相关逻辑。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `import vllm.ir.ops`, `from vllm._aiter_ops import rocm_aiter_ops`, `from vllm.compilation.passes.fusion.rms_quant_fusion import _rms_input_weight_dtype_match`, `from vllm.config import VllmConfig`, `from vllm.config.utils import Range`, `from vllm.distributed import get_tp_group, tensor_model_parallel_all_reduce`, `from vllm.distributed.device_communicators.custom_all_reduce import CustomAllreduce`, `from vllm.distributed.parallel_state import get_tensor_model_parallel_rank, get_tensor_model_parallel_world_size`, ...
- **PyTorch / Torch 栈**: `import torch`, `import torch._inductor.pattern_matcher as pm`, `import torch.fx as fx`, `from torch._higher_order_ops.auto_functionalize import auto_functionalized`, `from torch._inductor.pattern_matcher import PatternMatcherPass`
- **Stdlib / 标准库**: `import contextlib`, `from importlib.util import find_spec`, `from types import ModuleType`, `from typing import Any`
