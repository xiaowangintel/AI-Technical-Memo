# test_executor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/aligner/reorderer/test_executor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on reorderer executor in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 reorderer executor 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Import dependencies
```python
import sys

import pytest
import torch

from sglang.srt.debug_utils.comparator.aligner.reorderer.executor import (
    _reorder_zigzag_to_natural,
    _reorder_zigzag_to_natural_thd,
    execute_reorderer_plan,
)
from sglang.srt.debug_utils.comparator.aligner.reorderer.types import (
    ReordererPlan,
    ZigzagToNaturalThdParams,
)
from sglang.srt.debug_utils.comparator.aligner.unsharder.executor import (
    execute_unsharder_plan,
)
from sglang.srt.debug_utils.comparator.aligner.unsharder.types import (
    CpThdConcatParams,
    UnsharderPlan,
)
from sglang.srt.debug_utils.comparator.dims_spec import ParallelAxis
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 25-25: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 28-35: Define helper: zigzag order
```python
def _zigzag_order(cp_size: int) -> list[int]:
    """Build zigzag interleaving order for 2*cp_size chunks."""
    order: list[int] = []
    num_chunks: int = cp_size * 2
    for i in range(cp_size):
        order.append(i)
        order.append(num_chunks - 1 - i)
    return order
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 38-47: Define helper: zigzag split seq
```python
def _zigzag_split_seq(seq_natural: torch.Tensor, *, cp_size: int) -> list[torch.Tensor]:
    """Split a natural-order seq into per-rank zigzag segments.

    Returns: list of per-rank tensors, where rank_i holds chunks assigned by zigzag.
    """
    num_chunks: int = cp_size * 2
    chunks: list[torch.Tensor] = list(seq_natural.chunk(num_chunks, dim=0))
    order: list[int] = _zigzag_order(cp_size)
    zigzagged: torch.Tensor = torch.cat([chunks[i] for i in order], dim=0)
    return list(zigzagged.chunk(cp_size, dim=0))
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 50-50: Define class TestZigzagToNatural
```python
class TestZigzagToNatural:
```
**EN:** This declaration introduces the `TestZigzagToNatural` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestZigzagToNatural` 测试类，并说明它通过继承承担的职责。

### Lines 51-60: Run test: zigzag to natural cp2
```python
    def test_zigzag_to_natural_cp2(self) -> None:
        """cp_size=2: zigzag order [0,3,1,2] -> natural [0,1,2,3]."""
        natural = torch.arange(24).reshape(4, 6)
        chunks = list(natural.chunk(4, dim=0))

        zigzag_order: list[int] = [0, 3, 1, 2]
        zigzagged = torch.cat([chunks[i] for i in zigzag_order], dim=0)

        result = _reorder_zigzag_to_natural(zigzagged, dim=0, cp_size=2)
        assert torch.equal(result, natural)
```
**EN:** This test method exercises zigzag to natural cp2 and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 zigzag to natural cp2 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 62-71: Run test: zigzag to natural cp3
```python
    def test_zigzag_to_natural_cp3(self) -> None:
        """cp_size=3: zigzag 162534 -> natural 123456 (1-indexed)."""
        natural = torch.arange(60).reshape(6, 10)
        chunks = list(natural.chunk(6, dim=0))

        zigzag_order: list[int] = [0, 5, 1, 4, 2, 3]
        zigzagged = torch.cat([chunks[i] for i in zigzag_order], dim=0)

        result = _reorder_zigzag_to_natural(zigzagged, dim=0, cp_size=3)
        assert torch.equal(result, natural)
```
**EN:** This test method exercises zigzag to natural cp3 and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 zigzag to natural cp3 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 73-82: Run test: zigzag to natural arbitrary dim
```python
    def test_zigzag_to_natural_arbitrary_dim(self) -> None:
        """Reorder along dim=1 instead of dim=0."""
        natural = torch.arange(48).reshape(3, 4, 4)
        chunks = list(natural.chunk(4, dim=1))

        zigzag_order: list[int] = [0, 3, 1, 2]
        zigzagged = torch.cat([chunks[i] for i in zigzag_order], dim=1)

        result = _reorder_zigzag_to_natural(zigzagged, dim=1, cp_size=2)
        assert torch.equal(result, natural)
```
**EN:** This test method exercises zigzag to natural arbitrary dim and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 zigzag to natural arbitrary dim 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 85-85: Define class TestZigzagToNaturalThd
```python
class TestZigzagToNaturalThd:
```
**EN:** This declaration introduces the `TestZigzagToNaturalThd` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestZigzagToNaturalThd` 测试类，并说明它通过继承承担的职责。

### Lines 86-95: Run test: single seq
```python
    def test_single_seq(self) -> None:
        """Single seq THD reorder: equivalent to whole-tensor reorder."""
        natural = torch.arange(100)
        zigzag_ranks: list[torch.Tensor] = _zigzag_split_seq(natural, cp_size=2)
        zigzagged: torch.Tensor = torch.cat(zigzag_ranks, dim=0)

        result = _reorder_zigzag_to_natural_thd(
            zigzagged, dim=0, cp_size=2, seq_lens=[100]
        )
        assert torch.equal(result, natural)
