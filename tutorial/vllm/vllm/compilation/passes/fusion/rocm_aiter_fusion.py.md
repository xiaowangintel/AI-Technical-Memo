# rocm_aiter_fusion.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/compilation/passes/fusion/rocm_aiter_fusion.py`
- **Repository**: vllm-project/vllm
- **Purpose**: ROCm AITER-specific fused RMSNorm, activation, and quantization patterns. / 面向 ROCm AITER 的 RMSNorm、激活与量化融合模式。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-41)
```python
from collections.abc import Callable
from typing import Any

import torch
import torch._inductor.pattern_matcher as pm
from torch import fx
from torch._inductor.fx_passes.post_grad import view_to_reshape
from torch._inductor.pattern_matcher import PatternMatcherPass

import vllm.ir.ops
import vllm.model_executor.layers.quantization.utils.fp8_utils  # noqa: F401
from vllm._aiter_ops import rocm_aiter_ops
from vllm.config import VllmConfig, get_layers_from_vllm_config
from vllm.logger import init_logger
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    GroupShape,
    QuantKey,
    ScaleDesc,
    kFp8Dynamic128Sym,
)
from vllm.platforms import current_platform

from ..inductor_pass import enable_fake_mode
from ..vllm_inductor_pass import (
    VllmFusionPatternMatcherPass,
    VllmInductorPass,
    VllmPatternMatcherPass,
    VllmPatternReplacement,
...
```
**EN:** This import block assembles the PyTorch/FX, Inductor, and vLLM modules that the rest of the file relies on for graph rewriting, backend integration, caching, or runtime control in vLLM's post-grad fusion pipeline.
**CN:** 这一组导入语句组织了 PyTorch/FX、Inductor 与 vLLM 模块，供文件后续代码在vLLM 的后梯度融合流程中完成图改写、后端集成、缓存或运行时控制。

### Constants / assignments (lines 43-44)
```python
logger = init_logger(__name__)
FP8_DTYPE = current_platform.fp8_dtype()
```
**EN:** These assignments establish shared module state such as loggers, global flags, dtype/op handles, or counters that later logic reuses throughout vLLM's post-grad fusion pipeline.
**CN:** 这些赋值语句建立了共享模块状态，例如日志器、全局开关、dtype/算子句柄或计数器，供后续代码在vLLM 的后梯度融合流程中复用。

### Class `AiterRMSNormQuantPattern` (lines 47-64)
```python
class AiterRMSNormQuantPattern:
    def __init__(
        self, epsilon: float, key: FusedRMSQuantKey, match_aiter_quant: bool = True
    ):
        self.epsilon = epsilon
        self.quant_dtype = key.quant.dtype
        self.device = torch.device("cuda")

        self.quant_matcher = MatcherQuantFP8(
            key.quant,
            match_rocm_aiter=match_aiter_quant,
        )

    def empty(self, *args: Any, **kwargs: Any) -> torch.Tensor:
        return torch.empty(*args, dtype=torch.bfloat16, device=self.device, **kwargs)

    def empty_f32(self, *args: Any, **kwargs: Any) -> torch.Tensor:
        return torch.empty(*args, dtype=torch.float32, device=self.device, **kwargs)
```
**EN:** Class `AiterRMSNormQuantPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__, empty, empty_f32.
**CN:** 类 `AiterRMSNormQuantPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__, empty, empty_f32。

### Class `AiterRMSNormDynamicQuantPattern` (lines 67-117)
```python
class AiterRMSNormDynamicQuantPattern(AiterRMSNormQuantPattern):
    """AITER RMSNorm + Dynamic Quantization pattern."""

    FUSED_OP = rocm_aiter_ops.get_rmsnorm_fused_dynamic_quant_op()

    def __init__(
        self,
        epsilon: float,
        quant_dtype: torch.dtype,
        match_aiter_quant: bool = True,
        group_shape: GroupShape = GroupShape.PER_TOKEN,
        symmetric: bool = True,
    ) -> None:
        scale = ScaleDesc(torch.float32, False, group_shape)
        key = FusedRMSQuantKey(
            fused_add=False,
            quant=QuantKey(dtype=quant_dtype, scale=scale, symmetric=symmetric),
        )

        super().__init__(epsilon, key, match_aiter_quant)
...
```
**EN:** Class `AiterRMSNormDynamicQuantPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. AITER RMSNorm + Dynamic Quantization pattern. Key methods include __init__, register.
**CN:** 类 `AiterRMSNormDynamicQuantPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 文档字符串强调：AITER RMSNorm + Dynamic Quantization pattern. 关键方法包括 __init__, register。

