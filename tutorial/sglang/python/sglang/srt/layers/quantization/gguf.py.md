# gguf.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/quantization/gguf.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module defines configuration objects and method-selection logic for gguf quantization in SGLang SRT. / 该模块定义了用于 SGLang SRT 的 GGUF 量化配置对象与方法选择逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-24: module imports and setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
# Adapted from: https://github.com/vllm-project/vllm/blob/ab3e80042eac24dd362408e6d63ad98768046359/vllm/model_executor/layers/quantization/gguf.py
from __future__ import annotations

import logging
import warnings
from typing import TYPE_CHECKING, Any, List, Optional

import gguf
import torch
from gguf import GGMLQuantizationType as WeightType
from torch.nn.parameter import Parameter, UninitializedParameter

from sglang.srt.layers.linear import LinearBase
from sglang.srt.layers.moe import MoeRunnerConfig
from sglang.srt.layers.quantization.base_config import (
    FusedMoEMethodBase,
    LinearMethodBase,
    QuantizationConfig,
    QuantizeMethodBase,
)
from sglang.srt.layers.quantization.unquant import UnquantizedLinearMethod
from sglang.srt.utils import is_cuda, is_hip, is_musa, is_npu, is_xpu, set_weight_attrs
```
**EN:** This block imports __future__, gguf, logging, sgl_kernel, sglang.jit_kernel.activation, sglang.srt.distributed.communication_op, sglang.srt.layers.linear, sglang.srt.layers.moe and prepares the module namespace for the quantization implementation.
**CN:** 该代码块导入 __future__, gguf, logging, sgl_kernel, sglang.jit_kernel.activation, sglang.srt.distributed.communication_op, sglang.srt.layers.linear, sglang.srt.layers.moe 等依赖，并为当前量化实现准备模块命名空间。

### Lines 26-30: typing-only imports
```python
if TYPE_CHECKING:
    from sglang.srt.layers.moe.token_dispatcher import (
        CombineInput,
        StandardDispatchOutput,
    )
```
**EN:** This conditional block imports symbols that are only needed for static typing and do not affect runtime behavior.
**CN:** 该条件代码块只导入静态类型检查需要的符号，不影响运行时行为。

### Lines 32-32: initialize _is_cuda
```python
_is_cuda = is_cuda()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_cuda.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_cuda。

### Lines 33-33: initialize _is_hip
```python
_is_hip = is_hip()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_hip.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_hip。

### Lines 34-34: initialize _is_xpu
```python
_is_xpu = is_xpu()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_xpu.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_xpu。

### Lines 35-35: initialize _is_musa
```python
_is_musa = is_musa()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_musa.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_musa。

### Lines 36-36: initialize _is_npu
```python
_is_npu = is_npu()
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as _is_npu.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 _is_npu。

### Lines 38-64: conditional logic for _is_cuda
```python
if _is_cuda:
    from sgl_kernel import moe_align_block_size, moe_sum
    from sgl_kernel.quantization import (
        ggml_dequantize,
        ggml_moe_a8,
        ggml_moe_a8_vec,
        ggml_moe_get_block_size,
        ggml_mul_mat_a8,
        ggml_mul_mat_vec_a8,
    )

    from sglang.jit_kernel.activation import gelu_and_mul, silu_and_mul
elif _is_musa:
    from sgl_kernel import gelu_and_mul, moe_align_block_size, moe_sum, silu_and_mul
    from sgl_kernel.quantization import (
        ggml_dequantize,
        ggml_moe_a8,
        ggml_moe_a8_vec,
        ggml_moe_get_block_size,
        ggml_mul_mat_a8,
        ggml_mul_mat_vec_a8,
    )
elif _is_npu:
    from gguf import dequantize as gguf_dequantize
else:
    if not _is_hip:
        warnings.warn(f"Only CUDA, MUSA and NPU support GGUF quantization currently.")
```
**EN:** This block applies conditional logic controlled by `_is_cuda`, typically for platform-specific or feature-specific behavior.
**CN:** 该代码块根据 `_is_cuda` 执行条件逻辑，通常用于平台相关或特性相关行为。

### Lines 66-66: initialize logger
```python
logger = logging.getLogger(__name__)
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as logger.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 logger。

### Lines 69-71: class GGUFConfig: definition
```python
class GGUFConfig(QuantizationConfig):
    """Config class for GGUF."""
```
**EN:** This block declares `GGUFConfig`, a configuration class for the quantization stack. It organizes behaviors such as __init__, __repr__, get_scaled_act_names, get_name.
**CN:** 该代码块声明 `GGUFConfig`，它是量化栈中的配置类，组织了 __init__, __repr__, get_scaled_act_names, get_name 等行为。

### Lines 72-76: GGUFConfig.__init__()
```python
    def __init__(self, modules_to_not_convert: list[str] | None = None) -> None:
        super().__init__()
        if _is_hip:
            warnings.warn(f"Only CUDA and MUSA support GGUF quantization currently.")
        self.modules_to_not_convert = modules_to_not_convert or []
```
**EN:** This block defines `GGUFConfig.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `GGUFConfig.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 78-79: GGUFConfig.__repr__()
```python
    def __repr__(self) -> str:
        return "GGUFConfig()"
```
**EN:** This block defines `GGUFConfig.__repr__()`, which builds a readable debug representation.
**CN:** 该代码块定义了 `GGUFConfig.__repr__()`，用于构造可读的调试表示。

### Lines 81-82: GGUFConfig.get_scaled_act_names()
```python
    def get_scaled_act_names(self) -> List[str]:
        return []
```
**EN:** This block defines `GGUFConfig.get_scaled_act_names()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `GGUFConfig.get_scaled_act_names()`，用于为调用方获取或计算派生值。

### Lines 84-85: GGUFConfig.get_name()
```python
    def get_name(self) -> "str":
        return "gguf"
```
**EN:** This block defines `GGUFConfig.get_name()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `GGUFConfig.get_name()`，用于为调用方获取或计算派生值。

### Lines 87-88: GGUFConfig.get_supported_act_dtypes()
```python
    def get_supported_act_dtypes(self) -> list[torch.dtype]:
        return [torch.half, torch.bfloat16, torch.float32]
```
**EN:** This block defines `GGUFConfig.get_supported_act_dtypes()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `GGUFConfig.get_supported_act_dtypes()`，用于为调用方获取或计算派生值。

### Lines 90-92: GGUFConfig.get_min_capability()
```python
    @classmethod
    def get_min_capability(cls) -> int:
        return 60 if not _is_musa else 21
```
**EN:** This block defines `GGUFConfig.get_min_capability()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `GGUFConfig.get_min_capability()`，用于为调用方获取或计算派生值。

### Lines 94-96: GGUFConfig.get_config_filenames()
```python
    @classmethod
    def get_config_filenames(cls) -> list[str]:
        return []  # no extra configs.
```
**EN:** This block defines `GGUFConfig.get_config_filenames()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `GGUFConfig.get_config_filenames()`，用于为调用方获取或计算派生值。

