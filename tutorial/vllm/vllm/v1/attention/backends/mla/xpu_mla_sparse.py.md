# xpu_mla_sparse.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/mla/xpu_mla_sparse.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `XPUMLASparseBackend`, `XPUMLASparseMetadata`, `XPUMLASparseMetadataBuilder` for the V1 `attention/backends/mla` subsystem. / 为 V1 的 `attention/backends/mla` 子系统实现 `XPUMLASparseBackend`, `XPUMLASparseMetadata`, `XPUMLASparseMetadataBuilder`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
from dataclasses import dataclass
from typing import TYPE_CHECKING, ClassVar, Optional

import numpy as np
import torch

from vllm.config import VllmConfig
from vllm.config.cache import CacheDType
from vllm.logger import init_logger
from vllm.model_executor.layers.attention.mla_attention import (
    get_mla_dims,
)
from vllm.utils.torch_utils import is_quantized_kv_cache
from vllm.v1.attention.backend import (
    AttentionBackend,
    AttentionCGSupport,
    AttentionLayer,
    AttentionMetadata,
    AttentionMetadataBuilder,
    CommonAttentionMetadata,
    SparseMLAAttentionImpl,
)
from vllm.v1.attention.backends.mla.flashmla_sparse import (
    triton_convert_req_index_to_global_index,
)
from vllm.v1.attention.ops.xpu_mla_sparse import triton_bf16_mla_sparse_interface
from vllm.v1.kv_cache_interface import AttentionSpec

if TYPE_CHECKING:
    from vllm.model_executor.models.deepseek_v2 import Indexer
logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `XPUMLASparseBackend` class / `XPUMLASparseBackend` 类
```python
class XPUMLASparseBackend(AttentionBackend):
    supported_dtypes: ClassVar[list[torch.dtype]] = [torch.float16, torch.bfloat16]
    supported_kv_cache_dtypes: ClassVar[list[CacheDType]] = [
        "auto",
        "float16",
        "bfloat16",
    ]
```
**EN:** Introduces the `XPUMLASparseBackend` class on top of `AttentionBackend`. Core methods include `get_name`, `get_metadata_cls`, `get_builder_cls`, `get_impl_cls`, `is_mla`, `is_sparse`.
**CN:** 这里定义 `XPUMLASparseBackend` 类，其基类包括 `AttentionBackend`。核心方法包括 `get_name`, `get_metadata_cls`, `get_builder_cls`, `get_impl_cls`, `is_mla`, `is_sparse`。

### `XPUMLASparseBackend.get_name` method / `XPUMLASparseBackend.get_name` 方法
```python
    @staticmethod
    def get_name() -> str:
        return "XPU_MLA_SPARSE"
```
**EN:** This method returns or derives a value within `XPUMLASparseBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`XPUMLASparseBackend`。

### `XPUMLASparseBackend.get_metadata_cls` method / `XPUMLASparseBackend.get_metadata_cls` 方法
```python
    @staticmethod
    def get_metadata_cls() -> type["XPUMLASparseMetadata"]:
        return XPUMLASparseMetadata
```
**EN:** This method returns or derives a value within `XPUMLASparseBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`XPUMLASparseBackend`。

### `XPUMLASparseBackend.get_builder_cls` method / `XPUMLASparseBackend.get_builder_cls` 方法
```python
    @staticmethod
    def get_builder_cls() -> type["XPUMLASparseMetadataBuilder"]:
        return XPUMLASparseMetadataBuilder
```
**EN:** This method returns or derives a value within `XPUMLASparseBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`XPUMLASparseBackend`。

### `XPUMLASparseBackend.get_impl_cls` method / `XPUMLASparseBackend.get_impl_cls` 方法
```python
    @staticmethod
    def get_impl_cls() -> type["XPUMLASparseImpl"]:
        return XPUMLASparseImpl
```
**EN:** This method returns or derives a value within `XPUMLASparseBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`XPUMLASparseBackend`。

### `XPUMLASparseBackend.is_mla` method / `XPUMLASparseBackend.is_mla` 方法
```python
    @classmethod
    def is_mla(cls) -> bool:
        return True
```
**EN:** This method answers a boolean capability check within `XPUMLASparseBackend`.
**CN:** 该方法会回答布尔能力判断，其作用域位于`XPUMLASparseBackend`。

