# dump_loader.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/dump_loader.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on shared debugging utilities. It mainly implements the core logic needed by this part of the debug toolchain. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于共享调试工具。它主要用于实现该调试工具链所需的核心逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Import dependencies and shared types / 导入依赖与共享类型
```python
import functools
import os
from dataclasses import dataclass
from pathlib import Path
from typing import Any, Callable, Dict, Optional, Tuple

import polars as pl
import torch
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 10-10: Declare module-level symbols such as `LOAD_FAILED` / 声明模块级符号，例如 `LOAD_FAILED`
```python
LOAD_FAILED: object = object()
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

### Lines 13-23: Implement function `parse_meta_from_filename` / 实现函数 `parse_meta_from_filename`
```python
def parse_meta_from_filename(path: Path) -> Dict[str, Any]:
    stem = Path(path).stem
    result: Dict[str, Any] = {}
    for kv in stem.split("___"):
        if "=" in kv:
            k, v = kv.split("=", 1)
            result[k] = v
    for field_name, converter in _TYPED_FIELDS:
        if field_name in result:
            result[field_name] = converter(result[field_name])
    return result
```
**EN:** Function `parse_meta_from_filename` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `parse_meta_from_filename` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 27-27: Define class `ValueWithMeta` and class context / 定义类 `ValueWithMeta`及类上下文
```python
class ValueWithMeta:
```
**EN:** This section introduces `ValueWithMeta`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `ValueWithMeta`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 28-29: Declare fields for `ValueWithMeta` such as `value`, `meta` / 为 `ValueWithMeta` 声明字段，例如 `value`, `meta`
```python
    value: Any
    meta: Dict[str, Any]
```
**EN:** These lines declare the state carried by `ValueWithMeta`. The annotated fields show what data instances expose to the surrounding debugging pipeline.
**CN:** 这些代码声明了 `ValueWithMeta` 持有的状态。带注解的字段展示了实例会向周边调试流水线暴露哪些数据。

### Lines 32-48: Implement method `load` for `ValueWithMeta` / 为 `ValueWithMeta` 实现方法 `load`
```python
    def load(path: Path) -> "ValueWithMeta":
        path = Path(path)
        meta_from_filename = parse_meta_from_filename(path)

        try:
            raw = torch.load(path, weights_only=False, map_location="cpu")
        except Exception as e:
            print(f"Skip load {path} since error {e}")
            return ValueWithMeta(
                value=LOAD_FAILED, meta={**meta_from_filename, "filename": path.name}
            )

        value, meta_from_embedded = _unwrap_dict_format(raw)
        return ValueWithMeta(
            value=value,
            meta={**meta_from_filename, **meta_from_embedded, "filename": path.name},
        )
```
**EN:** Method `load` implements behavior on `ValueWithMeta`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `load` 为 `ValueWithMeta` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 51-56: Implement helper `_unwrap_dict_format` / 实现辅助函数 `_unwrap_dict_format`
```python
def _unwrap_dict_format(obj: Any) -> Tuple[Any, Dict[str, Any]]:
    if isinstance(obj, dict) and "value" in obj:
        meta = obj.get("meta", {})
        assert isinstance(meta, dict), f"Expected meta to be dict, got {type(meta)}"
        return obj["value"], meta
    return obj, {}
```
**EN:** Function `_unwrap_dict_format` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_unwrap_dict_format` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 59-59: Define class `DumpLoader` and class context / 定义类 `DumpLoader`及类上下文
```python
class DumpLoader:
```
**EN:** This section introduces `DumpLoader`, including its inheritance and any class-level context, so later fields or methods have a clear abstraction boundary.
**CN:** 这一部分引入了 `DumpLoader`，包括其继承关系和类级上下文，从而为后续字段或方法建立清晰的抽象边界。

### Lines 60-66: Implement method `__init__` for `DumpLoader` / 为 `DumpLoader` 实现方法 `__init__`
```python
    def __init__(self):
        directory = os.environ.get("SGLANG_DUMP_LOADER_DIR")

        self._enable = directory is not None
        if self._enable:
            self._directory = Path(directory)
            self._df = read_meta(directory)
```
**EN:** Method `__init__` implements behavior on `DumpLoader`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `__init__` 为 `DumpLoader` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 69-70: Implement method `enable` for `DumpLoader` / 为 `DumpLoader` 实现方法 `enable`
```python
    def enable(self):
        return self._enable
```
**EN:** Method `enable` implements behavior on `DumpLoader`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `enable` 为 `DumpLoader` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 72-92: Implement method `load` for `DumpLoader` / 为 `DumpLoader` 实现方法 `load`
```python
    def load(self, name, **kwargs):
        assert self._enable, "Please call DumpLoader.load only when it is enabled"

        from sglang.srt.debug_utils.dumper import dumper

        step = dumper._state.step
        conditions = dict(name=name, step=step, **kwargs)
        row = find_row(self._df, conditions=conditions)
        assert (
            row is not None
        ), f"DumpLoader cannot find row given query {name=} {kwargs=} {self._directory=}"

        path = self._directory / row["filename"]
        output = torch.load(path, weights_only=False)
        if isinstance(output, dict) and "value" in output:
            output = output["value"]

        print(
            f"[DumpLoader] load from {path=} (query: {name=} {kwargs=}, output: {type(output)})"
        )
        return output
