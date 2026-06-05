# ssm_conv_transfer_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/distributed/kv_transfer/kv_connector/v1/ssm_conv_transfer_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Mamba conv-state sub-projection decomposition for the 3-read transfer / 实现 v1 KV 传输连接器、接口或辅助模块。

## Line-by-Line Analysis / 逐行分析
### Module overview / 模块概览
```python
"""Mamba conv-state sub-projection decomposition for the 3-read transfer.

With DS conv state layout (dim, state_len), sub-projections are
contiguous in memory.  Each D rank reads its slices via 3 separate
RDMA transfers — no P-side permutation needed.

Supported model types:
  - Mamba2: conv = [x, B, C], temporal = (num_heads, head_dim)
  - GDN (Gated Delta Net): conv = [Q, K, V] (dim(Q)==dim(K)),
    temporal = (num_v_heads, v_dim, k_dim)
"""
```
**EN:** The opening docstring defines the module scope and high-level contract. It highlights: Mamba conv-state sub-projection decomposition for the 3-read transfer.
**CN:** 开头的文档字符串给出了模块范围与高层契约。 它重点概括了文件的职责、参与方以及主要接口。

### Imports and setup / 导入与初始化
```python
import math
from dataclasses import dataclass

import torch

from vllm.model_executor.layers.mamba.mamba_utils import is_conv_state_dim_first
from vllm.v1.attention.backends.registry import MambaAttentionBackendEnum
from vllm.v1.kv_cache_interface import MambaSpec
```
**EN:** This block imports `math`, `dataclasses`, `torch`, `vllm.model_executor.layers.mamba.mamba_utils`, `vllm.v1.attention.backends.registry`, `vllm.v1.kv_cache_interface` so the rest of the module can reuse the required runtime, typing, and helper APIs.
**CN:** 该代码块导入 `math`, `dataclasses`, `torch`, `vllm.model_executor.layers.mamba.mamba_utils`, `vllm.v1.attention.backends.registry`, `vllm.v1.kv_cache_interface`，为后续实现准备运行时、类型与辅助 API。

### Class `MambaConvSplitInfo` / 类 `MambaConvSplitInfo`
```python
@dataclass(frozen=True)
class MambaConvSplitInfo:
    """Per-rank byte sizes of the 3 conv sub-projections.

    Used by both P and D sides for NIXL descriptor registration.
    All fields are LOCAL to this engine's TP (already divided by TP size).

    DS memory layout within one page (contiguous):
      Mamba2: |-- x --|- B -|- C -|  (B == C)
      GDN:    |- Q -|- K -|-- V --|  (dim(Q)==dim(K), V may differ)
    """

    conv_rows: int  # conv_kernel - 1 (typically 3)
    local_proj_dims: tuple[int, int, int]  # per-rank column counts per sub-proj
    conv_dtype_size: int  # bytes per element (e.g. 2 for float16)
    ssm_sizes: tuple[int, int]  # (conv_state_bytes, ssm_state_bytes)

    @property
    def local_conv_dim(self) -> int:
        """Total conv columns per rank."""
        return sum(self.local_proj_dims)

    @property
    def proj_bytes(self) -> tuple[int, int, int]:
        """Byte sizes of the 3 sub-projections for one rank."""
        row_bytes = self.conv_rows * self.conv_dtype_size
        return tuple(d * row_bytes for d in self.local_proj_dims)  # type: ignore[return-value]

    @property
    def local_conv_offsets(self) -> list[tuple[int, int]]:
        """(byte_offset, byte_size) of each sub-projection within this
        engine's page.

        Used by both P and D for local descriptor registration.
        """
        conv0, conv1, conv2 = self.proj_bytes
        return [(0, conv0), (conv0, conv1), (conv0 + conv1, conv2)]

    def remote_conv_offsets(
        self, local_rank_offset: int, tp_ratio: int
    ) -> list[tuple[int, int]]:
        """(byte_offset, byte_size) of this D rank's sub-projection slices
        within one P page.

        Used by D side only, during remote descriptor registration.
# ... truncated for analysis ...
            # P-sized offsets. Scale down by |tp_ratio|.
            abs_ratio = -tp_ratio
            remote_conv0 = conv0 // abs_ratio
            remote_conv1 = conv1 // abs_ratio
            remote_conv2 = conv2 // abs_ratio
            return [
                (0, remote_conv0),
                (remote_conv0, remote_conv1),
                (remote_conv0 + remote_conv1, remote_conv2),
            ]
```
**EN:** Declares `MambaConvSplitInfo`, a class. Key methods include `local_conv_dim`, `proj_bytes`, `local_conv_offsets`, `remote_conv_offsets`. The docstring summarizes its role as: Per-rank byte sizes of the 3 conv sub-projections.
**CN:** 声明 `MambaConvSplitInfo`，它是一个类。 关键方法包括 `local_conv_dim`, `proj_bytes`, `local_conv_offsets`, `remote_conv_offsets`。 文档字符串概括了它在整体流程中的职责。

