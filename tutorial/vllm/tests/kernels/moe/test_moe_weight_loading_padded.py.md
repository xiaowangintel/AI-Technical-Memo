# test_moe_weight_loading_padded.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/kernels/moe/test_moe_weight_loading_padded.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for kernel-level behavior in kernels / moe / test_moe_weight_loading_padded, focusing on correctness, edge cases, and comparisons against trusted reference paths. / 面向 kernels / moe / test_moe_weight_loading_padded 的内核级 pytest 模块，重点覆盖正确性、边界情况以及与可信参考路径的对照。

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-10)
```python
"""Tests for FusedMoE weight loading with padded hidden dimensions.

When using DeepEP backends or NIXL EP with models like nemotron_h,
hidden_size may be rounded up (e.g., 2688 -> 3072) for backend requirements.
Weight parameters are created with the padded size, but checkpoint weights
have the original unpadded size. These tests verify that weight loading
correctly handles this mismatch.
"""
```
**EN:** This docstring states the file-level intent and helps readers understand the role of the module before they inspect the implementation.
**CN:** 这个文档字符串说明了文件级意图，让读者在查看实现前先理解模块职责。

### Imports and shared setup (lines 12-15)
```python
import pytest
import torch

from vllm.model_executor.layers.fused_moe.layer import FusedMoE
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest, torch; and vLLM components like vllm.model_executor.layers.fused_moe.layer.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest、torch；vLLM 内部组件，例如 vllm.model_executor.layers.fused_moe.layer。

### Class `TestGetHiddenDim` (lines 18-20)
```python
class TestGetHiddenDim:
    """Unit tests for _get_hidden_dim."""
