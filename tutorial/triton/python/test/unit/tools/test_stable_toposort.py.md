# test_stable_toposort.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/test/unit/tools/test_stable_toposort.py`
- **EN:** Pytest module covering stable toposort behavior in Triton's Python tests. It contains 9 top-level definition(s) and 5 imported module reference(s).
- **CN:** 这是一个 pytest 模块，用于覆盖 Triton Python 测试中的 stable toposort 行为。 该文件包含 9 个顶层定义，以及 5 个导入模块引用。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7

```python
from collections import deque
from dataclasses import dataclass

import pytest

from triton.tools.triton_to_gluon_translator.ordered_set import ordered_set
from triton.tools.triton_to_gluon_translator.stable_toposort import stable_toposort
```
- **EN:** Imports the modules used in this scope: `collections`, `dataclasses`, `pytest`, `triton.tools.triton_to_gluon_translator.ordered_set`, `triton.tools.triton_to_gluon_translator.stable_toposort`.
- **CN:** 导入此作用域使用的模块：`collections`、`dataclasses`、`pytest`、`triton.tools.triton_to_gluon_translator.ordered_set`、`triton.tools.triton_to_gluon_translator.stable_toposort`。

### Lines 8-11

```python


@dataclass(frozen=True)
class Case:
```
- **EN:** Defines class `Case`. Decorators: `dataclass(frozen=True)`.
- **CN:** 定义类 `Case`。 装饰器：`dataclass(frozen=True)`。

#### Lines 12-14

```python
    name: str
    graph: dict[int, ordered_set[int]]
    expected: list[int]
```
- **EN:** Prepares or updates state through `name`, `graph`, `expected`.
- **CN:** 通过 `name`、`graph`、`expected` 准备或更新状态。

### Lines 15-17

```python


CASES: list[Case] = []
```
- **EN:** Prepares or updates state through `CASES`.
- **CN:** 通过 `CASES` 准备或更新状态。

### Lines 18-20

```python


def add_case(name: str, mapping: list[tuple[int, list[int]]], expected_order: list[int]):
```
- **EN:** Defines the helper function `add_case`. Parameters: `name`, `mapping`, `expected_order`. Key calls include `CASES.append`, `ordered_set`, `Case`.
- **CN:** 定义辅助函数 `add_case`。 参数：`name`、`mapping`、`expected_order`。 关键调用包括 `CASES.append`、`ordered_set`、`Case`。

#### Lines 21-22

```python
    graph = {node: ordered_set(children) for node, children in mapping}
    CASES.append(Case(name, graph, expected_order))
```
- **EN:** Prepares or updates state through `graph`. Invokes `ordered_set`, `CASES.append`, `Case` to execute the test logic.
- **CN:** 通过 `graph` 准备或更新状态。 调用 `ordered_set`、`CASES.append`、`Case` 执行测试逻辑。

### Lines 23-30

```python


add_case("empty", [], [])
add_case("single_node", [(0, [])], [0])
add_case("single_self_loop", [(0, [0])], [0])
add_case("single_edge", [(0, [1]), (1, [])], [0, 1])
add_case("two_disconnected", [(0, []), (1, [])], [0, 1])
add_case("child_only_node", [(0, [1]), (1, [])], [0, 1])
```
- **EN:** Invokes `add_case` to execute the test logic.
- **CN:** 调用 `add_case` 执行测试逻辑。

### Lines 31-35

```python

for length in range(3, 11):
    mapping = [(idx, [idx + 1]) for idx in range(length - 1)]
    mapping.append((length - 1, []))
    add_case(f"chain_{length}", mapping, list(range(length)))
```
- **EN:** Invokes `mapping.append`, `add_case` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `mapping.append`、`add_case` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

### Lines 36-40

```python

for size in range(2, 7):
    leaves = list(range(1, size + 1))
    mapping = [(0, leaves)] + [(leaf, []) for leaf in leaves]
    add_case(f"star_out_{size}", mapping, [0] + leaves)
```
- **EN:** Invokes `add_case` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `add_case` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

