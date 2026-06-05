# sdar.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/sdar.py`
- **Repository**: sgl-project/sglang
- **Purpose**: SGLang SDARModelLM (block diffusion / dLLM-style forward). This file provides runtime-ready implementation details for SGLang inference. / 该文件围绕上述主题实现了可直接用于 SGLang 推理运行时的具体逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 2-4: Module docstring
```python
"""
SGLang SDARModelLM (block diffusion / dLLM-style forward).
"""
```
**EN:** States the module scope and documents the high-level intention of the implementation.
**CN:** 说明模块范围，并给出实现的高层设计意图。

### Lines 6-47: Module imports
```python
import logging
from typing import Iterable, Optional, Tuple, Union

import torch
from torch import nn
from transformers import PretrainedConfig

from sglang.srt.distributed import get_pp_group, get_tensor_model_parallel_world_size
from sglang.srt.layers.activation import SiluAndMul
from sglang.srt.layers.communicator import LayerCommunicator, LayerScatterModes
from sglang.srt.layers.dp_attention import (
    get_attention_tp_rank,
    get_attention_tp_size,
    is_dp_attention_enabled,
)
from sglang.srt.layers.layernorm import RMSNorm
from sglang.srt.layers.linear import (
    MergedColumnParallelLinear,
    QKVParallelLinear,
    RowParallelLinear,
)
from sglang.srt.layers.logits_processor import LogitsProcessor
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.layers.radix_attention import AttentionType, RadixAttention
from sglang.srt.layers.rotary_embedding import get_rope
from sglang.srt.layers.utils import PPMissingLayer, get_layer_id
from sglang.srt.layers.vocab_parallel_embedding import (
    ParallelLMHead,
    VocabParallelEmbedding,
)
from sglang.srt.model_executor.forward_batch_info import ForwardBatch, PPProxyTensors
from sglang.srt.model_loader.weight_utils import (
    default_weight_loader,
    maybe_remap_kv_scale_name,
)
from sglang.srt.models.utils import (
    apply_qk_norm,
    create_fused_set_kv_buffer_arg,
    enable_fused_set_kv_buffer,
)
# ... truncated for brevity ...
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 49-49: Top-level assign
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines or updates logger, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 logger，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 50-50: Top-level assign
```python
_is_cuda = is_cuda()
```
**EN:** Defines or updates _is_cuda, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_cuda，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 53-53: Class `SDARMLP` overview
```python
class SDARMLP(nn.Module):
```
**EN:** Defines `SDARMLP` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `SDARMLP`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 54-77: Method `SDARMLP.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config=None,
        reduce_results: bool = True,
        prefix: str = "",
    ):
        super().__init__()
        self.gate_up_proj = MergedColumnParallelLinear(
            config.hidden_size,
            [config.intermediate_size] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("gate_up_proj", prefix),
        )
        self.down_proj = RowParallelLinear(
            config.intermediate_size,
            config.hidden_size,
            bias=False,
            reduce_results=reduce_results,
            quant_config=quant_config,
            prefix=add_prefix("down_proj", prefix),
        )
        self.act_fn = SiluAndMul()
