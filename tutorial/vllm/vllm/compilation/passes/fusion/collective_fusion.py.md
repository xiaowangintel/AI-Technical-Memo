# collective_fusion.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/compilation/passes/fusion/collective_fusion.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Fusion patterns and custom ops that combine collectives with FlashInfer matmul kernels. / 将通信集体算子与 FlashInfer 矩阵乘内核融合的模式和自定义算子。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-30)
```python
from collections.abc import Callable
from contextlib import suppress

import torch
import torch._inductor.pattern_matcher as pm
import torch.distributed.distributed_c10d as c10d
import torch.fx as fx
from torch._inductor.pattern_matcher import PatternMatcherPass
from torch.distributed._symmetric_memory import enable_symm_mem_for_group

from vllm.config import VllmConfig
from vllm.config.utils import Range
from vllm.distributed import get_tp_group
from vllm.distributed.parallel_state import (
    get_tensor_model_parallel_world_size,
)
from vllm.logger import init_logger
from vllm.platforms import current_platform
from vllm.utils.torch_utils import direct_register_custom_op

from ..inductor_pass import enable_fake_mode
from ..vllm_inductor_pass import (
    VllmFusionPatternMatcherPass,
    VllmInductorPass,
    VllmPatternMatcherPass,
    VllmPatternReplacement,
)
```
**EN:** This import block assembles the PyTorch/FX, Inductor, and vLLM modules that the rest of the file relies on for graph rewriting, backend integration, caching, or runtime control in vLLM's post-grad fusion pipeline.
**CN:** 这一组导入语句组织了 PyTorch/FX、Inductor 与 vLLM 模块，供文件后续代码在vLLM 的后梯度融合流程中完成图改写、后端集成、缓存或运行时控制。

### Constants / assignments (lines 32-34)
```python
FP8_DTYPE = current_platform.fp8_dtype()
logger = init_logger(__name__)
```
**EN:** These assignments establish shared module state such as loggers, global flags, dtype/op handles, or counters that later logic reuses throughout vLLM's post-grad fusion pipeline.
**CN:** 这些赋值语句建立了共享模块状态，例如日志器、全局开关、dtype/算子句柄或计数器，供后续代码在vLLM 的后梯度融合流程中复用。

### Function `_flashinfer_scaled_mm_out` (lines 37-74)
```python
def _flashinfer_scaled_mm_out(
    A: torch.Tensor,
    B: torch.Tensor,
    *,
    scale_a: torch.Tensor,
    scale_b: torch.Tensor,
    out: torch.Tensor,
    bias: torch.Tensor | None = None,
    scale_result: torch.Tensor | None = None,
    out_dtype: torch.dtype | None = None,
    use_fast_accum: bool = False,
) -> None:
    # Import lazily to avoid a circular import during module initialization
    # when docs or other tooling import the pass without FlashInfer.
    from vllm.utils.flashinfer import flashinfer_scaled_fp8_mm_out

...
```
**EN:** Function `_flashinfer_scaled_mm_out` provides a focused helper used by the surrounding module. It fits into vLLM's post-grad fusion pipeline.
**CN:** 函数 `_flashinfer_scaled_mm_out` 提供供周边模块使用的聚焦辅助逻辑，并嵌入vLLM 的后梯度融合流程。

### Function `_flashinfer_fp4_mm_out` (lines 77-104)
```python
def _flashinfer_fp4_mm_out(
    A: torch.Tensor,
    B: torch.Tensor,
    *,
    scale_a: torch.Tensor,
    scale_b: torch.Tensor,
    out: torch.Tensor,
    alpha: torch.Tensor,
    out_dtype: torch.dtype | None = None,
    use_8x4_sf_layout: bool = False,
    backend: str = "cutlass",
) -> None:
    from vllm.utils.flashinfer import flashinfer_scaled_fp4_mm_out

    assert A.ndim == 2 and B.ndim == 2 and out.ndim == 2, (
        "FlashInfer FP4 symm_mem adapter expects 2D inputs and output"
...
```
**EN:** Function `_flashinfer_fp4_mm_out` provides a focused helper used by the surrounding module. It fits into vLLM's post-grad fusion pipeline.
**CN:** 函数 `_flashinfer_fp4_mm_out` 提供供周边模块使用的聚焦辅助逻辑，并嵌入vLLM 的后梯度融合流程。

