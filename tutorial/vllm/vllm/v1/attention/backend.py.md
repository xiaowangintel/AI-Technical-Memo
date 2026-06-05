# backend.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backend.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `AttentionType`, `MultipleOf`, `AttentionBackend` for the V1 `attention` subsystem. / 为 V1 的 `attention` 子系统实现 `AttentionType`, `MultipleOf`, `AttentionBackend`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
from abc import ABC, abstractmethod
from dataclasses import dataclass, replace
from enum import Enum
from typing import TYPE_CHECKING, Any, ClassVar, Generic, Protocol, TypeVar

import numpy as np
import torch
from typing_extensions import deprecated

from vllm.model_executor.layers.quantization.utils.quant_utils import (
    kFp8Dynamic64Sym,
    kFp8Dynamic128Sym,
    kFp8StaticTensorSym,
    kNvfp4Dynamic,
)

if TYPE_CHECKING:
    from vllm.config import VllmConfig
    from vllm.config.cache import CacheDType
    from vllm.model_executor.layers.linear import ColumnParallelLinear
    from vllm.model_executor.layers.quantization.utils.quant_utils import QuantKey
    from vllm.platforms.interface import DeviceCapability
    from vllm.v1.attention.backends.utils import KVCacheLayoutType
    from vllm.v1.kv_cache_interface import AttentionSpec, KVQuantMode

from vllm.v1.kv_cache_interface import get_kv_quant_mode
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `numpy`, `torch`, `typing_extensions`, and internal vLLM modules such as `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.config`, `vllm.config.cache`, `vllm.model_executor.layers.linear`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`numpy`, `torch`, `typing_extensions` 等外部依赖，以及 `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.config`, `vllm.config.cache`, `vllm.model_executor.layers.linear` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `AttentionType` class / `AttentionType` 类
```python
class AttentionType(str, Enum):
    """
    Attention type.
    Use string to be compatible with `torch.compile`.
    """

    DECODER = "decoder"
    """Decoder attention between previous layer Q/K/V."""
    ENCODER = "encoder"
    """Encoder attention between previous layer Q/K/V for encoder-decoder."""
    ENCODER_ONLY = "encoder_only"
    """Encoder attention between previous layer Q/K/V."""
    ENCODER_DECODER = "encoder_decoder"
    """Attention between dec. Q and enc. K/V for encoder-decoder."""
```
**EN:** Defines the `AttentionType` enum used to normalize modes or options across the subsystem. Representative members: `DECODER`, `ENCODER`, `ENCODER_ONLY`, `ENCODER_DECODER`.
**CN:** `AttentionType` 定义了一个枚举，用于统一子系统中的模式或选项。代表性成员包括：`DECODER`, `ENCODER`, `ENCODER_ONLY`, `ENCODER_DECODER`。

### `MultipleOf` class / `MultipleOf` 类
```python
class MultipleOf:
    base: int
```
**EN:** Introduces the `MultipleOf` class. Core methods include `__init__`.
**CN:** 这里定义 `MultipleOf` 类。核心方法包括 `__init__`。

### `MultipleOf.__init__` method / `MultipleOf.__init__` 方法
```python
    def __init__(self, base: int):
        self.base = base
```
**EN:** This method initializes the object state within `MultipleOf`. It touches state such as `base`.
**CN:** 该方法会初始化对象状态，其作用域位于`MultipleOf`。 它会读写 `base` 等状态。

### `AttentionBackend` class / `AttentionBackend` 类
```python
class AttentionBackend(ABC):
    """Abstract class for attention backends."""

    supported_dtypes: ClassVar[list[torch.dtype]] = [torch.float16, torch.bfloat16]
    supported_kv_cache_dtypes: ClassVar[list["CacheDType"]] = [
        "auto",
        "float16",
        "bfloat16",
    ]

    # Does attention's forward() include kv cache update?
    forward_includes_kv_cache_update: bool = True
```
**EN:** Declares the `AttentionBackend` interface. Downstream implementations are expected to provide methods such as `get_supported_kernel_block_sizes`, `get_name`, `get_impl_cls`, `get_builder_cls`, `get_kv_cache_shape`, `get_kv_cache_block_dim`.
**CN:** `AttentionBackend` 声明了一组接口约定。下游实现需要提供 `get_supported_kernel_block_sizes`, `get_name`, `get_impl_cls`, `get_builder_cls`, `get_kv_cache_shape`, `get_kv_cache_block_dim` 等方法。

### `AttentionBackend.get_builder_cls` method / `AttentionBackend.get_builder_cls` 方法
```python
    @staticmethod
    @abstractmethod
    def get_builder_cls():  # -> Type["AttentionMetadataBuilder"]:
        raise NotImplementedError
```
**EN:** This method returns or derives a value within `AttentionBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`AttentionBackend`。

### `AttentionBackend.get_kv_cache_stride_order` method / `AttentionBackend.get_kv_cache_stride_order` 方法
```python
    @staticmethod
    def get_kv_cache_stride_order(
        include_num_layers_dimension: bool = False,
    ) -> tuple[int, ...]:
        """
        Get the physical (memory layout) ordering of the kv cache dimensions.
        e.g. if the KV cache shape is
        [2, num_blocks, block_size, num_heads, head_size],
        and get_kv_cache_stride_order returns (1, 3, 0, 2, 4) then the physical
        ordering of dimensions is
        [num_blocks, num_heads, 2, block_size, head_size].

        If this function is unimplemented / raises NotImplementedError,
        the physical layout of the KV cache will match the logical shape.

        Args:
            include_num_layers_dimension: if True, includes an additional
                num_layers dimension, which is assumed to be prepended
                to the logical KV cache shape.
                With the above example, a return value (2, 4, 0, 1, 3, 5)
                corresponds to
                [num_blocks, num_heads, num_layers, 2, block_size, head_size].

                If an additional dimension is NOT included in the returned
                tuple, the physical layout will not include a layers dimension.

        Returns:
            A tuple of ints which is a permutation of range(len(shape)).
        """
        raise NotImplementedError
