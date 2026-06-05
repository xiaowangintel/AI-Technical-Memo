# matcher_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/compilation/passes/fusion/matcher_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Reusable matcher helpers for rotary, RMSNorm-gated, quantization, and activation patterns. / 可复用的匹配器辅助类，用于 rotary、RMSNorm-gated、量化与激活模式。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 3-30)
```python
from abc import ABC, abstractmethod
from typing import Any

import torch
from torch._higher_order_ops import auto_functionalized
from torch._ops import OpOverload

from vllm._aiter_ops import rocm_aiter_ops
from vllm.config import get_current_vllm_config
from vllm.model_executor.layers.activation import SiluAndMul
from vllm.model_executor.layers.layernorm import RMSNormGated
from vllm.model_executor.layers.quantization.input_quant_fp8 import QuantFP8
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    GroupShape,
    QuantKey,
    _normalize_quant_group_shape,
    kFp8Dynamic64Sym,
    kFp8Dynamic128Sym,
    kFp8DynamicTensorSym,
    kFp8DynamicTokenSym,
    kFp8StaticTensorSym,
    kNvfp4Dynamic,
)
from vllm.model_executor.layers.rotary_embedding import RotaryEmbedding
from vllm.model_executor.layers.rotary_embedding.deepseek_scaling_rope import (
    DeepseekScalingRotaryEmbedding,
)
from vllm.platforms import current_platform
```
**EN:** This import block assembles the PyTorch/FX, Inductor, and vLLM modules that the rest of the file relies on for graph rewriting, backend integration, caching, or runtime control in vLLM's post-grad fusion pipeline.
**CN:** 这一组导入语句组织了 PyTorch/FX、Inductor 与 vLLM 模块，供文件后续代码在vLLM 的后梯度融合流程中完成图改写、后端集成、缓存或运行时控制。

### Constants / assignments (lines 32-48)
```python
ROTARY_OP = torch.ops._C.rotary_embedding.default
FLASHINFER_ROTARY_OP = torch.ops.vllm.flashinfer_rotary_embedding.default
QUANT_OPS: dict[QuantKey, OpOverload] = {
    kFp8StaticTensorSym: torch.ops._C.static_scaled_fp8_quant.default,  # noqa: E501
    kFp8DynamicTensorSym: torch.ops._C.dynamic_scaled_fp8_quant.default,  # noqa: E501
    kFp8DynamicTokenSym: torch.ops._C.dynamic_per_token_scaled_fp8_quant.default,  # noqa: E501
}
SILU_MUL_OP = torch.ops._C.silu_and_mul.default
```
**EN:** These module-level assignments declare constants or lightweight configuration objects that shape later control flow inside vLLM's post-grad fusion pipeline.
**CN:** 这些模块级赋值语句声明了常量或轻量配置对象，用于影响vLLM 的后梯度融合流程中的后续控制流程。

### Class `MatcherCustomOp` (lines 51-82)
```python
class MatcherCustomOp(ABC):
    def __init__(self, enabled: bool) -> None:
        config = get_current_vllm_config()
        self.model_dtype = config.model_config.dtype if config.model_config else None
        self.device = config.device_config.device if config.device_config else None

        self.enabled = enabled
        self.forward = self.forward_custom if enabled else self.forward_native

    @abstractmethod
    def forward_custom(self, *args: Any, **kwargs: Any) -> Any:
        pass

    @abstractmethod
    def forward_native(self, *args: Any, **kwargs: Any) -> Any:
        pass

    def __call__(self, *args: Any, **kwargs: Any) -> Any:
        return self.forward(*args, **kwargs)

...
```
**EN:** Class `MatcherCustomOp` encapsulates reusable matcher helpers that make fusion patterns easier to express. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__, forward_custom, forward_native, __call__, empty.
**CN:** 类 `MatcherCustomOp` 封装了可复用的匹配辅助逻辑，使融合模式更易表达，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__, forward_custom, forward_native, __call__, empty。

