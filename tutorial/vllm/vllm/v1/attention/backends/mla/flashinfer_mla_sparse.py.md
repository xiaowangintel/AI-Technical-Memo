# flashinfer_mla_sparse.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/mla/flashinfer_mla_sparse.py`
- **Repository**: vllm-project/vllm
- **Purpose**: FlashInfer MLA Sparse Attention Backend. / 该模块位于 `attention/backends/mla` 子系统，主要围绕 `FlashInferMLASparseBackend`, `FlashInferMLASparseMetadata`, `FlashInferMLASparseMetadataBuilder` 组织实现。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
"""FlashInfer MLA Sparse Attention Backend.

This backend uses the FlashInfer TRT-LLM MLA kernel with sparse_mla_top_k
for models like DeepSeek-V3.2 that use index-based sparse attention.

For sparse MLA:
- block_tables shape changes from [batch_size, max_num_blocks] (dense)
  to [batch_size, q_len_per_request, sparse_mla_top_k] (sparse)
- The sparse indices represent physical cache slot positions to attend to
- sparse_mla_top_k parameter must be set to the topk value
"""

from dataclasses import dataclass
from typing import TYPE_CHECKING, ClassVar

import numpy as np
import torch
from flashinfer.decode import trtllm_batch_decode_with_kv_cache_mla

from vllm.config import VllmConfig
from vllm.config.cache import CacheDType
from vllm.logger import init_logger
from vllm.model_executor.layers.attention.mla_attention import (
    get_mla_dims,
)
from vllm.platforms.interface import DeviceCapability
from vllm.utils.torch_utils import is_quantized_kv_cache
from vllm.v1.attention.backend import (
    AttentionBackend,
    AttentionCGSupport,
    AttentionLayer,
    AttentionMetadata,
    AttentionMetadataBuilder,
    AttentionType,
    CommonAttentionMetadata,
    MultipleOf,
    SparseMLAAttentionImpl,
)
from vllm.v1.attention.backends.mla.sparse_utils import (
    triton_convert_req_index_to_global_index,
)
from vllm.v1.attention.backends.utils import KVCacheLayoutType
from vllm.v1.kv_cache_interface import AttentionSpec

if TYPE_CHECKING:
    from vllm.model_executor.models.deepseek_v2 import Indexer

logger = init_logger(__name__)

FLASHINFER_MLA_SPARSE_WORKSPACE_BUFFER_SIZE = 128 * 1024 * 1024
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`, `FLASHINFER_MLA_SPARSE_WORKSPACE_BUFFER_SIZE`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`, `FLASHINFER_MLA_SPARSE_WORKSPACE_BUFFER_SIZE`。

### `FlashInferMLASparseBackend` class / `FlashInferMLASparseBackend` 类
```python
class FlashInferMLASparseBackend(AttentionBackend):
    """FlashInfer MLA backend with sparse attention support.

    This backend uses the FlashInfer TRT-LLM MLA kernel with sparse_mla_top_k
    for models like DeepSeek-V3.2 that use index-based sparse attention.
    """

    supported_dtypes: ClassVar[list[torch.dtype]] = [torch.float16, torch.bfloat16]
    supported_kv_cache_dtypes: ClassVar[list[CacheDType]] = [
        "auto",
        "float16",
        "bfloat16",
        "fp8",
        "fp8_e4m3",
    ]
```
**EN:** Introduces the `FlashInferMLASparseBackend` class on top of `AttentionBackend`. Core methods include `get_supported_kernel_block_sizes`, `get_name`, `get_impl_cls`, `get_builder_cls`, `get_supported_head_sizes`, `is_mla`. Docstring signal: FlashInfer MLA backend with sparse attention support.
**CN:** 这里定义 `FlashInferMLASparseBackend` 类，其基类包括 `AttentionBackend`。核心方法包括 `get_supported_kernel_block_sizes`, `get_name`, `get_impl_cls`, `get_builder_cls`, `get_supported_head_sizes`, `is_mla`。

