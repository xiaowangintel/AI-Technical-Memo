# mova_dual_tower.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/models/bridges/mova_dual_tower.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `MOVADualTowerArchConfig`, `MOVADualTowerConfig`, and `_is_conditioner_block`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Configuration for MOVA dual tower bridge model. / 该文件属于配置层。它围绕 `MOVADualTowerArchConfig`、`MOVADualTowerConfig` 和 `_is_conditioner_block` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 2-6: module setup and imports / 模块初始化与导入
```python
"""Configuration for MOVA dual tower bridge model."""

from dataclasses import dataclass, field

from sglang.multimodal_gen.configs.models.dits.base import DiTArchConfig, DiTConfig
```
**EN:** This block establishes the module context and imports `dataclasses`, and `sglang.multimodal_gen.configs.models.dits.base`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses` 和 `sglang.multimodal_gen.configs.models.dits.base`。这些依赖为后续实现提供所需符号。

### Lines 9-11: `_is_conditioner_block` implementation / `_is_conditioner_block` 实现
```python
def _is_conditioner_block(name: str, module) -> bool:
    """Check if module is a ConditionalCrossAttentionBlock."""
    return "ConditionalCrossAttentionBlock" in type(module).__name__
```
**EN:** This block defines function `_is_conditioner_block`. Check if module is a ConditionalCrossAttentionBlock. Key calls include `type`. Parameters such as `name`, and `module` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_is_conditioner_block`。 它用于处理 is conditioner block 相关逻辑。 关键调用包括 `type`。 本段逻辑主要由 `name` 和 `module` 等参数驱动。

### Lines 15-15: `MOVADualTowerArchConfig` class overview / `MOVADualTowerArchConfig` 类概览
```python
class MOVADualTowerArchConfig(DiTArchConfig):
```
**EN:** This block defines class `MOVADualTowerArchConfig`. It encapsulates movadual tower arch config behavior. It inherits from `DiTArchConfig`.
**CN:** 该代码块定义了类 `MOVADualTowerArchConfig`。 它用于封装 movadual tower arch config 相关行为。 它继承自 `DiTArchConfig`。

### Lines 16-32: supporting statements / 辅助语句
```python
    _fsdp_shard_conditions: list = field(
        default_factory=lambda: [_is_conditioner_block]
    )

    # Model architecture parameters
    visual_layers: int = 40
    audio_layers: int = 30
    visual_hidden_dim: int = 5120
    audio_hidden_dim: int = 1536
    audio_fps: float = 50.0
    head_dim: int = 128
    interaction_strategy: str = "full"
    apply_cross_rope: bool = True
    apply_first_frame_bias_in_rope: bool = False
    trainable_condition_scale: bool = False
    pooled_adaln: bool = False
    eps: float = 1e-6
```
**EN:** This block gathers supporting statements inside `MOVADualTowerArchConfig`. It updates names such as `_fsdp_shard_conditions`, `visual_layers`, `audio_layers`, `visual_hidden_dim`, `audio_hidden_dim`, and `audio_fps`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `MOVADualTowerArchConfig` 内部的辅助语句。 它会更新 `_fsdp_shard_conditions`、`visual_layers`、`audio_layers`、`visual_hidden_dim`、`audio_hidden_dim` 和 `audio_fps` 等名称。 代码会与 `field` 协同工作。

### Lines 34-37: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self):
        super().__post_init__()
        self.hidden_size = self.visual_hidden_dim
        self.num_attention_heads = self.visual_hidden_dim // self.head_dim
```
**EN:** This block defines method `__post_init__` on `MOVADualTowerArchConfig`. It post-processes init. Key calls include `super.__post_init__`, and `super`.
**CN:** 该代码块定义了 `MOVADualTowerArchConfig` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `super.__post_init__` 和 `super`。

### Lines 41-41: `MOVADualTowerConfig` class overview / `MOVADualTowerConfig` 类概览
```python
class MOVADualTowerConfig(DiTConfig):
```
**EN:** This block defines class `MOVADualTowerConfig`. It encapsulates movadual tower config behavior. It inherits from `DiTConfig`.
**CN:** 该代码块定义了类 `MOVADualTowerConfig`。 它用于封装 movadual tower config 相关行为。 它继承自 `DiTConfig`。

### Lines 42-42: supporting statements / 辅助语句
```python
    arch_config: DiTArchConfig = field(default_factory=MOVADualTowerArchConfig)
```
**EN:** This block gathers supporting statements inside `MOVADualTowerConfig`. It updates names such as `arch_config`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `MOVADualTowerConfig` 内部的辅助语句。 它会更新 `arch_config` 等名称。 代码会与 `field` 协同工作。

## Key Concepts / 关键概念
- `_is_conditioner_block`: Check if module is a ConditionalCrossAttentionBlock. / 顶层函数，用于处理 is conditioner block 相关逻辑。
- `MOVADualTowerArchConfig`: Primary class that encapsulates movadual tower arch config behavior. / 核心类，用于封装 movadual tower arch config 相关行为。
- `MOVADualTowerConfig`: Primary class that encapsulates movadual tower config behavior. / 核心类，用于封装 movadual tower config 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.models.dits.base`

- **Total lines / 总行数**: 42
