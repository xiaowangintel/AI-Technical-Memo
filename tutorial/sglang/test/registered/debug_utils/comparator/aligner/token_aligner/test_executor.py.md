# test_executor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/aligner/token_aligner/test_executor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on token aligner executor in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 token aligner executor 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-26: Import dependencies
```python
from __future__ import annotations

import sys

import pytest
import torch

from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.executor import (
    execute_token_aligner,
)
from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.planner import (
    compute_token_aligner_plan,
)
from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.seq_info_builder import (
    build_seqs_info,
)
from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types import (
    SGLangSeqId,
    TokenAlignerGlobalAux,
    TokenAlignerPlan,
    TokenAlignerStepAux,
    TokenLocator,
)
from sglang.srt.debug_utils.comparator.dims_spec import TokenLayout
from sglang.srt.debug_utils.comparator.utils import Pair
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 28-28: Register CI metadata
```python
register_cpu_ci(est_time=15, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 31-32: Define helper: named
```python
def _named(tensor: torch.Tensor, names: list[str]) -> torch.Tensor:
    return tensor.refine_names(*names)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 35-35: Define class TestExecuteAlignment
```python
class TestExecuteAlignment:
```
**EN:** This declaration introduces the `TestExecuteAlignment` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestExecuteAlignment` 测试类，并说明它通过继承承担的职责。

### Lines 36-36: Document the class `TestExecuteAlignment`
```python
    """Tests for token alignment execution."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestExecuteAlignment`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestExecuteAlignment`的设计意图。

### Lines 38-72: Run test: thd vs thd identity
```python
    def test_thd_vs_thd_identity(self):
        """Two identical thd sides produce element-wise equal aligned tensors."""
        torch.manual_seed(42)
        hidden_step0 = torch.randn(5, 8).refine_names("t", "h")
        hidden_step1 = torch.randn(2, 8).refine_names("t", "h")

        aux = TokenAlignerStepAux(
            input_ids=[10, 20, 30, 40, 50],
            positions=[0, 1, 2, 0, 1],
            seq_lens=[3, 2],
            seq_ids=[SGLangSeqId(rid="A"), SGLangSeqId(rid="B")],
        )
        aux_step1 = TokenAlignerStepAux(
            input_ids=[31, 51],
            positions=[3, 2],
            seq_lens=[1, 1],
            seq_ids=[SGLangSeqId(rid="A"), SGLangSeqId(rid="B")],
        )

        side_aux = TokenAlignerGlobalAux(
            step_auxs={0: aux, 1: aux_step1},
            framework="sglang",
            layout=TokenLayout.T,
        )

        index = build_seqs_info(side_aux)
        plan = compute_token_aligner_plan(seqs_info_pair=Pair(x=index, y=index))

        tensors = {0: hidden_step0, 1: hidden_step1}
        aligned: Pair[torch.Tensor] = execute_token_aligner(
            plan=plan, tensor_of_step_pair=Pair(x=tensors, y=tensors)
        )

        assert torch.equal(aligned.x, aligned.y)
        assert aligned.x.shape[0] == len(plan.locators.x.steps)
```
**EN:** This test method exercises thd vs thd identity and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 thd vs thd identity 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 74-94: Run test: zero matched tokens
```python
    def test_zero_matched_tokens(self):
        """Empty TokenAlignerPlan (no matched tokens) returns shape[0]==0 without crash."""
        torch.manual_seed(42)

        plan = TokenAlignerPlan(
            locators=Pair(
                x=TokenLocator(steps=[], token_index_in_step=[]),
                y=TokenLocator(steps=[], token_index_in_step=[]),
            ),
            layouts=Pair(x=TokenLayout.T, y=TokenLayout.T),
        )

        tensors = {0: torch.randn(5, 8).refine_names("t", "h")}
        aligned: Pair[torch.Tensor] = execute_token_aligner(
            plan=plan, tensor_of_step_pair=Pair(x=tensors, y=tensors)
        )

        assert aligned.x.shape[0] == 0
        assert aligned.y.shape[0] == 0
        assert aligned.x.shape[1:] == (8,)
        assert aligned.y.shape[1:] == (8,)
```
**EN:** This test method exercises zero matched tokens and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 zero matched tokens 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 97-97: Define class TestTokenDim
```python
class TestTokenDim:
```
**EN:** This declaration introduces the `TestTokenDim` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestTokenDim` 测试类，并说明它通过继承承担的职责。

### Lines 98-98: Document the class `TestTokenDim`
```python
    """Tests for non-zero token_dim support."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestTokenDim`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestTokenDim`的设计意图。

### Lines 100-108: Define helper: make simple plan
```python
    def _make_simple_plan(self, *, num_tokens: int) -> TokenAlignerPlan:
        locator = TokenLocator(
            steps=[0] * num_tokens,
            token_index_in_step=list(range(num_tokens)),
        )
        return TokenAlignerPlan(
            locators=Pair(x=locator, y=locator),
            layouts=Pair(x=TokenLayout.T, y=TokenLayout.T),
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestTokenDim` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestTokenDim` 内部调用，从而让场景结构更清晰。

### Lines 110-128: Run test: token dim nonzero
```python
    def test_token_dim_nonzero(self) -> None:
        """tensor shape [3, 5, 8], token_dim=1 -> token dim stays at dim 1."""
        torch.manual_seed(42)
        tensor: torch.Tensor = _named(torch.randn(3, 5, 8), ["a", "t", "h"])
        plan: TokenAlignerPlan = self._make_simple_plan(num_tokens=5)

        tensors: dict[int, torch.Tensor] = {0: tensor}
        aligned: Pair[torch.Tensor] = execute_token_aligner(
            plan=plan,
            tensor_of_step_pair=Pair(x=tensors, y=tensors),
        )

        assert aligned.x.shape == (3, 5, 8)
        assert torch.equal(aligned.x, aligned.y)
        plain: torch.Tensor = tensor.rename(None)
        for i in range(5):
            assert torch.equal(
                aligned.x.select(dim=1, index=i), plain.select(dim=1, index=i)
            )
```
**EN:** This test method exercises token dim nonzero and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 token dim nonzero 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 130-147: Run test: token dim last
```python
    def test_token_dim_last(self) -> None:
        """tensor shape [3, 8, 5], token_dim=2 -> token dim stays at dim 2."""
        torch.manual_seed(42)
        tensor: torch.Tensor = _named(torch.randn(3, 8, 5), ["a", "h", "t"])
        plan: TokenAlignerPlan = self._make_simple_plan(num_tokens=5)

        tensors: dict[int, torch.Tensor] = {0: tensor}
        aligned: Pair[torch.Tensor] = execute_token_aligner(
            plan=plan,
            tensor_of_step_pair=Pair(x=tensors, y=tensors),
        )

        assert aligned.x.shape == (3, 8, 5)
        plain: torch.Tensor = tensor.rename(None)
        for i in range(5):
            assert torch.equal(
                aligned.x.select(dim=2, index=i), plain.select(dim=2, index=i)
            )
```
**EN:** This test method exercises token dim last and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 token dim last 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 149-164: Run test: token dim zero
```python
    def test_token_dim_zero(self) -> None:
        """token_dim=0 selects along first dimension (standard t-h-d layout)."""
        torch.manual_seed(42)
        tensor: torch.Tensor = _named(torch.randn(5, 8), ["t", "h"])
        plan: TokenAlignerPlan = self._make_simple_plan(num_tokens=5)

        tensors: dict[int, torch.Tensor] = {0: tensor}
        aligned: Pair[torch.Tensor] = execute_token_aligner(
            plan=plan,
            tensor_of_step_pair=Pair(x=tensors, y=tensors),
        )

        assert aligned.x.shape == (5, 8)
        plain: torch.Tensor = tensor.rename(None)
        for i in range(5):
            assert torch.equal(aligned.x[i], plain.select(dim=0, index=i))
```
**EN:** This test method exercises token dim zero and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 token dim zero 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 166-188: Run test: zero matched tokens nonzero token dim
```python
    def test_zero_matched_tokens_nonzero_token_dim(self) -> None:
        """Empty plan with token_dim=1 produces correct empty shape."""
        torch.manual_seed(42)

        plan = TokenAlignerPlan(
            locators=Pair(
                x=TokenLocator(steps=[], token_index_in_step=[]),
                y=TokenLocator(steps=[], token_index_in_step=[]),
            ),
            layouts=Pair(x=TokenLayout.T, y=TokenLayout.T),
        )

        tensors: dict[int, torch.Tensor] = {
            0: _named(torch.randn(3, 5, 8), ["a", "t", "h"])
        }
        aligned: Pair[torch.Tensor] = execute_token_aligner(
            plan=plan,
            tensor_of_step_pair=Pair(x=tensors, y=tensors),
        )

        # token dim (dim 1) set to 0, other dims preserved -> [3, 0, 8]
        assert aligned.x.shape == (3, 0, 8)
        assert aligned.y.shape == (3, 0, 8)
```
**EN:** This test method exercises zero matched tokens nonzero token dim and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 zero matched tokens nonzero token dim 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 190-209: Run test: high rank tensor
```python
    def test_high_rank_tensor(self) -> None:
        """tensor shape [2, 3, 5, 4, 8] (a b t c d), token_dim=2 -> stays at dim 2."""
        torch.manual_seed(42)
        tensor: torch.Tensor = _named(
            torch.randn(2, 3, 5, 4, 8), ["a", "x", "t", "c", "h"]
        )
        plan: TokenAlignerPlan = self._make_simple_plan(num_tokens=5)

        tensors: dict[int, torch.Tensor] = {0: tensor}
        aligned: Pair[torch.Tensor] = execute_token_aligner(
            plan=plan,
            tensor_of_step_pair=Pair(x=tensors, y=tensors),
        )

        assert aligned.x.shape == (2, 3, 5, 4, 8)
        plain: torch.Tensor = tensor.rename(None)
        for i in range(5):
            assert torch.equal(
                aligned.x.select(dim=2, index=i), plain.select(dim=2, index=i)
            )
```
**EN:** This test method exercises high rank tensor and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 high rank tensor 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 212-212: Define class TestBSHDExecutor
```python
class TestBSHDExecutor:
```
**EN:** This declaration introduces the `TestBSHDExecutor` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestBSHDExecutor` 测试类，并说明它通过继承承担的职责。

### Lines 213-213: Document the class `TestBSHDExecutor`
```python
    """BSHD tensor collapse: B+S dims -> flat token dim for alignment."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestBSHDExecutor`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestBSHDExecutor`的设计意图。

### Lines 215-239: Run test: bshd standard bs at front
```python
    def test_bshd_standard_bs_at_front(self):
        """Standard "b s h d": B=dim0, S=dim1. [2, 3, 4, 5] -> collapse -> [6, 4, 5]."""
        torch.manual_seed(42)
        tensor: torch.Tensor = _named(torch.randn(2, 3, 4, 5), ["b", "s", "h", "d"])
        flat: torch.Tensor = tensor.rename(None).reshape(6, 4, 5)

        locator = TokenLocator(
            steps=[0, 0, 0],
            token_index_in_step=[0, 3, 5],
        )
        plan = TokenAlignerPlan(
            locators=Pair(x=locator, y=locator),
            layouts=Pair(x=TokenLayout.BS, y=TokenLayout.BS),
        )

        tensors: dict[int, torch.Tensor] = {0: tensor}
        aligned: Pair[torch.Tensor] = execute_token_aligner(
            plan=plan,
            tensor_of_step_pair=Pair(x=tensors, y=tensors),
        )

        assert aligned.x.shape == (3, 4, 5)
        assert torch.equal(aligned.x[0], flat[0])
        assert torch.equal(aligned.x[1], flat[3])
        assert torch.equal(aligned.x[2], flat[5])
```
**EN:** This test method exercises bshd standard bs at front and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 bshd standard bs at front 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 241-266: Run test: bshd 3d bs at front
```python
    def test_bshd_3d_bs_at_front(self):
        """Minimal 3D "b s h": B=dim0, S=dim1. [2, 3, 4] -> collapse -> [6, 4]."""
        torch.manual_seed(42)
        tensor: torch.Tensor = _named(torch.randn(2, 3, 4), ["b", "s", "h"])
        flat: torch.Tensor = tensor.rename(None).reshape(6, 4)

        locator = TokenLocator(
            steps=[0, 0, 0, 0],
            token_index_in_step=[0, 2, 3, 5],
        )
        plan = TokenAlignerPlan(
            locators=Pair(x=locator, y=locator),
            layouts=Pair(x=TokenLayout.BS, y=TokenLayout.BS),
        )

        tensors: dict[int, torch.Tensor] = {0: tensor}
        aligned: Pair[torch.Tensor] = execute_token_aligner(
            plan=plan,
            tensor_of_step_pair=Pair(x=tensors, y=tensors),
        )

        assert aligned.x.shape == (4, 4)
        assert torch.equal(aligned.x[0], flat[0])
        assert torch.equal(aligned.x[1], flat[2])
        assert torch.equal(aligned.x[2], flat[3])
        assert torch.equal(aligned.x[3], flat[5])
```
**EN:** This test method exercises bshd 3d bs at front and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 bshd 3d bs at front 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 268-294: Run test: bshd bs not at front
```python
    def test_bshd_bs_not_at_front(self):
        """Non-leading "h b s d": B=dim1, S=dim2. [4, 2, 3, 5] -> collapse -> [4, 6, 5]."""
        torch.manual_seed(42)
        tensor: torch.Tensor = _named(torch.randn(4, 2, 3, 5), ["h", "b", "s", "d"])
        flat: torch.Tensor = tensor.rename(None).reshape(4, 6, 5)

        locator = TokenLocator(
            steps=[0, 0, 0],
            token_index_in_step=[0, 3, 5],
        )
        plan = TokenAlignerPlan(
            locators=Pair(x=locator, y=locator),
            layouts=Pair(x=TokenLayout.BS, y=TokenLayout.BS),
        )

        tensors: dict[int, torch.Tensor] = {0: tensor}
        aligned: Pair[torch.Tensor] = execute_token_aligner(
            plan=plan,
            tensor_of_step_pair=Pair(x=tensors, y=tensors),
        )

        assert aligned.x.shape == (4, 3, 5)
        for idx, flat_idx in enumerate([0, 3, 5]):
            assert torch.equal(
                aligned.x.select(dim=1, index=idx),
                flat.select(dim=1, index=flat_idx),
            )
```
**EN:** This test method exercises bshd bs not at front and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 bshd bs not at front 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 296-324: Run test: bshd expert before bs
```python
    def test_bshd_expert_before_bs(self):
        """Expert dim before B: "e b s h d". [2, 3, 4, 5, 6] -> collapse -> [2, 12, 5, 6]."""
        torch.manual_seed(42)
        tensor: torch.Tensor = _named(
            torch.randn(2, 3, 4, 5, 6), ["e", "b", "s", "h", "d"]
        )
        flat: torch.Tensor = tensor.rename(None).reshape(2, 12, 5, 6)

        locator = TokenLocator(
            steps=[0, 0, 0],
            token_index_in_step=[0, 5, 11],
        )
        plan = TokenAlignerPlan(
            locators=Pair(x=locator, y=locator),
            layouts=Pair(x=TokenLayout.BS, y=TokenLayout.BS),
        )

        tensors: dict[int, torch.Tensor] = {0: tensor}
        aligned: Pair[torch.Tensor] = execute_token_aligner(
            plan=plan,
            tensor_of_step_pair=Pair(x=tensors, y=tensors),
        )

        assert aligned.x.shape == (2, 3, 5, 6)
        for idx, flat_idx in enumerate([0, 5, 11]):
            assert torch.equal(
                aligned.x.select(dim=1, index=idx),
                flat.select(dim=1, index=flat_idx),
            )
```
**EN:** This test method exercises bshd expert before bs and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 bshd expert before bs 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 326-352: Run test: bshd bs at end
```python
    def test_bshd_bs_at_end(self):
        """B and S at end: "h d b s". [4, 5, 2, 3] -> collapse -> [4, 5, 6]."""
        torch.manual_seed(42)
        tensor: torch.Tensor = _named(torch.randn(4, 5, 2, 3), ["h", "d", "b", "s"])
        flat: torch.Tensor = tensor.rename(None).reshape(4, 5, 6)

        locator = TokenLocator(
            steps=[0, 0, 0],
            token_index_in_step=[1, 3, 5],
        )
        plan = TokenAlignerPlan(
            locators=Pair(x=locator, y=locator),
            layouts=Pair(x=TokenLayout.BS, y=TokenLayout.BS),
        )

        tensors: dict[int, torch.Tensor] = {0: tensor}
        aligned: Pair[torch.Tensor] = execute_token_aligner(
            plan=plan,
            tensor_of_step_pair=Pair(x=tensors, y=tensors),
        )

        assert aligned.x.shape == (4, 5, 3)
        for idx, flat_idx in enumerate([1, 3, 5]):
            assert torch.equal(
                aligned.x.select(dim=2, index=idx),
                flat.select(dim=2, index=flat_idx),
            )
```
**EN:** This test method exercises bshd bs at end and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 bshd bs at end 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 354-379: Run test: cross layout thd vs bshd
```python
    def test_cross_layout_thd_vs_bshd(self):
        """Cross-layout: x=THD [6, 8], y=BSHD [2, 3, 8] -> y collapse -> [6, 8]."""
        torch.manual_seed(42)
        tensor_thd: torch.Tensor = _named(torch.randn(6, 8), ["t", "h"])
        tensor_bshd: torch.Tensor = _named(torch.randn(2, 3, 8), ["b", "s", "h"])
        flat_bshd: torch.Tensor = tensor_bshd.rename(None).reshape(6, 8)

        locator = TokenLocator(
            steps=[0, 0, 0],
            token_index_in_step=[0, 2, 5],
        )
        plan = TokenAlignerPlan(
            locators=Pair(x=locator, y=locator),
            layouts=Pair(x=TokenLayout.T, y=TokenLayout.BS),
        )

        aligned: Pair[torch.Tensor] = execute_token_aligner(
            plan=plan,
            tensor_of_step_pair=Pair(x={0: tensor_thd}, y={0: tensor_bshd}),
        )

        assert aligned.x.shape == (3, 8)
        assert aligned.y.shape == (3, 8)
        assert torch.equal(aligned.x[0], tensor_thd.rename(None)[0])
        assert torch.equal(aligned.y[0], flat_bshd[0])
        assert torch.equal(aligned.y[2], flat_bshd[5])
```
**EN:** This test method exercises cross layout thd vs bshd and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cross layout thd vs bshd 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 381-408: Run test: bshd reversed sb order
```python
    def test_bshd_reversed_sb_order(self):
        """Reversed "s b h": S=dim0, B=dim1. Collapse is batch-major: (b s)."""
        torch.manual_seed(42)
        tensor: torch.Tensor = _named(torch.randn(3, 2, 4), ["s", "b", "h"])
        # batch-major flatten: rearrange("s b h -> (b s) h")
        from einops import rearrange

        flat: torch.Tensor = rearrange(tensor.rename(None), "s b h -> (b s) h")

        locator = TokenLocator(
            steps=[0, 0, 0],
            token_index_in_step=[0, 2, 5],
        )
        plan = TokenAlignerPlan(
            locators=Pair(x=locator, y=locator),
            layouts=Pair(x=TokenLayout.BS, y=TokenLayout.BS),
        )

        tensors: dict[int, torch.Tensor] = {0: tensor}
        aligned: Pair[torch.Tensor] = execute_token_aligner(
            plan=plan,
            tensor_of_step_pair=Pair(x=tensors, y=tensors),
        )

        assert aligned.x.shape == (3, 4)
        assert torch.equal(aligned.x[0], flat[0])
        assert torch.equal(aligned.x[1], flat[2])
        assert torch.equal(aligned.x[2], flat[5])
```
**EN:** This test method exercises bshd reversed sb order and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 bshd reversed sb order 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 410-429: Run test: bshd empty plan bs not at front
```python
    def test_bshd_empty_plan_bs_not_at_front(self):
        """Empty plan with non-leading B,S: "h b s d". [4, 2, 3, 5] -> collapse -> [4, 0, 5]."""
        plan = TokenAlignerPlan(
            locators=Pair(
                x=TokenLocator(steps=[], token_index_in_step=[]),
                y=TokenLocator(steps=[], token_index_in_step=[]),
            ),
            layouts=Pair(x=TokenLayout.BS, y=TokenLayout.BS),
        )

        tensors: dict[int, torch.Tensor] = {
            0: _named(torch.randn(4, 2, 3, 5), ["h", "b", "s", "d"])
        }
        aligned: Pair[torch.Tensor] = execute_token_aligner(
            plan=plan,
            tensor_of_step_pair=Pair(x=tensors, y=tensors),
        )

        assert aligned.x.shape == (4, 0, 5)
        assert aligned.y.shape == (4, 0, 5)
```
**EN:** This test method exercises bshd empty plan bs not at front and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 bshd empty plan bs not at front 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 431-450: Run test: bshd empty plan bs at front
```python
    def test_bshd_empty_plan_bs_at_front(self):
        """Empty plan with standard BSHD: "b s h". [2, 3, 4] -> collapse -> [0, 4]."""
        plan = TokenAlignerPlan(
            locators=Pair(
                x=TokenLocator(steps=[], token_index_in_step=[]),
                y=TokenLocator(steps=[], token_index_in_step=[]),
            ),
            layouts=Pair(x=TokenLayout.BS, y=TokenLayout.BS),
        )

        tensors: dict[int, torch.Tensor] = {
            0: _named(torch.randn(2, 3, 4), ["b", "s", "h"])
        }
        aligned: Pair[torch.Tensor] = execute_token_aligner(
            plan=plan,
            tensor_of_step_pair=Pair(x=tensors, y=tensors),
        )

        assert aligned.x.shape == (0, 4)
        assert aligned.y.shape == (0, 4)
```
**EN:** This test method exercises bshd empty plan bs at front and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 bshd empty plan bs at front 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 453-454: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.executor`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.planner`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.seq_info_builder`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types`, `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.srt.debug_utils.comparator.utils`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `__future__`, `einops`, `pytest`, `sys`, `torch`
- Notable symbols / 关键符号: None / 无
