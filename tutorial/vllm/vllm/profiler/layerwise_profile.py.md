# layerwise_profile.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/profiler/layerwise_profile.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `layerwise_profile`-related logic centered around `_ModuleTreeNode`, `SummaryStatsEntry`. / 实现与 `layerwise_profile` 相关的逻辑，核心符号包括 `_ModuleTreeNode`, `SummaryStatsEntry`。

## Line-by-Line Analysis / 逐行分析
### Module setup (lines 1-28)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project

import copy
from collections import defaultdict
from collections.abc import Callable
from dataclasses import asdict, dataclass, field
from typing import Any, Generic, TypeAlias, TypeVar

from torch._C._autograd import DeviceType, _KinetoEvent, _ProfilerResult
from torch._C._profiler import _EventType, _ExperimentalConfig, _ProfilerEvent
from torch.autograd.profiler import FunctionEvent
from torch.profiler import ProfilerActivity, profile

from vllm.profiler.utils import (
    TablePrinter,
    event_has_module,
    event_is_torch_op,
    event_module_repr,
    event_torch_op_stack_trace,
    indent_string,
)
from vllm.utils.import_utils import PlaceholderModule

try:
    import pandas as pd
except ImportError:
    pd = PlaceholderModule("pandas")
```
**EN:** Sets up the module with standard-library support such as `copy`, `collections`, `collections.abc`, external packages such as `torch._C._autograd`, `torch._C._profiler`, `torch.autograd.profiler`, vLLM modules such as `vllm.profiler.utils`, `vllm.utils.import_utils`. It prepares the symbols later used by `_ModuleTreeNode`, `SummaryStatsEntry`.
**CN:** 这一部分完成模块初始化，组织导入项与文件级定义，并接入 `vllm.profiler.utils`, `vllm.utils.import_utils` 等 vLLM 内部依赖。 这些准备工作为后续的 `_ModuleTreeNode`, `SummaryStatsEntry` 提供上下文。

### _ModuleTreeNode (lines 32-51)
```python
class _ModuleTreeNode:
    event: _ProfilerEvent
    parent: "_ModuleTreeNode | None" = None
    children: list["_ModuleTreeNode"] = field(default_factory=list)
    trace: str = ""

    @property
    def is_leaf(self):
        return self.event.children is None or len(self.event.children) == 0

    @property
    def is_torch_op(self):
        return event_is_torch_op(self.event)

    @property
    def is_cuda(self):
        return (
            self.event.tag == _EventType.Kineto
            and self.event.typed[1].device_type == DeviceType.CUDA
        )
```
**EN:** Defines the `_ModuleTreeNode` class used by this module. Key methods include `is_leaf`, `is_torch_op`, `is_cuda`.
**CN:** `_ModuleTreeNode` 是该文件中的核心类，用于封装与 `_ModuleTreeNode` 相关的状态和行为。 关键方法包括 `is_leaf`, `is_torch_op`, `is_cuda`。

### SummaryStatsEntry (lines 55-59)
```python
class SummaryStatsEntry:
    name: str
    cuda_time_us: float
    pct_cuda_time: float
    invocations: int
```
**EN:** Defines the `SummaryStatsEntry` class used by this module.
**CN:** `SummaryStatsEntry` 是该文件中的核心类，用于封装与 `SummaryStatsEntry` 相关的状态和行为。

### ModelStatsEntry (lines 63-68)
```python
class ModelStatsEntry:
    name: str
    cpu_time_us: float
    cuda_time_us: float
    pct_cuda_time: float
    trace: str
```
**EN:** Defines the `ModelStatsEntry` class used by this module.
**CN:** `ModelStatsEntry` 是该文件中的核心类，用于封装与 `ModelStatsEntry` 相关的状态和行为。

### _StatsTreeNode (lines 76-79)
```python
class _StatsTreeNode(Generic[StatsEntryT]):
    entry: StatsEntryT
    children: list["_StatsTreeNode[StatsEntryT]"] = field(default_factory=list)
    parent: "_StatsTreeNode[StatsEntryT] | None" = None