```
**EN:** This test method exercises single seq and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single seq 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 97-115: Run test: multi seq
```python
    def test_multi_seq(self) -> None:
        """Two seqs of different lengths, each independently reordered."""
        seq_a_natural = torch.arange(100)
        seq_b_natural = torch.arange(100, 164)

        seq_a_zigzag: torch.Tensor = torch.cat(
            _zigzag_split_seq(seq_a_natural, cp_size=2), dim=0
        )
        seq_b_zigzag: torch.Tensor = torch.cat(
            _zigzag_split_seq(seq_b_natural, cp_size=2), dim=0
        )

        combined_zigzag: torch.Tensor = torch.cat([seq_a_zigzag, seq_b_zigzag], dim=0)
        result = _reorder_zigzag_to_natural_thd(
            combined_zigzag, dim=0, cp_size=2, seq_lens=[100, 64]
        )

        expected: torch.Tensor = torch.cat([seq_a_natural, seq_b_natural], dim=0)
        assert torch.equal(result, expected)
```
**EN:** This test method exercises multi seq and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multi seq 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 117-132: Run test: with tail pad
```python
    def test_with_tail_pad(self) -> None:
        """THD reorder with trailing global padding preserved unchanged."""
        seq_natural = torch.arange(100)
        pad: torch.Tensor = torch.full((56,), fill_value=-1)

        seq_zigzag: torch.Tensor = torch.cat(
            _zigzag_split_seq(seq_natural, cp_size=2), dim=0
        )
        combined: torch.Tensor = torch.cat([seq_zigzag, pad], dim=0)

        result = _reorder_zigzag_to_natural_thd(
            combined, dim=0, cp_size=2, seq_lens=[100]
        )

        assert torch.equal(result[:100], seq_natural)
        assert torch.equal(result[100:], pad)
```
**EN:** This test method exercises with tail pad and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with tail pad 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 134-147: Run test: with hidden dim
```python
    def test_with_hidden_dim(self) -> None:
        """THD reorder with trailing hidden dimension (shape [T, H])."""
        torch.manual_seed(42)
        hidden: int = 8
        seq_natural = torch.randn(100, hidden)

        seq_zigzag: torch.Tensor = torch.cat(
            _zigzag_split_seq(seq_natural, cp_size=2), dim=0
        )

        result = _reorder_zigzag_to_natural_thd(
            seq_zigzag, dim=0, cp_size=2, seq_lens=[100]
        )
        assert torch.equal(result, seq_natural)
```
**EN:** This test method exercises with hidden dim and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with hidden dim 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 149-172: Run test: with leading batch dim
```python
    def test_with_leading_batch_dim(self) -> None:
        """THD reorder with leading batch dim: shape [B, T, H], t is dim=1."""
        torch.manual_seed(42)
        batch: int = 2
        hidden: int = 4
        seq_a_natural = torch.randn(batch, 100, hidden)
        seq_b_natural = torch.randn(batch, 64, hidden)
        full_natural: torch.Tensor = torch.cat([seq_a_natural, seq_b_natural], dim=1)

        # Zigzag each seq along dim=1
        def zigzag_along_dim1(t: torch.Tensor) -> torch.Tensor:
            num_chunks: int = 2 * 2  # cp_size=2
            chunks: list[torch.Tensor] = list(t.chunk(num_chunks, dim=1))
            order: list[int] = [0, 3, 1, 2]  # zigzag for cp_size=2
            return torch.cat([chunks[i] for i in order], dim=1)

        seq_a_zigzag: torch.Tensor = zigzag_along_dim1(seq_a_natural)
        seq_b_zigzag: torch.Tensor = zigzag_along_dim1(seq_b_natural)
        combined_zigzag: torch.Tensor = torch.cat([seq_a_zigzag, seq_b_zigzag], dim=1)

        result = _reorder_zigzag_to_natural_thd(
            combined_zigzag, dim=1, cp_size=2, seq_lens=[100, 64]
        )
        assert torch.equal(result, full_natural)
```
**EN:** This test method exercises with leading batch dim and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with leading batch dim 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 175-175: Define class TestThdCpZigzagE2E
```python
class TestThdCpZigzagE2E:
```
**EN:** This declaration introduces the `TestThdCpZigzagE2E` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestThdCpZigzagE2E` 测试类，并说明它通过继承承担的职责。

