# sequence_parallelism.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/compilation/passes/fusion/sequence_parallelism.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Fusion helpers and passes for sequence-parallel collectives around RMSNorm blocks. / 围绕 RMSNorm 模块进行序列并行通信融合的辅助逻辑与 pass。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-30)
```python
import functools
from collections.abc import Callable, Sequence
from typing import Any

import torch
import torch._inductor.pattern_matcher as pm
import torch.fx as fx
from torch._higher_order_ops.auto_functionalize import auto_functionalized
from torch._inductor.pattern_matcher import PatternMatcherPass

import vllm.ir.ops
from vllm.config import VllmConfig
from vllm.config.utils import Range
from vllm.distributed import get_tp_group, tensor_model_parallel_all_reduce
from vllm.distributed.parallel_state import (
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
)
from vllm.logger import init_logger
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    kFp8StaticTensorSym,
)

from ..inductor_pass import enable_fake_mode
from ..utility.noop_elimination import NoOpEliminationPass
from ..vllm_inductor_pass import VllmInductorPass, VllmPatternMatcherPass
from .matcher_utils import MatcherQuantFP8
```
**EN:** This import block assembles the PyTorch/FX, Inductor, and vLLM modules that the rest of the file relies on for graph rewriting, backend integration, caching, or runtime control in vLLM's post-grad fusion pipeline.
**CN:** 这一组导入语句组织了 PyTorch/FX、Inductor 与 vLLM 模块，供文件后续代码在vLLM 的后梯度融合流程中完成图改写、后端集成、缓存或运行时控制。

### Constants / assignments (lines 32-52)
```python
logger = init_logger(__name__)
SP_MIN_HIDDEN_SIZE: dict[int, int] = {
    90: 8192,  # H100: only for models with hidden_size >= 8192
    100: 8192,  # Blackwell family: only for models with hidden_size >= 8192
}
SP_MIN_PER_GPU_SIZE_MB: dict[int, float] = {
    90: 8,  # 8MB per GPU for H100
    # Use a more conservative threshold on Blackwell so TP8 starts later.
    100: 32,
}
```
**EN:** These assignments establish shared module state such as loggers, global flags, dtype/op handles, or counters that later logic reuses throughout vLLM's post-grad fusion pipeline.
**CN:** 这些赋值语句建立了共享模块状态，例如日志器、全局开关、dtype/算子句柄或计数器，供后续代码在vLLM 的后梯度融合流程中复用。

### Function `get_sequence_parallelism_threshold` (lines 55-101)
```python
def get_sequence_parallelism_threshold(
    hidden_size: int,
    tp_size: int,
    element_size: int,
) -> int | None:
    """
    Calculate the minimum token threshold for applying sequence parallelism.

    Returns None if sequence parallelism should not be applied based on model size.

    Branching logic based on device capability:
    - Check if hidden_size >= SP_MIN_HIDDEN_SIZE[device_capability]
    - If not, returns None (SP disabled for small models on this device)
    - If yes, calculates threshold based on per-GPU size

    Formula: min_token_num = (min_per_gpu_size_mb * tp_size * MiB) //
...
```
**EN:** Function `get_sequence_parallelism_threshold` retrieves or derives focused state needed by later logic. It fits into vLLM's post-grad fusion pipeline. Calculate the minimum token threshold for applying sequence parallelism.
**CN:** 函数 `get_sequence_parallelism_threshold` 用于获取或推导后续逻辑所需的聚焦状态，并嵌入vLLM 的后梯度融合流程。 文档字符串说明：Calculate the minimum token threshold for applying sequence parallelism.

### Function `get_first_out_wrapper` (lines 104-111)
```python
def get_first_out_wrapper(
    fn: Callable[..., Sequence[torch.Tensor]],
) -> Callable[..., torch.Tensor]:
    @functools.wraps(fn)
    def wrapper(*args: Any) -> torch.Tensor:
        return fn(*args)[0]

    return wrapper
```
**EN:** Function `get_first_out_wrapper` retrieves or derives focused state needed by later logic. It fits into vLLM's post-grad fusion pipeline.
**CN:** 函数 `get_first_out_wrapper` 用于获取或推导后续逻辑所需的聚焦状态，并嵌入vLLM 的后梯度融合流程。

### Class `_SequenceParallelPatternHelper` (lines 114-147)
```python
class _SequenceParallelPatternHelper:
    """Helper for sequence parallelism patterns."""

    def __init__(
        self,
        epsilon: float,
        dtype: torch.dtype,
        device: str | None,
    ) -> None:
        self.epsilon = epsilon
        self.dtype = dtype
        self.device = device
        self.tp_group = get_tp_group()
        self.tp_size = get_tensor_model_parallel_world_size()
        self.tp_rank = get_tensor_model_parallel_rank()

    def _all_reduce(self, x: torch.Tensor) -> torch.Tensor:
        return tensor_model_parallel_all_reduce(x)

    def _reduce_scatter(self, x: torch.Tensor) -> torch.Tensor:
...
```
**EN:** Class `_SequenceParallelPatternHelper` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Helper for sequence parallelism patterns. Key methods include __init__, _all_reduce, _reduce_scatter, _all_gather, empty.
**CN:** 类 `_SequenceParallelPatternHelper` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 文档字符串强调：Helper for sequence parallelism patterns. 关键方法包括 __init__, _all_reduce, _reduce_scatter, _all_gather, empty。

