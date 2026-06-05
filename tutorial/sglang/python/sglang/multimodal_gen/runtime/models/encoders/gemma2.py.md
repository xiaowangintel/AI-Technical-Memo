# gemma2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/models/encoders/gemma2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements model-side building blocks for gemma2 within the multimodal runtime. Key symbols include `Gemma2RMSNorm`, `Gemma2MLP`, `Gemma2Attention`. / 该模块实现多模态运行时中与 gemma2 相关的模型构件。 关键符号包括 `Gemma2RMSNorm`, `Gemma2MLP`, `Gemma2Attention`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-42: Imports and module setup / 导入与模块初始化
```python
# SPDX-License-Identifier: Apache-2.0
#
# Gemma2 2B text encoder for SANA.
#
# This is a decoder-only language model used as a text encoder: we feed
# in tokenized text and extract the final hidden states (not logits) as
# the conditioning signal for SANA's cross-attention layers.
#
# Architecture follows google/gemma-2-2b-it:
#   - 26 layers, alternating global / sliding-window attention
#   - GQA with 8 query heads, 4 KV heads, head_dim=256
#   - Pre/post attention + pre/post feedforward LayerNorm (Gemma2-style)
#   - GeGLU activation (gelu_pytorch_tanh)
#
# ...
    LayerwiseOffloadableModuleMixin,
)

logger = logging.getLogger(__name__)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 45-57: Class `Gemma2RMSNorm` / 类 `Gemma2RMSNorm`
```python
class Gemma2RMSNorm(nn.Module):
    def __init__(self, dim: int, eps: float = 1e-6):
        super().__init__()
        self.eps = eps
        self.weight = nn.Parameter(torch.zeros(dim))

    def _norm(self, x):
        return x * torch.rsqrt(x.pow(2).mean(-1, keepdim=True) + self.eps)

    def forward(self, x):
        output = self._norm(x.float())
        output = output * (1.0 + self.weight.float())
        return output.type_as(x)
```
**EN:** This class models `Gemma2RMSNorm` as a specialization of `nn.Module`. Important methods include `__init__`, `_norm`, `forward`.
**CN:** 该类实现 `Gemma2RMSNorm`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `_norm`, `forward`。

### Lines 60-95: Class `Gemma2MLP` / 类 `Gemma2MLP`
```python
class Gemma2MLP(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.gate_up_proj = MergedColumnParallelLinear(
            input_size=hidden_size,
            output_sizes=[intermediate_size] * 2,
            bias=False,
# ...
        x, _ = self.gate_up_proj(x)
        x = self.act_fn(x)
        x, _ = self.down_proj(x)
        return x
```
**EN:** This class models `Gemma2MLP` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `Gemma2MLP`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 98-224: Class `Gemma2Attention` / 类 `Gemma2Attention`
```python
class Gemma2Attention(nn.Module):
    def __init__(
        self,
        layer_id: int,
        config: Gemma2Config,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.layer_id = layer_id
        self.hidden_size = hidden_size
# ...
        )

        output, _ = self.o_proj(attn_output)
        return output
```
**EN:** This class models `Gemma2Attention` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `Gemma2Attention`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 227-283: Class `Gemma2DecoderLayer` / 类 `Gemma2DecoderLayer`
```python
class Gemma2DecoderLayer(nn.Module):
    def __init__(
        self,
        layer_id: int,
        config: Gemma2Config,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        arch = config.arch_config
        self.hidden_size = arch.hidden_size
        self.self_attn = Gemma2Attention(
            layer_id=layer_id,
            config=config,
# ...
        hidden_states = self.post_feedforward_layernorm(hidden_states)
        hidden_states = residual + hidden_states

        return hidden_states
```
**EN:** This class models `Gemma2DecoderLayer` as a specialization of `nn.Module`. Important methods include `__init__`, `forward`.
**CN:** 该类实现 `Gemma2DecoderLayer`，并继承/扩展 `nn.Module`。 其中较重要的方法包括 `__init__`, `forward`。

### Lines 286-444: Class `Gemma2Model` / 类 `Gemma2Model`
```python
class Gemma2Model(nn.Module, LayerwiseOffloadableModuleMixin):
    """Gemma2 text encoder model for SANA pipeline."""

    _fsdp_shard_conditions = []
    layerwise_offload_dit_group_enabled = False
    layer_names = ["layers"]

    def __init__(self, config: Gemma2Config, **kwargs):
        super().__init__()
        self.config = config
        arch = config.arch_config
        self.quant_config = None

        self.vocab_size = arch.vocab_size
# ...

        raise TypeError(
            f"Unsupported shard_id={shard_id!r} for weight_loader={weight_loader}"
        )
```
**EN:** This class models `Gemma2Model` as a specialization of `nn.Module`, `LayerwiseOffloadableModuleMixin`. Gemma2 text encoder model for SANA pipeline. Important methods include `__init__`, `get_input_embeddings`, `forward`, `load_weights`.
**CN:** 该类实现 `Gemma2Model`，并继承/扩展 `nn.Module`, `LayerwiseOffloadableModuleMixin`。 文档字符串指出：Gemma2 text encoder model for SANA pipeline. 其中较重要的方法包括 `__init__`, `get_input_embeddings`, `forward`, `load_weights`。

### Lines 445-447: Top-level configuration / 顶层配置
```python


EntryClass = Gemma2Model
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Quantization workflow / 量化工作流
- Model/component loading / 模型/组件加载
- Pipeline orchestration / 流水线编排

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.models.encoders.base`, `sglang.multimodal_gen.configs.models.encoders.gemma2`, `sglang.multimodal_gen.runtime.distributed`, `sglang.multimodal_gen.runtime.layers.activation`, `sglang.multimodal_gen.runtime.layers.linear`, `sglang.multimodal_gen.runtime.layers.quantization`, `sglang.multimodal_gen.runtime.layers.rotary_embedding`, `sglang.multimodal_gen.runtime.layers.vocab_parallel_embedding`
- **External / 外部**: `torch`
- **Stdlib / 标准库**: `logging`, `typing`