### Lines 98-103: GGUFConfig.from_config()
```python
    @classmethod
    def from_config(cls, config: dict[str, Any]) -> "GGUFConfig":
        modules_to_not_convert = cls.get_from_keys_or(
            config, ["modules_to_not_convert"], None
        )
        return cls(modules_to_not_convert)
```
**EN:** This block defines `GGUFConfig.from_config()`, which constructs an object from serialized configuration data.
**CN:** 该代码块定义了 `GGUFConfig.from_config()`，用于从序列化配置数据构造对象。

### Lines 105-125: GGUFConfig.get_quant_method()
```python
    def get_quant_method(
        self, layer: torch.nn.Module, prefix: str
    ) -> Optional["QuantizeMethodBase"]:
        from sglang.srt.layers.moe.fused_moe_triton import FusedMoE
        from sglang.srt.layers.vocab_parallel_embedding import VocabParallelEmbedding

        if isinstance(layer, LinearBase):
            if is_layer_skipped_gguf(prefix, self.modules_to_not_convert):
                return UnquantizedLinearMethod()
            if _is_npu:
                return GGUFLinearAscendMethod(self)
            return GGUFLinearMethod(self)
        elif isinstance(layer, VocabParallelEmbedding):
            if _is_npu:
                return GGUFEmbeddingAscendMethod(self)
            return GGUFEmbeddingMethod(self)
        elif isinstance(layer, FusedMoE):
            if _is_npu:
                return GGUFMoEAscendMethod(self)
            return GGUFMoEMethod(self)
        return None
```
**EN:** This block defines `GGUFConfig.get_quant_method()`, which retrieves or computes a derived value for callers.
**CN:** 该代码块定义了 `GGUFConfig.get_quant_method()`，用于为调用方获取或计算派生值。

### Lines 128-129: is_layer_skipped_gguf()
```python
def is_layer_skipped_gguf(prefix: str, modules_to_not_convert: list[str]):
    return any(module_name in prefix for module_name in modules_to_not_convert)
```
**EN:** This block defines `is_layer_skipped_gguf()`, which checks a condition used by higher-level control flow.
**CN:** 该代码块定义了 `is_layer_skipped_gguf()`，用于检查供上层控制流使用的条件。

### Lines 132-132: initialize UNQUANTIZED_TYPES
```python
UNQUANTIZED_TYPES = {WeightType.F32, WeightType.F16, WeightType.BF16}
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as UNQUANTIZED_TYPES.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 UNQUANTIZED_TYPES。

### Lines 133-140: initialize STANDARD_QUANT_TYPES
```python
STANDARD_QUANT_TYPES = {
    WeightType.Q4_0,
    WeightType.Q4_1,
    WeightType.Q5_0,
    WeightType.Q5_1,
    WeightType.Q8_0,
    WeightType.Q8_1,
}
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as STANDARD_QUANT_TYPES.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 STANDARD_QUANT_TYPES。

### Lines 141-147: initialize KQUANT_TYPES
```python
KQUANT_TYPES = {
    WeightType.Q2_K,
    WeightType.Q3_K,
    WeightType.Q4_K,
    WeightType.Q5_K,
    WeightType.Q6_K,
}
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as KQUANT_TYPES.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 KQUANT_TYPES。

### Lines 148-158: initialize IMATRIX_QUANT_TYPES
```python
IMATRIX_QUANT_TYPES = {
    WeightType.IQ1_M,
    WeightType.IQ1_S,
    WeightType.IQ2_XXS,
    WeightType.IQ2_XS,
    WeightType.IQ2_S,
    WeightType.IQ3_XXS,
    WeightType.IQ3_S,
    WeightType.IQ4_XS,
    WeightType.IQ4_NL,
}
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as IMATRIX_QUANT_TYPES.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 IMATRIX_QUANT_TYPES。

### Lines 162-162: initialize DEQUANT_TYPES
```python
DEQUANT_TYPES = STANDARD_QUANT_TYPES | KQUANT_TYPES | IMATRIX_QUANT_TYPES
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as DEQUANT_TYPES.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 DEQUANT_TYPES。

### Lines 163-163: initialize MMVQ_QUANT_TYPES
```python
MMVQ_QUANT_TYPES = STANDARD_QUANT_TYPES | KQUANT_TYPES | IMATRIX_QUANT_TYPES
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as MMVQ_QUANT_TYPES.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 MMVQ_QUANT_TYPES。

