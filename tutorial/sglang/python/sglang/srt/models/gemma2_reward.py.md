# gemma2_reward.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/models/gemma2_reward.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module integrates the gemma2 Reward architecture into SGLang's serving runtime, typically defining layers, model wrappers, and weight-loading behavior for inference. / 该模块将 gemma2 Reward 架构接入 SGLang 服务运行时，通常定义推理所需的层、模型封装以及权重加载逻辑。

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

### Lines 28-72: Class: Gemma2ForSequenceClassification / 类：Gemma2ForSequenceClassification
```python
class Gemma2ForSequenceClassification(nn.Module):
    def __init__(
        self,
        config: Gemma2Config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__()
        self.config = config
        self.quant_config = quant_config
        self.num_labels = config.num_labels
        self.model = Gemma2Model(
            config, quant_config=quant_config, prefix=add_prefix("model", prefix)
        )
        self.score = nn.Linear(config.hidden_size, self.num_labels, bias=False)
        self.pooler = Pooler(pooling_type=PoolingType.LAST, normalize=False)

        self.eos_token_id = config.eos_token_id

    @torch.no_grad()
    def forward(
        self,
        input_ids: torch.Tensor,
        positions: torch.Tensor,
        forward_batch: ForwardBatch,
        input_embeds: torch.Tensor = None,
        get_embedding: bool = True,
    ) -> EmbeddingPoolerOutput:
        assert (
            get_embedding
        ), "Gemma2ForSequenceClassification is only used for embedding"

        hidden_states = self.model(input_ids, positions, forward_batch, input_embeds)
        last_token_hidden = self.pooler(hidden_states, forward_batch).embeddings
        scores = self.score(last_token_hidden)

        return EmbeddingPoolerOutput(
            embeddings=scores,
            pooled_hidden_states=(
                last_token_hidden if forward_batch.return_pooled_hidden_states else None
# ... truncated for brevity ...
```
**EN:** This class defines Gemma2 For Sequence Classification inheriting from nn.Module, grouping state and methods for this model component.
**CN:** 该类定义了 Gemma2 For Sequence Classification，用于封装该模型组件的状态与方法。

### Lines 75-75: Assignment: EntryClass / 赋值：EntryClass
```python
EntryClass = [Gemma2ForSequenceClassification]
```
**EN:** This block defines module-level constants or cached flags that influence later execution paths.
**CN:** 该代码块定义模块级常量或缓存标记，用于影响后续执行分支。

## Key Concepts / 关键概念
- **EN:** Model integration with SGLang runtime components such as tensor parallel layers, attention kernels, and logits processing. / **CN:** 模型与 SGLang 运行时组件的集成，例如张量并行层、注意力内核和 logits 处理。
- **EN:** Inference-focused structure: configuration parsing, forward execution, and checkpoint/weight compatibility. / **CN:** 面向推理的结构设计：配置解析、前向执行以及检查点/权重兼容性。

## Dependencies / 依赖关系
- `typing: Iterable, Optional, Tuple`
- `torch`
- `torch: nn`
- `transformers: Gemma2Config`
- `sglang.srt.layers.pooler: EmbeddingPoolerOutput, Pooler, PoolingType`
- `sglang.srt.layers.quantization.base_config: QuantizationConfig`
- `sglang.srt.model_executor.forward_batch_info: ForwardBatch`
- `sglang.srt.models.gemma2: Gemma2ForCausalLM, Gemma2Model`
- `sglang.srt.utils: add_prefix`
