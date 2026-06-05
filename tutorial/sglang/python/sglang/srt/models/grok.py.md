# grok.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/grok.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Grok architecture into SGLang's serving runtime, typically defining layers, model wrappers, and weight-loading behavior for inference. / 该模块将 Grok 架构接入 SGLang 服务运行时，通常定义推理所需的层、模型封装以及权重加载逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14: Module header and imports / 模块头与导入
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

### Lines 65-65: Assignment: _is_npu / 赋值：_is_npu
```python
_is_npu = is_npu()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 67-67: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 70-108: Class: Grok1MLP / 类：Grok1MLP
```python
class Grok1MLP(nn.Module):
    def __init__(
        self,
        hidden_size: int,
        intermediate_size: int,
        layer_id: int,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
        reduce_results=True,
        use_presharded_weights: bool = False,
        split_gate_up: bool = False,
    ) -> None:
        super().__init__()

        self.gate_up_proj = MergedColumnParallelLinear(
            hidden_size,
            [intermediate_size] * 2,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("gate_up_proj", prefix),
            use_presharded_weights=use_presharded_weights,
        )
        self.down_proj = RowParallelLinear(
            intermediate_size,
            hidden_size,
            bias=False,
            quant_config=quant_config,
            prefix=add_prefix("down_proj", prefix),
            reduce_results=reduce_results,
            use_presharded_weights=use_presharded_weights,
        )
        self.act_fn = GeluAndMul(approximate="tanh")
        self.layer_id = layer_id

    def forward(self, x):
        gate_up, _ = self.gate_up_proj(x)
        x, _ = gelu_and_mul_triton(gate_up)
        x, _ = self.down_proj(x)
        return x
```
**EN:** This class defines Grok1 M L P inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Grok1 M L P，用于封装该模型组件的状态与方法。

### Lines 111-182: Class: Grok1MoE / 类：Grok1MoE
```python
class Grok1MoE(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        layer_id: int,
        num_experts: int,
        top_k: int,
        hidden_size: int,
        intermediate_size: int,
        params_dtype: Optional[torch.dtype] = None,
        quant_config: Optional[QuantizationConfig] = None,
        tp_size: Optional[int] = None,
        reduce_results: bool = True,
        use_presharded_weights: bool = False,
        inplace: bool = True,
        no_combine: bool = False,
        prefix: str = "",
    ):
        super().__init__()
        self.hidden_size = hidden_size

        self.gate = ReplicatedLinear(
            hidden_size,
            num_experts,
            bias=False,
            params_dtype=torch.float32,
            quant_config=None,
        )

        self.router_logit_softcapping = 30.0
        custom_routing_function = functools.partial(
            fused_moe_router_shim, self.router_logit_softcapping
        )

        self.topk = TopK(
            top_k=top_k,
            renormalize=False,
            layer_id=layer_id,
            custom_routing_function=None if _is_npu else custom_routing_function,
        )
# ... truncated for brevity ...
```
**EN:** This class defines Grok1 Mo E inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Grok1 Mo E，用于封装该模型组件的状态与方法。

### Lines 185-193: Function: _yarn_linear_ramp_mask() / 函数：_yarn_linear_ramp_mask()
```python
def _yarn_linear_ramp_mask(
    low: float, high: float, dim: int, dtype: torch.dtype
) -> torch.Tensor:
    if low == high:
        low -= 0.001  # Prevent singularity

    linear_func = (torch.arange(dim, dtype=dtype) - low) / (high - low)
    ramp_func = torch.clamp(linear_func, 0, 1)
    return ramp_func
