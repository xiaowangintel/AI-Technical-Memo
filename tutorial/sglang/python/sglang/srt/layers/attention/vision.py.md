# vision.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `python/sglang/srt/layers/attention/vision.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module manages vision logic around KV-cache layout, indexing, or paging for SGLang attention execution. / 该模块管理与 vision 相关的 KV 缓存布局、索引或分页逻辑，用于 SGLang 注意力执行。
## Line-by-Line Analysis / 逐行分析
### Lines 1-33: imports
```python
from __future__ import annotations

import dataclasses
import functools
import math
import warnings
from functools import lru_cache, partial
from typing import Any, Callable, Optional, Tuple

import torch
import torch.nn as nn
import torch.nn.functional as F
from einops import rearrange

from sglang.jit_kernel.norm import can_use_fused_inplace_qknorm as can_use_jit_qk_norm
from sglang.srt.environ import envs
from sglang.srt.layers.dp_attention import get_attention_tp_rank, get_attention_tp_size
from sglang.srt.models.utils import apply_qk_norm
from sglang.srt.utils import (
    get_bool_env_var,
    get_device_capability,
    is_blackwell_supported,
    is_cuda,
    is_hip,
    is_musa,
    is_npu,
    is_xpu,
    print_info_once,
)
from sglang.srt.utils.multi_stream_utils import (
    maybe_execute_in_parallel,
    with_multi_stream,
)
```
**EN:** Imports PyTorch, optional accelerator libraries, and internal SGLang modules required by the attention path.
**CN:** 导入该注意力路径所需的 PyTorch、可选加速库以及 SGLang 内部模块。

### Lines 35-39: module constants
```python
_is_cuda = is_cuda()
_is_musa = is_musa()
_is_npu = is_npu()
_is_hip = is_hip()
_is_xpu = is_xpu()
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 41-46: conditional branch
```python
if _is_cuda:
    from flashinfer.prefill import cudnn_batch_prefill_with_kv_cache

    from sglang.jit_kernel.flash_attention import (
        flash_attn_varlen_func,
    )
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 48-49: conditional branch
```python
if _is_musa:
    from flash_attn_interface import flash_attn_varlen_func
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 51-52: conditional branch
```python
if _is_npu:
    import torch_npu
```
**EN:** Branches on configuration or runtime conditions to enable different attention behaviors.
**CN:** 根据配置或运行时条件分支，以启用不同的注意力行为。

### Lines 54-71: imports
```python
from sglang.srt.distributed import (
    split_tensor_along_last_dim,
    tensor_model_parallel_all_gather,
)
from sglang.srt.distributed import utils as dist_utils
from sglang.srt.layers.attention.triton_ops.prefill_attention import (
    context_attention_fwd,
)
from sglang.srt.layers.layernorm import RMSNorm
from sglang.srt.layers.linear import (
    ColumnParallelLinear,
    QKVParallelLinear,
    RowParallelLinear,
)
from sglang.srt.layers.quantization import QuantizationConfig
from sglang.srt.layers.rotary_embedding import apply_rotary_pos_emb
from sglang.srt.server_args import get_global_server_args
from sglang.srt.utils import add_prefix, get_bool_env_var
```
**EN:** Imports neighboring SGLang modules so this file can reuse shared attention abstractions and utilities.
**CN:** 导入相邻的 SGLang 模块，以复用共享的注意力抽象和工具函数。

### Lines 73-95: module constants
```python
_use_aiter = get_bool_env_var("SGLANG_USE_AITER") and _is_hip

ROTARY_EMBED_CLASSES = {
    "normal": apply_rotary_pos_emb,
}

# === Vision Encoder === #
FLASHINFER_WORKSPACE_SIZE_BYTES = 128 * 1024 * 1024

# Batch buckets for cuDNN graph caching - graphs are cached per bucket size
# This avoids creating a new graph for each unique batch size at runtime
BATCH_BUCKETS = [8, 16, 32, 64]

# Bucketized max seqlens to reduce cuDNN recompilation frequency while
# preserving a tighter upper bound than a single fixed max seqlen.
FLASHINFER_MAX_SEQLEN_BUCKETS = [
    4 * 1024,
    8 * 1024,
    16 * 1024,
    32 * 1024,
    64 * 1024,
    128 * 1024,
]
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 98-101: class SingletonCache
```python
@dataclasses.dataclass
class SingletonCache:
    data: Any = None
```
**EN:** Dataclass-style container that stores structured runtime state for singleton cache.
**CN:** 该数据类风格的容器用于存储 singleton cache 的结构化运行时状态。

