# qwen3vl.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/models/encoders/qwen3vl.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `Qwen3VLArchConfig`, and `Qwen3VLConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `Qwen3VLArchConfig` 和 `Qwen3VLConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 3-13: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field

from sglang.multimodal_gen.configs.models.encoders.base import (
    TextEncoderArchConfig,
    TextEncoderConfig,
)
from sglang.multimodal_gen.configs.models.fsdp import (
    is_embed_tokens,
    is_final_norm,
    is_layer,
)
```
**EN:** This block establishes the module context and imports `dataclasses`, `sglang.multimodal_gen.configs.models.encoders.base`, and `sglang.multimodal_gen.configs.models.fsdp`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`sglang.multimodal_gen.configs.models.encoders.base` 和 `sglang.multimodal_gen.configs.models.fsdp`。这些依赖为后续实现提供所需符号。

### Lines 17-23: `Qwen3VLArchConfig` class overview / `Qwen3VLArchConfig` 类概览
```python
class Qwen3VLArchConfig(TextEncoderArchConfig):
    """Architecture configuration for Qwen3-VL text encoder.

    Qwen3-VL-8B-Instruct is used by JoyImage model.
    Architecture is similar to Qwen2.5-VL but with Qwen3 improvements.
    """
```
**EN:** This block defines class `Qwen3VLArchConfig`. Architecture configuration for Qwen3-VL text encoder. Qwen3-VL-8B-Instruct is used by JoyImage model. It inherits from `TextEncoderArchConfig`.
**CN:** 该代码块定义了类 `Qwen3VLArchConfig`。 它用于封装 qwen3 vlarch config 相关行为。 它继承自 `TextEncoderArchConfig`。

### Lines 24-67: supporting statements / 辅助语句
```python
    vocab_size: int = 32000
    hidden_size: int = 4096
    intermediate_size: int = 11008
    num_hidden_layers: int = 32
    num_attention_heads: int = 32
    num_key_value_heads: int | None = None
    hidden_act: str = "silu"
    max_position_embeddings: int = 2048
    initializer_range: float = 0.02
    rms_norm_eps: float = 1e-6
    use_cache: bool = True
    pad_token_id: int = -1
    eos_token_id: int = 2
    pretraining_tp: int = 1
    tie_word_embeddings: bool = False
    rope_theta: float = 10000.0
    rope_scaling: float | None = None
    attention_bias: bool = False
    attention_dropout: float = 0.0
    mlp_bias: bool = False
    head_dim: int | None = None
    hidden_state_skip_layer: int = 2
    text_len: int = 2048

    stacked_params_mapping: list[tuple[str, str, str]] = field(
        default_factory=lambda: [
            # (param_name, shard_name, shard_id)
            (".qkv_proj", ".q_proj", "q"),
            (".qkv_proj", ".k_proj", "k"),
            (".qkv_proj", ".v_proj", "v"),
            (".gate_up_proj", ".gate_proj", 0),
            (".gate_up_proj", ".up_proj", 1),
        ]
    )
    _fsdp_shard_conditions: list = field(
        default_factory=lambda: [is_layer, is_embed_tokens, is_final_norm]
    )

    # JoyImage specific settings
    text_token_max_length: int = 2048
    prompt_template_encode_start_idx = {
        "image": 34,
        "video": 91,
    }
```
**EN:** This block gathers supporting statements inside `Qwen3VLArchConfig`. It updates names such as `vocab_size`, `hidden_size`, `intermediate_size`, `num_hidden_layers`, `num_attention_heads`, and `num_key_value_heads`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `Qwen3VLArchConfig` 内部的辅助语句。 它会更新 `vocab_size`、`hidden_size`、`intermediate_size`、`num_hidden_layers`、`num_attention_heads` 和 `num_key_value_heads` 等名称。 代码会与 `field` 协同工作。

### Lines 69-77: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        super().__post_init__()
        self.tokenizer_kwargs = {
            "padding": True,
            "truncation": True,
            "max_length": self.text_len
            + self.prompt_template_encode_start_idx["image"],
            "return_tensors": "pt",
        }
```
**EN:** This block defines method `__post_init__` on `Qwen3VLArchConfig`. It post-processes init. Key calls include `super.__post_init__`, and `super`.
**CN:** 该代码块定义了 `Qwen3VLArchConfig` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `super.__post_init__` 和 `super`。

### Lines 81-86: `Qwen3VLConfig` class overview / `Qwen3VLConfig` 类概览
```python
class Qwen3VLConfig(TextEncoderConfig):
    """Configuration for Qwen3-VL text encoder.

    Used by JoyImage model.
    """
```
**EN:** This block defines class `Qwen3VLConfig`. Configuration for Qwen3-VL text encoder. Used by JoyImage model. It inherits from `TextEncoderConfig`.
**CN:** 该代码块定义了类 `Qwen3VLConfig`。 它用于封装 qwen3 vlconfig 相关行为。 它继承自 `TextEncoderConfig`。

### Lines 87-87: supporting statements / 辅助语句
```python
    arch_config: TextEncoderArchConfig = field(default_factory=Qwen3VLArchConfig)
```
**EN:** This block gathers supporting statements inside `Qwen3VLConfig`. It updates names such as `arch_config`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `Qwen3VLConfig` 内部的辅助语句。 它会更新 `arch_config` 等名称。 代码会与 `field` 协同工作。

## Key Concepts / 关键概念
- `Qwen3VLArchConfig`: Architecture configuration for Qwen3-VL text encoder. / 核心类，用于封装 qwen3 vlarch config 相关行为。
- `Qwen3VLConfig`: Configuration for Qwen3-VL text encoder. / 核心类，用于封装 qwen3 vlconfig 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.encoders.base`, `sglang.multimodal_gen.configs.models.fsdp`

- **Total lines / 总行数**: 87
