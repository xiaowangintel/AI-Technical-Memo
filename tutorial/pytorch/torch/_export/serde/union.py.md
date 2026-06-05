# union.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_export/serde/union.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements serialization and deserialization helpers for export artifacts and their structured metadata.
- **Purpose (CN)**: 实现导出产物及其结构化元数据的序列化与反序列化辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-15 / 第 1-15 行

````python
0001: # mypy: allow-untyped-defs
0002: import functools
0003: from collections.abc import Hashable
0004: from dataclasses import dataclass, fields
0005: from typing import TypeVar
0006: from typing_extensions import dataclass_transform
0007: 
0008: 
0009: T = TypeVar("T", bound="_Union")
0010: 
0011: 
0012: class _UnionTag(str):
0013:     __slots__ = ("_cls",)
0014:     _cls: Hashable
0015: 
````

- **L1** EN: Carries a file-level or type-checking directive. | CN: 携带文件级或类型检查相关指令。
- **L2** EN: Imports module dependencies: `functools`. | CN: 导入模块依赖：`functools`。
- **L3** EN: Imports `Hashable` from `collections.abc` so later code can reuse those definitions. | CN: 从 `collections.abc` 导入 `Hashable`，供后续代码复用这些定义。
- **L4** EN: Imports `dataclass, fields` from `dataclasses` so later code can reuse those definitions. | CN: 从 `dataclasses` 导入 `dataclass, fields`，供后续代码复用这些定义。
- **L5** EN: Imports `TypeVar` from `typing` so later code can reuse those definitions. | CN: 从 `typing` 导入 `TypeVar`，供后续代码复用这些定义。
- **L6** EN: Imports `dataclass_transform` from `typing_extensions` so later code can reuse those definitions. | CN: 从 `typing_extensions` 导入 `dataclass_transform`，供后续代码复用这些定义。
- **L7** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L8** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L9** EN: Assigns module-level configuration or cached state to `T`. | CN: 为 `T` 赋予模块级配置或缓存状态。
- **L10** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L11** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L12** EN: Defines class `_UnionTag` with bases `str`, which collects related state and methods into a named abstraction for this module. | CN: 定义类 `_UnionTag`，其基类为 `str`，作用是把相关状态与方法组织成该模块中的命名抽象。
- **L13** EN: Assigns module-level configuration or cached state to `__slots__`. | CN: 为 `__slots__` 赋予模块级配置或缓存状态。
- **L14** EN: Continues class `_UnionTag`, refining its attributes, helpers, or nested definitions. | CN: 继续类 `_UnionTag` 的实现，细化其属性、辅助逻辑或嵌套定义。
- **L15** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 16-37 / 第 16-37 行

````python
0016:     @staticmethod
0017:     def create(t, cls):
0018:         tag = _UnionTag(t)
0019:         if hasattr(tag, "_cls"):
0020:             raise AssertionError("tag already has _cls attribute")
0021:         tag._cls = cls
0022:         return tag
0023: 
0024:     def __eq__(self, cmp) -> bool:
0025:         if not isinstance(cmp, str):
0026:             raise AssertionError(f"expected str, got {type(cmp)}")
0027:         other = str(cmp)
0028:         if other not in _get_field_names(self._cls):
0029:             raise AssertionError(
0030:                 f"{other} is not a valid tag for {self._cls}. Available tags: {_get_field_names(self._cls)}"
0031:             )
0032:         return str(self) == other
0033: 
0034:     def __hash__(self):
0035:         return hash(str(self))
0036: 
0037: 
````