```
**EN:** This helper class groups the state and behavior needed for TestGetHiddenDim. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestGetHiddenDim 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestGetHiddenDim.test_2d_non_transposed_w2` (lines 21-23)
```python
    def test_2d_non_transposed_w2(self):
        # w2: shard_dim=1 (intermediate), hidden=0
        assert FusedMoE._get_hidden_dim(shard_dim=1, ndim=2) == 0
```
**EN:** This method on `TestGetHiddenDim` checks 2d non transposed w2. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestGetHiddenDim` 中的这个方法用于检查 2d non transposed w2。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestGetHiddenDim.test_2d_non_transposed_w13` (lines 25-27)
```python
    def test_2d_non_transposed_w13(self):
        # w1/w3: shard_dim=0 (intermediate), hidden=1
        assert FusedMoE._get_hidden_dim(shard_dim=0, ndim=2) == 1
```
**EN:** This method on `TestGetHiddenDim` checks 2d non transposed w13. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestGetHiddenDim` 中的这个方法用于检查 2d non transposed w13。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestGetHiddenDim.test_2d_transposed_w2` (lines 29-31)
```python
    def test_2d_transposed_w2(self):
        # transposed w2: shard_dim=0, hidden=1
        assert FusedMoE._get_hidden_dim(shard_dim=0, ndim=2) == 1
```
**EN:** This method on `TestGetHiddenDim` checks 2d transposed w2. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestGetHiddenDim` 中的这个方法用于检查 2d transposed w2。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestGetHiddenDim.test_2d_transposed_w13` (lines 33-35)
```python
    def test_2d_transposed_w13(self):
        # transposed w1/w3: shard_dim=1, hidden=0
        assert FusedMoE._get_hidden_dim(shard_dim=1, ndim=2) == 0
```
**EN:** This method on `TestGetHiddenDim` checks 2d transposed w13. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestGetHiddenDim` 中的这个方法用于检查 2d transposed w13。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestGetHiddenDim.test_3d_non_transposed_w2` (lines 37-39)
```python
    def test_3d_non_transposed_w2(self):
        # 3D w2: shard_dim=2, hidden=1
        assert FusedMoE._get_hidden_dim(shard_dim=2, ndim=3) == 1
```
**EN:** This method on `TestGetHiddenDim` checks 3d non transposed w2. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestGetHiddenDim` 中的这个方法用于检查 3d non transposed w2。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestGetHiddenDim.test_3d_non_transposed_w13` (lines 41-43)
```python
    def test_3d_non_transposed_w13(self):
        # 3D w1/w3: shard_dim=1, hidden=2
        assert FusedMoE._get_hidden_dim(shard_dim=1, ndim=3) == 2
```
**EN:** This method on `TestGetHiddenDim` checks 3d non transposed w13. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestGetHiddenDim` 中的这个方法用于检查 3d non transposed w13。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestGetHiddenDim.test_3d_transposed_w2` (lines 45-47)
```python
    def test_3d_transposed_w2(self):
        # transposed 3D w2: shard_dim=1, hidden=2
        assert FusedMoE._get_hidden_dim(shard_dim=1, ndim=3) == 2
```
**EN:** This method on `TestGetHiddenDim` checks 3d transposed w2. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestGetHiddenDim` 中的这个方法用于检查 3d transposed w2。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestGetHiddenDim.test_3d_transposed_w13` (lines 49-51)
```python
    def test_3d_transposed_w13(self):
        # transposed 3D w1/w3: shard_dim=2, hidden=1
        assert FusedMoE._get_hidden_dim(shard_dim=2, ndim=3) == 1
```
**EN:** This method on `TestGetHiddenDim` checks 3d transposed w13. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestGetHiddenDim` 中的这个方法用于检查 3d transposed w13。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestGetHiddenDim.test_1d_returns_zero` (lines 53-56)
```python
    def test_1d_returns_zero(self):
        # 1D per-channel scales: always returns 0
        assert FusedMoE._get_hidden_dim(shard_dim=0, ndim=1) == 0
        assert FusedMoE._get_hidden_dim(shard_dim=1, ndim=1) == 0
```
**EN:** This method on `TestGetHiddenDim` checks 1d returns zero. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestGetHiddenDim` 中的这个方法用于检查 1d returns zero。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestGetHiddenDim.test_invalid_shard_dim_raises` (lines 58-61)
```python
    def test_invalid_shard_dim_raises(self):
        # shard_dim outside the data dimensions should raise
        with pytest.raises(ValueError, match="not a valid data dimension"):
            FusedMoE._get_hidden_dim(shard_dim=0, ndim=3)
```
**EN:** This method on `TestGetHiddenDim` checks invalid shard dim raises. the expected failure path is asserted explicitly.
**CN:** `TestGetHiddenDim` 中的这个方法用于检查 invalid shard dim raises。 代码会显式断言预期的失败路径。

### Class `TestNarrowExpertDataForPadding` (lines 64-66)
```python
class TestNarrowExpertDataForPadding:
    """Unit tests for _narrow_expert_data_for_padding."""
```
**EN:** This helper class groups the state and behavior needed for TestNarrowExpertDataForPadding. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestNarrowExpertDataForPadding 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestNarrowExpertDataForPadding.test_no_narrowing_when_shapes_match` (lines 67-74)
```python
    def test_no_narrowing_when_shapes_match(self):
        expert_data = torch.zeros(1024, 1024)
        loaded_weight = torch.randn(1024, 1024)
        result = FusedMoE._narrow_expert_data_for_padding(
            expert_data, loaded_weight, hidden_dim=0
        )
        assert result.shape == loaded_weight.shape
        assert result.data_ptr() == expert_data.data_ptr()
```
**EN:** This method on `TestNarrowExpertDataForPadding` checks no narrowing when shapes match. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestNarrowExpertDataForPadding` 中的这个方法用于检查 no narrowing when shapes match。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestNarrowExpertDataForPadding.test_narrow_w2_hidden_dim` (lines 76-83)
```python
    def test_narrow_w2_hidden_dim(self):
        # w2: (hidden_size, intermediate_size) - hidden_size padded at dim 0
        expert_data = torch.zeros(3072, 1024)
        loaded_weight = torch.randn(2688, 1024)
        result = FusedMoE._narrow_expert_data_for_padding(
            expert_data, loaded_weight, hidden_dim=0
        )
        assert result.shape == (2688, 1024)
