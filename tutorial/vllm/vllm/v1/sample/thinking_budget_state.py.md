# thinking_budget_state.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/sample/thinking_budget_state.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Per-batch thinking token budget state; applied after penalties at sample time. / 该模块位于 `sample` 子系统，主要围绕 `maybe_create_thinking_budget_state_holder`, `ThinkingBudgetStateHolder` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""Per-batch thinking token budget state; applied after penalties at sample time."""

from typing import TYPE_CHECKING, Any

import torch

from vllm.v1.sample.logits_processor.interface import (
    BatchUpdate,
    MoveDirectionality,
)

if TYPE_CHECKING:
    from vllm.config.reasoning import ReasoningConfig
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。

### `maybe_create_thinking_budget_state_holder` function / `maybe_create_thinking_budget_state_holder` 函数
```python
def maybe_create_thinking_budget_state_holder(
    reasoning_config: "ReasoningConfig | None",
    max_num_seqs: int,
    num_spec_tokens: int,
    device: torch.device,
    is_pin_memory: bool,
) -> "ThinkingBudgetStateHolder | None":
    if reasoning_config is None:
        return None
    return ThinkingBudgetStateHolder(
        reasoning_config, max_num_seqs, num_spec_tokens, device, is_pin_memory
    )
```
**EN:** This function applies logic conditionally within the module. Key calls include `ThinkingBudgetStateHolder`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会按条件执行逻辑，其作用域位于the module。 关键调用包括 `ThinkingBudgetStateHolder`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `ThinkingBudgetStateHolder` class / `ThinkingBudgetStateHolder` 类
```python
class ThinkingBudgetStateHolder:
    """Tracks thinking sections and forces end tokens when budget is exceeded."""

    think_start_token_ids: list[int]
    think_end_token_ids: list[int]
```
**EN:** Introduces the `ThinkingBudgetStateHolder` class. Core methods include `__init__`, `has_tracked_requests`, `sync_batch`, `update_state`, `apply_to_logits`, `_find_last_sequence_index`. Docstring signal: Tracks thinking sections and forces end tokens when budget is exceeded.
**CN:** 这里定义 `ThinkingBudgetStateHolder` 类。核心方法包括 `__init__`, `has_tracked_requests`, `sync_batch`, `update_state`, `apply_to_logits`, `_find_last_sequence_index`。

