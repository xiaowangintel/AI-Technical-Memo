# qk_norm_rope_fusion.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/compilation/passes/fusion/qk_norm_rope_fusion.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Fuse QK normalization together with rotary embedding application. / 将 QK 归一化与 rotary embedding 应用过程进行融合。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 4-22)
```python
from collections.abc import Callable
from typing import ParamSpec

import torch
import torch._inductor.pattern_matcher as pm
from torch import fx
from torch._higher_order_ops.auto_functionalize import auto_functionalized
from torch._inductor.pattern_matcher import PatternMatcherPass

import vllm.ir.ops
from vllm.config import VllmConfig, get_layers_from_vllm_config
from vllm.logger import init_logger
from vllm.model_executor.layers.attention import Attention
from vllm.model_executor.layers.rotary_embedding import RotaryEmbedding

from ..inductor_pass import enable_fake_mode
from ..vllm_inductor_pass import VllmInductorPass, VllmPatternMatcherPass
from .matcher_utils import MatcherRotaryEmbedding
from .rms_quant_fusion import empty_bf16, empty_fp32, empty_i64
```
**EN:** This import block assembles the PyTorch/FX, Inductor, and vLLM modules that the rest of the file relies on for graph rewriting, backend integration, caching, or runtime control in vLLM's post-grad fusion pipeline.
**CN:** 这一组导入语句组织了 PyTorch/FX、Inductor 与 vLLM 模块，供文件后续代码在vLLM 的后梯度融合流程中完成图改写、后端集成、缓存或运行时控制。

### Constants / assignments (lines 24-28)
```python
logger = init_logger(__name__)
FUSED_QK_ROPE_OP = torch.ops._C.fused_qk_norm_rope.default
P = ParamSpec("P")
```
**EN:** These assignments establish shared module state such as loggers, global flags, dtype/op handles, or counters that later logic reuses throughout vLLM's post-grad fusion pipeline.
**CN:** 这些赋值语句建立了共享模块状态，例如日志器、全局开关、dtype/算子句柄或计数器，供后续代码在vLLM 的后梯度融合流程中复用。

### Class `QkNormRopePattern` (lines 31-185)
```python
class QkNormRopePattern:
    """
    Match the unfused sequence in attention blocks and replace with the fused op.

    Unfused (conceptually):
      q, k, v = split(qkv, [qsz, kvsz, kvsz], -1)
      qh = reshape(q, [-1, num_heads, head_dim])
      kh = reshape(k, [-1, num_kv_heads, head_dim])
      qn = rms_norm(qh, q_weight, eps)
      kn = rms_norm(kh, k_weight, eps)
      qf = reshape(qn, [-1, num_heads * head_dim])
      kf = reshape(kn, [-1, num_kv_heads * head_dim])
      qf, kf = rotary_embedding(positions, qf, kf, head_dim, cos_sin_cache, is_neox)
      return qf, kf, v

    Fused replacement:
      fused_qk_norm_rope(qkv, num_heads, num_kv_heads, num_kv_heads, head_dim,
                         eps, q_weight, k_weight, cos_sin_cache, is_neox,
                         positions.view(-1))
      return split(qkv, [qsz, kvsz, kvsz], -1)
...
```
**EN:** Class `QkNormRopePattern` captures a reusable match/rewrite pattern together with its constraints. It operates inside vLLM's post-grad fusion pipeline. Match the unfused sequence in attention blocks and replace with the fused op. Key methods include __init__, get_inputs, wrap_trace_fn, fx_view_to_reshape, register.
**CN:** 类 `QkNormRopePattern` 封装了可复用的匹配/改写模式及其约束条件，并服务于vLLM 的后梯度融合流程。 文档字符串强调：Match the unfused sequence in attention blocks and replace with the fused op. 关键方法包括 __init__, get_inputs, wrap_trace_fn, fx_view_to_reshape, register。

### Class `QKNormRoPEFusionPass` (lines 188-245)
```python
class QKNormRoPEFusionPass(VllmPatternMatcherPass):
    """Fuse Q/K RMSNorm + RoPE into fused_qk_norm_rope when the custom op exists."""

    @enable_fake_mode
    def __init__(self, config: VllmConfig) -> None:
        super().__init__(config)
        self.patterns: PatternMatcherPass = PatternMatcherPass(
            pass_name="qk_norm_rope_fusion_pass"
        )

        dtype = config.model_config.dtype
        if dtype not in (torch.bfloat16, torch.float16):
            logger.warning_once(
                "QK Norm+RoPE fusion not enabled: unsupported dtype %s", dtype
            )
            return

        # use one attn layer to get meta (such as head_dim) for QkNormRopePattern
        attn_layers: dict[str, Attention] = get_layers_from_vllm_config(
            config, Attention
...
```
**EN:** Class `QKNormRoPEFusionPass` defines a transformation pass that walks an FX/IR graph and rewrites matched structures. It operates inside vLLM's post-grad fusion pipeline. Fuse Q/K RMSNorm + RoPE into fused_qk_norm_rope when the custom op exists. Key methods include __init__, __call__, uuid.
**CN:** 类 `QKNormRoPEFusionPass` 定义了一个变换 pass，用于遍历 FX/IR 图并改写匹配到的结构，并服务于vLLM 的后梯度融合流程。 文档字符串强调：Fuse Q/K RMSNorm + RoPE into fused_qk_norm_rope when the custom op exists. 关键方法包括 __init__, __call__, uuid。

## Key Concepts / 关键概念
- **Compilation pipeline / 编译流水线**
  - **EN:** The file participates in turning FX graphs into optimized executable code or runtime wrappers.
  - **CN:** 该文件参与将 FX 图转换为优化后的可执行代码或运行时封装。
- **Pattern-based graph fusion / 基于模式的图融合**
  - **EN:** The implementation matches recognizable FX subgraphs and replaces them with fewer, more specialized ops.
  - **CN:** 该实现会匹配可识别的 FX 子图，并将其替换为数量更少、专用性更强的算子。
- **RoPE fusion / RoPE 融合**
  - **EN:** Rotary position embedding work is fused to reduce graph overhead and improve kernel locality.
  - **CN:** Rotary 位置编码相关工作会被融合，以降低图开销并提升内核局部性。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `import vllm.ir.ops`, `from vllm.config import VllmConfig, get_layers_from_vllm_config`, `from vllm.logger import init_logger`, `from vllm.model_executor.layers.attention import Attention`, `from vllm.model_executor.layers.rotary_embedding import RotaryEmbedding`, `from ..inductor_pass import enable_fake_mode`, `from ..vllm_inductor_pass import VllmInductorPass, VllmPatternMatcherPass`, `from .matcher_utils import MatcherRotaryEmbedding`, ...
- **PyTorch / Torch 栈**: `import torch`, `import torch._inductor.pattern_matcher as pm`, `from torch import fx`, `from torch._higher_order_ops.auto_functionalize import auto_functionalized`, `from torch._inductor.pattern_matcher import PatternMatcherPass`
- **Stdlib / 标准库**: `from collections.abc import Callable`, `from typing import ParamSpec`