### Lines 102-103: method SingletonCache.set_data
```python
    def set_data(self, value: Any) -> None:
        self.data = value
```
**EN:** Updates set data on the active object so later attention steps observe the latest runtime state.
**CN:** 更新活动对象上的 set data，以便后续注意力步骤读取最新运行时状态。

### Lines 105-106: method SingletonCache.get_data
```python
    def get_data(self) -> Optional[Any]:
        return self.data
```
**EN:** Computes and returns get data from the current inputs, cached tensors, or execution metadata.
**CN:** 根据当前输入、缓存张量或执行元数据计算并返回 get data。

### Lines 108-109: method SingletonCache.empty
```python
    def empty(self) -> bool:
        return self.get_data() is None
```
**EN:** Implements the empty routine used by this attention module.
**CN:** 实现该注意力模块使用的 empty 例程。

### Lines 113-126: function _get_cu_seqlens_for_shape
```python
@functools.lru_cache(maxsize=128)
def _get_cu_seqlens_for_shape(batch_size: int, seqlen: int, device) -> torch.Tensor:
    """
    Generates cumulative sequence lengths (cu_seqlens) for a given batch_size, seqlen, and device.
    Caches the result based on these parameters.
    """
    cu_seqlens = torch.arange(
        0,
        (batch_size + 1) * seqlen,
        step=seqlen,
        dtype=torch.int32,
        device=device,
    )
    return cu_seqlens
```
**EN:** Implements the get cu seqlens for shape routine used by this attention module.
**CN:** 实现该注意力模块使用的 get cu seqlens for shape 例程。

### Lines 129-147: function resolve_seqlens
```python
def resolve_seqlens(
    cu_seqlens: torch.Tensor | SingletonCache | None,
    bsz: int,
    seq_len: int,
    *,
    device: torch.device,
) -> torch.Tensor:
    if cu_seqlens is None:
        resolved_seqlens = _get_cu_seqlens_for_shape(bsz, seq_len, device=device)
    elif isinstance(cu_seqlens, SingletonCache):
        if cu_seqlens.empty():
            cu_seqlens.set_data(_get_cu_seqlens_for_shape(bsz, seq_len, device=device))
        resolved_seqlens = cu_seqlens.get_data()
    else:
        resolved_seqlens = cu_seqlens
    assert isinstance(
        resolved_seqlens, torch.Tensor
    ), "cu_seqlens must be a torch.Tensor"
    return resolved_seqlens
```
**EN:** Implements the resolve seqlens routine used by this attention module.
**CN:** 实现该注意力模块使用的 resolve seqlens 例程。

### Lines 150-155: class VisionSdpaAttention
```python
class VisionSdpaAttention(nn.Module):
    r"""
    Scaled Dot Product Attention inner product

    """
```
**EN:** Defines the vision sdpa attention type and the state it exposes to the rest of the attention stack.
**CN:** 定义 vision sdpa attention 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 156-178: method VisionSdpaAttention.__init__
```python
    def __init__(
        self,
        head_dim: int,
        num_heads: int,
        num_kv_heads: int,
        dropout: float = 0.0,
        flatten_batch: bool = False,
        softmax_in_single_precision: bool = False,
        softmax_scale: float | None = None,
        **kwargs,
    ):
        super().__init__()
        self.head_size = head_dim
        self.num_heads = num_heads
        self.num_kv_heads = num_kv_heads
        self.flatten_batch = flatten_batch
        self.softmax_in_single_precision = softmax_in_single_precision
        self.dropout = dropout
        self.scale = (
            softmax_scale
            if softmax_scale is not None
            else 1.0 / math.sqrt(self.head_size)
        )
```
**EN:** Initializes the VisionSdpaAttention instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 VisionSdpaAttention 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 180-212: method VisionSdpaAttention._generate_mask_cache
```python
    @staticmethod
    @lru_cache(maxsize=128)
    def _generate_mask_cache(
        s: int, flatten_batch: bool, cu_seqlens: tuple
    ) -> torch.BoolTensor:
        """
        Generate a boolean attention mask with caching mechanism.
        Args:
            s: sequence length
            flatten_batch: whether to flatten batch dimension
            cu_seqlens: tuple of cumulative sequence lengths
        Returns:
            attention mask tensor of shape [b, 1, s, s] or [1, s, s]
        """
        if flatten_batch:
            mask = torch.zeros([1, s, s], dtype=torch.bool)
            for i in range(1, len(cu_seqlens)):
                start = cu_seqlens[i - 1]
                end = cu_seqlens[i]
                mask[..., start:end, start:end] = True
        else:
            # [1, 1, 1, s]
            row_indices = torch.arange(s).view(1, 1, 1, s)
            # [1, 1, s, 1]
            col_indices = torch.arange(s).view(1, 1, s, 1)
            # [b, 1, 1, 1]
            seq_lens = torch.tensor(
                [end - start for start, end in zip(cu_seqlens[:-1], cu_seqlens[1:])],
            ).view(-1, 1, 1, 1)

            mask = (row_indices < seq_lens) & (col_indices < seq_lens)

        return mask
```
**EN:** Implements the generate mask cache routine used by this attention module.
**CN:** 实现该注意力模块使用的 generate mask cache 例程。

