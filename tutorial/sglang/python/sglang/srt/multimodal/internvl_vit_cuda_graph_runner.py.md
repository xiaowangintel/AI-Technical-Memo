# internvl_vit_cuda_graph_runner.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/multimodal/internvl_vit_cuda_graph_runner.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Implements an InternVL-specific CUDA graph runner for visual encoder execution. / 实现面向 InternVL 视觉编码执行的专用 CUDA Graph 运行器。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14: Comments and module notes
```python
# Copyright 2023-2026 SGLang Team
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
**EN:** This range contains comments or annotations that document assumptions, provenance, or implementation notes for the module.
**CN:** 该范围包含注释或说明，用于记录 模块 的假设、来源或实现备注。

### Lines 15-15: Documents the scope
```python
"""ViT CUDA Graph Runner class."""
```
**EN:** This string literal serves as documentation for the module, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 模块 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 16-16: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 17-25: Imports dependencies
```python
from __future__ import annotations

from typing import Dict, Hashable, Tuple

import torch
import torch.nn as nn

from sglang.srt.layers.attention.vision import VisionAttention
from sglang.srt.server_args import get_global_server_args
```
**EN:** This block groups related imports for the module, including __future__.annotations, typing.Dict, typing.Hashable, typing.Tuple, torch, and 3 more. It prepares the names required by later code.
**CN:** 该代码块为 模块 集中导入依赖，包括 __future__.annotations, typing.Dict, typing.Hashable, typing.Tuple, torch 等 3 项，为后续代码准备所需名称。

### Lines 26-27: Spacing and separators
```python


```
**EN:** These lines separate logical blocks inside the module, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 模块 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 28-28: Declares class InternViTCudaGraphRunner
```python
class InternViTCudaGraphRunner:
```
**EN:** This block introduces class `InternViTCudaGraphRunner` as a reusable abstraction inside the module. CUDA Graph runner for InternVL vision encoder. Captures: y = layer_N(...layer_2(layer_1(x))) Keyed by (B, S).
**CN:** 该代码块声明类 `InternViTCudaGraphRunner`，作为模块中的可复用抽象。 文档字符串摘要：CUDA Graph runner for InternVL vision encoder. Captures: y = layer_N(...layer_2(layer_1(x))) Keyed by (B, S).

### Lines 29-35: Documents the scope
```python
    """CUDA Graph runner for InternVL vision encoder.

    Captures:
      y = layer_N(...layer_2(layer_1(x)))

    Keyed by (B, S). This is REQUIRED because InternVL uses [B,S,H].
    """
```
**EN:** This string literal serves as documentation for the InternViTCudaGraphRunner, explaining intent or usage without affecting execution.
**CN:** 该字符串字面量作为 InternViTCudaGraphRunner 的文档说明，用于解释意图或用法，但不会影响执行。

### Lines 36-36: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternViTCudaGraphRunner, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternViTCudaGraphRunner 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 37-53: Defines function InternViTCudaGraphRunner.__init__
```python
    def __init__(self, encoder: nn.Module) -> None:
        self.encoder = encoder

        # key -> graph & stable buffers
        self.graphs: Dict[Hashable, torch.cuda.CUDAGraph] = {}
        self.inp: Dict[Hashable, torch.Tensor] = {}
        self.ws: Dict[Hashable, torch.Tensor] = {}
        self.out: Dict[Hashable, torch.Tensor] = {}

        # key -> stable cu_seqlens buffers (addresses must be stable)
        self.cu: Dict[Hashable, torch.Tensor] = {}
        self.cu_kk: Dict[Hashable, torch.Tensor] = {}

        # cache attention metadata
        first_layer = encoder.layers[0]
        # InternAttention wraps VisionAttention as first_layer.attn.attn
        self._attn: VisionAttention = first_layer.attn.attn  # type: ignore
```
**EN:** This block defines function `InternViTCudaGraphRunner.__init__`. Parameters: self, encoder.
**CN:** 该代码块定义函数 `InternViTCudaGraphRunner.__init__`。 参数包括 self、encoder。

### Lines 54-54: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternViTCudaGraphRunner, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternViTCudaGraphRunner 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 55-57: Defines function InternViTCudaGraphRunner.device
```python
    @property
    def device(self) -> torch.device:
        return next(self.encoder.parameters()).device
