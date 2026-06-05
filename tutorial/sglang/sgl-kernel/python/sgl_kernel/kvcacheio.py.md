# kvcacheio.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-kernel/python/sgl_kernel/kvcacheio.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module wraps native operators related to `KV-cache I/O`, performs Python-side validation, and exposes a convenient interface. / 该模块封装与 `KV-cache I/O` 相关的原生算子，执行 Python 侧校验，并暴露便捷接口。

## Line-by-Line Analysis / 逐行分析
### Lines 1-3: Imports and module setup
````python
from typing import List

import torch
````
**EN:** This block imports the libraries and symbols used later in the file, defining the module's dependency surface and startup context.
**CN:** 该部分导入后续代码所需的库与符号，定义模块的依赖面以及初始化上下文。

### Lines 6-7: `is_hip` definition
````python
def is_hip() -> bool:
    return torch.version.hip is not None
````
**EN:** This section defines `is_hip` and implements the core logic associated with is hip.
**CN:** 该部分定义 `is_hip`，并实现与 is hip 相关的核心逻辑。

### Lines 10-10: Constants and configuration
````python
_is_hip = is_hip()
````
**EN:** This block defines shared constants or configuration values such as `_is_hip`, which are reused by later functions or control flow.
**CN:** 该部分定义共享常量或配置项（例如 `_is_hip`），供后续函数或控制流程复用。

### Lines 13-34: `transfer_kv_per_layer` definition
````python
def transfer_kv_per_layer(
    src_k: torch.Tensor,
    dst_k: torch.Tensor,
    src_v: torch.Tensor,
    dst_v: torch.Tensor,
    src_indices: torch.Tensor,
    dst_indices: torch.Tensor,
    item_size: int,
    block_quota: int = 2,
    num_warps_per_block: int = 16 if _is_hip else 32,
):
    torch.ops.sgl_kernel.transfer_kv_per_layer.default(
        src_k,
        dst_k,
        src_v,
        dst_v,
        src_indices,
        dst_indices,
        item_size,
        block_quota,
        num_warps_per_block,
    )
````
**EN:** This section defines `transfer_kv_per_layer`. It transfers the `KV per layer` path used by the module. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `transfer_kv_per_layer`。它负责传输模块中与 `KV per layer` 相关的处理路径。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 37-62: `transfer_kv_per_layer_pf_lf` definition
````python
def transfer_kv_per_layer_pf_lf(
    src_k: torch.Tensor,
    dst_k: torch.Tensor,
    src_v: torch.Tensor,
    dst_v: torch.Tensor,
    src_indices: torch.Tensor,
    dst_indices: torch.Tensor,
    layer_id: int,
    item_size: int,
    src_layout_dim: int,
    block_quota: int = 2,
    num_warps_per_block: int = 16 if _is_hip else 32,
):
    torch.ops.sgl_kernel.transfer_kv_per_layer_pf_lf.default(
        src_k,
        dst_k,
        src_v,
        dst_v,
        src_indices,
        dst_indices,
        layer_id,
        item_size,
        src_layout_dim,
        block_quota,
        num_warps_per_block,
    )
````
**EN:** This section defines `transfer_kv_per_layer_pf_lf`. It transfers the `KV per layer pf lf` path used by the module. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `transfer_kv_per_layer_pf_lf`。它负责传输模块中与 `KV per layer pf lf` 相关的处理路径。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 65-94: `transfer_kv_per_layer_ph_lf` definition
````python
def transfer_kv_per_layer_ph_lf(
    src_k: torch.Tensor,
    dst_k: torch.Tensor,
    src_v: torch.Tensor,
    dst_v: torch.Tensor,
    src_indices: torch.Tensor,
    dst_indices: torch.Tensor,
    layer_id: int,
    item_size: int,
    src_layout_dim: int,
    page_size: int,
    head_num: int,
    block_quota: int = 2,
    num_warps_per_block: int = 16 if _is_hip else 32,
):
    torch.ops.sgl_kernel.transfer_kv_per_layer_ph_lf.default(
        src_k,
        dst_k,
        src_v,
        dst_v,
        src_indices,
        dst_indices,
        layer_id,
        item_size,
        src_layout_dim,
        page_size,
        head_num,
        block_quota,
        num_warps_per_block,
    )
````
**EN:** This section defines `transfer_kv_per_layer_ph_lf`. It transfers the `KV per layer ph lf` path used by the module. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `transfer_kv_per_layer_ph_lf`。它负责传输模块中与 `KV per layer ph lf` 相关的处理路径。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 97-120: `transfer_kv_all_layer` definition
````python
def transfer_kv_all_layer(
    src_k_layers: torch.Tensor,
    dst_k_layers: torch.Tensor,
    src_v_layers: torch.Tensor,
    dst_v_layers: torch.Tensor,
    src_indices: torch.Tensor,
    dst_indices: torch.Tensor,
    item_size: int,
    num_layers: int,
    block_quota: int = 2,
    num_warps_per_block: int = 16 if _is_hip else 32,
):
    torch.ops.sgl_kernel.transfer_kv_all_layer.default(
        src_k_layers,
        dst_k_layers,
        src_v_layers,
        dst_v_layers,
        src_indices,
        dst_indices,
        item_size,
        num_layers,
        block_quota,
        num_warps_per_block,
    )
