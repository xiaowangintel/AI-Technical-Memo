# mla_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/attention/mla_attention.py`
- **Repository**: vllm-project/vllm
- **Purpose**: # MLA Common Components This file implements common components for MLA implementations. / 注意力执行、后端分派与 KV 缓存处理

## Line-by-Line Analysis / 逐行分析
### Lines 190-283 — imports and setup
```python
import functools
from abc import abstractmethod
from dataclasses import dataclass
from enum import Enum
from typing import ClassVar, Generic, TypeVar, cast

import torch
import torch.nn as nn
from tqdm import tqdm

import vllm.envs as envs
from vllm import _custom_ops as ops
from vllm._aiter_ops import rocm_aiter_ops
from vllm.compilation.breakable_cudagraph import eager_break_during_capture
from vllm.config import (
    CacheConfig,
    ModelConfig,
    VllmConfig,
    get_current_vllm_config,
    get_current_vllm_config_or_none,
)
from vllm.distributed.parallel_state import (
    get_dcp_group,
    is_global_first_rank,
)
from vllm.forward_context import ForwardContext, get_forward_context
from vllm.logger import init_logger
from vllm.model_executor.custom_op import CustomOp
from vllm.model_executor.layers.attention.attention import (
    _init_kv_cache_quant,
    get_attention_context,
    set_default_quant_scales,
    should_load_quant_weights,
)
from vllm.model_executor.layers.attention.kv_transfer_utils import (
    maybe_transfer_kv_layer,
)
from vllm.model_executor.layers.attention_layer_base import AttentionLayerBase
from vllm.model_executor.layers.linear import (
    ColumnParallelLinear,
)
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.quantization.input_quant_fp8 import QuantFP8
from vllm.model_executor.layers.quantization.utils.quant_utils import (
    GroupShape,
    QuantKey,
    get_and_maybe_dequant_weights,
    kFp8Dynamic64Sym,
    kFp8Dynamic128Sym,
    kFp8StaticTensorSym,
    kNvfp4Dynamic,
)
from vllm.platforms import current_platform
from vllm.utils.flashinfer import has_flashinfer
from vllm.utils.math_utils import cdiv, round_down
from vllm.utils.torch_utils import (
# ... omitted for brevity ...

_FP8_DTYPE = current_platform.fp8_dtype()
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`, `current_platform.fp8_dtype`. It writes or updates `logger`, `_FP8_DTYPE`.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`, `current_platform.fp8_dtype`。 它会写入或更新 `logger`, `_FP8_DTYPE`。

### Lines 286-319 — function `_detect_output_quant_key`
```python
def _detect_output_quant_key(
    output: torch.Tensor,
    output_scale: torch.Tensor | None,
    output_block_scale: torch.Tensor | None,
    output_dim: int,
) -> QuantKey | None:
    """Detect the output quantization key from fusion pass parameters.

    Returns the appropriate QuantKey, or None if no quantization is needed.
    Detection is based on output dtype and which scale tensors are present.
    """
    if output_scale is None and output_block_scale is None:
        return None
    if output_block_scale is not None:
        if output.dtype == _FP8_DTYPE:
            # Per-group FP8 uses block scales only, not a separate output_scale
            assert output_scale is None
            # Infer group size from scale shape
            num_groups = output_block_scale.shape[-1]
            group_size = output_dim // num_groups
            if group_size == 128:
                return kFp8Dynamic128Sym
            elif group_size == 64:
                return kFp8Dynamic64Sym
            else:
                raise ValueError(
                    f"Unsupported group FP8 group_size={group_size} "
                    f"(output_dim={output_dim}, num_groups={num_groups}). "
                    f"Only group_size 128 and 64 are supported."
                )
        # output_scale None implies MXFP4, not supported
        assert output_scale is not None
        return kNvfp4Dynamic
    return kFp8StaticTensorSym
