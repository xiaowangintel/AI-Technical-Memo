# llama4.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/llama4.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the llama4 architecture into SGLang's serving runtime. Inference-only LLaMA model compatible with HuggingFace weights. / 该模块将 llama4 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：Inference-only LLaMA model compatible with HuggingFace weights。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18: Module header and imports / 模块头与导入
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
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
# https://github.com/vllm-project/vllm/blob/v0.8.3/vllm/model_executor/models/llama4.py
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 68-68: Assignment: _is_cuda / 赋值：_is_cuda
```python
_is_cuda = is_cuda()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 69-69: Assignment: _is_npu / 赋值：_is_npu
```python
_is_npu = is_npu()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 71-71: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 74-188: Class: Llama4MoE / 类：Llama4MoE
```python
class Llama4MoE(nn.Module):

    @torch.compile(dynamic=True, backend=get_compiler_backend())
    @staticmethod
    def custom_routing_function(
        hidden_states: torch.Tensor,
        gating_output: torch.Tensor,
        topk: int,
        renormalize: bool,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        router_scores_aK, router_indices_aK = fast_topk(gating_output, topk, dim=-1)
        router_scores_aK = torch.sigmoid(router_scores_aK.float()).to(
            hidden_states.dtype
        )
        return (
            router_scores_aK.view(-1).reshape(router_scores_aK.shape),
            router_indices_aK.to(torch.int32),
        )

    def __init__(
        self,
        config: Llama4TextConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.tp_size = get_tensor_model_parallel_world_size()
        self.top_k = config.num_experts_per_tok
        self.device_module = torch.get_device_module()

        intermediate_size_moe = config.intermediate_size
        self.router = ReplicatedLinear(
            config.hidden_size,
            config.num_local_experts,
            bias=False,
            quant_config=None,
            prefix=add_prefix("router", prefix),
        )

# ... truncated for brevity ...
```
**EN:** This class defines Llama4 Mo E inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Llama4 Mo E，用于封装该模型组件的状态与方法。

### Lines 191-191: Assignment: _alt_stream / 赋值：_alt_stream
```python
_alt_stream = None
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 194-198: Function: _get_or_create_alt_stream() / 函数：_get_or_create_alt_stream()
```python
def _get_or_create_alt_stream(device_module):
    global _alt_stream
    if _alt_stream is None:
        _alt_stream = device_module.Stream()
    return _alt_stream
