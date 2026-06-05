# stable_toposort.py — Code Analysis / 代码分析

## Source / 来源

- **Path:** `./python/triton/tools/triton_to_gluon_translator/stable_toposort.py`
- **EN:** This source file at `./python/triton/tools/triton_to_gluon_translator/stable_toposort.py` defines the main symbols `stable_toposort` for the surrounding Triton subsystem.
- **CN:** 位于 `./python/triton/tools/triton_to_gluon_translator/stable_toposort.py` 的这个源文件为周边 Triton 子系统定义了主要符号 `stable_toposort`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1
```python
from __future__ import annotations
```
**EN:** At module scope, this block imports annotations from `__future__` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `__future__` 导入 annotations，把当前文件与周边 API 和辅助工具连接起来。

### Lines 3-3
```python
from collections import deque
```
**EN:** At module scope, this block imports deque from `collections` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `collections` 导入 deque，把当前文件与周边 API 和辅助工具连接起来。

### Lines 4-4
```python
from collections.abc import Mapping
```
**EN:** At module scope, this block imports Mapping from `collections.abc` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `collections.abc` 导入 Mapping，把当前文件与周边 API 和辅助工具连接起来。

### Lines 6-6
```python
from triton.tools.triton_to_gluon_translator.ordered_set import ordered_set
```
**EN:** At module scope, this block imports ordered_set from `triton.tools.triton_to_gluon_translator.ordered_set` to connect this file with nearby APIs and helpers.
**CN:** 在模块级作用域中，这段代码从 `triton.tools.triton_to_gluon_translator.ordered_set` 导入 ordered_set，把当前文件与周边 API 和辅助工具连接起来。

### Lines 9-9
```python
def stable_toposort(edges: Mapping[int, ordered_set[int]]) -> list[int]:  # noqa: C901
```
**EN:** At module scope, this header declares the function `stable_toposort(edges)`, which is responsible for stable toposort.
**CN:** 在模块级作用域中，这段头部声明了函数 `stable_toposort(edges)`，它负责处理 stable toposort 相关逻辑。

### Lines 10-10
```python
    nodes: list[int] = list(edges.keys())
```
**EN:** Inside function `stable_toposort`, this assignment updates `nodes` with `list(edges.keys())`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `stable_toposort` 内部，这段赋值把 `list(edges.keys())` 写入 `nodes`，为后续逻辑建立状态、别名或配置。

### Lines 11-11
```python
    order = {node: idx for idx, node in enumerate(nodes)}
```
**EN:** Inside function `stable_toposort`, this assignment updates `order` with `{node: idx for idx, node in enumerate(nodes)}`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `stable_toposort` 内部，这段赋值把 `{node: idx for idx, node in enumerate(nodes)}` 写入 `order`，为后续逻辑建立状态、别名或配置。

### Lines 12-12
```python
    index: dict[int, int] = {}
```
**EN:** Inside function `stable_toposort`, this assignment updates `index` with `{}`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `stable_toposort` 内部，这段赋值把 `{}` 写入 `index`，为后续逻辑建立状态、别名或配置。

### Lines 13-13
```python
    low: dict[int, int] = {}
```
**EN:** Inside function `stable_toposort`, this assignment updates `low` with `{}`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `stable_toposort` 内部，这段赋值把 `{}` 写入 `low`，为后续逻辑建立状态、别名或配置。

### Lines 14-14
```python
    stack: list[int] = []
```
**EN:** Inside function `stable_toposort`, this assignment updates `stack` with `[]`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `stable_toposort` 内部，这段赋值把 `[]` 写入 `stack`，为后续逻辑建立状态、别名或配置。

### Lines 15-15
```python
    on_stack: ordered_set[int] = ordered_set()
```
**EN:** Inside function `stable_toposort`, this assignment updates `on_stack` with `ordered_set()`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `stable_toposort` 内部，这段赋值把 `ordered_set()` 写入 `on_stack`，为后续逻辑建立状态、别名或配置。

### Lines 16-16
```python
    comp_of: dict[int, int] = {}
```
**EN:** Inside function `stable_toposort`, this assignment updates `comp_of` with `{}`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `stable_toposort` 内部，这段赋值把 `{}` 写入 `comp_of`，为后续逻辑建立状态、别名或配置。