```
**EN:** This function defines `_detect_output_quant_key`. Detect the output quantization key from fusion pass parameters. The main inputs are `output`, `output_scale`, `output_block_scale`, `output_dim`. Key calls include `ValueError`. It writes or updates `num_groups`, `group_size`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_detect_output_quant_key`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `output`, `output_scale`, `output_block_scale`, `output_dim`。 关键调用包括 `ValueError`。 它会写入或更新 `num_groups`, `group_size`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 322-994 — class `MLAAttention`
```python
class MLAAttention(nn.Module, AttentionLayerBase):
    """Multi-Head Latent Attention layer.

    NOTE: Please read the comment at the top of the file before trying to
    understand this class

    This class takes query, and compressed key/value tensors as input.
    The class does the following:

    1. Store the input key and value tensors in the KV cache.
    2. Perform (multi-head/multi-query/grouped-query) attention.
    3. Return the output tensor.
    """

# ... omitted for brevity ...
            # Multiply + Transpose (N, B, L) x (N, L, V)->(N, B, V)->(B, N, V)
            torch.bmm(x, self.W_UV, out=out.transpose(0, 1))
```
**EN:** This class defines `MLAAttention`. It inherits from `nn.Module`, `AttentionLayerBase`. Multi-Head Latent Attention layer. Important methods include `__init__`, `forward`, `chunked_prefill_workspace_size`, `forward_impl`, `process_weights_after_loading`, `calc_kv_scales`. Key calls include `super.__init__`, `torch.get_default_dtype`, `_init_kv_cache_quant`, `cast`, `impl_cls`, `getattr`. It writes or updates `num_heads`, `scale`, `qk_nope_head_dim`, `qk_rope_head_dim`, `v_head_dim`, `q_lora_rank`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该类定义了 `MLAAttention`。 它继承自 `nn.Module`, `AttentionLayerBase`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `forward`, `chunked_prefill_workspace_size`, `forward_impl`, `process_weights_after_loading`, `calc_kv_scales`。 关键调用包括 `super.__init__`, `torch.get_default_dtype`, `_init_kv_cache_quant`, `cast`, `impl_cls`, `getattr`。 它会写入或更新 `num_heads`, `scale`, `qk_nope_head_dim`, `qk_rope_head_dim`, `v_head_dim`, `q_lora_rank`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 336-520 — method `MLAAttention.__init__`
```python
    def __init__(
        self,
        num_heads: int,
        scale: float,
        qk_nope_head_dim: int,
        qk_rope_head_dim: int,
        v_head_dim: int,
        q_lora_rank: int | None,
        kv_lora_rank: int,
        kv_b_proj: ColumnParallelLinear,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        prefix: str = "",
        attn_backend: type[AttentionBackend] | None = None,
        use_sparse: bool = False,
        indexer: object | None = None,
        **extra_impl_args,
    ):
        super().__init__()
        self.num_heads = num_heads
        self.scale = scale
        self.qk_nope_head_dim = qk_nope_head_dim
        self.qk_rope_head_dim = qk_rope_head_dim
        self.v_head_dim = v_head_dim
        self.q_lora_rank = q_lora_rank
        self.kv_lora_rank = kv_lora_rank
        self.kv_b_proj = kv_b_proj
        self.head_size = kv_lora_rank + qk_rope_head_dim
        self.layer_name = prefix
        self.indexer = indexer

        self.num_kv_heads = 1
        self.qk_head_dim = self.qk_nope_head_dim + self.qk_rope_head_dim

        if cache_config is not None:
            kv_cache_dtype = cache_config.cache_dtype
            calculate_kv_scales = cache_config.calculate_kv_scales
        else:
            kv_cache_dtype = "auto"
            calculate_kv_scales = False
        self.quant_config = quant_config

        dtype = torch.get_default_dtype()
        if attn_backend is not None:
# ... omitted for brevity ...
            compile_native=True,
        )
```
**EN:** This method defines `__init__`. It stores configuration and binds the backend-specific implementation details. The main inputs are `num_heads`, `scale`, `qk_nope_head_dim`, `qk_rope_head_dim`, `v_head_dim`, `q_lora_rank`. Key calls include `super.__init__`, `torch.get_default_dtype`, `_init_kv_cache_quant`, `cast`, `impl_cls`, `getattr`. It writes or updates `num_heads`, `scale`, `qk_nope_head_dim`, `qk_rope_head_dim`, `v_head_dim`, `q_lora_rank`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `__init__`。 它保存配置，并绑定具体后端实现所需的细节。 其主要输入参数包括 `num_heads`, `scale`, `qk_nope_head_dim`, `qk_rope_head_dim`, `v_head_dim`, `q_lora_rank`。 关键调用包括 `super.__init__`, `torch.get_default_dtype`, `_init_kv_cache_quant`, `cast`, `impl_cls`, `getattr`。 它会写入或更新 `num_heads`, `scale`, `qk_nope_head_dim`, `qk_rope_head_dim`, `v_head_dim`, `q_lora_rank`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 532-601 — method `MLAAttention.forward`
```python
    def forward(
        self,
        q: torch.Tensor,
        kv_c_normed: torch.Tensor,
        k_pe: torch.Tensor,
        output_shape: torch.Size | None = None,
    ) -> torch.Tensor:
        if self.calculate_kv_scales:
            torch.ops.vllm.maybe_calc_kv_scales(
                q,
                kv_c_normed,
                k_pe,
                _encode_layer_name(self.layer_name),
            )

        if self.use_direct_call:
            forward_context: ForwardContext = get_forward_context()
            attn_metadata_raw = forward_context.attn_metadata
            attn_metadata: MLACommonMetadata
            if isinstance(attn_metadata_raw, dict):
                attn_metadata = attn_metadata_raw[self.layer_name]  # type: ignore[assignment]
            elif isinstance(attn_metadata_raw, list):
                # list[dict[str, AttentionMetadata]]: used in speculative decoding
                # where [0] is the base-model (non-speculative) metadata dict.
                attn_metadata = attn_metadata_raw[0][self.layer_name]  # type: ignore[assignment]
            else:
                attn_metadata = attn_metadata_raw
            self_kv_cache = self.kv_cache
            slot_mapping = forward_context.slot_mapping

            assert isinstance(slot_mapping, dict), (
                f"Expected slot_mapping to be a dict, got {type(slot_mapping)}. "
            )
            self.impl.do_kv_cache_update(  # type: ignore[attr-defined]
                kv_c_normed,
                k_pe,
                self_kv_cache,
                slot_mapping.get(self.layer_name),
                self.kv_cache_dtype,
                self._k_scale,
            )
            output = torch.empty(output_shape, dtype=q.dtype, device=q.device)
            self.forward_impl(
                q,
# ... omitted for brevity ...
            )
            return output
```
**EN:** This method defines `forward`. It executes the main forward/runtime path for this component. The main inputs are `q`, `kv_c_normed`, `k_pe`, `output_shape`. Key calls include `torch.ops.vllm.maybe_calc_kv_scales`, `get_forward_context`, `isinstance`, `self.impl.do_kv_cache_update`, `torch.empty`, `self.forward_impl`. It writes or updates `forward_context`, `attn_metadata_raw`, `attn_metadata`, `self_kv_cache`, `slot_mapping`, `output`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `forward`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `q`, `kv_c_normed`, `k_pe`, `output_shape`。 关键调用包括 `torch.ops.vllm.maybe_calc_kv_scales`, `get_forward_context`, `isinstance`, `self.impl.do_kv_cache_update`, `torch.empty`, `self.forward_impl`。 它会写入或更新 `forward_context`, `attn_metadata_raw`, `attn_metadata`, `self_kv_cache`, `slot_mapping`, `output`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 522-530 — method `MLAAttention.chunked_prefill_workspace_size`
```python
    @property
    def chunked_prefill_workspace_size(self) -> int:
        if self._chunked_prefill_workspace_size is None:
            self._chunked_prefill_workspace_size = (
                MLACommonMetadataBuilder.determine_chunked_prefill_workspace_size(
                    self._vllm_config
                )
            )
        return self._chunked_prefill_workspace_size
```
**EN:** This method defines `chunked_prefill_workspace_size`. It handles attention-specific state, metadata, or tensor transforms. Key calls include `MLACommonMetadataBuilder.determine_chunked_prefill_workspace_size`. It writes or updates `_chunked_prefill_workspace_size`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `chunked_prefill_workspace_size`。 它处理注意力相关的状态、元数据或张量变换。 关键调用包括 `MLACommonMetadataBuilder.determine_chunked_prefill_workspace_size`。 它会写入或更新 `_chunked_prefill_workspace_size`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 603-833 — method `MLAAttention.forward_impl`
```python
    def forward_impl(
        self,
        q: torch.Tensor,
        k_c_normed: torch.Tensor,  # key in unified attn
        k_pe: torch.Tensor,  # value in unified attn
        kv_cache: torch.Tensor,
        attn_metadata: "MLACommonMetadata",
        output: torch.Tensor,
        output_scale: torch.Tensor | None = None,
        output_block_scale: torch.Tensor | None = None,
        quant_group_size: int | None = None,
        quant_scale_ue8m0: bool | None = None,
        quant_col_major: bool | None = None,
        quant_tma_aligned: bool | None = None,
    ) -> torch.Tensor:
        assert output is not None, "Output tensor must be provided."

        quant_key = _detect_output_quant_key(
            output, output_scale, output_block_scale, self.num_heads * self.v_head_dim
        )
        if quant_key is not None:
            # The fusion pass has allocated output with quantized dtype
            # (FP8 or uint8 for FP4). We can't write into it directly,
            # so we swap in a temp buffer for computation, then quantize
            # into the real output at the end.
            # NOTE(carlyou): this is temporary until kernels support fp8 output
            quant_output = output
            output = torch.empty(
                output.shape[0],
                self.num_heads * self.v_head_dim,
                dtype=q.dtype,
                device=output.device,
            )

        if attn_metadata is None:
            # During the profile run try to simulate to worse case output size
            # for `self.kv_b_proj(kv_c_normed)` in `_compute_prefill_context`
            # since this can be large
            _ = torch.empty(
                (
                    self.chunked_prefill_workspace_size,
                    self.num_heads,
                    self.qk_nope_head_dim + self.v_head_dim,
                ),
# ... omitted for brevity ...

        return output_padded
```
**EN:** This method defines `forward_impl`. It executes the main forward/runtime path for this component. The main inputs are `q`, `k_c_normed`, `k_pe`, `kv_cache`, `attn_metadata`, `output`. Key calls include `_detect_output_quant_key`, `is_quantized_kv_cache`, `isinstance`, `torch.empty`, `output.fill_`, `kv_cache.view`. It writes or updates `quant_key`, `fp8_attention`, `num_actual_toks`, `output_padded`, `output`, `q`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `forward_impl`。 它执行该组件的主前向/运行时路径。 其主要输入参数包括 `q`, `k_c_normed`, `k_pe`, `kv_cache`, `attn_metadata`, `output`。 关键调用包括 `_detect_output_quant_key`, `is_quantized_kv_cache`, `isinstance`, `torch.empty`, `output.fill_`, `kv_cache.view`。 它会写入或更新 `quant_key`, `fp8_attention`, `num_actual_toks`, `output_padded`, `output`, `q`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 835-933 — method `MLAAttention.process_weights_after_loading`
```python
    def process_weights_after_loading(self, act_dtype: torch.dtype):
        # we currently do not have quantized bmm's which are needed for
        # `W_UV` and `W_UK_T`, we just store fp16/bf16 copies and perform
        # the bmm's in 16-bit, the extra memory overhead of this is fairly low
        kv_b_proj_weight = get_and_maybe_dequant_weights(
            self.kv_b_proj, out_dtype=act_dtype
        ).T

        assert kv_b_proj_weight.shape == (
            self.kv_lora_rank,
            self.num_heads * (self.qk_nope_head_dim + self.v_head_dim),
        ), (
            f"{kv_b_proj_weight.shape=}, "
            f"{self.kv_lora_rank=}, "
            f"{self.num_heads=}, "
            f"{self.qk_nope_head_dim=}, "
            f"{self.v_head_dim=}"
        )
        kv_b_proj_weight = kv_b_proj_weight.view(
            self.kv_lora_rank,
            self.num_heads,
            self.qk_nope_head_dim + self.v_head_dim,
        )

        W_UK, W_UV = kv_b_proj_weight.split(
            [self.qk_nope_head_dim, self.v_head_dim], dim=-1
        )

        # If kv_b_proj_weight is unquantized, quantize it to mxfp4 if supported
        if self.is_aiter_triton_fp4_bmm_enabled:
            from vllm.model_executor.layers.quantization.quark.utils import (
                quark_quantize_weight_to_mxfp4,
            )

            self.W_K, self.W_K_scale = quark_quantize_weight_to_mxfp4(W_UK)
            # Convert from (L, N, P) to (N, L, P)
            self.W_K = self.W_K.transpose(0, 1)
            self.W_K_scale = self.W_K_scale.transpose(0, 1)

            self.W_V, self.W_V_scale = quark_quantize_weight_to_mxfp4(
                W_UV.permute(1, 2, 0)
            )
        elif self.is_aiter_triton_fp8_bmm_enabled:
            W_K = W_UK.transpose(0, 1)  # 16 512 128
# ... omitted for brevity ...
        if not should_load_quant_weights(quant_method):
            set_default_quant_scales(self, register_buffer=False)
```
**EN:** This method defines `process_weights_after_loading`. It handles attention-specific state, metadata, or tensor transforms. The main inputs are `act_dtype`. Key calls include `kv_b_proj_weight.view`, `kv_b_proj_weight.split`, `get_and_maybe_dequant_weights`, `quark_quantize_weight_to_mxfp4`, `self.W_K.transpose`, `self.W_K_scale.transpose`. It writes or updates `kv_b_proj_weight`, `W_UK`, `W_UV`, `quant_method`, `W_K`, `W_K_scale`. The body uses conditional branches and loops to cover different runtime cases.
**CN:** 该方法定义 `process_weights_after_loading`。 它处理注意力相关的状态、元数据或张量变换。 其主要输入参数包括 `act_dtype`。 关键调用包括 `kv_b_proj_weight.view`, `kv_b_proj_weight.split`, `get_and_maybe_dequant_weights`, `quark_quantize_weight_to_mxfp4`, `self.W_K.transpose`, `self.W_K_scale.transpose`。 它会写入或更新 `kv_b_proj_weight`, `W_UK`, `W_UV`, `quant_method`, `W_K`, `W_K_scale`。 函数体通过条件分支和循环来覆盖不同的运行时场景。

