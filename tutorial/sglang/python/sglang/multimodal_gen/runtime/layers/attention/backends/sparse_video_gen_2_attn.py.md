# sparse_video_gen_2_attn.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/attention/backends/sparse_video_gen_2_attn.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `SparseVideoGen2AttentionBackend`, `Svg2LayerCache`, and `Svg2Cache`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Sparse Video Gen 2 (SAP) attention backend. This is a baseline integration that wires the backend into the attention framework. / 该文件属于运行时算子层。它围绕 `SparseVideoGen2AttentionBackend`、`Svg2LayerCache` 和 `Svg2Cache` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: module setup and imports / 模块初始化与导入
```python
"""
Sparse Video Gen 2 (SAP) attention backend.

This is a baseline integration that wires the backend into the
attention framework.

Adapted from https://github.com/svg-project/Sparse-VideoGen/blob/main/svg/models/wan/attention.py
"""

from dataclasses import dataclass, field
from typing import Any

import torch
import torch.nn.functional as F
from torch.nn.attention import SDPBackend, sdpa_kernel
```
**EN:** This block establishes the module context and imports `dataclasses`, `typing`, `torch`, `torch.nn.functional`, and `torch.nn.attention`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `dataclasses`、`typing`、`torch`、`torch.nn.functional` 和 `torch.nn.attention`。这些依赖为后续实现提供所需符号。

### Lines 17-41: supporting statements / 辅助语句
```python
try:
    from svg.kernels.triton.permute import (
        apply_inverse_permutation_triton,
        permute_tensor_by_labels_triton,
    )
    from svg.kmeans_utils import (
        batch_kmeans_Euclid,
        dynamic_block_sparse_fwd_flashinfer,
        identify_dynamic_map,
    )

    svg2_available = True
except ImportError:
    svg2_available = False

from sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend import (
    AttentionBackend,
    AttentionImpl,
    AttentionMetadata,
    AttentionMetadataBuilder,
)
from sglang.multimodal_gen.runtime.platforms import AttentionBackendEnum
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger

logger = init_logger(__name__)
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 44-45: `SparseVideoGen2AttentionBackend` class overview / `SparseVideoGen2AttentionBackend` 类概览
```python
class SparseVideoGen2AttentionBackend(AttentionBackend):
```
**EN:** This block defines class `SparseVideoGen2AttentionBackend`. It encapsulates sparse video gen2 attention backend behavior. It inherits from `AttentionBackend`.
**CN:** 该代码块定义了类 `SparseVideoGen2AttentionBackend`。 它用于封装 sparse video gen2 attention backend 相关行为。 它继承自 `AttentionBackend`。

### Lines 46-46: supporting statements / 辅助语句
```python
    accept_output_buffer: bool = True
```
**EN:** This block gathers supporting statements inside `SparseVideoGen2AttentionBackend`. It updates names such as `accept_output_buffer`.
**CN:** 该代码块汇集了位于 `SparseVideoGen2AttentionBackend` 内部的辅助语句。 它会更新 `accept_output_buffer` 等名称。

### Lines 48-50: `get_supported_head_sizes` implementation / `get_supported_head_sizes` 实现
```python
    @staticmethod
    def get_supported_head_sizes() -> list[int]:
        return [64, 128, 256]
```
**EN:** This block defines method `get_supported_head_sizes` on `SparseVideoGen2AttentionBackend`. It retrieves supported head sizes.
**CN:** 该代码块定义了 `SparseVideoGen2AttentionBackend` 的方法 `get_supported_head_sizes`。 它用于获取supported head sizes。

### Lines 52-54: `get_enum` implementation / `get_enum` 实现
```python
    @staticmethod
    def get_enum() -> AttentionBackendEnum:
        return AttentionBackendEnum.SPARSE_VIDEO_GEN_2_ATTN
```
**EN:** This block defines method `get_enum` on `SparseVideoGen2AttentionBackend`. It retrieves enum.
**CN:** 该代码块定义了 `SparseVideoGen2AttentionBackend` 的方法 `get_enum`。 它用于获取enum。

### Lines 56-58: `get_impl_cls` implementation / `get_impl_cls` 实现
```python
    @staticmethod
    def get_impl_cls() -> type["SparseVideoGen2AttentionImpl"]:
        return SparseVideoGen2AttentionImpl