### Lines 41-45

```python

for size in range(2, 7):
    leaves = list(range(1, size + 1))
    mapping = [(leaf, [0]) for leaf in leaves] + [(0, [])]
    add_case(f"star_in_{size}", mapping, leaves + [0])
```
- **EN:** Invokes `add_case` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `add_case` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

### Lines 46-280

```python

add_case("diamond", [(0, [1, 2]), (1, [3]), (2, [3]), (3, [])], [0, 1, 2, 3])
add_case("wide_branch", [(0, [1, 2, 3]), (1, [4]), (2, [4]), (3, [4]), (4, [])], [0, 1, 2, 3, 4])
add_case(
    "deep_branch",
    [(0, [1]), (1, [2, 3]), (2, [4]), (3, [4]), (4, [5]), (5, [])],
    [0, 1, 2, 3, 4, 5],
)

add_case("two_cycle", [(1, [2]), (2, [1])], [1, 2])
add_case("three_cycle", [(0, [1]), (1, [2]), (2, [0])], [0, 1, 2])
add_case(
    "four_cycle_with_tail",
    [(0, [1]), (1, [2]), (2, [3]), (3, [0, 4]), (4, [])],
    [0, 1, 2, 3, 4],
)
add_case(
    "cycle_with_entry",
    [(5, [0]), (0, [1]), (1, [2]), (2, [0, 3]), (3, [])],
    [5, 0, 1, 2, 3],
)
add_case(
    "cycle_with_exit",
    [(0, [1]), (1, [2]), (2, [0, 3]), (3, [4]), (4, [])],
    [0, 1, 2, 3, 4],
)
add_case(
    "two_disjoint_cycles",
    [(0, [1]), (1, [0]), (2, [3]), (3, [2])],
    [0, 1, 2, 3],
)
add_case(
    "cycle_bridge",
    [(0, [1]), (1, [0, 2]), (2, [3]), (3, [4]), (4, [])],
    [0, 1, 2, 3, 4],
)
add_case(
    "self_loop_with_outgoing",
    [(0, [0, 1]), (1, [2]), (2, [])],
    [0, 1, 2],
)
add_case(
    "multi_roots",
    [(2, [3]), (0, [2]), (1, [3]), (3, [])],
    [0, 1, 2, 3],
)
add_case(
    "sink_only",
    [(0, [1, 2]), (1, []), (2, [])],
    [0, 1, 2],
)
add_case(
    "source_only",
    [(0, []), (1, [2]), (2, [3]), (3, [])],
    [0, 1, 2, 3],
)
add_case(
    "nested_cycles",
    [(0, [1]), (1, [2, 4]), (2, [0, 3]), (3, [2]), (4, [5]), (5, [])],
    [0, 1, 2, 3, 4, 5],
)
add_case(
    "cycle_and_chain",
    [(0, [1]), (1, [0]), (2, [3]), (3, [4]), (4, [])],
    [0, 1, 2, 3, 4],
)
add_case(
    "shared_child",
    [(0, [2]), (1, [2]), (2, [3]), (3, [])],
    [0, 1, 2, 3],
)
add_case(
    "shared_parent",
    [(0, [1, 2]), (1, []), (2, [])],
    [0, 1, 2],
)
add_case(
    "long_cycle_with_branch",
    [(0, [1]), (1, [2]), (2, [3, 5]), (3, [0]), (4, [0]), (5, [6]), (6, [])],
    [4, 0, 1, 2, 3, 5, 6],
)
add_case(
    "incoming_from_multiple",
    [(0, [3]), (1, [3]), (2, [3]), (3, [4]), (4, [])],
    [0, 1, 2, 3, 4],
)
add_case(
    "cycle_with_self_loop",
    [(0, [0, 1]), (1, [2]), (2, [0])],
    [0, 1, 2],
)
add_case(
    "reverse_chain",
    [(3, []), (2, [3]), (1, [2]), (0, [1])],
    [0, 1, 2, 3],
)
add_case(
    "disconnected_dag",
    [(0, [1]), (1, []), (2, [3]), (3, [])],
    [0, 2, 1, 3],
)
add_case(
    "disconnected_cycle_and_node",
    [(4, []), (0, [1]), (1, [2]), (2, [0])],
    [4, 0, 1, 2],
)
add_case(
    "cycle_with_multiple_exits",
    [(0, [1, 3]), (1, [2]), (2, [0]), (3, [4]), (4, [])],
    [0, 1, 2, 3, 4],
)
add_case(
    "parallel_paths",
    [(0, [2]), (1, [2]), (2, [3]), (3, [4]), (4, [])],
    [0, 1, 2, 3, 4],
)
add_case(
    "long_branching",
    [(0, [1, 2]), (1, [3, 4]), (2, [4]), (3, [5]), (4, [5]), (5, [])],
    [0, 1, 2, 3, 4, 5],
)
add_case(
    "looping_tail",
    [(0, [1]), (1, [2]), (2, [1, 3]), (3, [4]), (4, [])],
    [0, 1, 2, 3, 4],
)
add_case(
    "multi_component_dag",
    [(0, [1]), (1, []), (2, [3]), (3, []), (4, [5]), (5, [])],
    [0, 2, 4, 1, 3, 5],
)
add_case(
    "multiple_zero_indegree",
    [(2, [4]), (0, [3]), (1, [3]), (3, [5]), (4, [5]), (5, [])],
    [2, 0, 1, 4, 3, 5],
)
add_case(
    "interleaved_cycle",
    [(0, [2]), (1, [0]), (2, [1])],
    [0, 1, 2],
)
add_case(
    "self_loops_multiple",
    [(0, [0]), (1, [1]), (2, [2])],
    [0, 1, 2],
)
add_case(
    "cycle_with_disconnected_tail",
    [(0, [1]), (1, [0]), (2, []), (3, [4]), (4, [])],
    [0, 1, 2, 3, 4],
)
add_case(
    "cycle_with_child_only_nodes",
    [(0, [1, 2]), (1, [0]), (2, []), (3, [4]), (4, [])],
    [0, 1, 3, 2, 4],
)
add_case(
    "long_linear_with_skip",
    [(0, [1, 2]), (1, [3]), (2, [3]), (3, [4]), (4, [5]), (5, [])],
    [0, 1, 2, 3, 4, 5],
)
add_case(
    "incoming_chain_to_cycle",
    [(0, [1]), (1, [2]), (2, [3]), (3, [1])],
    [0, 1, 2, 3],
)
add_case(
    "outgoing_chain_from_cycle",
    [(0, [1]), (1, [0, 2]), (2, [3]), (3, [])],
    [0, 1, 2, 3],
)
add_case(
    "disconnected_self_loops",
    [(0, [0]), (2, [2]), (1, [])],
    [0, 2, 1],
)
add_case(
    "complex_cycle_mix",
    [(0, [1, 2]), (1, [0, 3]), (2, [4]), (3, [1, 5]), (4, [2]), (5, [])],
    [0, 1, 3, 2, 4, 5],
)
add_case(
    "isolated_node",
    [(0, []), (2, [3]), (3, [])],
    [0, 2, 3],
)
add_case(
    "large_cycle",
    [(0, [1]), (1, [2]), (2, [3]), (3, [4]), (4, [0])],
    [0, 1, 2, 3, 4],
)
add_case(
    "cycle_with_reentry",
    [(0, [1]), (1, [2, 3]), (2, [0]), (3, [4]), (4, [2])],
    [0, 1, 2, 3, 4],
)
add_case(
    "multi_branch_cycle",
    [(0, [1, 2]), (1, [0]), (2, [3]), (3, [4, 5]), (4, []), (5, [])],
    [0, 1, 2, 3, 4, 5],
)
add_case(
    "redundant_edges",
    [(0, [1, 1, 1]), (1, [2, 2]), (2, [])],
    [0, 1, 2],
)
add_case(
    "multiple_component_cycles",
    [(0, [1]), (1, [0]), (2, [3]), (3, [4]), (4, [2])],
    [0, 1, 2, 3, 4],
)
add_case(
    "child_only_multiple",
    [(0, [1, 2, 3]), (1, []), (2, []), (3, [])],
    [0, 1, 2, 3],
)
add_case(
    "complex_dag",
    [
        (0, [1, 2, 3]),
        (1, [4, 5]),
        (2, [5, 6]),
        (3, [6]),
        (4, [7]),
        (5, [7]),
        (6, [7]),
        (7, []),
    ],
    [0, 1, 2, 3, 4, 5, 6, 7],
)
add_case(
    "mixed_self_and_cycle",
    [(0, [0, 1]), (1, [2]), (2, [3]), (3, [1])],
    [0, 1, 2, 3],
)
```
- **EN:** Invokes `add_case` to execute the test logic.
- **CN:** 调用 `add_case` 执行测试逻辑。

