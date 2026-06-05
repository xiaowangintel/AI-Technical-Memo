# gpt2.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/gpt2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the gpt2 architecture into SGLang's serving runtime. Inference-only GPT-2 model compatible with HuggingFace weights. / 该模块将 gpt2 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：Inference-only GPT-2 model compatible with HuggingFace weights。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18: Module header and imports / 模块头与导入
```python
# coding=utf-8
# Adapted from
# https://github.com/huggingface/transformers/blob/v4.28.0/src/transformers/models/gpt2/modeling_gpt2.py
# Copyright 2023 The vLLM team.
# Copyright 2018 The OpenAI Team Authors and HuggingFace Inc. team.
# Copyright (c) 2018, NVIDIA CORPORATION.  All rights reserved.
#
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
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 43-94: Class: GPT2Attention / 类：GPT2Attention
```python
class GPT2Attention(nn.Module):

    def __init__(
        self,
        layer_id: int,
        config: GPT2Config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = config.hidden_size
        total_num_heads = config.num_attention_heads
        tensor_model_parallel_world_size = get_tensor_model_parallel_world_size()
        assert total_num_heads % tensor_model_parallel_world_size == 0
        self.num_heads = total_num_heads // tensor_model_parallel_world_size
        self.head_dim = self.hidden_size // total_num_heads
        self.scale = self.head_dim**-0.5

        self.c_attn = QKVParallelLinear(
            self.hidden_size,
            self.head_dim,
            total_num_heads,
            bias=True,
            quant_config=quant_config,
            prefix=add_prefix("c_attn", prefix),
        )
        self.c_proj = RowParallelLinear(
            self.hidden_size,
            self.hidden_size,
            bias=True,
            quant_config=quant_config,
            prefix=add_prefix("c_proj", prefix),
        )
        self.attn = RadixAttention(
            self.num_heads,
            self.head_dim,
            scaling=self.scale,
            num_kv_heads=total_num_heads,
            layer_id=layer_id,
            quant_config=quant_config,
# ... truncated for brevity ...
```
**EN:** This class defines G P T2 Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 G P T2 Attention，用于封装该模型组件的状态与方法。

### Lines 97-132: Class: GPT2MLP / 类：GPT2MLP
```python
class GPT2MLP(nn.Module):

    def __init__(
        self,
        intermediate_size: int,
        config: GPT2Config,
        act_layer: Type[nn.Module] = NewGELU,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        hidden_size = config.hidden_size
        self.c_fc = ColumnParallelLinear(
            hidden_size,
            intermediate_size,
            bias=True,
            quant_config=quant_config,
            prefix=add_prefix("c_fc", prefix),
        )
        self.c_proj = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=True,
            quant_config=quant_config,
            prefix=add_prefix("c_proj", prefix),
        )
        self.act = act_layer()

    def forward(
        self,
        hidden_states: torch.Tensor,
    ) -> torch.Tensor:
        hidden_states, _ = self.c_fc(hidden_states)
        hidden_states = self.act(hidden_states)
        hidden_states, _ = self.c_proj(hidden_states)
        return hidden_states
```
**EN:** This class defines G P T2 M L P inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 G P T2 M L P，用于封装该模型组件的状态与方法。

### Lines 135-181: Class: GPT2Block / 类：GPT2Block
```python
class GPT2Block(nn.Module):

    def __init__(
        self,
        layer_id: int,
        config: GPT2Config,
        act_layer: Type[nn.Module] = NewGELU,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        hidden_size = config.hidden_size
        inner_dim = config.n_inner if config.n_inner is not None else 4 * hidden_size

        self.ln_1 = nn.LayerNorm(hidden_size, eps=config.layer_norm_epsilon)
        self.attn = GPT2Attention(
            layer_id, config, quant_config, prefix=add_prefix("attn", prefix)
        )
        self.ln_2 = nn.LayerNorm(hidden_size, eps=config.layer_norm_epsilon)
        self.mlp = GPT2MLP(
            inner_dim,
            config,
            act_layer=act_layer,
            quant_config=quant_config,
            prefix=add_prefix("mlp", prefix),
        )

    def forward(
        self,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        residual = hidden_states
        hidden_states = self.ln_1(hidden_states)
        attn_output = self.attn(
            hidden_states=hidden_states,
            forward_batch=forward_batch,
        )
        # residual connection
        hidden_states = attn_output + residual
# ... truncated for brevity ...
```
**EN:** This class defines G P T2 Block inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 G P T2 Block，用于封装该模型组件的状态与方法。

### Lines 184-229: Class: GPT2Model / 类：GPT2Model
```python
class GPT2Model(nn.Module):

    def __init__(
        self,
        config: GPT2Config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        assert not config.add_cross_attention
        assert not config.scale_attn_by_inverse_layer_idx
        assert not config.reorder_and_upcast_attn
        self.embed_dim = config.hidden_size
        self.wte = VocabParallelEmbedding(config.vocab_size, self.embed_dim)
        self.wpe = nn.Embedding(config.max_position_embeddings, self.embed_dim)
        self.h = nn.ModuleList(
            [
                GPT2Block(
                    i,
                    config,
                    quant_config=quant_config,
                    prefix=add_prefix(f"h.{i}", prefix),
                )
                for i in range(config.num_hidden_layers)
            ]
        )

        self.ln_f = nn.LayerNorm(self.embed_dim, eps=config.layer_norm_epsilon)

    def forward(
        self,
        input_ids: torch.Tensor,
        position_ids: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        inputs_embeds = self.wte(input_ids)
        position_embeds = self.wpe(position_ids)
        hidden_states = inputs_embeds + position_embeds

# ... truncated for brevity ...
```
**EN:** This class defines G P T2 Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 G P T2 Model，用于封装该模型组件的状态与方法。

### Lines 232-286: Class: GPT2LMHeadModel / 类：GPT2LMHeadModel
```python
class GPT2LMHeadModel(nn.Module):

    def __init__(
        self,
        config: GPT2Config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.quant_config = quant_config
        self.transformer = GPT2Model(
            config, quant_config, prefix=add_prefix("transformer", prefix)
        )
        self.lm_head = self.transformer.wte

        self.logits_processor = LogitsProcessor(config)

    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        hidden_states = self.transformer(input_ids, positions, forward_batch)
        return self.logits_processor(
            input_ids, hidden_states, self.lm_head, forward_batch
        )

    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        params_dict = dict(self.named_parameters(remove_duplicate=False))
        for name, loaded_weight in weights:
            if "lm_head.weight" in name:
                # GPT-2 ties the weights of the embedding layer and the final
                # linear layer.
                continue
            if ".attn.bias" in name or ".attn.masked_bias" in name:
                # Skip attention mask.
                # NOTE: "c_attn.bias" should not be skipped.
                continue
# ... truncated for brevity ...
```
**EN:** This class defines G P T2 L M Head Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 G P T2 L M Head Model，用于封装该模型组件的状态与方法。

### Lines 289-289: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = GPT2LMHeadModel
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `typing: Iterable, Optional, Tuple, Type`
- `torch`
- `torch: nn`
- `transformers: GPT2Config`
- `sglang.srt.distributed.parallel_state: get_tensor_model_parallel_world_size`
- `sglang.srt.layers.activation: NewGELU`
- `sglang.srt.layers.linear: ColumnParallelLinear, QKVParallelLinear, RowParallelLinear`
- `sglang.srt.layers.logits_processor: LogitsProcessor`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.layers.radix_attention: RadixAttention`
- `sglang.srt.layers.vocab_parallel_embedding: VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch`
- `sglang.srt.model_loader.weight_utils: default_weight_loader`
- `sglang.srt.utils: add_prefix`