### Lines 176-193: Document the class `TestThdCpZigzagE2E`
```python
    """End-to-end unshard + reorder tests for THD CP zigzag format.

    Simulates Miles/Megatron forward data splitting:

    cp_size=2, batch with 2 seqs: seqA(100 tokens), seqB(61→pad to 64)

    Forward:
      seqA(100): chunk_size=25, 4 chunks → rank0=[chunk0+chunk3](50), rank1=[chunk1+chunk2](50)
      seqB(64):  chunk_size=16, 4 chunks → rank0=[chunk0+chunk3](32), rank1=[chunk1+chunk2](32)
      global pad → align to 128
      rank0: [seqA_r0(50) | seqB_r0(32) | pad(46)] = 128 tokens
      rank1: [seqA_r1(50) | seqB_r1(32) | pad(46)] = 128 tokens
      global cu_seqlens: [0, 100, 164, 256]

    Comparator undo:
      Step 1 THD unshard: per-seq cross-rank concat → [seqA_zigzag(100) | seqB_zigzag(64) | pad(92)]
      Step 2 THD reorder: per-seq zigzag→natural → [seqA_natural(100) | seqB_natural(64) | pad(92)]
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestThdCpZigzagE2E`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestThdCpZigzagE2E`的设计意图。

### Lines 195-245: Run test: thd cp2 two seqs
```python
    def test_thd_cp2_two_seqs(self) -> None:
        """cp_size=2, 2 seqs (100, 61→64) + global pad."""
        torch.manual_seed(42)
        cp_size: int = 2
        total_per_rank: int = 128

        seq_a_natural = torch.randn(100)
        seq_b_natural_raw = torch.randn(61)
        seq_b_padded = torch.cat([seq_b_natural_raw, torch.zeros(3)])  # pad 61→64

        seq_a_ranks: list[torch.Tensor] = _zigzag_split_seq(
            seq_a_natural, cp_size=cp_size
        )
        seq_b_ranks: list[torch.Tensor] = _zigzag_split_seq(
            seq_b_padded, cp_size=cp_size
        )

        # Build per-rank tensors: [seqA_r | seqB_r | pad_r]
        rank_tensors: list[torch.Tensor] = []
        for rank in range(cp_size):
            used: int = seq_a_ranks[rank].shape[0] + seq_b_ranks[rank].shape[0]
            pad_len: int = total_per_rank - used
            rank_tensor: torch.Tensor = torch.cat(
                [seq_a_ranks[rank], seq_b_ranks[rank], torch.zeros(pad_len)]
            ).refine_names("t")
            rank_tensors.append(rank_tensor)

        # Step 1: THD unshard
        seq_lens_per_rank: list[int] = [50, 32, 46]
        unshard_plan = UnsharderPlan(
            axis=ParallelAxis.CP,
            params=CpThdConcatParams(dim_name="t", seq_lens_per_rank=seq_lens_per_rank),
            groups=[[0, 1]],
        )
        unsharder_result = execute_unsharder_plan(unshard_plan, rank_tensors)
        unsharded: list[torch.Tensor] = unsharder_result.tensors
        assert len(unsharded) == 1

        # Step 2: THD reorder
        reorder_seq_lens: list[int] = [s * cp_size for s in seq_lens_per_rank]
        reorder_plan = ReordererPlan(
            params=ZigzagToNaturalThdParams(
                dim_name="t", cp_size=cp_size, seq_lens=reorder_seq_lens
            )
        )
        reordered: list[torch.Tensor] = execute_reorderer_plan(reorder_plan, unsharded)
        assert len(reordered) == 1

        result: torch.Tensor = reordered[0].rename(None)
        assert torch.equal(result[:100], seq_a_natural)
        assert torch.equal(result[100:164], seq_b_padded)
```
**EN:** This test method exercises thd cp2 two seqs and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 thd cp2 two seqs 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 247-280: Run test: thd cp3 single seq
```python
    def test_thd_cp3_single_seq(self) -> None:
        """cp_size=3, single seq (120 tokens)."""
        torch.manual_seed(42)
        cp_size: int = 3
        seq_natural = torch.randn(120)

        seq_ranks: list[torch.Tensor] = _zigzag_split_seq(seq_natural, cp_size=cp_size)

        rank_tensors: list[torch.Tensor] = [t.refine_names("t") for t in seq_ranks]

        # Step 1: THD unshard
        seq_len_per_rank: int = 120 // cp_size  # 40
        unshard_plan = UnsharderPlan(
            axis=ParallelAxis.CP,
            params=CpThdConcatParams(
                dim_name="t", seq_lens_per_rank=[seq_len_per_rank]
            ),
            groups=[list(range(cp_size))],
        )
        unsharder_result = execute_unsharder_plan(unshard_plan, rank_tensors)
        unsharded: list[torch.Tensor] = unsharder_result.tensors
        assert len(unsharded) == 1

        # Step 2: THD reorder
        reorder_plan = ReordererPlan(
            params=ZigzagToNaturalThdParams(
                dim_name="t", cp_size=cp_size, seq_lens=[120]
            )
        )
        reordered: list[torch.Tensor] = execute_reorderer_plan(reorder_plan, unsharded)
        assert len(reordered) == 1

        result: torch.Tensor = reordered[0].rename(None)
        assert torch.equal(result, seq_natural)
```
**EN:** This test method exercises thd cp3 single seq and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 thd cp3 single seq 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 283-284: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.aligner.reorderer.executor`, `sglang.srt.debug_utils.comparator.aligner.reorderer.types`, `sglang.srt.debug_utils.comparator.aligner.unsharder.executor`, `sglang.srt.debug_utils.comparator.aligner.unsharder.types`, `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pytest`, `sys`, `torch`
- Notable symbols / 关键符号: None / 无