### Function `fused_flashinfer_scaled_matmul_reduce_scatter_fake` (lines 107-126)
```python
def fused_flashinfer_scaled_matmul_reduce_scatter_fake(
    A: torch.Tensor,
    B: torch.Tensor,
    A_scale: torch.Tensor,
    B_scale: torch.Tensor,
    reduce_op: str,
    orig_scatter_dim: int,
    scatter_dim_after_maybe_reshape: int,
    group_name: str,
    output_shape: list[int],
    out_dtype: torch.dtype | None = None,
) -> torch.Tensor:
    world_size = c10d._resolve_process_group(group_name).size()
    result_shape = list(output_shape)
    result_shape[orig_scatter_dim] //= world_size
    return torch.empty(
...
```
**EN:** Function `fused_flashinfer_scaled_matmul_reduce_scatter_fake` defines a fused helper/kernel entry used by pattern-replacement logic. It fits into vLLM's post-grad fusion pipeline.
**CN:** 函数 `fused_flashinfer_scaled_matmul_reduce_scatter_fake` 定义供模式替换逻辑调用的融合辅助函数/内核入口，并嵌入vLLM 的后梯度融合流程。

### Function `fused_flashinfer_scaled_matmul_reduce_scatter` (lines 129-173)
```python
def fused_flashinfer_scaled_matmul_reduce_scatter(
    A: torch.Tensor,
    B: torch.Tensor,
    A_scale: torch.Tensor,
    B_scale: torch.Tensor,
    reduce_op: str,
    orig_scatter_dim: int,
    scatter_dim_after_maybe_reshape: int,
    group_name: str,
    output_shape: list[int],
    out_dtype: torch.dtype | None = None,
) -> torch.Tensor:
    assert orig_scatter_dim == 0 and scatter_dim_after_maybe_reshape == 0, (
        "FlashInfer symm_mem adapter currently only supports scatter_dim=0"
    )
    world_size = c10d._resolve_process_group(group_name).size()
...
```
**EN:** Function `fused_flashinfer_scaled_matmul_reduce_scatter` defines a fused helper/kernel entry used by pattern-replacement logic. It fits into vLLM's post-grad fusion pipeline.
**CN:** 函数 `fused_flashinfer_scaled_matmul_reduce_scatter` 定义供模式替换逻辑调用的融合辅助函数/内核入口，并嵌入vLLM 的后梯度融合流程。

### Function `fused_all_gather_flashinfer_scaled_matmul_fake` (lines 176-193)
```python
def fused_all_gather_flashinfer_scaled_matmul_fake(
    A_shard: torch.Tensor,
    B: torch.Tensor,
    A_scale: torch.Tensor,
    B_scale: torch.Tensor,
    gather_dim: int,
    group_name: str,
    out_dtype: torch.dtype | None = None,
) -> torch.Tensor:
    world_size = c10d._resolve_process_group(group_name).size()
    output_shape = list(A_shard.shape)
    output_shape[gather_dim] *= world_size
    output_shape[-1] = B.shape[1]
    return torch.empty(
        output_shape,
        dtype=out_dtype or torch.bfloat16,
...
```
**EN:** Function `fused_all_gather_flashinfer_scaled_matmul_fake` defines a fused helper/kernel entry used by pattern-replacement logic. It fits into vLLM's post-grad fusion pipeline.
**CN:** 函数 `fused_all_gather_flashinfer_scaled_matmul_fake` 定义供模式替换逻辑调用的融合辅助函数/内核入口，并嵌入vLLM 的后梯度融合流程。

