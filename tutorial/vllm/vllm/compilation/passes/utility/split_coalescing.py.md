# split_coalescing.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/compilation/passes/utility/split_coalescing.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Coalesce adjacent split operations into simpler graph structure. / 将相邻 split 操作合并为更简单的图结构。

## Line-by-Line Analysis / 逐行分析
### Imports (lines 18-26)
```python
import operator

import torch
from torch import fx

from vllm.logger import init_logger

from ..fx_utils import is_func
from ..vllm_inductor_pass import VllmInductorPass
```
**EN:** This import block assembles the PyTorch/FX, Inductor, and vLLM modules that the rest of the file relies on for graph rewriting, backend integration, caching, or runtime control in the graph-cleanup stage.
**CN:** 这一组导入语句组织了 PyTorch/FX、Inductor 与 vLLM 模块，供文件后续代码在图清理阶段中完成图改写、后端集成、缓存或运行时控制。

### Constants / assignments (lines 28-28)
```python
logger = init_logger(__name__)
```
**EN:** These assignments establish shared module state such as loggers, global flags, dtype/op handles, or counters that later logic reuses throughout the graph-cleanup stage.
**CN:** 这些赋值语句建立了共享模块状态，例如日志器、全局开关、dtype/算子句柄或计数器，供后续代码在图清理阶段中复用。

### Class `SplitCoalescingPass` (lines 31-70)
```python
class SplitCoalescingPass(VllmInductorPass):
    """Replace duplicate ``split_with_sizes`` nodes with a single canonical
    node when they share the same input tensor and split sizes."""

    @VllmInductorPass.time_and_log
    def __call__(self, graph: fx.Graph) -> None:
        count = 0

        # Map from input tensor node -> list of split nodes seen so far.
        split_nodes: dict[fx.Node, list[fx.Node]] = {}

        for node in graph.nodes:
            if not is_func(node, torch.ops.aten.split_with_sizes.default):
                continue
            if not all(is_func(user, operator.getitem) for user in node.users):
                continue

            arg_node, split_sizes = node.args[:2]

            if arg_node not in split_nodes:
...
```
**EN:** Class `SplitCoalescingPass` defines a transformation pass that walks an FX/IR graph and rewrites matched structures. It operates inside the graph-cleanup stage. Replace duplicate ``split_with_sizes`` nodes with a single canonical node when they share the same input tensor and split sizes. Key methods include __call__.
**CN:** 类 `SplitCoalescingPass` 定义了一个变换 pass，用于遍历 FX/IR 图并改写匹配到的结构，并服务于图清理阶段。 文档字符串强调：Replace duplicate ``split_with_sizes`` nodes with a single canonical node when they share the same input tensor and split sizes. 关键方法包括 __call__。

## Key Concepts / 关键概念
- **Compilation pipeline / 编译流水线**
  - **EN:** The file participates in turning FX graphs into optimized executable code or runtime wrappers.
  - **CN:** 该文件参与将 FX 图转换为优化后的可执行代码或运行时封装。
- **Graph partitioning / 图划分**
  - **EN:** The implementation reshapes a larger graph into regions that can be compiled or optimized independently.
  - **CN:** 该实现会把较大的图重塑为可以独立编译或优化的多个区域。

## Dependencies / 依赖关系
- **vLLM internal / vLLM 内部依赖**: `from vllm.logger import init_logger`, `from ..fx_utils import is_func`, `from ..vllm_inductor_pass import VllmInductorPass`
- **PyTorch / Torch 栈**: `import torch`, `from torch import fx`
- **Stdlib / 标准库**: `import operator`
