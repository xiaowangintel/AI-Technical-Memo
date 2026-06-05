# mm_encoder_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/attention/mm_encoder_attention.py`
- **Repository**: vllm-project/vllm
- **Purpose**: attention execution, backend dispatch, and KV-cache handling / 注意力执行、后端分派与 KV 缓存处理

## Line-by-Line Analysis / 逐行分析
### Lines 4-40 — imports and setup
```python
import functools
import json

import numpy as np
import torch

from vllm.config import MultiModalConfig
from vllm.kernels.triton.qkv_padded_fp8_quant import (
    quantize_fp8_maybe_pad_head_dim,
)
from vllm.logger import init_logger
from vllm.model_executor.custom_op import CustomOp, maybe_get_oot_by_class
from vllm.model_executor.layers.quantization.input_quant_fp8 import (
    QuantFP8,
)
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    GroupShape,
    get_fp8_min_max,
)
from vllm.model_executor.models.vision import (
    get_multimodal_config,
    get_vit_attn_backend,
)
from vllm.utils.flashinfer import (
    is_flashinfer_cudnn_fp8_prefill_attn_supported,
)
from vllm.utils.math_utils import round_up
from vllm.v1.attention.backends.fa_utils import get_flash_attn_version
from vllm.v1.attention.backends.registry import AttentionBackendEnum
from vllm.v1.attention.ops.vit_attn_wrappers import (
    vit_flash_attn_wrapper,
    vit_flashinfer_wrapper,
    vit_torch_sdpa_wrapper,
    vit_triton_attn_wrapper,
)

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。

### Lines 43-43 — constant `_FP8_AMAX_HISTORY_LEN`
```python
_FP8_AMAX_HISTORY_LEN = 16
```
**EN:** This constant defines `_FP8_AMAX_HISTORY_LEN`. This block defines module-level constants or shared state used later in execution.
**CN:** 该模块级常量/状态定义 `_FP8_AMAX_HISTORY_LEN`。 该代码块定义了后续执行会复用的模块级常量或共享状态。

### Lines 54-96 — function `_load_fp8_scales_file`
```python
@functools.cache
def _load_fp8_scales_file(path: str | None) -> dict[str, dict[str, float]]:
    """Load per-layer FP8 Q/K/V scales from a JSON file. Results are cached.

    Expected format (keys ``q_scale`` / ``k_scale`` / ``v_scale`` also accepted)::

        {
            "visual.blocks.0.attn.attn": {"q": 224.0, "k": 198.0, "v": 210.0},
            "visual.blocks.1.attn.attn": {"q": 218.0, "k": 195.0, "v": 207.0},
        }

    To produce such a file, run with ``mm_encoder_fp8_scale_save_path`` set.
    """
    if path is None:
        return {}

    with open(path, encoding="utf-8") as f:
        data = json.load(f)

    # Handle nested "layers" format
    if "layers" in data and isinstance(data["layers"], dict):
        data = data["layers"]

    scales: dict[str, dict[str, float]] = {}
    for layer_name, layer_scales in data.items():
        if not isinstance(layer_scales, dict):
            continue
        q = layer_scales.get("q", layer_scales.get("q_scale"))
        k = layer_scales.get("k", layer_scales.get("k_scale"))
        v = layer_scales.get("v", layer_scales.get("v_scale"))
        if q is not None and k is not None and v is not None:
            q_f, k_f, v_f = float(q), float(k), float(v)
            if q_f <= 0 or k_f <= 0 or v_f <= 0:
                raise ValueError(
                    f"FP8 scales must be positive, got q={q_f}, "
                    f"k={k_f}, v={v_f} for layer '{layer_name}'"
                )
            scales[layer_name] = {"q": q_f, "k": k_f, "v": v_f}

    logger.info_once(
        "Loaded FP8 attention scales from %s (%d layers)", path, len(scales)
    )
    return scales