````
**EN:** This section defines `transfer_kv_all_layer`. It transfers the `KV all layer` path used by the module. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `transfer_kv_all_layer`。它负责传输模块中与 `KV all layer` 相关的处理路径。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 123-148: `transfer_kv_all_layer_lf_pf` definition
````python
def transfer_kv_all_layer_lf_pf(
    src_k_layers: torch.Tensor,
    dst_k: torch.Tensor,
    src_v_layers: torch.Tensor,
    dst_v: torch.Tensor,
    src_indices: torch.Tensor,
    dst_indices: torch.Tensor,
    item_size: int,
    dst_layout_dim: int,
    num_layers: int,
    block_quota: int = 2,
    num_warps_per_block: int = 16 if _is_hip else 32,
):
    torch.ops.sgl_kernel.transfer_kv_all_layer_lf_pf.default(
        src_k_layers,
        dst_k,
        src_v_layers,
        dst_v,
        src_indices,
        dst_indices,
        item_size,
        dst_layout_dim,
        num_layers,
        block_quota,
        num_warps_per_block,
    )
````
**EN:** This section defines `transfer_kv_all_layer_lf_pf`. It transfers the `KV all layer lf pf` path used by the module. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `transfer_kv_all_layer_lf_pf`。它负责传输模块中与 `KV all layer lf pf` 相关的处理路径。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 151-180: `transfer_kv_all_layer_lf_ph` definition
````python
def transfer_kv_all_layer_lf_ph(
    src_k_layers: torch.Tensor,
    dst_k: torch.Tensor,
    src_v_layers: torch.Tensor,
    dst_v: torch.Tensor,
    src_indices: torch.Tensor,
    dst_indices: torch.Tensor,
    item_size: int,
    dst_layout_dim: int,
    num_layers: int,
    page_size: int,
    head_num: int,
    block_quota: int = 2,
    num_warps_per_block: int = 16 if _is_hip else 32,
):
    torch.ops.sgl_kernel.transfer_kv_all_layer_lf_ph.default(
        src_k_layers,
        dst_k,
        src_v_layers,
        dst_v,
        src_indices,
        dst_indices,
        item_size,
        dst_layout_dim,
        num_layers,
        page_size,
        head_num,
        block_quota,
        num_warps_per_block,
    )
````
**EN:** This section defines `transfer_kv_all_layer_lf_ph`. It transfers the `KV all layer lf ph` path used by the module. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `transfer_kv_all_layer_lf_ph`。它负责传输模块中与 `KV all layer lf ph` 相关的处理路径。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 183-192: `transfer_kv_direct` definition
````python
def transfer_kv_direct(
    src_layers: List[torch.Tensor],
    dst_layers: List[torch.Tensor],
    src_indices: torch.Tensor,
    dst_indices: torch.Tensor,
    page_size: int,
):
    torch.ops.sgl_kernel.transfer_kv_direct.default(
        src_layers, dst_layers, src_indices, dst_indices, page_size
    )
````
**EN:** This section defines `transfer_kv_direct`. It transfers the `KV direct` path used by the module. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `transfer_kv_direct`。它负责传输模块中与 `KV direct` 相关的处理路径。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 195-205: `transfer_kv_per_layer_direct_pf_lf` definition
````python
def transfer_kv_per_layer_direct_pf_lf(
    src_ptrs: List[torch.Tensor],
    dst_ptrs: List[torch.Tensor],
    src_indices: torch.Tensor,
    dst_indices: torch.Tensor,
    layer_id: int,
    page_size: int,
):
    torch.ops.sgl_kernel.transfer_kv_per_layer_direct_pf_lf.default(
        src_ptrs, dst_ptrs, src_indices, dst_indices, layer_id, page_size
    )
````
**EN:** This section defines `transfer_kv_per_layer_direct_pf_lf`. It transfers the `KV per layer direct pf lf` path used by the module. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `transfer_kv_per_layer_direct_pf_lf`。它负责传输模块中与 `KV per layer direct pf lf` 相关的处理路径。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 208-217: `transfer_kv_all_layer_direct_lf_pf` definition
````python
def transfer_kv_all_layer_direct_lf_pf(
    src_ptrs: List[torch.Tensor],
    dst_ptrs: List[torch.Tensor],
    src_indices: torch.Tensor,
    dst_indices: torch.Tensor,
    page_size: int,
):
    torch.ops.sgl_kernel.transfer_kv_all_layer_direct_lf_pf.default(
        src_ptrs, dst_ptrs, src_indices, dst_indices, page_size
    )