### Lines 281-283

```python


def normalize_graph(graph: dict[int, ordered_set[int]]) -> dict[int, ordered_set[int]]:
```
- **EN:** Defines the helper function `normalize_graph`. Parameters: `graph`. Key calls include `ordered_set`, `graph.values`, `nodes.update`, `graph.get`.
- **CN:** 定义辅助函数 `normalize_graph`。 参数：`graph`。 关键调用包括 `ordered_set`、`graph.values`、`nodes.update`、`graph.get`。

#### Lines 284-284

```python
    nodes = ordered_set(graph)
```
- **EN:** Prepares or updates state through `nodes`. Invokes `ordered_set` to execute the test logic.
- **CN:** 通过 `nodes` 准备或更新状态。 调用 `ordered_set` 执行测试逻辑。

#### Lines 285-286

```python
    for children in graph.values():
        nodes.update(children)
```
- **EN:** Invokes `graph.values`, `nodes.update` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `graph.values`、`nodes.update` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

#### Lines 287-287

```python
    return {node: ordered_set(graph.get(node, ordered_set())) for node in nodes}
```
- **EN:** Invokes `ordered_set`, `graph.get` to execute the test logic.
- **CN:** 调用 `ordered_set`、`graph.get` 执行测试逻辑。

### Lines 288-290