```
**EN:** This method on `TestNarrowExpertDataForPadding` checks narrow w2 hidden dim. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestNarrowExpertDataForPadding` 中的这个方法用于检查 narrow w2 hidden dim。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestNarrowExpertDataForPadding.test_narrow_w13_hidden_dim` (lines 85-92)
```python
    def test_narrow_w13_hidden_dim(self):
        # w1/w3: (intermediate_size, hidden_size) - hidden_size padded at dim 1
        expert_data = torch.zeros(2048, 3072)
        loaded_weight = torch.randn(2048, 2688)
        result = FusedMoE._narrow_expert_data_for_padding(
            expert_data, loaded_weight, hidden_dim=1
        )
        assert result.shape == (2048, 2688)
```
**EN:** This method on `TestNarrowExpertDataForPadding` checks narrow w13 hidden dim. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestNarrowExpertDataForPadding` 中的这个方法用于检查 narrow w13 hidden dim。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestNarrowExpertDataForPadding.test_narrow_transposed_w2` (lines 94-102)
```python
    def test_narrow_transposed_w2(self):
        # transposed w2: (intermediate_size, hidden_size) - hidden at dim 1
        expert_data = torch.zeros(1024, 3072)
        loaded_weight = torch.randn(1024, 2688)
        hidden_dim = FusedMoE._get_hidden_dim(shard_dim=0, ndim=2)
        result = FusedMoE._narrow_expert_data_for_padding(
            expert_data, loaded_weight, hidden_dim=hidden_dim
        )
        assert result.shape == (1024, 2688)
```
**EN:** This method on `TestNarrowExpertDataForPadding` checks narrow transposed w2. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestNarrowExpertDataForPadding` 中的这个方法用于检查 narrow transposed w2。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestNarrowExpertDataForPadding.test_narrow_3d_full_load` (lines 104-111)
```python
    def test_narrow_3d_full_load(self):
        # 3D tensor for full_load path: w2 (num_experts, hidden_size, intermediate)
        expert_data = torch.zeros(8, 3072, 1024)
        loaded_weight = torch.randn(8, 2688, 1024)
        result = FusedMoE._narrow_expert_data_for_padding(
            expert_data, loaded_weight, hidden_dim=1
        )
        assert result.shape == (8, 2688, 1024)
```
**EN:** This method on `TestNarrowExpertDataForPadding` checks narrow 3d full load. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestNarrowExpertDataForPadding` 中的这个方法用于检查 narrow 3d full load。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestNarrowExpertDataForPadding.test_narrow_1d_scale` (lines 113-120)
```python
    def test_narrow_1d_scale(self):
        # 1D scale tensor: per-channel w2 scale (hidden_size,)
        expert_data = torch.zeros(3072)
        loaded_weight = torch.randn(2688)
        result = FusedMoE._narrow_expert_data_for_padding(
            expert_data, loaded_weight, hidden_dim=0
        )
        assert result.shape == (2688,)
```
**EN:** This method on `TestNarrowExpertDataForPadding` checks narrow 1d scale. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestNarrowExpertDataForPadding` 中的这个方法用于检查 narrow 1d scale。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestNarrowExpertDataForPadding.test_scalar_weight_no_op` (lines 122-130)
```python
    def test_scalar_weight_no_op(self):
        # 0-dim tensor should be a no-op
        expert_data = torch.zeros(3072)
        loaded_weight = torch.tensor(1.0)
        result = FusedMoE._narrow_expert_data_for_padding(
            expert_data, loaded_weight, hidden_dim=0
        )
        # ndim == 0, so no narrowing
        assert result.shape == (3072,)
