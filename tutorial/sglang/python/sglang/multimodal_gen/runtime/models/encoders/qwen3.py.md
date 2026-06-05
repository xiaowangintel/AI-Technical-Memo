# qwen3.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/encoders/qwen3.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for qwen3 within the multimodal runtime. Key symbols include `Qwen3MLP`, `Qwen3Attention`, `Qwen3DecoderLayer`. / 该模块实现多模态运行时中与 qwen3 相关的模型构件。 关键符号包括 `Qwen3MLP`, `Qwen3Attention`, `Qwen3DecoderLayer`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-27: Imports and module setup / 导入与模块初始化
```python
from collections.abc import Iterable
from typing import Any

import torch
from torch import nn

from sglang.multimodal_gen.configs.models.encoders import BaseEncoderOutput
from sglang.multimodal_gen.configs.models.encoders.qwen3 import Qwen3TextConfig
from sglang.multimodal_gen.runtime.distributed import get_tp_world_size
from sglang.multimodal_gen.runtime.layers.activation import SiluAndMul
from sglang.multimodal_gen.runtime.layers.attention import LocalAttention
from sglang.multimodal_gen.runtime.layers.layernorm import RMSNorm
from sglang.multimodal_gen.runtime.layers.linear import (
    MergedColumnParallelLinear,
# ...
    default_weight_loader,
    maybe_remap_kv_scale_name,
)
from sglang.multimodal_gen.runtime.models.encoders.base import TextEncoder
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 30-67: Class `Qwen3MLP` / 类 `Qwen3MLP`
```python
class Qwen3MLP(nn.Module):
    """Qwen3 MLP with SwiGLU activation and tensor parallelism."""

    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        quant_config: QuantizationConfig | None = None,
        bias: bool = False,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.gate_up_proj = MergedColumnParallelLinear(
# ...
        x, _ = self.gate_up_proj(x)
        x = self.act_fn(x)
        x, _ = self.down_proj(x)
        return x
```
**EN:** This class models `Qwen3MLP` as a specialization of `nn.Module`. Qwen3 MLP with SwiGLU activation and tensor parallelism. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `Qwen3MLP`，并继承/扩展 `nn.Module`。 文档字符串指出：Qwen3 MLP with SwiGLU activation and tensor parallelism. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 70-242: Class `Qwen3Attention` / 类 `Qwen3Attention`
```python
class Qwen3Attention(nn.Module):
    """Qwen3 attention with QK-Norm and tensor parallelism.

    Key difference from LLaMA: RMSNorm is applied to Q and K before attention.
    """

    def __init__(
        self,
        config: Qwen3TextConfig,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        rope_theta: float = 1000000.0,
        rope_scaling: dict[str, Any] | None = None,
# ...
            ).transpose(1, 2)
            outputs.append(torch.cat([real_output, pad_output], dim=1))

        return torch.cat(outputs, dim=0)
```
**EN:** This class models `Qwen3Attention` as a specialization of `nn.Module`. Qwen3 attention with QK-Norm and tensor parallelism. Important methods include `__init__`, `forward`, `_masked_causal_attention`.
**CN:** 该类实现 `Qwen3Attention`，并继承/扩展 `nn.Module`。 文档字符串指出：Qwen3 attention with QK-Norm and tensor parallelism. 其中较重要的方法包括 `__init__`, `forward`, `_masked_causal_attention`。

### Lines 245-311: Class `Qwen3DecoderLayer` / 类 `Qwen3DecoderLayer`
```python
class Qwen3DecoderLayer(nn.Module):
    """Qwen3 transformer decoder layer."""

    def __init__(
        self,
        config: Qwen3TextConfig,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        rope_theta = config.rope_parameters["rope_theta"]
        rope_scaling = config.rope_parameters
        max_position_embeddings = getattr(config, "max_position_embeddings", 40960)
# ...
        # MLP
        hidden_states, residual = self.post_attention_layernorm(hidden_states, residual)
        hidden_states = self.mlp(hidden_states)
        return hidden_states, residual
```
**EN:** This class models `Qwen3DecoderLayer` as a specialization of `nn.Module`. Qwen3 transformer decoder layer. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `Qwen3DecoderLayer`，并继承/扩展 `nn.Module`。 文档字符串指出：Qwen3 transformer decoder layer. 其中较重要的方法包括 `__init__`, `forward`。

### Lines 314-482: Class `Qwen3ForCausalLM` / 类 `Qwen3ForCausalLM`
```python
class Qwen3ForCausalLM(TextEncoder):
    """Qwen3 causal language model for text encoding in diffusion models.

    Features:
    - Tensor parallelism support
    - FlashAttention/SageAttn/SDPA support via LocalAttention
    - QK-Norm for better training stability
    - FSDP sharding for CPU offload
    """

    def __init__(self, config: Qwen3TextConfig) -> None:
        super().__init__(config)

        self.config = config
# ...

            loaded_params.add(name)

        return loaded_params
```
**EN:** This class models `Qwen3ForCausalLM` as a specialization of `TextEncoder`. Qwen3 causal language model for text encoding in diffusion models. Important methods include `__init__`, `get_input_embeddings`, `forward`, `load_weights`.
**CN:** 该类实现 `Qwen3ForCausalLM`，并继承/扩展 `TextEncoder`。 文档字符串指出：Qwen3 causal language model for text encoding in diffusion models. 其中较重要的方法包括 `__init__`, `get_input_embeddings`, `forward`, `load_weights`。

### Lines 483-485: Top-level configuration / 顶层配置
```python


EntryClass = Qwen3ForCausalLM
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Quantization workflow / 量化工作流
- Model/component loading / 模型/组件加载
- Caching strategy / 缓存策略

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.encoders`, `sglang.multimodal_gen.configs.models.encoders.qwen3`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.layers.activation`, `sglang.multimodal_gen.runtime.layers.attention`, `sglang.multimodal_gen.runtime.layers.layernorm`, `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.layers.quantization`
- **External / 外部**: `torch`
- **Stdlib / 标准库**: `collections.abc`, `typing`