### Lines 935-955 — method `MLAAttention.calc_kv_scales`
```python
    def calc_kv_scales(
        self, q: torch.Tensor, kv_c_normed: torch.Tensor, k_pe: torch.Tensor
    ) -> None:
        """Optional scale calculation for MLA inputs.

        Mirrors Attention.calc_kv_scales. Not all MLA backends require this
        """
        # Use safe defaults if ranges are not present
        q_range = getattr(self, "q_range", torch.tensor(1.0))
        k_range = getattr(self, "k_range", torch.tensor(1.0))
        v_range = getattr(self, "v_range", torch.tensor(1.0))

        self._q_scale.copy_(torch.abs(q).max() / q_range)
        # kv_c_normed is the compressed KV representation; use it for k/v
        kv_abs_max = torch.abs(kv_c_normed).max()
        self._k_scale.copy_(kv_abs_max / k_range)
        self._v_scale.copy_(kv_abs_max / v_range)
        self._q_scale_float = self._q_scale.item()
        self._k_scale_float = self._k_scale.item()
        self._v_scale_float = self._v_scale.item()
        self.calculate_kv_scales = False
```
**EN:** This method defines `calc_kv_scales`. Optional scale calculation for MLA inputs. The main inputs are `q`, `kv_c_normed`, `k_pe`. Key calls include `getattr`, `self._q_scale.copy_`, `torch.abs.max`, `self._k_scale.copy_`, `self._v_scale.copy_`, `self._q_scale.item`. It writes or updates `q_range`, `k_range`, `v_range`, `kv_abs_max`, `_q_scale_float`, `_k_scale_float`.
**CN:** 该方法定义 `calc_kv_scales`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `q`, `kv_c_normed`, `k_pe`。 关键调用包括 `getattr`, `self._q_scale.copy_`, `torch.abs.max`, `self._k_scale.copy_`, `self._v_scale.copy_`, `self._q_scale.item`。 它会写入或更新 `q_range`, `k_range`, `v_range`, `kv_abs_max`, `_q_scale_float`, `_k_scale_float`。