### Lines 214-234: method VisionSdpaAttention.generate_patch_attention_mask
```python
    def generate_patch_attention_mask(
        self,
        s: int,
        cu_seqlens: Optional[torch.Tensor],
        flatten_batch: bool = False,
    ) -> Optional[torch.Tensor]:
        r"""
        Creates a non-causal 4D mask of shape `(b, 1, s, s)` or `(1, 1, s, s)`.
        Args:
            s: sequence length
            cu_seqlens: cumulative sequence lengths tensor. If not, returns an empty mask
            flatten_batch: whether to flatten batch dimension
        Returns:
            attention mask tensor or None
        """
        if cu_seqlens is None:
            return None

        cu_seqlens_tuple = tuple(cu_seqlens.cpu().tolist())

        return self._generate_mask_cache(s, flatten_batch, cu_seqlens_tuple)
```
**EN:** Implements the generate patch attention mask routine used by this attention module.
**CN:** 实现该注意力模块使用的 generate patch attention mask 例程。

### Lines 236-307: method VisionSdpaAttention.forward
```python
    def forward(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        bsz: int,
        cu_seqlens: Optional[torch.Tensor] = None,
        attention_mask: Optional[torch.Tensor] = None,
        softmax_scale: Optional[float] = None,
        **kwargs,
    ) -> torch.Tensor:
        r"""
        Args:
            cu_seqlens: [b]
        Returns:
             [b * s, h, head_size]
        """
        if self.flatten_batch:
            assert bsz == 1, "flatten_batch is True, bsz must be 1"

        assert q.dim() == 3, q.shape

        s = q.shape[0] // bsz

        # [b, 1, s, s]
        if attention_mask is None:
            attention_mask = self.generate_patch_attention_mask(
                s, cu_seqlens, flatten_batch=self.flatten_batch
            )

        if attention_mask is None:
            if self.softmax_in_single_precision:
# ... omitted 28 lines ...
                k,
                v,
                attn_mask=attention_mask,
                dropout_p=self.dropout,
                is_causal=False,
                scale=self.scale,
            )

        # [b, h, s, head_size] --> [b * s, h, head_size]
        output = rearrange(output, "b h s d -> (b s) h d")

        return output
```
**EN:** Runs the forward-path logic for forward, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 310-314: class VisionTritonAttention
```python
class VisionTritonAttention(nn.Module):
    """
    Triton-implemented attention without a causal mask
    """
```
**EN:** Defines the vision triton attention type and the state it exposes to the rest of the attention stack.
**CN:** 定义 vision triton attention 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 315-323: method VisionTritonAttention.__init__
```python
    def __init__(
        self,
        **kwargs,
    ):
        super().__init__()
        use_data_parallel = (
            kwargs["use_data_parallel"] if "use_data_parallel" in kwargs else False
        )
        self.tp_size = 1 if use_data_parallel else get_attention_tp_size()
```
**EN:** Initializes the VisionTritonAttention instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 VisionTritonAttention 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 325-382: method VisionTritonAttention.forward
```python
    def forward(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        cu_seqlens: torch.Tensor | SingletonCache | None,
        bsz: int,
        seq_len: int,
        softmax_scale: Optional[float] = None,
        **kwargs,
    ) -> torch.Tensor:
        r"""
        Args:
            cu_seqlens: [b]
            softmax_scale: override softmax scale (default 1/sqrt(head_dim))
        Returns:
             [b * s, h, head_size]
        """
        if envs.SGLANG_VIT_ENABLE_CUDA_GRAPH.get():
            if "output_ws" not in kwargs:
                raise RuntimeError("output_ws should be prepared for cuda-graph mode")

            if not isinstance(cu_seqlens, list):
                raise RuntimeError("cuda-graph mode cu_seqlens should be a list")

            output = kwargs["output_ws"]
            context_attention_fwd(
                q,
                k,
                v,
                output,
                cu_seqlens[0],
# ... omitted 14 lines ...
                q,
                k,
                v,
                output,
                cu_seqlens.to(q.device),
                seq_lens.to(q.device),
                max_seqlen,
                is_causal=False,
                sm_scale=softmax_scale,
            )

        return output
```
**EN:** Runs the forward-path logic for forward, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 385-385: class VisionFlash3Attention
```python
class VisionFlash3Attention(nn.Module):
```
**EN:** Defines the vision flash3 attention type and the state it exposes to the rest of the attention stack.
**CN:** 定义 vision flash3 attention 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 386-396: method VisionFlash3Attention.__init__
```python
    def __init__(
        self,
        **kwargs,
    ):
        if not (_is_cuda or _is_musa):
            raise Exception("VisionFlash3Attention is only available for cuda or musa")
        super().__init__()
        use_data_parallel = (
            kwargs["use_data_parallel"] if "use_data_parallel" in kwargs else False
        )
        self.tp_size = 1 if use_data_parallel else get_attention_tp_size()
```
**EN:** Initializes the VisionFlash3Attention instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 VisionFlash3Attention 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 398-449: method VisionFlash3Attention.forward
```python
    def forward(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        cu_seqlens: torch.Tensor | SingletonCache | None,
        bsz: int,
        seq_len: int,
        softmax_scale: Optional[float] = None,
        **kwargs,
    ) -> torch.Tensor:
        r"""
        Args:
            cu_seqlens: [b]
        Returns:
             [b * s, h, head_size]
        """
        window_size = kwargs.get("window_size", (-1, -1))
        s_aux = kwargs.get("s_aux", None)

        if envs.SGLANG_VIT_ENABLE_CUDA_GRAPH.get():
            max_seqlen = cu_seqlens[1]
            fa_kwargs = dict(
                cu_seqlens_q=cu_seqlens[0],
                cu_seqlens_k=cu_seqlens[0],
                max_seqlen_q=max_seqlen,
                max_seqlen_k=max_seqlen,
                softmax_scale=softmax_scale,
                window_size=window_size,
            )
            if s_aux is not None:
                fa_kwargs["sinks"] = s_aux
# ... omitted 8 lines ...
                cu_seqlens_q=cu_seqlens,
                cu_seqlens_k=cu_seqlens,
                max_seqlen_q=max_seqlen,
                max_seqlen_k=max_seqlen,
                softmax_scale=softmax_scale,
                window_size=window_size,
            )
            if s_aux is not None:
                fa_kwargs["sinks"] = s_aux
            output = flash_attn_varlen_func(q, k, v, **fa_kwargs)

        return output
```
**EN:** Runs the forward-path logic for forward, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 452-452: class VisionFlash4Attention
```python
class VisionFlash4Attention(nn.Module):
```
**EN:** Defines the vision flash4 attention type and the state it exposes to the rest of the attention stack.
**CN:** 定义 vision flash4 attention 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 453-459: method VisionFlash4Attention.__init__
```python
    def __init__(
        self,
        **kwargs,
    ):
        if not _is_cuda:
            raise Exception("VisionFlash4Attention is only available for cuda")
        super().__init__()
```
**EN:** Initializes the VisionFlash4Attention instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 VisionFlash4Attention 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 461-503: method VisionFlash4Attention.forward
```python
    def forward(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        cu_seqlens: torch.Tensor | SingletonCache | None,
        bsz: int,
        seq_len: int,
        softmax_scale: Optional[float] = None,
        **kwargs,
    ) -> torch.Tensor:
        r"""
        Args:
            cu_seqlens: [b]
        Returns:
             [b * s, h, head_size]
        """
        if cu_seqlens is None:
            cu_seqlens = _get_cu_seqlens_for_shape(bsz, seq_len, device=q.device)
        elif isinstance(cu_seqlens, SingletonCache):
            if cu_seqlens.empty():
                cu_seqlens.set_data(
                    _get_cu_seqlens_for_shape(bsz, seq_len, device=q.device)
                )
            cu_seqlens = cu_seqlens.get_data()

        cu_seqlens = cu_seqlens.to(dtype=torch.int32).to(q.device)
        seq_lens = cu_seqlens[1:] - cu_seqlens[:-1]
        max_seqlen = seq_lens.max().item()

        output = flash_attn_varlen_func(
            q,
            k,
            v,
            cu_seqlens_q=cu_seqlens,
            cu_seqlens_k=cu_seqlens,
            max_seqlen_q=max_seqlen,
            max_seqlen_k=max_seqlen,
            softmax_scale=softmax_scale,
            ver=4,
        )

        return output
```
**EN:** Runs the forward-path logic for forward, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 506-506: class VisionFlashInferAttention
```python
class VisionFlashInferAttention(nn.Module):
```
**EN:** Defines the vision flash infer attention type and the state it exposes to the rest of the attention stack.
**CN:** 定义 vision flash infer attention 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 507-516: method VisionFlashInferAttention.__init__
```python
    def __init__(
        self,
        **kwargs,
    ):
        if not _is_cuda:
            raise Exception("VisionFlashInferAttention is only available for cuda")
        super().__init__()
        self.workspace_buffer = (
            kwargs["workspace_buffer"] if "workspace_buffer" in kwargs else None
        )
```
**EN:** Initializes the VisionFlashInferAttention instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 VisionFlashInferAttention 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 518-626: method VisionFlashInferAttention.forward
```python
    def forward(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        cu_seqlens: torch.Tensor | SingletonCache | None,
        bsz: int,
        seq_len: int,
        softmax_scale: Optional[float] = None,
        **kwargs,
    ) -> torch.Tensor:
        r"""
        Args:
            cu_seqlens: [b]
        Returns:
             [b * s, h, head_size]
        """
        if "sequence_lengths" not in kwargs:
            raise RuntimeError(
                "sequence_lengths should be prepared for vision flashinfer_cudnn attention backend"
            )
        if "max_seqlen" not in kwargs:
            raise RuntimeError(
                "max_seqlen should be prepared for vision flashinfer_cudnn attention backend"
            )

        sequence_lengths = kwargs["sequence_lengths"]  # (B_padded,) or (B_padded,1,1,1)
        max_seqlen = kwargs["max_seqlen"]

        # max_seqlen must be python int
        if isinstance(max_seqlen, torch.Tensor):
            if max_seqlen.is_cuda:
# ... omitted 65 lines ...
            return_lse=True,
            batch_offsets_q=indptr_qk,
            batch_offsets_k=indptr_qk,
            batch_offsets_v=indptr_v,
            batch_offsets_o=indptr_o,
            is_cuda_graph_compatible=True,
        )

        if is_reshaped:
            output = rearrange(output, "(b s) h d -> b s h d", b=reshape_batch_size)

        return output
```
**EN:** Runs the forward-path logic for forward, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 629-629: class VisionAiterAttention
```python
class VisionAiterAttention(nn.Module):
```
**EN:** Defines the vision aiter attention type and the state it exposes to the rest of the attention stack.
**CN:** 定义 vision aiter attention 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 630-644: method VisionAiterAttention.__init__
```python
    def __init__(
        self,
        **kwargs,
    ):
        if not _is_hip:
            raise Exception("aiter_attn is only available for AMD")
        try:
            from aiter import flash_attn_varlen_func as aiter_flash_attn_varlen_func
        except ImportError as e:
            raise ImportError(
                "aiter is AMD specific kernel library. Please make sure aiter is installed on your AMD device."
            ) from e

        self.flash_attn_varlen_func = aiter_flash_attn_varlen_func
        super().__init__()
```
**EN:** Initializes the VisionAiterAttention instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 VisionAiterAttention 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 646-672: method VisionAiterAttention.forward
```python
    def forward(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        cu_seqlens: torch.Tensor | SingletonCache | None,
        bsz: int,
        seq_len: int,
        softmax_scale: Optional[float] = None,
        **kwargs,
    ) -> torch.Tensor:
        cu_seqlens = resolve_seqlens(cu_seqlens, bsz, seq_len, device=q.device)

        cu_seqlens = cu_seqlens.to(dtype=torch.int32).to(q.device)
        seq_lens = cu_seqlens[1:] - cu_seqlens[:-1]
        max_seqlen = seq_lens.max().item()

        return self.flash_attn_varlen_func(
            q=q,
            k=k,
            v=v,
            cu_seqlens_q=cu_seqlens,
            cu_seqlens_k=cu_seqlens,
            max_seqlen_q=max_seqlen,
            max_seqlen_k=max_seqlen,
            softmax_scale=softmax_scale,
        )
```
**EN:** Runs the forward-path logic for forward, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 675-676: class VisionAscendAttention
```python
class VisionAscendAttention(nn.Module):
```
**EN:** Defines the vision ascend attention type and the state it exposes to the rest of the attention stack.
**CN:** 定义 vision ascend attention 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 677-683: method VisionAscendAttention.__init__
```python
    def __init__(
        self,
        **kwargs,
    ):
        if not _is_npu:
            raise Exception("VisionAscendAttention is only available for ascend npu")
        super().__init__()
```
**EN:** Initializes the VisionAscendAttention instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 VisionAscendAttention 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 685-730: method VisionAscendAttention.forward
```python
    def forward(
        self,
        q: torch.Tensor,
        k: torch.Tensor,
        v: torch.Tensor,
        cu_seqlens: torch.Tensor | SingletonCache | None,
        bsz: int,
        seq_len: int,
        softmax_scale: Optional[float] = None,
        **kwargs,
    ) -> torch.Tensor:
        r"""
        Args:
            cu_seqlens: [b]
        Returns:
             [b * s, h, head_size]
        """
        if envs.SGLANG_VIT_ENABLE_CUDA_GRAPH.get():
            if "output_ws" not in kwargs:
                raise RuntimeError("output_ws should be prepared for npu-graph mode")
            output = kwargs["output_ws"]
            seq_len_arg = cu_seqlens
        else:
            cu_seqlens = resolve_seqlens(cu_seqlens, bsz, seq_len, device="cpu")
            seq_lens = cu_seqlens[1:] - cu_seqlens[:-1]
            if seq_lens.is_npu:
                seq_lens = seq_lens.to("cpu")
            output = torch.empty_like(q)
            seq_len_arg = seq_lens.to(torch.int32)

        _, num_heads, head_size = q.shape
        num_kv_heads = k.shape[1]

        scale_value = softmax_scale if softmax_scale is not None else head_size**-0.5

        torch_npu._npu_flash_attention_unpad(
            query=q,
            key=k,
            value=v,
            seq_len=seq_len_arg,
            scale_value=scale_value,
            num_heads=num_heads,
            num_kv_heads=num_kv_heads,
            out=output,
        )
        return output
```
**EN:** Runs the forward-path logic for forward, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