```
**EN:** This method returns or derives a value within `AttentionBackend`. The docstring frames it as: Get the physical (memory layout) ordering of the kv cache dimensions.
**CN:** 该方法会返回或推导一个值，其作用域位于`AttentionBackend`。

### `AttentionBackend.supports_compute_capability` method / `AttentionBackend.supports_compute_capability` 方法
```python
    @classmethod
    def supports_compute_capability(cls, capability: "DeviceCapability") -> bool:
        return True
```
**EN:** This method implements `supports_compute_capability` within `AttentionBackend`.
**CN:** 该方法会实现 `supports_compute_capability`，其作用域位于`AttentionBackend`。

### `AttentionBackend.validate_configuration` method / `AttentionBackend.validate_configuration` 方法
```python
    @classmethod
    def validate_configuration(
        cls,
        head_size: int,
        dtype: torch.dtype,
        kv_cache_dtype: "CacheDType | None",
        block_size: int | None,
        use_mla: bool,
        has_sink: bool,
        use_sparse: bool,
        use_mm_prefix: bool,
        use_per_head_quant_scales: bool,
        device_capability: "DeviceCapability",
        attn_type: str,
        use_non_causal: bool = False,
        use_batch_invariant: bool = False,
    ) -> list[str]:
        invalid_reasons = []
        if not cls.supports_head_size(head_size):
            invalid_reasons.append("head_size not supported")
        if not cls.supports_dtype(dtype):
            invalid_reasons.append("dtype not supported")
        if not cls.supports_kv_cache_dtype(kv_cache_dtype):
            invalid_reasons.append("kv_cache_dtype not supported")
        if not cls.supports_block_size(block_size):
            invalid_reasons.append("block_size not supported")
        if use_mm_prefix and not cls.supports_mm_prefix():
            invalid_reasons.append(
                "partial multimodal token full attention not supported"
            )
        if use_mla != cls.is_mla():
            if use_mla:
                invalid_reasons.append("MLA not supported")
            else:
                invalid_reasons.append("non-MLA not supported")
        if has_sink and not cls.supports_sink():
            invalid_reasons.append("attention sinks not supported")
        if use_sparse != cls.is_sparse():
            if use_sparse:
                invalid_reasons.append("sparse not supported")
            else:
                invalid_reasons.append("non-sparse not supported")
        if use_per_head_quant_scales and not cls.supports_per_head_quant_scales():
            invalid_reasons.append("per-head quant scales not supported")
        if not cls.supports_compute_capability(device_capability):
            invalid_reasons.append("compute capability not supported")
        if not cls.supports_attn_type(attn_type):
            invalid_reasons.append(f"attention type {attn_type} not supported")
        if use_non_causal and not cls.supports_non_causal():
            invalid_reasons.append("non-causal attention not supported")
        if use_batch_invariant and not cls.supports_batch_invariance():
            invalid_reasons.append("batch invariance not supported")
        combination_reason = cls.supports_combination(
            head_size,
            dtype,
            kv_cache_dtype,
            block_size,
            use_mla,
            has_sink,
            use_sparse,
            device_capability,
        )
        if combination_reason is not None:
            invalid_reasons.append(combination_reason)
        return invalid_reasons
```
**EN:** This method validates assumptions or constraints within `AttentionBackend`. Key calls include `supports_combination`, `supports_head_size`, `append`, `supports_dtype`, `supports_kv_cache_dtype`, `supports_block_size`. The control flow contains 16 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会校验前提与约束，其作用域位于`AttentionBackend`。 关键调用包括 `supports_combination`, `supports_head_size`, `append`, `supports_dtype`, `supports_kv_cache_dtype`, `supports_block_size`。 控制流包含 16 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `AttentionMetadata` class / `AttentionMetadata` 类
```python
class AttentionMetadata:
    pass
