# qwen3.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/qwen3.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the qwen3 model stack used by SGLang SRT inference, including network blocks, runtime wrappers, and checkpoint-loading utilities. / 该模块实现 SGLang SRT 推理所使用的 qwen3 模型栈，包括网络模块、运行时封装以及检查点加载工具。

## Line-by-Line Analysis / 逐行分析

### Lines 2-34: Module imports
```python
import logging
from typing import Any, Dict, Iterable, List, Optional, Tuple

import torch
from torch import nn

from sglang.srt.distributed import (
    get_pp_group,
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
)
from sglang.srt.layers.communicator import LayerCommunicator, LayerScatterModes
from sglang.srt.layers.dp_attention import get_attention_tp_rank, get_attention_tp_size
from sglang.srt.layers.layernorm import RMSNorm
from sglang.srt.layers.linear import QKVParallelLinear, RowParallelLinear
from sglang.srt.layers.logits_processor import LogitsProcessor
from sglang.srt.layers.pooler import Pooler, PoolingType
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.layers.radix_attention import RadixAttention
from sglang.srt.layers.rotary_embedding import get_rope
from sglang.srt.layers.rotary_embedding.mrope import MRotaryEmbedding
from sglang.srt.layers.utils import PPMissingLayer, get_layer_id
from sglang.srt.layers.vocab_parallel_embedding import ParallelLMHead
from sglang.srt.model_executor.forward_batch_info import ForwardBatch, PPProxyTensors
from sglang.srt.model_loader.weight_utils import (
    default_weight_loader,
    maybe_remap_kv_scale_name,
)
from sglang.srt.models.qwen2 import Qwen2MLP as Qwen3MLP
from sglang.srt.models.qwen2 import Qwen2Model
from sglang.srt.models.utils import apply_qk_norm
from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils import add_prefix, get_bool_env_var, is_cuda, is_hip, is_npu
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、SGLang 运行时组件。

### Lines 36-36: Top-level assign
```python
Qwen3Config = None
```
**EN:** Defines or updates Qwen3Config, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 Qwen3Config，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 38-38: Top-level assign
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines or updates logger, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 logger，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 39-39: Top-level assign
```python
_is_cuda = is_cuda()
```
**EN:** Defines or updates _is_cuda, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_cuda，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 40-40: Top-level assign
```python
_is_hip = is_hip()
```
**EN:** Defines or updates _is_hip, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_hip，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 41-41: Top-level assign
```python
_is_npu = is_npu()
```
**EN:** Defines or updates _is_npu, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_npu，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 42-42: Top-level assign
```python
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip
```
**EN:** Defines or updates _use_aiter, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _use_aiter，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 44-44: Top-level assign
```python
_has_fused_qk_norm_mrope = False
```
**EN:** Defines or updates _has_fused_qk_norm_mrope, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _has_fused_qk_norm_mrope，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 45-52: Top-level if
```python
if _use_aiter:
    try:
        from aiter import fused_qk_norm_mrope_3d_cache_pts_quant_shuffle

        _has_fused_qk_norm_mrope = True
        logger.info("aiter fused_qk_norm_mrope_3d kernel available")
    except ImportError:
        pass
```
**EN:** Applies conditional top-level logic so the module can adapt to optional dependencies, feature flags, or environment-specific behavior.
**CN:** 应用条件化的顶层逻辑，使模块能够适配可选依赖、功能开关或特定环境行为。

### Lines 54-57: Top-level if
```python
if _is_npu:
    from sgl_kernel_npu.norm.split_qkv_rmsnorm_rope import split_qkv_rmsnorm_rope

    from sglang.srt.hardware_backend.npu.cmo import get_cmo_stream, wait_cmo_stream
```
**EN:** Applies conditional top-level logic so the module can adapt to optional dependencies, feature flags, or environment-specific behavior.
**CN:** 应用条件化的顶层逻辑，使模块能够适配可选依赖、功能开关或特定环境行为。

### Lines 60-60: Class `Qwen3Attention` overview
```python
class Qwen3Attention(nn.Module):
```
**EN:** Defines `Qwen3Attention` as a reusable runtime type derived from nn.Module. The class groups 5 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3Attention`，其继承关系为 nn.Module。这个类组织了 5 个方法，用于实现模型相关行为。

