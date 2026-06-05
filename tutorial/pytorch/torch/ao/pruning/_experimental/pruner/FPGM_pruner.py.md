# FPGM_pruner.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/ao/pruning/_experimental/pruner/FPGM_pruner.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN) / 用途（英文）**: Implements pruning and sparsity utilities that remove or mask parameters while preserving model structure. This specific file centers on `FPGM_pruner.py`. The implementation is performance-facing and focuses on measurement fidelity or result reporting.
- **Purpose (CN) / 用途（中文）**: 实现剪枝与稀疏化工具，在保留模型结构的同时移除或屏蔽参数。 该文件具体围绕 `FPGM_pruner.py` 展开。 该实现面向性能分析，重点在于测量可信度或结果报告。
## Line-by-Line Analysis / 逐行分析

### Lines 1-15 / 第 1-15 行
```python
# mypy: allow-untyped-defs
from collections.abc import Callable

import torch

from .base_structured_sparsifier import BaseStructuredSparsifier


__all__ = ["FPGMPruner"]


class FPGMPruner(BaseStructuredSparsifier):
    r"""Filter Pruning via Geometric Median (FPGM) Structured Pruner
    This sparsifier prune filter (row) in a tensor according to distances among filters according to
    `Filter Pruning via Geometric Median for Deep Convolutional Neural Networks Acceleration <https://arxiv.org/abs/1811.00250>`_.