### Class `AiterFusedAddRMSNormDynamicQuantPattern` (lines 120-179)
```python
class AiterFusedAddRMSNormDynamicQuantPattern(AiterRMSNormQuantPattern):
    """AITER RMSNorm Fused Add + Dynamic Quantization pattern."""

    FUSED_OP = rocm_aiter_ops.get_rmsnorm_fused_add_dynamic_quant_op()

    def __init__(
        self,
        epsilon: float,
        quant_dtype: torch.dtype,
        match_aiter_quant: bool = True,
        group_shape: GroupShape = GroupShape.PER_TOKEN,
        symmetric: bool = True,
    ) -> None:
        scale = ScaleDesc(torch.float32, False, group_shape)
        key = FusedRMSQuantKey(
            fused_add=True,
            quant=QuantKey(dtype=quant_dtype, scale=scale, symmetric=symmetric),
        )

        super().__init__(epsilon, key, match_aiter_quant)
...
```
**EN:** Class `AiterFusedAddRMSNormDynamicQuantPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. AITER RMSNorm Fused Add + Dynamic Quantization pattern. Key methods include __init__, register.
**CN:** 类 `AiterFusedAddRMSNormDynamicQuantPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 文档字符串强调：AITER RMSNorm Fused Add + Dynamic Quantization pattern. 关键方法包括 __init__, register。

### Class `AiterRMSFp8GroupQuantPattern` (lines 182-235)
```python
class AiterRMSFp8GroupQuantPattern(AiterRMSNormQuantPattern):
    """
    This pattern fuses aiter rms_norm & group fp8 quant custom
    ops into an aiter rms_norm_group_fp8_quant op.
    """

    FUSED_OP = rocm_aiter_ops.get_rmsnorm_group_fused_quant_op()

    def __init__(
        self,
        epsilon: float,
        quant_dtype: torch.dtype,
        group_shape: GroupShape,
        match_aiter_quant: bool = True,
        symmetric: bool = True,
    ) -> None:
        scale = ScaleDesc(torch.float32, False, group_shape)
        key = FusedRMSQuantKey(
            fused_add=False,
            quant=QuantKey(dtype=quant_dtype, scale=scale, symmetric=symmetric),
...
```
**EN:** Class `AiterRMSFp8GroupQuantPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. This pattern fuses aiter rms_norm & group fp8 quant custom ops into an aiter rms_norm_group_fp8_quant op. Key methods include __init__, register.
**CN:** 类 `AiterRMSFp8GroupQuantPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 文档字符串强调：This pattern fuses aiter rms_norm & group fp8 quant custom ops into an aiter rms_norm_group_fp8_quant op. 关键方法包括 __init__, register。

### Class `AiterFusedAddRMSFp8GroupQuantPattern` (lines 238-297)
```python
class AiterFusedAddRMSFp8GroupQuantPattern(AiterRMSNormQuantPattern):
    """
    This pattern fuses aiter rms_norm_with_add & group fp8 quant custom ops
    into a aiter rms_norm_with_add_group_fp8_quant op.
    """

    FUSED_OP = rocm_aiter_ops.get_rmsnorm_group_add_fused_quant_op()

    def __init__(
        self,
        epsilon: float,
        quant_dtype: torch.dtype,
        group_shape: GroupShape,
        match_aiter_quant: bool = True,
        symmetric: bool = True,
    ) -> None:
        scale = ScaleDesc(torch.float32, False, group_shape)
        key = FusedRMSQuantKey(
            fused_add=True,
            quant=QuantKey(dtype=quant_dtype, scale=scale, symmetric=symmetric),
...
```
**EN:** Class `AiterFusedAddRMSFp8GroupQuantPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. This pattern fuses aiter rms_norm_with_add & group fp8 quant custom ops into a aiter rms_norm_with_add_group_fp8_quant op. Key methods include __init__, register.
**CN:** 类 `AiterFusedAddRMSFp8GroupQuantPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 文档字符串强调：This pattern fuses aiter rms_norm_with_add & group fp8 quant custom ops into a aiter rms_norm_with_add_group_fp8_quant op. 关键方法包括 __init__, register。