### `FlashInferMLASparseBackend.get_supported_kernel_block_sizes` method / `FlashInferMLASparseBackend.get_supported_kernel_block_sizes` 方法
```python
    @staticmethod
    def get_supported_kernel_block_sizes() -> list[int | MultipleOf]:
        return [32, 64]
```
**EN:** This method returns or derives a value within `FlashInferMLASparseBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`FlashInferMLASparseBackend`。

### `FlashInferMLASparseBackend.get_name` method / `FlashInferMLASparseBackend.get_name` 方法
```python
    @staticmethod
    def get_name() -> str:
        return "FLASHINFER_MLA_SPARSE"
```
**EN:** This method returns or derives a value within `FlashInferMLASparseBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`FlashInferMLASparseBackend`。

### `FlashInferMLASparseBackend.get_impl_cls` method / `FlashInferMLASparseBackend.get_impl_cls` 方法
```python
    @staticmethod
    def get_impl_cls() -> type["FlashInferMLASparseImpl"]:
        return FlashInferMLASparseImpl
```
**EN:** This method returns or derives a value within `FlashInferMLASparseBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`FlashInferMLASparseBackend`。

### `FlashInferMLASparseBackend.get_builder_cls` method / `FlashInferMLASparseBackend.get_builder_cls` 方法
```python
    @staticmethod
    def get_builder_cls() -> type["FlashInferMLASparseMetadataBuilder"]:
        return FlashInferMLASparseMetadataBuilder
```
**EN:** This method returns or derives a value within `FlashInferMLASparseBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`FlashInferMLASparseBackend`。

### `FlashInferMLASparseBackend.get_supported_head_sizes` method / `FlashInferMLASparseBackend.get_supported_head_sizes` 方法
```python
    @classmethod
    def get_supported_head_sizes(cls) -> list[int]:
        return [576]
```
**EN:** This method returns or derives a value within `FlashInferMLASparseBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`FlashInferMLASparseBackend`。

### `FlashInferMLASparseBackend.supports_compute_capability` method / `FlashInferMLASparseBackend.supports_compute_capability` 方法
```python
    @classmethod
    def supports_compute_capability(cls, capability: DeviceCapability) -> bool:
        # FlashInfer sparse MLA targets Blackwell (SM 10.x)
        return capability.major == 10
```
**EN:** This method implements `supports_compute_capability` within `FlashInferMLASparseBackend`.
**CN:** 该方法会实现 `supports_compute_capability`，其作用域位于`FlashInferMLASparseBackend`。

### `FlashInferMLASparseBackend.supports_combination` method / `FlashInferMLASparseBackend.supports_combination` 方法
```python
    @classmethod
    def supports_combination(
        cls,
        head_size: int,
        dtype: torch.dtype,
        kv_cache_dtype: CacheDType | None,
        block_size: int | None,
        use_mla: bool,
        has_sink: bool,
        use_sparse: bool,
        device_capability: DeviceCapability,
    ) -> str | None:
        # FlashInfer MLA sparse kernel requires qk_nope_head_dim in [128, 192]
        from vllm.config import get_current_vllm_config

        vllm_config = get_current_vllm_config()
        if vllm_config.model_config is not None:
            hf_text_config = vllm_config.model_config.hf_text_config
            qk_nope_head_dim = getattr(hf_text_config, "qk_nope_head_dim", 1)
            if qk_nope_head_dim not in [128, 192]:
                return (
                    "FlashInfer MLA Sparse kernel requires qk_nope_head_dim "
                    f"in [128, 192], but got {qk_nope_head_dim}"
                )
            # Check for index_topk which indicates sparse model
            if not hasattr(hf_text_config, "index_topk"):
                return "FlashInfer MLA Sparse requires model with index_topk config"
        return None
