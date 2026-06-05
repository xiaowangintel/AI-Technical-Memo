# longcat_flash_nextn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/longcat_flash_nextn.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements the longcat flash nextn model stack used by SGLang SRT inference, including network blocks, runtime wrappers, and checkpoint-loading utilities. / 该模块实现 SGLang SRT 推理所使用的 longcat flash nextn 模型栈，包括网络模块、运行时封装以及检查点加载工具。

## Line-by-Line Analysis / 逐行分析

### Lines 33-84: Module imports
```python
import concurrent.futures
import logging
from typing import Iterable, Optional, Tuple

import torch
from torch import nn

from sglang.srt.configs import LongcatFlashConfig
from sglang.srt.eplb.expert_distribution import get_global_expert_distribution_recorder
from sglang.srt.layers import deep_gemm_wrapper
from sglang.srt.layers.communicator import LayerCommunicator, LayerScatterModes
from sglang.srt.layers.dp_attention import (
    get_attention_tp_rank,
    get_attention_tp_size,
    is_dp_attention_enabled,
)
from sglang.srt.layers.layernorm import RMSNorm
from sglang.srt.layers.linear import ReplicatedLinear
from sglang.srt.layers.logits_processor import LogitsProcessor
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.layers.quantization.fp8_kernel import is_fp8_fnuz
from sglang.srt.layers.quantization.fp8_utils import (
    block_quant_dequant,
    block_quant_to_tensor_quant,
    channel_quant_to_tensor_quant,
    normalize_e4m3fn_to_e4m3fnuz,
    requant_weight_ue8m0_inplace,
)
from sglang.srt.layers.quantization.int8_utils import (
    block_dequant as int8_block_dequant,
)
from sglang.srt.layers.vocab_parallel_embedding import (
    ParallelLMHead,
    VocabParallelEmbedding,
)
from sglang.srt.model_executor.forward_batch_info import ForwardBatch
from sglang.srt.model_loader.utils import should_deepgemm_weight_requant_ue8m0
from sglang.srt.model_loader.weight_utils import default_weight_loader
from sglang.srt.models.deepseek_v2 import DeepseekV2AttentionMLA
from sglang.srt.models.longcat_flash import LongcatFlashForCausalLM, LongcatFlashMLP
# ... truncated for brevity ...
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、SGLang 运行时组件。

### Lines 86-86: Top-level assign
```python
_is_hip = is_hip()
```
**EN:** Defines or updates _is_hip, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_hip，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 87-87: Top-level assign
```python
_is_cuda = is_cuda()
```
**EN:** Defines or updates _is_cuda, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_cuda，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 88-88: Top-level assign
```python
_is_npu = is_npu()
```
**EN:** Defines or updates _is_npu, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_npu，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 89-89: Top-level assign
```python
_is_fp8_fnuz = is_fp8_fnuz()
```
**EN:** Defines or updates _is_fp8_fnuz, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_fp8_fnuz，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 90-90: Top-level assign
```python
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip
```
**EN:** Defines or updates _use_aiter, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _use_aiter，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 91-91: Top-level assign
```python
_is_cpu_amx_available = cpu_has_amx_support()
```
**EN:** Defines or updates _is_cpu_amx_available, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_cpu_amx_available，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 92-92: Top-level assign
```python
_is_cpu = is_cpu()
```
**EN:** Defines or updates _is_cpu, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _is_cpu，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 93-93: Top-level assign
```python
_device_sm = get_device_sm()
```
**EN:** Defines or updates _device_sm, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 _device_sm，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 95-104: Top-level if
```python
if _is_cuda:
    from sgl_kernel import awq_dequantize
elif _is_cpu and _is_cpu_amx_available:
    pass
elif _is_hip:
    from sglang.srt.layers.quantization.awq.awq_triton import (
        awq_dequantize_triton as awq_dequantize,
    )
else:
    pass