### Class `DoubleAiterRMSFp8GroupQuantPattern` (lines 300-366)
```python
class DoubleAiterRMSFp8GroupQuantPattern(AiterRMSNormQuantPattern):
    """
    Pattern matching ``rms_norm`` whose output feeds *two* distinct
    ``rocm_aiter_group_fp8_quant`` consumers, replacing it with two
    independent fused ``rms_norm_group_fp8_quant`` ops.

    Repeating the rms_norm in the replacement is preferable to leaving
    the fused 16-bit rms output materialized for two unfused quant
    consumers, and matches what the previous manual graph surgery
    achieved by cloning the rms_norm node.
    """

    FUSED_OP = rocm_aiter_ops.get_rmsnorm_group_fused_quant_op()

    def __init__(
        self,
        epsilon: float,
        quant_dtype: torch.dtype,
        group_shape: GroupShape,
        match_aiter_quant: bool = True,
...
```
**EN:** Class `DoubleAiterRMSFp8GroupQuantPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Pattern matching ``rms_norm`` whose output feeds *two* distinct ``rocm_aiter_group_fp8_quant`` consumers, replacing it with two independent fused ``rms_norm_group_fp8_quant`` ops. Key methods include __init__, register.
**CN:** 类 `DoubleAiterRMSFp8GroupQuantPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 文档字符串强调：Pattern matching ``rms_norm`` whose output feeds *two* distinct ``rocm_aiter_group_fp8_quant`` consumers, replacing it with two independent fused ``rms_norm_group_fp8_quant`` ops. 关键方法包括 __init__, register。

### Class `DoubleAiterRMSFp8GroupQuantViewPattern` (lines 369-452)
```python
class DoubleAiterRMSFp8GroupQuantViewPattern(AiterRMSNormQuantPattern):
    """
    View-tolerant variant of ``DoubleAiterRMSFp8GroupQuantPattern``.

    Matches the same 1-to-2 fan-out, but with a ``view``/``reshape`` between
    the ``rms_norm`` output and the two ``rocm_aiter_group_fp8_quant``
    consumers::

        rms_norm -> view -> rocm_aiter_group_fp8_quant
                \\-> view -> rocm_aiter_group_fp8_quant

    This shape arises in DeepSeek-V3.2's MLA indexer q_c norm, where the
    FP8 linear path's 2D-flatten boilerplate
    (``Fp8BlockScaledMMLinearKernel.apply_weights``) inserts a view between
    the rms_norm output and each FP8 group quant op. The non-view sibling
    pattern silently no-ops on this graph because the pattern matcher
    requires the in-graph and in-pattern node shapes to align.

    The trace_fn runs Inductor's ``view_to_reshape`` post-grad pass to
    normalize ``view`` to ``reshape`` in both the pattern and the input
...
```
**EN:** Class `DoubleAiterRMSFp8GroupQuantViewPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. View-tolerant variant of ``DoubleAiterRMSFp8GroupQuantPattern``. Key methods include __init__, register.
**CN:** 类 `DoubleAiterRMSFp8GroupQuantViewPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 文档字符串强调：View-tolerant variant of ``DoubleAiterRMSFp8GroupQuantPattern``. 关键方法包括 __init__, register。

### Class `AiterRMSNormGatedFp8GroupQuantPattern` (lines 455-543)
```python
class AiterRMSNormGatedFp8GroupQuantPattern(AiterRMSNormQuantPattern):
    """
    Matches decomposed RMSNormGated + reshape + group FP8 quant and replaces
    with rocm_aiter_fused_rms_gated_fp8_group_quant.

    The norm operates per-head on (N*H, D) tensors. The compiler folds the
    reshape chain so after norm the result goes through reshape->merge->quant.
    The pattern reshapes from (N*H, D) to (N, H*D) before calling
    MatcherQuantFP8 so that _quantize_group_native sees the full hidden dim
    and computes the correct num_groups.
    """

    FUSED_OP = rocm_aiter_ops.get_fused_rms_gated_fp8_group_quant_op()

    def __init__(
        self,
        epsilon: float,
        quant_dtype: torch.dtype,
        group_shape: GroupShape,
        num_heads: int,
...
```
**EN:** Class `AiterRMSNormGatedFp8GroupQuantPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Matches decomposed RMSNormGated + reshape + group FP8 quant and replaces with rocm_aiter_fused_rms_gated_fp8_group_quant. Key methods include __init__, register.
**CN:** 类 `AiterRMSNormGatedFp8GroupQuantPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 文档字符串强调：Matches decomposed RMSNormGated + reshape + group FP8 quant and replaces with rocm_aiter_fused_rms_gated_fp8_group_quant. 关键方法包括 __init__, register。

