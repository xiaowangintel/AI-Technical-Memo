# deepseek_compressor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/deepseek_compressor.py`
- **Repository**: vllm-project/vllm
- **Purpose**: DeepSeek-specific compression or attention logic / DeepSeek 专用压缩或注意力逻辑

## Line-by-Line Analysis / 逐行分析
### Lines 4-38 — imports and setup
```python
from dataclasses import dataclass
from typing import Any, ClassVar, cast

import torch
from torch import nn

from vllm.config import VllmConfig, get_current_vllm_config
from vllm.forward_context import get_forward_context
from vllm.model_executor.layers.attention_layer_base import AttentionLayerBase
from vllm.model_executor.layers.layernorm import RMSNorm
from vllm.model_executor.layers.linear import (
    MergedColumnParallelLinear,
)
from vllm.platforms import current_platform
from vllm.triton_utils import tl, triton
from vllm.v1.attention.backend import (
    AttentionBackend,
    AttentionCGSupport,
    AttentionMetadataBuilder,
    CommonAttentionMetadata,
    MultipleOf,
)
from vllm.v1.attention.ops.deepseek_v4_ops.fused_compress_quant_cache import (
    _fused_kv_compress_norm_rope_insert_indexer_attn,
    _fused_kv_compress_norm_rope_insert_indexer_mxfp4_attn,
    _fused_kv_compress_norm_rope_insert_sparse_attn,
)
from vllm.v1.attention.ops.deepseek_v4_ops.fused_indexer_q import (
    MXFP4_BLOCK_SIZE,
)
from vllm.v1.kv_cache_interface import (
    KVCacheSpec,
    MLAAttentionSpec,
    SlidingWindowMLASpec,
)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。

### Lines 41-78 — class `CompressorBackend`
```python
class CompressorBackend(AttentionBackend):
    def __init__(self):
        super().__init__()

    @staticmethod
    def get_name() -> str:
        return "CompressorBackend"

    @staticmethod
    def get_supported_kernel_block_sizes() -> list[int | MultipleOf]:
        return [MultipleOf(1)]

    @classmethod
    def get_supported_head_sizes(cls) -> list[int]:
# ... omitted for brevity ...
            return (0, 1, 2, 3)
        return (0, 1, 2)
```
**EN:** This class defines `CompressorBackend`. It inherits from `AttentionBackend`. It provides one of the file's main runtime building blocks. Important methods include `__init__`, `get_name`, `get_supported_kernel_block_sizes`, `get_supported_head_sizes`, `get_builder_cls`, `get_kv_cache_shape`. Key calls include `super.__init__`, `MultipleOf`, `super`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `CompressorBackend`。 它继承自 `AttentionBackend`。 它是该文件中的一个主要运行时构件。 重要方法包括 `__init__`, `get_name`, `get_supported_kernel_block_sizes`, `get_supported_head_sizes`, `get_builder_cls`, `get_kv_cache_shape`。 关键调用包括 `super.__init__`, `MultipleOf`, `super`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 42-43 — method `CompressorBackend.__init__`
```python
    def __init__(self):
        super().__init__()
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. Key calls include `super.__init__`, `super`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 关键调用包括 `super.__init__`, `super`。

### Lines 45-47 — method `CompressorBackend.get_name`
```python
    @staticmethod
    def get_name() -> str:
        return "CompressorBackend"
```
**EN:** This method defines `get_name`. It derives and returns computed metadata or outputs needed by later stages.
**CN:** 该方法定义 `get_name`。 它推导并返回后续阶段所需的元数据或输出。

### Lines 49-51 — method `CompressorBackend.get_supported_kernel_block_sizes`
```python
    @staticmethod
    def get_supported_kernel_block_sizes() -> list[int | MultipleOf]:
        return [MultipleOf(1)]
```
**EN:** This method defines `get_supported_kernel_block_sizes`. It implements a low-level kernel that works on tiled tensor blocks. Key calls include `MultipleOf`.
**CN:** 该方法定义 `get_supported_kernel_block_sizes`。 它实现了按分块张量执行的底层内核。 关键调用包括 `MultipleOf`。

