# graph_id_filter.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_dynamo/graph_id_filter.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Implements torch.compile front-end capture, guard management, and graph/lowering helpers in TorchDynamo. Key symbols exposed here include `GraphIdFilter`, `T`, `_GraphRouterBase`, `GraphBackendRouter`.
- **Purpose (CN)**: 实现 TorchDynamo 中的 torch.compile 前端捕获、守卫管理与图/降级辅助逻辑。 这里暴露的关键符号包括 `GraphIdFilter`, `T`, `_GraphRouterBase`, `GraphBackendRouter`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-26
```python
# mypy: disallow-untyped-defs

from __future__ import annotations

import functools
import logging
import re
import warnings
from typing import Any, Generic, TYPE_CHECKING, TypeVar


if TYPE_CHECKING:
    from collections.abc import Callable

    from torch._guards import CompileId


log = logging.getLogger(__name__)


class GraphIdFilter:
    """
    A filter for matching graph IDs based on various conditions.
    Supports individual IDs, ranges, and comparison operators.
    """
```
- **EN**: This typing-only branch imports symbols for static analysis without affecting runtime behavior.
- **CN**: 这个仅用于类型检查的分支会导入静态分析所需符号，而不会影响运行时行为。

### Lines 27-48
```python
    def __init__(self, filter_str: str) -> None:
        self._explicit_ids: frozenset[int] = frozenset()
        self._conditions: list[tuple[str, int]] = []
        self._parse(filter_str)

    def _parse(self, filter_str: str) -> None:
        if not filter_str or not filter_str.strip():
            return

        explicit_ids: set[int] = set()
        conditions: list[tuple[str, int]] = []

        # Pattern for comparison operators (>=, >, <=, <) followed by a number
        cmp_pattern = re.compile(r"^(>=|>|<=|<)(\d+)$")
        # Pattern for ranges like "10-20"
        range_pattern = re.compile(r"^(\d+)-(\d+)$")

        for part in filter_str.split(","):
            part = part.strip()
            if not part:
                continue
```
- **EN**: Declares `GraphIdFilter`; this class packages state and methods that hand work to a compiler/backend pipeline.
- **CN**: 声明 `GraphIdFilter`；该类封装了状态与方法，用于将工作移交给编译器或后端流水线。

### Lines 49-67
```python
            if match := cmp_pattern.match(part):
                conditions.append((match.group(1), int(match.group(2))))
            elif match := range_pattern.match(part):
                start, end = int(match.group(1)), int(match.group(2))
                explicit_ids.update(range(start, end + 1))
            else:
                try:
                    explicit_ids.add(int(part))
                except ValueError:
                    log.warning("Invalid graph ID filter: %s", part)

        self._explicit_ids = frozenset(explicit_ids)
        self._conditions = conditions

    def __contains__(self, graph_id: int) -> bool:
        """Check if the given graph ID matches this filter."""
        if graph_id in self._explicit_ids:
            return True
```
- **EN**: Declares `GraphIdFilter`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `GraphIdFilter`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 68-92
```python
        for op, val in self._conditions:
            if op == ">" and graph_id > val:
                return True
            elif op == ">=" and graph_id >= val:
                return True
            elif op == "<" and graph_id < val:
                return True
            elif op == "<=" and graph_id <= val:
                return True

        return False

    def __repr__(self) -> str:
        # pyrefly: ignore [implicit-any]
        parts = []
        if self._explicit_ids:
            parts.append(f"ids={sorted(self._explicit_ids)}")
        if self._conditions:
            parts.append(f"conditions={self._conditions}")
        return f"GraphIdFilter({', '.join(parts) if parts else 'empty'})"


T = TypeVar("T")
```
- **EN**: This range initializes module-level constants or registries that later code reuses to build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这一段初始化模块级常量或注册表，供后续代码重复使用以构建、遍历或改写图结构及其元数据。