```
**EN:** Introduces the `AttentionMetadata` class. Core methods include its methods defined below.
**CN:** 这里定义 `AttentionMetadata` 类。核心方法包括 下方定义的方法。

### Module constants / 模块常量
```python
T = TypeVar("T", bound=AttentionMetadata)
```
**EN:** Defines module-level constants or aliases such as `T`, which are reused by later definitions.
**CN:** 定义 `T` 等模块级常量或别名，供后续定义复用。

### `CommonAttentionMetadata` class / `CommonAttentionMetadata` 类
```python
@dataclass
class CommonAttentionMetadata:
    """
    Per-batch attention metadata, shared across layers and backends.
    AttentionMetadataBuilder instances use it to construct per-layer metadata.

    For many of the tensors we keep both GPU and CPU versions.
    """

    query_start_loc: torch.Tensor
    query_start_loc_cpu: torch.Tensor
    """(batch_size + 1,), the start location of each request in query Tensor"""

    seq_lens: torch.Tensor
    """(batch_size,), the number of computed tokens for each request"""

    num_reqs: int
    """Number of requests"""
    # TODO(lucas): rename to num_tokens since it may be padded and this is misleading
    num_actual_tokens: int
    """Total number of tokens in batch"""
    max_query_len: int
    """Longest query in batch"""
    max_seq_len: int
    """Longest context length (may be an upper bound)"""

    block_table_tensor: torch.Tensor
    slot_mapping: torch.Tensor

    causal: bool = True

    # Needed by FastPrefillAttentionBuilder
    logits_indices_padded: torch.Tensor | None = None
    num_logits_indices: int | None = None

    # Needed by CrossAttentionBuilder
    encoder_seq_lens: torch.Tensor | None = None
    encoder_seq_lens_cpu: np.ndarray | None = None

    dcp_local_seq_lens: torch.Tensor | None = None
    dcp_local_seq_lens_cpu: torch.Tensor | None = None
    """Sequence lengths of the local rank in decode context parallelism world"""

    positions: torch.Tensor | None = None
    """(num_actual_tokens,) token positions.  Optional; set when the caller
    has positions available so that builders can pre-compute position-dependent
    metadata (e.g. C128A topk indices for DeepSeek V4)."""

    is_prefilling: torch.Tensor | None = None
    """(batch_size,) bool tensor: True if request is still in prefill phase
    (num_computed_tokens < num_prompt_tokens). Used by some backends to
    distinguish actual decodes from short extends."""

    seq_lens_cpu_upper_bound: torch.Tensor | None = None
    """(batch_size,) CPU upper bound on seq_lens. Precise for prefill rows
    and for all rows outside async spec decode; optimistic for async-spec
    decode rows (assumes every draft was accepted). Not safe for kernels
    that need exact per-row context lengths on decode rows."""

    # WARNING: Deprecated fields. Will be removed in a future release (v0.15.0)
    _seq_lens_cpu: torch.Tensor | None = None
    _num_computed_tokens_cpu: torch.Tensor | None = None

    _num_computed_tokens_cache: torch.Tensor | None = None
```
**EN:** Uses `@dataclass` to package related state for `CommonAttentionMetadata`. Typical fields include `query_start_loc`, `query_start_loc_cpu`, `seq_lens`, `num_reqs`, `num_actual_tokens`, `max_query_len`.
**CN:** `CommonAttentionMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `query_start_loc`, `query_start_loc_cpu`, `seq_lens`, `num_reqs`, `num_actual_tokens`, `max_query_len`。

### `CommonAttentionMetadata.seq_lens_cpu` method / `CommonAttentionMetadata.seq_lens_cpu` 方法
```python
    @property
    @deprecated(
        """
    Prefer using device seq_lens directly to avoid implicit H<>D sync.
    If a CPU copy is needed, use `seq_lens.cpu()` instead.
    Will be removed in a future release, please migrate as soon as possible.
    """
    )
    def seq_lens_cpu(self) -> torch.Tensor:
        if self._seq_lens_cpu is None:
            self._seq_lens_cpu = self.seq_lens.to("cpu")
        return self._seq_lens_cpu
```
**EN:** This method implements `seq_lens_cpu` within `CommonAttentionMetadata`. Key calls include `deprecated`, `to`. It touches state such as `_seq_lens_cpu`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `seq_lens_cpu`，其作用域位于`CommonAttentionMetadata`。 关键调用包括 `deprecated`, `to`。 它会读写 `_seq_lens_cpu` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `CommonAttentionMetadata.num_computed_tokens_cpu` method / `CommonAttentionMetadata.num_computed_tokens_cpu` 方法
```python
    @property
    @deprecated(
        """
    Prefer using device seq_lens directly to avoid implicit H<>D sync which breaks full
    async scheduling. If a CPU copy is needed, it can be derived from 
    query_start_loc_cpu and seq_lens.
    Will be removed in a future release, please migrate as soon as possible.
    """
    )
    def num_computed_tokens_cpu(self) -> torch.Tensor:
        if self._num_computed_tokens_cpu is None:
            query_seq_lens = (
                self.query_start_loc_cpu[1:] - self.query_start_loc_cpu[:-1]
            )
            self._num_computed_tokens_cpu = self.seq_lens_cpu - query_seq_lens
        return self._num_computed_tokens_cpu
```
**EN:** This method implements `num_computed_tokens_cpu` within `CommonAttentionMetadata`. Key calls include `deprecated`. It touches state such as `_num_computed_tokens_cpu`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `num_computed_tokens_cpu`，其作用域位于`CommonAttentionMetadata`。 关键调用包括 `deprecated`。 它会读写 `_num_computed_tokens_cpu` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `CommonAttentionMetadata.compute_num_computed_tokens` method / `CommonAttentionMetadata.compute_num_computed_tokens` 方法
```python
    def compute_num_computed_tokens(self) -> torch.Tensor:
        """Compute num_computed_tokens on device (seq_lens - query_lens)."""
        if self._num_computed_tokens_cache is None:
            query_lens = self.query_start_loc[1:] - self.query_start_loc[:-1]
            self._num_computed_tokens_cache = self.seq_lens - query_lens
        return self._num_computed_tokens_cache
```
**EN:** This method computes derived values within `CommonAttentionMetadata`. The docstring frames it as: Compute num_computed_tokens on device (seq_lens - query_lens). It touches state such as `_num_computed_tokens_cache`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会计算派生值，其作用域位于`CommonAttentionMetadata`。 它会读写 `_num_computed_tokens_cache` 等状态。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `CommonAttentionMetadata.unpadded` method / `CommonAttentionMetadata.unpadded` 方法
```python
    def unpadded(
        self, num_actual_tokens: int, num_actual_reqs: int
    ) -> "CommonAttentionMetadata":
        maybe_slice_reqs = lambda x: x[:num_actual_reqs] if x is not None else None
        return CommonAttentionMetadata(
            query_start_loc=self.query_start_loc[: num_actual_reqs + 1],
            query_start_loc_cpu=self.query_start_loc_cpu[: num_actual_reqs + 1],
            seq_lens=self.seq_lens[:num_actual_reqs],
            _seq_lens_cpu=self._seq_lens_cpu[:num_actual_reqs]
            if self._seq_lens_cpu is not None
            else None,
            _num_computed_tokens_cpu=self._num_computed_tokens_cpu[:num_actual_reqs]
            if self._num_computed_tokens_cpu is not None
            else None,
            num_reqs=num_actual_reqs,
            num_actual_tokens=num_actual_tokens,
            max_query_len=self.max_query_len,
            max_seq_len=self.max_seq_len,
            block_table_tensor=self.block_table_tensor[:num_actual_reqs],
            slot_mapping=self.slot_mapping[:num_actual_tokens],
            causal=self.causal,
            logits_indices_padded=self.logits_indices_padded,
            num_logits_indices=self.num_logits_indices,
            encoder_seq_lens=maybe_slice_reqs(self.encoder_seq_lens),
            encoder_seq_lens_cpu=maybe_slice_reqs(self.encoder_seq_lens_cpu),
            dcp_local_seq_lens=maybe_slice_reqs(self.dcp_local_seq_lens),
            dcp_local_seq_lens_cpu=maybe_slice_reqs(self.dcp_local_seq_lens_cpu),
            is_prefilling=maybe_slice_reqs(self.is_prefilling),
        )
```
**EN:** This method implements `unpadded` within `CommonAttentionMetadata`. Key calls include `CommonAttentionMetadata`, `maybe_slice_reqs`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `unpadded`，其作用域位于`CommonAttentionMetadata`。 关键调用包括 `CommonAttentionMetadata`, `maybe_slice_reqs`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### Module constants / 模块常量
```python
M = TypeVar("M")
```
**EN:** Defines module-level constants or aliases such as `M`, which are reused by later definitions.
**CN:** 定义 `M` 等模块级常量或别名，供后续定义复用。