### Function `fused_all_gather_flashinfer_scaled_matmul` (lines 196-227)
```python
def fused_all_gather_flashinfer_scaled_matmul(
    A_shard: torch.Tensor,
    B: torch.Tensor,
    A_scale: torch.Tensor,
    B_scale: torch.Tensor,
    gather_dim: int,
    group_name: str,
    out_dtype: torch.dtype | None = None,
) -> torch.Tensor:
    assert gather_dim == 0, (
        "FlashInfer symm_mem adapter currently only supports gather_dim=0"
    )
    _, outputs = torch.distributed._symmetric_memory._fused_all_gather_matmul_impl(
        mm_out_op=_flashinfer_scaled_mm_out,
        A_shard=A_shard,
        Bs=[B],
...
```
**EN:** Function `fused_all_gather_flashinfer_scaled_matmul` defines a fused helper/kernel entry used by pattern-replacement logic. It fits into vLLM's post-grad fusion pipeline.
**CN:** 函数 `fused_all_gather_flashinfer_scaled_matmul` 定义供模式替换逻辑调用的融合辅助函数/内核入口，并嵌入vLLM 的后梯度融合流程。

### Function `fused_all_gather_flashinfer_fp4_matmul_fake` (lines 230-251)
```python
def fused_all_gather_flashinfer_fp4_matmul_fake(
    A_shard: torch.Tensor,
    B: torch.Tensor,
    A_scale_shard: torch.Tensor,
    B_scale: torch.Tensor,
    alpha: torch.Tensor,
    gather_dim: int,
    group_name: str,
    out_dtype: torch.dtype | None = None,
    view_a_scale_as_fp8: bool = False,
    use_8x4_sf_layout: bool = False,
    backend: str = "cutlass",
) -> torch.Tensor:
    world_size = c10d._resolve_process_group(group_name).size()
    output_shape = list(A_shard.shape)
    output_shape[gather_dim] *= world_size
...
```
**EN:** Function `fused_all_gather_flashinfer_fp4_matmul_fake` defines a fused helper/kernel entry used by pattern-replacement logic. It fits into vLLM's post-grad fusion pipeline.
**CN:** 函数 `fused_all_gather_flashinfer_fp4_matmul_fake` 定义供模式替换逻辑调用的融合辅助函数/内核入口，并嵌入vLLM 的后梯度融合流程。

### Function `fused_all_gather_flashinfer_fp4_matmul` (lines 254-311)
```python
def fused_all_gather_flashinfer_fp4_matmul(
    A_shard: torch.Tensor,
    B: torch.Tensor,
    A_scale_shard: torch.Tensor,
    B_scale: torch.Tensor,
    alpha: torch.Tensor,
    gather_dim: int,
    group_name: str,
    out_dtype: torch.dtype | None = None,
    view_a_scale_as_fp8: bool = False,
    use_8x4_sf_layout: bool = False,
    backend: str = "cutlass",
) -> torch.Tensor:
    assert gather_dim == 0, (
        "FlashInfer FP4 symm_mem adapter currently only supports gather_dim=0"
    )
...
```
**EN:** Function `fused_all_gather_flashinfer_fp4_matmul` defines a fused helper/kernel entry used by pattern-replacement logic. It fits into vLLM's post-grad fusion pipeline.
**CN:** 函数 `fused_all_gather_flashinfer_fp4_matmul` 定义供模式替换逻辑调用的融合辅助函数/内核入口，并嵌入vLLM 的后梯度融合流程。

### Class `BasePattern` (lines 333-338)
```python
class BasePattern:
    def __init__(self, dtype: torch.dtype, device: str | None) -> None:
        self.dtype = dtype
        self.device = device
        self.tp = get_tp_group()
        self.tp_size = get_tensor_model_parallel_world_size()
```
**EN:** Class `BasePattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__.
**CN:** 类 `BasePattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__。