````
**EN:** This section defines `transfer_kv_all_layer_direct_lf_pf`. It transfers the `KV all layer direct lf pf` path used by the module. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `transfer_kv_all_layer_direct_lf_pf`。它负责传输模块中与 `KV all layer direct lf pf` 相关的处理路径。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 220-237: `transfer_kv_per_layer_mla` definition
````python
def transfer_kv_per_layer_mla(
    src: torch.Tensor,
    dst: torch.Tensor,
    src_indices: torch.Tensor,
    dst_indices: torch.Tensor,
    item_size: int,
    block_quota: int = 2,
    num_warps_per_block: int = 16 if _is_hip else 32,
):
    torch.ops.sgl_kernel.transfer_kv_per_layer_mla.default(
        src,
        dst,
        src_indices,
        dst_indices,
        item_size,
        block_quota,
        num_warps_per_block,
    )
````
**EN:** This section defines `transfer_kv_per_layer_mla`. It transfers the `KV per layer MLA` path used by the module. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `transfer_kv_per_layer_mla`。它负责传输模块中与 `KV per layer MLA` 相关的处理路径。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 240-261: `transfer_kv_per_layer_mla_pf_lf` definition
````python
def transfer_kv_per_layer_mla_pf_lf(
    src: torch.Tensor,
    dst: torch.Tensor,
    src_indices: torch.Tensor,
    dst_indices: torch.Tensor,
    layer_id: int,
    item_size: int,
    src_layout_dim: int,
    block_quota: int = 2,
    num_warps_per_block: int = 16 if _is_hip else 32,
):
    torch.ops.sgl_kernel.transfer_kv_per_layer_mla_pf_lf.default(
        src,
        dst,
        src_indices,
        dst_indices,
        layer_id,
        item_size,
        src_layout_dim,
        block_quota,
        num_warps_per_block,
    )
````
**EN:** This section defines `transfer_kv_per_layer_mla_pf_lf`. It transfers the `KV per layer MLA pf lf` path used by the module. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `transfer_kv_per_layer_mla_pf_lf`。它负责传输模块中与 `KV per layer MLA pf lf` 相关的处理路径。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 264-283: `transfer_kv_all_layer_mla` definition
````python
def transfer_kv_all_layer_mla(
    src_layers: torch.Tensor,
    dst_layers: torch.Tensor,
    src_indices: torch.Tensor,
    dst_indices: torch.Tensor,
    item_size: int,
    num_layers: int,
    block_quota: int = 2,
    num_warps_per_block: int = 16 if _is_hip else 32,
):
    torch.ops.sgl_kernel.transfer_kv_all_layer_mla.default(
        src_layers,
        dst_layers,
        src_indices,
        dst_indices,
        item_size,
        num_layers,
        block_quota,
        num_warps_per_block,
    )
````
**EN:** This section defines `transfer_kv_all_layer_mla`. It transfers the `KV all layer MLA` path used by the module. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `transfer_kv_all_layer_mla`。它负责传输模块中与 `KV all layer MLA` 相关的处理路径。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

### Lines 286-307: `transfer_kv_all_layer_mla_lf_pf` definition
````python
def transfer_kv_all_layer_mla_lf_pf(
    src_layers: torch.Tensor,
    dst: torch.Tensor,
    src_indices: torch.Tensor,
    dst_indices: torch.Tensor,
    item_size: int,
    dst_layout_dim: int,
    num_layers: int,
    block_quota: int = 2,
    num_warps_per_block: int = 16 if _is_hip else 32,
):
    torch.ops.sgl_kernel.transfer_kv_all_layer_mla_lf_pf.default(
        src_layers,
        dst,
        src_indices,
        dst_indices,
        item_size,
        dst_layout_dim,
        num_layers,
        block_quota,
        num_warps_per_block,
    )
````
**EN:** This section defines `transfer_kv_all_layer_mla_lf_pf`. It transfers the `KV all layer MLA lf pf` path used by the module. The implementation eventually dispatches into the compiled `torch.ops.sgl_kernel` namespace.
**CN:** 该部分定义 `transfer_kv_all_layer_mla_lf_pf`。它负责传输模块中与 `KV all layer MLA lf pf` 相关的处理路径。 实现最终会分派到已编译的 `torch.ops.sgl_kernel` 命名空间。

## Key Concepts / 关键概念
- **Role / 角色**: Python wrapper over compiled ops / 已编译算子的 Python 封装
- **Primary symbols / 主要符号**: `is_hip`, `transfer_kv_per_layer`, `transfer_kv_per_layer_pf_lf`, `transfer_kv_per_layer_ph_lf`, `transfer_kv_all_layer`, `transfer_kv_all_layer_lf_pf`, `transfer_kv_all_layer_lf_ph`, `transfer_kv_direct`
- **Core technologies / 核心技术**: compiled custom ops / 已编译自定义算子, PyTorch tensors / PyTorch 张量

## Dependencies / 依赖关系
- **Internal / 内部**: None obvious / 无明显内部依赖
- **External / 外部**: `torch`, `typing`
- **Runtime hooks / 运行时钩子**: `torch.ops.sgl_kernel` custom operator namespace
