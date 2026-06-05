# glm4_moe_nextn.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/glm4_moe_nextn.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the glm4 Moe Nextn architecture into SGLang's serving runtime. Inference-only GLM-4.5, GLM-4.6 Speculative Decoding. / 该模块将 glm4 Moe Nextn 架构接入 SGLang 服务运行时，并提供与推理相关的核心组件。其文档字符串指出：Inference-only GLM-4.5, GLM-4.6 Speculative Decoding。

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

### Lines 41-41: Assignment: logger / 赋值：logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

### Lines 44-118: Class: Glm4MoeModelNextN / 类：Glm4MoeModelNextN
```python
class Glm4MoeModelNextN(nn.Module):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        if quant_config is not None and quant_config.get_name() == "modelopt_fp4":
            logger.warning(
                "Overriding Glm4MoeForCausalLMNextN quant config for modelopt_fp4 GLM-4.5 / GLM-4.6 model."
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

        self.eh_proj = nn.Linear(2 * config.hidden_size, config.hidden_size, bias=False)

        self.decoder = Glm4MoeDecoderLayer(
            config,
            0,
            quant_config=quant_config,
            is_nextn=True,
            prefix=add_prefix("decoder", prefix),
        )

        self.shared_head = nn.Module()
        self.shared_head.norm = RMSNorm(config.hidden_size, eps=config.rms_norm_eps)

    def forward(
# ... truncated for brevity ...
```
**EN:** This class defines Glm4 Moe Model Next N inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Glm4 Moe Model Next N，用于封装该模型组件的状态与方法。

### Lines 121-179: Class: Glm4MoeForCausalLMNextN / 类：Glm4MoeForCausalLMNextN
```python
class Glm4MoeForCausalLMNextN(Glm4MoeForCausalLM):
    def __init__(
        self,
        config: PretrainedConfig,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        nn.Module.__init__(self)
        self.config = config
        self.tp_size = get_tensor_model_parallel_world_size()
        self.needs_quant_draft = (
            get_global_server_args().speculative_draft_model_quantization is not None
            or quant_config is not None
        )
        quant_config = quant_config if self.needs_quant_draft else None
        self.model = Glm4MoeModelNextN(
            config, quant_config, prefix=add_prefix("model", prefix)
        )
        self.lm_head = ParallelLMHead(
            config.vocab_size,
            config.hidden_size,
            quant_config=quant_config,
            prefix=add_prefix("model.shared_head.head", prefix),
            use_attn_tp_group=get_global_server_args().enable_dp_lm_head,
        )
        self.logits_processor = LogitsProcessor(config)

        self.num_fused_shared_experts = (
            0 if get_global_server_args().disable_shared_experts_fusion else 1
        )

    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
    ) -> torch.Tensor:
        # Support unquant speculative draft model
        if self.needs_quant_draft:
# ... truncated for brevity ...
```
**EN:** This class defines Glm4 Moe For Causal L M Next N inheriting from Glm4MoeForCausalLM, grouping state and methods for this model component.
**CN:** 该类定义了 Glm4 Moe For Causal L M Next N，用于封装该模型组件的状态与方法。

### Lines 182-182: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = [Glm4MoeForCausalLMNextN]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `contextlib`
- `logging`
- `typing: Iterable, Optional, Tuple`
- `torch`
- `torch: nn`
- `transformers: PretrainedConfig`
- `sglang.srt.distributed: get_tensor_model_parallel_world_size`
- `sglang.srt.environ: temp_set_env`
- `sglang.srt.eplb.expert_distribution: get_global_expert_distribution_recorder`
- `sglang.srt.layers.dp_attention: is_dp_attention_enabled`
- `sglang.srt.layers.layernorm: RMSNorm`
- `sglang.srt.layers.logits_processor: LogitsProcessor`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.layers.vocab_parallel_embedding: ParallelLMHead, VocabParallelEmbedding`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch`
- `sglang.srt.models.glm4_moe: Glm4MoeDecoderLayer, Glm4MoeForCausalLM`
- `sglang.srt.server_args: get_global_server_args`
- `sglang.srt.utils: add_prefix`
