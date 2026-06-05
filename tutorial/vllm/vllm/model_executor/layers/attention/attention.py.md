# attention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/layers/attention/attention.py`
- **Repository**: vllm-project/vllm
- **Purpose**: attention execution, backend dispatch, and KV-cache handling / 注意力执行、后端分派与 KV 缓存处理

## Line-by-Line Analysis / 逐行分析
### Lines 4-51 — imports and setup
```python
from typing import TYPE_CHECKING, Any

import torch
import torch.nn as nn

import vllm.envs as envs
from vllm.config import CacheConfig, get_current_vllm_config
from vllm.config.vllm import VllmConfig
from vllm.forward_context import ForwardContext, get_forward_context
from vllm.logger import init_logger
from vllm.model_executor.layers.attention.kv_transfer_utils import (
    maybe_transfer_kv_layer,
)
from vllm.model_executor.layers.attention_layer_base import AttentionLayerBase
from vllm.model_executor.layers.linear import (
    UnquantizedLinearMethod,
)
from vllm.model_executor.layers.quantization import QuantizationConfig
from vllm.model_executor.layers.quantization.base_config import QuantizeMethodBase
from vllm.model_executor.layers.quantization.input_quant_fp8 import QuantFP8
from vllm.model_executor.layers.quantization.kv_cache import BaseKVCacheMethod
from vllm.model_executor.layers.quantization.utils.quant_utils import GroupShape
from vllm.platforms import current_platform
from vllm.utils.torch_utils import (
    LayerNameType,
    _encode_layer_name,
    _resolve_layer_name,
    direct_register_custom_op,
    kv_cache_dtype_str_to_dtype,
)
from vllm.v1.attention.backend import (
    AttentionBackend,
    AttentionMetadata,
    AttentionType,
)
from vllm.v1.attention.backends.registry import AttentionBackendEnum
from vllm.v1.attention.selector import get_attn_backend
from vllm.v1.kv_cache_interface import (
    FullAttentionSpec,
    KVCacheSpec,
    SlidingWindowSpec,
    get_kv_quant_mode,
)

if TYPE_CHECKING:
    from vllm.model_executor.layers.attention import MLAAttention

logger = init_logger(__name__)
```
**EN:** This block gathers the imports and module-level setup used by the rest of the file. Key calls include `init_logger`. It writes or updates `logger`. The body uses conditional branches to cover different runtime cases.
**CN:** 该代码块汇总了后续实现依赖的导入项与模块级初始化。 关键调用包括 `init_logger`。 它会写入或更新 `logger`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 54-84 — function `validate_kv_sharing_target`
```python
def validate_kv_sharing_target(
    current_layer_name, target_layer_name, static_forward_context
):
    error_msg = (
        f"Specified KV sharing target layer for {current_layer_name} "
        f"is not valid: target layer {target_layer_name} "
    )

    if current_layer_name == target_layer_name:
        raise ValueError(error_msg + "cannot be the same as the current layer.")

    if target_layer_name not in static_forward_context:
        from vllm.model_executor.models.utils import extract_layer_index

        # If target layer name is not in the static fwd context, it means either
        # a) the target layer does not come BEFORE the current layer, or
        # b) the target layer is not an Attention layer that exists in the model
        current_layer_idx = extract_layer_index(current_layer_name)
        target_layer_idx = extract_layer_index(target_layer_name)
        if current_layer_idx <= target_layer_idx:
            raise ValueError(error_msg + "must come before the current layer.")
        else:
            raise ValueError(error_msg + "is not a valid Attention layer in the model.")

    # Currently KV sharing is only supported between layers of the same type
    target_layer_attn_type = static_forward_context[target_layer_name].attn_type
    expected = static_forward_context[current_layer_name].attn_type
    if target_layer_attn_type != expected:
        raise ValueError(
            error_msg + f"must be the same type as the current layer ({expected})."
        )
```
**EN:** This function defines `validate_kv_sharing_target`. It validates runtime assumptions and raises errors for invalid configurations. The main inputs are `current_layer_name`, `target_layer_name`, `static_forward_context`. Key calls include `ValueError`, `extract_layer_index`. It writes or updates `error_msg`, `target_layer_attn_type`, `expected`, `current_layer_idx`, `target_layer_idx`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `validate_kv_sharing_target`。 它检查运行时假设，并在配置非法时抛出错误。 其主要输入参数包括 `current_layer_name`, `target_layer_name`, `static_forward_context`。 关键调用包括 `ValueError`, `extract_layer_index`。 它会写入或更新 `error_msg`, `target_layer_attn_type`, `expected`, `current_layer_idx`, `target_layer_idx`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 87-91 — function `should_load_quant_weights`
```python
def should_load_quant_weights(quant_method: QuantizeMethodBase | None) -> bool:
    """Returns whether the quantization method should load quantized weights."""
    return quant_method is not None and not isinstance(
        quant_method, UnquantizedLinearMethod
    )
```
**EN:** This function defines `should_load_quant_weights`. Returns whether the quantization method should load quantized weights. The main inputs are `quant_method`. Key calls include `isinstance`.
**CN:** 该函数定义 `should_load_quant_weights`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `quant_method`。 关键调用包括 `isinstance`。

