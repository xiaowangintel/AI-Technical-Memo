# flashattention_backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/hardware_backend/musa/attention/flashattention_backend.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Implements MUSA backend support for attention execution and masking inside the SGLang runtime. / 为 SGLang 运行时提供面向 MUSA 后端的注意力执行与掩码处理支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-41: Module setup and shared state / 模块设置与共享状态
```python
from __future__ import annotations

import threading
from typing import TYPE_CHECKING, Optional, Tuple, Union

import torch
from flash_attn_interface import flash_attn_varlen_func
from flash_attn_interface import flash_attn_with_kvcache as mate_flash_attn_with_kvcache
from flash_attn_interface import get_scheduler_metadata

from sglang.srt.distributed import get_pp_group, get_pp_indices
from sglang.srt.environ import envs
from sglang.srt.hardware_backend.musa.layers.utils.cp_utils import (
    musa_cp_attn_forward_extend as cp_attn_forward_extend,
)
from sglang.srt.layers.attention.flashattention_backend import (
    FlashAttentionBackend,
    FlashAttentionMultiStepBackend,
    merge_state_v2_wrapper,
)
from sglang.srt.layers.radix_attention import AttentionType
from sglang.srt.layers.utils.cp_utils import (
    cp_allgather_and_save_kv_cache,
)
from sglang.srt.server_args import get_global_server_args

if TYPE_CHECKING:
    from sglang.srt.layers.radix_attention import RadixAttention
# ... omitted for brevity ...
# Cache for non-MLA scheduler metadata by prefix
_MATE_NO_MLA_SCHEDULER_METADATA_DICT: dict = {}
_MATE_NO_MLA_SCHEDULER_METADATA_LOCK = threading.Lock()

# Global reference to the current backend instance (set during __init__)
_CURRENT_BACKEND: Optional["MusaFlashAttentionBackend"] = None
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `__future__`, `threading`, `typing`, `torch`, `flash_attn_interface`, `flash_attn_interface`. It also defines symbols such as `_MATE_MLA_WORKSPACE_SIZE_BYTES`, `_MATE_MLA_WORKSPACE_BUFFER`, `_MATE_NO_MLA_SCHEDULER_METADATA_DICT`, `_MATE_NO_MLA_SCHEDULER_METADATA_LOCK`, `_CURRENT_BACKEND` for later sections.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `__future__`, `threading`, `typing`, `torch`, `flash_attn_interface`, `flash_attn_interface`。 同时定义了 `_MATE_MLA_WORKSPACE_SIZE_BYTES`, `_MATE_MLA_WORKSPACE_BUFFER`, `_MATE_NO_MLA_SCHEDULER_METADATA_DICT`, `_MATE_NO_MLA_SCHEDULER_METADATA_LOCK`, `_CURRENT_BACKEND` 等符号，供后续逻辑使用。

### Lines 44-115: Function `_compute_scheduler_metadata` / 函数 `_compute_scheduler_metadata`
```python
def _compute_scheduler_metadata(
    backend: "MusaFlashAttentionBackend",
    cu_seqlens_q: torch.Tensor,
    cu_seqlens_k_new: Optional[torch.Tensor],
    cache_seqlens: torch.Tensor,
    max_seqlen_q: int,
    page_size: int,
    causal: bool,
    window_size: Tuple[int, int],
    num_splits: int,
) -> Tuple[torch.Tensor, bool] | torch.Tensor:
    """Compute scheduler metadata based on backend's current state."""
    global _MATE_MLA_WORKSPACE_BUFFER, _MATE_NO_MLA_SCHEDULER_METADATA_DICT

    layer = backend._current_layer
    current_layer_id = layer.layer_id
    batch_size = cu_seqlens_q.shape[-1] - 1

    # Determine if scheduler metadata should be updated
    should_update = True
    pp_group = get_pp_group()
    pp_rank = pp_group.rank_in_group
    start_layer_id, _ = get_pp_indices(
        backend.num_hidden_layers, pp_group.rank_in_group, pp_group.world_size
    )
    if backend._current_can_run_tbo and pp_rank == 0:
        start_layer_id += (
            backend.first_k_dense_replace
# ... omitted for brevity ...
                        causal=causal,
                        window_size=window_size,
                        num_splits=num_splits,
                    )
                )
            return _MATE_NO_MLA_SCHEDULER_METADATA_DICT[backend._current_prefix]
