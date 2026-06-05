# mamba2_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/attention/backends/mamba2_attn.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `compute_varlen_chunk_metadata`, `Mamba2AttentionBackend`, `Mamba2AttentionMetadata` for the V1 `attention/backends` subsystem. / 为 V1 的 `attention/backends` 子系统实现 `compute_varlen_chunk_metadata`, `Mamba2AttentionBackend`, `Mamba2AttentionMetadata`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import itertools
from dataclasses import dataclass, replace
from typing import Any

import torch

from vllm.config import VllmConfig
from vllm.v1.attention.backend import (
    AttentionBackend,
    CommonAttentionMetadata,
)
from vllm.v1.attention.backends.mamba_attn import (
    BaseMambaAttentionMetadata,
    BaseMambaAttentionMetadataBuilder,
)
from vllm.v1.kv_cache_interface import AttentionSpec
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `torch`, and internal vLLM modules such as `vllm.config`, `vllm.v1.attention.backend`, `vllm.v1.attention.backends.mamba_attn`, `vllm.v1.kv_cache_interface`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`torch` 等外部依赖，以及 `vllm.config`, `vllm.v1.attention.backend`, `vllm.v1.attention.backends.mamba_attn`, `vllm.v1.kv_cache_interface` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `compute_varlen_chunk_metadata` function / `compute_varlen_chunk_metadata` 函数
```python
def compute_varlen_chunk_metadata(
    query_start_loc: torch.Tensor,
    chunk_size: int,
) -> tuple[torch.Tensor, torch.Tensor, torch.Tensor]:
    """
    Build chunk-aligned, variable-length metadata used by Mamba2 SSD kernels.

    Given per-sequence cumulative token starts `query_start_loc` of shape [B+1]
    and a physical `chunk_size`, returns three tensors on the same device:
      - cu_chunk_seqlens:  (nchunks+1,) int32   exclusive prefix-sum of
        logical-chunk lengths (each logical chunk never crosses a sequence or
        physical-chunk boundary).
      - last_chunk_indices: (B,)       int32   index of the last logical chunk
        for each sequence (=-1 for empty sequences).
      - seq_idx_chunks:     (nchunks,) int32   sequence index for each logical
        chunk in order.

    This is intentionally lightweight and CPU-side; it mirrors the metadata
    produced by the V1 Mamba2 meta-data builder and is exported so tests
    (and other callers) can avoid duplicating the logic.
    """
    assert query_start_loc.ndim == 1, "query_start_loc must be 1-D [B+1]"
    assert int(query_start_loc[0].item()) == 0, "query_start_loc[0] must be 0"
    device = query_start_loc.device

    qsl64 = query_start_loc.to(torch.int64)
    starts = qsl64[:-1].tolist()
    ends = qsl64[1:].tolist()
    total = int(qsl64[-1].item())

    chunk_lens: list[int] = []
    seq_idx_chunks: list[int] = []
    last_chunk_indices: list[int] = [-1] * len(starts)

    for b, (s, e) in enumerate(zip(starts, ends)):
        if e <= s:
            # empty sequence
            continue
        pos = s
        while pos < e:
            # split at both sequence boundaries and physical chunk boundaries
            room = chunk_size - (pos % chunk_size)
            take = min(room, e - pos)
            chunk_lens.append(int(take))
            seq_idx_chunks.append(b)
            last_chunk_indices[b] = len(chunk_lens) - 1
            pos += take

    # Exclusive prefix sum over logical-chunk lengths
    if chunk_lens:
        cu_chunk_seqlens = torch.tensor(
            [0] + list(itertools.accumulate(chunk_lens)),
            device=device,
            dtype=torch.int32,
        )
        # Final boundary must equal total tokens
        assert int(cu_chunk_seqlens[-1].item()) == total
    else:
        cu_chunk_seqlens = torch.tensor([0], device=device, dtype=torch.int32)

    last_chunk_indices_t = (
        torch.tensor(last_chunk_indices, device=device, dtype=torch.int32)
        if len(starts) > 0
        else torch.empty((0,), device=device, dtype=torch.int32)
    )
    seq_idx_chunks_t = torch.tensor(seq_idx_chunks, device=device, dtype=torch.int32)
    return cu_chunk_seqlens, last_chunk_indices_t, seq_idx_chunks_t
```
**EN:** This function computes derived values within the module. The docstring frames it as: Build chunk-aligned, variable-length metadata used by Mamba2 SSD kernels. Key calls include `to`, `tolist`, `int`, `enumerate`, `tensor`, `item`. The control flow contains 3 branch(es) and 2 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会计算派生值，其作用域位于the module。 关键调用包括 `to`, `tolist`, `int`, `enumerate`, `tensor`, `item`。 控制流包含 3 个分支和 2 个循环，说明这里承担了较强的协调逻辑。