### Lines 997-1020 — function `unified_mla_kv_cache_update`
```python
def unified_mla_kv_cache_update(
    kv_c_normed: torch.Tensor,
    k_pe: torch.Tensor,
    layer_name: LayerNameType,
    kv_cache_dtype: str,
    k_scale: torch.Tensor,
) -> torch.Tensor:
    """
    Returns a dummy that is passed to unified_attention to signal a side effect and
    the data dependency between them to ensure torch.compile preserves ordering.
    """
    layer_name = _resolve_layer_name(layer_name)
    _, attn_layer, kv_cache, layer_slot_mapping = get_attention_context(layer_name)
    if layer_slot_mapping is not None:
        attn_layer.impl.do_kv_cache_update(  # type: ignore[attr-defined]
            kv_c_normed,
            k_pe,
            kv_cache,
            layer_slot_mapping,
            kv_cache_dtype,
            k_scale,
        )

    return torch.empty(0, device=kv_c_normed.device, dtype=kv_c_normed.dtype)
```
**EN:** This function defines `unified_mla_kv_cache_update`. Returns a dummy that is passed to unified_attention to signal a side effect and the data dependency between them to ensure torch.compile preserves ordering. The main inputs are `kv_c_normed`, `k_pe`, `layer_name`, `kv_cache_dtype`, `k_scale`. Key calls include `_resolve_layer_name`, `get_attention_context`, `torch.empty`, `attn_layer.impl.do_kv_cache_update`. It writes or updates `layer_name`, `_`, `attn_layer`, `kv_cache`, `layer_slot_mapping`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `unified_mla_kv_cache_update`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `kv_c_normed`, `k_pe`, `layer_name`, `kv_cache_dtype`, `k_scale`。 关键调用包括 `_resolve_layer_name`, `get_attention_context`, `torch.empty`, `attn_layer.impl.do_kv_cache_update`。 它会写入或更新 `layer_name`, `_`, `attn_layer`, `kv_cache`, `layer_slot_mapping`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 1023-1030 — function `unified_mla_kv_cache_update_fake`
```python
def unified_mla_kv_cache_update_fake(
    kv_c_normed: torch.Tensor,
    k_pe: torch.Tensor,
    layer_name: LayerNameType,
    kv_cache_dtype: str,
    k_scale: torch.Tensor,
) -> torch.Tensor:
    return torch.empty(0, device=kv_c_normed.device, dtype=kv_c_normed.dtype)
```
**EN:** This function defines `unified_mla_kv_cache_update_fake`. It handles attention-specific state, metadata, or tensor transforms. The main inputs are `kv_c_normed`, `k_pe`, `layer_name`, `kv_cache_dtype`, `k_scale`. Key calls include `torch.empty`.
**CN:** 该函数定义 `unified_mla_kv_cache_update_fake`。 它处理注意力相关的状态、元数据或张量变换。 其主要输入参数包括 `kv_c_normed`, `k_pe`, `layer_name`, `kv_cache_dtype`, `k_scale`。 关键调用包括 `torch.empty`。

