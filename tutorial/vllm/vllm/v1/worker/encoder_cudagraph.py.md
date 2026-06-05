# encoder_cudagraph.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/encoder_cudagraph.py`
- **Repository**: vllm-project/vllm
- **Purpose**: CUDA graph manager for vision encoder budget-batch execution. / 该模块位于 `worker` 子系统，主要围绕 `BudgetGraphMetadata`, `EncoderCudaGraphManager` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""CUDA graph manager for vision encoder budget-batch execution."""

from dataclasses import dataclass
from typing import Any

import torch

from vllm.config import VllmConfig
from vllm.distributed import (
    get_tensor_model_parallel_rank,
    get_tensor_model_parallel_world_size,
    tensor_model_parallel_all_gather,
)
from vllm.logger import init_logger
from vllm.model_executor.models.interfaces import (
    SupportsEncoderCudaGraph,
)
from vllm.model_executor.models.utils import scatter_output_slices
from vllm.model_executor.models.vision import get_load_balance_assignment
from vllm.v1.worker.encoder_cudagraph_defs import (
    EncoderCudaGraphConfig,
)

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `BudgetGraphMetadata` class / `BudgetGraphMetadata` 类
```python
@dataclass
class BudgetGraphMetadata:
    """Metadata for a single budget graph.

    CUDA graph replay pattern:
    1. Copy new batch data into input_buffer (e.g. pixel_values)
    2. Copy precomputed values into metadata_buffers
    3. Replay graph
    4. Read encoder outputs from output_buffer
    """

    token_budget: int
    max_batch_size: int  # Max number of images/videos per batch
    max_frames_per_batch: int  # Max total frames per batch (for video)
    graph: torch.cuda.CUDAGraph
    # The input tensor updated before replay (e.g. pixel_values)
    input_buffer: torch.Tensor
    # Buffers recorded into the CUDA graph (e.g. embeddings, sequence metadata).
    # Before replay the manager zeros then slice-copies new data into these.
    metadata_buffers: dict[str, torch.Tensor]
    # Output written by graph, read after replay
    output_buffer: torch.Tensor
```
**EN:** Uses `@dataclass` to package related state for `BudgetGraphMetadata`. Typical fields include `token_budget`, `max_batch_size`, `max_frames_per_batch`, `graph`, `input_buffer`, `metadata_buffers`.
**CN:** `BudgetGraphMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `token_budget`, `max_batch_size`, `max_frames_per_batch`, `graph`, `input_buffer`, `metadata_buffers`。

### `EncoderCudaGraphManager` class / `EncoderCudaGraphManager` 类
```python
class EncoderCudaGraphManager:
    """Budget-based CUDA graph capture/replay for vision encoders."""
```
**EN:** Introduces the `EncoderCudaGraphManager` class. Core methods include `__init__`, `_generate_budgets`, `supports_modality`, `capture`, `_capture_budget_graph`, `_find_smallest_fitting_budget_given_tokens`. Docstring signal: Budget-based CUDA graph capture/replay for vision encoders.
**CN:** 这里定义 `EncoderCudaGraphManager` 类。核心方法包括 `__init__`, `_generate_budgets`, `supports_modality`, `capture`, `_capture_budget_graph`, `_find_smallest_fitting_budget_given_tokens`。

### `EncoderCudaGraphManager.__init__` method / `EncoderCudaGraphManager.__init__` 方法
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        device: torch.device,
        dtype: torch.dtype,
        model: SupportsEncoderCudaGraph,
    ):
        """Initialize CUDA graph manager with provided token budgets
        and max batch size."""
        self.vllm_config = vllm_config
        self.device = device
        self.dtype = dtype
        self.model = model
        self.config: EncoderCudaGraphConfig = model.get_encoder_cudagraph_config()

        comp_config = vllm_config.compilation_config
        user_budgets = comp_config.encoder_cudagraph_token_budgets
        user_max_vision_items = comp_config.encoder_cudagraph_max_vision_items_per_batch
        user_max_frames = comp_config.encoder_cudagraph_max_frames_per_batch

        multimodal_config = vllm_config.model_config.multimodal_config

        # Invariant: max_batch_size <= min_token_budget.
        # This ensures per_image_output = budget // max_batch_size >= 1
        # for every captured budget, preventing reshape crashes on empty
        # tensors during CUDA graph capture. Validated/enforced below for
        # each configuration path.
        if user_budgets and user_max_vision_items > 0:
            # Fully user-specified: validate the invariant.
            self.token_budgets = sorted(user_budgets)
            self.max_batch_size = user_max_vision_items
            min_tok = min(self.token_budgets)
            if self.max_batch_size > min_tok:
                raise ValueError(
                    f"encoder_cudagraph_max_vision_items_per_batch "
                    f"({self.max_batch_size}) must be <= smallest token "
                    f"budget ({min_tok}). With budgets="
                    f"{self.token_budgets}, per_image_output = "
                    f"{min_tok} // {self.max_batch_size} = "
                    f"{min_tok // self.max_batch_size}, which would cause "
                    f"a capture failure. Either increase the smallest "
                    f"budget or decrease max_vision_items_per_batch."
                )
        else:
            # Auto-infer missing values from model.
            min_budget, max_budget = model.get_encoder_cudagraph_budget_range(
                vllm_config
            )
            if min_budget <= 0 or max_budget <= 0:
                raise ValueError(
    # ... omitted for brevity ...
        else:
            # Set it to the model-specific value according to its `processing_info`.
            max_frames_per_video = self.model.get_max_frames_per_video()
            self.max_frames_per_batch = self.max_batch_size * max_frames_per_video

        mm_config = vllm_config.model_config.multimodal_config
        self.use_dp = (
            mm_config is not None
            and mm_config.mm_encoder_tp_mode == "data"
            and vllm_config.parallel_config.tensor_parallel_size > 1
        )

        self.budget_graphs: dict[int, BudgetGraphMetadata] = {}
        self.graph_hits = 0
        self.graph_misses = 0
        self.log_stats_interval = 100

        logger.info(
            "EncoderCudaGraphManager initialized with "
            "budgets=%s, max_batch_size=%d, max_frames_per_batch=%s, use_dp=%s",
            self.token_budgets,
            self.max_batch_size,
            self.max_frames_per_batch,
            self.use_dp,
        )
```
**EN:** This method initializes the object state within `EncoderCudaGraphManager`. The docstring frames it as: Initialize CUDA graph manager with provided token budgets and max batch size. Key calls include `get_encoder_cudagraph_config`, `info`, `sorted`, `min`, `get_encoder_cudagraph_budget_range`, `get_limit_per_prompt`. It touches state such as `vllm_config`, `device`, `dtype`, `model`, `config`, `use_dp`, `budget_graphs`, `graph_hits`. The control flow contains 8 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会初始化对象状态，其作用域位于`EncoderCudaGraphManager`。 关键调用包括 `get_encoder_cudagraph_config`, `info`, `sorted`, `min`, `get_encoder_cudagraph_budget_range`, `get_limit_per_prompt`。 它会读写 `vllm_config`, `device`, `dtype`, `model`, `config`, `use_dp`, `budget_graphs`, `graph_hits` 等状态。 控制流包含 8 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `EncoderCudaGraphManager._execute_local` method / `EncoderCudaGraphManager._execute_local` 方法
```python
    def _execute_local(
        self,
        mm_kwargs: dict[str, Any],
    ) -> list[torch.Tensor]:
        """Execute encoder on local inputs using greedy-packed CUDA graphs.

        Sort images by output token count (smallest first), then greedily pack
        as many images as possible into each batch while staying within
        max_budget tokens and max_batch_size. Once a batch is finalised (next
        image would overflow either constraint), find the smallest fitting
        budget once for that batch.

        By exchange argument, greedy smallest-first packing minimises eager
        fallbacks -- any other ordering yields a higher token sum in some batch,
        making that batch more likely to exceed the budget.

        Stats note:
          graph_hits  -- counted inside _run_budget_graph after successful replay.
          graph_misses -- counted here for single-image batches where the image
                         exceeds max_budget. Batches split due to max_batch_size
                         always satisfy total_tokens <= max_budget and therefore
                         always find a valid budget (no miss).
        """
        num_items = self.model.get_encoder_cudagraph_num_items(mm_kwargs)
        max_budget = self.token_budgets[-1]

        per_item_out_tokens = self._get_per_item_out_tokens(mm_kwargs)

        # Sort ascending by output token count (smallest first)
        sorted_indices = sorted(range(num_items), key=lambda i: per_item_out_tokens[i])

        # Greedy pack against max_budget and max_batch_size.
        # _find_smallest_fitting_budget_given_tokens is called once per
        # finalised batch, not per image.
        batches: list[tuple[list[int], int | None]] = []
        current_batch: list[int] = []
        current_batch_tokens = 0

        for orig_idx in sorted_indices:
            item_tokens = per_item_out_tokens[orig_idx]
            if (
                current_batch_tokens + item_tokens <= max_budget
                and len(current_batch) < self.max_batch_size
            ):
                current_batch.append(orig_idx)
                current_batch_tokens += item_tokens
            else:
                if current_batch:
                    batches.append(
                        (
    # ... omitted for brevity ...
                    token_budget,
                    (token_budget - batch_out_tokens) / token_budget * 100,
                )
                replay = self.model.prepare_encoder_cudagraph_replay_buffers(
                    batch_mm_kwargs,
                    self.max_batch_size,
                    self.max_frames_per_batch,
                )

                # graph_hits counted inside _run_budget_graph after replay.
                output = self._run_budget_graph(
                    batch_mm_kwargs, token_budget, replay.buffers
                )
                assert output is not None
                self.model.postprocess_encoder_output(
                    output,
                    batch_orig_indices,
                    per_item_out_tokens,
                    outputs_by_orig_idx,
                    clone=True,
                    batch_mm_kwargs=batch_mm_kwargs,
                )

        # Return in original batch order (caller maps outputs to token positions)
        return [outputs_by_orig_idx[i] for i in range(num_items)]
```
**EN:** This method implements `_execute_local` within `EncoderCudaGraphManager`. The docstring frames it as: Execute encoder on local inputs using greedy-packed CUDA graphs. Key calls include `get_encoder_cudagraph_num_items`, `_get_per_item_out_tokens`, `sorted`, `range`, `append`, `select_encoder_cudagraph_items`. It touches state such as `graph_misses`. The control flow contains 4 branch(es) and 2 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `_execute_local`，其作用域位于`EncoderCudaGraphManager`。 关键调用包括 `get_encoder_cudagraph_num_items`, `_get_per_item_out_tokens`, `sorted`, `range`, `append`, `select_encoder_cudagraph_items`。 它会读写 `graph_misses` 等状态。 控制流包含 4 个分支和 2 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `EncoderCudaGraphManager.execute` method / `EncoderCudaGraphManager.execute` 方法
```python
    def execute(
        self,
        mm_kwargs: dict[str, Any],
    ) -> list[torch.Tensor]:
        """Execute encoder using CUDA graph with optional DP.

        Args:
            mm_kwargs: Multimodal keyword arguments containing the
                input tensor and grid dimensions.

        Returns:
            List of encoder outputs (one per item).
        """
        if self.use_dp:
            per_item_out_tokens = self._get_per_item_out_tokens(mm_kwargs)

            (
                local_mm_kwargs,
                image_rank_assignment,
                images_per_rank,
                max_output_tokens_per_rank,
            ) = self._dp_shard(mm_kwargs, per_item_out_tokens)

            local_outputs = self._execute_local(local_mm_kwargs)

            result = self._dp_gather(
                local_outputs,
                per_item_out_tokens,
                image_rank_assignment,
                images_per_rank,
                max_output_tokens_per_rank,
            )
        else:
            result = self._execute_local(mm_kwargs)

        # Log cumulative stats periodically
        stats = self.get_cumulative_stats()
        total_requests = self.graph_hits + self.graph_misses
        if total_requests > 0 and total_requests % self.log_stats_interval == 0:
            logger.debug(
                "Encoder CUDA graph cumulative stats: "
                "hits=%d, misses=%d, hit_rate=%.1f%%",
                stats["graph_hits"],
                stats["graph_misses"],
                stats["hit_rate"] * 100,
            )

        return result
```
**EN:** This method executes the main operation within `EncoderCudaGraphManager`. The docstring frames it as: Execute encoder using CUDA graph with optional DP. Key calls include `get_cumulative_stats`, `_get_per_item_out_tokens`, `_dp_shard`, `_execute_local`, `_dp_gather`, `debug`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会执行主要操作，其作用域位于`EncoderCudaGraphManager`。 关键调用包括 `get_cumulative_stats`, `_get_per_item_out_tokens`, `_dp_shard`, `_execute_local`, `_dp_gather`, `debug`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `BudgetGraphMetadata`: central class or interface in this module. / `BudgetGraphMetadata`：本模块中的核心类或接口。
- `EncoderCudaGraphManager`: central class or interface in this module. / `EncoderCudaGraphManager`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`, `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.distributed`, `vllm.logger`, `vllm.model_executor.models.interfaces`, `vllm.model_executor.models.utils`, `vllm.model_executor.models.vision`, `vllm.v1.worker.encoder_cudagraph_defs`
