# flashinfer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/flashinfer.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Attention layer with FlashInfer. / 该模块位于 `attention/backends` 子系统，主要围绕 `_get_trtllm_gen_workspace_buffer`, `_trtllm_prefill_attn_kvfp8_dequant`, `trtllm_prefill_attn_kvfp8_dequant` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""Attention layer with FlashInfer."""

from dataclasses import dataclass
from functools import partial
from typing import ClassVar

import numpy as np
import torch
from flashinfer import (
    BatchDecodeWithPagedKVCacheWrapper,
    BatchPrefillWithPagedKVCacheWrapper,
    BatchPrefillWithRaggedKVCacheWrapper,
    MultiLevelCascadeAttentionWrapper,
)
from flashinfer.decode import fast_decode_plan, trtllm_batch_decode_with_kv_cache
from flashinfer.prefill import trtllm_batch_context_with_kv_cache
from flashinfer.utils import FP4Tensor
from typing_extensions import override

from vllm import envs
from vllm.config import (
    CUDAGraphMode,
    VllmConfig,
    get_current_vllm_config_or_none,
)
from vllm.config.cache import CacheDType
from vllm.distributed.parallel_state import get_dcp_group
from vllm.logger import init_logger
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    QuantKey,
    kFp8StaticTensorSym,
    kNvfp4Dynamic,
)
from vllm.platforms import current_platform
from vllm.platforms.interface import DeviceCapability
from vllm.triton_utils import tl, triton
from vllm.utils.flashinfer import (
    can_use_trtllm_attention,
    use_trtllm_attention,
)
from vllm.utils.math_utils import cdiv
from vllm.utils.platform_utils import is_pin_memory_available
from vllm.utils.torch_utils import (
    canonicalize_singleton_dim_strides,
    is_quantized_kv_cache,
    is_strictly_contiguous,
    nvfp4_kv_cache_full_dim,
    nvfp4_kv_cache_split_views,
)
from vllm.v1.attention.backend import (
# ... omitted for brevity ...
    KVCacheLayoutType,
    get_dcp_local_seq_lens,
    get_kv_cache_layout,
    get_per_layer_parameters,
    infer_global_hyperparameters,
    split_decodes_and_prefills,
)
from vllm.v1.attention.ops.common import cp_lse_ag_out_rs
from vllm.v1.attention.ops.dcp_alltoall import dcp_a2a_lse_reduce
from vllm.v1.attention.ops.merge_attn_states import merge_attn_states
from vllm.v1.kv_cache_interface import (
    AttentionSpec,
    KVQuantMode,
    UniformTypeKVCacheSpecs,
)
from vllm.v1.utils import CpuGpuBuffer

FLASHINFER_WORKSPACE_BUFFER_SIZE_BATCH_INVARIANT = 2048 * 1024 * 1024

FP8_DTYPE = current_platform.fp8_dtype()
FP4_DTYPE = torch.uint8

logger = init_logger(__name__)

trtllm_gen_workspace_buffer = None
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `FLASHINFER_WORKSPACE_BUFFER_SIZE_BATCH_INVARIANT`, `FP8_DTYPE`, `FP4_DTYPE`, `logger`, `trtllm_gen_workspace_buffer`. This excerpt omits repetitive declarations.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `FLASHINFER_WORKSPACE_BUFFER_SIZE_BATCH_INVARIANT`, `FP8_DTYPE`, `FP4_DTYPE`, `logger`, `trtllm_gen_workspace_buffer`。 该片段省略了重复性声明。

### `_get_trtllm_gen_workspace_buffer` function / `_get_trtllm_gen_workspace_buffer` 函数
```python
def _get_trtllm_gen_workspace_buffer():
    global trtllm_gen_workspace_buffer
    if trtllm_gen_workspace_buffer is None:
        trtllm_gen_workspace_buffer = torch.zeros(
            envs.VLLM_FLASHINFER_WORKSPACE_BUFFER_SIZE, dtype=torch.uint8, device="cuda"
        )
    return trtllm_gen_workspace_buffer
```
**EN:** This function implements `_get_trtllm_gen_workspace_buffer` within the module. Key calls include `zeros`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_get_trtllm_gen_workspace_buffer`，其作用域位于the module。 关键调用包括 `zeros`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_trtllm_prefill_attn_kvfp8_dequant` function / `_trtllm_prefill_attn_kvfp8_dequant` 函数
```python
@triton.jit
def _trtllm_prefill_attn_kvfp8_dequant(
    kv_cache_ptr,
    block_tables_prefill_ptr,
    block_table_stride,
    mock_kv_cache_ptr,
    k_scale_ptr,
    v_scale_ptr,
    src_stride_page,
    src_stride_kv,
    src_stride_head,
    DST_K_CACHE_STRIDE: tl.constexpr,
    DST_KV_CACHE_STRIDE: tl.constexpr,
    HEAD_STRIDE: tl.constexpr,
    NUM_KV_HEADS: tl.constexpr,
):
    batch_idx = tl.program_id(0).to(tl.int64)
    mock_block_table_idx = tl.program_id(1).to(tl.int64)
    orig_page_num = tl.load(
        block_tables_prefill_ptr + batch_idx * block_table_stride + mock_block_table_idx
    ).to(tl.int64)
    if orig_page_num <= 0:
        return
    dequant_dtype = mock_kv_cache_ptr.dtype.element_ty

    k_scale_val = tl.load(k_scale_ptr)
    v_scale_val = tl.load(v_scale_ptr)

    mock_page_idx = batch_idx * block_table_stride + mock_block_table_idx + 1
    head_offsets = tl.arange(0, HEAD_STRIDE)

    for h in range(NUM_KV_HEADS):
        h_off = tl.cast(h, tl.int64)

        # Read K from source (supports non-contiguous page/kv/head strides)
        src_k = orig_page_num * src_stride_page + h_off * src_stride_head + head_offsets
        fp8_k = tl.load(kv_cache_ptr + src_k)
        dequant_k = (fp8_k.to(tl.float32) * k_scale_val).to(dequant_dtype)

        # Write K to contiguous mock cache
        dst_k = mock_page_idx * DST_KV_CACHE_STRIDE + h * HEAD_STRIDE + head_offsets
        tl.store(mock_kv_cache_ptr + dst_k, dequant_k)

        # Read V from source (offset by src_stride_kv for the V half)
        src_v = (
            orig_page_num * src_stride_page
            + src_stride_kv
            + h_off * src_stride_head
            + head_offsets
        )
        fp8_v = tl.load(kv_cache_ptr + src_v)
        dequant_v = (fp8_v.to(tl.float32) * v_scale_val).to(dequant_dtype)

        # Write V to contiguous mock cache
        dst_v = (
            mock_page_idx * DST_KV_CACHE_STRIDE
            + DST_K_CACHE_STRIDE
            + h * HEAD_STRIDE
            + head_offsets
        )
        tl.store(mock_kv_cache_ptr + dst_v, dequant_v)
```
**EN:** This function implements `_trtllm_prefill_attn_kvfp8_dequant` within the module. Key calls include `to`, `load`, `arange`, `range`, `cast`, `store`. The control flow contains 1 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_trtllm_prefill_attn_kvfp8_dequant`，其作用域位于the module。 关键调用包括 `to`, `load`, `arange`, `range`, `cast`, `store`。 控制流包含 1 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `trtllm_prefill_attn_kvfp8_dequant` function / `trtllm_prefill_attn_kvfp8_dequant` 函数
```python
def trtllm_prefill_attn_kvfp8_dequant(
    kv_cache: torch.Tensor,
    block_tables_prefill: torch.Tensor,
    k_scale: torch.Tensor,
    v_scale: torch.Tensor,
    dequant_dtype: torch.dtype,
) -> tuple[torch.Tensor, torch.Tensor]:
    batch_size, num_of_page_per_token = block_tables_prefill.shape
    s = kv_cache.shape
    assert s[1] == 2
    assert dequant_dtype in (torch.bfloat16, torch.float16)

    num_kv_heads, block_size, head_size = s[2], s[3], s[4]
    head_stride = block_size * head_size
    k_cache_stride = num_kv_heads * head_stride
    kv_cache_stride = k_cache_stride * s[1]

    strides = kv_cache.stride()
    assert strides[3] == head_size and strides[4] == 1, (
        "For kv cache layouts, (block_size, head_size) "
        f"dimensions must be contiguous, got strides {strides}"
    )

    new_s = (batch_size * num_of_page_per_token + 1, s[1], s[2], s[3], s[4])
    # mock kv cache contains just the pages needed by this prefill
    mock_kv_cache = torch.empty(new_s, dtype=dequant_dtype, device=kv_cache.device)
    # we simply sequentially index the pages needed by this prefill
    mock_block_table = torch.arange(
        start=1,
        end=batch_size * num_of_page_per_token + 1,
        dtype=torch.int32,
        device=block_tables_prefill.device,
    ).reshape(batch_size, num_of_page_per_token)
    grid = (batch_size, num_of_page_per_token)
    _trtllm_prefill_attn_kvfp8_dequant[grid](
        kv_cache,
        block_tables_prefill,
        num_of_page_per_token,
        mock_kv_cache,
        k_scale,
        v_scale,
        strides[0],
        strides[1],
        strides[2],
        k_cache_stride,
        kv_cache_stride,
        head_stride,
        num_kv_heads,
    )
    return mock_kv_cache, mock_block_table
