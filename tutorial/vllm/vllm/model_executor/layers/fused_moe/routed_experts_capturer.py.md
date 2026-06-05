# routed_experts_capturer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/fused_moe/routed_experts_capturer.py`
- **Repository**: vllm-project/vllm
- **Purpose**: fused Mixture-of-Experts routing, kernels, and runtime helpers / 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 6-19 — imports and setup
```python
from __future__ import annotations

import logging

import numpy as np
import torch

from vllm.config import VllmConfig
from vllm.distributed.parallel_state import get_tp_group
from vllm.forward_context import get_forward_context
from vllm.platforms import current_platform
from vllm.v1.kv_cache_interface import FullAttentionSpec, KVCacheConfig

logger = logging.getLogger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `logging.getLogger`. It writes or updates `logger`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `logging.getLogger`。 它会写入或更新 `logger`。

### Lines 22-36 — function `_get_num_experts_per_tok`
```python
def _get_num_experts_per_tok(hf_config) -> int:
    """Resolve the per-token expert count from the HF config.

    Different model families store this under different attribute names
    (e.g. ``num_experts_per_tok`` for DeepSeek, ``top_k_experts`` for Gemma 4).
    """
    val = getattr(hf_config, "num_experts_per_tok", None)
    if val is None:
        val = getattr(hf_config, "top_k_experts", None)
    if val is None:
        raise ValueError(
            "Cannot determine num_experts_per_tok: HF config has neither "
            "'num_experts_per_tok' nor 'top_k_experts'"
        )
    return val
