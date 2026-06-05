# late_interaction.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/pool/late_interaction.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `get_late_interaction_engine_index`, `build_late_interaction_query_params`, `build_late_interaction_doc_params` for the V1 `pool` subsystem. / 为 V1 的 `pool` 子系统实现 `get_late_interaction_engine_index`, `build_late_interaction_query_params`, `build_late_interaction_doc_params`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import zlib
from collections.abc import Sequence

import torch

from vllm.pooling_params import LateInteractionParams, PoolingParams

LATE_INTERACTION_MODE_CACHE_QUERY = "cache_query"
LATE_INTERACTION_MODE_SCORE_DOC = "score_doc"
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `LATE_INTERACTION_MODE_CACHE_QUERY`, `LATE_INTERACTION_MODE_SCORE_DOC`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `LATE_INTERACTION_MODE_CACHE_QUERY`, `LATE_INTERACTION_MODE_SCORE_DOC`。

### `get_late_interaction_engine_index` function / `get_late_interaction_engine_index` 函数
```python
def get_late_interaction_engine_index(
    pooling_params: PoolingParams | None,
    num_engines: int,
) -> int | None:
    if pooling_params is None or pooling_params.late_interaction_params is None:
        return None

    late_interaction_params = pooling_params.late_interaction_params
    mode = late_interaction_params.mode
    if mode not in (
        LATE_INTERACTION_MODE_CACHE_QUERY,
        LATE_INTERACTION_MODE_SCORE_DOC,
    ):
        return None

    query_key = late_interaction_params.query_key
    if not isinstance(query_key, str) or not query_key:
        return None

    # query embeddings are cached in process-local worker memory,
    # pin requests sharing the same query key to the same engine.
    return zlib.crc32(query_key.encode("utf-8")) % num_engines
```
**EN:** This function returns or derives a value within the module. Key calls include `crc32`, `isinstance`, `encode`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会返回或推导一个值，其作用域位于the module。 关键调用包括 `crc32`, `isinstance`, `encode`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `build_late_interaction_query_params` function / `build_late_interaction_query_params` 函数
```python
def build_late_interaction_query_params(
    query_key: str,
    query_uses: int,
) -> LateInteractionParams:
    return LateInteractionParams(
        mode=LATE_INTERACTION_MODE_CACHE_QUERY,
        query_key=query_key,
        query_uses=max(1, int(query_uses)),
    )
```
**EN:** This function builds derived structures within the module. Key calls include `LateInteractionParams`, `max`, `int`.
**CN:** 该函数会构建派生结构，其作用域位于the module。 关键调用包括 `LateInteractionParams`, `max`, `int`。

### `build_late_interaction_doc_params` function / `build_late_interaction_doc_params` 函数
```python
def build_late_interaction_doc_params(
    query_key: str,
) -> LateInteractionParams:
    return LateInteractionParams(
        mode=LATE_INTERACTION_MODE_SCORE_DOC,
        query_key=query_key,
    )
```
**EN:** This function builds derived structures within the module. Key calls include `LateInteractionParams`.
**CN:** 该函数会构建派生结构，其作用域位于the module。 关键调用包括 `LateInteractionParams`。

### `compute_maxsim_score_batched` function / `compute_maxsim_score_batched` 函数
```python
def compute_maxsim_score_batched(
    q_embs: Sequence[torch.Tensor],
    d_embs: Sequence[torch.Tensor],
    max_batch_size: int = 64,
    max_score_matrix_elements: int = 64_000_000,
) -> list[torch.Tensor]:
    """Compute MaxSim for multiple query/doc pairs in mini-batches."""
    if len(q_embs) != len(d_embs):
        raise ValueError("q_embs and d_embs must have the same length")

    num_pairs = len(q_embs)
    if num_pairs == 0:
        return []

    if max_batch_size <= 0:
        raise ValueError("max_batch_size must be greater than 0")
    if max_score_matrix_elements <= 0:
        raise ValueError("max_score_matrix_elements must be greater than 0")

    for q_emb, d_emb in zip(q_embs, d_embs):
        if q_emb.ndim != 2 or d_emb.ndim != 2:
            raise ValueError("Each embedding tensor must be 2-D")
        if q_emb.shape[1] != d_emb.shape[1]:
            raise ValueError("Query and document embeddings must have same dim")
        if q_emb.device != d_emb.device:
            raise ValueError("Query and document embeddings must be on same device")

    scores: list[torch.Tensor] = []
    start = 0
    while start < num_pairs:
        end = min(start + max_batch_size, num_pairs)
        max_q = max(int(x.shape[0]) for x in q_embs[start:end])
        max_d = max(int(x.shape[0]) for x in d_embs[start:end])

        # keep score matrix bounded to avoid oversized allocations.
        while (
            end - start > 1
            and (end - start) * max_q * max_d > max_score_matrix_elements
        ):
            end -= 1
            max_q = max(int(x.shape[0]) for x in q_embs[start:end])
            max_d = max(int(x.shape[0]) for x in d_embs[start:end])

        batch_q = q_embs[start:end]
        batch_d = d_embs[start:end]
        batch_size = end - start
        device = batch_q[0].device
        dim = int(batch_q[0].shape[1])

        q_batch = torch.zeros(
            (batch_size, max_q, dim), dtype=torch.float32, device=device
        )
        d_batch = torch.zeros(
            (batch_size, max_d, dim), dtype=torch.float32, device=device
        )
        q_mask = torch.zeros((batch_size, max_q), dtype=torch.bool, device=device)
        d_mask = torch.zeros((batch_size, max_d), dtype=torch.bool, device=device)

        # copy to padded tensors
        for i, (q_emb, d_emb) in enumerate(zip(batch_q, batch_d)):
            q_len = int(q_emb.shape[0])
            d_len = int(d_emb.shape[0])
            q_batch[i, :q_len] = q_emb.to(device=device, dtype=torch.float32)
            d_batch[i, :d_len] = d_emb.to(device=device, dtype=torch.float32)
            q_mask[i, :q_len] = True
            d_mask[i, :d_len] = True

        token_scores = torch.bmm(q_batch, d_batch.transpose(1, 2))
        token_scores.masked_fill_(~d_mask.unsqueeze(1), float("-inf"))
        max_per_query = token_scores.amax(dim=-1)
        max_per_query.masked_fill_(~q_mask, 0.0)
        batch_scores = max_per_query.sum(dim=-1)
        scores.extend(batch_scores.unbind(0))
        start = end

    return scores
```
**EN:** This function computes derived values within the module. The docstring frames it as: Compute MaxSim for multiple query/doc pairs in mini-batches. Key calls include `len`, `zip`, `ValueError`, `min`, `max`, `int`. The control flow contains 7 branch(es) and 4 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会计算派生值，其作用域位于the module。 关键调用包括 `len`, `zip`, `ValueError`, `min`, `max`, `int`。 控制流包含 7 个分支和 4 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `get_late_interaction_engine_index`: top-level helper or orchestration entry point. / `get_late_interaction_engine_index`：顶层辅助函数或编排入口。
- `build_late_interaction_query_params`: top-level helper or orchestration entry point. / `build_late_interaction_query_params`：顶层辅助函数或编排入口。
- `build_late_interaction_doc_params`: top-level helper or orchestration entry point. / `build_late_interaction_doc_params`：顶层辅助函数或编排入口。
- `compute_maxsim_score_batched`: top-level helper or orchestration entry point. / `compute_maxsim_score_batched`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- Standard library / 标准库: `zlib`, `collections`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.pooling_params`