### Lines 164-164: initialize MMQ_QUANT_TYPES
```python
MMQ_QUANT_TYPES = STANDARD_QUANT_TYPES | KQUANT_TYPES
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as MMQ_QUANT_TYPES.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 MMQ_QUANT_TYPES。

### Lines 167-199: fused_mul_mat_gguf()
```python
def fused_mul_mat_gguf(
    x: torch.Tensor, qweight: torch.Tensor, qweight_type: int
) -> torch.Tensor:
    if qweight_type in IMATRIX_QUANT_TYPES:
        mmvq_safe = 8 if qweight.shape[0] > 5120 else 16
    else:
        mmvq_safe = 2 if qweight.shape[0] > 5120 else 6
    # HACK: when doing chunked prefill we don't generate output tokens
    # so input to logits generator is empty which causes invalid parameter
    if x.shape[0] == 0:
        return torch.empty(x.shape[0], qweight.shape[0], dtype=x.dtype, device=x.device)
    # there is no need to call any kernel for fp16/bf16
    if qweight_type in UNQUANTIZED_TYPES:
        return x @ qweight.T
    # enable MMVQ in contiguous batching with batch_size=1
    if x.shape[0] <= mmvq_safe and qweight_type in MMVQ_QUANT_TYPES:
        y = ggml_mul_mat_vec_a8(qweight, x, qweight_type, qweight.shape[0])
    # Use MMQ Kernel if it's available (standard + k-quants)
    elif qweight_type in MMQ_QUANT_TYPES:
        y = ggml_mul_mat_a8(qweight, x, qweight_type, qweight.shape[0])
    # If there is no available MMQ kernel, fallback to dequantize
    elif qweight_type in DEQUANT_TYPES:
        block_size, type_size = gguf.GGML_QUANT_SIZES[qweight_type]
        shape = (qweight.shape[0], qweight.shape[1] // type_size * block_size)
        weight = ggml_dequantize(qweight, qweight_type, *shape, x.dtype)
        y = x @ weight.T
    else:
        # Raise an error if the quantization type is not supported.
        # Might be useful if llama.cpp adds a new quantization type.
        # Wrap to GGMLQuantizationType IntEnum to make sure it's a valid type.
        qweight_type = WeightType(qweight_type)
        raise NotImplementedError(f"Unsupported GGUF quantization type: {qweight_type}")
    return y
```
**EN:** This block defines `fused_mul_mat_gguf()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `fused_mul_mat_gguf()`，用于实现量化栈中的可复用模块逻辑。

### Lines 202-246: fused_moe_gguf() (part 1/3)
```python
def fused_moe_gguf(
    x: torch.Tensor,
    w1: torch.Tensor,
    w2: torch.Tensor,
    topk_weights: torch.Tensor,
    topk_ids: torch.Tensor,
    qweight_type: int,
    qweight_type2: int,
    activation: str,
) -> torch.Tensor:
    def act(x: torch.Tensor):
        if activation == "silu":
            return silu_and_mul(x)
        elif activation == "gelu":
            return gelu_and_mul(x)
        raise ValueError(f"Unsupported activation: {activation}")

    out_hidden_states = torch.empty_like(x)
    # unless we decent expert reuse we are better off running moe_vec kernel
    if (
        qweight_type2 in MMQ_QUANT_TYPES
        and qweight_type in MMQ_QUANT_TYPES
        and x.shape[0] > 64
    ):
        num_tokens, _ = x.shape
        E, N, _ = w1.shape
        top_k = topk_ids.shape[1]
        BLOCK_SIZE = ggml_moe_get_block_size(qweight_type)

        sorted_token_ids, expert_ids, num_tokens_post_padded = moe_align_block_size(
            topk_ids, BLOCK_SIZE, E
        )
        out = ggml_moe_a8(
            x,
            w1,
            sorted_token_ids,
            expert_ids,
            num_tokens_post_padded,
            qweight_type,
            N,
            top_k,
            num_tokens,
        )
        out = act(out)
        out = ggml_moe_a8(
```
**EN:** This segment of `fused_moe_gguf()` implements reusable module logic for the quantization stack. It covers one portion of the full implementation.
**CN:** `fused_moe_gguf()` 的这一段代码用于实现量化栈中的可复用模块逻辑，展示了完整实现中的一部分。

### Lines 247-291: fused_moe_gguf() (part 2/3)
```python
            out,
            w2,
            sorted_token_ids,
            expert_ids,
            num_tokens_post_padded,
            qweight_type2,
            w2.shape[1],
            1,
            num_tokens * top_k,
        )
        out = out.reshape(num_tokens, top_k, w2.shape[1]).mul_(
            topk_weights.view(num_tokens, top_k, 1)
        )
        # TODO(FlamingoPg): maybe we can use moe_sum_reduce here?
        moe_sum(out, out_hidden_states)
    elif qweight_type2 in MMVQ_QUANT_TYPES and qweight_type in MMVQ_QUANT_TYPES:
        num_tokens, _ = x.shape
        E, N, _ = w1.shape
        top_k = topk_ids.shape[1]

        out = ggml_moe_a8_vec(x, w1, topk_ids, top_k, qweight_type, N, num_tokens)
        out = act(out)

        out = ggml_moe_a8_vec(
            out, w2, topk_ids, 1, qweight_type2, w2.shape[1], num_tokens * top_k
        )
        out = out.reshape(num_tokens, top_k, w2.shape[1]).mul_(
            topk_weights.view(num_tokens, top_k, 1)
        )
        moe_sum(out, out_hidden_states)
    else:
        logger.warning_once(
            "There is no support for fast MoE kernel "
            "for current quantization method. "
            "Falling back to slow implementation. "
        )
        for tok, (w, idx) in enumerate(zip(topk_weights, topk_ids)):
            inp = x[tok].reshape((1,) + x.shape[1:])
            current_hidden_state = None
            for ww, ii in zip(w, idx):
                expert_up = w1[ii]

                out = fused_mul_mat_gguf(inp, expert_up, qweight_type)
                out = act(out)
```
**EN:** This segment of `fused_moe_gguf()` implements reusable module logic for the quantization stack. It covers one portion of the full implementation.
**CN:** `fused_moe_gguf()` 的这一段代码用于实现量化栈中的可复用模块逻辑，展示了完整实现中的一部分。

### Lines 292-301: fused_moe_gguf() (part 3/3)
```python
                expert_down = w2[ii]
                current_state = fused_mul_mat_gguf(
                    out, expert_down, qweight_type2
                ).mul_(ww)
                if current_hidden_state is None:
                    current_hidden_state = current_state
                else:
                    current_hidden_state.add_(current_state)
            out_hidden_states[tok] = current_hidden_state
    return out_hidden_states
```
**EN:** This segment of `fused_moe_gguf()` implements reusable module logic for the quantization stack. It covers one portion of the full implementation.
**CN:** `fused_moe_gguf()` 的这一段代码用于实现量化栈中的可复用模块逻辑，展示了完整实现中的一部分。

### Lines 304-324: apply_gguf_embedding()
```python
def apply_gguf_embedding(
    x: torch.Tensor,
    qweight: torch.Tensor,
    qweight_type: int,
    hidden_size: int,
    dtype: torch.dtype | None = None,
) -> torch.Tensor:
    if qweight_type in UNQUANTIZED_TYPES:
        return torch.embedding(qweight, x)
    elif qweight_type in DEQUANT_TYPES:
        block_size, type_size = gguf.GGML_QUANT_SIZES[qweight_type]
        x_flat = x.flatten()
        assert hidden_size == qweight.shape[1] // type_size * block_size
        quant = torch.index_select(qweight, dim=0, index=x_flat)
        dequant = ggml_dequantize(
            quant, qweight_type, hidden_size, x_flat.shape[0], dtype
        )
        return dequant.view(*x.shape, hidden_size)
    else:
        qweight_type = WeightType(qweight_type)
        raise NotImplementedError(f"Unsupported GGUF quantization type: {qweight_type}")
```
**EN:** This block defines `apply_gguf_embedding()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `apply_gguf_embedding()`，用于将量化计算应用到运行时输入上。

### Lines 327-333: class GGUFLinearMethod: definition
```python
class GGUFLinearMethod(LinearMethodBase):
    """Linear method for GGUF.

    Args:
        quant_config: The GGUF quantization config.
    """
```
**EN:** This block declares `GGUFLinearMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, process_weights_after_loading, _create_padded_weight_param.
**CN:** 该代码块声明 `GGUFLinearMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, process_weights_after_loading, _create_padded_weight_param 等行为。

### Lines 334-335: GGUFLinearMethod.__init__()
```python
    def __init__(self, quant_config: GGUFConfig):
        self.quant_config = quant_config
```
**EN:** This block defines `GGUFLinearMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `GGUFLinearMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 337-381: GGUFLinearMethod.create_weights()
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: list[int],
        input_size: int,
        output_size: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        self.params_dtype = params_dtype
        output_size_per_partition = sum(output_partition_sizes)

        tensor_shape = (output_size_per_partition, input_size_per_partition)
        qweight = GGUFUninitializedParameter(requires_grad=False)
        set_weight_attrs(
            qweight,
            {
                "input_dim": 1,
                "output_dim": 0,
                "tensor_shape": tensor_shape,
                "is_gguf_weight": True,
                "data_container": [],
                "shard_id": [],
                "shard_id_map": {},
            },
        )
        set_weight_attrs(qweight, extra_weight_attrs)
        layer.register_parameter("qweight", qweight)

        qweight_type = Parameter(
            torch.empty(len(output_partition_sizes), dtype=torch.uint8),
            requires_grad=False,
        )
        set_weight_attrs(
            qweight_type,
            {
                "is_gguf_weight_type": True,
                "weight_type": 0,
                "shard_weight_type": {},
                "ignore_warning": True,
            },
        )
        set_weight_attrs(qweight_type, extra_weight_attrs)
        layer.register_parameter("qweight_type", qweight_type)
```
**EN:** This block defines `GGUFLinearMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `GGUFLinearMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 383-392: GGUFLinearMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module):
        qweight_type = layer.qweight_type.weight_type
        if not (qweight_type in UNQUANTIZED_TYPES or qweight_type in DEQUANT_TYPES):
            qweight_type = WeightType(qweight_type)
            raise ValueError(
                f"Unsupported GGUF quantization type {qweight_type} in layer {layer}."
            )
        # For MergedColumnParallelLinear and QKVParallelLinear, we need to
        # materialize the padded weight parameter for CUDA Graph compatibility.
        self._create_padded_weight_param(layer)