```
**EN:** This function implements yarn linear ramp mask for the surrounding model/runtime logic. Key parameters include low, high, dim, dtype.
**CN:** 该函数实现了 yarn linear ramp mask 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 196-219: Function: get_rope_scaling() / 函数：get_rope_scaling()
```python
def get_rope_scaling(config):
    rope_type = getattr(config, "rope_type", None)
    if rope_type:
        original_max_position_embeddings = getattr(
            config, "original_max_position_embeddings", None
        )
        scaling_factor = getattr(config, "scaling_factor", None)
        extrapolation_factor = getattr(config, "extrapolation_factor", 1.0)
        attn_factor = getattr(config, "attn_factor", 1.0)
        beta_fast = getattr(config, "beta_fast", 32)
        beta_slow = getattr(config, "beta_slow", 1)
        rope_scaling = {
            "extra_method": rope_type,
            "max_position_embeddings": original_max_position_embeddings,
            "scaling_factor": scaling_factor,
            "extrapolation_factor": extrapolation_factor,
            "attn_factor": attn_factor,
            "beta_fast": beta_fast,
            "beta_slow": beta_slow,
            "dtype": torch.bfloat16,
        }
        return rope_scaling
    else:
        return None
```
**EN:** This function implements get rope scaling for the surrounding model/runtime logic. Key parameters include config.
**CN:** 该函数实现了 get rope scaling 相关逻辑，用于支撑周边模型或运行时流程。

### Lines 222-313: Class: ScalingRotaryEmbedding / 类：ScalingRotaryEmbedding
```python
class ScalingRotaryEmbedding(RotaryEmbedding):
    """Scale the RotaryEmbedding in a way similar to YaRN method. https://arxiv.org/pdf/2309.00071."""

    def __init__(
        self,
        head_size: int,
        rotary_dim: int,
        max_position_embeddings: int,
        base: int,
        is_neox_style: bool,
        scaling_factor: float,
        dtype: torch.dtype,
        *,
        extra_method: str = "yarn_log",
        extrapolation_factor: float = 1,
        attn_factor: float = 1,
        beta_fast: int = 32,
        beta_slow: int = 1,
    ) -> None:
        self.scaling_factor = scaling_factor
        self.extra_method = extra_method
        self.extrapolation_factor = extrapolation_factor
        self.attn_factor = attn_factor
        self.beta_fast = beta_fast
        self.beta_slow = beta_slow
        if _is_npu:
            dtype = torch.float32
        # Get n-d magnitude scaling corrected for interpolation
        self.mscale = float(_yarn_get_mscale(self.scaling_factor) * attn_factor)
        super().__init__(
            head_size, rotary_dim, max_position_embeddings, base, is_neox_style, dtype
        )

    def _compute_inv_freq(self, scaling_factor: float) -> torch.Tensor:
        pos_freqs = self.base ** (
            torch.arange(0, self.rotary_dim, 2, dtype=torch.float) / self.rotary_dim
        )
        inv_freq_extrapolation = 1.0 / pos_freqs
        inv_freq_interpolation = 1.0 / (scaling_factor * pos_freqs)