```
**EN:** Applies conditional top-level logic so the module can adapt to optional dependencies, feature flags, or environment-specific behavior.
**CN:** 应用条件化的顶层逻辑，使模块能够适配可选依赖、功能开关或特定环境行为。

### Lines 107-107: Top-level assign
```python
logger = logging.getLogger(__name__)
```
**EN:** Defines or updates logger, providing module-level configuration, metadata, or cached objects consumed later in the file.
**CN:** 定义或更新 logger，为后续代码提供模块级配置、元数据或缓存对象。

### Lines 110-111: Class `LongcatFlashDenseDecoderLayer` overview
```python
class LongcatFlashDenseDecoderLayer(nn.Module):
```
**EN:** Defines `LongcatFlashDenseDecoderLayer` as a reusable runtime type derived from nn.Module. The class groups 2 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `LongcatFlashDenseDecoderLayer`，其继承关系为 nn.Module。这个类组织了 2 个方法，用于实现模型相关行为。

### Lines 112-170: Method `LongcatFlashDenseDecoderLayer.__init__`
```python
    def __init__(
        self,
        config: LongcatFlashConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        alt_stream: Optional[torch.cuda.Stream] = None,
    ) -> None:
        super().__init__()
        self.config = config
        self.hidden_size = config.hidden_size
        self.layer_id = layer_id
        self.alt_stream = alt_stream

        self.self_attn = DeepseekV2AttentionMLA(
            config=config,
            hidden_size=config.hidden_size,
            num_heads=config.num_attention_heads,
            qk_nope_head_dim=config.qk_nope_head_dim,
            qk_rope_head_dim=config.qk_rope_head_dim,
            v_head_dim=config.v_head_dim,
            q_lora_rank=config.q_lora_rank,
            kv_lora_rank=config.kv_lora_rank,
            rope_theta=config.rope_parameters["rope_theta"],
            rope_scaling=None,
            max_position_embeddings=config.max_position_embeddings,
            quant_config=quant_config,
            layer_id=layer_id,
            reduce_results=False,
            prefix=add_prefix(f"self_attn", prefix),
            alt_stream=self.alt_stream,
        )

        self.mlp = LongcatFlashMLP(
            hidden_size=config.hidden_size,
            intermediate_size=config.intermediate_size,
            hidden_act=config.hidden_act,
            quant_config=quant_config,
            prefix=add_prefix(f"mlps", prefix),
        )
# ... truncated for brevity ...
```
**EN:** This method implements `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., layer_id: ..., quant_config: ...=..., prefix: ...=..., alt_stream: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 172-199: Method `LongcatFlashDenseDecoderLayer.forward`
```python
    def forward(
        self,
        positions: torch.Tensor,
        hidden_states: torch.Tensor,
        forward_batch: ForwardBatch,
        residual: Optional[torch.Tensor],
        zero_allocator: BumpAllocator,
    ) -> torch.Tensor:

        hidden_states, residual = self.layer_communicator.prepare_attn(
            hidden_states, residual, forward_batch
        )
        if hidden_states.shape[0] != 0:
            hidden_states = self.self_attn(
                positions=positions,
                hidden_states=hidden_states,
                forward_batch=forward_batch,
                zero_allocator=zero_allocator,
            )

        hidden_states, residual = self.layer_communicator.prepare_mlp(
            hidden_states, residual, forward_batch
        )
        hidden_states = self.mlp(hidden_states)
        hidden_states, residual = self.layer_communicator.postprocess_layer(
            hidden_states, residual, forward_batch
        )
        return hidden_states, residual
```
**EN:** This method implements `forward(positions: ..., hidden_states: ..., forward_batch: ..., residual: ..., zero_allocator: ...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(positions: ..., hidden_states: ..., forward_batch: ..., residual: ..., zero_allocator: ...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 202-202: Class `LongcatFlashModelNextN` overview
```python
class LongcatFlashModelNextN(nn.Module):
```
**EN:** Defines `LongcatFlashModelNextN` as a reusable runtime type derived from nn.Module. The class groups 3 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `LongcatFlashModelNextN`，其继承关系为 nn.Module。这个类组织了 3 个方法，用于实现模型相关行为。

### Lines 203-234: Method `LongcatFlashModelNextN.__init__`
```python
    def __init__(
        self,
        config: LongcatFlashConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.vocab_size = config.vocab_size
        self.alt_stream = torch.cuda.Stream()

        self.embed_tokens = VocabParallelEmbedding(
            config.vocab_size,
            config.hidden_size,
            use_attn_tp_group=is_dp_attention_enabled(),
            prefix=add_prefix("embed_tokens", prefix),
        )

        self.enorm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.hnorm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)

        self.eh_proj = ReplicatedLinear(
            2 * config.hidden_size,
            config.hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("eh_proj", ""),
        )
        self.decoder = LongcatFlashDenseDecoderLayer(
            config, 0, quant_config=quant_config, alt_stream=self.alt_stream
        )

        self.final_layernorm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 236-237: Method `LongcatFlashModelNextN.get_input_embeddings`
