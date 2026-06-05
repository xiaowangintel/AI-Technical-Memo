# bad_words.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/sample/ops/bad_words.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_apply_bad_words_single_batch`, `apply_bad_words`, `apply_bad_words_with_drafts` for the V1 `sample/ops` subsystem. / 为 V1 的 `sample/ops` 子系统实现 `_apply_bad_words_single_batch`, `apply_bad_words`, `apply_bad_words_with_drafts`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import torch

_SMALLEST_LOGIT = float("-inf")
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `_SMALLEST_LOGIT`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `_SMALLEST_LOGIT`。

### `_apply_bad_words_single_batch` function / `_apply_bad_words_single_batch` 函数
```python
def _apply_bad_words_single_batch(
    logits: torch.Tensor,
    bad_words_token_ids: list[list[int]],
    past_tokens_ids: list[int],
) -> None:
    for bad_word_ids in bad_words_token_ids:
        if len(bad_word_ids) > len(past_tokens_ids) + 1:
            continue

        prefix_length = len(bad_word_ids) - 1
        last_token_id = bad_word_ids[-1]
        actual_prefix = past_tokens_ids[-prefix_length:] if prefix_length > 0 else []
        expected_prefix = bad_word_ids[:prefix_length]

        assert len(actual_prefix) == len(expected_prefix)

        if actual_prefix == expected_prefix:
            # Assign to slice to avoid cpu->gpu sync.
            logits[last_token_id : last_token_id + 1] = _SMALLEST_LOGIT
```
**EN:** This function implements `_apply_bad_words_single_batch` within the module. Key calls include `len`. The control flow contains 3 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_apply_bad_words_single_batch`，其作用域位于the module。 关键调用包括 `len`。 控制流包含 3 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `apply_bad_words` function / `apply_bad_words` 函数
```python
def apply_bad_words(
    logits: torch.Tensor,
    bad_words_token_ids: dict[int, list[list[int]]],
    past_tokens_ids: list[list[int]],
) -> None:
    for i, bad_words_ids in bad_words_token_ids.items():
        _apply_bad_words_single_batch(logits[i], bad_words_ids, past_tokens_ids[i])
```
**EN:** This function implements `apply_bad_words` within the module. Key calls include `items`, `_apply_bad_words_single_batch`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `apply_bad_words`，其作用域位于the module。 关键调用包括 `items`, `_apply_bad_words_single_batch`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `apply_bad_words_with_drafts` function / `apply_bad_words_with_drafts` 函数
```python
def apply_bad_words_with_drafts(
    logits: torch.Tensor,
    bad_words_token_ids: dict[int, list[list[int]]],
    past_tokens_ids: list[list[int]],
    num_draft_tokens: list[int],
) -> None:
    start_idx = 0
    remaining = len(bad_words_token_ids)
    for i, n in enumerate(num_draft_tokens):
        if (bad_words_ids := bad_words_token_ids.get(i)) is not None:
            for draft_idx in range(start_idx, start_idx + n):
                _apply_bad_words_single_batch(
                    logits[draft_idx],
                    bad_words_ids,
                    past_tokens_ids[draft_idx],
                )
            remaining -= 1
            if not remaining:
                break
        start_idx += n
```
**EN:** This function implements `apply_bad_words_with_drafts` within the module. Key calls include `len`, `enumerate`, `range`, `get`, `_apply_bad_words_single_batch`. The control flow contains 2 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `apply_bad_words_with_drafts`，其作用域位于the module。 关键调用包括 `len`, `enumerate`, `range`, `get`, `_apply_bad_words_single_batch`。 控制流包含 2 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `_apply_bad_words_single_batch`: top-level helper or orchestration entry point. / `_apply_bad_words_single_batch`：顶层辅助函数或编排入口。
- `apply_bad_words`: top-level helper or orchestration entry point. / `apply_bad_words`：顶层辅助函数或编排入口。
- `apply_bad_words_with_drafts`: top-level helper or orchestration entry point. / `apply_bad_words_with_drafts`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- External / 外部依赖: `torch`