### Function `derive_mamba_conv_split` / 函数 `derive_mamba_conv_split`
```python
def derive_mamba_conv_split(
    mamba_spec: MambaSpec,
    local_tp: int,
) -> MambaConvSplitInfo:
    """Derive per-rank sub-projection byte sizes from a MambaSpec.

    Called once at init on both P and D.  Decomposes the conv dimension
    into its sub-projection parts based on the model type.

    Args:
        mamba_spec: MambaSpec whose shapes are:
            shapes[0] = conv state: (conv_dim_local, conv_rows) in DS layout.
            shapes[1] = temporal state (model-specific shape).
        local_tp: this engine's tensor-parallel size.

    Returns:
        MambaConvSplitInfo with per-rank sub-projection dims, conv_rows,
        conv_dtype_size, and ssm_sizes (conv_state_bytes, ssm_state_bytes).
    """
    _supported = (
        MambaAttentionBackendEnum.MAMBA2,
        MambaAttentionBackendEnum.GDN_ATTN,
    )
    if mamba_spec.mamba_type not in _supported:
        raise NotImplementedError(
            f"3-read conv transfer only supports Mamba2 and GDN models, "
            f"got mamba_type={mamba_spec.mamba_type!r}."
        )

    conv_shape = mamba_spec.shapes[0]
    assert len(conv_shape) == 2, f"Expected 2D conv state shape, got {conv_shape}"

    # NOTE (ZhanqiuHu): 3-read requires DS layout, which is already asserted
    # in nixl worker __init__.  Use it directly instead of heuristic detection.
    assert is_conv_state_dim_first(), "3-read requires DS conv state layout"
    local_conv_dim = conv_shape[0]  # DS: (conv_dim_local, conv_rows)
    conv_rows = conv_shape[1]

    conv_dtype_size = torch.tensor(
        [],
        dtype=mamba_spec.dtypes[0],  # type: ignore[misc]
    ).element_size()
    ssm_dtype_size = torch.tensor(
        [],
        dtype=mamba_spec.dtypes[1],  # type: ignore[misc]
# ... truncated for analysis ...
        raise NotImplementedError(
            f"Conv split not supported for mamba_type={mamba_spec.mamba_type!r}"
        )

    return MambaConvSplitInfo(
        conv_rows=conv_rows,
        local_proj_dims=local_proj_dims,
        conv_dtype_size=conv_dtype_size,
        ssm_sizes=(conv_state_bytes, ssm_state_bytes),
    )
```
**EN:** `derive_mamba_conv_split` implements a focused helper routine for this module. The docstring frames it as: Derive per-rank sub-projection byte sizes from a MambaSpec. It primarily works with arguments like `mamba_spec`, `local_tp`. Key calls include `is_conv_state_dim_first`, `torch.tensor().element_size`, `MambaConvSplitInfo`.
**CN:** `derive_mamba_conv_split` 实现了一个面向当前模块的辅助例程。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `mamba_spec`, `local_tp` 这样的参数。 关键调用包括 `is_conv_state_dim_first`, `torch.tensor().element_size`, `MambaConvSplitInfo`。

### Function `compute_physical_blocks_per_logical` / 函数 `compute_physical_blocks_per_logical`
```python
def compute_physical_blocks_per_logical(
    ssm_sizes: tuple[int, ...], block_len: int
) -> int:
    """Derive _physical_blocks_per_logical_kv_block from remote metadata.

    The remote engine's ratio is not sent directly in the handshake, so we
    reconstruct it: total mamba state per logical block / block_len.

    Args:
        ssm_sizes: (conv_state_bytes, ssm_state_bytes) from NixlAgentMetadata.
        block_len: the engine's block_len in bytes (from block_lens[0]).
    """
    return math.ceil((ssm_sizes[0] + ssm_sizes[1]) / block_len)
```
**EN:** `compute_physical_blocks_per_logical` is a thin wrapper around `math.ceil`, exposing that operation through a module-level helper. The docstring frames it as: Derive _physical_blocks_per_logical_kv_block from remote metadata. It primarily works with arguments like `ssm_sizes`, `block_len`. Key calls include `math.ceil`.
**CN:** `compute_physical_blocks_per_logical` 是对 `math.ceil` 的轻量封装，通过模块级辅助函数暴露该操作。 文档字符串进一步说明了它的职责与使用语义。 它主要处理诸如 `ssm_sizes`, `block_len` 这样的参数。 关键调用包括 `math.ceil`。

## Key Concepts / 关键概念
- KV cache movement across workers / 跨工作进程的 KV 缓存迁移
- `MambaConvSplitInfo`: class interface or data carrier / `MambaConvSplitInfo`：类接口或数据载体
- `derive_mamba_conv_split`: module-level helper or API entry / `derive_mamba_conv_split`：模块级辅助函数或 API 入口
- `compute_physical_blocks_per_logical`: module-level helper or API entry / `compute_physical_blocks_per_logical`：模块级辅助函数或 API 入口

## Dependencies / 依赖关系
- **Standard library / 标准库**: `math`, `dataclasses`
- **Third-party / 第三方**: `torch`
- **Internal modules / 内部模块**: `vllm.model_executor.layers.mamba.mamba_utils`, `vllm.v1.attention.backends.registry`, `vllm.v1.kv_cache_interface`