```python
    def get_input_embeddings(self) -> torch.Tensor:
        return self.embed_tokens
```
**EN:** This method implements `get_input_embeddings()` and prepares or accesses embedding representations used by the model.
**CN:** 这个方法实现了 `get_input_embeddings()`，其作用是准备或访问模型使用的嵌入表示。

### Lines 239-280: Method `LongcatFlashModelNextN.forward`
```python
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: torch.Tensor = None,
    ) -> torch.Tensor:
        total_num_layers = 1
        device = input_embeds.device if input_embeds is not None else input_ids.device
        zero_allocator = BumpAllocator(
            buffer_size=total_num_layers * 2 * (2 if forward_batch.can_run_tbo else 1),
            dtype=torch.float32,
            device=device,
        )
        if input_embeds is None:
            hidden_states = self.embed_tokens(input_ids)
        else:
            hidden_states = input_embeds

        if hidden_states.shape[0] > 0:
            hidden_states, _ = self.eh_proj(
                torch.cat(
                    (
                        self.enorm(hidden_states),
                        self.hnorm(forward_batch.spec_info.hidden_states),
                    ),
                    dim=-1,
                )
            )

        residual = None
        with get_global_expert_distribution_recorder().disable_this_region():
            hidden_states, residual = self.decoder(
                positions, hidden_states, forward_batch, residual, zero_allocator
            )

        if not forward_batch.forward_mode.is_idle():
            if residual is not None:
                hidden_states, _ = self.final_layernorm(hidden_states, residual)
            else:
# ... truncated for brevity ...
```
**EN:** This method implements `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=...)` and runs the main forward computation and returns tensors for the next inference stage.
**CN:** 这个方法实现了 `forward(input_ids: ..., positions: ..., forward_batch: ..., input_embeds: ...=...)`，其作用是执行核心前向计算，并返回下一阶段推理所需的张量。

### Lines 283-284: Class `LongcatFlashForCausalLMNextN` overview
```python
class LongcatFlashForCausalLMNextN(LongcatFlashForCausalLM):
```
**EN:** Defines `LongcatFlashForCausalLMNextN` as a reusable runtime type derived from LongcatFlashForCausalLM. The class groups 5 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `LongcatFlashForCausalLMNextN`，其继承关系为 LongcatFlashForCausalLM。这个类组织了 5 个方法，用于实现模型相关行为。

### Lines 285-303: Method `LongcatFlashForCausalLMNextN.__init__`
```python
    def __init__(
        self,
        config: LongcatFlashConfig,
        quant_config: Optional[QuantizationConfig] = None,
    ) -> None:
        nn.Module.__init__(self)
        self.config = config
        self.quant_config = (
            None
            if "mtp" in getattr(config, "disable_quant_module", [])
            else quant_config
        )
        self.model = LongcatFlashModelNextN(config, self.quant_config)
        self.lm_head = ParallelLMHead(
            config.vocab_size,
            config.hidden_size,
            quant_config=self.quant_config,
        )
        self.logits_processor = LogitsProcessor(config)
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 305-315: Method `LongcatFlashForCausalLMNextN.forward`
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

### Lines 317-459: Method `LongcatFlashForCausalLMNextN.post_load_weights`
```python
    def post_load_weights(self):
        self_attn = self.model.decoder.self_attn
        if hasattr(self_attn.kv_b_proj, "qweight"):
            # AWQ compatible
            if _is_cuda or _is_hip:
                w = awq_dequantize(
                    self_attn.kv_b_proj.qweight,
                    self_attn.kv_b_proj.scales,
                    self_attn.kv_b_proj.qzeros,
                ).T
            else:
                w = awq_dequantize(
                    self_attn.kv_b_proj.qweight,
                    self_attn.kv_b_proj.scales,
                    self_attn.kv_b_proj.qzeros,
                    0,
                    0,
                    0,
                ).T
        else:
            w = self_attn.kv_b_proj.weight
        use_deep_gemm_bmm = False
        if w.dtype in (
            torch.float8_e4m3fn,
            torch.float8_e4m3fnuz,
        ):
            if (
                hasattr(self.quant_config, "weight_block_size")
                and self.quant_config.weight_block_size is not None
            ):
                weight_block_size = self.quant_config.weight_block_size
                assert hasattr(self_attn.kv_b_proj, "weight_scale_inv")
                if _is_fp8_fnuz:
                    weight, weight_scale, _ = normalize_e4m3fn_to_e4m3fnuz(
                        weight=w,
                        weight_scale=self_attn.kv_b_proj.weight_scale_inv,
                        input_scale=None,
                    )
                else:
                    weight = w