### Lines 17-17
```python
    comps: list[tuple[int, ...]] = []
```
**EN:** Inside function `stable_toposort`, this assignment updates `comps` with `[]`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `stable_toposort` 内部，这段赋值把 `[]` 写入 `comps`，为后续逻辑建立状态、别名或配置。

### Lines 19-19
```python
    def dfs(node: int) -> None:
```
**EN:** Inside function `stable_toposort`, this header declares the function `dfs(node)`, which is responsible for dfs.
**CN:** 在函数 `stable_toposort` 内部，这段头部声明了函数 `dfs(node)`，它负责处理 dfs 相关逻辑。

### Lines 20-20
```python
        idx = len(index)
```
**EN:** Inside function `stable_toposort` -> `dfs`, this assignment updates `idx` with `len(index)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `stable_toposort` -> `dfs` 内部，这段赋值把 `len(index)` 写入 `idx`，为后续逻辑建立状态、别名或配置。

### Lines 21-21
```python
        index[node] = low[node] = idx
```
**EN:** Inside function `stable_toposort` -> `dfs`, this assignment updates `index[node], low[node]` with `idx`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `stable_toposort` -> `dfs` 内部，这段赋值把 `idx` 写入 `index[node], low[node]`，为后续逻辑建立状态、别名或配置。

### Lines 22-22
```python
        stack.append(node)
```
**EN:** Inside function `stable_toposort` -> `dfs`, this expression evaluates `stack.append` mainly for its side effects or registration behavior.
**CN:** 在函数 `stable_toposort` -> `dfs` 内部，这条表达式计算 `stack.append`，主要目的是触发副作用或完成注册行为。

### Lines 23-23
```python
        on_stack.add(node)
```
**EN:** Inside function `stable_toposort` -> `dfs`, this expression evaluates `on_stack.add` mainly for its side effects or registration behavior.
**CN:** 在函数 `stable_toposort` -> `dfs` 内部，这条表达式计算 `on_stack.add`，主要目的是触发副作用或完成注册行为。

### Lines 24-29
```python
        for nxt in edges[node]:
            if nxt not in index:
                dfs(nxt)
                low[node] = min(low[node], low[nxt])
            elif nxt in on_stack:
                low[node] = min(low[node], index[nxt])
```
**EN:** Inside function `stable_toposort` -> `dfs`, this loop iterates `nxt` over `edges[node]` and applies the loop body to each item.
**CN:** 在函数 `stable_toposort` -> `dfs` 内部，这段循环让 `nxt` 遍历 `edges[node]`，并对每个元素执行循环体。

### Lines 30-39
```python
        if low[node] == index[node]:
            comp: list[int] = []
            while True:
                cur = stack.pop()
                on_stack.remove(cur)
                comp.append(cur)
                comp_of[cur] = len(comps)
                if cur == node:
                    break
            comps.append(tuple(sorted(comp, key=order.__getitem__)))
```
**EN:** Inside function `stable_toposort` -> `dfs`, this conditional checks `low[node] == index[node]` and then runs the matching branch, including any nested `elif` or `else` logic.
**CN:** 在函数 `stable_toposort` -> `dfs` 内部，这段条件语句检查 `low[node] == index[node]`，然后执行匹配的分支，包括可能的 `elif` 或 `else` 逻辑。

### Lines 41-43
```python
    for node in nodes:
        if node not in index:
            dfs(node)
```
**EN:** Inside function `stable_toposort`, this loop iterates `node` over `nodes` and applies the loop body to each item.
**CN:** 在函数 `stable_toposort` 内部，这段循环让 `node` 遍历 `nodes`，并对每个元素执行循环体。

### Lines 45-45
```python
    priority = [min(order[n] for n in comp) for comp in comps]
```
**EN:** Inside function `stable_toposort`, this assignment updates `priority` with `[min((order[n] for n in comp)) for comp in comps]`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `stable_toposort` 内部，这段赋值把 `[min((order[n] for n in comp)) for comp in comps]` 写入 `priority`，为后续逻辑建立状态、别名或配置。

### Lines 46-46
```python
    indegree = [0] * len(comps)
