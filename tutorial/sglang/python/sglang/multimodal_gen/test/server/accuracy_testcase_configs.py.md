# accuracy_testcase_configs.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/server/accuracy_testcase_configs.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates accuracy testcase configs with focused assertions and fixtures. Key symbols include `_component_accuracy_key`, `_select_accuracy_cases`, `get_component_duplicate_skip_reason`. / 该测试模块通过有针对性的断言与夹具，验证 accuracy testcase configs 的实现。 关键符号包括 `_component_accuracy_key`, `_select_accuracy_cases`, `get_component_duplicate_skip_reason`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Imports and module setup / 导入与模块初始化
```python
from __future__ import annotations

from sglang.multimodal_gen.test.server.accuracy_config import (
    ComponentType,
    should_skip_component,
)
from sglang.multimodal_gen.test.server.accuracy_utils import (
    extract_component_path_overrides,
)
from sglang.multimodal_gen.test.server.component_accuracy import COMPONENT_SPECS
from sglang.multimodal_gen.test.server.gpu_cases import (
    ONE_GPU_CASES,
    TWO_GPU_CASES,
)
from sglang.multimodal_gen.test.server.testcase_configs import DiffusionTestCase
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 18-36: Function `_component_accuracy_key` / 函数 `_component_accuracy_key`
```python
def _component_accuracy_key(case: DiffusionTestCase, component: ComponentType) -> tuple:
    server_args = case.server_args
    component_paths = extract_component_path_overrides(server_args.extras)
    override_path = None
    for key in (component.value, *COMPONENT_SPECS[component].model_index_keys):
        if key in component_paths:
            override_path = component_paths[key]
            break

    return (
        component.value,
        server_args.model_path,
        override_path,
        server_args.num_gpus,
        server_args.tp_size,
        server_args.ulysses_degree,
        server_args.ring_degree,
        server_args.cfg_parallel,
    )
```
**EN:** This function drives `_component_accuracy_key` with inputs such as `case`, `component`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_component_accuracy_key`，主要处理 `case`, `component` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 37-39: Top-level configuration / 顶层配置
```python


_COMPONENT_DUPLICATE_REASONS: dict[tuple[str, ComponentType], str] = {}
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 42-66: Function `_select_accuracy_cases` / 函数 `_select_accuracy_cases`
```python
def _select_accuracy_cases(cases: list[DiffusionTestCase]) -> list[DiffusionTestCase]:
    selected: list[DiffusionTestCase] = []
    seen: dict[tuple, str] = {}
    for case in cases:
        if not case.run_component_accuracy_check:
            continue

        has_component_to_run = False
        for component in ComponentType:
            if should_skip_component(case, component):
                continue

            key = _component_accuracy_key(case, component)
            representative = seen.get(key)
# ...

        if has_component_to_run:
            selected.append(case)
    return selected
```
**EN:** This function drives `_select_accuracy_cases` with inputs such as `cases`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_select_accuracy_cases`，主要处理 `cases` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 69-72: Function `get_component_duplicate_skip_reason` / 函数 `get_component_duplicate_skip_reason`
```python
def get_component_duplicate_skip_reason(
    case: DiffusionTestCase, component: ComponentType
) -> str | None:
    return _COMPONENT_DUPLICATE_REASONS.get((case.id, component))
```
**EN:** This function drives `get_component_duplicate_skip_reason` with inputs such as `case`, `component`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `get_component_duplicate_skip_reason`，主要处理 `case`, `component` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 73-76: Top-level configuration / 顶层配置
```python


ACCURACY_ONE_GPU_CASES = _select_accuracy_cases(ONE_GPU_CASES)
ACCURACY_TWO_GPU_CASES = _select_accuracy_cases(TWO_GPU_CASES)
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

## Key Concepts / 关键概念
- Automated verification / 自动化验证
- Symbol `_component_accuracy_key` anchors the module API / 符号 `_component_accuracy_key` 构成该模块的核心 API
- Symbol `_select_accuracy_cases` anchors the module API / 符号 `_select_accuracy_cases` 构成该模块的核心 API
- Symbol `get_component_duplicate_skip_reason` anchors the module API / 符号 `get_component_duplicate_skip_reason` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.test.server.accuracy_config`, `sglang.multimodal_gen.test.server.accuracy_utils`, `sglang.multimodal_gen.test.server.component_accuracy`, `sglang.multimodal_gen.test.server.gpu_cases`, `sglang.multimodal_gen.test.server.testcase_configs`
- **External / 外部**: `__future__`