- **L16** EN: Applies decorator `staticmethod`, which modifies the behavior of the following definition. | CN: 应用装饰器 `staticmethod`，其作用是修改后续定义的行为。
- **L17** EN: Defines function `create`, which implements part of the export pipeline or its metadata handling. | CN: 定义函数 `create`，其作用是实现导出流水线或其元数据处理的一部分。
- **L18** EN: Assigns or updates `tag`. | CN: 对 `tag` 进行赋值或更新。
- **L19** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L20** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L21** EN: Assigns or updates `tag._cls`. | CN: 对 `tag._cls` 进行赋值或更新。
- **L22** EN: Returns from `_UnionTag.create` with the computed result or updated state. | CN: 从 `_UnionTag.create` 返回计算结果或更新后的状态。
- **L23** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L24** EN: Defines function `__eq__`, which implements part of the export pipeline or its metadata handling. | CN: 定义函数 `__eq__`，其作用是实现导出流水线或其元数据处理的一部分。
- **L25** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L26** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L27** EN: Assigns or updates `other`. | CN: 对 `other` 进行赋值或更新。
- **L28** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L29** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L30** EN: Invokes `_get_field_names` to advance the surrounding implementation. | CN: 调用 `_get_field_names` 来推进周围的实现逻辑。
- **L31** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L32** EN: Returns from `_UnionTag.__eq__` with the computed result or updated state. | CN: 从 `_UnionTag.__eq__` 返回计算结果或更新后的状态。
- **L33** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L34** EN: Defines function `__hash__`, which implements part of the export pipeline or its metadata handling. | CN: 定义函数 `__hash__`，其作用是实现导出流水线或其元数据处理的一部分。
- **L35** EN: Returns from `_UnionTag.__hash__` with the computed result or updated state. | CN: 从 `_UnionTag.__hash__` 返回计算结果或更新后的状态。
- **L36** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L37** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 38-58 / 第 38-58 行

````python
0038: @functools.cache
0039: def _get_field_names(cls) -> set[str]:
0040:     return {f.name for f in fields(cls)}
0041: 
0042: 
0043: # If you turn a schema class that inherits from union into a dataclass, please use
0044: # this decorator to configure it. It's safe, faster and allows code sharing.
0045: #
0046: # For example, _union_dataclass customizes the __eq__ method to only check the type
0047: # and value property instead of default implementation of dataclass which goes
0048: # through every field in the dataclass.
0049: @dataclass_transform(eq_default=False)
0050: def _union_dataclass(cls: type[T]) -> type[T]:
0051:     if not issubclass(cls, _Union):
0052:         raise AssertionError(f"{cls} must inherit from {_Union}.")
0053:     return dataclass(repr=False, eq=False)(cls)
0054: 
0055: 
0056: class _Union:
0057:     _type: _UnionTag
0058: 
````

- **L38** EN: Applies decorator `functools.cache`, which modifies the behavior of the following definition. | CN: 应用装饰器 `functools.cache`，其作用是修改后续定义的行为。
- **L39** EN: Defines function `_get_field_names`, which implements part of the export pipeline or its metadata handling. | CN: 定义函数 `_get_field_names`，其作用是实现导出流水线或其元数据处理的一部分。
- **L40** EN: Returns from `_get_field_names` with the computed result or updated state. | CN: 从 `_get_field_names` 返回计算结果或更新后的状态。
- **L41** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L42** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L43** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L44** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L45** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L46** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L47** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L48** EN: Adds a comment that documents intent, assumptions, or caveats. | CN: 添加注释，用于说明意图、假设或注意事项。
- **L49** EN: Applies decorator `dataclass_transform`, which modifies the behavior of the following definition. | CN: 应用装饰器 `dataclass_transform`，其作用是修改后续定义的行为。
- **L50** EN: Defines function `_union_dataclass`, which implements part of the export pipeline or its metadata handling. | CN: 定义函数 `_union_dataclass`，其作用是实现导出流水线或其元数据处理的一部分。
- **L51** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L52** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L53** EN: Returns from `_union_dataclass` with the computed result or updated state. | CN: 从 `_union_dataclass` 返回计算结果或更新后的状态。
- **L54** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L55** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L56** EN: Defines class `_Union`, which collects related state and methods into a named abstraction for this module. | CN: 定义类 `_Union`，其作用是把相关状态与方法组织成该模块中的命名抽象。
- **L57** EN: Continues class `_Union`, refining its attributes, helpers, or nested definitions. | CN: 继续类 `_Union` 的实现，细化其属性、辅助逻辑或嵌套定义。
- **L58** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 59-75 / 第 59-75 行

````python
0059:     @classmethod
0060:     def create(cls, **kwargs):
0061:         if len(kwargs) != 1:
0062:             raise AssertionError(f"expected exactly 1 kwarg, got {len(kwargs)}")
0063:         obj = cls(**{**{f.name: None for f in fields(cls)}, **kwargs})  # type: ignore[arg-type]
0064:         obj._type = _UnionTag.create(next(iter(kwargs.keys())), cls)
0065:         return obj
0066: 
0067:     def __post_init__(self):
0068:         if any(
0069:             f.name in ("type", "_type", "create", "value")
0070:             for f in fields(self)  # type: ignore[arg-type, misc]
0071:         ):
0072:             raise AssertionError(
0073:                 "field names 'type', '_type', 'create', 'value' are reserved"
0074:             )
0075: 
````