### `AttentionCGSupport` class / `AttentionCGSupport` 类
```python
class AttentionCGSupport(Enum):
    """Constants for the cudagraph support of the attention backend
    Here we do not consider the cascade attention, as currently
    it is never cudagraph supported."""

    ALWAYS = 3
    """Cudagraph always supported; supports mixed-prefill-decode"""
    UNIFORM_BATCH = 2
    """Cudagraph supported for batches the only contain query lengths that are
    the same, this can be used for spec-decode
        i.e. "decodes" are 1 + num_speculative_tokens"""
    UNIFORM_SINGLE_TOKEN_DECODE = 1
    """Cudagraph supported for batches the only contain query_len==1 decodes"""
    NEVER = 0
    """NO cudagraph support"""
```
**EN:** Defines the `AttentionCGSupport` enum used to normalize modes or options across the subsystem. Representative members: `ALWAYS`, `UNIFORM_BATCH`, `UNIFORM_SINGLE_TOKEN_DECODE`, `NEVER`.
**CN:** `AttentionCGSupport` 定义了一个枚举，用于统一子系统中的模式或选项。代表性成员包括：`ALWAYS`, `UNIFORM_BATCH`, `UNIFORM_SINGLE_TOKEN_DECODE`, `NEVER`。

### `AttentionMetadataBuilder` class / `AttentionMetadataBuilder` 类
```python
class AttentionMetadataBuilder(ABC, Generic[M]):
    # Does this backend/builder support CUDA Graphs for attention (default: no).
    # Do not access directly. Call get_cudagraph_support() instead.
    _cudagraph_support: ClassVar[AttentionCGSupport] = AttentionCGSupport.NEVER
    # Does this backend/builder reorder the batch?
    # If not, set this to None. Otherwise set it to the query
    # length that will be pulled into the front of the batch.
    reorder_batch_threshold: int | None = None
    # Does this backend/builder support updating the block table in existing
    # metadata
    supports_update_block_table: bool = False
```
**EN:** Declares the `AttentionMetadataBuilder` interface. Downstream implementations are expected to provide methods such as `__init__`, `get_cudagraph_support`, `_init_reorder_batch_threshold`, `build`, `update_block_table`, `build_for_cudagraph_capture`.
**CN:** `AttentionMetadataBuilder` 声明了一组接口约定。下游实现需要提供 `__init__`, `get_cudagraph_support`, `_init_reorder_batch_threshold`, `build`, `update_block_table`, `build_for_cudagraph_capture` 等方法。

### `AttentionMetadataBuilder.build` method / `AttentionMetadataBuilder.build` 方法
```python
    @abstractmethod
    def build(
        self,
        common_prefix_len: int,
        common_attn_metadata: CommonAttentionMetadata,
        fast_build: bool = False,
    ) -> M:
        """
        Central method that builds attention metadata.
        Some builders (MLA) require reorder_batch to be called prior to build.

        Args:
            common_prefix_len: The length of the common prefix of the batch.
            common_attn_metadata: The common attention metadata.
            fast_build: The meta-data will prioritize speed of building over
                then speed at execution. Can be used for spec-decode where the
                result of a build call may only be used for few layers/iters.
        """
        raise NotImplementedError
```
**EN:** This method builds derived structures within `AttentionMetadataBuilder`. The docstring frames it as: Central method that builds attention metadata.
**CN:** 该方法会构建派生结构，其作用域位于`AttentionMetadataBuilder`。

### `AttentionMetadataBuilder.update_block_table` method / `AttentionMetadataBuilder.update_block_table` 方法
```python
    def update_block_table(
        self,
        metadata: M,
        blk_table: torch.Tensor,
        slot_mapping: torch.Tensor,
    ) -> M:
        """
        Update the block table for the attention metadata.
        Faster when theres multiple kv-cache groups that create virtually the
        same metadata but just with different block tables.

        Only needs to be implemented if supports_update_block_table is True.
        """
        raise NotImplementedError
```
**EN:** This method updates existing state within `AttentionMetadataBuilder`. The docstring frames it as: Update the block table for the attention metadata.
**CN:** 该方法会更新现有状态，其作用域位于`AttentionMetadataBuilder`。

