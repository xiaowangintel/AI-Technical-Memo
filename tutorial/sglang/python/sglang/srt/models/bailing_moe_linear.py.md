# bailing_moe_linear.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/bailing_moe_linear.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Bailing Moe Linear architecture into SGLang's serving runtime, typically defining layers, model wrappers, and weight-loading behavior for inference. / 该模块将 Bailing Moe Linear 架构接入 SGLang 服务运行时，通常定义推理所需的层、模型封装以及权重加载逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2: Module header and imports / 模块头与导入
```python
# coding=utf-8
# Copyright 2023 Antgroup and The HuggingFace Inc. team. All rights reserved.
```
**EN:** This opening block establishes the file header, module documentation, and shared imports that the rest of the implementation builds upon.
**CN:** 该起始代码块建立了文件头、模块文档以及共享导入，为后续实现提供基础。

### Lines 83-83: Assignment: _is_hip / 赋值：_is_hip
```python
_is_hip = is_hip()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 84-84: Assignment: _is_cuda / 赋值：_is_cuda
```python
_is_cuda = is_cuda()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 85-85: Assignment: _is_npu / 赋值：_is_npu
```python
_is_npu = is_npu()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 86-86: Assignment: _is_fp8_fnuz / 赋值：_is_fp8_fnuz
```python
_is_fp8_fnuz = is_fp8_fnuz()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 87-87: Assignment: _use_aiter / 赋值：_use_aiter
```python
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 88-88: Assignment: _is_cpu_amx_available / 赋值：_is_cpu_amx_available
```python
_is_cpu_amx_available = cpu_has_amx_support()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 89-89: Assignment: _is_cpu / 赋值：_is_cpu
```python
_is_cpu = is_cpu()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 90-90: Assignment: _device_sm / 赋值：_device_sm
```python
_device_sm = get_device_sm()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 91-91: Assignment: _is_gfx95_supported / 赋值：_is_gfx95_supported
```python
_is_gfx95_supported = is_gfx95_supported()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 93-93: Assignment: _use_aiter_gfx95 / 赋值：_use_aiter_gfx95
```python
_use_aiter_gfx95 = _use_aiter and _is_gfx95_supported
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 95-96: Conditional setup: _use_aiter_gfx95 / 条件初始化：_use_aiter_gfx95
```python
if _use_aiter_gfx95:
    pass
```
**EN:** This conditional block enables backend-specific setup so the module can adapt to available hardware or runtime features.
**CN:** 该条件块启用特定硬件/后端的初始化逻辑，使模块能适配不同运行环境。

### Lines 98-107: Conditional setup: _is_cuda / 条件初始化：_is_cuda
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
    from vllm._custom_ops import awq_dequantize
```
**EN:** This conditional block enables backend-specific setup so the module can adapt to available hardware or runtime features.
**CN:** 该条件块启用特定硬件/后端的初始化逻辑，使模块能适配不同运行环境。

### Lines 109-110: Conditional setup: _is_hip / 条件初始化：_is_hip
```python
if _is_hip:
    pass
```
**EN:** This conditional block enables backend-specific setup so the module can adapt to available hardware or runtime features.
**CN:** 该条件块启用特定硬件/后端的初始化逻辑，使模块能适配不同运行环境。

### Lines 112-112: Assignment: _is_flashinfer_available / 赋值：_is_flashinfer_available
```python
_is_flashinfer_available = is_flashinfer_available()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 113-113: Assignment: _is_sm100_supported / 赋值：_is_sm100_supported
```python
_is_sm100_supported = is_cuda() and is_sm100_supported()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 116-120: Class: DsV3MLA / 类：DsV3MLA
```python
class DsV3MLA(DeepseekV2AttentionMLA):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        if kwargs["rope_scaling"]:
            self.rotary_emb.forward = self.rotary_emb.forward_cuda
```
**EN:** This class defines Ds V3 M L A inheriting from DeepseekV2AttentionMLA, grouping state and methods for this model component.
**CN:** 该类定义了 Ds V3 M L A，用于封装该模型组件的状态与方法。

### Lines 123-123: Assignment: LoraConfig / 赋值：LoraConfig
```python
LoraConfig = None
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 124-124: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 125-125: Assignment: _is_cpu / 赋值：_is_cpu
```python
_is_cpu = is_cpu()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 128-134: Function: is_linear_layer() / 函数：is_linear_layer()
```python
def is_linear_layer(layer_idx, layer_group_size):
    if layer_idx is None:
        return False
    if layer_group_size > 0:
        return (layer_idx + 1) % layer_group_size != 0
    else:
        return False