### Class `GEMMReduceScatterPattern` (lines 341-371)
```python
class GEMMReduceScatterPattern(BasePattern):
    def get_inputs(self) -> list[torch.Tensor]:
        mul = torch.empty([16, 4], device=self.device, dtype=self.dtype)
        mm_weight = torch.empty([4, 4], device=self.device, dtype=self.dtype)
        return [mul, mm_weight]

    def register(self, pm_pass: PatternMatcherPass) -> None:
        def pattern(mul: torch.Tensor, mm_weight: torch.Tensor) -> torch.Tensor:
            mm = torch.ops.aten.mm.default(mul, mm_weight)
            reduce_scatter = torch.ops.vllm.reduce_scatter.default(
                mm,
                dim=0,
                world_size=self.tp_size,
                group_name=self.tp.unique_name,
            )
            return reduce_scatter

        def replacement(mul: torch.Tensor, mm_weight: torch.Tensor) -> torch.Tensor:
            gemm_rs = torch.ops.symm_mem.fused_matmul_reduce_scatter(
                mul,
...
```
**EN:** Class `GEMMReduceScatterPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include get_inputs, register.
**CN:** 类 `GEMMReduceScatterPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 get_inputs, register。

### Class `AllGatherGEMMPattern` (lines 374-406)
```python
class AllGatherGEMMPattern(BasePattern):
    def get_inputs(self) -> list[torch.Tensor]:
        x = torch.empty([4, 4], device=self.device, dtype=self.dtype)
        weight = torch.empty([4, 4], device=self.device, dtype=self.dtype)

        return [x, weight]

    def register(self, pm_pass: PatternMatcherPass) -> None:
        def pattern(
            x: torch.Tensor,
            weight: torch.Tensor,
        ) -> torch.Tensor:
            all_gather = torch.ops.vllm.all_gather.default(
                x,
                dim=0,
                world_size=self.tp_size,
                group_name=self.tp.unique_name,
            )

            return torch.ops.aten.mm.default(all_gather, weight)
...
```
**EN:** Class `AllGatherGEMMPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include get_inputs, register.
**CN:** 类 `AllGatherGEMMPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 get_inputs, register。

### Class `ScaledMMReduceScatterPattern` (lines 409-474)
```python
class ScaledMMReduceScatterPattern(BasePattern):
    def get_inputs(self) -> list[torch.Tensor]:
        input = torch.empty([16, 16], device=self.device, dtype=FP8_DTYPE)
        mm_weight = (
            torch.empty([16, 16], device=self.device, dtype=FP8_DTYPE)
            .contiguous()
            .transpose(0, 1)
        )
        scale_a = torch.empty([16, 1], device=self.device, dtype=torch.float32)
        scale_b = torch.empty([1, 16], device=self.device, dtype=torch.float32)
        return [input, mm_weight, scale_a, scale_b]

    def register(self, pm_pass: PatternMatcherPass) -> None:
        def pattern(
            input: torch.Tensor,
            mat2: torch.Tensor,
            scale_a: torch.Tensor,
            scale_b: torch.Tensor,
        ) -> torch.Tensor:
            scaled_mm = torch.ops.aten._scaled_mm.default(
...
```
**EN:** Class `ScaledMMReduceScatterPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include get_inputs, register.
**CN:** 类 `ScaledMMReduceScatterPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 get_inputs, register。

### Class `AllGatherScaledMMPattern` (lines 477-536)
```python
class AllGatherScaledMMPattern(BasePattern):
    def get_inputs(self) -> list[torch.Tensor]:
        x = torch.empty([8, 16], device=self.device, dtype=FP8_DTYPE)
        weight = (
            torch.empty([16, 16], device=self.device, dtype=FP8_DTYPE)
            .contiguous()
            .transpose(0, 1)
        )

        s1 = x.shape[0] * self.tp_size

        scale_a = torch.empty([s1, 1], device=self.device, dtype=torch.float32)
        scale_b = torch.empty([1, 16], device=self.device, dtype=torch.float32)

        return [x, weight, scale_a, scale_b]

    def register(self, pm_pass: PatternMatcherPass) -> None:
        def pattern(
            x: torch.Tensor,
            weight: torch.Tensor,
...
```
**EN:** Class `AllGatherScaledMMPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include get_inputs, register.
**CN:** 类 `AllGatherScaledMMPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 get_inputs, register。