```
**EN:** This function implements `trtllm_prefill_attn_kvfp8_dequant` within the module. Key calls include `stride`, `empty`, `reshape`, `arange`.
**CN:** 该函数会实现 `trtllm_prefill_attn_kvfp8_dequant`，其作用域位于the module。 关键调用包括 `stride`, `empty`, `reshape`, `arange`。

### `BatchDCPPrefillWrapper` class / `BatchDCPPrefillWrapper` 类
```python
class BatchDCPPrefillWrapper:
```
**EN:** Introduces the `BatchDCPPrefillWrapper` class. Core methods include `__init__`, `plan`, `run`.
**CN:** 这里定义 `BatchDCPPrefillWrapper` 类。核心方法包括 `__init__`, `plan`, `run`。

### `BatchDCPPrefillWrapper.__init__` method / `BatchDCPPrefillWrapper.__init__` 方法
```python
    def __init__(
        self,
        workspace_buffer: torch.Tensor | None = None,
        dcp_a2a: bool = False,
    ):
        if dcp_a2a:
            self._dcp_combine = partial(dcp_a2a_lse_reduce, is_lse_base_on_e=False)
        else:
            self._dcp_combine = partial(cp_lse_ag_out_rs, is_lse_base_on_e=False)
        self._context = BatchPrefillWithPagedKVCacheWrapper(
            workspace_buffer, get_kv_cache_layout()
        )
        self._new_tokens = BatchPrefillWithRaggedKVCacheWrapper(workspace_buffer)
```
**EN:** This method initializes the object state within `BatchDCPPrefillWrapper`. Key calls include `BatchPrefillWithPagedKVCacheWrapper`, `BatchPrefillWithRaggedKVCacheWrapper`, `partial`, `get_kv_cache_layout`. It touches state such as `_context`, `_new_tokens`, `_dcp_combine`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`BatchDCPPrefillWrapper`。 关键调用包括 `BatchPrefillWithPagedKVCacheWrapper`, `BatchPrefillWithRaggedKVCacheWrapper`, `partial`, `get_kv_cache_layout`。 它会读写 `_context`, `_new_tokens`, `_dcp_combine` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `BatchDCPPrefillWrapper.plan` method / `BatchDCPPrefillWrapper.plan` 方法
```python
    def plan(
        self,
        qo_indptr_cpu: torch.Tensor,
        paged_kv_indptr_cpu: torch.Tensor,
        paged_kv_indices: torch.Tensor,
        paged_kv_last_page_len_cpu: torch.Tensor,
        page_size: int,
        num_qo_heads: int,
        dcp_world_size: int,
        num_kv_heads: int,
        head_dim: int,
        sm_scale: float,
        window_left: int,
        logits_soft_cap: float | None,
        q_data_type: torch.dtype,
        kv_cache_dtype: torch.dtype,
        prefill_fixed_split_size: int,
        disable_split_kv: bool,
    ):
        """Plan the prefill operation with given parameters."""
        self._context.plan(
            qo_indptr=qo_indptr_cpu,
            paged_kv_indptr=paged_kv_indptr_cpu,
            paged_kv_indices=paged_kv_indices,
            paged_kv_last_page_len=paged_kv_last_page_len_cpu,
            num_qo_heads=num_qo_heads * dcp_world_size,
            num_kv_heads=num_kv_heads,
            head_dim_qk=head_dim,
            page_size=page_size,
            causal=False,  # This is context run
            sm_scale=sm_scale,
            window_left=window_left,
            logits_soft_cap=logits_soft_cap,
            q_data_type=q_data_type,
            kv_data_type=kv_cache_dtype,
            fixed_split_size=prefill_fixed_split_size,
            disable_split_kv=disable_split_kv,
        )
        self._new_tokens.plan(
            qo_indptr=qo_indptr_cpu,
            kv_indptr=qo_indptr_cpu,
            num_qo_heads=num_qo_heads,
            num_kv_heads=num_kv_heads,
            head_dim_qk=head_dim,
            head_dim_vo=head_dim,
            causal=True,  # This is newtokens run
            sm_scale=sm_scale,
            window_left=window_left,
            logits_soft_cap=logits_soft_cap,
            q_data_type=q_data_type,
        )
