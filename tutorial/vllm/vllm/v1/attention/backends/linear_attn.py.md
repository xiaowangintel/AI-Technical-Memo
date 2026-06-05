# linear_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/linear_attn.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `LinearAttentionBackend`, `LinearAttentionMetadata`, `LinearAttentionMetadataBuilder` for the V1 `attention/backends` subsystem. / 为 V1 的 `attention/backends` 子系统实现 `LinearAttentionBackend`, `LinearAttentionMetadata`, `LinearAttentionMetadataBuilder`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
from dataclasses import dataclass

import torch

from vllm.config import VllmConfig
from vllm.v1.attention.backend import (
    AttentionBackend,
    AttentionCGSupport,
    AttentionMetadataBuilder,
    CommonAttentionMetadata,
)
from vllm.v1.attention.backends.utils import (
    mamba_get_block_table_tensor,
    split_decodes_and_prefills,
)
from vllm.v1.kv_cache_interface import AttentionSpec, MambaSpec
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `torch`, and internal vLLM modules such as `vllm.config`, `vllm.v1.attention.backend`, `vllm.v1.attention.backends.utils`, `vllm.v1.kv_cache_interface`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`torch` 等外部依赖，以及 `vllm.config`, `vllm.v1.attention.backend`, `vllm.v1.attention.backends.utils`, `vllm.v1.kv_cache_interface` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `LinearAttentionBackend` class / `LinearAttentionBackend` 类
```python
class LinearAttentionBackend(AttentionBackend):
```
**EN:** Introduces the `LinearAttentionBackend` class on top of `AttentionBackend`. Core methods include `get_name`, `get_builder_cls`, `is_ssm`.
**CN:** 这里定义 `LinearAttentionBackend` 类，其基类包括 `AttentionBackend`。核心方法包括 `get_name`, `get_builder_cls`, `is_ssm`。

### `LinearAttentionBackend.get_name` method / `LinearAttentionBackend.get_name` 方法
```python
    @staticmethod
    def get_name() -> str:
        return "LINEAR_ATTN"
```
**EN:** This method returns or derives a value within `LinearAttentionBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`LinearAttentionBackend`。

### `LinearAttentionBackend.get_builder_cls` method / `LinearAttentionBackend.get_builder_cls` 方法
```python
    @staticmethod
    def get_builder_cls() -> type["LinearAttentionMetadataBuilder"]:
        return LinearAttentionMetadataBuilder
```
**EN:** This method returns or derives a value within `LinearAttentionBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`LinearAttentionBackend`。

### `LinearAttentionBackend.is_ssm` method / `LinearAttentionBackend.is_ssm` 方法
```python
    @classmethod
    def is_ssm(cls) -> bool:
        return True
```
**EN:** This method answers a boolean capability check within `LinearAttentionBackend`.
**CN:** 该方法会回答布尔能力判断，其作用域位于`LinearAttentionBackend`。

### `LinearAttentionMetadata` class / `LinearAttentionMetadata` 类
```python
@dataclass
class LinearAttentionMetadata:
    num_prefills: int
    num_prefill_tokens: int
    num_decodes: int
    num_decode_tokens: int
    query_start_loc: torch.Tensor
    seq_lens: torch.Tensor

    state_indices_tensor: torch.Tensor  # shape: [batch,]
```
**EN:** Uses `@dataclass` to package related state for `LinearAttentionMetadata`. Typical fields include `num_prefills`, `num_prefill_tokens`, `num_decodes`, `num_decode_tokens`, `query_start_loc`, `seq_lens`.
**CN:** `LinearAttentionMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `num_prefills`, `num_prefill_tokens`, `num_decodes`, `num_decode_tokens`, `query_start_loc`, `seq_lens`。

### `LinearAttentionMetadataBuilder` class / `LinearAttentionMetadataBuilder` 类
```python
class LinearAttentionMetadataBuilder(AttentionMetadataBuilder[LinearAttentionMetadata]):
    reorder_batch_threshold: int = 1

    _cudagraph_support = AttentionCGSupport.UNIFORM_SINGLE_TOKEN_DECODE
```
**EN:** Introduces the `LinearAttentionMetadataBuilder` class on top of `AttentionMetadataBuilder[LinearAttentionMetadata]`. Core methods include `__init__`, `build`.
**CN:** 这里定义 `LinearAttentionMetadataBuilder` 类，其基类包括 `AttentionMetadataBuilder[LinearAttentionMetadata]`。核心方法包括 `__init__`, `build`。

### `LinearAttentionMetadataBuilder.__init__` method / `LinearAttentionMetadataBuilder.__init__` 方法
```python
    def __init__(
        self,
        kv_cache_spec: AttentionSpec,
        layer_names: list[str],
        vllm_config: VllmConfig,
        device: torch.device,
    ):
        super().__init__(kv_cache_spec, layer_names, vllm_config, device)
        assert isinstance(kv_cache_spec, MambaSpec)
```
**EN:** This method initializes the object state within `LinearAttentionMetadataBuilder`. Key calls include `__init__`, `isinstance`, `super`.
**CN:** 该方法会初始化对象状态，其作用域位于`LinearAttentionMetadataBuilder`。 关键调用包括 `__init__`, `isinstance`, `super`。

### `LinearAttentionMetadataBuilder.build` method / `LinearAttentionMetadataBuilder.build` 方法
```python
    def build(
        self,
        common_prefix_len: int,
        common_attn_metadata: CommonAttentionMetadata,
        fast_build: bool = False,
    ) -> LinearAttentionMetadata:
        query_start_loc = common_attn_metadata.query_start_loc
        seq_lens = common_attn_metadata.seq_lens

        state_indices_tensor = mamba_get_block_table_tensor(
            common_attn_metadata.block_table_tensor,
            common_attn_metadata.seq_lens,
            self.kv_cache_spec,
            self.vllm_config.cache_config.mamba_cache_mode,
        )[:, 0]

        num_decodes, num_prefills, num_decode_tokens, num_prefill_tokens = (
            split_decodes_and_prefills(
                common_attn_metadata, decode_threshold=self.reorder_batch_threshold
            )
        )

        attn_metadata = LinearAttentionMetadata(
            num_prefills=num_prefills,
            num_prefill_tokens=num_prefill_tokens,
            num_decodes=num_decodes,
            num_decode_tokens=num_decode_tokens,
            query_start_loc=query_start_loc,
            seq_lens=seq_lens,
            state_indices_tensor=state_indices_tensor,
        )
        return attn_metadata
```
**EN:** This method builds derived structures within `LinearAttentionMetadataBuilder`. Key calls include `split_decodes_and_prefills`, `LinearAttentionMetadata`, `mamba_get_block_table_tensor`.
**CN:** 该方法会构建派生结构，其作用域位于`LinearAttentionMetadataBuilder`。 关键调用包括 `split_decodes_and_prefills`, `LinearAttentionMetadata`, `mamba_get_block_table_tensor`。

## Key Concepts / 关键概念
- `LinearAttentionBackend`: central class or interface in this module. / `LinearAttentionBackend`：本模块中的核心类或接口。
- `LinearAttentionMetadata`: central class or interface in this module. / `LinearAttentionMetadata`：本模块中的核心类或接口。
- `LinearAttentionMetadataBuilder`: central class or interface in this module. / `LinearAttentionMetadataBuilder`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.v1.attention.backend`, `vllm.v1.attention.backends.utils`, `vllm.v1.kv_cache_interface`
