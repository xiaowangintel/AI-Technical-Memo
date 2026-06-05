# gpt_oss.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/gpt_oss.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Gpt Oss architecture into SGLang's serving runtime. Inference-only GptOss model compatible with HuggingFace weights. / 该模块将 Gpt Oss 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：Inference-only GptOss model compatible with HuggingFace weights。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15: Module header and imports / 模块头与导入
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

### Lines 92-92: Assignment: _is_npu / 赋值：_is_npu
```python
_is_npu = is_npu()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 93-93: Assignment: _is_cuda / 赋值：_is_cuda
```python
_is_cuda = is_cuda()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 94-98: Assignment: _is_tinygemm_supported / 赋值：_is_tinygemm_supported
```python
_is_tinygemm_supported = (
    _is_cuda
    and is_flashinfer_available()
    and (is_sm90_supported() or is_blackwell_supported())
)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 100-108: Conditional setup: _is_tinygemm_supported and get_cuda_version()[0] < 13 / 条件初始化：_is_tinygemm_supported and get_cuda_version()[0] < 13
```python
if _is_tinygemm_supported and get_cuda_version()[0] < 13:
    try:
        from flashinfer.gemm import tinygemm_bf16
    except ImportError:
        tinygemm_bf16 = None
        _is_tinygemm_supported = False
else:
    tinygemm_bf16 = None
    _is_tinygemm_supported = False
```
**EN:** This conditional block enables backend-specific setup so the module can adapt to available hardware or runtime features.
**CN:** 该条件块启用特定硬件/后端的初始化逻辑，使模块能适配不同运行环境。

### Lines 111-115: Class: GptOssConfig / 类：GptOssConfig
```python
class GptOssConfig(PretrainedConfig):
    model_type = "gpt_oss"

    def __init__(self, **kwargs):
        super().__init__(**kwargs)
```
**EN:** This class defines Gpt Oss Config inheriting from PretrainedConfig, grouping state and methods for this model component.
**CN:** 该类定义了 Gpt Oss Config，用于封装该模型组件的状态与方法。

### Lines 118-118: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 123-124: Function: get_attention_sliding_window_size() / 函数：get_attention_sliding_window_size()
```python
def get_attention_sliding_window_size(config):
    return config.sliding_window - 1
```
**EN:** This function implements get attention sliding window size for the surrounding model/runtime logic. Key parameters include config.
**CN:** 该函数实现了 get attention sliding window size 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 127-163: Class: TinyGemmLinear / 类：TinyGemmLinear
```python
class TinyGemmLinear(ReplicatedLinear):
    """ReplicatedLinear with a FlashInfer tinygemm BF16 fast path."""

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self._use_tinygemm = (
            _is_tinygemm_supported
            and not self.skip_bias_add
            and self.weight.is_contiguous()
            and self.weight.shape[0] % 16 == 0
            and self.weight.shape[1] % 64 == 0
            and self.weight.dtype == torch.bfloat16
            and (
                self.bias is None
                or (
                    self.bias.dtype == torch.bfloat16
                    and self.bias.is_contiguous()
                    and self.bias.shape[0] == self.weight.shape[0]
                )
            )
        )

    def forward(self, x: torch.Tensor) -> Tuple[torch.Tensor, Optional[torch.Tensor]]:
        if (
            self._use_tinygemm
            and x.ndim == 2
            and x.is_cuda
            and x.shape[0] <= 128
            and x.is_contiguous()
            and x.shape[1] == self.weight.shape[1]
            and x.dtype == torch.bfloat16
        ):
            out = x.new_empty((x.shape[0], self.output_size))
            tinygemm_bf16(x, self.weight, out, self.bias)
            return out, None

        return super().forward(x)