```
**EN:** This method implements `plan` within `BatchDCPPrefillWrapper`. The docstring frames it as: Plan the prefill operation with given parameters. Key calls include `plan`.
**CN:** 该方法会实现 `plan`，其作用域位于`BatchDCPPrefillWrapper`。 关键调用包括 `plan`。

### `BatchDCPPrefillWrapper.run` method / `BatchDCPPrefillWrapper.run` 方法
```python
    def run(
        self,
        layer: torch.nn.Module,
        prefill_query: torch.Tensor,
        kv_cache_permute: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        out: torch.Tensor,
    ):
        prefill_query_across_dcp = get_dcp_group().all_gather(
            prefill_query.contiguous(), dim=1
        )
        output_context_tmp, lse_context_tmp = self._context.run(
            prefill_query_across_dcp,
            kv_cache_permute,
            k_scale=layer._k_scale_float,
            v_scale=layer._v_scale_float,
            return_lse=True,
        )
        output_context, lse_context = self._dcp_combine(
            output_context_tmp,
            lse_context_tmp,
            get_dcp_group(),
            return_lse=True,
        )
        lse_context = lse_context.transpose(0, 1).contiguous()

        output_query, lse_query = self._new_tokens.run(
            prefill_query,
            key,
            value,
            return_lse=True,
        )
        lse_query = lse_query.transpose(0, 1).contiguous()

        merge_attn_states(
            out,
            output_context,
            lse_context,
            output_query,
            lse_query,
        )
        return out
```
**EN:** This method runs the main execution path within `BatchDCPPrefillWrapper`. Key calls include `all_gather`, `run`, `_dcp_combine`, `contiguous`, `merge_attn_states`, `get_dcp_group`.
**CN:** 该方法会执行主要运行路径，其作用域位于`BatchDCPPrefillWrapper`。 关键调用包括 `all_gather`, `run`, `_dcp_combine`, `contiguous`, `merge_attn_states`, `get_dcp_group`。

### `FlashInferBackend` class / `FlashInferBackend` 类
```python
class FlashInferBackend(AttentionBackend):
    supported_dtypes: ClassVar[list[torch.dtype]] = [torch.float16, torch.bfloat16]
    supported_kv_cache_dtypes: ClassVar[list[CacheDType]] = [
        "auto",
        "float16",
        "bfloat16",
        "fp8",
        "fp8_e4m3",
        "fp8_e5m2",
        "nvfp4",
    ]
```
**EN:** Introduces the `FlashInferBackend` class on top of `AttentionBackend`. Core methods include `get_supported_kernel_block_sizes`, `get_name`, `get_impl_cls`, `get_builder_cls`, `get_kv_cache_shape`, `get_kv_cache_stride_order`.
**CN:** 这里定义 `FlashInferBackend` 类，其基类包括 `AttentionBackend`。核心方法包括 `get_supported_kernel_block_sizes`, `get_name`, `get_impl_cls`, `get_builder_cls`, `get_kv_cache_shape`, `get_kv_cache_stride_order`。

### `FlashInferBackend.get_builder_cls` method / `FlashInferBackend.get_builder_cls` 方法
```python
    @staticmethod
    def get_builder_cls() -> type["FlashInferMetadataBuilder"]:
        return FlashInferMetadataBuilder
```
**EN:** This method returns or derives a value within `FlashInferBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`FlashInferBackend`。

### `FlashInferBackend.get_kv_cache_stride_order` method / `FlashInferBackend.get_kv_cache_stride_order` 方法
```python
    @staticmethod
    def get_kv_cache_stride_order(
        include_num_layers_dimension: bool = False,
    ) -> tuple[int, ...]:
        # `stride_order` indicates the permutation that gets us from
        # `get_kv_cache_shape` to the actual memory layout we want.
        cache_layout = get_kv_cache_layout()
        if cache_layout == "NHD" and include_num_layers_dimension:
            # (num_blocks, num_layers, 2, block_size, num_kv_heads, head_size)
            return (1, 0, 2, 3, 4, 5)
        elif cache_layout == "NHD":
            stride_order = (0, 1, 2, 3, 4)
        elif cache_layout == "HND" and include_num_layers_dimension:
            # (num_blocks, 2, num_kv_heads, num_layers, block_size, head_size)
            return (1, 2, 4, 0, 3, 5)
        elif cache_layout == "HND":
            stride_order = (0, 1, 3, 2, 4)
        else:
            raise ValueError(f"Unknown cache layout format {cache_layout}.")
        return stride_order
```
**EN:** This method returns or derives a value within `FlashInferBackend`. Key calls include `get_kv_cache_layout`, `ValueError`. The control flow contains 4 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会返回或推导一个值，其作用域位于`FlashInferBackend`。 关键调用包括 `get_kv_cache_layout`, `ValueError`。 控制流包含 4 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FlashInferBackend.supports_compute_capability` method / `FlashInferBackend.supports_compute_capability` 方法
```python
    @classmethod
    def supports_compute_capability(cls, capability: DeviceCapability) -> bool:
        return capability >= DeviceCapability(7, 5) and capability <= DeviceCapability(
            12, 1
        )
```
**EN:** This method implements `supports_compute_capability` within `FlashInferBackend`. Key calls include `DeviceCapability`.
**CN:** 该方法会实现 `supports_compute_capability`，其作用域位于`FlashInferBackend`。 关键调用包括 `DeviceCapability`。

### `FlashInferBackend.supports_sink` method / `FlashInferBackend.supports_sink` 方法
```python
    @classmethod
    def supports_sink(cls) -> bool:
        """FlashInfer supports sinks when TRTLLM attention is available (SM100)."""
        from vllm.utils.flashinfer import (
            force_use_trtllm_attention,
            supports_trtllm_attention,
        )

        # Respect explicit disable flag (e.g.,
        # --attention-config.use_trtllm_attention=0)
        if force_use_trtllm_attention() is False:
            return False

        # Check if TRTLLM is supported on this platform
        return supports_trtllm_attention()
```
**EN:** This method implements `supports_sink` within `FlashInferBackend`. The docstring frames it as: FlashInfer supports sinks when TRTLLM attention is available (SM100). Key calls include `supports_trtllm_attention`, `force_use_trtllm_attention`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `supports_sink`，其作用域位于`FlashInferBackend`。 关键调用包括 `supports_trtllm_attention`, `force_use_trtllm_attention`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FIPrefill` class / `FIPrefill` 类
```python
@dataclass
class FIPrefill:
    """Metadata for the native FlashInfer prefill pathway (non-TRTLLM)."""

    wrapper: BatchPrefillWithPagedKVCacheWrapper | BatchDCPPrefillWrapper
```
**EN:** Uses `@dataclass` to package related state for `FIPrefill`. Typical fields include `wrapper`.
**CN:** `FIPrefill` 使用 `@dataclass` 打包相关状态。典型字段包括 `wrapper`。

### `FIDecode` class / `FIDecode` 类
```python
@dataclass
class FIDecode:
    """Metadata for the native FlashInfer decode pathway (non-TRTLLM)."""

    wrapper: BatchDecodeWithPagedKVCacheWrapper
```
**EN:** Uses `@dataclass` to package related state for `FIDecode`. Typical fields include `wrapper`.
**CN:** `FIDecode` 使用 `@dataclass` 打包相关状态。典型字段包括 `wrapper`。

