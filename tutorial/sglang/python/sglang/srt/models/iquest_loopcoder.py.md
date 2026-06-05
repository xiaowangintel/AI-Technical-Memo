# iquest_loopcoder.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/iquest_loopcoder.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Iquest Loopcoder architecture into SGLang's serving runtime. Inference-only LoopCoder model compatible with HuggingFace weights. / 该模块将 Iquest Loopcoder 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：Inference-only LoopCoder model compatible with HuggingFace weights。

## Line-by-Line Analysis / 逐行分析

### Lines 1-13: Module header and imports / 模块头与导入
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 44-44: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 47-116: Class: LoopGateProjection / 类：LoopGateProjection
```python
class LoopGateProjection(nn.Module):
    """Gate projection for mixed attention in Loop 2+.

    Computes: g = sigmoid(linear(Q)) for each head independently.
    This gate determines how much to use Loop1's KV (global) vs current loop's KV (local).

    Supports tensor parallelism: each GPU handles a subset of heads.
    The weight matrix has shape [num_heads, head_dim] and is split along the head dimension.
    """

    def __init__(
        self,
        total_num_heads: int,
        head_dim: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.total_num_heads = total_num_heads
        self.head_dim = head_dim
        tp_size = get_tensor_model_parallel_world_size()
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size

        self.gate_proj = ColumnParallelLinear(
            head_dim,
            self.total_num_heads,
            bias=True,
            gather_output=False,
            quant_config=quant_config,
            prefix=add_prefix("gate_proj", prefix),
        )

    def forward(self, query: torch.Tensor) -> torch.Tensor:
        """Compute gate values from query tensor.

        Args:
            query: [num_heads, num_tokens, head_dim]
                where num_heads is the number of heads on this TP rank
                and num_tokens = batch * seq_len
# ... truncated for brevity ...
```
**EN:** This class defines Loop Gate Projection inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: Gate projection for mixed attention in Loop 2+..
**CN:** 该类定义了 Loop Gate Projection，用于封装该模型组件的状态与方法。 文档字符串摘要：Gate projection for mixed attention in Loop 2+.。

### Lines 119-249: Class: LoopCoderAttention / 类：LoopCoderAttention
```python
class LoopCoderAttention(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        layer_id: int = 0,
        max_position: int = 4096 * 32,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.layer_id = layer_id
        self.hidden_size = hidden_size
        tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = num_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = num_kv_heads
        if self.total_num_kv_heads >= tp_size:
            assert self.total_num_kv_heads % tp_size == 0
        else:
            assert tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)
        self.head_dim = hidden_size // self.total_num_heads
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scaling = self.head_dim**-0.5

        # Get loop_num from config, default to 2 if not specified
        self.loop_num = getattr(config, "loop_num", 2)
        self.loop_window_size = getattr(config, "loop_window_size", 64)

        self.qkv_proj = QKVParallelLinear(
            hidden_size,
            self.head_dim,
            self.total_num_heads,
            self.total_num_kv_heads,
            bias=False,
# ... truncated for brevity ...
```
**EN:** This class defines Loop Coder Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Loop Coder Attention，用于封装该模型组件的状态与方法。

### Lines 252-312: Class: LoopCoderDecoderLayer / 类：LoopCoderDecoderLayer
```python
class LoopCoderDecoderLayer(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        layer_id: int = 0,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        self.layer_id = layer_id

        self.self_attn = LoopCoderAttention(
            config=config,
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            num_kv_heads=config.num_key_value_heads,
            layer_id=layer_id,
            max_position=getattr(config, "max_position_embeddings", 4096 * 32),
            quant_config=quant_config,
            prefix=add_prefix("self_attn", prefix),
        )
        self.mlp = LoopCoderMLP(
            hidden_size=self.hidden_size,
            intermediate_size=config.intermediate_size,
            hidden_act=config.hidden_act,
            quant_config=quant_config,
            prefix=add_prefix("mlp", prefix),
        )
        self.input_layernorm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.post_attention_layernorm = RMSNorm(
            config.hidden_size, eps=config.rms_norm_eps
        )

    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
        loop_idx: int,
# ... truncated for brevity ...
```
**EN:** This class defines Loop Coder Decoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Loop Coder Decoder Layer，用于封装该模型组件的状态与方法。

### Lines 315-396: Class: IQuestLoopCoderModel / 类：IQuestLoopCoderModel
```python
class IQuestLoopCoderModel(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.quant_config = quant_config
        self.vocab_size = config.vocab_size

        self.embed_tokens = VocabParallelEmbedding(
            config.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=add_prefix("embed_tokens", prefix),
        )

        self.loop_num = getattr(self.config, "loop_num", 2)
        self.window_size = getattr(self.config, "loop_window_size", 64)

        # Gate projections for Loop 2+ (one per layer)
        head_dim = config.hidden_size // config.num_attention_heads
        gate_projections = make_layers(
            config.num_hidden_layers,
            lambda idx, prefix: LoopGateProjection(
                total_num_heads=config.num_attention_heads,
                head_dim=head_dim,
                quant_config=quant_config,
                prefix=prefix,
            ),
            prefix=add_prefix("gate_projections", prefix),
        )
        if isinstance(gate_projections, tuple):
            self.start_layer, self.end_layer, self.gate_projections = gate_projections
        else:
            self.start_layer, self.end_layer = 0, config.num_hidden_layers
            self.gate_projections = gate_projections

# ... truncated for brevity ...
```
**EN:** This class defines I Quest Loop Coder Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 I Quest Loop Coder Model，用于封装该模型组件的状态与方法。

### Lines 399-494: Class: IQuestLoopCoderForCausalLM / 类：IQuestLoopCoderForCausalLM
```python
class IQuestLoopCoderForCausalLM(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.quant_config = quant_config

        self.model = IQuestLoopCoderModel(
            config=config,
            quant_config=quant_config,
            prefix=add_prefix("model", prefix),
        )

        if config.tie_word_embeddings:
            self.lm_head = self.model.embed_tokens
        else:
            self.lm_head = ParallelLMHead(
                config.vocab_size,
                config.hidden_size,
                quant_config=quant_config,
                prefix=add_prefix("lm_head", prefix),
            )

        self.logits_processor = LogitsProcessor(config)

    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: torch.Tensor = None,
    ):
        hidden_states = self.model(input_ids, positions, forward_batch, input_embeds)
        return self.logits_processor(
            input_ids, hidden_states, self.lm_head, forward_batch
# ... truncated for brevity ...
```
**EN:** This class defines I Quest Loop Coder For Causal L M inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 I Quest Loop Coder For Causal L M，用于封装该模型组件的状态与方法。

### Lines 498-498: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = IQuestLoopCoderForCausalLM
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `logging`
- `typing: Iterable, Optional, Tuple`
- `torch`
- `torch: nn`
- `transformers: PretrainedConfig`
- `sglang.srt.distributed: get_tensor_model_parallel_world_size`
- `sglang.srt.layers.layernorm: RMSNorm`
- `sglang.srt.layers.linear: ColumnParallelLinear, QKVParallelLinear, RowParallelLinear`
- `sglang.srt.layers.logits_processor: LogitsProcessor`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.layers.radix_attention: RadixAttention`
- `sglang.srt.layers.rotary_embedding: get_rope`
- `sglang.srt.layers.vocab_parallel_embedding: ParallelLMHead, VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch`
- `sglang.srt.model_loader.weight_utils: default_weight_loader`
- `sglang.srt.models.llama: LlamaMLP`
- `sglang.srt.utils: add_prefix, make_layers`
- `sglang.srt.utils.hf_transformers_utils: get_rope_config`