### Lines 733-741: module constants
```python
QKV_BACKEND_IMPL = {
    "triton_attn": VisionTritonAttention,
    "sdpa": VisionSdpaAttention,
    "fa3": VisionFlash3Attention,
    "fa4": VisionFlash4Attention,
    "flashinfer_cudnn": VisionFlashInferAttention,
    "ascend_attn": VisionAscendAttention,
    "aiter_attn": VisionAiterAttention,
}
```
**EN:** Defines module-level constants, feature flags, or reusable helper objects used by later logic.
**CN:** 定义后续逻辑使用的模块级常量、功能开关或可复用辅助对象。

### Lines 744-756: class VisionAttention
```python
class VisionAttention(nn.Module):
    r"""
        Multi-headed attention without any cache, mostly used for multimodal transformers.


    Args:
        use_qkv_parallel (bool, optional): If True, use QKV-parallel attention.
        softmax_in_single_precision (bool, default to False):
            if ``True``, the softmax will be performed in single-precision
            Otherwise, it will be performed in half-precision

    """
```
**EN:** Defines the vision attention type and the state it exposes to the rest of the attention stack.
**CN:** 定义 vision attention 类型，以及它向注意力栈其余部分暴露的状态。

### Lines 757-907: method VisionAttention.__init__
```python
    def __init__(
        self,
        embed_dim: int,
        num_heads: int,
        projection_size: int,
        use_qkv_parallel: bool,
        num_kv_heads: Optional[int] = None,
        head_dim: Optional[int] = None,
        qkv_backend: Optional[str] = None,
        quant_config: Optional[QuantizationConfig] = None,
        dropout: float = 0.0,
        softmax_in_single_precision: bool = False,
        softmax_scale: Optional[float] = None,
        flatten_batch: bool = False,
        prefix: str = "",
        proj_bias: bool = True,
        num_dummy_heads: int = 0,
        qkv_bias: bool = True,
        qk_normalization: bool = False,
        qk_normalization_by_head_size: bool = False,
        layer_norm_eps: float = 1e-06,
        customized_position_embedding_applier: Callable[
            [torch.Tensor, torch.Tensor, Any, Any], Tuple[torch.Tensor, torch.Tensor]
        ] = None,
        use_data_parallel: bool = False,
        use_dp_attention_reduce: bool = False,
        aux_stream: Optional[torch.cuda.Stream] = None,
        workspace_buffer: Optional[torch.Tensor] = None,
        use_sink: bool = False,
        window_size: Tuple[int, int] = (-1, -1),
        **kwargs,
    ):
# ... omitted 107 lines ...
        if use_sink:
            # Allocate the full (unsharded) sink tensor for weight loading;
            # only the local TP slice is used in forward.
            self.sinks = nn.Parameter(
                torch.empty(
                    self.num_attention_heads_per_partition * self.tp_size,
                    dtype=torch.bfloat16,
                ),
                requires_grad=False,
            )
        else:
            self.sinks = None
```
**EN:** Initializes the VisionAttention instance, caches configuration, and prepares reusable runtime state or buffers.
**CN:** 初始化 VisionAttention 实例，缓存配置，并准备可复用的运行时状态或缓冲区。