```
**EN:** This function implements `_compute_scheduler_metadata`. It primarily calls `get_pp_group`, `get_pp_indices`, `envs.SGLANG_MUSA_FA3_FORCE_UPDATE_METADATA.get`, `torch.empty`, `get_scheduler_metadata` to complete its work. State updates are written into `layer`, `current_layer_id`, `batch_size`, `should_update`, `pp_group`, `pp_rank`. The implementation relies on conditional branches, context-managed resources.
**CN:** 该函数实现了 `_compute_scheduler_metadata`。 它主要通过调用 `get_pp_group`, `get_pp_indices`, `envs.SGLANG_MUSA_FA3_FORCE_UPDATE_METADATA.get`, `torch.empty`, `get_scheduler_metadata` 来完成任务。 状态更新主要写入 `layer`, `current_layer_id`, `batch_size`, `should_update`, `pp_group`, `pp_rank`。 实现中使用了条件分支、上下文管理资源。

### Lines 118-207: Function `flash_attn_with_kvcache` / 函数 `flash_attn_with_kvcache`
```python
def flash_attn_with_kvcache(
    q: torch.Tensor,
    k_cache: torch.Tensor,
    v_cache: torch.Tensor,
    k: Optional[torch.Tensor] = None,
    v: Optional[torch.Tensor] = None,
    qv: Optional[torch.Tensor] = None,
    rotary_cos: Optional[torch.Tensor] = None,
    rotary_sin: Optional[torch.Tensor] = None,
    cache_seqlens: Optional[Union[int, torch.Tensor]] = None,
    cache_batch_idx: Optional[torch.Tensor] = None,
    cache_leftpad: Optional[torch.Tensor] = None,
    page_table: Optional[torch.Tensor] = None,
    cu_seqlens_q: Optional[torch.Tensor] = None,
    cu_seqlens_k_new: Optional[torch.Tensor] = None,
    max_seqlen_q: Optional[int] = None,
    rotary_seqlens: Optional[torch.Tensor] = None,
    q_descale: Optional[torch.Tensor] = None,
    k_descale: Optional[torch.Tensor] = None,
    v_descale: Optional[torch.Tensor] = None,
    softmax_scale: Optional[float] = None,
    causal: bool = False,
    window_size: Tuple[int, int] = (-1, -1),
    attention_chunk: int = 0,
    softcap: float = 0.0,
    rotary_interleaved: bool = True,
    scheduler_metadata: Optional[torch.Tensor] = None,
    num_splits: int = 0,
# ... omitted for brevity ...
        num_splits=num_splits,
        pack_gqa=pack_gqa,
        sm_margin=sm_margin,
        return_softmax_lse=return_softmax_lse,
        sinks=sinks,
    )
```
**EN:** This function implements `flash_attn_with_kvcache`. It primarily calls `mate_flash_attn_with_kvcache`, `ValueError`, `_compute_scheduler_metadata` to complete its work. State updates are written into `backend`, `page_size`, `scheduler_metadata`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `flash_attn_with_kvcache`。 它主要通过调用 `mate_flash_attn_with_kvcache`, `ValueError`, `_compute_scheduler_metadata` 来完成任务。 状态更新主要写入 `backend`, `page_size`, `scheduler_metadata`。 实现中使用了条件分支。

### Lines 210-210: Class `MusaFlashAttentionBackend` declaration / 类 `MusaFlashAttentionBackend` 声明
```python
class MusaFlashAttentionBackend(FlashAttentionBackend):
```
**EN:** This class establishes `MusaFlashAttentionBackend` as the main container/coordinator for the surrounding logic. It inherits from `FlashAttentionBackend`. Its core interface includes methods such as `__init__`, `_set_current_state`, `forward_extend`, `forward_decode`.
**CN:** 该类将 `MusaFlashAttentionBackend` 定义为周边逻辑的主要封装体或协调者。 它继承自 `FlashAttentionBackend`。 其核心接口包括 `__init__`, `_set_current_state`, `forward_extend`, `forward_decode` 等方法。

### Lines 211-228: Method `MusaFlashAttentionBackend.__init__` / 方法 `MusaFlashAttentionBackend.__init__`
```python
    def __init__(self, model_runner: ModelRunner, **kwargs):
        super().__init__(model_runner, **kwargs)
        self.num_hidden_layers = model_runner.model_config.num_hidden_layers
        self.first_k_dense_replace = model_runner.model_config.first_k_dense_replace
        self.full_attention_interval = model_runner.model_config.full_attention_interval

        # State for current attention call (simplified from thread‑local context)
        self._current_layer: Optional[RadixAttention] = None
        self._current_prefix: str = ""
        self._current_max_seqlen_k: int = 0
        self._current_can_run_tbo: bool = False

        # Disable default scheduler metadata for fa3
        self._get_scheduler_metadata = None

        # Register this backend as the global current instance for the wrapper
        global _CURRENT_BACKEND
        _CURRENT_BACKEND = self
