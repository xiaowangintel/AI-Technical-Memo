# test_pooler_score_and_pool.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/layers/test_pooler_score_and_pool.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates pooler score and pool behavior in SGLang's unit / layers area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 层 领域中与 pooler score and pool 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: supporting statements / 辅助语句
```python
"""Unit tests for score_and_pool in sglang.srt.layers.pooler.

All tests run on CPU — no GPU required.  MIS delimiter positions are passed
via forward_batch.multi_item_delimiter_indices (pre-computed by the caller).
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 7-20: module imports and dependencies / 模块导入与依赖
```python
import unittest
from types import SimpleNamespace

import torch
import torch.nn as nn

from sglang.srt.layers.pooler import (
    EmbeddingPoolerOutput,
    Pooler,
    PoolingType,
    score_and_pool,
)
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `types`, `torch`, `torch.nn`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `types`, `torch`, `torch.nn`。

### Lines 22-22: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=10, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 25-39: function make forward batch / 函数 make forward batch
```python
def _make_forward_batch(
    extend_seq_lens,
    multi_item_delimiter_indices=None,
    return_pooled_hidden_states=False,
    is_prefill_only=True,
):
    """Build a minimal ForwardBatch stub for pooler unit tests."""
    return SimpleNamespace(
        extend_seq_lens=torch.tensor(extend_seq_lens, dtype=torch.long),
        extend_seq_lens_cpu=extend_seq_lens,
        multi_item_delimiter_indices=multi_item_delimiter_indices,
        dimensions=None,
        return_pooled_hidden_states=return_pooled_hidden_states,
        is_prefill_only=is_prefill_only,
    )
```
**EN:** Build a minimal ForwardBatch stub for pooler unit tests. This block implements `_make_forward_batch` and captures one focused piece of the module's behavior.
**CN:** Build a minimal ForwardBatch stub for pooler unit tests. 该代码块实现 `_make_forward_batch`，承担模块行为中的一个聚焦逻辑片段。