### `TRTLLMPrefill` class / `TRTLLMPrefill` 类
```python
@dataclass
class TRTLLMPrefill:
    """Metadata for the TRTLLM prefill pathway."""

    block_tables: torch.Tensor
    """
    The slice of the block table tensor corresponding *only* to prefill requests.
    Shape: [num_prefills, max_num_blocks_per_seq]
    """

    seq_lens: torch.Tensor
    """
    The slice of the sequence lengths tensor corresponding *only* to prefill requests.
    Shape: [num_prefills]
    """

    cum_seq_lens_q: torch.Tensor
    cum_seq_lens_kv: torch.Tensor

    max_q_len: int
    """
    The maximum query length *among prefill requests*.
    """

    max_seq_len: int
    """The maximum sequence length for KV Cache."""
```
**EN:** Uses `@dataclass` to package related state for `TRTLLMPrefill`. Typical fields include `block_tables`, `seq_lens`, `cum_seq_lens_q`, `cum_seq_lens_kv`, `max_q_len`, `max_seq_len`.
**CN:** `TRTLLMPrefill` 使用 `@dataclass` 打包相关状态。典型字段包括 `block_tables`, `seq_lens`, `cum_seq_lens_q`, `cum_seq_lens_kv`, `max_q_len`, `max_seq_len`。

### `TRTLLMDecode` class / `TRTLLMDecode` 类
```python
@dataclass
class TRTLLMDecode:
    """Metadata for the TRTLLM decode pathway."""

    block_tables: torch.Tensor
    """
    The slice of the block table tensor corresponding *only* to decode requests.
    Shape: [num_decodes, max_num_blocks_per_seq]
    """

    seq_lens: torch.Tensor
    """
    The slice of the sequence lengths tensor corresponding *only* to decode requests.
    Shape: [num_decodes]
    """

    max_seq_len: int
    """The maximum sequence length for KV Cache."""
```
**EN:** Uses `@dataclass` to package related state for `TRTLLMDecode`. Typical fields include `block_tables`, `seq_lens`, `max_seq_len`.
**CN:** `TRTLLMDecode` 使用 `@dataclass` 打包相关状态。典型字段包括 `block_tables`, `seq_lens`, `max_seq_len`。

### `FlashInferMetadata` class / `FlashInferMetadata` 类
```python
@dataclass
class FlashInferMetadata:
    num_actual_tokens: int
    """Total number of tokens in the batch (excluding padding)."""

    slot_mapping: torch.Tensor
    """Tensor for writing K/V to the cache. Shape: [num_actual_tokens]"""

    q_data_type: torch.dtype

    num_decodes: int
    num_decode_tokens: int
    num_prefills: int
    num_prefill_tokens: int

    prefill: FIPrefill | TRTLLMPrefill | None
    """
    Holds the metadata for the prefill portion of the batch.
    Will be `None` if `num_prefill_tokens == 0`.
    """

    decode: FIDecode | TRTLLMDecode | None
    """
    Holds the metadata for the decode portion of the batch.
    Will be `None` if `num_decode_tokens == 0`.
    """

    # --- Special Case: Cascade Attention ---

    use_cascade: bool
    """
    If True, the entire batch is a cascade attention call, and the
    `prefill` and `decode` fields will both be None.
    """

    cascade_wrapper: MultiLevelCascadeAttentionWrapper | None
```
**EN:** Uses `@dataclass` to package related state for `FlashInferMetadata`. Typical fields include `num_actual_tokens`, `slot_mapping`, `q_data_type`, `num_decodes`, `num_decode_tokens`, `num_prefills`.
**CN:** `FlashInferMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `num_actual_tokens`, `slot_mapping`, `q_data_type`, `num_decodes`, `num_decode_tokens`, `num_prefills`。

### `FlashInferMetadataBuilder` class / `FlashInferMetadataBuilder` 类
```python
class FlashInferMetadataBuilder(AttentionMetadataBuilder[FlashInferMetadata]):
    reorder_batch_threshold: int = 1
```
**EN:** Introduces the `FlashInferMetadataBuilder` class on top of `AttentionMetadataBuilder[FlashInferMetadata]`. Core methods include `__init__`, `_make_buffer`, `get_cudagraph_support`, `_get_workspace_buffer`, `set_workspace_buffer`, `_get_prefill_wrapper`.
**CN:** 这里定义 `FlashInferMetadataBuilder` 类，其基类包括 `AttentionMetadataBuilder[FlashInferMetadata]`。核心方法包括 `__init__`, `_make_buffer`, `get_cudagraph_support`, `_get_workspace_buffer`, `set_workspace_buffer`, `_get_prefill_wrapper`。

### `FlashInferMetadataBuilder.__init__` method / `FlashInferMetadataBuilder.__init__` 方法
```python
    def __init__(
        self,
        kv_cache_spec: AttentionSpec,
        layer_names: list[str],
        vllm_config: VllmConfig,
        device: torch.device,
    ):
        super().__init__(kv_cache_spec, layer_names, vllm_config, device)
        self.cache_config = vllm_config.cache_config
        self.model_config = vllm_config.model_config
        self.attention_config = vllm_config.attention_config
        self._workspace_buffer = None
        self._prefill_wrapper: (
            BatchPrefillWithPagedKVCacheWrapper | BatchDCPPrefillWrapper | None
        ) = None  # Wrapper for prefill/append
        self._decode_wrapper = None  # Wrapper for decode (general shape)

        if envs.VLLM_BATCH_INVARIANT:
            self.decode_fixed_split_size = 2048
            self.prefill_fixed_split_size = 4096
            self.disable_split_kv = True
        else:
            self.decode_fixed_split_size = -1
            self.prefill_fixed_split_size = -1
            self.disable_split_kv = False

        self.compilation_config = vllm_config.compilation_config
        max_num_pages_per_req = cdiv(
            self.model_config.max_model_len, self.kv_cache_spec.block_size
        )
        max_num_reqs = vllm_config.scheduler_config.max_num_seqs
        max_num_pages = max_num_reqs * max_num_pages_per_req
        speculative_config = vllm_config.speculative_config
        num_spec_tokens = (
            speculative_config.num_speculative_tokens
            if speculative_config is not None
            else 0
        )
        self.enable_cuda_graph = (
            self.compilation_config.cudagraph_mode.decode_mode() == CUDAGraphMode.FULL
        )
        if self.enable_cuda_graph:
            # For full cudagraph capture, one `decode_wrapper` for each batch
            # size is needed for FlashInfer.
            self._decode_wrappers_cudagraph: dict[
                int, BatchDecodeWithPagedKVCacheWrapper
            ] = {}
            self._decode_cudagraph_max_bs = (1 + num_spec_tokens) * max_num_reqs
            if self.compilation_config.max_cudagraph_capture_size is not None:
                self._decode_cudagraph_max_bs = min(
    # ... omitted for brevity ...
            get_per_layer_parameters(vllm_config, layer_names, FlashInferImpl)
        )
        self.sm_scale = self.global_hyperparameters.sm_scale
        self.window_left = self.global_hyperparameters.window_left
        self.logits_soft_cap = self.global_hyperparameters.logits_soft_cap
        self.has_sinks = self.global_hyperparameters.has_sinks
        if self.has_sinks and not can_use_trtllm:
            raise NotImplementedError(
                "FlashInfer backend currently does not support attention "
                "sinks, please use trtllm on blackwell or flash attention on "
                "earlier GPUs."
            )
        # Preparing persistent buffers
        # Since we do not have explicit synchronization in ModelRunnerV2, we do not pin
        # reused CPU buffers to avoid a race condition between step N async copies to
        # GPU and step N+1 buffer updates.
        self.pin_memory = (
            not vllm_config.use_v2_model_runner and is_pin_memory_available()
        )
        self.paged_kv_indptr = self._make_buffer(max_num_reqs + 1)
        self.paged_kv_indptr_cpu_buffer = torch.zeros_like(
            self.paged_kv_indptr.cpu, pin_memory=self.pin_memory
        )  # Extra buffer for mutable paged_kv_indptr.cpu in cuda graph mode
        self.paged_kv_indices = self._make_buffer(max_num_pages)
        self.paged_kv_last_page_len = self._make_buffer(max_num_reqs)
