# test_sequence_parallelism_threshold.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/compile/test_sequence_parallelism_threshold.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Pytest module for compilation behavior in compile / test_sequence_parallelism_threshold, covering compiler settings, graph transformations, and end-to-end correctness checks. / 面向 compile / test_sequence_parallelism_threshold 的编译测试模块，覆盖编译器配置、计算图变换以及端到端正确性校验。

## Line-by-Line Analysis / 逐行分析
### Imports and shared setup (lines 4-10)
```python
import pytest

from vllm.compilation.passes.fusion.sequence_parallelism import (
    SP_MIN_HIDDEN_SIZE,
    SP_MIN_PER_GPU_SIZE_MB,
    get_sequence_parallelism_threshold,
)
```
**EN:** This block imports the modules that the rest of the file relies on. It pulls in external libraries such as pytest; and vLLM components like vllm.compilation.passes.fusion.sequence_parallelism.
**CN:** 该代码块导入后续逻辑依赖的模块。 主要包括外部库，例如 pytest；vLLM 内部组件，例如 vllm.compilation.passes.fusion.sequence_parallelism。

### Class `TestGetSequenceParallelismThreshold` (lines 13-15)
```python
class TestGetSequenceParallelismThreshold:
    """Tests for get_sequence_parallelism_threshold function."""
```
**EN:** This helper class groups the state and behavior needed for TestGetSequenceParallelismThreshold. That structure makes repeated setup, graph inspection, or reusable assertions easier to manage.
**CN:** 这个辅助类把 TestGetSequenceParallelismThreshold 所需的状态与行为收拢到一起，使重复初始化、计算图检查或可复用断言更容易管理。

### Method `TestGetSequenceParallelismThreshold.test_non_cuda_returns_none` (lines 16-22)
```python
    def test_non_cuda_returns_none(self, mock_cuda_platform):
        """Non-CUDA platforms should return None."""
        with mock_cuda_platform(is_cuda=False):
            result = get_sequence_parallelism_threshold(
                hidden_size=8192, tp_size=2, element_size=2
            )
        assert result is None
```
**EN:** This method on `TestGetSequenceParallelismThreshold` checks non CUDA returns none. it consumes fixtures or inputs such as mock_cuda_platform. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestGetSequenceParallelismThreshold` 中的这个方法用于检查 non CUDA returns none。 它会使用诸如 mock_cuda_platform 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Method `TestGetSequenceParallelismThreshold.test_unsupported_device_capability_returns_none` (lines 24-30)
```python
    def test_unsupported_device_capability_returns_none(self, mock_cuda_platform):
        """Unsupported device capabilities (e.g., sm80) should return None."""
        with mock_cuda_platform(capability=(8, 0)):
            result = get_sequence_parallelism_threshold(
                hidden_size=8192, tp_size=2, element_size=2
            )
        assert result is None
```
**EN:** This method on `TestGetSequenceParallelismThreshold` checks unsupported device capability returns none. it consumes fixtures or inputs such as mock_cuda_platform. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestGetSequenceParallelismThreshold` 中的这个方法用于检查 unsupported device capability returns none。 它会使用诸如 mock_cuda_platform 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Method `TestGetSequenceParallelismThreshold.test_small_hidden_size_returns_none` (lines 32-40)
```python
    def test_small_hidden_size_returns_none(self, mock_cuda_platform):
        """H100 with hidden_size below threshold should return None."""
        with mock_cuda_platform(capability=(9, 0)):
            result = get_sequence_parallelism_threshold(
                hidden_size=4096,
                tp_size=2,
                element_size=2,  # 4096 < 8192
            )
        assert result is None
```
**EN:** This method on `TestGetSequenceParallelismThreshold` checks small hidden size returns none. it consumes fixtures or inputs such as mock_cuda_platform. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestGetSequenceParallelismThreshold` 中的这个方法用于检查 small hidden size returns none。 它会使用诸如 mock_cuda_platform 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Method `TestGetSequenceParallelismThreshold.test_h100_large_model_returns_threshold` (lines 42-62)
```python
    def test_h100_large_model_returns_threshold(self, mock_cuda_platform):
        """H100 with large enough hidden_size should return calculated threshold."""
        with mock_cuda_platform(capability=(9, 0)):
            hidden_size = 8192
            tp_size = 2
            element_size = 2  # float16/bfloat16

            result = get_sequence_parallelism_threshold(
                hidden_size=hidden_size,
                tp_size=tp_size,
                element_size=element_size,
            )

            # Verify calculation: (8 * 2 * 1024 * 1024) // (8192 * 2) = 1024
            MiB = 1024 * 1024
            expected = int(
                (SP_MIN_PER_GPU_SIZE_MB[90] * tp_size * MiB)
                // (hidden_size * element_size)
            )
            assert result == expected
            assert result == 1024
