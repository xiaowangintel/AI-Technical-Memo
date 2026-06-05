# test_dp_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/test_dp_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on comparator dp utils in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 comparator dp utils 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: Import dependencies
```python
import sys

import pytest
import torch

from sglang.srt.debug_utils.comparator.dims_spec import ParallelAxis
from sglang.srt.debug_utils.comparator.dp_utils import (
    _extract_dp_info,
    _group_has_data,
    filter_to_non_empty_dp_rank,
)
from sglang.srt.debug_utils.dump_loader import ValueWithMeta
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 15-15: Register CI metadata
```python
register_cpu_ci(est_time=15, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 18-28: Define helper: make sglang meta
```python
def _make_sglang_meta(
    *, tp_rank: int = 0, tp_size: int = 1, dp_rank: int = 0, dp_size: int = 1
) -> dict:
    return {
        "sglang_parallel_info": {
            "tp_rank": tp_rank,
            "tp_size": tp_size,
            "dp_rank": dp_rank,
            "dp_size": dp_size,
        }
    }
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 31-41: Define helper: make megatron meta
```python
def _make_megatron_meta(
    *, tp_rank: int = 0, tp_size: int = 1, dp_rank: int = 0, dp_size: int = 1
) -> dict:
    return {
        "megatron_parallel_info": {
            "tp_rank": tp_rank,
            "tp_size": tp_size,
            "dp_rank": dp_rank,
            "dp_size": dp_size,
        }
    }
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 44-45: Define helper: make item
```python
def _make_item(value: object, meta: dict) -> ValueWithMeta:
    return ValueWithMeta(value=value, meta=meta)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 53-53: Define class TestExtractDpInfo
```python
class TestExtractDpInfo:
```
**EN:** This declaration introduces the `TestExtractDpInfo` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestExtractDpInfo` 测试类，并说明它通过继承承担的职责。

### Lines 54-56: Run test: sglang dp
```python
    def test_sglang_dp(self) -> None:
        meta: dict = _make_sglang_meta(dp_rank=1, dp_size=4)
        assert _extract_dp_info(meta, dp_axis=ParallelAxis.DP) == (1, 4)
```
**EN:** This test method exercises sglang dp and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 sglang dp 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 58-60: Run test: megatron dp
```python
    def test_megatron_dp(self) -> None:
        meta: dict = _make_megatron_meta(dp_rank=2, dp_size=8)
        assert _extract_dp_info(meta, dp_axis=ParallelAxis.DP) == (2, 8)
```
**EN:** This test method exercises megatron dp and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 megatron dp 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 62-63: Run test: no parallel info
```python
    def test_no_parallel_info(self) -> None:
        assert _extract_dp_info({}, dp_axis=ParallelAxis.DP) is None
```
**EN:** This test method exercises no parallel info and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no parallel info 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 65-67: Run test: no dp fields
```python
    def test_no_dp_fields(self) -> None:
        meta: dict = {"sglang_parallel_info": {"tp_rank": 0, "tp_size": 2}}
        assert _extract_dp_info(meta, dp_axis=ParallelAxis.DP) is None
```
**EN:** This test method exercises no dp fields and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no dp fields 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 75-75: Define class TestGroupHasData
```python
class TestGroupHasData:
```
**EN:** This declaration introduces the `TestGroupHasData` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestGroupHasData` 测试类，并说明它通过继承承担的职责。

### Lines 76-78: Run test: non empty tensor
```python
    def test_non_empty_tensor(self) -> None:
        item: ValueWithMeta = _make_item(value=torch.tensor([1, 2, 3]), meta={})
        assert _group_has_data([item]) is True
```
**EN:** This test method exercises non empty tensor and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 non empty tensor 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 80-82: Run test: empty tensor
```python
    def test_empty_tensor(self) -> None:
        item: ValueWithMeta = _make_item(value=torch.tensor([]), meta={})
        assert _group_has_data([item]) is False
```
**EN:** This test method exercises empty tensor and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty tensor 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 84-86: Run test: non tensor value
```python
    def test_non_tensor_value(self) -> None:
        item: ValueWithMeta = _make_item(value="hello", meta={})
        assert _group_has_data([item]) is False
```
**EN:** This test method exercises non tensor value and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 non tensor value 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 88-89: Run test: empty group
```python
    def test_empty_group(self) -> None:
        assert _group_has_data([]) is False
```
**EN:** This test method exercises empty group and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty group 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 97-97: Define class TestFilterToNonEmptyDpRank
```python
class TestFilterToNonEmptyDpRank:
```
**EN:** This declaration introduces the `TestFilterToNonEmptyDpRank` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFilterToNonEmptyDpRank` 测试类，并说明它通过继承承担的职责。

### Lines 98-108: Run test: dp size 1 returns unchanged
```python
    def test_dp_size_1_returns_unchanged(self) -> None:
        items: list[ValueWithMeta] = [
            _make_item(
                value=torch.tensor([1.0]),
                meta=_make_sglang_meta(dp_size=1),
            ),
        ]
        result: list[ValueWithMeta] = filter_to_non_empty_dp_rank(
            items, dp_axis=ParallelAxis.DP
        )
        assert result is items
```
**EN:** This test method exercises dp size 1 returns unchanged and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dp size 1 returns unchanged 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 110-117: Run test: no parallel info returns unchanged
```python
    def test_no_parallel_info_returns_unchanged(self) -> None:
        items: list[ValueWithMeta] = [
            _make_item(value=torch.tensor([1.0]), meta={}),
        ]
        result: list[ValueWithMeta] = filter_to_non_empty_dp_rank(
            items, dp_axis=ParallelAxis.DP
        )
        assert result is items
```
**EN:** This test method exercises no parallel info returns unchanged and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no parallel info returns unchanged 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 119-123: Run test: empty list returns empty
```python
    def test_empty_list_returns_empty(self) -> None:
        result: list[ValueWithMeta] = filter_to_non_empty_dp_rank(
            [], dp_axis=ParallelAxis.DP
        )
        assert result == []
```
**EN:** This test method exercises empty list returns empty and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty list returns empty 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 125-142: Run test: dp2 all non tensor returns unchanged
```python
    def test_dp2_all_non_tensor_returns_unchanged(self) -> None:
        """DP=2 with non-tensor values: skip filtering, return unchanged."""
        items: list[ValueWithMeta] = [
            _make_item(
                value=["req_A"],
                meta=_make_sglang_meta(dp_rank=0, dp_size=2),
            ),
            _make_item(
                value=["req_A"],
                meta=_make_sglang_meta(dp_rank=1, dp_size=2),
            ),
        ]

        result: list[ValueWithMeta] = filter_to_non_empty_dp_rank(
            items, dp_axis=ParallelAxis.DP
        )

        assert result is items
```
**EN:** This test method exercises dp2 all non tensor returns unchanged and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dp2 all non tensor returns unchanged 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 144-162: Run test: dp2 one empty one nonempty sglang
```python
    def test_dp2_one_empty_one_nonempty_sglang(self) -> None:
        """DP=2, rank 0 has data, rank 1 has empty tensor."""
        items: list[ValueWithMeta] = [
            _make_item(
                value=torch.tensor([1.0, 2.0]),
                meta=_make_sglang_meta(dp_rank=0, dp_size=2),
            ),
            _make_item(
                value=torch.tensor([]),
                meta=_make_sglang_meta(dp_rank=1, dp_size=2),
            ),
        ]

        result: list[ValueWithMeta] = filter_to_non_empty_dp_rank(
            items, dp_axis=ParallelAxis.DP
        )

        assert len(result) == 1
        assert torch.equal(result[0].value, torch.tensor([1.0, 2.0]))
```
**EN:** This test method exercises dp2 one empty one nonempty sglang and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dp2 one empty one nonempty sglang 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 164-182: Run test: dp2 one empty one nonempty megatron
```python
    def test_dp2_one_empty_one_nonempty_megatron(self) -> None:
        """DP=2 megatron, rank 1 has data, rank 0 has empty tensor."""
        items: list[ValueWithMeta] = [
            _make_item(
                value=torch.tensor([]),
                meta=_make_megatron_meta(dp_rank=0, dp_size=2),
            ),
            _make_item(
                value=torch.tensor([3.0, 4.0]),
                meta=_make_megatron_meta(dp_rank=1, dp_size=2),
            ),
        ]

        result: list[ValueWithMeta] = filter_to_non_empty_dp_rank(
            items, dp_axis=ParallelAxis.DP
        )

        assert len(result) == 1
        assert torch.equal(result[0].value, torch.tensor([3.0, 4.0]))
```
**EN:** This test method exercises dp2 one empty one nonempty megatron and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dp2 one empty one nonempty megatron 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 184-200: Run test: dp2 both nonempty raises
```python
    def test_dp2_both_nonempty_raises(self) -> None:
        """DP=2, both ranks have data: assertion error."""
        items: list[ValueWithMeta] = [
            _make_item(
                value=torch.tensor([1.0]),
                meta=_make_sglang_meta(dp_rank=0, dp_size=2),
            ),
            _make_item(
                value=torch.tensor([2.0]),
                meta=_make_sglang_meta(dp_rank=1, dp_size=2),
            ),
        ]

        with pytest.raises(
            AssertionError, match="Expected exactly 1 non-empty dp_rank"
        ):
            filter_to_non_empty_dp_rank(items, dp_axis=ParallelAxis.DP)
```
**EN:** This test method exercises dp2 both nonempty raises and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dp2 both nonempty raises 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 202-229: Run test: dp2 with tp2 filters correctly
```python
    def test_dp2_with_tp2_filters_correctly(self) -> None:
        """DP=2 x TP=2: 4 items total, 2 non-empty from dp_rank=0."""
        items: list[ValueWithMeta] = [
            _make_item(
                value=torch.tensor([1.0]),
                meta=_make_sglang_meta(tp_rank=0, tp_size=2, dp_rank=0, dp_size=2),
            ),
            _make_item(
                value=torch.tensor([2.0]),
                meta=_make_sglang_meta(tp_rank=1, tp_size=2, dp_rank=0, dp_size=2),
            ),
            _make_item(
                value=torch.tensor([]),
                meta=_make_sglang_meta(tp_rank=0, tp_size=2, dp_rank=1, dp_size=2),
            ),
            _make_item(
                value=torch.tensor([]),
                meta=_make_sglang_meta(tp_rank=1, tp_size=2, dp_rank=1, dp_size=2),
            ),
        ]

        result: list[ValueWithMeta] = filter_to_non_empty_dp_rank(
            items, dp_axis=ParallelAxis.DP
        )

        assert len(result) == 2
        assert torch.equal(result[0].value, torch.tensor([1.0]))
        assert torch.equal(result[1].value, torch.tensor([2.0]))
```
**EN:** This test method exercises dp2 with tp2 filters correctly and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dp2 with tp2 filters correctly 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 237-237: Define class TestExtractDpInfoWithAxis
```python
class TestExtractDpInfoWithAxis:
```
**EN:** This declaration introduces the `TestExtractDpInfoWithAxis` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestExtractDpInfoWithAxis` 测试类，并说明它通过继承承担的职责。

### Lines 238-247: Run test: moe dp axis found
```python
    def test_moe_dp_axis_found(self) -> None:
        meta: dict = {
            "sglang_parallel_info": {
                "dp_rank": 0,
                "dp_size": 2,
                "moe_dp_rank": 1,
                "moe_dp_size": 4,
            }
        }
        assert _extract_dp_info(meta, dp_axis=ParallelAxis.MOE_DP) == (1, 4)
```
**EN:** This test method exercises moe dp axis found and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 moe dp axis found 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 249-251: Run test: moe dp axis not found returns none
```python
    def test_moe_dp_axis_not_found_returns_none(self) -> None:
        meta: dict = _make_sglang_meta(dp_rank=0, dp_size=2)
        assert _extract_dp_info(meta, dp_axis=ParallelAxis.MOE_DP) is None
```
**EN:** This test method exercises moe dp axis not found returns none and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 moe dp axis not found returns none 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 253-255: Run test: dp axis uses default fields
```python
    def test_dp_axis_uses_default_fields(self) -> None:
        meta: dict = _make_sglang_meta(dp_rank=1, dp_size=4)
        assert _extract_dp_info(meta, dp_axis=ParallelAxis.DP) == (1, 4)
```
**EN:** This test method exercises dp axis uses default fields and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dp axis uses default fields 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 258-258: Define class TestFilterToNonEmptyDpRankWithAxis
```python
class TestFilterToNonEmptyDpRankWithAxis:
```
**EN:** This declaration introduces the `TestFilterToNonEmptyDpRankWithAxis` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFilterToNonEmptyDpRankWithAxis` 测试类，并说明它通过继承承担的职责。

### Lines 259-277: Run test: dp axis unchanged behavior
```python
    def test_dp_axis_unchanged_behavior(self) -> None:
        """dp_axis=ParallelAxis.DP → same behavior as default (regression)."""
        items: list[ValueWithMeta] = [
            _make_item(
                value=torch.tensor([1.0, 2.0]),
                meta=_make_sglang_meta(dp_rank=0, dp_size=2),
            ),
            _make_item(
                value=torch.tensor([]),
                meta=_make_sglang_meta(dp_rank=1, dp_size=2),
            ),
        ]

        result: list[ValueWithMeta] = filter_to_non_empty_dp_rank(
            items, dp_axis=ParallelAxis.DP
        )

        assert len(result) == 1
        assert torch.equal(result[0].value, torch.tensor([1.0, 2.0]))
```
**EN:** This test method exercises dp axis unchanged behavior and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dp axis unchanged behavior 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 279-296: Run test: moe dp axis absent noop
```python
    def test_moe_dp_axis_absent_noop(self) -> None:
        """MOE_DP axis fields not in metadata → noop, return items unchanged."""
        items: list[ValueWithMeta] = [
            _make_item(
                value=torch.tensor([1.0]),
                meta=_make_sglang_meta(dp_rank=0, dp_size=2),
            ),
            _make_item(
                value=torch.tensor([2.0]),
                meta=_make_sglang_meta(dp_rank=1, dp_size=2),
            ),
        ]

        result: list[ValueWithMeta] = filter_to_non_empty_dp_rank(
            items, dp_axis=ParallelAxis.MOE_DP
        )

        assert result is items
```
**EN:** This test method exercises moe dp axis absent noop and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 moe dp axis absent noop 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 298-316: Run test: moe dp axis size 1 noop
```python
    def test_moe_dp_axis_size_1_noop(self) -> None:
        """MOE_DP axis present but size=1 → noop."""
        meta: dict = {
            "sglang_parallel_info": {
                "dp_rank": 0,
                "dp_size": 2,
                "moe_dp_rank": 0,
                "moe_dp_size": 1,
            }
        }
        items: list[ValueWithMeta] = [
            _make_item(value=torch.tensor([1.0]), meta=meta),
        ]

        result: list[ValueWithMeta] = filter_to_non_empty_dp_rank(
            items, dp_axis=ParallelAxis.MOE_DP
        )

        assert result is items
```
**EN:** This test method exercises moe dp axis size 1 noop and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 moe dp axis size 1 noop 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 318-346: Run test: moe dp axis filters correctly
```python
    def test_moe_dp_axis_filters_correctly(self) -> None:
        """MOE_DP axis size=2, one empty rank → correctly filters."""
        meta_rank0: dict = {
            "sglang_parallel_info": {
                "dp_rank": 0,
                "dp_size": 2,
                "moe_dp_rank": 0,
                "moe_dp_size": 2,
            }
        }
        meta_rank1: dict = {
            "sglang_parallel_info": {
                "dp_rank": 0,
                "dp_size": 2,
                "moe_dp_rank": 1,
                "moe_dp_size": 2,
            }
        }
        items: list[ValueWithMeta] = [
            _make_item(value=torch.tensor([1.0, 2.0]), meta=meta_rank0),
            _make_item(value=torch.tensor([]), meta=meta_rank1),
        ]

        result: list[ValueWithMeta] = filter_to_non_empty_dp_rank(
            items, dp_axis=ParallelAxis.MOE_DP
        )

        assert len(result) == 1
        assert torch.equal(result[0].value, torch.tensor([1.0, 2.0]))
```
**EN:** This test method exercises moe dp axis filters correctly and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 moe dp axis filters correctly 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 349-350: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.srt.debug_utils.comparator.dp_utils`, `sglang.srt.debug_utils.dump_loader`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pytest`, `sys`, `torch`
- Notable symbols / 关键符号: None / 无
