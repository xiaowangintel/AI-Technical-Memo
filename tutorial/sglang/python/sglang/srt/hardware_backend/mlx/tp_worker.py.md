# tp_worker.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/mlx/tp_worker.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements MLX backend support for worker coordination inside the SGLang runtime. / 为 SGLang 运行时提供面向 MLX 后端的工作线程协同支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-31: Module setup and shared state / 模块设置与共享状态
```python
"""MLX-specific TpModelWorker subclass for Apple Silicon.

Routes forward passes through the MLX model runner, bypassing PyTorch
MPS.  A lightweight stub provides scheduler bookkeeping; the actual
KV data lives in MlxKVPool.

The worker also exposes an async (lazy-eval) surface used by the MLX
overlap scheduler: ``async_forward_batch_generation_mlx`` launches a
batch without blocking on the GPU, ``async_chained_decode_mlx`` builds
the next decode step on top of a still-lazy previous decode, and
``finalize_mlx_result`` blocks on the lazy outputs and produces a
normal ``GenerationBatchResult``.
"""

import logging
from typing import Optional, Union

import mlx.core as mx
import torch

from sglang.srt.hardware_backend.mlx.model_runner import (
    MlxPendingDecode,
    MlxPendingExtend,
    MlxPendingPrefill,
)
from sglang.srt.managers.schedule_batch import ScheduleBatch
from sglang.srt.managers.tp_worker import TpModelWorker
from sglang.srt.managers.utils import GenerationBatchResult
from sglang.srt.model_executor.forward_batch_info import ForwardBatch, PPProxyTensors

logger = logging.getLogger(__name__)
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `logging`, `typing`, `mlx.core`, `torch`, `sglang.srt.hardware_backend.mlx.model_runner`, `sglang.srt.managers.schedule_batch`. It also defines symbols such as `logger` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `logging`, `typing`, `mlx.core`, `torch`, `sglang.srt.hardware_backend.mlx.model_runner`, `sglang.srt.managers.schedule_batch`。 同时定义了 `logger` 等符号，供后续逻辑使用。

### Lines 34-41: Class `MlxTpModelWorker` declaration / 类 `MlxTpModelWorker` 声明
```python
class MlxTpModelWorker(TpModelWorker):
    """A tensor parallel model worker that routes inference through MLX.

    Inherits from TpModelWorker for scheduler integration, but replaces
    the standard ModelRunner with MlxModelRunnerStub (no PyTorch weights,
    zero-memory KV cache) and delegates all forward passes to a native
    MlxModelRunner.
    """
```
**EN:** This class establishes `MlxTpModelWorker` as the main container/coordinator for the surrounding logic. It inherits from `TpModelWorker`. Its core interface includes methods such as `_init_model_runner`, `get_pad_input_ids_func`, `_ensure_mlx_pool_initialized`, `forward_batch_generation`, `_cleanup_stale_rids`, `_forward_batch_generation_mlx`.
**CN:** 该类将 `MlxTpModelWorker` 定义为周边逻辑的主要封装体或协调者。 它继承自 `TpModelWorker`。 其核心接口包括 `_init_model_runner`, `get_pad_input_ids_func`, `_ensure_mlx_pool_initialized`, `forward_batch_generation`, `_cleanup_stale_rids`, `_forward_batch_generation_mlx` 等方法。

### Lines 43-83: Method `MlxTpModelWorker._init_model_runner` / 方法 `MlxTpModelWorker._init_model_runner`
```python
    def _init_model_runner(self):
        """Create MLX runner first (auto-sizes pool), then stub with matching size."""
        from sglang.srt.hardware_backend.mlx.model_runner import MlxModelRunner
        from sglang.srt.hardware_backend.mlx.model_runner_stub import (
            MlxModelRunnerStub,
        )

        logger.info("Initializing MlxModelRunner for end-to-end MLX inference")
        init_kwargs = dict(
            model_path=self.server_args.model_path,
            trust_remote_code=self.server_args.trust_remote_code,
            disable_radix_cache=self.server_args.disable_radix_cache,
            mem_fraction_static=self.server_args.mem_fraction_static,
            quantization=self.server_args.quantization,
        )
        if self.server_args.max_total_tokens is not None:
            init_kwargs["pool_size"] = self.server_args.max_total_tokens
        self._mlx_runner = MlxModelRunner(**init_kwargs)

        self._model_runner = MlxModelRunnerStub(
            model_config=self.model_config,
            mem_fraction_static=self.server_args.mem_fraction_static,
            gpu_id=self.gpu_id,
            tp_rank=self.tp_rank,
            tp_size=self.tp_size,
            moe_ep_rank=self.moe_ep_rank,
            moe_ep_size=self.ep_size,
            pp_rank=self.pp_rank,
# ... omitted for brevity ...
            memory_pool_config=self.memory_pool_config,
            mlx_pool_size=self._mlx_runner.pool_size,
        )

        self._mlx_active_rids: set[str] = set()
        self._mlx_pool_initialized = False