### Class `RocmAiterRMSNormQuantFusionPass` (lines 546-661)
```python
class RocmAiterRMSNormQuantFusionPass(VllmPatternMatcherPass):
    """
    This pass fuses aiter rms_norm & vllm/aiter quant custom ops
    into a fused rms_norm_quant op.
    It also supports fused_add_rms_norm.
    """

    @enable_fake_mode
    def __init__(self, config: VllmConfig) -> None:
        super().__init__(config)

        self.patterns: PatternMatcherPass = PatternMatcherPass(
            pass_name="rocm_aiter_rms_norm_quant_fusion_pass"
        )

        # Discover (num_heads, head_dim) pairs for gated RMSNorm patterns
        # from GatedDeltaNetAttention layers in static_forward_context.
        from vllm.model_executor.layers.mamba.gdn_linear_attn import (
            GatedDeltaNetAttention,
        )
...
```
**EN:** Class `RocmAiterRMSNormQuantFusionPass` defines a transformation pass that walks an FX/IR graph and rewrites matched structures. It operates inside vLLM's post-grad fusion pipeline. This pass fuses aiter rms_norm & vllm/aiter quant custom ops into a fused rms_norm_quant op. Key methods include __init__, __call__, uuid.
**CN:** 类 `RocmAiterRMSNormQuantFusionPass` 定义了一个变换 pass，用于遍历 FX/IR 图并改写匹配到的结构，并服务于vLLM 的后梯度融合流程。 文档字符串强调：This pass fuses aiter rms_norm & vllm/aiter quant custom ops into a fused rms_norm_quant op. 关键方法包括 __init__, __call__, uuid。

### Class `AiterSiluMulFp8GroupQuantPattern` (lines 664-702)
```python
class AiterSiluMulFp8GroupQuantPattern(VllmPatternReplacement):
    """
    This pattern fuses aiter silu_and_mul & group fp8 quant custom
    ops into an aiter silu_and_mul_group_fp8_quant op.
    """

    FUSED_SILU_MUL_QUANT_OP = rocm_aiter_ops.get_act_mul_fused_fp8_group_quant_op()

    def __init__(self) -> None:
        self.silu_and_mul_matcher = MatcherSiluAndMul()
        self.quant_matcher = MatcherQuantFP8(
            quant_key=kFp8Dynamic128Sym, match_rocm_aiter=True
        )

    def get_inputs(self) -> list[torch.Tensor]:
        return [
            self.silu_and_mul_matcher.inputs()[0],
        ]

    @property
...
```
**EN:** Class `AiterSiluMulFp8GroupQuantPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. This pattern fuses aiter silu_and_mul & group fp8 quant custom ops into an aiter silu_and_mul_group_fp8_quant op. Key methods include __init__, get_inputs, pattern, replacement.
**CN:** 类 `AiterSiluMulFp8GroupQuantPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 文档字符串强调：This pattern fuses aiter silu_and_mul & group fp8 quant custom ops into an aiter silu_and_mul_group_fp8_quant op. 关键方法包括 __init__, get_inputs, pattern, replacement。