# ... truncated for brevity ...
```
**EN:** This method implements `post_load_weights()` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `post_load_weights()`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 461-492: Method `LongcatFlashForCausalLMNextN._weight_requant_ue8m0`
```python
    def _weight_requant_ue8m0(self):
        weight_block_size = self.quant_config.weight_block_size
        layer = self.model.decoder
        self_attn = layer.self_attn
        module_list = [
            self_attn.kv_b_proj,
            self_attn.o_proj,
        ]

        if self.config.q_lora_rank is not None:
            module_list.append(self_attn.fused_qkv_a_proj_with_mqa)
            module_list.append(self_attn.q_b_proj)
        else:
            module_list.append(self_attn.kv_a_proj_with_mqa)
            module_list.append(self_attn.q_proj)

        for module in module_list:
            if hasattr(module, "weight_scale_inv"):
                requant_weight_ue8m0_inplace(
                    module.weight, module.weight_scale_inv, weight_block_size
                )

        mlp = layer.mlps
        assert isinstance(mlp, LongcatFlashMLP)
        for module in [
            mlp.gate_up_proj,
            mlp.down_proj,
        ]:
            if hasattr(module, "weight_scale_inv"):
                requant_weight_ue8m0_inplace(
                    module.weight, module.weight_scale_inv, weight_block_size
                )