### `XPUMLASparseBackend.is_sparse` method / `XPUMLASparseBackend.is_sparse` 方法
```python
    @classmethod
    def is_sparse(cls) -> bool:
        return True
```
**EN:** This method answers a boolean capability check within `XPUMLASparseBackend`.
**CN:** 该方法会回答布尔能力判断，其作用域位于`XPUMLASparseBackend`。

### `XPUMLASparseBackend.get_kv_cache_shape` method / `XPUMLASparseBackend.get_kv_cache_shape` 方法
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
**EN:** This method returns or derives a value within `XPUMLASparseBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`XPUMLASparseBackend`。

### `XPUMLASparseBackend.get_supported_head_sizes` method / `XPUMLASparseBackend.get_supported_head_sizes` 方法
```python
    @classmethod
    def get_supported_head_sizes(cls) -> list[int]:
        return [576]
```
**EN:** This method returns or derives a value within `XPUMLASparseBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`XPUMLASparseBackend`。

### `XPUMLASparseMetadata` class / `XPUMLASparseMetadata` 类
```python
@dataclass
class XPUMLASparseMetadata(AttentionMetadata):
    num_reqs: int
    max_query_len: int
    max_seq_len: int

    num_actual_tokens: int  # Number of tokens excluding padding.
    query_start_loc: torch.Tensor
    slot_mapping: torch.Tensor

    block_table: torch.Tensor
    req_id_per_token: torch.Tensor

    block_size: int = 1
    topk_tokens: int = 2048
```
**EN:** Uses `@dataclass` to package related state for `XPUMLASparseMetadata`. Typical fields include `num_reqs`, `max_query_len`, `max_seq_len`, `num_actual_tokens`, `query_start_loc`, `slot_mapping`.
**CN:** `XPUMLASparseMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `num_reqs`, `max_query_len`, `max_seq_len`, `num_actual_tokens`, `query_start_loc`, `slot_mapping`。

### `XPUMLASparseMetadataBuilder` class / `XPUMLASparseMetadataBuilder` 类
```python
@dataclass
class XPUMLASparseMetadataBuilder(AttentionMetadataBuilder[XPUMLASparseMetadata]):
    _cudagraph_support: ClassVar[AttentionCGSupport] = AttentionCGSupport.NEVER
```
**EN:** Uses `@dataclass` to package related state for `XPUMLASparseMetadataBuilder`. Typical fields include `_cudagraph_support`.
**CN:** `XPUMLASparseMetadataBuilder` 使用 `@dataclass` 打包相关状态。典型字段包括 `_cudagraph_support`。

### `XPUMLASparseMetadataBuilder.__init__` method / `XPUMLASparseMetadataBuilder.__init__` 方法
```python
    def __init__(
        self,
        kv_cache_spec: AttentionSpec,
        layer_names: list[str],
        vllm_config: VllmConfig,
        device: torch.device,
    ):
        self.kv_cache_spec = kv_cache_spec
        self.model_config = vllm_config.model_config
        parallel_config = vllm_config.parallel_config
        self.device = device
        max_num_batched_tokens = vllm_config.scheduler_config.max_num_batched_tokens

        self.num_heads = self.model_config.get_num_attention_heads(parallel_config)
        self.mla_dims = get_mla_dims(self.model_config)
        self.topk_tokens = vllm_config.model_config.hf_config.index_topk
        self.topk_tokens_tensor = torch.tensor(
            [self.topk_tokens], device=device, dtype=torch.int32
        )
        self.max_model_len_tensor = torch.tensor(
            [self.model_config.max_model_len], device=device, dtype=torch.int32
        )
        # this is ignored by `flash_mla_with_kvcache` if indices not None
        self.dummy_block_table = torch.empty(
            (1, 1), dtype=torch.int32, device=self.device
        )

        self.req_id_per_token_buffer = torch.empty(
            (max_num_batched_tokens,),
            dtype=torch.int32,
            device=device,
        )
