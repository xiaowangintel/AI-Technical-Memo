# late_interaction_runner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/pool/late_interaction_runner.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `LateInteractionRunner` for the V1 `worker/gpu/pool` subsystem. / 为 V1 的 `worker/gpu/pool` 子系统实现 `LateInteractionRunner`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
from collections.abc import Iterable

import torch

from vllm.pooling_params import PoolingParams
from vllm.v1.outputs import PoolerOutput
from vllm.v1.pool.late_interaction import (
    LATE_INTERACTION_MODE_CACHE_QUERY,
    LATE_INTERACTION_MODE_SCORE_DOC,
    compute_maxsim_score_batched,
)
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `torch`, and internal vLLM modules such as `vllm.pooling_params`, `vllm.v1.outputs`, `vllm.v1.pool.late_interaction`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`torch` 等外部依赖，以及 `vllm.pooling_params`, `vllm.v1.outputs`, `vllm.v1.pool.late_interaction` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `LateInteractionRunner` class / `LateInteractionRunner` 类
```python
class LateInteractionRunner:
    """Worker-side state and postprocessing for late-interaction scoring."""
```
**EN:** Introduces the `LateInteractionRunner` class. Core methods include `__init__`, `clear`, `register_request`, `on_requests_finished`, `postprocess_pooler_output`, `_release_query_use`. Docstring signal: Worker-side state and postprocessing for late-interaction scoring.
**CN:** 这里定义 `LateInteractionRunner` 类。核心方法包括 `__init__`, `clear`, `register_request`, `on_requests_finished`, `postprocess_pooler_output`, `_release_query_use`。

### `LateInteractionRunner.__init__` method / `LateInteractionRunner.__init__` 方法
```python
    def __init__(self) -> None:
        # query_key -> token embeddings for late-interaction scoring.
        self._query_cache: dict[str, torch.Tensor] = {}
        # query_key -> remaining number of docs that should use this query.
        self._query_uses: dict[str, int] = {}
        # doc request id -> query key.
        self._doc_query_keys: dict[str, str] = {}
```
**EN:** This method initializes the object state within `LateInteractionRunner`. It touches state such as `_query_cache`, `_query_uses`, `_doc_query_keys`.
**CN:** 该方法会初始化对象状态，其作用域位于`LateInteractionRunner`。 它会读写 `_query_cache`, `_query_uses`, `_doc_query_keys` 等状态。

### `LateInteractionRunner.clear` method / `LateInteractionRunner.clear` 方法
```python
    def clear(self) -> None:
        self._query_cache.clear()
        self._query_uses.clear()
        self._doc_query_keys.clear()
```
**EN:** This method implements `clear` within `LateInteractionRunner`. Key calls include `clear`.
**CN:** 该方法会实现 `clear`，其作用域位于`LateInteractionRunner`。 关键调用包括 `clear`。