### Lines 1040-1075 — function `unified_mla_attention_with_output`
```python
@eager_break_during_capture
@maybe_transfer_kv_layer
def unified_mla_attention_with_output(
    q: torch.Tensor,
    kv_c_normed: torch.Tensor,
    k_pe: torch.Tensor,
    output: torch.Tensor,
    layer_name: LayerNameType,
    output_scale: torch.Tensor | None = None,
    output_block_scale: torch.Tensor | None = None,
    kv_cache_dummy_dep: torch.Tensor | None = None,
    quant_group_size: int | None = None,
    quant_scale_ue8m0: bool | None = None,
    quant_col_major: bool | None = None,
    quant_tma_aligned: bool | None = None,
) -> None:
    # kv_cache_dummy_dep is not used but accepting it creates a data dependency
    # that ensures torch.compile preserves ordering between KV cache update and
    # attention forward.
    del kv_cache_dummy_dep
    layer_name = _resolve_layer_name(layer_name)
    attn_metadata, layer, kv_cache, _ = get_attention_context(layer_name)
    layer.forward_impl(
        q,
        kv_c_normed,
        k_pe,
        kv_cache,
        attn_metadata,
        output=output,
        output_scale=output_scale,
        output_block_scale=output_block_scale,
        quant_group_size=quant_group_size,
        quant_scale_ue8m0=quant_scale_ue8m0,
        quant_col_major=quant_col_major,
        quant_tma_aligned=quant_tma_aligned,
    )
```
**EN:** This function defines `unified_mla_attention_with_output`. It handles attention-specific state, metadata, or tensor transforms. The main inputs are `q`, `kv_c_normed`, `k_pe`, `output`, `layer_name`, `output_scale`. Key calls include `_resolve_layer_name`, `get_attention_context`, `layer.forward_impl`. It writes or updates `layer_name`, `attn_metadata`, `layer`, `kv_cache`, `_`.
**CN:** 该函数定义 `unified_mla_attention_with_output`。 它处理注意力相关的状态、元数据或张量变换。 其主要输入参数包括 `q`, `kv_c_normed`, `k_pe`, `output`, `layer_name`, `output_scale`。 关键调用包括 `_resolve_layer_name`, `get_attention_context`, `layer.forward_impl`。 它会写入或更新 `layer_name`, `attn_metadata`, `layer`, `kv_cache`, `_`。