```
**EN:** This method implements `_weight_requant_ue8m0()` and handles weight mapping, filtering, or loading for this model component.
**CN:** 这个方法实现了 `_weight_requant_ue8m0()`，其作用是处理该模型组件的权重映射、筛选或加载逻辑。

### Lines 494-680: Method `LongcatFlashForCausalLMNextN.load_weights`
```python
    def load_weights(self, weights: Iterable[Tuple[str, torch.Tensor]]):
        stacked_params_mapping = [
            # (param_name, shard_name, shard_id)
            ("gate_up_proj", "gate_proj", 0),
            ("gate_up_proj", "up_proj", 1),
        ]

        # Fuse q_a_proj and kv_a_proj_with_mqa along output dimension when q_lora_rank is not None
        fuse_qkv_a_proj = hasattr(self.config, "q_lora_rank") and (
            self.config.q_lora_rank is not None
        )
        cached_a_proj = {} if fuse_qkv_a_proj else None

        nextn_layer_prefix = "model.layers.0"
        nextn_spec_weight_names = [
            "shared_head.norm",
            "eh_proj",
            "enorm",
            "hnorm",
            "final_layernorm",
        ]

        weight_names_mapping = {
            "model.mtp.embed_tokens.weight": "embed_tokens.weight",
            "model.mtp.layers.0.eh_proj.weight": "eh_proj.weight",
            "model.mtp.layers.0.eh_proj.weight_scale_inv": "eh_proj.weight_scale_inv",
            "model.mtp.layers.0.enorm.m.weight": "enorm.weight",
            "model.mtp.layers.0.hnorm.m.weight": "hnorm.weight",
            "model.mtp.layers.0.input_layernorm.weight": "layers.0.input_layernorm.weight",
            "model.mtp.layers.0.post_attention_layernorm.weight": "layers.0.post_attention_layernorm.weight",
            "model.mtp.layers.0.self_attn.kv_a_layernorm.weight": "layers.0.self_attn.kv_a_layernorm.weight",
            "model.mtp.layers.0.self_attn.kv_a_proj_with_mqa.weight": "layers.0.self_attn.kv_a_proj_with_mqa.weight",
            "model.mtp.layers.0.self_attn.kv_a_proj_with_mqa.weight_scale_inv": "layers.0.self_attn.kv_a_proj_with_mqa.weight_scale_inv",
            "model.mtp.layers.0.self_attn.kv_b_proj.weight": "layers.0.self_attn.kv_b_proj.weight",
            "model.mtp.layers.0.self_attn.kv_b_proj.weight_scale_inv": "layers.0.self_attn.kv_b_proj.weight_scale_inv",
            "model.mtp.layers.0.self_attn.o_proj.weight": "layers.0.self_attn.o_proj.weight",
            "model.mtp.layers.0.self_attn.o_proj.weight_scale_inv": "layers.0.self_attn.o_proj.weight_scale_inv",
            "model.mtp.layers.0.self_attn.q_a_layernorm.weight": "layers.0.self_attn.q_a_layernorm.weight",
            "model.mtp.layers.0.self_attn.q_a_proj.weight": "layers.0.self_attn.q_a_proj.weight",
            "model.mtp.layers.0.self_attn.q_a_proj.weight_scale_inv": "layers.0.self_attn.q_a_proj.weight_scale_inv",
# ... truncated for brevity ...
```
**EN:** This method implements `load_weights(weights: ...)` and loads checkpoint tensors and maps external parameter names into the in-memory module layout.
**CN:** 这个方法实现了 `load_weights(weights: ...)`，其作用是加载检查点张量，并把外部参数名映射到当前模块的内存布局。

### Lines 683-683: Top-level assign
```python
EntryClass = [LongcatFlashForCausalLMNextN]
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Attention execution and tensor shaping / **CN:** 注意力执行与张量整形
- **EN:** Rotary positional encoding / **CN:** 旋转位置编码
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Embedding lookup and token/state projection / **CN:** 嵌入查找与 token/状态投影

## Dependencies / 依赖关系
- `concurrent.futures`
- `logging`
- `typing.Iterable`
- `typing.Optional`
- `typing.Tuple`
- `torch`
- `torch.nn`
- `sglang.srt.configs.LongcatFlashConfig`
- `sglang.srt.eplb.expert_distribution.get_global_expert_distribution_recorder`
- `sglang.srt.layers.deep_gemm_wrapper`
- `sglang.srt.layers.communicator.LayerCommunicator`
- `sglang.srt.layers.communicator.LayerScatterModes`
- `sglang.srt.layers.dp_attention.get_attention_tp_rank`
- `sglang.srt.layers.dp_attention.get_attention_tp_size`
- `sglang.srt.layers.dp_attention.is_dp_attention_enabled`
- `sglang.srt.layers.layernorm.RMSNorm`
- `sglang.srt.layers.linear.ReplicatedLinear`
- `sglang.srt.layers.logits_processor.LogitsProcessor`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.layers.quantization.fp8_kernel.is_fp8_fnuz`
- `sglang.srt.layers.quantization.fp8_utils.block_quant_dequant`
- `sglang.srt.layers.quantization.fp8_utils.block_quant_to_tensor_quant`
- `sglang.srt.layers.quantization.fp8_utils.channel_quant_to_tensor_quant`
- `sglang.srt.layers.quantization.fp8_utils.normalize_e4m3fn_to_e4m3fnuz`
- `sglang.srt.layers.quantization.fp8_utils.requant_weight_ue8m0_inplace`
- `sglang.srt.layers.quantization.int8_utils.block_dequant`
- `sglang.srt.layers.vocab_parallel_embedding.ParallelLMHead`
- `sglang.srt.layers.vocab_parallel_embedding.VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
- `sglang.srt.model_loader.utils.should_deepgemm_weight_requant_ue8m0`
- `sglang.srt.model_loader.weight_utils.default_weight_loader`
- `sglang.srt.models.deepseek_v2.DeepseekV2AttentionMLA`
- `sglang.srt.models.longcat_flash.LongcatFlashForCausalLM`
- `sglang.srt.models.longcat_flash.LongcatFlashMLP`
- `sglang.srt.utils.BumpAllocator`
- `sglang.srt.utils.add_prefix`
- `sglang.srt.utils.bind_or_assign`
- `sglang.srt.utils.cpu_has_amx_support`
- `sglang.srt.utils.get_bool_env_var`
- `sglang.srt.utils.get_device_sm`
- `sglang.srt.utils.is_cpu`
- `sglang.srt.utils.is_cuda`
- `sglang.srt.utils.is_hip`
- `sglang.srt.utils.is_npu`
- `sgl_kernel.awq_dequantize`
- `sglang.srt.layers.quantization.awq.awq_triton.awq_dequantize_triton`