```
**EN:** This method initializes the object state within `FlashInferMetadataBuilder`. Key calls include `__init__`, `cdiv`, `get_num_attention_heads`, `can_use_trtllm_attention`, `_init_reorder_batch_threshold`, `infer_global_hyperparameters`. It touches state such as `cache_config`, `model_config`, `attention_config`, `_workspace_buffer`, `_prefill_wrapper`, `_decode_wrapper`, `compilation_config`, `enable_cuda_graph`. The control flow contains 10 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会初始化对象状态，其作用域位于`FlashInferMetadataBuilder`。 关键调用包括 `__init__`, `cdiv`, `get_num_attention_heads`, `can_use_trtllm_attention`, `_init_reorder_batch_threshold`, `infer_global_hyperparameters`。 它会读写 `cache_config`, `model_config`, `attention_config`, `_workspace_buffer`, `_prefill_wrapper`, `_decode_wrapper`, `compilation_config`, `enable_cuda_graph` 等状态。 控制流包含 10 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `FlashInferMetadataBuilder._get_decode_wrapper` method / `FlashInferMetadataBuilder._get_decode_wrapper` 方法
```python
    def _get_decode_wrapper(self, batch_size: int, use_cudagraph: bool = False):
        if use_cudagraph:
            decode_wrapper = self._decode_wrappers_cudagraph.get(batch_size, None)
        else:
            decode_wrapper = self._decode_wrapper

        if decode_wrapper is None:
            if use_cudagraph:
                paged_kv_indptr = self.paged_kv_indptr.gpu[: batch_size + 1]
                paged_kv_indices = self.paged_kv_indices.gpu
                paged_kv_last_page_len = self.paged_kv_last_page_len.gpu[:batch_size]
            else:
                paged_kv_indptr = None
                paged_kv_indices = None
                paged_kv_last_page_len = None
            # NVFP4 KV cache requires the trtllm-gen backend inside
            # the wrapper; fa2/fa3 do not support nvfp4.
            backend = "trtllm-gen" if self.is_kvcache_nvfp4 else "auto"
            decode_wrapper = BatchDecodeWithPagedKVCacheWrapper(
                self._get_workspace_buffer(),
                get_kv_cache_layout(),
                use_cuda_graph=use_cudagraph,
                paged_kv_indptr_buffer=paged_kv_indptr,
                paged_kv_indices_buffer=paged_kv_indices,
                paged_kv_last_page_len_buffer=paged_kv_last_page_len,
                # Tensor cores are enabled by default because the perf would be
                # at least as good as cuda cores for all attention ops in latest
                # gpus.
                use_tensor_cores=True,
                backend=backend,
            )

            # save the decode wrapper
            if use_cudagraph:
                self._decode_wrappers_cudagraph[batch_size] = decode_wrapper
            else:
                self._decode_wrapper = decode_wrapper

        return decode_wrapper
```
**EN:** This method implements `_get_decode_wrapper` within `FlashInferMetadataBuilder`. Key calls include `get`, `BatchDecodeWithPagedKVCacheWrapper`, `_get_workspace_buffer`, `get_kv_cache_layout`. It touches state such as `_decode_wrappers_cudagraph`, `_decode_wrapper`. The control flow contains 5 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_get_decode_wrapper`，其作用域位于`FlashInferMetadataBuilder`。 关键调用包括 `get`, `BatchDecodeWithPagedKVCacheWrapper`, `_get_workspace_buffer`, `get_kv_cache_layout`。 它会读写 `_decode_wrappers_cudagraph`, `_decode_wrapper` 等状态。 控制流包含 5 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FlashInferMetadataBuilder._compute_flashinfer_kv_metadata` method / `FlashInferMetadataBuilder._compute_flashinfer_kv_metadata` 方法
```python
    def _compute_flashinfer_kv_metadata(
        self,
        num_blocks_np: np.ndarray,
        seq_lens_np: np.ndarray,
        block_table_tensor: torch.Tensor,
        num_reqs: int,
        page_size: int,
    ) -> torch.Tensor:
        """
        Compute paged_kv_indptr, paged_kv_indices, paged_kv_last_page_len for FlashInfer
        attention.

        Results are stored in self.paged_kv_indptr,
        self.paged_kv_indices, self.paged_kv_last_page_len buffers.

        Returns paged_kv_indices, a GPU tensor with shape [num_actual_pages].
        """
        # write self.paged_kv_indptr_cpu inplace (0-index is always 0)
        np.cumsum(
            num_blocks_np,
            dtype=np.int32,
            out=self.paged_kv_indptr.np[1 : num_reqs + 1],
        )
        # NOTE(woosuk): Because self.paged_kv_indptr_cpu can be modified
        # after this line (e.g., for cuda graphs), we need to copy the data to
        # self.paged_kv_indptr_buffer to avoid race condition.
        self.paged_kv_indptr_cpu_buffer[: num_reqs + 1] = self.paged_kv_indptr.cpu[
            : num_reqs + 1
        ]
        paged_kv_indptr = self.paged_kv_indptr.gpu[: num_reqs + 1]
        paged_kv_indptr.copy_(
            self.paged_kv_indptr_cpu_buffer[: num_reqs + 1], non_blocking=True
        )

        # write self.paged_kv_indices inplace
        num_actual_pages = self.paged_kv_indptr.np[num_reqs]
        paged_kv_indices = self.paged_kv_indices.gpu[:num_actual_pages]
        _copy_page_indices_kernel[(num_reqs,)](
            paged_kv_indices,
            block_table_tensor,
            block_table_tensor.stride(0),
            paged_kv_indptr,
            BLOCK_SIZE=1024,
        )

        # write self.paged_kv_last_page_len_cpu inplace
        paged_kv_last_page_len_np = seq_lens_np % page_size
        self.paged_kv_last_page_len.np[:num_reqs] = np.where(
            (paged_kv_last_page_len_np == 0) & (seq_lens_np != 0),
            page_size,
            paged_kv_last_page_len_np,
        )
        self.paged_kv_last_page_len.gpu[:num_reqs].copy_(
            self.paged_kv_last_page_len.cpu[:num_reqs], non_blocking=True
        )
        return paged_kv_indices