```
**EN:** This block defines method `get_impl_cls` on `SparseVideoGen2AttentionBackend`. It retrieves impl cls.
**CN:** 该代码块定义了 `SparseVideoGen2AttentionBackend` 的方法 `get_impl_cls`。 它用于获取impl cls。

### Lines 60-62: `get_metadata_cls` implementation / `get_metadata_cls` 实现
```python
    @staticmethod
    def get_metadata_cls() -> type["SparseVideoGen2AttentionMetadata"]:
        return SparseVideoGen2AttentionMetadata
```
**EN:** This block defines method `get_metadata_cls` on `SparseVideoGen2AttentionBackend`. It retrieves metadata cls.
**CN:** 该代码块定义了 `SparseVideoGen2AttentionBackend` 的方法 `get_metadata_cls`。 它用于获取metadata cls。

### Lines 64-66: `get_builder_cls` implementation / `get_builder_cls` 实现
```python
    @staticmethod
    def get_builder_cls() -> type["SparseVideoGen2AttentionMetadataBuilder"]:
        return SparseVideoGen2AttentionMetadataBuilder
```
**EN:** This block defines method `get_builder_cls` on `SparseVideoGen2AttentionBackend`. It retrieves builder cls.
**CN:** 该代码块定义了 `SparseVideoGen2AttentionBackend` 的方法 `get_builder_cls`。 它用于获取builder cls。

### Lines 70-71: `Svg2LayerCache` class overview / `Svg2LayerCache` 类概览
```python
class Svg2LayerCache:
    # centroids for kmeans clustering
```
**EN:** This block defines class `Svg2LayerCache`. It encapsulates svg2 layer cache behavior.
**CN:** 该代码块定义了类 `Svg2LayerCache`。 它用于封装 svg2 layer cache 相关行为。

### Lines 72-74: supporting statements / 辅助语句
```python
    q_centroids: torch.Tensor | None = None
    k_centroids: torch.Tensor | None = None
    centroids_initialized: bool = False
```
**EN:** This block gathers supporting statements inside `Svg2LayerCache`. It updates names such as `q_centroids`, `k_centroids`, and `centroids_initialized`.
**CN:** 该代码块汇集了位于 `Svg2LayerCache` 内部的辅助语句。 它会更新 `q_centroids`、`k_centroids` 和 `centroids_initialized` 等名称。

### Lines 78-78: `Svg2Cache` class overview / `Svg2Cache` 类概览
```python
class Svg2Cache:
```
**EN:** This block defines class `Svg2Cache`. It encapsulates svg2 cache behavior.
**CN:** 该代码块定义了类 `Svg2Cache`。 它用于封装 svg2 cache 相关行为。

### Lines 79-79: supporting statements / 辅助语句
```python
    layers: dict[int, Svg2LayerCache] = field(default_factory=dict)
```
**EN:** This block gathers supporting statements inside `Svg2Cache`. It updates names such as `layers`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `Svg2Cache` 内部的辅助语句。 它会更新 `layers` 等名称。 代码会与 `field` 协同工作。

### Lines 81-86: `get_layer` implementation / `get_layer` 实现
```python
    def get_layer(self, layer_idx: int) -> Svg2LayerCache:
        layer_cache = self.layers.get(layer_idx)
        if layer_cache is None:
            layer_cache = Svg2LayerCache()
            self.layers[layer_idx] = layer_cache
        return layer_cache
```
**EN:** This block defines method `get_layer` on `Svg2Cache`. It retrieves layer. Key calls include `self.layers.get`, and `Svg2LayerCache`. The implementation branches on conditions. Parameters such as `layer_idx` drive the behavior in this section.
**CN:** 该代码块定义了 `Svg2Cache` 的方法 `get_layer`。 它用于获取layer。 关键调用包括 `self.layers.get` 和 `Svg2LayerCache`。 实现中包含条件分支。 本段逻辑主要由 `layer_idx` 等参数驱动。

### Lines 90-90: `SparseVideoGen2AttentionMetadata` class overview / `SparseVideoGen2AttentionMetadata` 类概览
```python
class SparseVideoGen2AttentionMetadata(AttentionMetadata):
```
**EN:** This block defines class `SparseVideoGen2AttentionMetadata`. It encapsulates sparse video gen2 attention metadata behavior. It inherits from `AttentionMetadata`.
**CN:** 该代码块定义了类 `SparseVideoGen2AttentionMetadata`。 它用于封装 sparse video gen2 attention metadata 相关行为。 它继承自 `AttentionMetadata`。

### Lines 91-107: supporting statements / 辅助语句
```python
    current_timestep: int
    num_q_centroids: int
    num_k_centroids: int
    top_p_kmeans: float
    min_kc_ratio: float
    kmeans_iter_init: int
    kmeans_iter_step: int
    zero_step_kmeans_init: bool
    first_layers_fp: float
    first_times_fp: float
    context_length: int
    num_frame: int
    frame_size: int
    cache: Svg2Cache
    prompt_length: int | None = None
    max_seqlen_q: int | None = None
    max_seqlen_k: int | None = None
