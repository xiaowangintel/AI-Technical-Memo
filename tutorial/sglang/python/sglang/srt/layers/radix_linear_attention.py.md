# radix_linear_attention.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/layers/radix_linear_attention.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module implements linear projections and GEMM backend integration with attention-related tensor transforms for the SGLang SRT runtime. It exposes symbols such as `RadixLinearAttention` and `unified_linear_attention_with_output` and connects them to backend-specific paths such as `CPU`. / 该模块为 SGLang 的 SRT 运行时实现了线性投影与 GEMM 后端集成，并结合注意力相关张量变换。它提供了 `RadixLinearAttention` 和 `unified_linear_attention_with_output` 等符号，并把这些符号连接到 `CPU` 等后端专用路径。

## Line-by-Line Analysis / 逐行分析
### Lines 1-13: File header and module overview
```python
# Copyright 2025-2026 SGLang Team
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

### Lines 14-30: Imports, conditional backend setup, and runtime guards
```python
"""Radix linear attention."""

from __future__ import annotations

from typing import TYPE_CHECKING, Optional, Tuple, Union

import torch
from torch import nn

from sglang.srt.compilation.compilation_config import register_split_op
from sglang.srt.compilation.piecewise_context_manager import get_forward_context
from sglang.srt.utils.custom_op import register_custom_op

if TYPE_CHECKING:
    from sglang.srt.model_executor.forward_batch_info import ForwardBatch
```
**EN:** This section prepares the module namespace. It imports `__future__.annotations`, `typing.TYPE_CHECKING`, `typing.Optional`, `typing.Tuple`, `typing.Union`, and `torch`, so later blocks can reuse runtime, tensor, or backend helpers. Control structures like `If` indicate compatibility branches or backend-specific setup.
**CN:** 该部分负责准备模块命名空间。 它导入了 `__future__.annotations`、`typing.TYPE_CHECKING`、`typing.Optional`、`typing.Tuple`、`typing.Union` 以及 `torch`，让后续代码可以复用运行时、张量或后端辅助逻辑。 `If` 等控制结构说明这里还承担兼容分支或后端专用初始化。

### Lines 31-35: Class `RadixLinearAttention` declaration and shared state
```python
class RadixLinearAttention(nn.Module):
    """
    The Linear Attention Layer Implementation.
    """
```
**EN:** This block introduces class `RadixLinearAttention` and the state shared by its methods. It inherits from `nn.Module`, which shows how it plugs into the SRT layer stack. The class docstring summarizes its role: The Linear Attention Layer Implementation.
**CN:** 该代码块引入类 `RadixLinearAttention`，并定义其方法共享的状态。 它继承自 `nn.Module`，说明了它在 SRT 层栈中的接入方式。 类级文档进一步概括了它的职责。

### Lines 36-70: `RadixLinearAttention` initialization and state setup
```python
    def __init__(
        self,
        layer_id: int,
        num_q_heads: int,
        num_k_heads: int,
        num_v_heads: int,
        head_q_dim: int,
        head_k_dim: int,
        head_v_dim: int,
        # GDN KDA Shared Weights
        conv_weights: Optional[Union[torch.Tensor, Tuple[torch.Tensor, ...]]] = None,
        bias: Optional[Union[torch.Tensor, Tuple[torch.Tensor, ...]]] = None,
        activation: str = "silu",
        A_log: Optional[torch.Tensor] = None,
        dt_bias: Optional[torch.Tensor] = None,
    ):
        super().__init__()
        self.layer_id = layer_id
        self.num_q_heads = num_q_heads
        self.num_k_heads = num_k_heads
        self.num_v_heads = num_v_heads
        self.head_q_dim = head_q_dim
        self.head_k_dim = head_k_dim
        self.head_v_dim = head_v_dim
        self.q_dim = num_q_heads * head_q_dim
        self.k_dim = num_k_heads * head_k_dim
        self.v_dim = num_v_heads * head_v_dim

        self.conv_weights = conv_weights
        self.bias = bias
        self.activation = activation

        self.A_log = A_log
        self.dt_bias = dt_bias
```
**EN:** This block defines `RadixLinearAttention.__init__` and contains the main logic for this step. It mainly invokes `super.__init__`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `self.layer_id`, `self.num_q_heads`, `self.num_k_heads`, `self.num_v_heads`, and `self.head_q_dim` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `RadixLinearAttention.__init__`，并承载这一阶段的核心逻辑。 它主要调用 `super.__init__`，说明该流程会编排底层辅助函数或计算内核。 像 `self.layer_id`、`self.num_q_heads`、`self.num_k_heads`、`self.num_v_heads` 以及 `self.head_q_dim` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

### Lines 71-103: `RadixLinearAttention.forward` main forward path
```python
    def forward(
        self,
        forward_batch: ForwardBatch,
        mixed_qkv: torch.Tensor,
        a: torch.Tensor,
        b: torch.Tensor,
    ) -> torch.Tensor:
        if forward_batch.forward_mode.is_extend() and get_forward_context() is not None:
            # Output shape from linear attention: (1, seq_len, num_v_heads, head_v_dim)
            seq_len = mixed_qkv.shape[0]
            output = torch.empty(
                (1, seq_len, self.num_v_heads, self.head_v_dim),
                dtype=mixed_qkv.dtype,
                device=mixed_qkv.device,
            )
            unified_linear_attention_with_output(
                mixed_qkv,
                a,
                b,
                output,
                self.layer_id,
            )
            return output
        else:
            return forward_batch.attn_backend.forward(
                layer=self,
                forward_batch=forward_batch,
                mixed_qkv=mixed_qkv,
                a=a,
                b=b,
            )