```
**EN:** This method implements `_compute_flashinfer_kv_metadata` within `FlashInferMetadataBuilder`. The docstring frames it as: Compute paged_kv_indptr, paged_kv_indices, paged_kv_last_page_len for FlashInfer attention. Key calls include `cumsum`, `copy_`, `where`, `stride`. It touches state such as `paged_kv_indptr_cpu_buffer`, `paged_kv_last_page_len`.
**CN:** 该方法会实现 `_compute_flashinfer_kv_metadata`，其作用域位于`FlashInferMetadataBuilder`。 关键调用包括 `cumsum`, `copy_`, `where`, `stride`。 它会读写 `paged_kv_indptr_cpu_buffer`, `paged_kv_last_page_len` 等状态。

### `FlashInferMetadataBuilder.build` method / `FlashInferMetadataBuilder.build` 方法
```python
    def build(
        self,
        common_prefix_len: int,
        common_attn_metadata: CommonAttentionMetadata,
        fast_build: bool = False,
    ) -> FlashInferMetadata:
        num_reqs = common_attn_metadata.num_reqs
        num_actual_tokens = common_attn_metadata.num_actual_tokens
        num_decodes, num_prefills, num_decode_tokens, num_prefill_tokens = (
            split_decodes_and_prefills(
                common_attn_metadata,
                decode_threshold=self.reorder_batch_threshold,
                require_uniform=True,
            )
        )

        page_size = self.page_size
        max_seq_len = common_attn_metadata.max_seq_len
        seq_lens = common_attn_metadata.seq_lens
        block_table_tensor = common_attn_metadata.block_table_tensor
        qo_indptr = common_attn_metadata.query_start_loc
        qo_indptr_cpu = common_attn_metadata.query_start_loc_cpu

        # Step 1: Decide which dispatch modes to use:
        # - Cascade attention (distinct mode)
        # - Prefill (FI native or TRTLLM)
        # - Decode (FI native or TRTLLM)
        use_cascade = common_prefix_len > 0
        uses_spec_reorder = self.reorder_batch_threshold > 1
        prefill_use_trtllm = use_trtllm_attention(
            self.num_qo_heads,
            self.num_kv_heads,
            num_prefill_tokens,
            max_seq_len,
            self.dcp_world_size,
            self.cache_dtype,
            self.q_data_type,
            is_prefill=True,
            force_use_trtllm=self.attention_config.use_trtllm_attention,
            has_sinks=self.has_sinks,
            has_spec=uses_spec_reorder,
        )
        decode_use_trtllm = (
            self.use_trtllm_decode_attention and self.dcp_world_size <= 1
        )

        all_uses_trtllm = (num_prefills == 0 or prefill_use_trtllm) and (
            num_decodes == 0 or decode_use_trtllm
        )

    # ... omitted for brevity ...
                )
                fast_plan_decode(
                    decode_wrapper,
                    indptr_cpu=self.paged_kv_indptr.cpu[: num_input_tokens + 1],
                    indices=paged_kv_indices,
                    last_page_len_cpu=self.paged_kv_last_page_len.cpu[
                        :num_input_tokens
                    ],
                    num_qo_heads=self.num_qo_heads * self.dcp_world_size,
                    num_kv_heads=self.num_kv_heads,
                    head_dim=self.head_dim,
                    page_size=self.page_size,
                    # Disable flashinfer's pos encoding and use vllm's rope.
                    pos_encoding_mode="NONE",
                    sm_scale=self.sm_scale,
                    window_left=self.window_left,
                    logits_soft_cap=self.logits_soft_cap,
                    q_data_type=self.q_data_type,
                    kv_data_type=self.kv_cache_dtype,
                    o_data_type=o_dtype,
                    fixed_split_size=self.decode_fixed_split_size,
                    disable_split_kv=self.disable_split_kv,
                )
                attn_metadata.decode = FIDecode(wrapper=decode_wrapper)
        return attn_metadata
```
**EN:** This method builds derived structures within `FlashInferMetadataBuilder`. Key calls include `split_decodes_and_prefills`, `use_trtllm_attention`, `FlashInferMetadata`, `numpy`, `get_dcp_local_seq_lens`, `_compute_flashinfer_kv_metadata`. It touches state such as `q_data_type`. The control flow contains 18 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会构建派生结构，其作用域位于`FlashInferMetadataBuilder`。 关键调用包括 `split_decodes_and_prefills`, `use_trtllm_attention`, `FlashInferMetadata`, `numpy`, `get_dcp_local_seq_lens`, `_compute_flashinfer_kv_metadata`。 它会读写 `q_data_type` 等状态。 控制流包含 18 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `FlashInferImpl` class / `FlashInferImpl` 类
```python
class FlashInferImpl(AttentionImpl):
    can_return_lse_for_decode: bool = True
```
**EN:** Introduces the `FlashInferImpl` class on top of `AttentionImpl`. Core methods include `__init__`, `fused_output_quant_supported`, `process_weights_after_loading`, `forward`, `do_kv_cache_update`.
**CN:** 这里定义 `FlashInferImpl` 类，其基类包括 `AttentionImpl`。核心方法包括 `__init__`, `fused_output_quant_supported`, `process_weights_after_loading`, `forward`, `do_kv_cache_update`。

### `FlashInferImpl.__init__` method / `FlashInferImpl.__init__` 方法
```python
    def __init__(
        self,
        num_heads: int,
        head_size: int,
        scale: float,
        num_kv_heads: int,
        alibi_slopes: list[float] | None,
        sliding_window: int | None,
        kv_cache_dtype: str,
        logits_soft_cap: float | None = None,
        attn_type: AttentionType = AttentionType.DECODER,
        kv_sharing_target_layer_name: int | None = None,
        sinks: torch.Tensor | None = None,
    ) -> None:
        self.num_heads = num_heads
        self.head_size = head_size
        self.scale = float(scale)
        self.num_kv_heads = num_kv_heads
        if alibi_slopes is not None:
            alibi_slopes = torch.tensor(alibi_slopes, dtype=torch.float32)
        self.alibi_slopes = alibi_slopes
        if sliding_window is None:
            self.sliding_window = (-1, -1)
        else:
            self.sliding_window = (sliding_window - 1, 0)
        self.window_left = (
            self.sliding_window[0] if self.sliding_window is not None else -1
        )
        self.kv_cache_dtype = kv_cache_dtype
        self.is_kvcache_nvfp4 = kv_cache_dtype == "nvfp4"
        self.fp4_data_dim = head_size // 2 if self.is_kvcache_nvfp4 else 0
        self.logits_soft_cap = logits_soft_cap
        self.kv_sharing_target_layer_name = kv_sharing_target_layer_name

        self.num_queries_per_kv = self.num_heads // self.num_kv_heads

        if attn_type != AttentionType.DECODER:
            raise NotImplementedError(
                "Encoder self-attention and "
                "encoder/decoder cross-attention "
                "are not implemented for "
                "FlashInferImpl"
            )

        self.sinks: torch.Tensor | None = None
        if sinks is not None:
            if sinks.shape[0] != num_heads:
                raise ValueError(
                    "Sinks must have the same number of heads as the number of "
                    f"heads in the layer. Expected {num_heads}, but got "
                    f"{sinks.shape[0]}."
                )
            self.sinks = sinks

        self.support_trtllm_attn = can_use_trtllm_attention(num_heads, num_kv_heads)
        vllm_config = get_current_vllm_config_or_none()
        self.supports_quant_query_input = (
            self.support_trtllm_attn
            and vllm_config is not None
            and not vllm_config.attention_config.disable_flashinfer_q_quantization
        )
        self.bmm1_scale: float | None = None
        self.bmm2_scale: float | None = None
        self.o_sf_scale: float | None = None

        # Pre-allocated FP8 output buffer for NVFP4 without fused output quant.
        if self.is_kvcache_nvfp4 and vllm_config is not None:
            max_num_tokens = vllm_config.scheduler_config.max_num_batched_tokens
            self._nvfp4_fp8_out = torch.empty(
                (max_num_tokens, num_heads, head_size),
                dtype=FP8_DTYPE,
                device="cuda",
            )
        else:
            self._nvfp4_fp8_out = None

        dcp_a2a = (
            vllm_config is not None
            and vllm_config.parallel_config.decode_context_parallel_size > 1
            and vllm_config.parallel_config.dcp_comm_backend == "a2a"
        )
        if dcp_a2a:
            self.dcp_combine = partial(dcp_a2a_lse_reduce, is_lse_base_on_e=False)
        else:
            self.dcp_combine = partial(cp_lse_ag_out_rs, is_lse_base_on_e=False)