### Lines 42-42: class TestScoreAndPool declaration / 类 TestScoreAndPool 声明
```python
class TestScoreAndPool(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 43-43: supporting statements / 辅助语句
```python
    """Unit tests for the score_and_pool helper function."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 45-50: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        torch.manual_seed(42)
        self.hidden_dim = 8
        self.num_labels = 2
        self.score_head = nn.Linear(self.hidden_dim, self.num_labels, bias=False)
        self.pooler = Pooler(pooling_type=PoolingType.LAST, normalize=False)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 52-61: test case single item returns scores / 测试用例 single item returns scores
```python
    def test_single_item_returns_scores(self):
        """No delimiter indices -> single-item path returns [batch, num_labels]."""
        hidden = torch.randn(8, self.hidden_dim)
        fb = _make_forward_batch(extend_seq_lens=[5, 3])
        input_ids = torch.arange(8)

        out = score_and_pool(self.score_head, self.pooler, hidden, fb, input_ids)

        self.assertIsInstance(out, EmbeddingPoolerOutput)
        self.assertEqual(out.embeddings.shape, (2, self.num_labels))
```
**EN:** No delimiter indices -> single-item path returns [batch, num_labels]. This test exercises `test_single_item_returns_scores` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** No delimiter indices -> single-item path returns [batch, num_labels]. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_item_returns_scores`。

### Lines 63-78: test case mis returns per request list / 测试用例 mis returns per request list
```python
    def test_mis_returns_per_request_list(self):
        """Delimiter indices provided -> returns a list with one tensor per request."""
        # Sequence: [0, 1, 2, D, 3, 4, 5, D, 6, 7, 8, D]
        # Delimiters at positions 3, 7, 11 -> extract at 2, 6, 10
        input_ids = torch.arange(12)
        hidden = torch.randn(len(input_ids), self.hidden_dim)
        fb = _make_forward_batch(
            extend_seq_lens=[len(input_ids)],
            multi_item_delimiter_indices=[torch.tensor([3, 7, 11])],
        )

        out = score_and_pool(self.score_head, self.pooler, hidden, fb, input_ids)

        self.assertIsInstance(out.embeddings, list)
        self.assertEqual(len(out.embeddings), 1)
        self.assertEqual(out.embeddings[0].shape, (3, self.num_labels))
```
**EN:** Delimiter indices provided -> returns a list with one tensor per request. This test exercises `test_mis_returns_per_request_list` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Delimiter indices provided -> returns a list with one tensor per request. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mis_returns_per_request_list`。

### Lines 80-101: test case mis batched splits per request / 测试用例 mis batched splits per request
```python
    def test_mis_batched_splits_per_request(self):
        """Two batched MIS requests -> returns a list of length 2."""
        # Request 1: [10, 11, D, 12, 13, D]  -> delimiters at 2, 5
        # Request 2: [20, 21, 22, D]          -> delimiter at 3
        req1 = [10, 11, 99, 12, 13, 99]
        req2 = [20, 21, 22, 99]
        input_ids = torch.tensor(req1 + req2)
        hidden = torch.randn(len(input_ids), self.hidden_dim)
        fb = _make_forward_batch(
            extend_seq_lens=[len(req1), len(req2)],
            multi_item_delimiter_indices=[
                torch.tensor([2, 5]),
                torch.tensor([3]),
            ],
        )

        out = score_and_pool(self.score_head, self.pooler, hidden, fb, input_ids)

        self.assertIsInstance(out.embeddings, list)
        self.assertEqual(len(out.embeddings), 2)
        self.assertEqual(out.embeddings[0].shape, (2, self.num_labels))
        self.assertEqual(out.embeddings[1].shape, (1, self.num_labels))
```
**EN:** Two batched MIS requests -> returns a list of length 2. This test exercises `test_mis_batched_splits_per_request` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Two batched MIS requests -> returns a list of length 2. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mis_batched_splits_per_request`。

### Lines 103-112: test case no delimiter indices falls back / 测试用例 no delimiter indices falls back
```python
    def test_no_delimiter_indices_falls_back(self):
        """multi_item_delimiter_indices=None -> single-item fallback."""
        input_ids = torch.tensor([0, 1, 2, 3, 4, 5, 6, 7])
        hidden = torch.randn(8, self.hidden_dim)
        fb = _make_forward_batch(extend_seq_lens=[5, 3])

        out = score_and_pool(self.score_head, self.pooler, hidden, fb, input_ids)

        self.assertIsInstance(out.embeddings, torch.Tensor)
        self.assertEqual(out.embeddings.shape, (2, self.num_labels))
```
**EN:** multi_item_delimiter_indices=None -> single-item fallback. This test exercises `test_no_delimiter_indices_falls_back` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** multi_item_delimiter_indices=None -> single-item fallback. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_delimiter_indices_falls_back`。

### Lines 114-137: test case mis extracts positions before delimiter / 测试用例 mis extracts positions before delimiter
```python
    def test_mis_extracts_positions_before_delimiter(self):
        """Verify MIS picks hidden states at index (delimiter_position - 1)."""
        # Delimiters at indices 2 and 5 -> extract hidden at indices 1 and 4
        input_ids = torch.tensor([10, 11, 99, 20, 21, 99])
        hidden = (
            torch.arange(len(input_ids))
            .unsqueeze(1)
            .float()
            .expand(-1, self.hidden_dim)
            .clone()
        )
        fb = _make_forward_batch(
            extend_seq_lens=[len(input_ids)],
            multi_item_delimiter_indices=[torch.tensor([2, 5])],
        )

        identity_head = nn.Linear(self.hidden_dim, self.hidden_dim, bias=False)
        nn.init.eye_(identity_head.weight)

        out = score_and_pool(identity_head, self.pooler, hidden, fb, input_ids)

        scores = out.embeddings[0]
        torch.testing.assert_close(scores[0], hidden[1])
        torch.testing.assert_close(scores[1], hidden[4])
```
**EN:** Verify MIS picks hidden states at index (delimiter_position - 1). This test exercises `test_mis_extracts_positions_before_delimiter` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Verify MIS picks hidden states at index (delimiter_position - 1). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mis_extracts_positions_before_delimiter`。

### Lines 139-162: test case mis delimiter at position one / 测试用例 mis delimiter at position one
```python
    def test_mis_delimiter_at_position_one(self):
        """Delimiters at positions 1 and 3 extract at indices 0 and 2."""
        input_ids = torch.tensor([10, 99, 11, 99])
        hidden = (
            torch.arange(len(input_ids))
            .unsqueeze(1)
            .float()
            .expand(-1, self.hidden_dim)
            .clone()
        )
        fb = _make_forward_batch(
            extend_seq_lens=[len(input_ids)],
            multi_item_delimiter_indices=[torch.tensor([1, 3])],
        )

        identity_head = nn.Linear(self.hidden_dim, self.hidden_dim, bias=False)
        nn.init.eye_(identity_head.weight)

        out = score_and_pool(identity_head, self.pooler, hidden, fb, input_ids)

        self.assertEqual(len(out.embeddings), 1)
        self.assertEqual(out.embeddings[0].shape[0], 2)
        torch.testing.assert_close(out.embeddings[0][0], hidden[0])
        torch.testing.assert_close(out.embeddings[0][1], hidden[2])
```
**EN:** Delimiters at positions 1 and 3 extract at indices 0 and 2. This test exercises `test_mis_delimiter_at_position_one` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Delimiters at positions 1 and 3 extract at indices 0 and 2. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mis_delimiter_at_position_one`。

### Lines 164-174: test case single item scores match manual computation / 测试用例 single item scores match manual computation
```python
    def test_single_item_scores_match_manual_computation(self):
        """Single-item scores equal score_head applied to pooled hidden states."""
        hidden = torch.randn(8, self.hidden_dim)
        fb = _make_forward_batch(extend_seq_lens=[5, 3])
        input_ids = torch.arange(8)

        out = score_and_pool(self.score_head, self.pooler, hidden, fb, input_ids)

        pooled = self.pooler(hidden, fb).embeddings
        expected = self.score_head(pooled)
        torch.testing.assert_close(out.embeddings, expected)
```
**EN:** Single-item scores equal score_head applied to pooled hidden states. This test exercises `test_single_item_scores_match_manual_computation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Single-item scores equal score_head applied to pooled hidden states. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_item_scores_match_manual_computation`。

### Lines 176-189: test case empty delimiter indices / 测试用例 empty delimiter indices
```python
    def test_empty_delimiter_indices(self):
        """Empty delimiter tensor per request -> returns list with empty tensor."""
        input_ids = torch.arange(6)
        hidden = torch.randn(6, self.hidden_dim)
        fb = _make_forward_batch(
            extend_seq_lens=[6],
            multi_item_delimiter_indices=[torch.tensor([], dtype=torch.long)],
        )

        out = score_and_pool(self.score_head, self.pooler, hidden, fb, input_ids)

        self.assertIsInstance(out.embeddings, list)
        self.assertEqual(len(out.embeddings), 1)
        self.assertEqual(out.embeddings[0].shape, (0, self.num_labels))
```
**EN:** Empty delimiter tensor per request -> returns list with empty tensor. This test exercises `test_empty_delimiter_indices` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Empty delimiter tensor per request -> returns list with empty tensor. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_delimiter_indices`。

### Lines 192-193: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_make_forward_batch`: Build a minimal ForwardBatch stub for pooler unit tests. / 该代码块实现 `_make_forward_batch`，承担模块行为中的一个聚焦逻辑片段。
- `TestScoreAndPool`: Unit tests for the score_and_pool helper function. / 用于组织相关测试、夹具或辅助方法。
- `TestScoreAndPool.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestScoreAndPool.test_single_item_returns_scores`: No delimiter indices -> single-item path returns [batch, num_labels]. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_item_returns_scores`。
- `TestScoreAndPool.test_mis_returns_per_request_list`: Delimiter indices provided -> returns a list with one tensor per request. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mis_returns_per_request_list`。
- `TestScoreAndPool.test_mis_batched_splits_per_request`: Two batched MIS requests -> returns a list of length 2. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mis_batched_splits_per_request`。
- `TestScoreAndPool.test_no_delimiter_indices_falls_back`: multi_item_delimiter_indices=None -> single-item fallback. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_delimiter_indices_falls_back`。
- `TestScoreAndPool.test_mis_extracts_positions_before_delimiter`: Verify MIS picks hidden states at index (delimiter_position - 1). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mis_extracts_positions_before_delimiter`。
- `TestScoreAndPool.test_mis_delimiter_at_position_one`: Delimiters at positions 1 and 3 extract at indices 0 and 2. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mis_delimiter_at_position_one`。
- `TestScoreAndPool.test_single_item_scores_match_manual_computation`: Single-item scores equal score_head applied to pooled hidden states. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_item_scores_match_manual_computation`。
- `TestScoreAndPool.test_empty_delimiter_indices`: Empty delimiter tensor per request -> returns list with empty tensor. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_delimiter_indices`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`, `types`
- **Third-party modules / 第三方模块**: `torch`, `torch.nn`
- **Internal modules / 内部模块**: `sglang.srt.layers.pooler`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 193
