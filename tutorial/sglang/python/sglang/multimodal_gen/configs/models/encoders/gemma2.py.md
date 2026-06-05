# gemma2.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/models/encoders/gemma2.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `Gemma2ArchConfig`, and `Gemma2Config`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `Gemma2ArchConfig` 和 `Gemma2Config` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 11-21: module setup and imports / 模块初始化与导入
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

### Lines 25-25: `Gemma2ArchConfig` class overview / `Gemma2ArchConfig` 类概览
```python
class Gemma2ArchConfig(TextEncoderArchConfig):
```
**EN:** This block defines class `Gemma2ArchConfig`. It encapsulates gemma2 arch config behavior. It inherits from `TextEncoderArchConfig`.
**CN:** 该代码块定义了类 `Gemma2ArchConfig`。 它用于封装 gemma2 arch config 相关行为。 它继承自 `TextEncoderArchConfig`。

### Lines 26-74: supporting statements / 辅助语句
```python
    vocab_size: int = 256000
    hidden_size: int = 2304
    intermediate_size: int = 9216
    num_hidden_layers: int = 26
    num_attention_heads: int = 8
    num_key_value_heads: int = 4
    head_dim: int = 256
    hidden_act: str = "gelu_pytorch_tanh"
    hidden_activation: str = "gelu_pytorch_tanh"
    max_position_embeddings: int = 8192
    rms_norm_eps: float = 1e-6
    use_cache: bool = True
    pad_token_id: int = 0
    eos_token_id: int = 1
    bos_token_id: int = 2
    tie_word_embeddings: bool = True
    rope_theta: float = 10000.0
    attention_bias: bool = False
    attention_dropout: float = 0.0

    # Gemma2 alternates between global and sliding-window attention
    # on odd/even layers, respectively.
    sliding_window: int = 4096

    # query_pre_attn_scalar replaces the standard 1/sqrt(head_dim) scaling.
    query_pre_attn_scalar: int = 256

    # Softcapping bounds raw attention logits via tanh(logits/cap)*cap.
    # NOTE: SDPA does not natively support softcapping; the runtime model
    # currently skips this (see Gemma2Attention.forward). Quality impact
    # is minimal for short text-encoder sequences but should be revisited
    # for longer context.
    attn_logit_softcapping: float = 50.0
    final_logit_softcapping: float = 30.0

    text_len: int = 300

    stacked_params_mapping: list[tuple[str, str, str]] = field(
        default_factory=lambda: [
            (".qkv_proj", ".q_proj", "q"),
            (".qkv_proj", ".k_proj", "k"),
            (".qkv_proj", ".v_proj", "v"),
            (".gate_up_proj", ".gate_proj", "0"),
            (".gate_up_proj", ".up_proj", "1"),
        ]
    )
    _fsdp_shard_conditions: list = field(
        default_factory=lambda: [is_layer, is_embed_tokens, is_final_norm]
    )
```
**EN:** This block gathers supporting statements inside `Gemma2ArchConfig`. It updates names such as `vocab_size`, `hidden_size`, `intermediate_size`, `num_hidden_layers`, `num_attention_heads`, and `num_key_value_heads`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `Gemma2ArchConfig` 内部的辅助语句。 它会更新 `vocab_size`、`hidden_size`、`intermediate_size`、`num_hidden_layers`、`num_attention_heads` 和 `num_key_value_heads` 等名称。 代码会与 `field` 协同工作。

### Lines 78-78: `Gemma2Config` class overview / `Gemma2Config` 类概览
```python
class Gemma2Config(TextEncoderConfig):
```
**EN:** This block defines class `Gemma2Config`. It encapsulates gemma2 config behavior. It inherits from `TextEncoderConfig`.
**CN:** 该代码块定义了类 `Gemma2Config`。 它用于封装 gemma2 config 相关行为。 它继承自 `TextEncoderConfig`。

### Lines 79-80: supporting statements / 辅助语句
```python
    arch_config: TextEncoderArchConfig = field(default_factory=Gemma2ArchConfig)
    prefix: str = "gemma_2"
```
**EN:** This block gathers supporting statements inside `Gemma2Config`. It updates names such as `arch_config`, and `prefix`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `Gemma2Config` 内部的辅助语句。 它会更新 `arch_config` 和 `prefix` 等名称。 代码会与 `field` 协同工作。

## Key Concepts / 关键概念
- `Gemma2ArchConfig`: Primary class that encapsulates gemma2 arch config behavior. / 核心类，用于封装 gemma2 arch config 相关行为。
- `Gemma2Config`: Primary class that encapsulates gemma2 config behavior. / 核心类，用于封装 gemma2 config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.encoders.base`, `sglang.multimodal_gen.configs.models.fsdp`

- **Total lines / 总行数**: 80
