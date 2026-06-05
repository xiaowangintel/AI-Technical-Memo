# qwen3_rm.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/models/qwen3_rm.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Qwen3 Reward Model for RLHF and best-of-N sampling. This file provides runtime-ready implementation details for SGLang inference. / 该文件围绕上述主题实现了可直接用于 SGLang 推理运行时的具体逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 14-14: Module docstring
```python
"""Qwen3 Reward Model for RLHF and best-of-N sampling."""
```
**EN:** States the module scope and documents the high-level intention of the implementation.
**CN:** 说明模块范围，并给出实现的高层设计意图。

### Lines 16-23: Module imports
```python
from typing import Optional

from torch import nn
from transformers import Qwen2Config  # Qwen3 uses Qwen2Config

from sglang.srt.layers.pooler import Pooler, PoolingType
from sglang.srt.layers.quantization.base_config import QuantizationConfig
from sglang.srt.models.qwen3_classification import Qwen3ForPooledOutput
```
**EN:** Imports the dependencies required by this file, especially PyTorch tensors/layers, Hugging Face config/model helpers, SGLang runtime components.
**CN:** 导入该文件所需的依赖，重点包括PyTorch 张量与层、Hugging Face 配置/模型辅助工具、SGLang 运行时组件。

### Lines 26-28: Class `Qwen3ForRewardModel` overview
```python
class Qwen3ForRewardModel(Qwen3ForPooledOutput):
    """Qwen3 Reward Model with 2-layer MLP scoring head for RLHF."""
```
**EN:** Defines `Qwen3ForRewardModel` as a reusable runtime type derived from Qwen3ForPooledOutput. The class groups 1 method(s) that implement model-specific behavior.
**CN:** 该代码块定义了可复用的运行时类型 `Qwen3ForRewardModel`，其继承关系为 Qwen3ForPooledOutput。这个类组织了 1 个方法，用于实现模型相关行为。

### Lines 29-42: Method `Qwen3ForRewardModel.__init__`
```python
    def __init__(
        self,
        config: Qwen2Config,
        quant_config: Optional[QuantizationConfig] = None,
        prefix: str = "",
    ) -> None:
        super().__init__(config, quant_config, prefix)
        self.num_labels = 1
        self.score = nn.Sequential(
            nn.Linear(config.hidden_size, config.hidden_size),
            nn.ReLU(),
            nn.Linear(config.hidden_size, self.num_labels),
        )
        self.pooler = Pooler(pooling_type=PoolingType.LAST, normalize=False)
```
**EN:** This method implements `__init__(config: ..., quant_config: ...=..., prefix: ...=...)` and initializes the object, stores configuration, and constructs the submodules needed later.
**CN:** 这个方法实现了 `__init__(config: ..., quant_config: ...=..., prefix: ...=...)`，其作用是初始化对象、保存配置，并构建后续执行所需的子模块。

### Lines 45-47: Top-level assign
```python
EntryClass = [
    Qwen3ForRewardModel,
]
```
**EN:** Publishes the entry class name(s) that SGLang uses when resolving this module through the model registry.
**CN:** 发布 SGLang 通过模型注册表解析该模块时使用的入口类名称。

## Key Concepts / 关键概念
- **EN:** Quantization-aware inference layers / **CN:** 量化感知推理层
- **EN:** Dynamic model registration / **CN:** 动态模型注册
- **EN:** Distributed/parallel inference layout / **CN:** 分布式/并行推理布局
- **EN:** Task-specific output heads / **CN:** 任务特定输出头

## Dependencies / 依赖关系
- `typing.Optional`
- `torch.nn`
- `transformers.Qwen2Config`
- `sglang.srt.layers.pooler.Pooler`
- `sglang.srt.layers.pooler.PoolingType`
- `sglang.srt.layers.quantization.base_config.QuantizationConfig`
- `sglang.srt.models.qwen3_classification.Qwen3ForPooledOutput`