### Lines 93-118
```python
class _GraphRouterBase(Generic[T]):
    """
    Base class for routing graphs to different values based on their IDs.

    The router parses a configuration string with rules in the format:
        "filter1:value1;filter2:value2;..."

    Rules are evaluated in order, and the first matching rule wins.
    """

    def __init__(self, config_str: str, rule_type: str) -> None:
        self._rules: list[tuple[GraphIdFilter, T]] = []
        self._values: list[T | None] = []
        self._overflow_value: T | None = None
        self._rule_type = rule_type
        self._parse(config_str)
        self._precompute()

    def _parse_value_str(self, value_str: str) -> T | None:
        """Parse a value string into the appropriate type. Returns None to skip."""
        raise NotImplementedError

    def _parse(self, config_str: str) -> None:
        if not config_str or not config_str.strip():
            return
```
- **EN**: Declares `_GraphRouterBase(Generic[T])`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `_GraphRouterBase(Generic[T])`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 119-144
```python
        rule_strs = config_str.split(";")
        for rule_str in rule_strs:
            rule_str = rule_str.strip()
            if not rule_str:
                continue

            colon_idx = rule_str.find(":")
            if colon_idx == -1:
                log.warning(
                    "Invalid %s override rule (missing ':'): %s",
                    self._rule_type,
                    rule_str,
                )
                continue

            filter_str = rule_str[:colon_idx].strip()
            value_str = rule_str[colon_idx + 1 :].strip()

            if not filter_str or not value_str:
                log.warning("Invalid %s override rule: %s", self._rule_type, rule_str)
                continue

            value = self._parse_value_str(value_str)
            if value is not None:
                self._rules.append((GraphIdFilter(filter_str), value))
```
- **EN**: This block continues `_GraphRouterBase` and works to build, traverse, or rewrite graph structures and their metadata. It also iterates across inputs, nodes, or bookkeeping structures; branches on runtime conditions.
- **CN**: 该代码块继续实现 `_GraphRouterBase`，用于构建、遍历或改写图结构及其元数据。 同时它还会遍历输入、节点或簿记结构；根据运行时条件分支处理。

### Lines 145-169
```python
    def _precompute(self) -> None:
        if not self._rules:
            return

        # Find max ID from explicit IDs and comparison thresholds
        max_id = 0
        for f, _ in self._rules:
            if f._explicit_ids:
                max_id = max(max_id, *f._explicit_ids)
            for _, val in f._conditions:
                max_id = max(max_id, val)

        # Pre-compute values for IDs 0 to max_id
        for i in range(max_id + 1):
            self._values.append(self._match_rules(i))

        # For IDs > max_id, the result is constant (only unbounded conditions apply)
        self._overflow_value = self._match_rules(max_id + 1)

    def _match_rules(self, graph_id: int) -> T | None:
        for f, value in self._rules:
            if graph_id in f:
                return value
        return None
```
- **EN**: Declares `_GraphRouterBase(Generic[T])`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `_GraphRouterBase(Generic[T])`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 170-195
```python
    def get_value_for_graph(self, graph_id: int) -> T | None:
        """Get the value for a given graph ID. Returns None if no rule matches."""
        if graph_id < len(self._values):
            return self._values[graph_id]
        return self._overflow_value

    def is_empty(self) -> bool:
        """Check if no rules are configured."""
        return len(self._rules) == 0


class GraphBackendRouter(_GraphRouterBase[Any]):
    """
    Routes graphs to different backends based on their IDs.

    The router parses a configuration string with rules in the format:
        "filter1:backend1;filter2:backend2;..."

    If a graph ID matches multiple rules with different backends, a ValueError
    is raised.

    Examples:
        "0-5:eager;>5:inductor"     - IDs 0-5 use eager, rest use inductor
        ">10:aot_eager"             - IDs > 10 use aot_eager
        "<=3:eager;4-10:aot_eager"  - IDs 0-3 use eager, 4-10 use aot_eager
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 196-216
```python
    Supported backends include "eager", "aot_eager", "aot_eager_decomp_partition",
    "inductor", and any other registered backend.
    """

    def __init__(self, config_str: str) -> None:
        self._backend_names: dict[int, str] = {}
        super().__init__(config_str, "backend")

    def _parse_value_str(self, value_str: str) -> Any | None:
        """Look up a backend by name."""
        from .backends.registry import lookup_backend
        from .eval_frame import cached_backends

        backend = lookup_backend(value_str)

        # Register the backend so its reset() is called during torch._dynamo.reset()
        assert backend is not None, "Invalid override backend: " + value_str
        cached_backends.setdefault(id(backend), backend)
        self._backend_names[id(backend)] = value_str
        return backend
```
- **EN**: Declares `GraphBackendRouter(_GraphRouterBase[Any])`; this class packages state and methods that hand work to a compiler/backend pipeline.
- **CN**: 声明 `GraphBackendRouter(_GraphRouterBase[Any])`；该类封装了状态与方法，用于将工作移交给编译器或后端流水线。

### Lines 217-241
```python
    def _match_rules(self, graph_id: int) -> Any | None:
        """Match rules with conflict detection for overlapping filters."""
        matches = {id(backend): backend for f, backend in self._rules if graph_id in f}
        if len(matches) > 1:
            names = [self._backend_names[bid] for bid in matches]
            raise ValueError(
                f"Conflicting backend override for graph {graph_id}: matched {names}"
            )
        if matches:
            return next(iter(matches.values()))
        return None

    def __repr__(self) -> str:
        if not self._rules:
            return "GraphBackendRouter(empty)"
        return f"GraphBackendRouter({self._rules})"


class GraphConfigRouter(_GraphRouterBase[dict[str, Any]]):
    """
    Routes graphs to different inductor configs based on their IDs.

    The router parses a configuration string with rules in the format:
        "filter1:config1;filter2:config2;..."
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; validates assumptions before proceeding; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；在继续前校验关键假设；根据运行时条件分支处理。