```
**EN:** This method implements `supports_combination` within `FlashInferMLASparseBackend`. Key calls include `get_current_vllm_config`, `getattr`, `hasattr`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `supports_combination`，其作用域位于`FlashInferMLASparseBackend`。 关键调用包括 `get_current_vllm_config`, `getattr`, `hasattr`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FlashInferMLASparseBackend.get_kv_cache_shape` method / `FlashInferMLASparseBackend.get_kv_cache_shape` 方法
```python
    @staticmethod
    def get_kv_cache_shape(
        num_blocks: int,
        block_size: int,
        num_kv_heads: int,  # assumed to be 1 for MLA
        head_size: int,
        cache_dtype_str: str = "auto",
    ) -> tuple[int, ...]:
        return (num_blocks, block_size, head_size)
```
**EN:** This method returns or derives a value within `FlashInferMLASparseBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`FlashInferMLASparseBackend`。

### `FlashInferMLASparseMetadata` class / `FlashInferMLASparseMetadata` 类
```python
@dataclass
class FlashInferMLASparseMetadata(AttentionMetadata):
    """Attention metadata for FlashInfer MLA Sparse backend."""

    num_reqs: int
    max_query_len: int
    max_seq_len: int
    num_actual_tokens: int

    # Query start locations
    query_start_loc: torch.Tensor
    slot_mapping: torch.Tensor
    block_table: torch.Tensor
    req_id_per_token: torch.Tensor

    # Sequence lengths for all requests (context + query)
    seq_lens: torch.Tensor

    # Sparse-specific
    block_size: int = 64
    topk_tokens: int = 2048
```
**EN:** Uses `@dataclass` to package related state for `FlashInferMLASparseMetadata`. Typical fields include `num_reqs`, `max_query_len`, `max_seq_len`, `num_actual_tokens`, `query_start_loc`, `slot_mapping`.
**CN:** `FlashInferMLASparseMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `num_reqs`, `max_query_len`, `max_seq_len`, `num_actual_tokens`, `query_start_loc`, `slot_mapping`。

### `FlashInferMLASparseMetadataBuilder` class / `FlashInferMLASparseMetadataBuilder` 类
```python
class FlashInferMLASparseMetadataBuilder(
    AttentionMetadataBuilder[FlashInferMLASparseMetadata]
):
    """Builder for FlashInfer MLA Sparse attention metadata."""

    _cudagraph_support: ClassVar[AttentionCGSupport] = AttentionCGSupport.UNIFORM_BATCH
```
**EN:** Introduces the `FlashInferMLASparseMetadataBuilder` class on top of `AttentionMetadataBuilder[FlashInferMLASparseMetadata]`. Core methods include `__init__`, `build`. Docstring signal: Builder for FlashInfer MLA Sparse attention metadata.
**CN:** 这里定义 `FlashInferMLASparseMetadataBuilder` 类，其基类包括 `AttentionMetadataBuilder[FlashInferMLASparseMetadata]`。核心方法包括 `__init__`, `build`。

### `FlashInferMLASparseMetadataBuilder.__init__` method / `FlashInferMLASparseMetadataBuilder.__init__` 方法
```python
    def __init__(
        self,
        kv_cache_spec: AttentionSpec,
        layer_names: list[str],
        vllm_config: VllmConfig,
        device: torch.device,
    ) -> None:
        self.vllm_config = vllm_config
        self.layer_names = layer_names
        self.kv_cache_spec = kv_cache_spec
        self.model_config = vllm_config.model_config
        self.device = device

        self.mla_dims = get_mla_dims(self.model_config)
        self.topk_tokens = vllm_config.model_config.hf_config.index_topk

        self.req_id_per_token_buffer = torch.empty(
            (vllm_config.scheduler_config.max_num_batched_tokens,),
            dtype=torch.int32,
            device=device,
        )
```
**EN:** This method initializes the object state within `FlashInferMLASparseMetadataBuilder`. Key calls include `get_mla_dims`, `empty`. It touches state such as `vllm_config`, `layer_names`, `kv_cache_spec`, `model_config`, `device`, `mla_dims`, `topk_tokens`, `req_id_per_token_buffer`.
**CN:** 该方法会初始化对象状态，其作用域位于`FlashInferMLASparseMetadataBuilder`。 关键调用包括 `get_mla_dims`, `empty`。 它会读写 `vllm_config`, `layer_names`, `kv_cache_spec`, `model_config`, `device`, `mla_dims`, `topk_tokens`, `req_id_per_token_buffer` 等状态。

