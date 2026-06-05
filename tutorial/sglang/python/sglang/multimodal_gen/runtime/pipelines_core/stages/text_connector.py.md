# text_connector.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/pipelines_core/stages/text_connector.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module coordinates pipeline execution for text connector, connecting stages, tensors, and runtime metadata. Key symbols include `LTX2TextConnectorStage`. / 该模块负责 text connector 的流水线执行，连接各阶段、张量与运行时元数据。 关键符号包括 `LTX2TextConnectorStage`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Imports and module setup / 导入与模块初始化
```python
import torch

from sglang.multimodal_gen.runtime.managers.forward_context import set_forward_context
from sglang.multimodal_gen.runtime.pipelines_core.schedule_batch import Req
from sglang.multimodal_gen.runtime.pipelines_core.stages.base import PipelineStage
from sglang.multimodal_gen.runtime.server_args import ServerArgs
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 9-105: Class `LTX2TextConnectorStage` / 类 `LTX2TextConnectorStage`
```python
class LTX2TextConnectorStage(PipelineStage):
    """
    Stage for applying LTX-2 Text Connectors to split/transform text embeddings
    into video and audio contexts.
    """

    def __init__(self, connectors):
        super().__init__()
        self.connectors = connectors

    def forward(self, batch: Req, server_args: ServerArgs) -> Req:
        # Input: batch.prompt_embeds (from Gemma, [B, S, D])
        # Output: batch.prompt_embeds (Video Context), batch.audio_prompt_embeds (Audio Context)

# ...
            batch.audio_prompt_embeds = [connector_audio_prompt_embeds]
            batch.prompt_attention_mask = connector_mask

        return batch
```
**EN:** This class models `LTX2TextConnectorStage` as a specialization of `PipelineStage`. Stage for applying LTX-2 Text Connectors to split/transform text embeddings Important methods include `__init__`, `forward`.
**CN:** 该类实现 `LTX2TextConnectorStage`，并继承/扩展 `PipelineStage`。 文档字符串指出：Stage for applying LTX-2 Text Connectors to split/transform text embeddings 其中较重要的方法包括 `__init__`, `forward`。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Pipeline orchestration / 流水线编排
- Video generation flow / 视频生成流程
- Audio generation flow / 音频生成流程

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.managers.forward_context`, `sglang.multimodal_gen.runtime.pipelines_core.schedule_batch`, `sglang.multimodal_gen.runtime.pipelines_core.stages.base`, `sglang.multimodal_gen.runtime.server_args`
- **External / 外部**: `torch`