### Lines 242-259
```python
    All matching rules are aggregated: configs from all matching rules are merged
    into a single dict. Conflicting keys (same key, different value) raise an error.
    Config format is "key=value" or "key1=value1,key2=value2" for multiple settings.

    Examples:
        "0-5:triton.cudagraph_skip_dynamic_graphs=False"
        ">10:triton.cudagraphs=False,triton.cudagraph_trees=False"

        With "0:a=1;>=0:b=2", graph 0 gets {"a": 1, "b": 2} (both rules match).
        With "0:a=1;>=0:a=2", graph 0 raises an error (conflicting values for "a").
        With "0:a=1;>=0:a=1", graph 0 gets {"a": 1} (same value is not a conflict).
    """

    def __init__(self, config_str: str) -> None:
        super().__init__(config_str, "config")

    @staticmethod
    def _parse_scalar_value(value_str: str) -> Any:
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 260-277
```python
        """Parse a string value into the appropriate Python type."""
        value_str = value_str.strip()
        if value_str.lower() == "true":
            return True
        if value_str.lower() == "false":
            return False
        if value_str.lower() == "none":
            return None
        try:
            if "." in value_str:
                return float(value_str)
            return int(value_str)
        except ValueError:
            return value_str

    def _match_rules(self, graph_id: int) -> dict[str, Any] | None:
        """Aggregate configs from all matching rules. Conflicts raise an error."""
        result: dict[str, Any] = {}
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 278-302
```python
        for f, value in self._rules:
            if graph_id in f:
                for k, v in value.items():
                    if k in result and result[k] != v:
                        raise ValueError(
                            f"Conflicting config override for graph {graph_id}: "
                            f"key '{k}' has value {result[k]!r} and {v!r}"
                        )
                    result[k] = v
        return result if result else None

    def _parse_value_str(self, value_str: str) -> dict[str, Any] | None:
        """Parse a config string like 'key1=val1,key2=val2' into a dict."""
        result: dict[str, Any] = {}
        for item in value_str.split(","):
            item = item.strip()
            if not item:
                continue
            if "=" not in item:
                log.warning("Invalid config item (missing '='): %s", item)
                continue
            key, value = item.split("=", 1)
            result[key.strip()] = self._parse_scalar_value(value)
        return result if result else None
```
- **EN**: Declares `GraphConfigRouter(_GraphRouterBase[dict[str, Any]])`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `GraphConfigRouter(_GraphRouterBase[dict[str, Any]])`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 303-326
```python
    def __repr__(self) -> str:
        if not self._rules:
            return "GraphConfigRouter(empty)"
        return f"GraphConfigRouter({self._rules})"


def _get_override_for_compile_id(
    compile_id: CompileId | None,
    config_str: str,
    create_router: Callable[[str], _GraphRouterBase[T]],
    label: str,
) -> T | None:
    """
    Get the override value for a given CompileId.

    Returns the value from the router, or None if no override applies.
    """
    if compile_id is None or not config_str:
        return None

    graph_id = compile_id.frame_id
    if graph_id is None:
        return None
```
- **EN**: This module-level block helps build, traverse, or rewrite graph structures and their metadata. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于构建、遍历或改写图结构及其元数据。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 327-346
```python
    router = create_router(config_str)
    value = router.get_value_for_graph(graph_id)
    if value is not None:
        log.info("Overriding %s: %s", label, value)
    return value


@functools.lru_cache
def _create_backend_router(config_str: str) -> GraphBackendRouter:
    """Create and cache GraphBackendRouter instances based on config string."""
    return GraphBackendRouter(config_str)


@functools.lru_cache
def _validate_backend_names(config_str: str) -> str | None:
    """Return an error message if any backend name is invalid, else None."""
    if not config_str or not config_str.strip():
        return None
    from .backends.registry import lookup_backend
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 347-370
```python
    for rule_str in config_str.split(";"):
        rule_str = rule_str.strip()
        if not rule_str or ":" not in rule_str:
            continue
        backend_name = rule_str[rule_str.find(":") + 1 :].strip()
        if not backend_name:
            continue
        try:
            lookup_backend(backend_name)
        except Exception:
            return (
                f"TORCH_COMPILE_OVERRIDE_BACKENDS: "
                f"'{backend_name}' is not a valid backend, "
                f"see `torch._dynamo.list_backends()` for available backends"
            )
    return None