```
**EN:** This method on `TestNarrowExpertDataForPadding` checks scalar weight no op. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestNarrowExpertDataForPadding` 中的这个方法用于检查 scalar weight no op。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestNarrowExpertDataForPadding.test_no_narrowing_when_loaded_weight_larger` (lines 132-140)
```python
    def test_no_narrowing_when_loaded_weight_larger(self):
        # Guard: don't narrow if loaded_weight is larger than expert_data
        expert_data = torch.zeros(2688, 1024)
        loaded_weight = torch.randn(3072, 1024)
        result = FusedMoE._narrow_expert_data_for_padding(
            expert_data, loaded_weight, hidden_dim=0
        )
        assert result.shape == (2688, 1024)
        assert result.data_ptr() == expert_data.data_ptr()
```
**EN:** This method on `TestNarrowExpertDataForPadding` checks no narrowing when loaded weight larger. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestNarrowExpertDataForPadding` 中的这个方法用于检查 no narrowing when loaded weight larger。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestNarrowExpertDataForPadding.test_negative_hidden_dim_is_noop` (lines 142-151)
```python
    def test_negative_hidden_dim_is_noop(self):
        # Negative hidden_dim should be a safe no-op (0 <= check)
        expert_data = torch.zeros(3072, 1024)
        loaded_weight = torch.randn(2688, 1024)
        result = FusedMoE._narrow_expert_data_for_padding(
            expert_data, loaded_weight, hidden_dim=-1
        )
        # -1 fails the 0 <= check, so no narrowing
        assert result.shape == (3072, 1024)
        assert result.data_ptr() == expert_data.data_ptr()
```
**EN:** This method on `TestNarrowExpertDataForPadding` checks negative hidden dim is noop. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestNarrowExpertDataForPadding` 中的这个方法用于检查 negative hidden dim is noop。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestNarrowExpertDataForPadding.test_only_narrows_hidden_dim` (lines 153-162)
```python
    def test_only_narrows_hidden_dim(self):
        # Verify that only the specified hidden_dim is narrowed,
        # even when other dimensions also differ
        expert_data = torch.zeros(3072, 2048)
        loaded_weight = torch.randn(2688, 1024)
        result = FusedMoE._narrow_expert_data_for_padding(
            expert_data, loaded_weight, hidden_dim=0
        )
        # Only dim 0 (hidden) should be narrowed; dim 1 stays at 2048
        assert result.shape == (2688, 2048)
```
**EN:** This method on `TestNarrowExpertDataForPadding` checks only narrows hidden dim. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestNarrowExpertDataForPadding` 中的这个方法用于检查 only narrows hidden dim。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestNarrowExpertDataForPadding.test_narrowed_data_shares_storage` (lines 164-175)
```python
    def test_narrowed_data_shares_storage(self):
        # Verify narrowing returns a view (writes go to original tensor)
        expert_data = torch.zeros(3072, 1024)
        loaded_weight = torch.randn(2688, 1024)
        result = FusedMoE._narrow_expert_data_for_padding(
            expert_data, loaded_weight, hidden_dim=0
        )
        result.copy_(loaded_weight)
        # The first 2688 rows of expert_data should now have loaded_weight
        assert torch.equal(expert_data[:2688, :], loaded_weight)
        # Padded region should remain zero
        assert torch.equal(expert_data[2688:, :], torch.zeros(3072 - 2688, 1024))
```
**EN:** This method on `TestNarrowExpertDataForPadding` checks narrowed data shares storage. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestNarrowExpertDataForPadding` 中的这个方法用于检查 narrowed data shares storage。 结尾处的断言会固定预期行为或计算图形态。

### Class `TestWeightLoadingWithPaddedHiddenSize` (lines 178-180)
```python
class TestWeightLoadingWithPaddedHiddenSize:
    """Integration-style tests that simulate padded weight loading."""
```
**EN:** This helper class groups the state and behavior needed for TestWeightLoadingWithPaddedHiddenSize. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestWeightLoadingWithPaddedHiddenSize 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestWeightLoadingWithPaddedHiddenSize.test_load_w2_with_padding` (lines 181-201)
```python
    def test_load_w2_with_padding(self):
        """Simulate loading w2 weights when hidden_size is padded."""
        padded_hidden = 3072
        original_hidden = 2688
        intermediate = 1024

        expert_data_full = torch.zeros(padded_hidden, intermediate)
        loaded_weight = torch.randn(original_hidden, intermediate)

        # w2 non-transposed: shard_dim=1, hidden_dim=0
        hidden_dim = FusedMoE._get_hidden_dim(shard_dim=1, ndim=2)
        expert_data = FusedMoE._narrow_expert_data_for_padding(
            expert_data_full, loaded_weight, hidden_dim=hidden_dim
        )
        expert_data.copy_(loaded_weight)

        assert torch.equal(expert_data_full[:original_hidden, :], loaded_weight)
        assert torch.equal(
            expert_data_full[original_hidden:, :],
            torch.zeros(padded_hidden - original_hidden, intermediate),
        )
