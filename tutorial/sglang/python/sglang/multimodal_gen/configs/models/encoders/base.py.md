# base.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/models/encoders/base.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `EncoderArchConfig`, `TextEncoderArchConfig`, and `ImageEncoderArchConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `EncoderArchConfig`、`TextEncoderArchConfig` 和 `ImageEncoderArchConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-11: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field
from typing import Any

import torch

from sglang.multimodal_gen.configs.models.base import ArchConfig, ModelConfig
from sglang.multimodal_gen.runtime.layers.quantization import QuantizationConfig
from sglang.multimodal_gen.runtime.platforms import AttentionBackendEnum
```
**EN:** This block establishes the module context and imports `dataclasses`, `typing`, `torch`, `sglang.multimodal_gen.configs.models.base`, `sglang.multimodal_gen.runtime.layers.quantization`, and `sglang.multimodal_gen.runtime.platforms`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`typing`、`torch`、`sglang.multimodal_gen.configs.models.base`、`sglang.multimodal_gen.runtime.layers.quantization` 和 `sglang.multimodal_gen.runtime.platforms`。这些依赖为后续实现提供所需符号。

### Lines 15-15: `EncoderArchConfig` class overview / `EncoderArchConfig` 类概览
```python
class EncoderArchConfig(ArchConfig):
```
**EN:** This block defines class `EncoderArchConfig`. It encapsulates encoder arch config behavior. It inherits from `ArchConfig`.
**CN:** 该代码块定义了类 `EncoderArchConfig`。 它用于封装 encoder arch config 相关行为。 它继承自 `ArchConfig`。

### Lines 16-26: supporting statements / 辅助语句
```python
    _fsdp_shard_conditions: list = field(default_factory=lambda: [])
    architectures: list[str] = field(default_factory=lambda: [])
    _supported_attention_backends: set[AttentionBackendEnum] = field(
        default_factory=lambda: {
            AttentionBackendEnum.FA,
            AttentionBackendEnum.TORCH_SDPA,
            AttentionBackendEnum.SAGE_ATTN_3,
        }
    )
    output_hidden_states: bool = False
    use_return_dict: bool = True
```
**EN:** This block gathers supporting statements inside `EncoderArchConfig`. It updates names such as `_fsdp_shard_conditions`, `architectures`, `_supported_attention_backends`, `output_hidden_states`, and `use_return_dict`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `EncoderArchConfig` 内部的辅助语句。 它会更新 `_fsdp_shard_conditions`、`architectures`、`_supported_attention_backends`、`output_hidden_states` 和 `use_return_dict` 等名称。 代码会与 `field` 协同工作。

### Lines 30-30: `TextEncoderArchConfig` class overview / `TextEncoderArchConfig` 类概览
```python
class TextEncoderArchConfig(EncoderArchConfig):
```
**EN:** This block defines class `TextEncoderArchConfig`. It encapsulates text encoder arch config behavior. It inherits from `EncoderArchConfig`.
**CN:** 该代码块定义了类 `TextEncoderArchConfig`。 它用于封装 text encoder arch config 相关行为。 它继承自 `EncoderArchConfig`。

### Lines 31-47: supporting statements / 辅助语句
```python
    vocab_size: int = 0
    hidden_size: int = 0
    num_hidden_layers: int = 0
    num_attention_heads: int = 0
    pad_token_id: int = 0
    eos_token_id: int = 0
    text_len: int = 0
    hidden_state_skip_layer: int = 0
    decoder_start_token_id: int = 0
    output_past: bool = True
    scalable_attention: bool = True
    tie_word_embeddings: bool = False
    stacked_params_mapping: list[tuple[str, str, str]] = field(
        default_factory=list
    )  # mapping from huggingface weight names to custom names
    tokenizer_kwargs: dict[str, Any] = field(default_factory=dict)
    _fsdp_shard_conditions: list = field(default_factory=lambda: [])
```
**EN:** This block gathers supporting statements inside `TextEncoderArchConfig`. It updates names such as `vocab_size`, `hidden_size`, `num_hidden_layers`, `num_attention_heads`, `pad_token_id`, and `eos_token_id`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `TextEncoderArchConfig` 内部的辅助语句。 它会更新 `vocab_size`、`hidden_size`、`num_hidden_layers`、`num_attention_heads`、`pad_token_id` 和 `eos_token_id` 等名称。 代码会与 `field` 协同工作。

### Lines 49-54: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self) -> None:
        self.tokenizer_kwargs = {
            "truncation": True,
            "max_length": self.text_len,
            "return_tensors": "pt",
        }
```
**EN:** This block defines method `__post_init__` on `TextEncoderArchConfig`. It post-processes init.
**CN:** 该代码块定义了 `TextEncoderArchConfig` 的方法 `__post_init__`。 它用于后处理init。

### Lines 58-58: `ImageEncoderArchConfig` class overview / `ImageEncoderArchConfig` 类概览
```python
class ImageEncoderArchConfig(EncoderArchConfig):
```
**EN:** This block defines class `ImageEncoderArchConfig`. It encapsulates image encoder arch config behavior. It inherits from `EncoderArchConfig`.
**CN:** 该代码块定义了类 `ImageEncoderArchConfig`。 它用于封装 image encoder arch config 相关行为。 它继承自 `EncoderArchConfig`。

### Lines 59-59: supporting statements / 辅助语句
```python
    pass
```
**EN:** This block gathers supporting statements inside `ImageEncoderArchConfig`.
**CN:** 该代码块汇集了位于 `ImageEncoderArchConfig` 内部的辅助语句。