```
**EN:** This block gathers supporting statements inside `SparseVideoGen2AttentionMetadata`. It updates names such as `current_timestep`, `num_q_centroids`, `num_k_centroids`, `top_p_kmeans`, `min_kc_ratio`, and `kmeans_iter_init`.
**CN:** 该代码块汇集了位于 `SparseVideoGen2AttentionMetadata` 内部的辅助语句。 它会更新 `current_timestep`、`num_q_centroids`、`num_k_centroids`、`top_p_kmeans`、`min_kc_ratio` 和 `kmeans_iter_init` 等名称。

### Lines 110-115: `_require_kwarg` implementation / `_require_kwarg` 实现
```python
def _require_kwarg(kwargs: dict[str, Any], name: str) -> Any:
    if name not in kwargs:
        raise ValueError(
            f"Missing required argument for SparseVideoGen2Attention: {name}"
        )
    return kwargs[name]
```
**EN:** This block defines function `_require_kwarg`. It handles require kwarg logic. Key calls include `ValueError`. The implementation branches on conditions. Parameters such as `kwargs`, and `name` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_require_kwarg`。 它用于处理 require kwarg 相关逻辑。 关键调用包括 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `kwargs` 和 `name` 等参数驱动。

### Lines 118-119: `SparseVideoGen2AttentionMetadataBuilder` class overview / `SparseVideoGen2AttentionMetadataBuilder` 类概览
```python
class SparseVideoGen2AttentionMetadataBuilder(AttentionMetadataBuilder):
```
**EN:** This block defines class `SparseVideoGen2AttentionMetadataBuilder`. It encapsulates sparse video gen2 attention metadata builder behavior. It inherits from `AttentionMetadataBuilder`.
**CN:** 该代码块定义了类 `SparseVideoGen2AttentionMetadataBuilder`。 它用于封装 sparse video gen2 attention metadata builder 相关行为。 它继承自 `AttentionMetadataBuilder`。

### Lines 120-121: `__init__` implementation / `__init__` 实现
```python
    def __init__(self) -> None:
        pass
```
**EN:** This block defines method `__init__` on `SparseVideoGen2AttentionMetadataBuilder`. It initializes the instance state.
**CN:** 该代码块定义了 `SparseVideoGen2AttentionMetadataBuilder` 的方法 `__init__`。 它用于初始化实例状态。

### Lines 123-124: `prepare` implementation / `prepare` 实现
```python
    def prepare(self) -> None:
        pass
```
**EN:** This block defines method `prepare` on `SparseVideoGen2AttentionMetadataBuilder`. It prepares function.
**CN:** 该代码块定义了 `SparseVideoGen2AttentionMetadataBuilder` 的方法 `prepare`。 它用于准备函数。

### Lines 126-179: `build` implementation / `build` 实现
```python
    def build(  # type: ignore[override]
        self,
        current_timestep: int,
        raw_latent_shape: tuple[int, ...],
        patch_size: tuple[int, int, int],
        cache: Svg2Cache,
        num_q_centroids: int,
        num_k_centroids: int,
        top_p_kmeans: float,
        min_kc_ratio: float,
        kmeans_iter_init: int,
        kmeans_iter_step: int,
        zero_step_kmeans_init: bool,
        first_layers_fp: float,
        first_times_fp: float,
        context_length: int = 0,
        prompt_length: int | None = None,
        **kwargs: dict[str, Any],
    ) -> SparseVideoGen2AttentionMetadata:
        raw_shape = tuple(raw_latent_shape)
        if len(raw_shape) == 5:
            t, h, w = raw_shape[2:5]
        elif len(raw_shape) == 3:
            t, h, w = raw_shape
        else:
            raise ValueError(
                "raw_latent_shape must be (T, H, W) or (B, C, T, H, W) for SAP attention"
            )
        pt, ph, pw = patch_size
        if t % pt != 0 or h % ph != 0 or w % pw != 0:
            raise ValueError(
                "raw_latent_shape must be divisible by patch_size for SAP attention"
            )

        num_frame = t // pt
        frame_size = (h // ph) * (w // pw)

        return SparseVideoGen2AttentionMetadata(
            current_timestep=current_timestep,
            num_q_centroids=num_q_centroids,
            num_k_centroids=num_k_centroids,
            top_p_kmeans=top_p_kmeans,
            min_kc_ratio=min_kc_ratio,
            kmeans_iter_init=kmeans_iter_init,
            kmeans_iter_step=kmeans_iter_step,
            zero_step_kmeans_init=zero_step_kmeans_init,
            first_layers_fp=first_layers_fp,
            first_times_fp=first_times_fp,
            context_length=context_length,
            prompt_length=prompt_length,
            num_frame=num_frame,
            frame_size=frame_size,
            cache=cache,
        )
```
**EN:** This block defines method `build` on `SparseVideoGen2AttentionMetadataBuilder`. It builds function. Key calls include `tuple`, `SparseVideoGen2AttentionMetadata`, `len`, and `ValueError`. The implementation branches on conditions. Parameters such as `current_timestep`, `raw_latent_shape`, `patch_size`, `cache`, and `num_q_centroids` drive the behavior in this section.
**CN:** 该代码块定义了 `SparseVideoGen2AttentionMetadataBuilder` 的方法 `build`。 它用于构建函数。 关键调用包括 `tuple`、`SparseVideoGen2AttentionMetadata`、`len` 和 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `current_timestep`、`raw_latent_shape`、`patch_size`、`cache` 和 `num_q_centroids` 等参数驱动。