### Class `RocmAiterSiluMulFp8GroupQuantFusionPass` (lines 705-720)
```python
class RocmAiterSiluMulFp8GroupQuantFusionPass(VllmFusionPatternMatcherPass):
    """
    This pass fuses a pre-defined set of custom ops into fused ops.
    It uses the torch pattern matcher to find the patterns and replace them.

    Because patterns can only be registered once, the pass is a singleton.
    This will be addressed in a future version of PyTorch:
    https://github.com/pytorch/pytorch/pull/139321#issuecomment-2452354980
    """

    def __init__(self, config: VllmConfig) -> None:
        super().__init__(config, "rocm_aiter_silu_mul_fp8_group_quant_fusion_pass")

        self.register(AiterSiluMulFp8GroupQuantPattern())

        self.dump_patterns(config, self.pm_pass)
```
**EN:** Class `RocmAiterSiluMulFp8GroupQuantFusionPass` defines a transformation pass that walks an FX/IR graph and rewrites matched structures. It operates inside vLLM's post-grad fusion pipeline. This pass fuses a pre-defined set of custom ops into fused ops. Key methods include __init__.
**CN:** 类 `RocmAiterSiluMulFp8GroupQuantFusionPass` 定义了一个变换 pass，用于遍历 FX/IR 图并改写匹配到的结构，并服务于vLLM 的后梯度融合流程。 文档字符串强调：This pass fuses a pre-defined set of custom ops into fused ops. 关键方法包括 __init__。

### Class `AddAiterRMSNormPadPattern` (lines 723-796)
```python
class AddAiterRMSNormPadPattern:
    """
    This pattern replaces an aiter_rmsnorm_with_add & a pad op
    with a custom triton_add_rmsnorm_pad op from AITER.
    """

    AITER_TRITON_ADD_RMSNORM_PAD_OP = rocm_aiter_ops.get_triton_add_rmsnorm_pad_op()

    def __init__(
        self,
        epsilon: float,
        hidden_size: int,
        x_pad_to_multiple: int,
    ):
        self.epsilon = epsilon
        self.hidden_size = hidden_size
        self.x_pad_to_multiple = x_pad_to_multiple

    def get_inputs(self) -> list[torch.Tensor]:
        device = torch.device("cuda")
...
```
**EN:** Class `AddAiterRMSNormPadPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. This pattern replaces an aiter_rmsnorm_with_add & a pad op with a custom triton_add_rmsnorm_pad op from AITER. Key methods include __init__, get_inputs, register.
**CN:** 类 `AddAiterRMSNormPadPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 文档字符串强调：This pattern replaces an aiter_rmsnorm_with_add & a pad op with a custom triton_add_rmsnorm_pad op from AITER. 关键方法包括 __init__, get_inputs, register。

### Class `RocmAiterTritonAddRMSNormPadFusionPass` (lines 799-828)
```python
class RocmAiterTritonAddRMSNormPadFusionPass(VllmPatternMatcherPass):
    """
    This pass replaces an AITER CK RMSNorm + residual add and a pad op
    with an triton_add_rmsnorm_pad op from AITER.
    """

    def __init__(self, config: VllmConfig):
        super().__init__(config)
        self.patterns: PatternMatcherPass = PatternMatcherPass(
            pass_name="rocm_aiter_triton_add_rmsnorm_pad_fusion_pass"
        )

        # gpt-oss has hidden size 2880
        # padded to a multiple of 128 on gfx942 and 256 on gfx950 respectively
        hidden_size = 2880
        for epsilon in [1e-5, 1e-6]:
            for x_pad_to_multiple in [128, 256]:
                AddAiterRMSNormPadPattern(
                    epsilon, hidden_size, x_pad_to_multiple
                ).register(self.patterns)
...
```
**EN:** Class `RocmAiterTritonAddRMSNormPadFusionPass` defines a transformation pass that walks an FX/IR graph and rewrites matched structures. It operates inside vLLM's post-grad fusion pipeline. This pass replaces an AITER CK RMSNorm + residual add and a pad op with an triton_add_rmsnorm_pad op from AITER. Key methods include __init__, __call__, uuid.
**CN:** 类 `RocmAiterTritonAddRMSNormPadFusionPass` 定义了一个变换 pass，用于遍历 FX/IR 图并改写匹配到的结构，并服务于vLLM 的后梯度融合流程。 文档字符串强调：This pass replaces an AITER CK RMSNorm + residual add and a pad op with an triton_add_rmsnorm_pad op from AITER. 关键方法包括 __init__, __call__, uuid。