### Class `MatcherRotaryEmbedding` (lines 85-161)
```python
class MatcherRotaryEmbedding(MatcherCustomOp):
    def __init__(
        self,
        is_neox: bool,
        head_size: int,
        num_heads: int,
        num_kv_heads: int,
        use_flashinfer: bool = False,
        match_rocm_aiter: bool | None = None,
        enabled: bool | None = None,
    ) -> None:
        if enabled is None:
            enabled = RotaryEmbedding.enabled()
        if match_rocm_aiter is None:
            match_rocm_aiter = rocm_aiter_ops.is_triton_rotary_embed_enabled()

        super().__init__(enabled)
        self.is_neox = is_neox
        self.head_size = head_size
        self.num_heads = num_heads
...
```
**EN:** Class `MatcherRotaryEmbedding` encapsulates reusable matcher helpers that make fusion patterns easier to express. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__, inputs, forward_custom, forward_native.
**CN:** 类 `MatcherRotaryEmbedding` 封装了可复用的匹配辅助逻辑，使融合模式更易表达，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__, inputs, forward_custom, forward_native。

### Class `MatcherRMSNormGated` (lines 164-222)
```python
class MatcherRMSNormGated(MatcherCustomOp):
    """Matches RMSNormGated with norm_before_gate=True and group_size=None."""

    def __init__(
        self,
        epsilon: float,
        enabled: bool | None = None,
        norm_before_gate: bool = True,
        group_size: int | None = None,
    ) -> None:
        if enabled is None:
            enabled = RMSNormGated.enabled()

        super().__init__(enabled)
        self.epsilon = epsilon
        self.norm_before_gate = norm_before_gate
        self.group_size = group_size

    def inputs(self) -> list[torch.Tensor]:
        x = self.empty(5, 16)
...
```
**EN:** Class `MatcherRMSNormGated` encapsulates reusable matcher helpers that make fusion patterns easier to express. It operates inside vLLM's post-grad fusion pipeline. Matches RMSNormGated with norm_before_gate=True and group_size=None. Key methods include __init__, inputs, forward_custom, forward_native.
**CN:** 类 `MatcherRMSNormGated` 封装了可复用的匹配辅助逻辑，使融合模式更易表达，并服务于vLLM 的后梯度融合流程。 文档字符串强调：Matches RMSNormGated with norm_before_gate=True and group_size=None. 关键方法包括 __init__, inputs, forward_custom, forward_native。

### Class `MatcherDeepseekScalingRotaryEmbedding` (lines 225-303)
```python
class MatcherDeepseekScalingRotaryEmbedding(MatcherCustomOp):
    def __init__(
        self,
        is_neox: bool,
        head_size: int,
        num_heads: int,
        num_kv_heads: int,
        use_flashinfer: bool = False,
        enabled: bool | None = None,
    ) -> None:
        if enabled is None:
            enabled = DeepseekScalingRotaryEmbedding.enabled()

        super().__init__(enabled)
        self.is_neox = is_neox
        self.head_size = head_size
        self.num_heads = num_heads
        self.num_kv_heads = num_kv_heads
        self.q_size = self.num_heads * self.head_size
        self.kv_size = self.num_kv_heads * self.head_size
...
```
**EN:** Class `MatcherDeepseekScalingRotaryEmbedding` encapsulates reusable matcher helpers that make fusion patterns easier to express. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__, inputs, forward_custom, forward_native.
**CN:** 类 `MatcherDeepseekScalingRotaryEmbedding` 封装了可复用的匹配辅助逻辑，使融合模式更易表达，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__, inputs, forward_custom, forward_native。