### `FlashInferMLASparseMetadataBuilder.build` method / `FlashInferMLASparseMetadataBuilder.build` 方法
```python
    def build(
        self,
        common_prefix_len: int,
        common_attn_metadata: CommonAttentionMetadata,
        fast_build: bool = False,
    ) -> FlashInferMLASparseMetadata:
        cm = common_attn_metadata
        num_tokens = cm.num_actual_tokens

        # Build req_id_per_token mapping
        starts = np.asarray(cm.query_start_loc_cpu, dtype=np.int32)
        seg_lengths = np.diff(starts)
        req_id_per_token = np.repeat(
            np.arange(seg_lengths.shape[0], dtype=np.int32), seg_lengths
        )

        # Zero-fill for cudagraphs
        self.req_id_per_token_buffer.fill_(0)
        self.req_id_per_token_buffer[: req_id_per_token.shape[0]].copy_(
            torch.from_numpy(req_id_per_token), non_blocking=True
        )
        req_id_per_token_tensor = self.req_id_per_token_buffer[:num_tokens]

        return FlashInferMLASparseMetadata(
            num_reqs=cm.num_reqs,
            max_query_len=cm.max_query_len,
            max_seq_len=cm.max_seq_len,
            num_actual_tokens=cm.num_actual_tokens,
            query_start_loc=cm.query_start_loc,
            slot_mapping=cm.slot_mapping,
            block_table=cm.block_table_tensor,
            req_id_per_token=req_id_per_token_tensor,
            seq_lens=cm.seq_lens,
            block_size=self.kv_cache_spec.block_size,
            topk_tokens=self.topk_tokens,
        )
```
**EN:** This method builds derived structures within `FlashInferMLASparseMetadataBuilder`. Key calls include `asarray`, `diff`, `repeat`, `fill_`, `copy_`, `FlashInferMLASparseMetadata`.
**CN:** 该方法会构建派生结构，其作用域位于`FlashInferMLASparseMetadataBuilder`。 关键调用包括 `asarray`, `diff`, `repeat`, `fill_`, `copy_`, `FlashInferMLASparseMetadata`。

### Module constants / 模块常量
```python
_fi_sparse_workspace: torch.Tensor | None = None
```
**EN:** Defines module-level constants or aliases such as `_fi_sparse_workspace`, which are reused by later definitions.
**CN:** 定义 `_fi_sparse_workspace` 等模块级常量或别名，供后续定义复用。

### `_get_workspace_buffer` function / `_get_workspace_buffer` 函数
```python
def _get_workspace_buffer(device: torch.device) -> torch.Tensor:
    global _fi_sparse_workspace
    if _fi_sparse_workspace is None:
        _fi_sparse_workspace = torch.zeros(
            FLASHINFER_MLA_SPARSE_WORKSPACE_BUFFER_SIZE,
            dtype=torch.uint8,
            device=device,
        )
    return _fi_sparse_workspace
```
**EN:** This function implements `_get_workspace_buffer` within the module. Key calls include `zeros`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_get_workspace_buffer`，其作用域位于the module。 关键调用包括 `zeros`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FlashInferMLASparseImpl` class / `FlashInferMLASparseImpl` 类
```python
class FlashInferMLASparseImpl(SparseMLAAttentionImpl[FlashInferMLASparseMetadata]):
    """FlashInfer MLA Sparse implementation.

    Uses the TRT-LLM MLA kernel with sparse_mla_top_k parameter for
    sparse attention computation.
    """
```
**EN:** Introduces the `FlashInferMLASparseImpl` class on top of `SparseMLAAttentionImpl[FlashInferMLASparseMetadata]`. Core methods include `__init__`, `forward_mqa`. Docstring signal: FlashInfer MLA Sparse implementation.
**CN:** 这里定义 `FlashInferMLASparseImpl` 类，其基类包括 `SparseMLAAttentionImpl[FlashInferMLASparseMetadata]`。核心方法包括 `__init__`, `forward_mqa`。

