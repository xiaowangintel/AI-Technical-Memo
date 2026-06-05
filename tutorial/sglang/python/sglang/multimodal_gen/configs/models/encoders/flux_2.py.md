# flux_2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/models/encoders/flux_2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `Flux2MistralTextArchConfig`, `Flux2MistralTextConfig`, and `build_flux2_text_messages`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: FLUX.2 Mistral text encoder configuration and prompt formatting. / 该文件属于配置层。它围绕 `Flux2MistralTextArchConfig`、`Flux2MistralTextConfig` 和 `build_flux2_text_messages` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 2-10: module setup and imports / 模块初始化与导入
```python
"""FLUX.2 Mistral text encoder configuration and prompt formatting."""

from dataclasses import dataclass, field

from sglang.multimodal_gen.configs.models.encoders.base import (
    TextEncoderArchConfig,
    TextEncoderConfig,
)
from sglang.multimodal_gen.configs.models.fsdp import is_layer
```
**EN:** This block establishes the module context and imports `dataclasses`, `sglang.multimodal_gen.configs.models.encoders.base`, and `sglang.multimodal_gen.configs.models.fsdp`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`sglang.multimodal_gen.configs.models.encoders.base` 和 `sglang.multimodal_gen.configs.models.fsdp`。这些依赖为后续实现提供所需符号。

### Lines 12-15: supporting statements / 辅助语句
```python
FLUX_2_SYSTEM_MESSAGE = (
    "You are an AI that reasons about image descriptions. You give structured responses focusing on object relationships, object\n"
    "attribution and actions without speculation."
)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `FLUX_2_SYSTEM_MESSAGE`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `FLUX_2_SYSTEM_MESSAGE` 等名称。

### Lines 18-29: `build_flux2_text_messages` implementation / `build_flux2_text_messages` 实现
```python
def build_flux2_text_messages(prompts: list[str]) -> list[list[dict]]:
    cleaned_prompts = [prompt.replace("[IMG]", "") for prompt in prompts]
    return [
        [
            {
                "role": "system",
                "content": [{"type": "text", "text": FLUX_2_SYSTEM_MESSAGE}],
            },
            {"role": "user", "content": [{"type": "text", "text": prompt}]},
        ]
        for prompt in cleaned_prompts
    ]
```
**EN:** This block defines function `build_flux2_text_messages`. It builds flux2 text messages. Key calls include `prompt.replace`. Parameters such as `prompts` drive the behavior in this section.
**CN:** 该代码块定义了函数 `build_flux2_text_messages`。 它用于构建flux2 text messages。 关键调用包括 `prompt.replace`。 本段逻辑主要由 `prompts` 等参数驱动。

### Lines 33-33: `Flux2MistralTextArchConfig` class overview / `Flux2MistralTextArchConfig` 类概览
```python
class Flux2MistralTextArchConfig(TextEncoderArchConfig):
```
**EN:** This block defines class `Flux2MistralTextArchConfig`. It encapsulates flux2 mistral text arch config behavior. It inherits from `TextEncoderArchConfig`.
**CN:** 该代码块定义了类 `Flux2MistralTextArchConfig`。 它用于封装 flux2 mistral text arch config 相关行为。 它继承自 `TextEncoderArchConfig`。

### Lines 34-41: supporting statements / 辅助语句
```python
    stacked_params_mapping: list[tuple[str, str, str]] = field(
        default_factory=lambda: [
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
        ]
    )
    _fsdp_shard_conditions: list = field(default_factory=lambda: [is_layer])
```
**EN:** This block gathers supporting statements inside `Flux2MistralTextArchConfig`. It updates names such as `stacked_params_mapping`, and `_fsdp_shard_conditions`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `Flux2MistralTextArchConfig` 内部的辅助语句。 它会更新 `stacked_params_mapping` 和 `_fsdp_shard_conditions` 等名称。 代码会与 `field` 协同工作。

### Lines 43-51: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self) -> None:
        self.tokenizer_kwargs = {
            "padding": "max_length",
            "truncation": True,
            "max_length": 512,
            "add_special_tokens": True,
            "return_attention_mask": True,
            "return_tensors": "pt",
        }
```
**EN:** This block defines method `__post_init__` on `Flux2MistralTextArchConfig`. It post-processes init.
**CN:** 该代码块定义了 `Flux2MistralTextArchConfig` 的方法 `__post_init__`。 它用于后处理init。

### Lines 55-55: `Flux2MistralTextConfig` class overview / `Flux2MistralTextConfig` 类概览
```python
class Flux2MistralTextConfig(TextEncoderConfig):
```
**EN:** This block defines class `Flux2MistralTextConfig`. It encapsulates flux2 mistral text config behavior. It inherits from `TextEncoderConfig`.
**CN:** 该代码块定义了类 `Flux2MistralTextConfig`。 它用于封装 flux2 mistral text config 相关行为。 它继承自 `TextEncoderConfig`。

### Lines 56-59: supporting statements / 辅助语句
```python
    arch_config: TextEncoderArchConfig = field(
        default_factory=Flux2MistralTextArchConfig
    )
    prefix: str = "flux_2_mistral"
```
**EN:** This block gathers supporting statements inside `Flux2MistralTextConfig`. It updates names such as `arch_config`, and `prefix`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `Flux2MistralTextConfig` 内部的辅助语句。 它会更新 `arch_config` 和 `prefix` 等名称。 代码会与 `field` 协同工作。

## Key Concepts / 关键概念
- `build_flux2_text_messages`: Top-level function that builds flux2 text messages. / 顶层函数，用于构建flux2 text messages。
- `Flux2MistralTextArchConfig`: Primary class that encapsulates flux2 mistral text arch config behavior. / 核心类，用于封装 flux2 mistral text arch config 相关行为。
- `Flux2MistralTextConfig`: Primary class that encapsulates flux2 mistral text config behavior. / 核心类，用于封装 flux2 mistral text config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.encoders.base`, `sglang.multimodal_gen.configs.models.fsdp`

- **Total lines / 总行数**: 59