### Lines 909-932: method VisionAttention._init_qk_norm
```python
    def _init_qk_norm(
        self, norm_dim: int, eps: float, var_hidden_size: Optional[int] = None
    ):
        norm_kwargs = (
            dict(
                weight_dtype=torch.float32,
                cast_x_before_out_mul=True,
            )
            if get_global_server_args().rl_on_policy_target is not None
            else {}
        )
        q_norm = RMSNorm(
            norm_dim,
            eps=eps,
            var_hidden_size=var_hidden_size,
            **norm_kwargs,
        )
        k_norm = RMSNorm(
            norm_dim,
            eps=eps,
            var_hidden_size=var_hidden_size,
            **norm_kwargs,
        )
        return q_norm, k_norm
```
**EN:** Implements the init qk norm routine used by this attention module.
**CN:** 实现该注意力模块使用的 init qk norm 例程。

### Lines 934-975: method VisionAttention._determine_attention_backend
```python
    def _determine_attention_backend(self, passed_backend: Optional[str]) -> str:
        """Decide the multimodal attention backend string.

        Priority: server args override > constructor arg > platform default.

        Platform defaults:
        - CUDA (Hopper SM90): "fa3"
        - CUDA (Blackwell SM100): "fa4"
        - CUDA (other): "triton_attn"
        - Non-CUDA: "sdpa"
        """
        override_backend = get_global_server_args().mm_attention_backend
        if override_backend is not None:
            backend = override_backend
        elif passed_backend is not None:
            backend = passed_backend
        elif is_cuda():
            major, minor = get_device_capability()
            if major == 9:
                backend = "fa3"
            elif major == 10:
                backend = "fa4"
            else:
                backend = "triton_attn"
        elif _is_musa:
            if get_device_capability() >= (3, 1):
                backend = "fa3"
            else:
                backend = "triton_attn"
        elif _is_hip:
            if get_device_capability() >= (9, 4) and _use_aiter:
                backend = "aiter_attn"
            else:
                backend = "triton_attn"
        elif _is_xpu:
            backend = "triton_attn"
        else:
            backend = "sdpa"
        if backend == "fa3" and is_blackwell_supported():
            raise ValueError("The 'fa3' backend is not supported on Blackwell GPUs")

        return backend
```
**EN:** Implements the determine attention backend routine used by this attention module.
**CN:** 实现该注意力模块使用的 determine attention backend 例程。