### Lines 1078-1092 — function `unified_mla_attention_with_output_fake`
```python
def unified_mla_attention_with_output_fake(
    q: torch.Tensor,
    kv_c_normed: torch.Tensor,
    k_pe: torch.Tensor,
    output: torch.Tensor,
    layer_name: LayerNameType,
    output_scale: torch.Tensor | None = None,
    output_block_scale: torch.Tensor | None = None,
    kv_cache_dummy_dep: torch.Tensor | None = None,
    quant_group_size: int | None = None,
    quant_scale_ue8m0: bool | None = None,
    quant_col_major: bool | None = None,
    quant_tma_aligned: bool | None = None,
) -> None:
    return
```
**EN:** This function defines `unified_mla_attention_with_output_fake`. It handles attention-specific state, metadata, or tensor transforms. The main inputs are `q`, `kv_c_normed`, `k_pe`, `output`, `layer_name`, `output_scale`.
**CN:** 该函数定义 `unified_mla_attention_with_output_fake`。 它处理注意力相关的状态、元数据或张量变换。 其主要输入参数包括 `q`, `kv_c_normed`, `k_pe`, `output`, `layer_name`, `output_scale`。

### Lines 1105-1119 — class `QueryLenSupport`
```python
class QueryLenSupport(Enum):
    """Defines the level of query length support for an attention backend's
    decode pipeline.

    - SINGLE_ONLY: Decode pipeline only supports single-token queries
                   (query_len=1)
    - UNIFORM: Decode pipeline supports uniform multi-token queries
               (all requests must have same query_len > 1)
    - VARLEN: Decode pipeline supports variable-length queries
              (mixed query lengths in same batch)
    """

    SINGLE_ONLY = "single_only"
    UNIFORM = "uniform"
    VARLEN = "varlen"
```
**EN:** This class defines `QueryLenSupport`. It inherits from `Enum`. Defines the level of query length support for an attention backend's decode pipeline. It writes or updates `SINGLE_ONLY`, `UNIFORM`, `VARLEN`.
**CN:** 该类定义了 `QueryLenSupport`。 它继承自 `Enum`。 该类的文档字符串说明了它在当前模块中的职责。 它会写入或更新 `SINGLE_ONLY`, `UNIFORM`, `VARLEN`。