```
**EN:** This function implements is linear layer for the surrounding model/runtime logic. Key parameters include layer_idx, layer_group_size.
**CN:** 该函数实现了 is linear layer 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 137-143: Function: is_pp_missing_parameter() / 函数：is_pp_missing_parameter()
```python
def is_pp_missing_parameter(
    name: str,
    model: torch.nn.Module,
) -> bool:
    if isinstance(model, PPMissingLayer):
        return True
    return False
```
**EN:** This function implements is pp missing parameter for the surrounding model/runtime logic. Key parameters include name, model.
**CN:** 该函数实现了 is pp missing parameter 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 146-161: Function: weight_loader_with_alias() / 函数：weight_loader_with_alias()
```python
def weight_loader_with_alias(alias: str):
    def wrapper(func: Callable):
        def inner_func(
            param: torch.Tensor,
            loaded_weight: torch.Tensor,
            *args,
            prefix: str = None,
            **kwargs,
        ):
            # pf = "[vLLM][load]" + " " if prefix is None else f"[{prefix}] "
            value = func(param, loaded_weight, *args, **kwargs)
            return value

        return inner_func

    return wrapper
```
**EN:** This function loads or remaps data so the runtime can consume checkpoints or cached tensors. Key parameters include alias.
**CN:** 该函数负责加载、重映射或整理数据，使运行时能够正确消费检查点或缓存张量。

### Lines 164-204: Class: BailingMLP / 类：BailingMLP
```python
class BailingMLP(nn.Module):

    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        reduce_results=True,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.gate_up_proj = MergedColumnParallelLinear(
            hidden_size,
            [intermediate_size] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=f"{prefix}.gate_up_proj",
        )
        self.down_proj = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=False,
            quant_config=quant_config,
            reduce_results=reduce_results,
            prefix=f"{prefix}.down_proj",
        )
        self.act_fn = SiluAndMul()

    def forward(
        self,
        x,
        should_allreduce_fusion: bool = False,
        use_reduce_scatter: bool = False,
    ):
        x, _ = self.gate_up_proj(x)
        x = self.act_fn(x)
        x, _ = self.down_proj(
            x,
            skip_all_reduce=use_reduce_scatter or should_allreduce_fusion,
        )
# ... truncated for brevity ...
```
**EN:** This class defines Bailing M L P inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Bailing M L P，用于封装该模型组件的状态与方法。

### Lines 207-235: Class: BailingMoEGate / 类：BailingMoEGate
```python
class BailingMoEGate(nn.Module):
    def __init__(
        self,
        config,
        params_dtype: Optional[torch.dtype] = None,
        prefix: str = "",
    ):
        super().__init__()
        if params_dtype is None:
            params_dtype = torch.get_default_dtype()
        self.params_dtype = params_dtype
        self.weight = nn.Parameter(
            torch.empty(
                (config.num_experts, config.hidden_size),
                dtype=self.params_dtype,
            ),
        )
        if getattr(config, "moe_router_enable_expert_bias", False):
            self.expert_bias = nn.Parameter(
                torch.empty((config.num_experts,), dtype=torch.float32),
            )
        else:
            self.expert_bias = None

    def forward(self, hidden_states):
        logits = F.linear(hidden_states.to(self.weight.dtype), self.weight, None).to(
            hidden_states.dtype
        )
        return logits
