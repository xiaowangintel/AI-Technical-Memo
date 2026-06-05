# qwen3.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/models/encoders/qwen3.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `Qwen3TextArchConfig`, and `Qwen3TextConfig`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Qwen3 text encoder configuration for SGLang diffusion models. / 该文件属于配置层。它围绕 `Qwen3TextArchConfig` 和 `Qwen3TextConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 2-14: module setup and imports / 模块初始化与导入
```python
"""Qwen3 text encoder configuration for SGLang diffusion models."""

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

### Lines 18-23: `Qwen3TextArchConfig` class overview / `Qwen3TextArchConfig` 类概览
```python
class Qwen3TextArchConfig(TextEncoderArchConfig):
    """Architecture config for Qwen3 text encoder.

    Qwen3 is similar to LLaMA but with QK-Norm (RMSNorm on Q and K before attention).
    """
```
**EN:** This block defines class `Qwen3TextArchConfig`. Architecture config for Qwen3 text encoder. Qwen3 is similar to LLaMA but with QK-Norm (RMSNorm on Q and K before attention). It inherits from `TextEncoderArchConfig`.
**CN:** 该代码块定义了类 `Qwen3TextArchConfig`。 它用于封装 qwen3 text arch config 相关行为。 它继承自 `TextEncoderArchConfig`。

### Lines 24-63: supporting statements / 辅助语句
```python
    vocab_size: int = 151936
    hidden_size: int = 2560
    intermediate_size: int = 9728
    num_hidden_layers: int = 36
    num_attention_heads: int = 32
    num_key_value_heads: int = 8
    hidden_act: str = "silu"
    max_position_embeddings: int = 40960
    initializer_range: float = 0.02
    rms_norm_eps: float = 1e-6
    use_cache: bool = True
    pad_token_id: int = 151643
    bos_token_id: int = 151643
    eos_token_id: int = 151645
    tie_word_embeddings: bool = True
    rope_theta: float = 1000000.0
    rope_scaling: dict | None = None
    attention_bias: bool = False
    attention_dropout: float = 0.0
    mlp_bias: bool = False
    head_dim: int = 128
    text_len: int = 512
    output_hidden_states: bool = True  # Klein needs hidden states from layers 9, 18, 27

    # Stacked params for weight loading with tensor parallelism
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

    # FSDP sharding conditions for CPU offload
    _fsdp_shard_conditions: list = field(
        default_factory=lambda: [is_layer, is_embed_tokens, is_final_norm]
    )
```
**EN:** This block gathers supporting statements inside `Qwen3TextArchConfig`. It updates names such as `vocab_size`, `hidden_size`, `intermediate_size`, `num_hidden_layers`, `num_attention_heads`, and `num_key_value_heads`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `Qwen3TextArchConfig` 内部的辅助语句。 它会更新 `vocab_size`、`hidden_size`、`intermediate_size`、`num_hidden_layers`、`num_attention_heads` 和 `num_key_value_heads` 等名称。 代码会与 `field` 协同工作。

### Lines 65-71: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self) -> None:
        self.tokenizer_kwargs = {
            "padding": "max_length",
            "truncation": True,
            "max_length": self.text_len,
            "return_tensors": "pt",
        }
```
**EN:** This block defines method `__post_init__` on `Qwen3TextArchConfig`. It post-processes init.
**CN:** 该代码块定义了 `Qwen3TextArchConfig` 的方法 `__post_init__`。 它用于后处理init。

### Lines 75-77: `Qwen3TextConfig` class overview / `Qwen3TextConfig` 类概览
```python
class Qwen3TextConfig(TextEncoderConfig):
    """Top-level config for Qwen3 text encoder."""
```
**EN:** This block defines class `Qwen3TextConfig`. Top-level config for Qwen3 text encoder. It inherits from `TextEncoderConfig`.
**CN:** 该代码块定义了类 `Qwen3TextConfig`。 它用于封装 qwen3 text config 相关行为。 它继承自 `TextEncoderConfig`。

### Lines 78-79: supporting statements / 辅助语句
```python
    arch_config: TextEncoderArchConfig = field(default_factory=Qwen3TextArchConfig)
    prefix: str = "qwen3"
```
**EN:** This block gathers supporting statements inside `Qwen3TextConfig`. It updates names such as `arch_config`, and `prefix`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `Qwen3TextConfig` 内部的辅助语句。 它会更新 `arch_config` 和 `prefix` 等名称。 代码会与 `field` 协同工作。

## Key Concepts / 关键概念
- `Qwen3TextArchConfig`: Architecture config for Qwen3 text encoder. / 核心类，用于封装 qwen3 text arch config 相关行为。
- `Qwen3TextConfig`: Top-level config for Qwen3 text encoder. / 核心类，用于封装 qwen3 text config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.encoders.base`, `sglang.multimodal_gen.configs.models.fsdp`

- **Total lines / 总行数**: 79