## Key Concepts / 关键概念
- [EN] Attention execution, backend dispatch, and kv-cache handling / [CN] 注意力执行、后端分派与 KV 缓存处理
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Distributed or tensor-parallel coordination / [CN] 分布式或张量并行协同
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `_detect_output_quant_key`, `MLAAttention`, `unified_mla_kv_cache_update`, `unified_mla_kv_cache_update_fake` / [CN] 核心符号：`_detect_output_quant_key`, `MLAAttention`, `unified_mla_kv_cache_update`, `unified_mla_kv_cache_update_fake`

## Dependencies / 依赖关系
- **External**: `functools`, `abc`, `dataclasses`, `enum`, `typing`, `torch`, `torch.nn`, `tqdm` / **外部依赖**: `functools`, `abc`, `dataclasses`, `enum`, `typing`, `torch`, `torch.nn`, `tqdm`
- **Internal**: `vllm.envs`, `vllm`, `vllm._aiter_ops`, `vllm.compilation.breakable_cudagraph`, `vllm.config`, `vllm.distributed.parallel_state`, `vllm.forward_context`, `vllm.logger`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.attention.attention` / **内部依赖**: `vllm.envs`, `vllm`, `vllm._aiter_ops`, `vllm.compilation.breakable_cudagraph`, `vllm.config`, `vllm.distributed.parallel_state`, `vllm.forward_context`, `vllm.logger`, `vllm.model_executor.custom_op`, `vllm.model_executor.layers.attention.attention`
- **Runtime traits**: platform-aware dispatch, custom C++/CUDA ops, Triton kernels, distributed collectives / **运行时特征**: platform-aware dispatch, custom C++/CUDA ops, Triton kernels, distributed collectives
