# base.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/models/adapter/base.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `AdapterArchConfig`, and `AdapterConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `AdapterArchConfig` 和 `AdapterConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 2-6: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field
from typing import Any

from sglang.multimodal_gen.configs.models.base import ArchConfig, ModelConfig
from sglang.multimodal_gen.runtime.platforms import AttentionBackendEnum
```
**EN:** This block establishes the module context and imports `dataclasses`, `typing`, `sglang.multimodal_gen.configs.models.base`, and `sglang.multimodal_gen.runtime.platforms`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`typing`、`sglang.multimodal_gen.configs.models.base` 和 `sglang.multimodal_gen.runtime.platforms`。这些依赖为后续实现提供所需符号。

### Lines 10-10: `AdapterArchConfig` class overview / `AdapterArchConfig` 类概览
```python
class AdapterArchConfig(ArchConfig):
```
**EN:** This block defines class `AdapterArchConfig`. It encapsulates adapter arch config behavior. It inherits from `ArchConfig`.
**CN:** 该代码块定义了类 `AdapterArchConfig`。 它用于封装 adapter arch config 相关行为。 它继承自 `ArchConfig`。

### Lines 11-37: supporting statements / 辅助语句
```python
    _fsdp_shard_conditions: list = field(default_factory=list)
    _compile_conditions: list = field(default_factory=list)

    # convert weights name from HF-format to SGLang-dit-format
    param_names_mapping: dict = field(default_factory=dict)

    # Reverse mapping for saving checkpoints: custom -> hf
    reverse_param_names_mapping: dict = field(default_factory=dict)
    _supported_attention_backends: set[AttentionBackendEnum] = field(
        default_factory=lambda: {
            AttentionBackendEnum.SLIDING_TILE_ATTN,
            AttentionBackendEnum.SAGE_ATTN,
            AttentionBackendEnum.FA,
            AttentionBackendEnum.AITER,
            AttentionBackendEnum.AITER_SAGE,
            AttentionBackendEnum.TORCH_SDPA,
            AttentionBackendEnum.VIDEO_SPARSE_ATTN,
            AttentionBackendEnum.VMOBA_ATTN,
            AttentionBackendEnum.SAGE_ATTN_3,
        }
    )

    hidden_size: int = 0
    num_attention_heads: int = 0
    num_channels_latents: int = 0
    exclude_lora_layers: list[str] = field(default_factory=list)
    boundary_ratio: float | None = None
```
**EN:** This block gathers supporting statements inside `AdapterArchConfig`. It updates names such as `_fsdp_shard_conditions`, `_compile_conditions`, `param_names_mapping`, `reverse_param_names_mapping`, `_supported_attention_backends`, and `hidden_size`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `AdapterArchConfig` 内部的辅助语句。 它会更新 `_fsdp_shard_conditions`、`_compile_conditions`、`param_names_mapping`、`reverse_param_names_mapping`、`_supported_attention_backends` 和 `hidden_size` 等名称。 代码会与 `field` 协同工作。

### Lines 39-41: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self) -> None:
        if not self._compile_conditions:
            self._compile_conditions = self._fsdp_shard_conditions.copy()
```
**EN:** This block defines method `__post_init__` on `AdapterArchConfig`. It post-processes init. Key calls include `self._fsdp_shard_conditions.copy`. The implementation branches on conditions.
**CN:** 该代码块定义了 `AdapterArchConfig` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `self._fsdp_shard_conditions.copy`。 实现中包含条件分支。

### Lines 45-45: `AdapterConfig` class overview / `AdapterConfig` 类概览
```python
class AdapterConfig(ModelConfig):
```
**EN:** This block defines class `AdapterConfig`. It encapsulates adapter config behavior. It inherits from `ModelConfig`.
**CN:** 该代码块定义了类 `AdapterConfig`。 它用于封装 adapter config 相关行为。 它继承自 `ModelConfig`。

### Lines 46-49: supporting statements / 辅助语句
```python
    arch_config: AdapterArchConfig = field(default_factory=AdapterArchConfig)

    # sglang-diffusion Adapter-specific parameters
    prefix: str = ""
```
**EN:** This block gathers supporting statements inside `AdapterConfig`. It updates names such as `arch_config`, and `prefix`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `AdapterConfig` 内部的辅助语句。 它会更新 `arch_config` 和 `prefix` 等名称。 代码会与 `field` 协同工作。

### Lines 51-62: `add_cli_args` implementation / `add_cli_args` 实现
```python
    @staticmethod
    def add_cli_args(parser: Any, prefix: str = "dit-config") -> Any:
        """Add CLI arguments for AdapterConfig fields"""
        parser.add_argument(
            f"--{prefix}.prefix",
            type=str,
            dest=f"{prefix.replace('-', '_')}.prefix",
            default=AdapterConfig.prefix,
            help="Prefix for the Adapter",
        )

        return parser
```
**EN:** This block defines method `add_cli_args` on `AdapterConfig`. Add CLI arguments for AdapterConfig fields Key calls include `parser.add_argument`, and `prefix.replace`. Parameters such as `parser`, and `prefix` drive the behavior in this section.
**CN:** 该代码块定义了 `AdapterConfig` 的方法 `add_cli_args`。 它用于处理 add cli args 相关逻辑。 关键调用包括 `parser.add_argument` 和 `prefix.replace`。 本段逻辑主要由 `parser` 和 `prefix` 等参数驱动。

## Key Concepts / 关键概念
- `AdapterArchConfig`: Primary class that encapsulates adapter arch config behavior. / 核心类，用于封装 adapter arch config 相关行为。
- `AdapterConfig`: Primary class that encapsulates adapter config behavior. / 核心类，用于封装 adapter config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.base`, `sglang.multimodal_gen.runtime.platforms`

- **Total lines / 总行数**: 62
