# xverse_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/xverse_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Inference-only XVERSE MoE model. This file provides runtime-ready implementation details for SGLang inference. / 该文件围绕上述主题实现了可直接用于 SGLang 推理运行时的具体逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 14-14: Module docstring
```python
"""Inference-only XVERSE MoE model."""
```
**EN:** States the module scope and documents the high-level intention of the implementation.
**CN:** 说明模块范围，并给出实现的高层设计意图。

### Lines 16-52: Module imports
```python
from typing import Any, Dict, Iterable, Optional, Tuple

import torch
from torch import nn
from transformers import PretrainedConfig

from sglang.srt.distributed import (
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
    tensor_model_parallel_all_reduce,
)
from sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu import (
    fused_moe_npu,
)
from sglang.srt.layers.activation import SiluAndMul
from sglang.srt.layers.layernorm import RMSNorm
from sglang.srt.layers.linear import (
    MergedColumnParallelLinear,
    QKVParallelLinear,
    ReplicatedLinear,
    RowParallelLinear,
)
from sglang.srt.layers.logits_processor import LogitsProcessor
from sglang.srt.layers.moe.moe_runner import MoeRunnerConfig
from sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe import fused_moe
from sglang.srt.layers.moe.topk import TopK
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.layers.radix_attention import RadixAttention
from sglang.srt.layers.rotary_embedding import get_rope
from sglang.srt.layers.vocab_parallel_embedding import (
    ParallelLMHead,
    VocabParallelEmbedding,
)
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.model_loader.weight_utils import default_weight_loader
from sglang.srt.utils import add_prefix, is_npu
from sglang.srt.utils.hf_transformers_utils import get_rope_config
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 55-56: Class `XverseMLP` overview
```python
class XverseMLP(nn.Module):
```
**EN:** Defines `XverseMLP` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `XverseMLP`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 57-87: Method `XverseMLP.__init__`
```python
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        hidden_act: str,
        quant_config: Optional[QuantizationConfig] = None,
        reduce_results: bool = True,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.gate_up_proj = MergedColumnParallelLinear(
            hidden_size,
            [intermediate_size] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("gate_up_proj", prefix),
        )
        self.down_proj = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=False,
            quant_config=quant_config,
            reduce_results=reduce_results,
            prefix=add_prefix("down_proj", prefix),
        )
        if hidden_act != "silu":
            raise ValueError(
                f"Unsupported activation: {hidden_act}. "
                "Only silu is supported for now."
            )
        self.act_fn = SiluAndMul()