### Lines 61-165: Method `Qwen3Attention.__init__`
```python
    def __init__(
        self,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        layer_id: int = 0,
        rope_theta: float = 1000000,
        rope_scaling: Optional[Dict[str, Any]] = None,
        head_dim: Optional[int] = None,
        max_position_embeddings: int = 32768,
        quant_config: Optional[QuantizationConfig] = None,
        rms_norm_eps: float = None,
        attention_bias: bool = False,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size
        self.tp_size = get_tensor_model_parallel_world_size()
        self.total_num_heads = num_heads
        attn_tp_rank = get_attention_tp_rank()
        attn_tp_size = get_attention_tp_size()

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
        self.head_dim = head_dim or hidden_size // self.total_num_heads
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scaling = self.head_dim**-0.5
        self.rope_theta = rope_theta
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(hidden_size: ..., num_heads: ..., num_kv_heads: ..., layer_id: ...=..., rope_theta: ...=..., rope_scaling: ...=..., ...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(hidden_size: ..., num_heads: ..., num_kv_heads: ..., layer_id: ...=..., rope_theta: ...=..., rope_scaling: ...=..., ...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 167-179: Method `Qwen3Attention.forward_prepare_native`
```python
    def forward_prepare_native(self, positions, hidden_states):
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
        q, k = self.rotary_emb(positions, q, k)
        return q, k, v