### Lines 182-183: `SparseVideoGen2AttentionImpl` class overview / `SparseVideoGen2AttentionImpl` 类概览
```python
class SparseVideoGen2AttentionImpl(AttentionImpl):
```
**EN:** This block defines class `SparseVideoGen2AttentionImpl`. It encapsulates sparse video gen2 attention impl behavior. It inherits from `AttentionImpl`.
**CN:** 该代码块定义了类 `SparseVideoGen2AttentionImpl`。 它用于封装 sparse video gen2 attention impl 相关行为。 它继承自 `AttentionImpl`。

### Lines 184-205: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        num_heads: int,
        head_size: int,
        causal: bool,
        softmax_scale: float,
        num_kv_heads: int | None = None,
        prefix: str = "",
        **extra_impl_args,
    ) -> None:
        if causal:
            raise ValueError(
                "Sparse Video Gen 2 attention does not support causal attention"
            )
        if not svg2_available:
            raise ImportError(
                "Sparse Video Gen 2 attention backend requires svg package to be installed"
                "Please install it by following the instructions at "
                "https://github.com/svg-project/Sparse-VideoGen"
            )
        self.prefix = prefix
        self.layer_idx = self._get_layer_idx(prefix)
```
**EN:** This block defines method `__init__` on `SparseVideoGen2AttentionImpl`. It initializes the instance state. Key calls include `self._get_layer_idx`, `ValueError`, and `ImportError`. The implementation branches on conditions. Parameters such as `num_heads`, `head_size`, `causal`, `softmax_scale`, and `num_kv_heads` drive the behavior in this section.
**CN:** 该代码块定义了 `SparseVideoGen2AttentionImpl` 的方法 `__init__`。 它用于初始化实例状态。 关键调用包括 `self._get_layer_idx`、`ValueError` 和 `ImportError`。 实现中包含条件分支。 本段逻辑主要由 `num_heads`、`head_size`、`causal`、`softmax_scale` 和 `num_kv_heads` 等参数驱动。

### Lines 207-213: `_get_layer_idx` implementation / `_get_layer_idx` 实现
```python
    def _get_layer_idx(self, prefix: str) -> int:
        parts = prefix.split(".")
        if len(parts) < 3:
            raise ValueError(
                f"Invalid prefix for SparseVideoGen2AttentionImpl: {prefix}"
            )
        return int(parts[-3])