```
**EN:** This function defines `_load_fp8_scales_file`. Load per-layer FP8 Q/K/V scales from a JSON file. The main inputs are `path`. Key calls include `data.items`, `logger.info_once`, `open`, `json.load`, `isinstance`, `layer_scales.get`. It writes or updates `scales`, `data`, `q`, `k`, `v`, `q_f`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该函数定义 `_load_fp8_scales_file`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `path`。 关键调用包括 `data.items`, `logger.info_once`, `open`, `json.load`, `isinstance`, `layer_scales.get`。 它会写入或更新 `scales`, `data`, `q`, `k`, `v`, `q_f`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 99-141 — function `_maybe_save_fp8_scales`
```python
def _maybe_save_fp8_scales(
    layer_name: str,
    q_scale: torch.Tensor,
    k_scale: torch.Tensor,
    v_scale: torch.Tensor,
    buffer_wrapped: bool,
) -> None:
    """Accumulate a layer's scale tensors; on the first amax buffer wrap,
    dump all accumulated scales to ``mm_encoder_fp8_scale_save_path``.

    No-op unless auto-save is configured. Tensor references are stored on
    every call (no GPU->CPU sync); ``.item()`` is only called at the single
    save point to avoid stalling the forward path.
    """
    global _fp8_scale_save_path
    # Fast path: auto-save either disabled or already finished. Path is
    # captured at layer init and cleared once the save fires.
    if _fp8_scale_save_path is None:
        return

    # Stash scale tensor refs (no GPU->CPU sync yet); wait until the amax
    # history has seen a full cycle before committing scales to disk.
    _fp8_saved_scale_refs[layer_name] = (q_scale, k_scale, v_scale)
    if not buffer_wrapped:
        return

    # Buffer just wrapped for the first time: materialize scales (with
    # safety margin) and dump to disk. Clearing _fp8_scale_save_path
    # makes this a one-shot across all layers.
    path, margin = _fp8_scale_save_path, _fp8_scale_save_margin
    scales = {
        name: {
            "q": q.item() * margin,
            "k": k.item() * margin,
            "v": v.item() * margin,
        }
        for name, (q, k, v) in _fp8_saved_scale_refs.items()
    }
    _fp8_scale_save_path = None
    _fp8_saved_scale_refs.clear()
    with open(path, "w", encoding="utf-8") as f:
        json.dump(scales, f, indent=2)
    logger.info("Saved FP8 scales (%d layers) to %s", len(scales), path)
```
**EN:** This function defines `_maybe_save_fp8_scales`. Accumulate a layer's scale tensors; on the first amax buffer wrap, dump all accumulated scales to ``mm_encoder_fp8_scale_save_path``. The main inputs are `layer_name`, `q_scale`, `k_scale`, `v_scale`, `buffer_wrapped`. Key calls include `_fp8_saved_scale_refs.clear`, `logger.info`, `open`, `json.dump`, `len`, `_fp8_saved_scale_refs.items`. It writes or updates `path`, `margin`, `scales`, `_fp8_scale_save_path`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_maybe_save_fp8_scales`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `layer_name`, `q_scale`, `k_scale`, `v_scale`, `buffer_wrapped`。 关键调用包括 `_fp8_saved_scale_refs.clear`, `logger.info`, `open`, `json.dump`, `len`, `_fp8_saved_scale_refs.items`。 它会写入或更新 `path`, `margin`, `scales`, `_fp8_scale_save_path`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 156-156 — constant `FLASHINFER_BATCH_BUCKETS`
```python
FLASHINFER_BATCH_BUCKETS = [8, 16, 32, 64]
```
**EN:** This constant defines `FLASHINFER_BATCH_BUCKETS`. This block defines module-level constants or shared state used later in execution.
**CN:** 该模块级常量/状态定义 `FLASHINFER_BATCH_BUCKETS`。 该代码块定义了后续执行会复用的模块级常量或共享状态。

### Lines 157-166 — constant `FLASHINFER_MAX_SEQLEN_BUCKETS`
```python
FLASHINFER_MAX_SEQLEN_BUCKETS = [
    1 * 1024,
    2 * 1024,
    4 * 1024,
    8 * 1024,
    16 * 1024,
    32 * 1024,
    64 * 1024,
    128 * 1024,
]
```
**EN:** This constant defines `FLASHINFER_MAX_SEQLEN_BUCKETS`. This block defines module-level constants or shared state used later in execution.
**CN:** 该模块级常量/状态定义 `FLASHINFER_MAX_SEQLEN_BUCKETS`。 该代码块定义了后续执行会复用的模块级常量或共享状态。