### Lines 94-118 — function `set_default_quant_scales`
```python
def set_default_quant_scales(layer: nn.Module, register_buffer: bool = False) -> None:
    """Sets default quantization scales for the layer."""
    if register_buffer:
        layer.register_buffer("_k_scale", torch.tensor(1.0, dtype=torch.float32))
        layer.register_buffer("_v_scale", torch.tensor(1.0, dtype=torch.float32))
        layer.register_buffer("_q_scale", torch.tensor(1.0, dtype=torch.float32))
        layer.register_buffer("_prob_scale", torch.tensor(1.0, dtype=torch.float32))
    else:
        layer._k_scale.fill_(1.0)
        layer._v_scale.fill_(1.0)
        layer._q_scale.fill_(1.0)
        layer._prob_scale.fill_(1.0)

    # We also keep q/k/v_scale on host (cpu) memory for attention
    # backends that require the scales to be on host instead of on device.
    # e.g. Flashinfer
    layer._q_scale_float = 1.0
    layer._k_scale_float = 1.0
    layer._v_scale_float = 1.0
    layer._prob_scale_float = 1.0

    # Initialize q/k/v range constants used by calc_kv_scales
    layer.q_range = torch.tensor(envs.Q_SCALE_CONSTANT, dtype=torch.float32)
    layer.k_range = torch.tensor(envs.K_SCALE_CONSTANT, dtype=torch.float32)
    layer.v_range = torch.tensor(envs.V_SCALE_CONSTANT, dtype=torch.float32)
```
**EN:** This function defines `set_default_quant_scales`. Sets default quantization scales for the layer. The main inputs are `layer`, `register_buffer`. Key calls include `torch.tensor`, `layer.register_buffer`, `layer._k_scale.fill_`, `layer._v_scale.fill_`, `layer._q_scale.fill_`, `layer._prob_scale.fill_`. It writes or updates `_q_scale_float`, `_k_scale_float`, `_v_scale_float`, `_prob_scale_float`, `q_range`, `k_range`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `set_default_quant_scales`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `layer`, `register_buffer`。 关键调用包括 `torch.tensor`, `layer.register_buffer`, `layer._k_scale.fill_`, `layer._v_scale.fill_`, `layer._q_scale.fill_`, `layer._prob_scale.fill_`。 它会写入或更新 `_q_scale_float`, `_k_scale_float`, `_v_scale_float`, `_prob_scale_float`, `q_range`, `k_range`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 121-174 — function `_init_kv_cache_quant`
```python
def _init_kv_cache_quant(
    layer: nn.Module,
    quant_config: QuantizationConfig | None,
    prefix: str,
) -> None:
    """Initializes KV cache scaling factors and quantization method.

    This helper function sets up the KV cache quantization attributes that are
    shared between Attention and MLAAttention layers. It initializes scale
    tensors for query, key, value, and probability, and configures the
    quantization method if applicable.

    Args:
        layer: The attention layer instance to initialize.
        quant_config: Optional quantization configuration.
        prefix: Layer name prefix for quantization method lookup.
    """

    # Note [Register q/k/v/prob scales in state dict]
    # When calling model.to(device), only parameters/buffers in state dict are
    # moved. If not registering q/k/v/prob scales in state dict, there would
    # be an IMA error when a cuda kernel (e.g., quant_fp8) accesses the tensor
    # on cpu.
    # Registering in state dict means it interacts with weight loading. One edge
    # case is when quant_method is None, or quant_method is UnquantizedLinearMethod
    # (i.e., should_load_quant_weights(quant_method) == False).
    # In this case, the checkpoint does not have the scales. We need to
    # initialize the scales to 1.0 and update the scales after weight loading.
    # This is espectially important when we load dummy weights first (providing
    # wrong scales) and then load real weights (which misses scales and keeps the
    # wrong scales from dummy load).
    set_default_quant_scales(layer, register_buffer=True)

    # The output scale on host memory. This should be the input scale of
    # the quant op after this attention layer.
    layer._o_scale_float = None

    quant_method = (
        quant_config.get_quant_method(layer, prefix=prefix) if quant_config else None
    )

    # See [Note: Register q/k/v/prob scales in state dict]
    if should_load_quant_weights(quant_method):
        assert isinstance(quant_method, BaseKVCacheMethod)
# ... omitted for brevity ...
        layer.quant_method = quant_method
        layer.quant_method.create_weights(layer)
```
**EN:** This function defines `_init_kv_cache_quant`. Initializes KV cache scaling factors and quantization method. The main inputs are `layer`, `quant_config`, `prefix`. Key calls include `set_default_quant_scales`, `should_load_quant_weights`, `quant_config.get_quant_method`, `isinstance`, `layer.quant_method.create_weights`, `ValueError`. It writes or updates `_o_scale_float`, `quant_method`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `_init_kv_cache_quant`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `layer`, `quant_config`, `prefix`。 关键调用包括 `set_default_quant_scales`, `should_load_quant_weights`, `quant_config.get_quant_method`, `isinstance`, `layer.quant_method.create_weights`, `ValueError`。 它会写入或更新 `_o_scale_float`, `quant_method`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 177-610 — class `Attention`
```python
class Attention(nn.Module, AttentionLayerBase):
    """Attention layer.

    This class takes query, key, and value tensors as input. The input tensors
    can either contain prompt tokens or generation tokens.
    The class does the following:

    1. Store the input key and value tensors in the KV cache.
    2. Perform (multi-head/multi-query/grouped-query) attention.
    3. Return the output tensor.
    """

    def __init__(
        self,
# ... omitted for brevity ...
                kv_quant_mode=quant_mode,
            )
```
**EN:** This class defines `Attention`. It inherits from `nn.Module`, `AttentionLayerBase`. Attention layer. Important methods include `__init__`, `forward`, `extra_repr`, `calc_kv_scales`, `process_weights_after_loading`, `get_attn_backend`. Key calls include `super.__init__`, `get_current_vllm_config`, `getattr`, `kv_cache_dtype_str_to_dtype`, `torch.get_default_dtype`, `self.attn_backend.supports_alibi_sqrt`. It writes or updates `sliding_window`, `vllm_config`, `kv_cache_scheme`, `use_per_head_quant_scales`, `kv_cache_torch_dtype`, `kv_cache_dtype`. The body uses conditional branches to cover different runtime cases.
**CN:** 该类定义了 `Attention`。 它继承自 `nn.Module`, `AttentionLayerBase`。 该类的文档字符串说明了它在当前模块中的职责。 重要方法包括 `__init__`, `forward`, `extra_repr`, `calc_kv_scales`, `process_weights_after_loading`, `get_attn_backend`。 关键调用包括 `super.__init__`, `get_current_vllm_config`, `getattr`, `kv_cache_dtype_str_to_dtype`, `torch.get_default_dtype`, `self.attn_backend.supports_alibi_sqrt`。 它会写入或更新 `sliding_window`, `vllm_config`, `kv_cache_scheme`, `use_per_head_quant_scales`, `kv_cache_torch_dtype`, `kv_cache_dtype`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 189-435 — method `Attention.__init__`
```python
    def __init__(
        self,
        num_heads: int,
        head_size: int,
        scale: float,
        num_kv_heads: int | None = None,
        alibi_slopes: list[float] | None = None,
        use_alibi_sqrt: bool | None = None,
        cache_config: CacheConfig | None = None,
        quant_config: QuantizationConfig | None = None,
        logits_soft_cap: float | None = None,
        per_layer_sliding_window: int | None = None,
        prefix: str = "",
        attn_type: str = AttentionType.DECODER,
        kv_sharing_target_layer_name: str | None = None,
        attn_backend: type[AttentionBackend] | None = None,
        head_size_v: int | None = None,
        **extra_impl_args,
    ) -> None:
        """
        The KV cache is stored inside this class and is accessed via
        `self.kv_cache`.
        """
        super().__init__()
        sliding_window: int | None
        if per_layer_sliding_window is not None:
            # per-layer sliding window
            sliding_window = per_layer_sliding_window
        elif cache_config is not None:
            # model-level sliding window
            sliding_window = cache_config.sliding_window
        else:
            sliding_window = None

        vllm_config = get_current_vllm_config()
        if cache_config is not None:
            kv_cache_dtype = cache_config.cache_dtype
            calculate_kv_scales = cache_config.calculate_kv_scales
        else:
            kv_cache_dtype = "auto"
            calculate_kv_scales = False

        # llm-compressor models declare an FP8 KV-cache scheme in their
        # checkpoint config. Honor it only when the user did not explicitly
# ... omitted for brevity ...
                else GroupShape.PER_TENSOR,
            )
```
**EN:** This method defines `__init__`. The KV cache is stored inside this class and is accessed via `self.kv_cache`. The main inputs are `num_heads`, `head_size`, `scale`, `num_kv_heads`, `alibi_slopes`, `use_alibi_sqrt`. Key calls include `super.__init__`, `get_current_vllm_config`, `getattr`, `kv_cache_dtype_str_to_dtype`, `torch.get_default_dtype`, `self.attn_backend.supports_alibi_sqrt`. It writes or updates `sliding_window`, `vllm_config`, `kv_cache_scheme`, `use_per_head_quant_scales`, `kv_cache_torch_dtype`, `kv_cache_dtype`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `__init__`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `num_heads`, `head_size`, `scale`, `num_kv_heads`, `alibi_slopes`, `use_alibi_sqrt`。 关键调用包括 `super.__init__`, `get_current_vllm_config`, `getattr`, `kv_cache_dtype_str_to_dtype`, `torch.get_default_dtype`, `self.attn_backend.supports_alibi_sqrt`。 它会写入或更新 `sliding_window`, `vllm_config`, `kv_cache_scheme`, `use_per_head_quant_scales`, `kv_cache_torch_dtype`, `kv_cache_dtype`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 437-529 — method `Attention.forward`
```python
    def forward(
        self,
        query: torch.Tensor,
        key: torch.Tensor,
        value: torch.Tensor,
        # For some alternate attention backends like MLA the attention output
        # shape does not match the query shape, so we optionally let the model
        # definition specify the output tensor shape.
        output_shape: torch.Size | None = None,
    ) -> torch.Tensor:
        """
        The KV cache is stored inside this class and is accessed via
        `self.kv_cache`.

        Attention metadata (`attn_metadata`) is set using a context manager in
        the model runner's `execute_model` method. It is accessed via forward
        context using
        `vllm.forward_context.get_forward_context().attn_metadata`.
        """
        if self.calculate_kv_scales:
            torch.ops.vllm.maybe_calc_kv_scales(
                query, key, value, _encode_layer_name(self.layer_name)
            )
        output_dtype = query.dtype
        if self.query_quant is not None:
            # quantizing with a simple torch operation enables
            # torch.compile to fuse this into previous ops
            # which reduces overheads during decoding.
            # Otherwise queries are quantized using custom ops
            # which causes decoding overheads
            assert self.kv_cache_dtype in {"fp8", "fp8_e4m3", "nvfp4"}

            # check if query quantization is supported
            if self.impl.supports_quant_query_input:
                query, _ = self.query_quant(query, self._q_scale)

        if output_shape is None:
            # Handle both 2D [num_tokens, hidden] and
            # 3D [num_tokens, heads, head_dim] query
            num_tokens = query.shape[0]
            output_shape = torch.Size((num_tokens, self.num_heads * self.head_size_v))
        output = torch.empty(output_shape, dtype=output_dtype, device=query.device)
        hidden_size = output_shape[-1]
        # Reshape the query, key, and value tensors.
# ... omitted for brevity ...
            )
        return output.view(-1, hidden_size)
```
**EN:** This method defines `forward`. The KV cache is stored inside this class and is accessed via `self.kv_cache`. The main inputs are `query`, `key`, `value`, `output_shape`. Key calls include `torch.empty`, `query.view`, `output.view`, `torch.ops.vllm.maybe_calc_kv_scales`, `torch.Size`, `key.view`. It writes or updates `output_dtype`, `output`, `hidden_size`, `query`, `kv_cache_dummy_dep`, `num_tokens`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `forward`。 该函数/方法的文档字符串直接说明了它的职责。 其主要输入参数包括 `query`, `key`, `value`, `output_shape`。 关键调用包括 `torch.empty`, `query.view`, `output.view`, `torch.ops.vllm.maybe_calc_kv_scales`, `torch.Size`, `key.view`。 它会写入或更新 `output_dtype`, `output`, `hidden_size`, `query`, `kv_cache_dummy_dep`, `num_tokens`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 541-547 — method `Attention.extra_repr`
```python
    def extra_repr(self) -> str:
        s = f"head_size={self.impl.head_size}"  # type: ignore
        s += f", num_heads={self.impl.num_heads}"  # type: ignore
        s += f", num_kv_heads={self.impl.num_kv_heads}"  # type: ignore
        s += f", scale={self.impl.scale}"  # type: ignore
        s += f", backend={self.impl.__class__.__name__}"
        return s
```
**EN:** This method defines `extra_repr`. It handles attention-specific state, metadata, or tensor transforms. It writes or updates `s`.
**CN:** 该方法定义 `extra_repr`。 它处理注意力相关的状态、元数据或张量变换。 它会写入或更新 `s`。