```
**EN:** Inside function `stable_toposort`, this assignment updates `indegree` with `[0] * len(comps)`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `stable_toposort` 内部，这段赋值把 `[0] * len(comps)` 写入 `indegree`，为后续逻辑建立状态、别名或配置。

### Lines 47-47
```python
    outgoing: list[ordered_set[int]] = [ordered_set() for _ in comps]
```
**EN:** Inside function `stable_toposort`, this assignment updates `outgoing` with `[ordered_set() for _ in comps]`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `stable_toposort` 内部，这段赋值把 `[ordered_set() for _ in comps]` 写入 `outgoing`，为后续逻辑建立状态、别名或配置。

### Lines 48-54
```python
    for node in nodes:
        src = comp_of[node]
        for nbr in edges[node]:
            dst = comp_of[nbr]
            if src != dst and dst not in outgoing[src]:
                outgoing[src].add(dst)
                indegree[dst] += 1
```
**EN:** Inside function `stable_toposort`, this loop iterates `node` over `nodes` and applies the loop body to each item.
**CN:** 在函数 `stable_toposort` 内部，这段循环让 `node` 遍历 `nodes`，并对每个元素执行循环体。

### Lines 56-56
```python
    queue = deque(sorted((cid for cid, deg in enumerate(indegree) if deg == 0), key=priority.__getitem__))
```
**EN:** Inside function `stable_toposort`, this assignment updates `queue` with `deque(sorted((cid for cid, deg in enumerate(indegree) if deg == 0), key=prior...`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `stable_toposort` 内部，这段赋值把 `deque(sorted((cid for cid, deg in enumerate(indegree) if deg == 0), key=prior...` 写入 `queue`，为后续逻辑建立状态、别名或配置。

### Lines 57-57
```python
    result: list[int] = []
```
**EN:** Inside function `stable_toposort`, this assignment updates `result` with `[]`, establishing state, aliases, or configuration used later.
**CN:** 在函数 `stable_toposort` 内部，这段赋值把 `[]` 写入 `result`，为后续逻辑建立状态、别名或配置。

### Lines 58-64
```python
    while queue:
        cid = queue.popleft()
        result.extend(comps[cid])
        for nxt in sorted(outgoing[cid], key=priority.__getitem__):
            indegree[nxt] -= 1
            if indegree[nxt] == 0:
                queue.append(nxt)
```
**EN:** Inside function `stable_toposort`, this loop keeps running while `queue` remains true.
**CN:** 在函数 `stable_toposort` 内部，这段循环会在 `queue` 为真时持续执行。

### Lines 66-66
```python
    assert len(result) == len(nodes), "component graph contains a cycle"
```
**EN:** Inside function `stable_toposort`, this assertion enforces `len(result) == len(nodes)` so invalid states are caught early during execution.
**CN:** 在函数 `stable_toposort` 内部，这条断言要求 `len(result) == len(nodes)` 成立，从而在执行早期捕获非法状态。

### Lines 67-67
```python
    return result
```
**EN:** Inside function `stable_toposort`, this return statement sends `result` back to the caller as the result of the current routine.
**CN:** 在函数 `stable_toposort` 内部，这条返回语句把 `result` 作为当前过程的结果返回给调用方。

## Key Concepts / 关键概念

- **EN:** Path theme: `python/triton/tools/triton_to_gluon_translator` places this module in Triton's triton / tools / triton_to_gluon_translator area.
  **CN:** 路径主题：`python/triton/tools/triton_to_gluon_translator` 表明该模块位于 Triton 的 triton / tools / triton_to_gluon_translator 领域。
- **EN:** Primary functions: `stable_toposort`.
  **CN:** 主要函数：`stable_toposort`。
- **EN:** Tooling surface: this file is part of Triton's CLI or code-transformation utilities.
  **CN:** 工具层：该文件属于 Triton 的命令行或代码转换工具。

## Dependencies / 依赖关系

- **EN:** Standard-library dependencies: __future__, collections, collections.abc.
  **CN:** 标准库依赖：__future__, collections, collections.abc。
- **EN:** Internal Triton modules: triton.tools.triton_to_gluon_translator.ordered_set.
  **CN:** Triton 内部模块：triton.tools.triton_to_gluon_translator.ordered_set。