```
**EN:** This method implements `_init_model_runner` on `MlxTpModelWorker`. It primarily calls `logger.info`, `dict`, `MlxModelRunner`, `MlxModelRunnerStub`, `set` to complete its work. State updates are written into `init_kwargs`, `self._mlx_runner`, `self._model_runner`, `self._mlx_active_rids`, `self._mlx_pool_initialized`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MlxTpModelWorker`）实现了 `_init_model_runner`。 它主要通过调用 `logger.info`, `dict`, `MlxModelRunner`, `MlxModelRunnerStub`, `set` 来完成任务。 状态更新主要写入 `init_kwargs`, `self._mlx_runner`, `self._model_runner`, `self._mlx_active_rids`, `self._mlx_pool_initialized`。 实现中使用了条件分支。

### Lines 85-87: Method `MlxTpModelWorker.get_pad_input_ids_func` / 方法 `MlxTpModelWorker.get_pad_input_ids_func`
```python
    def get_pad_input_ids_func(self):
        """Override since the stub ModelRunner has no real model."""
        return None
```
**EN:** This method implements `get_pad_input_ids_func` on `MlxTpModelWorker`.
**CN:** 该方法（属于 `MlxTpModelWorker`）实现了 `get_pad_input_ids_func`。

### Lines 89-93: Method `MlxTpModelWorker._ensure_mlx_pool_initialized` / 方法 `MlxTpModelWorker._ensure_mlx_pool_initialized`
```python
    def _ensure_mlx_pool_initialized(self):
        """Lazily initialize the MlxKVPool after the stub's pools are ready."""
        if not self._mlx_pool_initialized:
            self._mlx_runner.init_kv_pool(self._model_runner.req_to_token_pool)
            self._mlx_pool_initialized = True
```
**EN:** This method implements `_ensure_mlx_pool_initialized` on `MlxTpModelWorker`. It primarily calls `self._mlx_runner.init_kv_pool` to complete its work. State updates are written into `self._mlx_pool_initialized`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MlxTpModelWorker`）实现了 `_ensure_mlx_pool_initialized`。 它主要通过调用 `self._mlx_runner.init_kv_pool` 来完成任务。 状态更新主要写入 `self._mlx_pool_initialized`。 实现中使用了条件分支。

### Lines 95-115: Method `MlxTpModelWorker.forward_batch_generation` / 方法 `MlxTpModelWorker.forward_batch_generation`
```python
    def forward_batch_generation(
        self,
        batch: Optional[ScheduleBatch],
        forward_batch: Optional[ForwardBatch] = None,
        pp_proxy_tensors: Optional[PPProxyTensors] = None,
        is_verify: bool = False,
        skip_attn_backend_init=False,
    ) -> GenerationBatchResult:
        """Override to route through MLX model runner."""
        if batch is not None:
            self._ensure_mlx_pool_initialized()
            return self._forward_batch_generation_mlx(batch)

        # Fallback to standard path for None batches
        return super().forward_batch_generation(
            batch,
            forward_batch,
            pp_proxy_tensors,
            is_verify,
            skip_attn_backend_init,
        )