```
**EN:** This function implements get or create alt stream for the surrounding model/runtime logic. Key parameters include device_module.
**CN:** 该函数实现了 get or create alt stream 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 201-376: Class: Llama4Attention / 类：Llama4Attention
```python
class Llama4Attention(nn.Module):

    def __init__(
        self,
        config: Llama4TextConfig,
        layer_id: int,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        rope_theta: float = 10000,
        rope_scaling: Optional[Dict[str, Any]] = None,
        max_position_embeddings: int = 8192,
        quant_config: Optional[QuantizationConfig] = None,
        bias: bool = False,
        bias_o_proj: bool = False,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.layer_id = layer_id
        self.hidden_size = hidden_size
        self.use_rope = (layer_id + 1) % 4 != 0
        self.use_qk_norm = config.use_qk_norm and self.use_rope

        attn_tp_rank = get_attention_tp_rank()
        attn_tp_size = get_attention_tp_size()

        self.total_num_heads = num_heads
        assert self.total_num_heads % attn_tp_size == 0
        self.num_heads = self.total_num_heads // attn_tp_size
        self.total_num_kv_heads = num_kv_heads
        if self.total_num_kv_heads >= attn_tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % attn_tp_size == 0
        else:
            # Number of KV heads is less than TP size, so we replicate
            # the KV heads across multiple tensor parallel GPUs.
            assert attn_tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // attn_tp_size)
        self.head_dim = config.head_dim
# ... truncated for brevity ...
```
**EN:** This class defines Llama4 Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Llama4 Attention，用于封装该模型组件的状态与方法。

### Lines 379-496: Class: Llama4DecoderLayer / 类：Llama4DecoderLayer
```python
class Llama4DecoderLayer(nn.Module):
    def __init__(
        self,
        config: Llama4TextConfig,
        layer_id: int = 0,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.layer_id = layer_id
        self.hidden_size = config.hidden_size
        rope_theta = config.rope_parameters["rope_theta"]
        rope_scaling = config.rope_parameters
        max_position_embeddings = config.max_position_embeddings
        self.attn_tp_size = get_attention_tp_size()
        self.attn_tp_rank = get_attention_tp_rank()

        self.self_attn = Llama4Attention(
            config=config,
            layer_id=layer_id,
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            num_kv_heads=config.num_key_value_heads,
            rope_theta=rope_theta,
            rope_scaling=rope_scaling,
            max_position_embeddings=max_position_embeddings,
            quant_config=quant_config,
            bias=False,
            bias_o_proj=False,
            prefix=add_prefix("self_attn", prefix),
        )
        self.config = config
        is_moe_layer = self._is_moe_layer(layer_id)
        is_previous_moe_layer = self._is_moe_layer(layer_id - 1)
        is_next_moe_layer = self._is_moe_layer(layer_id + 1)

        if is_moe_layer:
            self.feed_forward = Llama4MoE(
                config=config,
                layer_id=layer_id,
# ... truncated for brevity ...
```
**EN:** This class defines Llama4 Decoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Llama4 Decoder Layer，用于封装该模型组件的状态与方法。

### Lines 499-558: Class: Llama4Model / 类：Llama4Model
```python
class Llama4Model(nn.Module):
    def __init__(
        self,
        config: Llama4TextConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.padding_idx = config.pad_token_id
        self.vocab_size = config.vocab_size
        self.embed_tokens = VocabParallelEmbedding(
            config.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=add_prefix("embed_tokens", prefix),
            use_attn_tp_group=is_dp_attention_enabled(),
        )
        self.layers = make_layers(
            config.num_hidden_layers,
            lambda idx, prefix: Llama4DecoderLayer(
                config=config, layer_id=idx, quant_config=quant_config, prefix=prefix
            ),
            prefix=add_prefix("layers", prefix),
        )

        self.norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.layers_to_capture = []

    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: torch.Tensor = None,
        pp_proxy_tensors: Optional[PPProxyTensors] = None,
    ) -> Union[torch.Tensor, Tuple[torch.Tensor, List[torch.Tensor]]]:
        if input_embeds is None:
            hidden_states = self.embed_tokens(input_ids)
        else:
# ... truncated for brevity ...
```
**EN:** This class defines Llama4 Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Llama4 Model，用于封装该模型组件的状态与方法。

### Lines 561-587: Class: Llama4ForCausalLM / 类：Llama4ForCausalLM
```python
class Llama4ForCausalLM(LlamaForCausalLM):
    packed_modules_mapping = {
        "qkv_proj": ["q_proj", "k_proj", "v_proj"],
        "gate_up_proj": ["gate_proj", "up_proj"],
    }

    def __init__(
        self,
        config: Llama4TextConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__(config, quant_config, prefix)

    def get_input_embeddings(self):
        return self.model.embed_tokens

    def get_layers(self):
        return self.model.layers

    def _init_model(
        self,
        config: Llama4TextConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        return Llama4Model(config, quant_config=quant_config, prefix=prefix)
```
**EN:** This class defines Llama4 For Causal L M inheriting from LlamaForCausalLM, grouping state and methods for this model component.
**CN:** 该类定义了 Llama4 For Causal L M，用于封装该模型组件的状态与方法。

### Lines 590-590: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = [Llama4ForCausalLM]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `logging`
- `typing: Any, Dict, List, Optional, Tuple, Union`
- `torch`
- `torch: nn`
- `transformers: Llama4TextConfig`
- `sglang.srt.distributed: get_tensor_model_parallel_world_size, tensor_model_parallel_all_reduce`
- `sglang.srt.layers.communicator: LayerCommunicator, LayerScatterModes`
- `sglang.srt.layers.dp_attention: get_attention_tp_rank, get_attention_tp_size, is_dp_attention_enabled`
- `sglang.srt.layers.layernorm: RMSNorm`
- `sglang.srt.layers.linear: QKVParallelLinear, ReplicatedLinear, RowParallelLinear`
- `sglang.srt.layers.moe: should_skip_post_experts_all_reduce`
- `sglang.srt.layers.moe.fused_moe_triton: FusedMoE`
- `sglang.srt.layers.moe.topk: TopK`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.layers.radix_attention: RadixAttention`
- `sglang.srt.layers.rotary_embedding: get_rope`
- `sglang.srt.layers.vocab_parallel_embedding: VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch, ForwardMode, PPProxyTensors`
- `sglang.srt.models.llama: LlamaForCausalLM, LlamaMLP`
- `sglang.srt.models.utils: apply_qk_norm`