```
**EN:** Method `load` implements behavior on `DumpLoader`. It turns the class state into a focused operation that other parts of the toolchain can call.
**CN:** 方法 `load` 为 `DumpLoader` 提供具体行为，把类中的状态转换为工具链其他部分可调用的定向操作。

### Lines 95-120: Implement function `read_meta` / 实现函数 `read_meta`
```python
def read_meta(directory):
    directory = Path(directory)
    assert directory.is_dir(), f"{directory=} should be a directory"

    rows = []
    for p in directory.glob("*.pt"):
        try:
            full_kwargs = parse_meta_from_filename(p)
            rows.append(
                {
                    "filename": str(p.name),
                    **full_kwargs,
                }
            )
        except Exception as e:
            print(f"[DumpLoader] skip loading {p} due to error {e}")

    df = pl.DataFrame(rows)
    df = df.with_columns(
        pl.col("step").cast(int),
        pl.col("rank").cast(int),
        pl.col("dump_index").cast(int),
    )
    df = _add_duplicate_index(df)
    df = df.sort("rank", "dump_index")
    return df
```
**EN:** Function `read_meta` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `read_meta` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 123-129: Implement helper `_add_duplicate_index` / 实现辅助函数 `_add_duplicate_index`
```python
def _add_duplicate_index(df: pl.DataFrame) -> pl.DataFrame:
    group_cols = [c for c in df.columns if c not in ["filename", "dump_index"]]
    df = df.sort(group_cols + ["dump_index"])
    df = df.with_columns(
        pl.cum_count("dump_index").over(group_cols).sub(1).alias("duplicate_index")
    )
    return df
```
**EN:** Function `_add_duplicate_index` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_add_duplicate_index` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 132-144: Implement function `filter_rows` / 实现函数 `filter_rows`
```python
def filter_rows(df: pl.DataFrame, conditions: Dict[str, Any]) -> list[dict]:
    filter_exprs = [
        (
            pl.col(col) == _cast_to_polars_dtype(conditions[col], df.schema[col])
            if conditions[col] is not None
            else pl.col(col).is_null()
        )
        for col in conditions
        if col in df.columns
    ]
    if not filter_exprs:
        return []
    return df.filter(functools.reduce(lambda a, b: a & b, filter_exprs)).to_dicts()
```
**EN:** Function `filter_rows` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `filter_rows` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 147-152: Implement function `find_row` / 实现函数 `find_row`
```python
def find_row(df: pl.DataFrame, conditions: Dict[str, Any]):
    rows = filter_rows(df, conditions)
    if len(rows) > 1:
        print(f"find_row find ambiguous results: {rows=}")
        return None
    return rows[0] if rows else None
```
**EN:** Function `find_row` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `find_row` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 155-165: Implement helper `_cast_to_polars_dtype` / 实现辅助函数 `_cast_to_polars_dtype`
```python
def _cast_to_polars_dtype(value, target_dtype):
    if target_dtype in (pl.Int64, pl.Int32, pl.UInt64, pl.UInt32):
        return int(value)
    elif target_dtype in (pl.Float64, pl.Float32):
        return float(value)
    elif target_dtype == pl.Boolean:
        return bool(value)
    elif target_dtype == pl.String:
        return str(value)
    else:
        return value
```
**EN:** Function `_cast_to_polars_dtype` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_cast_to_polars_dtype` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 168-175: Implement function `read_tokenizer_path` / 实现函数 `read_tokenizer_path`
```python
def read_tokenizer_path(directory: Path) -> Optional[str]:
    """Read tokenizer_path from any .pt file's embedded metadata in a dump directory."""
    for p in directory.glob("*.pt"):
        item: ValueWithMeta = ValueWithMeta.load(p)
        tokenizer_path: Optional[str] = item.meta.get("tokenizer_path")
        if tokenizer_path is not None:
            return str(tokenizer_path)
    return None
```
**EN:** Function `read_tokenizer_path` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `read_tokenizer_path` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 178-183: Declare module-level symbols such as `_TYPED_FIELDS`, `dump_loader` / 声明模块级符号，例如 `_TYPED_FIELDS`, `dump_loader`
```python
_TYPED_FIELDS: list[tuple[str, Callable[[str], Any]]] = [
    ("rank", int),
]


dump_loader = DumpLoader()
```
**EN:** This block defines reusable module-level symbols such as constants, aliases, or configuration values that simplify the later control flow.
**CN:** 这一段定义了可复用的模块级符号，例如常量、别名或配置值，从而简化后续控制流程。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `ValueWithMeta`, `DumpLoader`, `parse_meta_from_filename`, `_unwrap_dict_format`, `read_meta`, `_add_duplicate_index`, `filter_rows`, `find_row`, `_cast_to_polars_dtype`, `read_tokenizer_path`
- **Module role / 模块角色**: Shared debugging utilities / 共享调试工具
- **Implementation focus / 实现重点**: Implements the core logic needed by this part of the debug toolchain / 实现该调试工具链所需的核心逻辑

## Dependencies / 依赖关系
- **Standard library / 标准库**: `functools`, `os`, `dataclasses`, `pathlib`, `typing`
- **Third-party / 第三方**: `polars`, `torch`
- **Internal / 内部**: `sglang.srt.debug_utils.dumper`
