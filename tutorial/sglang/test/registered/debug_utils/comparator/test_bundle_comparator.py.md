# test_bundle_comparator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/test_bundle_comparator.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on bundle comparator in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 bundle comparator 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16: Import dependencies
```python
import sys
from pathlib import Path
from unittest.mock import patch

import pytest
import torch

from sglang.srt.debug_utils.comparator.bundle_comparator import (
    _build_skip_from_one_empty_side,
    _load_all_values,
)
from sglang.srt.debug_utils.comparator.log_sink import LogSink
from sglang.srt.debug_utils.comparator.output_types import ErrorLog
from sglang.srt.debug_utils.comparator.utils import Pair
from sglang.srt.debug_utils.dump_loader import ValueWithMeta
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 18-18: Register CI metadata
```python
register_cpu_ci(est_time=15, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 21-31: Define helper: save tensor
```python
def _save_tensor(
    dump_path: Path,
    *,
    name: str,
    step: int = 0,
    rank: int = 0,
) -> str:
    filename: str = f"step={step}___rank={rank}___dump_index=0___name={name}.pt"
    tensor: torch.Tensor = torch.randn(4)
    torch.save({"value": tensor, "meta": {}}, dump_path / filename)
    return filename
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 34-34: Define class TestLoadAllValues
```python
class TestLoadAllValues:
```
**EN:** This declaration introduces the `TestLoadAllValues` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestLoadAllValues` 测试类，并说明它通过继承承担的职责。

### Lines 35-49: Run test: all success
```python
    def test_all_success(self, tmp_path: Path) -> None:
        """All files load successfully — no warnings emitted."""
        fn0: str = _save_tensor(tmp_path, name="a", rank=0)
        fn1: str = _save_tensor(tmp_path, name="a", rank=1)

        sink = LogSink()
        with sink.context() as warnings:
            with patch(
                "sglang.srt.debug_utils.comparator.bundle_comparator.log_sink",
                sink,
            ):
                result = _load_all_values(filenames=[fn0, fn1], base_path=tmp_path)

        assert len(result) == 2
        assert len(warnings) == 0
```
**EN:** This test method exercises all success and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 all success 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 51-72: Run test: one corrupted emits warning
```python
    def test_one_corrupted_emits_warning(self, tmp_path: Path) -> None:
        """One corrupted file is filtered out and emits a load_failed warning."""
        fn_good: str = _save_tensor(tmp_path, name="a", rank=0)

        fn_bad: str = "step=0___rank=1___dump_index=0___name=a.pt"
        (tmp_path / fn_bad).write_text("not a valid pt file")

        sink = LogSink()
        with sink.context() as warnings:
            with patch(
                "sglang.srt.debug_utils.comparator.bundle_comparator.log_sink",
                sink,
            ):
                result = _load_all_values(
                    filenames=[fn_good, fn_bad], base_path=tmp_path
                )

        assert len(result) == 1
        assert len(warnings) == 1
        assert isinstance(warnings[0], ErrorLog)
        assert warnings[0].category == "load_failed"
        assert fn_bad in warnings[0].message
```
**EN:** This test method exercises one corrupted emits warning and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 one corrupted emits warning 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 74-91: Run test: all corrupted emits warnings returns empty
```python
    def test_all_corrupted_emits_warnings_returns_empty(self, tmp_path: Path) -> None:
        """All files corrupted — returns empty list and emits one warning per file."""
        fn0: str = "step=0___rank=0___dump_index=0___name=a.pt"
        fn1: str = "step=0___rank=1___dump_index=0___name=a.pt"
        (tmp_path / fn0).write_text("corrupt")
        (tmp_path / fn1).write_text("corrupt")

        sink = LogSink()
        with sink.context() as warnings:
            with patch(
                "sglang.srt.debug_utils.comparator.bundle_comparator.log_sink",
                sink,
            ):
                result = _load_all_values(filenames=[fn0, fn1], base_path=tmp_path)

        assert len(result) == 0
        assert len(warnings) == 2
        assert all(w.category == "load_failed" for w in warnings)
```
**EN:** This test method exercises all corrupted emits warnings returns empty and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 all corrupted emits warnings returns empty 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 94-104: Define helper: tensor item
```python
def _tensor_item(value: torch.Tensor, rank: int = 0) -> ValueWithMeta:
    return ValueWithMeta(
        value=value,
        meta={
            "rank": rank,
            "dims": "b s",
            "sglang_parallel_info": {},
            "megatron_parallel_info": {},
            "filename": f"rank_{rank}.pt",
        },
    )
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 107-107: Define class TestBuildSkipFromOneEmptySide
```python
class TestBuildSkipFromOneEmptySide:
```
**EN:** This declaration introduces the `TestBuildSkipFromOneEmptySide` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestBuildSkipFromOneEmptySide` 测试类，并说明它通过继承承担的职责。

### Lines 108-117: Run test: baseline empty sets reason and side
```python
    def test_baseline_empty_sets_reason_and_side(self) -> None:
        """Empty baseline → reason='baseline_load_failed', available_side='target'."""
        item = _tensor_item(torch.randn(2, 3))
        record = _build_skip_from_one_empty_side(
            name="test_tensor",
            pair=Pair(x=[], y=[item]),
        )
        assert record.reason == "baseline_load_failed"
        assert record.available_side == "target"
        assert record.available_tensor_info is not None
```
**EN:** This test method exercises baseline empty sets reason and side and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 baseline empty sets reason and side 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 119-128: Run test: target empty sets reason and side
```python
    def test_target_empty_sets_reason_and_side(self) -> None:
        """Empty target → reason='target_load_failed', available_side='baseline'."""
        item = _tensor_item(torch.randn(2, 3))
        record = _build_skip_from_one_empty_side(
            name="test_tensor",
            pair=Pair(x=[item], y=[]),
        )
        assert record.reason == "target_load_failed"
        assert record.available_side == "baseline"
        assert record.available_tensor_info is not None
```
**EN:** This test method exercises target empty sets reason and side and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 target empty sets reason and side 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 130-139: Run test: no tensor items returns minimal skip
```python
    def test_no_tensor_items_returns_minimal_skip(self) -> None:
        """All items are non-tensor → skip record with no tensor info."""
        non_tensor_item = ValueWithMeta(value="not_a_tensor", meta={"rank": 0})
        record = _build_skip_from_one_empty_side(
            name="test_tensor",
            pair=Pair(x=[], y=[non_tensor_item]),
        )
        assert record.reason == "baseline_load_failed"
        assert record.available_tensor_info is None
        assert record.available_bundle_info is None
```
**EN:** This test method exercises no tensor items returns minimal skip and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no tensor items returns minimal skip 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 141-151: Run test: with tensor items populates info
```python
    def test_with_tensor_items_populates_info(self) -> None:
        """Tensor items present → tensor_info and bundle_info are populated."""
        item = _tensor_item(torch.randn(2, 3))
        record = _build_skip_from_one_empty_side(
            name="test_tensor",
            pair=Pair(x=[], y=[item]),
        )
        assert record.available_tensor_info is not None
        assert record.available_tensor_info.shape == [2, 3]
        assert record.available_bundle_info is not None
        assert record.available_bundle_info.num_files >= 1
```
**EN:** This test method exercises with tensor items populates info and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with tensor items populates info 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 153-164: Run test: multiple tensor items uses first for info
```python
    def test_multiple_tensor_items_uses_first_for_info(self) -> None:
        """When multiple tensor items exist, tensor_info comes from the first."""
        item1 = _tensor_item(torch.randn(2, 3), rank=0)
        item2 = _tensor_item(torch.randn(4, 5), rank=1)
        record = _build_skip_from_one_empty_side(
            name="multi",
            pair=Pair(x=[], y=[item1, item2]),
        )
        assert record.available_tensor_info is not None
        assert record.available_tensor_info.shape == [2, 3]
        assert record.available_bundle_info is not None
        assert record.available_bundle_info.num_files == 2
```
**EN:** This test method exercises multiple tensor items uses first for info and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multiple tensor items uses first for info 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 166-177: Run test: mixed tensor and non tensor filters non tensor
```python
    def test_mixed_tensor_and_non_tensor_filters_non_tensor(self) -> None:
        """Non-tensor items are filtered; tensor_info comes from tensor items only."""
        non_tensor = ValueWithMeta(value="string_value", meta={"rank": 0})
        tensor_item = _tensor_item(torch.randn(5, 6), rank=1)
        record = _build_skip_from_one_empty_side(
            name="mixed",
            pair=Pair(x=[], y=[non_tensor, tensor_item]),
        )
        assert record.available_tensor_info is not None
        assert record.available_tensor_info.shape == [5, 6]
        assert record.available_bundle_info is not None
        assert record.available_bundle_info.num_files == 1
```
**EN:** This test method exercises mixed tensor and non tensor filters non tensor and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 mixed tensor and non tensor filters non tensor 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 179-187: Run test: tensor info includes sample
```python
    def test_tensor_info_includes_sample(self) -> None:
        """Tensor info should include a sample string for skip records."""
        item = _tensor_item(torch.tensor([1.0, 2.0, 3.0]))
        record = _build_skip_from_one_empty_side(
            name="sample_check",
            pair=Pair(x=[item], y=[]),
        )
        assert record.available_tensor_info is not None
        assert record.available_tensor_info.sample is not None
```
**EN:** This test method exercises tensor info includes sample and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 tensor info includes sample 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 189-196: Run test: name preserved in record
```python
    def test_name_preserved_in_record(self) -> None:
        """The tensor name is preserved in the skip record."""
        item = _tensor_item(torch.randn(2, 3))
        record = _build_skip_from_one_empty_side(
            name="my_layer.weight",
            pair=Pair(x=[], y=[item]),
        )
        assert record.name == "my_layer.weight"
```
**EN:** This test method exercises name preserved in record and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 name preserved in record 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 198-206: Run test: bundle info has dims from meta
```python
    def test_bundle_info_has_dims_from_meta(self) -> None:
        """Bundle info dims field should come from the meta."""
        item = _tensor_item(torch.randn(2, 3))
        record = _build_skip_from_one_empty_side(
            name="dims_check",
            pair=Pair(x=[], y=[item]),
        )
        assert record.available_bundle_info is not None
        assert record.available_bundle_info.dims == "b s"
```
**EN:** This test method exercises bundle info has dims from meta and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 bundle info has dims from meta 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 209-210: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.bundle_comparator`, `sglang.srt.debug_utils.comparator.log_sink`, `sglang.srt.debug_utils.comparator.output_types`, `sglang.srt.debug_utils.comparator.utils`, `sglang.srt.debug_utils.dump_loader`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pathlib`, `pytest`, `sys`, `torch`, `unittest.mock`
- Notable symbols / 关键符号: None / 无
