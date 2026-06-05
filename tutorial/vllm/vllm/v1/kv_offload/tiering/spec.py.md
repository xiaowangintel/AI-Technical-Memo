# spec.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/kv_offload/tiering/spec.py`
- **Repository**: vllm-project/vllm
- **Purpose**: TieringOffloadingSpec: Spec for multi-tier KV cache offloading. / 该模块位于 `kv_offload/tiering` 子系统，主要围绕 `TieringOffloadingSpec` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""
TieringOffloadingSpec: Spec for multi-tier KV cache offloading.

This spec creates a TieringOffloadingManager with a CPU primary tier
and configurable secondary tiers (e.g., Storage, Network).

Configuration via kv_connector_extra_config:
  - cpu_bytes_to_use: (required) Bytes to allocate for CPU primary tier
  - block_size: (optional) Block size for offloaded blocks (default: GPU block size)
  - eviction_policy: (optional) Primary tier eviction policy: "lru" or
    "arc" (default: "lru")
  - secondary_tiers: (optional) List of secondary tier configurations
    Each secondary tier config is a dict with:
      - type: (required) Type of secondary tier (e.g., "example", "storage", "network")
      - Additional tier-specific parameters are passed directly to the tier
        constructor. See each tier's documentation for supported parameters.

Example configuration:
{
    "cpu_bytes_to_use": 10737418240,  # 10 GB
    "block_size": 16,
    "eviction_policy": "lru",
    "secondary_tiers": [
        {
            "type": "example",
            # Tier-specific parameters (for ExampleSecondaryTier):
            "max_blocks": 10000,
            "simulate_async": False
        }
    ]
}
"""

import torch

from vllm.config import VllmConfig
from vllm.logger import init_logger
from vllm.v1.kv_cache_interface import KVCacheConfig
from vllm.v1.kv_offload.base import CanonicalKVCaches, OffloadingManager
from vllm.v1.kv_offload.cpu.gpu_worker import CpuGpuOffloadingHandlers
from vllm.v1.kv_offload.cpu.shared_offload_region import SharedOffloadRegion
from vllm.v1.kv_offload.cpu.spec import CPUOffloadingSpec
from vllm.v1.kv_offload.tiering.factory import create_secondary_tier
from vllm.v1.kv_offload.tiering.manager import (
    CPUPrimaryTierOffloadingManager,
    TieringOffloadingManager,
)

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `TieringOffloadingSpec` class / `TieringOffloadingSpec` 类
```python
class TieringOffloadingSpec(CPUOffloadingSpec):
    """
    Spec for multi-tier KV cache offloading.

    Creates a TieringOffloadingManager with:
    - Primary tier: CPU (LRU or ARC eviction policy)
    - Secondary tiers: Configurable via extra_config

    The CPU primary tier has direct GPU access and serves as the gateway for
    all GPU↔offload operations. Secondary tiers cannot directly access GPU
    memory and must transfer data through the primary tier.
    """
```
**EN:** Introduces the `TieringOffloadingSpec` class on top of `CPUOffloadingSpec`. Core methods include `__init__`, `get_manager`, `_create_handlers`. Docstring signal: Spec for multi-tier KV cache offloading.
**CN:** 这里定义 `TieringOffloadingSpec` 类，其基类包括 `CPUOffloadingSpec`。核心方法包括 `__init__`, `get_manager`, `_create_handlers`。

### `TieringOffloadingSpec.__init__` method / `TieringOffloadingSpec.__init__` 方法
```python
    def __init__(self, vllm_config: VllmConfig, kv_cache_config: KVCacheConfig):
        super().__init__(vllm_config, kv_cache_config)
        # Redeclare for mypy: parent sets this but `--follow-imports skip` hides it
        self._manager: OffloadingManager | None = None

        # Parse secondary tier configurations
        self.secondary_tier_configs = self.extra_config.get("secondary_tiers", [])
        if not isinstance(self.secondary_tier_configs, list):
            raise ValueError("secondary_tiers must be a list of tier configurations")

        # Scheduler-side mmap (rank=None); kept for cleanup
        self._scheduler_mmap: SharedOffloadRegion | None = None
```
**EN:** This method initializes the object state within `TieringOffloadingSpec`. Key calls include `__init__`, `get`, `isinstance`, `ValueError`, `super`. It touches state such as `_manager`, `secondary_tier_configs`, `_scheduler_mmap`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`TieringOffloadingSpec`。 关键调用包括 `__init__`, `get`, `isinstance`, `ValueError`, `super`。 它会读写 `_manager`, `secondary_tier_configs`, `_scheduler_mmap` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `TieringOffloadingSpec.get_manager` method / `TieringOffloadingSpec.get_manager` 方法
```python
    def get_manager(self) -> OffloadingManager:
        """
        Get the TieringOffloadingManager.

        Creates a TieringOffloadingManager with:
        - Primary tier: CPU (LRU or ARC)
        - Secondary tiers: As configured in extra_config

        Returns:
            TieringOffloadingManager instance
        """
        if not self._manager:
            kv_events_config = self.vllm_config.kv_events_config
            enable_events = (
                kv_events_config is not None and kv_events_config.enable_kv_cache_events
            )

            # Create scheduler-side SharedOffloadRegion (rank=None) so the
            # primary tier can eagerly create a memoryview over _base.
            world_size = self.vllm_config.parallel_config.world_size
            scheduler_mmap = SharedOffloadRegion(
                instance_id=self.vllm_config.instance_id,
                total_size_bytes=self.cpu_page_size_per_worker
                * world_size
                * self.num_blocks,
                num_blocks=self.num_blocks,
                rank=None,
                num_workers=world_size,
                cpu_page_size=self.cpu_page_size_per_worker,
            )
            self._scheduler_mmap = scheduler_mmap

            # Create primary tier (CPU-based)
            assert len(self.gpu_block_size) == 1
            primary_tier = CPUPrimaryTierOffloadingManager(
                num_blocks=self.num_blocks,
                cache_policy=self.eviction_policy,  # type: ignore[arg-type]
                enable_events=enable_events,
                mmap_region=scheduler_mmap,
            )

            # Create secondary tiers
            primary_kv_view = primary_tier.get_kv_memoryview()
            secondary_tiers = []
            for i, tier_config in enumerate(self.secondary_tier_configs):
                try:
                    tier = create_secondary_tier(
                        tier_config, primary_kv_view, self.vllm_config
                    )
                    secondary_tiers.append(tier)
                    logger.info(
                        "Created secondary tier #%d (%s)",
                        i,
                        tier.get_tier_type(),
                    )
                except Exception as e:
                    logger.error(
                        "Failed to create secondary tier from config %s: %s",
                        tier_config,
                        e,
                    )
                    raise

            # Create TieringOffloadingManager. GPU↔CPU transfers use the inherited
            # get_handlers(); secondary tier transfers are handled by the
            # secondary tier managers and need no additional handlers here.
            tiering_manager = TieringOffloadingManager(
                primary_tier=primary_tier,
                secondary_tiers=secondary_tiers,
                enable_events=enable_events,
            )
            if int(self.extra_config.get("store_threshold", 0)) >= 2:
                raise ValueError(
                    "store_threshold is not supported for TieringOffloadingSpec"
                )
            self._manager = tiering_manager

            logger.info(
                "Created TieringOffloadingManager with primary tier "
                "(%s, %s blocks) and %s secondary tier(s)",
                self.eviction_policy,
                self.num_blocks,
                len(secondary_tiers),
            )

        return self._manager
```
**EN:** This method returns or derives a value within `TieringOffloadingSpec`. The docstring frames it as: Get the TieringOffloadingManager. Key calls include `SharedOffloadRegion`, `CPUPrimaryTierOffloadingManager`, `get_kv_memoryview`, `enumerate`, `TieringOffloadingManager`, `info`. It touches state such as `_scheduler_mmap`, `_manager`. The control flow contains 3 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`TieringOffloadingSpec`。 关键调用包括 `SharedOffloadRegion`, `CPUPrimaryTierOffloadingManager`, `get_kv_memoryview`, `enumerate`, `TieringOffloadingManager`, `info`。 它会读写 `_scheduler_mmap`, `_manager` 等状态。 控制流包含 3 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `TieringOffloadingSpec._create_handlers` method / `TieringOffloadingSpec._create_handlers` 方法
```python
    def _create_handlers(
        self, kv_caches: CanonicalKVCaches
    ) -> CpuGpuOffloadingHandlers:
        world_size = self.vllm_config.parallel_config.world_size
        rank = torch.accelerator.current_device_index()
        worker_mmap = SharedOffloadRegion(
            instance_id=self.vllm_config.instance_id,
            total_size_bytes=self.cpu_page_size_per_worker
            * world_size
            * self.num_blocks,
            num_blocks=self.num_blocks,
            rank=rank,
            num_workers=world_size,
            cpu_page_size=self.cpu_page_size_per_worker,
        )
        return CpuGpuOffloadingHandlers(
            kv_caches=kv_caches,
            block_size_factor=self.block_size_factor,
            num_cpu_blocks=self.num_blocks,
            mmap_region=worker_mmap,
        )
```
**EN:** This method implements `_create_handlers` within `TieringOffloadingSpec`. Key calls include `current_device_index`, `SharedOffloadRegion`, `CpuGpuOffloadingHandlers`.
**CN:** 该方法会实现 `_create_handlers`，其作用域位于`TieringOffloadingSpec`。 关键调用包括 `current_device_index`, `SharedOffloadRegion`, `CpuGpuOffloadingHandlers`。

## Key Concepts / 关键概念
- `TieringOffloadingSpec`: central class or interface in this module. / `TieringOffloadingSpec`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.logger`, `vllm.v1.kv_cache_interface`, `vllm.v1.kv_offload.base`, `vllm.v1.kv_offload.cpu.gpu_worker`, `vllm.v1.kv_offload.cpu.shared_offload_region`, `vllm.v1.kv_offload.cpu.spec`, `vllm.v1.kv_offload.tiering.factory`, `vllm.v1.kv_offload.tiering.manager`
