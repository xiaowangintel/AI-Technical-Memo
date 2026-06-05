# radix_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/radix_attention.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements attention-related tensor transforms for the SGLang SRT runtime. It exposes symbols such as `AttentionType`, `RadixAttention`, and `unified_attention_with_output` and connects them to backend-specific paths such as `CUDA` and `CPU`. / 该模块为 SGLang 的 SRT 运行时实现了注意力相关张量变换。它提供了 `AttentionType`、`RadixAttention` 以及 `unified_attention_with_output` 等符号，并把这些符号连接到 `CUDA` 和 `CPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: File header and module overview
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
```
**EN:** This opening block contains comments, licensing text, or other context that frames the rest of the file before executable code begins.
**CN:** 这一开头部分包含注释、许可证文本或其他上下文信息，用来为后续可执行代码建立背景。

### Lines 14-38: Imports, conditional backend setup, and runtime guards
```python
"""Radix attention."""

from __future__ import annotations

from enum import Enum
from typing import TYPE_CHECKING, Optional

import torch
from torch import nn

from sglang.srt.compilation.compilation_config import register_split_op
from sglang.srt.compilation.piecewise_context_manager import get_forward_context
from sglang.srt.model_executor.breakable_cuda_graph.breakable_cuda_graph import (
    eager_on_graph,
)
from sglang.srt.model_executor.breakable_cuda_graph.context import (
    is_in_breakable_cuda_graph,
)
from sglang.srt.utils.custom_op import register_custom_op