### Class `CutlassScaledMMReduceScatterPattern` (lines 539-609)
```python
class CutlassScaledMMReduceScatterPattern(BasePattern):
    def get_inputs(self) -> list[torch.Tensor]:
        input = torch.empty([16, 16], device=self.device, dtype=FP8_DTYPE)
        mm_weight = (
            torch.empty([16, 16], device=self.device, dtype=FP8_DTYPE)
            .contiguous()
            .transpose(0, 1)
        )
        scale_a = torch.empty([16, 1], device=self.device, dtype=torch.float32)
        scale_b = torch.empty([1, 16], device=self.device, dtype=torch.float32)

        cutlass_mm_output = torch.empty([16, 16], device=self.device, dtype=self.dtype)
        return [input, mm_weight, scale_a, scale_b, cutlass_mm_output]

    def register(self, pm_pass: PatternMatcherPass) -> None:
        def pattern(
            input: torch.Tensor,
            weight: torch.Tensor,
            scale_a: torch.Tensor,
            scale_b: torch.Tensor,
...
```
**EN:** Class `CutlassScaledMMReduceScatterPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include get_inputs, register.
**CN:** 类 `CutlassScaledMMReduceScatterPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 get_inputs, register。

### Class `AllGatherCutlassScaledMMPattern` (lines 612-677)
```python
class AllGatherCutlassScaledMMPattern(BasePattern):
    def get_inputs(self) -> list[torch.Tensor]:
        x = torch.empty([8, 16], device=self.device, dtype=FP8_DTYPE)
        weight = (
            torch.empty([16, 16], device=self.device, dtype=FP8_DTYPE)
            .contiguous()
            .transpose(0, 1)
        )

        s1 = x.shape[0] * self.tp_size

        scale_a = torch.empty([s1, 1], device=self.device, dtype=torch.float32)
        scale_b = torch.empty([1, 16], device=self.device, dtype=torch.float32)

        s2 = weight.shape[1]
        output = torch.empty([s1, s2], device=self.device, dtype=self.dtype)

        return [x, weight, scale_a, scale_b, output]

    def register(self, pm_pass: PatternMatcherPass) -> None:
...
```
**EN:** Class `AllGatherCutlassScaledMMPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include get_inputs, register.
**CN:** 类 `AllGatherCutlassScaledMMPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 get_inputs, register。

### Class `FlashInferBMMFP8ReduceScatterPattern` (lines 680-741)
```python
class FlashInferBMMFP8ReduceScatterPattern(
    BasePattern, VllmPatternReplacement[..., torch.Tensor]
):
    def get_inputs(self) -> list[torch.Tensor]:
        a_2d = torch.empty([16, 16], device=self.device, dtype=FP8_DTYPE)
        b_2d = (
            torch.empty([16, 16], device=self.device, dtype=FP8_DTYPE)
            .contiguous()
            .transpose(0, 1)
        )
        a_scale = torch.empty([1], device=self.device, dtype=torch.float32)
        b_scale = torch.empty([1], device=self.device, dtype=torch.float32)
        return [a_2d, b_2d, a_scale, b_scale]

    @property
    def pattern(self) -> Callable[..., torch.Tensor]:
        def _pattern(
            a_2d: torch.Tensor,
            b_2d: torch.Tensor,
            a_scale: torch.Tensor,
...
```
**EN:** Class `FlashInferBMMFP8ReduceScatterPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include get_inputs, pattern, replacement.
**CN:** 类 `FlashInferBMMFP8ReduceScatterPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 get_inputs, pattern, replacement。