```
**EN:** This block defines `RadixLinearAttention.forward` and contains the main logic for this step. It mainly invokes `forward_batch.forward_mode.is_extend`, `torch.empty`, `unified_linear_attention_with_output`, `forward_batch.attn_backend.forward`, and `get_forward_context`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `seq_len` and `output` keep track of tensors, shapes, handles, or runtime state. The function returns the computed value back to its caller.
**CN:** 该代码块定义了 `RadixLinearAttention.forward`，并承载这一阶段的核心逻辑。 它主要调用 `forward_batch.forward_mode.is_extend`、`torch.empty`、`unified_linear_attention_with_output`、`forward_batch.attn_backend.forward` 以及 `get_forward_context`，说明该流程会编排底层辅助函数或计算内核。 像 `seq_len` 和 `output` 这样的中间名称用于记录张量、形状、句柄或运行时状态。 该函数最终会把计算结果返回给调用方。

### Lines 104-149: Function `unified_linear_attention_with_output` and its core logic
```python
@register_custom_op(mutates_args=["output"])
@register_split_op()
def unified_linear_attention_with_output(
    mixed_qkv: torch.Tensor,
    a: torch.Tensor,
    b: torch.Tensor,
    output: torch.Tensor,
    layer_id: int,
) -> None:
    """
    Custom op wrapper for linear attention computation only.
    """
    context = get_forward_context()
    forward_batch = context.forward_batch
    attention_layers = context.attention_layers
    attention_layer = attention_layers[layer_id]
    real_num_tokens = forward_batch.num_token_non_padded_cpu

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

    ret = forward_batch.attn_backend.forward(
        layer=attention_layer,
        forward_batch=forward_batch,
        mixed_qkv=mixed_qkv[:real_num_tokens],
        a=a[:real_num_tokens],
        b=b[:real_num_tokens],
    )
    forward_batch.out_cache_loc = original_out_cache_loc
    forward_batch.out_cache_loc_swa = original_out_cache_loc_swa
    if original_out_cache_loc_swa is not None and hasattr(
        token_to_kv_pool, "set_swa_loc"
    ):
        token_to_kv_pool.set_swa_loc(original_swa_loc)

    output[:, :real_num_tokens].copy_(ret)
    return
```
**EN:** This block defines `unified_linear_attention_with_output` and contains the main logic for this step. Decorators like `register_custom_op` and `register_split_op` connect it to registration, caching, tracing, or dispatch hooks. It mainly invokes `register_custom_op`, `register_split_op`, `get_forward_context`, `getattr`, and `forward_batch.attn_backend.forward`, showing how the routine orchestrates lower-level helpers or kernels. Intermediate names such as `context`, `forward_batch`, `attention_layers`, `attention_layer`, and `real_num_tokens` keep track of tensors, shapes, handles, or runtime state.
**CN:** 该代码块定义了 `unified_linear_attention_with_output`，并承载这一阶段的核心逻辑。 像 `register_custom_op` 和 `register_split_op` 这样的装饰器会把它接入注册、缓存、跟踪或分发钩子。 它主要调用 `register_custom_op`、`register_split_op`、`get_forward_context`、`getattr` 以及 `forward_batch.attn_backend.forward`，说明该流程会编排底层辅助函数或计算内核。 像 `context`、`forward_batch`、`attention_layers`、`attention_layer` 以及 `real_num_tokens` 这样的中间名称用于记录张量、形状、句柄或运行时状态。

## Key Concepts / 关键概念
- **Primary symbols**: Key entry points include `RadixLinearAttention` and `unified_linear_attention_with_output`. / **主要符号**：核心入口包括 `RadixLinearAttention` 和 `unified_linear_attention_with_output`。
- **Distributed coordination**: Shows how tensors or metadata move across parallel workers and devices. / **分布式协同**：展示张量或元数据如何在并行 worker 与设备之间流动。
- **Activation math**: Captures fused nonlinear transforms and their device-specific fast paths. / **激活数学**：描述融合非线性变换及其设备专用快速路径。
- **Projection layers**: Focuses on matrix multiplication, weight layout, and projection-oriented wrappers. / **投影层**：关注矩阵乘法、权重布局与面向投影的包装层。
- **Convolution support**: Tracks convolution-specific tensor layout, parameter handling, or fused compute. / **卷积支持**：跟踪卷积特有的张量布局、参数处理或融合计算。

## Dependencies / 依赖关系
- **Standard library**: `__future__.annotations`, `typing.TYPE_CHECKING`, `typing.Optional`, `typing.Tuple`, and `typing.Union` / **标准库**：`__future__.annotations`、`typing.TYPE_CHECKING`、`typing.Optional`、`typing.Tuple` 以及 `typing.Union`
- **Third-party**: `torch` and `torch.nn` / **第三方依赖**：`torch` 和 `torch.nn`
- **Internal SGLang modules**: `sglang.srt.compilation.compilation_config.register_split_op`, `sglang.srt.compilation.piecewise_context_manager.get_forward_context`, `sglang.srt.utils.custom_op.register_custom_op`, and `sglang.srt.model_executor.forward_batch_info.ForwardBatch` / **SGLang 内部模块**：`sglang.srt.compilation.compilation_config.register_split_op`、`sglang.srt.compilation.piecewise_context_manager.get_forward_context`、`sglang.srt.utils.custom_op.register_custom_op` 以及 `sglang.srt.model_executor.forward_batch_info.ForwardBatch`