### Lines 531-539 — method `Attention.calc_kv_scales`
```python
    def calc_kv_scales(self, query, key, value):
        self._q_scale.copy_(torch.abs(query).max() / self.q_range)
        self._k_scale.copy_(torch.abs(key).max() / self.k_range)
        self._v_scale.copy_(torch.abs(value).max() / self.v_range)
        self._q_scale_float = self._q_scale.item()
        self._k_scale_float = self._k_scale.item()
        self._v_scale_float = self._v_scale.item()
        # We only calculate the scales once
        self.calculate_kv_scales = False
```
**EN:** This method defines `calc_kv_scales`. It handles attention-specific state, metadata, or tensor transforms. The main inputs are `query`, `key`, `value`. Key calls include `self._q_scale.copy_`, `self._k_scale.copy_`, `self._v_scale.copy_`, `self._q_scale.item`, `self._k_scale.item`, `self._v_scale.item`. It writes or updates `_q_scale_float`, `_k_scale_float`, `_v_scale_float`, `calculate_kv_scales`.
**CN:** 该方法定义 `calc_kv_scales`。 它处理注意力相关的状态、元数据或张量变换。 其主要输入参数包括 `query`, `key`, `value`。 关键调用包括 `self._q_scale.copy_`, `self._k_scale.copy_`, `self._v_scale.copy_`, `self._q_scale.item`, `self._k_scale.item`, `self._v_scale.item`。 它会写入或更新 `_q_scale_float`, `_k_scale_float`, `_v_scale_float`, `calculate_kv_scales`。