### Class `FirstAllReduceRMSNormPattern` (lines 150-180)
```python
class FirstAllReduceRMSNormPattern(_SequenceParallelPatternHelper):
    def __init__(self, epsilon: float, dtype: torch.dtype, device: str | None) -> None:
        super().__init__(epsilon, dtype, device)

    def get_inputs(self) -> list[torch.Tensor]:
        # input, weight
        return [self.empty([1, 8, 4]), self.empty([4])]

    def register(self, pm_pass: PatternMatcherPass) -> None:
        def pattern(
            input: torch.Tensor,
            weight: torch.Tensor,
        ) -> tuple[torch.Tensor, torch.Tensor]:
            all_reduce = self._all_reduce(input)
            rmsnorm = vllm.ir.ops.rms_norm(all_reduce, weight, self.epsilon)

            return rmsnorm, all_reduce

        def replacement(
            input: torch.Tensor,
...
```
**EN:** Class `FirstAllReduceRMSNormPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__, get_inputs, register.
**CN:** 类 `FirstAllReduceRMSNormPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__, get_inputs, register。

### Class `MiddleAllReduceRMSNormPattern` (lines 183-256)
```python
class MiddleAllReduceRMSNormPattern(_SequenceParallelPatternHelper):
    def __init__(self, epsilon: float, dtype: torch.dtype, device: str | None) -> None:
        super().__init__(epsilon, dtype, device)

    def get_inputs(self) -> list[torch.Tensor]:
        mm_1 = torch.empty([4, 4], device=self.device, dtype=self.dtype)

        residual = torch.empty([4, 4], device=self.device, dtype=self.dtype)
        rms_norm_weights = torch.empty([4, 4], device=self.device, dtype=self.dtype)

        return [
            residual,
            mm_1,
            rms_norm_weights,
        ]

    def register(self, pm_pass: PatternMatcherPass) -> None:
        def pattern(
            residual: torch.Tensor,
            mm_1: torch.Tensor,
...
```
**EN:** Class `MiddleAllReduceRMSNormPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__, get_inputs, register.
**CN:** 类 `MiddleAllReduceRMSNormPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__, get_inputs, register。

### Class `FirstAllReduceRMSNormStaticFP8Pattern` (lines 259-298)
```python
class FirstAllReduceRMSNormStaticFP8Pattern(_SequenceParallelPatternHelper):
    def __init__(
        self,
        epsilon: float,
        dtype: torch.dtype,
        device: str | None,
    ) -> None:
        super().__init__(epsilon, dtype, device)
        self.quant_matcher = MatcherQuantFP8(kFp8StaticTensorSym)

    def get_inputs(self) -> list[torch.Tensor]:
        # input, weight, scale
        return [self.empty([1, 8, 4]), self.empty([4]), self.empty_f32([1, 1])]

    def register(self, pm_pass: PatternMatcherPass) -> None:
        def pattern(
            input: torch.Tensor,
            weight: torch.Tensor,
            scale: torch.Tensor,
        ) -> tuple[torch.Tensor, torch.Tensor]:
...
```
**EN:** Class `FirstAllReduceRMSNormStaticFP8Pattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__, get_inputs, register.
**CN:** 类 `FirstAllReduceRMSNormStaticFP8Pattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__, get_inputs, register。

### Class `MiddleAllReduceRMSNormStaticFP8Pattern` (lines 301-369)
```python
class MiddleAllReduceRMSNormStaticFP8Pattern(_SequenceParallelPatternHelper):
    def __init__(self, epsilon: float, dtype: torch.dtype, device: str | None) -> None:
        super().__init__(epsilon, dtype, device)
        self.quant_matcher = MatcherQuantFP8(kFp8StaticTensorSym)

    def get_inputs(self) -> list[torch.Tensor]:
        mm_1 = torch.empty([4, 4], device=self.device, dtype=self.dtype)
        residual = torch.empty([4, 4], device=self.device, dtype=self.dtype)
        rms_norm_weights = torch.empty([4, 4], device=self.device, dtype=self.dtype)
        scale = torch.empty([1, 1], device=self.device, dtype=torch.float32)

        return [residual, mm_1, rms_norm_weights, scale]

    def register(self, pm_pass: PatternMatcherPass) -> None:
        def pattern(
            residual: torch.Tensor,
            mm_1: torch.Tensor,
            rms_norm_weights: torch.Tensor,
            scale: torch.Tensor,
        ) -> tuple[torch.Tensor, torch.Tensor]:
...
```
**EN:** Class `MiddleAllReduceRMSNormStaticFP8Pattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__, get_inputs, register.
**CN:** 类 `MiddleAllReduceRMSNormStaticFP8Pattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__, get_inputs, register。

