# factory.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/kv_offload/tiering/factory.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Factory for creating secondary tier implementations. / 该模块位于 `kv_offload/tiering` 子系统，主要围绕 `create_secondary_tier` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""
Factory for creating secondary tier implementations.
"""

from typing import TYPE_CHECKING

from vllm.v1.kv_offload.tiering.base import SecondaryTierManager
from vllm.v1.kv_offload.tiering.example import ExampleSecondaryTier

if TYPE_CHECKING:
    from vllm.config import VllmConfig

SUPPORTED_TIERS: tuple[type[SecondaryTierManager], ...] = (ExampleSecondaryTier,)

_TIER_REGISTRY: dict[str, type[SecondaryTierManager]] = {
    cls.get_tier_type(): cls for cls in SUPPORTED_TIERS
}
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `SUPPORTED_TIERS`, `_TIER_REGISTRY`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `SUPPORTED_TIERS`, `_TIER_REGISTRY`。

### `create_secondary_tier` function / `create_secondary_tier` 函数
```python
def create_secondary_tier(
    tier_config: dict,
    primary_kv_view: memoryview,
    vllm_config: "VllmConfig",
) -> SecondaryTierManager:
    """
    Create a secondary tier from configuration.

    Args:
        tier_config: Dictionary with tier configuration containing:
            - type (required): Type of secondary tier (e.g., "example")
            - Additional tier-specific parameters are passed directly
              to the tier constructor
        primary_kv_view: Memoryview of the primary tier's CPU KV cache.
        vllm_config: Global vLLM configuration.

    Returns:
        SecondaryTierManager instance

    Raises:
        ValueError: If tier type is unknown or configuration is invalid
    """
    config = tier_config.copy()

    tier_type = config.pop("type", None)
    if not tier_type:
        raise ValueError("Secondary tier configuration must include 'type'")

    cls = _TIER_REGISTRY.get(tier_type)
    if cls is None:
        raise ValueError(
            f"Unknown secondary tier type: {tier_type!r}. "
            f"Supported types: {list(_TIER_REGISTRY)}"
        )
    return cls(vllm_config=vllm_config, primary_kv_view=primary_kv_view, **config)
```
**EN:** This function creates a new object or plan within the module. The docstring frames it as: Create a secondary tier from configuration. Key calls include `copy`, `pop`, `get`, `cls`, `ValueError`, `list`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会创建新的对象或计划，其作用域位于the module。 关键调用包括 `copy`, `pop`, `get`, `cls`, `ValueError`, `list`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `create_secondary_tier`: top-level helper or orchestration entry point. / `create_secondary_tier`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `typing`
- Internal vLLM / 内部依赖: `vllm.v1.kv_offload.tiering.base`, `vllm.v1.kv_offload.tiering.example`, `vllm.config`
