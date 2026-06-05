# test_aux_plugins.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/aligner/token_aligner/test_aux_plugins.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on token aligner aux plugins in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 token aligner aux plugins 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Import dependencies
```python
import sys

import pytest
import torch

from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.aux_plugins import (
    _infer_positions,
    _MegatronPlugin,
    _SGLangPlugin,
)
from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types import (
    PositionalSeqId,
    SGLangSeqId,
    TokenAlignerStepAux,
)
from sglang.srt.debug_utils.comparator.dims_spec import TokenLayout
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 19-19: Register CI metadata
```python
register_cpu_ci(est_time=15, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 21-22: Define module constants
```python
_sglang_plugin = _SGLangPlugin()
_megatron_plugin = _MegatronPlugin()
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 25-25: Define class TestNormalizeSGLang
```python
class TestNormalizeSGLang:
```
**EN:** This declaration introduces the `TestNormalizeSGLang` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNormalizeSGLang` 测试类，并说明它通过继承承担的职责。

### Lines 26-26: Document the class `TestNormalizeSGLang`
```python
    """Tests for SGLang aux tensor normalization."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestNormalizeSGLang`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestNormalizeSGLang`的设计意图。

### Lines 28-44: Run test: with rids
```python
    def test_with_rids(self):
        """SGLang tensors with rids produce string seq_ids."""
        step_data: dict = {
            "input_ids": torch.tensor([10, 20, 30]),
            "positions": torch.tensor([0, 1, 2]),
            "seq_lens": torch.tensor([3]),
            "rids": ["A"],
        }

        result: TokenAlignerStepAux = _sglang_plugin.compute_step_aux(
            step_data, layout=TokenLayout.T, step=0
        )

        assert result.input_ids == [10, 20, 30]
        assert result.positions == [0, 1, 2]
        assert result.seq_lens == [3]
        assert result.seq_ids == [SGLangSeqId(rid="A")]
```
**EN:** This test method exercises with rids and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with rids 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 46-57: Run test: rids none fallback
```python
    def test_rids_none_fallback(self):
        """Missing rids results in (step, index) fallback seq_ids."""
        step_data: dict = {
            "input_ids": torch.tensor([10, 20]),
            "positions": torch.tensor([0, 1]),
            "seq_lens": torch.tensor([2]),
        }

        result: TokenAlignerStepAux = _sglang_plugin.compute_step_aux(
            step_data, layout=TokenLayout.T, step=3
        )
        assert result.seq_ids == [PositionalSeqId(step=3, seq_index=0)]
```
**EN:** This test method exercises rids none fallback and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 rids none fallback 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 59-71: Run test: multiple seqs with rids
```python
    def test_multiple_seqs_with_rids(self):
        """Multiple sequences with rids."""
        step_data: dict = {
            "input_ids": torch.tensor([10, 20, 30, 40, 50]),
            "positions": torch.tensor([0, 1, 2, 0, 1]),
            "seq_lens": torch.tensor([3, 2]),
            "rids": ["A", "B"],
        }

        result: TokenAlignerStepAux = _sglang_plugin.compute_step_aux(
            step_data, layout=TokenLayout.T, step=0
        )
        assert result.seq_ids == [SGLangSeqId(rid="A"), SGLangSeqId(rid="B")]
```
**EN:** This test method exercises multiple seqs with rids and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multiple seqs with rids 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 74-74: Define class TestNormalizeMegatron
```python
class TestNormalizeMegatron:
```
**EN:** This declaration introduces the `TestNormalizeMegatron` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNormalizeMegatron` 测试类，并说明它通过继承承担的职责。

### Lines 75-75: Document the class `TestNormalizeMegatron`
```python
    """Tests for Megatron aux tensor normalization."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestNormalizeMegatron`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestNormalizeMegatron`的设计意图。

### Lines 77-88: Run test: cu seqlens to seq lens
```python
    def test_cu_seqlens_to_seq_lens(self):
        """cu_seqlens_q is converted to seq_lens via differencing."""
        step_data: dict = {
            "input_ids": torch.tensor([10, 20, 30, 40, 50]),
            "cu_seqlens_q": torch.tensor([0, 3, 5]),
        }

        result: TokenAlignerStepAux = _megatron_plugin.compute_step_aux(
            step_data, layout=TokenLayout.T, step=0
        )

        assert result.seq_lens == [3, 2]
```
**EN:** This test method exercises cu seqlens to seq lens and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cu seqlens to seq lens 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 90-101: Run test: positions inferred thd
```python
    def test_positions_inferred_thd(self):
        """Positions inferred from seq_lens in thd layout."""
        step_data: dict = {
            "input_ids": torch.tensor([10, 20, 30, 40, 50]),
            "cu_seqlens_q": torch.tensor([0, 3, 5]),
        }

        result: TokenAlignerStepAux = _megatron_plugin.compute_step_aux(
            step_data, layout=TokenLayout.T, step=0
        )

        assert result.positions == [0, 1, 2, 0, 1]
```
**EN:** This test method exercises positions inferred thd and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 positions inferred thd 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 103-115: Run test: position ids passthrough
```python
    def test_position_ids_passthrough(self):
        """Explicit position_ids used directly instead of inference."""
        step_data: dict = {
            "input_ids": torch.tensor([10, 20, 30, 40, 50]),
            "position_ids": torch.tensor([5, 6, 7, 8, 9]),
            "cu_seqlens_q": torch.tensor([0, 5]),
        }

        result: TokenAlignerStepAux = _megatron_plugin.compute_step_aux(
            step_data, layout=TokenLayout.T, step=0
        )

        assert result.positions == [5, 6, 7, 8, 9]
```
**EN:** This test method exercises position ids passthrough and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 position ids passthrough 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 117-130: Run test: seq ids are step index tuples
```python
    def test_seq_ids_are_step_index_tuples(self):
        """Megatron seq_ids are (step, seq_index) tuples."""
        step_data: dict = {
            "input_ids": torch.tensor([10, 20, 30, 40, 50]),
            "cu_seqlens_q": torch.tensor([0, 3, 5]),
        }

        result: TokenAlignerStepAux = _megatron_plugin.compute_step_aux(
            step_data, layout=TokenLayout.T, step=5
        )
        assert result.seq_ids == [
            PositionalSeqId(step=5, seq_index=0),
            PositionalSeqId(step=5, seq_index=1),
        ]
```
**EN:** This test method exercises seq ids are step index tuples and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 seq ids are step index tuples 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 133-133: Define class TestDetectLayoutMegatron
```python
class TestDetectLayoutMegatron:
```
**EN:** This declaration introduces the `TestDetectLayoutMegatron` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDetectLayoutMegatron` 测试类，并说明它通过继承承担的职责。

### Lines 134-134: Document the class `TestDetectLayoutMegatron`
```python
    """Tests for Megatron layout detection."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestDetectLayoutMegatron`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestDetectLayoutMegatron`的设计意图。

### Lines 136-141: Run test: detect layout bshd via qkv format
```python
    def test_detect_layout_bshd_via_qkv_format(self):
        """qkv_format containing 'bshd' → layout 'bshd'."""
        raw: dict[int, dict[str, object]] = {
            0: {"qkv_format": "bshd", "input_ids": torch.tensor([1, 2, 3])}
        }
        assert _megatron_plugin.detect_layout(raw) == TokenLayout.BS
```
**EN:** This test method exercises detect layout bshd via qkv format and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 detect layout bshd via qkv format 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 143-148: Run test: detect layout bshd via ndim
```python
    def test_detect_layout_bshd_via_ndim(self):
        """2D input_ids → layout BS."""
        raw: dict[int, dict[str, object]] = {
            0: {"input_ids": torch.tensor([[1, 2], [3, 4]])}
        }
        assert _megatron_plugin.detect_layout(raw) == TokenLayout.BS
```
**EN:** This test method exercises detect layout bshd via ndim and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 detect layout bshd via ndim 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 150-155: Run test: detect layout thd via qkv format
```python
    def test_detect_layout_thd_via_qkv_format(self):
        """qkv_format 'thd' → layout T."""
        raw: dict[int, dict[str, object]] = {
            0: {"qkv_format": "thd", "input_ids": torch.tensor([1, 2, 3])}
        }
        assert _megatron_plugin.detect_layout(raw) == TokenLayout.T
```
**EN:** This test method exercises detect layout thd via qkv format and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 detect layout thd via qkv format 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 158-158: Define class TestNormalizeMegatronBSHD
```python
class TestNormalizeMegatronBSHD:
```
**EN:** This declaration introduces the `TestNormalizeMegatronBSHD` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNormalizeMegatronBSHD` 测试类，并说明它通过继承承担的职责。

### Lines 159-159: Document the class `TestNormalizeMegatronBSHD`
```python
    """Tests for Megatron BSHD normalization."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestNormalizeMegatronBSHD`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestNormalizeMegatronBSHD`的设计意图。

### Lines 161-177: Run test: basic bshd
```python
    def test_basic_bshd(self):
        """2D input_ids [2,4] → flat [8], seq_lens=[4,4], positions=[0,1,2,3,0,1,2,3]."""
        step_data: dict = {
            "input_ids": torch.tensor([[10, 20, 30, 40], [50, 60, 70, 80]]),
        }

        result: TokenAlignerStepAux = _megatron_plugin.compute_step_aux(
            step_data, layout=TokenLayout.BS, step=0
        )

        assert result.input_ids == [10, 20, 30, 40, 50, 60, 70, 80]
        assert result.seq_lens == [4, 4]
        assert result.positions == [0, 1, 2, 3, 0, 1, 2, 3]
        assert result.seq_ids == [
            PositionalSeqId(step=0, seq_index=0),
            PositionalSeqId(step=0, seq_index=1),
        ]
```
**EN:** This test method exercises basic bshd and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 basic bshd 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 179-190: Run test: bshd with cu seqlens
```python
    def test_bshd_with_cu_seqlens(self):
        """BSHD with cu_seqlens_q → uses cu_seqlens for seq_lens."""
        step_data: dict = {
            "input_ids": torch.tensor([[10, 20, 30, 40], [50, 60, 70, 80]]),
            "cu_seqlens_q": torch.tensor([0, 3, 8]),
        }

        result: TokenAlignerStepAux = _megatron_plugin.compute_step_aux(
            step_data, layout=TokenLayout.BS, step=0
        )

        assert result.seq_lens == [3, 5]
```
**EN:** This test method exercises bshd with cu seqlens and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 bshd with cu seqlens 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 192-203: Run test: bshd with position ids
```python
    def test_bshd_with_position_ids(self):
        """BSHD with 2D position_ids → flattened positions."""
        step_data: dict = {
            "input_ids": torch.tensor([[10, 20], [30, 40]]),
            "position_ids": torch.tensor([[5, 6], [10, 11]]),
        }

        result: TokenAlignerStepAux = _megatron_plugin.compute_step_aux(
            step_data, layout=TokenLayout.BS, step=0
        )

        assert result.positions == [5, 6, 10, 11]
```
**EN:** This test method exercises bshd with position ids and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 bshd with position ids 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 206-206: Define class TestInferPositions
```python
class TestInferPositions:
```
**EN:** This declaration introduces the `TestInferPositions` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestInferPositions` 测试类，并说明它通过继承承担的职责。

### Lines 207-207: Document the class `TestInferPositions`
```python
    """Tests for position inference helper."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestInferPositions`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestInferPositions`的设计意图。

### Lines 209-214: Run test: thd multiple sequences
```python
    def test_thd_multiple_sequences(self):
        """thd: positions reset to 0 for each sequence."""
        result = _infer_positions(
            seq_lens=torch.tensor([2, 3]),
        )
        assert torch.equal(result, torch.tensor([0, 1, 0, 1, 2]))
```
**EN:** This test method exercises thd multiple sequences and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 thd multiple sequences 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 217-217: Define class TestInferCpShardedDims
```python
class TestInferCpShardedDims:
```
**EN:** This declaration introduces the `TestInferCpShardedDims` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestInferCpShardedDims` 测试类，并说明它通过继承承担的职责。

### Lines 218-218: Document the class `TestInferCpShardedDims`
```python
    """Tests for infer_cp_sharded_dims on each plugin."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestInferCpShardedDims`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestInferCpShardedDims`的设计意图。

### Lines 220-223: Run test: megatron infer 1d
```python
    def test_megatron_infer_1d(self) -> None:
        """Megatron 1D → 't[cp:zigzag]'."""
        result: str = _megatron_plugin.infer_cp_sharded_dims(name="input_ids", ndim=1)
        assert result == "t[cp:zigzag]"
```
**EN:** This test method exercises megatron infer 1d and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 megatron infer 1d 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 225-228: Run test: megatron infer 2d
```python
    def test_megatron_infer_2d(self) -> None:
        """Megatron 2D → 'b s[cp:zigzag]'."""
        result: str = _megatron_plugin.infer_cp_sharded_dims(name="input_ids", ndim=2)
        assert result == "b s[cp:zigzag]"
```
**EN:** This test method exercises megatron infer 2d and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 megatron infer 2d 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 230-233: Run test: sglang infer 1d
```python
    def test_sglang_infer_1d(self) -> None:
        """SGLang 1D → 't[cp:zigzag]'."""
        result: str = _sglang_plugin.infer_cp_sharded_dims(name="input_ids", ndim=1)
        assert result == "t[cp:zigzag]"
```
**EN:** This test method exercises sglang infer 1d and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 sglang infer 1d 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 235-238: Run test: megatron infer 3d raises
```python
    def test_megatron_infer_3d_raises(self) -> None:
        """Megatron 3D raises ValueError."""
        with pytest.raises(ValueError, match="cannot infer dims"):
            _megatron_plugin.infer_cp_sharded_dims(name="input_ids", ndim=3)
```
**EN:** This test method exercises megatron infer 3d raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 megatron infer 3d raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 240-243: Run test: sglang infer 2d raises
```python
    def test_sglang_infer_2d_raises(self) -> None:
        """SGLang 2D raises ValueError."""
        with pytest.raises(ValueError, match="cannot infer dims"):
            _sglang_plugin.infer_cp_sharded_dims(name="input_ids", ndim=2)
```
**EN:** This test method exercises sglang infer 2d raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 sglang infer 2d raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 246-247: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.aux_plugins`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types`, `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pytest`, `sys`, `torch`
- Notable symbols / 关键符号: None / 无