- **L59** EN: Applies decorator `classmethod`, which modifies the behavior of the following definition. | CN: 应用装饰器 `classmethod`，其作用是修改后续定义的行为。
- **L60** EN: Defines function `create`, which implements part of the export pipeline or its metadata handling. | CN: 定义函数 `create`，其作用是实现导出流水线或其元数据处理的一部分。
- **L61** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L62** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L63** EN: Assigns or updates `obj`. | CN: 对 `obj` 进行赋值或更新。
- **L64** EN: Assigns or updates `obj._type`. | CN: 对 `obj._type` 进行赋值或更新。
- **L65** EN: Returns from `_Union.create` with the computed result or updated state. | CN: 从 `_Union.create` 返回计算结果或更新后的状态。
- **L66** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L67** EN: Defines function `__post_init__`, which implements part of the export pipeline or its metadata handling. | CN: 定义函数 `__post_init__`，其作用是实现导出流水线或其元数据处理的一部分。
- **L68** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L69** EN: Invokes `in` to advance the surrounding implementation. | CN: 调用 `in` 来推进周围的实现逻辑。
- **L70** EN: Starts an iteration that applies the same rule across elements or steps. | CN: 开始一个迭代过程，把同一规则应用到多个元素或步骤上。
- **L71** EN: Continues `_Union.__post_init__`, which implements part of the export pipeline or its metadata handling. | CN: 继续 `_Union.__post_init__` 的实现，其作用是实现导出流水线或其元数据处理的一部分。
- **L72** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L73** EN: Continues `_Union.__post_init__`, which implements part of the export pipeline or its metadata handling. | CN: 继续 `_Union.__post_init__` 的实现，其作用是实现导出流水线或其元数据处理的一部分。
- **L74** EN: Closes the surrounding multi-line expression, container, or argument list. | CN: 结束外围的多行表达式、容器或参数列表。
- **L75** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 76-94 / 第 76-94 行

````python
0076:     @property
0077:     def type(self) -> str:
0078:         try:
0079:             return self._type
0080:         except AttributeError as e:
0081:             raise RuntimeError(
0082:                 f"Please use {type(self).__name__}.create to instantiate the union type."
0083:             ) from e
0084: 
0085:     @property
0086:     def value(self):
0087:         return getattr(self, self.type)
0088: 
0089:     def __getattribute__(self, name):
0090:         attr = super().__getattribute__(name)
0091:         if attr is None and name in _get_field_names(type(self)) and name != self.type:  # type: ignore[arg-type]
0092:             raise AttributeError(f"Field {name} is not set.")
0093:         return attr
0094: 
````

- **L76** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L77** EN: Defines function `type`, which implements part of the export pipeline or its metadata handling. | CN: 定义函数 `type`，其作用是实现导出流水线或其元数据处理的一部分。
- **L78** EN: Begins a guarded region that can recover from runtime failures. | CN: 开始一个可从运行时失败中恢复的受保护区域。
- **L79** EN: Returns from `_Union.type` with the computed result or updated state. | CN: 从 `_Union.type` 返回计算结果或更新后的状态。
- **L80** EN: Handles an exception path for a specific failure mode. | CN: 处理某种特定失败模式对应的异常路径。
- **L81** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L82** EN: Invokes `type` to advance the surrounding implementation. | CN: 调用 `type` 来推进周围的实现逻辑。
- **L83** EN: Continues `_Union.type`, which implements part of the export pipeline or its metadata handling. | CN: 继续 `_Union.type` 的实现，其作用是实现导出流水线或其元数据处理的一部分。
- **L84** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L85** EN: Applies decorator `property`, which exposes a computed attribute through Python property access. | CN: 应用装饰器 `property`，其作用是通过 Python 属性访问暴露一个计算得到的属性。
- **L86** EN: Defines function `value`, which implements part of the export pipeline or its metadata handling. | CN: 定义函数 `value`，其作用是实现导出流水线或其元数据处理的一部分。
- **L87** EN: Returns from `_Union.value` with the computed result or updated state. | CN: 从 `_Union.value` 返回计算结果或更新后的状态。
- **L88** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L89** EN: Defines function `__getattribute__`, which implements part of the export pipeline or its metadata handling. | CN: 定义函数 `__getattribute__`，其作用是实现导出流水线或其元数据处理的一部分。
- **L90** EN: Assigns or updates `attr`. | CN: 对 `attr` 进行赋值或更新。
- **L91** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L92** EN: Raises an exception to reject an invalid or unsupported path. | CN: 抛出异常，以拒绝非法或不受支持的路径。
- **L93** EN: Returns from `_Union.__getattribute__` with the computed result or updated state. | CN: 从 `_Union.__getattribute__` 返回计算结果或更新后的状态。
- **L94** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。

