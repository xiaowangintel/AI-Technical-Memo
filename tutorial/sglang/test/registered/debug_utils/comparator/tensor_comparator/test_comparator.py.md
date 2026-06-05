# test_comparator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/tensor_comparator/test_comparator.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on tensor comparator comparator in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 tensor comparator comparator 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Import dependencies
```python
import sys

import pytest
import torch

from sglang.srt.debug_utils.comparator.tensor_comparator.comparator import (
    QUANTILE_NUMEL_THRESHOLD,
    SAMPLE_DIFF_THRESHOLD,
    _compute_tensor_stats,
    compare_tensor_pair,
    compute_diff,
    compute_tensor_info,
)
from sglang.srt.debug_utils.comparator.tensor_comparator.types import DiffInfo
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 17-17: Register CI metadata
```python
register_cpu_ci(est_time=20, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 20-20: Define class TestComputeTensorInfo
```python
class TestComputeTensorInfo:
```
**EN:** This declaration introduces the `TestComputeTensorInfo` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestComputeTensorInfo` 测试类，并说明它通过继承承担的职责。

### Lines 21-26: Run test: basic tensor returns correct shape and dtype
```python
    def test_basic_tensor_returns_correct_shape_and_dtype(self) -> None:
        tensor = torch.randn(2, 3)
        info = compute_tensor_info(tensor)
        assert info.shape == [2, 3]
        assert info.dtype == "torch.float32"
        assert info.stats.mean == pytest.approx(tensor.float().mean().item(), abs=1e-4)
```
**EN:** This test method exercises basic tensor returns correct shape and dtype and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 basic tensor returns correct shape and dtype 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 28-31: Run test: include sample false returns none sample
```python
    def test_include_sample_false_returns_none_sample(self) -> None:
        tensor = torch.randn(2, 3)
        info = compute_tensor_info(tensor, include_sample=False)
        assert info.sample is None
```
**EN:** This test method exercises include sample false returns none sample and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 include sample false returns none sample 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 33-37: Run test: include sample true returns string sample
```python
    def test_include_sample_true_returns_string_sample(self) -> None:
        tensor = torch.randn(2, 3)
        info = compute_tensor_info(tensor, include_sample=True)
        assert info.sample is not None
        assert isinstance(info.sample, str)
```
**EN:** This test method exercises include sample true returns string sample and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 include sample true returns string sample 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 39-44: Run test: empty tensor stats are zero
```python
    def test_empty_tensor_stats_are_zero(self) -> None:
        tensor = torch.tensor([])
        info = compute_tensor_info(tensor)
        assert info.stats.mean == 0.0
        assert info.stats.std == 0.0
        assert info.shape == [0]
```
**EN:** This test method exercises empty tensor stats are zero and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty tensor stats are zero 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 46-53: Run test: integer tensor converted to float for stats
```python
    def test_integer_tensor_converted_to_float_for_stats(self) -> None:
        """Integer tensors should be cast to float internally for stats computation."""
        tensor = torch.tensor([1, 2, 3, 4], dtype=torch.int32)
        info = compute_tensor_info(tensor)
        assert info.dtype == "torch.int32"
        assert info.stats.mean == pytest.approx(2.5, abs=1e-4)
        assert info.stats.min == pytest.approx(1.0, abs=1e-4)
        assert info.stats.max == pytest.approx(4.0, abs=1e-4)
```
**EN:** This test method exercises integer tensor converted to float for stats and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 integer tensor converted to float for stats 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 55-61: Run test: bfloat16 tensor shape and stats
```python
    def test_bfloat16_tensor_shape_and_stats(self) -> None:
        """bfloat16 tensors produce correct shape and dtype string."""
        tensor = torch.ones(3, 4, dtype=torch.bfloat16)
        info = compute_tensor_info(tensor)
        assert info.shape == [3, 4]
        assert info.dtype == "torch.bfloat16"
        assert info.stats.mean == pytest.approx(1.0, abs=1e-2)
```
**EN:** This test method exercises bfloat16 tensor shape and stats and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 bfloat16 tensor shape and stats 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 63-67: Run test: multidimensional shape
```python
    def test_multidimensional_shape(self) -> None:
        """Shape is preserved for high-rank tensors."""
        tensor = torch.randn(2, 3, 4, 5)
        info = compute_tensor_info(tensor)
        assert info.shape == [2, 3, 4, 5]
```
**EN:** This test method exercises multidimensional shape and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multidimensional shape 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 69-76: Run test: scalar tensor
```python
    def test_scalar_tensor(self) -> None:
        """Scalar (0-dim) tensor produces empty shape list."""
        tensor = torch.tensor(3.14)
        info = compute_tensor_info(tensor)
        assert info.shape == []
        assert info.stats.mean == pytest.approx(3.14, abs=1e-4)
        assert info.stats.min == pytest.approx(3.14, abs=1e-4)
        assert info.stats.max == pytest.approx(3.14, abs=1e-4)
```
**EN:** This test method exercises scalar tensor and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 scalar tensor 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 78-83: Run test: include sample true contains tensor representation
```python
    def test_include_sample_true_contains_tensor_representation(self) -> None:
        """Sample string should contain some recognizable tensor content."""
        tensor = torch.tensor([1.0, 2.0])
        info = compute_tensor_info(tensor, include_sample=True)
        assert info.sample is not None
        assert "1." in info.sample or "2." in info.sample
```
**EN:** This test method exercises include sample true contains tensor representation and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 include sample true contains tensor representation 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 85-90: Run test: percentiles present for small tensor
```python
    def test_percentiles_present_for_small_tensor(self) -> None:
        """Small tensors (< threshold) should have percentile data."""
        tensor = torch.randn(100)
        info = compute_tensor_info(tensor)
        assert len(info.stats.percentiles) > 0
        assert 50 in info.stats.percentiles
```
**EN:** This test method exercises percentiles present for small tensor and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 percentiles present for small tensor 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 93-93: Define class TestComputeTensorInfo
```python
class TestComputeTensorInfo:
```
**EN:** This declaration introduces the `TestComputeTensorInfo` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestComputeTensorInfo` 测试类，并说明它通过继承承担的职责。

### Lines 94-99: Run test: basic tensor returns correct shape and dtype
```python
    def test_basic_tensor_returns_correct_shape_and_dtype(self) -> None:
        tensor = torch.randn(2, 3)
        info = compute_tensor_info(tensor)
        assert info.shape == [2, 3]
        assert info.dtype == "torch.float32"
        assert info.stats.mean == pytest.approx(tensor.float().mean().item(), abs=1e-4)
```
**EN:** This test method exercises basic tensor returns correct shape and dtype and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 basic tensor returns correct shape and dtype 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 101-104: Run test: include sample false returns none sample
```python
    def test_include_sample_false_returns_none_sample(self) -> None:
        tensor = torch.randn(2, 3)
        info = compute_tensor_info(tensor, include_sample=False)
        assert info.sample is None
```
**EN:** This test method exercises include sample false returns none sample and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 include sample false returns none sample 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 106-110: Run test: include sample true returns string sample
```python
    def test_include_sample_true_returns_string_sample(self) -> None:
        tensor = torch.randn(2, 3)
        info = compute_tensor_info(tensor, include_sample=True)
        assert info.sample is not None
        assert isinstance(info.sample, str)
```
**EN:** This test method exercises include sample true returns string sample and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 include sample true returns string sample 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 112-117: Run test: empty tensor stats are zero
```python
    def test_empty_tensor_stats_are_zero(self) -> None:
        tensor = torch.tensor([])
        info = compute_tensor_info(tensor)
        assert info.stats.mean == 0.0
        assert info.stats.std == 0.0
        assert info.shape == [0]
```
**EN:** This test method exercises empty tensor stats are zero and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty tensor stats are zero 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 119-126: Run test: integer tensor converted to float for stats
```python
    def test_integer_tensor_converted_to_float_for_stats(self) -> None:
        """Integer tensors should be cast to float internally for stats computation."""
        tensor = torch.tensor([1, 2, 3, 4], dtype=torch.int32)
        info = compute_tensor_info(tensor)
        assert info.dtype == "torch.int32"
        assert info.stats.mean == pytest.approx(2.5, abs=1e-4)
        assert info.stats.min == pytest.approx(1.0, abs=1e-4)
        assert info.stats.max == pytest.approx(4.0, abs=1e-4)
```
**EN:** This test method exercises integer tensor converted to float for stats and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 integer tensor converted to float for stats 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 128-134: Run test: bfloat16 tensor shape and stats
```python
    def test_bfloat16_tensor_shape_and_stats(self) -> None:
        """bfloat16 tensors produce correct shape and dtype string."""
        tensor = torch.ones(3, 4, dtype=torch.bfloat16)
        info = compute_tensor_info(tensor)
        assert info.shape == [3, 4]
        assert info.dtype == "torch.bfloat16"
        assert info.stats.mean == pytest.approx(1.0, abs=1e-2)
```
**EN:** This test method exercises bfloat16 tensor shape and stats and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 bfloat16 tensor shape and stats 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 136-140: Run test: multidimensional shape
```python
    def test_multidimensional_shape(self) -> None:
        """Shape is preserved for high-rank tensors."""
        tensor = torch.randn(2, 3, 4, 5)
        info = compute_tensor_info(tensor)
        assert info.shape == [2, 3, 4, 5]
```
**EN:** This test method exercises multidimensional shape and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multidimensional shape 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 142-149: Run test: scalar tensor
```python
    def test_scalar_tensor(self) -> None:
        """Scalar (0-dim) tensor produces empty shape list."""
        tensor = torch.tensor(3.14)
        info = compute_tensor_info(tensor)
        assert info.shape == []
        assert info.stats.mean == pytest.approx(3.14, abs=1e-4)
        assert info.stats.min == pytest.approx(3.14, abs=1e-4)
        assert info.stats.max == pytest.approx(3.14, abs=1e-4)
```
**EN:** This test method exercises scalar tensor and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 scalar tensor 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 151-156: Run test: include sample true contains tensor representation
```python
    def test_include_sample_true_contains_tensor_representation(self) -> None:
        """Sample string should contain some recognizable tensor content."""
        tensor = torch.tensor([1.0, 2.0])
        info = compute_tensor_info(tensor, include_sample=True)
        assert info.sample is not None
        assert "1." in info.sample or "2." in info.sample
```
**EN:** This test method exercises include sample true contains tensor representation and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 include sample true contains tensor representation 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 158-163: Run test: percentiles present for small tensor
```python
    def test_percentiles_present_for_small_tensor(self) -> None:
        """Small tensors (< threshold) should have percentile data."""
        tensor = torch.randn(100)
        info = compute_tensor_info(tensor)
        assert len(info.stats.percentiles) > 0
        assert 50 in info.stats.percentiles
```
**EN:** This test method exercises percentiles present for small tensor and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 percentiles present for small tensor 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 166-166: Define class TestComputeTensorStats
```python
class TestComputeTensorStats:
```
**EN:** This declaration introduces the `TestComputeTensorStats` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestComputeTensorStats` 测试类，并说明它通过继承承担的职责。

### Lines 167-175: Run test: basic stats
```python
    def test_basic_stats(self):
        x = torch.tensor([1.0, 2.0, 3.0, 4.0, 5.0])
        stats = _compute_tensor_stats(x)

        assert stats.mean == pytest.approx(3.0, abs=1e-4)
        assert stats.abs_mean == pytest.approx(3.0, abs=1e-4)
        assert stats.std == pytest.approx(1.5811, abs=1e-3)
        assert stats.min == pytest.approx(1.0, abs=1e-4)
        assert stats.max == pytest.approx(5.0, abs=1e-4)
```
**EN:** This test method exercises basic stats and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 basic stats 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 177-182: Run test: abs mean with negative values
```python
    def test_abs_mean_with_negative_values(self):
        x = torch.tensor([-3.0, -1.0, 1.0, 3.0])
        stats = _compute_tensor_stats(x)

        assert stats.mean == pytest.approx(0.0, abs=1e-4)
        assert stats.abs_mean == pytest.approx(2.0, abs=1e-4)
```
**EN:** This test method exercises abs mean with negative values and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 abs mean with negative values 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 184-192: Run test: quantile values
```python
    def test_quantile_values(self):
        x = torch.linspace(0.0, 100.0, steps=1000)
        stats = _compute_tensor_stats(x)

        assert stats.percentiles[1] == pytest.approx(1.0, abs=0.5)
        assert stats.percentiles[5] == pytest.approx(5.0, abs=0.5)
        assert stats.percentiles[50] == pytest.approx(50.0, abs=0.5)
        assert stats.percentiles[95] == pytest.approx(95.0, abs=0.5)
        assert stats.percentiles[99] == pytest.approx(99.0, abs=0.5)
```
**EN:** This test method exercises quantile values and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 quantile values 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 194-199: Run test: large tensor skips quantiles
```python
    def test_large_tensor_skips_quantiles(self):
        x = torch.randn(QUANTILE_NUMEL_THRESHOLD + 1)
        stats = _compute_tensor_stats(x)

        assert stats.mean is not None
        assert stats.percentiles == {}
```
**EN:** This test method exercises large tensor skips quantiles and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 large tensor skips quantiles 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 202-202: Define class TestComputeDiff
```python
class TestComputeDiff:
```
**EN:** This declaration introduces the `TestComputeDiff` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestComputeDiff` 测试类，并说明它通过继承承担的职责。

### Lines 203-213: Run test: identical tensors
```python
    def test_identical_tensors(self):
        x = torch.ones(10, 10)
        diff = compute_diff(x_baseline=x, x_target=x)

        assert diff.rel_diff == pytest.approx(0.0, abs=1e-5)
        assert diff.max_abs_diff == pytest.approx(0.0, abs=1e-5)
        assert diff.mean_abs_diff == pytest.approx(0.0, abs=1e-5)
        assert diff.abs_diff_percentiles[50] == pytest.approx(0.0, abs=1e-5)
        assert diff.abs_diff_percentiles[95] == pytest.approx(0.0, abs=1e-5)
        assert diff.abs_diff_percentiles[99] == pytest.approx(0.0, abs=1e-5)
        assert diff.passed is True
```
**EN:** This test method exercises identical tensors and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 identical tensors 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 215-230: Run test: known offset
```python
    def test_known_offset(self):
        x = torch.ones(10, 10)
        y = x.clone()
        y[3, 7] = 1.5

        diff = compute_diff(x_baseline=x, x_target=y)

        assert diff.max_abs_diff == pytest.approx(0.5, abs=1e-4)
        assert diff.max_diff_coord == [3, 7]
        assert diff.baseline_at_max == pytest.approx(1.0, abs=1e-4)
        assert diff.target_at_max == pytest.approx(1.5, abs=1e-4)
        assert diff.mean_abs_diff == pytest.approx(0.5 / 100, abs=1e-4)
        assert diff.abs_diff_percentiles[1] == pytest.approx(0.0, abs=1e-4)
        assert diff.abs_diff_percentiles[50] == pytest.approx(0.0, abs=1e-4)
        assert diff.abs_diff_percentiles[99] > 0
        assert diff.passed is False
```
**EN:** This test method exercises known offset and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 known offset 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 232-237: Run test: large tensor skips diff quantiles
```python
    def test_large_tensor_skips_diff_quantiles(self):
        x = torch.randn(QUANTILE_NUMEL_THRESHOLD + 1)
        y = x + 0.001
        diff = compute_diff(x_baseline=x, x_target=y)

        assert diff.abs_diff_percentiles == {}
```
**EN:** This test method exercises large tensor skips diff quantiles and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 large tensor skips diff quantiles 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 239-245: Run test: rel diff value
```python
    def test_rel_diff_value(self):
        x = torch.tensor([1.0, 0.0])
        y = torch.tensor([0.0, 1.0])
        diff = compute_diff(x_baseline=x, x_target=y)

        assert diff.rel_diff == pytest.approx(1.0, abs=1e-5)
        assert diff.passed is False
```
**EN:** This test method exercises rel diff value and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 rel diff value 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 247-260: Run test: per token with seq dim
```python
    def test_per_token_with_seq_dim(self) -> None:
        """seq_dim provided → per_token_rel_diff is list[float]."""
        torch.manual_seed(42)
        x: torch.Tensor = torch.randn(8, 16)
        y: torch.Tensor = x + torch.randn_like(x) * 0.01

        diff: DiffInfo = compute_diff(
            x_baseline=x, x_target=y, diff_threshold=1e-3, seq_dim=0
        )

        assert diff.per_token_rel_diff is not None
        assert isinstance(diff.per_token_rel_diff, list)
        assert len(diff.per_token_rel_diff) == 8
        assert all(isinstance(v, float) for v in diff.per_token_rel_diff)
```
**EN:** This test method exercises per token with seq dim and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 per token with seq dim 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 262-269: Run test: per token without seq dim
```python
    def test_per_token_without_seq_dim(self) -> None:
        """No seq_dim → per_token_rel_diff is None."""
        x: torch.Tensor = torch.randn(8, 16)
        y: torch.Tensor = x + torch.randn_like(x) * 0.01

        diff: DiffInfo = compute_diff(x_baseline=x, x_target=y, diff_threshold=1e-3)

        assert diff.per_token_rel_diff is None
```
**EN:** This test method exercises per token without seq dim and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 per token without seq dim 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 271-286: Run test: per token json roundtrip
```python
    def test_per_token_json_roundtrip(self) -> None:
        """DiffInfo with per_token_rel_diff survives JSON serialization."""
        torch.manual_seed(42)
        x: torch.Tensor = torch.randn(4, 8)
        y: torch.Tensor = x + torch.randn_like(x) * 0.01

        diff: DiffInfo = compute_diff(
            x_baseline=x, x_target=y, diff_threshold=1e-3, seq_dim=0
        )

        json_str: str = diff.model_dump_json()
        assert "per_token_rel_diff" in json_str

        roundtripped: DiffInfo = DiffInfo.model_validate_json(json_str)
        assert roundtripped.per_token_rel_diff is not None
        assert len(roundtripped.per_token_rel_diff) == 4
```
**EN:** This test method exercises per token json roundtrip and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 per token json roundtrip 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 289-289: Define class TestCompareTensors
```python
class TestCompareTensors:
```
**EN:** This declaration introduces the `TestCompareTensors` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestCompareTensors` 测试类，并说明它通过继承承担的职责。

### Lines 290-301: Run test: normal
```python
    def test_normal(self):
        x = torch.randn(5, 5)
        y = x + torch.randn(5, 5) * 0.001

        info = compare_tensor_pair(x_baseline=x, x_target=y, name="test")

        assert info.name == "test"
        assert info.baseline.shape == [5, 5]
        assert info.target.shape == [5, 5]
        assert info.shape_mismatch is False
        assert info.diff is not None
        assert info.diff_downcast is None
```
**EN:** This test method exercises normal and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 normal 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 303-310: Run test: shape mismatch
```python
    def test_shape_mismatch(self):
        x = torch.randn(3, 4)
        y = torch.randn(5, 6)

        info = compare_tensor_pair(x_baseline=x, x_target=y, name="mismatch")

        assert info.shape_mismatch is True
        assert info.diff is None
```
**EN:** This test method exercises shape mismatch and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 shape mismatch 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 312-321: Run test: dtype mismatch
```python
    def test_dtype_mismatch(self):
        x = torch.randn(5, 5, dtype=torch.float32)
        y = torch.randn(5, 5, dtype=torch.bfloat16)

        info = compare_tensor_pair(x_baseline=x, x_target=y, name="dtype_test")

        assert info.shape_mismatch is False
        assert info.diff is not None
        assert info.diff_downcast is not None
        assert info.downcast_dtype == "torch.bfloat16"
```
**EN:** This test method exercises dtype mismatch and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dtype mismatch 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 323-335: Run test: shape unification
```python
    def test_shape_unification(self):
        torch.manual_seed(0)
        core = torch.randn(4, 8)
        x = core.unsqueeze(0).unsqueeze(0)  # [1, 1, 4, 8]
        y = core.clone()  # [4, 8]

        info = compare_tensor_pair(x_baseline=x, x_target=y, name="unify")

        assert info.baseline.shape == [1, 1, 4, 8]
        assert info.unified_shape == [4, 8]
        assert info.shape_mismatch is False
        assert info.diff is not None
        assert info.diff.max_abs_diff == pytest.approx(0.0, abs=1e-5)
```
**EN:** This test method exercises shape unification and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 shape unification 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 337-346: Run test: sample generated when large diff
```python
    def test_sample_generated_when_large_diff(self):
        x = torch.zeros(5, 5)
        y = torch.ones(5, 5)

        info = compare_tensor_pair(x_baseline=x, x_target=y, name="big_diff")

        assert info.diff is not None
        assert info.diff.max_abs_diff > SAMPLE_DIFF_THRESHOLD
        assert info.baseline.sample is not None
        assert info.target.sample is not None
```
**EN:** This test method exercises sample generated when large diff and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 sample generated when large diff 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 348-357: Run test: no sample when small diff
```python
    def test_no_sample_when_small_diff(self):
        x = torch.ones(5, 5)
        y = x + 1e-5

        info = compare_tensor_pair(x_baseline=x, x_target=y, name="tiny_diff")

        assert info.diff is not None
        assert info.diff.max_abs_diff < SAMPLE_DIFF_THRESHOLD
        assert info.baseline.sample is None
        assert info.target.sample is None
```
**EN:** This test method exercises no sample when small diff and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no sample when small diff 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 360-361: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.tensor_comparator.comparator`, `sglang.srt.debug_utils.comparator.tensor_comparator.types`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pytest`, `sys`, `torch`
- Notable symbols / 关键符号: None / 无