### `FlashInferMLASparseImpl.__init__` method / `FlashInferMLASparseImpl.__init__` 方法
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
        logits_soft_cap: float | None,
        attn_type: str,
        kv_sharing_target_layer_name: str | None,
        # MLA Specific Arguments
        topk_indice_buffer: torch.Tensor | None = None,
        indexer: "Indexer | None" = None,
        **mla_args,
    ) -> None:
        unsupported_features = [alibi_slopes, sliding_window, logits_soft_cap]
        if any(unsupported_features):
            raise NotImplementedError(
                "FlashInferMLASparseImpl does not support one of the following: "
                "alibi_slopes, sliding_window, logits_soft_cap"
            )

        if attn_type != AttentionType.DECODER:
            raise NotImplementedError(
                "Encoder self-attention and "
                "encoder/decoder cross-attention "
                "are not implemented for "
                "FlashInferMLASparseImpl"
            )

        self.num_heads = num_heads
        self.head_size = head_size
        self.scale = float(scale)
        self.num_kv_heads = num_kv_heads
        self.kv_cache_dtype = kv_cache_dtype

        # MLA-specific dimensions
        self.kv_lora_rank: int = mla_args["kv_lora_rank"]
        self.qk_nope_head_dim: int = mla_args["qk_nope_head_dim"]
        self.qk_rope_head_dim: int = mla_args["qk_rope_head_dim"]

        assert indexer is not None, "Indexer required for sparse MLA"
        self.topk_indices_buffer: torch.Tensor | None = indexer.topk_indices_buffer

        self._workspace_buffer: torch.Tensor | None = None
        self.bmm1_scale: float | None = None
        self.bmm2_scale: float | None = None

        # fp8 query quantization is required when using fp8 kv_cache,
        # as the TRTLLM-GEN sparse MLA kernel requires matching dtypes
        # for query and kv_cache (mixed bf16+fp8 is not supported).
        self.supports_quant_query_input = True
