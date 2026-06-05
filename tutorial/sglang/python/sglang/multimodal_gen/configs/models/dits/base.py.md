# base.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/models/dits/base.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `DiTArchConfig`, and `DiTConfig`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `DiTArchConfig` 和 `DiTConfig` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-9: module setup and imports / 模块初始化与导入
```python
from dataclasses import dataclass, field
from typing import Any

from sglang.multimodal_gen.configs.models.base import ArchConfig, ModelConfig
from sglang.multimodal_gen.runtime.layers.quantization import QuantizationConfig
from sglang.multimodal_gen.runtime.platforms import AttentionBackendEnum
```
**EN:** This block establishes the module context and imports `dataclasses`, `typing`, `sglang.multimodal_gen.configs.models.base`, `sglang.multimodal_gen.runtime.layers.quantization`, and `sglang.multimodal_gen.runtime.platforms`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`typing`、`sglang.multimodal_gen.configs.models.base`、`sglang.multimodal_gen.runtime.layers.quantization` 和 `sglang.multimodal_gen.runtime.platforms`。这些依赖为后续实现提供所需符号。

### Lines 13-13: `DiTArchConfig` class overview / `DiTArchConfig` 类概览
```python
class DiTArchConfig(ArchConfig):
```
**EN:** This block defines class `DiTArchConfig`. It encapsulates di tarch config behavior. It inherits from `ArchConfig`.
**CN:** 该代码块定义了类 `DiTArchConfig`。 它用于封装 di tarch config 相关行为。 它继承自 `ArchConfig`。

### Lines 14-45: supporting statements / 辅助语句
```python
    _fsdp_shard_conditions: list = field(default_factory=list)
    _compile_conditions: list = field(default_factory=list)

    # convert weights name from HF-format to SGLang-dit-format
    param_names_mapping: dict = field(default_factory=dict)

    # convert weights name from misc-format to HF-format
    # usually applicable if the LoRA is trained with official repo implementation
    lora_param_names_mapping: dict = field(default_factory=dict)

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
            AttentionBackendEnum.SPARSE_VIDEO_GEN_2_ATTN,
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
**EN:** This block gathers supporting statements inside `DiTArchConfig`. It updates names such as `_fsdp_shard_conditions`, `_compile_conditions`, `param_names_mapping`, `lora_param_names_mapping`, `reverse_param_names_mapping`, and `_supported_attention_backends`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `DiTArchConfig` 内部的辅助语句。 它会更新 `_fsdp_shard_conditions`、`_compile_conditions`、`param_names_mapping`、`lora_param_names_mapping`、`reverse_param_names_mapping` 和 `_supported_attention_backends` 等名称。 代码会与 `field` 协同工作。

### Lines 47-49: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self) -> None:
        if not self._compile_conditions:
            self._compile_conditions = self._fsdp_shard_conditions.copy()
```
**EN:** This block defines method `__post_init__` on `DiTArchConfig`. It post-processes init. Key calls include `self._fsdp_shard_conditions.copy`. The implementation branches on conditions.
**CN:** 该代码块定义了 `DiTArchConfig` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `self._fsdp_shard_conditions.copy`。 实现中包含条件分支。

### Lines 53-53: `DiTConfig` class overview / `DiTConfig` 类概览
```python
class DiTConfig(ModelConfig):
```
**EN:** This block defines class `DiTConfig`. It encapsulates di tconfig behavior. It inherits from `ModelConfig`.
**CN:** 该代码块定义了类 `DiTConfig`。 它用于封装 di tconfig 相关行为。 它继承自 `ModelConfig`。

### Lines 54-58: supporting statements / 辅助语句
```python
    arch_config: DiTArchConfig = field(default_factory=DiTArchConfig)

    # sglang-diffusion DiT-specific parameters
    prefix: str = ""
    quant_config: QuantizationConfig | None = None
```
**EN:** This block gathers supporting statements inside `DiTConfig`. It updates names such as `arch_config`, `prefix`, and `quant_config`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `DiTConfig` 内部的辅助语句。 它会更新 `arch_config`、`prefix` 和 `quant_config` 等名称。 代码会与 `field` 协同工作。

### Lines 60-79: `add_cli_args` implementation / `add_cli_args` 实现
```python
    @staticmethod
    def add_cli_args(parser: Any, prefix: str = "dit-config") -> Any:
        """Add CLI arguments for DiTConfig fields"""
        parser.add_argument(
            f"--{prefix}.prefix",
            type=str,
            dest=f"{prefix.replace('-', '_')}.prefix",
            default=DiTConfig.prefix,
            help="Prefix for the DiT model",
        )

        parser.add_argument(
            f"--{prefix}.quant-config",
            type=str,
            dest=f"{prefix.replace('-', '_')}.quant_config",
            default=None,
            help="Quantization configuration for the DiT model",
        )

        return parser
```
**EN:** This block defines method `add_cli_args` on `DiTConfig`. Add CLI arguments for DiTConfig fields Key calls include `parser.add_argument`, and `prefix.replace`. Parameters such as `parser`, and `prefix` drive the behavior in this section.
**CN:** 该代码块定义了 `DiTConfig` 的方法 `add_cli_args`。 它用于处理 add cli args 相关逻辑。 关键调用包括 `parser.add_argument` 和 `prefix.replace`。 本段逻辑主要由 `parser` 和 `prefix` 等参数驱动。

## Key Concepts / 关键概念
- `DiTArchConfig`: Primary class that encapsulates di tarch config behavior. / 核心类，用于封装 di tarch config 相关行为。
- `DiTConfig`: Primary class that encapsulates di tconfig behavior. / 核心类，用于封装 di tconfig 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.base`, `sglang.multimodal_gen.runtime.layers.quantization`, `sglang.multimodal_gen.runtime.platforms`

- **Total lines / 总行数**: 79