### `AttentionMetadataBuilder.build_for_cudagraph_capture` method / `AttentionMetadataBuilder.build_for_cudagraph_capture` 方法
```python
    def build_for_cudagraph_capture(
        self, common_attn_metadata: CommonAttentionMetadata
    ) -> M:
        """
        Build attention metadata for CUDA graph capture. Uses build by default.
        Subclasses that override this method should call self.build or
        super().build_for_cudagraph_capture.
        """
        return self.build(
            common_prefix_len=0, common_attn_metadata=common_attn_metadata
        )
```
**EN:** This method builds derived structures within `AttentionMetadataBuilder`. The docstring frames it as: Build attention metadata for CUDA graph capture. Key calls include `build`.
**CN:** 该方法会构建派生结构，其作用域位于`AttentionMetadataBuilder`。 关键调用包括 `build`。

### `AttentionMetadataBuilder.build_for_drafting` method / `AttentionMetadataBuilder.build_for_drafting` 方法
```python
    def build_for_drafting(
        self,
        common_attn_metadata: CommonAttentionMetadata,
        draft_index: int,
    ) -> M:
        """
        Build attention metadata for draft model. Uses build by default.

        Args:
            common_attn_metadata: The common attention metadata.
            draft_index: The index of the current draft operation.
                When speculating a chain of tokens, this index refers to the
                draft attempt for the i-th token.
                For tree-based attention, this index instead refers to the
                draft attempt for the i-th level in the tree of tokens.
        """
        return self.build(
            common_prefix_len=0,
            common_attn_metadata=common_attn_metadata,
            fast_build=True,
        )
```
**EN:** This method builds derived structures within `AttentionMetadataBuilder`. The docstring frames it as: Build attention metadata for draft model. Key calls include `build`.
**CN:** 该方法会构建派生结构，其作用域位于`AttentionMetadataBuilder`。 关键调用包括 `build`。

### `AttentionLayer` class / `AttentionLayer` 类
```python
class AttentionLayer(Protocol):
    _q_scale: torch.Tensor
    _k_scale: torch.Tensor
    _v_scale: torch.Tensor
    _q_scale_float: float
    _k_scale_float: float
    _v_scale_float: float
    _prob_scale: torch.Tensor
```
**EN:** Declares the `AttentionLayer` interface. Downstream implementations are expected to provide methods such as `forward`.
**CN:** `AttentionLayer` 声明了一组接口约定。下游实现需要提供 `forward` 等方法。

### `AttentionLayer.forward` method / `AttentionLayer.forward` 方法
```python
    def forward(
        self,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        kv_cache: torch.Tensor,
        attn_metadata: AttentionMetadata,
    ) -> torch.Tensor: ...
```
**EN:** This method drives the forward-pass computation within `AttentionLayer`.
**CN:** 该方法会驱动前向计算流程，其作用域位于`AttentionLayer`。

### `AttentionImplBase` class / `AttentionImplBase` 类
```python
class AttentionImplBase(ABC, Generic[T]):
    """Base class for attention implementations.

    Contains common attributes and initialization logic shared by both
    standard AttentionImpl and MLAAttentionImpl. Does not define a forward
    method - subclasses define their own forward interfaces.
    """

    # Required attributes that all impls should have
    num_heads: int
    head_size: int
    scale: float

    # Whether the attention impl can return the softmax lse for decode.
    # Some features like decode context parallelism require the softmax lse.
    can_return_lse_for_decode: bool = False

    # Whether the attention impl supports Prefill Context Parallelism.
    supports_pcp: bool = False
    # Whether the attention impl(or ops) supports MTP
    # when cp_kv_cache_interleave_size > 1
    supports_mtp_with_cp_non_trivial_interleave_size: bool = False

    # some attention backends might not always want to return lse
    # even if they can return lse (for efficiency reasons)
    need_to_return_lse_for_decode: bool = False

    # Whether this attention implementation supports pre-quantized query input.
    # When True, the attention layer will quantize queries before passing them
    # to this backend, allowing torch.compile to fuse the quantization with
    # previous operations. This is typically supported when using FP8 KV cache
    # with compatible attention kernels (e.g., TRT-LLM).
    # Subclasses should set this in __init__.
    # TODO add support to more backends:
    # https://github.com/vllm-project/vllm/issues/25584
    supports_quant_query_input: bool = False

    dcp_world_size: int
    dcp_rank: int

    pcp_world_size: int
    pcp_rank: int

    total_cp_world_size: int
    total_cp_rank: int
```
**EN:** Declares the `AttentionImplBase` interface. Downstream implementations are expected to provide methods such as `__new__`, `process_weights_after_loading`.
**CN:** `AttentionImplBase` 声明了一组接口约定。下游实现需要提供 `__new__`, `process_weights_after_loading` 等方法。