```
**EN:** This function defines `_get_num_experts_per_tok`. Resolve the per-token expert count from the HF config. The main inputs are `hf_config`. Key calls include `getattr`, `ValueError`. It writes or updates `val`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_get_num_experts_per_tok`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `hf_config`。 关键调用包括 `getattr`, `ValueError`。 它会写入或更新 `val`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 39-55 — function `get_num_experts`
```python
def get_num_experts(hf_config) -> int:
    """Resolve ``num_experts`` across HuggingFace config naming conventions.

    Different MoE model families expose this under different keys:
      - ``num_experts``: Mixtral, Qwen2-MoE, Qwen3-MoE
      - ``n_routed_experts``: DeepSeek-V2/V3
      - ``num_local_experts``: Mixtral (older exports)
    """
    for key in ("num_experts", "n_routed_experts", "num_local_experts"):
        val = getattr(hf_config, key, None)
        if val is not None:
            return val
    raise ValueError(
        "Could not resolve num_experts from model config. "
        "Expected one of 'num_experts', 'n_routed_experts', "
        "or 'num_local_experts'."
    )
```
**EN:** This function defines `get_num_experts`. Resolve ``num_experts`` across HuggingFace config naming conventions. The main inputs are `hf_config`. Key calls include `ValueError`, `getattr`. It writes or updates `val`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `get_num_experts`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `hf_config`。 关键调用包括 `ValueError`, `getattr`。 它会写入或更新 `val`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 58-220 — class `RoutedExpertsCapturer`
```python
class RoutedExpertsCapturer:
    """Worker-side capturer for routed experts, lives on GPU.

    Layer-level hooks call :meth:`capture` from inside the forward pass
    with the per-layer ``topk_ids`` tensor. The tensor is sliced to the
    tokens owned by this DP rank and written into a preallocated device
    buffer. At the end of the step, :class:`GPUModelRunner` reads the
    device buffer, issues a D2H copy into a pinned CPU buffer, and hands
    the result to the scheduler via :class:`RoutedExpertsLists`.

    The device / pinned-CPU transit buffers use ``torch.int32`` (not a
    narrow ``uint8``/``uint16`` sized by ``num_experts``). This keeps the
    SP all-gather path free of dtype casts, matches the router's native
    ``topk_ids`` indices dtype more closely, and costs only a few MB per
# ... omitted for brevity ...
        """
        return self.device_buffer
```
**EN:** This class defines `RoutedExpertsCapturer`. Worker-side capturer for routed experts, lives on GPU. Important methods include `__init__`, `capture`, `clear_buffer`, `get_device_buffer`. Key calls include `_get_num_experts_per_tok`, `torch.zeros`, `get_forward_context`, `self.device_buffer.zero_`, `int`, `num_tokens_dp.item`. It writes or updates `hf_config`, `num_experts_per_tok`, `device_buffer`, `dp_rank`, `tp_size`, `ctx`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `RoutedExpertsCapturer`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `capture`, `clear_buffer`, `get_device_buffer`。 关键调用包括 `_get_num_experts_per_tok`, `torch.zeros`, `get_forward_context`, `self.device_buffer.zero_`, `int`, `num_tokens_dp.item`。 它会写入或更新 `hf_config`, `num_experts_per_tok`, `device_buffer`, `dp_rank`, `tp_size`, `ctx`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 86-108 — method `RoutedExpertsCapturer.__init__`
```python
    def __init__(
        self,
        max_num_batched_tokens: int,
        vllm_config: VllmConfig,
    ) -> None:
        hf_config = vllm_config.model_config.hf_text_config
        num_experts_per_tok = _get_num_experts_per_tok(hf_config)
        self.device_buffer = torch.zeros(
            (
                max_num_batched_tokens,
                hf_config.num_hidden_layers,
                num_experts_per_tok,
            ),
            # Use int32 for the device / host transit buffers: it
            # matches the router's native topk_ids dtype, is universally
            # supported by NCCL (uint8/uint16 are version-dependent),
            # and the extra bytes are small (few MB per worker). The
            # big scheduler-side slot buffer stays narrow.
            dtype=torch.int32,
            device=current_platform.device_type,
        )
        self.dp_rank = vllm_config.parallel_config.data_parallel_rank
        self.tp_size = vllm_config.parallel_config.tensor_parallel_size
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `max_num_batched_tokens`, `vllm_config`. Key calls include `_get_num_experts_per_tok`, `torch.zeros`. It writes or updates `hf_config`, `num_experts_per_tok`, `device_buffer`, `dp_rank`, `tp_size`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `max_num_batched_tokens`, `vllm_config`。 关键调用包括 `_get_num_experts_per_tok`, `torch.zeros`。 它会写入或更新 `hf_config`, `num_experts_per_tok`, `device_buffer`, `dp_rank`, `tp_size`。

### Lines 110-205 — method `RoutedExpertsCapturer.capture`
```python
    def capture(self, layer_id: int, topk_ids: torch.Tensor) -> None:
        """Capture expert routing decisions for a specific layer.

        Under data parallelism, ``topk_ids`` may have three different batch
        layouts depending on where the DP combine happens and whether
        Sequence Parallelism (SP) is active for the MoE layer:
          - ``n == total`` (naive dispatch): all DP ranks' tokens are
            concatenated before routing; we slice out this rank's span
            using the cumulative per-rank counts.
          - ``n == token_num_per_dp`` (modular-kernel path): DP combine
            happens inside ``quant_method.apply``; ``select_experts`` only
            ever sees this rank's tokens, so we take the whole tensor.
          - ``n == ceil(token_num_per_dp / tp_size)`` (SP + modular-kernel
            path): tokens were split along dim=0 across the TP group by
            ``_sequence_parallel_context``
            (``moe_runner_base.py:_sequence_parallel_context``), so each
            TP rank only sees its shard. We all-gather along dim=0 to
            reconstruct this DP rank's full routing tensor. SP pads with
            ceil-div (see ``_compute_sp_num_tokens`` in
            ``forward_context.py``), so the gathered tensor may contain a
            few trailing padding rows which are trimmed by the downstream
            ``[:token_num_per_dp]`` slice.

        Args:
            layer_id: The layer index.
            topk_ids: Tensor of shape (batch_size, num_routed_experts).
        """

        ctx = get_forward_context()
        if ctx.dp_metadata is None:  # single dp
            start_loc = 0
            end_loc = topk_ids.shape[0]
            token_num_per_dp = topk_ids.shape[0]
        else:  # multi dp
            num_tokens_dp = ctx.dp_metadata.num_tokens_across_dp_cpu
            token_num_per_dp = int(num_tokens_dp[self.dp_rank].item())
            total = int(num_tokens_dp.sum().item())
            n = topk_ids.shape[0]

            if n == total:
                # Naive dispatch: all DP ranks' tokens concatenated
                # before routing. This rank owns tokens
                # [end_loc - token_num_per_dp, end_loc).
                cumsum = torch.cumsum(num_tokens_dp, dim=0)