### Lines 53-55 — method `CompressorBackend.get_supported_head_sizes`
```python
    @classmethod
    def get_supported_head_sizes(cls) -> list[int]:
        return [512, 1024]
```
**EN:** This method defines `get_supported_head_sizes`. It derives and returns computed metadata or outputs needed by later stages.
**CN:** 该方法定义 `get_supported_head_sizes`。 它推导并返回后续阶段所需的元数据或输出。

### Lines 57-59 — method `CompressorBackend.get_builder_cls`
```python
    @staticmethod
    def get_builder_cls() -> type["CompressorMetadataBuilder"]:
        return CompressorMetadataBuilder
```
**EN:** This method defines `get_builder_cls`. It derives and returns computed metadata or outputs needed by later stages.
**CN:** 该方法定义 `get_builder_cls`。 它推导并返回后续阶段所需的元数据或输出。

### Lines 61-70 — method `CompressorBackend.get_kv_cache_shape`
```python
    @staticmethod
    def get_kv_cache_shape(
        num_blocks: int,
        block_size: int,
        num_kv_heads: int,
        head_size: int,
        cache_dtype_str: str = "auto",
    ) -> tuple[int, ...]:
        assert num_kv_heads == 1
        return (num_blocks, block_size, head_size)
```
**EN:** This method defines `get_kv_cache_shape`. It derives and returns computed metadata or outputs needed by later stages. The main inputs are `num_blocks`, `block_size`, `num_kv_heads`, `head_size`, `cache_dtype_str`.
**CN:** 该方法定义 `get_kv_cache_shape`。 它推导并返回后续阶段所需的元数据或输出。 其主要输入参数包括 `num_blocks`, `block_size`, `num_kv_heads`, `head_size`, `cache_dtype_str`。

### Lines 81-87 — class `CompressorMetadata`
```python
@dataclass
class CompressorMetadata:
    block_table: torch.Tensor
    slot_mapping: torch.Tensor
    block_size: int

    token_to_req_indices: torch.Tensor | None = None  # [num_tokens]
```
**EN:** This class defines `CompressorMetadata`. It provides one of the file's main runtime building blocks. It writes or updates `block_table`, `slot_mapping`, `block_size`, `token_to_req_indices`.
**CN:** 该类定义了 `CompressorMetadata`。 它是该文件中的一个主要运行时构件。 它会写入或更新 `block_table`, `slot_mapping`, `block_size`, `token_to_req_indices`。

### Lines 90-122 — class `CompressorMetadataBuilder`
```python
class CompressorMetadataBuilder(AttentionMetadataBuilder):
    _cudagraph_support: ClassVar[AttentionCGSupport] = AttentionCGSupport.ALWAYS

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        assert isinstance(self.kv_cache_spec, SlidingWindowMLASpec | MLAAttentionSpec)
        mla_spec = cast(SlidingWindowMLASpec | MLAAttentionSpec, self.kv_cache_spec)
        self.block_size = mla_spec.block_size

        self.token_to_req_indices = torch.zeros(
            self.vllm_config.scheduler_config.max_num_batched_tokens,
            dtype=torch.int32,
            device=self.device,
        )
# ... omitted for brevity ...
            token_to_req_indices=token_to_req_indices,
        )
```
**EN:** This class defines `CompressorMetadataBuilder`. It inherits from `AttentionMetadataBuilder`. It provides one of the file's main runtime building blocks. Important methods include `__init__`, `build`. Key calls include `super.__init__`, `isinstance`, `cast`, `torch.zeros`, `torch.repeat_interleave.pin_memory`, `token_to_req_indices.copy_`. It writes or updates `_cudagraph_support`, `mla_spec`, `block_size`, `token_to_req_indices`, `query_start_loc_cpu`, `num_reqs`.
**CN:** 该类定义了 `CompressorMetadataBuilder`。 它继承自 `AttentionMetadataBuilder`。 它是该文件中的一个主要运行时构件。 重要方法包括 `__init__`, `build`。 关键调用包括 `super.__init__`, `isinstance`, `cast`, `torch.zeros`, `torch.repeat_interleave.pin_memory`, `token_to_req_indices.copy_`。 它会写入或更新 `_cudagraph_support`, `mla_spec`, `block_size`, `token_to_req_indices`, `query_start_loc_cpu`, `num_reqs`。

