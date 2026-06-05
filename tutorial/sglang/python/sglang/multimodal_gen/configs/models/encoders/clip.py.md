# clip.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/models/encoders/clip.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `CLIPTextArchConfig`, `CLIPVisionArchConfig`, and `CLIPTextConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `CLIPTextArchConfig`、`CLIPVisionArchConfig` 和 `CLIPTextConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-16: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field

from sglang.multimodal_gen.configs.models.encoders.base import (
    ImageEncoderArchConfig,
    ImageEncoderConfig,
    TextEncoderArchConfig,
    TextEncoderConfig,
)
from sglang.multimodal_gen.configs.models.fsdp import (
    is_embeddings,
    is_layer,
)
from sglang.multimodal_gen.runtime.platforms import AttentionBackendEnum
```
**EN:** This block establishes the module context and imports `dataclasses`, `sglang.multimodal_gen.configs.models.encoders.base`, `sglang.multimodal_gen.configs.models.fsdp`, and `sglang.multimodal_gen.runtime.platforms`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`sglang.multimodal_gen.configs.models.encoders.base`、`sglang.multimodal_gen.configs.models.fsdp` 和 `sglang.multimodal_gen.runtime.platforms`。这些依赖为后续实现提供所需符号。

### Lines 20-20: `CLIPTextArchConfig` class overview / `CLIPTextArchConfig` 类概览
```python
class CLIPTextArchConfig(TextEncoderArchConfig):
```
**EN:** This block defines class `CLIPTextArchConfig`. It encapsulates cliptext arch config behavior. It inherits from `TextEncoderArchConfig`.
**CN:** 该代码块定义了类 `CLIPTextArchConfig`。 它用于封装 cliptext arch config 相关行为。 它继承自 `TextEncoderArchConfig`。

### Lines 21-53: supporting statements / 辅助语句
```python
    vocab_size: int = 49408
    hidden_size: int = 512
    intermediate_size: int = 2048
    projection_dim: int = 512
    num_hidden_layers: int = 12
    num_attention_heads: int = 8
    max_position_embeddings: int = 77
    hidden_act: str = "quick_gelu"
    layer_norm_eps: float = 1e-5
    dropout: float = 0.0
    attention_dropout: float = 0.0
    initializer_range: float = 0.02
    initializer_factor: float = 1.0
    pad_token_id: int = 1
    bos_token_id: int = 49406
    eos_token_id: int = 49407
    text_len: int = 77
    _supported_attention_backends: set[AttentionBackendEnum] = field(
        default_factory=lambda: {
            AttentionBackendEnum.TORCH_SDPA,  # Force TORCH_SDPA to support attention_mask
        }
    )
    stacked_params_mapping: list[tuple[str, str, str]] = field(
        default_factory=lambda: [
            # (param_name, shard_name, shard_id)
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
        ]
    )
    _fsdp_shard_conditions: list = field(
        default_factory=lambda: [is_layer, is_embeddings]
    )
```
**EN:** This block gathers supporting statements inside `CLIPTextArchConfig`. It updates names such as `vocab_size`, `hidden_size`, `intermediate_size`, `projection_dim`, `num_hidden_layers`, and `num_attention_heads`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `CLIPTextArchConfig` 内部的辅助语句。 它会更新 `vocab_size`、`hidden_size`、`intermediate_size`、`projection_dim`、`num_hidden_layers` 和 `num_attention_heads` 等名称。 代码会与 `field` 协同工作。

### Lines 57-57: `CLIPVisionArchConfig` class overview / `CLIPVisionArchConfig` 类概览
```python
class CLIPVisionArchConfig(ImageEncoderArchConfig):
```
**EN:** This block defines class `CLIPVisionArchConfig`. It encapsulates clipvision arch config behavior. It inherits from `ImageEncoderArchConfig`.
**CN:** 该代码块定义了类 `CLIPVisionArchConfig`。 它用于封装 clipvision arch config 相关行为。 它继承自 `ImageEncoderArchConfig`。

### Lines 58-79: supporting statements / 辅助语句
```python
    hidden_size: int = 768
    intermediate_size: int = 3072
    projection_dim: int = 512
    num_hidden_layers: int = 12
    num_attention_heads: int = 12
    num_channels: int = 3
    image_size: int = 224
    patch_size: int = 32
    hidden_act: str = "quick_gelu"
    layer_norm_eps: float = 1e-5
    dropout: float = 0.0
    attention_dropout: float = 0.0
    initializer_range: float = 0.02
    initializer_factor: float = 1.0
    stacked_params_mapping: list[tuple[str, str, str]] = field(
        default_factory=lambda: [
            # (param_name, shard_name, shard_id)
            ("qkv_proj", "q_proj", "q"),
            ("qkv_proj", "k_proj", "k"),
            ("qkv_proj", "v_proj", "v"),
        ]
    )