```python


def reachable(graph: dict[int, ordered_set[int]], src: int, dst: int) -> bool:
```
- **EN:** Defines the helper function `reachable`. Parameters: `graph`, `src`, `dst`. Key calls include `deque`, `queue.popleft`, `graph.get`, `seen.add`, `queue.append`.
- **CN:** 定义辅助函数 `reachable`。 参数：`graph`、`src`、`dst`。 关键调用包括 `deque`、`queue.popleft`、`graph.get`、`seen.add`、`queue.append`。

#### Lines 291-292

```python
    if src == dst:
        return True
```
- **EN:** Branches on runtime or test conditions.
- **CN:** 根据运行时或测试条件进行分支。

#### Lines 293-294

```python
    seen = {src}
    queue = deque([src])
```
- **EN:** Prepares or updates state through `seen`, `queue`. Invokes `deque` to execute the test logic.
- **CN:** 通过 `seen`、`queue` 准备或更新状态。 调用 `deque` 执行测试逻辑。

#### Lines 295-302

```python
    while queue:
        node = queue.popleft()
        for nbr in graph.get(node, ()):  # type: ignore[arg-type]
            if nbr == dst:
                return True
            if nbr not in seen:
                seen.add(nbr)
                queue.append(nbr)
```
- **EN:** Invokes `queue.popleft`, `graph.get`, `seen.add`, `queue.append` to execute the test logic. Iterates across cases or data tiles.
- **CN:** 调用 `queue.popleft`、`graph.get`、`seen.add`、`queue.append` 执行测试逻辑。 通过循环覆盖多个用例或数据分块。

#### Lines 303-303

```python
    return False
```
- **EN:** Carries supporting logic for the surrounding test or helper scope.
- **CN:** 为周围的测试或辅助作用域提供支撑逻辑。

### Lines 304-306