```
**EN:** This method initializes the object state within `FlashInferMLASparseImpl`. Key calls include `any`, `float`, `NotImplementedError`. It touches state such as `num_heads`, `head_size`, `scale`, `num_kv_heads`, `kv_cache_dtype`, `kv_lora_rank`, `qk_nope_head_dim`, `qk_rope_head_dim`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`FlashInferMLASparseImpl`。 关键调用包括 `any`, `float`, `NotImplementedError`。 它会读写 `num_heads`, `head_size`, `scale`, `num_kv_heads`, `kv_cache_dtype`, `kv_lora_rank`, `qk_nope_head_dim`, `qk_rope_head_dim` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `FlashInferMLASparseImpl.forward_mqa` method / `FlashInferMLASparseImpl.forward_mqa` 方法
```python
    def forward_mqa(
        self,
        q: torch.Tensor | tuple[torch.Tensor, torch.Tensor],
        kv_c_and_k_pe_cache: torch.Tensor,
        attn_metadata: FlashInferMLASparseMetadata,
        layer: AttentionLayer,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        if isinstance(q, tuple):
            q = torch.cat(q, dim=-1)

        num_actual_toks = q.shape[0]

        assert self.topk_indices_buffer is not None
        topk_indices = self.topk_indices_buffer[:num_actual_toks]

        topk_indices_physical, seq_lens = triton_convert_req_index_to_global_index(
            attn_metadata.req_id_per_token[:num_actual_toks],
            attn_metadata.block_table,
            topk_indices,
            BLOCK_SIZE=attn_metadata.block_size,
            NUM_TOPK_TOKENS=topk_indices.shape[1],
            return_valid_counts=True,
        )

        if self._workspace_buffer is None:
            self._workspace_buffer = _get_workspace_buffer(q.device)

        if self.bmm1_scale is None:
            self.bmm1_scale = self.scale
            if is_quantized_kv_cache(self.kv_cache_dtype):
                self.bmm1_scale *= layer._q_scale_float * layer._k_scale_float
        if self.bmm2_scale is None:
            self.bmm2_scale = 1.0
            if is_quantized_kv_cache(self.kv_cache_dtype):
                self.bmm2_scale *= layer._k_scale_float

        o = trtllm_batch_decode_with_kv_cache_mla(
            query=q.unsqueeze(1),
            kv_cache=kv_c_and_k_pe_cache.unsqueeze(1),
            workspace_buffer=self._workspace_buffer,
            qk_nope_head_dim=self.qk_nope_head_dim,
            kv_lora_rank=self.kv_lora_rank,
            qk_rope_head_dim=self.qk_rope_head_dim,
            block_tables=topk_indices_physical.unsqueeze(1),
            seq_lens=seq_lens,
            max_seq_len=attn_metadata.topk_tokens,
            bmm1_scale=self.bmm1_scale,
            bmm2_scale=self.bmm2_scale,
            sparse_mla_top_k=attn_metadata.topk_tokens,
        )
        return o.view(-1, o.shape[-2], o.shape[-1]), None
```
**EN:** This method drives the forward-pass computation within `FlashInferMLASparseImpl`. Key calls include `isinstance`, `triton_convert_req_index_to_global_index`, `trtllm_batch_decode_with_kv_cache_mla`, `cat`, `_get_workspace_buffer`, `is_quantized_kv_cache`. It touches state such as `_workspace_buffer`, `bmm1_scale`, `bmm2_scale`. The control flow contains 6 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会驱动前向计算流程，其作用域位于`FlashInferMLASparseImpl`。 关键调用包括 `isinstance`, `triton_convert_req_index_to_global_index`, `trtllm_batch_decode_with_kv_cache_mla`, `cat`, `_get_workspace_buffer`, `is_quantized_kv_cache`。 它会读写 `_workspace_buffer`, `bmm1_scale`, `bmm2_scale` 等状态。 控制流包含 6 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `FlashInferMLASparseBackend`: central class or interface in this module. / `FlashInferMLASparseBackend`：本模块中的核心类或接口。
- `FlashInferMLASparseMetadata`: central class or interface in this module. / `FlashInferMLASparseMetadata`：本模块中的核心类或接口。
- `FlashInferMLASparseMetadataBuilder`: central class or interface in this module. / `FlashInferMLASparseMetadataBuilder`：本模块中的核心类或接口。
- `_get_workspace_buffer`: top-level helper or orchestration entry point. / `_get_workspace_buffer`：顶层辅助函数或编排入口。
- `FlashInferMLASparseImpl`: central class or interface in this module. / `FlashInferMLASparseImpl`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`, `typing`
- External / 外部依赖: `numpy`, `torch`, `flashinfer`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.config.cache`, `vllm.logger`, `vllm.model_executor.layers.attention.mla_attention`, `vllm.platforms.interface`, `vllm.utils.torch_utils`, `vllm.v1.attention.backend`, `vllm.v1.attention.backends.mla.sparse_utils`, `vllm.v1.attention.backends.utils`, `vllm.v1.kv_cache_interface`, `vllm.model_executor.models.deepseek_v2`