```
**EN:** This block defines method `_get_layer_idx` on `SparseVideoGen2AttentionImpl`. It retrieves layer idx. Key calls include `prefix.split`, `int`, `len`, and `ValueError`. The implementation branches on conditions. Parameters such as `prefix` drive the behavior in this section.
**CN:** 该代码块定义了 `SparseVideoGen2AttentionImpl` 的方法 `_get_layer_idx`。 它用于获取layer idx。 关键调用包括 `prefix.split`、`int`、`len` 和 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `prefix` 等参数驱动。

### Lines 215-246: `kmeans_init` implementation / `kmeans_init` 实现
```python
    def kmeans_init(
        self,
        query: torch.Tensor,
        key: torch.Tensor,
        attn_metadata: SparseVideoGen2AttentionMetadata,
    ):
        cfg, num_heads, seq_len, dim = query.size()
        qlabels, qcentroids, qcluster_sizes, qiter = batch_kmeans_Euclid(
            query.reshape(cfg * num_heads, seq_len, dim),
            n_clusters=attn_metadata.num_q_centroids,
            max_iters=attn_metadata.kmeans_iter_init,
        )
        klabels, kcentroids, kcluster_sizes, kiter = batch_kmeans_Euclid(
            key.reshape(cfg * num_heads, seq_len, dim),
            n_clusters=attn_metadata.num_k_centroids,
            max_iters=attn_metadata.kmeans_iter_init,
        )

        layer_cache = attn_metadata.cache.get_layer(self.layer_idx)
        layer_cache.q_centroids = qcentroids
        layer_cache.k_centroids = kcentroids

        return (
            qlabels,
            qcentroids,
            qcluster_sizes,
            qiter,
            klabels,
            kcentroids,
            kcluster_sizes,
            kiter,
        )
```
**EN:** This block defines method `kmeans_init` on `SparseVideoGen2AttentionImpl`. It handles kmeans init logic. Key calls include `query.size`, `batch_kmeans_Euclid`, `attn_metadata.cache.get_layer`, `query.reshape`, and `key.reshape`. Parameters such as `query`, `key`, and `attn_metadata` drive the behavior in this section.
**CN:** 该代码块定义了 `SparseVideoGen2AttentionImpl` 的方法 `kmeans_init`。 它用于处理 kmeans init 相关逻辑。 关键调用包括 `query.size`、`batch_kmeans_Euclid`、`attn_metadata.cache.get_layer`、`query.reshape` 和 `key.reshape`。 本段逻辑主要由 `query`、`key` 和 `attn_metadata` 等参数驱动。

### Lines 248-281: `kmeans_step` implementation / `kmeans_step` 实现
```python
    def kmeans_step(
        self,
        query: torch.Tensor,
        key: torch.Tensor,
        attn_metadata: SparseVideoGen2AttentionMetadata,
    ):
        cfg, num_heads, seq_len, dim = query.size()
        layer_cache = attn_metadata.cache.get_layer(self.layer_idx)
        qlabels, qcentroids, qcluster_sizes, qiter = batch_kmeans_Euclid(
            query.reshape(cfg * num_heads, seq_len, dim),
            n_clusters=attn_metadata.num_q_centroids,
            max_iters=attn_metadata.kmeans_iter_step,
            init_centroids=layer_cache.q_centroids,
        )
        klabels, kcentroids, kcluster_sizes, kiter = batch_kmeans_Euclid(
            key.reshape(cfg * num_heads, seq_len, dim),
            n_clusters=attn_metadata.num_k_centroids,
            max_iters=attn_metadata.kmeans_iter_step,
            init_centroids=layer_cache.k_centroids,
        )

        layer_cache.q_centroids = qcentroids
        layer_cache.k_centroids = kcentroids

        return (
            qlabels,
            qcentroids,
            qcluster_sizes,
            qiter,
            klabels,
            kcentroids,
            kcluster_sizes,
            kiter,
        )
```
**EN:** This block defines method `kmeans_step` on `SparseVideoGen2AttentionImpl`. It handles kmeans step logic. Key calls include `query.size`, `attn_metadata.cache.get_layer`, `batch_kmeans_Euclid`, `query.reshape`, and `key.reshape`. Parameters such as `query`, `key`, and `attn_metadata` drive the behavior in this section.
**CN:** 该代码块定义了 `SparseVideoGen2AttentionImpl` 的方法 `kmeans_step`。 它用于处理 kmeans step 相关逻辑。 关键调用包括 `query.size`、`attn_metadata.cache.get_layer`、`batch_kmeans_Euclid`、`query.reshape` 和 `key.reshape`。 本段逻辑主要由 `query`、`key` 和 `attn_metadata` 等参数驱动。

### Lines 283-328: `kmeans_clustering` implementation / `kmeans_clustering` 实现
```python
    def kmeans_clustering(
        self,
        query: torch.Tensor,
        key: torch.Tensor,
        attn_metadata: SparseVideoGen2AttentionMetadata,
    ):
        layer_cache = attn_metadata.cache.get_layer(self.layer_idx)
        if not layer_cache.centroids_initialized:
            (
                qlabels,
                qcentroids,
                qcluster_sizes,
                qiter,
                klabels,
                kcentroids,
                kcluster_sizes,
                kiter,
            ) = self.kmeans_init(query, key, attn_metadata)
            layer_cache.centroids_initialized = True
            logger.debug(
                "Centroids initialized at layer %s (init iters: %s).",
                self.layer_idx,
                attn_metadata.kmeans_iter_init,
            )
        else:
            (
                qlabels,
                qcentroids,
                qcluster_sizes,
                qiter,
                klabels,
                kcentroids,
                kcluster_sizes,
                kiter,
            ) = self.kmeans_step(query, key, attn_metadata)

        return (
            qlabels,
            qcentroids,
            qcluster_sizes,
            qiter,
            klabels,
            kcentroids,
            kcluster_sizes,
            kiter,
        )