```
**EN:** This method implements `__init__(config: ..., quant_config=..., reduce_results: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config=..., reduce_results: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 79-85: Method `SDARMLP.forward`
```python
    def forward(self, hidden_states: torch.Tensor, use_reduce_scatter: bool = False):
        gate_up, _ = self.gate_up_proj(hidden_states)
        hidden_states = self.act_fn(gate_up)
        hidden_states, _ = self.down_proj(
            hidden_states, skip_all_reduce=use_reduce_scatter
        )
        return hidden_states
```
**EN:** This method implements `forward(hidden_states: ..., use_reduce_scatter: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(hidden_states: ..., use_reduce_scatter: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 88-88: Class `SDARAttention` overview
```python
class SDARAttention(nn.Module):
```
**EN:** Defines `SDARAttention` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `SDARAttention`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 89-174: Method `SDARAttention.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        layer_id: int,
        quant_config=None,
        reduce_results: bool = True,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
    ):
        super().__init__()
        self.layer_id = layer_id
        self.hidden_size = config.hidden_size
        self.total_num_heads = config.num_attention_heads
        self.tp_size = get_tensor_model_parallel_world_size()
        attn_tp_rank = get_attention_tp_rank()
        attn_tp_size = get_attention_tp_size()

        assert self.total_num_heads % attn_tp_size == 0
        self.num_heads = self.total_num_heads // attn_tp_size
        self.total_num_kv_heads = config.num_key_value_heads
        if self.total_num_kv_heads >= attn_tp_size:
            # Number of KV heads is greater than TP size, so we partition
            # the KV heads across multiple tensor parallel GPUs.
            assert self.total_num_kv_heads % attn_tp_size == 0
        else:
            # Number of KV heads is less than TP size, so we replicate
            # the KV heads across multiple tensor parallel GPUs.
            assert attn_tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // attn_tp_size)

        self.head_dim = getattr(
            config, "head_dim", self.hidden_size // self.total_num_heads
        )
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scale = self.head_dim**-0.5

        self.qkv_proj = QKVParallelLinear(
            self.hidden_size,
            self.head_dim,
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., layer_id: ..., quant_config=..., reduce_results: ...=..., prefix: ...=..., alt_stream: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ..., quant_config=..., reduce_results: ...=..., prefix: ...=..., alt_stream: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 176-201: Method `SDARAttention.forward_prepare_native`
```python
    def forward_prepare_native(self, positions, hidden_states, forward_batch):
        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        q, k = apply_qk_norm(
            q=q,
            k=k,
            q_norm=self.q_norm,
            k_norm=self.k_norm,
            head_dim=self.head_dim,
            alt_stream=self.alt_stream,
        )
        q, k = self.rotary_emb(
            positions,
            q,
            k,
            fused_set_kv_buffer_arg=(
                create_fused_set_kv_buffer_arg(
                    value=v,
                    layer=self.attn,
                    forward_batch=forward_batch,
                )
                if enable_fused_set_kv_buffer(forward_batch)
                else None
            ),
        )
        return q, k, v
```
**EN:** This method implements `forward_prepare_native(positions, hidden_states, forward_batch)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward_prepare_native(positions, hidden_states, forward_batch)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 203-249: Method `SDARAttention.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
    ):
        if get_global_server_args().rl_on_policy_target is not None:
            hidden_states = hidden_states.bfloat16()

        qkv, _ = self.qkv_proj(hidden_states)
        q, k, v = qkv.split([self.q_size, self.kv_size, self.kv_size], dim=-1)
        q, k = apply_qk_norm(
            q=q,
            k=k,
            q_norm=self.q_norm,
            k_norm=self.k_norm,
            head_dim=self.head_dim,
            alt_stream=self.alt_stream,
        )
        q, k = self.rotary_emb(
            positions,
            q,
            k,
            fused_set_kv_buffer_arg=(
                create_fused_set_kv_buffer_arg(
                    value=v,
                    layer=self.attn,
                    forward_batch=forward_batch,
                )
                if enable_fused_set_kv_buffer(forward_batch)
                else None
            ),
        )

        if get_global_server_args().rl_on_policy_target is not None:
            q = q.to(torch.bfloat16)
            k = k.to(torch.bfloat16)

        context_layer = self.attn(
            q,
# ... truncated for brevity ...
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 252-252: Class `SDARBlock` overview
```python
class SDARBlock(nn.Module):
```
**EN:** Defines `SDARBlock` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `SDARBlock`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 253-310: Method `SDARBlock.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
    ):
        super().__init__()
        self.hidden_size = config.hidden_size
        self.layer_id = layer_id

        norm_kwargs = (
            dict(
                weight_dtype=torch.float32,
                cast_x_before_out_mul=True,
                override_orig_dtype=torch.float32,
                fp32_residual=True,
            )
            if get_global_server_args().rl_on_policy_target is not None
            else {}
        )
        self.input_layernorm = RMSNorm(
            self.hidden_size, eps=config.rms_norm_eps, **norm_kwargs
        )
        self.post_attention_layernorm = RMSNorm(
            self.hidden_size, eps=config.rms_norm_eps, **norm_kwargs
        )

        self.self_attn = SDARAttention(
            layer_id=layer_id,
            config=config,
            quant_config=quant_config,
            reduce_results=False,
            prefix=add_prefix("self_attn", prefix),
            alt_stream=alt_stream,
        )

        self.mlp = SDARMLP(
            config=config,
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 312-347: Method `SDARBlock.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
        residual: Optional[torch.Tensor],
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        hidden_states, residual = self.layer_communicator.prepare_attn(
            hidden_states,
            residual,
            forward_batch,
        )

        if hidden_states.shape[0] != 0:
            hidden_states = self.self_attn(
                positions=positions,
                hidden_states=hidden_states,
                forward_batch=forward_batch,
            )

        hidden_states, residual = self.layer_communicator.prepare_mlp(
            hidden_states,
            residual,
            forward_batch,
        )

        use_reduce_scatter = self.layer_communicator.should_use_reduce_scatter(
            forward_batch
        )
        hidden_states = self.mlp(hidden_states, use_reduce_scatter=use_reduce_scatter)

        hidden_states, residual = self.layer_communicator.postprocess_layer(
            hidden_states, residual, forward_batch
        )

        return hidden_states, residual
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ..., residual: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ..., residual: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 350-350: Class `SDARModel` overview
```python
class SDARModel(nn.Module):
```
**EN:** Defines `SDARModel` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `SDARModel`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 351-401: Method `SDARModel.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
    ):
        super().__init__()
        self.config = config
        self.vocab_size = config.vocab_size
        self.embed_dim = config.hidden_size
        self.pp_group = get_pp_group()

        if self.pp_group.is_first_rank:
            self.embed_tokens = VocabParallelEmbedding(
                self.vocab_size,
                self.embed_dim,
                quant_config=quant_config,
                use_attn_tp_group=is_dp_attention_enabled(),
                prefix=add_prefix("embed_tokens", prefix),
            )
        else:
            self.embed_tokens = PPMissingLayer()

        self.layers, self.start_layer, self.end_layer = make_layers(
            config.num_hidden_layers,
            lambda idx, prefix: SDARBlock(
                layer_id=idx,
                config=config,
                quant_config=quant_config,
                prefix=prefix,
                alt_stream=alt_stream,
            ),
            pp_rank=self.pp_group.rank_in_group,
            pp_size=self.pp_group.world_size,
            prefix=add_prefix("layers", prefix),
        )
        if self.pp_group.is_last_rank:
            norm_kwargs = (
                dict(
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 403-434: Method `SDARModel.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: torch.Tensor = None,
        pp_proxy_tensors: Optional[PPProxyTensors] = None,
    ) -> Union[torch.Tensor, PPProxyTensors]:
        if self.pp_group.is_first_rank:
            hidden_states = (
                self.embed_tokens(input_ids) if input_embeds is None else input_embeds
            )
            residual = None
        else:
            assert pp_proxy_tensors is not None
            hidden_states = pp_proxy_tensors["hidden_states"]
            residual = pp_proxy_tensors.get("residual", None)

        for i in range(self.start_layer, self.end_layer):
            layer = self.layers[i]
            hidden_states, residual = layer(
                positions, hidden_states, forward_batch, residual
            )

        if not self.pp_group.is_last_rank:
            return PPProxyTensors(
                {"hidden_states": hidden_states, "residual": residual}
            )
        else:
            if not forward_batch.forward_mode.is_idle():
                hidden_states, residual = self.norm(hidden_states, residual)
            return hidden_states
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 437-437: Class `SDARForCausalLM` overview
```python
class SDARForCausalLM(nn.Module):
```
**EN:** Defines `SDARForCausalLM` as a reusable runtime type derived from nn.Module. The class groups 5 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `SDARForCausalLM`，其继承关系为 nn.Module。这个类组织了 5 个方法，用于实现模型相关行为。