### Lines 549-561 — method `Attention.process_weights_after_loading`
```python
    def process_weights_after_loading(self, act_dtype: torch.dtype):
        self.impl.process_weights_after_loading(act_dtype)

        # If we should not load quant weights, we initialize the scales to 1.0
        # as the default value. See [Note: Register q/k/v/prob scales in state dict]
        # for more details.
        quant_method = (
            self.quant_config.get_quant_method(self, prefix=self.layer_name)
            if self.quant_config
            else None
        )
        if not should_load_quant_weights(quant_method):
            set_default_quant_scales(self, register_buffer=False)
```
**EN:** This method defines `process_weights_after_loading`. It handles attention-specific state, metadata, or tensor transforms. The main inputs are `act_dtype`. Key calls include `self.impl.process_weights_after_loading`, `self.quant_config.get_quant_method`, `should_load_quant_weights`, `set_default_quant_scales`. It writes or updates `quant_method`. The body uses conditional branches to cover different runtime cases.
**CN:** 该方法定义 `process_weights_after_loading`。 它处理注意力相关的状态、元数据或张量变换。 其主要输入参数包括 `act_dtype`。 关键调用包括 `self.impl.process_weights_after_loading`, `self.quant_config.get_quant_method`, `should_load_quant_weights`, `set_default_quant_scales`。 它会写入或更新 `quant_method`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 563-564 — method `Attention.get_attn_backend`
```python
    def get_attn_backend(self) -> type[AttentionBackend]:
        return self.attn_backend
```
**EN:** This method defines `get_attn_backend`. It derives and returns computed metadata or outputs needed by later stages.
**CN:** 该方法定义 `get_attn_backend`。 它推导并返回后续阶段所需的元数据或输出。

