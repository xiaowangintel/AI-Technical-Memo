# test_dump_loader.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/test_dump_loader.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on debug utils dump loader in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 debug utils dump loader 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Import dependencies
```python
import sys

import polars as pl
import pytest
import torch

from sglang.srt.debug_utils.dump_loader import (
    LOAD_FAILED,
    ValueWithMeta,
    _add_duplicate_index,
    _cast_to_polars_dtype,
    find_row,
    parse_meta_from_filename,
    read_meta,
)
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 18-18: Register CI metadata
```python
register_cpu_ci(est_time=30, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 21-21: Define class TestReadMeta
```python
class TestReadMeta:
```
**EN:** This declaration introduces the `TestReadMeta` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestReadMeta` 测试类，并说明它通过继承承担的职责。

### Lines 22-31: Run test: basic
```python
    def test_basic(self, tmp_path):
        for fn in [
            "step=1___rank=0___dump_index=1___name=a.pt",
            "step=2___rank=0___dump_index=2___name=b.pt",
        ]:
            torch.save(torch.randn(5), tmp_path / fn)

        df = read_meta(str(tmp_path))
        assert len(df) == 2
        assert all(c in df.columns for c in ["step", "rank", "name"])
```
**EN:** This test method exercises basic and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 basic 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 34-34: Define class TestFindRow
```python
class TestFindRow:
```
**EN:** This declaration introduces the `TestFindRow` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFindRow` 测试类，并说明它通过继承承担的职责。

### Lines 35-37: Run test: single match
```python
    def test_single_match(self):
        df = pl.DataFrame({"id": [1, 2], "name": ["a", "b"], "file": ["f1", "f2"]})
        assert find_row(df, {"id": 2})["file"] == "f2"
```
**EN:** This test method exercises single match and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single match 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 39-41: Run test: no match
```python
    def test_no_match(self):
        df = pl.DataFrame({"id": [1, 2], "name": ["a", "b"], "file": ["f1", "f2"]})
        assert find_row(df, {"id": 999}) is None
```
**EN:** This test method exercises no match and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no match 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 43-45: Run test: ambiguous
```python
    def test_ambiguous(self):
        df = pl.DataFrame({"id": [1, 1], "file": ["f1", "f2"]})
        assert find_row(df, {"id": 1}) is None
```
**EN:** This test method exercises ambiguous and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 ambiguous 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 48-48: Define class TestCastToPolars
```python
class TestCastToPolars:
```
**EN:** This declaration introduces the `TestCastToPolars` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestCastToPolars` 测试类，并说明它通过继承承担的职责。

### Lines 49-50: Run test: int
```python
    def test_int(self):
        assert _cast_to_polars_dtype("42", pl.Int64) == 42
```
**EN:** This test method exercises int and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 int 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 52-53: Run test: float
```python
    def test_float(self):
        assert _cast_to_polars_dtype("3.14", pl.Float64) == pytest.approx(3.14)
```
**EN:** This test method exercises float and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 float 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 56-56: Define class TestAddDuplicateIndex
```python
class TestAddDuplicateIndex:
```
**EN:** This declaration introduces the `TestAddDuplicateIndex` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestAddDuplicateIndex` 测试类，并说明它通过继承承担的职责。

### Lines 57-68: Run test: basic
```python
    def test_basic(self):
        df = pl.DataFrame(
            {
                "name": ["a", "a", "b"],
                "dump_index": [1, 2, 3],
                "filename": ["f1", "f2", "f3"],
            }
        )
        result = _add_duplicate_index(df)
        assert result.filter(pl.col("name") == "a").sort("dump_index")[
            "duplicate_index"
        ].to_list() == [0, 1]
```
**EN:** This test method exercises basic and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 basic 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 71-71: Define class TestValueWithMeta
```python
class TestValueWithMeta:
```
**EN:** This declaration introduces the `TestValueWithMeta` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestValueWithMeta` 测试类，并说明它通过继承承担的职责。

### Lines 72-81: Run test: load dict format
```python
    def test_load_dict_format(self, tmp_path) -> None:
        path = tmp_path / "step=0___rank=0___dump_index=1___name=hidden.pt"
        tensor = torch.randn(4, 8)
        torch.save({"value": tensor, "meta": {"custom": "field"}}, path)

        loaded = ValueWithMeta.load(path)
        assert torch.allclose(loaded.value, tensor)
        assert loaded.meta["custom"] == "field"
        assert loaded.meta["name"] == "hidden"
        assert loaded.meta["rank"] == 0
```
**EN:** This test method exercises load dict format and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 load dict format 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 83-90: Run test: load bare tensor
```python
    def test_load_bare_tensor(self, tmp_path) -> None:
        path = tmp_path / "step=0___rank=0___dump_index=1___name=bare.pt"
        tensor = torch.randn(3, 3)
        torch.save(tensor, path)

        loaded = ValueWithMeta.load(path)
        assert torch.allclose(loaded.value, tensor)
        assert loaded.meta["name"] == "bare"
```
**EN:** This test method exercises load bare tensor and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 load bare tensor 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 92-98: Run test: load corrupted file
```python
    def test_load_corrupted_file(self, tmp_path) -> None:
        path = tmp_path / "step=0___rank=0___dump_index=1___name=bad.pt"
        path.write_text("not a valid pt file")

        loaded = ValueWithMeta.load(path)
        assert loaded.value is LOAD_FAILED
        assert loaded.meta["name"] == "bad"
```
**EN:** This test method exercises load corrupted file and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 load corrupted file 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 101-101: Define class TestRecomputeStatusParsing
```python
class TestRecomputeStatusParsing:
```
**EN:** This declaration introduces the `TestRecomputeStatusParsing` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestRecomputeStatusParsing` 测试类，并说明它通过继承承担的职责。

### Lines 102-124: Run test: parse recompute status from filename
```python
    def test_parse_recompute_status_from_filename(self) -> None:
        from pathlib import Path

        meta_disabled = parse_meta_from_filename(
            Path(
                "step=0___rank=0___dump_index=1___name=x___recompute_status=disabled.pt"
            )
        )
        assert meta_disabled["recompute_status"] == "disabled"

        meta_recompute = parse_meta_from_filename(
            Path(
                "step=0___rank=0___dump_index=1___name=x___recompute_status=recompute.pt"
            )
        )
        assert meta_recompute["recompute_status"] == "recompute"

        meta_original = parse_meta_from_filename(
            Path(
                "step=0___rank=0___dump_index=1___name=x___recompute_status=original.pt"
            )
        )
        assert meta_original["recompute_status"] == "original"
```
**EN:** This test method exercises parse recompute status from filename and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 parse recompute status from filename 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 127-128: Expose unittest entrypoint
```python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
```
**EN:** This conditional keeps the module executable as a standalone unittest target.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.debug_utils.dump_loader`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pathlib`, `polars`, `pytest`, `sys`, `torch`
- Notable symbols / 关键符号: None / 无