### Lines 95-104 / 第 95-104 行

````python
0095:     def __eq__(self, other: object) -> bool:
0096:         if not isinstance(other, _Union):
0097:             return False
0098:         return self.type == other.type and self.value == other.value
0099: 
0100:     def __str__(self):
0101:         return self.__repr__()
0102: 
0103:     def __repr__(self):
0104:         return f"{type(self).__name__}({self.type}={getattr(self, self.type)})"
````

- **L95** EN: Defines function `__eq__`, which implements part of the export pipeline or its metadata handling. | CN: 定义函数 `__eq__`，其作用是实现导出流水线或其元数据处理的一部分。
- **L96** EN: Starts or refines a conditional branch that guards a special case or invariant. | CN: 开始或细化一个条件分支，用于保护特殊情况或不变量。
- **L97** EN: Returns from `_Union.__eq__` with the computed result or updated state. | CN: 从 `_Union.__eq__` 返回计算结果或更新后的状态。
- **L98** EN: Returns from `_Union.__eq__` with the computed result or updated state. | CN: 从 `_Union.__eq__` 返回计算结果或更新后的状态。
- **L99** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L100** EN: Defines function `__str__`, which implements part of the export pipeline or its metadata handling. | CN: 定义函数 `__str__`，其作用是实现导出流水线或其元数据处理的一部分。
- **L101** EN: Returns from `_Union.__str__` with the computed result or updated state. | CN: 从 `_Union.__str__` 返回计算结果或更新后的状态。
- **L102** EN: Blank line separating nearby logical blocks. | CN: 空行，用于分隔相邻的逻辑代码块。
- **L103** EN: Defines function `__repr__`, which implements part of the export pipeline or its metadata handling. | CN: 定义函数 `__repr__`，其作用是实现导出流水线或其元数据处理的一部分。
- **L104** EN: Returns from `_Union.__repr__` with the computed result or updated state. | CN: 从 `_Union.__repr__` 返回计算结果或更新后的状态。

## Key Concepts / 关键概念

- **EN**: Program export — The file contributes to capturing eager programs into stable graph-based export artifacts.
  **CN**: Program export——该文件有助于把 eager 程序捕获为稳定的基于图的导出产物。
- **EN**: Graph normalization — Export logic often rewrites or annotates graphs so downstream backends see a cleaner contract.
  **CN**: Graph normalization——导出逻辑通常会改写或标注图，以便下游后端看到更清晰的契约。
- **EN**: Metadata and constraints — Structured metadata, symbolic shapes, and validation rules are central to reliable export.
  **CN**: Metadata and constraints——结构化元数据、符号形状与校验规则是可靠导出的核心。
- **EN**: Transforms — The file applies mathematical or graph-level transforms to values or programs.
  **CN**: Transforms——该文件对数值或程序施加数学变换或图级变换。
- **EN**: Primary type `_UnionTag` — the file exposes `_UnionTag` as a central abstraction or implementation unit.
  **CN**: 核心类型 `_UnionTag`——该文件把 `_UnionTag` 作为重要抽象或实现单元。
- **EN**: Primary type `_Union` — the file exposes `_Union` as a central abstraction or implementation unit.
  **CN**: 核心类型 `_Union`——该文件把 `_Union` 作为重要抽象或实现单元。

## Dependencies / 依赖关系

- **Torch imports / Torch 导入**: 无
- **Other imports / 其他导入**: `functools`、`collections.abc:Hashable`、`dataclasses:dataclass, fields`、`typing:TypeVar`、`typing_extensions:dataclass_transform`
- **Top-level classes / 顶层类**: `_UnionTag`、`_Union`
- **Top-level functions / 顶层函数**: `_get_field_names`、`_union_dataclass`
- **Base classes / 基类**: `str`
- **Decorators / 装饰器**: `functools.cache`、`dataclass_transform`
- **Module assignments / 模块级赋值**: `T`