### Lines 169-169 — constant `FLASHINFER_CUDNN_WORKSPACE_SIZE_BYTES`
```python
FLASHINFER_CUDNN_WORKSPACE_SIZE_BYTES = 128 * 1024 * 1024
```
**EN:** This constant defines `FLASHINFER_CUDNN_WORKSPACE_SIZE_BYTES`. This block defines module-level constants or shared state used later in execution.
**CN:** 该模块级常量/状态定义 `FLASHINFER_CUDNN_WORKSPACE_SIZE_BYTES`。 该代码块定义了后续执行会复用的模块级常量或共享状态。

### Lines 173-181 — function `_get_flashinfer_workspace_buffer`
```python
def _get_flashinfer_workspace_buffer() -> torch.Tensor:
    global _flashinfer_workspace_buffer
    if _flashinfer_workspace_buffer is None:
        _flashinfer_workspace_buffer = torch.zeros(
            FLASHINFER_CUDNN_WORKSPACE_SIZE_BYTES,
            dtype=torch.uint8,
            device="cuda",
        )
    return _flashinfer_workspace_buffer
```
**EN:** This function defines `_get_flashinfer_workspace_buffer`. It handles attention-specific state, metadata, or tensor transforms. Key calls include `torch.zeros`. It writes or updates `_flashinfer_workspace_buffer`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_get_flashinfer_workspace_buffer`。 它处理注意力相关的状态、元数据或张量变换。 关键调用包括 `torch.zeros`。 它会写入或更新 `_flashinfer_workspace_buffer`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 184-200 — function `add_padding_to_seqlens`
```python
def add_padding_to_seqlens(
    seq: np.ndarray,
    batch_size: int,
    padding_value: int,
) -> np.ndarray:
    batch_size_padded = next(
        (b for b in FLASHINFER_BATCH_BUCKETS if b >= batch_size),
        round_up(batch_size, FLASHINFER_BATCH_BUCKETS[0]),
    )
    if batch_size_padded == batch_size:
        return seq
    return np.concatenate(
        [
            seq,
            np.full((batch_size_padded - batch_size,), padding_value, dtype=seq.dtype),
        ]
    )
```
**EN:** This function defines `add_padding_to_seqlens`. It handles attention-specific state, metadata, or tensor transforms. The main inputs are `seq`, `batch_size`, `padding_value`. Key calls include `next`, `np.concatenate`, `round_up`, `np.full`. It writes or updates `batch_size_padded`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `add_padding_to_seqlens`。 它处理注意力相关的状态、元数据或张量变换。 其主要输入参数包括 `seq`, `batch_size`, `padding_value`。 关键调用包括 `next`, `np.concatenate`, `round_up`, `np.full`。 它会写入或更新 `batch_size_padded`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 203-211 — function `bucket_flashinfer_max_seqlen`
```python
def bucket_flashinfer_max_seqlen(
    real_max_seqlen: int,
) -> int:
    if real_max_seqlen <= 0:
        return FLASHINFER_MAX_SEQLEN_BUCKETS[0]
    return next(
        (s for s in FLASHINFER_MAX_SEQLEN_BUCKETS if s >= real_max_seqlen),
        round_up(real_max_seqlen, FLASHINFER_MAX_SEQLEN_BUCKETS[-1]),
    )
```
**EN:** This function defines `bucket_flashinfer_max_seqlen`. It handles attention-specific state, metadata, or tensor transforms. The main inputs are `real_max_seqlen`. Key calls include `next`, `round_up`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `bucket_flashinfer_max_seqlen`。 它处理注意力相关的状态、元数据或张量变换。 其主要输入参数包括 `real_max_seqlen`。 关键调用包括 `next`, `round_up`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 215-768 — class `MMEncoderAttention`
```python
@CustomOp.register("mm_encoder_attn")
class MMEncoderAttention(CustomOp):
    """Multi-headed attention without any cache, used for multimodal encoder."""

    # --8<-- [end:mm_encoder_attn]
    @classmethod
    def compute_max_seqlen(
        cls,
        attn_backend: AttentionBackendEnum,
        cu_seqlens: np.ndarray,
    ) -> int:
        max_seqlen = 0
        if (
            attn_backend
# ... omitted for brevity ...
                f"{self.attn_backend}."
            )