```
**EN:** This method on `TestWeightLoadingWithPaddedHiddenSize` checks load w2 with padding. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestWeightLoadingWithPaddedHiddenSize` 中的这个方法用于检查 load w2 with padding。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestWeightLoadingWithPaddedHiddenSize.test_load_w13_with_padding` (lines 203-224)
```python
    def test_load_w13_with_padding(self):
        """Simulate loading w1/w3 weights when hidden_size is padded."""
        padded_hidden = 3072
        original_hidden = 2688
        intermediate = 1024

        # w1/w3: (intermediate_size, hidden_size)
        expert_data_full = torch.zeros(intermediate, padded_hidden)
        loaded_weight = torch.randn(intermediate, original_hidden)

        # w1 non-transposed: shard_dim=0, hidden_dim=1
        hidden_dim = FusedMoE._get_hidden_dim(shard_dim=0, ndim=2)
        expert_data = FusedMoE._narrow_expert_data_for_padding(
            expert_data_full, loaded_weight, hidden_dim=hidden_dim
        )
        expert_data.copy_(loaded_weight)

        assert torch.equal(expert_data_full[:, :original_hidden], loaded_weight)
        assert torch.equal(
            expert_data_full[:, original_hidden:],
            torch.zeros(intermediate, padded_hidden - original_hidden),
        )
```
**EN:** This method on `TestWeightLoadingWithPaddedHiddenSize` checks load w13 with padding. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestWeightLoadingWithPaddedHiddenSize` 中的这个方法用于检查 load w13 with padding。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestWeightLoadingWithPaddedHiddenSize.test_load_transposed_w2_with_padding` (lines 226-242)
```python
    def test_load_transposed_w2_with_padding(self):
        """Simulate loading transposed w2 (GPTQ) with padded hidden_size."""
        padded_hidden = 3072
        original_hidden = 2688
        intermediate = 1024

        # transposed w2: (intermediate_size, hidden_size), shard_dim=0
        expert_data_full = torch.zeros(intermediate, padded_hidden)
        loaded_weight = torch.randn(intermediate, original_hidden)

        hidden_dim = FusedMoE._get_hidden_dim(shard_dim=0, ndim=2)
        expert_data = FusedMoE._narrow_expert_data_for_padding(
            expert_data_full, loaded_weight, hidden_dim=hidden_dim
        )
        expert_data.copy_(loaded_weight)

        assert torch.equal(expert_data_full[:, :original_hidden], loaded_weight)
```
**EN:** This method on `TestWeightLoadingWithPaddedHiddenSize` checks load transposed w2 with padding. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestWeightLoadingWithPaddedHiddenSize` 中的这个方法用于检查 load transposed w2 with padding。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestWeightLoadingWithPaddedHiddenSize.test_no_padding_is_noop` (lines 244-258)
```python
    def test_no_padding_is_noop(self):
        """Verify that when sizes match, behavior is unchanged."""
        hidden = 2048
        intermediate = 1024

        expert_data_full = torch.zeros(hidden, intermediate)
        loaded_weight = torch.randn(hidden, intermediate)

        hidden_dim = FusedMoE._get_hidden_dim(shard_dim=1, ndim=2)
        expert_data = FusedMoE._narrow_expert_data_for_padding(
            expert_data_full, loaded_weight, hidden_dim=hidden_dim
        )
        expert_data.copy_(loaded_weight)

        assert torch.equal(expert_data_full, loaded_weight)