```
**EN:** This class defines Tiny Gemm Linear inheriting from ReplicatedLinear, grouping state and methods for this model component. Docstring summary: ReplicatedLinear with a FlashInfer tinygemm BF16 fast path..
**CN:** 该类定义了 Tiny Gemm Linear，用于封装该模型组件的状态与方法。 文档字符串摘要：ReplicatedLinear with a FlashInfer tinygemm BF16 fast path.。

### Lines 166-263: Class: GptOssSparseMoeBlock / 类：GptOssSparseMoeBlock
```python
class GptOssSparseMoeBlock(nn.Module):
    def __init__(
        self,
        layer_id: int,
        config: GptOssConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ):
        super().__init__()
        self.tp_size = get_tensor_model_parallel_world_size()
        self.layer_id = layer_id
        self.activation = config.hidden_act
        self.gemm1_alpha = getattr(config, "hidden_act_alpha", 1.702)
        self.gemm1_clamp_limit = config.swiglu_limit

        self.topk = TopK(
            top_k=config.num_experts_per_tok,
            renormalize=True,
            layer_id=layer_id,
        )

        self.top_k = config.num_experts_per_tok
        experts_type = get_moe_impl_class(quant_config)
        extra_kwargs = {}
        if experts_type.__name__ == "FusedMoE":
            quant_config_name = (
                quant_config.get_name() if quant_config is not None else None
            )
            extra_kwargs = {
                # for moe gate_up_proj and down_proj and their bias loading
                "use_weight_loader_fused": quant_config_name
                != "mxfp4"
            }

        self.experts = experts_type(
            num_experts=config.num_local_experts
            + get_global_server_args().ep_num_redundant_experts,
            top_k=config.num_experts_per_tok,
            layer_id=layer_id,
            hidden_size=config.hidden_size,
# ... truncated for brevity ...
```
**EN:** This class defines Gpt Oss Sparse Moe Block inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gpt Oss Sparse Moe Block，用于封装该模型组件的状态与方法。

### Lines 267-273: Function: moe_impl() / 函数：moe_impl()
```python
def moe_impl(layer_id: int, hidden_states: torch.Tensor) -> torch.Tensor:
    forward_context = get_forward_context()
    moe_fusion = forward_context.moe_fusions[layer_id]
    router_logits, _ = moe_fusion.router(hidden_states)
    topk_output = moe_fusion.topk(hidden_states, router_logits)
    final_hidden_states = moe_fusion.experts(hidden_states, topk_output)
    return final_hidden_states
```
**EN:** This function implements moe impl for the surrounding model/runtime logic. Key parameters include layer_id, hidden_states.
**CN:** 该函数实现了 moe impl 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 276-428: Class: GptOssAttention / 类：GptOssAttention
```python
class GptOssAttention(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        layer_id: int = 0,
        rope_theta: float = 10000,
        rope_scaling: Optional[Dict[str, Any]] = None,
        max_position_embeddings: int = 8192,
        head_dim: Optional[int] = None,
        rms_norm_eps: float = 1e-06,
        attention_bias: bool = False,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        sliding_window_size: int = -1,  # if -1, normal attention, else, window attention.
        layer_type: str = "",
        params_dtype: torch.dtype = torch.bfloat16,
    ) -> None:
        super().__init__()
        self.hidden_size = hidden_size
        self.sliding_window_size = sliding_window_size

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
        self.head_dim = head_dim or hidden_size // self.total_num_heads
# ... truncated for brevity ...
```
**EN:** This class defines Gpt Oss Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gpt Oss Attention，用于封装该模型组件的状态与方法。

### Lines 431-557: Class: GptOssDecoderLayer / 类：GptOssDecoderLayer
```python
class GptOssDecoderLayer(nn.Module):
    def __init__(
        self,
        config: GptOssConfig,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        sliding_window_size: int | None = None,
    ) -> None:
        super().__init__()
        self.config = config
        self.hidden_size = config.hidden_size
        rope_theta = config.rope_parameters["rope_theta"]
        rope_scaling = config.rope_parameters
        max_position_embeddings = getattr(config, "max_position_embeddings", 8192)
        head_dim = getattr(
            config, "head_dim", config.hidden_size // config.num_attention_heads
        )
        rms_norm_eps = config.rms_norm_eps
        attention_bias = config.attention_bias

        if sliding_window_size is None:
            self.sliding_window_size = get_attention_sliding_window_size(self.config)
        else:
            self.sliding_window_size = sliding_window_size

        self.self_attn = GptOssAttention(
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            num_kv_heads=config.num_key_value_heads,
            layer_id=layer_id,
            rope_theta=rope_theta,
            rope_scaling=rope_scaling,
            max_position_embeddings=max_position_embeddings,
            head_dim=head_dim,
            rms_norm_eps=rms_norm_eps,
            attention_bias=attention_bias,
            prefix=add_prefix("self_attn", prefix),
            sliding_window_size=self.sliding_window_size,
            layer_type=config.layer_types[layer_id],
# ... truncated for brevity ...
```
**EN:** This class defines Gpt Oss Decoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gpt Oss Decoder Layer，用于封装该模型组件的状态与方法。

### Lines 560-651: Class: GptOssModel / 类：GptOssModel
```python
class GptOssModel(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        decoder_layer_type: type[nn.Module] = GptOssDecoderLayer,
    ) -> None:
        super().__init__()
        self.padding_idx = config.pad_token_id
        self.vocab_size = config.vocab_size
        self.pp_group = get_pp_group()

        if _is_npu:
            config.hidden_act = "npu_swiglu_oai"

        if self.pp_group.is_first_rank:
            self.embed_tokens = VocabParallelEmbedding(
                config.vocab_size,
                config.hidden_size,
                use_attn_tp_group=is_dp_attention_enabled(),
                prefix=add_prefix("embed_tokens", prefix),
            )
        else:
            self.embed_tokens = PPMissingLayer()

        # Use the provided decoder layer type or default to GptOssDecoderLayer
        decoder_layer_type = decoder_layer_type or GptOssDecoderLayer
        self.layers, self.start_layer, self.end_layer = make_layers(
            config.num_hidden_layers,
            lambda idx, prefix: decoder_layer_type(
                layer_id=idx,
                config=config,
                quant_config=quant_config,
                prefix=prefix,
            ),
            pp_rank=self.pp_group.rank_in_group,
            pp_size=self.pp_group.world_size,
            prefix=add_prefix("layers", prefix),
        )
# ... truncated for brevity ...
```
**EN:** This class defines Gpt Oss Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gpt Oss Model，用于封装该模型组件的状态与方法。

### Lines 654-1226: Class: GptOssForCausalLM / 类：GptOssForCausalLM
```python
class GptOssForCausalLM(nn.Module):
    fall_back_to_pt_during_load = False

    _lora_pattern_moe = re.compile(
        r"^(?:model\.layers\.\d+\.(?:self_attn\.(?:qkv_proj|o_proj)|mlp\.experts)|lm_head|model\.embed_tokens)$"
    )

    def should_apply_lora(self, module_name: str) -> bool:
        return bool(self._lora_pattern_moe.match(module_name))

    def __init__(
        self,
        config: GptOssConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.pp_group = get_pp_group()
        self.config = config
        self.quant_config = quant_config
        self.model = GptOssModel(
            config, quant_config, prefix=add_prefix("model", prefix)
        )
        self.lm_head = ParallelLMHead(
            config.vocab_size,
            config.hidden_size,
            # quant_config=quant_config,
            prefix=add_prefix("lm_head", prefix),
            use_attn_tp_group=get_global_server_args().enable_dp_lm_head,
        )
        self.logits_processor = LogitsProcessor(config)
        self.capture_aux_hidden_states = False

        self._routed_experts_weights_of_layer = LazyValue(
            lambda: {
                layer_id: self.model.layers[layer_id].mlp.get_moe_weights()
                for layer_id in range(self.start_layer, self.end_layer)
                if isinstance(self.model.layers[layer_id].mlp, GptOssSparseMoeBlock)
            }
        )
# ... truncated for brevity ...
```
**EN:** This class defines Gpt Oss For Causal L M inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gpt Oss For Causal L M，用于封装该模型组件的状态与方法。

### Lines 1229-1247: Function: _canonicalize_weights() / 函数：_canonicalize_weights()
```python
def _canonicalize_weights(config, weights_in: Iterable[Tuple[str, torch.Tensor]]):
    weights_out_dict = dict(weights_in)

    for layer_id in range(config.num_hidden_layers):
        for name_chunk in ["mlp1_weight", "mlp2_weight"]:
            name_prefix = f"block.{layer_id}.mlp.{name_chunk}"
            w_blocks = weights_out_dict.pop(f"{name_prefix}.blocks", None)
            w_scales = weights_out_dict.pop(f"{name_prefix}.scales", None)
            if w_blocks is not None:
                weights_out_dict[name_prefix] = _WeightCreator(
                    partial(
                        _dequant_mlp_weight,
                        debug_name=name_prefix,
                        w_blocks=w_blocks,
                        w_scales=w_scales,
                    )
                )

    return list(weights_out_dict.items())
```
**EN:** This function implements canonicalize weights for the surrounding model/runtime logic. Key parameters include config, weights_in.
**CN:** 该函数实现了 canonicalize weights 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 1250-1267: Function: _dequant_mlp_weight() / 函数：_dequant_mlp_weight()
```python
def _dequant_mlp_weight(debug_name, w_blocks, w_scales):
    if get_tensor_model_parallel_rank() == 0:
        logger.info(f"Dequantize {debug_name} start")

    original_device = w_blocks.device

    w_blocks = w_blocks.cuda()
    w_scales = w_scales.cuda()

    w_bf16 = dequant_mxfp4(w_block=w_blocks, w_scale=w_scales, out_dtype=torch.bfloat16)
    w_bf16 = w_bf16.transpose(-2, -1).contiguous()

    if get_tensor_model_parallel_rank() == 0:
        logger.info(
            f"Dequantize {debug_name} end {w_blocks.shape=} {w_scales.shape=} {w_bf16.shape=}"
        )

    return w_bf16.to(original_device)
```
**EN:** This function implements dequant mlp weight for the surrounding model/runtime logic. Key parameters include debug_name, w_blocks, w_scales.
**CN:** 该函数实现了 dequant mlp weight 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 1270-1281: Class: _WeightCreator / 类：_WeightCreator
```python
class _WeightCreator:
    def __init__(self, fn):
        self._fn = fn

    @staticmethod
    def maybe_materialize(obj):
        if isinstance(obj, _WeightCreator):
            output = obj._fn()
            obj._fn = None
            return output

        return obj
```
**EN:** This class defines Weight Creator, grouping state and methods for this model component.
**CN:** 该类定义了 Weight Creator，用于封装该模型组件的状态与方法。

### Lines 1284-1284: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = GptOssForCausalLM
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `logging`
- `math`
- `re`
- `collections.abc: Iterable`
- `functools: partial`
- `typing: Any, Dict, List, Optional, Tuple, Union`
- `torch`
- `torch: nn`
- `transformers: PretrainedConfig`
- `sglang.srt.compilation.piecewise_context_manager: get_forward_context, is_in_piecewise_cuda_graph`
- `sglang.srt.distributed: get_moe_expert_parallel_rank, get_moe_expert_parallel_world_size, get_moe_tensor_parallel_rank, get_moe_tensor_parallel_world_size, get_pp_group, get_tensor_model_parallel_rank, get_tensor_model_parallel_world_size, tensor_model_parallel_all_reduce`
- `sglang.srt.eplb.expert_distribution: get_global_expert_distribution_recorder`
- `sglang.srt.eplb.expert_location: ModelConfigForExpertLocation`
- `sglang.srt.layers.communicator: LayerCommunicator, LayerScatterModes`
- `sglang.srt.layers.dp_attention: get_attention_tp_rank, get_attention_tp_size, is_dp_attention_enabled`
- `sglang.srt.layers.layernorm: RMSNorm`
- `sglang.srt.layers.linear: QKVParallelLinear, ReplicatedLinear, RowParallelLinear`
- `sglang.srt.layers.logits_processor: LogitsProcessor`
- `sglang.srt.layers.moe: get_moe_a2a_backend`
- `sglang.srt.layers.moe.ep_moe.layer: get_moe_impl_class`