```
**EN:** This method initializes the object state within `FlashInferImpl`. Key calls include `float`, `can_use_trtllm_attention`, `get_current_vllm_config_or_none`, `tensor`, `NotImplementedError`, `empty`. It touches state such as `num_heads`, `head_size`, `scale`, `num_kv_heads`, `alibi_slopes`, `window_left`, `kv_cache_dtype`, `is_kvcache_nvfp4`. The control flow contains 9 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`FlashInferImpl`。 关键调用包括 `float`, `can_use_trtllm_attention`, `get_current_vllm_config_or_none`, `tensor`, `NotImplementedError`, `empty`。 它会读写 `num_heads`, `head_size`, `scale`, `num_kv_heads`, `alibi_slopes`, `window_left`, `kv_cache_dtype`, `is_kvcache_nvfp4` 等状态。 控制流包含 9 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FlashInferImpl.process_weights_after_loading` method / `FlashInferImpl.process_weights_after_loading` 方法
```python
    def process_weights_after_loading(self, act_dtype: torch.dtype):
        if self.sinks is not None and self.sinks.dtype != torch.float32:
            self.sinks = self.sinks.to(torch.float32)
```
**EN:** This method implements `process_weights_after_loading` within `FlashInferImpl`. Key calls include `to`. It touches state such as `sinks`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `process_weights_after_loading`，其作用域位于`FlashInferImpl`。 关键调用包括 `to`。 它会读写 `sinks` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FlashInferImpl.forward` method / `FlashInferImpl.forward` 方法
```python
    def forward(
        self,
        layer: torch.nn.Module,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        kv_cache: torch.Tensor,
        attn_metadata: FlashInferMetadata,
        output: torch.Tensor,
        output_scale: torch.Tensor | None = None,
        output_block_scale: torch.Tensor | None = None,
    ) -> torch.Tensor:
        """Forward pass with FlashInfer.

        Args:
            query: shape = [num_tokens, num_heads, head_size]
            key: shape = [num_tokens, num_kv_heads, head_size]
            value: shape = [num_tokens, num_kv_heads, head_size]
            kv_cache: KV cache tensor with different possible shapes:
                - NHD: [num_blocks, 2, block_size, num_kv_heads, head_size]
                - HND: [num_blocks, 2, num_kv_heads, block_size, head_size]
            attn_metadata: Metadata for attention.
        Returns:
            shape = [num_tokens, num_heads * head_size]
        """
        if attn_metadata is None:
            # Profiling run.
            return output.fill_(0)

        # Ensure query dtype matches the expected dtype from attention metadata
        assert attn_metadata.q_data_type == query.dtype, (
            f"Query dtype mismatch: expected {attn_metadata.q_data_type}, "
            f"got {query.dtype}"
        )

        if self.bmm1_scale is None:
            self.bmm1_scale = self.scale
            if is_quantized_kv_cache(self.kv_cache_dtype):
                self.bmm1_scale *= layer._q_scale_float * layer._k_scale_float

        if self.bmm2_scale is None:
            self.bmm2_scale = 1.0
            if is_quantized_kv_cache(self.kv_cache_dtype):
                self.bmm2_scale *= layer._v_scale_float

        prefill_use_trtllm = isinstance(attn_metadata.prefill, TRTLLMPrefill)
        decode_use_trtllm = isinstance(attn_metadata.decode, TRTLLMDecode)

        # The attn+quant fusion happens when output_scale is provided.
        if output_scale is None:
    # ... omitted for brevity ...

                trtllm_batch_decode_with_kv_cache(
                    query=decode_query,
                    kv_cache=(
                        nvfp4_kv_data if self.is_kvcache_nvfp4 else kv_cache_permute
                    ),
                    workspace_buffer=workspace_buffer,
                    block_tables=block_tables_decode,
                    seq_lens=seq_lens_decode,
                    max_seq_len=attn_metadata.decode.max_seq_len,
                    bmm1_scale=self.bmm1_scale,
                    bmm2_scale=self.bmm2_scale,
                    window_left=self.window_left,
                    sinks=self.sinks,
                    o_sf_scale=self.o_sf_scale,
                    out=out,
                    q_len_per_req=q_len_per_req,
                    kv_cache_sf=(
                        nvfp4_kv_block_scales if self.is_kvcache_nvfp4 else None
                    ),
                )

                if needs_fp8_out:
                    output[:num_decode_tokens].copy_(out.to(output.dtype))
        return output_padded
```
**EN:** This method drives the forward-pass computation within `FlashInferImpl`. The docstring frames it as: Forward pass with FlashInfer. Key calls include `isinstance`, `get_kv_cache_stride_order`, `permute`, `canonicalize_singleton_dim_strides`, `fill_`, `is_quantized_kv_cache`. It touches state such as `bmm1_scale`, `bmm2_scale`, `o_sf_scale`. The control flow contains 40 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该方法会驱动前向计算流程，其作用域位于`FlashInferImpl`。 关键调用包括 `isinstance`, `get_kv_cache_stride_order`, `permute`, `canonicalize_singleton_dim_strides`, `fill_`, `is_quantized_kv_cache`。 它会读写 `bmm1_scale`, `bmm2_scale`, `o_sf_scale` 等状态。 控制流包含 40 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `FlashInferImpl.do_kv_cache_update` method / `FlashInferImpl.do_kv_cache_update` 方法
```python
    def do_kv_cache_update(
        self,
        layer: torch.nn.Module,
        key: torch.Tensor,
        value: torch.Tensor,
        kv_cache: torch.Tensor,
        slot_mapping: torch.Tensor,
    ) -> None:
        if self.kv_sharing_target_layer_name is None:
            # Reshape the input keys and values and store them in the cache.
            # Skip this if sharing KV cache with an earlier attention layer.
            # NOTE(woosuk): Here, key and value are padded while slot_mapping is
            # not padded. However, we don't need to do key[:num_actual_tokens]
            # and value[:num_actual_tokens] because the reshape_and_cache_flash
            # op uses the slot_mapping's shape to determine the number of
            # actual tokens.
            k_cache = kv_cache[:, 0]
            v_cache = kv_cache[:, 1]
            torch.ops._C_cache_ops.reshape_and_cache_flash(
                key,
                value,
                k_cache,
                v_cache,
                slot_mapping,
                self.kv_cache_dtype,
                layer._k_scale,
                layer._v_scale,
            )