```
**EN:** This method on `TestWeightLoadingWithPaddedHiddenSize` checks no padding is noop. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestWeightLoadingWithPaddedHiddenSize` 中的这个方法用于检查 no padding is noop。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestWeightLoadingWithPaddedHiddenSize.test_narrow_shard_dim` (lines 260-293)
```python
    def test_narrow_shard_dim(self):
        """Simulate loading w2 when both hidden_size and intermediate_size
        are padded.
        """
        padded_hidden = 3072
        original_hidden = 2688
        padded_intermediate = 1024
        original_intermediate = 896

        expert_data_full = torch.zeros(padded_hidden, padded_intermediate)
        loaded_weight = torch.randn(original_hidden, original_intermediate)

        shard_dim = 1
        hidden_dim = FusedMoE._get_hidden_dim(shard_dim=shard_dim, ndim=2)
        expert_data = FusedMoE._narrow_expert_data_for_padding(
            expert_data_full,
            loaded_weight,
            hidden_dim=hidden_dim,
            shard_dim=shard_dim,
        )
        expert_data.copy_(loaded_weight)

        assert torch.equal(
            expert_data_full[:original_hidden, :original_intermediate],
            loaded_weight,
        )
        assert torch.equal(
            expert_data_full[original_hidden:, :],
            torch.zeros(padded_hidden - original_hidden, padded_intermediate),
        )
        assert torch.equal(
            expert_data_full[:original_hidden, original_intermediate:],
            torch.zeros(original_hidden, padded_intermediate - original_intermediate),
        )
```
**EN:** This method on `TestWeightLoadingWithPaddedHiddenSize` checks narrow shard dim. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestWeightLoadingWithPaddedHiddenSize` 中的这个方法用于检查 narrow shard dim。 结尾处的断言会固定预期行为或计算图形态。

### Method `TestWeightLoadingWithPaddedHiddenSize.test_bnb_shape_mismatch_raises` (lines 295-327)
```python
    def test_bnb_shape_mismatch_raises(self):
        """BnB + padded hidden_size should raise via weight_loader."""
        from unittest.mock import MagicMock

        num_experts = 1
        padded_packed = 3072  # padded packed size
        original_packed = 2688  # original packed size

        # Build a param that looks like a BnB 4-bit MoE weight.
        param_data = torch.zeros(num_experts, padded_packed, 1, dtype=torch.uint8)
        param = torch.nn.Parameter(param_data, requires_grad=False)
        param.use_bitsandbytes_4bit = True

        loaded_weight = torch.randint(0, 255, (original_packed, 1), dtype=torch.uint8)

        # Minimal FusedMoE mock so weight_loader reaches the BnB path.
        moe = MagicMock(spec=FusedMoE)
        moe.quant_config = None
        moe.quant_method = MagicMock()
        moe.quant_method.__class__.__name__ = "BitsAndBytesMethod"
        moe._expert_map = None
        moe.tp_rank = 0

        # Call the real weight_loader (unbound) with our mock as self.
        with pytest.raises(ValueError, match="BitsAndBytes"):
            FusedMoE.weight_loader(
                moe,
                param,
                loaded_weight,
                weight_name="w2",
                shard_id="w2",
                expert_id=0,
            )
```
**EN:** This method on `TestWeightLoadingWithPaddedHiddenSize` checks bnb shape mismatch raises. the expected failure path is asserted explicitly.
**CN:** `TestWeightLoadingWithPaddedHiddenSize` 中的这个方法用于检查 bnb shape mismatch raises。 代码会显式断言预期的失败路径。

## Key Concepts / 关键概念
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Reference comparisons / 参考实现对照:** Optimized kernels are commonly checked against simpler reference implementations or fallback paths. / 优化后的内核通常会与更直接的参考实现或回退路径进行比对。

## Dependencies / 依赖关系
- `pytest`
- `torch`
- `vllm.model_executor.layers.fused_moe.layer -> FusedMoE`