```
**EN:** Defines the `_StatsTreeNode` class used by this module. It extends `Generic`.
**CN:** `_StatsTreeNode` 是该文件中的核心类，用于封装与 `_StatsTreeNode` 相关的状态和行为。 它继承自 `Generic`。

### LayerwiseProfileResults overview (lines 83-370)
```python
class LayerwiseProfileResults(profile):
    _kineto_results: _ProfilerResult
    _kineto_event_correlation_map: dict[int, list[_KinetoEvent]] = field(init=False)
    _event_correlation_map: dict[int, list[FunctionEvent]] = field(init=False)
    _module_tree: list[_ModuleTreeNode] = field(init=False)
    _model_stats_tree: list[_StatsTreeNode[ModelStatsEntry]] = field(init=False)
    _summary_stats_tree: list[_StatsTreeNode[SummaryStatsEntry]] = field(init=False)

    # profile metadata
    num_running_seqs: int | None = None

    def __post_init__(self):
        self._build_correlation_map()
        self._build_module_tree()
        self._build_stats_trees()

    def print_model_table(self, column_widths: dict[str, int] | None = None):
        _column_widths = dict(
            name=60, cpu_time_us=12, cuda_time_us=12, pct_cuda_time=12, trace=60
        )
        if column_widths:
            _column_widths.update(**column_widths)
        filtered_model_table = [
            (depth, row)
            for depth, row in self._flatten_stats_tree(self._model_stats_tree)
            if row.cuda_time_us > 0 or row.cpu_time_us > 0
        ]
    # ...
```
**EN:** Defines the `LayerwiseProfileResults` class used by this module. It extends `profile`. Key methods include `print_model_table`, `print_summary_table`, `export_model_stats_table_csv`, `export_summary_stats_table_csv`. The excerpt below focuses on the class entry point; key methods are analyzed separately.
**CN:** `LayerwiseProfileResults` 是该文件中的核心类，用于封装与 `LayerwiseProfileResults` 相关的状态和行为。 它继承自 `profile`。 关键方法包括 `print_model_table`, `print_summary_table`, `export_model_stats_table_csv`, `export_summary_stats_table_csv`。 下面的代码片段重点展示类的入口部分，关键方法会在后续分块单独分析。

### LayerwiseProfileResults._build_stats_trees (lines 243-338)
```python
    def _build_stats_trees(self):
        summary_dict: dict[tuple[str, ...], _StatsTreeNode[SummaryStatsEntry]] = {}
        total_cuda_time = self._total_cuda_time()

        def pct_cuda_time(cuda_time_us):
            return (cuda_time_us / total_cuda_time) * 100

        def build_summary_stats_tree_df(
            node: _ModuleTreeNode,
            parent: _StatsTreeNode[SummaryStatsEntry] | None = None,
            summary_trace: tuple[str, ...] = (),
        ) -> _StatsTreeNode[SummaryStatsEntry] | None:
            if event_has_module(node.event):
                name = event_module_repr(node.event)
                cuda_time_us = self._cumulative_cuda_time(node)
            elif gpu_kineto_event := self._get_kineto_gpu_event(node):
                name = gpu_kineto_event.name()
                cuda_time_us = gpu_kineto_event.duration_ns() / 1000.0
            else:
                return None
    # ...

        self._model_stats_tree = []
        for root in self._module_tree:
            model_node = build_model_stats_tree_df(root)
            if model_node is not None:
                self._model_stats_tree.append(model_node)
```
**EN:** `_build_stats_trees` constructs objects for downstream execution. Inside the body, it relies on `self._total_cuda_time`, `event_has_module`, `build_summary_stats_tree_df` to complete the main steps.
**CN:** `_build_stats_trees` 负责为下游执行构造对象。 实现过程中会调用 `self._total_cuda_time`, `event_has_module`, `build_summary_stats_tree_df` 等函数完成关键步骤。

### LayerwiseProfileResults._build_module_tree (lines 176-211)
```python
    def _build_module_tree(self):
        self._module_tree = []
        event_tree = self._kineto_results.experimental_event_tree()

        def _df_traversal(
            event: _ProfilerEvent, curr_node: _ModuleTreeNode | None = None
        ):
            # For the tensor parallel case for now only look at task 1
            if event.start_tid != 1:
                return

            if event_has_module(event):
                node = _ModuleTreeNode(event=event, parent=curr_node)
                if curr_node:
                    curr_node.children.append(node)
                else:
                    self._module_tree.append(node)
                curr_node = node

            is_leaf = event.children is None or len(event.children) == 0
    # ...

            for child in event.children:
                _df_traversal(child, curr_node)

        for root in event_tree:
            _df_traversal(root)