```
**EN:** This class defines Bailing Mo E Gate inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Bailing Mo E Gate，用于封装该模型组件的状态与方法。

### Lines 238-357: Class: BailingMoE / 类：BailingMoE
```python
class BailingMoE(nn.Module):

    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        layer_id: int = 0,
        prefix: str = "moe",
    ):
        super().__init__()

        self.layer_id = layer_id

        self.tp_size = get_tensor_model_parallel_world_size()
        self.tp_rank = get_tensor_model_parallel_rank()

        self.top_k = config.num_experts_per_tok
        self.norm_expert_prob = getattr(config, "norm_topk_prob", False)
        self.hidden_size = config.hidden_size
        self.intermediate_size = config.moe_intermediate_size
        self.num_shared_experts = getattr(config, "num_shared_experts", 0)
        self.routed_scaling_factor = getattr(config, "routed_scaling_factor", 1.0)
        self.score_function = getattr(config, "score_function", None)

        # Gate always runs at half / full precision for now.
        router_dtype = getattr(config, "router_dtype", None)
        if router_dtype is None:
            self.router_dtype = torch.float32
        elif router_dtype == "fp32":
            self.router_dtype = torch.float32
        else:
            self.router_dtype = torch.bfloat16

        # check group topk
        self.num_expert_group = getattr(config, "n_group", 0)
        self.topk_group = getattr(config, "topk_group", 0)
        if self.num_expert_group > 0 or self.topk_group > 0:
            assert (
                self.num_expert_group > 0
                and 0 < self.topk_group <= self.num_expert_group
# ... truncated for brevity ...
```
**EN:** This class defines Bailing Mo E inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Bailing Mo E，用于封装该模型组件的状态与方法。

### Lines 360-391: Class: BailingGroupRMSNormGate / 类：BailingGroupRMSNormGate
```python
class BailingGroupRMSNormGate(RMSNormGated):
    def __init__(
        self,
        hidden_size,
        eps=1e-5,
        group_size=None,
        norm_before_gate=True,
        device=None,
        dtype=None,
    ):
        super().__init__(
            hidden_size,
            eps=eps,
            group_size=group_size,
            norm_before_gate=norm_before_gate,
            device=device,
            dtype=dtype,
            activation="sigmoid",
        )
        self.weight.weight_loader = self.weight_loader

    @staticmethod
    def weight_loader(
        param: torch.nn.Parameter,
        loaded_weight: torch.Tensor,
    ) -> None:
        tp_size = get_attention_tp_size()
        tp_rank = get_attention_tp_rank()
        shard_size = loaded_weight.shape[0] // tp_size
        shard = slice(tp_rank * shard_size, (tp_rank + 1) * shard_size)
        param.data.copy_(loaded_weight[shard].contiguous())
        return
```
**EN:** This class defines Bailing Group R M S Norm Gate inheriting from RMSNormGated, grouping state and methods for this model component.
**CN:** 该类定义了 Bailing Group R M S Norm Gate，用于封装该模型组件的状态与方法。

### Lines 394-591: Class: BailingMoELinearAttention / 类：BailingMoELinearAttention
```python
class BailingMoELinearAttention(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        layer_id: int = 0,
        prefix: str = "linear_attn",
    ):
        super().__init__()

        self.layer_id = layer_id
        self.hidden_size = config.hidden_size
        self.total_num_heads = config.num_attention_heads
        self.total_kv_heads = config.num_attention_heads  # MHA

        self.head_dim = getattr(config, "head_dim", None)
        if self.head_dim is None:
            self.head_dim = config.hidden_size // self.total_num_heads

        self.hidden_inner_size = self.head_dim * self.total_num_heads
        self.scaling = self.head_dim**-0.5
        self.tp_size = get_attention_tp_size()
        self.tp_rank = get_attention_tp_rank()

        assert self.total_num_heads % self.tp_size == 0
        self.tp_heads = self.total_num_heads // self.tp_size

        self.max_position_embeddings = config.max_position_embeddings
        self.rope_theta = getattr(config, "rope_theta", 600000)

        self.tp_kv_heads = self.total_kv_heads // self.tp_size
        self.q_size_per_rank = self.head_dim * self.tp_heads
        self.kv_size_per_rank = self.head_dim * self.tp_kv_heads

        self.use_qk_norm = getattr(config, "use_qk_norm", False)
        # minimax / seg_la / fla
        # TODO support fla
        self.linear_backend = getattr(config, "linear_backend", "seg_la")
        logger.debug(f"linear_backend in bailing_moe_linear: {self.linear_backend}")
        self.linear_scale = True if self.linear_backend == "minimax" else False
# ... truncated for brevity ...
```
**EN:** This class defines Bailing Mo E Linear Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Bailing Mo E Linear Attention，用于封装该模型组件的状态与方法。

### Lines 594-700: Class: BailingMoEAttention / 类：BailingMoEAttention
```python
class BailingMoEAttention(nn.Module):

    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        layer_id: int = None,
        prefix: str = "mha",
    ) -> None:
        super().__init__()
        self.layer_id = layer_id

        self.hidden_size = config.hidden_size
        tp_size = get_attention_tp_size()
        self.total_num_heads = config.num_attention_heads
        assert self.total_num_heads % tp_size == 0
        self.num_heads = self.total_num_heads // tp_size
        self.total_num_kv_heads = config.num_key_value_heads
        if self.total_num_kv_heads >= tp_size:
            assert self.total_num_kv_heads % tp_size == 0
        else:
            assert tp_size % self.total_num_kv_heads == 0
        self.num_kv_heads = max(1, self.total_num_kv_heads // tp_size)
        self.head_dim = getattr(config, "head_dim", None)
        if self.head_dim is None:
            self.head_dim = self.hidden_size // self.total_num_heads

        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scaling = self.head_dim**-0.5

        self.split_qkv = getattr(config, "using_split_qkv_in_self_attention", False)
        assert not self.split_qkv, "split_qkv is not supported for now"
        self.use_qk_norm = getattr(config, "use_qk_norm", False)

        self.query_key_value = QKVParallelLinear(
            self.hidden_size,
            self.head_dim,
            self.total_num_heads,
            self.total_num_kv_heads,
# ... truncated for brevity ...
```
**EN:** This class defines Bailing Mo E Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Bailing Mo E Attention，用于封装该模型组件的状态与方法。

### Lines 703-881: Class: BailingMoELinearDecoderLayer / 类：BailingMoELinearDecoderLayer
```python
class BailingMoELinearDecoderLayer(nn.Module):

    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        layer_id: int = 0,
        prefix: str = "layer",
        is_nextn: bool = False,
    ) -> None:
        super().__init__()
        self.layer_id = layer_id
        self.use_mla = getattr(config, "full_attention_type", "mla") == "mla"
        alt_stream = None  # tptest
        # todo nextn

        if config.attention_type == 0:  # Linear layer
            self.attention = BailingMoELinearAttention(
                config,
                quant_config=quant_config,
                layer_id=self.layer_id,
                prefix=prefix + ".attention",
            )
        elif config.attention_type == 1:  # softmax layer
            if self.use_mla:
                self.attention = DsV3MLA(
                    config=config,
                    hidden_size=config.hidden_size,
                    num_heads=config.num_attention_heads,
                    qk_nope_head_dim=config.qk_nope_head_dim,
                    qk_rope_head_dim=config.qk_rope_head_dim,
                    v_head_dim=config.v_head_dim,
                    q_lora_rank=(
                        config.q_lora_rank if hasattr(config, "q_lora_rank") else None
                    ),
                    kv_lora_rank=config.kv_lora_rank,
                    rope_theta=getattr(config, "rope_theta", 600000),
                    rope_scaling=config.rope_scaling,
                    max_position_embeddings=262144,
                    quant_config=quant_config,
# ... truncated for brevity ...
```
**EN:** This class defines Bailing Mo E Linear Decoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Bailing Mo E Linear Decoder Layer，用于封装该模型组件的状态与方法。

### Lines 884-999: Class: BailingMoELinearModel / 类：BailingMoELinearModel
```python
class BailingMoELinearModel(nn.Module):

    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.pp_group = get_pp_group()
        self.config = config
        self.vocab_size = config.vocab_size
        self.embed_dim = config.hidden_size
        self.num_layers = config.num_hidden_layers

        self.layer_group_size = getattr(config, "layer_group_size", 1)
        self.decoder_attention_types = [
            0 if is_linear_layer(i, self.layer_group_size) else 1
            for i in range(self.num_layers)
        ]
        num_linear = sum(1 for t in self.decoder_attention_types if t == 0)
        num_full = sum(1 for t in self.decoder_attention_types if t == 1)
        rank0_log(
            f"Layer config: {num_linear} linear attention layers, {num_full} full attention layers"
        )

        assert (
            self.num_layers % self.layer_group_size == 0
        ), f"num_layers={self.num_layers} must be divided by layer_group_size={self.layer_group_size}"

        if self.pp_group.is_first_rank:
            self.word_embeddings = VocabParallelEmbedding(
                self.vocab_size,
                self.embed_dim,
                enable_tp=not is_dp_attention_enabled(),
                org_num_embeddings=self.vocab_size,
            )
        else:
            self.word_embeddings = PPMissingLayer()

# ... truncated for brevity ...
```
**EN:** This class defines Bailing Mo E Linear Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Bailing Mo E Linear Model，用于封装该模型组件的状态与方法。

### Lines 1002-1557: Class: BailingMoELinearForCausalLM / 类：BailingMoELinearForCausalLM
```python
class BailingMoELinearForCausalLM(nn.Module):

    packed_modules_mapping = {
        "fused_qkv_a_proj_with_mqa": ["q_a_proj", "kv_a_proj_with_mqa"],
        "gate_up_proj": ["gate_proj", "up_proj"],
    }
    # To ensure correct weight loading and mapping.
    hf_to_sglang_mapper = WeightsMapper(
        orig_to_new_substr={
            "attention.dense": "attention.out_proj",
            "layers.7.attention.out_proj": "layers.7.attention.o_proj",
            "layers.15.attention.out_proj": "layers.15.attention.o_proj",
            "layers.23.attention.out_proj": "layers.23.attention.o_proj",
            "layers.31.attention.out_proj": "layers.31.attention.o_proj",
            "layers.39.attention.out_proj": "layers.39.attention.o_proj",
            "layers.47.attention.out_proj": "layers.47.attention.o_proj",
            "layers.55.attention.out_proj": "layers.55.attention.o_proj",
            "layers.63.attention.out_proj": "layers.63.attention.o_proj",
            "layers.71.attention.out_proj": "layers.71.attention.o_proj",
            "layers.79.attention.out_proj": "layers.79.attention.o_proj",
            "attention.query_key_value": "attention.qkv_proj",
            "attention.g_proj": "attention.output_gate",
        },
    )

    def __init__(
        self,
        *,
        config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.pp_group = get_pp_group()
        self.config = config
        self.quant_config = quant_config
        self.model = BailingMoELinearModel(
            self.config, quant_config, prefix=add_prefix("model", prefix)
        )

# ... truncated for brevity ...
```
**EN:** This class defines Bailing Mo E Linear For Causal L M inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Bailing Mo E Linear For Causal L M，用于封装该模型组件的状态与方法。

### Lines 1560-1561: Class: BailingMoeV2_5ForCausalLM / 类：BailingMoeV2_5ForCausalLM
```python
class BailingMoeV2_5ForCausalLM(BailingMoELinearForCausalLM):
    pass
```
**EN:** This class defines Bailing Moe V2 5 For Causal L M inheriting from BailingMoELinearForCausalLM, grouping state and methods for this model component.
**CN:** 该类定义了 Bailing Moe V2 5 For Causal L M，用于封装该模型组件的状态与方法。

### Lines 1564-1566: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = [
    BailingMoeV2_5ForCausalLM,
]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `copy`
- `logging`
- `typing: Callable, Iterable, Optional, Set, Tuple, Union`
- `torch`
- `torch.nn.functional`
- `torch: nn`
- `transformers: PretrainedConfig`
- `sglang.srt.distributed: get_pp_group, get_tensor_model_parallel_rank, get_tensor_model_parallel_world_size, tensor_model_parallel_all_reduce`
- `sglang.srt.eplb.expert_distribution: get_global_expert_distribution_recorder`
- `sglang.srt.layers: deep_gemm_wrapper`
- `sglang.srt.layers.activation: SiluAndMul`
- `sglang.srt.layers.attention.fla.layernorm_gated: RMSNorm`
- `sglang.srt.layers.attention.fla.layernorm_gated: layernorm_fn`
- `sglang.srt.layers.communicator: LayerCommunicator, LayerScatterModes`
- `sglang.srt.layers.dp_attention: get_attention_tp_rank, get_attention_tp_size, is_dp_attention_enabled`
- `sglang.srt.layers.layernorm: RMSNorm`
- `sglang.srt.layers.linear: ColumnParallelLinear, MergedColumnParallelLinear, QKVParallelLinear, RowParallelLinear`
- `sglang.srt.layers.logits_processor: LogitsProcessor`
- `sglang.srt.layers.moe: should_skip_post_experts_all_reduce`
- `sglang.srt.layers.moe.ep_moe.layer: DeepEPMoE, get_moe_impl_class`