### Lines 63-63: `BaseEncoderOutput` class overview / `BaseEncoderOutput` 类概览
```python
class BaseEncoderOutput:
```
**EN:** This block defines class `BaseEncoderOutput`. It encapsulates base encoder output behavior.
**CN:** 该代码块定义了类 `BaseEncoderOutput`。 它用于封装 base encoder output 相关行为。

### Lines 64-68: supporting statements / 辅助语句
```python
    last_hidden_state: torch.FloatTensor | None = None
    pooler_output: torch.FloatTensor | None = None
    hidden_states: tuple[torch.FloatTensor, ...] | None = None
    attentions: tuple[torch.FloatTensor, ...] | None = None
    attention_mask: torch.Tensor | None = None
```
**EN:** This block gathers supporting statements inside `BaseEncoderOutput`. It updates names such as `last_hidden_state`, `pooler_output`, `hidden_states`, `attentions`, and `attention_mask`.
**CN:** 该代码块汇集了位于 `BaseEncoderOutput` 内部的辅助语句。 它会更新 `last_hidden_state`、`pooler_output`、`hidden_states`、`attentions` 和 `attention_mask` 等名称。

### Lines 72-72: `EncoderConfig` class overview / `EncoderConfig` 类概览
```python
class EncoderConfig(ModelConfig):
```
**EN:** This block defines class `EncoderConfig`. It encapsulates encoder config behavior. It inherits from `ModelConfig`.
**CN:** 该代码块定义了类 `EncoderConfig`。 它用于封装 encoder config 相关行为。 它继承自 `ModelConfig`。

### Lines 73-77: supporting statements / 辅助语句
```python
    arch_config: ArchConfig = field(default_factory=EncoderArchConfig)

    prefix: str = ""
    quant_config: QuantizationConfig | None = None
    lora_config: Any | None = None
```
**EN:** This block gathers supporting statements inside `EncoderConfig`. It updates names such as `arch_config`, `prefix`, `quant_config`, and `lora_config`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `EncoderConfig` 内部的辅助语句。 它会更新 `arch_config`、`prefix`、`quant_config` 和 `lora_config` 等名称。 代码会与 `field` 协同工作。

### Lines 81-81: `TextEncoderConfig` class overview / `TextEncoderConfig` 类概览
```python
class TextEncoderConfig(EncoderConfig):
```
**EN:** This block defines class `TextEncoderConfig`. It encapsulates text encoder config behavior. It inherits from `EncoderConfig`.
**CN:** 该代码块定义了类 `TextEncoderConfig`。 它用于封装 text encoder config 相关行为。 它继承自 `EncoderConfig`。

### Lines 82-87: supporting statements / 辅助语句
```python
    arch_config: ArchConfig = field(default_factory=TextEncoderArchConfig)

    # Use the SP Group of the transformer as the TP Group of T5.
    parallel_folding: bool = False
    # "sp" or "ulysses" or "ring"
    parallel_folding_mode: str = "sp"
```
**EN:** This block gathers supporting statements inside `TextEncoderConfig`. It updates names such as `arch_config`, `parallel_folding`, and `parallel_folding_mode`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `TextEncoderConfig` 内部的辅助语句。 它会更新 `arch_config`、`parallel_folding` 和 `parallel_folding_mode` 等名称。 代码会与 `field` 协同工作。

### Lines 91-91: `ImageEncoderConfig` class overview / `ImageEncoderConfig` 类概览
```python
class ImageEncoderConfig(EncoderConfig):
```
**EN:** This block defines class `ImageEncoderConfig`. It encapsulates image encoder config behavior. It inherits from `EncoderConfig`.
**CN:** 该代码块定义了类 `ImageEncoderConfig`。 它用于封装 image encoder config 相关行为。 它继承自 `EncoderConfig`。

### Lines 92-92: supporting statements / 辅助语句
```python
    arch_config: ArchConfig = field(default_factory=ImageEncoderArchConfig)
```
**EN:** This block gathers supporting statements inside `ImageEncoderConfig`. It updates names such as `arch_config`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `ImageEncoderConfig` 内部的辅助语句。 它会更新 `arch_config` 等名称。 代码会与 `field` 协同工作。

## Key Concepts / 关键概念
- `EncoderArchConfig`: Primary class that encapsulates encoder arch config behavior. / 核心类，用于封装 encoder arch config 相关行为。
- `TextEncoderArchConfig`: Primary class that encapsulates text encoder arch config behavior. / 核心类，用于封装 text encoder arch config 相关行为。
- `ImageEncoderArchConfig`: Primary class that encapsulates image encoder arch config behavior. / 核心类，用于封装 image encoder arch config 相关行为。
- `BaseEncoderOutput`: Primary class that encapsulates base encoder output behavior. / 核心类，用于封装 base encoder output 相关行为。
- `EncoderConfig`: Primary class that encapsulates encoder config behavior. / 核心类，用于封装 encoder config 相关行为。
- `TextEncoderConfig`: Primary class that encapsulates text encoder config behavior. / 核心类，用于封装 text encoder config 相关行为。
- `ImageEncoderConfig`: Primary class that encapsulates image encoder config behavior. / 核心类，用于封装 image encoder config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `torch`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.base`, `sglang.multimodal_gen.runtime.layers.quantization`, `sglang.multimodal_gen.runtime.platforms`

- **Total lines / 总行数**: 92