```
**EN:** This method on `TestGetSequenceParallelismThreshold` checks h100 large model returns threshold. it consumes fixtures or inputs such as mock_cuda_platform. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestGetSequenceParallelismThreshold` 中的这个方法用于检查 h100 large model returns threshold。 它会使用诸如 mock_cuda_platform 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Method `TestGetSequenceParallelismThreshold.test_threshold_calculation_variations` (lines 64-91)
```python
    @pytest.mark.parametrize(
        "hidden_size,tp_size,element_size,expected",
        [
            # Boundary: exactly at min hidden size threshold, tp_size=1
            # (8 * 1 * 1024 * 1024) // (8192 * 2) = 512
            (8192, 1, 2, 512),
            # Larger hidden size reduces token threshold
            # (8 * 1 * 1024 * 1024) // (16384 * 2) = 256
            (16384, 1, 2, 256),
            # Larger tp_size increases token threshold
            # (8 * 4 * 1024 * 1024) // (8192 * 2) = 2048
            (8192, 4, 2, 2048),
            # Larger element_size (fp32) reduces token threshold
            # (8 * 2 * 1024 * 1024) // (8192 * 4) = 512
            (8192, 2, 4, 512),
        ],
    )
    def test_threshold_calculation_variations(
        self, mock_cuda_platform, hidden_size, tp_size, element_size, expected
    ):
        """Test threshold calculation with various parameter combinations."""
        with mock_cuda_platform(capability=(9, 0)):
            result = get_sequence_parallelism_threshold(
                hidden_size=hidden_size,
                tp_size=tp_size,
                element_size=element_size,
            )
            assert result == expected
```
**EN:** This method on `TestGetSequenceParallelismThreshold` checks threshold calculation variations. It is parameterized across 1 input dimension so one definition covers many scenarios. it consumes fixtures or inputs such as mock_cuda_platform, hidden_size, tp_size, element_size. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestGetSequenceParallelismThreshold` 中的这个方法用于检查 threshold calculation variations。 它通过 1 组参数化输入覆盖多种场景；它会使用诸如 mock_cuda_platform、hidden_size、tp_size、element_size 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

### Method `TestGetSequenceParallelismThreshold.test_hidden_size_boundary` (lines 93-110)
```python
    def test_hidden_size_boundary(self, mock_cuda_platform):
        """Test behavior at the exact hidden_size boundary."""
        with mock_cuda_platform(capability=(9, 0)):
            # Just below threshold
            result = get_sequence_parallelism_threshold(
                hidden_size=SP_MIN_HIDDEN_SIZE[90] - 1,
                tp_size=2,
                element_size=2,
            )
            assert result is None

            # Exactly at threshold
            result = get_sequence_parallelism_threshold(
                hidden_size=SP_MIN_HIDDEN_SIZE[90],
                tp_size=2,
                element_size=2,
            )
            assert result is not None
```
**EN:** This method on `TestGetSequenceParallelismThreshold` checks hidden size boundary. it consumes fixtures or inputs such as mock_cuda_platform. assertions at the end lock in the intended behavior or graph shape.
**CN:** `TestGetSequenceParallelismThreshold` 中的这个方法用于检查 hidden size boundary。 它会使用诸如 mock_cuda_platform 等 fixture 或输入；结尾处的断言会固定预期行为或计算图形态。

## Key Concepts / 关键概念
- **Pytest parameterization / pytest 参数化:** The module expands one definition into many concrete scenarios through parametrized inputs. / 该模块通过参数化输入把单个定义扩展成多组具体场景。
- **Behavioral assertions / 行为断言:** Assertions check tensor values, graph structure, or expected error conditions. / 断言用于检查张量结果、计算图结构或预期的报错路径。
- **Compilation coverage / 编译路径覆盖:** The file exercises compile modes, graph passes, backend choices, or output stability across configurations. / 该文件覆盖编译模式、图优化 pass、后端选择以及不同配置下的输出稳定性。

## Dependencies / 依赖关系
- `pytest`
- `vllm.compilation.passes.fusion.sequence_parallelism -> SP_MIN_HIDDEN_SIZE, SP_MIN_PER_GPU_SIZE_MB, get_sequence_parallelism_threshold`
