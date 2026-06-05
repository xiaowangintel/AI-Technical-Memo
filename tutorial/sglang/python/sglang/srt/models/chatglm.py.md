# chatglm.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/chatglm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Chatglm architecture into SGLang's serving runtime. Inference-only ChatGLM model compatible with THUDM weights. / 该模块将 Chatglm 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：Inference-only ChatGLM model compatible with THUDM weights。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16: Module header and imports / 模块头与导入
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

# Adapted from
# https://github.com/THUDM/ChatGLM2-6B
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 46-46: Assignment: LoraConfig / 赋值：LoraConfig
```python
LoraConfig = None
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 49-136: Class: GLMAttention / 类：GLMAttention
```python
class GLMAttention(nn.Module):
    def __init__(
        self,
        config,
        layer_id: int = 0,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = config.hidden_size
        tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = config.num_attention_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.multi_query_attention = config.multi_query_attention
        self.total_num_kv_heads = (
            config.multi_query_group_num
            if config.multi_query_attention
            else config.num_attention_heads
        )
        if self.total_num_kv_heads >= tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % tp_size == 0
        else:
            # Number of KV heads is less than TP size, so we replicate
            # the KV heads across multiple tensor parallel GPUs.
            assert tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)
        self.head_dim = config.hidden_size // self.total_num_heads
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scaling = self.head_dim**-0.5

        self.query_key_value = QKVParallelLinear(
            self.hidden_size,
            self.head_dim,
            self.total_num_heads,
            self.total_num_kv_heads,
            bias=config.add_bias_linear or config.add_qkv_bias,
# ... truncated for brevity ...
```
**EN:** This class defines G L M Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 G L M Attention，用于封装该模型组件的状态与方法。

### Lines 139-183: Class: GLMMLP / 类：GLMMLP
```python
class GLMMLP(nn.Module):
    """MLP.

    MLP will take the input with h hidden state, project it to 4*h
    hidden dimension, perform nonlinear transformation, and project the
    state back into h hidden dimension.
    """

    def __init__(
        self,
        config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()

        self.add_bias = config.add_bias_linear

        # Project to 4h.
        self.dense_h_to_4h = MergedColumnParallelLinear(
            config.hidden_size,
            [config.ffn_hidden_size] * 2,
            bias=config.add_bias_linear,
            quant_config=quant_config,
            prefix=add_prefix("dense_h_to_4h", prefix),
        )

        self.activation_func = SiluAndMul()

        # Project back to h.
        self.dense_4h_to_h = RowParallelLinear(
            config.ffn_hidden_size,
            config.hidden_size,
            bias=config.add_bias_linear,
            quant_config=quant_config,
            prefix=add_prefix("dense_4h_to_h", prefix),
        )

    def forward(self, hidden_states):
        # [s, b, 4hp]
# ... truncated for brevity ...
```
**EN:** This class defines G L M M L P inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: MLP..
**CN:** 该类定义了 G L M M L P，用于封装该模型组件的状态与方法。 文档字符串摘要：MLP.。

### Lines 186-262: Class: GLMBlock / 类：GLMBlock
```python
class GLMBlock(nn.Module):
    """A single transformer layer.

    Transformer layer takes input with size [s, b, h] and returns an
    output of the same size.
    """

    def __init__(
        self,
        config,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.apply_residual_connection_post_layernorm = (
            config.apply_residual_connection_post_layernorm
        )

        self.fp32_residual_connection = config.fp32_residual_connection

        layer_norm_func = RMSNorm if config.rmsnorm else LayerNorm
        # Layernorm on the input data.
        self.input_layernorm = layer_norm_func(
            config.hidden_size, eps=config.layernorm_epsilon
        )

        # Self attention.
        self.self_attention = GLMAttention(
            config, layer_id, quant_config, prefix=add_prefix("self_attention", prefix)
        )
        self.hidden_dropout = config.hidden_dropout

        # Layernorm on the attention output
        self.post_attention_layernorm = layer_norm_func(
            config.hidden_size, eps=config.layernorm_epsilon
        )

        # MLP
        self.mlp = GLMMLP(config, quant_config, prefix=add_prefix("mlp", prefix))
# ... truncated for brevity ...
```
**EN:** This class defines G L M Block inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: A single transformer layer..
**CN:** 该类定义了 G L M Block，用于封装该模型组件的状态与方法。 文档字符串摘要：A single transformer layer.。

### Lines 265-317: Class: GLMTransformer / 类：GLMTransformer
```python
class GLMTransformer(nn.Module):
    """Transformer class."""

    def __init__(
        self,
        config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.post_layer_norm = config.post_layer_norm

        # Number of layers.
        self.num_layers = config.num_layers

        # Transformer layers.
        self.layers = nn.ModuleList(
            [
                GLMBlock(
                    config,
                    i,
                    quant_config,
                    prefix=add_prefix(f"layers.{i}", prefix),
                )
                for i in range(self.num_layers)
            ]
        )

        if self.post_layer_norm:
            layer_norm_func = RMSNorm if config.rmsnorm else LayerNorm
            # Final layer norm before output.
            self.final_layernorm = layer_norm_func(
                config.hidden_size, eps=config.layernorm_epsilon
            )

    def forward(
        self,
        hidden_states: torch.Tensor,
        position_ids: torch.Tensor,
        forward_batch: ForwardBatch,
# ... truncated for brevity ...
```
**EN:** This class defines G L M Transformer inheriting from nn.Module, grouping state and methods for this model component. Docstring summary: Transformer class..
**CN:** 该类定义了 G L M Transformer，用于封装该模型组件的状态与方法。 文档字符串摘要：Transformer class.。

### Lines 320-362: Class: ChatGLMM / 类：ChatGLMM
```python
class ChatGLMM(nn.Module):
    def __init__(
        self,
        config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()

        self.embedding = VocabParallelEmbedding(
            config.padded_vocab_size,
            config.hidden_size,
            prefix=add_prefix("embedding", prefix),
        )

        self.num_layers = config.num_layers
        self.multi_query_group_num = config.multi_query_group_num
        self.kv_channels = config.kv_channels
        self.encoder = GLMTransformer(
            config, quant_config, add_prefix("encoder", prefix)
        )

        self.output_layer = ParallelLMHead(
            config.padded_vocab_size,
            config.hidden_size,
            prefix=add_prefix("output_layer", prefix),
        )

    def forward(
        self,
        input_ids: torch.Tensor,
        position_ids: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        inputs_embeds = self.embedding(input_ids)

        # Run encoder.
        hidden_states = self.encoder(
            hidden_states=inputs_embeds,
            position_ids=position_ids,
# ... truncated for brevity ...
```
**EN:** This class defines Chat G L M M inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Chat G L M M，用于封装该模型组件的状态与方法。

### Lines 365-420: Class: ChatGLMForCausalLM / 类：ChatGLMForCausalLM
```python
class ChatGLMForCausalLM(nn.Module):
    packed_modules_mapping = {
        "query_key_value": ["query_key_value"],
        "dense_h_to_4h": ["dense_h_to_4h"],
    }
    # LoRA specific attributes
    supported_lora_modules = [
        "query_key_value",
        "dense",
        "dense_h_to_4h",
        "dense_4h_to_h",
    ]
    embedding_modules = {}
    embedding_padding_modules = []

    def __init__(
        self,
        config: ChatGLMConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config: ChatGLMConfig = config
        self.quant_config = quant_config
        self.max_position_embeddings = getattr(config, "max_sequence_length", 8192)
        self.transformer = ChatGLMM(
            config, quant_config, prefix=add_prefix("transformer", prefix)
        )
        self.lm_head = self.transformer.output_layer
        self.logits_processor = LogitsProcessor(config)

    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        hidden_states = self.transformer(input_ids, positions, forward_batch)
        return self.logits_processor(
# ... truncated for brevity ...
```
**EN:** This class defines Chat G L M For Causal L M inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Chat G L M For Causal L M，用于封装该模型组件的状态与方法。

### Lines 423-424: Class: ChatGLMModel / 类：ChatGLMModel
```python
class ChatGLMModel(ChatGLMForCausalLM):
    pass
```
**EN:** This class defines Chat G L M Model inheriting from ChatGLMForCausalLM, grouping state and methods for this model component.
**CN:** 该类定义了 Chat G L M Model，用于封装该模型组件的状态与方法。

### Lines 427-427: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = [ChatGLMModel]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `typing: Iterable, Optional, Tuple`
- `torch`
- `torch: nn`
- `torch.nn: LayerNorm`
- `sglang.srt.configs: ChatGLMConfig`
- `sglang.srt.distributed: get_tensor_model_parallel_world_size`
- `sglang.srt.layers.activation: SiluAndMul`
- `sglang.srt.layers.layernorm: RMSNorm`
- `sglang.srt.layers.linear: MergedColumnParallelLinear, QKVParallelLinear, RowParallelLinear`
- `sglang.srt.layers.logits_processor: LogitsProcessor`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.layers.radix_attention: RadixAttention`
- `sglang.srt.layers.rotary_embedding: get_rope`
- `sglang.srt.layers.vocab_parallel_embedding: ParallelLMHead, VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch`
- `sglang.srt.model_loader.weight_utils: default_weight_loader`
- `sglang.srt.utils: add_prefix`