```
**EN:** This method initializes the object state within `XPUMLASparseMetadataBuilder`. Key calls include `get_num_attention_heads`, `get_mla_dims`, `tensor`, `empty`. It touches state such as `kv_cache_spec`, `model_config`, `device`, `num_heads`, `mla_dims`, `topk_tokens`, `topk_tokens_tensor`, `max_model_len_tensor`.
**CN:** 该方法会初始化对象状态，其作用域位于`XPUMLASparseMetadataBuilder`。 关键调用包括 `get_num_attention_heads`, `get_mla_dims`, `tensor`, `empty`。 它会读写 `kv_cache_spec`, `model_config`, `device`, `num_heads`, `mla_dims`, `topk_tokens`, `topk_tokens_tensor`, `max_model_len_tensor` 等状态。

### `XPUMLASparseMetadataBuilder.build` method / `XPUMLASparseMetadataBuilder.build` 方法
```python
    def build(
        self,
        common_prefix_len: int,
        common_attn_metadata: CommonAttentionMetadata,
        fast_build: bool = False,
    ) -> XPUMLASparseMetadata:
        num_tokens = common_attn_metadata.num_actual_tokens
        starts = np.asarray(common_attn_metadata.query_start_loc_cpu, dtype=np.int32)
        seg_lengths = np.diff(starts)
        req_id_per_token = np.repeat(
            np.arange(seg_lengths.shape[0], dtype=np.int32), seg_lengths
        )
        # Zero-fill for cudagraphs
        self.req_id_per_token_buffer.fill_(0)
        self.req_id_per_token_buffer[: req_id_per_token.shape[0]].copy_(
            torch.from_numpy(req_id_per_token), non_blocking=True
        )

        req_id_per_token = self.req_id_per_token_buffer[:num_tokens]

        metadata = XPUMLASparseMetadata(
            num_reqs=common_attn_metadata.num_reqs,
            max_query_len=common_attn_metadata.max_query_len,
            max_seq_len=common_attn_metadata.max_seq_len,
            num_actual_tokens=common_attn_metadata.num_actual_tokens,
            query_start_loc=common_attn_metadata.query_start_loc,
            slot_mapping=common_attn_metadata.slot_mapping,
            block_table=common_attn_metadata.block_table_tensor,
            req_id_per_token=req_id_per_token,
            block_size=self.kv_cache_spec.block_size,
            topk_tokens=self.topk_tokens,
        )
        return metadata
```
**EN:** This method builds derived structures within `XPUMLASparseMetadataBuilder`. Key calls include `asarray`, `diff`, `repeat`, `fill_`, `copy_`, `XPUMLASparseMetadata`.
**CN:** 该方法会构建派生结构，其作用域位于`XPUMLASparseMetadataBuilder`。 关键调用包括 `asarray`, `diff`, `repeat`, `fill_`, `copy_`, `XPUMLASparseMetadata`。

### `XPUMLASparseImpl` class / `XPUMLASparseImpl` 类
```python
class XPUMLASparseImpl(SparseMLAAttentionImpl[XPUMLASparseMetadata]):
```
**EN:** Introduces the `XPUMLASparseImpl` class on top of `SparseMLAAttentionImpl[XPUMLASparseMetadata]`. Core methods include `__init__`, `_forward_bf16_kv`, `forward_mqa`.
**CN:** 这里定义 `XPUMLASparseImpl` 类，其基类包括 `SparseMLAAttentionImpl[XPUMLASparseMetadata]`。核心方法包括 `__init__`, `_forward_bf16_kv`, `forward_mqa`。

### `XPUMLASparseImpl.__init__` method / `XPUMLASparseImpl.__init__` 方法
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
        indexer: Optional["Indexer"] = None,
        **mla_args,
    ) -> None:
        self.num_heads = num_heads
        self.head_size = head_size
        self.scale = float(scale)
        self.num_kv_heads = num_kv_heads
        self.kv_cache_dtype = kv_cache_dtype
        self.kv_lora_rank: int = mla_args["kv_lora_rank"]
        self.softmax_scale = scale
        assert indexer is not None
        self.topk_indices_buffer: torch.Tensor | None = indexer.topk_indices_buffer
```
**EN:** This method initializes the object state within `XPUMLASparseImpl`. Key calls include `float`. It touches state such as `num_heads`, `head_size`, `scale`, `num_kv_heads`, `kv_cache_dtype`, `kv_lora_rank`, `softmax_scale`, `topk_indices_buffer`.
**CN:** 该方法会初始化对象状态，其作用域位于`XPUMLASparseImpl`。 关键调用包括 `float`。 它会读写 `num_heads`, `head_size`, `scale`, `num_kv_heads`, `kv_cache_dtype`, `kv_lora_rank`, `softmax_scale`, `topk_indices_buffer` 等状态。