### Class `MatcherQuantFP8` (lines 306-454)
```python
class MatcherQuantFP8(MatcherCustomOp):
    def __init__(
        self,
        quant_key: QuantKey,
        enabled: bool | None = None,
        has_col_major_scales: bool = False,
        is_e8m0: bool = False,
        match_rocm_aiter: bool = False,
        is_tma_aligned: bool = False,
    ) -> None:
        if enabled is None:
            enabled = QuantFP8.enabled()

        super().__init__(enabled)
        self.quant_key = quant_key
        self.has_col_major_scales = has_col_major_scales
        self.is_e8m0 = is_e8m0
        self.match_rocm_aiter = match_rocm_aiter
        self.is_tma_aligned = is_tma_aligned

...
```
**EN:** Class `MatcherQuantFP8` encapsulates reusable matcher helpers that make fusion patterns easier to express. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__, forward_rocm_aiter, forward_custom, forward_native, make_scale.
**CN:** 类 `MatcherQuantFP8` 封装了可复用的匹配辅助逻辑，使融合模式更易表达，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__, forward_rocm_aiter, forward_custom, forward_native, make_scale。

### Class `MatcherSiluAndMul` (lines 457-481)
```python
class MatcherSiluAndMul(MatcherCustomOp):
    def __init__(self, enabled: bool | None = None) -> None:
        if enabled is None:
            enabled = SiluAndMul.enabled()
        super().__init__(enabled)

    def inputs(self) -> list[torch.Tensor]:
        input = self.empty(5, 4)
        return [input]

    def forward_custom(
        self,
        x: torch.Tensor,
    ) -> torch.Tensor:
        d = x.shape[-1] // 2
        output_shape = x.shape[:-1] + (d,)
        out = torch.empty(output_shape, dtype=x.dtype, device=x.device)
        result = auto_functionalized(SILU_MUL_OP, result=out, input=x)
        return result[1]

...
```
**EN:** Class `MatcherSiluAndMul` encapsulates reusable matcher helpers that make fusion patterns easier to express. It operates inside vLLM's post-grad fusion pipeline. Key methods include __init__, inputs, forward_custom, forward_native.
**CN:** 类 `MatcherSiluAndMul` 封装了可复用的匹配辅助逻辑，使融合模式更易表达，并服务于vLLM 的后梯度融合流程。 关键方法包括 __init__, inputs, forward_custom, forward_native。

## Key Concepts / 关键概念
- **Compilation pipeline / 编译流水线**
  - **EN:** The file participates in turning FX graphs into optimized executable code or runtime wrappers.
  - **CN:** 该文件参与将 FX 图转换为优化后的可执行代码或运行时封装。
- **Pattern-based graph fusion / 基于模式的图融合**
  - **EN:** The implementation matches recognizable FX subgraphs and replaces them with fewer, more specialized ops.
  - **CN:** 该实现会匹配可识别的 FX 子图，并将其替换为数量更少、专用性更强的算子。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `from vllm._aiter_ops import rocm_aiter_ops`, `from vllm.config import get_current_vllm_config`, `from vllm.model_executor.layers.activation import SiluAndMul`, `from vllm.model_executor.layers.layernorm import RMSNormGated`, `from vllm.model_executor.layers.quantization.input_quant_fp8 import QuantFP8`, `from vllm.model_executor.layers.quantization.utils.quant_utils import GroupShape, QuantKey, _normalize_quant_group_shape, kFp8Dynamic64Sym, kFp8Dynamic128Sym, kFp8DynamicTensorSym, kFp8DynamicTokenSym, kFp8StaticTensorSym, kNvfp4Dynamic`, `from vllm.model_executor.layers.rotary_embedding import RotaryEmbedding`, `from vllm.model_executor.layers.rotary_embedding.deepseek_scaling_rope import DeepseekScalingRotaryEmbedding`, ...
- **PyTorch / Torch 栈**: `import torch`, `from torch._higher_order_ops import auto_functionalized`, `from torch._ops import OpOverload`
- **Stdlib / 标准库**: `from abc import ABC, abstractmethod`, `from typing import Any`