if TYPE_CHECKING:
    from sglang.srt.layers.quantization.base_config import QuantizationConfig
    from sglang.srt.model_executor.forward_batch_info import ForwardBatch
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `enum.Enum`, `typing.TYPE_CHECKING`, `typing.Optional`, `torch`, and `torch.nn`, so later blocks can reuse runtime, tensor, or backend helpers. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`enum.Enum`、`typing.TYPE_CHECKING`、`typing.Optional`、`torch` 以及 `torch.nn`，让后续代码可以复用运行时、张量或后端辅助逻辑。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 39-52: Class `AttentionType` declaration and shared state
```python
class AttentionType(Enum):
    """
    Attention type.
    Use string to be compatible with `torch.compile`.
    """

    # Decoder attention between previous layer Q/K/V
    DECODER = "decoder"
    # Decoder bidirectional attention between image tokens
    DECODER_BIDIRECTIONAL = "decoder_bidirectional"
    # Encoder attention between previous layer Q/K/V
    ENCODER_ONLY = "encoder_only"
```
**EN:** This block introduces class `AttentionType` and the state shared by its methods. It inherits from `Enum`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: Attention type.
**CN:** 该代码块引入类 `AttentionType`，并定义其方法共享的状态。 它继承自 `Enum`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 53-57: Class `RadixAttention` declaration and shared state
```python
class RadixAttention(nn.Module):
    """
    The attention layer implementation.
    """
```
**EN:** This block introduces class `RadixAttention` and the state shared by its methods. It inherits from `nn.Module`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: The attention layer implementation.
**CN:** 该代码块引入类 `RadixAttention`，并定义其方法共享的状态。 它继承自 `nn.Module`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 58-104: `RadixAttention` initialization and state setup
```python
    def __init__(
        self,
        num_heads: int,
        head_dim: int,
        scaling: float,
        num_kv_heads: int,
        layer_id: int,
        logit_cap: float = 0.0,
        v_head_dim: int = -1,
        sliding_window_size: int = -1,
        is_cross_attention: bool = False,
        pos_encoding_mode: str = "NONE",
        logit_capping_method: str = "tanh",
        quant_config: Optional[QuantizationConfig] = None,
        attn_type: AttentionType = AttentionType.DECODER,
        use_irope: bool = False,
        prefix: str = "",
    ):
        super().__init__()
        self.tp_q_head_num = num_heads
        self.tp_k_head_num = num_kv_heads
        self.tp_v_head_num = num_kv_heads
        self.head_dim = head_dim
        self.qk_head_dim = head_dim
        self.v_head_dim = v_head_dim if v_head_dim != -1 else head_dim
        self.scaling = scaling
        self.layer_id = layer_id
        self.logit_cap = logit_cap
        self.sliding_window_size = sliding_window_size or -1
        self.is_cross_attention = is_cross_attention
        self.use_irope = use_irope
        self.k_scale = None
        self.v_scale = None
        self.k_scale_float = None
        self.v_scale_float = None
        self.quant_method = None

        if quant_config is not None:
            self.quant_method = quant_config.get_quant_method(self, prefix=prefix)
        if self.quant_method is not None:
            self.quant_method.create_weights(self)
        self.attn_type = attn_type

        self.pos_encoding_mode = pos_encoding_mode
        self.logit_capping_method = logit_capping_method
        self.xai_temperature_len = -1
```
**EN:** This block defines `RadixAttention.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, `quant_config.get_quant_method`, and `self.quant_method.create_weights`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.tp_q_head_num`, `self.tp_k_head_num`, `self.tp_v_head_num`, `self.head_dim`, and `self.qk_head_dim` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `RadixAttention.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`、`quant_config.get_quant_method` 以及 `self.quant_method.create_weights`，说明该流程会编排底层辅助函数或计算内核。 像 `self.tp_q_head_num`、`self.tp_k_head_num`、`self.tp_v_head_num`、`self.head_dim` 以及 `self.qk_head_dim` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 105-148: `RadixAttention.forward` main forward path
```python
    def forward(
        self,
        q,
        k,
        v,
        forward_batch: ForwardBatch,
        save_kv_cache: bool = True,
        **kwargs,
    ):
        if k is not None:
            # For cross-layer sharing, kv can be None
            assert v is not None
            if "k_rope" not in kwargs:
                k = k.view(-1, self.tp_k_head_num, self.qk_head_dim)
                v = v.view(-1, self.tp_v_head_num, self.v_head_dim)
            else:
                k = k.view(-1, self.tp_k_head_num, self.v_head_dim)

        if forward_batch.forward_mode.is_extend() and get_forward_context() is not None:
            if self.qk_head_dim != self.v_head_dim:
                output = q.new_empty((q.shape[0], self.tp_q_head_num * self.v_head_dim))
            else:
                output = torch.empty_like(q)
            if is_in_breakable_cuda_graph():
                bcg_unified_attention_with_output(
                    q, k, v, output, save_kv_cache, self.layer_id, **kwargs
                )
            else:
                unified_attention_with_output(
                    q, k, v, output, save_kv_cache, self.layer_id, **kwargs
                )
            return output
        else:
            return forward_batch.attn_backend.forward(
                q,
                k,
                v,
                self,
                forward_batch,
                save_kv_cache,
                **kwargs,
            )
```
**EN:** This block defines `RadixAttention.forward` and contains the main logic for this step. It mainly invokes `forward_batch.forward_mode.is_extend`, `is_in_breakable_cuda_graph`, `forward_batch.attn_backend.forward`, `k.view`, and `v.view`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `k`, `v`, and `output` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RadixAttention.forward`，并承载这一阶段的核心逻辑。 它主要调用 `forward_batch.forward_mode.is_extend`、`is_in_breakable_cuda_graph`、`forward_batch.attn_backend.forward`、`k.view` 以及 `v.view`，说明该流程会编排底层辅助函数或计算内核。 像 `k`、`v` 以及 `output` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 149-218: Function `unified_attention_with_output` and its core logic
```python
@register_custom_op(mutates_args=["output"])
@register_split_op()
def unified_attention_with_output(
    query: torch.Tensor,
    key: torch.Tensor,
    value: torch.Tensor,
    output: torch.Tensor,
    save_kv_cache: bool,
    layer_id: int,
    *,
    q_rope: Optional[torch.Tensor] = None,
    k_rope: Optional[torch.Tensor] = None,
    sinks: Optional[torch.Tensor] = None,
) -> None:
    context = get_forward_context()
    forward_batch = context.forward_batch
    attention_layers = context.attention_layers
    attention_layer = attention_layers[layer_id]
    real_num_tokens = forward_batch.num_token_non_padded_cpu

    query = query[:real_num_tokens]
    key = key[:real_num_tokens]
    value = value[:real_num_tokens]

    kwargs = {}
    if q_rope is not None:
        kwargs["q_rope"] = q_rope[:real_num_tokens]
    if k_rope is not None:
        kwargs["k_rope"] = k_rope[:real_num_tokens]
    if sinks is not None:
        kwargs["sinks"] = sinks

    original_out_cache_loc = forward_batch.out_cache_loc
    original_out_cache_loc_swa = forward_batch.out_cache_loc_swa
    token_to_kv_pool = forward_batch.token_to_kv_pool
    original_swa_loc = getattr(token_to_kv_pool, "swa_loc", None)
    # Keep the original ForwardBatch object and only narrow cache locations for
    # this backend call so model/backend state is still written to the same batch.
    forward_batch.out_cache_loc = original_out_cache_loc[:real_num_tokens]
    if original_out_cache_loc_swa is not None:
        forward_batch.out_cache_loc_swa = original_out_cache_loc_swa[:real_num_tokens]
        if hasattr(token_to_kv_pool, "set_swa_loc"):
            token_to_kv_pool.set_swa_loc(forward_batch.out_cache_loc_swa)

    # Store pre-allocated output for FA backend to write directly into.
    # Must slice to real_num_tokens to match the narrowed query shape —
    # the FA kernel validates out.size(0) == q.size(0).
    forward_batch._attn_output = output[:real_num_tokens]

    ret = forward_batch.attn_backend.forward(
        query,
        key,
        value,
        attention_layer,
        forward_batch,
        save_kv_cache,
        **kwargs,
    )
    forward_batch.out_cache_loc = original_out_cache_loc
    forward_batch.out_cache_loc_swa = original_out_cache_loc_swa
    if original_out_cache_loc_swa is not None and hasattr(
        token_to_kv_pool, "set_swa_loc"
    ):
        token_to_kv_pool.set_swa_loc(original_swa_loc)

    if ret.data_ptr() != output.data_ptr():
        output[:real_num_tokens].view(ret.shape).copy_(ret)
    return