```
**EN:** This block gathers supporting statements inside `CLIPVisionArchConfig`. It updates names such as `hidden_size`, `intermediate_size`, `projection_dim`, `num_hidden_layers`, `num_attention_heads`, and `num_channels`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `CLIPVisionArchConfig` 内部的辅助语句。 它会更新 `hidden_size`、`intermediate_size`、`projection_dim`、`num_hidden_layers`、`num_attention_heads` 和 `num_channels` 等名称。 代码会与 `field` 协同工作。

### Lines 83-83: `CLIPTextConfig` class overview / `CLIPTextConfig` 类概览
```python
class CLIPTextConfig(TextEncoderConfig):
```
**EN:** This block defines class `CLIPTextConfig`. It encapsulates cliptext config behavior. It inherits from `TextEncoderConfig`.
**CN:** 该代码块定义了类 `CLIPTextConfig`。 它用于封装 cliptext config 相关行为。 它继承自 `TextEncoderConfig`。

### Lines 84-88: supporting statements / 辅助语句
```python
    arch_config: TextEncoderArchConfig = field(default_factory=CLIPTextArchConfig)

    num_hidden_layers_override: int | None = None
    require_post_norm: bool | None = None
    prefix: str = "clip"
```
**EN:** This block gathers supporting statements inside `CLIPTextConfig`. It updates names such as `arch_config`, `num_hidden_layers_override`, `require_post_norm`, and `prefix`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `CLIPTextConfig` 内部的辅助语句。 它会更新 `arch_config`、`num_hidden_layers_override`、`require_post_norm` 和 `prefix` 等名称。 代码会与 `field` 协同工作。

### Lines 92-92: `CLIPVisionConfig` class overview / `CLIPVisionConfig` 类概览
```python
class CLIPVisionConfig(ImageEncoderConfig):
```
**EN:** This block defines class `CLIPVisionConfig`. It encapsulates clipvision config behavior. It inherits from `ImageEncoderConfig`.
**CN:** 该代码块定义了类 `CLIPVisionConfig`。 它用于封装 clipvision config 相关行为。 它继承自 `ImageEncoderConfig`。

### Lines 93-97: supporting statements / 辅助语句
```python
    arch_config: ImageEncoderArchConfig = field(default_factory=CLIPVisionArchConfig)

    num_hidden_layers_override: int | None = None
    require_post_norm: bool | None = None
    prefix: str = "clip"
```
**EN:** This block gathers supporting statements inside `CLIPVisionConfig`. It updates names such as `arch_config`, `num_hidden_layers_override`, `require_post_norm`, and `prefix`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `CLIPVisionConfig` 内部的辅助语句。 它会更新 `arch_config`、`num_hidden_layers_override`、`require_post_norm` 和 `prefix` 等名称。 代码会与 `field` 协同工作。

## Key Concepts / 关键概念
- `CLIPTextArchConfig`: Primary class that encapsulates cliptext arch config behavior. / 核心类，用于封装 cliptext arch config 相关行为。
- `CLIPVisionArchConfig`: Primary class that encapsulates clipvision arch config behavior. / 核心类，用于封装 clipvision arch config 相关行为。
- `CLIPTextConfig`: Primary class that encapsulates cliptext config behavior. / 核心类，用于封装 cliptext config 相关行为。
- `CLIPVisionConfig`: Primary class that encapsulates clipvision config behavior. / 核心类，用于封装 clipvision config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.encoders.base`, `sglang.multimodal_gen.configs.models.fsdp`, `sglang.multimodal_gen.runtime.platforms`

- **Total lines / 总行数**: 97