```
- **EN**: The `__all__` assignment defines the intended public surface that other modules are expected to import or re-export. It introduces or extends class-level abstractions such as `FPGMPruner`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way.
- **CN**: `__all__` 赋值定义了预期的公开接口，供其他模块导入或再次导出。 它引入或扩展了 `FPGMPruner` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。

### Lines 17-33 / 第 17-33 行
```python
    This sparsifier is controlled by three variables:
    1. `sparsity_level` defines the number of filters (rows) that are zeroed-out.
    2. `dist` defines the distance measurement type. Default: 3 (L2 distance).
    Available options are: [1, 2, (custom callable distance function)].

    Note::
        Inputs should be a 4D convolutional tensor of shape (N, C, H, W).
            - N: output channels size
            - C: input channels size
            - H: height of kernel
            - W: width of kernel
    """

    def __init__(self, sparsity_level: float = 0.5, dist: Callable | int | None = None):
        defaults = {
            "sparsity_level": sparsity_level,
        }
```
- **EN**: It introduces or extends class-level abstractions such as `FPGMPruner`, which organize state and behavior for this subsystem.
- **CN**: 它引入或扩展了 `FPGMPruner` 等类级抽象，用于组织该子系统的状态与行为。

### Lines 35-46 / 第 35-46 行
```python
        if dist is None:
            dist = 2

        if callable(dist):
            self.dist_fn = dist
        elif dist == 1:
            self.dist_fn = lambda x: torch.cdist(x, x, p=1)
        elif dist == 2:
            self.dist_fn = lambda x: torch.cdist(x, x, p=2)
        else:
            raise NotImplementedError("Distance function is not yet implemented.")
        super().__init__(defaults=defaults)
```
- **EN**: It introduces or extends class-level abstractions such as `FPGMPruner`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms. Assertions or raised errors defend assumptions early, which is especially useful in framework plumbing code.
- **CN**: 它引入或扩展了 `FPGMPruner` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。 断言或显式抛错会尽早防御关键假设，这在框架基础逻辑中尤其重要。

### Lines 48-59 / 第 48-59 行
```python
    def _compute_distance(self, t):
        r"""Compute distance across all entries in tensor `t` along all dimension
        except for the one identified by dim.
        Args:
            t (torch.Tensor): tensor representing the parameter to prune
        Returns:
            distance (torch.Tensor): distance computed across filtters
        """
        dim = 0  # prune filter (row)

        size = t.size(dim)
        slc = [slice(None)] * t.dim()
```
- **EN**: It introduces or extends class-level abstractions such as `FPGMPruner`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. Structured exception handling keeps failure modes explicit instead of letting partial state escape silently.
- **CN**: 它引入或扩展了 `FPGMPruner` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 结构化异常处理让失败模式保持显式，而不是让部分状态悄悄泄漏出去。

### Lines 61-74 / 第 61-74 行
```python
        # flatten the tensor along the dimension
        t_flatten = [
            t[tuple(slc[:dim] + [slice(i, i + 1)] + slc[dim + 1 :])].reshape(-1)
            for i in range(size)
        ]
        t_flatten = torch.stack(t_flatten)

        # distance measurement
        dist_matrix = self.dist_fn(t_flatten)

        # more similar with other filter indicates large in the sum of row
        distance = torch.sum(torch.abs(dist_matrix), 1)

        return distance
```
- **EN**: It introduces or extends class-level abstractions such as `FPGMPruner`, which organize state and behavior for this subsystem. Looping code applies the same rule over collections, parameters, graph nodes, or samples in a systematic way. The tail of the block returns computed state or forwards the next value to the caller.
- **CN**: 它引入或扩展了 `FPGMPruner` 等类级抽象，用于组织该子系统的状态与行为。 循环代码会系统地把同一规则应用到集合、参数、图节点或样本上。 该代码块末尾会返回计算后的状态，或把下一个值转交给调用方。

### Lines 76-87 / 第 76-87 行
```python
    def update_mask(  # type: ignore[override]
        self, module, tensor_name, sparsity_level, **kwargs
    ):
        tensor_weight = getattr(module, tensor_name)
        mask = getattr(module.parametrizations, tensor_name)[0].mask

        if sparsity_level <= 0:
            mask.data = torch.ones_like(mask).bool()
        elif sparsity_level >= 1.0:
            mask.data = torch.zeros_like(mask).bool()
        else:
            distance = self._compute_distance(tensor_weight)
```
- **EN**: It introduces or extends class-level abstractions such as `FPGMPruner`, which organize state and behavior for this subsystem. Conditional branches handle special cases and preserve invariants expected by callers or downstream transforms.
- **CN**: 它引入或扩展了 `FPGMPruner` 等类级抽象，用于组织该子系统的状态与行为。 条件分支用于处理特殊情况，并维护调用方或下游变换所依赖的不变量。

### Lines 89-95 / 第 89-95 行
```python
            tensor_size = tensor_weight.shape[0]  # prune filter (row)
            nparams_toprune = round(sparsity_level * tensor_size)
            nparams_toprune = min(
                max(nparams_toprune, 0), tensor_size
            )  # clamp to [0, tensor_size]
            topk = torch.topk(distance, k=nparams_toprune, largest=False)
            mask[topk.indices] = False
```
- **EN**: It introduces or extends class-level abstractions such as `FPGMPruner`, which organize state and behavior for this subsystem.
- **CN**: 它引入或扩展了 `FPGMPruner` 等类级抽象，用于组织该子系统的状态与行为。

## Key Concepts / 关键概念

- **Module role**
  - EN: Implements pruning and sparsity utilities that remove or mask parameters while preserving model structure.
  - CN: 实现剪枝与稀疏化工具，在保留模型结构的同时移除或屏蔽参数。
- **Pruning and sparsity**
  - EN: Controls parameter masking, sparse layouts, or pruning schedules that reduce effective model size.
  - CN: 控制参数掩码、稀疏布局或剪枝计划，以降低模型的有效规模。
- **Benchmarking**
  - EN: Focuses on repeatable measurement, result comparison, and performance diagnostics.
  - CN: 聚焦可重复的性能测量、结果比较与性能诊断。
- **Configuration management**
  - EN: Centralizes feature flags, environment overrides, and runtime-config wiring.
  - CN: 集中管理特性开关、环境变量覆盖以及运行时配置接线。
- **Pytree processing**
  - EN: Manipulates nested Python container structure while preserving leaves and shape metadata.
  - CN: 在保留叶子节点与结构元数据的同时处理嵌套 Python 容器。
- **FPGMPruner**
  - EN: `FPGMPruner` is one of the main classes that structures the file's behavior.
  - CN: `FPGMPruner` 是组织该文件行为的核心类之一。
## Dependencies / 依赖关系

- **Internal PyTorch modules / 内部 PyTorch 模块**: `torch`, `.base_structured_sparsifier:BaseStructuredSparsifier`
- **Python standard library / Python 标准库**: `collections.abc:Callable`
- **Explicit exports / 显式导出**: `FPGMPruner`
- **Primary symbols / 核心符号**: `FPGMPruner`