### `Mamba2AttentionBackend` class / `Mamba2AttentionBackend` 类
```python
class Mamba2AttentionBackend(AttentionBackend):
```
**EN:** Introduces the `Mamba2AttentionBackend` class on top of `AttentionBackend`. Core methods include `get_name`, `get_builder_cls`, `is_ssm`.
**CN:** 这里定义 `Mamba2AttentionBackend` 类，其基类包括 `AttentionBackend`。核心方法包括 `get_name`, `get_builder_cls`, `is_ssm`。

### `Mamba2AttentionBackend.get_name` method / `Mamba2AttentionBackend.get_name` 方法
```python
    @staticmethod
    def get_name() -> str:
        return "MAMBA2_ATTN"
```
**EN:** This method returns or derives a value within `Mamba2AttentionBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`Mamba2AttentionBackend`。

### `Mamba2AttentionBackend.get_builder_cls` method / `Mamba2AttentionBackend.get_builder_cls` 方法
```python
    @staticmethod
    def get_builder_cls() -> type["Mamba2AttentionMetadataBuilder"]:
        return Mamba2AttentionMetadataBuilder
```
**EN:** This method returns or derives a value within `Mamba2AttentionBackend`.
**CN:** 该方法会返回或推导一个值，其作用域位于`Mamba2AttentionBackend`。

### `Mamba2AttentionBackend.is_ssm` method / `Mamba2AttentionBackend.is_ssm` 方法
```python
    @classmethod
    def is_ssm(cls) -> bool:
        return True
```
**EN:** This method answers a boolean capability check within `Mamba2AttentionBackend`.
**CN:** 该方法会回答布尔能力判断，其作用域位于`Mamba2AttentionBackend`。

### `Mamba2AttentionMetadata` class / `Mamba2AttentionMetadata` 类
```python
@dataclass
class Mamba2AttentionMetadata(BaseMambaAttentionMetadata):
    prep_initial_states: bool = False
    chunk_size: int = 0

    # Chunk-related metadata (only for prefill)
    seq_idx_p: torch.Tensor | None = None
```
**EN:** Uses `@dataclass` to package related state for `Mamba2AttentionMetadata`. Typical fields include `prep_initial_states`, `chunk_size`, `seq_idx_p`.
**CN:** `Mamba2AttentionMetadata` 使用 `@dataclass` 打包相关状态。典型字段包括 `prep_initial_states`, `chunk_size`, `seq_idx_p`。

### `Mamba2AttentionMetadataBuilder` class / `Mamba2AttentionMetadataBuilder` 类
```python
class Mamba2AttentionMetadataBuilder(
    BaseMambaAttentionMetadataBuilder[Mamba2AttentionMetadata]
):
    metadata_cls = Mamba2AttentionMetadata