```
**EN:** This method implements `__init__` on `MusaFlashAttentionBackend`. It primarily calls `super.__init__`, `super` to complete its work. State updates are written into `self.num_hidden_layers`, `self.first_k_dense_replace`, `self.full_attention_interval`, `self._current_layer`, `self._current_prefix`, `self._current_max_seqlen_k`.
**CN:** 该方法（属于 `MusaFlashAttentionBackend`）实现了 `__init__`。 它主要通过调用 `super.__init__`, `super` 来完成任务。 状态更新主要写入 `self.num_hidden_layers`, `self.first_k_dense_replace`, `self.full_attention_interval`, `self._current_layer`, `self._current_prefix`, `self._current_max_seqlen_k`。

### Lines 230-237: Method `MusaFlashAttentionBackend._set_current_state` / 方法 `MusaFlashAttentionBackend._set_current_state`
```python
    def _set_current_state(
        self, layer: RadixAttention, prefix: str, max_seqlen_k: int, can_run_tbo: bool
    ):
        """Set the dynamic state for the upcoming flash attention call."""
        self._current_layer = layer
        self._current_prefix = prefix
        self._current_max_seqlen_k = max_seqlen_k
        self._current_can_run_tbo = can_run_tbo
```
**EN:** This method implements `_set_current_state` on `MusaFlashAttentionBackend`. State updates are written into `self._current_layer`, `self._current_prefix`, `self._current_max_seqlen_k`, `self._current_can_run_tbo`.
**CN:** 该方法（属于 `MusaFlashAttentionBackend`）实现了 `_set_current_state`。 状态更新主要写入 `self._current_layer`, `self._current_prefix`, `self._current_max_seqlen_k`, `self._current_can_run_tbo`。

### Lines 239-637: Method `MusaFlashAttentionBackend.forward_extend` / 方法 `MusaFlashAttentionBackend.forward_extend`
```python
    def forward_extend(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        save_kv_cache=True,
        q_rope: Optional[torch.Tensor] = None,
        k_rope: Optional[torch.Tensor] = None,
        sinks: Optional[torch.Tensor] = None,
    ):
        if k is not None:
            assert v is not None

            is_cp_mode = (
                forward_batch.forward_mode.is_context_parallel_extend()
                and forward_batch.attn_cp_metadata is not None
                and self.attn_cp_size > 1
            )

            if save_kv_cache and not is_cp_mode:
                cache_loc = (
                    forward_batch.out_cache_loc
                    if not layer.is_cross_attention
                    else forward_batch.encoder_out_cache_loc
                )
                if not self.use_mla:
# ... omitted for brevity ...
                        softmax_lse_expand.T.contiguous(),
                    )
                else:
                    o = result

        return o.view(-1, layer.tp_q_head_num * layer.v_head_dim)
```
**EN:** This method implements `forward_extend` on `MusaFlashAttentionBackend`. It primarily calls `self._set_current_state`, `o.view`, `q.to`, `forward_batch.forward_mode.is_target_verify`, `forward_batch.token_to_kv_pool.get_kv_buffer`, `key_cache.view` to complete its work. State updates are written into `metadata`, `is_swa_layer`, `window_size`, `causal`, `use_local_attn`, `use_cascade_attn`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MusaFlashAttentionBackend`）实现了 `forward_extend`。 它主要通过调用 `self._set_current_state`, `o.view`, `q.to`, `forward_batch.forward_mode.is_target_verify`, `forward_batch.token_to_kv_pool.get_kv_buffer`, `key_cache.view` 来完成任务。 状态更新主要写入 `metadata`, `is_swa_layer`, `window_size`, `causal`, `use_local_attn`, `use_cascade_attn`。 实现中使用了条件分支。