### Lines 613-628 — function `maybe_calc_kv_scales`
```python
def maybe_calc_kv_scales(
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    layer_name: LayerNameType,
) -> None:
    layer_name = _resolve_layer_name(layer_name)
    forward_context: ForwardContext = get_forward_context()
    self = forward_context.no_compile_layers[layer_name]

    # Only calculate if the layer's calculate_kv_scales flag is True
    # This flag gets set to False after the first forward pass
    if not self.calculate_kv_scales:
        return

    self.calc_kv_scales(query, key, value)
```
**EN:** This function defines `maybe_calc_kv_scales`. It handles attention-specific state, metadata, or tensor transforms. The main inputs are `query`, `key`, `value`, `layer_name`. Key calls include `_resolve_layer_name`, `get_forward_context`, `self.calc_kv_scales`. It writes or updates `layer_name`, `forward_context`, `self`. The body uses conditional branches to cover different runtime cases.
**CN:** 该函数定义 `maybe_calc_kv_scales`。 它处理注意力相关的状态、元数据或张量变换。 其主要输入参数包括 `query`, `key`, `value`, `layer_name`。 关键调用包括 `_resolve_layer_name`, `get_forward_context`, `self.calc_kv_scales`。 它会写入或更新 `layer_name`, `forward_context`, `self`。 函数体通过条件分支来覆盖不同的运行时场景。