# ... omitted for brevity ...
            start_loc:end_loc, :
        ]
```
**EN:** This method defines `capture`. Capture expert routing decisions for a specific layer. The main inputs are `layer_id`, `topk_ids`. Key calls include `get_forward_context`, `int`, `num_tokens_dp.item`, `num_tokens_dp.sum.item`, `torch.cumsum`, `cumsum.item`. It writes or updates `ctx`, `start_loc`, `end_loc`, `token_num_per_dp`, `num_tokens_dp`, `total`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `capture`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `layer_id`, `topk_ids`。 关键调用包括 `get_forward_context`, `int`, `num_tokens_dp.item`, `num_tokens_dp.sum.item`, `torch.cumsum`, `cumsum.item`。 它会写入或更新 `ctx`, `start_loc`, `end_loc`, `token_num_per_dp`, `num_tokens_dp`, `total`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 207-212 — method `RoutedExpertsCapturer.clear_buffer`
```python
    def clear_buffer(self) -> None:
        """Zero the device buffer. Called at the start of every step so
        slots belonging to finished / preempted tokens don't leak into
        the next step.
        """
        self.device_buffer.zero_()
```
**EN:** This method defines `clear_buffer`. Zero the device buffer. Key calls include `self.device_buffer.zero_`.
**CN:** 该方法定义 `clear_buffer`。 该函数/方法的文档字符串直接说明了它的职责。 关键调用包括 `self.device_buffer.zero_`。

### Lines 214-220 — method `RoutedExpertsCapturer.get_device_buffer`
```python
    def get_device_buffer(self) -> torch.Tensor:
        """Return the underlying device buffer so the model runner can
        issue the D2H copy. The tensor is shared; callers must either
        clone or fully drain it before the next forward pass runs
        :meth:`clear_buffer`.
        """
        return self.device_buffer