### Class `FlashInferAllGatherBMMFP8Pattern` (lines 744-802)
```python
class FlashInferAllGatherBMMFP8Pattern(
    BasePattern, VllmPatternReplacement[..., torch.Tensor]
):
    def get_inputs(self) -> list[torch.Tensor]:
        a_shard_2d = torch.empty([8, 16], device=self.device, dtype=FP8_DTYPE)
        b_2d = (
            torch.empty([16, 16], device=self.device, dtype=FP8_DTYPE)
            .contiguous()
            .transpose(0, 1)
        )
        a_scale = torch.empty([1], device=self.device, dtype=torch.float32)
        b_scale = torch.empty([1], device=self.device, dtype=torch.float32)
        return [a_shard_2d, b_2d, a_scale, b_scale]

    @property
    def pattern(self) -> Callable[..., torch.Tensor]:
        def _pattern(
            a_shard_2d: torch.Tensor,
            b_2d: torch.Tensor,
            a_scale: torch.Tensor,
...
```
**EN:** Class `FlashInferAllGatherBMMFP8Pattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include get_inputs, pattern, replacement.
**CN:** 类 `FlashInferAllGatherBMMFP8Pattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 get_inputs, pattern, replacement。

### Class `FlashInferAllGatherFP4Pattern` (lines 805-897)
```python
class FlashInferAllGatherFP4Pattern(
    BasePattern, VllmPatternReplacement[..., torch.Tensor]
):
    def __init__(
        self,
        dtype: torch.dtype,
        device: str | None,
        backend: str,
        use_8x4_sf_layout: bool,
        a_scale_view: str,
    ) -> None:
        super().__init__(dtype, device)
        self.backend = backend
        self.use_8x4_sf_layout = use_8x4_sf_layout
        self.a_scale_view = a_scale_view

    def get_inputs(self) -> list[torch.Tensor]:
        a_shard_2d = torch.empty([8, 8], device=self.device, dtype=torch.uint8)
        b_2d = torch.empty([8, 16], device=self.device, dtype=torch.uint8)
        a_scale_shard = torch.empty([128, 4], device=self.device, dtype=torch.int32)
...
```
**EN:** Class `FlashInferAllGatherFP4Pattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__, get_inputs, pattern, replacement.
**CN:** 类 `FlashInferAllGatherFP4Pattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__, get_inputs, pattern, replacement。

### Class `AsyncTPPass` (lines 900-980)
```python
class AsyncTPPass(VllmFusionPatternMatcherPass):
    @enable_fake_mode
    def __init__(self, config: VllmConfig) -> None:
        super().__init__(config, pass_name="async_tp_pass")

        enable_symm_mem_for_group(get_tp_group().device_group.group_name)
        GEMMReduceScatterPattern(self.model_dtype, self.device).register(self.pm_pass)

        AllGatherGEMMPattern(self.model_dtype, self.device).register(self.pm_pass)

        # These fusions are enabled only for bfloat16 models because
        # `scaled_mm` or `cutlass_scaled_mm` with per-token (row-wise) scaling
        # only supports bfloat16 as the output dtype.
        if self.model_dtype == torch.bfloat16:
            ScaledMMReduceScatterPattern(self.model_dtype, self.device).register(
                self.pm_pass
            )
            AllGatherScaledMMPattern(self.model_dtype, self.device).register(
                self.pm_pass
            )
...
```
**EN:** Class `AsyncTPPass` defines a transformation pass that walks an FX/IR graph and rewrites matched structures. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__, is_applicable_for_range, __call__.
**CN:** 类 `AsyncTPPass` 定义了一个变换 pass，用于遍历 FX/IR 图并改写匹配到的结构，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__, is_applicable_for_range, __call__。

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
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig`, `from vllm.config.utils import Range`, `from vllm.distributed import get_tp_group`, `from vllm.distributed.parallel_state import get_tensor_model_parallel_world_size`, `from vllm.logger import init_logger`, `from vllm.platforms import current_platform`, `from vllm.utils.torch_utils import direct_register_custom_op`, `from ..inductor_pass import enable_fake_mode`, ...
- **PyTorch / Torch 栈**: `import torch`, `import torch._inductor.pattern_matcher as pm`, `import torch.distributed.distributed_c10d as c10d`, `import torch.fx as fx`, `from torch._inductor.pattern_matcher import PatternMatcherPass`, `from torch.distributed._symmetric_memory import enable_symm_mem_for_group`
- **Stdlib / 标准库**: `from collections.abc import Callable`, `from contextlib import suppress`