### `XPUMLASparseImpl._forward_bf16_kv` method / `XPUMLASparseImpl._forward_bf16_kv` 方法
```python
    def _forward_bf16_kv(
        self,
        q: torch.Tensor,  # [sq, heads, d_qk]
        kv_c_and_k_pe_cache: torch.Tensor,  # [blocks, heads, d_qk]
        topk_indices: torch.Tensor,  # [sq, topk]
        attn_metadata: XPUMLASparseMetadata,
    ) -> torch.Tensor:
        num_tokens = q.shape[0]
        kv_c_and_k_pe_cache = kv_c_and_k_pe_cache.view(
            -1, 1, kv_c_and_k_pe_cache.shape[-1]
        )

        topk_indices = topk_indices.view(num_tokens, 1, -1)

        output, _, _ = triton_bf16_mla_sparse_interface(
            q,
            kv_c_and_k_pe_cache,
            topk_indices,
            sm_scale=self.softmax_scale,
        )

        return output[:, : self.num_heads, :]
```
**EN:** This method implements `_forward_bf16_kv` within `XPUMLASparseImpl`. Key calls include `view`, `triton_bf16_mla_sparse_interface`.
**CN:** 该方法会实现 `_forward_bf16_kv`，其作用域位于`XPUMLASparseImpl`。 关键调用包括 `view`, `triton_bf16_mla_sparse_interface`。

### `XPUMLASparseImpl.forward_mqa` method / `XPUMLASparseImpl.forward_mqa` 方法
```python
    def forward_mqa(
        self,
        q: torch.Tensor | tuple[torch.Tensor, torch.Tensor],
        kv_c_and_k_pe_cache: torch.Tensor,
        attn_metadata: XPUMLASparseMetadata,
        layer: AttentionLayer,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        # NOTE(lucas): for the sparse FlashMLA kernels the kernels want to use
        # MQA 576/512 approach for both prefill and decode

        if is_quantized_kv_cache(self.kv_cache_dtype):
            raise NotImplementedError("FP8 kv is not supported with XPU MLA Sparse yet")

        # Concatenate q if it's a tuple (ql_nope, q_pe)
        if isinstance(q, tuple):
            q = torch.cat(q, dim=-1)

        num_actual_toks = q.shape[0]

        assert self.topk_indices_buffer is not None
        topk_indices = self.topk_indices_buffer[:num_actual_toks]

        topk_indices_global = triton_convert_req_index_to_global_index(
            attn_metadata.req_id_per_token,
            attn_metadata.block_table,
            topk_indices,
            BLOCK_SIZE=attn_metadata.block_size,
            NUM_TOPK_TOKENS=attn_metadata.topk_tokens,
        )

        attn_out = self._forward_bf16_kv(
            q, kv_c_and_k_pe_cache, topk_indices_global, attn_metadata
        )

        return attn_out, None
```
**EN:** This method drives the forward-pass computation within `XPUMLASparseImpl`. Key calls include `is_quantized_kv_cache`, `isinstance`, `triton_convert_req_index_to_global_index`, `_forward_bf16_kv`, `NotImplementedError`, `cat`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会驱动前向计算流程，其作用域位于`XPUMLASparseImpl`。 关键调用包括 `is_quantized_kv_cache`, `isinstance`, `triton_convert_req_index_to_global_index`, `_forward_bf16_kv`, `NotImplementedError`, `cat`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `XPUMLASparseBackend`: central class or interface in this module. / `XPUMLASparseBackend`：本模块中的核心类或接口。
- `XPUMLASparseMetadata`: central class or interface in this module. / `XPUMLASparseMetadata`：本模块中的核心类或接口。
- `XPUMLASparseMetadataBuilder`: central class or interface in this module. / `XPUMLASparseMetadataBuilder`：本模块中的核心类或接口。
- `XPUMLASparseImpl`: central class or interface in this module. / `XPUMLASparseImpl`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`, `typing`
- External / 外部依赖: `numpy`, `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.config.cache`, `vllm.logger`, `vllm.model_executor.layers.attention.mla_attention`, `vllm.utils.torch_utils`, `vllm.v1.attention.backend`, `vllm.v1.attention.backends.mla.flashmla_sparse`, `vllm.v1.attention.ops.xpu_mla_sparse`, `vllm.v1.kv_cache_interface`, `vllm.model_executor.models.deepseek_v2`