```
**EN:** This method implements `forward_batch_generation` on `MlxTpModelWorker`. It primarily calls `super.forward_batch_generation`, `self._ensure_mlx_pool_initialized`, `self._forward_batch_generation_mlx`, `super` to complete its work. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MlxTpModelWorker`）实现了 `forward_batch_generation`。 它主要通过调用 `super.forward_batch_generation`, `self._ensure_mlx_pool_initialized`, `self._forward_batch_generation_mlx`, `super` 来完成任务。 实现中使用了条件分支。

### Lines 117-125: Method `MlxTpModelWorker._cleanup_stale_rids` / 方法 `MlxTpModelWorker._cleanup_stale_rids`
```python
    def _cleanup_stale_rids(self, forward_mode, current_rids: set[str]) -> None:
        """Remove MLX state for decode-mode requests that dropped out of the batch."""
        if forward_mode.is_decode():
            stale_rids = self._mlx_active_rids - current_rids
            for rid in stale_rids:
                self._mlx_runner.remove_request(rid)
            self._mlx_active_rids = current_rids
        else:
            self._mlx_active_rids |= current_rids
```
**EN:** This method implements `_cleanup_stale_rids` on `MlxTpModelWorker`. It primarily calls `forward_mode.is_decode`, `self._mlx_runner.remove_request` to complete its work. State updates are written into `stale_rids`, `self._mlx_active_rids`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `MlxTpModelWorker`）实现了 `_cleanup_stale_rids`。 它主要通过调用 `forward_mode.is_decode`, `self._mlx_runner.remove_request` 来完成任务。 状态更新主要写入 `stale_rids`, `self._mlx_active_rids`。 实现中使用了条件分支、迭代逻辑。

### Lines 127-226: Method `MlxTpModelWorker._forward_batch_generation_mlx` / 方法 `MlxTpModelWorker._forward_batch_generation_mlx`
```python
    def _forward_batch_generation_mlx(
        self, batch: ScheduleBatch
    ) -> GenerationBatchResult:
        """Run forward pass through the MLX model runner (greedy only)."""
        from sglang.srt.layers.logits_processor import LogitsProcessorOutput

        forward_mode = batch.forward_mode
        reqs = batch.reqs

        if forward_mode.is_idle():
            return GenerationBatchResult(
                logits_output=LogitsProcessorOutput(next_token_logits=None),
                can_run_cuda_graph=False,
            )

        self._cleanup_stale_rids(forward_mode, {req.rid for req in reqs})

        next_token_ids_list: list[int] = []

        if forward_mode.is_extend():
            # Ensure pool is up-to-date before PoolBackedCache reads it
            # for prefix-cached prefills.  Only runs on extend batches.
            self._mlx_runner.flush_all_decode_kv()
            input_ids_cpu = batch.input_ids.cpu().tolist()
            out_cache_loc_cpu = batch.out_cache_loc.cpu().tolist()
            extend_seq_lens = batch.extend_lens

            offset = 0  # into input_ids_cpu
# ... omitted for brevity ...

        return GenerationBatchResult(
            logits_output=LogitsProcessorOutput(next_token_logits=None),
            next_token_ids=next_token_ids,
            can_run_cuda_graph=False,
        )
```
**EN:** This method implements `_forward_batch_generation_mlx` on `MlxTpModelWorker`. It primarily calls `forward_mode.is_idle`, `self._cleanup_stale_rids`, `forward_mode.is_extend`, `torch.tensor`, `GenerationBatchResult`, `self._mlx_runner.flush_all_decode_kv` to complete its work. State updates are written into `forward_mode`, `reqs`, `next_token_ids_list`, `next_token_ids`, `input_ids_cpu`, `out_cache_loc_cpu`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `MlxTpModelWorker`）实现了 `_forward_batch_generation_mlx`。 它主要通过调用 `forward_mode.is_idle`, `self._cleanup_stale_rids`, `forward_mode.is_extend`, `torch.tensor`, `GenerationBatchResult`, `self._mlx_runner.flush_all_decode_kv` 来完成任务。 状态更新主要写入 `forward_mode`, `reqs`, `next_token_ids_list`, `next_token_ids`, `input_ids_cpu`, `out_cache_loc_cpu`。 实现中使用了条件分支、迭代逻辑。