```
**EN:** This method implements `forward_prepare_native(positions, hidden_states)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward_prepare_native(positions, hidden_states)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 181-199: Method `Qwen3Attention.forward_prepare_npu`
```python
    def forward_prepare_npu(self, positions, hidden_states, forward_batch):
        qkv, _ = self.qkv_proj(hidden_states)

        if self.attn.layer_id == forward_batch.token_to_kv_pool.start_layer:
            self.rotary_emb.get_cos_sin_with_position(positions)
        q, k, v = split_qkv_rmsnorm_rope(
            qkv,
            self.rotary_emb.position_sin,
            self.rotary_emb.position_cos,
            self.q_size,
            self.kv_size,
            self.head_dim,
            eps=self.q_norm.variance_epsilon,
            q_weight=self.q_norm.weight,
            k_weight=self.k_norm.weight,
            q_bias=getattr(self.q_norm, "bias", None),
            k_bias=getattr(self.k_norm, "bias", None),
        )
        return q, k, v
```
**EN:** This method implements `forward_prepare_npu(positions, hidden_states, forward_batch)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward_prepare_npu(positions, hidden_states, forward_batch)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 201-261: Method `Qwen3Attention.forward_prepare_aiter_fused_mrope`
```python
    def forward_prepare_aiter_fused_mrope(
        self, positions, hidden_states, forward_batch
    ):
        """Fused QK-norm + 3D mRoPE + KV cache write for decode (ROCm/aiter).

        The fused HIP kernel replaces split → QK norm → mRoPE → cache write,
        so KV is already in the paged cache when this returns.
        Returns (q, None, None); caller must pass save_kv_cache=False to attn.
        """
        qkv, _ = self.qkv_proj(hidden_states)
        num_tokens = qkv.shape[0]

        qkv_3d = qkv.view(num_tokens, -1, self.head_dim)

        token_to_kv_pool = forward_batch.token_to_kv_pool
        k_cache, v_cache = token_to_kv_pool.get_kv_buffer(self.attn.layer_id)
        slot_mapping = forward_batch.out_cache_loc

        cos_sin = self.rotary_emb.cos_sin_cache
        if cos_sin.dtype != qkv.dtype:
            cos_sin = cos_sin.to(dtype=qkv.dtype)

        q_out = torch.empty(
            num_tokens,
            self.num_heads,
            self.head_dim,
            dtype=qkv.dtype,
            device=qkv.device,
        )

        fused_qk_norm_mrope_3d_cache_pts_quant_shuffle(
            qkv_3d,
            self.q_norm.weight,
            self.k_norm.weight,
            cos_sin,
            positions,
            num_tokens,
            self.num_heads,
            self.num_kv_heads,
            self.num_kv_heads,
# ... truncated for brevity ...
```
**EN:** This method implements `forward_prepare_aiter_fused_mrope(positions, hidden_states, forward_batch)` and Fused QK-norm + 3D mRoPE + KV cache write for decode (ROCm/aiter).
**CN:** 这个方法实现了 `forward_prepare_aiter_fused_mrope(positions, hidden_states, forward_batch)`，其作用是该代码块的文档字符串直接说明了此例程在模型流程中的职责。

### Lines 263-305: Method `Qwen3Attention.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        if get_global_server_args().rl_on_policy_target is not None:
            hidden_states = hidden_states.bfloat16()

        save_kv_cache = True
        use_aiter_fused = (
            self.use_fused_qk_norm_mrope
            and forward_batch.forward_mode.is_decode()
            and get_global_server_args().rl_on_policy_target is None
        )

        if use_aiter_fused:
            q, k, v = self.forward_prepare_aiter_fused_mrope(
                positions, hidden_states, forward_batch
            )
            save_kv_cache = False
        elif (
            not _is_npu
            or forward_batch.forward_mode.is_extend_or_draft_extend_or_mixed()
        ):
            q, k, v = self.forward_prepare_native(
                positions=positions,
                hidden_states=hidden_states,
            )
        else:
            q, k, v = self.forward_prepare_npu(
                positions=positions,
                hidden_states=hidden_states,
                forward_batch=forward_batch,
            )

        if get_global_server_args().rl_on_policy_target is not None:
            q = q.to(torch.bfloat16)
            k = k.to(torch.bfloat16)

# ... truncated for brevity ...
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 308-308: Class `Qwen3DecoderLayer` overview
```python
class Qwen3DecoderLayer(nn.Module):
```
**EN:** Defines `Qwen3DecoderLayer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3DecoderLayer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 309-382: Method `Qwen3DecoderLayer.__init__`
```python
    def __init__(
        self,
        config: Qwen3Config,
        layer_id: int = 0,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
    ) -> None:
        super().__init__()
        self.hidden_size = config.hidden_size
        if (
            hasattr(config, "rope_parameters")
            and config.rope_parameters
            and "rope_theta" in config.rope_parameters
        ):
            rope_theta = config.rope_parameters["rope_theta"]
            rope_scaling = config.rope_parameters
        else:
            rope_theta = getattr(config, "rope_theta", 1000000)
            rope_scaling = getattr(config, "rope_scaling", None)
        max_position_embeddings = getattr(config, "max_position_embeddings", 32768)
        head_dim = getattr(config, "head_dim", None)
        self.self_attn = Qwen3Attention(
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            num_kv_heads=config.num_key_value_heads,
            layer_id=layer_id,
            rope_theta=rope_theta,
            rope_scaling=rope_scaling,
            head_dim=head_dim,
            max_position_embeddings=max_position_embeddings,
            quant_config=quant_config,
            rms_norm_eps=config.rms_norm_eps,
            attention_bias=config.attention_bias,
            prefix=add_prefix("self_attn", prefix),
            alt_stream=alt_stream,
        )
        self.mlp = Qwen3MLP(
            hidden_size=self.hidden_size,
            intermediate_size=config.intermediate_size,
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., layer_id: ...=..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ...=..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 384-428: Method `Qwen3DecoderLayer.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
        residual: Optional[torch.Tensor],
        post_residual_addition: Optional[torch.Tensor] = None,
    ) -> Tuple[torch.Tensor, torch.Tensor]:
        # Self Attention
        hidden_states, residual = self.layer_communicator.prepare_attn(
            hidden_states,
            residual,
            forward_batch,
            post_residual_addition=post_residual_addition,
        )
        if hidden_states.shape[0] != 0:
            hidden_states = self.self_attn(
                positions=positions,
                hidden_states=hidden_states,
                forward_batch=forward_batch,
            )

        # Fully Connected
        hidden_states, residual = self.layer_communicator.prepare_mlp(
            hidden_states,
            residual,
            forward_batch,
            cache=(
                [self.mlp.gate_up_proj.weight, self.mlp.down_proj.weight]
                if _is_npu
                and not get_global_server_args().disable_piecewise_cuda_graph
                and (
                    hasattr(self.mlp.gate_up_proj, "weight")
                    and hasattr(self.mlp.down_proj, "weight")
                )
                else None
            ),
        )
        hidden_states = self.mlp(hidden_states, forward_batch=forward_batch)
        if _is_npu and get_cmo_stream():
# ... truncated for brevity ...
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ..., residual: ..., post_residual_addition: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ..., residual: ..., post_residual_addition: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 431-431: Class `Qwen3Model` overview
```python
class Qwen3Model(Qwen2Model):
```
**EN:** Defines `Qwen3Model` as a reusable runtime type derived from Qwen2Model. The class groups 1 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3Model`，其继承关系为 Qwen2Model。这个类组织了 1 个方法，用于实现模型相关行为。

### Lines 432-445: Method `Qwen3Model.__init__`
```python
    def __init__(
        self,
        config: Qwen3Config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        alt_stream = torch.cuda.Stream() if _is_cuda else None
        super().__init__(
            config=config,
            quant_config=quant_config,
            prefix=prefix,
            decoder_layer_type=Qwen3DecoderLayer,
            alt_stream=alt_stream,
        )
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 448-449: Class `Qwen3ForCausalLM` overview
```python
class Qwen3ForCausalLM(nn.Module):
    # BitandBytes specific attributes
```
**EN:** Defines `Qwen3ForCausalLM` as a reusable runtime type derived from nn.Module. The class groups 12 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3ForCausalLM`，其继承关系为 nn.Module。这个类组织了 12 个方法，用于实现模型相关行为。

### Lines 450-466: Class `Qwen3ForCausalLM` attributes
```python
    default_bitsandbytes_target_modules = [
        ".gate_proj.",
        ".down_proj.",
        ".up_proj.",
        ".q_proj.",
        ".k_proj.",
        ".v_proj.",
        ".o_proj.",
    ]
    bitsandbytes_stacked_params_mapping = {
        # shard_name, weight_name, index
        "q_proj": ("qkv_proj", 0),
        "k_proj": ("qkv_proj", 1),
        "v_proj": ("qkv_proj", 2),
        "gate_proj": ("gate_up_proj", 0),
        "up_proj": ("gate_up_proj", 1),
    }
```
**EN:** Defines class-level attributes and metadata that shape how `Qwen3ForCausalLM` behaves at runtime.
**CN:** 定义类级属性与元数据，用于决定 `Qwen3ForCausalLM` 在运行时的行为。

### Lines 468-502: Method `Qwen3ForCausalLM.__init__`
```python
    def __init__(
        self,
        config: Qwen3Config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.pp_group = get_pp_group()
        self.config = config
        self.quant_config = quant_config
        self.model = Qwen3Model(
            config, quant_config=quant_config, prefix=add_prefix("model", prefix)
        )

        # handle the lm head on different pp ranks
        if self.pp_group.is_last_rank:
            if self.pp_group.world_size == 1 and config.tie_word_embeddings:
                self.lm_head = self.model.embed_tokens
            else:
                self.lm_head = ParallelLMHead(
                    config.vocab_size,
                    config.hidden_size,
                    quant_config=quant_config,
                    use_attn_tp_group=get_global_server_args().enable_dp_lm_head,
                    prefix=add_prefix("lm_head", prefix),
                )
        else:
            # ranks other than the last rank will have a placeholder layer
            self.lm_head = PPMissingLayer()

        self.logits_processor = LogitsProcessor(config)
        self.pooler = Pooler(pooling_type=PoolingType.LAST, normalize=True)

        # For EAGLE3 support
        self.capture_aux_hidden_states = False
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 504-505: Method `Qwen3ForCausalLM.get_input_embeddings`
```python
    def get_input_embeddings(self) -> nn.Embedding:
        return self.model.get_input_embeddings()
```
**EN:** This method implements `get_input_embeddings()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 507-541: Method `Qwen3ForCausalLM.forward`
```python
    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: torch.Tensor = None,
        get_embedding: bool = False,
        pp_proxy_tensors: Optional[PPProxyTensors] = None,
    ) -> torch.Tensor:
        hidden_states = self.model(
            input_ids,
            positions,
            forward_batch,
            input_embeds,
            pp_proxy_tensors=pp_proxy_tensors,
        )

        aux_hidden_states = None
        if self.capture_aux_hidden_states:
            hidden_states, aux_hidden_states = hidden_states

        if self.pp_group.is_last_rank:
            if not get_embedding:
                return self.logits_processor(
                    input_ids,
                    hidden_states,
                    self.lm_head,
                    forward_batch,
                    aux_hidden_states,
                )
            else:
                return self.pooler(hidden_states, forward_batch)
        else:
            return hidden_states
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., get_embedding: ...=..., pp_proxy_tensors: ...=...)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=..., get_embedding: ...=..., pp_proxy_tensors: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 543-582: Method `Qwen3ForCausalLM.forward_split_prefill`
```python
    @torch.no_grad()
    def forward_split_prefill(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        split_interval: Tuple[int, int],  # [start, end) 0-based
        input_embeds: torch.Tensor = None,
    ):
        start, end = split_interval
        # embed
        if start == 0:
            if input_embeds is None:
                forward_batch.hidden_states = self.model.embed_tokens(input_ids)
            else:
                forward_batch.hidden_states = input_embeds
        # decoder layer
        for i in range(start, end):
            layer = self.model.layers[i]
            forward_batch.hidden_states, forward_batch.residual = layer(
                positions,
                forward_batch.hidden_states,
                forward_batch,
                forward_batch.residual,
            )

        if end == self.model.config.num_hidden_layers:
            # norm
            hidden_states, _ = self.model.norm(
                forward_batch.hidden_states, forward_batch.residual
            )
            forward_batch.hidden_states = hidden_states
            # logits process
            result = self.logits_processor(
                input_ids, forward_batch.hidden_states, self.lm_head, forward_batch
            )
        else:
            result = None

        return result
```
**EN:** This method implements `forward_split_prefill(input_ids: ..., positions: ..., forward_batch: ..., split_interval: ..., input_embeds: ...=...)` and runs the main forward computation and returns tensors for the next inference stage Decorators: torch.no_grad().
**CN:** 这个方法实现了 `forward_split_prefill(input_ids: ..., positions: ..., forward_batch: ..., split_interval: ..., input_embeds: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量 装饰器：torch.no_grad()。

### Lines 584-586: Method `Qwen3ForCausalLM.start_layer`
```python
    @property
    def start_layer(self):
        return self.model.start_layer
```
**EN:** This method implements `start_layer()` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3ForCausalLM` Decorators: property.
**CN:** 这个方法实现了 `start_layer()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3ForCausalLM` 内部调用 装饰器：property。

### Lines 588-590: Method `Qwen3ForCausalLM.end_layer`
```python
    @property
    def end_layer(self):
        return self.model.end_layer
```
**EN:** This method implements `end_layer()` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3ForCausalLM` Decorators: property.
**CN:** 这个方法实现了 `end_layer()`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3ForCausalLM` 内部调用 装饰器：property。

### Lines 592-666: Method `Qwen3ForCausalLM.load_weights`
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
                    if "lm_head.weight" in params_dict:
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
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 668-669: Method `Qwen3ForCausalLM.get_embed_and_head`
```python
    def get_embed_and_head(self):
        return self.model.embed_tokens.weight, self.lm_head.weight
```
**EN:** This method implements `get_embed_and_head()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_embed_and_head()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 671-677: Method `Qwen3ForCausalLM.set_embed_and_head`
```python
    def set_embed_and_head(self, embed, head):
        del self.model.embed_tokens.weight
        del self.lm_head.weight
        self.model.embed_tokens.weight = embed
        self.lm_head.weight = head
        torch.cuda.empty_cache()
        torch.cuda.synchronize()
```
**EN:** This method implements `set_embed_and_head(embed, head)` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `set_embed_and_head(embed, head)`，其作用是准备或访问模型使用的嵌入表示。

### Lines 679-680: Method `Qwen3ForCausalLM.load_kv_cache_scales`
```python
    def load_kv_cache_scales(self, quantization_param_path: str) -> None:
        self.model.load_kv_cache_scales(quantization_param_path)
```
**EN:** This method implements `load_kv_cache_scales(quantization_param_path: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3ForCausalLM`.
**CN:** 这个方法实现了 `load_kv_cache_scales(quantization_param_path: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3ForCausalLM` 内部调用。

### Lines 682-695: Method `Qwen3ForCausalLM.set_eagle3_layers_to_capture`
```python
    def set_eagle3_layers_to_capture(self, layer_ids: Optional[List[int]] = None):
        if not self.pp_group.is_last_rank:
            return

        self.capture_aux_hidden_states = True
        if layer_ids is None:
            num_layers = self.config.num_hidden_layers
            self.model.layers_to_capture = [
                2,
                num_layers // 2,
                num_layers - 3,
            ]  # Specific layers for EAGLE3 support
        else:
            self.model.layers_to_capture = [val + 1 for val in layer_ids]
```
**EN:** This method implements `set_eagle3_layers_to_capture(layer_ids: ...=...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3ForCausalLM`.
**CN:** 这个方法实现了 `set_eagle3_layers_to_capture(layer_ids: ...=...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3ForCausalLM` 内部调用。

### Lines 697-709: Method `Qwen3ForCausalLM.set_dflash_layers_to_capture`
```python
    def set_dflash_layers_to_capture(self, layer_ids: List[int]):
        if not self.pp_group.is_last_rank:
            return

        if layer_ids is None:
            raise ValueError(
                "DFLASH requires explicit layer_ids for aux hidden capture."
            )

        self.capture_aux_hidden_states = True
        # SGLang captures "before layer i". To capture the hidden state after target
        # layer `k` (HF-style), we capture before layer `k + 1`.
        self.model.layers_to_capture = [val + 1 for val in layer_ids]
```
**EN:** This method implements `set_dflash_layers_to_capture(layer_ids: ...)` and implements a focused helper that supports the surrounding runtime flow inside `Qwen3ForCausalLM`.
**CN:** 这个方法实现了 `set_dflash_layers_to_capture(layer_ids: ...)`，其作用是实现一个支撑周边运行时流程的辅助逻辑，供 `Qwen3ForCausalLM` 内部调用。

### Lines 712-712: Top-level assign
```python
EntryClass = Qwen3ForCausalLM
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
- `typing.Any`
- `typing.Dict`
- `typing.Iterable`
- `typing.List`
- `typing.Optional`
- `typing.Tuple`
- `torch`
- `torch.nn`
- `sglang.srt.distributed.get_pp_group`
- `sglang.srt.distributed.get_tensor_model_parallel_rank`
- `sglang.srt.distributed.get_tensor_model_parallel_world_size`
- `sglang.srt.layers.communicator.LayerCommunicator`
- `sglang.srt.layers.communicator.LayerScatterModes`
- `sglang.srt.layers.dp_attention.get_attention_tp_rank`
- `sglang.srt.layers.dp_attention.get_attention_tp_size`
- `sglang.srt.layers.layernorm.RMSNorm`
- `sglang.srt.layers.linear.QKVParallelLinear`
- `sglang.srt.layers.linear.RowParallelLinear`
- `sglang.srt.layers.logits_processor.LogitsProcessor`
- `sglang.srt.layers.pooler.Pooler`
- `sglang.srt.layers.pooler.PoolingType`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.layers.radix_attention.RadixAttention`
- `sglang.srt.layers.rotary_embedding.get_rope`
- `sglang.srt.layers.rotary_embedding.mrope.MRotaryEmbedding`
- `sglang.srt.layers.utils.PPMissingLayer`
- `sglang.srt.layers.utils.get_layer_id`
- `sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_executor.forward_batch_info.PPProxyTensors`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.model_loader.weight_utils.maybe_remap_kv_scale_name`
- `sglang.srt.models.qwen2.Qwen2MLP`
- `sglang.srt.models.qwen2.Qwen2Model`
- `sglang.srt.models.utils.apply_qk_norm`
- `sglang.srt.server_args.get_global_server_args`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.get_bool_env_var`
- `sglang.srt.utils.is_cuda`
- `sglang.srt.utils.is_hip`
- `sglang.srt.utils.is_npu`
- `sgl_kernel_npu.norm.split_qkv_rmsnorm_rope.split_qkv_rmsnorm_rope`
- `sglang.srt.hardware_backend.npu.cmo.get_cmo_stream`
- `sglang.srt.hardware_backend.npu.cmo.wait_cmo_stream`
- `aiter.fused_qk_norm_mrope_3d_cache_pts_quant_shuffle`