# ... truncated for brevity ...
```
**EN:** This class defines Scaling Rotary Embedding inheriting from RotaryEmbedding, grouping state and methods for this model component. Docstring summary: Scale the RotaryEmbedding in a way similar to YaRN method. https://arxiv.org/pdf/2309.00071..
**CN:** 该类定义了 Scaling Rotary Embedding，用于封装该模型组件的状态与方法。 文档字符串摘要：Scale the RotaryEmbedding in a way similar to YaRN method. https://arxiv.org/pdf/2309.00071.。

### Lines 316-457: Class: Grok1Attention / 类：Grok1Attention
```python
class Grok1Attention(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        hidden_size: int,
        num_heads: int,
        num_kv_heads: int,
        layer_id: int = 0,
        max_position: int = 4096 * 32,
        rope_theta: float = 10000,
        quant_config: Optional[QuantizationConfig] = None,
        reduce_results: bool = True,
        alt_stream: Optional[torch.cuda.Stream] = None,
        load_presharded_attn: bool = False,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.layer_id = layer_id
        self.hidden_size = hidden_size
        attn_tp_rank = get_tensor_model_parallel_rank()
        attn_tp_size = get_tensor_model_parallel_world_size()
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
        self.head_dim = getattr(config, "head_dim", 128)
        self.q_size = self.num_heads * self.head_dim
        self.kv_size = self.num_kv_heads * self.head_dim
        self.scaling = self.head_dim**-0.5
        self.rope_theta = rope_theta
# ... truncated for brevity ...
```
**EN:** This class defines Grok1 Attention inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Grok1 Attention，用于封装该模型组件的状态与方法。

### Lines 460-622: Class: Grok1DecoderLayer / 类：Grok1DecoderLayer
```python
class Grok1DecoderLayer(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        layer_id: int = 0,
        quant_config: Optional[QuantizationConfig] = None,
        load_presharded_moe: bool = False,
        load_presharded_attn: bool = False,
        load_presharded_mlp: bool = False,
        alt_stream: Optional[torch.cuda.Stream] = None,
        skip_moe: bool = False,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.num_experts = config.num_local_experts
        self.hidden_size = config.hidden_size
        self.residual_moe = getattr(config, "residual_moe", False)
        self.layer_id = layer_id
        self.alt_stream = alt_stream or torch.cuda.Stream()

        rope_theta = getattr(config, "rope_theta", None)
        if rope_theta is None:
            rope_params = getattr(config, "rope_parameters", None)
            rope_theta = rope_params["rope_theta"] if rope_params else 10000
        self.self_attn = Grok1Attention(
            config=config,
            hidden_size=self.hidden_size,
            num_heads=config.num_attention_heads,
            max_position=(
                config.context_len
                if hasattr(config, "context_len")
                else config.max_position_embeddings
            ),
            num_kv_heads=config.num_key_value_heads,
            layer_id=layer_id,
            rope_theta=rope_theta,
            quant_config=quant_config,
            reduce_results=False,
            alt_stream=self.alt_stream,
            load_presharded_attn=load_presharded_attn,
# ... truncated for brevity ...
```
**EN:** This class defines Grok1 Decoder Layer inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Grok1 Decoder Layer，用于封装该模型组件的状态与方法。

### Lines 625-694: Class: Grok1Model / 类：Grok1Model
```python
class Grok1Model(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        load_presharded_moe: bool = False,
        load_presharded_embedding: bool = False,
        load_presharded_attn: bool = False,
        load_presharded_mlp: bool = False,
        replicate_embedding: bool = False,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.padding_idx = config.pad_token_id
        self.vocab_size = config.vocab_size

        self.embed_tokens = VocabParallelEmbedding(
            config.vocab_size,
            config.hidden_size,
            use_presharded_weights=load_presharded_embedding,
            enable_tp=not replicate_embedding,
            prefix=add_prefix("embed_tokens", prefix),
        )

        self.alt_stream = torch.cuda.Stream()
        self.layers = nn.ModuleList(
            [
                Grok1DecoderLayer(
                    config,
                    i,
                    quant_config=quant_config,
                    load_presharded_moe=load_presharded_moe,
                    load_presharded_attn=load_presharded_attn,
                    load_presharded_mlp=load_presharded_mlp,
                    alt_stream=self.alt_stream,
                )
                for i in range(config.num_hidden_layers)
            ]
        )
# ... truncated for brevity ...
```
**EN:** This class defines Grok1 Model inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Grok1 Model，用于封装该模型组件的状态与方法。

### Lines 697-945: Class: Grok1ForCausalLM / 类：Grok1ForCausalLM
```python
class Grok1ForCausalLM(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.quant_config = quant_config

        # Get presharded weights.
        self.load_presharded_mlp = getattr(config, "load_presharded_mlp", False)
        self.load_presharded_moe = (
            getattr(config, "load_presharded_moe", True)
            and self.config.num_local_experts > 0
            and get_tensor_model_parallel_world_size() > 1
        )
        self.load_presharded_attn = getattr(config, "load_presharded_attn", False)
        self.load_presharded_embedding = getattr(
            config, "load_presharded_embedding", False
        )

        default_replicate_lm_head = False
        self.replicate_lm_head = getattr(
            config, "replicate_lm_head", default_replicate_lm_head
        )

        if get_tensor_model_parallel_world_size() > 1:
            setattr(DefaultModelLoader, "_prepare_weights", _prepare_presharded_weights)

        self.replicate_embedding = getattr(config, "replicate_embedding", False)

        self.model = Grok1Model(
            config,
            quant_config=quant_config,
            load_presharded_moe=self.load_presharded_moe,
            load_presharded_embedding=self.load_presharded_embedding,
            load_presharded_attn=self.load_presharded_attn,
            load_presharded_mlp=self.load_presharded_mlp,
# ... truncated for brevity ...
```
**EN:** This class defines Grok1 For Causal L M inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Grok1 For Causal L M，用于封装该模型组件的状态与方法。

### Lines 948-948: Assignment: old_prepare_weights / 赋值：old_prepare_weights
```python
old_prepare_weights = getattr(DefaultModelLoader, "_prepare_weights")
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 951-994: Function: _prepare_presharded_weights() / 函数：_prepare_presharded_weights()
```python
def _prepare_presharded_weights(
    self, model_name_or_path: str, revision: Optional[str], fall_back_to_pt: bool
) -> Tuple[str, list[str], bool]:
    import glob
    import os

    if get_tensor_model_parallel_world_size() == 1:
        return old_prepare_weights(self, model_name_or_path, revision, fall_back_to_pt)

    if not os.path.isdir(model_name_or_path):
        from sglang.srt.model_loader.weight_utils import download_weights_from_hf

        allow_patterns = ["*.safetensors", "*.bin"]
        hf_folder = download_weights_from_hf(
            model_name_or_path,
            self.load_config.download_dir,
            allow_patterns,
            revision,
            ignore_patterns=self.load_config.ignore_patterns,
        )
    else:
        hf_folder = model_name_or_path

    tp_rank = get_tensor_model_parallel_rank()

    # The old format
    allow_patterns = [f"*-{tp_rank:03d}.bin"]

    # The new format
    allow_patterns += [f"*-TP-{tp_rank:03d}.safetensors", "*-TP-common.safetensors"]

    hf_weights_files = []
    for pattern in allow_patterns:
        hf_weights_files += glob.glob(os.path.join(hf_folder, pattern))

    if not hf_weights_files:
        return old_prepare_weights(self, model_name_or_path, revision, fall_back_to_pt)

    if hf_weights_files[0].endswith("safetensors"):
        use_safetensors = True
# ... truncated for brevity ...
```
**EN:** This function prepares inputs or selects an execution path before the main compute step. Key parameters include model_name_or_path, revision, fall_back_to_pt.
**CN:** 该函数在主计算之前准备输入，或根据条件选择合适的执行路径。

### Lines 997-1000: Class: Grok1ModelForCausalLM / 类：Grok1ModelForCausalLM
```python
class Grok1ModelForCausalLM(Grok1ForCausalLM):
    """An alias for backward-compatbility."""

    pass
```
**EN:** This class defines Grok1 Model For Causal L M inheriting from Grok1ForCausalLM, grouping state and methods for this model component. Docstring summary: An alias for backward-compatbility..
**CN:** 该类定义了 Grok1 Model For Causal L M，用于封装该模型组件的状态与方法。 文档字符串摘要：An alias for backward-compatbility.。

### Lines 1003-1003: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = [Grok1ForCausalLM, Grok1ModelForCausalLM]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `functools`
- `logging`
- `math`
- `typing: Iterable, Optional, Tuple`
- `torch`
- `torch.nn.functional`
- `torch: nn`
- `transformers: PretrainedConfig`
- `sglang.srt.distributed: get_tensor_model_parallel_rank, get_tensor_model_parallel_world_size, tensor_model_parallel_all_reduce`
- `sglang.srt.layers.activation: GeluAndMul`
- `sglang.srt.layers.elementwise: fused_dual_residual_rmsnorm, fused_rmsnorm, gelu_and_mul_triton`
- `sglang.srt.layers.layernorm: RMSNorm`
- `sglang.srt.layers.linear: MergedColumnParallelLinear, QKVParallelLinear, ReplicatedLinear, RowParallelLinear`
- `sglang.srt.layers.logits_processor: LogitsProcessor`
- `sglang.srt.layers.moe.fused_moe_triton: FusedMoE`
- `sglang.srt.layers.moe.router: fused_moe_router_shim`
- `sglang.srt.layers.moe.topk: TopK`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.layers.radix_attention: RadixAttention`
- `sglang.srt.layers.rotary_embedding: RotaryEmbedding, _yarn_find_correction_range, _yarn_get_mscale, get_rope`
