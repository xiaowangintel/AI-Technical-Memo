# ngram_proposer_gpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/spec_decode/ngram_proposer_gpu.py`
- **Repository**: vllm-project/vllm
- **Purpose**: GPU-accelerated N-gram proposer using fully async PyTorch tensor operations. / 该模块位于 `spec_decode` 子系统，主要围绕 `NgramGPUKernel`, `NgramProposerGPU`, `update_scheduler_for_invalid_drafts` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""
GPU-accelerated N-gram proposer using fully async PyTorch tensor operations.

This version uses a fully vectorized approach with unfold and argmax for
finding the first match across all sequences in parallel.
"""

import torch
from torch import nn

from vllm.compilation.decorators import support_torch_compile
from vllm.config import (
    CompilationConfig,
    CompilationMode,
    CUDAGraphMode,
    VllmConfig,
)
from vllm.forward_context import set_forward_context
from vllm.utils.torch_utils import async_tensor_h2d
from vllm.v1.core.sched.output import SchedulerOutput
from vllm.v1.utils import record_function_or_nullcontext
from vllm.v1.worker.gpu_input_batch import CachedRequestState, InputBatch
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。

### `NgramGPUKernel` class / `NgramGPUKernel` 类
```python
@support_torch_compile()
class NgramGPUKernel(nn.Module):
    """GPU-accelerated N-gram proposer using fully async tensor operations."""
```
**EN:** Introduces the `NgramGPUKernel` class on top of `nn.Module`. Core methods include `__init__`, `_find_first_and_extract_all_n_parallel`, `forward`, `load_model`. Docstring signal: GPU-accelerated N-gram proposer using fully async tensor operations.
**CN:** 这里定义 `NgramGPUKernel` 类，其基类包括 `nn.Module`。核心方法包括 `__init__`, `_find_first_and_extract_all_n_parallel`, `forward`, `load_model`。

### `NgramGPUKernel.__init__` method / `NgramGPUKernel.__init__` 方法
```python
    def __init__(
        self, vllm_config: VllmConfig, prefix: str = "", device: torch.device = "cuda"
    ):
        super().__init__()

        assert vllm_config.speculative_config is not None
        assert vllm_config.speculative_config.prompt_lookup_min is not None
        assert vllm_config.speculative_config.prompt_lookup_max is not None

        self.min_n = vllm_config.speculative_config.prompt_lookup_min
        self.max_n = vllm_config.speculative_config.prompt_lookup_max
        self.k = vllm_config.speculative_config.num_speculative_tokens
        self.max_model_len = vllm_config.model_config.max_model_len
        self.max_num_seqs = vllm_config.scheduler_config.max_num_seqs
        self.device = device
```
**EN:** This method initializes the object state within `NgramGPUKernel`. Key calls include `__init__`, `super`. It touches state such as `min_n`, `max_n`, `k`, `max_model_len`, `max_num_seqs`, `device`.
**CN:** 该方法会初始化对象状态，其作用域位于`NgramGPUKernel`。 关键调用包括 `__init__`, `super`。 它会读写 `min_n`, `max_n`, `k`, `max_model_len`, `max_num_seqs`, `device` 等状态。

### `NgramGPUKernel._find_first_and_extract_all_n_parallel` method / `NgramGPUKernel._find_first_and_extract_all_n_parallel` 方法
```python
    def _find_first_and_extract_all_n_parallel(
        self,
        token_ids: torch.Tensor,
        seq_lengths: torch.Tensor,
        min_ngram_len: int,
        max_ngram_len: int,
        num_draft_tokens: int,
    ) -> torch.Tensor:
        """
        Find suffix n-gram matches and extract following tokens.
        Searches for the earliest prior occurrence of the trailing n-gram,
        tries multiple lengths, and picks the longest valid match.

        Args:
            token_ids: Token IDs for each sequence
            seq_lengths: Actual length of each sequence (excluding padding)
            min_ngram_len: Minimum n-gram size to search for (e.g., 2)
            max_ngram_len: Maximum n-gram size to search for (e.g., 5)
            num_draft_tokens: Number of tokens to extract after match (k)

        Returns:
            Draft token predictions; -1 means invalid/no match.
        """
        batch_size = token_ids.shape[0]
        max_seq_len = token_ids.shape[1]
        device = token_ids.device
        num_ngram_sizes = max_ngram_len - min_ngram_len + 1

        # All n-gram sizes to try.
        ngram_lengths = torch.arange(min_ngram_len, max_ngram_len + 1, device=device)
        batch_indices = torch.arange(batch_size, device=device)

        # Earliest match per (sequence, ngram_len); -1 means no match.
        first_match_positions = torch.full(
            (batch_size, num_ngram_sizes), -1, dtype=torch.long, device=device
        )

        for i, ngram_len in enumerate(range(min_ngram_len, max_ngram_len + 1)):
            # Sliding windows of size ngram_len; unfold is O(1) view.
            search_windows = token_ids.unfold(1, ngram_len, 1)
            num_windows = search_windows.shape[1]

            # Trailing suffix (last ngram_len tokens) for each sequence.
            suffix_starts = seq_lengths - ngram_len
            suffix_indices = suffix_starts.unsqueeze(1) + torch.arange(
                ngram_len, device=device
            )
            suffix = torch.gather(token_ids, 1, suffix_indices.clamp(min=0))

            # Window matches for each sequence.
    # ... omitted for brevity ...
            num_draft_tokens, device=device
        )
        draft_indices = draft_indices.clamp(min=0, max=max_seq_len - 1)

        # Extract draft tokens; gather always runs.
        draft_tokens = torch.gather(token_ids, 1, draft_indices)

        # Mask positions beyond available tokens.
        position_indices = torch.arange(num_draft_tokens, device=device).unsqueeze(0)
        valid_positions = position_indices < tokens_available.unsqueeze(1)

        draft_tokens = torch.where(
            valid_positions,
            draft_tokens,
            torch.full_like(draft_tokens, -1),
        )

        # If no match, mask all positions.
        draft_tokens = torch.where(
            has_any_match.unsqueeze(1),
            draft_tokens,
            torch.full_like(draft_tokens, -1),
        )

        return draft_tokens
```
**EN:** This method implements `_find_first_and_extract_all_n_parallel` within `NgramGPUKernel`. The docstring frames it as: Find suffix n-gram matches and extract following tokens. Key calls include `arange`, `full`, `enumerate`, `argmax`, `where`, `clamp`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会实现 `_find_first_and_extract_all_n_parallel`，其作用域位于`NgramGPUKernel`。 关键调用包括 `arange`, `full`, `enumerate`, `argmax`, `where`, `clamp`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `NgramGPUKernel.forward` method / `NgramGPUKernel.forward` 方法
```python
    def forward(
        self,
        num_tokens_no_spec: torch.Tensor,
        token_ids_gpu: torch.Tensor,
        combined_mask: torch.Tensor,
    ) -> tuple[torch.Tensor, torch.Tensor]:
        """
        Forward pass for N-gram proposal using GPU tensor operations.

        Args:
            num_tokens_no_spec: Number of tokens for each sequence [batch_size]
            token_ids_gpu: Token IDs [batch_size, max_len]
            combined_mask: Whether each sequence is valid for spec decode [batch_size]

        Returns:
            draft_tokens: [batch_size, k] on GPU
            num_valid_draft_tokens: [batch_size] int32 on GPU, count of
                leading valid (non -1) tokens per request.
        """

        device = token_ids_gpu.device

        # Infer batch size to preserve dynamic shape.
        actual_batch_size = token_ids_gpu.shape[0]

        # Allocate in forward so torch.compile can optimize.
        # NOTE(patchy): Do NOT pre-allocate this as a buffer
        #               it breaks torch.compile
        draft_tokens = torch.full(
            (actual_batch_size, self.k), -1, dtype=torch.int32, device=device
        )

        results = self._find_first_and_extract_all_n_parallel(
            token_ids_gpu,
            num_tokens_no_spec,
            min_ngram_len=self.min_n,
            max_ngram_len=self.max_n,
            num_draft_tokens=self.k,
        )

        draft_tokens = torch.where(combined_mask.unsqueeze(1), results, -1)

        # Count leading contiguous valid (non -1) tokens per request.
        is_valid = draft_tokens != -1  # [batch, k]
        cum_valid = is_valid.int().cumsum(dim=1)  # [batch, k]
        positions = torch.arange(1, self.k + 1, device=device).unsqueeze(0)
        num_valid_draft_tokens = (cum_valid == positions).int().sum(dim=1)

        return draft_tokens, num_valid_draft_tokens
```
**EN:** This method drives the forward-pass computation within `NgramGPUKernel`. The docstring frames it as: Forward pass for N-gram proposal using GPU tensor operations. Key calls include `full`, `_find_first_and_extract_all_n_parallel`, `where`, `cumsum`, `unsqueeze`, `sum`.
**CN:** 该方法会驱动前向计算流程，其作用域位于`NgramGPUKernel`。 关键调用包括 `full`, `_find_first_and_extract_all_n_parallel`, `where`, `cumsum`, `unsqueeze`, `sum`。

### `NgramProposerGPU` class / `NgramProposerGPU` 类
```python
class NgramProposerGPU:
```
**EN:** Introduces the `NgramProposerGPU` class. Core methods include `__init__`, `_dummy_run`, `_generate_dummy_data`, `propose`, `update_token_ids_ngram`, `load_model`.
**CN:** 这里定义 `NgramProposerGPU` 类。核心方法包括 `__init__`, `_dummy_run`, `_generate_dummy_data`, `propose`, `update_token_ids_ngram`, `load_model`。

### `NgramProposerGPU.__init__` method / `NgramProposerGPU.__init__` 方法
```python
    def __init__(self, vllm_config: VllmConfig, device: torch.device, runner=None):
        assert vllm_config.speculative_config is not None
        assert vllm_config.speculative_config.prompt_lookup_min is not None
        assert vllm_config.speculative_config.prompt_lookup_max is not None

        compilation_config = CompilationConfig(
            mode=CompilationMode.VLLM_COMPILE,
            custom_ops=["none"],
            splitting_ops=[],
            compile_sizes=[],
            inductor_compile_config={
                "enable_auto_functionalized_v2": False,
                "max_autotune": True,
                "aggressive_fusion": True,
                "triton.autotune_pointwise": True,
                "coordinate_descent_tuning": True,
                "use_mixed_mm": False,
            },
            cudagraph_mode=CUDAGraphMode.NONE,
        )
        model_config = vllm_config.model_config
        speculative_config = vllm_config.speculative_config
        scheduler_config = vllm_config.scheduler_config

        self.vllm_config = VllmConfig(
            compilation_config=compilation_config,
            model_config=model_config,
            speculative_config=speculative_config,
            scheduler_config=scheduler_config,
        )

        self.min_n = vllm_config.speculative_config.prompt_lookup_min
        self.max_n = vllm_config.speculative_config.prompt_lookup_max
        self.k = vllm_config.speculative_config.num_speculative_tokens
        self.max_model_len = vllm_config.model_config.max_model_len
        self.max_num_seqs = vllm_config.scheduler_config.max_num_seqs
        self.device = device

        self.kernel = NgramGPUKernel(
            vllm_config=self.vllm_config, prefix="ngram_gpu_kernel", device=device
        )
        self.kernel.to(device)
        self.kernel.eval()

        self._dummy_run()
```
**EN:** This method initializes the object state within `NgramProposerGPU`. Key calls include `CompilationConfig`, `VllmConfig`, `NgramGPUKernel`, `to`, `eval`, `_dummy_run`. It touches state such as `vllm_config`, `min_n`, `max_n`, `k`, `max_model_len`, `max_num_seqs`, `device`, `kernel`.
**CN:** 该方法会初始化对象状态，其作用域位于`NgramProposerGPU`。 关键调用包括 `CompilationConfig`, `VllmConfig`, `NgramGPUKernel`, `to`, `eval`, `_dummy_run`。 它会读写 `vllm_config`, `min_n`, `max_n`, `k`, `max_model_len`, `max_num_seqs`, `device`, `kernel` 等状态。

### `NgramProposerGPU.update_token_ids_ngram` method / `NgramProposerGPU.update_token_ids_ngram` 方法
```python
    def update_token_ids_ngram(
        self,
        sampled_token_ids: torch.Tensor | list[list[int]],
        gpu_input_batch: InputBatch,
        token_ids_gpu: torch.Tensor,
        num_tokens_no_spec: torch.Tensor,
        discard_request_mask: torch.Tensor,
    ) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
        """
        Prepare speculative decoding inputs on device:
        compute next token ids and valid counts, honoring discarded requests
        and rejected tokens, without CPU-GPU sync.
        """
        num_reqs = gpu_input_batch.num_reqs

        if isinstance(sampled_token_ids, list):
            # When disable_padded_drafter_batch=True, sampled_token_ids is
            # an irregular list[list[int]] where sublists may have different
            # lengths (including empty lists for discarded requests).
            # Pad all sublists to the same length with -1 before converting
            # to tensor.
            max_len = max(
                (len(sublist) for sublist in sampled_token_ids),
                default=0,
            )
            # Ensure at least length 1 for tensor creation
            max_len = max(max_len, 1)
            padded_list = [
                sublist + [-1] * (max_len - len(sublist))
                for sublist in sampled_token_ids
            ]
            sampled_token_ids = torch.tensor(
                padded_list, dtype=torch.int32, device=self.device
            )
        assert isinstance(sampled_token_ids, torch.Tensor), (
            "sampled_token_ids should be a torch.Tensor for ngram_gpu"
        )

        # Backup last valid token before speculative tokens.
        backup_indices = (num_tokens_no_spec[:num_reqs] - 1).clamp(min=0).long()
        backup_next_token_ids = torch.gather(
            token_ids_gpu[:num_reqs], dim=1, index=backup_indices.unsqueeze(1)
        ).squeeze(1)

        valid_sampled_token_ids_gpu = sampled_token_ids.clone()
        # Invalidate sampled tokens for discarded requests.
        discard_mask_expanded = discard_request_mask[:num_reqs].unsqueeze(1)
        valid_sampled_token_ids_gpu.masked_fill_(discard_mask_expanded, -1)

        # Mask valid tokens within each request.
        valid_mask = (valid_sampled_token_ids_gpu != -1) & (
            valid_sampled_token_ids_gpu < gpu_input_batch.vocab_size
        )

        # Count valid tokens per request.
        valid_sampled_tokens_count = valid_mask.sum(dim=1).to(torch.int32)

        # Rightmost valid index per row.
        last_valid_indices = valid_sampled_tokens_count - 1
        last_valid_indices_safe = torch.clamp(last_valid_indices, min=0)

        # Last valid token from each row; undefined if none.
        selected_tokens = torch.gather(
            valid_sampled_token_ids_gpu, 1, last_valid_indices_safe.unsqueeze(1)
        ).squeeze(1)

        # Use last token if valid; otherwise fallback to backup.
        next_token_ids = torch.where(
            last_valid_indices != -1,
            selected_tokens,
            backup_next_token_ids,
        )

        return next_token_ids, valid_sampled_tokens_count, valid_sampled_token_ids_gpu
```
**EN:** This method updates existing state within `NgramProposerGPU`. The docstring frames it as: Prepare speculative decoding inputs on device: compute next token ids and valid counts, honoring discarded requests and rejected tokens, without CPU-GPU sync. Key calls include `isinstance`, `long`, `squeeze`, `clone`, `unsqueeze`, `masked_fill_`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会更新现有状态，其作用域位于`NgramProposerGPU`。 关键调用包括 `isinstance`, `long`, `squeeze`, `clone`, `unsqueeze`, `masked_fill_`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `NgramProposerGPU.load_model` method / `NgramProposerGPU.load_model` 方法
```python
    def load_model(self, *args, **kwargs):
        self.kernel.load_model(*args, **kwargs)
```
**EN:** This method loads external or cached state within `NgramProposerGPU`. Key calls include `load_model`.
**CN:** 该方法会加载外部或缓存状态，其作用域位于`NgramProposerGPU`。 关键调用包括 `load_model`。

### `update_scheduler_for_invalid_drafts` function / `update_scheduler_for_invalid_drafts` 函数
```python
def update_scheduler_for_invalid_drafts(
    num_valid_draft_tokens_event: torch.cuda.Event,
    num_valid_draft_tokens_cpu: torch.Tensor,
    scheduler_output: "SchedulerOutput",
    req_id_to_index: dict[str, int],
) -> None:
    """Trim invalid speculative slots using per-request valid draft counts.

    Args:
        num_valid_draft_tokens_event: Event for async D2H completion.
        num_valid_draft_tokens_cpu: CPU buffer of valid draft counts.
        scheduler_output: Scheduler metadata to update in-place.
        req_id_to_index: Request-id to batch-index mapping.
    """
    req_data = scheduler_output.scheduled_cached_reqs
    num_valid_draft_tokens_event.synchronize()

    for req_id in req_data.req_ids:
        req_index = req_id_to_index.get(req_id)
        if req_index is None:
            continue

        spec_token_ids = scheduler_output.scheduled_spec_decode_tokens.get(req_id)
        if spec_token_ids is None:
            continue

        scheduled_k = len(spec_token_ids)

        valid_k = int(num_valid_draft_tokens_cpu[req_index].item())
        valid_k = max(0, min(valid_k, scheduled_k))

        tokens_to_trim = scheduled_k - valid_k
        scheduler_output.total_num_scheduled_tokens -= tokens_to_trim
        scheduler_output.num_scheduled_tokens[req_id] -= tokens_to_trim

        if valid_k == 0:
            scheduler_output.scheduled_spec_decode_tokens.pop(req_id, None)
        else:
            scheduler_output.scheduled_spec_decode_tokens[req_id] = spec_token_ids[
                :valid_k
            ]
```
**EN:** This function updates existing state within the module. The docstring frames it as: Trim invalid speculative slots using per-request valid draft counts. Key calls include `synchronize`, `get`, `len`, `int`, `max`, `item`. The control flow contains 3 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会更新现有状态，其作用域位于the module。 关键调用包括 `synchronize`, `get`, `len`, `int`, `max`, `item`。 控制流包含 3 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `update_ngram_gpu_tensors_incremental` function / `update_ngram_gpu_tensors_incremental` 函数
```python
def update_ngram_gpu_tensors_incremental(
    input_batch: InputBatch,
    token_ids_gpu_tensor: torch.Tensor,
    num_tokens_no_spec_gpu: torch.Tensor,
    new_reqs: list[CachedRequestState],
    device: torch.device,
    _pinned_idx_buf: torch.Tensor,
    _pinned_val_buf: torch.Tensor,
) -> None:
    """Incrementally update token_ids_gpu_tensor and num_tokens_no_spec_gpu
    for ngram GPU proposer.
    """
    prev_req_id_to_index = input_batch.prev_req_id_to_index
    curr_req_id_to_index = input_batch.req_id_to_index

    if not curr_req_id_to_index:
        return

    active_indices = list(curr_req_id_to_index.values())
    n_active = len(active_indices)

    # Use resident pinned buffers to avoid per-call allocation.
    active_idx_cpu = _pinned_idx_buf[:n_active]
    active_idx_cpu.copy_(torch.as_tensor(active_indices, dtype=torch.long))

    active_idx_gpu = active_idx_cpu.to(device=device, non_blocking=True)

    new_req_ids = {req.req_id for req in new_reqs}

    # First run, no previous state.
    if prev_req_id_to_index is None:
        for idx in active_indices:
            num_tokens = input_batch.num_tokens_no_spec[idx]
            if num_tokens > 0:
                token_ids_gpu_tensor[idx, :num_tokens].copy_(
                    input_batch.token_ids_cpu_tensor[idx, :num_tokens],
                    non_blocking=True,
                )

        _sync_num_tokens(
            input_batch,
            num_tokens_no_spec_gpu,
            active_idx_cpu,
            active_idx_gpu,
            n_active,
            device,
            _pinned_val_buf,
        )
        return

# ... omitted for brevity ...
        num_tokens_no_spec_gpu[dst_tensor] = temp_num_tokens

    # Full copy for new/resumed requests.
    for req_state in new_reqs:
        new_req_idx = curr_req_id_to_index.get(req_state.req_id)
        if new_req_idx is None:
            continue

        num_tokens = input_batch.num_tokens_no_spec[new_req_idx]
        if num_tokens > 0:
            token_ids_gpu_tensor[new_req_idx, :num_tokens].copy_(
                input_batch.token_ids_cpu_tensor[new_req_idx, :num_tokens],
                non_blocking=True,
            )

    # Always batch-sync sequence lengths from CPU for ALL active requests.
    _sync_num_tokens(
        input_batch,
        num_tokens_no_spec_gpu,
        active_idx_cpu,
        active_idx_gpu,
        n_active,
        device,
        _pinned_val_buf,
    )
```
**EN:** This function updates existing state within the module. The docstring frames it as: Incrementally update token_ids_gpu_tensor and num_tokens_no_spec_gpu for ngram GPU proposer. Key calls include `list`, `len`, `copy_`, `to`, `items`, `_sync_num_tokens`. The control flow contains 8 branch(es) and 3 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会更新现有状态，其作用域位于the module。 关键调用包括 `list`, `len`, `copy_`, `to`, `items`, `_sync_num_tokens`。 控制流包含 8 个分支和 3 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `_sync_num_tokens` function / `_sync_num_tokens` 函数
```python
def _sync_num_tokens(
    input_batch: InputBatch,
    num_tokens_no_spec_gpu: torch.Tensor,
    active_idx_cpu: torch.Tensor,
    active_idx_gpu: torch.Tensor,
    n_active: int,
    device: torch.device,
    _pinned_val_buf: torch.Tensor,
) -> None:
    """Batch-sync GPU sequence lengths from CPU source of truth.

    Inputs:
        input_batch: Batch container with CPU length tensor.
        num_tokens_no_spec_gpu: Destination GPU length tensor.
        active_idx_cpu: Active request indices on CPU.
        active_idx_gpu: Active request indices on GPU.
        n_active: Number of active requests.
        device: Target CUDA device.
        _pinned_val_buf: Resident pinned int32 staging buffer.
    Outputs:
        None (updates num_tokens_no_spec_gpu in-place).
    """
    src_cpu = input_batch.num_tokens_no_spec_cpu_tensor
    vals = _pinned_val_buf[:n_active]
    vals.copy_(src_cpu.index_select(0, active_idx_cpu))

    num_tokens_no_spec_gpu.index_copy_(
        0,
        active_idx_gpu,
        vals.to(device=device, non_blocking=True),
    )
```
**EN:** This function implements `_sync_num_tokens` within the module. The docstring frames it as: Batch-sync GPU sequence lengths from CPU source of truth. Key calls include `copy_`, `index_copy_`, `index_select`, `to`.
**CN:** 该函数会实现 `_sync_num_tokens`，其作用域位于the module。 关键调用包括 `copy_`, `index_copy_`, `index_select`, `to`。

### `copy_num_valid_draft_tokens` function / `copy_num_valid_draft_tokens` 函数
```python
def copy_num_valid_draft_tokens(
    num_valid_draft_tokens_cpu: torch.Tensor,
    num_valid_draft_tokens_copy_stream: torch.cuda.Stream,
    num_valid_draft_tokens_event: torch.cuda.Event,
    num_valid_draft_tokens: torch.Tensor | None,
    batch_size: int,
) -> None:
    """
    Async D2H copy of per-request valid draft counts.
    """
    if num_valid_draft_tokens is None:
        return

    num_reqs_to_copy = min(batch_size, num_valid_draft_tokens.shape[0])
    if num_reqs_to_copy <= 0:
        return

    default_stream = torch.cuda.current_stream()
    with torch.cuda.stream(num_valid_draft_tokens_copy_stream):
        num_valid_draft_tokens_copy_stream.wait_stream(default_stream)
        num_valid_draft_tokens_cpu[:num_reqs_to_copy].copy_(
            num_valid_draft_tokens[:num_reqs_to_copy], non_blocking=True
        )
        num_valid_draft_tokens_event.record()
```
**EN:** This function implements `copy_num_valid_draft_tokens` within the module. The docstring frames it as: Async D2H copy of per-request valid draft counts. Key calls include `min`, `current_stream`, `stream`, `wait_stream`, `copy_`, `record`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `copy_num_valid_draft_tokens`，其作用域位于the module。 关键调用包括 `min`, `current_stream`, `stream`, `wait_stream`, `copy_`, `record`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `NgramGPUKernel`: central class or interface in this module. / `NgramGPUKernel`：本模块中的核心类或接口。
- `NgramProposerGPU`: central class or interface in this module. / `NgramProposerGPU`：本模块中的核心类或接口。
- `update_scheduler_for_invalid_drafts`: top-level helper or orchestration entry point. / `update_scheduler_for_invalid_drafts`：顶层辅助函数或编排入口。
- `update_ngram_gpu_tensors_incremental`: top-level helper or orchestration entry point. / `update_ngram_gpu_tensors_incremental`：顶层辅助函数或编排入口。
- `_sync_num_tokens`: top-level helper or orchestration entry point. / `_sync_num_tokens`：顶层辅助函数或编排入口。
- `copy_num_valid_draft_tokens`: top-level helper or orchestration entry point. / `copy_num_valid_draft_tokens`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.compilation.decorators`, `vllm.config`, `vllm.forward_context`, `vllm.utils.torch_utils`, `vllm.v1.core.sched.output`, `vllm.v1.utils`, `vllm.v1.worker.gpu_input_batch`