### Class `FirstAllReduceRMSNormStaticNVFP4Pattern` (lines 372-424)
```python
class FirstAllReduceRMSNormStaticNVFP4Pattern(_SequenceParallelPatternHelper):
    def get_inputs(self) -> list[torch.Tensor]:
        input = self.empty([8, 16])
        weight = self.empty([16])
        input_global_scale = self.empty_f32([1, 1])
        quant_output = torch.empty([8, 8], device=self.device, dtype=torch.uint8)
        output_scale = torch.empty([128, 4], device=self.device, dtype=torch.int32)
        return [input, weight, input_global_scale, quant_output, output_scale]

    def register(self, pm_pass: PatternMatcherPass) -> None:
        def pattern(
            input: torch.Tensor,
            weight: torch.Tensor,
            input_global_scale: torch.Tensor,
            quant_output: torch.Tensor,
            output_scale: torch.Tensor,
        ) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
            all_reduce = self._all_reduce(input)
            rms = vllm.ir.ops.rms_norm(all_reduce, weight, self.epsilon)
            quant = auto_functionalized(
...
```
**EN:** Class `FirstAllReduceRMSNormStaticNVFP4Pattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include get_inputs, register.
**CN:** 类 `FirstAllReduceRMSNormStaticNVFP4Pattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 get_inputs, register。

### Class `MiddleAllReduceRMSNormStaticNVFP4Pattern` (lines 427-492)
```python
class MiddleAllReduceRMSNormStaticNVFP4Pattern(_SequenceParallelPatternHelper):
    def get_inputs(self) -> list[torch.Tensor]:
        mm_1 = self.empty([8, 16])
        residual = self.empty([8, 16])
        rms_norm_weights = self.empty([16])
        input_global_scale = self.empty_f32([1, 1])
        quant_output = torch.empty([8, 8], device=self.device, dtype=torch.uint8)
        output_scale = torch.empty([128, 4], device=self.device, dtype=torch.int32)
        return [
            residual,
            mm_1,
            rms_norm_weights,
            input_global_scale,
            quant_output,
            output_scale,
        ]

    def register(self, pm_pass: PatternMatcherPass) -> None:
        def pattern(
            residual: torch.Tensor,
...
```
**EN:** Class `MiddleAllReduceRMSNormStaticNVFP4Pattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include get_inputs, register.
**CN:** 类 `MiddleAllReduceRMSNormStaticNVFP4Pattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 get_inputs, register。

### Class `SequenceParallelismPass` (lines 495-620)
```python
class SequenceParallelismPass(VllmPatternMatcherPass):
    """
    This pass enables sequence parallelism for models.
    It identifies patterns where an AllReduce operation is followed by
    an RMSNorm (or RMSNorm and then Quantization) operation.
    These patterns are replaced with a ReduceScatter operation, followed by
    a local RMSNorm/Quantization, and then an AllGather operation.

    The general transformation is:
    Input -> AllReduce -> RMSNorm -> Output
    becomes
    Input -> ReduceScatter -> RMSNorm -> AllGather -> Output

    While this pass itself does not directly yield performance improvements,
    it lays the groundwork for subsequent fusion passes, such as
    GEMM + ReduceScatter and AllGather + GEMM fusions. These fusions can
    significantly reduce communication overhead and improve overall model
    performance.

    This pass is only supported when compiling the whole graph (fullgraph
...
```
**EN:** Class `SequenceParallelismPass` defines a transformation pass that walks an FX/IR graph and rewrites matched structures. It operates inside vLLM's post-grad fusion pipeline. This pass enables sequence parallelism for models. Key methods include __init__, is_applicable_for_range, __call__.
**CN:** 类 `SequenceParallelismPass` 定义了一个变换 pass，用于遍历 FX/IR 图并改写匹配到的结构，并服务于vLLM 的后梯度融合流程。 文档字符串强调：This pass enables sequence parallelism for models. 关键方法包括 __init__, is_applicable_for_range, __call__。

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
- **vLLM internal / vLLM 内部依赖**: `import vllm.ir.ops`, `from vllm.config import VllmConfig`, `from vllm.config.utils import Range`, `from vllm.distributed import get_tp_group, tensor_model_parallel_all_reduce`, `from vllm.distributed.parallel_state import get_tensor_model_parallel_rank, get_tensor_model_parallel_world_size`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.quantization.utils.quant_utils import kFp8StaticTensorSym`, `from ..inductor_pass import enable_fake_mode`, ...
- **PyTorch / Torch 栈**: `import torch`, `import torch._inductor.pattern_matcher as pm`, `import torch.fx as fx`, `from torch._higher_order_ops.auto_functionalize import auto_functionalized`, `from torch._inductor.pattern_matcher import PatternMatcherPass`
- **Stdlib / 标准库**: `import functools`, `from collections.abc import Callable, Sequence`, `from typing import Any`
