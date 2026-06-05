# attn_quant_fusion.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/compilation/passes/fusion/attn_quant_fusion.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Fusion patterns for attention outputs followed by quantization. / 将注意力输出与后续量化融合的模式。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 5-24)
```python
from collections.abc import Callable

import torch
from torch._higher_order_ops.auto_functionalize import auto_functionalized

from vllm.config import VllmConfig, get_layers_from_vllm_config
from vllm.logger import init_logger
from vllm.model_executor.layers.attention import Attention
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
    kNvfp4Dynamic,
    kStaticTensorScale,
)
from vllm.platforms import current_platform
from vllm.utils.math_utils import round_up
from vllm.utils.torch_utils import _USE_LAYERNAME, _encode_layer_name

from ..vllm_inductor_pass import VllmFusionPatternMatcherPass, VllmPatternReplacement
from .matcher_utils import MatcherQuantFP8
from .rms_quant_fusion import QUANT_OPS
```
**EN:** This import block assembles the PyTorch/FX, Inductor, and vLLM modules that the rest of the file relies on for graph rewriting, backend integration, caching, or runtime control in vLLM's post-grad fusion pipeline.
**CN:** 这一组导入语句组织了 PyTorch/FX、Inductor 与 vLLM 模块，供文件后续代码在vLLM 的后梯度融合流程中完成图改写、后端集成、缓存或运行时控制。

### Constants / assignments (lines 26-35)
```python
logger = init_logger(__name__)
FP8_DTYPE = current_platform.fp8_dtype()
FP4_DTYPE = torch.uint8
ATTN_OP = torch.ops.vllm.unified_attention_with_output.default
RESHAPE_OP = torch.ops.aten.reshape.default
_FP8_QUANT_KEY = QuantKey(dtype=FP8_DTYPE, scale=kStaticTensorScale, symmetric=True)
```
**EN:** These assignments establish shared module state such as loggers, global flags, dtype/op handles, or counters that later logic reuses throughout vLLM's post-grad fusion pipeline.
**CN:** 这些赋值语句建立了共享模块状态，例如日志器、全局开关、dtype/算子句柄或计数器，供后续代码在vLLM 的后梯度融合流程中复用。

### Class `AttnFp8StaticQuantPattern` (lines 38-169)
```python
class AttnFp8StaticQuantPattern(VllmPatternReplacement[..., torch.Tensor]):
    """
    Fusion for Attention+Fp8StaticQuant.

    Only triggers when the attention implementation returns True in
    `fused_output_quant_supported()`. If the pattern is found, the
    Fp8StaticQuant op will be removed from the graph, and its scale
    will be passed into Attention op as the `output_scale` argument.
    """

    def __init__(self, layer: Attention, dtype: torch.dtype):
        self._layer_name = layer.layer_name
        self._num_heads = layer.num_heads
        self._head_size = layer.head_size
        self._dtype = dtype
        self._quant_matcher = MatcherQuantFP8(_FP8_QUANT_KEY)

    @property
    def pattern(self) -> Callable[..., torch.Tensor]:
        # When _USE_LAYERNAME is enabled (torch >= 2.11), layer_name is
...
```
**EN:** Class `AttnFp8StaticQuantPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Fusion for Attention+Fp8StaticQuant. Key methods include __init__, pattern, replacement, get_inputs.
**CN:** 类 `AttnFp8StaticQuantPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 文档字符串强调：Fusion for Attention+Fp8StaticQuant. 关键方法包括 __init__, pattern, replacement, get_inputs。

### Class `AttnNvfp4QuantPattern` (lines 172-359)
```python
class AttnNvfp4QuantPattern(
    VllmPatternReplacement[..., tuple[torch.Tensor, torch.Tensor]]
):
    """
    Fusion for Attention+Nvfp4Quant.

    Only triggers when the attention implementation returns True in
    `fused_output_quant_supported()`. If the pattern is found, the
    Nvfp4Quant op will be removed from the graph, and its scale
    will be passed into Attention op as the `output_scale` argument.
    """

    def __init__(self, layer: Attention, dtype: torch.dtype):
        self._layer_name = layer.layer_name
        self._num_heads = layer.num_heads
        self._head_size = layer.head_size
        self._dtype = dtype
        self._QUANT_OP = QUANT_OPS[kNvfp4Dynamic]

    @property
...
```
**EN:** Class `AttnNvfp4QuantPattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Fusion for Attention+Nvfp4Quant. Key methods include __init__, pattern, replacement, get_inputs.
**CN:** 类 `AttnNvfp4QuantPattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 文档字符串强调：Fusion for Attention+Nvfp4Quant. 关键方法包括 __init__, pattern, replacement, get_inputs。

### Class `AttnQuantFusionPass` (lines 362-403)
```python
class AttnQuantFusionPass(VllmFusionPatternMatcherPass):
    """
    This pass fuses post-attention quantization onto attention if supported.

    It uses the pattern matcher and matches each layer manually, as strings
    cannot be wildcarded. This also lets us check support on attention layers
    upon registration instead of during pattern matching.

    Currently, only static fp8 quant is supported, but patterns could easily be
    added for other quant schemes and dtypes. The bigger hurdle for wider
    support are attention kernels, which need to support fusing output quant.
    """

    def __init__(self, config: VllmConfig) -> None:
        super().__init__(config, "attn_quant_fusion")

        dtype = config.model_config.dtype
        layers = list(get_layers_from_vllm_config(config, Attention).values())

        if len(layers) == 0:
...
```
**EN:** Class `AttnQuantFusionPass` defines a transformation pass that walks an FX/IR graph and rewrites matched structures. It operates inside vLLM's post-grad fusion pipeline. This pass fuses post-attention quantization onto attention if supported. Key methods include __init__.
**CN:** 类 `AttnQuantFusionPass` 定义了一个变换 pass，用于遍历 FX/IR 图并改写匹配到的结构，并服务于vLLM 的后梯度融合流程。 文档字符串强调：This pass fuses post-attention quantization onto attention if supported. 关键方法包括 __init__。

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
- **vLLM internal / vLLM 内部依赖**: `from vllm.config import VllmConfig, get_layers_from_vllm_config`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.attention import Attention`, `from vllm.model_executor.layers.quantization.utils.quant_utils import QuantKey, kNvfp4Dynamic, kStaticTensorScale`, `from vllm.platforms import current_platform`, `from vllm.utils.math_utils import round_up`, `from vllm.utils.torch_utils import _USE_LAYERNAME, _encode_layer_name`, `from ..vllm_inductor_pass import VllmFusionPatternMatcherPass, VllmPatternReplacement`, ...
- **PyTorch / Torch 栈**: `import torch`, `from torch._higher_order_ops.auto_functionalize import auto_functionalized`
- **Stdlib / 标准库**: `from collections.abc import Callable`
