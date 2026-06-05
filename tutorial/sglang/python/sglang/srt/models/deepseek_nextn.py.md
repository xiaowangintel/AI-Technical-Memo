# deepseek_nextn.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/deepseek_nextn.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the Deepseek Nextn architecture into SGLang's serving runtime. Inference-only DeepSeek NextN Speculative Decoding. / 该模块将 Deepseek Nextn 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：Inference-only DeepSeek NextN Speculative Decoding。

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

### Lines 62-62: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 65-65: Assignment: _is_cuda / 赋值：_is_cuda
```python
_is_cuda = is_cuda()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 66-66: Assignment: _is_npu / 赋值：_is_npu
```python
_is_npu = is_npu()
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 69-224: Class: DeepseekModelNextN / 类：DeepseekModelNextN
```python
class DeepseekModelNextN(nn.Module):

    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        if enable_nextn_moe_bf16_cast_to_fp8(quant_config):
            # refer to real DeepSeek V3 quant config
            moe_quant_config_override = Fp8Config(
                is_checkpoint_fp8_serialized=True,
                weight_block_size=[128, 128],
            )
        else:
            moe_quant_config_override = None

        if quant_config is not None and quant_config.get_name() == "modelopt_fp4":
            logger.warning(
                "Overriding DeepseekV3ForCausalLMNextN quant config for modelopt_fp4 Deepseek model."
            )
            quant_config = None

        self.vocab_size = config.vocab_size

        self.embed_tokens = VocabParallelEmbedding(
            config.vocab_size,
            config.hidden_size,
            use_attn_tp_group=is_dp_attention_enabled(),
            prefix=add_prefix("embed_tokens", prefix),
        )

        self.enorm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)
        self.hnorm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)

        if quant_config is not None and quant_config.get_name() == "quark":
            self.eh_proj = ReplicatedLinear(
                2 * config.hidden_size,
                config.hidden_size,
# ... truncated for brevity ...
```
**EN:** This class defines Deepseek Model Next N inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Deepseek Model Next N，用于封装该模型组件的状态与方法。

### Lines 227-314: Class: DeepseekV3ForCausalLMNextN / 类：DeepseekV3ForCausalLMNextN
```python
class DeepseekV3ForCausalLMNextN(DeepseekV3ForCausalLM):

    # Support amd/DeepSeek-R1-0528-MXFP4 renaming: model.layers.61*.
    # Ref: HF config.json for amd/DeepSeek-R1-0528-MXFP4
    # https://huggingface.co/amd/DeepSeek-R1-0528-MXFP4/blob/main/config.json
    hf_to_sglang_mapper = WeightsMapper(
        orig_to_new_substr={
            "model.layers.61": "model.decoder",
        },
    )

    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        nn.Module.__init__(self)
        self.config = config
        self.tp_size = get_tensor_model_parallel_world_size()
        self.quant_config = quant_config
        # if not set, model load will be broken in DeepseekV3ForCausalLM load_weights()
        self.pp_group = get_pp_group()
        self.determine_num_fused_shared_experts("DeepseekV3ForCausalLMNextN")
        self.use_nsa = is_deepseek_nsa(config)
        self.nsa_enable_prefill_cp = is_nsa_enable_prefill_cp()
        if self.nsa_enable_prefill_cp:
            self.cp_rank = get_attention_cp_rank()
            self.cp_size = get_attention_cp_size()
        else:
            self.cp_rank = None
            self.cp_size = None

        nextn_quant_config = quant_config
        # For quark, if the MTP layer is listed in exclude_layers, set quant_config to None.
        if nextn_quant_config is not None and nextn_quant_config.get_name() == "quark":
            from sglang.srt.layers.quantization.quark.utils import (
                should_ignore_layer,
            )

# ... truncated for brevity ...
```
**EN:** This class defines Deepseek V3 For Causal L M Next N inheriting from DeepseekV3ForCausalLM, grouping state and methods for this model component.
**CN:** 该类定义了 Deepseek V3 For Causal L M Next N，用于封装该模型组件的状态与方法。

### Lines 317-317: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = [DeepseekV3ForCausalLMNextN]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `logging`
- `os`
- `typing: Iterable, Optional, Tuple`
- `torch`
- `safetensors.torch: load_file`
- `torch: nn`
- `transformers: PretrainedConfig`
- `sglang.srt.configs.model_config: is_deepseek_nsa`
- `sglang.srt.distributed: get_pp_group, get_tensor_model_parallel_world_size`
- `sglang.srt.environ: envs`
- `sglang.srt.eplb.expert_distribution: get_global_expert_distribution_recorder`
- `sglang.srt.layers.attention.nsa.utils: can_nsa_cp_split, is_nsa_enable_prefill_cp, nsa_use_prefill_cp`
- `sglang.srt.layers.dp_attention: get_attention_cp_rank, get_attention_cp_size, is_dp_attention_enabled`
- `sglang.srt.layers.layernorm: RMSNorm`
- `sglang.srt.layers.linear: ReplicatedLinear`
- `sglang.srt.layers.logits_processor: LogitsProcessor`
- `sglang.srt.layers.quantization: Fp8Config`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.layers.utils.cp_utils: cp_all_gather_rerange_output, cp_split_and_rebuild_data, cp_split_and_rebuild_position, prepare_context_parallel_metadata`
- `sglang.srt.layers.vocab_parallel_embedding: ParallelLMHead, VocabParallelEmbedding`