### Lines 438-481: Method `SDARForCausalLM.__init__`
```python
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.pp_group = get_pp_group()
        assert self.pp_group.world_size == 1, (
            f"SDARMoeForCausalLM does not support pipeline parallel (pp_size={self.pp_group.world_size}). "
            "Please set pp_size=1."
        )

        self.config = config
        self.quant_config = quant_config
        alt_stream = torch.cuda.Stream() if _is_cuda else None

        self.model = SDARModel(
            config,
            quant_config=quant_config,
            prefix=add_prefix("model", ""),
            alt_stream=alt_stream,
        )

        if self.pp_group.is_last_rank:
            tp_size = get_tensor_model_parallel_world_size()
            if (
                self.pp_group.world_size == 1
                and config.tie_word_embeddings
                and tp_size == 1
            ):
                self.lm_head = self.model.embed_tokens
            else:
                self.lm_head = ParallelLMHead(
                    config.vocab_size,
                    config.hidden_size,
                    quant_config=quant_config,
                    use_attn_tp_group=get_global_server_args().enable_dp_lm_head,
                    prefix=add_prefix("lm_head", prefix),
                )
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 483-485: Method `SDARForCausalLM.start_layer`
```python
    @property
    def start_layer(self):
        return self.model.start_layer
```
**EN:** This method implements `start_layer()` and implements a focused helper that supports the surrounding runtime flow inside `SDARForCausalLM` Decorators: property.
**CN:** 这个方法实现了 `start_layer()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `SDARForCausalLM` 内部调用 装饰器：property。