@functools.lru_cache
def _validate_inductor_config_keys(config_str: str) -> str | None:
    """Return an error message if any config key is invalid, else None."""
    router = GraphConfigRouter(config_str)
    from torch._inductor import config
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 371-396
```python
    for _, config_dict in router._rules:
        for key in config_dict:
            if not hasattr(config, key):
                return (
                    f"TORCH_COMPILE_OVERRIDE_INDUCTOR_CONFIGS: "
                    f"'{key}' is not a valid torch._inductor.config option"
                )
    return None


@functools.lru_cache
def _validate_dynamo_config_keys(config_str: str) -> str | None:
    """Return an error message if any config key is invalid, else None."""
    router = GraphConfigRouter(config_str)
    from torch._dynamo import config

    for _, config_dict in router._rules:
        for key in config_dict:
            if not hasattr(config, key):
                return (
                    f"TORCH_COMPILE_OVERRIDE_DYNAMO_CONFIGS: "
                    f"'{key}' is not a valid torch._dynamo.config option"
                )
    return None
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 397-420
```python
@functools.lru_cache
def _create_inductor_config_router(config_str: str) -> GraphConfigRouter:
    """Create and cache GraphConfigRouter for inductor config overrides."""
    return GraphConfigRouter(config_str)


@functools.lru_cache
def _create_dynamo_config_router(config_str: str) -> GraphConfigRouter:
    """Create and cache GraphConfigRouter for dynamo config overrides.

    Warns that dynamo config overrides are keyed by frame ID and some configs
    can affect graph breaks, which may shift frame IDs.
    """
    router = GraphConfigRouter(config_str)
    if not router.is_empty():
        warnings.warn(
            "TORCH_COMPILE_OVERRIDE_DYNAMO_CONFIGS is set. Dynamo config overrides are "
            "keyed by frame ID. Some dynamo configs can affect graph breaks, "
            "which may alter the number of frames and shift frame IDs, causing "
            "overrides to target the wrong graphs.",
        )
    return router