### `ThinkingBudgetStateHolder.__init__` method / `ThinkingBudgetStateHolder.__init__` 方法
```python
    def __init__(
        self,
        reasoning_config: "ReasoningConfig | None",
        max_num_seqs: int,
        num_spec_tokens: int,
        device: torch.device,
        is_pin_memory: bool,
    ):
        _ = is_pin_memory  # API parity with logits processors
        max_num_reqs = max_num_seqs
        self.in_spec_mode = num_spec_tokens > 0
        self.num_spec_tokens = num_spec_tokens

        # No separate enable flag: a non-``None`` ``reasoning_config`` is the switch.
        self.is_enabled = reasoning_config is not None

        if reasoning_config is None:
            self.think_start_token_ids = []
            self.think_end_token_ids = []
        else:
            rs = reasoning_config.reasoning_start_token_ids
            re = reasoning_config.reasoning_end_token_ids
            self.think_start_token_ids = rs if rs else []
            self.think_end_token_ids = re if re else []

        self.device = device
        self._state: dict[int, dict[str, Any]] = {}
        self.cu_num_tokens: dict[int, int] = {}

        if self.num_spec_tokens > 0:
            self.mask = torch.zeros(
                max_num_reqs * (self.num_spec_tokens + 1),
                dtype=torch.bool,
                device=device,
            )
            self.force_token_ids = torch.full(
                (max_num_reqs * (self.num_spec_tokens + 1),),
                -1,
                dtype=torch.long,
                device=device,
            )
        else:
            self.mask = torch.zeros(max_num_reqs, dtype=torch.bool, device=device)
            self.force_token_ids = torch.full(
                (max_num_reqs,), -1, dtype=torch.long, device=device
            )
```
**EN:** This method initializes the object state within `ThinkingBudgetStateHolder`. Key calls include `zeros`, `full`. It touches state such as `in_spec_mode`, `num_spec_tokens`, `is_enabled`, `device`, `_state`, `cu_num_tokens`, `think_start_token_ids`, `think_end_token_ids`. The control flow contains 4 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`ThinkingBudgetStateHolder`。 关键调用包括 `zeros`, `full`。 它会读写 `in_spec_mode`, `num_spec_tokens`, `is_enabled`, `device`, `_state`, `cu_num_tokens`, `think_start_token_ids`, `think_end_token_ids` 等状态。 控制流包含 4 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `ThinkingBudgetStateHolder.update_state` method / `ThinkingBudgetStateHolder.update_state` 方法
```python
    def update_state(
        self,
        output_token_ids: list[list[int]],
        spec_token_ids: list[list[int]] | None,
        repeat_indices: torch.Tensor | None = None,
    ) -> None:
        """Refresh output/spec from sampling rows and recompute think state."""
        if not self.is_enabled or not self._state:
            return

        spec_lists = spec_token_ids or []
        last_row_for_req: dict[int, int] | None = None
        if repeat_indices is not None:
            last_row_for_req = {}
            rpt = repeat_indices.cpu().tolist()
            for batch_row, req_i in enumerate(rpt):
                last_row_for_req[req_i] = batch_row

        for seq_idx, state in list(self._state.items()):
            if last_row_for_req is not None:
                output_row: int | None = last_row_for_req.get(seq_idx)
                if output_row is None or output_row >= len(output_token_ids):
                    continue
                state["output_tok_ids"] = output_token_ids[output_row]
            elif seq_idx >= len(output_token_ids):
                continue
            else:
                state["output_tok_ids"] = output_token_ids[seq_idx]
            if seq_idx < len(spec_lists):
                state["spec_token_ids"] = list(spec_lists[seq_idx])
            else:
                state["spec_token_ids"] = []
            state["in_spec_mode"] = self.in_spec_mode
            state["force_index"] = []
            if len(state["output_tok_ids"]) > 0:
                spec_len = len(state["spec_token_ids"])
                # Only strip draft suffix when there are spec tokens; ``[:-0]`` would
                # clear the whole list (Python treats stop index 0 as "up to empty").
                if spec_len > 0 and len(state["output_tok_ids"]) >= spec_len:
                    state["output_tok_ids"] = state["output_tok_ids"][:-spec_len]
            self._update_think_state(state)
```
**EN:** This method updates existing state within `ThinkingBudgetStateHolder`. The docstring frames it as: Refresh output/spec from sampling rows and recompute think state. Key calls include `list`, `tolist`, `enumerate`, `items`, `_update_think_state`, `get`. The control flow contains 8 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会更新现有状态，其作用域位于`ThinkingBudgetStateHolder`。 关键调用包括 `list`, `tolist`, `enumerate`, `items`, `_update_think_state`, `get`。 控制流包含 8 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `ThinkingBudgetStateHolder._update_think_state` method / `ThinkingBudgetStateHolder._update_think_state` 方法
```python
    def _update_think_state(self, state: dict[str, Any]) -> None:
        if state.get("thinking_token_budget", -1) == -1:
            return
        if len(self.think_end_token_ids) == 0:
            state["thinking_token_budget"] = -1
            state["in_end"] = False
            state["force_index"] = []
            return

        if state["start_thinking"] == -1:
            start_thinking = self._find_last_sequence_index(
                state.get("output_tok_ids", []), self.think_start_token_ids
            )
            state["start_thinking"] = start_thinking
        if state["end_thinking"] == -1:
            end_thinking = self._find_last_sequence_index(
                state.get("output_tok_ids", []), self.think_end_token_ids
            )
            state["end_thinking"] = end_thinking

        if state["start_thinking"] == -1:
            return

        if state["continue_thinking"]:
            sampled_tokens_from_previous_step = len(
                state.get("output_tok_ids", [])
            ) - state.get("prev_output_length", 0)
        else:
            if state["prev_output_length"] == 0:
                sampled_tokens_from_previous_step = len(
                    state.get("output_tok_ids", [])
                ) - len(self.think_start_token_ids)
            else:
                sampled_tokens_from_previous_step = (
                    len(state.get("output_tok_ids", [])) - state["prev_output_length"]
                )
        current_step_countdown = (
            state["check_count_down"] - sampled_tokens_from_previous_step
        )
        predicted_countdown = current_step_countdown - len(state["spec_token_ids"]) - 1
        # We only proceed further if we have counted down the thinking budget
        # to 0 or less and when we are in the "in think" mode.
        if (
            not state.get("in_end", False)
            and predicted_countdown >= 0
            and state["start_thinking"] > -1
        ):
            state["check_count_down"] = current_step_countdown
            state["prev_output_length"] = len(state.get("output_tok_ids", []))
            return
    # ... omitted for brevity ...
            if len(state["spec_token_ids"]) > 0:
                for i, token_id in enumerate(state["spec_token_ids"]):
                    if state["end_count"] + 1 < len(self.think_end_token_ids):
                        if token_id == self.think_end_token_ids[state["end_count"] + 1]:
                            state["end_count"] += 1
                        else:
                            state["end_count"] += 1
                            state["force_index"] = [i]
                            break
                    else:
                        state["end_count"] += 1
                if len(state["force_index"]) == 0:
                    state["end_count"] += 1
                    state["force_index"] = [len(state["spec_token_ids"])]
            else:
                state["end_count"] += 1
                state["force_index"] = [0]
            if state["end_count"] >= len(self.think_end_token_ids):
                state.update(
                    {
                        "in_end": False,
                        "end_count": 0,
                        "check_count_down": state["thinking_token_budget"],
                    }
                )
```
**EN:** This method implements `_update_think_state` within `ThinkingBudgetStateHolder`. Key calls include `get`, `len`, `_find_last_sequence_index`, `max`, `enumerate`, `update`. The control flow contains 33 branch(es) and 1 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `_update_think_state`，其作用域位于`ThinkingBudgetStateHolder`。 关键调用包括 `get`, `len`, `_find_last_sequence_index`, `max`, `enumerate`, `update`。 控制流包含 33 个分支和 1 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

## Key Concepts / 关键概念
- `maybe_create_thinking_budget_state_holder`: top-level helper or orchestration entry point. / `maybe_create_thinking_budget_state_holder`：顶层辅助函数或编排入口。
- `ThinkingBudgetStateHolder`: central class or interface in this module. / `ThinkingBudgetStateHolder`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.v1.sample.logits_processor.interface`, `vllm.config.reasoning`
