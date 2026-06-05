# test_batch_reordering.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/v1/attention/test_batch_reordering.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Validates `batch reordering` behavior and regressions in the v1 stack. / 验证 v1 栈中 `批处理 reordering` 相关行为与回归。

## Line-by-Line Analysis / 逐行分析
### Module header / 模块头 (lines 1-3)
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
```
**EN:** File header with 2 non-empty line(s) of license or metadata comments.
**CN:** 文件头包含 2 行非空的许可证或元数据注释。

### Imports and setup / 导入与设置 (lines 4-9)
```python
from dataclasses import dataclass

import numpy as np
import pytest

from vllm.v1.attention.backends.utils import reorder_batch_to_split_decodes_and_prefills
```
**EN:** Imports the libraries needed to build the test harness. External dependencies include `numpy, pytest`. vLLM modules under test include `vllm.v1.attention.backends.utils`.
**CN:** 该代码块导入构建测试环境所需的库。 外部依赖包括 `numpy, pytest`。 被测试的 vLLM 模块包括 `vllm.v1.attention.backends.utils`。

### MockInputBatch (lines 12-27)
```python
class MockInputBatch:
    def __init__(self, req_ids, num_computed_tokens_cpu, num_prompt_tokens):
        self.req_ids = req_ids
        self.num_computed_tokens_cpu = num_computed_tokens_cpu
        self.num_prompt_tokens = num_prompt_tokens

    def swap_states(self, i, j):
        self.req_ids[i], self.req_ids[j] = self.req_ids[j], self.req_ids[i]
        self.num_computed_tokens_cpu[i], self.num_computed_tokens_cpu[j] = (
            self.num_computed_tokens_cpu[j],
            self.num_computed_tokens_cpu[i],
        )
        self.num_prompt_tokens[i], self.num_prompt_tokens[j] = (
            self.num_prompt_tokens[j],
            self.num_prompt_tokens[i],
        )
```
**EN:** Class `MockInputBatch` groups 0 test method(s) and 2 helper/fixture method(s).
**CN:** 类 `MockInputBatch` 组织了 0 个测试方法，以及 2 个辅助或 fixture 方法。

### MockSchedulerOutput (lines 30-32)
```python
class MockSchedulerOutput:
    def __init__(self, num_scheduled_tokens):
        self.num_scheduled_tokens = num_scheduled_tokens
```
**EN:** Class `MockSchedulerOutput` groups 0 test method(s) and 1 helper/fixture method(s).
**CN:** 类 `MockSchedulerOutput` 组织了 0 个测试方法，以及 1 个辅助或 fixture 方法。

### ReorderTestCase (lines 36-41)
```python
class ReorderTestCase:
    # (num_scheduled_tokens, num_computed_tokens, num_prompt_tokens)
    requests: list[tuple[int, int, int]]
    expected_order: list[int]
    expected_modified: bool
    decode_threshold: int = 1
```
**EN:** Class `ReorderTestCase` groups 0 test method(s).
**CN:** 类 `ReorderTestCase` 组织了 0 个测试方法。

### Module state / 模块级状态 (lines 46-170)
```python
REORDER_TEST_CASES = {
    "all_decodes": ReorderTestCase(
        requests=[(1, 10, 10), (1, 20, 20), (1, 30, 30)],
        expected_order=[0, 1, 2],
        expected_modified=False,
    ),
    "all_long_extends": ReorderTestCase(
        requests=[(100, 100, 100), (200, 200, 200), (300, 300, 300)],
    "mixed_decodes_long_extends": ReorderTestCase(
        requests=[(100, 100, 100), (1, 10, 10), (200, 200, 200), (1, 20, 20)],
        expected_order=[3, 1, 2, 0],
        expected_modified=True,
    "already_ordered": ReorderTestCase(
        requests=[(1, 10, 10), (1, 20, 20), (100, 100, 100), (200, 0, 200)],
    # ... excerpt omitted for brevity ...
        expected_order=[0, 1, 2, 3],
        expected_order=[0],
        ],
        expected_order=[0, 1, 3, 2],
        decode_threshold=5,
}
```
**EN:** Defines module-level constants, feature gates, or shared state. Representative names: `REORDER_TEST_CASES`. Shared setup calls include `ReorderTestCase`.
**CN:** 定义模块级常量、特性开关或共享状态。 代表性名称：`REORDER_TEST_CASES`。 共享初始化调用包括 `ReorderTestCase`。

### test_reorder_batch_to_split_decodes_and_prefills (lines 176-196)
```python
def test_reorder_batch_to_split_decodes_and_prefills(test_case: ReorderTestCase):
    req_ids = [f"r{i}" for i in range(len(test_case.requests))]
    num_computed_tokens = np.array([r[1] for r in test_case.requests], dtype=np.int32)
    num_scheduled_tokens = {f"r{i}": r[0] for i, r in enumerate(test_case.requests)}
    num_prompt_tokens = np.array([r[2] for r in test_case.requests], dtype=np.int32)

    input_batch = MockInputBatch(req_ids, num_computed_tokens, num_prompt_tokens)
    scheduler_output = MockSchedulerOutput(num_scheduled_tokens)

    modified = reorder_batch_to_split_decodes_and_prefills(
        input_batch, scheduler_output, decode_threshold=test_case.decode_threshold
    )

    expected_req_ids = [f"r{i}" for i in test_case.expected_order]

    assert modified == test_case.expected_modified, (
        f"Expected modified={test_case.expected_modified}, got {modified}"
    )
    assert input_batch.req_ids == expected_req_ids, (
        f"Expected order {expected_req_ids}, got {input_batch.req_ids}"
    )
```
**EN:** Parameterized test covering `reorder batch to split decodes and prefills`. Parameter axes: `test_case`. Inputs/fixtures: `test_case`. It exercises `mark.parametrize, np.array, MockInputBatch, MockSchedulerOutput, reorder_batch_to_split_decodes_and_prefills, REORDER_TEST_CASES.values`. The body contains 2 explicit assertion(s).
**CN:** 该代码块是覆盖 `reorder 批处理 to split decodes and prefills` 的测试用例。 参数维度：`test_case`。 输入或 fixture：`test_case`。 该测试会调用 `mark.parametrize, np.array, MockInputBatch, MockSchedulerOutput, reorder_batch_to_split_decodes_and_prefills, REORDER_TEST_CASES.values`。 代码主体包含 2 个显式断言。

## Key Concepts / 关键概念
- **EN:** Pytest parametrization broadens scenario coverage
- **CN:** Pytest 参数化扩展了场景覆盖范围
- **EN:** Mocks isolate dependencies and expose state transitions
- **CN:** Mock 用于隔离依赖并暴露状态迁移
- **EN:** Tensor or array assertions validate numerical/runtime behavior
- **CN:** 张量或数组断言用于验证数值与运行时行为
- **EN:** Attention backend and metadata coverage
- **CN:** 注意力后端与元数据覆盖
- **EN:** Scheduler state transitions and queue management
- **CN:** 调度器状态迁移与队列管理

## Dependencies / 依赖关系
- **EN:** External libraries: `numpy, pytest`.
- **CN:** 外部库：`numpy, pytest`。
- **EN:** vLLM modules under test: `vllm.v1.attention.backends.utils`.
- **CN:** 被测试的 vLLM 模块：`vllm.v1.attention.backends.utils`。
- **EN:** Standard-library support: `dataclasses`.
- **CN:** 标准库支持：`dataclasses`。