### Lines 631-637 — function `maybe_calc_kv_scales_fake`
```python
def maybe_calc_kv_scales_fake(
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    layer_name: LayerNameType,
) -> None:
    return
```
**EN:** This function defines `maybe_calc_kv_scales_fake`. It handles attention-specific state, metadata, or tensor transforms. The main inputs are `query`, `key`, `value`, `layer_name`.
**CN:** 该函数定义 `maybe_calc_kv_scales_fake`。 它处理注意力相关的状态、元数据或张量变换。 其主要输入参数包括 `query`, `key`, `value`, `layer_name`。

## Key Concepts / 关键概念
- [EN] Attention execution, backend dispatch, and kv-cache handling / [CN] 注意力执行、后端分派与 KV 缓存处理
- [EN] PyTorch tensor computation / [CN] PyTorch 张量计算
- [EN] Quantization-aware execution / [CN] 量化感知执行
- [EN] Core symbols: `validate_kv_sharing_target`, `should_load_quant_weights`, `set_default_quant_scales`, `_init_kv_cache_quant` / [CN] 核心符号：`validate_kv_sharing_target`, `should_load_quant_weights`, `set_default_quant_scales`, `_init_kv_cache_quant`

## Dependencies / 依赖关系
- **External**: `typing`, `torch`, `torch.nn` / **外部依赖**: `typing`, `torch`, `torch.nn`
- **Internal**: `vllm.envs`, `vllm.config`, `vllm.config.vllm`, `vllm.forward_context`, `vllm.logger`, `vllm.model_executor.layers.attention.kv_transfer_utils`, `vllm.model_executor.layers.attention_layer_base`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`, `vllm.model_executor.layers.quantization.base_config` / **内部依赖**: `vllm.envs`, `vllm.config`, `vllm.config.vllm`, `vllm.forward_context`, `vllm.logger`, `vllm.model_executor.layers.attention.kv_transfer_utils`, `vllm.model_executor.layers.attention_layer_base`, `vllm.model_executor.layers.linear`, `vllm.model_executor.layers.quantization`, `vllm.model_executor.layers.quantization.base_config`
- **Runtime traits**: platform-aware dispatch / **运行时特征**: platform-aware dispatch