```
**EN:** This block defines `GGUFLinearMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `GGUFLinearMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 394-426: GGUFLinearMethod._create_padded_weight_param()
```python
    def _create_padded_weight_param(self, layer: torch.nn.Module):
        """Create padded weight parameter for GGUF MergedLinear layer."""
        qweight = layer.qweight
        shard_id_map = qweight.shard_id_map
        shard_id = qweight.shard_id
        if len(data_container := qweight.data_container) > 1:
            dtype = {data.dtype for data in data_container}
            assert len(dtype) == 1, ValueError(
                f"Data container has mixed dtypes: {dtype}"
            )
            dtype = next(iter(dtype))
            # concat dim0 and pad dim1
            padded_side = max(x.size(1) for x in data_container)
            concat_side = sum(x.size(0) for x in data_container)
            # Pad the quantized weights to dense tensor, and create a map
            # with the location of each shard in the padded tensor.
            padded_data = torch.zeros(
                (concat_side, padded_side), dtype=dtype, device=qweight.device
            )
            # (dim0_start, dim0_end, dim1_size)
            shard_offset_map = dict[str, tuple[int, int, int]]()
            for idx in shard_id:
                id_in_container = shard_id_map[idx]
                start = sum(x.size(0) for x in data_container[:id_in_container])
                end = start + data_container[id_in_container].size(0)
                size = data_container[id_in_container].size(1)
                padded_data[start:end, :size] = data_container[id_in_container]
                shard_offset_map[idx] = (start, end, size)
            qweight.data_container.clear()
            padded_param = Parameter(padded_data, requires_grad=False)
            set_weight_attrs(padded_param, vars(qweight))
            set_weight_attrs(padded_param, {"shard_offset_map": shard_offset_map})
            layer.register_parameter("qweight", padded_param)
```
**EN:** This block defines `GGUFLinearMethod._create_padded_weight_param()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `GGUFLinearMethod._create_padded_weight_param()`，用于实现量化栈中的可复用模块逻辑。

### Lines 428-456: GGUFLinearMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        shard_id = layer.qweight.shard_id

        if shard_id:
            # dequantize shard weights respectively
            shard_id = ["q", "k", "v"] if "q" in shard_id else shard_id
            qweight = layer.qweight
            result = []
            for idx in shard_id:
                start, end, offset = layer.qweight.shard_offset_map[idx]
                qweight_type = layer.qweight_type.shard_weight_type[idx]
                result.append(
                    fused_mul_mat_gguf(
                        x, qweight[start:end, :offset].contiguous(), qweight_type
                    )
                )
            out = torch.cat(result, axis=1)
        else:
            qweight = layer.qweight
            qweight_type = layer.qweight_type.weight_type
            out = fused_mul_mat_gguf(x, qweight, qweight_type)
        if bias is not None:
            out.add_(bias)
        return out
```
**EN:** This block defines `GGUFLinearMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `GGUFLinearMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 459-465: class GGUFMoEMethod: definition
```python
class GGUFMoEMethod(FusedMoEMethodBase):
    """MoE method for GGUF.

    Args:
        quant_config: The GGUF quantization config.
    """
```
**EN:** This block declares `GGUFMoEMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, create_moe_runner, apply.
**CN:** 该代码块声明 `GGUFMoEMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, create_moe_runner, apply 等行为。

### Lines 466-467: GGUFMoEMethod.__init__()
```python
    def __init__(self, quant_config: GGUFConfig):
        self.quant_config = quant_config
```
**EN:** This block defines `GGUFMoEMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `GGUFMoEMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 469-529: GGUFMoEMethod.create_weights()
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        tensor_shape = (num_experts, 2 * intermediate_size_per_partition, hidden_size)
        # gate up proj
        w13_qweight = GGUFUninitializedParameter(requires_grad=False)
        set_weight_attrs(
            w13_qweight,
            {
                "input_dim": 1,
                "output_dim": 0,
                "tensor_shape": tensor_shape,
                "is_gguf_weight": True,
                "data_container": [],
            },
        )
        set_weight_attrs(w13_qweight, extra_weight_attrs)
        layer.register_parameter("w13_qweight", w13_qweight)

        w13_qweight_type = Parameter(
            torch.empty(1, dtype=torch.uint8), requires_grad=False
        )
        set_weight_attrs(
            w13_qweight_type,
            {"is_gguf_weight_type": True, "weight_type": 0, "ignore_warning": True},
        )
        set_weight_attrs(w13_qweight_type, extra_weight_attrs)
        layer.register_parameter("w13_qweight_type", w13_qweight_type)

        tensor_shape = (num_experts, intermediate_size_per_partition, hidden_size)
        # gate down proj
        w2_qweight = GGUFUninitializedParameter(requires_grad=False)
        set_weight_attrs(
            w2_qweight,
            {
                "input_dim": 1,
                "output_dim": 0,
                "tensor_shape": tensor_shape,
                "is_gguf_weight": True,
                "data_container": [],
            },
        )
        set_weight_attrs(w2_qweight, extra_weight_attrs)
        layer.register_parameter("w2_qweight", w2_qweight)

        w2_qweight_type = Parameter(
            torch.empty(1, dtype=torch.uint8), requires_grad=False
        )
        set_weight_attrs(
            w2_qweight_type,
            {"is_gguf_weight_type": True, "weight_type": 0, "ignore_warning": True},
        )

        set_weight_attrs(w2_qweight_type, extra_weight_attrs)
        layer.register_parameter("w2_qweight_type", w2_qweight_type)
```
**EN:** This block defines `GGUFMoEMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `GGUFMoEMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 531-534: GGUFMoEMethod.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        self.moe_runner_config = moe_runner_config
```
**EN:** This block defines `GGUFMoEMethod.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `GGUFMoEMethod.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 536-565: GGUFMoEMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> CombineInput:
        assert self.fused_experts is None

        from sglang.srt.layers.moe.token_dispatcher import StandardCombineInput

        assert (
            self.moe_runner_config.activation == "silu"
        ), "Only SiLU activation is supported."

        x = dispatch_output.hidden_states
        topk_output = dispatch_output.topk_output

        moe_runner_config = self.moe_runner_config

        topk_weights, topk_ids, _ = topk_output
        output = fused_moe_gguf(
            x=x,
            w1=layer.w13_qweight,
            w2=layer.w2_qweight,
            topk_weights=topk_weights,
            topk_ids=topk_ids,
            qweight_type=layer.w13_qweight_type.weight_type,
            qweight_type2=layer.w2_qweight_type.weight_type,
            activation=moe_runner_config.activation,
        )
        return StandardCombineInput(hidden_states=output)
```
**EN:** This block defines `GGUFMoEMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `GGUFMoEMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 568-574: class GGUFEmbeddingMethod: definition
```python
class GGUFEmbeddingMethod(GGUFLinearMethod):
    """Embedding method for GGUF.

    Args:
        quant_config: The GGUF quantization config.
    """
```
**EN:** This block declares `GGUFEmbeddingMethod`, a runtime method class for the quantization stack. It organizes behaviors such as embedding.
**CN:** 该代码块声明 `GGUFEmbeddingMethod`，它是量化栈中的运行方法类，组织了 embedding 等行为。