```
**EN:** This method implements `do_kv_cache_update` within `FlashInferImpl`. Key calls include `reshape_and_cache_flash`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `do_kv_cache_update`，其作用域位于`FlashInferImpl`。 关键调用包括 `reshape_and_cache_flash`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `fast_plan_decode` function / `fast_plan_decode` 函数
```python
def fast_plan_decode(
    self,  # decode wrapper
    indptr_cpu: torch.Tensor,
    indices: torch.Tensor,
    last_page_len_cpu: torch.Tensor,
    num_qo_heads: int,
    num_kv_heads: int,
    head_dim: int,
    page_size: int,
    pos_encoding_mode: str = "NONE",
    window_left: int = -1,
    logits_soft_cap: float | None = None,
    q_data_type: str | torch.dtype | None = "float16",
    kv_data_type: str | torch.dtype | None = None,
    o_data_type: str | torch.dtype | None = None,
    data_type: str | torch.dtype | None = None,
    sm_scale: float | None = None,
    rope_scale: float | None = None,
    rope_theta: float | None = None,
    non_blocking: bool = True,
    fixed_split_size: int = -1,
    disable_split_kv: bool = False,
) -> None:
    """
    A faster version of BatchDecodeWithPagedKVCacheWrapper::plan used for
    cudagraph capture/replay, while the no cudagraph version turns back
    to the original plan.
    using original plan after passing host-side buffers:
    - only host-to-device copy of indptr and last_page_len buffers
    Modifications for cudagraph:
    - only host-to-device copy of indptr and last_page_len buffers.
    - avoid device-to-device copy of indices buffer.

    Part of the code get inspiration from the original plan from FlashInfer repo
    and the implementation of fast_decode_plan for FlashInfer in SGlang repo.
    """
    # Warm up with the original plan if it is first call, and always run the
    # original plan if we run for dynamic shape. For fixed shape (cudagraph),
    # this warm up is to generate the _cached_module for the decode wrapper.
    if not self.is_cuda_graph_enabled or getattr(self, "vllm_first_call", True):
        self.plan(
            indptr=indptr_cpu,
            indices=indices,
            last_page_len=last_page_len_cpu,
            num_qo_heads=num_qo_heads,
            num_kv_heads=num_kv_heads,
            head_dim=head_dim,
            page_size=page_size,
            pos_encoding_mode=pos_encoding_mode,
            window_left=window_left,
# ... omitted for brevity ...

    assert self.is_cuda_graph_enabled, "Should be cudagraph only here"

    fast_decode_plan(
        self,
        indptr=indptr_cpu,
        indices=indices,
        last_page_len=last_page_len_cpu,
        num_qo_heads=num_qo_heads,
        num_kv_heads=num_kv_heads,
        head_dim=head_dim,
        page_size=page_size,
        pos_encoding_mode=pos_encoding_mode,
        window_left=window_left,
        logits_soft_cap=logits_soft_cap,
        q_data_type=q_data_type,
        kv_data_type=kv_data_type,
        data_type=data_type,
        sm_scale=sm_scale,
        rope_scale=rope_scale,
        rope_theta=rope_theta,
        non_blocking=non_blocking,
        fixed_split_size=fixed_split_size,
        disable_split_kv=disable_split_kv,
    )
```
**EN:** This function implements `fast_plan_decode` within the module. The docstring frames it as: A faster version of BatchDecodeWithPagedKVCacheWrapper::plan used for cudagraph capture/replay, while the no cudagraph version turns back to the original plan. Key calls include `fast_decode_plan`, `getattr`, `plan`. It touches state such as `vllm_first_call`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic. The excerpt highlights the most representative entry and exit logic because the full block is large.
**CN:** 该函数会实现 `fast_plan_decode`，其作用域位于the module。 关键调用包括 `fast_decode_plan`, `getattr`, `plan`。 它会读写 `vllm_first_call` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。 由于代码块较大，这里展示的是最能代表入口与出口行为的片段。

### `_copy_page_indices_kernel` function / `_copy_page_indices_kernel` 函数
```python
@triton.jit
def _copy_page_indices_kernel(
    page_indices,
    block_table,
    block_table_stride,
    cu_num_blocks,
    BLOCK_SIZE: tl.constexpr,
):
    req_idx = tl.program_id(0)
    row_ptr = block_table + req_idx * block_table_stride
    start_idx = tl.load(cu_num_blocks + req_idx)
    end_idx = tl.load(cu_num_blocks + req_idx + 1)
    num_blocks = end_idx - start_idx

    offset = tl.arange(0, BLOCK_SIZE)
    for i in tl.range(0, num_blocks, BLOCK_SIZE):
        block_ids = tl.load(row_ptr + i + offset, mask=i + offset < num_blocks)
        tl.store(
            page_indices + start_idx + i + offset,
            block_ids,
            mask=i + offset < num_blocks,
        )
```
**EN:** This function implements `_copy_page_indices_kernel` within the module. Key calls include `program_id`, `load`, `arange`, `range`, `store`. The control flow contains 0 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_copy_page_indices_kernel`，其作用域位于the module。 关键调用包括 `program_id`, `load`, `arange`, `range`, `store`。 控制流包含 0 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `_get_trtllm_gen_workspace_buffer`: top-level helper or orchestration entry point. / `_get_trtllm_gen_workspace_buffer`：顶层辅助函数或编排入口。
- `_trtllm_prefill_attn_kvfp8_dequant`: top-level helper or orchestration entry point. / `_trtllm_prefill_attn_kvfp8_dequant`：顶层辅助函数或编排入口。
- `trtllm_prefill_attn_kvfp8_dequant`: top-level helper or orchestration entry point. / `trtllm_prefill_attn_kvfp8_dequant`：顶层辅助函数或编排入口。
- `BatchDCPPrefillWrapper`: central class or interface in this module. / `BatchDCPPrefillWrapper`：本模块中的核心类或接口。
- `FlashInferBackend`: central class or interface in this module. / `FlashInferBackend`：本模块中的核心类或接口。
- `FIPrefill`: central class or interface in this module. / `FIPrefill`：本模块中的核心类或接口。
- `FIDecode`: central class or interface in this module. / `FIDecode`：本模块中的核心类或接口。
- `TRTLLMPrefill`: central class or interface in this module. / `TRTLLMPrefill`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`, `functools`, `typing`
- External / 外部依赖: `numpy`, `torch`, `flashinfer`, `typing_extensions`
- Internal vLLM / 内部依赖: `vllm`, `vllm.config`, `vllm.config.cache`, `vllm.distributed.parallel_state`, `vllm.logger`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.platforms`, `vllm.platforms.interface`, `vllm.triton_utils`, `vllm.utils.flashinfer`, `vllm.utils.math_utils`, `vllm.utils.platform_utils`