```
**EN:** This block defines method `kmeans_clustering` on `SparseVideoGen2AttentionImpl`. It handles kmeans clustering logic. Key calls include `attn_metadata.cache.get_layer`, `self.kmeans_init`, `logger.debug`, and `self.kmeans_step`. The implementation branches on conditions. Parameters such as `query`, `key`, and `attn_metadata` drive the behavior in this section.
**CN:** 该代码块定义了 `SparseVideoGen2AttentionImpl` 的方法 `kmeans_clustering`。 它用于处理 kmeans clustering 相关逻辑。 关键调用包括 `attn_metadata.cache.get_layer`、`self.kmeans_init`、`logger.debug` 和 `self.kmeans_step`。 实现中包含条件分支。 本段逻辑主要由 `query`、`key` 和 `attn_metadata` 等参数驱动。

### Lines 330-387: `semantic_aware_permutation` implementation / `semantic_aware_permutation` 实现
```python
    def semantic_aware_permutation(
        self,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        attn_metadata: SparseVideoGen2AttentionMetadata,
    ):
        cfg, num_heads, seq_len, dim = query.size()

        # 1. Kmeans clustering
        (
            qlabels,
            qcentroids,
            qcluster_sizes,
            qiter,
            klabels,
            kcentroids,
            kcluster_sizes,
            kiter,
        ) = self.kmeans_clustering(query, key, attn_metadata)

        # 2. Identify dynamic map
        q_cluster_sizes = qcluster_sizes.view(
            cfg, num_heads, attn_metadata.num_q_centroids
        )
        k_cluster_sizes = kcluster_sizes.view(
            cfg, num_heads, attn_metadata.num_k_centroids
        )

        dynamic_map = identify_dynamic_map(
            qcentroids.view(cfg, num_heads, attn_metadata.num_q_centroids, dim),
            kcentroids.view(cfg, num_heads, attn_metadata.num_k_centroids, dim),
            q_cluster_sizes,
            k_cluster_sizes,
            attn_metadata.top_p_kmeans,
            attn_metadata.min_kc_ratio,
        )

        # 3. Permute the query, key, value
        q_permuted, q_sorted_indices = permute_tensor_by_labels_triton(
            query, qlabels, dim=2
        )
        k_permuted, k_sorted_indices = permute_tensor_by_labels_triton(
            key, klabels, dim=2
        )
        v_permuted, v_sorted_indices = permute_tensor_by_labels_triton(
            value, klabels, dim=2, sorted_indices=k_sorted_indices
        )

        return (
            q_permuted,
            k_permuted,
            v_permuted,
            dynamic_map,
            q_cluster_sizes,
            k_cluster_sizes,
            q_sorted_indices,
        )