```
**EN:** This method defines `get_device_buffer`. Return the underlying device buffer so the model runner can issue the D2H copy.
**CN:** 该方法定义 `get_device_buffer`。 该函数/方法的文档字符串直接说明了它的职责。

### Lines 223-349 — class `RoutedExpertsManager`
```python
class RoutedExpertsManager:
    """Scheduler-side slot-indexed buffer for routed experts.

    Lives on CPU in the scheduler process. Each slot corresponds to
    ``block_id * block_size + offset_in_block`` where ``block_id`` is
    drawn from the physical KV-cache block pool, so routing data is
    tied to physical blocks and naturally survives preemption for
    prefix-cached blocks (prefix hits re-expose the same slots).

    Data flow per step:
      1. Worker D2Hs its device capture buffer into
         :class:`RoutedExpertsLists` and returns it via
         :class:`ModelRunnerOutput`.
      2. Scheduler calls :meth:`store_batch` with that step's
# ... omitted for brevity ...
        slot_mapping = slot_mapping[token_start:]
        return self.routed_experts_by_slot[slot_mapping]
```
**EN:** This class defines `RoutedExpertsManager`. Scheduler-side slot-indexed buffer for routed experts. Important methods include `__init__`, `store_batch`, `get`. Key calls include `next`, `get_num_experts`, `_get_num_experts_per_tok`, `np.zeros`, `logger.info`, `np.array`. It writes or updates `attn_gid`, `attn_group`, `block_size`, `hf_config`, `num_experts`, `num_experts_per_tok`.
**CN:** 该类定义了 `RoutedExpertsManager`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `store_batch`, `get`。 关键调用包括 `next`, `get_num_experts`, `_get_num_experts_per_tok`, `np.zeros`, `logger.info`, `np.array`。 它会写入或更新 `attn_gid`, `attn_group`, `block_size`, `hf_config`, `num_experts`, `num_experts_per_tok`。

### Lines 248-296 — method `RoutedExpertsManager.__init__`
```python
    def __init__(
        self,
        vllm_config: VllmConfig,
        kv_cache_config: KVCacheConfig,
    ) -> None:
        # Pick the attention group for block/slot mapping. We require
        # a FullAttentionSpec group rather than any AttentionSpec to
        # stay consistent with the worker-side lookup in
        # ``GPUModelRunner._get_attention_kv_cache_gid``; hybrid models
        # (Mamba / linear attention) also have other AttentionSpec
        # groups whose slot layout differs.
        self.attn_gid = next(
            gid
            for gid, g in enumerate(kv_cache_config.kv_cache_groups)
            if isinstance(g.kv_cache_spec, FullAttentionSpec)
        )
        attn_group = kv_cache_config.kv_cache_groups[self.attn_gid]
        self.block_size = attn_group.kv_cache_spec.block_size

        # All kv_cache_groups share the same physical block pool, so
        # block IDs span [0, num_blocks) regardless of how many groups
        # exist. Sizing to the full pool avoids index-out-of-range
        # when different groups happen to land on the same block.
        hf_config = vllm_config.model_config.hf_text_config
        num_experts = get_num_experts(hf_config)
        num_experts_per_tok = _get_num_experts_per_tok(hf_config)
        max_num_slots = kv_cache_config.num_blocks * self.block_size
        # Expert IDs are 0..num_experts-1; uint8 fits 256 distinct
        # values so the boundary is ``<= 256`` (NOT ``< 256``). Keeping
        # this narrow matters because the slot buffer is sized for the
        # whole block pool and can reach multiple GB.
        expert_id_dtype = np.uint8 if num_experts <= 256 else np.uint16
        self.routed_experts_by_slot = np.zeros(
            (
                max_num_slots,
                hf_config.num_hidden_layers,
                num_experts_per_tok,
            ),
            dtype=expert_id_dtype,
        )
        logger.info(
            "RoutedExpertsManager CPU buffer: %.2f GB "
            "(slots=%d, layers=%d, top_k=%d, dtype=%s)",
            self.routed_experts_by_slot.nbytes / 1e9,
# ... omitted for brevity ...
            self.routed_experts_by_slot.dtype.name,
        )
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `vllm_config`, `kv_cache_config`. Key calls include `next`, `get_num_experts`, `_get_num_experts_per_tok`, `np.zeros`, `logger.info`, `enumerate`. It writes or updates `attn_gid`, `attn_group`, `block_size`, `hf_config`, `num_experts`, `num_experts_per_tok`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `vllm_config`, `kv_cache_config`。 关键调用包括 `next`, `get_num_experts`, `_get_num_experts_per_tok`, `np.zeros`, `logger.info`, `enumerate`。 它会写入或更新 `attn_gid`, `attn_group`, `block_size`, `hf_config`, `num_experts`, `num_experts_per_tok`。

### Lines 298-305 — method `RoutedExpertsManager.store_batch`
```python
    def store_batch(self, data: np.ndarray, slot_mapping: np.ndarray) -> None:
        """Persist one step's routed experts into the slot buffer.

        Equivalent to ``slot_buffer[slot_mapping] = data``; numpy fancy
        indexing handles repeated / out-of-order indices. Called once
        per scheduler step in ``update_from_output``.
        """
        self.routed_experts_by_slot[slot_mapping] = data
```
**EN:** This method defines `store_batch`. Persist one step's routed experts into the slot buffer. The main inputs are `data`, `slot_mapping`.
**CN:** 该方法定义 `store_batch`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `data`, `slot_mapping`。

### Lines 307-349 — method `RoutedExpertsManager.get`
```python
    def get(
        self,
        block_ids: list[int],
        num_tokens: int,
        token_start: int = 0,
    ) -> np.ndarray:
        """Read routed experts data for a completed / preempted request.

        Reconstructs a per-token slot_mapping from the request's block
        IDs and returns the routing slice. Because numpy fancy indexing
        returns a **copy** (not a view), the returned ndarray is safe
        to hold across subsequent :meth:`store_batch` calls — do not
        replace the fancy index with a slice without re-verifying.

        Args:
            block_ids: Block IDs from the attention KV-cache group.
            num_tokens: Number of tokens that have gone through a forward
                pass and therefore have routing data written to their
                slots (typically ``request.num_tokens - 1``; the last
                sampled token has not been forwarded yet). Slots beyond
                ``request.num_computed_tokens`` are zero-initialized.
            token_start: Skip the first ``token_start`` tokens from the
                result. The slot_mapping is sliced before the fancy-index
                read, so only the requested slots are fetched — no large
                intermediate array is allocated. Clamped to
                ``[0, num_tokens]`` automatically.

        Returns:
            Array of shape (num_tokens - token_start, num_layers,
            num_experts_per_tok).
        """
        bs = self.block_size
        block_ids_array = np.array(block_ids, dtype=np.int32)
        block_offsets = np.arange(bs)
        # slot = block_id * block_size + offset_in_block; flatten the
        # (num_blocks, block_size) grid and trim to num_tokens, then
        # skip the first token_start entries so only the requested
        # range is fetched in a single fancy-index read.
        slot_mapping = (
            block_ids_array.reshape(-1, 1) * bs + block_offsets.reshape(1, -1)
        ).flatten()[:num_tokens]
        slot_mapping = slot_mapping[token_start:]
        return self.routed_experts_by_slot[slot_mapping]
```
**EN:** This method defines `get`. Read routed experts data for a completed / preempted request. The main inputs are `block_ids`, `num_tokens`, `token_start`. Key calls include `np.array`, `np.arange`, `flatten`, `block_offsets.reshape`, `block_ids_array.reshape`. It writes or updates `bs`, `block_ids_array`, `block_offsets`, `slot_mapping`.
**CN:** 该方法定义 `get`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `block_ids`, `num_tokens`, `token_start`。 关键调用包括 `np.array`, `np.arange`, `flatten`, `block_offsets.reshape`, `block_ids_array.reshape`。 它会写入或更新 `bs`, `block_ids_array`, `block_offsets`, `slot_mapping`。

## Key Concepts / 关键概念
- [EN] Fused mixture-of-experts routing, kernels, and runtime helpers / [CN] 融合式 Mixture-of-Experts 路由、内核与运行时辅助逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Distributed or tensor-parallel coordination / [CN] 分布式或张量并行协同
- [EN] Core symbols: `_get_num_experts_per_tok`, `get_num_experts`, `RoutedExpertsCapturer`, `RoutedExpertsManager` / [CN] 核心符号：`_get_num_experts_per_tok`, `get_num_experts`, `RoutedExpertsCapturer`, `RoutedExpertsManager`

## Dependencies / 依赖关系
- **External**: `__future__`, `logging`, `numpy`, `torch` / **外部依赖**: `__future__`, `logging`, `numpy`, `torch`
- **Internal**: `vllm.config`, `vllm.distributed.parallel_state`, `vllm.forward_context`, `vllm.platforms`, `vllm.v1.kv_cache_interface` / **内部依赖**: `vllm.config`, `vllm.distributed.parallel_state`, `vllm.forward_context`, `vllm.platforms`, `vllm.v1.kv_cache_interface`
- **Runtime traits**: platform-aware dispatch, distributed collectives / **运行时特征**: platform-aware dispatch, distributed collectives