### Lines 977-985: method VisionAttention._apply_qk_norm_head_size
```python
    def _apply_qk_norm_head_size(self, q: torch.Tensor, k: torch.Tensor):
        """apply qk norm for GLM-OCR vit attn"""
        q_by_head = q.reshape(-1, self.head_size)
        q_by_head = self.q_norm(q_by_head)
        k_by_head = k.reshape(-1, self.head_size)
        k_by_head = self.k_norm(k_by_head)
        q = q_by_head.view(q.shape)
        k = k_by_head.view(k.shape)
        return q, k
```
**EN:** Implements the apply qk norm head size routine used by this attention module.
**CN:** 实现该注意力模块使用的 apply qk norm head size 例程。

### Lines 987-1023: method VisionAttention._apply_qk_norm
```python
    def _apply_qk_norm(self, q: torch.Tensor, k: torch.Tensor):
        """apply qk norm for internvl vit attn"""

        def q_l2norm():
            q_ = q.flatten(1, 2)
            if self.tp_size > 1:
                q_ = tensor_model_parallel_all_gather(q_.contiguous())
            q_ = self.q_norm(q_)
            if self.tp_size > 1:
                splitter = partial(
                    split_tensor_along_last_dim, num_partitions=self.tp_size
                )
                q_ = splitter(q_)[self.tp_rank]
            q_ = q_.unflatten(-1, (-1, self.head_size))
            return q_

        def k_l2norm():
            k_ = k.flatten(1, 2)
            if self.tp_size > 1:
                k_ = tensor_model_parallel_all_gather(k_.contiguous())
            k_ = self.k_norm(k_)
            if self.tp_size > 1:
                splitter = partial(
                    split_tensor_along_last_dim, num_partitions=self.tp_size
                )
                k_ = splitter(k_)[self.tp_rank]
            k_ = k_.unflatten(-1, (-1, self.head_size))
            return k_

        with with_multi_stream(True):
            q, k = maybe_execute_in_parallel(
                q_l2norm,
                k_l2norm,
                self.ln_events,
                self.aux_stream,
            )
        return q, k
```
**EN:** Implements the apply qk norm routine used by this attention module.
**CN:** 实现该注意力模块使用的 apply qk norm 例程。