```
**EN:** This block defines method `semantic_aware_permutation` on `SparseVideoGen2AttentionImpl`. It handles semantic aware permutation logic. Key calls include `query.size`, `self.kmeans_clustering`, `qcluster_sizes.view`, `kcluster_sizes.view`, and `identify_dynamic_map`. Parameters such as `query`, `key`, `value`, and `attn_metadata` drive the behavior in this section.
**CN:** 该代码块定义了 `SparseVideoGen2AttentionImpl` 的方法 `semantic_aware_permutation`。 它用于处理 semantic aware permutation 相关逻辑。 关键调用包括 `query.size`、`self.kmeans_clustering`、`qcluster_sizes.view`、`kcluster_sizes.view` 和 `identify_dynamic_map`。 本段逻辑主要由 `query`、`key`、`value` 和 `attn_metadata` 等参数驱动。

### Lines 389-438: `_hunyuan_dynamic_map_post_processing` implementation / `_hunyuan_dynamic_map_post_processing` 实现
```python
    def _hunyuan_dynamic_map_post_processing(
        self,
        q_perm: torch.Tensor,
        k_perm: torch.Tensor,
        v_perm: torch.Tensor,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        dyn_map: torch.Tensor,
        qc_sz_s: torch.Tensor,
        kc_sz_s: torch.Tensor,
        q_sorted_indices: torch.Tensor,
        video_length: int,
        context_length: int,
        prompt_length: int,
        unprompt_length: int,
    ) -> tuple[
        torch.Tensor,
        torch.Tensor,
        torch.Tensor,
        torch.Tensor,
        torch.Tensor,
        torch.Tensor,
        torch.Tensor,
    ]:
        # Place the permuted video tokens back and keep text tokens at the tail.
        query[:, :, :-context_length, :] = q_perm
        key[:, :, :-context_length, :] = k_perm
        value[:, :, :-context_length, :] = v_perm

        # Add prompt/unprompt clusters to the dynamic map.
        dyn_map = F.pad(dyn_map, (0, 2, 0, 2), value=0)
        dyn_map[:, :, -2, :-1] = True
        dyn_map[:, :, :-1, -2] = True
        dyn_map[:, :, -1, -1] = True

        qc_sz_s = F.pad(qc_sz_s, (0, 2), value=0)
        qc_sz_s[:, :, -2] = prompt_length
        qc_sz_s[:, :, -1] = unprompt_length
        kc_sz_s = F.pad(kc_sz_s, (0, 2), value=0)
        kc_sz_s[:, :, -2] = prompt_length
        kc_sz_s[:, :, -1] = unprompt_length

        q_sorted_indices = F.pad(q_sorted_indices, (0, context_length), value=0)
        q_sorted_indices[:, video_length:] = torch.arange(
            video_length,
            video_length + context_length,
            device=q_sorted_indices.device,
        )
        return query, key, value, dyn_map, qc_sz_s, kc_sz_s, q_sorted_indices
```
**EN:** This block defines method `_hunyuan_dynamic_map_post_processing` on `SparseVideoGen2AttentionImpl`. It handles hunyuan dynamic map post processing logic. Key calls include `F.pad`, and `torch.arange`. Parameters such as `q_perm`, `k_perm`, `v_perm`, `query`, and `key` drive the behavior in this section.
**CN:** 该代码块定义了 `SparseVideoGen2AttentionImpl` 的方法 `_hunyuan_dynamic_map_post_processing`。 它用于处理 hunyuan dynamic map post processing 相关逻辑。 关键调用包括 `F.pad` 和 `torch.arange`。 本段逻辑主要由 `q_perm`、`k_perm`、`v_perm`、`query` 和 `key` 等参数驱动。

### Lines 440-562: `forward` implementation / `forward` 实现
```python
    def forward(
        self,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        attn_metadata: SparseVideoGen2AttentionMetadata,
    ) -> torch.Tensor:
        torch.backends.cuda.preferred_linalg_library(backend="magma")
        res = None
        # bshd -> bhsd
        query = query.transpose(1, 2).contiguous()
        key = key.transpose(1, 2).contiguous()
        value = value.transpose(1, 2).contiguous()
        batch_size, num_heads, seq_len, dim = query.size()

        context_length, num_frame, frame_size = (
            attn_metadata.context_length,
            attn_metadata.num_frame,
            attn_metadata.frame_size,
        )
        prompt_length = attn_metadata.prompt_length
        if prompt_length is None:
            prompt_length = context_length

        assert (
            seq_len == context_length + num_frame * frame_size
        ), f"Query Shape: {seq_len} is not equivalent to {context_length} + {num_frame} * {frame_size}"

        # Determine if we use Full Attention to calculate
        full_attention_flag = False

        if self.layer_idx < attn_metadata.first_layers_fp:
            full_attention_flag = True
        if attn_metadata.current_timestep > attn_metadata.first_times_fp:
            full_attention_flag = True

        if full_attention_flag:
            if attn_metadata.zero_step_kmeans_init:
                video_length = attn_metadata.num_frame * attn_metadata.frame_size
                query_video = query[:, :, :video_length, :].contiguous()
                key_video = key[:, :, :video_length, :].contiguous()
                self.kmeans_clustering(query_video, key_video, attn_metadata)

            with sdpa_kernel(
                SDPBackend.CUDNN_ATTENTION
            ):  # not sure why we need to force cudnn here, but it's faster than flash attention
                output_hidden_states = torch.nn.functional.scaled_dot_product_attention(
                    query, key, value, dropout_p=0.0, is_causal=False
                )

            res = output_hidden_states.reshape(
                batch_size, num_heads, seq_len, dim
            ).transpose(1, 2)
        else:
            if context_length > 0:
                video_length = num_frame * frame_size
                unprompt_length = max(context_length - prompt_length, 0)
                query_video = query[:, :, :video_length, :].contiguous()
                key_video = key[:, :, :video_length, :].contiguous()
                value_video = value[:, :, :video_length, :].contiguous()

                (
                    q_perm,
                    k_perm,
                    v_perm,
                    dyn_map,
                    qc_sz_s,
                    kc_sz_s,
                    q_sorted_indices,
                ) = self.semantic_aware_permutation(
                    query_video, key_video, value_video, attn_metadata
                )
                (
                    q_perm,
                    k_perm,
                    v_perm,
                    dyn_map,
                    qc_sz_s,
                    kc_sz_s,
                    q_sorted_indices,
                ) = self._hunyuan_dynamic_map_post_processing(
                    q_perm,
                    k_perm,
                    v_perm,
                    query,
                    key,
                    value,
                    dyn_map,
                    qc_sz_s,
                    kc_sz_s,
                    q_sorted_indices,
                    video_length,
                    context_length,
                    prompt_length,
                    unprompt_length,
                )
            else:
                (
                    q_perm,
                    k_perm,
                    v_perm,
                    dyn_map,
                    qc_sz_s,
                    kc_sz_s,
                    q_sorted_indices,
                ) = self.semantic_aware_permutation(query, key, value, attn_metadata)

            output_permuted = dynamic_block_sparse_fwd_flashinfer(
                q_perm, k_perm, v_perm, dyn_map, qc_sz_s, kc_sz_s, is_cpu=False
            )

            attn_output = apply_inverse_permutation_triton(
                output_permuted, q_sorted_indices, dim=2
            )

            res = attn_output.reshape(batch_size, num_heads, seq_len, dim).transpose(
                1, 2
            )

        torch.backends.cuda.preferred_linalg_library(
            backend="default"
        )  # reset to default
        return res.contiguous()