### Lines 639-913: Method `MusaFlashAttentionBackend.forward_decode` / 方法 `MusaFlashAttentionBackend.forward_decode`
```python
    def forward_decode(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        layer: RadixAttention,
        forward_batch: ForwardBatch,
        save_kv_cache=True,
        q_rope: Optional[torch.Tensor] = None,
        k_rope: Optional[torch.Tensor] = None,
        sinks: Optional[torch.Tensor] = None,
    ) -> torch.Tensor:
        if k is not None:
            assert v is not None
            if save_kv_cache:
                cache_loc = (
                    forward_batch.out_cache_loc
                    if not layer.is_cross_attention
                    else forward_batch.encoder_out_cache_loc
                )
                if not self.use_mla:
                    forward_batch.token_to_kv_pool.set_kv_buffer(
                        layer, cache_loc, k, v, layer.k_scale, layer.v_scale
                    )
                else:
                    forward_batch.token_to_kv_pool.set_mla_kv_buffer(
                        layer,
                        cache_loc,
# ... omitted for brevity ...
                    softmax_lse_expand.T.contiguous(),
                )
            else:
                o = result

        return o.view(-1, layer.tp_q_head_num * layer.v_head_dim)
```
**EN:** This method implements `forward_decode` on `MusaFlashAttentionBackend`. It primarily calls `getattr`, `self._set_current_state`, `o.view`, `q.to`, `forward_batch.token_to_kv_pool.get_kv_buffer`, `key_cache.view` to complete its work. State updates are written into `metadata`, `local_attn_metadata`, `use_local_attn`, `use_cascade_attn`, `is_swa_layer`, `window_size`. The implementation relies on conditional branches.
**CN:** 该方法（属于 `MusaFlashAttentionBackend`）实现了 `forward_decode`。 它主要通过调用 `getattr`, `self._set_current_state`, `o.view`, `q.to`, `forward_batch.token_to_kv_pool.get_kv_buffer`, `key_cache.view` 来完成任务。 状态更新主要写入 `metadata`, `local_attn_metadata`, `use_local_attn`, `use_cascade_attn`, `is_swa_layer`, `window_size`。 实现中使用了条件分支。

### Lines 916-916: Class `MusaFlashAttentionMultiStepBackend` declaration / 类 `MusaFlashAttentionMultiStepBackend` 声明
```python
class MusaFlashAttentionMultiStepBackend(FlashAttentionMultiStepBackend):
```
**EN:** This class establishes `MusaFlashAttentionMultiStepBackend` as the main container/coordinator for the surrounding logic. It inherits from `FlashAttentionMultiStepBackend`. Its core interface includes methods such as `__init__`.
**CN:** 该类将 `MusaFlashAttentionMultiStepBackend` 定义为周边逻辑的主要封装体或协调者。 它继承自 `FlashAttentionMultiStepBackend`。 其核心接口包括 `__init__` 等方法。

### Lines 918-938: Method `MusaFlashAttentionMultiStepBackend.__init__` / 方法 `MusaFlashAttentionMultiStepBackend.__init__`
```python
    def __init__(
        self,
        model_runner: ModelRunner,
        topk: int,
        speculative_num_steps: int,
        fa_impl_ver: int = 3,
    ):
        self.model_runner = model_runner
        self.topk = topk
        self.speculative_num_steps = speculative_num_steps
        self.attn_backends = []
        for i in range(self.speculative_num_steps - 1):
            self.attn_backends.append(
                MusaFlashAttentionBackend(
                    model_runner,
                    speculative_step_id=i,
                    topk=self.topk,
                    speculative_num_steps=self.speculative_num_steps,
                    fa_impl_ver=fa_impl_ver,
                )
            )
```
**EN:** This method implements `__init__` on `MusaFlashAttentionMultiStepBackend`. It primarily calls `range`, `self.attn_backends.append`, `MusaFlashAttentionBackend` to complete its work. State updates are written into `self.model_runner`, `self.topk`, `self.speculative_num_steps`, `self.attn_backends`. The implementation relies on iteration.
**CN:** 该方法（属于 `MusaFlashAttentionMultiStepBackend`）实现了 `__init__`。 它主要通过调用 `range`, `self.attn_backends.append`, `MusaFlashAttentionBackend` 来完成任务。 状态更新主要写入 `self.model_runner`, `self.topk`, `self.speculative_num_steps`, `self.attn_backends`。 实现中使用了迭代逻辑。

## Key Concepts / 关键概念
- **Classes / 类**: `MusaFlashAttentionBackend`, `MusaFlashAttentionMultiStepBackend`
- **Functions / 函数**: `_compute_scheduler_metadata`, `flash_attn_with_kvcache`, `__init__`, `_set_current_state`, `forward_extend`, `forward_decode`, `__init__`, `_fa_cp_attn`
- **Themes / 主题**: `attention`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: `sglang.srt.distributed`, `sglang.srt.environ`, `sglang.srt.hardware_backend.musa.layers.utils.cp_utils`, `sglang.srt.layers.attention.flashattention_backend`, `sglang.srt.layers.radix_attention`, `sglang.srt.layers.utils.cp_utils`, `sglang.srt.server_args`, `sglang.srt.model_executor.forward_batch_info`, `sglang.srt.model_executor.model_runner`
- **External / 外部依赖**: `torch`, `flash_attn_interface`
- **Standard library / 标准库**: `__future__`, `threading`, `typing`