```python


def span_order(nodes: list[int]) -> dict[int, int]:
```
- **EN:** Defines the helper function `span_order`. Parameters: `nodes`. Key calls include `enumerate`.
- **CN:** 定义辅助函数 `span_order`。 参数：`nodes`。 关键调用包括 `enumerate`。

#### Lines 307-307

```python
    return {node: idx for idx, node in enumerate(nodes)}
```
- **EN:** Invokes `enumerate` to execute the test logic.
- **CN:** 调用 `enumerate` 执行测试逻辑。

### Lines 308-310

```python


def check_case(graph: dict[int, ordered_set[int]], expected_order: list[int]) -> None:
```
- **EN:** Defines the helper function `check_case`. Parameters: `graph`, `expected_order`. Key calls include `normalize_graph`, `stable_toposort`, `span_order`, `original.items`, `ordered_set`, `graph.items`, and 1 more.
- **CN:** 定义辅助函数 `check_case`。 参数：`graph`、`expected_order`。 关键调用包括 `normalize_graph`、`stable_toposort`、`span_order`、`original.items`、`ordered_set`、`graph.items` 等另外 1 项。

#### Lines 311-313

```python
    graph_copy = {node: ordered_set(children) for node, children in graph.items()}
    original = normalize_graph(graph_copy)
    result = stable_toposort(graph_copy)
```
- **EN:** Prepares or updates state through `graph_copy`, `original`, `result`. Invokes `ordered_set`, `graph.items`, `normalize_graph`, `stable_toposort` to execute the test logic.
- **CN:** 通过 `graph_copy`、`original`、`result` 准备或更新状态。 调用 `ordered_set`、`graph.items`、`normalize_graph`、`stable_toposort` 执行测试逻辑。

#### Lines 314-315

```python
    assert list(result) == expected_order
    assert ordered_set(result) == ordered_set(original)
```
- **EN:** Invokes `ordered_set` to execute the test logic. Validates behavior with 2 assertion(s).
- **CN:** 调用 `ordered_set` 执行测试逻辑。 通过 2 个断言验证行为。

#### Lines 316-317

```python

    order = span_order(list(result))
```
- **EN:** Prepares or updates state through `order`. Invokes `span_order` to execute the test logic.
- **CN:** 通过 `order` 准备或更新状态。 调用 `span_order` 执行测试逻辑。

#### Lines 318-321

```python
    for src, children in original.items():
        for dst in children:
            if not reachable(original, dst, src):
                assert order[src] < order[dst]
```
- **EN:** Invokes `original.items`, `reachable` to execute the test logic. Validates behavior with 1 assertion(s). Iterates across cases or data tiles.
- **CN:** 调用 `original.items`、`reachable` 执行测试逻辑。 通过 1 个断言验证行为。 通过循环覆盖多个用例或数据分块。

#### Lines 322-324

```python

    for node in original:
        assert node in result
```
- **EN:** Validates behavior with 1 assertion(s). Iterates across cases or data tiles.
- **CN:** 通过 1 个断言验证行为。 通过循环覆盖多个用例或数据分块。

### Lines 325-328

```python


@pytest.mark.parametrize("case", CASES, ids=lambda case: case.name)
def test_stable_toposort_cases(case: Case) -> None:
```
- **EN:** Defines the test function `test_stable_toposort_cases`. Decorators: `pytest.mark.parametrize('case', CASES, ids=lambda case: case.name)`. Parameters: `case`. Key calls include `pytest.mark.parametrize`, `check_case`. This scope touches pytest parametrization.
- **CN:** 定义测试函数 `test_stable_toposort_cases`。 装饰器：`pytest.mark.parametrize('case', CASES, ids=lambda case: case.name)`。 参数：`case`。 关键调用包括 `pytest.mark.parametrize`、`check_case`。 该作用域涉及pytest 参数化。

#### Lines 329-329

```python
    check_case(case.graph, case.expected)
```
- **EN:** Invokes `check_case` to execute the test logic.
- **CN:** 调用 `check_case` 执行测试逻辑。