### Lines 93-103 — method `CompressorMetadataBuilder.__init__`
```python
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        assert isinstance(self.kv_cache_spec, SlidingWindowMLASpec | MLAAttentionSpec)
        mla_spec = cast(SlidingWindowMLASpec | MLAAttentionSpec, self.kv_cache_spec)
        self.block_size = mla_spec.block_size

        self.token_to_req_indices = torch.zeros(
            self.vllm_config.scheduler_config.max_num_batched_tokens,
            dtype=torch.int32,
            device=self.device,
        )
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `*args`, `**kwargs`. Key calls include `super.__init__`, `isinstance`, `cast`, `torch.zeros`, `super`. It writes or updates `mla_spec`, `block_size`, `token_to_req_indices`.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `*args`, `**kwargs`。 关键调用包括 `super.__init__`, `isinstance`, `cast`, `torch.zeros`, `super`。 它会写入或更新 `mla_spec`, `block_size`, `token_to_req_indices`。

### Lines 105-122 — method `CompressorMetadataBuilder.build`
```python
    def build(
        self,
        common_prefix_len: int,
        common_attn_metadata: CommonAttentionMetadata,
        fast_build: bool = False,
    ) -> CompressorMetadata:
        query_start_loc_cpu = common_attn_metadata.query_start_loc_cpu
        num_reqs = common_attn_metadata.num_reqs
        query_lens = query_start_loc_cpu[1:] - query_start_loc_cpu[:-1]
        x = torch.repeat_interleave(torch.arange(num_reqs), query_lens).pin_memory()
        token_to_req_indices = self.token_to_req_indices[: x.shape[0]]
        token_to_req_indices.copy_(x, non_blocking=True)
        return CompressorMetadata(
            block_table=common_attn_metadata.block_table_tensor.clamp_(min=0),
            slot_mapping=common_attn_metadata.slot_mapping,
            block_size=self.block_size,
            token_to_req_indices=token_to_req_indices,
        )
```
**EN:** This method defines `build`. It provides one of the file's main runtime building blocks. The main inputs are `common_prefix_len`, `common_attn_metadata`, `fast_build`. Key calls include `torch.repeat_interleave.pin_memory`, `token_to_req_indices.copy_`, `CompressorMetadata`, `torch.repeat_interleave`, `common_attn_metadata.block_table_tensor.clamp_`, `torch.arange`. It writes or updates `query_start_loc_cpu`, `num_reqs`, `query_lens`, `x`, `token_to_req_indices`.
**CN:** 该方法定义 `build`。 它是该文件中的一个主要运行时构件。 其主要输入参数包括 `common_prefix_len`, `common_attn_metadata`, `fast_build`。 关键调用包括 `torch.repeat_interleave.pin_memory`, `token_to_req_indices.copy_`, `CompressorMetadata`, `torch.repeat_interleave`, `common_attn_metadata.block_table_tensor.clamp_`, `torch.arange`。 它会写入或更新 `query_start_loc_cpu`, `num_reqs`, `query_lens`, `x`, `token_to_req_indices`。

### Lines 125-174 — class `CompressorStateCache`
```python
class CompressorStateCache(torch.nn.Module, AttentionLayerBase):
    def __init__(
        self,
        state_dim: int,
        dtype: torch.dtype,
        compress_ratio: int,
        prefix: str,
    ):
        super().__init__()
        self.state_dim = state_dim
        self.dtype = dtype
        self.prefix = prefix
        self.kv_cache = torch.tensor([])
        compilation_config = get_current_vllm_config().compilation_config
# ... omitted for brevity ...
    def get_attn_backend(self) -> type[AttentionBackend]:
        return CompressorBackend
