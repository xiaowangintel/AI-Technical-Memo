# test_parallel_info.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/aligner/unsharder/test_parallel_info.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on unsharder parallel info in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 unsharder parallel info 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: Import dependencies
```python
import sys

import pytest

from sglang.srt.debug_utils.comparator.aligner.unsharder.parallel_info import (
    normalize_parallel_info,
)
from sglang.srt.debug_utils.comparator.aligner.unsharder.types import AxisInfo
from sglang.srt.debug_utils.comparator.dims_spec import ParallelAxis
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 12-12: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 15-15: Define class TestNormalizeParallelInfo
```python
class TestNormalizeParallelInfo:
```
**EN:** This declaration introduces the `TestNormalizeParallelInfo` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNormalizeParallelInfo` 测试类，并说明它通过继承承担的职责。

### Lines 16-26: Run test: sglang info
```python
    def test_sglang_info(self) -> None:
        meta = {
            "sglang_parallel_info": {
                "tp_rank": 2,
                "tp_size": 4,
                "pp_rank": 0,
                "pp_size": 1,
            }
        }
        result = normalize_parallel_info(meta)
        assert result == {ParallelAxis.TP: AxisInfo(axis_rank=2, axis_size=4)}
```
**EN:** This test method exercises sglang info and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 sglang info 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 28-43: Run test: megatron info
```python
    def test_megatron_info(self) -> None:
        meta = {
            "megatron_parallel_info": {
                "tp_rank": 1,
                "tp_size": 2,
                "cp_rank": 0,
                "cp_size": 4,
                "dp_rank": 0,
                "dp_size": 1,
            }
        }
        result = normalize_parallel_info(meta)
        assert result == {
            ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=2),
            ParallelAxis.CP: AxisInfo(axis_rank=0, axis_size=4),
        }
```
**EN:** This test method exercises megatron info and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 megatron info 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 45-47: Run test: no parallel info
```python
    def test_no_parallel_info(self) -> None:
        assert normalize_parallel_info({}) == {}
        assert normalize_parallel_info({"other_key": 42}) == {}
```
**EN:** This test method exercises no parallel info and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no parallel info 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 49-55: Run test: both present raises
```python
    def test_both_present_raises(self) -> None:
        meta = {
            "sglang_parallel_info": {"tp_rank": 0, "tp_size": 2},
            "megatron_parallel_info": {"tp_rank": 0, "tp_size": 2},
        }
        with pytest.raises(ValueError, match="multiple parallel_info"):
            normalize_parallel_info(meta)
```
**EN:** This test method exercises both present raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 both present raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 57-71: Run test: megatron with sp
```python
    def test_megatron_with_sp(self) -> None:
        """Megatron SP reuses TP group: sp_rank==tp_rank, sp_size==tp_size."""
        meta = {
            "megatron_parallel_info": {
                "tp_rank": 1,
                "tp_size": 4,
                "sp_rank": 1,
                "sp_size": 4,
            }
        }
        result = normalize_parallel_info(meta)
        assert result == {
            ParallelAxis.TP: AxisInfo(axis_rank=1, axis_size=4),
            ParallelAxis.SP: AxisInfo(axis_rank=1, axis_size=4),
        }
```
**EN:** This test method exercises megatron with sp and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 megatron with sp 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 73-82: Run test: size 1 filtered
```python
    def test_size_1_filtered(self) -> None:
        meta = {
            "sglang_parallel_info": {
                "tp_rank": 0,
                "tp_size": 1,
                "cp_rank": 0,
                "cp_size": 1,
            }
        }
        assert normalize_parallel_info(meta) == {}
```
**EN:** This test method exercises size 1 filtered and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 size 1 filtered 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 84-95: Run test: recompute pseudo from top level meta
```python
    def test_recompute_pseudo_from_top_level_meta(self) -> None:
        """recompute_pseudo_rank/size at top-level meta is extracted alongside TP."""
        meta = {
            "recompute_pseudo_rank": 1,
            "recompute_pseudo_size": 2,
            "sglang_parallel_info": {"tp_rank": 0, "tp_size": 2},
        }
        result = normalize_parallel_info(meta)
        assert result == {
            ParallelAxis.RECOMPUTE_PSEUDO: AxisInfo(axis_rank=1, axis_size=2),
            ParallelAxis.TP: AxisInfo(axis_rank=0, axis_size=2),
        }
```
**EN:** This test method exercises recompute pseudo from top level meta and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 recompute pseudo from top level meta 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 98-99: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.aligner.unsharder.parallel_info`, `sglang.srt.debug_utils.comparator.aligner.unsharder.types`, `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pytest`, `sys`
- Notable symbols / 关键符号: None / 无