```
**EN:** This class defines `MMEncoderAttention`. It inherits from `CustomOp`. Multi-headed attention without any cache, used for multimodal encoder. Important methods include `__init__`, `forward_native`, `forward_cuda`, `forward_xpu`, `compute_max_seqlen`, `maybe_compute_seq_lens`. Key calls include `CustomOp.register`, `torch.no_grad`, `add_padding_to_seqlens`, `torch.from_numpy.to`, `super.__init__`, `torch.get_default_dtype`. It writes or updates `max_seqlen`, `sequence_lengths`, `cu_seqlens`, `num_heads`, `head_size`, `scale`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该类定义了 `MMEncoderAttention`。 它继承自 `CustomOp`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `forward_native`, `forward_cuda`, `forward_xpu`, `compute_max_seqlen`, `maybe_compute_seq_lens`。 关键调用包括 `CustomOp.register`, `torch.no_grad`, `add_padding_to_seqlens`, `torch.from_numpy.to`, `super.__init__`, `torch.get_default_dtype`。 它会写入或更新 `max_seqlen`, `sequence_lengths`, `cu_seqlens`, `num_heads`, `head_size`, `scale`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 317-374 — method `MMEncoderAttention.__init__`
```python
    def __init__(
        self,
        num_heads: int,
        head_size: int,
        scale: float | None = None,
        num_kv_heads: int | None = None,
        prefix: str = "",
    ) -> None:
        """
        Args:
            num_heads: number of attention heads per partition.
            head_size: hidden_size per attention head.
            scale: scale factor.
            num_kv_heads: number of kv heads.
            prefix: This has no effect, it is only here to make it easier to
                    swap between Attention and MultiHeadAttention
        """
        super().__init__()

        self.num_heads = num_heads
        self.head_size = head_size
        self.scale = 1.0 / (head_size**0.5) if scale is None else scale
        self.num_kv_heads = num_heads if num_kv_heads is None else num_kv_heads
        self.layer_name = prefix
        assert self.num_heads % self.num_kv_heads == 0, (
            f"num_heads ({self.num_heads}) is not "
            f"divisible by num_kv_heads ({self.num_kv_heads})"
        )
        self.num_queries_per_kv = self.num_heads // self.num_kv_heads

        # During model initialization, the default dtype is set as the model
        # weight and activation dtype.
        dtype = torch.get_default_dtype()
        self.dtype = dtype

        # Get device-specific vision attention backend.
        self.attn_backend = get_vit_attn_backend(
            head_size=head_size,
            dtype=dtype,
        )

        self.is_flash_attn_backend = self.attn_backend in {
            AttentionBackendEnum.FLASH_ATTN,
            AttentionBackendEnum.ROCM_AITER_FA,
# ... omitted for brevity ...

        self._init_fp8_state()
```
**EN:** This method defines `__init__`. Args: num_heads: number of attention heads per partition. The main inputs are `num_heads`, `head_size`, `scale`, `num_kv_heads`, `prefix`. Key calls include `super.__init__`, `torch.get_default_dtype`, `get_vit_attn_backend`, `logger.info_once`, `self._init_fp8_state`, `get_flash_attn_version`. It writes or updates `num_heads`, `head_size`, `scale`, `num_kv_heads`, `layer_name`, `num_queries_per_kv`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `__init__`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `num_heads`, `head_size`, `scale`, `num_kv_heads`, `prefix`。 关键调用包括 `super.__init__`, `torch.get_default_dtype`, `get_vit_attn_backend`, `logger.info_once`, `self._init_fp8_state`, `get_flash_attn_version`。 它会写入或更新 `num_heads`, `head_size`, `scale`, `num_kv_heads`, `layer_name`, `num_queries_per_kv`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 698-708 — method `MMEncoderAttention.forward_native`
```python
    def forward_native(
        self,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        cu_seqlens: torch.Tensor | None = None,
        max_seqlen: torch.Tensor | None = None,  # Only used for Flash Attention
        sequence_lengths: torch.Tensor
        | None = None,  # Only used for FlashInfer CuDNN backend
    ) -> torch.Tensor:
        return self._forward_sdpa(query, key, value, cu_seqlens)
```
**EN:** This method defines `forward_native`. It executes the main forward/runtime path for this component. The main inputs are `query`, `key`, `value`, `cu_seqlens`, `max_seqlen`, `sequence_lengths`. Key calls include `self._forward_sdpa`.
**CN:** 该方法定义 `forward_native`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `query`, `key`, `value`, `cu_seqlens`, `max_seqlen`, `sequence_lengths`。 关键调用包括 `self._forward_sdpa`。

### Lines 710-734 — method `MMEncoderAttention.forward_cuda`
```python
    def forward_cuda(
        self,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        cu_seqlens: torch.Tensor | None = None,
        max_seqlen: torch.Tensor | None = None,  # Only used for Flash Attention
        sequence_lengths: torch.Tensor
        | None = None,  # Only used for FlashInfer CuDNN backend
    ) -> torch.Tensor:
        if self.is_flash_attn_backend:
            return self._forward_fa(query, key, value, cu_seqlens, max_seqlen)
        elif self.attn_backend == AttentionBackendEnum.TRITON_ATTN:
            return self._forward_triton(query, key, value, cu_seqlens, max_seqlen)
        elif self.attn_backend == AttentionBackendEnum.FLASHINFER:
            return self._forward_flashinfer(
                query, key, value, cu_seqlens, max_seqlen, sequence_lengths
            )
        elif self.attn_backend == AttentionBackendEnum.TORCH_SDPA:
            return self._forward_sdpa(query, key, value, cu_seqlens)
        else:
            raise ValueError(
                f"Unsupported multi-modal encoder attention backend for CUDA: "
                f"{self.attn_backend}."
            )
```
**EN:** This method defines `forward_cuda`. It executes the main forward/runtime path for this component. The main inputs are `query`, `key`, `value`, `cu_seqlens`, `max_seqlen`, `sequence_lengths`. Key calls include `self._forward_fa`, `self._forward_triton`, `self._forward_flashinfer`, `self._forward_sdpa`, `ValueError`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `forward_cuda`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `query`, `key`, `value`, `cu_seqlens`, `max_seqlen`, `sequence_lengths`。 关键调用包括 `self._forward_fa`, `self._forward_triton`, `self._forward_flashinfer`, `self._forward_sdpa`, `ValueError`。 函数体通过条件分支来覆盖不同的运行时场景。

## Key Concepts / 关键概念
- [EN] Attention execution, backend dispatch, and kv-cache handling / [CN] 注意力执行、后端分派与 KV 缓存处理
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Triton kernel specialization / [CN] Triton 内核特化
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `_load_fp8_scales_file`, `_maybe_save_fp8_scales`, `_get_flashinfer_workspace_buffer`, `add_padding_to_seqlens` / [CN] 核心符号：`_load_fp8_scales_file`, `_maybe_save_fp8_scales`, `_get_flashinfer_workspace_buffer`, `add_padding_to_seqlens`

## Dependencies / 依赖关系
- **External**: `functools`, `json`, `numpy`, `torch` / **外部依赖**: `functools`, `json`, `numpy`, `torch`
- **Internal**: `vllm.config`, `vllm.kernels.triton.qkv_padded_fp8_quant`, `vllm.logger`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.quantization.input_quant_fp8`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.models.vision`, `vllm.utils.flashinfer`, `vllm.utils.math_utils`, `vllm.v1.attention.backends.fa_utils` / **内部依赖**: `vllm.config`, `vllm.kernels.triton.qkv_padded_fp8_quant`, `vllm.logger`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.quantization.input_quant_fp8`, `vllm.model_executor.layers.quantization.utils.quant_utils`, `vllm.model_executor.models.vision`, `vllm.utils.flashinfer`, `vllm.utils.math_utils`, `vllm.v1.attention.backends.fa_utils`
- **Runtime traits**: Triton kernels / **运行时特征**: Triton kernels