```
**EN:** This method implements `__init__(hidden_size: ..., intermediate_size: ..., hidden_act: ..., quant_config: ...=..., reduce_results: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(hidden_size: ..., intermediate_size: ..., hidden_act: ..., quant_config: ...=..., reduce_results: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 89-93: Method `XverseMLP.forward`
```python
    def forward(self, x):
        gate_up, _ = self.gate_up_proj(x)
        x = self.act_fn(gate_up)
        x, _ = self.down_proj(x)
        return x
```
**EN:** This method implements `forward(x)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(x)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 96-97: Class `XverseMoE` overview
```python
class XverseMoE(nn.Module):
```
**EN:** Defines `XverseMoE` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `XverseMoE`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 98-154: Method `XverseMoE.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.config = config
        self.rank = get_tensor_model_parallel_rank()
        self.tp_size = get_tensor_model_parallel_world_size()
        self.n_routed_experts = config.num_experts
        self.top_k = config.moe_top_k
        if self.tp_size > self.n_routed_experts:
            raise ValueError(
                f"Tensor parallel size {self.tp_size} is greater than "
                f"the number of experts {self.n_routed_experts}."
            )

        self.experts = nn.ModuleList(
            [
                XverseMLP(
                    hidden_size=config.hidden_size,
                    intermediate_size=config.intermediate_size,
                    hidden_act=config.hidden_act,
                    quant_config=quant_config,
                    reduce_results=False,
                    prefix=add_prefix(f"experts.{i}", prefix),
                )
                for i in range(self.n_routed_experts)
            ]
        )
        self.pack_params()
        self.moe_runner_config = MoeRunnerConfig(inplace=True)

        self.router = ReplicatedLinear(
            config.hidden_size,
            self.n_routed_experts,
            bias=False,
            quant_config=None,
            prefix=add_prefix("router", prefix),
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 156-173: Method `XverseMoE.pack_params`
```python
    def pack_params(self):
        w1 = []
        w2 = []
        for expert in self.experts:
            w1.append(expert.gate_up_proj.weight)
            w2.append(expert.down_proj.weight)
        self.w1 = torch._utils._flatten_dense_tensors(w1)
        w1s = torch._utils._unflatten_dense_tensors(self.w1, w1)
        for data, param in zip(w1s, w1):
            param.data = data
        self.w1 = self.w1.view(len(w1), *w1s[0].shape)

        self.w2 = torch._utils._flatten_dense_tensors(w2)
        w2s = torch._utils._unflatten_dense_tensors(self.w2, w2)
        for data, param in zip(w2s, w2):
            param.data = data

        self.w2 = self.w2.view(len(w2), *w2s[0].shape)
```
**EN:** This method implements `pack_params()` and implements a focused helper that supports the surrounding runtime flow inside `XverseMoE`.
**CN:** 这个方法实现了 `pack_params()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `XverseMoE` 内部调用。

### Lines 175-195: Method `XverseMoE.forward`
```python
    def forward(self, hidden_states: torch.Tensor) -> torch.Tensor:
        num_tokens, hidden_dim = hidden_states.shape
        hidden_states = hidden_states.view(-1, hidden_dim)
        if self.config.num_shared_experts is not None:
            shared_output = self.shared_experts(hidden_states)
        # router_logits: (num_tokens, n_experts)
        router_logits, _ = self.router(hidden_states)
        topk_output = self.topk(hidden_states, router_logits)
        final_hidden_states = self.fused_moe_method(
            hidden_states,
            self.w1,
            self.w2,
            topk_output,
            self.moe_runner_config,
        )

        if self.config.num_shared_experts is not None:
            final_hidden_states = final_hidden_states + shared_output
        final_hidden_states = tensor_model_parallel_all_reduce(final_hidden_states)

        return final_hidden_states.view(num_tokens, hidden_dim)
```
**EN:** This method implements `forward(hidden_states: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 198-199: Class `XverseAttention` overview
```python
class XverseAttention(nn.Module):
```
**EN:** Defines `XverseAttention` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `XverseAttention`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 200-268: Method `XverseAttention.__init__`
```python
    def __init__(
        self,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        layer_id: int = 0,
        rope_theta: float = 10000,
        rope_scaling: Optional[Dict[str, Any]] = None,
        max_position_embeddings: int = 8192,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size
        tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = num_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = num_kv_heads
        if self.total_num_kv_heads >= tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % tp_size == 0
        else:
            # Number of KV heads is less than TP size, so we replicate
            # the KV heads across multiple tensor parallel GPUs.
            assert tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)
        self.head_dim = hidden_size // self.total_num_heads
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scaling = self.head_dim**-0.5
        self.rope_theta = rope_theta
        self.max_position_embeddings = max_position_embeddings

        self.qkv_proj = QKVParallelLinear(
            hidden_size,
            self.head_dim,
            self.total_num_heads,
            self.total_num_kv_heads,
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(hidden_size: ..., num_heads: ..., num_kv_heads: ..., layer_id: ...=..., rope_theta: ...=..., rope_scaling: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(hidden_size: ..., num_heads: ..., num_kv_heads: ..., layer_id: ...=..., rope_theta: ...=..., rope_scaling: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 270-281: Method `XverseAttention.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        q, k = self.rotary_emb(positions, q, k)
        attn_output = self.attn(q, k, v, forward_batch)
        output, _ = self.o_proj(attn_output)
        return output
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 284-285: Class `XverseDecoderLayer` overview
```python
class XverseDecoderLayer(nn.Module):
```
**EN:** Defines `XverseDecoderLayer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `XverseDecoderLayer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 286-328: Method `XverseDecoderLayer.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        rope_theta, rope_scaling = get_rope_config(config)
        max_position_embeddings = getattr(config, "max_position_embeddings", 8192)
        num_key_value_heads = getattr(
            config, "num_key_value_heads", config.num_attention_heads
        )
        self.self_attn = XverseAttention(
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            num_kv_heads=num_key_value_heads,
            layer_id=layer_id,
            rope_theta=rope_theta,
            rope_scaling=rope_scaling,
            max_position_embeddings=max_position_embeddings,
            quant_config=quant_config,
            prefix=add_prefix("self_attn", prefix),
        )
        if config.num_experts is not None:
            self.mlp = XverseMoE(
                config=config,
                quant_config=quant_config,
                prefix=add_prefix("mlp", prefix),
            )
        else:
            self.mlp = XverseMLP(
                hidden_size=config.hidden_size,
                intermediate_size=config.intermediate_size,
                hidden_act=config.hidden_act,
                quant_config=quant_config,
                prefix=add_prefix("mlp", prefix),
            )
        self.input_layernorm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 330-352: Method `XverseDecoderLayer.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
        residual: Optional[torch.Tensor],
    ) -> torch.Tensor:
        # Self Attention
        if residual is None:
            residual = hidden_states
            hidden_states = self.input_layernorm(hidden_states)
        else:
            hidden_states, residual = self.input_layernorm(hidden_states, residual)
        hidden_states = self.self_attn(
            positions=positions,
            hidden_states=hidden_states,
            forward_batch=forward_batch,
        )

        # Fully Connected
        hidden_states, residual = self.post_attention_layernorm(hidden_states, residual)
        hidden_states = self.mlp(hidden_states)
        return hidden_states, residual
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ..., residual: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ..., residual: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 355-356: Class `XverseModel` overview
```python
class XverseModel(nn.Module):
```
**EN:** Defines `XverseModel` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `XverseModel`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 357-357: Class `XverseModel` attributes
```python
    fall_back_to_pt_during_load = False
```
**EN:** Defines class-level attributes and metadata that shape how `XverseModel` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `XverseModel` 在运行时的行为。

### Lines 359-385: Method `XverseModel.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.padding_idx = config.pad_token_id
        self.vocab_size = config.vocab_size

        self.embed_tokens = VocabParallelEmbedding(
            config.vocab_size,
            config.hidden_size,
            prefix=add_prefix("embed_tokens", prefix),
        )
        self.layers = nn.ModuleList(
            [
                XverseDecoderLayer(
                    config,
                    layer_id,
                    quant_config=quant_config,
                    prefix=add_prefix(f"layers.{layer_id}", prefix),
                )
                for layer_id in range(config.num_hidden_layers)
            ]
        )
        self.norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 387-401: Method `XverseModel.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        hidden_states = self.embed_tokens(input_ids)
        residual = None
        for i in range(len(self.layers)):
            layer = self.layers[i]
            hidden_states, residual = layer(
                positions, hidden_states, forward_batch, residual
            )
        hidden_states, _ = self.norm(hidden_states, residual)
        return hidden_states
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 404-405: Class `XverseMoeForCausalLM` overview
```python
class XverseMoeForCausalLM(nn.Module):
```
**EN:** Defines `XverseMoeForCausalLM` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `XverseMoeForCausalLM`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 406-424: Method `XverseMoeForCausalLM.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.quant_config = quant_config
        self.model = XverseModel(
            config, quant_config, prefix=add_prefix("model", prefix)
        )
        self.lm_head = ParallelLMHead(
            config.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=add_prefix("lm_head", prefix),
        )
        self.logits_processor = LogitsProcessor(config)
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 426-436: Method `XverseMoeForCausalLM.forward`
```python
    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        hidden_states = self.model(input_ids, positions, forward_batch)
        return self.logits_processor(
            input_ids, hidden_states, self.lm_head, forward_batch
        )
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 438-479: Method `XverseMoeForCausalLM.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
            ("gate_up_proj", "gate_proj", 0),
            ("gate_up_proj", "up_proj", 1),
        ]
        params_dict = dict(self.named_parameters())

        for name, loaded_weight in weights:
            if "rotary_emb.inv_freq" in name:
                continue
            for param_name, weight_name, shard_id in stacked_params_mapping:
                if weight_name not in name:
                    continue
                name = name.replace(weight_name, param_name)
                # Skip loading extra bias for GPTQ models.
                if name.endswith(".bias") and name not in params_dict:
                    continue
                # Skip experts that are not assigned to this worker.
                if (
                    "mlp.experts." in name or "mlp.shared_experts." in name
                ) and name not in params_dict:
                    continue
                param = params_dict[name]
                weight_loader = param.weight_loader
                weight_loader(param, loaded_weight, shard_id)
                break
            else:
                # Skip loading extra bias for GPTQ models.
                if name.endswith(".bias") and name not in params_dict:
                    continue
                # Skip experts that are not assigned to this worker.
                if (
                    "mlp.experts." in name or "mlp.shared_experts." in name
                ) and name not in params_dict:
                    continue
                param = params_dict[name]
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 482-482: Top-level assign
```python
EntryClass = XverseMoeForCausalLM
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `typing.Any`
- `typing.Dict`
- `typing.Iterable`
- `typing.Optional`
- `typing.Tuple`
- `torch`
- `torch.nn`
- `transformers.PretrainedConfig`
- `sglang.srt.distributed.get_tensor_model_parallel_rank`
- `sglang.srt.distributed.get_tensor_model_parallel_world_size`
- `sglang.srt.distributed.tensor_model_parallel_all_reduce`
- `sglang.srt.hardware_backend.npu.quantization.fused_moe_method_npu.fused_moe_npu`
- `sglang.srt.layers.activation.SiluAndMul`
- `sglang.srt.layers.layernorm.RMSNorm`
- `sglang.srt.layers.linear.MergedColumnParallelLinear`
- `sglang.srt.layers.linear.QKVParallelLinear`
- `sglang.srt.layers.linear.ReplicatedLinear`
- `sglang.srt.layers.linear.RowParallelLinear`
- `sglang.srt.layers.logits_processor.LogitsProcessor`
- `sglang.srt.layers.moe.moe_runner.MoeRunnerConfig`
- `sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe.fused_moe`
- `sglang.srt.layers.moe.topk.TopK`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.layers.rotary_embedding.get_rope`
- `sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead`
- `sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.is_npu`
- `sglang.srt.utils.hf_transformers_utils.get_rope_config`