### Lines 228-280: Method `MlxTpModelWorker.async_forward_batch_generation_mlx` / 方法 `MlxTpModelWorker.async_forward_batch_generation_mlx`
```python
    def async_forward_batch_generation_mlx(self, batch: ScheduleBatch) -> tuple[
        Union[mx.array, None],
        list[MlxPendingPrefill],
        list[MlxPendingExtend],
        Optional[MlxPendingDecode],
        str,
    ]:
        """Start an async (lazy) forward pass through the MLX model runner.

        Returns ``(lazy_result, prefills, extends, decode, mode)``:

        * ``lazy_result`` — an ``mx.array`` that, when evaluated, forces
          materialisation of the whole batch's outputs.  ``None`` for
          idle batches.
        * ``prefills`` — list of :class:`MlxPendingPrefill` for new
          requests in an extend batch.
        * ``extends`` — list of :class:`MlxPendingExtend` for chunked
          prefill continuations in an extend batch.
        * ``decode`` — :class:`MlxPendingDecode` for the decode
          sub-batch (covers full decode mode AND mixed decodes inside
          an extend batch).
        * ``mode`` — one of ``"idle"``, ``"decode"``, ``"extend"``.

        The caller must make sure the returned pendings are fed into a
        subsequent ``mx.async_eval`` or ``.item()`` / ``.tolist()`` call
        — :meth:`finalize_mlx_result` does that.
        """
        self._ensure_mlx_pool_initialized()
# ... omitted for brevity ...
            self._mlx_runner.flush_all_decode_kv()
            return self._async_extend_batch(batch)

        raise ValueError(
            f"MLX async runner does not support forward mode: {forward_mode}"
        )
```
**EN:** This method implements `async_forward_batch_generation_mlx` on `MlxTpModelWorker`. It primarily calls `self._ensure_mlx_pool_initialized`, `forward_mode.is_idle`, `self._cleanup_stale_rids`, `forward_mode.is_decode`, `forward_mode.is_extend`, `ValueError` to complete its work. State updates are written into `forward_mode`, `reqs`, `req_ids`, `pending_decode`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MlxTpModelWorker`）实现了 `async_forward_batch_generation_mlx`。 它主要通过调用 `self._ensure_mlx_pool_initialized`, `forward_mode.is_idle`, `self._cleanup_stale_rids`, `forward_mode.is_decode`, `forward_mode.is_extend`, `ValueError` 来完成任务。 状态更新主要写入 `forward_mode`, `reqs`, `req_ids`, `pending_decode`。 实现中使用了条件分支。

### Lines 282-375: Method `MlxTpModelWorker._async_extend_batch` / 方法 `MlxTpModelWorker._async_extend_batch`
```python
    def _async_extend_batch(self, batch: ScheduleBatch) -> tuple[
        Union[mx.array, None],
        list[MlxPendingPrefill],
        list[MlxPendingExtend],
        Optional[MlxPendingDecode],
        str,
    ]:
        """Launch each request in an EXTEND batch lazily and kick GPU work."""
        reqs = batch.reqs
        input_ids_cpu = batch.input_ids.cpu().tolist()
        out_cache_loc_cpu = batch.out_cache_loc.cpu().tolist()
        extend_seq_lens = batch.extend_lens

        offset = 0
        slot_offset = 0
        pending_prefills: list[MlxPendingPrefill] = []
        pending_extends: list[MlxPendingExtend] = []
        mixed_decode_rids: list[str] = []

        for i, req in enumerate(reqs):
            seq_len = extend_seq_lens[i]
            req_token_ids = input_ids_cpu[offset : offset + seq_len]
            req_new_slots = out_cache_loc_cpu[slot_offset : slot_offset + seq_len]
            offset += seq_len
            slot_offset += seq_len

            if self._mlx_runner.has_request(req.rid):
                if seq_len > 1:
# ... omitted for brevity ...
            lazy_stacked,
            pending_prefills,
            pending_extends,
            pending_mixed_decode,
            "extend",
        )
```
**EN:** This method implements `_async_extend_batch` on `MlxTpModelWorker`. It primarily calls `batch.input_ids.cpu.tolist`, `batch.out_cache_loc.cpu.tolist`, `enumerate`, `prefill_ext_tokens.extend`, `self._mlx_runner.has_request`, `self._mlx_runner.decode_batch_start` to complete its work. State updates are written into `reqs`, `input_ids_cpu`, `out_cache_loc_cpu`, `extend_seq_lens`, `offset`, `slot_offset`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `MlxTpModelWorker`）实现了 `_async_extend_batch`。 它主要通过调用 `batch.input_ids.cpu.tolist`, `batch.out_cache_loc.cpu.tolist`, `enumerate`, `prefill_ext_tokens.extend`, `self._mlx_runner.has_request`, `self._mlx_runner.decode_batch_start` 来完成任务。 状态更新主要写入 `reqs`, `input_ids_cpu`, `out_cache_loc_cpu`, `extend_seq_lens`, `offset`, `slot_offset`。 实现中使用了条件分支、迭代逻辑。

### Lines 377-380: Method `MlxTpModelWorker._cache_state` / 方法 `MlxTpModelWorker._cache_state`
```python
    @staticmethod
    def _cache_state(cache_list) -> list[mx.array]:
        """Flatten a per-layer cache list to its ``state`` arrays."""
        return [s for c in cache_list for s in c.state]
```
**EN:** This method implements `_cache_state` on `MlxTpModelWorker`.
**CN:** 该方法（属于 `MlxTpModelWorker`）实现了 `_cache_state`。

### Lines 382-414: Method `MlxTpModelWorker.async_chained_decode_mlx` / 方法 `MlxTpModelWorker.async_chained_decode_mlx`
```python
    def async_chained_decode_mlx(
        self,
        prev_pending: MlxPendingDecode,
    ) -> tuple[mx.array, list, list, MlxPendingDecode, str]:
        """Launch a decode step that chains off a still-lazy previous decode.

        This is the "no idle gap" pipelining primitive: build the next
        decode's compute graph using ``prev_pending.lazy_tokens`` (still
        unevaluated) as its input ids, hand the combined graph to
        ``mx.async_eval``, and return.  The GPU runs the new step
        immediately after ``prev_pending`` with no scheduling gap, while
        the caller is free to block on ``prev_pending`` and run CPU-side
        bookkeeping.

        Preconditions (caller must ensure):

        * ``prev_pending`` was produced by a previous decode start
          (either :meth:`async_forward_batch_generation_mlx` in decode
          mode or a previous :meth:`async_chained_decode_mlx`).
        * The batch composition for this step is identical to
          ``prev_pending`` — same requests, same order.  Composition
          changes (finished reqs, new prefills) must break the chain.
        * ``prev_pending`` should be finalised BEFORE the returned
          pending, so per-request token lists are appended in order.

        Returns a 5-tuple matching
        :meth:`async_forward_batch_generation_mlx` for the decode case:
        ``(lazy_tokens, [], [], pending_decode, "decode")``.  The empty
        prefill/extend lists are always absent for chained decodes.
        """
        pending = self._mlx_runner.decode_batch_start_chained(prev_pending)
        mx.async_eval(pending.lazy_tokens)
        return pending.lazy_tokens, [], [], pending, "decode"