```
**EN:** This block defines function `InternViTCudaGraphRunner.device`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `InternViTCudaGraphRunner.device`。 参数包括 self。 装饰器包括 property。

### Lines 58-58: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternViTCudaGraphRunner, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternViTCudaGraphRunner 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 59-61: Defines function InternViTCudaGraphRunner.dtype
```python
    @property
    def dtype(self) -> torch.dtype:
        return next(self.encoder.parameters()).dtype
```
**EN:** This block defines function `InternViTCudaGraphRunner.dtype`. Parameters: self. Decorators: property.
**CN:** 该代码块定义函数 `InternViTCudaGraphRunner.dtype`。 参数包括 self。 装饰器包括 property。

### Lines 62-62: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternViTCudaGraphRunner, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternViTCudaGraphRunner 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 63-65: Defines function InternViTCudaGraphRunner._graph_key
```python
    def _graph_key(self, x: torch.Tensor) -> Tuple[int, int]:
        # x: [B,S,H]
        return (x.shape[0], x.shape[1])
```
**EN:** This block defines function `InternViTCudaGraphRunner._graph_key`. Parameters: self, x.
**CN:** 该代码块定义函数 `InternViTCudaGraphRunner._graph_key`。 参数包括 self、x。

### Lines 66-66: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternViTCudaGraphRunner, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternViTCudaGraphRunner 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 67-69: Defines function InternViTCudaGraphRunner._build_cu
```python
    def _build_cu(self, B: int, S: int, device: torch.device) -> torch.Tensor:
        # [0, S, 2S, ..., B*S]
        return torch.arange(0, (B + 1) * S, step=S, device=device, dtype=torch.int32)
```
**EN:** This block defines function `InternViTCudaGraphRunner._build_cu`. Parameters: self, B, S, device.
**CN:** 该代码块定义函数 `InternViTCudaGraphRunner._build_cu`。 参数包括 self、B、S、device。

### Lines 70-70: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternViTCudaGraphRunner, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternViTCudaGraphRunner 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 71-94: Defines function InternViTCudaGraphRunner._alloc_ws
```python
    def _alloc_ws(
        self, B: int, S: int, H: int, device: torch.device, dtype: torch.dtype
    ) -> torch.Tensor:
        # InternVL shape: [tokens, nheads, head_dim]
        tokens = B * S

        num_heads = getattr(self._attn, "num_attention_heads_per_partition", None)
        if num_heads is None:
            num_heads = getattr(self._attn, "num_heads", None)
        if num_heads is None:
            raise RuntimeError("Cannot infer num_heads from VisionAttention")

        head_dim = getattr(self._attn, "head_size", None)
        if head_dim is None:
            # fallback (should rarely happen)
            head_dim = H // int(num_heads)

        return torch.empty(
            tokens,
            int(num_heads),
            int(head_dim),
            device=device,
            dtype=dtype,
        )
```
**EN:** This block defines function `InternViTCudaGraphRunner._alloc_ws`. Parameters: self, B, S, H, device, dtype.
**CN:** 该代码块定义函数 `InternViTCudaGraphRunner._alloc_ws`。 参数包括 self、B、S、H、device、dtype。

### Lines 95-95: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternViTCudaGraphRunner, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternViTCudaGraphRunner 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 96-114: Defines function InternViTCudaGraphRunner._warmup_once
```python
    def _warmup_once(self, key: Hashable) -> None:
        """Run a tiny eager warmup on the preallocated buffers to trigger lazy init."""
        override_backend = get_global_server_args().mm_attention_backend
        cu = self.cu[key]
        cu_kk = self.cu_kk[key]
        max_len = int(cu_kk.max().item()) if cu_kk.numel() else 0

        if override_backend == "triton_attn":
            cu_ws = [cu, cu_kk, max_len]
        elif override_backend == "fa3":
            cu_ws = [cu, max_len]
        else:
            raise RuntimeError("Not supported ViT attention backend for InternVL CG")

        x = self.inp[key]
        y = x
        with torch.no_grad():
            for blk in self.encoder.layers:
                y = blk(y, cu_seqlens=cu_ws, output_ws=self.ws[key])