### `AttentionImplBase.__new__` method / `AttentionImplBase.__new__` 方法
```python
    def __new__(cls, *args, **kwargs):
        # use __new__ so that all subclasses will call this
        self = super().__new__(cls)
        try:
            from vllm.distributed.parallel_state import get_dcp_group

            self.dcp_world_size = get_dcp_group().world_size
            self.dcp_rank = get_dcp_group().rank_in_group
        except AssertionError:
            # DCP might not be initialized in testing
            self.dcp_world_size = 1
            self.dcp_rank = 0
        try:
            from vllm.distributed.parallel_state import get_pcp_group

            self.pcp_world_size = get_pcp_group().world_size
            self.pcp_rank = get_pcp_group().rank_in_group
        except AssertionError:
            self.pcp_world_size = 1
            self.pcp_rank = 0
        self.total_cp_world_size = self.pcp_world_size * self.dcp_world_size
        self.total_cp_rank = self.pcp_rank * self.dcp_world_size + self.dcp_rank

        self.need_to_return_lse_for_decode = (
            self.dcp_world_size > 1 and self.can_return_lse_for_decode
        )
        return self
```
**EN:** This method implements `__new__` within `AttentionImplBase`. Key calls include `__new__`, `super`, `get_dcp_group`, `get_pcp_group`. It touches state such as `total_cp_world_size`, `total_cp_rank`, `need_to_return_lse_for_decode`, `dcp_world_size`, `dcp_rank`, `pcp_world_size`, `pcp_rank`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `__new__`，其作用域位于`AttentionImplBase`。 关键调用包括 `__new__`, `super`, `get_dcp_group`, `get_pcp_group`。 它会读写 `total_cp_world_size`, `total_cp_rank`, `need_to_return_lse_for_decode`, `dcp_world_size`, `dcp_rank`, `pcp_world_size`, `pcp_rank` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `AttentionImplBase.process_weights_after_loading` method / `AttentionImplBase.process_weights_after_loading` 方法
```python
    def process_weights_after_loading(self, act_dtype: torch.dtype):
        pass
```
**EN:** This method implements `process_weights_after_loading` within `AttentionImplBase`.
**CN:** 该方法会实现 `process_weights_after_loading`，其作用域位于`AttentionImplBase`。

### `AttentionImpl` class / `AttentionImpl` 类
```python
class AttentionImpl(AttentionImplBase[T], Generic[T]):
    """Standard attention implementation with forward method."""

    kv_cache_dtype: str
```
**EN:** Introduces the `AttentionImpl` class on top of `AttentionImplBase[T]`, `Generic[T]`. Core methods include `kv_quant_mode`, `__init__`, `forward`, `fused_output_quant_supported`, `fused_rope_kvcache_supported`, `do_rope_and_kv_cache_update`. Docstring signal: Standard attention implementation with forward method.
**CN:** 这里定义 `AttentionImpl` 类，其基类包括 `AttentionImplBase[T]`, `Generic[T]`。核心方法包括 `kv_quant_mode`, `__init__`, `forward`, `fused_output_quant_supported`, `fused_rope_kvcache_supported`, `do_rope_and_kv_cache_update`。

### `AttentionImpl.__init__` method / `AttentionImpl.__init__` 方法
```python
    @abstractmethod
    def __init__(
        self,
        num_heads: int,
        head_size: int,
        scale: float,
        num_kv_heads: int | None = None,
        alibi_slopes: list[float] | None = None,
        sliding_window: int | None = None,
        kv_cache_dtype: str = "auto",
        logits_soft_cap: float | None = None,
        attn_type: str = AttentionType.DECODER,
        kv_sharing_target_layer_name: str | None = None,
    ) -> None:
        raise NotImplementedError
```
**EN:** This method initializes the object state within `AttentionImpl`.
**CN:** 该方法会初始化对象状态，其作用域位于`AttentionImpl`。

### `AttentionImpl.forward` method / `AttentionImpl.forward` 方法
```python
    @abstractmethod
    def forward(
        self,
        layer: AttentionLayer,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        kv_cache: torch.Tensor,
        attn_metadata: T,
        output: torch.Tensor,
        output_scale: torch.Tensor | None = None,
        output_block_scale: torch.Tensor | None = None,
    ) -> torch.Tensor:
        raise NotImplementedError
```
**EN:** This method drives the forward-pass computation within `AttentionImpl`.
**CN:** 该方法会驱动前向计算流程，其作用域位于`AttentionImpl`。

### `AttentionImpl.fused_output_quant_supported` method / `AttentionImpl.fused_output_quant_supported` 方法
```python
    def fused_output_quant_supported(self, quant_key: "QuantKey"):
        """
        Does this attention implementation support fused output quantization.
        This is used by the AttnFusionPass to only fuse output quantization
        onto implementations that support it.

        :param quant_key: QuantKey object that describes the quantization op
        :return: is fusion supported for this type of quantization
        """
        return False
```
**EN:** This method implements `fused_output_quant_supported` within `AttentionImpl`. The docstring frames it as: Does this attention implementation support fused output quantization.
**CN:** 该方法会实现 `fused_output_quant_supported`，其作用域位于`AttentionImpl`。

### `AttentionImpl.do_rope_and_kv_cache_update` method / `AttentionImpl.do_rope_and_kv_cache_update` 方法
```python
    def do_rope_and_kv_cache_update(
        self,
        layer: AttentionLayer,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        positions: torch.Tensor,
        cos_sin_cache: torch.Tensor,
        is_neox: bool,
        kv_cache: torch.Tensor,
        layer_slot_mapping: torch.Tensor,
    ):
        """
        If `fused_rope_kvcache_supported` returns True, this method will be called
        by torch.ops.vllm.fused_rope_and_unified_kv_cache_update
        to perform the inplace RoPE and KV cache update.
        """
        raise NotImplementedError
```
**EN:** This method implements `do_rope_and_kv_cache_update` within `AttentionImpl`. The docstring frames it as: If `fused_rope_kvcache_supported` returns True, this method will be called by torch.ops.vllm.fused_rope_and_unified_kv_cache_update to perform the inplace RoPE and KV cache update.
**CN:** 该方法会实现 `do_rope_and_kv_cache_update`，其作用域位于`AttentionImpl`。