### Class `MLADualRMSNormPattern` (lines 831-912)
```python
class MLADualRMSNormPattern(
    VllmPatternReplacement[..., tuple[torch.Tensor, torch.Tensor, torch.Tensor]]
):
    """
    Fuse paired q_a_layernorm + kv_a_layernorm in MLA attention into
    AITER's ``fused_qk_rmsnorm`` HIP kernel.

    Target FX-graph pattern (unfused, ``vllm_ir`` stage)::

        gemm -> split_with_sizes([q_dim, kv_dim])
            +-- q_c     -> vllm_ir.rms_norm(q_c, q_w, eps)
            +-- kv_lora -> split_with_sizes([kv_c_dim, k_pe_dim])
                            +-- kv_c -> vllm_ir.rms_norm(kv_c, kv_w, eps)
                            +-- k_pe

    The pattern covers the connected subgraph rooted at the first
    ``split_with_sizes`` (which produces ``q_c`` and ``kv_lora``),
    through the two ``rms_norm`` calls, and the ``k_pe`` passthrough.
    """

...
```
**EN:** Class `MLADualRMSNormPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Fuse paired q_a_layernorm + kv_a_layernorm in MLA attention into AITER's ``fused_qk_rmsnorm`` HIP kernel. Key methods include __init__, get_inputs, pattern, replacement.
**CN:** 类 `MLADualRMSNormPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 文档字符串强调：Fuse paired q_a_layernorm + kv_a_layernorm in MLA attention into AITER's ``fused_qk_rmsnorm`` HIP kernel. 关键方法包括 __init__, get_inputs, pattern, replacement。

### Class `MLADualRMSNormFusionPass` (lines 915-926)
```python
class MLADualRMSNormFusionPass(VllmFusionPatternMatcherPass):
    """
    Post-grad PatternMatcher pass that fuses paired q / kv RMS norms in
    MLA attention into ``fused_mla_dual_rms_norm`` backed by aiter's
    ``fused_qk_rmsnorm`` HIP kernel.
    """

    def __init__(self, config: VllmConfig) -> None:
        super().__init__(config, "mla_dual_rms_norm_fusion_pass")

        for epsilon in [1e-5, 1e-6]:
            self.register(MLADualRMSNormPattern(epsilon))
```
**EN:** Class `MLADualRMSNormFusionPass` defines a transformation pass that walks an FX/IR graph and rewrites matched structures. It operates inside vLLM's post-grad fusion pipeline. Post-grad PatternMatcher pass that fuses paired q / kv RMS norms in MLA attention into ``fused_mla_dual_rms_norm`` backed by aiter's ``fused_qk_rmsnorm`` HIP kernel. Key methods include __init__.
**CN:** 类 `MLADualRMSNormFusionPass` 定义了一个变换 pass，用于遍历 FX/IR 图并改写匹配到的结构，并服务于vLLM 的后梯度融合流程。 文档字符串强调：Post-grad PatternMatcher pass that fuses paired q / kv RMS norms in MLA attention into ``fused_mla_dual_rms_norm`` backed by aiter's ``fused_qk_rmsnorm`` HIP kernel. 关键方法包括 __init__。

## Key Concepts / 关键概念
- **Compilation pipeline / 编译流水线**
  - **EN:** The file participates in turning FX graphs into optimized executable code or runtime wrappers.
  - **CN:** 该文件参与将 FX 图转换为优化后的可执行代码或运行时封装。
- **Pattern-based graph fusion / 基于模式的图融合**
  - **EN:** The implementation matches recognizable FX subgraphs and replaces them with fewer, more specialized ops.
  - **CN:** 该实现会匹配可识别的 FX 子图，并将其替换为数量更少、专用性更强的算子。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `import vllm.ir.ops`, `import vllm.model_executor.layers.quantization.utils.fp8_utils`, `from vllm._aiter_ops import rocm_aiter_ops`, `from vllm.config import VllmConfig, get_layers_from_vllm_config`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.quantization.utils.quant_utils import GroupShape, QuantKey, ScaleDesc, kFp8Dynamic128Sym`, `from vllm.platforms import current_platform`, `from ..inductor_pass import enable_fake_mode`, ...
- **PyTorch / Torch 栈**: `import torch`, `import torch._inductor.pattern_matcher as pm`, `from torch import fx`, `from torch._inductor.fx_passes.post_grad import view_to_reshape`, `from torch._inductor.pattern_matcher import PatternMatcherPass`
- **Stdlib / 标准库**: `from collections.abc import Callable`, `from typing import Any`