### `LateInteractionRunner.register_request` method / `LateInteractionRunner.register_request` 方法
```python
    def register_request(
        self, req_id: str, pooling_params: PoolingParams | None
    ) -> None:
        mode, query_key, _ = self._parse_late_interaction_meta(pooling_params)
        if mode == LATE_INTERACTION_MODE_SCORE_DOC and query_key is not None:
            self._doc_query_keys[req_id] = query_key
        else:
            self._doc_query_keys.pop(req_id, None)
```
**EN:** This method implements `register_request` within `LateInteractionRunner`. Key calls include `_parse_late_interaction_meta`, `pop`. It touches state such as `_doc_query_keys`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `register_request`，其作用域位于`LateInteractionRunner`。 关键调用包括 `_parse_late_interaction_meta`, `pop`。 它会读写 `_doc_query_keys` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `LateInteractionRunner.on_requests_finished` method / `LateInteractionRunner.on_requests_finished` 方法
```python
    def on_requests_finished(self, finished_req_ids: Iterable[str]) -> None:
        for req_id in finished_req_ids:
            query_key = self._doc_query_keys.pop(req_id, None)
            if query_key is not None:
                self._release_query_use(query_key)
```
**EN:** This method implements `on_requests_finished` within `LateInteractionRunner`. Key calls include `pop`, `_release_query_use`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `on_requests_finished`，其作用域位于`LateInteractionRunner`。 关键调用包括 `pop`, `_release_query_use`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `LateInteractionRunner.postprocess_pooler_output` method / `LateInteractionRunner.postprocess_pooler_output` 方法
```python
    def postprocess_pooler_output(
        self,
        raw_pooler_output: PoolerOutput,
        pooling_params: list[PoolingParams],
        req_ids: list[str],
        finished_mask: list[bool],
    ) -> PoolerOutput:
        if not isinstance(raw_pooler_output, list):
            return raw_pooler_output

        num_reqs = len(pooling_params)
        if len(raw_pooler_output) != num_reqs:
            raise ValueError(
                "raw_pooler_output and pooling_params must have the same length."
            )
        if len(req_ids) != num_reqs:
            raise ValueError("req_ids and pooling_params must have the same length.")
        if len(finished_mask) != num_reqs:
            raise ValueError(
                "finished_mask and pooling_params must have the same length."
            )

        if not any(finished_mask):
            return raw_pooler_output
        if not any(p.late_interaction_params is not None for p in pooling_params):
            return raw_pooler_output

        outputs: list[torch.Tensor | None] = list(raw_pooler_output)
        score_indices: list[int] = []
        score_req_ids: list[str] = []
        score_query_keys: list[str] = []
        score_queries: list[torch.Tensor] = []
        score_docs: list[torch.Tensor] = []
        for i, (req_id, output, params, finished) in enumerate(
            zip(req_ids, outputs, pooling_params, finished_mask)
        ):
            if not finished or output is None:
                continue

            mode, query_key, query_uses = self._parse_late_interaction_meta(params)
            if mode is None:
                continue

            assert query_key is not None
            if mode == LATE_INTERACTION_MODE_CACHE_QUERY:
                assert query_uses is not None
                # `output` can be a view into the current step's hidden-states
                # buffer, so clone it before storing across scheduling steps.
                self._query_cache[query_key] = output.clone()
                self._query_uses[query_key] = query_uses
                outputs[i] = torch.zeros((), device=output.device, dtype=torch.float32)
                continue

            if mode == LATE_INTERACTION_MODE_SCORE_DOC:
                query_output = self._query_cache.get(query_key)
                if query_output is None:
                    raise ValueError(
                        "late-interaction query cache miss for key "
                        f"{query_key!r}. Ensure query requests are executed "
                        "before their paired document requests."
                    )

                score_indices.append(i)
                score_req_ids.append(req_id)
                score_query_keys.append(query_key)
                score_queries.append(query_output)
                score_docs.append(output)
                continue

            raise ValueError(f"Unsupported late-interaction mode: {mode!r}")

        if score_indices:
            score_values = compute_maxsim_score_batched(score_queries, score_docs)
            for i, req_id, query_key, score in zip(
                score_indices, score_req_ids, score_query_keys, score_values
            ):
                outputs[i] = score
                self._doc_query_keys.pop(req_id, None)
                self._release_query_use(query_key)

        return outputs
```
**EN:** This method implements `postprocess_pooler_output` within `LateInteractionRunner`. Key calls include `len`, `list`, `enumerate`, `isinstance`, `ValueError`, `any`. It touches state such as `_query_cache`, `_query_uses`. The control flow contains 12 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `postprocess_pooler_output`，其作用域位于`LateInteractionRunner`。 关键调用包括 `len`, `list`, `enumerate`, `isinstance`, `ValueError`, `any`。 它会读写 `_query_cache`, `_query_uses` 等状态。 控制流包含 12 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `LateInteractionRunner._release_query_use` method / `LateInteractionRunner._release_query_use` 方法
```python
    def _release_query_use(self, query_key: str) -> None:
        remaining = self._query_uses.get(query_key, 1) - 1
        if remaining <= 0:
            self._query_uses.pop(query_key, None)
            self._query_cache.pop(query_key, None)
        else:
            self._query_uses[query_key] = remaining
```
**EN:** This method implements `_release_query_use` within `LateInteractionRunner`. Key calls include `get`, `pop`. It touches state such as `_query_uses`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_release_query_use`，其作用域位于`LateInteractionRunner`。 关键调用包括 `get`, `pop`。 它会读写 `_query_uses` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `LateInteractionRunner._parse_late_interaction_meta` method / `LateInteractionRunner._parse_late_interaction_meta` 方法
```python
    @staticmethod
    def _parse_late_interaction_meta(
        pooling_params: PoolingParams | None,
    ) -> tuple[str | None, str | None, int | None]:
        if pooling_params is None or pooling_params.late_interaction_params is None:
            return None, None, None

        late_interaction_params = pooling_params.late_interaction_params
        mode = late_interaction_params.mode

        query_key = late_interaction_params.query_key
        if not isinstance(query_key, str) or not query_key:
            raise ValueError(
                "late-interaction request is missing a valid query key in "
                "pooling_params.late_interaction_params."
            )

        if mode == LATE_INTERACTION_MODE_CACHE_QUERY:
            query_uses_raw = late_interaction_params.query_uses
            if query_uses_raw is None:
                query_uses_raw = 1
            try:
                query_uses = max(1, int(query_uses_raw))
            except (TypeError, ValueError) as exc:
                raise ValueError(
                    "late-interaction query uses must be an integer value."
                ) from exc
            return mode, query_key, query_uses

        return mode, query_key, None
```
**EN:** This method implements `_parse_late_interaction_meta` within `LateInteractionRunner`. Key calls include `ValueError`, `isinstance`, `max`, `int`. The control flow contains 5 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_parse_late_interaction_meta`，其作用域位于`LateInteractionRunner`。 关键调用包括 `ValueError`, `isinstance`, `max`, `int`。 控制流包含 5 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `LateInteractionRunner`: central class or interface in this module. / `LateInteractionRunner`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `collections`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.pooling_params`, `vllm.v1.outputs`, `vllm.v1.pool.late_interaction`