### `MLAAttentionImpl` class / `MLAAttentionImpl` 类
```python
class MLAAttentionImpl(AttentionImplBase[T], Generic[T]):
    """MLA attention implementation with forward_mqa and forward_mha methods."""
```
**EN:** Introduces the `MLAAttentionImpl` class on top of `AttentionImplBase[T]`, `Generic[T]`. Core methods include `__init__`, `forward_mha`, `forward_mqa`, `fused_output_quant_supported`, `do_kv_cache_update`. Docstring signal: MLA attention implementation with forward_mqa and forward_mha methods.
**CN:** 这里定义 `MLAAttentionImpl` 类，其基类包括 `AttentionImplBase[T]`, `Generic[T]`。核心方法包括 `__init__`, `forward_mha`, `forward_mqa`, `fused_output_quant_supported`, `do_kv_cache_update`。

### `MLAAttentionImpl.__init__` method / `MLAAttentionImpl.__init__` 方法
```python
    @abstractmethod
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
        q_lora_rank: int | None,
        kv_lora_rank: int,
        qk_nope_head_dim: int,
        qk_rope_head_dim: int,
        qk_head_dim: int,
        v_head_dim: int,
        kv_b_proj: "ColumnParallelLinear",
        indexer: object | None = None,
        q_pad_num_heads: int | None = None,
    ) -> None:
        raise NotImplementedError
```
**EN:** This method initializes the object state within `MLAAttentionImpl`.
**CN:** 该方法会初始化对象状态，其作用域位于`MLAAttentionImpl`。

### `MLAAttentionImpl.forward_mha` method / `MLAAttentionImpl.forward_mha` 方法
```python
    @abstractmethod
    def forward_mha(
        self,
        q: torch.Tensor,
        kv_c_normed: torch.Tensor,
        k_pe: torch.Tensor,
        kv_c_and_k_pe_cache: torch.Tensor,
        attn_metadata: T,
        k_scale: torch.Tensor,
        output: torch.Tensor,
    ) -> None:
        """MHA-style prefill forward pass."""
        raise NotImplementedError
```
**EN:** This method drives the forward-pass computation within `MLAAttentionImpl`. The docstring frames it as: MHA-style prefill forward pass.
**CN:** 该方法会驱动前向计算流程，其作用域位于`MLAAttentionImpl`。

### `MLAAttentionImpl.forward_mqa` method / `MLAAttentionImpl.forward_mqa` 方法
```python
    @abstractmethod
    def forward_mqa(
        self,
        q: torch.Tensor | tuple[torch.Tensor, torch.Tensor],
        kv_c_and_k_pe_cache: torch.Tensor,
        attn_metadata: T,
        layer: AttentionLayer,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        """MQA-style decode forward pass."""
        raise NotImplementedError
```
**EN:** This method drives the forward-pass computation within `MLAAttentionImpl`. The docstring frames it as: MQA-style decode forward pass.
**CN:** 该方法会驱动前向计算流程，其作用域位于`MLAAttentionImpl`。

### `MLAAttentionImpl.do_kv_cache_update` method / `MLAAttentionImpl.do_kv_cache_update` 方法
```python
    def do_kv_cache_update(
        self,
        kv_c_normed: torch.Tensor,
        k_pe: torch.Tensor,
        kv_cache: torch.Tensor,
        slot_mapping: torch.Tensor,
        kv_cache_dtype: str,
        k_scale: torch.Tensor,
    ) -> None:
        if kv_cache.numel() == 0:
            return
        from vllm import _custom_ops as ops

        ops.concat_and_cache_mla(
            kv_c_normed,
            k_pe.squeeze(1),
            kv_cache,
            slot_mapping.flatten(),
            kv_cache_dtype=kv_cache_dtype,
            scale=k_scale,
        )
```
**EN:** This method implements `do_kv_cache_update` within `MLAAttentionImpl`. Key calls include `concat_and_cache_mla`, `numel`, `squeeze`, `flatten`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `do_kv_cache_update`，其作用域位于`MLAAttentionImpl`。 关键调用包括 `concat_and_cache_mla`, `numel`, `squeeze`, `flatten`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `SparseMLAAttentionImpl` class / `SparseMLAAttentionImpl` 类
```python
class SparseMLAAttentionImpl(AttentionImplBase[T], Generic[T]):
    """Sparse MLA attention implementation with only forward_mqa method.

    Sparse MLA implementations only support decode (MQA-style) attention.
    They do not support prefill (MHA-style) attention.
    """
```
**EN:** Introduces the `SparseMLAAttentionImpl` class on top of `AttentionImplBase[T]`, `Generic[T]`. Core methods include `fused_output_quant_supported`, `__init__`, `forward_mqa`, `do_kv_cache_update`. Docstring signal: Sparse MLA attention implementation with only forward_mqa method.
**CN:** 这里定义 `SparseMLAAttentionImpl` 类，其基类包括 `AttentionImplBase[T]`, `Generic[T]`。核心方法包括 `fused_output_quant_supported`, `__init__`, `forward_mqa`, `do_kv_cache_update`。

### `SparseMLAAttentionImpl.fused_output_quant_supported` method / `SparseMLAAttentionImpl.fused_output_quant_supported` 方法
```python
    def fused_output_quant_supported(self, quant_key: "QuantKey"):
        """
        Does this attention implementation support fused output quantization.
        Since MLA quantization is done manually in forward_impl (common code),
        all MLA backends support it by default.
        """
        return quant_key in (
            kFp8StaticTensorSym,
            kNvfp4Dynamic,
            kFp8Dynamic128Sym,
            kFp8Dynamic64Sym,
        )
```
**EN:** This method implements `fused_output_quant_supported` within `SparseMLAAttentionImpl`. The docstring frames it as: Does this attention implementation support fused output quantization.
**CN:** 该方法会实现 `fused_output_quant_supported`，其作用域位于`SparseMLAAttentionImpl`。

