# parallel_groups.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/distributed/parallel_groups.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the distributed runtime layer. It centers on `Singleton`, `ProcessGroupSingleton`, and `set_seq_parallel_pg_by_sp_groups`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于分布式运行时层。它围绕 `Singleton`、`ProcessGroupSingleton` 和 `set_seq_parallel_pg_by_sp_groups` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-4: module setup and imports / 模块初始化与导入
```python
import torch
```
**EN:** This block establishes the module context and imports `torch`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `torch`。这些依赖为后续实现提供所需符号。

### Lines 7-7: `Singleton` class overview / `Singleton` 类概览
```python
class Singleton:
```
**EN:** This block defines class `Singleton`. It encapsulates singleton behavior.
**CN:** 该代码块定义了类 `Singleton`。 它用于封装 singleton 相关行为。

### Lines 8-8: supporting statements / 辅助语句
```python
    _instance = None
```
**EN:** This block gathers supporting statements inside `Singleton`. It updates names such as `_instance`.
**CN:** 该代码块汇集了位于 `Singleton` 内部的辅助语句。 它会更新 `_instance` 等名称。

### Lines 10-13: `__new__` implementation / `__new__` 实现
```python
    def __new__(cls, *args, **kwargs):
        if not cls._instance:
            cls._instance = super(Singleton, cls).__new__(cls, *args, **kwargs)
        return cls._instance
```
**EN:** This block defines method `__new__` on `Singleton`. It handles new logic. Key calls include `super.__new__`, and `super`. The implementation branches on conditions.
**CN:** 该代码块定义了 `Singleton` 的方法 `__new__`。 它用于处理 new 相关逻辑。 关键调用包括 `super.__new__` 和 `super`。 实现中包含条件分支。

### Lines 16-16: `ProcessGroupSingleton` class overview / `ProcessGroupSingleton` 类概览
```python
class ProcessGroupSingleton(Singleton):
```
**EN:** This block defines class `ProcessGroupSingleton`. It processes group singleton. It inherits from `Singleton`.
**CN:** 该代码块定义了类 `ProcessGroupSingleton`。 它用于处理group singleton。 它继承自 `Singleton`。

### Lines 17-19: `__init__` implementation / `__init__` 实现
```python
    def __init__(self):
        self.ULYSSES_PG = None
        self.RING_PG = None
```
**EN:** This block defines method `__init__` on `ProcessGroupSingleton`. It initializes the instance state.
**CN:** 该代码块定义了 `ProcessGroupSingleton` 的方法 `__init__`。 它用于初始化实例状态。

### Lines 22-22: supporting statements / 辅助语句
```python
PROCESS_GROUP = ProcessGroupSingleton()
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `PROCESS_GROUP`. The code collaborates with `ProcessGroupSingleton`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `PROCESS_GROUP` 等名称。 代码会与 `ProcessGroupSingleton` 协同工作。

### Lines 25-91: `set_seq_parallel_pg_by_sp_groups` implementation / `set_seq_parallel_pg_by_sp_groups` 实现
```python
def set_seq_parallel_pg_by_sp_groups(
    sp_ulysses_degree,
    sp_ring_degree,
    rank: int,
    sp_groups: list[list[int]],
    use_ulysses_low: bool = True,
):
    """Create Ulysses/Ring process groups inside each SP group.

    This is required when TP>1, because SP groups are not necessarily made of
    consecutive global ranks (e.g., tp-sp order makes SP ranks strided).

    Args:
        sp_ulysses_degree: ulysses degree inside SP.
        sp_ring_degree: ring degree inside SP.
        rank: global rank of current process.
        sp_groups: list of global-rank lists for each SP group.
        use_ulysses_low: keep the same semantics as the original function.
    """
    sp_degree = sp_ring_degree * sp_ulysses_degree
    assert sp_degree > 0
    assert all(
        len(g) == sp_degree for g in sp_groups
    ), f"Each SP group must have size {sp_degree}, got sizes {[len(g) for g in sp_groups]}"

    ulyssess_pg = None
    ring_pg = None

    num_ulysses_pgs = sp_ring_degree
    num_ring_pgs = sp_ulysses_degree

    def _map_indices_to_ranks(ranks: list[int], indices: list[int]) -> list[int]:
        return [ranks[i] for i in indices]

    # Important: call torch.distributed.new_group in the same order on all ranks.
    for sp_ranks in sp_groups:
        if use_ulysses_low:
            for i in range(num_ulysses_pgs):
                idx = list(range(i * sp_ulysses_degree, (i + 1) * sp_ulysses_degree))
                ulysses_ranks = _map_indices_to_ranks(sp_ranks, idx)
                group = torch.distributed.new_group(ulysses_ranks)
                if rank in ulysses_ranks:
                    ulyssess_pg = group

            for i in range(num_ring_pgs):
                idx = list(range(i, sp_degree, num_ring_pgs))
                ring_ranks = _map_indices_to_ranks(sp_ranks, idx)
                group = torch.distributed.new_group(ring_ranks)
                if rank in ring_ranks:
                    ring_pg = group
        else:
            for i in range(num_ring_pgs):
                idx = list(range(i * sp_ring_degree, (i + 1) * sp_ring_degree))
                ring_ranks = _map_indices_to_ranks(sp_ranks, idx)
                group = torch.distributed.new_group(ring_ranks)
                if rank in ring_ranks:
                    ring_pg = group

            for i in range(num_ulysses_pgs):
                idx = list(range(i, sp_degree, num_ulysses_pgs))
                ulysses_ranks = _map_indices_to_ranks(sp_ranks, idx)
                group = torch.distributed.new_group(ulysses_ranks)
                if rank in ulysses_ranks:
                    ulyssess_pg = group

    PROCESS_GROUP.ULYSSES_PG = ulyssess_pg
    PROCESS_GROUP.RING_PG = ring_pg
```
**EN:** This block defines function `set_seq_parallel_pg_by_sp_groups`. Create Ulysses/Ring process groups inside each SP group. This is required when TP>1, because SP groups are not necessarily made of consecutive global ranks (e.g., tp-sp order makes SP ranks strided). Key calls include `all`, `range`, `len`, `list`, and `_map_indices_to_ranks`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `sp_ulysses_degree`, `sp_ring_degree`, `rank`, `sp_groups`, and `use_ulysses_low` drive the behavior in this section.
**CN:** 该代码块定义了函数 `set_seq_parallel_pg_by_sp_groups`。 它用于设置seq parallel pg by sp groups。 关键调用包括 `all`、`range`、`len`、`list` 和 `_map_indices_to_ranks`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `sp_ulysses_degree`、`sp_ring_degree`、`rank`、`sp_groups` 和 `use_ulysses_low` 等参数驱动。

## Key Concepts / 关键概念
- `Singleton`: Primary class that encapsulates singleton behavior. / 核心类，用于封装 singleton 相关行为。
- `ProcessGroupSingleton`: Primary class that processes group singleton. / 核心类，用于处理group singleton。
- `set_seq_parallel_pg_by_sp_groups`: Create Ulysses/Ring process groups inside each SP group. / 顶层函数，用于设置seq parallel pg by sp groups。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `torch`

- **Total lines / 总行数**: 91