### Lines 1025-1209: method VisionAttention.forward
```python
    def forward(
        self,
        x: torch.Tensor,
        cu_seqlens: Optional[torch.Tensor] = None,
        position_embeddings: Optional[Tuple[torch.Tensor, torch.Tensor]] = None,
        rotary_pos_emb_cos: Optional[torch.Tensor] = None,
        rotary_pos_emb_sin: Optional[torch.Tensor] = None,
        attention_mask: Optional[torch.Tensor] = None,
        full_attn: bool = True,
        **kwargs,
    ) -> torch.Tensor:
        r"""
        Args:
            x: [b, s, embed_dim]
            cu_seqlens: [b]
        Returns:
             [s, b, head * head_size]
        """
        if x.dim() == 2:
            x = x.unsqueeze(0)
        assert x.dim() == 3, x.shape
        if (
            get_global_server_args().rl_on_policy_target is not None
            and position_embeddings is not None
        ):
            assert isinstance(position_embeddings, tuple), (
                "expected position_embeddings to be a tuple of two tensors,\n"
                f"but got {type(position_embeddings)}, change if needed"
            )
            position_embeddings = tuple(p.to(x.dtype) for p in position_embeddings)
        x_shape = x.shape
        bsz, s, _ = x_shape
# ... omitted 141 lines ...
            # [b * s, h, head_size] --> [s, b, h * head_size]
            context_layer = rearrange(
                output, "(b s) h d -> s b (h d)", b=bsz, s=s
            ).contiguous()

            # [s, b, h * head_size] --> [s, b, h * head_size]
            output, _ = self.proj(context_layer)

            # [s, b, h * head_size] --> [b, s, h * head_size]
            output = output.view(bsz, s, -1)

        return output
```
**EN:** Runs the forward-path logic for forward, transforming tensors and dispatching the required compute steps.
**CN:** 执行 forward 的前向路径逻辑，对张量进行变换并分发所需的计算步骤。