### Lines 575-582: GGUFEmbeddingMethod.embedding()
```python
    def embedding(self, layer: torch.nn.Module, x: torch.Tensor) -> torch.Tensor:
        qweight = layer.qweight
        qweight_type = layer.qweight_type.weight_type
        hidden_size = qweight.tensor_shape[1]

        return apply_gguf_embedding(
            x, qweight, qweight_type, hidden_size, dtype=self.params_dtype
        )
```
**EN:** This block defines `GGUFEmbeddingMethod.embedding()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `GGUFEmbeddingMethod.embedding()`，用于实现量化栈中的可复用模块逻辑。

### Lines 585-585: class GGUFUninitializedParameter: definition
```python
class GGUFUninitializedParameter(UninitializedParameter):
```
**EN:** This block declares `GGUFUninitializedParameter`, a supporting class for the quantization stack. It organizes behaviors such as class state.
**CN:** 该代码块声明 `GGUFUninitializedParameter`，它是量化栈中的支撑类，组织了 class state 等行为。

### Lines 586-586: GGUFUninitializedParameter member: initialize cls_to_become
```python
    cls_to_become = Parameter
```
**EN:** This block initializes module-level constants, lookup tables, or flags such as cls_to_become.
**CN:** 该代码块初始化模块级常量、查找表或标志位，例如 cls_to_become。

### Lines 587-587: GGUFUninitializedParameter member: annotate data_container
```python
    data_container: list[torch.Tensor]
```
**EN:** This block declares and initializes the annotated symbol `data_container`.
**CN:** 该代码块声明并初始化带类型注解的符号 `data_container`。

### Lines 593-617: ggml_dequantize_ascend()
```python
def ggml_dequantize_ascend(
    qweight: torch.Tensor,
    qweight_type: int,
    rows: int,
    cols: int,
    dtype: torch.dtype,
) -> torch.Tensor:
    """Dequantize GGML quantized weights for NPU.

    Uses gguf library's reference implementation which supports all GGML formats
    and is guaranteed to be correct. The dequantization runs on CPU during model
    loading, then the dequantized weights are transferred to NPU for inference.
    """

    # Move to CPU for dequantization using gguf library
    qweight_cpu = qweight.cpu().numpy()

    # Use gguf library's dequantize (supports all GGML formats)
    dequant_np = gguf_dequantize(qweight_cpu, qweight_type)

    # Convert to torch and move to target device
    result = torch.from_numpy(dequant_np).to(dtype=dtype, device=qweight.device)
    result = result.reshape(rows, cols)

    return result
```
**EN:** This block defines `ggml_dequantize_ascend()`, which restores low-precision data for downstream compute.
**CN:** 该代码块定义了 `ggml_dequantize_ascend()`，用于为后续计算恢复低精度数据。

### Lines 620-622: class GGUFLinearAscendMethod: definition
```python
class GGUFLinearAscendMethod(LinearMethodBase):
    """Linear method for GGUF on Ascend NPU."""
```
**EN:** This block declares `GGUFLinearAscendMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, process_weights_after_loading, _create_padded_weight_param.
**CN:** 该代码块声明 `GGUFLinearAscendMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, process_weights_after_loading, _create_padded_weight_param 等行为。

### Lines 623-624: GGUFLinearAscendMethod.__init__()
```python
    def __init__(self, quant_config: GGUFConfig):
        self.quant_config = quant_config
```
**EN:** This block defines `GGUFLinearAscendMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `GGUFLinearAscendMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 626-670: GGUFLinearAscendMethod.create_weights()
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        input_size_per_partition: int,
        output_partition_sizes: list[int],
        input_size: int,
        output_size: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        self.params_dtype = params_dtype
        output_size_per_partition = sum(output_partition_sizes)

        tensor_shape = (output_size_per_partition, input_size_per_partition)
        qweight = GGUFUninitializedParameter(requires_grad=False)
        set_weight_attrs(
            qweight,
            {
                "input_dim": 1,
                "output_dim": 0,
                "tensor_shape": tensor_shape,
                "is_gguf_weight": True,
                "data_container": [],
                "shard_id": [],
                "shard_id_map": {},
            },
        )
        set_weight_attrs(qweight, extra_weight_attrs)
        layer.register_parameter("qweight", qweight)

        qweight_type = Parameter(
            torch.empty(len(output_partition_sizes), dtype=torch.uint8),
            requires_grad=False,
        )
        set_weight_attrs(
            qweight_type,
            {
                "is_gguf_weight_type": True,
                "weight_type": 0,
                "shard_weight_type": {},
                "ignore_warning": True,
            },
        )
        set_weight_attrs(qweight_type, extra_weight_attrs)
        layer.register_parameter("qweight_type", qweight_type)
```
**EN:** This block defines `GGUFLinearAscendMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `GGUFLinearAscendMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 672-680: GGUFLinearAscendMethod.process_weights_after_loading()
```python
    def process_weights_after_loading(self, layer: torch.nn.Module):
        qweight_type = layer.qweight_type.weight_type
        if not (qweight_type in UNQUANTIZED_TYPES or qweight_type in DEQUANT_TYPES):
            raise ValueError(
                f"Unsupported GGUF quantization type {WeightType(qweight_type)} in layer."
            )
        self._create_padded_weight_param(layer)
        # Pre-dequantize weights for faster inference
        self._pre_dequantize_weights(layer)
```
**EN:** This block defines `GGUFLinearAscendMethod.process_weights_after_loading()`, which post-processes previously loaded weights or metadata.
**CN:** 该代码块定义了 `GGUFLinearAscendMethod.process_weights_after_loading()`，用于对已加载的权重或元数据做后处理。