```
**EN:** This method implements `async_chained_decode_mlx` on `MlxTpModelWorker`. It primarily calls `self._mlx_runner.decode_batch_start_chained`, `mx.async_eval` to complete its work. State updates are written into `pending`.
**CN:** 该方法（属于 `MlxTpModelWorker`）实现了 `async_chained_decode_mlx`。 它主要通过调用 `self._mlx_runner.decode_batch_start_chained`, `mx.async_eval` 来完成任务。 状态更新主要写入 `pending`。

### Lines 416-479: Method `MlxTpModelWorker.finalize_mlx_result` / 方法 `MlxTpModelWorker.finalize_mlx_result`
```python
    def finalize_mlx_result(
        self,
        prefills: list[MlxPendingPrefill],
        extends: list[MlxPendingExtend],
        decode: Optional[MlxPendingDecode],
        mode: str,
        reqs: list,
    ) -> GenerationBatchResult:
        """Materialise a lazy MLX result into a :class:`GenerationBatchResult`.

        The blocking wait happens inside ``decode_batch_finalize`` /
        ``prefill_finalize`` / ``extend_finalize`` via ``.tolist()`` /
        ``.item()`` on the specific lazy outputs.
        """
        from sglang.srt.layers.logits_processor import LogitsProcessorOutput

        if mode == "idle":
            return GenerationBatchResult(
                logits_output=LogitsProcessorOutput(next_token_logits=None),
                can_run_cuda_graph=False,
            )

        if mode == "decode":
            assert decode is not None
            next_tokens_list = self._mlx_runner.decode_batch_finalize(decode)

        elif mode == "extend":
            prefill_map: dict[str, int] = {}
# ... omitted for brevity ...
        next_token_ids = torch.tensor(next_tokens_list, dtype=torch.long, device="cpu")
        return GenerationBatchResult(
            logits_output=LogitsProcessorOutput(next_token_logits=None),
            next_token_ids=next_token_ids,
            can_run_cuda_graph=False,
        )
```
**EN:** This method implements `finalize_mlx_result` on `MlxTpModelWorker`. It primarily calls `torch.tensor`, `GenerationBatchResult`, `self._mlx_runner.decode_batch_finalize`, `ValueError`, `LogitsProcessorOutput`, `self._mlx_runner.prefill_finalize` to complete its work. State updates are written into `next_token_ids`, `next_tokens_list`, `prefill_map`, `extend_map`, `decode_map`, `mixed_tokens`. The implementation relies on conditional branches, iteration.
**CN:** 该方法（属于 `MlxTpModelWorker`）实现了 `finalize_mlx_result`。 它主要通过调用 `torch.tensor`, `GenerationBatchResult`, `self._mlx_runner.decode_batch_finalize`, `ValueError`, `LogitsProcessorOutput`, `self._mlx_runner.prefill_finalize` 来完成任务。 状态更新主要写入 `next_token_ids`, `next_tokens_list`, `prefill_map`, `extend_map`, `decode_map`, `mixed_tokens`。 实现中使用了条件分支、迭代逻辑。

## Key Concepts / 关键概念
- **Classes / 类**: `MlxTpModelWorker`
- **Functions / 函数**: `_init_model_runner`, `get_pad_input_ids_func`, `_ensure_mlx_pool_initialized`, `forward_batch_generation`, `_cleanup_stale_rids`, `_forward_batch_generation_mlx`, `async_forward_batch_generation_mlx`, `_async_extend_batch`
- **Themes / 主题**: `worker`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.hardware_backend.mlx.model_runner`, `sglang.srt.managers.schedule_batch`, `sglang.srt.managers.tp_worker`, `sglang.srt.managers.utils`, `sglang.srt.model_executor.forward_batch_info`, `sglang.srt.hardware_backend.mlx.model_runner_stub`, `sglang.srt.layers.logits_processor`
- **External / 外部依赖**: `mlx.core`, `torch`
- **Standard library / 标准库**: `logging`, `typing`