```
**EN:** This block defines `unified_attention_with_output` and contains the main logic for this step. Decorators like `register_custom_op` and `register_split_op` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_custom_op`, `register_split_op`, `get_forward_context`, `getattr`, and `forward_batch.attn_backend.forward`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `context`, `forward_batch`, `attention_layers`, `attention_layer`, and `real_num_tokens` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `unified_attention_with_output`，并承载这一阶段的核心逻辑。 像 `register_custom_op` 和 `register_split_op` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_custom_op`、`register_split_op`、`get_forward_context`、`getattr` 以及 `forward_batch.attn_backend.forward`，说明该流程会编排底层辅助函数或计算内核。 像 `context`、`forward_batch`、`attention_layers`、`attention_layer` 以及 `real_num_tokens` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 219-219: Module constants and shared configuration
```python
bcg_unified_attention_with_output = eager_on_graph(True)(unified_attention_with_output)
```
**EN:** This section prepares the module namespace. Shared names such as `bcg_unified_attention_with_output` capture configuration, cached handles, or feature flags.
**CN:** 该部分负责准备模块命名空间。 像 `bcg_unified_attention_with_output` 这样的共享名称用于保存配置、缓存句柄或特性开关。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `AttentionType`, `RadixAttention`, and `unified_attention_with_output`. / **主要符号**：核心入口包括 `AttentionType`、`RadixAttention` 以及 `unified_attention_with_output`。
- **Low-precision execution**: Highlights how the module handles quantized weights, activations, or scaling factors. / **低精度执行**：强调模块如何处理量化权重、激活值或缩放因子。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Custom kernel integration**: Connects Python control flow to Triton, CUTLASS, FlashInfer, or other accelerated kernels. / **自定义内核集成**：把 Python 控制流连接到 Triton、CUTLASS、FlashInfer 等加速内核。
- **Position encoding**: Describes how rotary embeddings or related position transforms are applied. / **位置编码**：说明如何应用旋转位置编码或相关位置变换。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `enum.Enum`, `typing.TYPE_CHECKING`, and `typing.Optional` / **标准库**：`__future__.annotations`、`enum.Enum`、`typing.TYPE_CHECKING` 以及 `typing.Optional`
- **Third-party**: `torch` and `torch.nn` / **第三方依赖**：`torch` 和 `torch.nn`
- **Internal SGLang modules**: `sglang.srt.compilation.compilation_config.register_split_op`, `sglang.srt.compilation.piecewise_context_manager.get_forward_context`, `sglang.srt.model_executor.breakable_cuda_graph.breakable_cuda_graph.eager_on_graph`, `sglang.srt.model_executor.breakable_cuda_graph.context.is_in_breakable_cuda_graph`, `sglang.srt.utils.custom_op.register_custom_op`, `sglang.srt.layers.quantization.base_config.QuantizationConfig`, and `sglang.srt.model_executor.forward_batch_info.ForwardBatch` / **SGLang 内部模块**：`sglang.srt.compilation.compilation_config.register_split_op`、`sglang.srt.compilation.piecewise_context_manager.get_forward_context`、`sglang.srt.model_executor.breakable_cuda_graph.breakable_cuda_graph.eager_on_graph`、`sglang.srt.model_executor.breakable_cuda_graph.context.is_in_breakable_cuda_graph`、`sglang.srt.utils.custom_op.register_custom_op`、`sglang.srt.layers.quantization.base_config.QuantizationConfig` 以及 `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