### `SparseMLAAttentionImpl.__init__` method / `SparseMLAAttentionImpl.__init__` 方法
```python
    @abstractmethod
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
        q_lora_rank: int | None,
        kv_lora_rank: int,
        qk_nope_head_dim: int,
        qk_rope_head_dim: int,
        qk_head_dim: int,
        v_head_dim: int,
        kv_b_proj: "ColumnParallelLinear",
        indexer: object | None = None,
        q_pad_num_heads: int | None = None,
    ) -> None:
        raise NotImplementedError
```
**EN:** This method initializes the object state within `SparseMLAAttentionImpl`.
**CN:** 该方法会初始化对象状态，其作用域位于`SparseMLAAttentionImpl`。

### `SparseMLAAttentionImpl.forward_mqa` method / `SparseMLAAttentionImpl.forward_mqa` 方法
```python
    @abstractmethod
    def forward_mqa(
        self,
        q: torch.Tensor | tuple[torch.Tensor, torch.Tensor],
        kv_c_and_k_pe_cache: torch.Tensor,
        attn_metadata: T,
        layer: AttentionLayer,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        """MQA-style decode forward pass."""
        raise NotImplementedError
```
**EN:** This method drives the forward-pass computation within `SparseMLAAttentionImpl`. The docstring frames it as: MQA-style decode forward pass.
**CN:** 该方法会驱动前向计算流程，其作用域位于`SparseMLAAttentionImpl`。

### `SparseMLAAttentionImpl.do_kv_cache_update` method / `SparseMLAAttentionImpl.do_kv_cache_update` 方法
```python
    def do_kv_cache_update(
        self,
        kv_c_normed: torch.Tensor,
        k_pe: torch.Tensor,
        kv_cache: torch.Tensor,
        slot_mapping: torch.Tensor,
        kv_cache_dtype: str,
        k_scale: torch.Tensor,
    ) -> None:
        if kv_cache.numel() == 0:
            return
        from vllm import _custom_ops as ops

        ops.concat_and_cache_mla(
            kv_c_normed,
            k_pe.squeeze(1),
            kv_cache,
            slot_mapping.flatten(),
            kv_cache_dtype=kv_cache_dtype,
            scale=k_scale,
        )
```
**EN:** This method implements `do_kv_cache_update` within `SparseMLAAttentionImpl`. Key calls include `concat_and_cache_mla`, `numel`, `squeeze`, `flatten`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `do_kv_cache_update`，其作用域位于`SparseMLAAttentionImpl`。 关键调用包括 `concat_and_cache_mla`, `numel`, `squeeze`, `flatten`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `subclass_attention_backend` function / `subclass_attention_backend` 函数
```python
def subclass_attention_backend(
    name_prefix: str,
    attention_backend_cls: type[AttentionBackend],
    builder_cls: type[AttentionMetadataBuilder[M]],
) -> type[AttentionBackend]:
    """
    Return a new subclass where `get_builder_cls` returns `builder_cls`.
    """
    name: str = name_prefix + attention_backend_cls.__name__  # type: ignore

    return type(
        name, (attention_backend_cls,), {"get_builder_cls": lambda: builder_cls}
    )
```
**EN:** This function implements `subclass_attention_backend` within the module. The docstring frames it as: Return a new subclass where `get_builder_cls` returns `builder_cls`. Key calls include `type`.
**CN:** 该函数会实现 `subclass_attention_backend`，其作用域位于the module。 关键调用包括 `type`。

### `subclass_attention_backend_with_overrides` function / `subclass_attention_backend_with_overrides` 函数
```python
def subclass_attention_backend_with_overrides(
    name_prefix: str,
    attention_backend_cls: type[AttentionBackend],
    overrides: dict[str, Any],
) -> type[AttentionBackend]:
    name: str = name_prefix + attention_backend_cls.__name__  # type: ignore
    return type(name, (attention_backend_cls,), overrides)
```
**EN:** This function implements `subclass_attention_backend_with_overrides` within the module. Key calls include `type`.
**CN:** 该函数会实现 `subclass_attention_backend_with_overrides`，其作用域位于the module。 关键调用包括 `type`。

## Key Concepts / 关键概念
- `AttentionType`: central class or interface in this module. / `AttentionType`：本模块中的核心类或接口。
- `MultipleOf`: central class or interface in this module. / `MultipleOf`：本模块中的核心类或接口。
- `AttentionBackend`: central class or interface in this module. / `AttentionBackend`：本模块中的核心类或接口。
- `AttentionMetadata`: central class or interface in this module. / `AttentionMetadata`：本模块中的核心类或接口。
- `CommonAttentionMetadata`: central class or interface in this module. / `CommonAttentionMetadata`：本模块中的核心类或接口。
- `AttentionCGSupport`: central class or interface in this module. / `AttentionCGSupport`：本模块中的核心类或接口。
- `AttentionMetadataBuilder`: central class or interface in this module. / `AttentionMetadataBuilder`：本模块中的核心类或接口。
- `AttentionLayer`: central class or interface in this module. / `AttentionLayer`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `abc`, `dataclasses`, `enum`, `typing`
- External / 外部依赖: `numpy`, `torch`, `typing_extensions`
- Internal vLLM / 内部依赖: `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.config`, `vllm.config.cache`, `vllm.model_executor.layers.linear`, `vllm.platforms.interface`, `vllm.v1.attention.backends.utils`, `vllm.v1.kv_cache_interface`, `vllm.distributed.parallel_state`, `vllm`