```
**EN:** Introduces the `Mamba2AttentionMetadataBuilder` class on top of `BaseMambaAttentionMetadataBuilder[Mamba2AttentionMetadata]`. Core methods include `__init__`, `build`.
**CN:** 这里定义 `Mamba2AttentionMetadataBuilder` 类，其基类包括 `BaseMambaAttentionMetadataBuilder[Mamba2AttentionMetadata]`。核心方法包括 `__init__`, `build`。

### `Mamba2AttentionMetadataBuilder.__init__` method / `Mamba2AttentionMetadataBuilder.__init__` 方法
```python
    def __init__(
        self,
        kv_cache_spec: AttentionSpec,
        layer_names: list[str],
        vllm_config: VllmConfig,
        device: torch.device,
    ):
        super().__init__(kv_cache_spec, layer_names, vllm_config, device)
        chunk_size = vllm_config.model_config.get_mamba_chunk_size()
        assert chunk_size is not None, (
            "chunk_size needs to be set in the model config for Mamba2 models"
        )
        self.chunk_size: int = chunk_size
```
**EN:** This method initializes the object state within `Mamba2AttentionMetadataBuilder`. Key calls include `__init__`, `get_mamba_chunk_size`, `super`. It touches state such as `chunk_size`.
**CN:** 该方法会初始化对象状态，其作用域位于`Mamba2AttentionMetadataBuilder`。 关键调用包括 `__init__`, `get_mamba_chunk_size`, `super`。 它会读写 `chunk_size` 等状态。

### `Mamba2AttentionMetadataBuilder.build` method / `Mamba2AttentionMetadataBuilder.build` 方法
```python
    def build(
        self,
        common_prefix_len: int,
        common_attn_metadata: CommonAttentionMetadata,
        fast_build: bool = False,
        **kwargs: Any,
    ) -> Mamba2AttentionMetadata:
        common = self._compute_common_metadata(
            common_attn_metadata,
            num_accepted_tokens=kwargs.get("num_accepted_tokens"),
            prev_last_scheduled_idx=kwargs.get("prev_last_scheduled_idx"),
        )

        seq_idx_p = None
        cu_chunk_seqlen_p = None
        last_chunk_indices_p = None
        prep_initial_states = False

        # Compute seq_idx for prefill only
        if common.num_prefills > 0:
            prep_initial_states = (
                torch.any(common.has_initial_states_p).item()
                if common.has_initial_states_p is not None
                else False
            )

            cu_chunk_seqlen_p, seq_idx_p, last_chunk_indices_p = (
                self._build_chunk_metadata_tensors(
                    self.chunk_size,
                    common,
                    common_attn_metadata,
                )
            )

        return replace(
            common,
            prep_initial_states=prep_initial_states,
            chunk_size=self.chunk_size,
            seq_idx_p=seq_idx_p,
            cu_chunk_seqlen_p=cu_chunk_seqlen_p,
            last_chunk_indices_p=last_chunk_indices_p,
        )
```
**EN:** This method builds derived structures within `Mamba2AttentionMetadataBuilder`. Key calls include `_compute_common_metadata`, `replace`, `_build_chunk_metadata_tensors`, `get`, `item`, `any`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会构建派生结构，其作用域位于`Mamba2AttentionMetadataBuilder`。 关键调用包括 `_compute_common_metadata`, `replace`, `_build_chunk_metadata_tensors`, `get`, `item`, `any`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `compute_varlen_chunk_metadata`: top-level helper or orchestration entry point. / `compute_varlen_chunk_metadata`：顶层辅助函数或编排入口。
- `Mamba2AttentionBackend`: central class or interface in this module. / `Mamba2AttentionBackend`：本模块中的核心类或接口。
- `Mamba2AttentionMetadata`: central class or interface in this module. / `Mamba2AttentionMetadata`：本模块中的核心类或接口。
- `Mamba2AttentionMetadataBuilder`: central class or interface in this module. / `Mamba2AttentionMetadataBuilder`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `itertools`, `dataclasses`, `typing`
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.v1.attention.backend`, `vllm.v1.attention.backends.mamba_attn`, `vllm.v1.kv_cache_interface`
