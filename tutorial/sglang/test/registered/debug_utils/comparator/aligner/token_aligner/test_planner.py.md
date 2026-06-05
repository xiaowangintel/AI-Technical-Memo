# test_planner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/aligner/token_aligner/test_planner.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on token aligner planner in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 token aligner planner 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Import dependencies
```python
import sys

import pytest

from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.planner import (
    _match_sequences,
    compute_token_aligner_plan,
)
from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.seq_info_builder import (
    build_seqs_info,
)
from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types import (
    PositionalSeqId,
    SeqId,
    SGLangSeqId,
    TokenAlignerGlobalAux,
    TokenAlignerSeqInfo,
    TokenAlignerSeqsInfo,
    TokenAlignerStepAux,
    TokenLocator,
)
from sglang.srt.debug_utils.comparator.dims_spec import TokenLayout
from sglang.srt.debug_utils.comparator.utils import Pair
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 26-26: Register CI metadata
```python
register_cpu_ci(est_time=30, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 29-29: Define class TestBuildTokenIndexSGLangThd
```python
class TestBuildTokenIndexSGLangThd:
```
**EN:** This declaration introduces the `TestBuildTokenIndexSGLangThd` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestBuildTokenIndexSGLangThd` 测试类，并说明它通过继承承担的职责。

### Lines 30-30: Document the class `TestBuildTokenIndexSGLangThd`
```python
    """Tests for SGLang thd token index building."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestBuildTokenIndexSGLangThd`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestBuildTokenIndexSGLangThd`的设计意图。

### Lines 32-59: Run test: single step prefill
```python
    def test_single_step_prefill(self):
        """Single prefill step with two sequences."""
        side_aux = TokenAlignerGlobalAux(
            step_auxs={
                0: TokenAlignerStepAux(
                    input_ids=[10, 20, 30, 40, 50],
                    positions=[0, 1, 2, 0, 1],
                    seq_lens=[3, 2],
                    seq_ids=[SGLangSeqId(rid="A"), SGLangSeqId(rid="B")],
                ),
            },
            framework="sglang",
            layout=TokenLayout.T,
        )

        index = build_seqs_info(side_aux)
        assert len(index.sequences) == 2

        seq_a = index.sequences[SGLangSeqId(rid="A")]
        assert seq_a.input_ids == [10, 20, 30]
        assert seq_a.positions == [0, 1, 2]
        assert seq_a.locator.steps == [0, 0, 0]
        assert seq_a.locator.token_index_in_step == [0, 1, 2]

        seq_b = index.sequences[SGLangSeqId(rid="B")]
        assert seq_b.input_ids == [40, 50]
        assert seq_b.positions == [0, 1]
        assert seq_b.locator.token_index_in_step == [3, 4]
```
**EN:** This test method exercises single step prefill and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single step prefill 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 61-92: Run test: multi step prefill decode
```python
    def test_multi_step_prefill_decode(self):
        """Prefill step followed by decode steps, sequences accumulate tokens."""
        side_aux = TokenAlignerGlobalAux(
            step_auxs={
                0: TokenAlignerStepAux(
                    input_ids=[10, 20, 30, 40, 50],
                    positions=[0, 1, 2, 0, 1],
                    seq_lens=[3, 2],
                    seq_ids=[SGLangSeqId(rid="A"), SGLangSeqId(rid="B")],
                ),
                1: TokenAlignerStepAux(
                    input_ids=[31, 51],
                    positions=[3, 2],
                    seq_lens=[1, 1],
                    seq_ids=[SGLangSeqId(rid="A"), SGLangSeqId(rid="B")],
                ),
            },
            framework="sglang",
            layout=TokenLayout.T,
        )

        index = build_seqs_info(side_aux)
        assert len(index.sequences) == 2

        seq_a = index.sequences[SGLangSeqId(rid="A")]
        assert seq_a.input_ids == [10, 20, 30, 31]
        assert seq_a.positions == [0, 1, 2, 3]
        assert seq_a.locator.steps == [0, 0, 0, 1]

        seq_b = index.sequences[SGLangSeqId(rid="B")]
        assert seq_b.input_ids == [40, 50, 51]
        assert seq_b.positions == [0, 1, 2]
```
**EN:** This test method exercises multi step prefill decode and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multi step prefill decode 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 94-116: Run test: sequence exit and join
```python
    def test_sequence_exit_and_join(self):
        """Sequence A exits, new sequence D joins with different seq_id."""
        side_aux = TokenAlignerGlobalAux(
            step_auxs={
                0: TokenAlignerStepAux(
                    input_ids=[10, 20, 30],
                    positions=[0, 1, 2],
                    seq_lens=[3],
                    seq_ids=[SGLangSeqId(rid="A")],
                ),
                1: TokenAlignerStepAux(
                    input_ids=[100, 200],
                    positions=[0, 1],
                    seq_lens=[2],
                    seq_ids=[SGLangSeqId(rid="D")],
                ),
            },
            framework="sglang",
            layout=TokenLayout.T,
        )

        index = build_seqs_info(side_aux)
        assert len(index.sequences) == 2
```
**EN:** This test method exercises sequence exit and join and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 sequence exit and join 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 118-146: Run test: different seq ids produce separate sequences
```python
    def test_different_seq_ids_produce_separate_sequences(self):
        """Different seq_ids at different steps → separate sequences."""
        side_aux = TokenAlignerGlobalAux(
            step_auxs={
                0: TokenAlignerStepAux(
                    input_ids=[10, 20],
                    positions=[0, 1],
                    seq_lens=[2],
                    seq_ids=[SGLangSeqId(rid="A")],
                ),
                1: TokenAlignerStepAux(
                    input_ids=[100, 200, 300],
                    positions=[0, 1, 2],
                    seq_lens=[3],
                    seq_ids=[SGLangSeqId(rid="D")],
                ),
            },
            framework="sglang",
            layout=TokenLayout.T,
        )

        index = build_seqs_info(side_aux)
        assert len(index.sequences) == 2

        all_input_ids = {
            seq_id: rec.input_ids for seq_id, rec in index.sequences.items()
        }
        assert [10, 20] in all_input_ids.values()
        assert [100, 200, 300] in all_input_ids.values()
```
**EN:** This test method exercises different seq ids produce separate sequences and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 different seq ids produce separate sequences 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 149-149: Define class TestBuildTokenIndexMegatronThd
```python
class TestBuildTokenIndexMegatronThd:
```
**EN:** This declaration introduces the `TestBuildTokenIndexMegatronThd` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestBuildTokenIndexMegatronThd` 测试类，并说明它通过继承承担的职责。

### Lines 150-150: Document the class `TestBuildTokenIndexMegatronThd`
```python
    """Tests for Megatron thd token index building."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestBuildTokenIndexMegatronThd`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestBuildTokenIndexMegatronThd`的设计意图。

### Lines 152-182: Run test: single step two sequences
```python
    def test_single_step_two_sequences(self):
        """Single step with two sequences in thd layout."""
        side_aux = TokenAlignerGlobalAux(
            step_auxs={
                0: TokenAlignerStepAux(
                    input_ids=[10, 20, 30, 40, 50],
                    positions=[0, 1, 2, 0, 1],
                    seq_lens=[3, 2],
                    seq_ids=[
                        PositionalSeqId(step=0, seq_index=0),
                        PositionalSeqId(step=0, seq_index=1),
                    ],
                ),
            },
            framework="megatron",
            layout=TokenLayout.T,
        )

        index = build_seqs_info(side_aux)
        assert len(index.sequences) == 2

        seq0 = index.sequences[PositionalSeqId(step=0, seq_index=0)]
        assert seq0.input_ids == [10, 20, 30]
        assert seq0.positions == [0, 1, 2]
        assert seq0.locator.steps == [0, 0, 0]
        assert seq0.locator.token_index_in_step == [0, 1, 2]

        seq1 = index.sequences[PositionalSeqId(step=0, seq_index=1)]
        assert seq1.input_ids == [40, 50]
        assert seq1.positions == [0, 1]
        assert seq1.locator.token_index_in_step == [3, 4]
```
**EN:** This test method exercises single step two sequences and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single step two sequences 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 184-220: Run test: multi step accumulation
```python
    def test_multi_step_accumulation(self):
        """Two steps with different seq_ids produce separate sequences."""
        side_aux = TokenAlignerGlobalAux(
            step_auxs={
                0: TokenAlignerStepAux(
                    input_ids=[10, 20, 30, 40],
                    positions=[0, 1, 0, 1],
                    seq_lens=[2, 2],
                    seq_ids=[
                        PositionalSeqId(step=0, seq_index=0),
                        PositionalSeqId(step=0, seq_index=1),
                    ],
                ),
                1: TokenAlignerStepAux(
                    input_ids=[50, 60, 70, 80],
                    positions=[0, 1, 0, 1],
                    seq_lens=[2, 2],
                    seq_ids=[
                        PositionalSeqId(step=1, seq_index=0),
                        PositionalSeqId(step=1, seq_index=1),
                    ],
                ),
            },
            framework="megatron",
            layout=TokenLayout.T,
        )

        index = build_seqs_info(side_aux)
        assert len(index.sequences) == 4

        seq0 = index.sequences[PositionalSeqId(step=0, seq_index=0)]
        assert seq0.input_ids == [10, 20]
        assert seq0.locator.steps == [0, 0]

        seq2 = index.sequences[PositionalSeqId(step=1, seq_index=0)]
        assert seq2.input_ids == [50, 60]
        assert seq2.locator.steps == [1, 1]
```
**EN:** This test method exercises multi step accumulation and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multi step accumulation 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 223-223: Define class TestMatchSequences
```python
class TestMatchSequences:
```
**EN:** This declaration introduces the `TestMatchSequences` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestMatchSequences` 测试类，并说明它通过继承承担的职责。

### Lines 224-224: Document the class `TestMatchSequences`
```python
    """Tests for _match_sequences: for each y, find matching x."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestMatchSequences`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestMatchSequences`的设计意图。

### Lines 226-233: Run test: exact match simple
```python
    def test_exact_match_simple(self):
        """Identical input_ids on both sides → all matched."""
        matched = _match_seqs(
            x={0: (10, 20, 30), 1: (40, 50)},
            y={0: (10, 20, 30), 1: (40, 50)},
        )
        S = _int_to_seq_id
        assert _matched_ids(matched) == {(S(0), S(0)), (S(1), S(1))}
```
**EN:** This test method exercises exact match simple and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 exact match simple 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 235-242: Run test: exact match different order
```python
    def test_exact_match_different_order(self):
        """Sequences in different order still match by content."""
        matched = _match_seqs(
            x={0: (10, 20), 1: (40, 50)},
            y={0: (40, 50), 1: (10, 20)},
        )
        S = _int_to_seq_id
        assert _matched_ids(matched) == {(S(1), S(0)), (S(0), S(1))}
```
**EN:** This test method exercises exact match different order and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 exact match different order 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 244-251: Run test: exact match different seq ids
```python
    def test_exact_match_different_seq_ids(self):
        """Seq IDs don't need to correspond — matching is by content."""
        matched = _match_seqs(
            x={5: (10, 20), 9: (30, 40)},
            y={2: (30, 40), 7: (10, 20)},
        )
        S = _int_to_seq_id
        assert _matched_ids(matched) == {(S(9), S(2)), (S(5), S(7))}
```
**EN:** This test method exercises exact match different seq ids and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 exact match different seq ids 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 253-259: Run test: no match
```python
    def test_no_match(self):
        """Completely different input_ids → no matches."""
        matched = _match_seqs(
            x={0: (10, 20)},
            y={0: (99, 88)},
        )
        assert matched == []
```
**EN:** This test method exercises no match and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no match 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 261-265: Run test: empty sides
```python
    def test_empty_sides(self):
        """Empty x or y → no matches."""
        assert _match_seqs(x={}, y={0: (10,)}) == []
        assert _match_seqs(x={0: (10,)}, y={}) == []
        assert _match_seqs(x={}, y={}) == []
```
**EN:** This test method exercises empty sides and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty sides 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 267-274: Run test: x has more sequences
```python
    def test_x_has_more_sequences(self):
        """Extra x sequences are ignored (no y needs them)."""
        matched = _match_seqs(
            x={0: (10, 20), 1: (30, 40), 2: (50, 60)},
            y={0: (30, 40)},
        )
        S = _int_to_seq_id
        assert _matched_ids(matched) == {(S(1), S(0))}
```
**EN:** This test method exercises x has more sequences and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 x has more sequences 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 276-283: Run test: y has more sequences
```python
    def test_y_has_more_sequences(self):
        """Extra y sequences remain unmatched."""
        matched = _match_seqs(
            x={0: (10, 20)},
            y={0: (10, 20), 1: (30, 40), 2: (50, 60)},
        )
        S = _int_to_seq_id
        assert _matched_ids(matched) == {(S(0), S(0))}
```
**EN:** This test method exercises y has more sequences and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 y has more sequences 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 285-291: Run test: one x not reused
```python
    def test_one_x_not_reused(self):
        """Each x can only be claimed once, even if multiple y want it."""
        matched = _match_seqs(
            x={0: (10, 20)},
            y={0: (10, 20), 1: (10, 20)},
        )
        assert len(matched) == 1
```
**EN:** This test method exercises one x not reused and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 one x not reused 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 293-304: Run test: ambiguous all matched
```python
    def test_ambiguous_all_matched(self):
        """Multiple identical sequences on both sides → all paired (greedy 1:1)."""
        matched = _match_seqs(
            x={0: (10, 20), 1: (10, 20), 2: (10, 20)},
            y={0: (10, 20), 1: (10, 20), 2: (10, 20)},
        )
        S = _int_to_seq_id
        assert len(matched) == 3
        x_ids = {m[0] for m in matched}
        y_ids = {m[1] for m in matched}
        assert x_ids == {S(0), S(1), S(2)}
        assert y_ids == {S(0), S(1), S(2)}
```
**EN:** This test method exercises ambiguous all matched and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 ambiguous all matched 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 306-313: Run test: prefix x shorter
```python
    def test_prefix_x_shorter(self):
        """x has fewer tokens (prefix of y) → prefix match."""
        matched = _match_seqs(
            x={0: (10, 20)},
            y={0: (10, 20, 30)},
        )
        S = _int_to_seq_id
        assert _matched_ids(matched) == {(S(0), S(0))}
```
**EN:** This test method exercises prefix x shorter and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 prefix x shorter 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 315-322: Run test: prefix y shorter
```python
    def test_prefix_y_shorter(self):
        """y has fewer tokens (prefix of x) → prefix match."""
        matched = _match_seqs(
            x={0: (10, 20, 30)},
            y={0: (10, 20)},
        )
        S = _int_to_seq_id
        assert _matched_ids(matched) == {(S(0), S(0))}
```
**EN:** This test method exercises prefix y shorter and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 prefix y shorter 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 324-331: Run test: prefix picks longest
```python
    def test_prefix_picks_longest(self):
        """Among multiple prefix candidates, picks the one with longest overlap."""
        matched = _match_seqs(
            x={0: (10,), 1: (10, 20, 30)},
            y={0: (10, 20, 30, 40)},
        )
        S = _int_to_seq_id
        assert _matched_ids(matched) == {(S(1), S(0))}
```
**EN:** This test method exercises prefix picks longest and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 prefix picks longest 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 333-340: Run test: exact preferred over prefix
```python
    def test_exact_preferred_over_prefix(self):
        """Exact match is tried first, even if a longer prefix candidate exists."""
        matched = _match_seqs(
            x={0: (10, 20), 1: (10, 20, 30)},
            y={0: (10, 20)},
        )
        S = _int_to_seq_id
        assert _matched_ids(matched) == {(S(0), S(0))}
```
**EN:** This test method exercises exact preferred over prefix and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 exact preferred over prefix 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 342-352: Run test: prefix fallback after exact
```python
    def test_prefix_fallback_after_exact(self):
        """Exact matches consume sequences, remaining use prefix match."""
        matched = _match_seqs(
            x={0: (10, 20, 30), 1: (40, 50)},
            y={0: (10, 20, 30), 1: (40, 50, 60)},
        )
        S = _int_to_seq_id
        assert len(matched) == 2
        matched_set = _matched_ids(matched)
        assert (S(0), S(0)) in matched_set
        assert (S(1), S(1)) in matched_set
```
**EN:** This test method exercises prefix fallback after exact and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 prefix fallback after exact 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 354-361: Run test: single token sequences
```python
    def test_single_token_sequences(self):
        """Single-token sequences can match."""
        matched = _match_seqs(
            x={0: (42,)},
            y={0: (42,)},
        )
        S = _int_to_seq_id
        assert _matched_ids(matched) == {(S(0), S(0))}
```
**EN:** This test method exercises single token sequences and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single token sequences 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 363-369: Run test: no partial overlap without prefix
```python
    def test_no_partial_overlap_without_prefix(self):
        """Overlapping content that isn't a prefix → no match."""
        matched = _match_seqs(
            x={0: (10, 20, 30)},
            y={0: (20, 30, 40)},
        )
        assert matched == []
```
**EN:** This test method exercises no partial overlap without prefix and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no partial overlap without prefix 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 372-372: Define class TestComputeAlignmentPlanCrossFramework
```python
class TestComputeAlignmentPlanCrossFramework:
```
**EN:** This declaration introduces the `TestComputeAlignmentPlanCrossFramework` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestComputeAlignmentPlanCrossFramework` 测试类，并说明它通过继承承担的职责。

### Lines 373-373: Document the class `TestComputeAlignmentPlanCrossFramework`
```python
    """Tests for alignment plan across different frameworks and layouts."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestComputeAlignmentPlanCrossFramework`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestComputeAlignmentPlanCrossFramework`的设计意图。

### Lines 375-412: Run test: thd vs thd different step splits
```python
    def test_thd_vs_thd_different_step_splits(self):
        """Two thd sides with same tokens but different step distributions."""
        side_aux_a = TokenAlignerGlobalAux(
            step_auxs={
                0: TokenAlignerStepAux(
                    input_ids=[10, 20],
                    positions=[0, 1],
                    seq_lens=[2],
                    seq_ids=[SGLangSeqId(rid="X")],
                ),
                1: TokenAlignerStepAux(
                    input_ids=[30],
                    positions=[2],
                    seq_lens=[1],
                    seq_ids=[SGLangSeqId(rid="X")],
                ),
            },
            framework="sglang",
            layout=TokenLayout.T,
        )
        side_aux_b = TokenAlignerGlobalAux(
            step_auxs={
                0: TokenAlignerStepAux(
                    input_ids=[10, 20, 30],
                    positions=[0, 1, 2],
                    seq_lens=[3],
                    seq_ids=[SGLangSeqId(rid="X")],
                ),
            },
            framework="sglang",
            layout=TokenLayout.T,
        )

        index_a = build_seqs_info(side_aux_a)
        index_b = build_seqs_info(side_aux_b)

        plan = compute_token_aligner_plan(seqs_info_pair=Pair(x=index_a, y=index_b))
        assert len(plan.locators.x.steps) == 3
```
**EN:** This test method exercises thd vs thd different step splits and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 thd vs thd different step splits 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 414-455: Run test: sglang vs megatron thd
```python
    def test_sglang_vs_megatron_thd(self):
        """SGLang multi-step thd aligned with Megatron single-step thd."""
        side_aux_a = TokenAlignerGlobalAux(
            step_auxs={
                0: TokenAlignerStepAux(
                    input_ids=[10, 20, 30, 40, 50],
                    positions=[0, 1, 2, 0, 1],
                    seq_lens=[3, 2],
                    seq_ids=[SGLangSeqId(rid="A"), SGLangSeqId(rid="B")],
                ),
                1: TokenAlignerStepAux(
                    input_ids=[31, 51],
                    positions=[3, 2],
                    seq_lens=[1, 1],
                    seq_ids=[SGLangSeqId(rid="A"), SGLangSeqId(rid="B")],
                ),
            },
            framework="sglang",
            layout=TokenLayout.T,
        )
        side_aux_b = TokenAlignerGlobalAux(
            step_auxs={
                0: TokenAlignerStepAux(
                    input_ids=[10, 20, 30, 31, 40, 50, 51],
                    positions=[0, 1, 2, 3, 0, 1, 2],
                    seq_lens=[4, 3],
                    seq_ids=[
                        PositionalSeqId(step=0, seq_index=0),
                        PositionalSeqId(step=0, seq_index=1),
                    ],
                ),
            },
            framework="megatron",
            layout=TokenLayout.T,
        )

        index_a = build_seqs_info(side_aux_a)
        index_b = build_seqs_info(side_aux_b)

        plan = compute_token_aligner_plan(seqs_info_pair=Pair(x=index_a, y=index_b))

        assert len(plan.locators.x.steps) == 7
```
**EN:** This test method exercises sglang vs megatron thd and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 sglang vs megatron thd 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 457-506: Run test: cross layout sglang thd vs megatron bshd
```python
    def test_cross_layout_sglang_thd_vs_megatron_bshd(self):
        """SGLang THD vs Megatron BSHD end-to-end alignment via planner.

        SGLang side: two sequences [10,20,30] and [40,50] across 2 steps.
        Megatron BSHD side: same tokens as 2 batch slots [10,20,30,PAD] and [40,50,PAD,PAD],
        where PAD tokens (99) are included because BSHD treats whole padded row as one seq.
        Planner should match by prefix and align the common 5 tokens.
        """
        side_sglang = TokenAlignerGlobalAux(
            step_auxs={
                0: TokenAlignerStepAux(
                    input_ids=[10, 20, 30, 40, 50],
                    positions=[0, 1, 2, 0, 1],
                    seq_lens=[3, 2],
                    seq_ids=[SGLangSeqId(rid="A"), SGLangSeqId(rid="B")],
                ),
            },
            framework="sglang",
            layout=TokenLayout.T,
        )

        side_megatron_bshd = TokenAlignerGlobalAux(
            step_auxs={
                # BSHD normalized: flat [B*S] with each batch slot as one seq
                0: TokenAlignerStepAux(
                    input_ids=[10, 20, 30, 99, 40, 50, 99, 99],
                    positions=[0, 1, 2, 3, 0, 1, 2, 3],
                    seq_lens=[4, 4],
                    seq_ids=[
                        PositionalSeqId(step=0, seq_index=0),
                        PositionalSeqId(step=0, seq_index=1),
                    ],
                ),
            },
            framework="megatron",
            layout=TokenLayout.BS,
        )

        index_sglang = build_seqs_info(side_sglang)
        index_megatron = build_seqs_info(side_megatron_bshd)

        plan = compute_token_aligner_plan(
            seqs_info_pair=Pair(x=index_sglang, y=index_megatron)
        )

        # Seq A: [10,20,30] matches prefix of [10,20,30,99] → 3 tokens
        # Seq B: [40,50] matches prefix of [40,50,99,99] → 2 tokens
        assert len(plan.locators.x.steps) == 5
        assert plan.layouts.x == TokenLayout.T
        assert plan.layouts.y == TokenLayout.BS
```
**EN:** This test method exercises cross layout sglang thd vs megatron bshd and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cross layout sglang thd vs megatron bshd 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 514-516: Define helper: int to seq id
```python
def _int_to_seq_id(k: int) -> SeqId:
    """Convert an int key to a SeqId for test convenience."""
    return SGLangSeqId(rid=str(k))
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 519-536: Define helper: make index
```python
def _make_index(
    *,
    sequences: dict[int, tuple[int, ...]],
    layout: TokenLayout = TokenLayout.T,
) -> TokenAlignerSeqsInfo:
    """Create a TokenAlignerSeqsInfo from simplified input_ids-only specification."""
    records: dict[SeqId, TokenAlignerSeqInfo] = {}
    for k, input_ids in sequences.items():
        num_tokens = len(input_ids)
        records[_int_to_seq_id(k)] = TokenAlignerSeqInfo(
            input_ids=list(input_ids),
            positions=list(range(num_tokens)),
            locator=TokenLocator(
                steps=[0] * num_tokens,
                token_index_in_step=list(range(num_tokens)),
            ),
        )
    return TokenAlignerSeqsInfo(sequences=records, layout=layout)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 539-554: Define helper: make seq info dict
```python
def _make_seq_info_dict(
    sequences: dict[int, tuple[int, ...]],
) -> dict[SeqId, TokenAlignerSeqInfo]:
    """Create a dict of TokenAlignerSeqInfo from {int_key: input_ids_tuple}."""
    result: dict[SeqId, TokenAlignerSeqInfo] = {}
    for k, input_ids in sequences.items():
        num_tokens = len(input_ids)
        result[_int_to_seq_id(k)] = TokenAlignerSeqInfo(
            input_ids=list(input_ids),
            positions=list(range(num_tokens)),
            locator=TokenLocator(
                steps=[0] * num_tokens,
                token_index_in_step=list(range(num_tokens)),
            ),
        )
    return result
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 557-565: Define helper: match seqs
```python
def _match_seqs(
    *,
    x: dict[int, tuple[int, ...]],
    y: dict[int, tuple[int, ...]],
) -> list[tuple[SeqId, SeqId]]:
    """Shorthand: build SeqInfo dicts and call _match_sequences."""
    return _match_sequences(
        seqs=Pair(x=_make_seq_info_dict(x), y=_make_seq_info_dict(y))
    )
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 568-570: Define helper: matched ids
```python
def _matched_ids(matched: list[tuple[SeqId, SeqId]]) -> set[tuple[SeqId, SeqId]]:
    """Convert matched pairs list to set for order-independent comparison."""
    return set(matched)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 573-574: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.planner`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.seq_info_builder`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types`, `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.srt.debug_utils.comparator.utils`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pytest`, `sys`
- Notable symbols / 关键符号: None / 无