### Lines 330-332

```python


def test_input_not_mutated() -> None:
```
- **EN:** Defines the test function `test_input_not_mutated`. Key calls include `stable_toposort`, `ordered_set`, `graph.items`.
- **CN:** 定义测试函数 `test_input_not_mutated`。 关键调用包括 `stable_toposort`、`ordered_set`、`graph.items`。

#### Lines 333-335

```python
    graph = {0: {1}, 1: {2}, 2: ordered_set()}
    snapshot = {k: ordered_set(v) for k, v in graph.items()}
    stable_toposort(graph)
```
- **EN:** Prepares or updates state through `graph`, `snapshot`. Invokes `ordered_set`, `graph.items`, `stable_toposort` to execute the test logic.
- **CN:** 通过 `graph`、`snapshot` 准备或更新状态。 调用 `ordered_set`、`graph.items`、`stable_toposort` 执行测试逻辑。

#### Lines 336-336

```python
    assert graph == snapshot
```
- **EN:** Validates behavior with 1 assertion(s).
- **CN:** 通过 1 个断言验证行为。

### Lines 337-339

```python


def test_stable_toposort_preserves_component_order():
```
- **EN:** Defines the test function `test_stable_toposort_preserves_component_order`. Key calls include `ordered_set`, `stable_toposort`.
- **CN:** 定义测试函数 `test_stable_toposort_preserves_component_order`。 关键调用包括 `ordered_set`、`stable_toposort`。

#### Lines 340-346

```python
    graph = {
        0: ordered_set([1]),
        1: ordered_set([2]),
        2: ordered_set([0, 3]),
        3: ordered_set([4]),
        4: ordered_set(),
    }
```
- **EN:** Prepares or updates state through `graph`. Invokes `ordered_set` to execute the test logic.
- **CN:** 通过 `graph` 准备或更新状态。 调用 `ordered_set` 执行测试逻辑。

#### Lines 347-347

```python
    assert stable_toposort(graph) == [0, 1, 2, 3, 4]
```
- **EN:** Invokes `stable_toposort` to execute the test logic. Validates behavior with 1 assertion(s).
- **CN:** 调用 `stable_toposort` 执行测试逻辑。 通过 1 个断言验证行为。

## Key Concepts / 关键概念

- **EN:** Top-level scopes such as `Case`, `add_case`, `normalize_graph`, `reachable`, `span_order`, `check_case`, `test_stable_toposort_cases`, `test_input_not_mutated`
  **CN:** 顶层作用域，例如 `Case`、`add_case`、`normalize_graph`、`reachable`、`span_order`、`check_case`、`test_stable_toposort_cases`、`test_input_not_mutated`
- **EN:** pytest parametrization
  **CN:** pytest 参数化

## Dependencies / 依赖关系

- **EN:** External or absolute imports include `collections`, `dataclasses`, `pytest`, `triton.tools.triton_to_gluon_translator.ordered_set`, `triton.tools.triton_to_gluon_translator.stable_toposort`.
  **CN:** 外部或绝对导入包括 `collections`、`dataclasses`、`pytest`、`triton.tools.triton_to_gluon_translator.ordered_set`、`triton.tools.triton_to_gluon_translator.stable_toposort`。
- **EN:** Execution centers on top-level definitions such as `Case`, `add_case`, `normalize_graph`, `reachable`, `span_order`, `check_case`, `test_stable_toposort_cases`, `test_input_not_mutated`, `test_stable_toposort_preserves_component_order`.
  **CN:** 执行逻辑主要围绕顶层定义展开，例如 `Case`、`add_case`、`normalize_graph`、`reachable`、`span_order`、`check_case`、`test_stable_toposort_cases`、`test_input_not_mutated`、`test_stable_toposort_preserves_component_order`。
- **EN:** Runtime behavior also depends on pytest collection, Python execution semantics, and the imported Triton/PyTorch utilities visible above.
  **CN:** 运行时行为还依赖 pytest 的收集机制、Python 执行语义，以及上文可见的 Triton/PyTorch 工具。