```
**EN:** This class defines `CompressorStateCache`. It inherits from `torch.nn.Module`, `AttentionLayerBase`. It provides one of the file's main runtime building blocks. Important methods include `__init__`, `forward`, `get_kv_cache_spec`, `get_attn_backend`. Key calls include `super.__init__`, `torch.tensor`, `SlidingWindowMLASpec`, `get_current_vllm_config`, `ValueError`, `super`. It writes or updates `state_dim`, `dtype`, `prefix`, `kv_cache`, `compilation_config`, `coff`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `CompressorStateCache`。 它继承自 `torch.nn.Module`, `AttentionLayerBase`。 它是该文件中的一个主要运行时构件。 重要方法包括 `__init__`, `forward`, `get_kv_cache_spec`, `get_attn_backend`。 关键调用包括 `super.__init__`, `torch.tensor`, `SlidingWindowMLASpec`, `get_current_vllm_config`, `ValueError`, `super`。 它会写入或更新 `state_dim`, `dtype`, `prefix`, `kv_cache`, `compilation_config`, `coff`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 126-159 — method `CompressorStateCache.__init__`
```python
    def __init__(
        self,
        state_dim: int,
        dtype: torch.dtype,
        compress_ratio: int,
        prefix: str,
    ):
        super().__init__()
        self.state_dim = state_dim
        self.dtype = dtype
        self.prefix = prefix
        self.kv_cache = torch.tensor([])
        compilation_config = get_current_vllm_config().compilation_config
        if prefix in compilation_config.static_forward_context:
            raise ValueError(f"Duplicate layer name: {prefix}")
        compilation_config.static_forward_context[prefix] = self

        assert self.dtype == torch.float32
        assert compress_ratio in [4, 128]
        coff = 1 + (compress_ratio == 4)
        self.sliding_window = coff * compress_ratio
        # Block size is constrained by tensor sharing between compressor states
        # and KV blocks. Since compressor states share the same physical tensor
        # as KV blocks, they must use the same page size.
        # The KV block shape [256//4, head_dim] = [64, 584] determines:
        # - C4 compressor block shape [4, 2*512*2*4] -> block_size = 4
        # - C128 compressor block shape [8, 512*2*4] -> block_size = 8
        # TODO(yifan): make block size automatically determined and configurable.
        if compress_ratio == 4:
            self.block_size = 4
        elif compress_ratio == 128:
            self.block_size = 8
        else:
            raise ValueError(f"Invalid compress ratio: {compress_ratio}")
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `state_dim`, `dtype`, `compress_ratio`, `prefix`. Key calls include `super.__init__`, `torch.tensor`, `get_current_vllm_config`, `ValueError`, `super`. It writes or updates `state_dim`, `dtype`, `prefix`, `kv_cache`, `compilation_config`, `coff`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `state_dim`, `dtype`, `compress_ratio`, `prefix`。 关键调用包括 `super.__init__`, `torch.tensor`, `get_current_vllm_config`, `ValueError`, `super`。 它会写入或更新 `state_dim`, `dtype`, `prefix`, `kv_cache`, `compilation_config`, `coff`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Deepseek-specific compression or attention logic / [CN] DeepSeek 专用压缩或注意力逻辑
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `CompressorBackend`, `CompressorMetadata`, `CompressorMetadataBuilder`, `CompressorStateCache` / [CN] 核心符号：`CompressorBackend`, `CompressorMetadata`, `CompressorMetadataBuilder`, `CompressorStateCache`

## Dependencies / 依赖关系
- **External**: `dataclasses`, `typing`, `torch` / **外部依赖**: `dataclasses`, `typing`, `torch`
- **Internal**: `vllm.config`, `vllm.forward_context`, `vllm.model_executor.layers.attention_layer_base`, `vllm.model_executor.layers.layernorm`, `vllm.model_executor.layers.linear`, `vllm.platforms`, `vllm.triton_utils`, `vllm.v1.attention.backend`, `vllm.v1.attention.ops.deepseek_v4_ops.fused_compress_quant_cache`, `vllm.v1.attention.ops.deepseek_v4_ops.fused_indexer_q` / **内部依赖**: `vllm.config`, `vllm.forward_context`, `vllm.model_executor.layers.attention_layer_base`, `vllm.model_executor.layers.layernorm`, `vllm.model_executor.layers.linear`, `vllm.platforms`, `vllm.triton_utils`, `vllm.v1.attention.backend`, `vllm.v1.attention.ops.deepseek_v4_ops.fused_compress_quant_cache`, `vllm.v1.attention.ops.deepseek_v4_ops.fused_indexer_q`
- **Runtime traits**: platform-aware dispatch, Triton kernels / **运行时特征**: platform-aware dispatch, Triton kernels