```
- **EN**: These decorators register or transform the following definition so it can build, traverse, or rewrite graph structures and their metadata.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够构建、遍历或改写图结构及其元数据。

### Lines 421-444
```python
def get_backend_override_for_compile_id(
    compile_id: CompileId | None,
    config_str: str,
) -> Any:
    """
    Get the backend override for a given CompileId.

    Returns the backend function to use, or None if no override applies.
    """
    return _get_override_for_compile_id(
        compile_id,
        config_str,
        _create_backend_router,
        "torch.compile backend",
    )


def get_inductor_config_override_for_compile_id(
    compile_id: CompileId | None,
    config_str: str,
) -> dict[str, Any] | None:
    """
    Get the inductor config override for a given CompileId.
```
- **EN**: This module-level block helps hand work to a compiler/backend pipeline. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于将工作移交给编译器或后端流水线。 同时它还会计算并返回中间值或结果。

### Lines 445-471
```python
    Returns a dict of config patches to apply, or None if no override applies.
    """
    return _get_override_for_compile_id(
        compile_id,
        config_str,
        _create_inductor_config_router,  # type: ignore[arg-type]
        "inductor config",
    )


def get_dynamo_config_override_for_compile_id(
    compile_id: CompileId | None,
    config_str: str,
) -> dict[str, Any] | None:
    """
    Get the dynamo config override for a given CompileId.

    Returns a dict of config patches to apply, or None if no override applies.
    """
    return _get_override_for_compile_id(
        compile_id,
        config_str,
        _create_dynamo_config_router,  # type: ignore[arg-type]
        "dynamo config",
    )
```
- **EN**: This module-level block helps hand work to a compiler/backend pipeline. It also computes and returns an intermediate/result value.
- **CN**: 这个模块级代码块用于将工作移交给编译器或后端流水线。 同时它还会计算并返回中间值或结果。

### Lines 472-473
```python
# Keep old name for backwards compatibility
_create_router = _create_backend_router
```
- **EN**: This module-level block helps hand work to a compiler/backend pipeline.
- **CN**: 这个模块级代码块用于将工作移交给编译器或后端流水线。

## Key Concepts / 关键概念

- **Graph capture / 图捕获**
  - EN: The file participates in converting eager Python execution into compilable graph form.
  - CN: 该文件参与把即时执行的 Python 程序转换为可编译的图形式。
- **Guards / 守卫**
  - EN: Runtime predicates preserve correctness when cached compiled graphs are reused.
  - CN: 运行时谓词用于在复用缓存编译图时保持正确性。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Compiler handoff / 编译器交接**
  - EN: The implementation prepares captured work for backend compilation or execution.
  - CN: 该实现会把捕获到的工作整理后交给后端编译或执行。
- **Python execution capture / Python 执行捕获**
  - EN: The file hooks or rewrites Python execution machinery to observe user programs.
  - CN: 该文件会挂接或改写 Python 执行机制，以观察用户程序。
- **Initialization / 参数初始化**
  - EN: The file seeds tensors with statistically meaningful initial values.
  - CN: 该文件使用具有统计意义的初始值来填充张量。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch._guards`, `.backends.registry`, `torch._inductor`, `torch._dynamo`, `.eval_frame`
- **Standard library / 标准库**: `__future__`, `functools`, `logging`, `re`, `warnings`, `typing`, `collections.abc`
- **Primary symbols / 核心符号**: `GraphIdFilter`, `T`, `_GraphRouterBase`, `GraphBackendRouter`, `GraphConfigRouter`, `_get_override_for_compile_id`, `_create_backend_router`, `_validate_backend_names`, `_validate_inductor_config_keys`, `_validate_dynamo_config_keys`, `_create_inductor_config_router`, `_create_dynamo_config_router`
- **Relationship summary / 关系总结**: Dependencies center on graph capture, guard checking, bytecode analysis, and backend handoff. / 依赖关系主要围绕图捕获、守卫检查、字节码分析和后端交接展开。