```
**EN:** This block defines method `forward` on `SparseVideoGen2AttentionImpl`. It executes function. Key calls include `torch.backends.cuda.preferred_linalg_library`, `query.transpose.contiguous`, `key.transpose.contiguous`, `value.transpose.contiguous`, and `query.size`. The implementation branches on conditions, uses context-managed resources. Parameters such as `query`, `key`, `value`, and `attn_metadata` drive the behavior in this section.
**CN:** 该代码块定义了 `SparseVideoGen2AttentionImpl` 的方法 `forward`。 它用于执行前向计算函数。 关键调用包括 `torch.backends.cuda.preferred_linalg_library`、`query.transpose.contiguous`、`key.transpose.contiguous`、`value.transpose.contiguous` 和 `query.size`。 实现中包含条件分支，使用上下文管理资源。 本段逻辑主要由 `query`、`key`、`value` 和 `attn_metadata` 等参数驱动。

## Key Concepts / 关键概念
- `SparseVideoGen2AttentionBackend`: Primary class that encapsulates sparse video gen2 attention backend behavior. / 核心类，用于封装 sparse video gen2 attention backend 相关行为。
- `Svg2LayerCache`: Primary class that encapsulates svg2 layer cache behavior. / 核心类，用于封装 svg2 layer cache 相关行为。
- `Svg2Cache`: Primary class that encapsulates svg2 cache behavior. / 核心类，用于封装 svg2 cache 相关行为。
- `SparseVideoGen2AttentionMetadata`: Primary class that encapsulates sparse video gen2 attention metadata behavior. / 核心类，用于封装 sparse video gen2 attention metadata 相关行为。
- `_require_kwarg`: Top-level function that handles require kwarg logic. / 顶层函数，用于处理 require kwarg 相关逻辑。
- `SparseVideoGen2AttentionMetadataBuilder`: Primary class that encapsulates sparse video gen2 attention metadata builder behavior. / 核心类，用于封装 sparse video gen2 attention metadata builder 相关行为。
- `SparseVideoGen2AttentionImpl`: Primary class that encapsulates sparse video gen2 attention impl behavior. / 核心类，用于封装 sparse video gen2 attention impl 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `dataclasses`, `typing`
- **Third-party / 第三方依赖**: `torch`, `torch.nn.functional`, `torch.nn.attention`, `svg.kernels.triton.permute`, `svg.kmeans_utils`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.layers.attention.backends.attention_backend`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.runtime.utils.logging_utils`

- **Total lines / 总行数**: 562