## Key Concepts / 关键概念
- **EN:** PyTorch tensor orchestration / **CN:** PyTorch 张量编排
- **EN:** Triton kernel integration / **CN:** Triton 内核集成
- **EN:** FlashInfer execution path / **CN:** FlashInfer 执行路径
- **EN:** Flash-style fused attention kernels / **CN:** Flash 风格融合注意力内核
- **EN:** KV-cache management / **CN:** KV 缓存管理
- **EN:** CUDA-aware runtime coordination / **CN:** 面向 CUDA 的运行时协调

## Dependencies / 依赖关系
- `__future__.annotations`
- `dataclasses`
- `functools`
- `math`
- `warnings`
- `functools.lru_cache`
- `functools.partial`
- `typing.Any`
- `typing.Callable`
- `typing.Optional`
- `typing.Tuple`
- `torch`
- `torch.nn`
- `torch.nn.functional`
- `einops.rearrange`
- `sglang.jit_kernel.norm.can_use_fused_inplace_qknorm`
- `sglang.srt.environ.envs`
- `sglang.srt.layers.dp_attention.get_attention_tp_rank`
- `sglang.srt.layers.dp_attention.get_attention_tp_size`
- `sglang.srt.models.utils.apply_qk_norm`
- `sglang.srt.utils.get_bool_env_var`
- `sglang.srt.utils.get_device_capability`
- `sglang.srt.utils.is_blackwell_supported`
- `sglang.srt.utils.is_cuda`
- `sglang.srt.utils.is_hip`
- `sglang.srt.utils.is_musa`
- `sglang.srt.utils.is_npu`
- `sglang.srt.utils.is_xpu`
- `sglang.srt.utils.print_info_once`
- `sglang.srt.utils.multi_stream_utils.maybe_execute_in_parallel`
- `sglang.srt.utils.multi_stream_utils.with_multi_stream`
- `sglang.srt.distributed.split_tensor_along_last_dim`
- `sglang.srt.distributed.tensor_model_parallel_all_gather`
- `sglang.srt.distributed.utils`
- `sglang.srt.layers.attention.triton_ops.prefill_attention.context_attention_fwd`
- `sglang.srt.layers.layernorm.RMSNorm`
- `sglang.srt.layers.linear.ColumnParallelLinear`
- `sglang.srt.layers.linear.QKVParallelLinear`
- `sglang.srt.layers.linear.RowParallelLinear`
- `sglang.srt.layers.quantization.QuantizationConfig`
- `sglang.srt.layers.rotary_embedding.apply_rotary_pos_emb`
- `sglang.srt.server_args.get_global_server_args`
- `sglang.srt.utils.add_prefix`
- `flashinfer.prefill.cudnn_batch_prefill_with_kv_cache`
- `sglang.jit_kernel.flash_attention.flash_attn_varlen_func`
- `flash_attn_interface.flash_attn_varlen_func`
- `torch_npu`
- `aiter.flash_attn_varlen_func`