```
**EN:** This block defines function `InternViTCudaGraphRunner._warmup_once`. Parameters: self, key. Run a tiny eager warmup on the preallocated buffers to trigger lazy init.
**CN:** 该代码块定义函数 `InternViTCudaGraphRunner._warmup_once`。 参数包括 self、key。 文档字符串摘要：Run a tiny eager warmup on the preallocated buffers to trigger lazy init.

### Lines 115-115: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternViTCudaGraphRunner, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternViTCudaGraphRunner 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 116-140: Defines function InternViTCudaGraphRunner._capture_graph
```python
    def _capture_graph(self, key: Hashable) -> None:
        g = torch.cuda.CUDAGraph()
        override_backend = get_global_server_args().mm_attention_backend

        cu = self.cu[key]
        cu_kk = self.cu_kk[key]
        max_len = int(cu_kk.max().item()) if cu_kk.numel() else 0

        if override_backend == "triton_attn":
            cu_ws = [cu, cu_kk, max_len]
        elif override_backend == "fa3":
            cu_ws = [cu, max_len]
        else:
            raise RuntimeError("Not supported ViT attention backend for InternVL CG")

        torch.cuda.synchronize()

        with torch.cuda.graph(g):
            y = self.inp[key]
            for blk in self.encoder.layers:
                y = blk(y, cu_seqlens=cu_ws, output_ws=self.ws[key])
            # y is a stable output tensor produced during capture; keep reference
            self.out[key] = y

        self.graphs[key] = g
```
**EN:** This block defines function `InternViTCudaGraphRunner._capture_graph`. Parameters: self, key.
**CN:** 该代码块定义函数 `InternViTCudaGraphRunner._capture_graph`。 参数包括 self、key。

### Lines 141-141: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternViTCudaGraphRunner, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternViTCudaGraphRunner 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 142-169: Defines function InternViTCudaGraphRunner.create_graph
```python
    def create_graph(self, x: torch.Tensor) -> Hashable:
        # x: [B, S, H]
        x = x.contiguous()
        key = self._graph_key(x)
        if key in self.graphs:
            return key

        B, S, H = x.shape
        device = x.device
        dtype = x.dtype

        # stable input buffer
        self.inp[key] = torch.empty_like(x, device=device).contiguous()

        # stable cu buffers
        cu = self._build_cu(B, S, device=device)
        self.cu[key] = cu
        self.cu_kk[key] = cu[1:] - cu[:-1]

        # stable attention workspace
        self.ws[key] = self._alloc_ws(B, S, H, device=device, dtype=dtype)

        self.inp[key].copy_(x)
        self._warmup_once(key)

        # capture
        self._capture_graph(key)
        return key
```
**EN:** This block defines function `InternViTCudaGraphRunner.create_graph`. Parameters: self, x.
**CN:** 该代码块定义函数 `InternViTCudaGraphRunner.create_graph`。 参数包括 self、x。

### Lines 170-170: Spacing and separators
```python

```
**EN:** These lines separate logical blocks inside the InternViTCudaGraphRunner, improving readability without changing runtime behavior.
**CN:** 这些行用于分隔 InternViTCudaGraphRunner 内的逻辑代码块，提升可读性，但不改变运行时行为。

### Lines 171-184: Defines function InternViTCudaGraphRunner.run
```python
    def run(self, x: torch.Tensor) -> torch.Tensor:
        # x: [B, S, H]
        x = x.contiguous()
        key = self._graph_key(x)
        if key not in self.graphs:
            self.create_graph(x)

        # update input content (address stable)
        self.inp[key].copy_(x)

        # replay
        self.graphs[key].replay()

        return self.out[key]
```
**EN:** This block defines function `InternViTCudaGraphRunner.run`. Parameters: self, x.
**CN:** 该代码块定义函数 `InternViTCudaGraphRunner.run`。 参数包括 self、x。

## Key Concepts / 关键概念
- **Classes / 类**: `InternViTCudaGraphRunner`

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `__future__`, `typing`
- **Third-Party / 第三方**: `torch`, `torch.nn`
- **Local Modules / 本地模块**: `sglang.srt.layers.attention.vision`, `sglang.srt.server_args`