### Lines 682-708: GGUFLinearAscendMethod._create_padded_weight_param()
```python
    def _create_padded_weight_param(self, layer: torch.nn.Module):
        """Create padded weight parameter for GGUF MergedLinear layer."""
        qweight = layer.qweight
        shard_id_map = qweight.shard_id_map
        shard_id = qweight.shard_id
        if len(data_container := qweight.data_container) > 1:
            dtype = {data.dtype for data in data_container}
            assert len(dtype) == 1
            dtype = next(iter(dtype))
            padded_side = max(x.size(1) for x in data_container)
            concat_side = sum(x.size(0) for x in data_container)
            padded_data = torch.zeros(
                (concat_side, padded_side), dtype=dtype, device=qweight.device
            )
            shard_offset_map = dict[str, tuple[int, int, int]]()
            for idx in shard_id:
                id_in_container = shard_id_map[idx]
                start = sum(x.size(0) for x in data_container[:id_in_container])
                end = start + data_container[id_in_container].size(0)
                size = data_container[id_in_container].size(1)
                padded_data[start:end, :size] = data_container[id_in_container]
                shard_offset_map[idx] = (start, end, size)
            qweight.data_container.clear()
            padded_param = Parameter(padded_data, requires_grad=False)
            set_weight_attrs(padded_param, vars(qweight))
            set_weight_attrs(padded_param, {"shard_offset_map": shard_offset_map})
            layer.register_parameter("qweight", padded_param)
```
**EN:** This block defines `GGUFLinearAscendMethod._create_padded_weight_param()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `GGUFLinearAscendMethod._create_padded_weight_param()`，用于实现量化栈中的可复用模块逻辑。

### Lines 710-762: GGUFLinearAscendMethod._pre_dequantize_weights()
```python
    def _pre_dequantize_weights(self, layer: torch.nn.Module):
        """Pre-dequantize GGML weights to FP16 for faster inference.

        This eliminates runtime dequantization overhead at the cost of more memory.
        """
        qweight = layer.qweight
        qweight_type = layer.qweight_type.weight_type

        if qweight_type in UNQUANTIZED_TYPES and qweight.dtype in (
            torch.float16,
            torch.bfloat16,
            torch.float32,
        ):
            layer.dequantized_weight = qweight
            return

        shard_id = getattr(qweight, "shard_id", None)
        has_shard_offset = hasattr(qweight, "shard_offset_map")

        if shard_id and has_shard_offset:
            # Handle sharded weights (QKV merged)
            shard_id = ["q", "k", "v"] if "q" in shard_id else shard_id
            dequant_shards = []
            for idx in shard_id:
                start, end, offset = qweight.shard_offset_map[idx]
                shard_qtype = layer.qweight_type.shard_weight_type[idx]
                shard_data = qweight[start:end, :offset].contiguous()

                block_size, type_size = gguf.GGML_QUANT_SIZES[shard_qtype]
                shape = (
                    shard_data.shape[0],
                    shard_data.shape[1] // type_size * block_size,
                )
                dequant = ggml_dequantize_ascend(
                    shard_data, shard_qtype, *shape, self.params_dtype
                )
                dequant_shards.append(dequant)

            dequant_weight = torch.cat(dequant_shards, dim=0)
        else:
            # Handle single weight
            block_size, type_size = gguf.GGML_QUANT_SIZES[qweight_type]
            shape = (qweight.shape[0], qweight.shape[1] // type_size * block_size)
            dequant_weight = ggml_dequantize_ascend(
                qweight, qweight_type, *shape, self.params_dtype
            )

        layer.dequantized_weight = dequant_weight

        if hasattr(layer, "qweight"):
            del layer.qweight
        if hasattr(layer, "qweight_type"):
            del layer.qweight_type
```
**EN:** This block defines `GGUFLinearAscendMethod._pre_dequantize_weights()`, which restores low-precision data for downstream compute.
**CN:** 该代码块定义了 `GGUFLinearAscendMethod._pre_dequantize_weights()`，用于为后续计算恢复低精度数据。

### Lines 764-775: GGUFLinearAscendMethod.apply()
```python
    def apply(
        self,
        layer: torch.nn.Module,
        x: torch.Tensor,
        bias: torch.Tensor | None = None,
    ) -> torch.Tensor:
        # Use pre-dequantized weight (always available after process_weights_after_loading)
        weight = layer.dequantized_weight
        out = x @ weight.T
        if bias is not None:
            out.add_(bias)
        return out
```
**EN:** This block defines `GGUFLinearAscendMethod.apply()`, which applies quantized computation to runtime inputs.
**CN:** 该代码块定义了 `GGUFLinearAscendMethod.apply()`，用于将量化计算应用到运行时输入上。

### Lines 778-780: class GGUFMoEAscendMethod: definition
```python
class GGUFMoEAscendMethod(FusedMoEMethodBase):
    """MoE method for GGUF on Ascend NPU."""
```
**EN:** This block declares `GGUFMoEAscendMethod`, a runtime method class for the quantization stack. It organizes behaviors such as __init__, create_weights, process_weights_after_loading, create_moe_runner.
**CN:** 该代码块声明 `GGUFMoEAscendMethod`，它是量化栈中的运行方法类，组织了 __init__, create_weights, process_weights_after_loading, create_moe_runner 等行为。

### Lines 781-782: GGUFMoEAscendMethod.__init__()
```python
    def __init__(self, quant_config: GGUFConfig):
        self.quant_config = quant_config
```
**EN:** This block defines `GGUFMoEAscendMethod.__init__()`, which initializes object state and validates constructor inputs.
**CN:** 该代码块定义了 `GGUFMoEAscendMethod.__init__()`，用于初始化对象状态并校验构造参数。

### Lines 784-844: GGUFMoEAscendMethod.create_weights()
```python
    def create_weights(
        self,
        layer: torch.nn.Module,
        num_experts: int,
        hidden_size: int,
        intermediate_size_per_partition: int,
        params_dtype: torch.dtype,
        **extra_weight_attrs,
    ):
        tensor_shape = (num_experts, 2 * intermediate_size_per_partition, hidden_size)
        w13_qweight = GGUFUninitializedParameter(requires_grad=False)
        set_weight_attrs(
            w13_qweight,
            {
                "input_dim": 1,
                "output_dim": 0,
                "tensor_shape": tensor_shape,
                "is_gguf_weight": True,
                "data_container": [],
            },
        )
        set_weight_attrs(w13_qweight, extra_weight_attrs)
        layer.register_parameter("w13_qweight", w13_qweight)

        w13_qweight_type = Parameter(
            torch.empty(1, dtype=torch.uint8), requires_grad=False
        )
        set_weight_attrs(
            w13_qweight_type,
            {"is_gguf_weight_type": True, "weight_type": 0, "ignore_warning": True},
        )
        set_weight_attrs(w13_qweight_type, extra_weight_attrs)
        layer.register_parameter("w13_qweight_type", w13_qweight_type)

        tensor_shape = (num_experts, intermediate_size_per_partition, hidden_size)
        w2_qweight = GGUFUninitializedParameter(requires_grad=False)
        set_weight_attrs(
            w2_qweight,
            {
                "input_dim": 1,
                "output_dim": 0,
                "tensor_shape": tensor_shape,
                "is_gguf_weight": True,
                "data_container": [],
            },
        )
        set_weight_attrs(w2_qweight, extra_weight_attrs)
        layer.register_parameter("w2_qweight", w2_qweight)

        w2_qweight_type = Parameter(
            torch.empty(1, dtype=torch.uint8), requires_grad=False
        )
        set_weight_attrs(
            w2_qweight_type,
            {"is_gguf_weight_type": True, "weight_type": 0, "ignore_warning": True},
        )
        set_weight_attrs(w2_qweight_type, extra_weight_attrs)
        layer.register_parameter("w2_qweight_type", w2_qweight_type)

        # Store params_dtype for pre-dequantization
        self.params_dtype = params_dtype
```
**EN:** This block defines `GGUFMoEAscendMethod.create_weights()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `GGUFMoEAscendMethod.create_weights()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 846-890: GGUFMoEAscendMethod.process_weights_after_loading() (part 1/2)
```python
    def process_weights_after_loading(self, layer: torch.nn.Module):
        """Pre-dequantize MoE weights to FP16 for faster inference."""

        if hasattr(layer, "materialize_gguf_weights"):
            layer.materialize_gguf_weights()

        # Check if weights are actually loaded (not still UninitializedParameter/empty)
        w13_qweight = layer.w13_qweight
        w13_qtype = layer.w13_qweight_type.weight_type

        # Pre-dequantize w13 weights (gate+up projections)
        if w13_qtype not in UNQUANTIZED_TYPES:
            num_experts = w13_qweight.shape[0]
            w13_dequant_list = []

            block_size, type_size = gguf.GGML_QUANT_SIZES[w13_qtype]

            for e in range(num_experts):
                qweight_cpu = w13_qweight[e].cpu().numpy()
                rows = w13_qweight[e].shape[0]
                cols = w13_qweight[e].shape[1] // type_size * block_size

                dequant_np = gguf_dequantize(qweight_cpu.flatten(), w13_qtype)
                dequant = (
                    torch.from_numpy(dequant_np)
                    .to(dtype=self.params_dtype, device=w13_qweight.device)
                    .reshape(rows, cols)
                    .transpose(-1, -2)
                    .contiguous()
                )
                w13_dequant_list.append(dequant)

            w13_full = torch.stack(w13_dequant_list, dim=0)

            layer.register_buffer("w13_dequant", w13_full, persistent=False)
        else:
            layer.register_buffer("w13_dequant", w13_qweight.data, persistent=False)

        # Pre-dequantize w2 weights (down projection)
        w2_qweight = layer.w2_qweight
        w2_qtype = layer.w2_qweight_type.weight_type

        if w2_qtype not in UNQUANTIZED_TYPES:
            num_experts = w2_qweight.shape[0]
            w2_dequant_list = []
```
**EN:** This segment of `GGUFMoEAscendMethod.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `GGUFMoEAscendMethod.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 891-918: GGUFMoEAscendMethod.process_weights_after_loading() (part 2/2)
```python

            block_size, type_size = gguf.GGML_QUANT_SIZES[w2_qtype]

            for e in range(num_experts):
                qweight_cpu = w2_qweight[e].cpu().numpy()
                rows = w2_qweight[e].shape[0]
                cols = w2_qweight[e].shape[1] // type_size * block_size

                dequant_np = gguf_dequantize(qweight_cpu.flatten(), w2_qtype)
                dequant = (
                    torch.from_numpy(dequant_np)
                    .to(dtype=self.params_dtype, device=w2_qweight.device)
                    .reshape(rows, cols)
                    .transpose(-1, -2)
                    .contiguous()
                )
                w2_dequant_list.append(dequant)

            w2_full = torch.stack(w2_dequant_list, dim=0)

            layer.register_buffer("w2_dequant", w2_full, persistent=False)
        else:
            layer.register_buffer("w2_dequant", w2_qweight.data, persistent=False)

        if hasattr(layer, "w2_qweight"):
            del layer.w2_qweight
        if hasattr(layer, "w13_qweight"):
            del layer.w13_qweight
```
**EN:** This segment of `GGUFMoEAscendMethod.process_weights_after_loading()` post-processes previously loaded weights or metadata. It covers one portion of the full implementation.
**CN:** `GGUFMoEAscendMethod.process_weights_after_loading()` 的这一段代码用于对已加载的权重或元数据做后处理，展示了完整实现中的一部分。

### Lines 920-923: GGUFMoEAscendMethod.create_moe_runner()
```python
    def create_moe_runner(
        self, layer: torch.nn.Module, moe_runner_config: MoeRunnerConfig
    ):
        self.moe_runner_config = moe_runner_config
```
**EN:** This block defines `GGUFMoEAscendMethod.create_moe_runner()`, which creates parameters, buffers, or helper objects on demand.
**CN:** 该代码块定义了 `GGUFMoEAscendMethod.create_moe_runner()`，用于按需创建参数、缓冲区或辅助对象。

### Lines 925-969: GGUFMoEAscendMethod.apply() (part 1/3)
```python
    def apply(
        self,
        layer: torch.nn.Module,
        dispatch_output: StandardDispatchOutput,
    ) -> CombineInput:
        """Apply MoE forward pass on NPU using npu_grouped_matmul for maximum performance."""
        from sglang.srt.distributed.communication_op import (
            tensor_model_parallel_all_gather,
        )
        from sglang.srt.layers.moe.token_dispatcher import StandardCombineInput

        x = dispatch_output.hidden_states
        topk_output = dispatch_output.topk_output
        topk_weights, topk_ids, _ = topk_output

        # Check if pre-dequantized weights are available
        use_pre_dequant = hasattr(layer, "w13_dequant") and hasattr(layer, "w2_dequant")

        if not use_pre_dequant:
            raise RuntimeError(
                "GGUF MoE on NPU requires pre-dequantization (FusedMoE fix). Please report if this occurs."
            )

        w13 = layer.w13_dequant
        w2 = layer.w2_dequant

        num_experts = w13.shape[0]

        tp_size = getattr(layer, "moe_tp_size", 1)

        original_dtype = x.dtype
        num_tokens = x.shape[0]
        top_k = topk_ids.shape[1]

        # Ensure correct dtypes for NPU ops
        topk_ids = topk_ids.to(torch.int32)
        topk_weights = topk_weights.to(x.dtype)

        #  MoE routing initialization - reorder tokens by expert
        row_idx_len = num_tokens * top_k
        row_idx = (
            torch.arange(0, row_idx_len, dtype=torch.int32, device=x.device)
            .view(top_k, -1)
            .permute(1, 0)
            .contiguous()
```
**EN:** This segment of `GGUFMoEAscendMethod.apply()` applies quantized computation to runtime inputs. It covers one portion of the full implementation.
**CN:** `GGUFMoEAscendMethod.apply()` 的这一段代码用于将量化计算应用到运行时输入上，展示了完整实现中的一部分。

### Lines 970-1014: GGUFMoEAscendMethod.apply() (part 2/3)
```python
        )

        sorted_hidden_states, expanded_row_idx, expanded_expert_idx = (
            torch.ops.npu.npu_moe_init_routing(
                x, row_idx=row_idx, expert_idx=topk_ids, active_num=num_tokens
            )
        )

        # Compute tokens per expert
        expert_tokens = torch.ops.npu.npu_moe_compute_expert_tokens(
            expanded_expert_idx, num_experts
        )
        expert_tokens = expert_tokens.to(torch.int64)

        w13_gmm = w13  # No transpose needed

        hidden_states = torch.ops.npu.npu_grouped_matmul(
            x=[sorted_hidden_states],
            weight=[w13_gmm],
            split_item=2,
            group_list_type=0,
            group_type=0,
            group_list=expert_tokens,
            output_dtype=original_dtype,
        )[0]

        #  Activation (SwiGLU)
        hidden_states = torch.ops.npu.npu_swiglu(hidden_states)

        # TP all-gather for intermediate dimension if needed
        if tp_size > 1:
            hidden_states = tensor_model_parallel_all_gather(hidden_states, dim=-1)

        w2_gmm = w2

        hidden_states = torch.ops.npu.npu_grouped_matmul(
            x=[hidden_states],
            weight=[w2_gmm],
            split_item=2,
            group_list_type=0,
            group_type=0,
            group_list=expert_tokens,
            output_dtype=original_dtype,
        )[0]
```
**EN:** This segment of `GGUFMoEAscendMethod.apply()` applies quantized computation to runtime inputs. It covers one portion of the full implementation.
**CN:** `GGUFMoEAscendMethod.apply()` 的这一段代码用于将量化计算应用到运行时输入上，展示了完整实现中的一部分。

### Lines 1015-1034: GGUFMoEAscendMethod.apply() (part 3/3)
```python
        # Finalize routing - reorder back and apply weights
        final_hidden_states = torch.ops.npu.npu_moe_finalize_routing(
            hidden_states,
            skip1=None,
            skip2=None,
            bias=None,
            scales=topk_weights,
            expanded_src_to_dst_row=expanded_row_idx,
            export_for_source_row=topk_ids,
        )

        if tp_size > 1:
            final_hidden_states = tensor_model_parallel_all_gather(
                final_hidden_states, dim=-1
            )

        # Ensure output matches input dtype
        final_hidden_states = final_hidden_states.to(dtype=original_dtype)

        return StandardCombineInput(hidden_states=final_hidden_states)
```
**EN:** This segment of `GGUFMoEAscendMethod.apply()` applies quantized computation to runtime inputs. It covers one portion of the full implementation.
**CN:** `GGUFMoEAscendMethod.apply()` 的这一段代码用于将量化计算应用到运行时输入上，展示了完整实现中的一部分。

### Lines 1037-1039: class GGUFEmbeddingAscendMethod: definition
```python
class GGUFEmbeddingAscendMethod(GGUFLinearAscendMethod):
    """Embedding method for GGUF on Ascend NPU."""
```
**EN:** This block declares `GGUFEmbeddingAscendMethod`, a runtime method class for the quantization stack. It organizes behaviors such as embedding.
**CN:** 该代码块声明 `GGUFEmbeddingAscendMethod`，它是量化栈中的运行方法类，组织了 embedding 等行为。

### Lines 1040-1041: GGUFEmbeddingAscendMethod.embedding()
```python
    def embedding(self, layer: torch.nn.Module, x: torch.Tensor) -> torch.Tensor:
        return torch.embedding(layer.dequantized_weight, x)
```
**EN:** This block defines `GGUFEmbeddingAscendMethod.embedding()`, which implements reusable module logic for the quantization stack.
**CN:** 该代码块定义了 `GGUFEmbeddingAscendMethod.embedding()`，用于实现量化栈中的可复用模块逻辑。

## Key Concepts / 关键概念
- `GGUFConfig`: A configuration class that structures file-level quantization behavior. / `GGUFConfig` 是一个配置类，用于组织该文件中的量化行为。
- `is_layer_skipped_gguf()` : A public function that checks a condition used by higher-level control flow. / `is_layer_skipped_gguf()`：一个公开函数，用于检查供上层控制流使用的条件。
- `UNQUANTIZED_TYPES`: A module-level constant or registry. / `UNQUANTIZED_TYPES`：模块级常量或注册表。
- `STANDARD_QUANT_TYPES`: A module-level constant or registry. / `STANDARD_QUANT_TYPES`：模块级常量或注册表。
- `KQUANT_TYPES`: A module-level constant or registry. / `KQUANT_TYPES`：模块级常量或注册表。
- `IMATRIX_QUANT_TYPES`: A module-level constant or registry. / `IMATRIX_QUANT_TYPES`：模块级常量或注册表。
- `DEQUANT_TYPES`: A module-level constant or registry. / `DEQUANT_TYPES`：模块级常量或注册表。
- `MMVQ_QUANT_TYPES`: A module-level constant or registry. / `MMVQ_QUANT_TYPES`：模块级常量或注册表。
- `MMQ_QUANT_TYPES`: A module-level constant or registry. / `MMQ_QUANT_TYPES`：模块级常量或注册表。
- `fused_mul_mat_gguf()` : A public function that implements reusable module logic for the quantization stack. / `fused_mul_mat_gguf()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `fused_moe_gguf()` : A public function that implements reusable module logic for the quantization stack. / `fused_moe_gguf()`：一个公开函数，用于实现量化栈中的可复用模块逻辑。
- `apply_gguf_embedding()` : A public function that applies quantized computation to runtime inputs. / `apply_gguf_embedding()`：一个公开函数，用于将量化计算应用到运行时输入上。
- `GGUFLinearMethod`: A runtime method class that structures file-level quantization behavior. / `GGUFLinearMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `GGUFMoEMethod`: A runtime method class that structures file-level quantization behavior. / `GGUFMoEMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `GGUFEmbeddingMethod`: A runtime method class that structures file-level quantization behavior. / `GGUFEmbeddingMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `GGUFUninitializedParameter`: A supporting class that structures file-level quantization behavior. / `GGUFUninitializedParameter` 是一个支撑类，用于组织该文件中的量化行为。
- `ggml_dequantize_ascend()` : A public function that restores low-precision data for downstream compute. / `ggml_dequantize_ascend()`：一个公开函数，用于为后续计算恢复低精度数据。
- `GGUFLinearAscendMethod`: A runtime method class that structures file-level quantization behavior. / `GGUFLinearAscendMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `GGUFMoEAscendMethod`: A runtime method class that structures file-level quantization behavior. / `GGUFMoEAscendMethod` 是一个运行方法类，用于组织该文件中的量化行为。
- `GGUFEmbeddingAscendMethod`: A runtime method class that structures file-level quantization behavior. / `GGUFEmbeddingAscendMethod` 是一个运行方法类，用于组织该文件中的量化行为。

## Dependencies / 依赖关系
- **External / 外部**: `__future__`, `gguf`, `logging`, `sgl_kernel`, `sgl_kernel.quantization`, `torch`, `torch.nn.parameter`, `typing`, `warnings`
- **Internal / 内部**: `sglang.jit_kernel.activation`, `sglang.srt.distributed.communication_op`, `sglang.srt.layers.linear`, `sglang.srt.layers.moe`, `sglang.srt.layers.moe.fused_moe_triton`, `sglang.srt.layers.moe.token_dispatcher`, `sglang.srt.layers.quantization.base_config`, `sglang.srt.layers.quantization.unquant`, `sglang.srt.layers.vocab_parallel_embedding`, `sglang.srt.utils`