### Lines 487-489: Method `SDARForCausalLM.end_layer`
```python
    @property
    def end_layer(self):
        return self.model.end_layer
```
**EN:** This method implements `end_layer()` and implements a focused helper that supports the surrounding runtime flow inside `SDARForCausalLM` Decorators: property.
**CN:** 这个方法实现了 `end_layer()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `SDARForCausalLM` 内部调用 装饰器：property。

### Lines 491-512: Method `SDARForCausalLM.forward`
```python
    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: Optional[torch.Tensor] = None,
        pp_proxy_tensors: Optional[PPProxyTensors] = None,
    ) -> torch.Tensor:
        hidden_states = self.model(
            input_ids=input_ids,
            positions=positions,
            forward_batch=forward_batch,
            input_embeds=input_embeds,
            pp_proxy_tensors=pp_proxy_tensors,
        )
        if self.pp_group.is_last_rank:
            return self.logits_processor(
                input_ids, hidden_states, self.lm_head, forward_batch
            )
        else:
            return hidden_states
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=...)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., pp_proxy_tensors: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 514-586: Method `SDARForCausalLM.load_weights`
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
            if not name.startswith("model.") and (
                name.startswith("layers.")
                or name.startswith("embed_tokens.")
                or name.startswith("norm.")
            ):
                name = add_prefix(name, "model")

            if name == "model.embed_tokens.weight":
                if self.pp_group.is_last_rank and self.config.tie_word_embeddings:
                    param = params_dict["lm_head.weight"]
                    weight_loader = getattr(
                        param, "weight_loader", default_weight_loader
                    )
                    weight_loader(param, loaded_weight)

            layer_id = get_layer_id(name)
            if (
                layer_id is not None
                and hasattr(self.model, "start_layer")
                and (
                    layer_id < self.model.start_layer
                    or layer_id >= self.model.end_layer
                )
            ):
                continue

            if "rotary_emb.inv_freq" in name or "projector" in name:
                continue
            if "rotary_emb.cos_cached" in name or "rotary_emb.sin_cached" in name:
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 589-589: Top-level assign
```python
EntryClass = SDARForCausalLM
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `logging`
- `typing.Iterable`
- `typing.Optional`
- `typing.Tuple`
- `typing.Union`
- `torch`
- `torch.nn`
- `transformers.PretrainedConfig`
- `sglang.srt.distributed.get_pp_group`
- `sglang.srt.distributed.get_tensor_model_parallel_world_size`
- `sglang.srt.layers.activation.SiluAndMul`
- `sglang.srt.layers.communicator.LayerCommunicator`
- `sglang.srt.layers.communicator.LayerScatterModes`
- `sglang.srt.layers.dp_attention.get_attention_tp_rank`
- `sglang.srt.layers.dp_attention.get_attention_tp_size`
- `sglang.srt.layers.dp_attention.is_dp_attention_enabled`
- `sglang.srt.layers.layernorm.RMSNorm`
- `sglang.srt.layers.linear.MergedColumnParallelLinear`
- `sglang.srt.layers.linear.QKVParallelLinear`
- `sglang.srt.layers.linear.RowParallelLinear`
- `sglang.srt.layers.logits_processor.LogitsProcessor`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.layers.radix_attention.AttentionType`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.layers.rotary_embedding.get_rope`
- `sglang.srt.layers.utils.PPMissingLayer`
- `sglang.srt.layers.utils.get_layer_id`
- `sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead`
- `sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_executor.forward_batch_info.PPProxyTensors`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.model_loader.weight_utils.maybe_remap_kv_scale_name`
- `sglang.srt.models.utils.apply_qk_norm`
- `sglang.srt.models.utils.create_fused_set_kv_buffer_arg`
- `sglang.srt.models.utils.enable_fused_set_kv_buffer`
- `sglang.srt.server_args.get_global_server_args`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.is_cuda`
- `sglang.srt.utils.make_layers`