```
**EN:** `_build_module_tree` constructs objects for downstream execution. Inside the body, it relies on `self._kineto_results.experimental_event_tree`, `event_has_module`, `_df_traversal` to complete the main steps.
**CN:** `_build_module_tree` 负责为下游执行构造对象。 实现过程中会调用 `self._kineto_results.experimental_event_tree`, `event_has_module`, `_df_traversal` 等函数完成关键步骤。

### LayerwiseProfileResults.print_model_table (lines 99-115)
```python
    def print_model_table(self, column_widths: dict[str, int] | None = None):
        _column_widths = dict(
            name=60, cpu_time_us=12, cuda_time_us=12, pct_cuda_time=12, trace=60
        )
        if column_widths:
            _column_widths.update(**column_widths)
        filtered_model_table = [
            (depth, row)
            for depth, row in self._flatten_stats_tree(self._model_stats_tree)
            if row.cuda_time_us > 0 or row.cpu_time_us > 0
        ]
        TablePrinter(ModelStatsEntry, _column_widths).print_table(
            self._indent_row_names_based_on_depth(
                filtered_model_table,
                indent_style=lambda indent: "|" + "-" * indent + " ",
            )
        )
```
**EN:** `print_model_table` implements helper logic used by this module. It mainly works with `column_widths`. Inside the body, it relies on `TablePrinter.print_table`, `_column_widths.update`, `self._indent_row_names_based_on_depth` to complete the main steps.
**CN:** `print_model_table` 负责实现本模块使用的辅助逻辑。 它主要处理 `column_widths` 等参数。 实现过程中会调用 `TablePrinter.print_table`, `_column_widths.update`, `self._indent_row_names_based_on_depth` 等函数完成关键步骤。

### LayerwiseProfileResults.print_summary_table (lines 117-133)
```python
    def print_summary_table(self, column_widths: dict[str, int] | None = None):
        _column_widths = dict(
            name=80, cuda_time_us=12, pct_cuda_time=12, invocations=15
        )
        if column_widths:
            _column_widths.update(**column_widths)
        filtered_summary_table = [
            (depth, row)
            for depth, row in self._flatten_stats_tree(self._summary_stats_tree)
            if row.cuda_time_us > 0
        ]
        TablePrinter(SummaryStatsEntry, _column_widths).print_table(
            self._indent_row_names_based_on_depth(
                filtered_summary_table,
                indent_style=lambda indent: "|" + "-" * indent + " ",
            )
        )
```
**EN:** `print_summary_table` implements helper logic used by this module. It mainly works with `column_widths`. Inside the body, it relies on `TablePrinter.print_table`, `_column_widths.update`, `self._indent_row_names_based_on_depth` to complete the main steps.
**CN:** `print_summary_table` 负责实现本模块使用的辅助逻辑。 它主要处理 `column_widths` 等参数。 实现过程中会调用 `TablePrinter.print_table`, `_column_widths.update`, `self._indent_row_names_based_on_depth` 等函数完成关键步骤。

### layerwise_profile (lines 373-400)
```python
class layerwise_profile(profile):
    def __init__(self, num_running_seqs: int | None = None):
        """
        layerwise profile constructor.

        Args:
            num_running_seqs (Optional[int], optional): When given,
                num_running_seqs will be passed to LayerProfileResults
                for metadata update. Defaults to None.
        """
        super().__init__(
            activities=[ProfilerActivity.CPU, ProfilerActivity.CUDA],
            record_shapes=True,
            with_stack=True,
            with_modules=True,
            experimental_config=_ExperimentalConfig(verbose=True),
        )

        self.num_running_seqs = num_running_seqs

    def __enter__(self):
        return super().__enter__()

    def __exit__(self, exc_type, exc_val, exc_tb):
        super().__exit__(exc_type, exc_val, exc_tb)
        self.results = LayerwiseProfileResults(
            self.profiler.kineto_results, num_running_seqs=self.num_running_seqs
        )
```
**EN:** Defines the `layerwise_profile` class used by this module. It extends `profile`. Key methods include `__init__`.
**CN:** `layerwise_profile` 是该文件中的核心类，用于封装与 `layerwise_profile` 相关的状态和行为。 它继承自 `profile`。 关键方法包括 `__init__`。

## Key Concepts / 关键概念
- **`_ModuleTreeNode`**: Core class that organizes module behavior. / **`_ModuleTreeNode`**：组织模块行为的核心类。
- **`SummaryStatsEntry`**: Core class that organizes module behavior. / **`SummaryStatsEntry`**：组织模块行为的核心类。
- **`ModelStatsEntry`**: Core class that organizes module behavior. / **`ModelStatsEntry`**：组织模块行为的核心类。

## Dependencies / 依赖关系
- **Standard library / 标准库**: copy, collections, collections.abc, dataclasses, typing
- **Third-party / 第三方**: torch._C._autograd, torch._C._profiler, torch.autograd.profiler, torch.profiler, pandas
- **Internal vLLM / vLLM 内部依赖**: vllm.profiler.utils, vllm.utils.import_utils
